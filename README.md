# Flutter Feature Spec Skill

> 一個 Claude Code skill，用於從 Android 原生 App 逆向產出 Flutter Feature Spec（畫面 / 區塊 / 流程），輸出純 markdown，不寫 code、不寫 test。
>
> ⚠️ 預設情境是 Android → Flutter 逆向；其他來源（iOS、舊 Flutter 專案）邏輯類似但用詞需自行斟酌。

[![License](https://img.shields.io/github/license/hey-jason404/flutter-feature-spec-skill)](LICENSE)

---

## 這是什麼

這個 repo 收錄一個 Claude Code skill，定義「Feature Spec」的產生流程與文件結構：

- **SCREEN-XX** — 單一頁面
- **BLOCK-XX** — 跨頁 UI 區塊（Rule of Three：第 3 次出現才抽）
- **UC-XX** — 跨頁流程

設計目標：

- ✅ **單一來源（SSOT）** —— 多個 Flutter repo 透過 git submodule 共用同一份 skill
- ✅ **AI 友善** —— 結構化、可被 Claude Code 直接消費
- ✅ **逆向專用** —— 強制限制讀 source 的範圍、把不確定塞進 Open Questions，避免 AI 自行腦補

---

## Skill 本體

- [`SKILL.md`](./SKILL.md) — Claude Code 載入的 skill 主體
- [`templates/SCREEN.md`](./templates/SCREEN.md)
- [`templates/BLOCK.md`](./templates/BLOCK.md)
- [`templates/UC.md`](./templates/UC.md)

---

## 4. 如何使用

把本 repo 用 git submodule 接到你的 Flutter repo，掛在 Claude Code 規定的路徑 `.claude/skills/feature-spec/`。

### 4.1 第一次使用

**加 submodule（接入者一次性執行）**

在你的 Flutter repo 根目錄執行：

```bash
git submodule add https://github.com/hey-jason404/flutter-feature-spec-skill.git .claude/skills/feature-spec
```

> ⚠️ **一定用 HTTPS，不要用 SSH**。
> 本 repo 是 public，HTTPS 不需要任何 GitHub auth；SSH 反而要求每位團隊成員都先設好 GitHub SSH key，徒增摩擦。

> 📌 **路徑固定**為 `.claude/skills/feature-spec/`。Claude Code 從此路徑載入 skill，不能改成別的位置。

**Clone 含 submodule 的 repo（團隊成員）**

首次 clone：

```bash
git clone --recurse-submodules <your-repo-url>
```

已 clone 過 / 切到含 submodule 的 branch：

```bash
git submodule update --init --recursive
```

**全域設定（建議每人設一次）**

```bash
git config --global submodule.recurse true
```

之後 `git pull` / `git checkout` / `git switch` 會自動同步 submodule。

### 4.2 如何更新

skill 改了之後，下游 repo 二選一：

**A. Pin 版本（推薦）**

每次手動 bump，commit 走 review：

```bash
cd .claude/skills/feature-spec
git pull origin master
cd ../../..
git add .claude/skills/feature-spec
git commit -m "chore: bump feature-spec skill"
```

✅ skill 變更走 PR review、版本可追蹤、不會被上游 breaking change 突襲。

**B. 跟最新**

```bash
git submodule update --remote --merge
```

✅ 永遠最新；❌ 上游壞了你跟著壞。

> **預設用 A**（pin 版本）。

### 4.3 如何刪除

不再使用本 skill 時：

```bash
# 1. 取消 submodule 註冊
git submodule deinit -f .claude/skills/feature-spec

# 2. 從 working tree 與 .gitmodules 移除
git rm -f .claude/skills/feature-spec

# 3. 清理 .git/modules 殘留
rm -rf .git/modules/.claude/skills/feature-spec

# 4. Commit
git commit -m "chore: remove feature-spec skill"
```

---

## 5. 進階：讓 Claude Code 自動套用 skill（建議）

§4 完成後，skill 已接到你的 repo，Claude Code 會自動發現並依 skill 內 `description` 的觸發條件啟用。

若你想在 `CLAUDE.md` 顯式引用，加這段：

```markdown
## Feature Spec 規範

Feature Spec 撰寫流程沿用 `feature-spec` skill：

@.claude/skills/feature-spec/SKILL.md
```

並在你的 repo 建立預期路徑（skill 第一次跑時也會自動補建）：

```bash
mkdir -p src/docs/use-cases/{pages,blocks,flows}
```

---

## 6. 使用範例

完成 §4–§5 後，直接在 Claude Code 對話中下指令即可，不需要喊 skill 名稱。

### 範例 1：從 Android 逆向產出 SCREEN

```
請從 Android 逆向產出 SCREEN spec：
- Fragment：app/src/main/java/com/app/login/LoginFragment.kt
- ViewModel：app/src/main/java/com/app/login/LoginViewModel.kt
```

預期 Claude 行為：

1. 讀上述兩個檔案 + 它們**直接**引用的 Repository / Retrofit interface（one hop）
2. 回問 2–3 個關鍵問題（spec 類型確認、要保留 / 拋棄哪些 Android 行為、無法推論的 `TODO` / feature flag）
3. 你回答後產出 `src/docs/use-cases/pages/SCREEN-XX_login.md`，frontmatter + 四節（📋 / 🧪 / 🔧 / ❓）齊全
4. 不確定的事項全部進「❓ Open Questions」
5. 自動更新 `src/docs/use-cases/README.md` 索引

### 範例 2：更新已存在的 spec

```
LoginFragment 的邏輯改了，新的入口會帶一個 fromDeepLink 參數，
請更新 SCREEN-01 對應的 spec。
```

預期：Claude 走 Step 0 update flow——先讀現有 spec、再讀 Android 改動、做差異補丁，**不重寫**、**不改 ID**，新的不確定塞進 Open Questions。

### 範例 3：問 BLOCK 抽取（Rule of Three）

```
這個錢包餘額卡片在好幾個畫面都看到，要不要抽成 BLOCK？
```

預期：Claude 先 grep 所有 SCREEN spec，列出實際出現次數。少於 3 次會拒絕抽取，並告訴你目前在第幾次。

### 觸發關鍵字

下列任一條件 Claude 都會主動 invoke 此 skill：

- 編輯 `src/docs/use-cases/` 底下的檔案
- 提到 `SCREEN-XX` / `BLOCK-XX` / `UC-XX`
- 給 Android `Activity` / `Fragment` / `ViewModel` / `Composable` 路徑並要求 Flutter spec
- 問「這個畫面 / feature 該怎麼文件化」

---

## 路徑與目錄約定

本 skill 寫死下列路徑，不提供參數化：

| 路徑 | 用途 |
|---|---|
| `src/docs/use-cases/` | spec 根目錄 |
| `src/docs/use-cases/pages/` | SCREEN-XX |
| `src/docs/use-cases/blocks/` | BLOCK-XX |
| `src/docs/use-cases/flows/` | UC-XX |
| `src/docs/use-cases/README.md` | spec 索引 |

採用 convention over config——所有用此 skill 的 repo 一律遵守。如你的 repo 結構無法配合，請 fork 後自行修改 `SKILL.md` 內的路徑。

---

## 問題回報

有任何使用上的疑問、bug、或想討論 skill 內容，請到 [GitHub Issues](https://github.com/hey-jason404/flutter-feature-spec-skill/issues) 提報。

---

## 授權

[MIT](./LICENSE)
