# Serving stack (exact configuration)

Everything below is the literal configuration the engine ran with during the benchmark,
retrieved from the live container (`docker inspect`). Internal coordination addresses are
masked; everything else is verbatim.

## Engine

- Image: `vllm/vllm-openai:v0.30.0` (official, arm64; build commit `ced6857afa0ea7b2e3f0846a62e1394e90f15607`)
- Two nodes (head + worker), NVIDIA DGX Spark (GB10) each, connected via ConnectX-7 200 Gb/s RoCE

## `vllm serve` arguments (head node)

```
nvidia/Qwen3.8-Flash-Next-NVFP4
--enable-prompt-tokens-details
--served-model-name qwen38-flash-next
--tensor-parallel-size 2
--gpu-memory-utilization 0.80
--max-num-seqs 8
--max-num-batched-tokens 8192
--max-model-len 262144
--kv-cache-dtype auto
--mamba-ssm-cache-dtype bfloat16
--load-format safetensors
--safetensors-load-strategy lazy
--enable-chunked-prefill
--reasoning-parser qwen3
--enable-auto-tool-choice
--tool-call-parser qwen3_coder
--distributed-executor-backend mp
--nnodes 2
--master-addr <head-node-coordination-addr>
--master-port 50000
--enable-expert-parallel
--all2all-backend allgather_reducescatter
--speculative-config {"method":"mtp","num_speculative_tokens":3,"use_local_argmax_reduction":true,"disable_eagle_block_drop":true,"index_share_for_mtp_iteration":true}
--compilation-config {"mode":0,"cudagraph_mode":"FULL_DECODE_ONLY"}
--node-rank 0
--host 0.0.0.0
--port 8000
```

The worker node runs the same command with `--node-rank 1`.

## Relevant environment

- `VLLM_MTP_DRAFT_VOCAB=/etc/vllm-draft-vocab.txt` — custom ru/en/code draft vocabulary
  (65,536 ids) for MTP speculative decoding; output-safe by construction (only ids that
  decode identically under both full and draft vocabularies are kept).
- `TP_SOCKET_IFNAME=enp1s0f0np0`, `GLOO_SOCKET_IFNAME=enp1s0f0np0` — the ConnectX-7
  RoCE interface used for tensor-parallel transport.
- `VLLM_ALLOW_LONG_MAX_MODEL_LEN=1` — required for the 262k window on this checkpoint.
- `TRANSFORMERS_OFFLINE=1` — weights served from local disk, no hub access at runtime.

## Notes

- `--kv-cache-dtype auto` resolves to bf16 on GB10 for this checkpoint (SSM state is
  explicitly pinned to bfloat16).
- KV cache occupies ~80% of unified memory per node at GMU 0.80.
- The InfiniBand/RoCE GID index can change across node reboots; the startup gate
  auto-detects it per rank before launching the engine.
