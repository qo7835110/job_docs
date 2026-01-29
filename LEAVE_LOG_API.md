# 出勤紀錄 API 文件

## 概述
此 API 提供出勤紀錄和假別管理功能，包含請假記錄的新增、修改、刪除以及假別類型的管理。所有 API 都需要使用 Sanctum Token 進行認證。

**認證 Header:**
```
Authorization: Bearer {token}
```

---

## 假別類型管理 (Leave Types)

### 1. 取得假別列表

**GET** `/v1/leave-types`

取得當前使用者自定義的假別列表。

#### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

#### 請求範例
```bash
curl -X GET "https://example.com/v1/leave-types" \
  -H "Authorization: Bearer {token}"
```

#### 回應範例 (200 OK)
```json
{
  "msg": "success",
  "res": [
    {
      "id": 1,
      "name": "特休假",
      "days": 14,
      "refresh_cycle": "yearly",
      "salary_percentage": 100,
      "description": "年度特別休假",
      "owner_id": 1,
      "is_default": false
    },
    {
      "id": 2,
      "name": "病假",
      "days": 30,
      "refresh_cycle": "yearly",
      "salary_percentage": 50,
      "description": "生病時使用",
      "owner_id": 1,
      "is_default": false
    },
    {
      "id": 3,
      "name": "事假",
      "days": 7,
      "refresh_cycle": "yearly",
      "salary_percentage": 0,
      "description": "因私人事務需要請假",
      "owner_id": 1,
      "is_default": false
    }
  ]
}
```

#### 欄位說明
| 欄位 | 類型 | 描述 |
|------|------|------|
| id | integer | 假別 ID |
| name | string | 假別名稱 |
| days | integer | 假別天數 |
| refresh_cycle | string | 更新週期：yearly(每年)、monthly(每月)、once(一次性) |
| salary_percentage | integer | 薪資百分比 (0-100)，100 為全薪、50 為半薪、0 為無薪 |
| description | string | 假別說明 |
| owner_id | integer | 擁有者 ID |
| is_default | boolean | 是否為預設假別 |

---

### 2. 新增假別

**POST** `/v1/leave-types`

新增自定義的假別類型。

#### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

#### 請求參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| name | string | 是 | 假別名稱 (最多 50 字，同一擁有者下不可重複) |
| days | integer | 是 | 假別天數 (最小 0) |
| refresh_cycle | string | 是 | 更新週期 (yearly/monthly/once) |
| salary_percentage | integer | 是 | 薪資百分比 (0-100) |
| description | string | 否 | 描述 (最多 255 字) |

#### 請求範例
```bash
curl -X POST "https://example.com/v1/leave-types" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "特休假",
    "days": 14,
    "refresh_cycle": "yearly",
    "salary_percentage": 100,
    "description": "年度特別休假"
  }'
```

#### 回應範例 (200 OK)
```json
{
  "msg": "建立成功",
  "res": {
    "id": 1,
    "name": "特休假",
    "days": 14,
    "refresh_cycle": "yearly",
    "salary_percentage": 100,
    "description": "年度特別休假",
    "owner_id": 1,
    "is_default": false
  }
}
```

#### 錯誤回應 (400 Bad Request)
```json
{
  "msg": "The name has already been taken.",
  "status": "error"
}
```

---

### 3. 更新假別

**PUT** `/v1/leave-types/{id}`

更新指定的假別類型。

#### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

#### URL 參數
| 參數 | 類型 | 描述 |
|------|------|------|
| id | integer | 假別 ID |

#### 請求參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| name | string | 否 | 假別名稱 (最多 50 字) |
| days | integer | 否 | 假別天數 (最小 0) |
| refresh_cycle | string | 否 | 更新週期 (yearly/monthly/once) |
| salary_percentage | integer | 否 | 薪資百分比 (0-100) |
| description | string | 否 | 描述 (最多 255 字) |

#### 請求範例
```bash
curl -X PUT "https://example.com/v1/leave-types/1" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "days": 15,
    "description": "更新後的特休假說明"
  }'
```

#### 回應範例 (200 OK)
```json
{
  "msg": "更新成功",
  "res": {
    "id": 1,
    "name": "特休假",
    "days": 15,
    "refresh_cycle": "yearly",
    "salary_percentage": 100,
    "description": "更新後的特休假說明",
    "owner_id": 1,
    "is_default": false
  }
}
```

#### 權限說明
- 只能更新自己建立的假別
- 預設假別的名稱不可修改

#### 錯誤回應
```json
{
  "msg": "無權限修改此假別",
  "status": "error"
}
```

```json
{
  "msg": "找不到該假別",
  "status": "error"
}
```

---

### 4. 刪除假別

**DELETE** `/v1/leave-types/{id}`

刪除指定的假別類型。

#### Headers
```
Authorization: Bearer {access_token}
```

#### URL 參數
| 參數 | 類型 | 描述 |
|------|------|------|
| id | integer | 假別 ID |

#### 請求範例
```bash
curl -X DELETE "https://example.com/v1/leave-types/1" \
  -H "Authorization: Bearer {token}"
```

#### 回應範例 (200 OK)
```json
{
  "msg": "刪除成功"
}
```

#### 權限說明
- 只能刪除自己建立的假別
- 預設假別不可刪除

#### 錯誤回應
```json
{
  "msg": "預設假別不可刪除",
  "status": "error"
}
```

```json
{
  "msg": "無權限刪除此假別",
  "status": "error"
}
```

---


## 請假記錄管理 (Leave Logs)

### 5. 創建請假記錄

**POST** `/v1/leave-logs/create`

創建新的請假記錄。當前用戶必須是該任務的擁有者。

#### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

#### 請求參數
| 參數 | 類型 | 必填 | 描述 |
|------|------|------|------|
| task_id | integer | 是 | 任務 ID |
| leave_type_id | integer | 是 | 假別 ID |
| start_at | datetime | 是 | 請假開始時間 (格式: YYYY-MM-DD HH:mm:ss) |
| end_at | datetime | 是 | 請假結束時間 (格式: YYYY-MM-DD HH:mm:ss，必須晚於 start_at) |
| reason | string | 否 | 請假原因 |

#### 請求範例
```bash
curl -X POST "https://example.com/v1/leave-logs/create" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "task_id": 123,
    "leave_type_id": 1,
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
        "leave_type_id": 1,
        "start_at": "2026-02-01 09:00:00",
        "end_at": "2026-02-01 18:00:00",
        "leave_hours": 9,
        "reason": "身體不適",
        "created_at": "2026-01-29 10:00:00",
        "updated_at": "2026-01-29 10:00:00",
        "employee": {
            "id": 456,
            "name": "員工姓名",
            "email": "employee@example.com"
        },
        "leaveType": {
            "id": 1,
            "name": "特休假",
            "days": 14,
            "salary_percentage": 100
        },
        "task": {
            "id": 123,
            "title": "任務標題"
        }
    }
}
```

#### 欄位說明
- `leave_hours`: 請假小時數 (自動計算，最少 1 小時)
- `employee_id`: 員工 ID (從任務中自動取得)
- `owner_id`: 擁有者 ID (從任務中自動取得)

#### 權限說明
- 必須是任務的擁有者才能新增請假記錄
- 假別必須屬於該任務擁有者或為系統預設

#### 錯誤回應

##### 422 Validation Error
```json
{
    "msg": "validation error",
    "errors": {
        "task_id": ["The task id field is required."],
        "end_at": ["The end at must be a date after start at."]
    }
}
```

##### 404 Task Not Found
```json
{
    "msg": "task not found",
    "res": null
}
```

##### 403 Unauthorized
```json
{
    "msg": "unauthorized to add leave log for this task",
    "res": null
}
```

##### 400 Invalid Leave Type
```json
{
    "msg": "invalid leave type for this task owner",
    "res": null
}
```

---

### 6. 更新請假記錄

**POST** `/v1/leave-logs/update/{id}`

更新現有的請假記錄。

#### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

#### URL 參數
| 參數 | 類型 | 描述 |
|------|------|------|
| id | integer | 請假記錄 ID |

#### 請求參數
| 參數 | 類型 | 必填 | 描述 |
|------|------|------|------|
| task_id | integer | 是 | 任務 ID |
| leave_type_id | integer | 是 | 假別 ID |
| start_at | datetime | 是 | 請假開始時間 (格式: YYYY-MM-DD HH:mm:ss) |
| end_at | datetime | 是 | 請假結束時間 (格式: YYYY-MM-DD HH:mm:ss，必須晚於 start_at) |
| reason | string | 否 | 請假原因 |

#### 請求範例
```bash
curl -X POST "https://example.com/v1/leave-logs/update/1" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "task_id": 123,
    "leave_type_id": 1,
    "start_at": "2026-02-01 09:00:00",
    "end_at": "2026-02-02 18:00:00",
    "reason": "身體不適，需延長請假"
  }'
```

#### 回應範例 (200 OK)
```json
{
    "msg": "leave log updated successfully",
    "res": {
        "id": 1,
        "task_id": 123,
        "employee_id": 456,
        "owner_id": 1,
        "leave_type_id": 1,
        "start_at": "2026-02-01 09:00:00",
        "end_at": "2026-02-02 18:00:00",
        "leave_hours": 33,
        "reason": "身體不適，需延長請假",
        "created_at": "2026-01-29 10:00:00",
        "updated_at": "2026-01-29 11:00:00",
        "employee": {
            "id": 456,
            "name": "員工姓名"
        },
        "leaveType": {
            "id": 1,
            "name": "特休假"
        },
        "task": {
            "id": 123,
            "title": "任務標題"
        }
    }
}
```

#### 說明
- 除了 `reason` 外，所有欄位都是必填的
- 系統會重新計算請假時數 (leave_hours)
- employee_id 和 owner_id 會根據 task_id 自動更新
- 更新操作需要提供完整的資料

#### 錯誤回應

##### 404 Leave Log Not Found
```json
{
    "msg": "leave log not found",
    "res": null
}
```

##### 404 Task Not Found
```json
{
    "msg": "task not found",
    "res": null
}
```

##### 404 Leave Type Not Found
```json
{
    "msg": "leave type not found",
    "res": null
}
```

##### 422 Validation Error
```json
{
    "msg": "validation error",
    "errors": {
        "end_at": ["The end at must be a date after start at."]
    }
}
```

##### 400 Invalid Leave Type
```json
{
    "msg": "invalid leave type for this task owner",
    "res": null
}
```

---

### 7. 刪除請假記錄

**DELETE** `/v1/leave-logs/{id}`

刪除指定的請假記錄。

#### Headers
```
Authorization: Bearer {access_token}
```

#### URL 參數
| 參數 | 類型 | 描述 |
|------|------|------|
| id | integer | 請假記錄 ID |

#### 請求範例
```bash
curl -X DELETE "https://example.com/v1/leave-logs/1" \
  -H "Authorization: Bearer {token}"
```

#### 回應範例 (200 OK)
```json
{
    "msg": "leave log deleted successfully",
    "res": null
}
```

#### 說明
- 刪除操作是永久性的，無法復原
- 建議在刪除前確認操作者有適當的權限

#### 錯誤回應

##### 404 Leave Log Not Found
```json
{
    "msg": "leave log not found",
    "res": null
}
```

---

## 更新週期說明

假別的更新週期決定了假期如何計算和重置：

### yearly (每年更新)
每年度重新計算可用天數，適用於年度特休假、年度病假等。

**範例：** 特休假 14 天，每年 1 月 1 日重置為 14 天。

### monthly (每月更新)
每月重新計算可用天數，適用於月度假別。

**範例：** 生理假 1 天，每月 1 日重置為 1 天。

### once (一次性)
僅能使用一次，不會重新計算，適用於特殊假別。

**範例：** 婚假 8 天，結婚時可請，用完即不再提供。

---

## 薪資百分比說明

薪資百分比表示請假期間員工可獲得的薪資比例：

| 百分比 | 類型 | 說明 |
|--------|------|------|
| 100 | 全薪 | 請假期間全額支付薪資 (特休假、婚假等) |
| 50 | 半薪 | 請假期間支付一半薪資 (部分病假) |
| 0 | 無薪 | 請假期間不支付薪資 (事假) |

可以設定 0-100 之間的任意值來支持不同的薪資計算規則。

---

## 狀態碼說明

| 狀態碼 | 說明 |
|--------|------|
| 200 | 成功 |
| 201 | 建立成功 |
| 400 | 請求參數錯誤 |
| 403 | 無權限 |
| 404 | 資源不存在 |
| 422 | 驗證失敗 |
| 500 | 伺服器錯誤 |

---

## 注意事項

### 1. 權限管理
- 假別只能由建立者自行管理（新增、修改、刪除）
- 請假記錄只能由任務擁有者建立
- 預設假別不可修改名稱或刪除

### 2. 請假時數計算
- 系統自動計算請假小時數
- 最少計算為 1 小時
- 計算方式：結束時間 - 開始時間

### 3. 假別使用限制
- 假別必須屬於任務擁有者或為系統預設
- 確保任務擁有者與假別擁有者一致

### 4. 時間格式
- 所有時間欄位使用格式：`YYYY-MM-DD HH:mm:ss`
- 範例：`2026-02-01 09:00:00`

### 5. 驗證規則
- 假別名稱在同一擁有者下不可重複
- 結束時間必須晚於開始時間
- 薪資百分比範圍：0-100
- 假別天數最小為 0

### 6. 資料關聯
- **LeaveLog**: 請假記錄
- **Task**: 任務
- **LeaveType**: 假別
- **User**: 員工/用戶

### 7. 認證要求
所有 API 端點都需要使用 Bearer Token 進行認證：
```
Authorization: Bearer {your_access_token}
```
