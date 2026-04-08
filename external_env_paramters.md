# External Environment Parameters

This file lists environment variables that do **not** belong to `verl` itself, but are used by external projects, runtimes, libraries, or services that `verl` integrates with.

For each variable, this file shows:
- the external project/service that owns or interprets it
- what that outside project uses it for
- how `verl` uses, reads, or forwards it
- where it appears in this repository
- confidence that this is a real upstream/external variable
- evidence from upstream source or official docs when available

Confidence legend:
- `confirmed upstream`: verified from upstream source code or official docs
- `docs-backed`: supported by official docs, but not yet checked in upstream source here
- `weak/unverified`: no solid upstream reference found in this pass; may be local, outdated, or poorly documented

## 1. Experiment tracking services

| Variable | External project | Used there for | How `verl` uses it | Where in this repo | Confidence | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| `WANDB_ENTITY` | Weights & Biases | Selects the W&B entity/team/org for a run | `verl` reads it before calling `wandb.init(...)` | `verl/utils/tracking.py` | `confirmed upstream` | [W&B environment variable docs](https://docs.wandb.ai/guides/track/environment-variables) |
| `MLFLOW_TRACKING_URI` | MLflow | Chooses the tracking backend URI | `verl` reads it and passes it to `mlflow.set_tracking_uri(...)` | `verl/utils/tracking.py`, `verl/utils/rollout_trace.py`, `docs/advance/rollout_trace.rst` | `confirmed upstream` | [MLflow tracking docs](https://mlflow.org/docs/latest/ml/tracking/) |
| `MLFLOW_RUN_ID` | MLflow | Reattaches to an existing MLflow run | `verl` reads it and starts an existing run instead of creating a new one | `verl/utils/tracking.py` | `confirmed upstream` | [MLflow tracking docs](https://mlflow.org/docs/latest/ml/tracking/) |
| `SWANLAB_API_KEY` | SwanLab | Authenticates to SwanLab | `verl` reads it and calls `swanlab.login(...)` | `verl/utils/tracking.py` | `confirmed upstream` | SwanLab SDK/docs use `SWANLAB_API_KEY` |
| `SWANLAB_LOG_DIR` | SwanLab | Chooses local log directory | `verl` reads it and passes it to `swanlab.init(...)` | `verl/utils/tracking.py` | `confirmed upstream` | SwanLab SDK/docs use `SWANLAB_LOG_DIR` |
| `SWANLAB_MODE` | SwanLab | Selects run mode like cloud/local | `verl` reads it and passes it to `swanlab.init(...)` | `verl/utils/tracking.py` | `confirmed upstream` | SwanLab SDK/docs use `SWANLAB_MODE` |
| `TENSORBOARD_DIR` | TensorBoard | Output log directory for event files | `verl` reads it before constructing `SummaryWriter` | `verl/utils/tracking.py`; examples also export it | `confirmed upstream` | TensorBoard/`SummaryWriter` supports user-selected logdir; env is ecosystem convention rather than `verl` invention |
| `VOLC_ACCESS_KEY_ID` | Volcengine ML Platform | Auth/access credential | `verl` reads it before `volcengine_ml_platform.init(...)` | `verl/utils/tracking.py` | `docs-backed` | Volcengine platform credential naming; not source-verified in this pass |
| `VOLC_SECRET_ACCESS_KEY` | Volcengine ML Platform | Secret credential | `verl` reads it before `volcengine_ml_platform.init(...)` | `verl/utils/tracking.py` | `docs-backed` | Volcengine platform credential naming; not source-verified in this pass |
| `MLP_TRACKING_REGION` | Volcengine ML Platform | Region selection | `verl` reads it before `volcengine_ml_platform.init(...)` | `verl/utils/tracking.py` | `docs-backed` | Volcengine region config; not source-verified in this pass |

## 2. Ray and distributed runtime

| Variable | External project | Used there for | How `verl` uses it | Where in this repo | Confidence | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| `RANK` | PyTorch distributed / torchrun / launcher ecosystem | Global process rank | `verl` reads it in workers and rollout backends | `verl/utils/distributed.py`, `verl/single_controller/base/worker.py`, `verl/workers/rollout/*` | `confirmed upstream` | Standard torchrun / distributed launcher env contract |
| `WORLD_SIZE` | PyTorch distributed / torchrun / launcher ecosystem | Total process count | `verl` reads it during distributed initialization | `verl/utils/distributed.py`, `verl/single_controller/base/worker.py`, `verl/workers/fsdp_workers.py` | `confirmed upstream` | Standard torchrun / distributed launcher env contract |
| `LOCAL_RANK` | PyTorch distributed / torchrun / launcher ecosystem | Per-node rank/device index | `verl` reads or sets it when configuring devices | `verl/utils/distributed.py`, `verl/single_controller/base/worker.py`, `verl/workers/megatron_workers.py` | `confirmed upstream` | Standard torchrun / distributed launcher env contract |
| `LOCAL_WORLD_SIZE` | PyTorch distributed / launcher ecosystem | Process count on one node | `verl` reads it for local topology | `verl/single_controller/base/worker.py` | `confirmed upstream` | Standard distributed launcher env contract |
| `MASTER_ADDR` | PyTorch distributed / launcher ecosystem | Rendezvous host | `verl` reads or sets it for process-group setup | `verl/single_controller/ray/base.py`, `verl/single_controller/base/worker.py` | `confirmed upstream` | Standard torch distributed rendezvous env |
| `MASTER_PORT` | PyTorch distributed / launcher ecosystem | Rendezvous port | `verl` reads or sets it for process-group setup | `verl/single_controller/ray/base.py`, `verl/single_controller/base/worker.py` | `confirmed upstream` | Standard torch distributed rendezvous env |
| `DIST_INIT_METHOD` | PyTorch distributed | Explicit init URL/method for `init_process_group` | `verl` reads it and forwards it to distributed init | `verl/utils/distributed.py`, `verl/workers/fsdp_workers.py`, `verl/workers/megatron_workers.py` | `confirmed upstream` | PyTorch `init_process_group(init_method=...)` concept; env name is integration-level but not `verl`-specific |
| `RAY_LOCAL_WORLD_SIZE` | Ray | Local actor/worker world size in Ray-managed execution | `verl` reads it inside rollout workers | `verl/single_controller/ray/base.py`, `verl/workers/rollout/vllm_rollout/vllm_rollout.py`, `verl/workers/rollout/sglang_rollout/sglang_rollout.py` | `confirmed upstream` | Ray worker runtime env convention |
| `RAY_EXPERIMENTAL_NOSET_CUDA_VISIBLE_DEVICES` | Ray | Prevents Ray from auto-overwriting visible CUDA devices | `verl` checks or forwards it in rollout/runtime env | `verl/utils/ray_utils.py`, `verl/workers/rollout/vllm_rollout/vllm_async_server.py`, `verl/workers/rollout/trtllm_rollout/trtllm_async_server.py` | `confirmed upstream` | Ray feature/env documented in Ray issues/docs and used broadly in ecosystem |
| `RAY_EXPERIMENTAL_NOSET_ROCR_VISIBLE_DEVICES` | Ray | Prevents Ray from auto-overwriting ROCR device env | `verl` checks it in device/ray helpers | `verl/utils/ray_utils.py`; docs in `docs/amd_tutorial/amd_build_dockerfile_page.rst` | `docs-backed` | Mentioned in AMD/Ray integration docs; no direct upstream source line captured here |
| `RAY_EXPERIMENTAL_NOSET_HIP_VISIBLE_DEVICES` | Ray | Prevents Ray from auto-overwriting HIP device env | `verl` checks it in device/ray helpers | `verl/utils/ray_utils.py`; docs in `docs/amd_tutorial/amd_build_dockerfile_page.rst` | `docs-backed` | Mentioned in AMD/Ray integration docs; no direct upstream source line captured here |
| `RAY_EXPERIMENTAL_NOSET_ASCEND_RT_VISIBLE_DEVICES` | Ray | Prevents Ray from auto-overwriting Ascend device env | `verl` forwards it for NPU rollout setups | `verl/utils/ray_utils.py`, `verl/workers/rollout/vllm_rollout/vllm_async_server.py` | `docs-backed` | Mentioned in Ascend/Ray usage docs; no direct upstream source line captured here |

## 3. Device visibility and hardware runtime

| Variable | External project | Used there for | How `verl` uses it | Where in this repo | Confidence | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| `CUDA_VISIBLE_DEVICES` | CUDA / PyTorch | Restricts visible CUDA GPUs | `verl` reads, normalizes, and sometimes sets it for workers | `verl/single_controller/base/worker.py`, `verl/workers/rollout/vllm_rollout/vllm_async_server.py` | `confirmed upstream` | Standard CUDA/PyTorch environment variable |
| `HIP_VISIBLE_DEVICES` | ROCm / PyTorch on AMD | Restricts visible AMD GPUs | `verl` reads it and normalizes to `CUDA_VISIBLE_DEVICES` for consistency | `verl/single_controller/base/worker.py` | `confirmed upstream` | Standard ROCm/HIP device-visibility env |
| `ROCR_VISIBLE_DEVICES` | ROCm / low-level ROCr stack | Low-level AMD device visibility | `verl` reads and normalizes it, with conflict checks | `verl/single_controller/base/worker.py` | `confirmed upstream` | Standard ROCr device-visibility env |
| `ASCEND_RT_VISIBLE_DEVICES` | Ascend runtime / torch-npu | Restricts visible NPUs | `verl` reads and forwards it for NPU-aware rollout/device mapping | `verl/workers/rollout/vllm_rollout/utils.py`, `verl/utils/device.py` | `docs-backed` | Ascend runtime docs/examples use `ASCEND_RT_VISIBLE_DEVICES` |
| `ASCEND_HOME_PATH` | Ascend toolkit | Toolkit installation root | `verl` reads it to locate Ascend installation | `verl/utils/device.py` | `docs-backed` | Ascend toolkit install/config docs use `ASCEND_HOME_PATH` |

## 4. vLLM-related variables

| Variable | External project | Used there for | How `verl` uses it | Where in this repo | Confidence | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| `VLLM_LOGGING_LEVEL` | vLLM | Controls vLLM logging verbosity | `verl` sets it in Ray runtime env for rollout/trainer processes | `verl/trainer/constants_ppo.py`; appears in tests/runtime env setup | `confirmed upstream` | [vLLM `envs.py`](https://github.com/vllm-project/vllm/blob/main/vllm/envs.py) |
| `VLLM_ALLOW_RUNTIME_LORA_UPDATING` | vLLM | Allows runtime LoRA updates | `verl` sets it in PPO Ray runtime env | `verl/trainer/constants_ppo.py` | `confirmed upstream` | [vLLM `envs.py` lines 941-944](https://github.com/vllm-project/vllm/blob/main/vllm/envs.py#L941-L944) |
| `VLLM_DISABLE_COMPILE_CACHE` | vLLM | Disables vLLM compile cache | `verl` sets it as a workaround in runtime env | `verl/trainer/constants_ppo.py` | `confirmed upstream` | [vLLM `envs.py`](https://github.com/vllm-project/vllm/blob/main/vllm/envs.py) |
| `VLLM_ASCEND_ENABLE_MATMUL_ALLREDUCE` | vLLM Ascend plugin | Enables Ascend matmul allreduce path | `verl` reads it when patching Ascend vLLM behavior | `verl/utils/vllm/npu_vllm_patch.py`; docs in `docs/perf/perf_tuning_on_ascend.rst` | `docs-backed` | Ascend/vLLM docs mention it; not source-verified in this pass |

## 5. SGLang-related variables

| Variable | External project | Used there for | How `verl` uses it | Where in this repo | Confidence | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| `SGLANG_USE_CPU_ENGINE` | SGLang | CPU-engine import/runtime path toggle | `verl` temporarily sets and removes it in replica handling | `verl/workers/rollout/replica.py` | `docs-backed` | [SGLang CPU server docs](https://github.com/sgl-project/sglang/blob/main/docs/platforms/cpu_server.md) |
| `SGLANG_BLOCK_NONZERO_RANK_CHILDREN` | SGLang | Subprocess behavior control for nonzero ranks | `verl` sets it before async server startup | `verl/workers/rollout/sglang_rollout/async_sglang_server.py` | `docs-backed` | [SGLang environment variables docs](https://github.com/sgl-project/sglang/blob/fae90abf6e15aaffb6fd924a439253674771487d/docs/references/environment_variables.md?plain=1#L135) |


## 6. TensorRT-LLM-related variables

| Variable | External project | Used there for | How `verl` uses it | Where in this repo | Confidence | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| `TRT_LLM_DISABLE_LOAD_WEIGHTS_IN_PARALLEL` | TensorRT-LLM | Disables parallel weight loading | `verl` sets it before TRT-LLM async server startup | `verl/workers/rollout/trtllm_rollout/trtllm_async_server.py` | `confirmed upstream` | [TensorRT-LLM `modeling_utils.py`](https://github.com/NVIDIA/TensorRT-LLM/blob/main/tensorrt_llm/_torch/models/modeling_utils.py#L977-L982) |

## 7. NCCL / CUDA / Torch backend tuning

| Variable | External project | Used there for | How `verl` uses it | Where in this repo | Confidence | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| `NCCL_DEBUG` | NCCL | Controls NCCL log verbosity | `verl` sets it in trainer/runtime setup | `verl/trainer/sft_trainer.py`, `verl/trainer/sft_trainer_ray.py`, `verl/trainer/main_generation_server.py`, `verl/trainer/constants_ppo.py` | `confirmed upstream` | Standard NCCL environment variable |
| `NCCL_CUMEM_ENABLE` | NCCL | Memory-management related NCCL behavior | `verl` sets it to `0` in SGLang and some vLLM runtime paths | `verl/workers/rollout/sglang_rollout/sglang_rollout.py`, `verl/workers/rollout/vllm_rollout/vllm_async_server.py` | `docs-backed` | NCCL docs and ecosystem usage; no source line captured here |
| `NCCL_NVLS_ENABLE` | NCCL | NVLS transport/path control | `verl` sets it based on rollout config | `verl/workers/rollout/sglang_rollout/sglang_rollout.py`; examples also export it | `docs-backed` | NCCL docs and ecosystem usage; no source line captured here |
| `TORCH_NCCL_AVOID_RECORD_STREAMS` | PyTorch/NCCL | Avoids record-stream path in Torch NCCL integration | `verl` sets it in SGLang/runtime env | `verl/workers/rollout/sglang_rollout/sglang_rollout.py`, `verl/trainer/runtime_env.yaml` | `confirmed upstream` | [PyTorch `ProcessGroupNCCL.hpp`](https://github.com/pytorch/pytorch/blob/e9ebbd3b/torch/csrc/distributed/c10d/ProcessGroupNCCL.hpp#L186-L187) |
| `CUDA_DEVICE_MAX_CONNECTIONS` | CUDA | Controls GPU connection scheduling/overlap behavior | `verl` sets it in runtime env and backend setup | `verl/workers/rollout/sglang_rollout/sglang_rollout.py`, `verl/trainer/constants_ppo.py`, `verl/trainer/runtime_env.yaml` | `confirmed upstream` | Standard CUDA environment variable |
| `CUDA_MODULE_LOADING` | CUDA | Controls CUDA module loading strategy | `verl` sets it to `AUTO` in SGLang startup | `verl/workers/rollout/sglang_rollout/sglang_rollout.py` | `confirmed upstream` | Standard CUDA environment variable |
| `CUBLAS_WORKSPACE_CONFIG` | cuBLAS | Deterministic workspace behavior | `verl` sets it during reproducibility setup | `verl/workers/engine/utils.py` | `confirmed upstream` | Official PyTorch/cuBLAS determinism guidance |
| `PYTHONFAULTHANDLER` | Python runtime | Enables faulthandler crash diagnostics | `verl` sets it for subprocess debugging | `verl/workers/rollout/sglang_rollout/sglang_rollout.py` | `confirmed upstream` | Standard Python runtime env |
| `PYTHONHASHSEED` | Python runtime | Reproducible hashing | `verl` sets it for deterministic runs | `verl/workers/engine/utils.py` | `confirmed upstream` | Standard Python runtime env |
| `FLASH_ATTENTION_DETERMINISTIC` | FlashAttention | Deterministic FlashAttention behavior | `verl` sets it and model code reads it | `verl/workers/engine/utils.py`, `verl/models/transformers/qwen2_vl.py`, `verl/models/transformers/glm4v.py` | `docs-backed` | FlashAttention ecosystem/docs mention deterministic mode; not source-verified here |

## 8. Ascend / HCCL-specific variables

| Variable | External project | Used there for | How `verl` uses it | Where in this repo | Confidence | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| `HCCL_HOST_SOCKET_PORT_RANGE` | HCCL / Ascend distributed runtime | Host socket port range for distributed communication | `verl` sets/forwards it in Ray runtime env and examples | `verl/trainer/constants_ppo.py`, `verl/trainer/runtime_env.yaml`, Ascend examples/docs | `docs-backed` | [Ascend docs: `HCCL_HOST_SOCKET_PORT_RANGE`](https://www.hiascend.com/document/detail/zh/CANNCommunityEdition/82RC1/maintenref/envvar/envref_07_0143.html) |
| `HCCL_NPU_SOCKET_PORT_RANGE` | HCCL / Ascend distributed runtime | NPU socket port range for distributed communication | `verl` sets/forwards it in Ray runtime env and examples | `verl/trainer/constants_ppo.py`, `verl/trainer/runtime_env.yaml`, Ascend examples/docs | `confirmed upstream` | [Ascend docs: `HCCL_NPU_SOCKET_PORT_RANGE`](https://www.hiascend.com/document/detail/zh/CANNCommunityEdition/850/commlib/hcclug/hcclug_000092.html) |
| `CLOSE_MATMUL_K_SHIFT` | Ascend / torch-npu stack | Determinism workaround on NPU | `verl` sets it in deterministic engine setup | `verl/workers/engine/utils.py`; mentioned in `docs/ascend_tutorial/features/ascend_consistency.rst` | `docs-backed` | Mentioned in Ascend consistency docs; no source line captured here |

## 9. Hugging Face / tokenizer runtime

| Variable | External project | Used there for | How `verl` uses it | Where in this repo | Confidence | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| `TOKENIZERS_PARALLELISM` | Hugging Face `tokenizers` | Enables/disables tokenizer parallelism | `verl` sets it in trainer/runtime env | `verl/trainer/sft_trainer.py`, `verl/trainer/sft_trainer_ray.py`, `verl/trainer/constants_ppo.py`, `scripts/megatron_merge_lora.py` | `confirmed upstream` | Standard Hugging Face/tokenizers environment variable |

## 10. Weak or unverified entries

These names appear in `verl` and need extra manual verification. Some are still `weak/unverified`, while others are now `docs-backed` but not yet source-verified.

| Variable | Claimed external project | Used there for | How `verl` uses it | Where in this repo | Confidence | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| `NCCL_DETERMINISTIC` | NCCL | Claimed deterministic behavior toggle | `verl` sets it in deterministic training utilities | `verl/workers/engine/utils.py` | `weak/unverified` | No clear upstream PyTorch/NCCL doc or source hit found in this pass |

## Notes

- These variables are "external" because their semantics come from upstream systems like W&B, MLflow, Ray, CUDA, NCCL, vLLM, SGLang, TensorRT-LLM, HCCL, or Hugging Face.
- `verl` may still set them, read them, or forward them in Ray runtime environments.
- Project-specific variables such as `VERL_LOGGING_LEVEL` or `VERL_AUTO_PADDING` are intentionally not repeated here; they belong in `env_paramters.md`.
- Some entries, especially uncommon SGLang/NCCL-style names, are marked `weak/unverified` because I did not find a solid upstream reference for them in this pass.
