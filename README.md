# vllm-benchmark-t4
Load-tested vLLM serving on a T4 GPU: found the throughput knee at 10–15 req/s, measured TTFT/TPOT under load, and sized capacity for production.

# vLLM Serving Benchmark on a T4 GPU

Load-tested vLLM serving Qwen2.5-0.5B-Instruct on a single NVIDIA T4 to find how much traffic one GPU can handle before latency degrades.

## Setup
- GPU: NVIDIA T4 (16 GB), Google Colab
- Model: Qwen2.5-0.5B-Instruct, FP16
- Engine: vLLM (OpenAI-compatible server)
- Workload: 1024 input tokens, 128 output tokens, 300 prompts, 10 warm-up requests

## Results

| Target rate | Achieved (req/s) | Mean TTFT | P99 TTFT | Mean TPOT |
|---|---|---|---|---|
| 5 req/s  | 4.93  | 58 ms  | 123 ms | 9.7 ms  |
| 10 req/s | 9.61  | 87 ms  | 218 ms | 18.6 ms |
| 15 req/s | 13.08 | 135 ms | 328 ms | 35.2 ms |

## Key findings
- **Saturation knee at 10–15 req/s.** At 15 req/s the GPU could only deliver 13.08 req/s, requests queued (up to 101 in flight), and TPOT nearly doubled.
- **Capacity plan: ~10 req/s per T4** keeps P99 TTFT near 200 ms with headroom.
- **Warm-up matters.** The first cold run showed 436 ms TTFT because of CUDA graph capture and cache warm-up; benchmarks should always include warm-up requests.
- **Noisy environments need repeat runs.** Colab's 2 vCPUs caused run-to-run variance, so outliers were rerun.

## How to run
Open the notebook in Colab with a T4 runtime and run all cells: install, start the server, wait until ready, then benchmark.

## What I'd do next
- Tune `max-num-seqs` and test longer prompts
- Deploy on Kubernetes with autoscaling based on queue depth
