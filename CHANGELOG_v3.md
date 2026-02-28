# v3 API 更新紀錄

> 更新日期：2026-02-28

---

## 廢棄的 Task API

以下 `TaskController` API 已廢棄，請遷移至對應的新 API。

| 廢棄 API                             | 取代 API                          | 說明                                                         |
| ------------------------------------ | --------------------------------- | ------------------------------------------------------------ |
| `POST /v1/task/create`               | `POST /v1/contracts`              | 透過合約建立流程取代，雙方簽署後自動產生 Task                |
| `GET /v1/task/contract/sign/{id}`    | `POST /v1/contracts/{id}/sign`    | 新流程含自動排班 + Task ↔ WorkDay 雙向綁定                   |
| `GET /v1/task/{id}/checkout/preview` | `POST /v1/salary/period/preview`  | 新版支援區間試算（日結/週結/月結），含發薪日、勞退、加班上限 |
| `POST /v1/task/checkout`             | `POST /v1/salary/period/checkout` | 同上，新版含雇主授權驗證與月薪匯入                           |

---

## 新增 / 更新 API

### 勞動契約（EmployeeContractController）

| 方法 | 路徑                           | 說明                                                              |
| ---- | ------------------------------ | ----------------------------------------------------------------- |
| POST | `/v1/contracts`                | 建立勞動契約（`legal_status` 自動從職缺帶入）                     |
| GET  | `/v1/contracts/owner`          | 雇主查詢合約列表（支援 `status`、`job_id`、`employee_id` 等篩選） |
| GET  | `/v1/contracts/employee`       | 員工查詢合約列表（支援 `status`、`job_id`、`owner_id` 等篩選）    |
| GET  | `/v1/contracts/{id}`           | 查看合約詳情（含 `work_periods`）                                 |
| POST | `/v1/contracts/{id}/update`    | 更新合約（已終止合約不可修改）                                    |
| POST | `/v1/contracts/{id}/sign`      | 簽署合約（雙方完成後自動排班 + 產生 Task）                        |
| POST | `/v1/contracts/{id}/terminate` | 終止合約                                                          |

### 排班管理（WorkScheduleController）

| 方法 | 路徑                       | 說明                      |
| ---- | -------------------------- | ------------------------- |
| POST | `/v1/work-periods`         | 建立排班週期              |
| GET  | `/v1/work-periods/{id}`    | 查詢週期（含工作日）      |
| POST | `/v1/work-periods/publish` | 發布班表                  |
| POST | `/v1/work-periods/lock`    | 鎖定班表                  |
| POST | `/v1/work-days/upsert`     | 批次新增/更新工作日       |
| POST | `/v1/schedule/generate`    | 依合約自動產生 3 個月排班 |

### 薪資管理（SalaryController）

| 方法 | 路徑                            | 說明                     |
| ---- | ------------------------------- | ------------------------ |
| GET  | `/v1/salary/monthly`            | 雇主查詢所有員工月薪摘要 |
| GET  | `/v1/salary/monthly/{userId}`   | 查詢特定員工月薪明細     |
| GET  | `/v1/task/{id}/settlement/slip` | 取得單一任務薪資結算單   |
| POST | `/v1/salary/period/preview`     | 區間薪資試算（不寫 DB）  |
| POST | `/v1/salary/period/checkout`    | 區間薪資正式結算         |

### 仍保留的 Task API

| 方法 | 路徑                               | 說明         |
| ---- | ---------------------------------- | ------------ |
| GET  | `/v1/tasks/{user_id?}`             | 任務列表     |
| GET  | `/v1/task/{id}`                    | 任務詳情     |
| GET  | `/v1/task/{id}/punch/{action}`     | 打卡         |
| POST | `/v1/task/re-punch`                | 補打卡       |
| POST | `/v1/task/{id}/settlement/confirm` | 員工確認結算 |
| POST | `/v1/task/settlement/batchConfirm` | 批量確認結算 |
