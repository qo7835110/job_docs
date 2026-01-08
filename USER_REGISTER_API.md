# 使用者註冊 API

## 概述

建立新使用者帳號的 API 文檔。

---

## 使用者註冊

### 端點

```
POST /user/create
```

### 請求標頭

```
Content-Type: application/json
```

### 請求參數

#### 必填欄位

| 參數                           | 類型    | 說明                           |
| ------------------------------ | ------- | ------------------------------ |
| id_number                      | string  | 身分證字號（唯一）             |
| name                           | string  | 姓名                           |
| sex                            | integer | 性別（0=女, 1=男）             |
| birthday                       | string  | 生日（格式：YYYY-MM-DD）       |
| seniority                      | integer | 年資（年）                     |
| marital_status                 | integer | 婚姻狀態（0=未婚, 1=已婚）     |
| phone.phone_1                  | string  | 手機號碼                       |
| email                          | string  | 電子信箱（唯一）               |
| password                       | string  | 密碼                           |
| password_confirmation          | string  | 確認密碼（需與 password 相同） |
| skills                         | array   | 技能列表                       |
| address                        | string  | 地址                           |
| privacy_setting.job_match      | boolean | 隱私設定：工作媒合             |
| privacy_setting.match_inform   | boolean | 隱私設定：媒合通知             |
| privacy_setting.public_profile | boolean | 隱私設定：公開個人資料         |
| privacy_setting.resumes        | boolean | 隱私設定：履歷                 |
| privacy_setting.jobs           | boolean | 隱私設定：工作                 |

#### 選填欄位

| 參數                 | 類型    | 說明           |
| -------------------- | ------- | -------------- |
| en_name              | string  | 英文名字       |
| special_id           | integer | 特殊身分       |
| education.school     | string  | 學校名稱       |
| education.status     | integer | 就學狀態       |
| education.department | string  | 科系           |
| education.degree     | integer | 學位           |
| mug_shot_img         | string  | 大頭照圖片路徑 |
| county               | string  | 縣市           |
| district             | string  | 區域           |

### 請求範例

#### 完整範例

```json
{
    "id_number": "A123456789",
    "name": "王小明",
    "en_name": "Ming Wang",
    "sex": 1,
    "birthday": "1990-05-15",
    "seniority": 5,
    "marital_status": 0,
    "phone": {
        "phone_1": "0912345678"
    },
    "special_id": null,
    "email": "ming.wang@example.com",
    "password": "password123",
    "password_confirmation": "password123",
    "education": {
        "school": "國立台灣大學",
        "status": 1,
        "department": "資訊工程學系",
        "degree": 2
    },
    "skills": ["JavaScript", "Python", "React"],
    "mug_shot_img": "https://example.com/avatar.jpg",
    "address": "台北市大安區信義路四段1號",
    "county": "台北市",
    "district": "大安區",
    "privacy_setting": {
        "job_match": true,
        "match_inform": true,
        "public_profile": true,
        "resumes": true,
        "jobs": false
    }
}
```

#### 最簡範例

```json
{
    "id_number": "A123456789",
    "name": "痴痴痴",
    "sex": 1,
    "birthday": "2026-01-01",
    "seniority": 0,
    "marital_status": 0,
    "phone": {
        "phone_1": "0912666666"
    },
    "email": "qweasdzxc@gmail.com",
    "skills": [""],
    "address": "aaaaaaaa",
    "privacy_setting": {
        "job_match": false,
        "match_inform": false,
        "public_profile": false,
        "jobs": false,
        "resumes": false
    },
    "password": "123456789",
    "password_confirmation": "123456789"
}
```

### 成功回應 (200)

```json
{
    "msg": "success",
    "res": {
        "id": 42,
        "id_number": "A123456789",
        "name": "王小明",
        "en_name": "Ming Wang",
        "sex": 1,
        "birthday": "1990-05-15",
        "seniority": 5,
        "marital_status": 0,
        "phone": {
            "phone_1": "0912345678"
        },
        "special_id": null,
        "email": "ming.wang@example.com",
        "education": {
            "school": "國立台灣大學",
            "status": 1,
            "department": "資訊工程學系",
            "degree": 2
        },
        "skills": ["JavaScript", "Python", "React"],
        "mug_shot_img": "https://example.com/avatar.jpg",
        "address": "台北市大安區信義路四段1號",
        "county": "台北市",
        "district": "大安區",
        "privacy_setting": {
            "job_match": true,
            "match_inform": true,
            "public_profile": true,
            "resumes": true,
            "jobs": false
        },
        "last_login": "2026-01-08 10:30:00",
        "created_at": "2026-01-08 10:30:00",
        "updated_at": "2026-01-08 10:30:00"
    }
}
```

### 錯誤回應

#### 驗證失敗 (422)

```json
{
    "message": "The given data was invalid.",
    "errors": {
        "email": ["The email has already been taken."],
        "id_number": ["The id number has already been taken."],
        "password": ["The password confirmation does not match."],
        "skills": ["The skills field is required."]
    }
}
```

### 範例請求

```bash
curl -X POST http://localhost:8000/user/create \
  -H "Content-Type: application/json" \
  -d '{
    "id_number": "A123456789",
    "name": "王小明",
    "sex": 1,
    "birthday": "1990-05-15",
    "seniority": 5,
    "marital_status": 0,
    "phone": {
      "phone_1": "0912345678"
    },
    "email": "ming.wang@example.com",
    "password": "password123",
    "password_confirmation": "password123",
    "skills": ["JavaScript", "Python"],
    "address": "台北市大安區信義路四段1號",
    "privacy_setting": {
      "job_match": true,
      "match_inform": true,
      "public_profile": true,
      "resumes": true,
      "jobs": false
    }
  }'
```

---

## 欄位說明

### 性別 (sex)

-   **0**: 男性
-   **1**: 女性

### 婚姻狀態 (marital_status)

-   **0**: 未婚
-   **1**: 已婚
-   **2**: 離婚
-   **3**: 喪偶

