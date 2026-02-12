# 使用者建立與更新 API

## Base URL

```
/v1
```

---

## 1. 建立使用者 (store)

### Endpoint

```
POST /user/create
```

### 說明

建立新使用者帳號。可建立一般使用者或企業使用者。

### Headers

```
Content-Type: application/json
Accept: application/json
```

### Request Body

#### 必填欄位 (User)

| 參數                  | 類型    | 說明                          |
| --------------------- | ------- | ----------------------------- |
| email                 | string  | 電子信箱 (需唯一)             |
| password              | string  | 密碼                          |
| password_confirmation | string  | 確認密碼 (需與 password 相同) |
| is_corporate          | boolean | 是否為企業帳號 (true/false)   |

#### 選填欄位 (一般使用者 user_info)

> 當 is_corporate = false 時可傳入。

| 參數                           | 類型    | 說明                                              |
| ------------------------------ | ------- | ------------------------------------------------- |
| id_number                      | string  | 身分證字號 (唯一，格式 /^[A-Z]{1}[1-2]{1}\d{8}$/) |
| name                           | string  | 姓名                                              |
| en_name                        | string  | 英文名 (可為 null)                                |
| sex                            | integer | 性別代碼 (0-2)                                    |
| birthday                       | string  | 生日                                              |
| seniority                      | integer | 年資                                              |
| marital_status                 | integer | 婚姻狀態 (0-4)                                    |
| phone.phone_1                  | string  | 主要電話                                          |
| contact_time                   | string  | 方便聯絡時間                                      |
| special_id                     | integer | 特殊身分代碼                                      |
| education.school               | string  | 學校名稱                                          |
| education.status               | integer | 就學狀態                                          |
| education.department           | string  | 科系                                              |
| education.degree               | integer | 學位                                              |
| skills                         | array   | 技能陣列                                          |
| mug_shot_img                   | string  | 大頭照圖片路徑                                    |
| address                        | string  | 地址                                              |
| county                         | string  | 縣市                                              |
| district                       | string  | 區域                                              |
| privacy_setting.job_match      | boolean | 隱私設定：工作媒合                                |
| privacy_setting.match_inform   | boolean | 隱私設定：媒合通知                                |
| privacy_setting.public_profile | boolean | 隱私設定：公開個人資料                            |
| privacy_setting.resumes        | boolean | 隱私設定：公開履歷                                |
| privacy_setting.jobs           | boolean | 隱私設定：公開工作                                |

#### 選填欄位 (企業使用者 company_info)

> 當 is_corporate = true 時可傳入。

| 參數          | 類型    | 說明         |
| ------------- | ------- | ------------ |
| name          | string  | 公司名稱     |
| tax_id        | string  | 統一編號     |
| owner_name    | string  | 負責人       |
| avatar        | string  | 公司頭像路徑 |
| industry      | string  | 產業代碼     |
| members       | integer | 員工人數     |
| contact       | string  | 聯絡人       |
| phone.phone_1 | string  | 主要電話     |
| fax           | string  | 傳真         |
| email         | string  | 公司信箱     |
| address       | string  | 地址         |
| county        | string  | 縣市         |
| district      | string  | 區域         |
| website       | string  | 公司網站     |
| contact_time  | string  | 方便聯絡時間 |
| others        | array   | 其他資訊     |
| others.\*     | string  | 其他資訊項目 |
| description   | string  | 公司描述     |
| benefit       | string  | 福利         |
| remark        | string  | 備註         |

### Request 範例 (一般使用者)

```json
{
    "email": "user@example.com",
    "password": "password123",
    "password_confirmation": "password123",
    "is_corporate": false,
    "user_info": {
        "name": "王小明",
        "id_number": "A123456789",
        "sex": 1,
        "phone": {
            "phone_1": "0912345678"
        },
        "skills": ["PHP", "Laravel"],
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
}
```

### Request 範例 (企業使用者)

```json
{
    "email": "corp@example.com",
    "password": "password123",
    "password_confirmation": "password123",
    "is_corporate": true,
    "company_info": {
        "name": "範例股份有限公司",
        "tax_id": "12345678",
        "owner_name": "王大明",
        "industry": "123456",
        "members": 100,
        "phone": {
            "phone_1": "0223456789"
        },
        "address": "台北市信義區松高路1號",
        "website": "https://example.com",
        "description": "公司簡介"
    }
}
```

### Success Response (200)

```json
{
    "msg": "success",
    "res": {
        "id": 42,
        "email": "user@example.com",
        "is_corporate": 0,
        "last_login": "2026-02-13 10:30:00",
        "created_at": "2026-02-13 10:30:00",
        "updated_at": "2026-02-13 10:30:00"
    }
}
```

### Error Response (422)

```json
{
    "msg": "validation error",
    "errors": {
        "email": ["The email has already been taken."],
        "password": ["The password confirmation does not match."],
        "is_corporate": ["The is corporate field is required."]
    }
}
```

---

## 2. 更新使用者 (update)

### Endpoint

```
POST /user/update
```

### 說明

更新登入者的個人資料。此端點需登入授權。

### Headers

```
Authorization: Bearer {access_token}
Content-Type: application/json
Accept: application/json
```

### Request Body

> 皆為選填欄位。若未提供 county 或 district，系統會依 address 嘗試解析。

| 參數                           | 類型    | 說明                                              |
| ------------------------------ | ------- | ------------------------------------------------- |
| id_number                      | string  | 身分證字號 (唯一，格式 /^[A-Z]{1}[1-2]{1}\d{8}$/) |
| name                           | string  | 姓名                                              |
| en_name                        | string  | 英文名 (可為 null)                                |
| sex                            | integer | 性別代碼 (0-2)                                    |
| birthday                       | string  | 生日                                              |
| seniority                      | integer | 年資                                              |
| marital_status                 | integer | 婚姻狀態 (0-4)                                    |
| phone.phone_1                  | string  | 主要電話                                          |
| contact_time                   | string  | 方便聯絡時間                                      |
| special_id                     | integer | 特殊身分代碼                                      |
| education.school               | string  | 學校名稱                                          |
| education.status               | integer | 就學狀態                                          |
| education.department           | string  | 科系                                              |
| education.degree               | integer | 學位                                              |
| skills                         | array   | 技能陣列                                          |
| mug_shot_img                   | string  | 大頭照圖片路徑                                    |
| address                        | string  | 地址                                              |
| county                         | string  | 縣市                                              |
| district                       | string  | 區域                                              |
| privacy_setting.job_match      | boolean | 隱私設定：工作媒合                                |
| privacy_setting.match_inform   | boolean | 隱私設定：媒合通知                                |
| privacy_setting.public_profile | boolean | 隱私設定：公開個人資料                            |
| privacy_setting.resumes        | boolean | 隱私設定：公開履歷                                |
| privacy_setting.jobs           | boolean | 隱私設定：公開工作                                |

### Request 範例

```json
{
    "name": "王小明",
    "phone": {
        "phone_1": "0912345678"
    },
    "skills": ["PHP", "Laravel"],
    "address": "台北市大安區信義路四段1號",
    "privacy_setting": {
        "job_match": true,
        "match_inform": true,
        "public_profile": true,
        "resumes": true,
        "jobs": false
    }
}
```

### Success Response (200)

```json
{
    "msg": "success",
    "res": {
        "id": 42,
        "name": "王小明",
        "mug_shot_img": "images/abc123.png",
        "updated_at": "2026-02-13 11:00:00"
    }
}
```

### Error Response (422)

```json
{
    "msg": "validation error",
    "errors": {
        "id_number": ["The id number has already been taken."],
        "skills": ["The skills must be an array."]
    }
}
```

## 欄位說明

### 性別 (sex)

- **0**: 男性
- **1**: 女性

### 婚姻狀態 (marital_status)

- **0**: 未婚
- **1**: 已婚
- **2**: 離婚
- **3**: 喪偶
