# DeepSWE v1.1 — Qwen3.8-Flash-Next NVFP4 on 2× DGX Spark: run passport

## Result

**Resolved 54/113 = 47.8%** (n-attempts=1), full 113-task DeepSWE v1.1 run.

- Runtime: 28.2 h wall clock (2026-10-03 07:51 UTC → 2026-10-04 12:05 UTC), concurrency 6
- Tokens: 815.3M input / 9.0M output
- Hardware: 2× NVIDIA DGX Spark (GB10, sm_121, aarch64, 128 GB unified per node),
  ConnectX-7 200 Gb/s RoCE interconnect, TP=2 + expert parallel
- Checkpoint: nvidia/Qwen3.8-Flash-Next-NVFP4 — official NVIDIA QAT export (rev fc694b54),
  125B total / ~5B active MoE, MTP module + PLE included
- Serving: stock vLLM 0.30.0 (official arm64 image), MTP ×3 speculative decoding +
  ru/en/code draft vocabulary 65,536 ids (output-safe by construction), 262k context
- Harness: Pier 0.3.0 + mini-swe-agent 2.4.6 (the canonical leaderboard stack),
  temperature 0.95, top_p 1.0, 1 attempt per task, agent budget 90 min/task (task.toml default)

## Caveats (disclosed)

1. **Agent budget 90 min** — the canonical `task.toml` value; some entries cite longer
   budgets (GLM-5.3 mentions a 6h timeout). Our number is a lower bound under that
   comparison. 90/113 tasks hit the budget cap (see `agent_duration_min` in `verdicts.csv`);
   54 resolved, incl. 39 after the agent had already converged.
2. **Context 262k** (checkpoint-native). Some entries used 400k; history was truncated
   on the longest tasks.
3. Inference entirely on local edge hardware (no cloud API calls, no fee-paid tokens).
   Prefix caching enabled (standard for the harness agent loop).
4. **Restricted agent egress**: agent containers ran behind an allow-list HTTP proxy;
   a small number of external hosts returned 403 to the agent (visible as `Server: squid`
   responses in the trajectories). Package mirrors used in the canonical setup were allowed.

## Verification

- Score cross-checked against Pier's own `result.json` aggregate: metrics mean 0.4779 == 54/113.
- Per-task artifacts: `result.json` (aggregate), `config.json` (sanitized run config),
  `verdicts.csv` (per-task reward, language, agent timing), `serving-stack.md`
  (verbatim engine configuration), 113 ATIF trajectories — all in this repository
  and its release assets: https://github.com/aiglobaluser/edge-llm-benchmarks

## Contact

aiglobaluser (GitHub) — open an issue at
https://github.com/aiglobaluser/edge-llm-benchmarks/issues
