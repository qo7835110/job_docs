# 訂閱系統 API 文件

> **版本**：v1.0  
> **最後更新**：2026-04-14  
> **Base URL**：`/api`

---

## 目錄

- [認證說明](#認證說明)
- [資料結構與狀態說明](#資料結構與狀態說明)
- [User — 方案瀏覽 (Public)](#user--方案瀏覽-public)
- [User — 訂閱操作](#user--訂閱操作)
- [Admin — 方案管理 (Plans)](#admin--方案管理-plans)
- [Admin — 訂閱管理 (Subscriptions)](#admin--訂閱管理-subscriptions)
- [Admin — 交易紀錄管理 (Payments)](#admin--交易紀錄管理-payments)
- [Admin — 使用者發放免費通行證](#admin--使用者發放免費通行證)
- [狀態碼一覽](#狀態碼一覽)

---

## 認證說明

所有 `admin` 路由均受 `admin.auth` Middleware 保護，需在請求標頭帶上管理員 Token。

```
Content-Type: application/json
Authorization: Bearer {admin_token}
```

---

## 資料結構與狀態說明

### Plan 物件

| 欄位 | 類型 | 說明 |
|---|---|---|
| `id` | integer | 主鍵 |
| `name` | string | 方案名稱 |
| `description` | string\|null | 方案描述 |
| `price` | string (decimal) | 價格（例：`"299.00"`）|
| `currency` | string | 幣別，預設 `TWD` |
| `interval_type` | string | 週期：`month` \| `year` |
| `features_json` | object\|null | 方案功能設定（自訂 Key-Value）|
| `is_active` | boolean | 是否對外上架 |
| `created_at` | string (ISO 8601) | 建立時間 |
| `updated_at` | string (ISO 8601) | 更新時間 |
| `deleted_at` | string\|null | 軟刪除時間（null 表示未刪除）|

### Subscription 物件

| 欄位 | 類型 | 說明 |
|---|---|---|
| `id` | integer | 主鍵 |
| `user_id` | integer | 使用者 ID |
| `plan_id` | integer | 方案 ID |
| `provider_subscription_id` | string\|null | 外部金流訂閱代號（預留欄位）|
| `status` | string | 見下方狀態說明 |
| `starts_at` | string\|null | 訂閱開始時間 |
| `ends_at` | string\|null | 訂閱到期時間（null 代表無限期）|
| `canceled_at` | string\|null | 申請取消時間（訂閱仍有效至 ends_at）|
| `comment` | string\|null | 後台備註 |
| `metadata` | string\|null | 額外 JSON 資訊（如免費通行證標記）|
| `created_at` | string (ISO 8601) | 建立時間 |
| `deleted_at` | string\|null | 軟刪除時間 |

#### Subscription `status` 狀態說明

| 值 | 說明 |
|---|---|
| `unpaid` | 使用者已送出訂閱申請，等待使用者完成匯款並提交確認 |
| `reviewing` | 使用者已提交付款確認，**等待管理員核驗匯款是否到帳** |
| `active` | 管理員已確認收款，訂閱開通中且在有效期限內 |
| `past_due` | 帳單逾期未繳 |
| `canceled` | 已取消（無法使用），管理員拒絕或到期後由排程更新 |

> **標準狀態流程**：`unpaid` → *(使用者提交付款確認)* → `reviewing` → *(管理員核驗)* → `active` 或 `canceled`

### Payment 物件

| 欄位 | 類型 | 說明 |
|---|---|---|
| `id` | integer | 主鍵 |
| `subscription_id` | integer | 對應的訂閱 ID |
| `provider_transaction_id` | string\|null | 外部金流交易序號（預留欄位）|
| `amount` | string (decimal) | 交易金額（例：`"299.00"`）|
| `status` | string | 見下方狀態說明 |
| `paid_at` | string\|null | 實際付款時間 |
| `error_message` | string\|null | 失敗原因描述 |
| `created_at` | string (ISO 8601) | 建立時間 |
| `deleted_at` | string\|null | 軟刪除時間 |

#### Payment `status` 狀態說明

| 值 | 說明 |
|---|---|
| `pending` | 待付款（已建立但尚未確認）|
| `success` | 付款成功 |
| `failed` | 付款失敗 |
| `refunded` | 已退款 |

---

## User — 方案瀏覽 (Public)

> 這支 API 為公開路由，**不需要認證 Token**，前端可直接呼叫用於渲染選購頁面。

### 取得所有上架方案

```
GET /v1/open/plans
```

> 僅回傳 `is_active = true` 的方案，管理員專用的免費體驗方案（`is_active = false`）不會出現。

#### 成功回應 (200)

```json
[
  {
    "id": 2,
    "name": "進階月訂閱",
    "description": "解鎖所有進階功能，支援無限制發送訊息。",
    "price": "299.00",
    "currency": "TWD",
    "interval_type": "month",
    "features_json": {
      "can_view_premium_content": true,
      "max_messages": -1
    },
    "is_active": true,
    "created_at": "2026-04-02T10:00:00.000000Z",
    "updated_at": "2026-04-02T10:00:00.000000Z"
  },
  {
    "id": 3,
    "name": "尊榮年訂閱",
    "description": "更划算的年度一次性計費方案。",
    "price": "2990.00",
    "currency": "TWD",
    "interval_type": "year",
    "features_json": {
      "can_view_premium_content": true,
      "max_messages": -1
    },
    "is_active": true,
    "created_at": "2026-04-02T10:00:00.000000Z",
    "updated_at": "2026-04-02T10:00:00.000000Z"
  }
]
```

> **注意**：回應為陣列（Array），非分頁物件。

---

## User — 訂閱操作

> 以下 API 皆需要使用者 Token 認證。

```
Authorization: Bearer {user_token}
```

---

### 1. 取得我的所有訂閱紀錄

```
GET /v1/subscriptions
```

> 回傳分頁列表，每筆含 `plan` 關聯，依建立時間倒序排列。適合用於訂閱歷史紀錄頁面。

#### Query Parameters

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| `per_page` | integer | 否 | 每頁筆數，預設 `10` |

#### 成功回應 (200)

```json
{
  "data": {
    "current_page": 1,
    "data": [
      {
        "id": 5,
        "user_id": 12,
        "plan_id": 2,
        "status": "active",
        "starts_at": "2026-04-14T15:00:00.000000Z",
        "ends_at": "2026-05-14T15:00:00.000000Z",
        "canceled_at": null,
        "comment": "匯款帳號末五碼 12345",
        "created_at": "2026-04-14T15:00:00.000000Z",
        "plan": {
          "id": 2,
          "name": "進階月訂閱",
          "price": "299.00",
          "interval_type": "month"
        }
      }
    ],
    "per_page": 10,
    "total": 1
  }
}
```

---

### 2. 查詢我目前的訂閱狀態

```
GET /v1/subscriptions/me
```

> 回傳最新有效訂閱（含 `plan` 關聯）與 `temp_access` 臨時開通資格資訊，前端可據此決定是否顯示臨時開通按鈕。

#### 成功回應 (200)

**有效訂閱存在時（不顯示臨時開通按鈕）：**

```json
{
  "data": {
    "has_active_subscription": true,
    "subscription": {
      "id": 5,
      "status": "active",
      "ends_at": "2026-05-14T15:00:00.000000Z",
      "plan": { "id": 2, "name": "進階月訂閱", "price": "299.00" }
    },
    "temp_access": {
      "can_request": false,
      "remaining": 2,
      "limit": 2,
      "days": 3,
      "reference_starts_at": "2026-04-14T15:00:00.000000Z"
    }
  }
}
```

**訂閱已過期、可申請臨時開通時：**

```json
{
  "data": {
    "has_active_subscription": false,
    "subscription": null,
    "temp_access": {
      "can_request": true,
      "remaining": 2,
      "limit": 2,
      "days": 3,
      "reference_starts_at": "2026-04-14T15:00:00.000000Z"
    }
  }
}
```

**臨時開通次數已用完時：**

```json
{
  "data": {
    "has_active_subscription": false,
    "subscription": null,
    "temp_access": {
      "can_request": false,
      "remaining": 0,
      "limit": 2,
      "days": 3,
      "reference_starts_at": "2026-04-14T15:00:00.000000Z"
    }
  }
}
```

**從未付費、無法使用臨時開通時：**

```json
{
  "data": {
    "has_active_subscription": false,
    "subscription": null,
    "temp_access": null
  }
}
```

> **`temp_access` 欄位說明**：
> - `can_request`：`true` 代表目前可以申請臨時開通
> - `remaining`：本付款週期剩餘次數
> - `reference_starts_at`：額度所屬的付款週期開始時間

---

### 3. 申請臨時開通（3 天）

```
POST /v1/subscriptions/temp-access
```

> 讓使用者在訂閱過期後，自行啟用 3 天的臨時存取權限，無需管理員介入。
>
> **額度規則**：每個付款週期提供 **2 次**臨時開通機會，付款成功後自動重置。

#### 業務規則

- 目前**有效訂閱存在**時，不可申請（已有訂閱無需臨時開通）。
- 從未有過成功付款紀錄的用戶，**不可**使用此功能。
- 本付款週期內已使用 **2 次**，回傳 400。

#### 無需 Request Body

#### 成功回應 (201)

```json
{
  "message": "臨時開通成功！有效期限至 2026-05-15 12:00，本週期剩餘 1 次臨時開通次數。",
  "data": {
    "id": 10,
    "user_id": 12,
    "plan_id": 1,
    "status": "active",
    "starts_at": "2026-05-12T12:00:00.000000Z",
    "ends_at": "2026-05-15T12:00:00.000000Z",
    "metadata": "{\"is_temp_access\":true,\"reference_subscription_id\":5}"
  },
  "remaining_temp_access": 1
}
```

#### 錯誤回應 — 已有有效訂閱 (400)

```json
{ "message": "您目前已有有效的訂閱，無須申請臨時開通。" }
```

#### 錯誤回應 — 無付費紀錄 (400)

```json
{ "message": "您目前沒有訂閱記錄，無法使用臨時開通功能。" }
```

#### 錯誤回應 — 額度已用完 (400)

```json
{ "message": "本付款週期的臨時開通次數（2 次）已用完，請重新訂閱以獲得新的使用次數。" }
```

---

### 4. 送出訂閱申請

```
POST /v1/subscriptions/subscribe
```

> 目前為人工審核流程。送出後系統會建立一筆 `status = unpaid` 的訂閱紀錄，等待管理員確認匯款後手動開通（呼叫 `approve` API）。

#### 業務規則

- 若已有 `status = active` 的訂閱，會拒絕並回傳 400。

#### Request Body

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| `plan_id` | integer | 是 | 要訂閱的方案 ID（需在 `plans` 表中存在）|

#### 請求範例

```json
{
  "plan_id": 2
}
```

#### 成功回應 (201)

```json
{
  "message": "已收到訂閱請求，請等待管理員確認收款後開通。",
  "data": {
    "id": 5,
    "user_id": 12,
    "plan_id": 2,
    "status": "unpaid",
    "starts_at": null,
    "ends_at": null,
    "created_at": "2026-04-14T15:00:00.000000Z"
  }
}
```

#### 錯誤回應 — 已有有效訂閱 (400)

```json
{
  "message": "您已有有效的訂閱"
}
```

#### 錯誤回應 — 驗證失敗 (422)

```json
{
  "message": "The given data was invalid.",
  "errors": {
    "plan_id": ["The selected plan id is invalid."]
  }
}
```

---

### 4. 提交付款確認（含匯款備註）

```
POST /v1/subscriptions/{id}/submit-payment
```

> 使用者完成匯款後，填寫匯款資訊（如帳號末五碼、轉帳時間）並同步提交付款確認，通知管理員開始核驗。
>
> **已整合備註與狀態推進**：一支 API 同時完成「填寫資訊」與「提交確認」兩個動作。

#### 業務規則

| 目前狀態 | 行為 |
|---|---|
| `unpaid` | 寫入 `comment` + 狀態推進至 `reviewing` |
| `reviewing` | 僅更新 `comment`（補充或修正資訊），狀態維持 `reviewing` |
| `active` / 其他 | 回傳 400 錯誤 |

- 只能操作**屬於自己**的訂閱，否則回傳 404。

#### Request Body

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| `comment` | string | 是 | 匯款資訊，最多 1000 字元（例如：帳號末五碼、轉帳日期等）|

#### 請求範例

```json
{
  "comment": "已匯款，帳號末五碼 12345，2026-05-12 轉帳"
}
```

#### 成功回應 — 首次提交 `unpaid → reviewing` (200)

```json
{
  "message": "付款資訊已提交，請等待管理員審查（通常於 1 個工作天內處理）。",
  "data": {
    "id": 5,
    "status": "reviewing",
    "comment": "已匯款，帳號末五碼 12345，2026-05-12 轉帳",
    "plan": { "id": 2, "name": "進階月訂閱", "price": "299.00" }
  }
}
```

#### 成功回應 — 補充資訊（已在 `reviewing` 狀態）(200)

```json
{
  "message": "付款資訊已更新，管理員審查中。",
  "data": {
    "id": 5,
    "status": "reviewing",
    "comment": "已重新確認，帳號末五碼 12345，2026-05-13 再次轉帳",
    "plan": { "id": 2, "name": "進階月訂閱", "price": "299.00" }
  }
}
```

#### 錯誤回應 — 已開通 (400)

```json
{ "message": "此訂閱已完成開通，無需重複提交。" }
```

---

## Admin — 方案管理 (Plans)

### 1. 取得方案列表

```
GET /api/admin/plans
```


#### Query Parameters

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| `is_active` | boolean | 否 | 過濾是否上架（`true` / `false`）|
| `per_page` | integer | 否 | 每頁筆數，預設 `15`，最大 `100` |

#### 成功回應 (200)

```json
{
  "current_page": 1,
  "data": [
    {
      "id": 1,
      "name": "免費體驗方案",
      "description": "管理員發放的系統免費體驗方案",
      "price": "0.00",
      "currency": "TWD",
      "interval_type": "month",
      "features_json": {
        "can_view_premium_content": true,
        "max_messages": 50
      },
      "is_active": false,
      "created_at": "2026-04-02T10:00:00.000000Z",
      "updated_at": "2026-04-02T10:00:00.000000Z",
      "deleted_at": null
    },
    {
      "id": 2,
      "name": "進階月訂閱",
      "description": "解鎖所有進階功能，支援無限制發送訊息。",
      "price": "299.00",
      "currency": "TWD",
      "interval_type": "month",
      "features_json": {
        "can_view_premium_content": true,
        "max_messages": -1
      },
      "is_active": true,
      "created_at": "2026-04-02T10:00:00.000000Z",
      "updated_at": "2026-04-02T10:00:00.000000Z",
      "deleted_at": null
    }
  ],
  "per_page": 15,
  "total": 2
}
```

---

### 2. 新增方案

```
POST /api/admin/plans
```

#### Request Body

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| `name` | string | 是 | 方案名稱，最多 255 字元 |
| `description` | string | 否 | 方案描述，最多 1000 字元 |
| `price` | number | 是 | 價格，最小 `0` |
| `currency` | string | 否 | 幣別，3 碼，預設 `TWD` |
| `interval_type` | string | 是 | `month` \| `year` |
| `features_json` | object | 否 | 功能設定，自訂 Key-Value 物件 |
| `is_active` | boolean | 否 | 是否上架，預設 `true` |

#### 請求範例

```json
{
  "name": "進階月訂閱",
  "description": "解鎖所有進階功能，支援無限制發送訊息。",
  "price": 299,
  "currency": "TWD",
  "interval_type": "month",
  "features_json": {
    "can_view_premium_content": true,
    "max_messages": -1
  },
  "is_active": true
}
```

#### 成功回應 (201)

```json
{
  "message": "方案建立成功",
  "data": {
    "id": 2,
    "name": "進階月訂閱",
    "price": "299.00",
    "currency": "TWD",
    "interval_type": "month",
    "features_json": { "can_view_premium_content": true, "max_messages": -1 },
    "is_active": true,
    "created_at": "2026-04-14T15:00:00.000000Z",
    "updated_at": "2026-04-14T15:00:00.000000Z"
  }
}
```

---

### 3. 取得單一方案

```
GET /api/admin/plans/{id}
```

#### 成功回應 (200)

```json
{
  "data": {
    "id": 2,
    "name": "進階月訂閱",
    "price": "299.00",
    "currency": "TWD",
    "interval_type": "month",
    "features_json": { "can_view_premium_content": true, "max_messages": -1 },
    "is_active": true,
    "created_at": "2026-04-02T10:00:00.000000Z",
    "updated_at": "2026-04-02T10:00:00.000000Z",
    "deleted_at": null
  }
}
```

---

### 4. 更新方案

```
PUT /api/admin/plans/{id}
```

所有參數皆為選填，只傳入需要修改的欄位即可。

#### Request Body

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| `name` | string | 否 | 方案名稱 |
| `description` | string | 否 | 方案描述 |
| `price` | number | 否 | 價格 |
| `currency` | string | 否 | 幣別（3 碼）|
| `interval_type` | string | 否 | `month` \| `year` |
| `features_json` | object | 否 | 功能設定物件 |
| `is_active` | boolean | 否 | 是否上架 |

#### 請求範例

```json
{
  "price": 399,
  "is_active": false
}
```

#### 成功回應 (200)

```json
{
  "message": "方案更新成功",
  "data": { ...更新後的 Plan 物件 }
}
```

---

### 5. 刪除方案（軟刪除）

```
DELETE /api/admin/plans/{id}
```

> **注意**：執行後 `deleted_at` 欄位會被填入時間，資料不會真正從資料庫移除。

#### 成功回應 (200)

```json
{
  "message": "方案已成功刪除"
}
```

---

## Admin — 訂閱管理 (Subscriptions)

### 1. 取得訂閱列表

```
GET /api/admin/subscriptions
```

#### Query Parameters

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| `status` | string | 否 | `active` \| `past_due` \| `canceled` \| `unpaid` |
| `user_id` | integer | 否 | 依使用者 ID 過濾 |
| `per_page` | integer | 否 | 每頁筆數，預設 `15`，最大 `100` |

#### 成功回應 (200)

```json
{
  "current_page": 1,
  "data": [
    {
      "id": 5,
      "user_id": 12,
      "plan_id": 2,
      "status": "unpaid",
      "starts_at": null,
      "ends_at": null,
      "canceled_at": null,
      "comment": null,
      "metadata": null,
      "created_at": "2026-04-14T15:00:00.000000Z",
      "updated_at": "2026-04-14T15:00:00.000000Z",
      "deleted_at": null,
      "user": {
        "id": 12,
        "name": "王小明",
        "email": "user@example.com"
      },
      "plan": {
        "id": 2,
        "name": "進階月訂閱",
        "price": "299.00",
        "interval_type": "month"
      }
    }
  ],
  "per_page": 15,
  "total": 1
}
```

---

### 2. 取得單一訂閱詳情

```
GET /api/admin/subscriptions/{id}
```

> 回應包含 `user`、`plan`、`payments` 三個關聯資料。

#### 成功回應 (200)

```json
{
  "data": {
    "id": 5,
    "user_id": 12,
    "plan_id": 2,
    "status": "unpaid",
    "starts_at": null,
    "ends_at": null,
    "canceled_at": null,
    "comment": null,
    "metadata": null,
    "user": { "id": 12, "name": "王小明", "email": "user@example.com" },
    "plan": { "id": 2, "name": "進階月訂閱", "price": "299.00" },
    "payments": [
      {
        "id": 8,
        "subscription_id": 5,
        "amount": "299.00",
        "status": "pending",
        "paid_at": null,
        "created_at": "2026-04-14T15:00:00.000000Z"
      }
    ]
  }
}
```

---

### 3. 核准訂閱（人工開通）

> 用於管理員確認已收到匯款後，手動觸發開通訂閱。  
> 此 API 會自動根據 Plan 的 `interval_type` 計算 `ends_at`，並將對應的 `pending` Payment 更新為 `success`。

```
POST /api/admin/subscriptions/{id}/approve
```

- 僅接受 `status = unpaid` 的訂閱
- 若訂閱已非 `unpaid` 狀態，回傳 400

#### 成功回應 (200)

```json
{
  "message": "訂閱已成功開通",
  "data": {
    "id": 5,
    "status": "active",
    "starts_at": "2026-04-14T15:00:00.000000Z",
    "ends_at": "2026-05-14T15:00:00.000000Z",
    ...
  }
}
```

#### 錯誤回應 — 狀態不符 (400)

```json
{
  "message": "該訂閱狀態無法執行開通（僅接受 unpaid 或 reviewing 狀態）。"
}
```

---

### 4. 標記為審查中 (`unpaid` → `reviewing`)

```
POST /api/admin/subscriptions/{id}/mark-reviewing
```

> 管理員可主動將訂閱標記為「審查中」，適用於管理員看到使用者的匯款備註後手動記錄。
> 使用者自行提交付款確認時也會自動觸發此狀態轉換。

#### 業務規則

- 僅接受 `unpaid` 狀態

#### 無需 Request Body

#### 成功回應 (200)

```json
{
  "message": "訂閱已標記為審查中",
  "data": {
    "id": 5,
    "status": "reviewing",
    "comment": "已匯款，帳號未五碼 12345"
  }
}
```

---

### 5. 拒絕 / 取消訂閱

```
POST /api/admin/subscriptions/{id}/reject
```

> 執行後訂閱狀態會變更為 `canceled`，並記錄 `canceled_at` 時間戳。

#### 成功回應 (200)

```json
{
  "message": "訂閱已拒絕/取消"
}
```

---

### 5. 手動更新訂閱資訊

> 適用於手動延長效期、修改備註等場景。  
> **注意**：若要「開通訂閱」請使用 `approve` 而非此 API，以確保 `ends_at` 正確計算。  
> `status` 僅允許變更為非 `active` 的值，若需開通請使用 `approve`。

```
PUT /api/admin/subscriptions/{id}
```

#### Request Body

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| `status` | string | 否 | `past_due` \| `canceled` \| `unpaid` \| `reviewing`（**不可設為 active**）|
| `starts_at` | string (date) \| null | 否 | 訂閱開始日，可傳 `null` 清空 |
| `ends_at` | string (date) \| null | 否 | 訂閱到期日，可傳 `null` 清空，需 >= `starts_at` |
| `comment` | string \| null | 否 | 後台備註，最多 1000 字元，可傳 `null` 清空 |

#### 請求範例（手動延長一個月）

```json
{
  "ends_at": "2026-06-14",
  "comment": "用戶反映 App Bug，補償延長一個月"
}
```

#### 成功回應 (200)

```json
{
  "message": "訂閱資料已更新",
  "data": { ...更新後的 Subscription 物件 }
}
```

---

### 6. 刪除訂閱（軟刪除）

```
DELETE /api/admin/subscriptions/{id}
```

#### 成功回應 (200)

```json
{
  "message": "訂閱紀錄已成功刪除"
}
```

---

## Admin — 交易紀錄管理 (Payments)

### 1. 取得交易紀錄列表

```
GET /api/admin/payments
```

#### Query Parameters

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| `status` | string | 否 | `pending` \| `success` \| `failed` \| `refunded` |
| `subscription_id` | integer | 否 | 依訂閱 ID 過濾 |
| `date_from` | string (date) | 否 | 建立日期起（`YYYY-MM-DD`）|
| `date_to` | string (date) | 否 | 建立日期迄（必須 >= `date_from`）|
| `per_page` | integer | 否 | 每頁筆數，預設 `15`，最大 `100` |

#### 成功回應 (200)

```json
{
  "current_page": 1,
  "data": [
    {
      "id": 8,
      "subscription_id": 5,
      "provider_transaction_id": null,
      "amount": "299.00",
      "status": "pending",
      "paid_at": null,
      "error_message": null,
      "created_at": "2026-04-14T15:00:00.000000Z",
      "deleted_at": null,
      "subscription": {
        "id": 5,
        "user": { "id": 12, "name": "王小明" },
        "plan": { "id": 2, "name": "進階月訂閱" }
      }
    }
  ],
  "per_page": 15,
  "total": 1
}
```

---

### 2. 手動補登交易紀錄

> 用於人工確認款項後，補建交易紀錄。  
> **若 `status` 設為 `success`，系統會自動觸發對應訂閱的開通流程**（等同執行 approve）。

```
POST /api/admin/payments
```

#### Request Body

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| `subscription_id` | integer | 是 | 對應的訂閱 ID（需存在）|
| `provider_transaction_id` | string | 否 | 金流交易序號（需唯一）|
| `amount` | number | 是 | 交易金額，最小 `0` |
| `status` | string | 是 | `pending` \| `success` \| `failed` \| `refunded` |
| `paid_at` | string (date) | 否 | 實際付款時間 |
| `error_message` | string | 否 | 失敗說明，最多 2000 字元 |

#### 請求範例

```json
{
  "subscription_id": 5,
  "amount": 299,
  "status": "success",
  "paid_at": "2026-04-14 15:00:00"
}
```

#### 成功回應 (201)

```json
{
  "message": "交易紀錄已手動新增",
  "data": {
    "id": 9,
    "subscription_id": 5,
    "amount": "299.00",
    "status": "success",
    "paid_at": "2026-04-14T15:00:00.000000Z",
    "subscription": {
      "user": { "id": 12, "name": "王小明" }
    }
  }
}
```

---

### 3. 取得單一交易紀錄

```
GET /api/admin/payments/{id}
```

> 回應包含 `subscription.user` 與 `subscription.plan` 關聯資料。

#### 成功回應 (200)

```json
{
  "data": {
    "id": 8,
    "subscription_id": 5,
    "amount": "299.00",
    "status": "pending",
    "paid_at": null,
    "error_message": null,
    "subscription": {
      "id": 5,
      "status": "unpaid",
      "user": { "id": 12, "name": "王小明", "email": "user@example.com" },
      "plan": { "id": 2, "name": "進階月訂閱", "price": "299.00" }
    }
  }
}
```

---

### 4. 更新交易狀態

> 適用場景：確認退款完成後，將 `status` 改為 `refunded`。  
> **若將狀態由非 `success` 改為 `success`，系統會自動觸發對應訂閱的開通流程。**

```
PUT /api/admin/payments/{id}
```

#### Request Body

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| `provider_transaction_id` | string | 否 | 金流交易序號（需唯一）|
| `status` | string | 否 | `pending` \| `success` \| `failed` \| `refunded` |
| `paid_at` | string (date) | 否 | 實際付款時間 |
| `error_message` | string | 否 | 失敗說明，最多 2000 字元 |

#### 請求範例（確認退款）

```json
{
  "status": "refunded"
}
```

#### 成功回應 (200)

```json
{
  "message": "交易紀錄已更新",
  "data": { ...更新後的 Payment 物件 }
}
```

---

### 5. 封存交易紀錄（軟刪除）

```
DELETE /api/admin/payments/{id}
```

> **業務規則**：`status = success` 的交易紀錄**不允許直接刪除**。  
> 若需移除成功的交易，應先透過 `PUT` 將 `status` 更新為 `refunded`，再執行刪除。

#### 成功回應 (200)

```json
{
  "message": "交易紀錄已成功封存"
}
```

#### 錯誤回應 — 已成功的交易不可刪除 (400)

```json
{
  "message": "已成功的交易紀錄不可直接刪除，請先執行退款流程並更新狀態。"
}
```

---

## Admin — 使用者發放免費通行證

> 管理員可以對任意使用者發放一段時間的免費通行證。  
> 系統會直接建立一筆 `status = active` 的訂閱，綁定免費體驗方案（ID: 1）。  
> 通行證到期後，使用者的訂閱權限會自動失效（依 `ends_at` 判斷）。

```
POST /api/admin/users/{user}/grant-free-pass
```

> `{user}` 為使用者的 ID（整數），例如：`/api/admin/users/12/grant-free-pass`

#### Request Body

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| `days` | integer | 是 | 發放天數，範圍 `1~365` |
| `reason` | string | 否 | 發放原因備註，最多 255 字元 |

#### 請求範例

```json
{
  "days": 7,
  "reason": "App Bug 補償"
}
```

#### 成功回應 (201)

```json
{
  "message": "已成功發放 7 天的免費通行證",
  "data": {
    "id": 10,
    "user_id": 12,
    "plan_id": 1,
    "status": "active",
    "starts_at": "2026-04-14T15:00:00.000000Z",
    "ends_at": "2026-04-21T15:00:00.000000Z",
    "comment": "管理員手動贈送：App Bug 補償",
    "metadata": "{\"granted_by_admin\":true}",
    "created_at": "2026-04-14T15:00:00.000000Z"
  }
}
```

---

## 狀態碼一覽

| 狀態碼 | 說明 |
|---|---|
| `200` | 請求成功 |
| `201` | 建立成功 |
| `400` | 業務邏輯錯誤（例如：刪除已成功的交易、訂閱狀態不符）|
| `401` | 未認證（Token 遺失或無效）|
| `404` | 資源不存在 |
| `422` | 驗證失敗（欄位格式錯誤）|
| `500` | 系統內部錯誤 |
