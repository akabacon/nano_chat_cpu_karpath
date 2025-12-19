uv run python -m scripts.base_train \
  --depth=8 \
  --device_batch_size=2 \
  --max_seq_len=512 \
  --total_batch_size=1024 \
  --num_iterations=200 \
  --sample_every=50
