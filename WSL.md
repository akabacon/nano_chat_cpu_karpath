這是一份為你整理好的 **WSL (Windows Subsystem for Linux) 常用指令與備份教學** 筆記。你可以將以下內容複製並儲存為 `WSL_Guide.md`。

---

# WSL (Windows Subsystem for Linux) 操作指南

這份文件紀錄了 WSL 的基本安裝、狀態查詢、關閉以及系統備份（建立還原點）的完整流程。

## 1. 基本管理指令

在 Windows PowerShell 或命令提示字元 (CMD) 中使用：

| 功能 | 指令 | 說明 |
| --- | --- | --- |
| **查看狀態** | `wsl -l -v` | 列出所有已安裝的發行版、版本及執行狀態。 |
| **安裝 WSL** | `wsl --install` | 預設安裝最新版 WSL 與 Ubuntu。 |
| **搜尋發行版** | `wsl -l -o` | 線上列出所有可安裝的 Linux 版本。 |
| **關閉特定版** | `wsl -t <名稱>` | 強制停止指定的 Linux 發行版。 |
| **完全關閉** | `wsl --shutdown` | 立即停止所有 WSL 發行版及虛擬機（釋放記憶體）。 |
| **設定預設版** | `wsl -s <名稱>` | 設定開啟 `wsl` 指令時預設啟動的版本。 |

---

## 2. 備份與還原 (建立系統還原點)

由於 WSL 沒有內建的「還原點」按鈕，我們使用 **Export (匯出)** 與 **Import (匯入)** 來達成。

### A. 建立備份 (Export)

1. **關閉 WSL**：確保資料一致。
```powershell
wsl --shutdown

```


2. **執行匯出**：將系統打包成 `.tar` 檔。
```powershell
# 語法：wsl --export <發行版名稱> <備份路徑>
wsl --export Ubuntu D:\WSL_Backups\ubuntu_backup.tar

```



### B. 還原系統 (Import)

1. **註銷舊系統** (注意：這會刪除原有的所有資料)：
```powershell
wsl --unregister Ubuntu

```


2. **匯入備份檔**：
```powershell
# 語法：wsl --import <新名稱> <安裝位置> <備份檔路徑>
wsl --import Ubuntu C:\WSL_Data D:\WSL_Backups\ubuntu_backup.tar

```



### C. 恢復預設使用者 (還原後必做)

還原後預設會以 `root` 登入，請依序執行：

1. 進入 Ubuntu：`wsl`
2. 修改設定：`sudo nano /etc/wsl.conf`
3. 加入以下內容：
```ini
[user]
default=你的使用者名稱

```


4. 儲存離開後，回 PowerShell 執行 `wsl --shutdown` 重啟。

---

## 3. 進階：VHDX 快速備份 (Windows 11 適用)

如果你使用的是 Windows 11，建議使用 `--vhd` 參數，備份速度更快且維持虛擬硬碟格式：

* **備份**：`wsl --export Ubuntu D:\backup.vhdx --vhd`
* **還原**：`wsl --import Ubuntu C:\WSL_Path D:\backup.vhdx --vhd`

---

## 4. 自動化備份腳本 (PowerShell)

你可以將以下程式碼存成 `backup_wsl.ps1`，未來只需右鍵「使用 PowerShell 執行」即可：

```powershell
$name = "Ubuntu"
$date = Get-Date -Format "yyyyMMdd"
$dest = "D:\WSL_Backups\${name}_${date}.tar"

echo "正在停止 WSL..."
wsl --shutdown

echo "正在備份 $name 到 $dest..."
wsl --export $name $dest

echo "備份完成！"
pause

```

---

**需要我幫你把這個 `.md` 檔案直接轉換成可以下載的格式，或是解釋如何執行 PowerShell 腳本嗎？**
