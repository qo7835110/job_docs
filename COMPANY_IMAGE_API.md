# 公司圖片管理 API

## 概述
上傳和刪除公司相關圖片的 API 文檔，包含公司頭像和各類證書圖片。

**Base Path**: `/company`

**認證方式**: Bearer Token（需要使用者登入後取得的 token）

---

## 1. 上傳公司圖片

### 端點
```
POST /company/uploadImg
```

### 請求標頭
```
Authorization: Bearer {token}
Content-Type: multipart/form-data
```

### 請求參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| category | string | 是 | 圖片類型（見下方類型說明）|
| image | file | 是 | 圖片檔案 |

### 圖片類型 (category)
| 值 | 說明 |
|------|------|
| avatar | 公司頭像/Logo |
| registration_certificate_front | 公司登記證正面 |
| registration_certificate_back | 公司登記證反面 |
| factory_certificate_front | 工廠登記證正面 |
| factory_certificate_back | 工廠登記證反面 |

### 圖片規格
- **格式**: jpeg, png, jpg, gif, svg
- **大小上限**: 64 MB (65536 KB)

### 成功回應 (200)
```json
{
  "msg": "success",
  "res": {
    "id": 1,
    "user_id": 5,
    "name": "科技股份有限公司",
    "avatar": "images/abc123def456.jpg",
    "industry": {
      "code": "J01",
      "type": "資訊服務業/電腦系統設計服務業",
      "name": "軟體開發"
    },
    "members": 50,
    "contact": "張經理",
    "phone": "02-12345678",
    "fax": "02-12345679",
    "email": "info@company.com",
    "address": "台北市信義區信義路五段7號",
    "website": "https://company.com",
    "contact_time": "週一至週五 09:00-18:00",
    "description": "公司簡介...",
    "benefit": "員工福利...",
    "remark": null,
    "tax_id": "12345678",
    "verification_status": 1,
    "verification_reason": null,
    "registration_certificate_front": "images/cert_front.jpg",
    "registration_certificate_back": "images/cert_back.jpg",
    "factory_certificate_front": null,
    "factory_certificate_back": null,
    "created_at": "2026-01-01 10:00:00",
    "updated_at": "2026-01-09 14:30:00"
  }
}
```

### 錯誤回應

#### 公司不存在 (404)
```json
{
  "msg": "company not found",
  "res": null
}
```

#### 驗證失敗 (422)
```json
{
  "msg": "validation error",
  "errors": {
    "category": [
      "The selected category is invalid."
    ],
    "image": [
      "The image must be a file of type: jpeg, png, jpg, gif, svg.",
      "The image may not be greater than 65536 kilobytes."
    ]
  }
}
```

#### 未授權 (401)
```json
{
  "message": "Unauthenticated."
}
```

### 範例請求

#### 使用 curl
```bash
curl -X POST http://localhost:8000/company/uploadImg \
  -H "Authorization: Bearer {token}" \
  -F "category=avatar" \
  -F "image=@/path/to/company-logo.jpg"
```

#### 使用 curl - 上傳證書
```bash
curl -X POST http://localhost:8000/company/uploadImg \
  -H "Authorization: Bearer {token}" \
  -F "category=registration_certificate_front" \
  -F "image=@/path/to/certificate.jpg"
```

#### 使用 JavaScript (FormData)
```javascript
const formData = new FormData();
formData.append('category', 'avatar');
formData.append('image', fileInput.files[0]);

fetch('http://localhost:8000/company/uploadImg', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`
  },
  body: formData
})
.then(response => response.json())
.then(data => console.log(data));
```

---

## 2. 刪除公司圖片

### 端點
```
DELETE /company/deleteImg
```

### 請求標頭
```
Authorization: Bearer {token}
Content-Type: application/json
```

### 請求參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| category | string | 是 | 要刪除的圖片類型（見下方類型說明）|

### 圖片類型 (category)
| 值 | 說明 |
|------|------|
| avatar | 公司頭像/Logo |
| registration_certificate_front | 公司登記證正面 |
| registration_certificate_back | 公司登記證反面 |
| factory_certificate_front | 工廠登記證正面 |
| factory_certificate_back | 工廠登記證反面 |
| all | 刪除所有圖片 |

### 請求範例

#### 刪除單一圖片
```json
{
  "category": "avatar"
}
```

#### 刪除所有圖片
```json
{
  "category": "all"
}
```

### 成功回應 (200)
```json
{
  "msg": "success",
  "res": null
}
```

### 錯誤回應

#### 公司不存在 (404)
```json
{
  "msg": "company not found",
  "res": null
}
```

#### 驗證失敗 (422)
```json
{
  "msg": "validation error",
  "errors": {
    "category": [
      "The selected category is invalid."
    ]
  }
}
```

#### 未授權 (401)
```json
{
  "message": "Unauthenticated."
}
```
