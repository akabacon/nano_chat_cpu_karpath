# WSL2 Ubuntu 24.04 + CUDA + PyTorch GPU 安裝指南

本文件整理 **在 Windows + WSL2 (Ubuntu 24.04)** 環境下，
使用 **NVIDIA GPU + CUDA + PyTorch** 的必要且正確步驟。

---

## 一、系統與前置條件

### 1. Windows 端（只做一次）

- Windows 10 21H2+ 或 Windows 11
- NVIDIA 顯示卡（GTX 10xx / RTX / 更新）
- 安裝 **支援 WSL2 的 NVIDIA Driver**

驗證：
```powershell
nvidia-smi
```
應看到 GPU 與 CUDA Version（例如 13.1）

> ⚠️ Windows **不要安裝 CUDA Toolkit**

---

### 2. WSL2 Ubuntu 24.04

確認版本：
```bash
lsb_release -a
```

確認 WSL 使用 GPU：
```bash
nvidia-smi
```

> ⚠️ WSL2 **不要安裝 nvidia-driver / cuda-drivers**

---

## 二、安裝 CUDA Toolkit（WSL 專用）

### 1. 加入 NVIDIA WSL Repository

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600

wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
```

---

### 2. 安裝 CUDA Toolkit 13.0

```bash
sudo apt update
sudo apt install -y cuda-toolkit-13-0
```

---

### 3. 設定環境變數

```bash
echo 'export PATH=/usr/local/cuda-13.0/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-13.0/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

確認：
```bash
nvcc --version
```

---

## 三、安裝 PyTorch（GPU 版本）

### 1. 建立 Python 虛擬環境（建議）

```bash
sudo apt install python3-venv -y
python3 -m venv torch_env
source torch_env/bin/activate
pip install --upgrade pip
```

---

### 2. 安裝 PyTorch（CUDA 13.0）

```bash
pip install torch torchvision torchaudio \
  --index-url https://download.pytorch.org/whl/cu130
```

---

## 四、驗證 PyTorch GPU 是否可用

```python
import torch

print("CUDA 可用：", torch.cuda.is_available())
print("GPU 名稱：", torch.cuda.get_device_name(0))
print("CUDA 版本：", torch.version.cuda)
```

期望輸出：
```
CUDA 可用： True
GPU 名稱： NVIDIA ...
CUDA 版本： 13.0
```

---

### GPU 計算測試

```python
x = torch.randn(1000, 1000, device='cuda')
y = torch.randn(1000, 1000, device='cuda')
z = x @ y
print(z[0,0])
```

---

## 五、重要觀念總結

```
Windows
 └── NVIDIA Driver（唯一）
      ↓
WSL2 Ubuntu
 ├── CUDA Toolkit
 └── PyTorch (cuXXX wheel)
```

- 驅動只裝在 **Windows**
- CUDA Toolkit 裝在 **WSL2**
- PyTorch wheel 版本要對應 CUDA

---

## 六、常見錯誤

### ❌ nvcc 找不到
→ CUDA Toolkit 尚未安裝或 PATH 未設定

### ❌ torch.cuda.is_available() = False
→ PyTorch wheel 版本不對（需 cu130）

### ❌ 安裝 nvidia-driver
→ 需移除，WSL2 不能裝 Linux driver

---

## 七、適合用途

- 深度學習（PyTorch / TensorFlow）
- CUDA C/C++ 開發
- 科學計算 / GPU 加速

---

（完）