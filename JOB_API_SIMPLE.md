# Job API - 搜尋與建立

## 搜尋工作職缺

**端點：** `GET /jobs`

**說明：** 根據條件搜尋工作職缺（只搜尋 status=1 的啟用職缺）。

**查詢參數：**
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| keyword | string | 否 | 關鍵字搜尋（搜尋名稱、地址、描述、標籤） |
| county | string | 否 | 縣市代碼（可用逗號分隔多個，如："01,03"） |
| district | string | 否 | 區域代碼（可用逗號分隔多個，如："0101,0102"） |
| wage_type | integer | 否 | 薪資類型 |
| wage_min | integer | 否 | 最低薪資（預設 1） |
| wage_max | integer | 否 | 最高薪資（預設 10000000） |
| ind_codes | string | 否 | 產業代碼（可用逗號分隔多個，如："123456,654321"） |
| rating | integer | 否 | 最低評分（預設 0） |
| tag | string | 否 | 標籤過濾 |
| overdue | integer | 否 | 是否包含過期職缺（0=不包含，1=包含，預設 0） |
| sort_by | string | 否 | 排序欄位（rating=評分, salary=薪資, date=日期, location=地點） |
| sort_order | string | 否 | 排序方向（asc=升序, desc=降序，預設 desc） |
| limit | integer | 否 | 回傳數量限制（預設 10） |

**特殊說明：**
- 縣市、區域、產業代碼支援多個值，使用逗號分隔
- 不設定 `overdue` 或設為 0 時，只顯示未過期的職缺
- 薪資過濾只在指定 `wage_type` 時生效
- 評分過濾會篩選大於等於指定評分的職缺

---

## 建立新工作職缺

**端點：** `POST /job/create`

**說明：** 建立新的工作職缺。需要認證。

**認證：** 必須

**請求參數：**
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| status | integer | 是 | 狀態（0=關閉，1=啟用） |
| name | string | 是 | 職缺名稱 |
| type | integer | 是 | 職缺類型 |
| industry | array | 是 | 產業類別（代碼或文字） |
| industry.* | string | 是 | 產業代碼（6位數字）或自訂名稱 |
| insurance | boolean | 是 | 是否提供保險 |
| demand_amount | integer | 是 | 需求人數 |
| seniority | integer | 是 | 年資要求 |
| address | string | 是 | 工作地址 |
| county | string | 否 | 縣市代碼（系統可自動分析） |
| district | string | 否 | 區域代碼（系統可自動分析） |
| working | object | 是 | 工作時間資訊 |
| working.type | integer | 是 | 工作時間類型（1=一般，2=輪班） |
| working.workday | array | type=2 必填 | 工作日，使用3字母英文縮寫（Mon, Tue, Wed, Thu, Fri, Sat, Sun） |
| working.li_rest_day | string | type=2 必填 | 例假日，使用3字母英文縮寫（如："Sun"） |
| working.sho_rest_day | string | type=2 必填 | 休息日，使用3字母英文縮寫（如："Sat"） |
| working.start_date | string | 否 | 開始日期 |
| working.start_time | string | 否 | 開始時間 |
| working.end_date | string | 否 | 結束日期 |
| working.end_time | string | 否 | 結束時間 |
| description | string | 是 | 職缺描述 |
| wage | object | 是 | 薪資資訊 |
| wage.type | integer | 是 | 薪資類型 |
| wage.amount | integer | 是 | 薪資金額 |
| wage.checkout_cycle | integer | 是 | 結算週期 |
| wage.checkout_date | array | 是 | 結算日期 |
| wage.checkout_date.* | integer | 是 | 日期數字 |
| tags | array | 是 | 標籤 |
| tags.* | string | 是 | 標籤內容 |
| detail.educations | array | 是 | 學歷要求 |
| detail.educations.*.department | string | 是 | 科系 |
| detail.educations.*.degree | integer | 是 | 學位等級 |
| detail.certificates | array | 是 | 證照要求 |
| detail.certificates.* | string | 是 | 證照代碼（6位數字）或自訂名稱 |
| detail.lang_skills | array | 是 | 語言技能要求 |
| detail.lang_skills.*.lang | string | 是 | 語言名稱 |
| detail.lang_skills.*.l | integer | 是 | 聽力程度（1-5） |
| detail.lang_skills.*.s | integer | 是 | 口說程度（1-5） |
| detail.lang_skills.*.r | integer | 是 | 閱讀程度（1-5） |
| detail.lang_skills.*.w | integer | 是 | 寫作程度（1-5） |
| detail.skills | array | 是 | 技能要求 |
| detail.skills.* | string | 是 | 技能代碼（6位數字）或自訂名稱 |
| detail.other_demand | string | 否 | 其他需求 |

**請求範例：**
```json
{
  "status": 1,
  "name": "全端工程師",
  "type": 1,
  "industry": ["123456", "自訂產業"],
  "insurance": true,
  "demand_amount": 2,
  "seniority": 3,
  "address": "台北市大安區信義路三段",
  "working": {
    "type": 2,
    "workday": ["Mon", "Tue", "Wed", "Thu", "Fri"],
    "li_rest_day": "Sun",
    "sho_rest_day": "Sat",
    "start_date": "2026-02-01",
    "start_time": "09:00",
    "end_date": "2026-12-31",
    "end_time": "18:00"
  },
  "description": "我們正在尋找一位有經驗的全端工程師...",
  "wage": {
    "type": 1,
    "amount": 50000,
    "checkout_cycle": 1,
    "checkout_date": [25]
  },
  "tags": ["彈性工時", "遠端工作"],
  "detail": {
    "educations": [
      {
        "department": "資訊工程",
        "degree": 3
      }
    ],
    "certificates": ["123456"],
    "lang_skills": [
      {
        "lang": "英文",
        "l": 3,
        "s": 3,
        "r": 4,
        "w": 3
      }
    ],
    "skills": ["654321", "自訂技能"],
    "other_demand": "需有團隊合作精神"
  }
}
```

**錯誤回應：**
- `404` - 產業/證照/技能代碼不存在
- `422` - 驗證失敗
