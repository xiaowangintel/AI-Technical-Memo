# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/ops/triton_ops/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provide pointer caching, kernel-config loading, capability checks, and config normalization for Triton LoRA ops. / 为 Triton LoRA 算子提供指针缓存、内核配置加载、能力检测与配置键规范化功能。

## Line-by-Line Analysis / 逐行分析
### Caching LoRA-A tensor pointers and strides
```python
def _get_lora_a_ptr(lora_a_weights: list[torch.Tensor], device: torch.device):
    key = tuple(lora_weight.data_ptr() for lora_weight in lora_a_weights)
    if values := _LORA_A_PTR_DICT.get(key):
        return values
    ...
    for lora_a_weight in lora_a_weights:
        if lora_a_weight.ndim == 4:
            lora_a_weight = lora_a_weight.squeeze(dim=1)
        ...
    if len(lora_a_weights) > 1:
        lora_ptr_tensor = torch.tensor(tensor_ptrs, device=device, dtype=torch.uint64)
    else:
        lora_ptr_tensor = lora_a_weights[0]
```
**EN:** `_get_lora_a_ptr` turns a list of LoRA-A tensors into cached device metadata: either a pointer LUT for multiple slices or a direct tensor reference for the single-slice fast path, plus shared stride information.
**CN:** `_get_lora_a_ptr` 会把 LoRA-A 张量列表转换为缓存的设备元数据：多切片时使用指针查找表，单切片时直接返回张量引用，并附带共享步长信息。

### Caching LoRA-B tensor pointers, slice offsets, and sizes
```python
def _get_lora_b_ptr(lora_weights: list[torch.Tensor], offset_start: int, device: torch.device):
    ...
    for lora_b_weight in lora_weights:
        if lora_b_weight.ndim == 4:
            lora_b_weight = lora_b_weight.squeeze(dim=1)
        ...
        slice_offset_lst.append(slice_offset)
        slice_offset += lora_b_weight.size(1)
        hidden_sizes.append(lora_b_weight.size(1))
    ...
    if same_stride:
        lora_strides_d0_tensor = lora_strides_d0[0]
        ...
    else:
        lora_strides_d0_tensor = torch.tensor(lora_strides_d0, device=device)
        ...
```
**EN:** `_get_lora_b_ptr` prepares the expand path’s metadata: base pointers, per-slice output offsets, hidden sizes, and either scalar or tensor stride descriptors depending on whether all slices share the same layout.
**CN:** `_get_lora_b_ptr` 为 expand 路径准备元数据：基础指针、各 slice 的输出偏移、隐藏维大小，以及在所有 slice 布局相同或不同情况下对应的标量/张量步长描述。

### Loading tuned JSON configs
```python
@functools.lru_cache
def load_lora_op_config(op_type: str, add_inputs: bool | None) -> dict | None:
    user_defined_config_folder = envs.VLLM_TUNED_CONFIG_FOLDER
    if user_defined_config_folder is not None and not is_batch_invariant:
        gpu_name = torch.cuda.get_device_name().replace(" ", "_").replace("-", "_")
        ...
        with open(str(config_path)) as f:
            config_data = json.load(f)
    else:
        config_data = None
    return config_data
```
**EN:** This function optionally loads hardware-specific LoRA kernel tuning data from JSON files selected by GPU name and operation type. If no tuned file is found, the callers fall back to built-in defaults.
**CN:** 该函数可按 GPU 名称和操作类型，从 JSON 文件中加载硬件特定的 LoRA 内核调优数据；若未找到调优文件，调用方会退回到内置默认配置。

### Selecting effective kernel parameters
```python
@functools.lru_cache
def get_lora_op_configs(...):
    if op_type == "shrink":
        default = {...}
    elif op_type in ["fused_moe_lora_w13_shrink", "fused_moe_lora_w2_shrink"]:
        default = {...}
    elif op_type in ["fused_moe_lora_w13_expand", "fused_moe_lora_w2_expand"]:
        default = {...}
    else:
        default = {...}
    ...
    config_data = load_lora_op_config(op_type, add_inputs)
    if not config_data:
        return default
    ...
    return config_data
```
**EN:** `get_lora_op_configs` encapsulates default heuristics for shrink, expand, and fused-MoE kernels, then refines them with the nearest matching tuned config from the JSON hierarchy when available.
**CN:** `get_lora_op_configs` 封装了 shrink、expand 和 fused-MoE 内核的默认启发式配置；如果存在调优 JSON，则进一步选择层级中最接近当前工作负载的配置。

### Capability probes and config-key normalization
```python
@lru_cache
def supports_pdl(device: torch.device | None = None) -> bool:
    return (
        current_platform.is_cuda()
        and current_platform.has_device_capability(90)
        and not envs.VLLM_LORA_DISABLE_PDL
    )

def _normalize_lora_config_keys(config: dict[str, int | None]) -> dict[str, int | None]:
    for key, val in config.items():
        if key.islower():
            if key.startswith("block_"):
                nk = "BLOCK_SIZE_" + key.split("_")[-1].upper()
            else:
                nk = key.upper()
```
**EN:** `supports_pdl` and `supports_tma` gate advanced CUDA features to Hopper-class GPUs, while `_normalize_lora_config_keys` converts config dictionaries into the uppercase naming convention expected by some Triton call sites.
**CN:** `supports_pdl` 和 `supports_tma` 用于把高级 CUDA 特性限制在 Hopper 级 GPU 上，而 `_normalize_lora_config_keys` 会把配置字典转换成某些 Triton 调用点所需的大写命名格式。

## Key Concepts / 关键概念
- EN: Pointer/stride metadata is cached because LoRA weights are reused across many launches. / CN: 指针和步长元数据会被缓存，因为 LoRA 权重会在多次启动中重复使用。
- EN: Kernel tuning is workload-aware and can be overridden by external JSON profiles. / CN: 内核调优与工作负载相关，并可由外部 JSON 配置覆盖。
- EN: Capability checks isolate optional features like PDL/TMA behind simple predicates. / CN: 能力检测通过简单谓词隔离了 PDL/TMA 等可选特性。

## Dependencies / 依赖关系
- EN: Depends on `torch`, `vllm.envs`, `vllm.logger`, `vllm.platforms.current_platform`, and `vllm.utils.math_utils.next_power_of_2`; used by Triton LoRA operator modules to prepare launches. / CN: 依赖 `torch`、`vllm.envs`、`vllm.logger`、`vllm.platforms.current_platform` 与 `vllm.utils.math_utils.next_power_of_2`；被 Triton LoRA 算子模块用于准备内核启动。
