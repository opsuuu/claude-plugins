# handoff

把目前對話整理成一份交接檔，再產生一段可以直接貼到新 session 的 prompt，讓沒有任何上下文的新 session 也能接手工作。

## 使用方式

只能手動呼叫，model 不會自動觸發：

```
/wendy-skills:handoff <下一個 session 的重點>
```

- 有帶參數：交接檔以這個重點為主，保留相關細節，其他內容精簡帶過。
- 沒帶參數：預設下一個 session 接著做計畫的下一階段。

## 交接檔存放位置

交接檔寫在專案根目錄的 `.handoff/handoff-<YYYYMMDD>-<stage>.md`。這些檔案是給下一個 session 讀的，只留在本機，不推上遠端。

同名檔案已存在時，`handoff` 不會直接覆蓋，而是讓你選擇：加上 `-2` 等編號、換一個 stage 名稱、覆蓋舊檔，或沿用舊檔、只重新產生 prompt。

## 建議設定：把 `.handoff/` 加入全域 gitignore

每台電腦設定一次全域 gitignore，就不用逐一修改每個專案的 `.gitignore`。先查詢全域 gitignore 的位置：

```bash
git config --global core.excludesFile
```

- **有輸出**：在該檔案加入一行 `.handoff/`。
- **沒有輸出**：git 會讀取預設位置 `~/.config/git/ignore`（有設定 `$XDG_CONFIG_HOME` 時改為 `$XDG_CONFIG_HOME/git/ignore`），可直接執行：

  ```bash
  mkdir -p ~/.config/git && echo '.handoff/' >> ~/.config/git/ignore
  ```

## 沒有設定時會怎樣

- `handoff` 照常寫入交接檔，寫完後詢問是否幫你把 `.handoff/` 加入全域 gitignore。
  - 同意：直接幫你加入，並確認設定生效。不會修改專案的 `.gitignore`。
  - 不同意：記在 memory 裡，之後的 session 會提醒你處理。
- `commit` 一律不會把 `.handoff/` 裡的檔案放進 commit。
