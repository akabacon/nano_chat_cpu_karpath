這份整理非常專業！你已經把 **NanoChat** 從環境配置、Rust 編譯到多階段訓練（Pretrain -> Midtrain -> SFT）的邏輯全部打通了。

針對你目前的 **RTX 4050 (6GB)** 測試環境，以及未來可能的 **8xH100** 正式環境，我將你的指令整理成一份**「階梯式執行清單」**。這份清單分為：環境準備、資料下載、單卡測試、以及**多卡正式訓練**。

---

### 第一階段：環境與 Tokenizer 編譯

這是最基礎的一步，確保 Rust BPE 速度優化到位。

```bash
# 1. 安裝環境與依賴
uv venv
source .venv/bin/activate
export UV_HTTP_TIMEOUT=600
uv sync --extra gpu

# 2. 編譯 Rust Tokenizer (沒這步訓練會極慢)
uv run maturin develop --release --manifest-path rustbpe/Cargo.toml

```

---

### 第二階段：資料集準備 (重要)

**這是你之前卡住的關鍵**。必須先有數據分片（Shards），DataLoader 才能運作。

```bash
# 1. 下載身份資訊 (用於後續 SFT)
curl -L -o identity_conversations.jsonl https://karpathy-public.s3.us-west-2.amazonaws.com/identity_conversations.jsonl

# 2. 下載初步訓練資料 (先抓 16 個分片測試，確保 4050 跑得動)
uv run python -m nanochat.dataset -n 16

# 3. 訓練 Tokenizer (針對這批資料學習詞表)
uv run python -m scripts.tok_train --max_chars=100000000

```

---

### 第三階段：RTX 4050 穩定測試指令 (Sanity Check)

在 6GB 顯存上，請使用這組參數確認流程 100% 噴出 `dt`：

```bash
# 測試 0.04B 極小模型
uv run python -m scripts.base_train \
  --depth=4 \
  --device_batch_size=1 \
  --max_seq_len=256 \
  --total_batch_size=1024 \
  --num_iterations=100 \
  --sample_every=20

```

---

### 第四階段：8xH100 正式訓練 (4 小時 Speedrun 版)

當你切換到 H100 伺服器時，直接執行這套完整流程。

```bash
# 1. 下載完整資料集 (800 個分片)
uv run python -m nanochat.dataset -n 800

# 2. 啟動 DDP 分散式預訓練 (目標 0.39B 模型)
# 設定總卡數
export NPROC_PER_NODE=8

uv run torchrun --standalone --nproc_per_node=$NPROC_PER_NODE -m scripts.base_train \
  --depth=32 \
  --device_batch_size=8 \
  --run="h100_speedrun_base"

# 3. 知識微調 (Mid-train)
uv run torchrun --standalone --nproc_per_node=$NPROC_PER_NODE -m scripts.mid_train \
  --device_batch_size=8 \
  --run="h100_speedrun_mid"

# 4. 指令微調 (SFT)
uv run torchrun --standalone --nproc_per_node=$NPROC_PER_NODE -m scripts.chat_sft \
  --run="h100_speedrun_sft"

```

---

### 第五階段：評估與對話

```bash
# 生成訓練報告
uv run python -m nanochat.report generate

# 啟動網頁介面與你自己訓練的模型對話
uv run python -m scripts.chat_web

```

---

### 💡 核心參數對照表 (針對不同顯存)

| 參數 | RTX 4050 (6GB) | H100 (80GB) | 說明 |
| --- | --- | --- | --- |
| **`--depth`** | 4 ~ 6 | 32 | 決定模型深度與維度 |
| **`--device_batch_size`** | 1 | 8 ~ 32 | 單卡一次吞多少 Token |
| **`--max_seq_len`** | 256 | 2048 | 模型的「記憶長度」 |
| **`--nproc_per_node`** | 1 | 8 | 使用的 GPU 數量 |

**目前你已經準備好資料集了嗎？** 如果已經執行了 `python -m nanochat.dataset -n 16`，你可以現在嘗試跑一次第三階段的 **4050 測試指令**，這次應該在 1 分鐘內就會看到 `step 00000` 出現！
