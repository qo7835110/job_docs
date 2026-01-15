# Manage API Documentation

## 概述
管理 API 用於處理員工管理相關功能，包括員工列表查詢、薪資統計、薪資更新和任務信息查詢。所有 API 都需要管理者身份認證。

**Base URL:** `/v1/manage`

**認證要求:** 所有端點需要 `auth:sanctum` 認證

**通用 Headers:**
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

---

## 獲取員工列表 (Get Employees List)

### Endpoint
```
GET /employees
```

### 描述
獲取當前管理者的所有員工列表，包含員工基本信息、當月薪資、加班時數、請假時數和打卡狀態。

### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

### Request Parameters
無需請求參數（可在代碼中調整查詢月份）

### Request Example
```
GET /v1/manage/employees
```

### Success Response (200 OK)
```json
{
    "msg": "success",
    "res": {
        "current_page": 1,
        "data": [
            {
                "employee_name": "張三",
                "employee_id": 10,
                "mug_shot_img": "https://example.com/image.jpg",
                "last_punch_in": "2026-01-14 09:00:00",
                "last_punch_out": "2026-01-14 18:00:00",
                "total_leave_hours": 8,
                "total_salary": 45000,
                "total_ot_pay": 3000,
                "total_work_hours": 160,
                "total_ot_hours": 10
            },
            {
                "employee_name": "李四",
                "employee_id": 11,
                "mug_shot_img": "https://example.com/image2.jpg",
                "last_punch_in": "2026-01-14 08:30:00",
                "last_punch_out": "2026-01-14 17:30:00",
                "total_leave_hours": 0,
                "total_salary": 50000,
                "total_ot_pay": 5000,
                "total_work_hours": 168,
                "total_ot_hours": 15
            }
        ],
        "per_page": 20,
        "total": 2,
        "last_page": 1
    }
}
```

### 回應欄位說明
| 欄位 | 類型 | 描述 |
|------|------|------|
| employee_name | string | 員工姓名 |
| employee_id | integer | 員工 ID |
| mug_shot_img | string | 員工頭像 URL |
| last_punch_in | datetime | 最後打卡上班時間 |
| last_punch_out | datetime | 最後打卡下班時間 |
| total_leave_hours | integer | 當月總請假時數 |
| total_salary | integer | 當月總薪資 |
| total_ot_pay | integer | 當月總加班費 |
| total_work_hours | integer | 當月總工作時數 |
| total_ot_hours | integer | 當月總加班時數 |

### 說明
- 需要管理者認證 token
- 預設每頁顯示 20 筆資料
- 按最後打卡時間降序排列
- 包含當月薪資和請假統計

---

## 獲取薪資資訊 (Get Salary Info)

### Endpoint
```
GET /salary/info
```

### 描述
查詢指定日期範圍內的薪資統計，包含總薪資、基本薪資和加班費，按月份分組。

### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

### Query Parameters
| 參數 | 類型 | 必填 | 描述 |
|------|------|------|------|
| start_at | date | 條件必填* | 開始日期 (格式: YYYY-MM-DD) |
| end_at | date | 條件必填* | 結束日期 (格式: YYYY-MM-DD) |

**條件必填說明**：start_at 和 end_at 可以都不填，但如果提供其中一個，則另一個也必須提供

### Request Example
```
GET /v1/manage/salary/info?start_at=2026-01-01&end_at=2026-03-31
```

### Success Response (200 OK)
```json
{
    "msg": "success",
    "res": [
        {
            "month": 1,
            "year": 2026,
            "total_sum_pay": 450000,
            "total_pay": 400000,
            "total_ot_pay": 50000
        },
        {
            "month": 2,
            "year": 2026,
            "total_sum_pay": 480000,
            "total_pay": 420000,
            "total_ot_pay": 60000
        },
        {
            "month": 3,
            "year": 2026,
            "total_sum_pay": 510000,
            "total_pay": 440000,
            "total_ot_pay": 70000
        }
    ]
}
```

### 回應欄位說明
| 欄位 | 類型 | 描述 |
|------|------|------|
| month | integer | 月份 (1-12) |
| year | integer | 年份 |
| total_sum_pay | integer | 總薪資（包含基本薪資、加班費、獎金等） |
| total_pay | integer | 基本薪資 |
| total_ot_pay | integer | 加班費 |

### Error Responses

#### 422 Validation Error
```json
{
    "message": "The start at field is required when end at is present.",
    "errors": {
        "start_at": ["The start at field is required when end at is present."]
    }
}
```

### 說明
- start_at 和 end_at 可以都不提供（返回所有數據），但如果提供其中一個，則另一個也必須提供
- 數據按月份和年份分組統計
- 需要管理者認證 token

---

## 獲取員工薪資詳情 (Get Employee Salary Details)

### Endpoint
```
GET /employee/{id}/salary/{year}/{month}
```

### 描述
獲取指定員工在特定月份的薪資詳情，包含基本薪資、加班費、獎金、全勤獎金和各項扣款。

### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

### URL Parameters
| 參數 | 類型 | 描述 |
|------|------|------|
| id | integer | 員工 ID |
| year | integer | 年份 (例如: 2026) |
| month | integer | 月份 (1-12) |

### Request Example
```
GET /v1/manage/employee/10/salary/2026/1
```

### Success Response (200 OK)
```json
{
    "msg": "success",
    "res": {
        "year": 2026,
        "month": 1,
        "total_pay": 40000,
        "total_ot_pay": 5000,
        "total_sum_pay": 47000,
        "pf_attendance": 2000,
        "health_in": 1000,
        "labor_in": 800,
        "leave_deduction": 500,
        "absence_deduction": 0,
        "others": 200,
        "bonus": 1000
    }
}
```

### 回應欄位說明
| 欄位 | 類型 | 描述 |
|------|------|------|
| year | integer | 年份 |
| month | integer | 月份 |
| total_pay | integer | 基本薪資 |
| total_ot_pay | integer | 加班費 |
| total_sum_pay | integer | 總薪資（包含所有項目） |
| pf_attendance | integer | 全勤獎金 |
| health_in | integer | 健保費（扣款） |
| labor_in | integer | 勞保費（扣款） |
| leave_deduction | decimal | 請假扣款 |
| absence_deduction | decimal | 曠職扣款 |
| others | integer | 其他費用（扣款） |
| bonus | integer | 獎金 |

### Error Responses

#### 403 Forbidden
```json
{
    "msg": "user error",
    "res": null
}
```
**說明**：不能查詢自己的薪資

#### 404 Not Found
```json
{
    "msg": "user not found",
    "res": null
}
```
**說明**：員工不屬於當前管理者或不存在

### 說明
- 管理者不能查詢自己的薪資
- 只能查詢屬於自己管理的員工
- 需要管理者認證 token

---

## 更新員工薪資 (Update Employee Salary)

### Endpoint
```
POST /employee/{id}/salary/update
```

### 描述
更新指定員工在特定月份的薪資。可調整基本薪資、加班費、全勤獎金和獎金。更新值必須大於原有值，系統會計算差額並新增記錄。

### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

### URL Parameters
| 參數 | 類型 | 描述 |
|------|------|------|
| id | integer | 員工 ID |

### Request Body
| 參數 | 類型 | 必填 | 描述 |
|------|------|------|------|
| year | integer | 是 | 年份 (例如: 2026) |
| month | integer | 是 | 月份 (1-12) |
| total_pay | integer | 條件必填* | 基本薪資（必須大於原有值） |
| total_ot_pay | integer | 條件必填* | 加班費（必須大於原有值） |
| pf_attendance | integer | 條件必填* | 全勤獎金（必須大於原有值） |
| bonus | integer | 條件必填* | 獎金（必須大於原有值） |

**條件必填說明**：至少需要提供 total_pay、total_ot_pay、pf_attendance、bonus 其中一個欄位

### Request Example
```json
{
    "year": 2026,
    "month": 1,
    "total_pay": 42000,
    "bonus": 2000
}
```

### Success Response (200 OK)
```json
{
    "msg": "success",
    "res": {
        "year": 2026,
        "month": 1,
        "total_pay": 42000,
        "total_ot_pay": 5000,
        "total_sum_pay": 49000,
        "pf_attendance": 2000,
        "bonus": 2000
    }
}
```

### Error Responses

#### 403 Forbidden
```json
{
    "msg": "user error",
    "res": null
}
```
**說明**：不能更新自己的薪資

#### 404 Not Found
```json
{
    "msg": "user not found",
    "res": null
}
```
**說明**：員工不屬於當前管理者或不存在

#### 400 Bad Request
```json
{
    "msg": "value error",
    "res": null
}
```
**說明**：新值必須大於原有值

#### 422 Validation Error
```json
{
    "message": "The given data was invalid.",
    "errors": {
        "year": ["The year field is required."],
        "month": ["The month field is required."]
    }
}
```

### 說明
- 管理者不能更新自己的薪資
- 只能更新屬於自己管理的員工
- **重要**：新值必須大於原有值，系統會自動計算差額
- 系統會新增一筆薪資調整記錄，而非覆蓋原有記錄
- 至少需要提供一個薪資項目進行更新
- 需要管理者認證 token

---

## 獲取員工任務信息 (Get Employee Task Info)

### Endpoint
```
GET /employee/{id}/tasks/info/{year}/{month}
```

### 描述
獲取指定員工在特定月份的任務詳情，包含工作時數、加班時數、請假時數、曠職時數、遲到時數和所有任務列表（含請假記錄）。

### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

### URL Parameters
| 參數 | 類型 | 描述 |
|------|------|------|
| id | integer | 員工 ID |
| year | integer | 年份 (例如: 2026) |
| month | integer | 月份 (1-12) |

### Request Example
```
GET /v1/manage/employee/10/tasks/info/2026/1
```

### Success Response (200 OK)
```json
{
    "msg": "success",
    "res": {
        "work_hours": 160,
        "over_time": 10,
        "leave_hours": 8,
        "absent_hours": 0,
        "late_hours": 2,
        "tasks": [
            {
                "id": 1,
                "job_id": 5,
                "user_id": 10,
                "status": 3,
                "start_at": "2026-01-14 09:00:00",
                "end_at": "2026-01-14 18:00:00",
                "punch_in": "2026-01-14 09:05:00",
                "punch_out": "2026-01-14 18:10:00",
                "wage": 1600,
                "fee": 1600,
                "settlement_at": "2026-01-20 10:00:00",
                "settlement_confirmed": 1,
                "settlement_confirmed_at": "2026-01-21 15:30:00",
                "leave_logs": [
                    {
                        "id": 1,
                        "task_id": 1,
                        "employee_id": 10,
                        "leave_type_id": 2,
                        "start_at": "2026-01-15 09:00:00",
                        "end_at": "2026-01-15 13:00:00",
                        "leave_hours": 4,
                        "reason": "個人事務",
                        "leave_type": {
                            "id": 2,
                            "name": "事假",
                            "days": 0,
                            "salary_percentage": 0
                        }
                    }
                ]
            }
        ]
    }
}
```

### 回應欄位說明
| 欄位 | 類型 | 描述 |
|------|------|------|
| work_hours | integer | 總工作時數 |
| over_time | integer | 總加班時數 |
| leave_hours | integer | 總請假時數（不含曠職和遲到） |
| absent_hours | integer | 總曠職時數 |
| late_hours | integer | 總遲到時數 |
| tasks | array | 任務列表（包含請假記錄） |

#### 任務欄位說明
| 欄位 | 類型 | 描述 |
|------|------|------|
| settlement_at | datetime | 結算時間 |
| settlement_confirmed | integer | 確認結算狀態（0: 未確認, 1: 已確認） |
| settlement_confirmed_at | datetime | 確認結算時間 |

### Error Responses

#### 403 Forbidden
```json
{
    "msg": "user error",
    "res": null
}
```
**說明**：不能查詢自己的任務

#### 404 Not Found
```json
{
    "msg": "user not found",
    "res": null
}
```
**說明**：員工不屬於當前管理者或不存在

### 說明
- 管理者不能查詢自己的任務
- 只能查詢屬於自己管理的員工
- 系統會自動統計曠職、遲到和一般請假時數
- 每個任務會包含關聯的請假記錄
- 需要管理者認證 token

---

## 獲取所有員工任務信息 (Get All Employees Tasks Info)

### Endpoint
```
GET /employees/tasks/info/{year}/{month}
```

### 描述
獲取所有員工在特定月份的任務列表，按日期分組顯示。

### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

### URL Parameters
| 參數 | 類型 | 描述 |
|------|------|------|
| year | integer | 年份 (例如: 2026) |
| month | integer | 月份 (1-12) |

### Request Example
```
GET /v1/manage/employees/tasks/info/2026/1
```

### Success Response (200 OK)
```json
{
    "msg": "success",
    "res": {
        "2026-01-14": [
            {
                "id": 1,
                "job_id": 5,
                "user_id": 10,
                "user_name": "張三",
                "status": 3,
                "start_at": "2026-01-14 09:00:00",
                "end_at": "2026-01-14 18:00:00",
                "punch_in": "2026-01-14 09:05:00",
                "punch_out": "2026-01-14 18:10:00",
                "wage": 1600,
                "fee": 1600,
                "settlement_at": "2026-01-20 10:00:00",
                "settlement_confirmed": 1,
                "settlement_confirmed_at": "2026-01-21 15:30:00"
            },
            {
                "id": 2,
                "job_id": 6,
                "user_id": 11,
                "user_name": "李四",
                "status": 3,
                "start_at": "2026-01-14 08:00:00",
                "end_at": "2026-01-14 17:00:00",
                "punch_in": "2026-01-14 08:00:00",
                "punch_out": "2026-01-14 17:00:00",
                "wage": 1800,
                "fee": 1800,
                "settlement_at": "2026-01-20 10:00:00",
                "settlement_confirmed": 0,
                "settlement_confirmed_at": null
            }
        ],
        "2026-01-15": [
            {
                "id": 3,
                "job_id": 5,
                "user_id": 10,
                "user_name": "張三",
                "status": 2,
                "start_at": "2026-01-15 09:00:00",
                "end_at": "2026-01-15 18:00:00",
                "punch_in": null,
                "punch_out": null,
                "wage": 1600,
                "fee": 0
            }
        ]
    }
}
```

### 回應說明
- 數據按日期（YYYY-MM-DD）分組
- 每個日期下包含該日所有員工的任務列表
- 每個任務會自動添加 `user_name` 欄位顯示員工姓名

### 說明
- 查詢當前管理者所有員工的任務
- 按日期分組便於查看每日排班狀況
- 需要管理者認證 token

---
