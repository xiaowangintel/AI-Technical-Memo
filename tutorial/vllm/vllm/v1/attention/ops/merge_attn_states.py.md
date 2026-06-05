# merge_attn_states.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/ops/merge_attn_states.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `merge_attn_states` for the V1 `attention/ops` subsystem. / 为 V1 的 `attention/ops` 子系统实现 `merge_attn_states`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import torch

from vllm.platforms import current_platform
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.platforms`, `vllm._custom_ops`, `vllm.v1.attention.ops.triton_merge_attn_states`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.platforms`, `vllm._custom_ops`, `vllm.v1.attention.ops.triton_merge_attn_states` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

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
    """Merge partial attention outputs from prefix (KV cache) and suffix
    (new tokens) into a single output tensor using the log-sum-exp (LSE)
    rescaling method described in section 2.2 of
    https://www.arxiv.org/pdf/2501.01005.

    For tokens that have prefix context (token index < prefill_tokens_with_context),
    the prefix and suffix partial outputs are combined as a weighted sum.
    For tokens without prefix context, the suffix output is copied directly.

    Args:
        output: Output tensor of shape [NUM_TOKENS, NUM_HEADS, HEAD_SIZE].
        prefix_output: Partial attention output over the prefix (KV cache),
            shape [NUM_TOKENS, NUM_HEADS, HEAD_SIZE].
        prefix_lse: Log-sum-exp values for the prefix attention,
            shape [NUM_HEADS, NUM_TOKENS].
        suffix_output: Partial attention output over the suffix (new KV),
            shape [NUM_TOKENS, NUM_HEADS, HEAD_SIZE].
        suffix_lse: Log-sum-exp values for the suffix attention,
            shape [NUM_HEADS, NUM_TOKENS].
        output_lse: Optional tensor to store the merged LSE values,
            shape [NUM_HEADS, NUM_TOKENS]. If None, LSE is not written out.
        prefill_tokens_with_context: Number of prefill tokens that have
            prefix context and therefore require merging. Tokens at indices
            >= this value are decode or context-free prefill tokens whose
            output is taken directly from suffix_output. If None, all tokens
            are treated as having context.
        output_scale: Optional scalar tensor for FP8 static quantization.
            When provided, output must be FP8 dtype.
    """

    # NOTE(DefTruth): Currently, custom merge_attn_states CUDA kernel
    # does not support FP8 dtype for inputs, fallback to use Triton kernel.
    # However, when output_scale is provided, the inputs are still BF16/FP16
    # and the output is FP8 — both CUDA and Triton support this.
    # FP8 output requires output_scale to be set.
    if output.dtype not in (torch.float32, torch.half, torch.bfloat16):
        assert output_scale is not None, (
            f"output_scale is required when output is {output.dtype}"
        )

# ... omitted for brevity ...
        return merge_attn_states(
            output,
            prefix_output,
            prefix_lse,
            suffix_output,
            suffix_lse,
            output_lse,
            prefill_tokens_with_context,
            output_scale,
        )
    else:
        from vllm.v1.attention.ops.triton_merge_attn_states import (
            merge_attn_states,
        )

        return merge_attn_states(
            output,
            prefix_output,
            prefix_lse,
            suffix_output,
            suffix_lse,
            output_lse,
            prefill_tokens_with_context,
            output_scale,
        )
```
**EN:** This function implements `merge_attn_states` within the module. The docstring frames it as: Merge partial attention outputs from prefix (KV cache) and suffix (new tokens) into a single output tensor using the log-sum-exp (LSE) rescaling method described in section 2.2 of https://www.arxiv.org/pdf/2501.01005. Key calls include `is_cuda`, `supported_dtypes`, `supported_headdim`, `merge_attn_states`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `merge_attn_states`，其作用域位于the module。 关键调用包括 `is_cuda`, `supported_dtypes`, `supported_headdim`, `merge_attn_states`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

## Key Concepts / 关键概念
- `merge_attn_states`: top-level helper or orchestration entry point. / `merge_attn_states`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.platforms`, `vllm._custom_ops`, `vllm.v1.attention.ops.triton_merge_attn_states`
