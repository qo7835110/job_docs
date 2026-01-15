# Leave Log API Documentation

## 獲取所有假別列表 (Get Leave Types)

### Endpoint
```
GET /v1/leave-type
```

### 描述
獲取系統中所有可用的假別類型。此 API 用於在創建或更新請假記錄時選擇假別。

### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

### Request Parameters
無需請求參數

### Request Example
```
GET /v1/leave-type
```

### Success Response (200 OK)
```json
{
    "msg": "success",
    "res": [
        {
            "id": 1,
            "name": "特休",
            "days": 14,
            "refresh_cycle": "yearly",
            "description": "特別休假",
            "salary_percentage": 100
        },
        {
            "id": 2,
            "name": "事假",
            "days": 7,
            "refresh_cycle": "yearly",
            "description": "因私人事務需要請假",
            "salary_percentage": 0
        },
        {
            "id": 3,
            "name": "病假",
            "days": 30,
            "refresh_cycle": "yearly",
            "description": "因疾病或傷害需要請假",
            "salary_percentage": 50
        },
        {
            "id": 4,
            "name": "婚假",
            "days": 8,
            "refresh_cycle": "once",
            "description": "結婚假期",
            "salary_percentage": 100
        },
        {
            "id": 5,
            "name": "喪假",
            "days": 8,
            "refresh_cycle": "once",
            "description": "親屬喪亡假期",
            "salary_percentage": 100
        }
    ]
}
```

### 假別欄位說明
| 欄位 | 類型 | 描述 |
|------|------|------|
| id | integer | 假別 ID |
| name | string | 假別名稱 |
| days | integer | 可請假天數 |
| refresh_cycle | string | 更新週期：yearly(每年), once(一次性), monthly(每月) |
| description | string | 假別說明 |
| salary_percentage | integer | 薪資百分比 (0-100) |

### 說明
- 此 API 返回所有系統中定義的假別類型
- 需要有效的認證 token
- 假別資料由系統管理員維護
- 使用此 API 獲取的 `id` 可用於創建或更新請假記錄時的 `leave_type_id` 參數

---

## 創建請假記錄 (Create Leave Log)

### Endpoint
```
POST /v1/leave-logs/create
```

### 描述
創建新的請假記錄。當前用戶必須是該任務的擁有者。

### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

### Request Body
| 參數 | 類型 | 必填 | 描述 |
|------|------|------|------|
| task_id | integer | 是 | 任務 ID |
| leave_type_id | integer | 是 | 假別 ID |
| start_at | date | 是 | 請假開始時間 (格式: YYYY-MM-DD HH:mm:ss) |
| end_at | date | 是 | 請假結束時間 (格式: YYYY-MM-DD HH:mm:ss，必須晚於 start_at) |
| reason | string | 否 | 請假原因 |

### Request Example
```json
{
    "task_id": 1,
    "leave_type_id": 2,
    "start_at": "2026-01-15 09:00:00",
    "end_at": "2026-01-16 18:00:00",
    "reason": "個人事務"
}
```

### Success Response (201 Created)
```json
{
    "msg": "leave log created successfully",
    "res": {
        "id": 1,
        "employee_id": 10,
        "owner_id": 5,
        "task_id": 1,
        "leave_type_id": 2,
        "start_at": "2026-01-15 09:00:00",
        "end_at": "2026-01-16 18:00:00",
        "leave_hours": 33,
        "reason": "個人事務",
        "created_at": "2026-01-14T12:00:00.000000Z",
        "updated_at": "2026-01-14T12:00:00.000000Z",
        "employee": {
            "id": 10,
            "name": "張三",
            "email": "user@example.com"
        },
        "leaveType": {
            "id": 2,
            "name": "事假",
            "days": 7,
            "description": "事假說明"
        },
        "task": {
            "id": 1,
            "name": "專案任務",
            "user_id": 10,
            "owner_id": 5
        }
    }
}
```

### Error Responses

#### 422 Validation Error
```json
{
    "msg": "validation error",
    "errors": {
        "task_id": ["The task id field is required."],
        "end_at": ["The end at must be a date after start at."]
    }
}
```

#### 404 Task Not Found
```json
{
    "msg": "task not found",
    "res": null
}
```

#### 403 Unauthorized
```json
{
    "msg": "unauthorized to add leave log for this task",
    "res": null
}
```

### 說明
- 系統會自動計算請假時數 (leave_hours)，最少為 1 小時
- employee_id 和 owner_id 會自動從任務中取得
- 需要有效的認證 token
- 只有任務擁有者可以為該任務新增請假記錄

---

## 更新請假記錄 (Update Leave Log)

### Endpoint
```
POST /v1/leave-logs/update/{id}
```

### 描述
更新現有的請假記錄。

### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

### URL Parameters
| 參數 | 類型 | 描述 |
|------|------|------|
| id | integer | 請假記錄 ID |

### Request Body
| 參數 | 類型 | 必填 | 描述 |
|------|------|------|------|
| task_id | integer | 是 | 任務 ID |
| leave_type_id | integer | 是 | 假別 ID |
| start_at | date | 是 | 請假開始時間 (格式: YYYY-MM-DD HH:mm:ss) |
| end_at | date | 是 | 請假結束時間 (格式: YYYY-MM-DD HH:mm:ss，必須晚於 start_at) |
| reason | string | 否 | 請假原因 |

### Request Example
```json
{
    "task_id": 1,
    "leave_type_id": 3,
    "start_at": "2026-01-15 09:00:00",
    "end_at": "2026-01-17 18:00:00",
    "reason": "更新請假原因"
}
```

### Success Response (200 OK)
```json
{
    "msg": "leave log updated successfully",
    "res": {
        "id": 1,
        "employee_id": 10,
        "owner_id": 5,
        "task_id": 1,
        "leave_type_id": 3,
        "start_at": "2026-01-15 09:00:00",
        "end_at": "2026-01-17 18:00:00",
        "leave_hours": 57,
        "reason": "更新請假原因",
        "created_at": "2026-01-14T12:00:00.000000Z",
        "updated_at": "2026-01-14T13:30:00.000000Z",
        "employee": {
            "id": 10,
            "name": "張三",
            "email": "user@example.com"
        },
        "leaveType": {
            "id": 3,
            "name": "病假",
            "days": 30,
            "description": "病假說明"
        },
        "task": {
            "id": 1,
            "name": "專案任務",
            "user_id": 10,
            "owner_id": 5
        }
    }
}
```

### Error Responses

#### 404 Leave Log Not Found
```json
{
    "msg": "leave log not found",
    "res": null
}
```

#### 404 Task Not Found
```json
{
    "msg": "task not found",
    "res": null
}
```

#### 422 Validation Error
```json
{
    "msg": "validation error",
    "errors": {
        "end_at": ["The end at must be a date after start at."]
    }
}
```

### 說明
- 除了 `reason` 外，所有欄位都是必填的
- 系統會重新計算請假時數 (leave_hours)
- employee_id 和 owner_id 會根據 task_id 自動更新
- 需要有效的認證 token
- 更新操作需要提供完整的資料，無法僅更新部分欄位

---

## 刪除請假記錄 (Delete Leave Log)

### Endpoint
```
DELETE /v1/leave-logs/{id}
```

### 描述
刪除指定的請假記錄。

### Headers
```
Authorization: Bearer {access_token}
```

### URL Parameters
| 參數 | 類型 | 描述 |
|------|------|------|
| id | integer | 請假記錄 ID |

### Request Example
```
DELETE /v1/leave-logs/1
```

### Success Response (200 OK)
```json
{
    "msg": "leave log deleted successfully",
    "res": null
}
```

### Error Responses

#### 404 Leave Log Not Found
```json
{
    "msg": "leave log not found",
    "res": null
}
```

### 說明
- 刪除操作是永久性的，無法復原
- 需要有效的認證 token
- 建議在刪除前確認操作者有適當的權限

---

## 通用說明

### 認證
所有 API 端點都需要使用 Bearer Token 進行認證：
```
Authorization: Bearer {your_access_token}
```

### 日期格式
- 所有日期時間欄位使用格式: `YYYY-MM-DD HH:mm:ss`
- 範例: `2026-01-15 09:00:00`

### 請假時數計算
- 系統自動計算 `start_at` 和 `end_at` 之間的小時數
- 最少計算為 1 小時

### 相關資料模型
- **LeaveLog**: 請假記錄
- **Task**: 任務
- **LeaveType**: 假別
- **User**: 員工/用戶

### HTTP 狀態碼
- `200 OK`: 成功 (GET, DELETE)
- `201 Created`: 成功建立資源 (POST create)
- `403 Forbidden`: 無權限執行操作
- `404 Not Found`: 資源不存在
- `422 Unprocessable Entity`: 驗證錯誤
