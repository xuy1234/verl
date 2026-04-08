# Environment Parameters Summary

This file summarizes the main environment variables used by this repository and shows where they come from in the codebase.

Scope notes:
- This focuses on variables that `verl` reads, sets, or propagates in Python/runtime code.
- It also includes a small section for common script-only variables used by examples/tests/docs.
- "Source" means the main file(s) where the variable is read or set inside this repo.

## 1. `VERL_*` variables

| Name | Default | Function | Works for | Source |
| --- | --- | --- | --- | --- |
| `VERL_LOGGING_LEVEL` | usually `WARN`, sometimes `INFO` | Controls logger verbosity in most runtime modules | Core runtime, workers, rollout, tools | Read in many modules, e.g. `verl/workers/fsdp_workers.py`, `verl/workers/rollout/vllm_rollout/vllm_rollout.py`; exported in scripts like `verl/experimental/vla/run_pi05_libero_sac.sh` |
| `VERL_SFT_LOGGING_LEVEL` | `WARN` | Controls logging for SFT/checkpoint paths | SFT | `verl/trainer/sft_trainer.py`, `verl/trainer/sft_trainer_ray.py`, `verl/utils/checkpoint/checkpoint_handler.py`, `verl/utils/hdfs_io.py` |
| `VERL_USE_EXTERNAL_MODULES` | empty string | Imports extra external modules/plugins during package init | Core/plugin extension | `verl/__init__.py` |
| `VERL_USE_MODELSCOPE` | `False` | Enables ModelScope hub patching for model download | Model download/integration | `verl/__init__.py`; mentioned in `docs/start/quickstart.rst` |
| `VERL_AUTO_PADDING` | `FALSE` | Enables `DataProto` automatic padding | Core protocol | `verl/protocol.py`; referenced in `tests/single_controller/test_auto_padding_on_cpu.py` |
| `VERL_DATAPROTO_SERIALIZATION_METHOD` | unset | If set to `numpy`, switches DataProto serialization/deserialization mode | Core protocol | `verl/protocol.py` |
| `VERL_USE_GPT_OSS` | `0` | Enables GPT-OSS/Harmony tokenizer/encoding init in vLLM server | vLLM rollout | `verl/workers/rollout/vllm_rollout/vllm_async_server.py` |
| `VERL_VLLM_FP8_QUANT_ENABLED` | `0` unless set internally | Enables FP8 quantization-related handling for vLLM | vLLM rollout | Set in `verl/workers/rollout/vllm_rollout/vllm_async_server.py`; read in `verl/workers/rollout/vllm_rollout/utils.py` |
| `VERL_DISABLE_TRITON_FP8` | `0` | Disables Triton FP8 kernel path | FP8 kernel utils | `verl/utils/kernel/fp8_kernel.py` |
| `VERL_FORCE_DEVICE` | unset | Forces device selection such as `cpu` or `cuda:0` | Utility/test execution | `verl/utils/groupwise.py`; used in `tests/utils/test_groupwise.py` |
| `VERL_FILE_LOGGER_PATH` | unset | Explicit JSONL path for file logger output | Tracking | `verl/utils/tracking.py`; example in `examples/grpo_trainer/run_qwen3-4b_gsm8k_grpo_lora_merge.sh` |
| `VERL_FILE_LOGGER_ROOT` | `.` | Root directory used when file logger path is not set | Tracking | `verl/utils/tracking.py`; used in `tests/special_e2e/sft/test_sft_engine_all.sh` |
| `VERL_ENABLE_TRACKER` | `0` | Enables trajectory dump/tracker behavior | Debug/tracing | `verl/utils/debug/trajectory_tracker.py` |
| `VERL_TRACKER_HDFS_DIR` | unset | Output HDFS directory for trajectory tracker | Debug/tracing | `verl/utils/debug/trajectory_tracker.py` |
| `VERL_TRACKER_VERBOSE` | `0` | Verbose logging for trajectory tracker | Debug/tracing | `verl/utils/debug/trajectory_tracker.py` |
| `VERL_NPU_ENABLE_A2_PATCH_VLLM_ASCEND_MC2` | `1` | Enables Ascend A2 compatibility patch around vLLM MC2 behavior | Ascend/NPU + vLLM | `verl/utils/vllm/npu_vllm_patch.py` |

## 2. Tracking and experiment backends

| Name | Default | Function | Works for | Source |
| --- | --- | --- | --- | --- |
| `WANDB_ENTITY` | unset | W&B entity/organization name | Weights & Biases | `verl/utils/tracking.py` |
| `MLFLOW_TRACKING_URI` | `sqlite:////tmp/mlruns.db` | MLflow backend URI | MLflow | `verl/utils/tracking.py`, `verl/utils/rollout_trace.py`; documented in `docs/advance/rollout_trace.rst` |
| `MLFLOW_RUN_ID` | unset | Reattaches to an existing MLflow run if present | MLflow | `verl/utils/tracking.py` |
| `SWANLAB_API_KEY` | unset | Auth key for SwanLab | SwanLab | `verl/utils/tracking.py` |
| `SWANLAB_LOG_DIR` | `swanlog` | SwanLab log output directory | SwanLab | `verl/utils/tracking.py` |
| `SWANLAB_MODE` | `cloud` | SwanLab running mode | SwanLab | `verl/utils/tracking.py` |
| `VOLC_ACCESS_KEY_ID` | required if used | Volcengine access key | Volcengine MLP tracking | `verl/utils/tracking.py` |
| `VOLC_SECRET_ACCESS_KEY` | required if used | Volcengine secret key | Volcengine MLP tracking | `verl/utils/tracking.py` |
| `MLP_TRACKING_REGION` | required if used | Volcengine region | Volcengine MLP tracking | `verl/utils/tracking.py` |
| `TENSORBOARD_DIR` | derived path under `tensorboard_log/...` | TensorBoard output directory | TensorBoard | `verl/utils/tracking.py`; examples in `examples/sft/gsm8k/run_mimo_megatron_mtp.sh`, `examples/grpo_trainer/run_qwen3_4b_grpo_vllm_1k_npu.sh` |

## 3. Distributed runtime and worker-group variables

These are mostly framework/internal launch variables rather than normal user config knobs.

| Name | Default | Function | Works for | Source |
| --- | --- | --- | --- | --- |
| `RANK` | none | Global rank of the current process | Distributed runtime | Set in `verl/single_controller/ray/base.py`; read in `verl/single_controller/base/worker.py`, `verl/utils/distributed.py`, rollout workers |
| `WORLD_SIZE` | none | Total number of distributed processes | Distributed runtime | Set in `verl/single_controller/ray/base.py`; read in `verl/single_controller/base/worker.py`, `verl/utils/distributed.py` |
| `LOCAL_RANK` | `0` in some fallback paths | Per-node local process/device rank | Distributed runtime | Set/read in `verl/single_controller/base/worker.py`, `verl/utils/distributed.py`, `verl/workers/megatron_workers.py` |
| `LOCAL_WORLD_SIZE` | `1` fallback | Number of worker processes on a node | Distributed runtime | `verl/single_controller/base/worker.py` |
| `MASTER_ADDR` | none | Distributed rendezvous host | Distributed runtime | Set in `verl/single_controller/ray/base.py`; read in `verl/single_controller/base/worker.py` |
| `MASTER_PORT` | none | Distributed rendezvous port | Distributed runtime | Set in `verl/single_controller/ray/base.py`; read in `verl/single_controller/base/worker.py` |
| `DIST_INIT_METHOD` | unset | Optional explicit process-group init method | Advanced distributed setup | `verl/workers/fsdp_workers.py`, `verl/workers/megatron_workers.py`, `verl/utils/distributed.py` |
| `RAY_LOCAL_WORLD_SIZE` | none | Ray local world size used by rollout workers | Ray rollout runtime | Set in `verl/single_controller/ray/base.py`; read in `verl/workers/rollout/vllm_rollout/vllm_rollout.py`, `verl/workers/rollout/sglang_rollout/sglang_rollout.py` |
| `WG_BACKEND` | unset, expected `ray` | Worker-group backend selector | Single-controller/Ray | Set in `verl/single_controller/ray/base.py`; read in `verl/single_controller/base/worker.py` |
| `REDIS_STORE_SERVER_HOST` | set internally | Internal store host derived from master address | Single-controller internals | Set in `verl/single_controller/base/worker.py` |

## 4. Device visibility and hardware selection

| Name | Default | Function | Works for | Source |
| --- | --- | --- | --- | --- |
| `CUDA_VISIBLE_DEVICES` | unset | Selects visible CUDA GPUs | CUDA jobs, rollout, training | Normalized/read in `verl/single_controller/base/worker.py`; also set in examples and docs |
| `HIP_VISIBLE_DEVICES` | unset | Selects visible AMD GPUs | ROCm | Read/normalized in `verl/single_controller/base/worker.py`; documented in `docs/start/multinode.rst` |
| `ROCR_VISIBLE_DEVICES` | unset | Low-level ROCm device visibility | ROCm | Read/normalized in `verl/single_controller/base/worker.py`; documented in `docs/start/multinode.rst` |
| `ASCEND_RT_VISIBLE_DEVICES` | unset | Selects visible Ascend NPUs | Ascend/NPU | Read in `verl/workers/rollout/vllm_rollout/utils.py`; examples/docs export it |
| `ASCEND_HOME_PATH` | `/usr/local/Ascend/ascend-toolkit/latest` | Ascend toolkit installation path | Ascend/NPU | `verl/utils/device.py` |
| `RAY_EXPERIMENTAL_NOSET_CUDA_VISIBLE_DEVICES` | unset | Stops Ray from auto-setting visible CUDA devices | Ray + CUDA | Referenced in `verl/utils/ray_utils.py`, rollout async servers |
| `RAY_EXPERIMENTAL_NOSET_ROCR_VISIBLE_DEVICES` | unset | Stops Ray from auto-setting ROCR visible devices | Ray + ROCm | `verl/utils/ray_utils.py`; documented in `docs/amd_tutorial/amd_build_dockerfile_page.rst` |
| `RAY_EXPERIMENTAL_NOSET_HIP_VISIBLE_DEVICES` | unset | Stops Ray from auto-setting HIP visible devices | Ray + ROCm | `verl/utils/ray_utils.py`; documented in `docs/amd_tutorial/amd_build_dockerfile_page.rst` |
| `RAY_EXPERIMENTAL_NOSET_ASCEND_RT_VISIBLE_DEVICES` | unset | Stops Ray from auto-setting visible NPU devices | Ray + Ascend | `verl/utils/ray_utils.py`; used by vLLM async server and Ascend examples/docs |

## 5. Backend-specific variables set or propagated by `verl`

These are mostly for upstream systems like vLLM, SGLang, NCCL, CUDA, and HCCL. `verl` sets or forwards them so those systems behave correctly.

| Name | Default/value in repo | Function | Works for | Source |
| --- | --- | --- | --- | --- |
| `TOKENIZERS_PARALLELISM` | `true` | Controls tokenizer parallelism | HF/tokenizers | Set in `verl/trainer/sft_trainer.py`, `verl/trainer/sft_trainer_ray.py`, `verl/trainer/constants_ppo.py`, `scripts/megatron_merge_lora.py` |
| `NCCL_DEBUG` | `WARN` | NCCL log level | NCCL | Set in `verl/trainer/sft_trainer.py`, `verl/trainer/sft_trainer_ray.py`, `verl/trainer/main_generation_server.py`, `verl/trainer/constants_ppo.py` |
| `VLLM_LOGGING_LEVEL` | `WARN` | vLLM log level | vLLM | Set in `verl/trainer/constants_ppo.py`; also in test runtime envs |
| `VLLM_ALLOW_RUNTIME_LORA_UPDATING` | `true` | Allows runtime LoRA updates | vLLM | `verl/trainer/constants_ppo.py` |
| `VLLM_DISABLE_COMPILE_CACHE` | `1` | Disables compile cache due to known issue/workaround | vLLM | `verl/trainer/constants_ppo.py` |
| `VLLM_ASCEND_ENABLE_MATMUL_ALLREDUCE` | `0` | Controls Ascend vLLM matmul allreduce behavior | Ascend + vLLM | `verl/utils/vllm/npu_vllm_patch.py`; documented in `docs/perf/perf_tuning_on_ascend.rst` |
| `TF_CPP_MIN_LOG_LEVEL` | `3` | Silences TensorFlow logs | SGLang path | `verl/workers/rollout/sglang_rollout/sglang_rollout.py` |
| `NCCL_CUMEM_ENABLE` | `0` | NCCL memory-related workaround/tuning | SGLang/vLLM path | `verl/workers/rollout/sglang_rollout/sglang_rollout.py`, `verl/workers/rollout/vllm_rollout/vllm_async_server.py` |
| `NCCL_NVLS_ENABLE` | derived from config or examples | NCCL NVLS toggle | SGLang/NCCL | `verl/workers/rollout/sglang_rollout/sglang_rollout.py`; also set in examples |
| `TORCH_NCCL_AVOID_RECORD_STREAMS` | `1` | NCCL/Torch workaround | Torch/NCCL | `verl/workers/rollout/sglang_rollout/sglang_rollout.py`, `verl/trainer/runtime_env.yaml` |
| `CUDA_DEVICE_MAX_CONNECTIONS` | `1` or `4` depending on path | CUDA scheduling/performance tuning | CUDA, vLLM, Megatron, SGLang | `verl/workers/rollout/sglang_rollout/sglang_rollout.py`, `verl/trainer/constants_ppo.py`, `verl/trainer/runtime_env.yaml`, many examples |
| `CUDA_MODULE_LOADING` | `AUTO` | CUDA module loading mode | CUDA/SGLang | `verl/workers/rollout/sglang_rollout/sglang_rollout.py` |
| `PYTHONFAULTHANDLER` | `1` | Enables Python fault handler | SGLang subprocesses | `verl/workers/rollout/sglang_rollout/sglang_rollout.py` |
| `SGLANG_BLOCK_NONZERO_RANK_CHILDREN` | `0` | SGLang subprocess behavior tuning | SGLang | `verl/workers/rollout/sglang_rollout/async_sglang_server.py` |
| `SGLANG_USE_CPU_ENGINE` | internal temporary value | Temporary import/runtime behavior for CPU engine path | SGLang | Set/unset in `verl/workers/rollout/replica.py` |
| `TRT_LLM_DISABLE_LOAD_WEIGHTS_IN_PARALLEL` | `1` | Disables TensorRT-LLM parallel weight loading | TensorRT-LLM | `verl/workers/rollout/trtllm_rollout/trtllm_async_server.py` |
| `HCCL_HOST_SOCKET_PORT_RANGE` | `auto` in runtime env; sometimes explicit ranges in examples | HCCL socket port range | Ascend/HCCL | `verl/trainer/constants_ppo.py`, `verl/trainer/runtime_env.yaml`, Ascend examples/docs |
| `HCCL_NPU_SOCKET_PORT_RANGE` | `auto` in runtime env; sometimes explicit ranges in examples | HCCL NPU socket port range | Ascend/HCCL | `verl/trainer/constants_ppo.py`, `verl/trainer/runtime_env.yaml`, Ascend examples/docs |

## 6. Reproducibility and deterministic execution

| Name | Default/value in repo | Function | Works for | Source |
| --- | --- | --- | --- | --- |
| `PYTHONHASHSEED` | set from training seed | Reproducible Python hashing | Core reproducibility | `verl/workers/engine/utils.py` |
| `CUBLAS_WORKSPACE_CONFIG` | `:16:8` | Deterministic cuBLAS behavior | CUDA | `verl/workers/engine/utils.py` |
| `NCCL_DETERMINISTIC` | `1`, or `true` on NPU path | Deterministic NCCL behavior | NCCL | `verl/workers/engine/utils.py` |
| `FLASH_ATTENTION_DETERMINISTIC` | `1` | Deterministic FlashAttention behavior | FlashAttention/models | Set in `verl/workers/engine/utils.py`; read in `verl/models/transformers/qwen2_vl.py`, `verl/models/transformers/glm4v.py` |
| `CLOSE_MATMUL_K_SHIFT` | `1` on NPU path | Ascend determinism workaround | Ascend/NPU | `verl/workers/engine/utils.py`; mentioned in `docs/ascend_tutorial/features/ascend_consistency.rst` |
| `TORCH_CUDA_ARCH_LIST` | temporary/internal | Temporary architecture setting for import/build path | Megatron/CUDA | Set/unset in `verl/workers/engine/megatron/__init__.py` |

## 7. Cloud/preemption metadata

| Name | Default | Function | Works for | Source |
| --- | --- | --- | --- | --- |
| `MLP_CURRENT_CAPACITY_BLOCK_EXPIRATION_TIMESTAMP` | unset | Volcengine capacity block expiration metadata | Checkpoint/preemption handling | `verl/utils/checkpoint/checkpoint_manager.py` |
| `SAGEMAKER_CURRENT_CAPACITY_BLOCK_EXPIRATION_TIMESTAMP` | unset | SageMaker capacity block expiration metadata | Checkpoint/preemption handling | `verl/utils/checkpoint/checkpoint_manager.py` |

## 8. Common script-only variables in examples/tests/docs

These are widely used in shell scripts, but they are not core runtime variables parsed by `verl` Python code.

| Name | Typical use | Source examples |
| --- | --- | --- |
| `MODEL_PATH` | Model checkpoint path | many `examples/` and `verl/experimental/**/shell/*.sh` |
| `TRAIN_FILE` / `TEST_FILE` | Dataset file paths | many `examples/` and `verl/experimental/**/shell/*.sh` |
| `CKPTS_DIR` | Checkpoint output directory | many `examples/` and `verl/experimental/**/shell/*.sh` |
| `NNODES`, `NGPUS_PER_NODE` | Cluster sizing in shell launchers | many `examples/` and `verl/experimental/**/shell/*.sh` |
| `RAY_DATA_HOME` | Shared data/model root | many `examples/` and `verl/experimental/**/shell/*.sh` |
| `MUJOCO_GL` | MuJoCo backend selection | `verl/experimental/vla/run_pi05_libero_sac.sh`, related VLA scripts |
