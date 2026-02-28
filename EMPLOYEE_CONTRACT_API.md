# 勞動契約 API 文件

> 最後更新：2026-02-26（修正 store/update 端點欄位說明）

## 概述

提供雇主與員工之間的勞動契約建立、查詢、簽署與終止功能。所有 API 均需使用 Sanctum Token 認證。

**Base Path：** `/v1/contracts`

**認證 Header：**
```
Authorization: Bearer {token}
```

---

## 核心概念

### 勞動身分（legal_status）

| 值 | 常數 | 說明 |
|---|---|---|
| `0` | `LEGAL_STATUS_FULL_TIME` | 全時（正職） |
| `1` | `LEGAL_STATUS_PART_TIME` | 部分工時（兼職） |
| `2` | `LEGAL_STATUS_FREELANCE` | 承攬 |

### 合約簽署狀態（status）

| 值 | label | 說明 |
|---|---|---|
| `0` | `unsigned` | 未簽署 |
| `1` | `employer_signed` | 雇主已簽署，等待員工 |
| `2` | `employee_signed` | 員工已簽署，等待雇主 |
| `3` | `both_signed` | 雙方已完成簽署 |
| `4` | `terminated` | 合約已終止 |

### 合約生效時序

```
建立 (status=0)
  ↓ 雇主簽署  →  status=1
  ↓ 員工簽署  →  status=3（雙方完成）
                  ├─ 全時/兼職：自動對應職缺排班（WorkPeriod + WorkDay）
                  └─ 產生 Task

終止 (terminate) → status=4，end_date 更新為指定終止日
```

### 重疊合約保護

同一雇主、同一員工、同一職缺（job_id）不可存在日期重疊的合約。若新建或更新後發生重疊，系統將回傳 `400` 錯誤。

---

## API 端點

### POST `/v1/contracts`

**雇主建立勞動契約**

建立後合約狀態為 `unsigned (0)`，需雙方各自呼叫 sign API 完成簽署。

#### Request Body（JSON）

| 欄位 | 型別 | 必填 | 說明 |
|---|---|---|---|
| `employee_id` | integer | ✅ | 員工 User ID（必須存在於 users 表） |
| `job_id` | integer | ✅ | 關聯職缺 ID（必須存在且狀態為上架中） |
| `start_date` | string (Y-m-d) | ✅ | 合約起始日 |
| `end_date` | string (Y-m-d) \| null | — | 合約結束日（需 ≥ start_date） |
| `insurance_opt_in` | boolean | — | 是否加保（預設 false） |
| `insurance_subsidy_amount` | integer | — | 保險補助金額，最小 0 |
| `retirement_contribution` | boolean | — | 是否提繳勞退（預設 false） |

> `legal_status` 不需傳入，系統自動從關聯職缺的 `type` 欄位帶入。

#### Request 範例

```json
{
  "employee_id": 42,
  "job_id": 10,
  "legal_status": 0,
  "start_date": "2026-03-01",
  "end_date": "2027-02-28",
  "insurance_opt_in": true,
  "insurance_subsidy_amount": 300,
  "retirement_contribution": true
}
```

#### 回應範例（201）

```json
{
  "msg": "success",
  "res": {
    "id": 1,
    "owner_id": 5,
    "employee_id": 42,
    "job_id": 10,
    "legal_status": 0,
    "legal_status_label": "full_time",
    "start_date": "2026-03-01",
    "end_date": "2027-02-28",
    "is_active": false,
    "insurance_opt_in": true,
    "retirement_contribution": true,
    "status": 0,
    "status_label": "unsigned",
    "created_at": "2026-02-26T08:00:00+00:00",
    "updated_at": "2026-02-26T08:00:00+00:00",
    "employee": { "id": 42, "name": "王小明" },
    "job": { "id": 10, "title": "全端工程師" }
  }
}
```

#### 錯誤回應

| HTTP | msg | 原因 |
|---|---|---|
| `400` | 雇主與員工不可為同一人 | owner_id == employee_id |
| `400` | 關聯職缺不存在或已下架，無法建立合約 | job_id 對應的職缺不存在或 status ≠ 1 |
| `400` | 關聯職缺的工作期限已逾期，無法建立合約 | job.working.end_date 早於今日 |
| `400` | 該員工在此職缺已有重疊的有效合約 | 日期區間重疊 |
| `422` | validation error | 欄位驗證失敗 |

---

### GET `/v1/contracts/owner`

**雇主查看自己發出的合約列表**

#### Query String

| 參數 | 必填 | 預設 | 說明 |
|---|---|---|---|
| `status` | — | `all` | 有效期篩選：`active`（有效）/ `expired`（已過期）/ `all` |
| `job_id` | — | — | 篩選指定職缺的合約 |
| `employee_id` | — | — | 篩選指定員工的合約 |
| `start_date_from` | — | — | 合約 `start_date >=` 此日期（格式 `Y-m-d`） |
| `start_date_to` | — | — | 合約 `start_date <=` 此日期（格式 `Y-m-d`） |

> - `active`：`start_date <= 今日 AND (end_date IS NULL OR end_date >= 今日)`
> - `expired`：`end_date IS NOT NULL AND end_date < 今日`

#### 回應範例（200）

```json
{
  "msg": "success",
  "res": [
    {
      "id": 1,
      "owner_id": 5,
      "employee_id": 42,
      "job_id": 10,
      "legal_status": 0,
      "legal_status_label": "full_time",
      "start_date": "2026-03-01",
      "end_date": "2027-02-28",
      "is_active": false,
      "insurance_opt_in": true,
      "retirement_contribution": true,
      "status": 0,
      "status_label": "unsigned",
      "created_at": "2026-02-26T08:00:00+00:00",
      "updated_at": "2026-02-26T08:00:00+00:00",
      "employee": { "id": 42, "name": "王小明" },
      "job": { "id": 10, "title": "全端工程師" }
    }
  ]
}
```

---

### GET `/v1/contracts/employee`

**員工查看自己被受雇的合約列表**

#### Query String

| 參數 | 必填 | 預設 | 說明 |
|---|---|---|---|
| `status` | — | `all` | 有效期篩選：`active` / `expired` / `all`（同上） |
| `job_id` | — | — | 篩選指定職缺的合約 |
| `owner_id` | — | — | 篩選指定雇主的合約 |
| `start_date_from` | — | — | 合約 `start_date >=` 此日期（格式 `Y-m-d`） |
| `start_date_to` | — | — | 合約 `start_date <=` 此日期（格式 `Y-m-d`） |

#### 回應範例（200）

```json
{
  "msg": "success",
  "res": [
    {
      "id": 1,
      "owner_id": 5,
      "employee_id": 42,
      "job_id": 10,
      "legal_status": 0,
      "legal_status_label": "full_time",
      "start_date": "2026-03-01",
      "end_date": "2027-02-28",
      "is_active": false,
      "insurance_opt_in": true,
      "retirement_contribution": true,
      "status": 0,
      "status_label": "unsigned",
      "created_at": "2026-02-26T08:00:00+00:00",
      "updated_at": "2026-02-26T08:00:00+00:00",
      "owner": { "id": 5, "name": "陳老闆" },
      "job": { "id": 10, "title": "全端工程師" }
    }
  ]
}
```

---

### GET `/v1/contracts/{id}`

**查看單筆合約詳情**

雇主或員工（合約當事人）皆可呼叫。回應包含 `owner`、`employee`、`job`、`work_periods` 關聯資料。

#### Path Parameter

| 參數 | 型別 | 說明 |
|---|---|---|
| `id` | integer | 合約 ID |

#### 回應範例（200）

```json
{
  "msg": "success",
  "res": {
    "id": 1,
    "owner_id": 5,
    "employee_id": 42,
    "job_id": 10,
    "legal_status": 0,
    "legal_status_label": "full_time",
    "start_date": "2026-03-01",
    "end_date": "2027-02-28",
    "is_active": true,
    "insurance_opt_in": true,
    "retirement_contribution": true,
    "status": 3,
    "status_label": "both_signed",
    "created_at": "2026-02-26T08:00:00+00:00",
    "updated_at": "2026-03-01T00:00:00+00:00",
    "owner": { "id": 5, "name": "陳老闆" },
    "employee": { "id": 42, "name": "王小明" },
    "job": { "id": 10, "title": "全端工程師" },
    "work_periods": [
      {
        "id": 1,
        "period_type": "monthly",
        "start_date": "2026-03-01",
        "end_date": "2026-03-31",
        "published_at": "2026-02-28T10:00:00+00:00",
        "locked_at": null
      }
    ]
  }
}
```

#### 錯誤回應

| HTTP | msg | 原因 |
|---|---|---|
| `404` | contract not found | 合約不存在或無存取權 |

---

### POST `/v1/contracts/{id}/update`

**更新勞動契約**（僅雇主可更新）

#### Path Parameter

| 參數 | 型別 | 說明 |
|---|---|---|
| `id` | integer | 合約 ID |

#### Request Body（JSON，所有欄位皆選填）

| 欄位 | 型別 | 說明 |
|---|---|---|
| `start_date` | string (Y-m-d) | 合約起始日 |
| `end_date` | string (Y-m-d) \| null | 合約結束日（需 ≥ start_date） |
| `insurance_opt_in` | boolean | 是否加保 |
| `insurance_subsidy_amount` | integer | 保險補助金額，最小 0 |
| `retirement_contribution` | boolean | 是否提繳勞退 |

#### Request 範例

```json
{
  "end_date": "2026-12-31",
  "insurance_subsidy_amount": 500
}
```

#### 回應範例（200）

```json
{
  "msg": "success",
  "res": { "...同 show 結構..." }
}
```

#### 錯誤回應

| HTTP | msg | 原因 |
|---|---|---|
| `403` | unauthorized | 非合約雇主 |
| `404` | contract not found | 合約不存在或無存取權 |
| `400` | 合約已終止，不可修改 | status 為 4（terminated） |
| `400` | 該員工在此職缺已有重疊的有效合約 | 日期變更後造成重疊 |

---

### POST `/v1/contracts/{id}/sign`

**簽署合約**（雇主或員工皆可呼叫）

依當前簽署狀態自動推進，雙方皆完成後（`status=3`）觸發：
1. **全時/兼職**：自動產生排班（WorkPeriod + WorkDay）
2. 依合約類型產生 Task

#### 簽署狀態轉換邏輯

| 當前 status | 簽署者 | 結果 status |
|---|---|---|
| `0` unsigned | 雇主 | `1` employer_signed |
| `0` unsigned | 員工 | `2` employee_signed |
| `1` employer_signed | 員工 | `3` both_signed ✅ |
| `2` employee_signed | 雇主 | `3` both_signed ✅ |
| `1` employer_signed | 雇主 | `400` 雇主已簽署，等待員工簽署 |
| `2` employee_signed | 員工 | `400` 員工已簽署，等待雇主簽署 |
| `3` both_signed | 任何人 | `400` 合約已完成雙方簽署 |
| `4` terminated | 任何人 | `400` 合約已終止，無法簽署 |

#### Path Parameter

| 參數 | 型別 | 說明 |
|---|---|---|
| `id` | integer | 合約 ID |

#### Request Body

無（空 POST）

#### 回應範例（200）

```json
{
  "msg": "success",
  "res": {
    "id": 1,
    "status": 3,
    "status_label": "both_signed",
    "owner": { "id": 5, "name": "陳老闆" },
    "employee": { "id": 42, "name": "王小明" },
    "job": { "id": 10, "title": "全端工程師" },
    "...其他欄位..."
  }
}
```

#### 錯誤回應

| HTTP | msg | 原因 |
|---|---|---|
| `404` | contract not found | 合約不存在或無存取權 |
| `400` | 關聯職缺不存在或已下架，無法簽署合約 | job_id 對應的職缺不存在或 status ≠ 1 |
| `400` | 關聯職缺的工作期限已逾期，無法簽署合約 | job.working.end_date 早於今日 |
| `400` | 合約已完成雙方簽署 | status 已為 3 |
| `400` | 合約已終止，無法簽署 | status 為 4 |
| `400` | 雇主已簽署，等待員工簽署 | 雇主重複簽署 |
| `400` | 員工已簽署，等待雇主簽署 | 員工重複簽署 |
| `400` | 僅合約當事人可進行簽署 | 非合約雙方 |

---

### POST `/v1/contracts/{id}/terminate`

**終止合約**（僅雇主可操作）

設定終止日並將合約狀態更新為 `terminated (4)`。終止日不可早於 `start_date`，也不可晚於原有的 `end_date`。

#### Path Parameter

| 參數 | 型別 | 說明 |
|---|---|---|
| `id` | integer | 合約 ID |

#### Request Body（JSON）

| 欄位 | 型別 | 必填 | 說明 |
|---|---|---|---|
| `end_date` | string (Y-m-d) | ✅ | 終止日期 |

#### Request 範例

```json
{
  "end_date": "2026-05-31"
}
```

#### 回應範例（200）

```json
{
  "msg": "success",
  "res": {
    "id": 1,
    "end_date": "2026-05-31",
    "status": 4,
    "status_label": "terminated",
    "...其他欄位..."
  }
}
```

#### 錯誤回應

| HTTP | msg | 原因 |
|---|---|---|
| `403` | unauthorized | 非合約雇主 |
| `404` | contract not found | 合約不存在或無存取權 |
| `400` | 終止日期不可早於合約起始日 | end_date < start_date |
| `400` | 終止日期不可晚於原合約結束日 | end_date > 原 end_date |
| `422` | validation error | 欄位驗證失敗 |

---

## Response 資料結構（EmployeeContractResource）

| 欄位 | 型別 | 說明 |
|---|---|---|
| `id` | integer | 合約 ID |
| `owner_id` | integer | 雇主 User ID |
| `employee_id` | integer | 員工 User ID |
| `job_id` | integer \| null | 關聯職缺 ID |
| `legal_status` | integer | 勞動身分值（0/1/2） |
| `legal_status_label` | string | `full_time` / `part_time` / `freelance` |
| `start_date` | string (Y-m-d) | 合約起始日 |
| `end_date` | string (Y-m-d) \| null | 合約結束日 |
| `is_active` | boolean | 是否在有效期內（`start_date <= 今日 AND end_date >= 今日 或 null`） |
| `insurance_opt_in` | boolean | 是否加保 |
| `retirement_contribution` | boolean | 是否提繳勞退 |
| `status` | integer | 簽署狀態值（0–4） |
| `status_label` | string | 簽署狀態文字 |
| `created_at` | string (ISO 8601) | 建立時間 |
| `updated_at` | string (ISO 8601) | 更新時間 |
| `owner` | object \| null | 雇主資訊（`id`, `name`），僅 `show` / `sign` 載入 |
| `employee` | object \| null | 員工資訊（`id`, `name`），大多端點皆載入 |
| `job` | object \| null | 職缺資訊（`id`, `title`），大多端點皆載入 |
| `work_periods` | array \| null | 排班週期列表，僅 `show` 載入 |

---

## 資料庫欄位（employee_contracts）

| 欄位 | 型別 | 說明 |
|---|---|---|
| `id` | bigint unsigned | PK |
| `owner_id` | bigint unsigned | 雇主 user ID |
| `employee_id` | bigint unsigned | 員工 user ID |
| `job_id` | bigint unsigned \| NULL | 關聯職缺 ID |
| `legal_status` | tinyint | 勞動身分：0=全時、1=部分工時、2=承攬 |
| `start_date` | date | 合約起始日 |
| `end_date` | date \| NULL | 合約結束日（NULL 表示無固定期限） |
| `insurance_opt_in` | boolean | 是否加保（預設 false） |
| `insurance_subsidy_amount` | integer | 保險補助金額（預設 0） |
| `retirement_contribution` | boolean | 是否提繳勞退（預設 false） |
| `status` | tinyint | 簽署狀態（預設 0） |
| `created_at` | timestamp | — |
| `updated_at` | timestamp | — |

### 索引

- `(owner_id, employee_id)` 複合索引
- `job_id` 單欄索引
