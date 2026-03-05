# 特殊身分 API 文件

> 最後更新：2026-03-05

## 概述

提供特殊身分類別的管理功能（CRUD）。所有 API 均需使用 Sanctum Token 認證。

**認證 Header：**
```
Authorization: Bearer {token}
```

---

## 資料表結構

### special_identities（特殊身分類別主表）

| 欄位 | 類型 | 說明 |
|---|---|---|
| id | bigint | 主鍵 |
| name | varchar(50) | 特殊身分名稱 |
| description | varchar(255) | 特殊身分描述（nullable） |
| status | boolean | 啟用狀態：`true`＝啟用、`false`＝停用 |
| sort_order | unsigned int | 排序（數字越小越前面） |
| created_at | timestamp | 建立時間 |
| updated_at | timestamp | 更新時間 |

### 預設資料（Seeder）

| ID | 名稱 | 描述 |
|---|---|---|
| 1 | 一般 | 無特殊身分 |
| 2 | 外籍人士 | 外國籍人士 |
| 3 | 僑生 | 海外僑胞學生 |
| 4 | 新住民 | 外籍配偶或歸化者 |
| 5 | 原住民 | 臺灣原住民族 |
| 6 | 二度就業 | 重返職場之求職者 |
| 7 | 身心障礙 | 持有身心障礙證明 |

### 與 user_infos 的關聯

`user_infos.special_identity_id` → FK 關聯至 `special_identities.id`（nullable，ON DELETE SET NULL）。

---

## API 列表

| 方法 | 路徑 | 說明 |
|---|---|---|
| GET | `/v1/special-identities` | 取得所有啟用中的特殊身分 |
| GET | `/v1/special-identities/{id}` | 取得單一特殊身分 |
| POST | `/v1/special-identities` | 新增特殊身分 |
| PUT | `/v1/special-identities/{id}` | 更新特殊身分 |
| DELETE | `/v1/special-identities/{id}` | 刪除特殊身分 |

---

## GET `/v1/special-identities`

取得所有啟用中的特殊身分列表（依 `sort_order` 排序）。

### 回應範例 (200)
```json
{
  "msg": "success",
  "res": [
    {
      "id": 1,
      "name": "一般",
      "description": "無特殊身分",
      "status": true,
      "sort_order": 1,
      "created_at": "2026-03-05 05:15:27",
      "updated_at": "2026-03-05 05:15:27"
    },
    {
      "id": 2,
      "name": "外籍人士",
      "description": "外國籍人士",
      "status": true,
      "sort_order": 2,
      "created_at": "2026-03-05 05:15:27",
      "updated_at": "2026-03-05 05:15:27"
    }
  ]
}
```

### 回應欄位說明

| 欄位 | 類型 | 說明 |
|---|---|---|
| id | integer | 特殊身分 ID |
| name | string | 特殊身分名稱 |
| description | string\|null | 描述 |
| status | boolean | 啟用狀態 |
| sort_order | integer | 排序順序 |
| created_at | string | 建立時間 |
| updated_at | string | 更新時間 |

---

## GET `/v1/special-identities/{id}`

取得單一特殊身分詳情。

### 路徑參數

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| id | integer | ✅ | 特殊身分 ID |

### 回應範例 (200)
```json
{
  "msg": "success",
  "res": {
    "id": 1,
    "name": "一般",
    "description": "無特殊身分",
    "status": true,
    "sort_order": 1,
    "created_at": "2026-03-05 05:15:27",
    "updated_at": "2026-03-05 05:15:27"
  }
}
```

### 錯誤回應 (404)
```json
{
  "msg": "找不到該特殊身分",
  "status": "error"
}
```

---

## POST `/v1/special-identities`

新增特殊身分。

### 請求參數

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| name | string | ✅ | 特殊身分名稱（最長 50 字，不可重複） |
| description | string | ❌ | 描述（最長 255 字） |
| status | boolean | ❌ | 啟用狀態（預設 `true`） |
| sort_order | integer | ❌ | 排序（預設 `0`） |

### 請求範例
```json
{
  "name": "中高齡者",
  "description": "年滿 45 歲以上之求職者",
  "status": true,
  "sort_order": 8
}
```

### 回應範例 (201)
```json
{
  "msg": "建立成功",
  "res": {
    "id": 8,
    "name": "中高齡者",
    "description": "年滿 45 歲以上之求職者",
    "status": true,
    "sort_order": 8,
    "created_at": "2026-03-05 06:00:00",
    "updated_at": "2026-03-05 06:00:00"
  }
}
```

### 錯誤回應 (400)
```json
{
  "message": "此特殊身分名稱已存在",
  "errors": {
    "name": ["此特殊身分名稱已存在"]
  }
}
```

---

## PUT `/v1/special-identities/{id}`

更新特殊身分。

### 路徑參數

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| id | integer | ✅ | 特殊身分 ID |

### 請求參數

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| name | string | ❌ | 特殊身分名稱（最長 50 字，不可與其他重複） |
| description | string | ❌ | 描述（最長 255 字） |
| status | boolean | ❌ | 啟用狀態 |
| sort_order | integer | ❌ | 排序 |

### 請求範例
```json
{
  "description": "更新後的描述",
  "sort_order": 10
}
```

### 回應範例 (200)
```json
{
  "msg": "更新成功",
  "res": {
    "id": 1,
    "name": "一般",
    "description": "更新後的描述",
    "status": true,
    "sort_order": 10,
    "created_at": "2026-03-05 05:15:27",
    "updated_at": "2026-03-05 06:30:00"
  }
}
```

### 錯誤回應 (404)
```json
{
  "msg": "找不到該特殊身分",
  "status": "error"
}
```

---

## DELETE `/v1/special-identities/{id}`

刪除特殊身分。

> ⚠️ 若有使用者的 `user_infos.special_identity_id` 關聯到此特殊身分，刪除後該欄位會被設為 `NULL`（ON DELETE SET NULL）。

### 路徑參數

| 參數 | 類型 | 必填 | 說明 |
|---|---|---|---|
| id | integer | ✅ | 特殊身分 ID |

### 回應範例 (200)
```json
{
  "msg": "刪除成功"
}
```

### 錯誤回應 (404)
```json
{
  "msg": "找不到該特殊身分",
  "status": "error"
}
```
