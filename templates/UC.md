---
id: UC-XX
title: 流程名稱
status: draft
last_updated: YYYY-MM-DD
domain: xxx                    # 主 domain；跨 domain 用陣列 [auth, push]
android_ref: com.app.xxx
flutter_ref: lib/features/xxx/
appears_in: []                 # 哪些 SCREEN 會觸發這個流程
---

# UC-XX 流程名稱

> **怎麼讀這份文件**
> 📋 PM/PO：讀「What & Why」+「Open Questions」
> 🧪 QA：讀「Scenarios」，每條 = 一個測試案例
> 🔧 Dev：全部都讀

---

## 📋 What & Why

**做什麼**
（這個流程做什麼，例如「使用者透過 OTP 驗證手機號碼」）

**為什麼**
（為什麼抽成獨立 UC——通常因為跨多個畫面、多個步驟）

**觸發來源**
- SCREEN-XX「綁定手機」按鈕
- SCREEN-YY「修改手機」按鈕

**結束條件**
- 成功：（離開到哪裡、回傳什麼）
- 失敗：（取消、超時、錯誤）

---

## 🧪 Scenarios

### Scenario 1: 完整成功流程
- **Given** ...
- **When** ...
- **Then** ...

### Scenario 2: OTP 錯誤
- **Given** ...
- **When** 輸入錯誤 OTP
- **Then** ...

### Scenario 3: 重送 OTP
- **Given** ...
- **When** ...
- **Then** ...

### Scenario 4: 取消 / 返回
- **Given** ...
- **When** ...
- **Then** ...

---

## 🔧 Implementation

**對應 code**：`lib/features/xxx/`

**API**

| Method | Path | 用途 |
|--------|------|------|
| POST | `/xxx/send` | 發送 OTP |
| POST | `/xxx/verify` | 驗證 OTP |

**State**
- （流程過程中的狀態管理）

**Error mapping**

| 情境 | AppFailure | UI 處理 |
|------|-----------|---------|
| OTP 錯誤 | `OtpInvalidFailure` | inline error |
| 超過重送次數 | `OtpRateLimitFailure` | 鎖定 N 分鐘 |

**Android 對照**
- 對應的 Activity / Fragment 鏈

---

## ❓ Open Questions

- [ ]
