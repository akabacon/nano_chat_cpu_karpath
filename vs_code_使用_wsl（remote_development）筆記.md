# VS Code 使用 WSL（快速安裝與使用筆記）

## 一、安裝（只做一次）

### 1️⃣ 安裝 WSL（Windows）
在 **PowerShell（系統管理員）** 執行：
```powershell
wsl --install
```

重開機後，設定 Linux 使用者名稱與密碼。

確認成功：
```powershell
wsl --list --verbose
```
（建議顯示 WSL 2）

---

### 2️⃣ 安裝 VS Code 擴充套件

在 VS Code Extensions 搜尋並安裝：
- **Remote - WSL**

（或安裝 **Remote Development** 擴展包也可以）

---

## 二、使用方式（重點）

### 方法一：從 VS Code 連接 WSL（最推薦 ⭐）

1. 開啟 VS Code
2. 按 `Ctrl + Shift + P`
3. 執行：
```
Remote-WSL: New Window
```
4. 新視窗左下角顯示：
```
WSL: Ubuntu
```

代表已成功進入 WSL 環境 🎉

---

### 方法二：從 WSL 終端啟動 VS Code

在 Ubuntu 終端：
```bash
cd ~/project
code .
```

---

## 三、WSL 模式下怎麼用？

### ✅ 終端就是 Linux

VS Code 內建終端：
```
Ctrl + `
```

可以直接使用 Linux 指令與工具：
```bash
apt
python3
gcc
node
```

---

### ⚠️ 擴充套件要裝在 WSL

在 WSL 視窗中，像是：
- Python
- C/C++

請選擇：
```
Install in WSL
```

---

## 四、重要提醒（效能）

- ❌ 不要在 `/mnt/c/...` 目錄下開發
- ✅ 專案請放在：
```bash
/home/yourname/project
```

---

## 五、快速總結

- ✔ 用 **Remote - WSL** 讓 VS Code 連接 Linux
- ✔ VS Code 介面在 Windows，程式跑在 WSL
- ✔ 安裝一次，之後開新視窗就能用

---

（完）

