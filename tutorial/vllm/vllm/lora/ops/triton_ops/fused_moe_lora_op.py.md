# fused_moe_lora_op.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/ops/triton_ops/fused_moe_lora_op.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implement fused Triton kernels for non-quantized MoE-routed LoRA shrink and expand. / 实现非量化的、面向 MoE 路由场景的 Triton 融合 LoRA shrink 与 expand 内核。

## Line-by-Line Analysis / 逐行分析
### Routing helpers and pointer cache / 路由辅助函数与指针缓存
```python
@triton.jit
def _get_lora_id(...): ...

@triton.jit
def _get_expert_id(...): ...

@triton.jit
def _get_token_offs(...): ...

_LORA_PTR_DICT: dict[tuple[int, ...], torch.tensor] = {}
```
**EN:** These helpers translate a program ID into the active LoRA ID, expert ID, and token offsets. `_LORA_PTR_DICT` caches device-side pointer tables so a list of LoRA tensors can be accessed efficiently from Triton.
**CN:** 这些辅助函数把 program ID 映射为当前 LoRA ID、专家 ID 和 token 偏移；`_LORA_PTR_DICT` 则缓存设备侧指针表，便于 Triton 高效访问一组 LoRA 张量。

### Core fused kernel / 核心融合内核
```python
@triton.jit(...)
def _fused_moe_lora_kernel(...):
    lora_id = _get_lora_id(...)
    expert_id = _get_expert_id(...)
    offs_token = _get_token_offs(...)
    ...
    for k in range(0, grid_k):
        b = ...
        a = ...
        accumulator += tl.dot(a.to(tl.bfloat16), b.to(tl.bfloat16))
```
**EN:** `_fused_moe_lora_kernel` is the main compute path. It filters inactive adapters, resolves token routing, loads tiles from hidden states and LoRA weights, optionally uses TMA/GDC features, and accumulates a block GEMM result.
**CN:** `_fused_moe_lora_kernel` 是主要计算路径：它会跳过未启用的适配器，解析 token 路由关系，从隐藏状态和 LoRA 权重中加载 tile，并在可选的 TMA/GDC 支持下完成块级 GEMM 累加。

### Writeback policy / 写回策略
```python
if MUL_ROUTED_WEIGHT:
    moe_weight = tl.load(topk_weights_ptr + offs_token, mask=token_mask, other=0.0)
    accumulator = accumulator * moe_weight[:, None]
...
if SPLIT_K == 1:
    if ADD_INPUTS:
        prev = tl.load(c_ptrs, mask=c_mask, other=0.0)
        tl.store(c_ptrs, prev + accumulator, mask=c_mask)
    else:
        tl.store(c_ptrs, accumulator, mask=c_mask)
else:
    tl.atomic_add(c_ptrs, accumulator, mask=c_mask, sem="relaxed")
```
**EN:** After GEMM, the kernel can multiply by MoE routing weights, then either overwrite, accumulate, or atomically reduce into the output tensor depending on kernel mode.
**CN:** GEMM 完成后，内核可以再乘上 MoE 路由权重，并根据模式选择覆盖写回、累加写回或使用原子加归约到输出张量。

### Shrink launcher / Shrink 启动器
```python
@torch.inference_mode()
def _fused_moe_lora_shrink(...):
    b_ptr = _get_ptr(lora_a_stacked, device)
    grid_lora_dim, stride_tl, stride_el = _adjust_kernel_inputs(...)
    _fused_moe_lora_kernel[grid](..., MUL_ROUTED_WEIGHT=False, ADD_INPUTS=False, ...)
```
**EN:** The shrink wrapper prepares launch metadata for LoRA-A weights and writes the low-rank intermediate activations into `a_intermediate_cache1`.
**CN:** shrink 包装器为 LoRA-A 权重准备启动参数，并把低秩中间激活写入 `a_intermediate_cache1`。

### Expand launcher / Expand 启动器
```python
@torch.inference_mode()
def _fused_moe_lora_expand(...):
    out_view = output[:, :, offset : offset + num_slices * N]
    _fused_moe_lora_kernel[grid](..., MUL_ROUTED_WEIGHT=mul_routed_weight, ADD_INPUTS=True, ...)
```
**EN:** The expand wrapper reinterprets the shrink cache as a flat matrix, launches the same Triton kernel on LoRA-B weights, and accumulates results into the correct slice of the final output.
**CN:** expand 包装器把 shrink 缓存重排为扁平矩阵，随后使用同一个 Triton 内核处理 LoRA-B 权重，并将结果累加到最终输出的对应分片中。

### End-to-end orchestration / 端到端编排
```python
@torch.inference_mode()
def _fused_moe_lora(...):
    a_intermediate_cache1 = torch.zeros(...)
    _fused_moe_lora_shrink(...)
    if fully_sharded:
        a_intermediate_cache1 = tensor_model_parallel_all_reduce(...) or tensor_model_parallel_all_gather(...)
    _fused_moe_lora_expand(...)
```
**EN:** `_fused_moe_lora` validates shapes, allocates the intermediate rank-space buffer, runs shrink then expand, and inserts tensor-parallel communication when LoRA ranks are sharded across devices.
**CN:** `_fused_moe_lora` 会先校验形状并分配低秩中间缓冲区，再依次执行 shrink 和 expand；若 LoRA rank 在多卡间分片，还会插入张量并行通信。

### Registration and fake implementations / 注册与伪实现
```python
try:
    direct_register_custom_op(op_name="fused_moe_lora", ...)
    ...
    fused_moe_lora = torch.ops.vllm.fused_moe_lora
except AttributeError:
    fused_moe_lora = _fused_moe_lora
```
**EN:** The module registers custom operators for eager/runtime integration and falls back to the Python functions when the direct op path is unavailable.
**CN:** 该模块注册自定义算子以便与运行时集成；如果直接注册路径不可用，则回退到 Python 函数实现。

## Key Concepts / 关键概念
- **EN:** One Triton kernel serves both shrink and expand by changing pointers and flags.
- **CN:** 通过切换指针和标志位，同一个 Triton 内核同时服务 shrink 与 expand。
- **EN:** Supports naive token order and pre-sorted token routing layouts.
- **CN:** 同时支持朴素 token 顺序和预排序 token 路由布局。
- **EN:** Optional TMA/GDC paths optimize memory movement and dependent launch scheduling.
- **CN:** 可选的 TMA/GDC 路径用于优化内存搬运与依赖内核调度。
- **EN:** Fully sharded mode inserts collective communication between LoRA-A and LoRA-B stages.
- **CN:** fully sharded 模式会在 LoRA-A 与 LoRA-B 两阶段之间插入集合通信。

## Dependencies / 依赖关系
- **EN:** Uses `torch`, `triton`, and `tl` for tensor management and kernel generation.
- **CN:** 使用 `torch`、`triton` 与 `tl` 进行张量管理和内核生成。
- **EN:** Depends on tensor-parallel collectives from `vllm.distributed`.
- **CN:** 依赖 `vllm.distributed` 中的张量并行集合通信函数。
- **EN:** Uses `set_triton_allocator`, `supports_pdl`, and `supports_tma` for advanced launch features.
- **CN:** 借助 `set_triton_allocator`、`supports_pdl`、`supports_tma` 启用高级启动特性。
- **EN:** Registers operators through `direct_register_custom_op`.
- **CN:** 通过 `direct_register_custom_op` 注册算子。
