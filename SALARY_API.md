# 薪資管理 API

> 所有路由均需 `Authorization: Bearer {token}`（Sanctum）。

---

## 目錄

| 方法 | 路徑 | 說明 |
|------|------|------|
| GET | `/v1/salary/monthly` | 雇主查詢所有員工月薪摘要 |
| GET | `/v1/salary/monthly/{userId}` | 查詢特定員工月薪明細 |
| GET | `/v1/task/{id}/settlement/slip` | 取得單一任務薪資結算單 |
| POST | `/v1/salary/period/preview` | 週期薪資試算（dry-run，不寫入 DB） |
| POST | `/v1/salary/period/checkout` | 週期薪資正式結算（寫入 DB） |

---

## 1. 月薪摘要列表

### `GET /v1/salary/monthly`

取得當前登入雇主旗下所有員工，在指定年月的薪資摘要。

#### Query Parameters

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| `year` | integer | 否 | 年份（2000–2100），預設當年 |
| `month` | integer | 否 | 月份（1–12），預設當月 |

#### 成功回應 `200 OK`

```json
{
  "msg": "success",
  "res": [
    {
      "user_id": 10,
      "user_name": "陳大明",
      "job_id": 3,
      "job_title": "後端工程師",
      "year": 2026,
      "month": 2,
      "total_pay": 30000.00,
      "total_ot_pay": 1500.00,
      "total_sum_pay": 31500.00,
      "total_work_hours": 160.00,
      "total_ot_hours": 8.00,
      "health_in": 450.00,
      "labor_in": 318.00,
      "leave_deduction": 0.00,
      "absence_deduction": 0.00
    }
  ]
}
```

#### 回應欄位說明

| 欄位 | 類型 | 說明 |
|------|------|------|
| `user_id` | integer | 員工 ID |
| `user_name` | string\|null | 員工姓名 |
| `job_id` | integer | 職缺 ID |
| `job_title` | string\|null | 職缺名稱 |
| `year` | integer | 年份 |
| `month` | integer | 月份 |
| `total_pay` | number | 底薪合計 |
| `total_ot_pay` | number | 加班費合計 |
| `total_sum_pay` | number | 薪資總計（底薪 + 加班費） |
| `total_work_hours` | number | 總工作時數 |
| `total_ot_hours` | number | 總加班時數 |
| `health_in` | number | 健保費（員工負擔） |
| `labor_in` | number | 勞保費（員工負擔） |
| `leave_deduction` | number | 請假扣款 |
| `absence_deduction` | number | 曠職扣款 |

#### 錯誤回應

| HTTP | msg | 說明 |
|------|-----|------|
| `401` | Unauthenticated | 未登入 |
| `500` | query failed | 查詢異常 |

---

## 2. 單一員工月薪明細

### `GET /v1/salary/monthly/{userId}`

取得雇主旗下特定員工的多筆月薪紀錄。支援年月篩選，不指定時回傳全部歷史。

#### Path Parameters

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| `userId` | integer | 是 | 員工 ID |

#### Query Parameters

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| `year` | integer | 否 | 年份篩選（2000–2100） |
| `month` | integer | 否 | 月份篩選（1–12） |

#### 成功回應 `200 OK`

```json
{
  "msg": "success",
  "res": [
    {
      "id": 5,
      "user_id": 10,
      "job_id": 3,
      "owner_id": 1,
      "company_id": 2,
      "year": 2026,
      "month": 2,
      "total_pay": 30000.00,
      "total_ot_pay": 1500.00,
      "total_sum_pay": 31500.00,
      "pf_attendance": 0.00,
      "bonus": 0.00,
      "health_in": 450.00,
      "labor_in": 318.00,
      "others": 0.00,
      "leave_deduction": 0.00,
      "absence_deduction": 0.00,
      "total_work_hours": 160.00,
      "total_ot_hours": 8.00,
      "created_at": "2026-02-28 10:00:00",
      "updated_at": "2026-02-28 10:00:00"
    }
  ]
}
```

#### 無資料回應 `200 OK`

```json
{
  "msg": "no records found",
  "res": []
}
```

#### 錯誤回應

| HTTP | msg | 說明 |
|------|-----|------|
| `401` | Unauthenticated | 未登入 |
| `500` | query failed | 查詢異常 |

---

## 3. 任務薪資結算單

### `GET /v1/task/{id}/settlement/slip`

取得已結算任務的薪資結算單（dry-run 重計，不更動 DB）。雇主或對應員工均可查詢。

#### Path Parameters

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| `id` | integer | 是 | 任務 ID |

#### 成功回應 `200 OK`

```json
{
  "msg": "success",
  "res": {
    "task_id": 42,
    "settlement_at": "2026-02-28 10:00:00",
    "pay_way": 1,
    "wage": 30000.00,
    "ot_pay": 1500.00,
    "total": 31500.00,
    "work_time": 8.00,
    "ot_time": 1.00,
    "late_hours": 0.00,
    "absence_hours": 0.00,
    "leave_hours": 0.00,
    "leave_deduction": 0.00,
    "absence_deduction": 0.00,
    "labor_in": 318.00,
    "health_in": 450.00
  }
}
```

#### 錯誤回應

| HTTP | msg | 說明 |
|------|-----|------|
| `400` | task not settled yet | 任務尚未結算（`fee ≠ 1`） |
| `401` | Unauthenticated | 未登入 |
| `403` | unauthorized | 非雇主或員工本人 |
| `404` | task not found | 任務不存在 |
| `500` | failed to retrieve slip | 計算異常 |

---

## 4. 週期薪資試算（Dry-run）

### `POST /v1/salary/period/preview`

依指定日期範圍試算薪資，不寫入資料庫。結算週期（日結 / 週結 / 月結）由系統從 `job.wage.checkout_cycle` 自動讀取。

#### Request Body（JSON）

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| `job_id` | integer | 是 | 職缺 ID（需存在於 `jobs` 表） |
| `user_id` | integer | 是 | 員工 ID（需存在於 `users` 表） |
| `period_from` | string | 是 | 結算起始日（格式：`Y-m-d`） |
| `period_to` | string | 是 | 結算截止日（格式：`Y-m-d`，需 ≥ `period_from`） |

```json
{
  "job_id": 3,
  "user_id": 10,
  "period_from": "2026-02-01",
  "period_to": "2026-02-28"
}
```

#### 成功回應 `200 OK`

```json
{
  "msg": "success",
  "res": {
    "checkout_cycle": 3,
    "period_from": "2026-02-01",
    "period_to": "2026-02-28",
    "payday": "2026-03-05",
    "task_count": 20,
    "total_wage": 30000.00,
    "total_ot_pay": 1500.00,
    "total_sum": 31500.00,
    "total_work_time": 160.00,
    "total_ot_time": 8.00,
    "total_late_time": 0.00,
    "total_absent_time": 0.00,
    "total_leave_time": 0.00,
    "total_leave_deduction": 0.00,
    "total_absence_deduction": 0.00,
    "labor_sum": 6360.00,
    "health_sum": 9000.00,
    "retirement": {
      "rate": 0.06,
      "amount": 1890.00
    },
    "overtime_limit": {
      "month": "2026-02",
      "used_ot_hours": 8.00,
      "limit": 46,
      "exceeded": false
    },
    "data": [
      {
        "task_id": 42,
        "work_time": 8.00,
        "ot_time": 0.50,
        "wage": 1500.00,
        "ot_pay": 75.00
      }
    ]
  }
}
```

#### 區間無未結算任務時 `200 OK`

```json
{
  "msg": "success",
  "res": {
    "checkout_cycle": 3,
    "period_from": "2026-02-01",
    "period_to": "2026-02-28",
    "payday": "2026-03-05",
    "task_count": 0,
    "total_wage": 0,
    "total_ot_pay": 0,
    "total_sum": 0,
    "data": []
  }
}
```

#### 回應欄位說明

| 欄位 | 類型 | 說明 |
|------|------|------|
| `checkout_cycle` | integer | 結算週期：`1` 日結 / `2` 週結 / `3` 月結 |
| `period_from` | string | 結算起始日 |
| `period_to` | string | 結算截止日 |
| `payday` | string\|null | 預計發薪日（根據 `job.wage.checkout_date` 計算） |
| `task_count` | integer | 納入結算的任務筆數 |
| `total_wage` | number | 底薪合計 |
| `total_ot_pay` | number | 加班費合計 |
| `total_sum` | number | 應付薪資總計（底薪 + 加班費） |
| `total_work_time` | number | 總工作時數（小時） |
| `total_ot_time` | number | 總加班時數（小時） |
| `total_late_time` | number | 總遲到時數（小時） |
| `total_absent_time` | number | 總曠職時數（小時） |
| `total_leave_time` | number | 總請假時數（小時） |
| `total_leave_deduction` | number | 請假薪資扣款 |
| `total_absence_deduction` | number | 曠職薪資扣款 |
| `labor_sum` | number | 勞保費合計（員工負擔） |
| `health_sum` | number | 健保費合計（員工負擔） |
| `retirement` | object\|null | 退休金提撥資訊 |
| `overtime_limit` | object\|null | 當月加班上限狀態（法定 46 小時） |
| `data` | array | 各任務明細清單 |

#### 錯誤回應

| HTTP | msg | 說明 |
|------|-----|------|
| `401` | Unauthenticated | 未登入 |
| `422` | validation error / 業務錯誤訊息 | 參數驗證失敗或 Job 不存在 |
| `500` | preview failed | 計算異常 |

---

## 5. 週期薪資正式結算

### `POST /v1/salary/period/checkout`

依指定日期範圍對所有未結算任務（`fee = 0`）執行正式結算：
- 更新 `tasks.fee = 1` 及 `settlement_at`
- 寫入或更新 `monthly_salary_infos` 月薪摘要紀錄

**權限限制：** 僅該職位的雇主（`job.user_id === Auth::id()`）可執行。

> **冪等說明：** 同一區間二次執行，因已結算任務 `fee = 1` 不會再被撈取，`task_count` 回傳 `0`，不會重複寫入。

#### Request Body（JSON）

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| `job_id` | integer | 是 | 職缺 ID（需存在於 `jobs` 表） |
| `user_id` | integer | 是 | 員工 ID（需存在於 `users` 表） |
| `period_from` | string | 是 | 結算起始日（格式：`Y-m-d`） |
| `period_to` | string | 是 | 結算截止日（格式：`Y-m-d`，需 ≥ `period_from`） |
| `pay_way` | integer | 是 | 結算方式：`0` 現金 / `1` 匯款 |

```json
{
  "job_id": 3,
  "user_id": 10,
  "period_from": "2026-02-01",
  "period_to": "2026-02-28",
  "pay_way": 1
}
```

#### 成功回應 `200 OK`

回應結構與 [週期薪資試算](#4-週期薪資試算dry-run) 相同，另加入 `task_count`（已結算筆數）。

```json
{
  "msg": "success",
  "res": {
    "checkout_cycle": 3,
    "period_from": "2026-02-01",
    "period_to": "2026-02-28",
    "payday": "2026-03-05",
    "task_count": 20,
    "total_wage": 30000.00,
    "total_ot_pay": 1500.00,
    "total_sum": 31500.00,
    "data": []
  }
}
```

#### 錯誤回應

| HTTP | msg | 說明 |
|------|-----|------|
| `401` | Unauthenticated | 未登入 |
| `403` | unauthorized | 非該職位雇主 |
| `422` | validation error / 業務錯誤訊息 | 參數驗證失敗或 Job 不存在 |
| `500` | checkout failed | 結算異常 |

---

## 附錄：結算週期常數

| 值 | 說明 |
|----|------|
| `1` | 日結（Daily） |
| `2` | 週結（Weekly） |
| `3` | 月結（Monthly） |

## 附錄：發薪日計算規則

發薪日根據 `job.wage.checkout_date[0]`（1–31）及 `period_to` 月份計算：

1. 若當月沒有該日（如 2 月沒有 30 日）→ 延展到下個月
2. 若當月有該日但已早於 `period_to` → 延展到下個月
3. 未設定（`checkout_date` 為 0）→ 回傳 `null`

**範例：**

| `checkout_date` | `period_to` | 發薪日 |
|-----------------|-------------|--------|
| 30 | 2026-02-28 | 2026-03-30 |
| 31 | 2026-04-15 | 2026-05-31 |
| 5 | 2026-06-12 | 2026-07-05 |
| 20 | 2026-06-12 | 2026-06-20 |
| 0 | 任意 | `null` |
