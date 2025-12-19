# 🚀 NanoChat 部署與訓練全攻略 (RTX 4050 / H100 雙模版)

## 一、 環境初始化

此步驟確保編譯環境（Rust）與 Python 虛擬環境就緒。

```bash
# 1. 更新系統並安裝基礎工具
sudo apt update && sudo apt install -y python3-pip git build-essential

# 2. 安裝 Rust (Tokenizer 編譯核心，必要)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source $HOME/.cargo/env

# 3. 安裝 uv 加速器
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.cargo/env
export PATH="$HOME/.local/bin:$PATH"

# 4. 安裝依賴 (針對台灣網路優化)
export UV_HTTP_TIMEOUT=600
uv venv
source .venv/bin/activate
uv sync --extra gpu

# 5. 編譯 Rust Tokenizer (沒這步無法執行訓練)
uv run maturin develop --release --manifest-path rustbpe/Cargo.toml

```

---

## 二、 核心關鍵：資料集準備

**如果你跳過這步，訓練會永遠卡在 `Distributed world size` 畫面。**

```bash
# 設定環境變數 (根據你的目錄位置)
export NANOCHAT_BASE_DIR=$(pwd)

# 1. 下載身分識別對話資料
curl -L -o $NANOCHAT_BASE_DIR/identity_conversations.jsonl https://karpathy-public.s3.us-west-2.amazonaws.com/identity_conversations.jsonl

# 2. 下載預訓練數據分片 (Shard)
# 先下載 16 個分片進行小規模測試
uv run python -m nanochat.dataset -n 16

# 3. (選配) 若要正式完整訓練，下載 800 個分片 (需較大硬碟空間)
# uv run python -m nanochat.dataset -n 800

```

---

## 三、 訓練指令 (根據你的硬體選擇)

### 方案 A：筆電/單卡測試 (RTX 4050 6GB)

專為小顯存設計，關閉編譯以求快速看到結果。

```bash
# 深度設為 4，自動推算 dim=256，適合 6GB VRAM
uv run python -m scripts.base_train \
  --depth=4 \
  --device_batch_size=1 \
  --max_seq_len=256 \
  --total_batch_size=512 \
  --num_iterations=100 \
  --sample_every=20

```

### 方案 B：高階單卡實測 (RTX 3090 / 4090)

可以觀察 Loss 下降與 `dt` 表現。

```bash
uv run python -m scripts.base_train \
  --depth=12 \
  --device_batch_size=4 \
  --max_seq_len=512 \
  --total_batch_size=4096 \
  --num_iterations=100 \
  --sample_every=50

```

### 方案 C：專業伺服器 (8x H100 滿血版)

這是原作者的 31 小時訓練配置（預算約 $1000 USD）。

```bash
# 確保環境變數正確
export NPROC_PER_NODE=8

# 啟動分散式訓練
uv run torchrun --standalone --nproc_per_node=$NPROC_PER_NODE -m scripts.base_train \
  --depth=32 \
  --device_batch_size=8 \
  --run="h100_speedrun_01"

```

---

## 四、 常見問題排除 (Cheat Sheet)

| 錯誤訊息 / 現象 | 原因 | 解決方案 |
| --- | --- | --- |
| `invalid device ordinal` | 請求的 GPU 數量超過實際擁有數 | 檢查 `nproc_per_node` 是否設為 1 |
| `ValueError: Unknown config key` | 試圖從 CLI 修改 `model_dim` | 修改 `depth` 讓系統自動縮放維度 |
| **卡在啟動畫面 30 分鐘** | 數據集未下載或 `torch.compile` 過久 | 1. 執行 `nanochat.dataset` <br>

<br> 2. 調小 `depth` |
| `maturin` 找不到編譯器 | Rust 環境變數未載入 | 執行 `source $HOME/.cargo/env` |
| **GPU 佔用極低 (2%)** | 系統正在使用 Swap (顯存爆了) | 調小 `max_seq_len` 或 `device_batch_size` |

---

## 五、 後續步驟

訓練完成後，可以使用以下指令與模型對話：

```bash
uv run python -m scripts.chat_web

```

---

**最後提醒：** 在執行 `nanochat.dataset -n 16` 時，請確保你的網路暢通，因為它會從 S3 下載約數 GB 的數據。

