# 請假記錄 & 假別 API 文件

> 最後更新：2026-02-26

## 概述

提供假別管理（LeaveType）與請假記錄管理（LeaveLog）功能。所有 API 均需使用 Sanctum Token 認證。

**認證 Header：**
```
Authorization: Bearer {token}
```

---

## 核心概念

### 假別擁有者（owner_id）

| owner_id | 意義 |
|---|---|
| **0** | 系統模板假別（由 Seeder 建立，全公司共用，不可刪除） |
| **> 0** | 該 User ID 自訂的假別（僅限該公司使用） |

### 假別額度單位

所有額度以**分鐘（minutes）**儲存，而非天數。原因是變形工時下每日班表工時不固定（8h / 10h / 12h），無法以固定天數換算。

### 計算流程（store / update）

```
1. 驗證請求欄位
2. 確認 task 存在 → 確認 Auth::user() 是該 task 的擁有者（job owner）
3. 確認假別屬於該擁有者或為系統模板
4. buildLeavePayload()：計算 leave_minutes / leave_hours / paid_minutes / day_tag
5. LeaveLog::create()
6. LeaveBalanceService::applyLeaveLog() → 扣減 LeaveBalance.used_minutes（遲到/曠職跳過）
```

---

## 假別類型管理 (Leave Types)

### GET `/v1/leave-types`

取得當前使用者的假別列表（自訂 + 系統模板）。

#### 回應範例 (200)
```json
{
  "msg": "success",
  "res": [
    {
      "id": 1,
      "name": "特別休假",
      "days": 0,
      "quota_minutes": 0,
      "refresh_cycle": "yearly",
      "quota_period": null,
      "gender_limit": null,
      "min_seniority_months": 6,
      "salary_percentage": 100,
      "description": "依年資（3~30天），系統自動推算",
      "owner_id": 0,
      "is_default": true
    },
    {
      "id": 5,
      "name": "情緒假",
      "days": 3,
      "quota_minutes": 1440,
      "refresh_cycle": "yearly",
      "quota_period": null,
      "gender_limit": null,
      "min_seniority_months": 3,
      "salary_percentage": 100,
      "description": "公司自訂：滿 3 個月可請",
      "owner_id": 123,
      "is_default": false
    }
  ]
}
```

#### 欄位說明
| 欄位 | 類型 | 說明 |
|---|---|---|
| id | integer | 假別 ID |
| name | string | 假別名稱 |
| days | integer | 可請天數（參考用；系統以 quota_minutes 為準） |
| quota_minutes | integer | 額度上限（分鐘），0 表示無上限或系統動態計算 |
| refresh_cycle | string | `yearly`＝每年、`monthly`＝每月、`once`＝一次性 |
| quota_period | string\|null | 自訂額度週期描述 |
| gender_limit | integer\|null | `null`＝不限、`1`＝男性、`2`＝女性 |
| min_seniority_months | integer | 最低年資門檻（月），0 表示無門檻 |
| salary_percentage | integer | 薪資比例：`100`＝全薪、`50`＝半薪、`0`＝無薪 |
| owner_id | integer | `0`＝系統模板，其餘為自訂假別擁有者 User ID |
| is_default | boolean | 是否為系統預設假別 |

---

### POST `/v1/leave-types`

新增自訂假別。

#### 請求參數
| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| name | string | ✅ | 假別名稱（同一 owner 下不可重複） |
| days | integer | ✅ | 可請天數（min: 0） |
| quota_minutes | integer | ❌ | 額度分鐘數（未填時以 days × 8 × 60 計算） |
| refresh_cycle | string | ✅ | `yearly` / `monthly` / `once` |
| salary_percentage | integer | ✅ | 0–100 |
| gender_limit | integer | ❌ | `null` / `1` / `2` |
| min_seniority_months | integer | ❌ | 最低年資門檻（月），預設 0 |
| description | string | ❌ | 假別說明 |

#### 回應範例 (200)
```json
{
  "msg": "建立成功",
  "res": { ...假別物件... }
}
```

---

### PUT `/v1/leave-types/{id}`

更新指定假別（需為該假別擁有者）。

---

### DELETE `/v1/leave-types/{id}`

刪除指定假別（系統模板不可刪除）。

---

## 請假記錄管理 (Leave Logs)

### GET `/v1/leave-logs`

取得請假記錄列表（管理員用）。

#### Query 參數
| 參數 | 類型 | 說明 |
|---|---|---|
| employee_id | integer | 篩選員工 |
| task_id | integer | 篩選任務 |
| leave_type_id | integer | 篩選假別 |
| start_date | date | 請假開始日 >= |
| end_date | date | 請假結束日 <= |

#### 回應範例 (200)
```json
{
  "msg": "success",
  "res": {
    "data": [
      {
        "id": 1,
        "task_id": 123,
        "employee_id": 456,
        "owner_id": 1,
        "leave_type_id": 2,
        "start_at": "2026-02-01 09:00:00",
        "end_at": "2026-02-01 18:00:00",
        "leave_hours": 9,
        "leave_minutes": 540,
        "paid_minutes": 270,
        "day_tag": 0,
        "reason": "身體不適",
        "employee": { "id": 456, "name": "王小明" },
        "leaveType": { "id": 2, "name": "普通傷病假" },
        "task": { "id": 123 }
      }
    ],
    "current_page": 1,
    "per_page": 20
  }
}
```

---

### GET `/v1/leave-logs/my`

取得當前登入使用者的請假記錄（分頁，每頁 20 筆，依 start_at 倒序）。

---

### GET `/v1/leave-logs/employee/{employeeId}`

取得指定員工的請假記錄。

---

### GET `/v1/leave-logs/task/{taskId}`

取得指定任務的所有請假記錄（不分頁，回傳完整列表）。

---

### GET `/v1/leave-logs/{id}`

取得單筆請假記錄詳情。

---

### POST `/v1/leave-logs/create`

**建立請假記錄。**

> 登入者必須是該 `task` 對應 Job 的擁有者（雇主），才有權限為員工建立請假記錄。

#### 請求參數
| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| task_id | integer | ✅ | 任務 ID |
| leave_type_id | integer | ✅ | 假別 ID（系統模板或屬於同一 owner） |
| start_at | datetime | ✅ | 開始時間，格式 `YYYY-MM-DD HH:mm:ss` |
| end_at | datetime | ✅ | 結束時間，需晚於 start_at |
| reason | string | ❌ | 請假原因 |

#### 請求範例
```bash
curl -X POST "https://example.com/v1/leave-logs/create" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "task_id": 123,
    "leave_type_id": 2,
    "start_at": "2026-02-01 09:00:00",
    "end_at": "2026-02-01 18:00:00",
    "reason": "身體不適"
  }'
```

#### 回應範例 (201 Created)
```json
{
  "msg": "leave log created successfully",
  "res": {
    "id": 1,
    "task_id": 123,
    "employee_id": 456,
    "owner_id": 1,
    "leave_type_id": 2,
    "start_at": "2026-02-01 09:00:00",
    "end_at": "2026-02-01 18:00:00",
    "leave_hours": 9,
    "leave_minutes": 540,
    "paid_minutes": 270,
    "day_tag": 0,
    "reason": "身體不適",
    "employee": { "id": 456, "name": "王小明" },
    "leaveType": { "id": 2, "name": "普通傷病假", "salary_percentage": 50 },
    "task": { "id": 123 }
  }
}
```

#### 自動計算欄位說明
| 欄位 | 計算方式 |
|---|---|
| employee_id | 從 task.user_id 自動帶入 |
| owner_id | 從 task → job → user_id 自動帶入 |
| leave_minutes | `end_at` 與 `start_at` 的實際分鐘差（不設最低門檻） |
| leave_hours | `ceil(leave_minutes / 60)` |
| paid_minutes | `floor(leave_minutes × salary_percentage / 100)` |
| day_tag | 從 WorkDay 或 Job.working 設定推算（見下表） |

#### day_tag 說明
| 值 | 意義 |
|---|---|
| 0 | 一般工作日（或無法判斷） |
| 1 | 例假日（Legal Holiday） |
| 2 | 休息日（Rest Day） |

#### 錯誤回應
| 狀態碼 | msg | 原因 |
|---|---|---|
| 422 | validation error | 欄位驗證失敗 |
| 404 | task not found | 任務不存在 |
| 403 | unauthorized to add leave log for this task | 登入者不是該任務的 Job Owner |
| 400 | invalid leave type for this task owner | 假別不屬於該 owner 且非系統模板 |
| 400 | insufficient leave balance | 假別餘額不足 |
| 400 | contract not found | 找不到對應的勞動合約 |
| 500 | leave log create failed | 伺服器錯誤 |

---

### POST `/v1/leave-logs/update/{id}`

**更新請假記錄。**

> 登入者必須是更新後 `task_id` 對應 Job 的擁有者。系統會先釋放原有 LeaveBalance 扣除量，再重新套用新的扣除量。

#### 請求參數

與 `store` 相同（task_id、leave_type_id、start_at、end_at、reason）。

#### 回應範例 (200)
```json
{
  "msg": "leave log updated successfully",
  "res": { ...更新後的請假記錄... }
}
```

#### 錯誤回應
| 狀態碼 | msg | 原因 |
|---|---|---|
| 404 | leave log not found | 記錄不存在 |
| 422 | validation error | 欄位驗證失敗 |
| 404 | task not found | 任務不存在 |
| 403 | unauthorized to update leave log for this task | 登入者不是 Job Owner |
| 404 | leave type not found | 假別不存在 |
| 400 | invalid leave type for this task owner | 假別歸屬錯誤 |
| 400 | insufficient leave balance | 餘額不足 |

---

### DELETE `/v1/leave-logs/{id}`

刪除請假記錄，同時回滾 LeaveBalance 已扣除的分鐘數。

#### 回應範例 (200)
```json
{
  "msg": "leave log deleted successfully",
  "res": null
}
```

> **注意：** 若關聯的 Task 已被刪除，LeaveBalance 無法回滾（找不到合約），但刪除仍會繼續執行。

---

### GET `/v1/leave-logs/statistics/{employeeId}`

取得指定員工的年度請假統計（依 LeaveBalance 個人實際額度計算，PT 員工已套比例折算）。

#### Query 參數
| 參數 | 類型 | 預設 | 說明 |
|---|---|---|---|
| year | integer | 當年 | 統計年度 |

#### 回應範例 (200)
```json
{
  "msg": "success",
  "res": {
    "year": 2026,
    "employee_id": 456,
    "statistics": [
      {
        "leave_type_id": 2,
        "leave_type_name": "普通傷病假",
        "total_minutes": 960,
        "total_hours": 16.0,
        "available_minutes": 14400,
        "used_minutes": 960,
        "remaining_minutes": 13440,
        "count": 2
      },
      {
        "leave_type_id": 5,
        "leave_type_name": "遲到",
        "total_minutes": 30,
        "total_hours": 0.5,
        "available_minutes": 0,
        "used_minutes": 30,
        "remaining_minutes": 0,
        "count": 3
      }
    ]
  }
}
```

#### 統計欄位說明
| 欄位 | 說明 |
|---|---|
| total_minutes | 本年度已請假分鐘數（所有 leave_logs 加總） |
| total_hours | `total_minutes / 60`（四捨五入 2 位） |
| available_minutes | 該員工本週期的總額度（來自 LeaveBalance，含結轉分鐘） |
| used_minutes | LeaveBalance 記錄的已使用分鐘數 |
| remaining_minutes | `available_minutes - used_minutes`（最小 0） |
| count | 請假筆數 |

> **遲到 / 曠職** 不建立 LeaveBalance，`available_minutes` 以 `quota_minutes` 或 `days × 8 × 60` 顯示（通常為 0）。

---

## LeaveBalance（假別餘額）說明

每次建立或更新請假記錄時，系統會透過 `LeaveBalanceService` 自動維護 `leave_balances` 資料表。

### 跳過扣減的假別

下列假別**不扣減 LeaveBalance**，僅作記錄用途：
- 遲到
- 曠職

### 额度不足處理

若 `remaining < leave_minutes`，API 回傳 `400 insufficient leave balance`，整筆交易 Rollback。

### 部分工時（PT）折算邏輯

| 假別類型 | 折算公式 |
|---|---|
| 特別休假 | `前一年實際工時 / 2088h × 法定天數 × 8h` |
| 其他假別 | `(平均每週工時 / 40h) × 法定天數 × 8h` |

---

## 系統預設假別一覽

由 `LeaveTypesTableSeeder`（owner_id = 0）建立，所有公司共用：

| 名稱 | 天數 | 薪資 | 週期 | 性別限制 | 年資門檻 |
|---|---|---|---|---|---|
| 特別休假 | 依年資 | 全薪 | yearly | 無 | 6 個月 |
| 普通傷病假 | 30 | 半薪 | yearly | 無 | 無 |
| 生理假 | 1/月 | 半薪 | monthly | 女性 | 無 |
| 事假 | 14 | 無薪 | yearly | 無 | 無 |
| 婚假 | 8 | 全薪 | once | 無 | 無 |
| 喪假 | 8 | 全薪 | once | 無 | 無 |
| 產假 | 56 | 全薪* | once | 女性 | 無 |
| 陪產檢及陪產假 | 7 | 全薪 | once | 男性 | 無 |
| 公假 | 依需要 | 全薪 | once | 無 | 無 |
| 遲到 | — | 無薪 | — | 無 | 無 |
| 曠職 | — | 無薪 | — | 無 | 無 |

> *產假薪資：年資滿半年全薪；未滿半年半薪，實際薪資比於結算時依合約判斷。

---

## 狀態碼

| 狀態碼 | 說明 |
|---|---|
| 200 | 成功 |
| 201 | 建立成功 |
| 400 | 請求邏輯錯誤（餘額不足、假別歸屬錯誤、找不到合約等） |
| 403 | 無操作權限 |
| 404 | 資源不存在 |
| 422 | 欄位驗證失敗 |
| 500 | 伺服器錯誤 |
