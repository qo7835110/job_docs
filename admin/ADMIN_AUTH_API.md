# 管理員認證 API

## 概述
管理員登入和登出功能的 API 文檔。

---

## 1. 管理員登入

### 端點
```
POST /api/admin/login
```

### 請求標頭
```
Content-Type: application/json
```

### 請求參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| email | string | 是 | 管理員信箱 |
| password | string | 是 | 密碼（最少 6 個字元）|

### 請求範例
```json
{
  "email": "admin@example.com",
  "password": "password"
}
```

### 成功回應 (200)
```json
{
  "message": "Login successful",
  "admin": {
    "id": 1,
    "name": "Super Admin",
    "email": "admin@example.com",
    "role": "super_admin",
    "avatar": null
  },
  "token": "1|abc123xyz456..."
}
```

### 錯誤回應

#### 驗證失敗 (422)
```json
{
  "message": "Validation failed",
  "errors": {
    "email": [
      "The email field is required."
    ],
    "password": [
      "The password must be at least 6 characters."
    ]
  }
}
```

#### 帳號或密碼錯誤 (401)
```json
{
  "message": "Invalid credentials"
}
```

#### 帳號已停用 (403)
```json
{
  "message": "Your account has been deactivated"
}
```

### 範例請求
```bash
curl -X POST http://localhost:8000/api/admin/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@example.com",
    "password": "password"
  }'
```

---

## 2. 管理員登出

### 端點
```
POST /api/admin/logout
```

### 請求標頭
```
Content-Type: application/json
Authorization: Bearer {token}
```

### 請求參數
無需額外參數

### 成功回應 (200)
```json
{
  "message": "Logout successful"
}
```

### 錯誤回應

#### 未授權 (401)
```json
{
  "message": "Unauthorized. Admin authentication required."
}
```

### 範例請求
```bash
curl -X POST http://localhost:8000/api/admin/logout \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer 1|abc123xyz456..."
```

---