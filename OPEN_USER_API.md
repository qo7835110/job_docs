# 開放用戶查詢 API

## Base URL
```
/v1/open/
```

## 1. 根據身分證字號查詢用戶資訊

### Endpoint
```
GET /user/idNumber
```

### Description
根據身分證字號查詢用戶的公開資訊，包含用戶基本資料、技能、學歷、評論及應徵紀錄等詳細信息。

**注意：查詢參數使用底線命名 `id_number`，非駝峰式命名。**

### Headers
```
Content-Type: application/json
Accept: application/json
```

### Query Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id_number | string | Yes | 用戶的身分證字號 |

### Response Success (200 OK)
```json
{
    "msg": "success",
    "res": {
        "id": 138,
        "id_number": "A123456789",
        "name": "林佾廷",
        "en_name": null,
        "sex": 0,
        "birthday": "2001-12-24",
        "seniority": 1,
        "marital_status": 0,
        "phone": {
            "phone_1": "0908888888"
        },
        "special_id": 0,
        "email": "ggg2201@gmail.com",
        "contact_time": [
            "09:00 - 18:00"
        ],
        "education": {
            "school": "法大王超級學院",
            "status": 0,
            "degree": 4,
            "department": "行銷管理"
        },
        "skills": [
            null,
            "行銷管理"
        ],
        "mug_shot_img": "images/9ii6HiyYlbh8wFlEHQEKfJW7OJRb0rdfWOkfJuLf.png",
        "address": "新北市板橋區",
        "district": null,
        "county": null,
        "public_info": null,
        "public_contact_info": null,
        "privacy_setting": {
            "job_match": false,
            "match_inform": false,
            "public_profile": false,
            "jobs": false,
            "resumes": true
        },
        "last_login": "2026-01-15 19:42:24",
        "sso_user": 0,
        "created_at": "2025-12-19 00:02:38",
        "updated_at": "2025-12-19 00:41:11",
        "job_comments": [],
        "task_comments": [],
        "job_received_comments": [],
        "task_received_comments": [],
        "applied_jobs": []
    }
}
```

### Response Error (404 Not Found)
```json
{
    "msg": "user not found",
    "res": null
}
```

### Example Request
```bash
curl -X GET "https://yourdomain.com/v1/open/user/idNumber?id_number=A123456789" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json"
```

### Response Fields Description

#### User Basic Info
- `id`: 用戶ID
- `id_number`: 身分證字號
- `name`: 姓名
- `en_name`: 英文名（可為 null）
- `sex`: 性別代碼（0: 女, 1: 男）
- `birthday`: 生日（格式：YYYY-MM-DD）
- `seniority`: 年資（數字）
- `marital_status`: 婚姻狀態代碼（0: 未婚, 1: 已婚）
- `phone`: 電話物件
  - `phone_1`: 主要電話號碼
- `special_id`: 特殊身份代碼（0: 無, 其他數字代表不同身份）
- `email`: 電子郵件
- `contact_time`: 聯絡時間陣列（字串陣列，如 `["09:00 - 18:00"]`）
- `education`: 學歷物件
  - `school`: 學校名稱
  - `status`: 就學狀態代碼（0: 畢業, 1: 在學, 2: 肄業）
  - `degree`: 學位代碼（1: 國中, 2: 高中職, 3: 專科, 4: 大學, 5: 碩士, 6: 博士）
  - `department`: 科系/部門
- `skills`: 技能陣列（字串陣列，可能包含 null）
- `mug_shot_img`: 大頭照相對路徑
- `address`: 地址
- `county`: 縣市（可為 null）
- `district`: 區域（可為 null）
- `public_info`: 公開資訊設定（可為 null）
- `public_contact_info`: 公開聯絡方式設定（可為 null）
- `privacy_setting`: 隱私設定物件
  - `job_match`: 工作媒合（boolean）
  - `match_inform`: 媒合通知（boolean）
  - `public_profile`: 公開個人檔案（boolean）
  - `jobs`: 公開工作（boolean）
  - `resumes`: 公開履歷（boolean）
- `last_login`: 最後登入時間
- `sso_user`: SSO 用戶標記（0: 否, 1: 是）
- `created_at`: 建立時間
- `updated_at`: 更新時間
