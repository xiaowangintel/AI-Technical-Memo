# gumbel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/sample/gumbel.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_temperature_kernel`, `apply_temperature`, `tl_rand64` for the V1 `worker/gpu/sample` subsystem. / 为 V1 的 `worker/gpu/sample` 子系统实现 `_temperature_kernel`, `apply_temperature`, `tl_rand64`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import torch

from vllm.triton_utils import HAS_TRITON, tl, triton

# Smallest positive normal fp32 value. Used to clamp the uniform draw so that
# `log(u)` cannot produce -inf (and thus `-log(-log(u))` stays finite).
#
# Triton requires globals accessed from `@triton.jit` functions to be wrapped
# in `tl.constexpr(...)`. We can only do that when Triton is actually
# available — on the CPU worker path `tl` is a placeholder whose `constexpr`
# attribute is `None`, and `tl.constexpr(...)` would crash at import time.
_FP32_TINY = (
    tl.constexpr(float.fromhex("0x1p-126")) if HAS_TRITON else float.fromhex("0x1p-126")
)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `_FP32_TINY`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `_FP32_TINY`。

### `_temperature_kernel` function / `_temperature_kernel` 函数
```python
@triton.jit
def _temperature_kernel(
    logits_ptr,
    logits_stride,
    expanded_idx_mapping_ptr,
    temperature_ptr,
    vocab_size,
    BLOCK_SIZE: tl.constexpr,
):
    token_idx = tl.program_id(0)
    req_state_idx = tl.load(expanded_idx_mapping_ptr + token_idx)
    temperature = tl.load(temperature_ptr + req_state_idx).to(tl.float32)
    if temperature == 0.0 or temperature == 1.0:
        # Early return to avoid loading logits.
        return

    block_idx = tl.program_id(1)
    block = block_idx * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = block < vocab_size

    logits = tl.load(logits_ptr + token_idx * logits_stride + block, mask=mask)
    logits = logits.to(tl.float32)
    logits = logits / temperature
    tl.store(logits_ptr + token_idx * logits_stride + block, logits, mask=mask)
```
**EN:** This function implements `_temperature_kernel` within the module. Key calls include `program_id`, `load`, `to`, `store`, `arange`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_temperature_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `to`, `store`, `arange`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `apply_temperature` function / `apply_temperature` 函数
```python
def apply_temperature(
    logits: torch.Tensor,
    expanded_idx_mapping: torch.Tensor,
    temperature: torch.Tensor,
) -> None:
    num_tokens, vocab_size = logits.shape
    BLOCK_SIZE = 8192
    num_blocks = triton.cdiv(vocab_size, BLOCK_SIZE)
    _temperature_kernel[(num_tokens, num_blocks)](
        logits,
        logits.stride(0),
        expanded_idx_mapping,
        temperature,
        vocab_size,
        BLOCK_SIZE=BLOCK_SIZE,
    )
```
**EN:** This function implements `apply_temperature` within the module. Key calls include `cdiv`, `stride`.
**CN:** 该函数会实现 `apply_temperature`，其作用域位于the module。 关键调用包括 `cdiv`, `stride`。

### `tl_rand64` function / `tl_rand64` 函数
```python
@triton.jit
def tl_rand64(seed, offset, includes_zero: tl.constexpr):
    lo, hi, _, _ = tl.randint4x(seed, offset)
    lo = lo.to(tl.uint32, bitcast=True).to(tl.uint64)
    hi = hi.to(tl.uint32, bitcast=True).to(tl.uint64)
    r = (hi << 32) | lo

    # 1 / 2**64
    scale = 5.421010862427522170037e-20
    u = r.to(tl.float64) * scale
    if not includes_zero:
        u = tl.maximum(u, 2.2250738585072014e-308)  # float64 tiny
    return u
```
**EN:** This function implements `tl_rand64` within the module. Key calls include `randint4x`, `to`, `maximum`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `tl_rand64`，其作用域位于the module。 关键调用包括 `randint4x`, `to`, `maximum`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `gumbel_block_argmax` function / `gumbel_block_argmax` 函数
```python
@triton.jit
def gumbel_block_argmax(
    logits,
    block,
    mask,
    token_idx,
    expanded_idx_mapping_ptr,
    temp_ptr,
    seeds_ptr,
    pos_ptr,
    processed_logits_ptr,
    processed_logits_stride,
    processed_logits_col_ptr,
    vocab_size,
    APPLY_TEMPERATURE: tl.constexpr,
    USE_FP64: tl.constexpr,
):
    req_state_idx = tl.load(expanded_idx_mapping_ptr + token_idx)
    temp = tl.load(temp_ptr + req_state_idx).to(tl.float32)
    if temp != 0.0 and APPLY_TEMPERATURE:
        # Apply temperature.
        # NOTE(woosuk): Match the behavior of _temperature_kernel.
        # E.g., if the kernel uses tl.div_rn, we should use tl.div_rn here too.
        logits = logits / temp

    if processed_logits_ptr is not None:
        # Store the temperature-applied logits.
        if processed_logits_col_ptr is not None:
            col = tl.load(processed_logits_col_ptr)
        else:
            col = 0
        tl.store(
            processed_logits_ptr
            + req_state_idx * processed_logits_stride
            + col * vocab_size
            + block,
            logits,
            mask=mask,
        )

    # fp32 is the default reduction dtype; fp64 is ~1/32–1/64x the throughput
    # on H100/Ada/Blackwell and empirically indistinguishable for Gumbel-max.
    if USE_FP64:
        logits = logits.to(tl.float64)
    if temp != 0.0:
        # Calculate the seed for gumbel noise.
        seed = tl.load(seeds_ptr + req_state_idx)
        pos = tl.load(pos_ptr + token_idx)
        gumbel_seed = tl.randint(seed, pos)

        if USE_FP64:
            u = tl_rand64(gumbel_seed, block, includes_zero=False)
        else:
            u = tl.rand(gumbel_seed, block)
            u = tl.maximum(u, _FP32_TINY)
        gumbel_noise = -tl.log(-tl.log(u))

        # Apply gumbel noise.
        logits = tl.where(mask, logits + gumbel_noise, float("-inf"))

    value, idx = tl.max(logits, axis=0, return_indices=True)
    return value, idx
```
**EN:** This function implements `gumbel_block_argmax` within the module. Key calls include `load`, `to`, `max`, `store`, `randint`, `where`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `gumbel_block_argmax`，其作用域位于the module。 关键调用包括 `load`, `to`, `max`, `store`, `randint`, `where`。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_gumbel_sample_kernel` function / `_gumbel_sample_kernel` 函数
```python
@triton.jit
def _gumbel_sample_kernel(
    local_argmax_ptr,
    local_argmax_stride,
    local_max_ptr,
    local_max_stride,
    processed_logits_ptr,
    processed_logits_stride,
    processed_logits_col_ptr,
    logits_ptr,
    logits_stride,
    expanded_idx_mapping_ptr,
    seeds_ptr,
    pos_ptr,
    temp_ptr,
    vocab_size,
    BLOCK_SIZE: tl.constexpr,
    APPLY_TEMPERATURE: tl.constexpr,
    USE_FP64: tl.constexpr,
):
    token_idx = tl.program_id(0)
    block_idx = tl.program_id(1)
    block = block_idx * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = block < vocab_size
    logits = tl.load(
        logits_ptr + token_idx * logits_stride + block,
        mask=mask,
        other=float("-inf"),
    )
    logits = logits.to(tl.float32)

    value, idx = gumbel_block_argmax(
        logits,
        block,
        mask,
        token_idx,
        expanded_idx_mapping_ptr,
        temp_ptr,
        seeds_ptr,
        pos_ptr,
        processed_logits_ptr,
        processed_logits_stride,
        processed_logits_col_ptr,
        vocab_size,
        APPLY_TEMPERATURE=APPLY_TEMPERATURE,
        USE_FP64=USE_FP64,
    )
    token_id = block_idx * BLOCK_SIZE + idx
    tl.store(local_argmax_ptr + token_idx * local_argmax_stride + block_idx, token_id)
    tl.store(local_max_ptr + token_idx * local_max_stride + block_idx, value)
```
**EN:** This function implements `_gumbel_sample_kernel` within the module. Key calls include `program_id`, `load`, `to`, `gumbel_block_argmax`, `store`, `arange`.
**CN:** 该函数会实现 `_gumbel_sample_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `to`, `gumbel_block_argmax`, `store`, `arange`。

### `gumbel_sample` function / `gumbel_sample` 函数
```python
def gumbel_sample(
    logits: torch.Tensor,  # [num_tokens, vocab_size]
    expanded_idx_mapping: torch.Tensor,  # [num_tokens]
    temperature: torch.Tensor,  # [max_num_reqs]
    seed: torch.Tensor,  # [max_num_reqs]
    pos: torch.Tensor,  # [num_tokens]
    apply_temperature: bool,
    output_processed_logits: torch.Tensor | None = None,
    output_processed_logits_col: torch.Tensor | None = None,
    use_fp64: bool = False,
) -> torch.Tensor:
    num_tokens, vocab_size = logits.shape
    BLOCK_SIZE = 1024
    num_blocks = triton.cdiv(vocab_size, BLOCK_SIZE)
    local_argmax = logits.new_empty(num_tokens, num_blocks, dtype=torch.int64)
    local_max_dtype = torch.float64 if use_fp64 else torch.float32
    local_max = logits.new_empty(num_tokens, num_blocks, dtype=local_max_dtype)
    _gumbel_sample_kernel[(num_tokens, num_blocks)](
        local_argmax,
        local_argmax.stride(0),
        local_max,
        local_max.stride(0),
        output_processed_logits,
        output_processed_logits.stride(0) if output_processed_logits is not None else 0,
        output_processed_logits_col,
        logits,
        logits.stride(0),
        expanded_idx_mapping,
        seed,
        pos,
        temperature,
        vocab_size,
        BLOCK_SIZE=BLOCK_SIZE,
        APPLY_TEMPERATURE=apply_temperature,
        USE_FP64=use_fp64,
    )
    # NOTE(woosuk): Use int64 for later indexing.
    max_block_idx = local_max.argmax(dim=-1, keepdim=True)
    sampled = local_argmax.gather(dim=-1, index=max_block_idx).view(-1)
    return sampled
```
**EN:** This function implements `gumbel_sample` within the module. Key calls include `cdiv`, `new_empty`, `argmax`, `view`, `stride`, `gather`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `gumbel_sample`，其作用域位于the module。 关键调用包括 `cdiv`, `new_empty`, `argmax`, `view`, `stride`, `gather`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_temperature_kernel`: top-level helper or orchestration entry point. / `_temperature_kernel`：顶层辅助函数或编排入口。
- `apply_temperature`: top-level helper or orchestration entry point. / `apply_temperature`：顶层辅助函数或编排入口。
- `tl_rand64`: top-level helper or orchestration entry point. / `tl_rand64`：顶层辅助函数或编排入口。
- `gumbel_block_argmax`: top-level helper or orchestration entry point. / `gumbel_block_argmax`：顶层辅助函数或编排入口。
- `_gumbel_sample_kernel`: top-level helper or orchestration entry point. / `_gumbel_sample_kernel`：顶层辅助函数或编排入口。
- `gumbel_sample`: top-level helper or orchestration entry point. / `gumbel_sample`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.triton_utils`
