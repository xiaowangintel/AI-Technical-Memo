# lora_expand_fp8_op.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/ops/triton_ops/lora_expand_fp8_op.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implement the FP8-capable Triton LoRA expand operator and register it as a Torch custom op. / 实现支持 FP8 的 Triton LoRA expand 算子，并将其注册为 Torch 自定义算子。

## Line-by-Line Analysis / 逐行分析
### Caching scale-pointer lookup tables
```python
_EXPAND_LORA_SCALE_PTR_DICT: dict[tuple[int, ...], torch.tensor] = {}

def _get_expand_lora_scale_ptr(lora_weights: list[torch.Tensor], device: torch.device):
    key = tuple(lora_weight.data_ptr() for lora_weight in lora_weights)
    if (ptr_tensor := _EXPAND_LORA_SCALE_PTR_DICT.get(key)) is not None:
        return ptr_tensor
    if len(lora_weights) > 1:
        tensor_ptrs = [lora_weight.data_ptr() for lora_weight in lora_weights]
        ptr_tensor = torch.tensor(tensor_ptrs, device=device, dtype=torch.uint64)
    else:
        ptr_tensor = lora_weights[0]
    _EXPAND_LORA_SCALE_PTR_DICT[key] = ptr_tensor
    return _EXPAND_LORA_SCALE_PTR_DICT.get(key)
```
**EN:** This helper memoizes device pointers for FP8 scale tensors. Multi-slice launches use a LUT of raw pointers, while the single-slice case passes the tensor directly to avoid extra indirection.
**CN:** 该辅助函数缓存 FP8 scale 张量的设备指针。多切片场景使用原始指针查找表，单切片场景则直接传入张量以避免额外间接访问。

### Triton kernel wrapper over per-LoRA token groups
```python
@triton.jit
def _lora_expand_kernel_fp8(...):
    pid_mn = tl.program_id(axis=0)
    pid_m = pid_mn % cta_m_num
    pid_n = (pid_mn // cta_m_num) % cta_n_num
    slice_id = tl.program_id(axis=1)
    lora_idx = tl.program_id(axis=2)
    lora_id = tl.load(lora_ids + lora_idx)
    if lora_id == -1:
        return
    ...
    ram = tl.load(cta_lora_seq_indices + offset_m)
    do_expand_kernel_fp8(...)
```
**EN:** The Triton wrapper maps CTA indices to `(token block, output block, slice, lora)` coordinates, skips inactive LoRA IDs, gathers the token rows belonging to the current LoRA, and delegates the actual FP8 matmul to `do_expand_kernel_fp8`.
**CN:** 这个 Triton 包装核把 CTA 索引映射到 `(token 块、输出块、slice、lora)` 坐标，跳过无效 LoRA ID，收集当前 LoRA 对应的 token 行，再把实际 FP8 矩阵乘法交给 `do_expand_kernel_fp8`。

### Python entry point and launch preparation
```python
@torch.inference_mode()
def _lora_expand_fp8(...):
    if no_lora_flag_cpu.item():
        return
    ...
    (slice_start_tensor, lora_ptr_tensor, ..., same_stride, MAX_N) = _get_lora_b_ptr(...)
    if b_scale is not None:
        b_scale_ptr_tensor = _get_expand_lora_scale_ptr(b_scale, inputs.device)
    kernel_config = get_lora_op_configs(op_type="expand", ...)
    grid = (
        triton.cdiv(M, BLOCK_M) * triton.cdiv(MAX_N, BLOCK_N),
        NUM_SLICES,
        num_active_loras,
    )
    _lora_expand_kernel_fp8[grid](...)
```
**EN:** The host function validates dtypes and metadata, builds pointer/stride metadata for LoRA-B and FP8 scales, selects a tuned kernel configuration, computes the 3D launch grid, and dispatches the Triton kernel.
**CN:** 该宿主函数会校验数据类型和元数据，为 LoRA-B 与 FP8 scale 构建设备指针/步长信息，选择调优后的内核配置，计算三维 launch grid，并发射 Triton 内核。

### Torch custom-op registration
```python
try:
    direct_register_custom_op(
        op_name="lora_expand_fp8",
        op_func=_lora_expand_fp8,
        mutates_args=["output_tensor"],
        fake_impl=_lora_expand_fp8_fake,
    )
    lora_expand_fp8 = torch.ops.vllm.lora_expand_fp8
except AttributeError:
    lora_expand_fp8 = _lora_expand_fp8
```
**EN:** vLLM prefers exposing the operator through `torch.ops.vllm.*` when registration is available, while still keeping a Python fallback for environments where direct registration is unsupported.
**CN:** vLLM 优先在可注册环境中通过 `torch.ops.vllm.*` 暴露该算子，同时保留 Python 回退路径，以兼容不支持直接注册的环境。

## Key Concepts / 关键概念
- EN: FP8 expand adds scale management on top of the normal LoRA-B projection path. / CN: FP8 expand 在普通 LoRA-B 投影路径之上增加了 scale 管理。
- EN: Token rows are pre-grouped by LoRA ID so each kernel instance processes a homogeneous LoRA batch. / CN: token 行会按 LoRA ID 预分组，使每个内核实例处理同质的 LoRA 批次。
- EN: Kernel launch parameters come from `get_lora_op_configs`, enabling tuned configurations per workload. / CN: 内核启动参数来自 `get_lora_op_configs`，可按工作负载选择调优配置。

## Dependencies / 依赖关系
- EN: Depends on `fp8_kernel_utils.do_expand_kernel_fp8`, `utils._get_lora_b_ptr`, `utils.get_lora_op_configs`, Triton (`tl`, `triton`), and `direct_register_custom_op`. / CN: 依赖 `fp8_kernel_utils.do_expand_kernel_fp8`、`utils._get_lora_b_ptr`、`utils.get_lora_op_configs`、Triton（`tl`、`triton`）以及 `direct_register_custom_op`。
