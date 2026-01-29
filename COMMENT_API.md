# 評論功能 API 文件

### Comment（評論）

**資料表欄位：**
- `id` - 評論 ID
- `user_id` - 評論者 ID
- `job_id` - 工作 ID（與 task_id 二選一）
- `task_id` - 任務 ID（與 job_id 二選一）
- `type` - 評論類型（0: 員工評論業主, 1: 業主評論員工）
- `score` - 總評分（所有項目分數總和）
- `content` - 評論內容
- `img_paths` - 評論圖片路徑（JSON 格式：{"data": ["path1", "path2"]}）
- `created_at` - 建立時間
- `updated_at` - 更新時間

**關聯：**
- `user()` - 多對一關聯到 User
- `job()` - 多對一關聯到 Job
- `task()` - 多對一關聯到 Task
- `items()` - 一對多關聯到 CommentItem（評論項目）

### CommentItem（評論項目）

**資料表欄位：**
- `id` - 評論項目 ID
- `comment_id` - 評論 ID（外鍵）
- `item_name` - 評論項目名稱
- `score` - 項目分數（0-5 分）
- `created_at` - 建立時間
- `updated_at` - 更新時間

**關聯：**
- `comment()` - 多對一關聯到 Comment

**評論項目定義：**

業主評論員工（type=1）必須包含：
- 誠實守信
- 主動體貼
- 專業品質
- 高效自律

員工評論業主（type=0）必須包含：
- 報酬準時
- 尊重休息
- 平等溝通
- 安全保障

---

## API 端點

### 0. 獲取評論選項

**端點：** `GET /v1/comment/options`

**說明：** 獲取評論項目的預設選項。

**認證：** 需要（`auth:sanctum`）

**回應格式：**
```json
{
  "msg": "success",
  "res": {
    "owner": [
      {"item_name": "誠實守信", "score": 0},
      {"item_name": "主動體貼", "score": 0},
      {"item_name": "專業品質", "score": 0},
      {"item_name": "高效自律", "score": 0}
    ],
    "employee": [
      {"item_name": "報酬準時", "score": 0},
      {"item_name": "尊重休息", "score": 0},
      {"item_name": "平等溝通", "score": 0},
      {"item_name": "安全保障", "score": 0}
    ]
  }
}
```

---

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
        "job_id": null,
        "task_id": 10,
        "type": 1,
        "score": 18.5,
        "content": "工作態度認真負責，準時完成所有任務。",
        "img_paths": {
          "data": [
            "images/comment1.jpg",
            "images/comment2.jpg"
          ]
        },
        "items": [
          {
            "id": 1,
            "comment_id": 1,
            "item_name": "誠實守信",
            "score": 5,
            "created_at": "2026-01-22 10:30:00",
            "updated_at": "2026-01-22 10:30:00"
          },
          {
            "id": 2,
            "comment_id": 1,
            "item_name": "主動體貼",
            "score": 4.5,
            "created_at": "2026-01-22 10:30:00",
            "updated_at": "2026-01-22 10:30:00"
          },
          {
            "id": 3,
            "comment_id": 1,
            "item_name": "專業品質",
            "score": 5,
            "created_at": "2026-01-22 10:30:00",
            "updated_at": "2026-01-22 10:30:00"
          },
          {
            "id": 4,
            "comment_id": 1,
            "item_name": "高效自律",
            "score": 4,
            "created_at": "2026-01-22 10:30:00",
            "updated_at": "2026-01-22 10:30:00"
          }
        ],
        "created_at": "2026-01-22 10:30:00",
        "updated_at": "2026-01-22 10:30:00"
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
        "type": 0,
        "score": 17.5,
        "content": "雇主準時支付薪資，工作環境安全。",
        "img_paths": {
          "data": [
            "images/comment1.jpg",
            "images/comment2.jpg"
          ]
        },
        "items": [
          {
            "id": 5,
            "comment_id": 1,
            "item_name": "報酬準時",
            "score": 5,
            "created_at": "2026-01-22 10:30:00",
            "updated_at": "2026-01-22 10:30:00"
          },
          {
            "id": 6,
            "comment_id": 1,
            "item_name": "尊重休息",
            "score": 4,
            "created_at": "2026-01-22 10:30:00",
            "updated_at": "2026-01-22 10:30:00"
          },
          {
            "id": 7,
            "comment_id": 1,
            "item_name": "平等溝通",
            "score": 4.5,
            "created_at": "2026-01-22 10:30:00",
            "updated_at": "2026-01-22 10:30:00"
          },
          {
            "id": 8,
            "comment_id": 1,
            "item_name": "安全保障",
            "score": 4,
            "created_at": "2026-01-22 10:30:00",
            "updated_at": "2026-01-22 10:30:00"
          }
        ],
        "created_at": "2026-01-22 10:30:00",
        "updated_at": "2026-01-22 10:30:00"
      }
    ]
  }
}
```

---

### 3. 建立評論

**端點：** `POST /v1/comment/create`

**說明：** 建立新的評論。使用者只能對同一個工作或任務評論一次。評論必須包含所有必填項目。

**認證：** 需要（`auth:sanctum`）

**請求參數：**
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| job_id | integer | 條件必填* | 工作 ID |
| task_id | integer | 條件必填* | 任務 ID |
| type | integer | 是 | 評論類型（0: 員工評論業主, 1: 業主評論員工） |
| content | string | 否 | 評論內容 |
| images | file[] | 否 | 評論圖片（可上傳多張） |
| items | array | 是 | 評論項目陣列 |
| items.*.item_name | string | 是 | 項目名稱 |
| items.*.score | numeric | 是 | 項目分數（0-5） |

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
- `type`：必填，整數（0 或 1）
- `content`：選填，字串
- `images`：選填，陣列
- `images.*`：
  - 必須是圖片檔案
  - 格式：jpeg, png, jpg, gif, svg
  - 大小上限：1024 KB (1 MB)
- `items`：必填，陣列
- `items.*.item_name`：必填，字串
- `items.*.score`：必填，數字，範圍 0-5

**必填項目檢查：**
- type = 1（業主評論員工）必須包含：誠實守信、主動體貼、專業品質、高效自律
- type = 0（員工評論業主）必須包含：報酬準時、尊重休息、平等溝通、安全保障

**請求範例（JSON）：**
```json
{
  "task_id": 1,
  "type": 1,
  "content": "工作態度認真負責，準時完成所有任務。",
  "items": [
    {"item_name": "誠實守信", "score": 5},
    {"item_name": "主動體貼", "score": 4.5},
    {"item_name": "專業品質", "score": 5},
    {"item_name": "高效自律", "score": 4}
  ]
}
```

**請求範例（multipart/form-data - 帶圖片）：**
```
POST /v1/comment/create
Content-Type: multipart/form-data

task_id: 1
type: 1
content: "工作態度認真負責，準時完成所有任務。"
items[0][item_name]: "誠實守信"
items[0][score]: 5
items[1][item_name]: "主動體貼"
items[1][score]: 4.5
items[2][item_name]: "專業品質"
items[2][score]: 5
items[3][item_name]: "高效自律"
items[3][score]: 4
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
    "job_id": null,
    "task_id": 1,
    "type": 1,
    "score": 18.5,
    "content": "工作態度認真負責，準時完成所有任務。",
    "img_paths": {
      "data": [
        "images/abc123.jpg",
        "images/def456.jpg"
      ]
    },
    "items": [
      {
        "id": 20,
        "comment_id": 15,
        "item_name": "誠實守信",
        "score": 5,
        "created_at": "2026-01-22 14:20:00",
        "updated_at": "2026-01-22 14:20:00"
      },
      {
        "id": 21,
        "comment_id": 15,
        "item_name": "主動體貼",
        "score": 4.5,
        "created_at": "2026-01-22 14:20:00",
        "updated_at": "2026-01-22 14:20:00"
      },
      {
        "id": 22,
        "comment_id": 15,
        "item_name": "專業品質",
        "score": 5,
        "created_at": "2026-01-22 14:20:00",
        "updated_at": "2026-01-22 14:20:00"
      },
      {
        "id": 23,
        "comment_id": 15,
        "item_name": "高效自律",
        "score": 4,
        "created_at": "2026-01-22 14:20:00",
        "updated_at": "2026-01-22 14:20:00"
      }
    ],
    "created_at": "2026-01-22 14:20:00",
    "updated_at": "2026-01-22 14:20:00"
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
    "items": ["The items field is required."],
    "items.0.score": ["The items.0.score must be between 0 and 5."],
    "images.0": ["The images.0 must be an image.", "The images.0 must not be greater than 1024 kilobytes."]
  }
}
```
- HTTP Status: 422 Unprocessable Entity

2. **缺少必填項目：**
```json
{
  "msg": "Missing required comment item: 誠實守信",
  "res": null
}
```
- HTTP Status: 422 Unprocessable Entity
- 說明：根據 type 不同，必須包含所有必填的評論項目

3. **重複評論錯誤：**
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
