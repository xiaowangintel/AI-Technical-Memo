# triton_merge_attn_states.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/ops/triton_merge_attn_states.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `merge_attn_states`, `merge_attn_states_kernel` for the V1 `attention/ops` subsystem. / 为 V1 的 `attention/ops` 子系统实现 `merge_attn_states`, `merge_attn_states_kernel`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import torch

from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton

float8_info = torch.finfo(current_platform.fp8_dtype())
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `float8_info`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `float8_info`。

### `merge_attn_states` function / `merge_attn_states` 函数
```python
def merge_attn_states(
    output: torch.Tensor,
    prefix_output: torch.Tensor,
    prefix_lse: torch.Tensor,
    suffix_output: torch.Tensor,
    suffix_lse: torch.Tensor,
    output_lse: torch.Tensor | None = None,
    prefill_tokens_with_context: int | None = None,
    output_scale: torch.Tensor | None = None,
) -> None:
    num_tokens = output.shape[0]
    num_query_heads = output.shape[1]
    head_size = output.shape[2]
    padded_head_size = triton.next_power_of_2(head_size)
    # We assume the output stride on num_head is not always as same as the
    # `suffix_output` and `prefix_output`, as them might be padded by the
    # attention backend.
    prefix_head_stride = prefix_output.stride(1)
    output_head_stride = output.stride(1)

    # If prefill_tokens_with_context is None, all tokens should use prefix context
    if prefill_tokens_with_context is None:
        prefill_tokens_with_context = num_tokens

    # TODO(woosuk): Use CUDA kernel instead of Triton to minimize CPU overhead.
    merge_attn_states_kernel[(num_tokens, num_query_heads)](
        output,
        output_lse,
        prefix_output,
        prefix_lse,
        suffix_output,
        suffix_lse,
        prefix_head_stride,
        output_head_stride,
        output_scale,
        head_size,
        padded_head_size,
        output_lse is not None,
        prefill_tokens_with_context,
        output_scale is not None,
    )
```
**EN:** This function implements `merge_attn_states` within the module. Key calls include `next_power_of_2`, `stride`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `merge_attn_states`，其作用域位于the module。 关键调用包括 `next_power_of_2`, `stride`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `merge_attn_states_kernel` function / `merge_attn_states_kernel` 函数
```python
@triton.jit
def merge_attn_states_kernel(
    output,  # [NUM_TOKENS, NUM_HEADS, HEAD_SIZE]
    output_lse,  # [NUM_HEADS, NUM_TOKENS]
    prefix_output,  # [NUM_TOKENS, NUM_HEADS, HEAD_SIZE]
    prefix_lse,  # [NUM_HEADS, NUM_TOKENS]
    suffix_output,  # [NUM_TOKENS, NUM_HEADS, HEAD_SIZE]
    suffix_lse,  # [NUM_HEADS, NUM_TOKENS]
    prefix_head_stride,
    output_head_stride,
    output_scale,  # scale tensor or None
    HEAD_SIZE: tl.constexpr,
    PADDED_HEAD_SIZE: tl.constexpr,
    OUTPUT_LSE: tl.constexpr,
    prefill_tokens_with_context: tl.constexpr,
    USE_FP8: tl.constexpr,
    FP8_MIN: tl.constexpr = float8_info.min,
    FP8_MAX: tl.constexpr = float8_info.max,
):
    token_idx = tl.program_id(0)
    num_tokens = tl.num_programs(0)
    head_idx = tl.program_id(1)
    num_heads = tl.num_programs(1)

    prefix_mask = token_idx < prefill_tokens_with_context

    head_arange = tl.arange(0, PADDED_HEAD_SIZE)
    head_mask = head_arange < HEAD_SIZE

    # For tokens without context (token_idx >= prefill_tokens_with_context),
    # directly copy from suffix_output
    if not prefix_mask:
        s_lse = tl.load(suffix_lse + head_idx * num_tokens + token_idx)
        if OUTPUT_LSE:
            tl.store(output_lse + head_idx * num_tokens + token_idx, s_lse)

        s_out = tl.load(
            suffix_output
            + token_idx * num_heads * prefix_head_stride
            + head_idx * prefix_head_stride
            + head_arange,
            mask=head_mask,
        )

        if USE_FP8:
            s_out = s_out * (1.0 / tl.load(output_scale))
            s_out = tl.clamp(s_out, FP8_MIN, FP8_MAX)
            s_out = s_out.to(output.dtype.element_ty)

        tl.store(
# ... omitted for brevity ...
        + head_idx * prefix_head_stride
        + head_arange,
        mask=head_mask,
    )

    # NOTE(woosuk): Be careful with the numerical stability.
    # We should compute the scale first, and then multiply it with the output.
    # Do not multiply the output with tl.exp(p_lse) or tl.exp(s_lse) directly.
    p_scale = p_se / out_se
    s_scale = s_se / out_se
    out = p_out * p_scale + s_out * s_scale

    if USE_FP8:
        out = out * (1.0 / tl.load(output_scale))
        out = tl.clamp(out, FP8_MIN, FP8_MAX)
        out = out.to(output.dtype.element_ty)

    tl.store(
        output
        + token_idx * num_heads * output_head_stride
        + head_idx * output_head_stride
        + head_arange,
        out,
        mask=head_mask,
    )
```
**EN:** This function implements `merge_attn_states_kernel` within the module. Key calls include `program_id`, `num_programs`, `arange`, `load`, `maximum`, `exp`. The control flow contains 7 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `merge_attn_states_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `num_programs`, `arange`, `load`, `maximum`, `exp`。 控制流包含 7 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

## Key Concepts / 关键概念
- `merge_attn_states`: top-level helper or orchestration entry point. / `merge_attn_states`：顶层辅助函数或编排入口。
- `merge_attn_states_kernel`: top-level helper or orchestration entry point. / `merge_attn_states_kernel`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.platforms`, `vllm.triton_utils`
