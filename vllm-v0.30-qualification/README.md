# vLLM 0.30.0 qualification on 2× DGX Spark (GB10)

Qualification of the serving stack used for the DeepSWE v1.1 run: stock `vllm/vllm-openai:v0.30.0`
on two GB10 nodes (TP=2 + expert parallel), official NVFP4 checkpoint of Qwen3.8-Flash-Next.
Covers generation throughput (1–8 concurrent streams), long-context behavior to 245k,
speculative decoding, power/thermals, and self-healing of the two-node setup.

The report (`report-ru.html`) is in Russian; numbers are self-contained.
