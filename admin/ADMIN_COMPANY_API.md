# 管理員 - 公司管理 API

## 概述
管理員管理公司資料和認證狀態的 API 文檔。

**Base URL**: `/api/admin/companies`

**認證方式**: Bearer Token（需要管理員登入後取得的 token）

---

## 1. 取得公司列表

### 端點
```
GET /api/admin/companies
```

### 請求標頭
```
Authorization: Bearer {token}
Content-Type: application/json
```

### 查詢參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| search | string | 否 | 搜尋關鍵字（公司名稱、email、電話、統編）|
| verification_status | integer | 否 | 驗證狀態篩選（0=未驗證, 1=已驗證, 2=驗證中, 3=驗證失敗）|
| industry | string | 否 | 產業篩選 |
| sort_by | string | 否 | 排序欄位（預設：created_at）|
| sort_order | string | 否 | 排序方向（asc/desc，預設：desc）|
| per_page | integer | 否 | 每頁筆數（預設：15）|
| page | integer | 否 | 頁碼（預設：1）|

### 成功回應 (200)
```json
{
  "current_page": 1,
  "data": [
    {
      "id": 1,
      "user_id": 5,
      "name": "科技公司",
      "avatar": "https://example.com/avatar.jpg",
      "industry": ["資訊科技", "軟體開發"],
      "members": "50-100人",
      "contact": "張先生",
      "phone": "02-12345678",
      "fax": "02-12345679",
      "email": "contact@company.com",
      "address": "台北市信義區信義路五段7號",
      "website": "https://company.com",
      "contact_time": "週一至週五 09:00-18:00",
      "description": "公司簡介...",
      "remark": "管理員備註",
      "benefit": "員工福利說明",
      "tax_id": "12345678",
      "verification_status": 1,
      "verification_reason": null,
      "registration_certificate_front": "https://example.com/cert_front.jpg",
      "registration_certificate_back": "https://example.com/cert_back.jpg",
      "factory_certificate_front": null,
      "factory_certificate_back": null,
      "created_at": "2026-01-01 10:00:00",
      "updated_at": "2026-01-05 15:30:00",
      "images": [
        {
          "id": 1,
          "company_id": 1,
          "name": "辦公室照片",
          "img_path": "https://example.com/office.jpg",
          "created_at": "2026-01-01 10:00:00"
        }
      ],
      "attachments": [
        {
          "id": 1,
          "company_id": 1,
          "name": "公司簡介",
          "file_path": "https://example.com/brochure.pdf",
          "created_at": "2026-01-01 10:00:00"
        }
      ]
    }
  ],
  "first_page_url": "http://localhost/api/admin/companies?page=1",
  "from": 1,
  "last_page": 5,
  "last_page_url": "http://localhost/api/admin/companies?page=5",
  "links": [...],
  "next_page_url": "http://localhost/api/admin/companies?page=2",
  "path": "http://localhost/api/admin/companies",
  "per_page": 15,
  "prev_page_url": null,
  "to": 15,
  "total": 67
}
```

### 範例請求
```bash
# 基本請求
curl -X GET "http://localhost:8000/api/admin/companies" \
  -H "Authorization: Bearer {token}"

# 搜尋並篩選
curl -X GET "http://localhost:8000/api/admin/companies?search=科技&verification_status=1&per_page=20" \
  -H "Authorization: Bearer {token}"

# 排序
curl -X GET "http://localhost:8000/api/admin/companies?sort_by=name&sort_order=asc" \
  -H "Authorization: Bearer {token}"
```

---

## 2. 更新公司認證狀態

### 端點
```
PUT /api/admin/companies/{id}/verification-status
```

### 請求標頭
```
Authorization: Bearer {token}
Content-Type: application/json
```

### 路徑參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| id | integer | 是 | 公司 ID |

### 請求參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| verification_status | integer | 是 | 驗證狀態（0=未驗證, 1=已驗證, 2=驗證中, 3=驗證失敗）|
| verification_reason | string | 否 | 驗證原因或備註（通常在拒絕時使用）|

### 驗證狀態說明
- **0**: 未驗證 - 初始狀態，尚未申請驗證
- **1**: 已驗證 - 通過驗證
- **2**: 驗證中 - 正在審核中
- **3**: 驗證失敗 - 未通過驗證

### 請求範例
```json
{
  "verification_status": 1,
  "verification_reason": null
}
```

```json
{
  "verification_status": 3,
  "verification_reason": "統編格式錯誤，請重新提供正確資料"
}
```

### 成功回應 (200)
```json
{
  "message": "Verification status updated successfully",
  "company": {
    "id": 1,
    "name": "科技公司",
    "verification_status": 1,
    "verification_reason": null,
    "updated_at": "2026-01-07 14:30:00",
    ...
  }
}
```

### 錯誤回應

#### 公司不存在 (404)
```json
{
  "message": "Company not found"
}
```

#### 驗證失敗 (422)
```json
{
  "message": "Validation failed",
  "errors": {
    "verification_status": [
      "The selected verification status is invalid."
    ]
  }
}
```

### 範例請求
```bash
# 設定為已驗證
curl -X PUT "http://localhost:8000/api/admin/companies/1/verification-status" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "verification_status": 1,
    "verification_reason": null
  }'

# 設定為驗證失敗
curl -X PUT "http://localhost:8000/api/admin/companies/1/verification-status" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "verification_status": 3,
    "verification_reason": "資料不完整"
  }'
```

---

## 3. 取得待認證公司列表

### 端點
```
GET /api/admin/companies/pending-verification
```

### 請求標頭
```
Authorization: Bearer {token}
Content-Type: application/json
```

### 查詢參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| search | string | 否 | 搜尋關鍵字（公司名稱、email、電話、統編）|
| per_page | integer | 否 | 每頁筆數（預設：15）|
| page | integer | 否 | 頁碼（預設：1）|

### 說明
此 API 返回驗證狀態為「未驗證（0）」或「驗證中（2）」的公司列表，按建立時間倒序排列。

### 成功回應 (200)
```json
{
  "current_page": 1,
  "data": [
    {
      "id": 10,
      "name": "新創公司",
      "email": "info@startup.com",
      "phone": "02-87654321",
      "tax_id": "87654321",
      "verification_status": 2,
      "verification_reason": null,
      "registration_certificate_front": "https://example.com/cert_front.jpg",
      "registration_certificate_back": "https://example.com/cert_back.jpg",
      "created_at": "2026-01-07 10:00:00",
      "updated_at": "2026-01-07 10:00:00",
      "images": [...],
      "attachments": [...]
    },
    {
      "id": 8,
      "name": "傳統產業公司",
      "email": "contact@traditional.com",
      "phone": "02-23456789",
      "tax_id": "23456789",
      "verification_status": 0,
      "verification_reason": null,
      "created_at": "2026-01-06 15:30:00",
      "updated_at": "2026-01-06 15:30:00",
      "images": [],
      "attachments": []
    }
  ],
  "first_page_url": "http://localhost/api/admin/companies/pending-verification?page=1",
  "from": 1,
  "last_page": 2,
  "last_page_url": "http://localhost/api/admin/companies/pending-verification?page=2",
  "next_page_url": "http://localhost/api/admin/companies/pending-verification?page=2",
  "path": "http://localhost/api/admin/companies/pending-verification",
  "per_page": 15,
  "prev_page_url": null,
  "to": 15,
  "total": 23
}
```

### 範例請求
```bash
# 基本請求
curl -X GET "http://localhost:8000/api/admin/companies/pending-verification" \
  -H "Authorization: Bearer {token}"

# 搜尋待認證公司
curl -X GET "http://localhost:8000/api/admin/companies/pending-verification?search=新創" \
  -H "Authorization: Bearer {token}"

# 自訂分頁
curl -X GET "http://localhost:8000/api/admin/companies/pending-verification?per_page=30&page=2" \
  -H "Authorization: Bearer {token}"
```

---

## 錯誤回應

### 未授權 (401)
```json
{
  "message": "Unauthorized. Admin authentication required."
}
```

### 帳號已停用 (403)
```json
{
  "message": "Your account has been deactivated."
}
```

---

## 使用流程範例

### 審核公司認證的完整流程

1. **取得待認證公司列表**
```bash
GET /api/admin/companies/pending-verification
```

2. **查看特定公司詳情**（使用 show API）
```bash
GET /api/admin/companies/{id}
```

3. **審核公司 - 通過**
```bash
PUT /api/admin/companies/{id}/verification-status
{
  "verification_status": 1
}
```

4. **審核公司 - 拒絕**
```bash
PUT /api/admin/companies/{id}/verification-status
{
  "verification_status": 3,
  "verification_reason": "統編錯誤"
}
```

---

## 狀態碼說明

| 狀態碼 | 說明 |
|--------|------|
| 200 | 請求成功 |
| 401 | 未授權（未提供 Token 或 Token 無效）|
| 403 | 禁止訪問（帳號已停用）|
| 404 | 資源不存在（公司不存在）|
| 422 | 驗證失敗（參數格式錯誤）|
