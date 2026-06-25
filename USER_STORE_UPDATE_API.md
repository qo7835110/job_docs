# 使用者建立與更新 API

## Base URL

```
/v1
```

---

## 0. 帳號密碼登入 (authenticate)

### Endpoint

```
POST /user/login
```

### 說明

使用帳號密碼登入。支援兩種帳號格式（二擇一）：

- **Email 登入**：傳送 `email` + `password`
- **識別碼登入**：傳送 `identifier`（身分證字號或統一編號）+ `password`

### Headers

```
Content-Type: application/json
Accept: application/json
```

### Request Body

| 參數       | 類型   | 必填 | 說明 |
| ---------- | ------ | ---- | ---- |
| email      | string | 二擇一 | 電子信箱（與 identifier 擇一） |
| identifier | string | 二擇一 | 身分證字號或統一編號（與 email 擇一） |
| password   | string | ✅   | 密碼 |

> 同時傳入 `email` 與 `identifier` 時，系統優先使用 `email` 登入。

### Request 範例 (Email 登入)

```json
{
    "email": "user@example.com",
    "password": "password123"
}
```

### Request 範例 (識別碼登入)

```json
{
    "identifier": "A123456789",
    "password": "password123"
}
```

### Success Response (200)

```json
{
    "msg": "success",
    "res": {
        "token": "1|xxxxxxxxxxxxxxxx",
        "data": { }
    }
}
```

### Error Response (401)

```json
{
    "msg": "login failed",
    "res": null
}
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
| phone                 | string  | 主要手機號碼 (需唯一，格式 09XXXXXXXX) |
| identifier            | string  | 政府核發識別碼（需唯一）。個人使用者填**身分證字號**（格式 `/^[A-Z][1-2]\d{8}$/`，例：`A123456789`）；企業使用者填**統一編號**（8 碼數字，例：`12345678`） |
| password              | string  | 密碼                          |
| password_confirmation | string  | 確認密碼 (需與 password 相同) |
| is_corporate          | boolean | 是否為企業帳號 (true/false)   |
| otp                   | string  | 手機驗證碼（6 位數字，需先呼叫 `POST /otp/send` 以 `scene=register` 取得） |

> **注意：** `identifier` 在帳號建立後也可作為登入帳號使用，替代 Email 登入（詳見 [帳號密碼登入](#0-帳號密碼登入-authenticate)）。
>
> **自動同步：** 後端會自動將 `identifier` 的值寫入子表：個人帳號寫入 `user_infos.id_number`、企業帳號寫入 `companies.tax_id`。因此 **`user_info` 不需傳 `id_number`**，**`company_info` 不需傳 `tax_id`**。

#### 選填欄位 (一般使用者 user_info)

> 當 `is_corporate = false` 時需提供。

| 參數                           | 類型    | 說明                                               |
| ------------------------------ | ------- | -------------------------------------------------- |
| name                           | string  | 姓名                                               |
| en_name                        | string  | 英文名 (可為 null)                                 |
| sex                            | integer | 性別代碼 (0-2)                                     |
| birthday                       | string  | 生日                                               |
| seniority                      | integer | 年資                                               |
| marital_status                 | integer | 婚姻狀態 (0-4)                                     |
| other_phone.phone_1            | string  | 其他電話（寫入 `user_infos.other_phone`）           |
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
| owner_name    | string  | 負責人       |
| avatar        | string  | 公司頭像路徑 |
| industry      | string  | 產業代碼     |
| members       | integer | 員工人數     |
| contact       | string  | 聯絡人       |
| other_phone.phone_1 | string  | 電話（寫入 `companies.other_phone`） |
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
    "phone": "0912345678",
    "identifier": "A123456789",
    "password": "password123",
    "password_confirmation": "password123",
    "otp": "123456",
    "is_corporate": false,
    "user_info": {
        "name": "王小明",
        "sex": 1,
        "other_phone": {
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
    "phone": "0912345678",
    "identifier": "12345678",
    "password": "password123",
    "password_confirmation": "password123",
    "otp": "123456",
    "is_corporate": true,
    "company_info": {
        "name": "範例股份有限公司",
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
  -F "user[phone]=0912345678" \
  -F "user[identifier]=12345678" \
  -F "user[password]=password123" \
  -F "user[password_confirmation]=password123" \
  -F "user[otp]=123456" \
  -F "user[is_corporate]=1" \
  -F "company_info[name]=範例股份有限公司" \
  -F "company_info[owner_name]=王大明" \
  -F "company_info[industry]=123456" \
  -F "company_info[members]=100" \
  -F "company_info[other_phone][phone_1]=0223456789" \
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
formData.append('user[phone]', '0912345678');
formData.append('user[identifier]', '12345678');
formData.append('user[password]', 'password123');
formData.append('user[password_confirmation]', 'password123');
formData.append('user[otp]', '123456');
formData.append('user[is_corporate]', '1');

// company_info 欄位
formData.append('company_info[name]', '範例股份有限公司');
formData.append('company_info[owner_name]', '王大明');
formData.append('company_info[industry]', '123456');
formData.append('company_info[members]', '100');
formData.append('company_info[other_phone][phone_1]', '0223456789');
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

#### OTP 驗證失敗 (422)

```json
{
    "msg": "otp 驗證失敗，請重新發送驗證碼",
    "res": null
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

> 皆為選填欄位。
> 若提供 `address` 且未提供 `county` 或 `district`，系統會依 `address` 嘗試解析。
> `user_infos.phone` 支援兩種寫法：`other_phone.phone_1`（新）與 `phone.phone_1`（舊版相容）。

| 參數                           | 類型    | 說明                                              |
| ------------------------------ | ------- | ------------------------------------------------- |
| phone                          | string  | 使用者主手機（寫入 `users.phone`，需唯一，格式 09XXXXXXXX） |
| id_number                      | string  | 身分證字號 (唯一，格式 /^[A-Z]{1}[1-2]{1}\d{8}$/) |
| name                           | string  | 姓名                                              |
| en_name                        | string  | 英文名 (可為 null)                                |
| sex                            | integer | 性別代碼 (0-2)                                    |
| birthday                       | string  | 生日                                              |
| seniority                      | integer | 年資                                              |
| marital_status                 | integer | 婚姻狀態 (0-4)                                    |
| other_phone.phone_1            | string  | 其他電話（寫入 `user_infos.phone`）               |
| phone.phone_1                  | string  | 舊版相容寫法（同樣寫入 `user_infos.phone`）       |
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
    "phone": "0912345678",
    "other_phone": {
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

---

## 3. 發送手機 OTP (sendOtp)

### Endpoint

```
POST /otp/send
```

### 說明

發送手機 OTP 驗證碼（共用入口，依 `scene` 區分場景）。OTP 有效期為 5 分鐘，同一手機號碼 60 秒內禁止重複發送。

### Headers

```
Content-Type: application/json
Accept: application/json
```

### Request Body

| 參數  | 類型   | 必填 | 說明                                               |
| ----- | ------ | ---- | -------------------------------------------------- |
| phone | string | ✅   | 手機號碼（格式 09XXXXXXXX）                         |
| scene | string | ✅   | 場景：`register`（註冊）、`login`（登入）、`forgot`（忘記密碼） |

### Request 範例

```json
{
    "phone": "0912345678",
    "scene": "register"
}
```

### Success Response (200)

```json
{
    "msg": "驗證碼已發送",
    "res": null
}
```

### Error Responses

#### 發送過於頻繁 (429)

```json
{
    "msg": "請勿在 60 秒內重複發送驗證碼",
    "res": null
}
```

#### 手機號碼未綁定帳號（scene=login 或 forgot）(404)

```json
{
    "msg": "此手機號碼尚未綁定任何帳號",
    "res": null
}
```

#### 簡訊發送失敗 (503)

```json
{
    "msg": "簡訊發送失敗，請稍後再試",
    "res": null
}
```

---

## 4. 手機 OTP 快速登入 (loginByPhone)

### Endpoint

```
POST /login/phone
```

### 說明

使用手機號碼與 OTP 快速登入，無需密碼。請先呼叫 `POST /otp/send`（`scene=login`）取得驗證碼。

### Headers

```
Content-Type: application/json
Accept: application/json
```

### Request Body

| 參數  | 類型   | 必填 | 說明                        |
| ----- | ------ | ---- | --------------------------- |
| phone | string | ✅   | 手機號碼（格式 09XXXXXXXX） |
| otp   | string | ✅   | 6 位數驗證碼                |

### Request 範例

```json
{
    "phone": "0912345678",
    "otp": "123456"
}
```

### Success Response (200)

```json
{
    "msg": "success",
    "res": {
        "token": "1|xxxxxxxxxxxxxxxx",
        "data": { }
    }
}
```

### Error Responses

#### OTP 驗證失敗或已過期 (422)

```json
{
    "msg": "otp 驗證失敗或已過期",
    "res": null
}
```

#### 使用者不存在 (404)

```json
{
    "msg": "找不到對應的使用者",
    "res": null
}
```

---

## 5. 忘記密碼 — 驗證 OTP (verifyForgotPasswordOtp)

### Endpoint

```
POST /forgot-password/verify
```

### 說明

忘記密碼流程第一步：驗證 OTP，成功後回傳一次性 `reset_token`（有效期 10 分鐘），供下一步重設密碼使用。請先呼叫 `POST /otp/send`（`scene=forgot`）取得驗證碼。

### Headers

```
Content-Type: application/json
Accept: application/json
```

### Request Body

| 參數  | 類型   | 必填 | 說明                        |
| ----- | ------ | ---- | --------------------------- |
| phone | string | ✅   | 手機號碼（格式 09XXXXXXXX） |
| otp   | string | ✅   | 6 位數驗證碼                |

### Request 範例

```json
{
    "phone": "0912345678",
    "otp": "123456"
}
```

### Success Response (200)

```json
{
    "msg": "驗證成功，請在 10 分鐘內完成密碼重設",
    "res": {
        "reset_token": "xxxxxxxxxxxxxxxxxxxxxxxx"
    }
}
```

### Error Responses

#### OTP 驗證失敗或已過期 (422)

```json
{
    "msg": "otp 驗證失敗或已過期",
    "res": null
}
```

---

## 6. 忘記密碼 — 重設密碼 (resetPassword)

### Endpoint

```
POST /forgot-password/reset
```

### 說明

忘記密碼流程第二步：使用 `verifyForgotPasswordOtp` 回傳的 `reset_token` 重設密碼。`reset_token` 為一次性，使用後即失效。重設成功後，所有 Sanctum Token 將一併撤銷，需重新登入。

### Headers

```
Content-Type: application/json
Accept: application/json
```

### Request Body

| 參數                  | 類型   | 必填 | 說明                          |
| --------------------- | ------ | ---- | ----------------------------- |
| reset_token           | string | ✅   | 由 `forgot-password/verify` 取得的一次性 Token |
| password              | string | ✅   | 新密碼（至少 8 個字元）       |
| password_confirmation | string | ✅   | 確認新密碼                    |

### Request 範例

```json
{
    "reset_token": "xxxxxxxxxxxxxxxxxxxxxxxx",
    "password": "newpassword123",
    "password_confirmation": "newpassword123"
}
```

### Success Response (200)

```json
{
    "msg": "密碼重設成功，請重新登入",
    "res": null
}
```

### Error Responses

#### reset_token 無效或已過期 (422)

```json
{
    "msg": "重設 Token 無效或已過期，請重新申請",
    "res": null
}
```

#### 使用者不存在 (404)

```json
{
    "msg": "找不到對應的使用者",
    "res": null
}
```

---

## 7. 取得登入者資料 (info)

### Endpoint

```
GET /v1/user/info
```

### 說明

取得目前登入者的個人資料。除基本使用者欄位外，額外附加當日勞健保自付額（每日分攤金額，從 SSO 系統查詢並快取 24 小時）。

> 若 SSO 系統查詢失敗，`labor_one_day_pay` 與 `health_one_day_pay` 預設為 `0`，不影響主要資料回傳。

### Headers

```
Authorization: Bearer {access_token}
Accept: application/json
```

### Success Response (200)

```json
{
    "msg": "success",
    "res": {
        "id": 42,
        "email": "user@example.com",
        "phone": "0912345678",
        "identifier": "A123456789",
        "is_corporate": 0,
        "last_login": "2026-06-25 10:00:00",
        "created_at": "2026-01-01 00:00:00",
        "updated_at": "2026-06-25 10:00:00",
        "user_info": {
            "name": "王小明",
            "sex": 1,
            "birthday": "1990-01-01",
            "address": "台北市大安區信義路四段1號",
            "county": "台北市",
            "district": "大安區"
        },
        "labor_one_day_pay": 33,
        "health_one_day_pay": 15
    }
}
```

### 回應欄位說明

| 欄位                | 類型    | 說明                                                      |
| ------------------- | ------- | --------------------------------------------------------- |
| labor_one_day_pay   | integer | 勞保每日自付額（月繳金額 ÷ 30，無條件捨去）；查無資料時為 `0` |
| health_one_day_pay  | integer | 健保每日自付額（月繳金額 ÷ 30，無條件捨去）；查無資料時為 `0` |

### Error Response (401)

```json
{
    "msg": "Unauthenticated.",
    "res": null
}
```

