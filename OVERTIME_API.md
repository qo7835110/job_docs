# 加班 / 補班申請 API 文件

> 最後更新：2026-05-13

## 概述

提供員工加班申請與雇主補班指派功能。所有 API 均需使用 Sanctum Token 認證。

**認證 Header：**
```
Authorization: Bearer {token}
```

---

## 核心概念

### 申請類型（request_type）

| request_type | 名稱 | 說明 |
|---|---|---|
| **0** | 加班 | 排班日內超時出勤，需綁定已存在的 Task |
| **1** | 補班 | 非排班日（休息日/例假日）全天出勤，核准後系統自動建立 WorkDay + Task |

### 審核狀態（status）

| status | 名稱 | 說明 |
|---|---|---|
| **0** | pending | 待審核（僅此狀態可編輯/撤銷） |
| **1** | approved | 已核准（薪資結算時納入計算） |
| **2** | rejected | 已拒絕 |

### 加班計薪規則

- **只有 status = 1（approved）** 的申請才納入 `SalarySettlementService` 加班費計算
- **只有 request_type = 0（加班）** 的申請計入加班費，補班（type=1）依 WorkDay.day_tag 費率計薪
- 計薪分鐘數：`actual_minutes` 優先，不存在則採 `requested_minutes`

### 補班核准流程

```
核准補班申請
  └─ OvertimeRequestService::createMakeupWorkDayAndTask()
       ├─ detectDayTag(job.working, 國定假日表)
       │    公假(4) > li_rest_day(1) > sho_rest_day(2) > 預設(2)
       ├─ WorkDay::updateOrCreate(contract_id, work_date)
       │    period_id = null（不屬於任何排班週期）
       └─ Task::create(work_day_id = workDay.id)
            └─ 員工直接打卡即可
```

### 加班時段驗證規則（request_type=0）

| 規則 | 說明 |
|---|---|
| start ≥ task.end_at | 加班開始不得早於任務預定結束時間（防止與正常工時重疊） |
| start ≤ task.end_at + 24hr | 防止拿舊任務申請未來日期的加班（夜班緩衝 24hr） |
| end > start | 時段必須正向 |
| 正常工時 + 加班工時 ≤ 720 分鐘 | 當日總工時上限 12 小時（勞基法第 32 條） |

---

## API 端點

### GET `/v1/overtime-requests`

取得加班/補班申請列表。

> 同一個 user 可能同時扮演雇主與員工，透過 `role` 參數明確指定查詢視角。

#### Query 參數

| 參數 | 類型 | 說明 |
|---|---|---|
| role | string | ✅ 必填：`employer`=雇主視角（owner_id=我）、`employee`=員工視角（employee_id=我） |
| request_type | integer | 篩選類型：`0`=加班、`1`=補班 |
| status | integer | 篩選狀態：`0`=待審、`1`=核准、`2`=拒絕 |
| task_id | integer | 篩選任務 |
| start_date | date | 申請開始日 >= |
| end_date | date | 申請結束日 <= |

#### 回應範例 (200)

```json
{
  "msg": "success",
  "res": {
    "data": {
      "current_page": 1,
      "data": [
        {
          "id": 1,
          "request_type": 0,
          "status": 0,
          "task_id": 123,
          "job_id": 10,
          "employee_id": 456,
          "owner_id": 1,
          "initiated_by": 456,
          "work_date": null,
          "requested_start_at": "2026-05-13 18:00:00",
          "requested_end_at": "2026-05-13 20:00:00",
          "requested_minutes": 120,
          "actual_start_at": null,
          "actual_end_at": null,
          "actual_minutes": null,
          "reason": "專案趕工",
          "approved_at": null,
          "approved_by": null,
          "created_task_id": null,
          "reject_reason": null,
          "employee": { "id": 456, "name": "王小明" },
          "task": { "id": 123 }
        }
      ],
      "per_page": 20,
      "total": 1
    }
  }
}
```

---

### GET `/v1/overtime-requests/{id}`

取得單筆申請詳情。

> 僅限申請人（employee_id）或雇主（owner_id）存取。

#### 回應範例 (200)

```json
{
  "msg": "success",
  "res": {
    "data": {
      "id": 1,
      "request_type": 1,
      "status": 1,
      "task_id": 200,
      "job_id": 10,
      "employee_id": 456,
      "owner_id": 1,
      "work_date": "2026-05-17",
      "requested_start_at": "2026-05-17 09:00:00",
      "requested_end_at": "2026-05-17 18:00:00",
      "requested_minutes": 540,
      "approved_at": "2026-05-13 15:00:00",
      "approved_by": 1,
      "created_task_id": 200,
      "employee": { "id": 456, "name": "王小明" },
      "owner": { "id": 1, "name": "陳老闆" },
      "approvedBy": { "id": 1, "name": "陳老闆" },
      "task": { "id": 200 }
    }
  }
}
```

#### 錯誤回應

| 狀態碼 | msg | 原因 |
|---|---|---|
| 404 | overtime request not found | 申請不存在 |
| 403 | unauthorized | 非申請人或雇主 |

---

### POST `/v1/overtime-requests`

建立加班或補班申請。

#### 共用參數

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| request_type | integer | ❌ | `0`=加班（預設）、`1`=補班 |
| requested_start_at | datetime | ✅ | 申請開始時間 |
| requested_end_at | datetime | ✅ | 申請結束時間（需晚於開始） |
| reason | string | ❌ | 申請事由（max 500） |

#### 加班（request_type=0）專用參數

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| task_id | integer | ✅ | 已排班的任務 ID（申請人必須是該任務員工） |
| actual_start_at | datetime | ❌ | 實際加班開始（B 方案事後申報） |
| actual_end_at | datetime | ❌ | 實際加班結束（B 方案事後申報） |

#### 補班（request_type=1）專用參數

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| job_id | integer | ✅ | 職位 ID |
| employee_id | integer | ❌ | 員工 user_id（雇主代為申請時必填；員工自行申請可省略） |
| work_date | date | ✅ | 補班日期，格式 `YYYY-MM-DD` |

#### 加班申請範例

```bash
curl -X POST "https://example.com/v1/overtime-requests" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "request_type": 0,
    "task_id": 123,
    "requested_start_at": "2026-05-13 18:00:00",
    "requested_end_at": "2026-05-13 20:00:00",
    "reason": "專案趕工"
  }'
```

#### 補班申請範例（雇主指派）

```bash
curl -X POST "https://example.com/v1/overtime-requests" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "request_type": 1,
    "job_id": 10,
    "employee_id": 456,
    "work_date": "2026-05-17",
    "requested_start_at": "2026-05-17 09:00:00",
    "requested_end_at": "2026-05-17 18:00:00",
    "reason": "五一補班"
  }'
```

#### 回應範例 (201 Created)

```json
{
  "msg": "overtime request created",
  "res": {
    "data": {
      "id": 1,
      "request_type": 0,
      "status": 0,
      "task_id": 123,
      "employee": { "id": 456, "name": "王小明" },
      "owner": { "id": 1, "name": "陳老闆" }
    }
  }
}
```

#### 錯誤回應

| 狀態碼 | msg | 原因 |
|---|---|---|
| 422 | validation error | 欄位驗證失敗 |
| 404 | task not found | 任務不存在（type=0） |
| 403 | unauthorized: only task employee can apply | 非任務員工（type=0） |
| 400 | task already settled | 任務已結算（type=0） |
| 422 | requested time range is invalid | 計算時距為 0 |
| 422 | requested time range cannot start before task end time | 加班時段與正常工時重疊 |
| 422 | requested time range start time is too far from task end time (max +24h) | 加班起始超過任務結束 24hr |
| 422 | total work hours (normal + overtime) cannot exceed 12 hours | 當日總工時超過 12 小時 |
| 404 | job not found | 職位不存在（type=1） |
| 403 | unauthorized | 非雇主也非員工本人（type=1） |
| 400 | no active contract found for this employee | 員工無有效合約（type=1） |
| 409 | duplicate makeup request: a pending/approved request already exists for this date | 同日已有補班申請（type=1） |
| 500 | create failed | 伺服器錯誤 |

---

### PUT `/v1/overtime-requests/{id}`

修改待審申請（status=0）。

> 僅申請人（employee_id）可修改，且狀態必須為 `pending`。

#### 參數

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| requested_start_at | datetime | ✅ | 新的申請開始時間 |
| requested_end_at | datetime | ✅ | 新的申請結束時間 |
| reason | string | ❌ | 申請事由 |
| work_date | date | ❌ | 新的補班日期（type=1 專用） |
| actual_start_at | datetime | ❌ | 新的實際加班開始（type=0 B 方案） |
| actual_end_at | datetime | ❌ | 新的實際加班結束（type=0 B 方案） |

#### 回應範例 (200)

```json
{
  "msg": "overtime request updated",
  "res": {
    "data": { ...更新後的申請物件... }
  }
}
```

#### 錯誤回應

| 狀態碼 | msg | 原因 |
|---|---|---|
| 404 | overtime request not found | 申請不存在 |
| 403 | unauthorized | 非申請人 |
| 400 | only pending requests can be updated | 非待審狀態 |
| 400 | task not found for overtime request | 找不到綁定任務（type=0） |
| 422 | 時段驗證錯誤 | 同 store 的加班驗證規則 |

---

### DELETE `/v1/overtime-requests/{id}`

撤銷待審申請。

> 申請人（employee_id）或雇主（owner_id）均可撤銷，狀態必須為 `pending`。

#### 回應範例 (200)

```json
{
  "msg": "request cancelled",
  "res": {
    "data": null
  }
}
```

---

### POST `/v1/overtime-requests/{id}/approve`

核准申請（雇主專用）。

> 使用 `SELECT ... FOR UPDATE` 鎖定，防止雙擊重複核准。

#### 參數（選填）

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| actual_start_at | datetime | ❌ | 修正實際開始時間（雇主調整） |
| actual_end_at | datetime | ❌ | 修正實際結束時間（雇主調整） |

> 帶入 actual_* 時，`actual_minutes` 自動計算並覆寫，作為後續計薪依據。

#### 補班核准後自動建立

| 步驟 | 說明 |
|---|---|
| 1. detectDayTag | 依 job.working（li_rest_day/sho_rest_day）+ 國定假日表判斷日類型 |
| 2. WorkDay 建立 | period_id=null，contract_id=員工有效合約，day_tag=判斷結果 |
| 3. Task 建立 | 綁定 WorkDay，員工憑此 Task 打卡 |
| 4. 回填 | overtime_request.task_id 和 created_task_id 更新為新 Task ID |

#### 回應範例 (200)

```json
{
  "msg": "overtime request approved",
  "res": {
    "data": {
      "id": 1,
      "status": 1,
      "approved_at": "2026-05-13 15:00:00",
      "approved_by": 1,
      "task_id": 200,
      "created_task_id": 200,
      "employee": { "id": 456, "name": "王小明" },
      "approvedBy": { "id": 1, "name": "陳老闆" },
      "task": { "id": 200 }
    }
  }
}
```

#### 錯誤回應

| 狀態碼 | msg | 原因 |
|---|---|---|
| 404 | overtime request not found | 申請不存在 |
| 403 | unauthorized: only employer can approve | 非雇主 |
| 400 | only pending requests can be approved | 非待審狀態 |
| 400 | task not found for overtime request | 找不到任務（type=0 修正 actual_* 時） |
| 422 | actual time range ... | 修正時段不合規則（type=0） |
| 400 | work day already has a task assigned | 補班日已有任務綁定（type=1） |
| 400 | work day is already a scheduled working day | 補班日本身就是工作日（type=1） |
| 400 | no active contract found | 找不到有效合約（type=1） |
| 500 | approve failed | 伺服器錯誤 |

---

### POST `/v1/overtime-requests/{id}/reject`

拒絕申請（雇主專用）。

#### 參數

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| reject_reason | string | ❌ | 拒絕原因（max 500） |

#### 回應範例 (200)

```json
{
  "msg": "overtime request rejected",
  "res": {
    "data": {
      "id": 1,
      "status": 2,
      "reject_reason": "人力充足，無需補班",
      "approved_at": "2026-05-13 16:00:00",
      "approved_by": 1
    }
  }
}
```

---

## 資料欄位說明

### overtime_requests 欄位

| 欄位 | 類型 | 說明 |
|---|---|---|
| id | integer | 主鍵 |
| request_type | integer | `0`=加班 `1`=補班 |
| task_id | integer\|null | 對應任務 ID（加班申請必填；補班核准後回填） |
| job_id | integer\|null | 職位 ID（補班必填） |
| employee_id | integer | 員工 user_id |
| owner_id | integer | 雇主 user_id |
| initiated_by | integer\|null | 申請發起者 user_id（員工或雇主皆可） |
| work_date | date\|null | 補班日期（type=1 必填） |
| requested_start_at | datetime | 申請開始時間 |
| requested_end_at | datetime | 申請結束時間 |
| requested_minutes | integer | 申請分鐘數（系統計算） |
| actual_start_at | datetime\|null | 實際開始時間（事後申報或雇主修正） |
| actual_end_at | datetime\|null | 實際結束時間 |
| actual_minutes | integer\|null | 實際分鐘數（計薪依據，優先於 requested_minutes） |
| reason | string\|null | 申請事由 |
| status | integer | `0`=待審 `1`=核准 `2`=拒絕 |
| approved_at | datetime\|null | 審核時間 |
| approved_by | integer\|null | 審核者 user_id |
| created_task_id | integer\|null | 補班核准後自動建立的 Task ID |
| reject_reason | string\|null | 拒絕原因 |

### day_tag 對應費率（補班核准後的 WorkDay）

| day_tag | 名稱 | 費率規則 |
|---|---|---|
| 0 | 工作日 | 依一般工時計算（補班流程不應產生此值） |
| 1 | 例假日 | 法規上原則不得出勤，出勤費率最高 |
| 2 | 休息日 | 前 2hr ×1.34、2–8hr ×1.67、8hr 後 ×2.67 |
| 4 | 國定假日 | 時薪制×2、月薪制另加一日薪 |

---

## 狀態碼

| 狀態碼 | 說明 |
|---|---|
| 200 | 成功 |
| 201 | 建立成功 |
| 400 | 業務邏輯錯誤（合約、重複申請、工時衝突等） |
| 403 | 無操作權限 |
| 404 | 資源不存在 |
| 409 | 資源衝突（重複補班申請） |
| 422 | 欄位驗證失敗 |
| 500 | 伺服器錯誤 |

---

## 與薪資結算的關係

`SalarySettlementService::calculateByPeriod()` 在批次結算時透過 Eager Loading 預載核准的加班申請：

```php
'overtimeRequests' => function ($query) {
    $query->where('status', STATUS_APPROVED)
          ->where('request_type', REQUEST_TYPE_OVERTIME); // 只載入加班，補班不計加班費
}
```

`resolveApprovedOvertimeHours()` 計算可計薪的加班分鐘數：
- 有 `actual_minutes` → 採用 actual_minutes
- 無 → 採用 requested_minutes
- 無任何核准申請 → 回傳 0（向後相容舊資料：依打卡時間計算）
