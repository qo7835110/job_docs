# 評論功能 API 文件

### Comment（評論）

**資料表欄位：**
- `id` - 評論 ID
- `user_id` - 評論者 ID
- `job_id` - 工作 ID（與 task_id 二選一）
- `task_id` - 任務 ID（與 job_id 二選一）
- `type` - 評論類型
- `score` - 評分（0-5 分）
- `content` - 評論內容
- `img_paths` - 評論圖片路徑（JSON 陣列格式）
- `created_at` - 建立時間
- `updated_at` - 更新時間

**關聯：**
- `user()` - 多對一關聯到 User
- `job()` - 多對一關聯到 Job
- `task()` - 多對一關聯到 Task

---

## API 端點

### 1. 獲取評論列表

**端點：** `GET /v1/comments`

**說明：** 獲取評論列表，可根據使用者、任務或工作進行篩選。

**認證：** 需要（`auth:sanctum`）

**請求參數：**
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| user_id | integer | 否 | 使用者 ID，不提供則使用當前登入使用者 ID |
| task_id | integer | 否 | 任務 ID，用於篩選特定任務的評論 |
| job_id | integer | 否 | 工作 ID，用於篩選特定工作的評論 |

**驗證規則：**
- `user_id`：必須存在於 users 資料表
- `task_id`：必須存在於 tasks 資料表
- `job_id`：必須存在於 jobs 資料表

**查詢邏輯：**
- 如果提供 `task_id`：返回指定使用者對該任務的評論
- 如果提供 `job_id`：返回指定使用者對該工作的評論
- 如果都不提供：返回該使用者的所有評論

**回應格式：**
```json
{
  "msg": "success",
  "res": {
    "data": [
      {
        "id": 1,
        "user_id": 2,
        "job_id": 10,
        "task_id": null,
        "type": 1,
        "score": 4.5,
        "content": "工作環境很好，同事相處融洽。",
        "img_paths": [
          "images/comment1.jpg",
          "images/comment2.jpg"
        ],
        "created_at": "2026-01-05 10:30:00",
        "updated_at": "2026-01-05 10:30:00"
      }
    ]
  }
}
```

**錯誤回應：**
```json
{
  "msg": "validation error",
  "res": {
    "user_id": ["The selected user id is invalid."],
    "task_id": ["The selected task id is invalid."]
  }
}
```
- HTTP Status: 422 Unprocessable Entity

---

### 2. 獲取評論（開放端點）

**端點：** `GET /v1/open/comments/{user_id}`

**說明：** 公開端點，獲取指定使用者的所有評論。

**認證：** 不需要

**路徑參數：**
| 參數 | 類型 | 說明 |
|------|------|------|
| user_id | integer | 使用者 ID（必須為數字） |

**回應格式：**
```json
{
  "msg": "success",
  "res": {
    "data": [
      {
        "id": 1,
        "user_id": 2,
        "job_id": 10,
        "task_id": null,
        "type": 1,
        "score": 4.5,
        "content": "工作環境很好，同事相處融洽。",
        "img_paths": [
          "images/comment1.jpg",
          "images/comment2.jpg"
        ],
        "created_at": "2026-01-05 10:30:00",
        "updated_at": "2026-01-05 10:30:00"
      }
    ]
  }
}
```

---

### 3. 建立評論

**端點：** `POST /v1/comment/create`

**說明：** 建立新的評論。使用者只能對同一個工作或任務評論一次。

**認證：** 需要（`auth:sanctum`）

**請求參數：**
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| job_id | integer | 條件必填* | 工作 ID |
| task_id | integer | 條件必填* | 任務 ID |
| type | integer | 是 | 評論類型 |
| score | numeric | 是 | 評分（0-5 分） |
| content | string | 否 | 評論內容 |
| images | file[] | 否 | 評論圖片（可上傳多張） |

**條件必填說明：**
- `job_id` 和 `task_id` 必須提供其中一個
- 不能同時為空，但可以只提供一個

**驗證規則：**
- `job_id`：
  - 當沒有提供 task_id 時必填
  - 必須存在於 jobs 資料表
- `task_id`：
  - 當沒有提供 job_id 時必填
  - 必須存在於 tasks 資料表
- `type`：必填，整數
- `score`：必填，數字，範圍 0-5
- `content`：選填，字串
- `images`：選填，陣列
- `images.*`：
  - 必須是圖片檔案
  - 格式：jpeg, png, jpg, gif, svg
  - 大小上限：1024 KB (1 MB)

**請求範例（multipart/form-data）：**
```
POST /v1/comment/create
Content-Type: multipart/form-data

job_id: 10
type: 1
score: 4.5
content: "工作環境很好，同事相處融洽。"
images[0]: [圖片檔案]
images[1]: [圖片檔案]
```

**成功回應：**
```json
{
  "msg": "success",
  "res": {
    "id": 15,
    "user_id": 2,
    "job_id": 10,
    "task_id": null,
    "type": 1,
    "score": 4.5,
    "content": "工作環境很好，同事相處融洽。",
    "img_paths": [
      "images/abc123.jpg",
      "images/def456.jpg"
    ],
    "created_at": "2026-01-07 14:20:00",
    "updated_at": "2026-01-07 14:20:00"
  }
}
```
- HTTP Status: 200 OK

**錯誤回應：**

1. **驗證錯誤：**
```json
{
  "msg": "validation error",
  "res": {
    "job_id": ["The job id field is required when task id is not present."],
    "score": ["The score must be between 0 and 5."],
    "images.0": ["The images.0 must be an image.", "The images.0 must not be greater than 1024 kilobytes."]
  }
}
```
- HTTP Status: 422 Unprocessable Entity

2. **重複評論錯誤：**
```json
{
  "msg": "You have already commented on this task.",
  "res": null
}
```
或
```json
{
  "msg": "You have already commented on this job.",
  "res": null
}
```
- HTTP Status: 409 Conflict
- 說明：同一使用者不能對同一個工作或任務重複評論

---
