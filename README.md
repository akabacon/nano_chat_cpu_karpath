# NanoChat CPU 訓練流程指南

這份指南整理了在 CPU 上訓練 NanoChat 的最小化可跑流程，保留了正確步驟，適合用於測試整個 pipeline。

---

## 1. 建立並啟用虛擬環境
```bash
python -m venv venv
source venv/bin/activate
pip install -U pip
pip install -r requirements.txt
```

## 2. 安裝 Rust（用於 rustbpe tokenizer）
```bash
sudo apt install -y curl
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source $HOME/.cargo/env
```

## 3. 訓練 tokenizer（最小化測試）
```bash
python -m scripts.tok_train --max_chars=100000
```
這會產生：
```
~/.cache/nanochat/tokenizer/tokenizer.pkl
~/.cache/nanochat/tokenizer/token_bytes.pt
```

## 4. 訓練最小化模型
```bash
export OMP_NUM_THREADS=10
export MKL_NUM_THREADS=10

python -m scripts.base_train 
  --depth=1 
  --max_seq_len=8 
  --device_batch_size=1 
  --total_batch_size=8 
  --num_iterations=1 
  --target_param_data_ratio=-1 
  --target_flops=-1 
  --eval_tokens=8 
  --core_metric_every=-1 
  --sample_every=1 
  --save_every=-1

    
python -m scripts.base_train 
  --depth=2 
  --max_seq_len=64 
  --n_head=2 
  --device_batch_size=1 
  --total_batch_size=64 
  --num_iterations=20 
  --eval_tokens=128 
  --core_metric_every=-1 
  --sample_every=10 
  --force_override_config=True

```
- `depth=2`：Transformer 層數，非常小
- `max_seq_len=64`：序列長度
- `device_batch_size=1`：每個 step 的微批次
- `total_batch_size=64`：梯度累積後的總批次大小
- `num_iterations=20`：總步數
- `eval_tokens=128`：驗證使用的 token 數
- `core_metric_every=-1`：跳過 CORE metric 評估
- `sample_every=10`：每 10 step 生成一次樣本

## 5. 檢查結果
- 確認訓練輸出中有 loss、validation bpb 與 sample generation。
- 模型 checkpoint 會保存在：`~/.cache/nanochat/base_checkpoints/d2/`。

## 6. 聊天測試（可選）
```bash
python -m scripts.chat_web
```
打開瀏覽器即可與最小模型互動。
## 7. 重新開啟（可選）
```bash
cd nanochat
python -m venv venv
```
---

### 註解
- 這個流程是 CPU 友好的最小化測試，用來確認 tokenizer 與模型能順利運行。
- 真正大模型訓練需要 GPU 與更多資源。
- 這份指南省略了 GPU、分布式、多步下載資料集等高階配置。
