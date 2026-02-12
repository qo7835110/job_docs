# 公司資料建立/更新 API

## Base URL
```
/v1/company
```

---

## 1. 建立或更新公司資料 (setCompany)

### Endpoint
```
POST /update
```

### 說明
建立或更新登入者的公司資料。若公司資料已存在則更新，否則建立。

### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
Accept: application/json
```

### Request Body
> 皆為選填欄位。

| 參數 | 類型 | 說明 |
| --- | --- | --- |
| name | string | 公司名稱 |
| tax_id | string | 統一編號 |
| owner_name | string | 負責人 |
| avatar | string | 公司頭像路徑 |
| industry | string | 產業代碼 |
| members | integer | 員工人數 |
| contact | string | 聯絡人 |
| phone.phone_1 | string | 主要電話 |
| fax | string | 傳真 |
| email | string | 公司信箱 |
| address | string | 地址 |
| county | string | 縣市 |
| district | string | 區域 |
| website | string | 公司網站 |
| contact_time | string | 方便聯絡時間 |
| others | array | 其他資訊 |
| others.* | string | 其他資訊項目 |
| description | string | 公司描述 |
| benefit | string | 福利 |
| remark | string | 備註 |

### Request 範例
```json
{
  "name": "範例股份有限公司",
  "tax_id": "12345678",
  "owner_name": "王大明",
  "industry": "123456",
  "members": 100,
  "phone": {
    "phone_1": "0223456789"
  },
  "email": "corp@example.com",
  "address": "台北市信義區松高路1號",
  "county": "台北市",
  "district": "信義區",
  "website": "https://example.com",
  "description": "公司簡介"
}
```

### Success Response (200)
```json
{
  "msg": "success",
  "res": {
    "id": 10,
    "user_id": 5,
    "name": "範例股份有限公司",
    "industry": {
      "code": "123456",
      "type": "資訊服務業/電腦系統設計服務業",
      "name": "軟體開發"
    },
    "updated_at": "2026-02-13 14:30:00"
  }
}
```

### Error Responses

#### 驗證失敗 (422)
```json
{
  "msg": "validation error",
  "errors": {
    "email": ["The email must be a valid email address."],
    "members": ["The members must be an integer."]
  }
}
```

#### 產業代碼不存在 (404)
```json
{
  "msg": "industry not found",
  "res": null
}
```
