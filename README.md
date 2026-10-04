# Edge LLM Benchmarks

Full-stack benchmark runs of LLMs served on **local edge hardware** (NVIDIA DGX Spark / GB10 cluster) — with everything needed for independent verification: run passports, aggregate results, sanitized configs, per-task verdicts, and raw agent trajectories.

No cloud inference is used anywhere: serving, agents and verification all run on-prem.

## Runs

| Benchmark | Model | Result | Details | Raw data |
|---|---|---|---|---|
| DeepSWE v1.1 (113 tasks) | Qwen3.8-Flash-Next 125B-A5B (NVFP4, TP=2) | **54/113 = 47.8% resolved** | [PASSPORT.md](deepswe-v1.1/qwen38-flash-next-nvfp4/PASSPORT.md) · [report.html](deepswe-v1.1/qwen38-flash-next-nvfp4/report.html) | [verdicts.csv](deepswe-v1.1/qwen38-flash-next-nvfp4/verdicts.csv) · [trajectories (release asset)](../../releases) |

Supporting material:

- [vllm-v0.30-qualification/](vllm-v0.30-qualification/) — how the serving stack itself was qualified before the benchmark run (stock vLLM 0.30.0 on 2× GB10: throughput, long-context, thermals; report in Russian).

## How to verify

1. `result.json` is the raw aggregate from the [Pier](https://github.com/datacurve-ai/pier) harness — its `metrics.mean` (0.4779) equals 54/113.
2. `verdicts.csv` lists the per-task verifier reward (1 = resolved), the task language, and the agent execution window: count the 1s for the score; `agent_duration_min ≈ 90` marks tasks that hit the budget cap (90/113); the per-language split can be recomputed from the `language` column.
3. The release asset contains all 113 ATIF agent trajectories so agent behavior can be inspected end-to-end.
4. The exact engine configuration is published in [`serving-stack.md`](deepswe-v1.1/qwen38-flash-next-nvfp4/serving-stack.md) (verbatim `vllm serve` arguments and environment).

## Methodology notes

- Serving: stock vLLM 0.30.0 (official arm64 image), MTP ×3 speculative decoding with an output-safe draft vocabulary, 262k context.
- Agent containers ran with restricted egress (allow-list HTTP proxy); a small number of external hosts returned 403 to the agent — visible as `Server: squid` responses inside trajectories.
- To our knowledge the DeepSWE v1.1 run is the first published DeepSWE measurement served from an NVFP4-quantized checkpoint, and the first run entirely on DGX Spark-class edge hardware.
- Caveats and disclosures for each run are listed in its PASSPORT.md — please read them before comparing numbers across entries.

## License

Contents are released under [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/). Model outputs within trajectories belong to their respective model providers' terms; task snippets belong to the upstream task repositories.

## Contact

Open an issue in this repository, or find the corresponding community thread referenced in each run's PASSPORT.md.
