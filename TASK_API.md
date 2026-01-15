# 任務結算確認 API

## Base URL
```
/v1/task
```

## 確認結算 (Confirm Settlement)

### Endpoint
```
POST /{id}/settlement/confirm
```

### Description
員工確認任務結算，更新結算確認狀態並記錄確認時間。此 API 只能由任務的執行者（員工）調用。

### Authentication
需要使用者認證（Bearer Token）

### Headers
```
Authorization: Bearer {token}
Content-Type: application/json
Accept: application/json
```

### Path Parameters
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| id | integer | 是 | 任務ID |

### Request Example
```
POST /v1/task/123/settlement/confirm
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
```

### Success Response (200 OK)
```json
{
    "msg": "success",
    "res": {
        "id": 123,
        "job_id": 45,
        "user_id": 78,
        "wage": 200,
        "status": 3,
        "fee": 1,
        "pay_way": 1,
        "checkout_wage": 1600,
        "settlement_at": "2026-01-15 10:30:00",
        "settlement_confirmed": 1,
        "settlement_confirmed_at": "2026-01-15 14:20:35",
        "start_at": "2026-01-10 09:00:00",
        "end_at": "2026-01-10 17:00:00",
        "punch_in": "2026-01-10 08:55:00",
        "punch_out": "2026-01-10 17:05:00",
        "memo": null,
        "created_at": "2026-01-08 12:00:00",
        "updated_at": "2026-01-15 14:20:35"
    }
}
```

### Error Responses

#### 404 Not Found - 任務不存在
```json
{
    "msg": "task not found",
    "res": null
}
```

#### 403 Forbidden - 無權限操作
```json
{
    "msg": "unauthorized",
    "res": null
}
```

**原因**：只有任務的執行者（user_id）可以確認結算

#### 400 Bad Request - 任務尚未結算
```json
{
    "msg": "task not settled yet",
    "res": null
}
```

**原因**：任務尚未執行結算（fee != 1）

#### 400 Bad Request - 已經確認過
```json
{
    "msg": "settlement already confirmed",
    "res": null
}
```

**原因**：該任務結算已經確認過（settlement_confirmed == 1）

### Response Fields Description

#### Task Object
- `id`: 任務ID
- `job_id`: 關聯的工作ID
- `user_id`: 執行任務的用戶ID（員工）
- `wage`: 時薪或日薪金額
- `status`: 任務狀態（1: 待確認, 2: 待簽署, 3: 進行中, 4: 已完成）
- `fee`: 結算狀態（0: 未結算, 1: 已結算）
- `pay_way`: 支付方式（0: 現金, 1: 轉帳）
- `checkout_wage`: 結算總金額（包含加班費）
- `settlement_at`: 結算時間
- `settlement_confirmed`: 結算確認狀態（0: 未確認, 1: 已確認）
- `settlement_confirmed_at`: 結算確認時間
- `start_at`: 任務開始時間
- `end_at`: 任務結束時間
- `punch_in`: 實際上班打卡時間
- `punch_out`: 實際下班打卡時間
- `memo`: 備註
- `created_at`: 建立時間
- `updated_at`: 更新時間

### Business Logic

#### 驗證流程
1. **身份驗證**：檢查當前登入用戶
2. **任務存在性**：驗證任務ID是否存在
3. **權限驗證**：確認操作者是任務的執行者（user_id）
4. **結算狀態**：確認任務已完成結算（fee = 1）
5. **確認狀態**：確認尚未確認過（settlement_confirmed = 0）

#### 執行動作
- 設定 `settlement_confirmed = 1`
- 記錄 `settlement_confirmed_at` 為當前時間
- 返回更新後的完整任務資料

### Usage Notes

1. **執行順序**：必須先執行結算（checkout）才能確認結算
2. **單次確認**：每個任務只能確認一次，確認後無法撤銷
3. **權限限制**：只有任務的執行者（員工）可以確認，雇主無法代為確認
4. **時間記錄**：系統自動記錄確認時間，用於審計追蹤

