# lora_expand_op.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/ops/triton_ops/lora_expand_op.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implement the standard Triton LoRA expand operator that projects LoRA rank outputs back to hidden dimensions. / 实现标准 Triton LoRA expand 算子，把 LoRA rank 输出投影回隐藏维度。

## Line-by-Line Analysis / 逐行分析
### Triton wrapper for grouped token processing
```python
@triton.jit
def _lora_expand_kernel(...):
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
    do_expand_kernel(...)
```
**EN:** The kernel wrapper partitions work by token block, output block, slice, and LoRA ID. It skips empty work quickly and uses the sorted token index list so each CTA only touches tokens assigned to one LoRA adapter.
**CN:** 该包装核按 token 块、输出块、slice 和 LoRA ID 划分工作，能够快速跳过空任务，并利用排序后的 token 索引，使每个 CTA 只处理同一个 LoRA 适配器对应的 token。

### Host-side validation and kernel configuration
```python
@torch.inference_mode()
def _lora_expand(...):
    if no_lora_flag_cpu.item():
        return
    assert inputs.dtype in [torch.float16, torch.bfloat16, torch.float32]
    ...
    (slice_start_tensor, lora_ptr_tensor, ..., same_stride, MAX_N) = _get_lora_b_ptr(...)
    kernel_config = get_lora_op_configs(op_type="expand", ...)
    EVEN_K = K % BLOCK_K == 0
    if inputs.dtype == torch.float32 and lora_b_weights[0].dtype in [torch.float16, torch.bfloat16]:
        CAST_TYPE = True
```
**EN:** The Python front end checks shapes and dtypes, derives LoRA-B pointer metadata, loads a tuned Triton configuration, and decides whether activations must be cast down to match weight dtype during the kernel.
**CN:** Python 前端会检查形状与数据类型，推导 LoRA-B 指针元数据，加载调优后的 Triton 配置，并决定是否需要在内核中把激活降精度到与权重一致的数据类型。

### Grid selection and optional PDL support
```python
grid = (
    triton.cdiv(M, BLOCK_M) * triton.cdiv(MAX_N, BLOCK_N),
    NUM_SLICES,
    num_active_loras.item(),
)
use_gdc = supports_pdl(inputs.device) and envs.VLLM_LORA_ENABLE_DUAL_STREAM
_lora_expand_kernel[grid](..., use_gdc, num_warps=NUM_WARPS, num_ctas=NUM_CTAS, num_stages=NUM_STAGES, launch_pdl=use_gdc)
```
**EN:** The launch grid spans token tiles, slices, and active LoRA adapters. PDL/GDC is enabled only when the platform supports it and the LoRA dual-stream feature flag is turned on.
**CN:** 启动网格覆盖 token 分块、slice 和活跃 LoRA 适配器。只有平台支持且启用了 LoRA 双流特性开关时，才会开启 PDL/GDC。

### Registration fallback logic
```python
try:
    direct_register_custom_op(...)
    lora_expand = torch.ops.vllm.lora_expand
except AttributeError:
    lora_expand = _lora_expand
```
**EN:** Registration exposes a compiled operator namespace while preserving a direct Python callable fallback.
**CN:** 注册后可通过编译好的算子命名空间访问，同时保留直接调用 Python 函数的回退方式。

## Key Concepts / 关键概念
- EN: Expand is the second LoRA projection, mapping rank-space activations back to model hidden size. / CN: Expand 是 LoRA 的第二次投影，把 rank 空间激活映射回模型隐藏维度。
- EN: Token grouping by LoRA ID avoids mixing different adapter weights inside one CTA. / CN: 按 LoRA ID 对 token 分组可避免一个 CTA 内混用不同适配器权重。
- EN: The operator is designed for dynamic numbers of active LoRAs while staying friendly to `torch.compile`. / CN: 该算子兼顾活跃 LoRA 数量的动态变化与 `torch.compile` 的友好性。

## Dependencies / 依赖关系
- EN: Depends on `kernel_utils.do_expand_kernel`, `utils._get_lora_b_ptr`, `utils.get_lora_op_configs`, `utils.supports_pdl`, `vllm.envs`, Triton, and Torch custom-op registration utilities. / CN: 依赖 `kernel_utils.do_expand_kernel`、`utils._get_lora_b_ptr`、`utils.get_lora_op_configs`、`utils.supports_pdl`、`vllm.envs`、Triton 以及 Torch 自定义算子注册工具。
