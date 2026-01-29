# 聊天室功能 API 文件

### ChatRoom（聊天室）

**資料表欄位：**
- `id` - 聊天室 ID
- `user_1` - 第一位用戶 ID
- `user_1_type` - 第一位用戶類型（`user` 或 `company`）
- `user_2` - 第二位用戶 ID
- `user_2_type` - 第二位用戶類型（`user` 或 `company`）
- `created_at` - 建立時間
- `updated_at` - 更新時間

**關聯：**
- `messages()` - 一對多關聯到 ChatMessage
- `latestMessage()` - 一對一關聯到最新訊息

### ChatMessage（聊天訊息）

**資料表欄位：**
- `id` - 訊息 ID
- `user_id` - 發送者 ID
- `chat_room_id` - 聊天室 ID
- `event_id` - 事件 ID
- `event_type` - 事件類型（詳見下方 event_type 對照表）
- `message` - 訊息內容（最多 1000 字元）
- `file_path` - 檔案路徑（如有附件）
- `file_type` - 檔案類型（1=圖片，2=文件）
- `is_read` - 是否已讀（boolean）
- `read_at` - 讀取時間
- `is_recalled` - 是否已撤回（boolean）
- `recalled_at` - 撤回時間
- `created_at` - 建立時間
- `updated_at` - 更新時間

**event_type 事件類型對照表：**

| event_type | 說明 | 備註 |
|-----------|------|------|
| 0 | 一般對話 | 一般文字聊天訊息，可被撤回 |
| 1 | 任務<接受/完成/邀約> | 任務相關通知 |
| 2 | 職缺<應徵/招聘> | 求職者應徵或雇主招聘 |
| 3 | 發送<職缺/任務> | 發送職缺或任務相關訊息 |
| 4 | 面試<邀約> | 面試邀約通知 |
| 5 | 任務<收取費用> | 任務費用收取通知 |
| 6 | 任務<上班打卡> | 上班打卡通知 (TaskPunchingPusherCheckCommand) |
| 7 | 任務<下班打卡> | 下班打卡通知 (TaskPunchingPusherCheckCommand) |
| 8 | 任務<15分鐘前上班打卡> | 提前15分鐘上班打卡提醒 (TaskPunchingPusherCheckCommand) |
| 9 | 任務<結清費用> | 薪資結清完成通知 (TaskCheckoutPusherCheckCommand) |
| 10 | 任務<補打卡> | 補打卡通知 |
| 12 | 履歷發送訊息 | 聊聊中發送履歷（影響個資權限判斷） |
| 13 | 合約通知 | 合約相關通知 (TaskRepoService.signContract) |
| 14 | 合約簽署完成通知 | 雙方簽署完成 (TaskContractCheckCommand) |
| 15 | 該職缺最後一次任務結束 | 該職缺所有任務已完成 (TaskPunchingPusherCheckCommand) |
| 16 | 結算完成通知
| 17 | 結算確認完成通知
| 100 | 系統文字訊息 | 系統自動發送的文字訊息 |

**重要說明：**
- 只有 `event_type = 0` 的訊息可以被撤回
- `event_type = 12` 的履歷發送記錄會影響個資顯示權限（見 UserRepository.hasFullAccessPermission）
  - 有履歷互動且合作中：可查看完整個資
  - 有履歷互動但合作結束：恢復遮罩
  - 有履歷互動但無合作：開放一個月查看權限

**關聯：**
- `room()` - 多對一關聯到 ChatRoom
- `user()` - 多對一關聯到 User

---

## API 端點

所有 API 端點需要用戶認證（`auth` middleware）。

### 1. 獲取聊天室列表

**端點：** `GET /chat/message/rooms`

**說明：** 獲取當前用戶的所有聊天室列表，包含最新訊息和未讀數量。

**請求參數：**
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| type | string | 是 | 用戶類型（`user` 或 `company`） |

**回應格式：**
```json
{
  "msg": "success",
  "res": {
    "data": [
      {
        "id": 1,
        "user_id": 2,
        "user_type": "user",
        "my_type": "company",
        "latest_message": "最新訊息內容",
        "unread_count": 3,
        "created_at": "2026-01-01 10:00:00",
        "updated_at": "2026-01-02 15:30:00"
      }
    ]
  }
}
```

---

### 2. 獲取聊天室資訊

**端點：** `GET /chat/message/room/{userId}`

**說明：** 根據對方用戶 ID 獲取或確認聊天室是否存在。

**路徑參數：**
| 參數 | 類型 | 說明 |
|------|------|------|
| userId | integer | 對方用戶 ID |

**回應格式：**
```json
{
  "msg": "success",
  "res": {
    "data": 5
  }
}
```

**錯誤回應：**
- `404` - 聊天室不存在

---

### 3. 獲取聊天訊息

**端點：** `GET /chat/messages/{roomId}`

**說明：** 獲取指定聊天室的所有訊息，並自動將未讀訊息標記為已讀。

**路徑參數：**
| 參數 | 類型 | 說明 |
|------|------|------|
| roomId | integer | 聊天室 ID |

**回應格式：**
```json
{
  "msg": "success",
  "res": {
    "data": [
      {
        "id": 1,
        "user_id": 1,
        "chat_room_id": 5,
        "event_id": 10,
        "event_type": 0,
        "message": "訊息內容",
        "file_path": null,
        "file_type": null,
        "is_read": true,
        "is_recalled": false,
        "created_at": "2026-01-02 14:20:00",
        "updated_at": "2026-01-02 14:20:00"
      }
    ]
  }
}
```

**特殊處理：**
- 已撤回的訊息 `message` 欄位會被清空
- 對方未讀訊息會自動標記為已讀

**錯誤回應：**
- `404` - 聊天室不存在或無權限存取

---

### 4. 發送新訊息

**端點：** `POST /chat/message/{userId}`

**說明：** 向指定用戶發送新訊息。如果聊天室不存在會自動建立。

**路徑參數：**
| 參數 | 類型 | 說明 |
|------|------|------|
| userId | integer | 接收者用戶 ID |

**請求參數：**
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| event_id | integer | 是 | 事件 ID |
| event_type | integer | 是 | 事件類型 |
| message | string | 否 | 訊息內容（最多 1000 字元） |
| my_type | string | 是 | 發送者類型（`user` 或 `company`） |

**回應格式：**
```json
{
  "msg": "success",
  "res": {
    "data": {
      "id": 100,
      "user_id": 1,
      "chat_room_id": 5,
      "event_id": 10,
      "event_type": 0,
      "message": "訊息內容",
      "created_at": "2026-01-02 16:00:00",
      "updated_at": "2026-01-02 16:00:00"
    }
  }
}
```

**特殊行為：**
- 如果是新聊天室，會觸發 `NewChatRoomNotification` 廣播事件
- 訊息會透過 `NewChatMessage` 事件廣播到聊天室

**錯誤回應：**
- `403` - 不能發送訊息給自己
- `404` - 接收者用戶不存在

---

### 5. 上傳附件

**端點：** `POST /chat/message/{userId}/attachment`

**說明：** 向指定用戶發送包含附件的訊息。

**路徑參數：**
| 參數 | 類型 | 說明 |
|------|------|------|
| userId | integer | 接收者用戶 ID |

**請求參數：**
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| file | file | 是 | 檔案（最大 64MB） |
| file_type | integer | 是 | 檔案類型（1=圖片，2=文件） |
| message | string | 否 | 附加訊息 |
| my_type | string | 是 | 發送者類型（`user` 或 `company`） |

**回應格式：**
```json
{
  "msg": "success",
  "res": {
    "data": {
      "id": 101,
      "user_id": 1,
      "chat_room_id": 5,
      "event_id": 2,
      "event_type": 0,
      "message": "附加訊息",
      "file_path": "images/xxx.jpg",
      "file_type": 1,
      "created_at": "2026-01-02 16:05:00",
      "updated_at": "2026-01-02 16:05:00"
    }
  }
}
```

**檔案儲存：**
- 圖片儲存在 `storage/app/public/images/`
- 文件儲存在 `storage/app/public/files/`

**錯誤回應：**
- `403` - 不能發送訊息給自己
- `404` - 接收者用戶不存在
- `422` - 檔案驗證失敗

---

### 6. 撤回訊息

**端點：** `GET /chat/message/recall/{messageId}`

**說明：** 撤回已發送的訊息（僅限未讀訊息）。

**路徑參數：**
| 參數 | 類型 | 說明 |
|------|------|------|
| messageId | integer | 訊息 ID |

**回應格式：**
```json
{
  "msg": "success",
  "res": null
}
```

**限制條件：**
- 只能撤回自己發送的訊息
- 只能撤回 `event_type` 為 0 的訊息
- 只能撤回未被讀取的訊息
- 已撤回的訊息不能再次撤回

**廣播事件：**
- 成功撤回會觸發 `MessageRecalled` 廣播事件通知對方

**錯誤回應：**
- `404` - 訊息不存在
- `403` - 無權限撤回或訊息已被讀取

---

### 7. 標記訊息已讀

**端點：** `POST /chat/messages/read`

**說明：** 批量標記訊息為已讀。

**請求參數：**
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| ids | array | 是 | 訊息 ID 陣列 |
| ids.* | integer | 是 | 訊息 ID |

**請求範例：**
```json
{
  "ids": [1, 2, 3, 4, 5]
}
```

**回應格式：**
```json
{
  "msg": "success",
  "res": {
    "updated_count": 5
  }
}
```

**特殊處理：**
- 只會更新接收到的訊息（不是自己發送的）
- 只會更新未讀訊息

---

## WebSocket 廣播事件

系統使用 Laravel Broadcasting 實現即時通訊，以下是所有廣播事件：

### 1. NewChatMessage（新訊息）

**事件名稱：** `chat.message`

**頻道：** `private-chat.{roomId}`

**觸發時機：** 
- 發送新訊息時
- 上傳附件時

**廣播資料：**
```javascript
{
  chatMessage: {
    id: 100,
    user_id: 1,
    chat_room_id: 5,
    event_id: 10,
    event_type: 0,
    message: "訊息內容",
    file_path: null,
    file_type: null,
    is_read: false,
    is_recalled: false,
    created_at: "2026-01-02 16:00:00",
    updated_at: "2026-01-02 16:00:00"
  }
}
```

**前端監聽範例：**
```javascript
Echo.private(`chat.${roomId}`)
    .listen('.chat.message', (e) => {
        console.log('收到新訊息:', e.chatMessage);
        // 處理新訊息
    });
```

---

### 2. NewChatRoomNotification（新聊天室通知）

**事件名稱：** `new.chat.room`

**頻道：** `private-user.{userId}`

**觸發時機：** 
- 首次與某用戶建立聊天室時

**廣播資料：**
```javascript
{
  chatMessage: {
    id: 100,
    user_id: 1,
    chat_room_id: 5,
    // ... 其他訊息欄位
  },
  receiverId: 2
}
```

**前端監聽範例：**
```javascript
Echo.private(`user.${currentUserId}`)
    .listen('.new.chat.room', (e) => {
        console.log('收到新聊天室通知:', e.chatMessage);
        // 更新聊天室列表
    });
```

---

### 3. MessageRecalled（訊息撤回）

**事件名稱：** `message.recalled`

**頻道：** `private-chat.{roomId}`

**觸發時機：** 
- 撤回訊息時

**廣播資料：**
```javascript
{
  messageId: 100,
  chatRoomId: 5
}
```

**前端監聽範例：**
```javascript
Echo.private(`chat.${roomId}`)
    .listen('.message.recalled', (e) => {
        console.log('訊息已撤回:', e.messageId);
        // 從界面移除或更新訊息
    });
```
