# wendy — Wendy Lai 的個人 Claude Code Plugin Marketplace

這個 repo 是一個 Claude Code plugin marketplace（名稱：`wendy`），目前包含一個 plugin：`wendy-skills`。

所有 skill 都放在 `plugins/wendy-skills/skills/` 底下，安裝後以 `/wendy-skills:<skill-name>` 呼叫。

## 技能清單

### 使用者或 model 都可以呼叫

- **[commit](plugins/wendy-skills/skills/commit/SKILL.md)**：分析目前的改動，依目的拆成多個 Conventional Commit，經你確認後逐一 commit。

### 只能由使用者手動呼叫

- **[handoff](plugins/wendy-skills/skills/handoff/SKILL.md)**：把目前對話整理成交接檔和一段可直接貼上的 prompt，交給新 session 接手。需要額外設定，見 [guides/handoff.md](guides/handoff.md)。

## 目錄結構

```
.
├── .claude-plugin/
│   └── marketplace.json          # marketplace 設定（name: wendy）
├── .gitignore
├── LICENSE                       # MIT
├── README.md
├── guides/                       # 個別 skill 的補充說明（給人看，不會隨 plugin 安裝）
│   └── <skill-name>.md
└── plugins/
    └── wendy-skills/
        ├── .claude-plugin/
        │   └── plugin.json       # plugin 設定（不設 version，改為追蹤 git commit）
        ├── LICENSE               # symlink → ../../LICENSE（從 git 來源安裝時會被複製成實體檔案）
        └── skills/
            └── <skill-name>/SKILL.md
```

新增 skill：在 `plugins/wendy-skills/skills/<skill-name>/SKILL.md` 建立檔案，並在上方「技能清單」加一行說明。如果需要額外設定或較長的說明，寫在 `guides/<skill-name>.md`，不要放進 skill 資料夾。（`docs/` 是只留在本機的筆記，已被 `.gitignore` 排除。）

新增 plugin：在 `plugins/` 下建立新資料夾，再到 `.claude-plugin/marketplace.json` 的 `plugins` 陣列加一筆設定。

## 在新裝置安裝

```bash
claude plugin marketplace add opsuuu/claude-plugins
claude plugin install wendy-skills@wendy
```

或合併成一行：

```bash
claude plugin install wendy-skills --marketplace opsuuu/claude-plugins
```

也可以在 Claude Code session 內執行 `/plugin marketplace add opsuuu/claude-plugins`，再用 `/plugin` 選擇安裝。

### 非 GitHub 主機（GitLab 等）

使用完整的 clone URL，一定要帶 `https://` 或使用 SSH 的 `git@` 格式。省略的話會被當成 GitHub 簡寫而失敗：

```bash
claude plugin marketplace add https://gitlab.com/<group>/<repo>.git
claude plugin marketplace add git@gitlab.com:<group>/<repo>.git
```

### Private repo

> Public repo 不需要任何憑證，可略過本節。

Claude Code 會直接使用那台電腦上已設定好的 git 憑證來 clone，**不會跳出任何提示**（帳密、passphrase、主機信任都不會問），憑證沒設好就會直接失敗。

GitHub 的 `owner/repo` 寫法會先嘗試 SSH，失敗才改走 HTTPS；沒有 GitHub SSH key 的電腦可設定環境變數 `CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1` 跳過 SSH 嘗試。

- **SSH**：key 已加到 GitHub，passphrase 已存進 ssh-agent / macOS Keychain（`ssh-add --apple-use-keychain <key>`），且已執行過一次 `ssh -T git@github.com` 讓主機進入 `known_hosts`。另外要在 `~/.ssh/config` 加上以下設定，否則重開機後 ssh-agent 會清空，背景更新就會靜默失敗：

  ```
  Host github.com
    AddKeysToAgent yes
    UseKeychain yes
  ```

- **HTTPS**：credential helper 已存好憑證，例如 `gh auth login` 後再執行 `gh auth setup-git`，或 `credential.helper` 為 `osxkeychain` 並手動 clone 過一次（以 PAT 當密碼）

驗證方式（模擬 Claude Code 不允許互動的情境，能直接列出 refs 就代表設定正確）：

```bash
# SSH
GIT_TERMINAL_PROMPT=0 GIT_SSH_COMMAND="ssh -o BatchMode=yes" git ls-remote git@github.com:opsuuu/claude-plugins.git

# HTTPS
GIT_TERMINAL_PROMPT=0 git ls-remote https://github.com/opsuuu/claude-plugins.git
```

每台要安裝的電腦都需各自設定一次。

## 更新

**建議做法：開啟自動更新**（自建 marketplace 預設是關閉的）

在每台裝置的 Claude Code session 執行 `/plugin` → Marketplaces 分頁 → 選 `wendy` → **Enable auto-update**。之後每次開啟 session 並送出第一則訊息後，Claude Code 會在背景（隨機延遲最多約 10 分鐘）檢查並拉取最新版。

> Private repo 注意：背景更新同樣不允許互動，若憑證沒有事先存好（見上方「Private repo」），更新會**靜默失敗**，plugin 停留在上次同步的版本，不會有任何錯誤提示。

**手動更新：**

```bash
claude plugin marketplace update wendy
claude plugin update wendy-skills@wendy
```

更新完成後需重開 session（或在 session 內執行 `/reload-plugins`）才會生效。

因為 `plugin.json` 沒有設定 `version`，更新是以 git commit 為準，不需要手動升版號。

## 本機開發與驗證

```bash
# 驗證 manifest
claude plugin validate .

# 不安裝，直接載入本機 plugin 測試
claude --plugin-dir plugins/wendy-skills
```

## 授權

本 repo 以 [MIT License](LICENSE) 授權。

## 待辦

- [ ] 確認 plugin 可用後，刪除 `~/.claude/skills/commit`、`~/.claude/skills/handoff` 原檔，並同步把全域 `~/.claude/CLAUDE.md` 裡的 `commit` / `handoff` skill 引用改成 `wendy-skills:commit` / `wendy-skills:handoff`
