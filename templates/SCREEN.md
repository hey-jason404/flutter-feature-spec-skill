---
id: SCREEN-XX
title: 畫面名稱
status: draft
last_updated: YYYY-MM-DD
domain: xxx           # 主 domain；跨 domain 用陣列 [auth, push]
android_ref: com.app.xxx.XxxFragment
flutter_ref: lib/features/xxx/
contains: []          # 引用的 BLOCK / UC，如 [BLOCK-01, UC-01]
---

# SCREEN-XX 畫面名稱

> **怎麼讀這份文件**
> 📋 PM/PO：讀「What & Why」+「Open Questions」
> 🧪 QA：讀「Scenarios」，每條 = 一個測試案例
> 🔧 Dev：全部都讀

---

## 📋 What & Why

**做什麼**
（一兩句話描述使用者在這頁做什麼）

**為什麼**
（為什麼需要這個畫面，業務動機）

**入口**
- （從哪些地方可以進到這個畫面）

**前置條件**
- （登入？KYC？網路？）

---

## 🧪 Scenarios

### Scenario 1: 正常進入畫面
- **Given** （前置狀態）
- **When** 進入畫面
- **Then** （預期結果）

### Scenario 2: （下一個情境）
- **Given** ...
- **When** ...
- **Then** ...

<!-- 一條一個案例，最多 6 行；超過 10 條代表這頁該拆 BLOCK / UC -->

---

## 🔧 Implementation

**對應 code**：`lib/features/xxx/`

**API**

| Method | Path | 用途 | 備註 |
|--------|------|------|------|
| GET | `/xxx` | ... | ... |

**State**
- （是否快取、何時刷新、跨畫面共用狀態）

**Error mapping**

| 情境 | AppFailure | UI 處理 |
|------|-----------|---------|
| 401 | `AuthFailure` | 全域攔截，跳登入 |
| 5xx | `ServerFailure` | 重試 |
| 無網路 | `NetworkFailure` | 錯誤畫面 + 重試 |

**Android 對照**
- Activity / Fragment：`XxxFragment`
- ViewModel：`XxxViewModel`
- 注意：（與 Android 版的差異）

---

## ❓ Open Questions

- [ ] （需 PM 確認的事項）
- [ ] （Android code 看不出意圖的部分）
