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

建立新使用者帳號。支援兩種使用者類型：

- **個人使用者** (`is_corporate = false` 或未填)：需提供 `user_info`
- **企業使用者** (`is_corporate = true`)：需提供 `company_info`，可同時上傳公司圖片

### Headers

- **個人使用者**（純 JSON）：

```
Content-Type: application/json
Accept: application/json
```

- **企業使用者**（需上傳圖片時）：

```
Content-Type: multipart/form-data
Accept: application/json
```

> 企業註冊若不需上傳圖片，也可使用 `application/json`。

### Request Body

#### 必填欄位 (User)

| 參數                  | 類型    | 說明                          |
| --------------------- | ------- | ----------------------------- |
| email                 | string  | 電子信箱 (需唯一)             |
| password              | string  | 密碼                          |
| password_confirmation | string  | 確認密碼 (需與 password 相同) |
| is_corporate          | boolean | 是否為企業帳號 (true/false)   |

#### 選填欄位 (一般使用者 user_info)

> 當 `is_corporate = false` 時需提供。

| 參數                           | 類型    | 說明                                               |
| ------------------------------ | ------- | -------------------------------------------------- |
| id_number                      | string  | 身分證字號 (唯一，格式 /^[A-Z]{1}[1-2]{1}\d{8}$/) |
| name                           | string  | 姓名                                               |
| en_name                        | string  | 英文名 (可為 null)                                 |
| sex                            | integer | 性別代碼 (0-2)                                     |
| birthday                       | string  | 生日                                               |
| seniority                      | integer | 年資                                               |
| marital_status                 | integer | 婚姻狀態 (0-4)                                     |
| phone.phone_1                  | string  | 主要電話                                           |
| contact_time                   | string  | 方便聯絡時間                                       |
| special_identity_id            | integer | 特殊身分 ID（關聯 `special_identities` 表）        |
| education.school               | string  | 學校名稱                                           |
| education.status               | integer | 就學狀態                                           |
| education.department           | string  | 科系                                               |
| education.degree               | integer | 學位                                               |
| skills                         | array   | 技能陣列                                           |
| mug_shot_img                   | string  | 大頭照圖片路徑                                     |
| address                        | string  | 地址                                               |
| county                         | string  | 縣市                                               |
| district                       | string  | 區域                                               |
| privacy_setting.job_match      | boolean | 隱私設定：工作媒合                                 |
| privacy_setting.match_inform   | boolean | 隱私設定：媒合通知                                 |
| privacy_setting.public_profile | boolean | 隱私設定：公開個人資料                             |
| privacy_setting.resumes        | boolean | 隱私設定：公開履歷                                 |
| privacy_setting.jobs           | boolean | 隱私設定：公開工作                                 |

#### 選填欄位 (企業使用者 company_info)

> 當 `is_corporate = true` 時需提供。

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

#### 選填欄位 (企業使用者 company_images — 圖片上傳)

> 當 `is_corporate = true` 且使用 `multipart/form-data` 時可傳入。

| 參數                                           | 類型 | 說明           |
| ---------------------------------------------- | ---- | -------------- |
| company_images[avatar]                         | file | 公司頭像/Logo  |
| company_images[registration_certificate_front] | file | 公司登記證正面 |
| company_images[registration_certificate_back]  | file | 公司登記證反面 |
| company_images[factory_certificate_front]      | file | 工廠登記證正面 |
| company_images[factory_certificate_back]       | file | 工廠登記證反面 |

**圖片規格：**

- **格式**：jpeg, png, jpg, gif, svg
- **大小上限**：64 MB（65536 KB）

> 註冊後亦可使用 `POST /company/uploadImg` 逐張補傳或更新圖片（參考 [COMPANY_IMAGE_API.md](COMPANY_IMAGE_API.md)）。

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

### Request 範例 (企業使用者 — JSON，不含圖片)

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

### Request 範例 (企業使用者 — FormData + 圖片上傳)

#### 使用 curl

```bash
curl -X POST http://localhost:8000/user/create \
  -F "user[email]=corp@example.com" \
  -F "user[password]=password123" \
  -F "user[password_confirmation]=password123" \
  -F "user[is_corporate]=1" \
  -F "company_info[name]=範例股份有限公司" \
  -F "company_info[tax_id]=12345678" \
  -F "company_info[owner_name]=王大明" \
  -F "company_info[industry]=123456" \
  -F "company_info[members]=100" \
  -F "company_info[phone][phone_1]=0223456789" \
  -F "company_info[email]=corp@example.com" \
  -F "company_info[address]=台北市信義區松高路1號" \
  -F "company_info[county]=台北市" \
  -F "company_info[district]=信義區" \
  -F "company_info[website]=https://example.com" \
  -F "company_info[description]=公司簡介" \
  -F "company_images[avatar]=@/path/to/logo.jpg" \
  -F "company_images[registration_certificate_front]=@/path/to/cert_front.jpg" \
  -F "company_images[registration_certificate_back]=@/path/to/cert_back.jpg"
```

#### 使用 JavaScript

```javascript
const formData = new FormData();

// user 欄位
formData.append('user[email]', 'corp@example.com');
formData.append('user[password]', 'password123');
formData.append('user[password_confirmation]', 'password123');
formData.append('user[is_corporate]', '1');

// company_info 欄位
formData.append('company_info[name]', '範例股份有限公司');
formData.append('company_info[tax_id]', '12345678');
formData.append('company_info[owner_name]', '王大明');
formData.append('company_info[industry]', '123456');
formData.append('company_info[members]', '100');
formData.append('company_info[phone][phone_1]', '0223456789');
formData.append('company_info[email]', 'corp@example.com');
formData.append('company_info[address]', '台北市信義區松高路1號');

// company_images 圖片上傳（選填）
formData.append('company_images[avatar]', avatarFile);
formData.append('company_images[registration_certificate_front]', certFrontFile);

fetch('http://localhost:8000/user/create', {
    method: 'POST',
    body: formData,
})
    .then((response) => response.json())
    .then((data) => console.log(data));
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

### Error Responses

#### 驗證失敗 (422)

```json
{
    "msg": "validation error",
    "errors": {
        "email": ["The email has already been taken."],
        "password": ["The password confirmation does not match."]
    }
}
```

#### 個人使用者缺少 user_info (422)

```json
{
    "msg": "user info is required for individual users",
    "res": null
}
```

#### 企業使用者缺少 company_info (422)

```json
{
    "msg": "company info is required for corporate users",
    "res": null
}
```

#### 產業代碼不存在 (404)

```json
{
    "msg": "industry not found",
    "res": null
}
```

#### 圖片驗證失敗 (422)

```json
{
    "msg": "validation error",
    "errors": {
        "company_images.avatar": [
            "The company_images.avatar must be a file of type: jpeg, png, jpg, gif, svg.",
            "The company_images.avatar may not be greater than 65536 kilobytes."
        ]
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
| special_identity_id            | integer | 特殊身分 ID（關聯 `special_identities` 表）       |
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
