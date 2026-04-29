---
id: BLOCK-XX
title: 區塊名稱
status: draft
last_updated: YYYY-MM-DD
domain: xxx                    # 所屬 domain（wallet / profile / casino...）
android_ref: com.app.xxx.XxxView
flutter_ref: lib/features/xxx/presentation/widgets/xxx/
appears_in: []                 # 出現在哪些 SCREEN，如 [SCREEN-02, SCREEN-03]
---

# BLOCK-XX 區塊名稱

> **怎麼讀這份文件**
> 📋 PM/PO：讀「What & Why」+「Open Questions」
> 🧪 QA：讀「Scenarios」，每條 = 一個測試案例
> 🔧 Dev：全部都讀

---

## 📋 What & Why

**做什麼**
（一兩句話描述這個區塊做什麼）

**為什麼**
（為什麼把它抽成獨立 BLOCK——通常是因為跨多個畫面共用）

**出現位置**
- SCREEN-XX 的 X 區塊
- SCREEN-YY 的 Y 區塊

---

## 🧪 Scenarios

### Scenario 1: 正常載入
- **Given** ...
- **When** 元件 mount
- **Then** ...

### Scenario 2: （互動 / 錯誤情境）
- **Given** ...
- **When** ...
- **Then** ...

---

## 🔧 Implementation

**對應 code**：`lib/features/xxx/presentation/widgets/xxx/`

**API**

| Method | Path | 用途 |
|--------|------|------|
| GET | `/xxx` | ... |

**State**
- （這個 BLOCK 自帶 BLoC 還是吃父層 state？）
- （錯誤時是否影響所屬畫面？通常獨立失敗，不阻塞畫面）

**Android 對照**
- View / Composable：`XxxView`

---

## ❓ Open Questions

- [ ]
