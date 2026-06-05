# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides shared helper utilities used across model-executor layers. / 提供模型执行层共享的辅助工具。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-3)
```python
"""Utility methods for model layers."""
```
**EN:** This docstring gives the module author's high-level intent: Utility methods for model layers. It is useful for reading the rest of the file because later classes and helpers refine this stated purpose.
**CN:** 这个文档字符串给出了模块作者的高层意图：Utility methods for model layers. 在阅读后续类和辅助函数时，可以把它视为整个文件的总纲。

### Imports and module setup (lines 5-15)
```python
from collections.abc import Callable

import torch

from vllm import _custom_ops as ops
from vllm import envs
from vllm._aiter_ops import rocm_aiter_ops
from vllm.logger import init_logger
from vllm.platforms import CpuArchEnum, current_platform
from vllm.utils.platform_utils import num_compute_units
from vllm.utils.torch_utils import direct_register_custom_op
```
**EN:** This opening block pulls in external dependencies such as `collections`, `torch` and internal modules such as `vllm`, `vllm._aiter_ops`, `vllm.logger`, `vllm.platforms`, `vllm.utils.platform_utils`, `vllm.utils.torch_utils`. That import mix shows the file is part of the model-executor layers and helpers stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `collections`, `torch`）以及内部模块（如 `vllm`, `vllm._aiter_ops`, `vllm.logger`, `vllm.platforms`, `vllm.utils.platform_utils`, `vllm.utils.torch_utils`）。这些导入关系表明该文件属于模型执行层与辅助组件栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 17-25)
```python
logger = init_logger(__name__)

MOE_LAYER_ROUTER_GATE_SUFFIXES = {
    "gate",
    "router",
    "router_gate",
    "shared_expert_gate",
    "expert_gate",
}
```
**EN:** This block defines module-level metadata or constants such as `logger`, `MOE_LAYER_ROUTER_GATE_SUFFIXES`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the model-executor layers and helpers pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`, `MOE_LAYER_ROUTER_GATE_SUFFIXES`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在模型执行层与辅助组件流程中复用。

### Function `get_token_bin_counts_and_mask` (lines 34-48)
```python
def get_token_bin_counts_and_mask(
    tokens: torch.Tensor,
    vocab_size: int,
    num_seqs: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    # Compute the bin counts for the tokens.
    # vocab_size + 1 for padding.
    bin_counts = torch.zeros(
        (num_seqs, vocab_size + 1), dtype=torch.long, device=tokens.device
    )
    bin_counts.scatter_add_(1, tokens, torch.ones_like(tokens))
    bin_counts = bin_counts[:, :vocab_size]
    mask = bin_counts > 0

    return bin_counts, mask
```
**EN:** Defines function `get_token_bin_counts_and_mask` with signature `get_token_bin_counts_and_mask(tokens: torch.Tensor, vocab_size: int, num_seqs: int) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `tokens`, `vocab_size`, `num_seqs`; returns a derived property or capability check. The body uses tensor/kernel operations. Key calls include `torch.zeros`, `bin_counts.scatter_add_`, `torch.ones_like`.
**CN:** 定义函数 `get_token_bin_counts_and_mask`，其签名为 `get_token_bin_counts_and_mask(tokens: torch.Tensor, vocab_size: int, num_seqs: int) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `tokens`, `vocab_size`, `num_seqs` 展开；返回派生属性或能力判断结果。函数体包含张量或内核操作。关键调用包括 `torch.zeros`, `bin_counts.scatter_add_`, `torch.ones_like`。

### Function `apply_penalties` (lines 51-89)
```python
def apply_penalties(
    logits: torch.Tensor,
    prompt_tokens_tensor: torch.Tensor,
    output_tokens_tensor: torch.Tensor,
    presence_penalties: torch.Tensor,
    frequency_penalties: torch.Tensor,
    repetition_penalties: torch.Tensor,
) -> torch.Tensor:
    """
    Applies penalties in place to the logits tensor
    logits : The input logits tensor of shape [num_seqs, vocab_size]
    prompt_tokens_tensor: A tensor containing the prompt tokens. The prompts
        are padded to the maximum prompt length within the batch using
        `vocab_size` as the padding value. The value `vocab_size` is used
        for padding because it does not correspond to any valid token ID
        in the vocabulary.
    output_tokens_tensor: The output tokens tensor.
    presence_penalties: The presence penalties of shape (num_seqs, )
    frequency_penalties: The frequency penalties of shape (num_seqs, )
    repetition_penalties: The repetition penalties of shape (num_seqs, )
    """
    num_seqs, vocab_size = logits.shape
    _, prompt_mask = get_token_bin_counts_and_mask(
        prompt_tokens_tensor, vocab_size, num_seqs
    )
    output_bin_counts, output_mask = get_token_bin_counts_and_mask(
        output_tokens_tensor, vocab_size, num_seqs
    )

    # Apply repetition penalties as a custom op
    from vllm._custom_ops import apply_repetition_penalties

    apply_repetition_penalties(logits, prompt_mask, output_mask, repetition_penalties)

    # We follow the definition in OpenAI API.
    # Refer to https://platform.openai.com/docs/api-reference/parameter-details
    logits -= frequency_penalties.unsqueeze(dim=1) * output_bin_counts
    logits -= presence_penalties.unsqueeze(dim=1) * output_mask
    return logits
```
**EN:** Defines function `apply_penalties` with signature `apply_penalties(logits: torch.Tensor, prompt_tokens_tensor: torch.Tensor, output_tokens_tensor: torch.Tensor, presence_penalties: torch.Tensor, frequency_penalties: torch.Tensor, repetition_penalties: torch.Tensor) -> torch.Tensor`. It mainly works with `logits`, `prompt_tokens_tensor`, `output_tokens_tensor`, `presence_penalties`, `frequency_penalties`, `repetition_penalties`; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include `get_token_bin_counts_and_mask`, `apply_repetition_penalties`, `frequency_penalties.unsqueeze`, `presence_penalties.unsqueeze`.
**CN:** 定义函数 `apply_penalties`，其签名为 `apply_penalties(logits: torch.Tensor, prompt_tokens_tensor: torch.Tensor, output_tokens_tensor: torch.Tensor, presence_penalties: torch.Tensor, frequency_penalties: torch.Tensor, repetition_penalties: torch.Tensor) -> torch.Tensor`。它主要围绕 `logits`, `prompt_tokens_tensor`, `output_tokens_tensor`, `presence_penalties`, `frequency_penalties`, `repetition_penalties` 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `get_token_bin_counts_and_mask`, `apply_repetition_penalties`, `frequency_penalties.unsqueeze`, `presence_penalties.unsqueeze`。

### Function `default_unquantized_gemm` (lines 92-98)
```python
def default_unquantized_gemm(
    layer: torch.nn.Module,
    x: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor | None = None,
):
    return torch.nn.functional.linear(x, weight, bias)
```
**EN:** Defines function `default_unquantized_gemm` with signature `default_unquantized_gemm(layer: torch.nn.Module, x: torch.Tensor, weight: torch.Tensor, bias: torch.Tensor | None=None)`. It mainly works with `layer`, `x`, `weight`, `bias`; handles quantization-related transformation logic. The body uses tensor/kernel operations. Key calls include `torch.nn.functional.linear`.
**CN:** 定义函数 `default_unquantized_gemm`，其签名为 `default_unquantized_gemm(layer: torch.nn.Module, x: torch.Tensor, weight: torch.Tensor, bias: torch.Tensor | None=None)`。它主要围绕 `layer`, `x`, `weight`, `bias` 展开；处理量化相关的变换逻辑。函数体包含张量或内核操作。关键调用包括 `torch.nn.functional.linear`。

### Function `use_aiter_triton_gemm` (lines 101-119)
```python
def use_aiter_triton_gemm(n, m, k, dtype):
    if (
        not rocm_aiter_ops.is_triton_gemm_enabled()
        # MI300's - fp8nuz=True
        or current_platform.is_fp8_fnuz()
        or dtype not in [torch.float16, torch.bfloat16]
    ):
        return False

    # use hipblaslt for the larger GEMMs
    if n > 2048 and m > 512:
        return False
    return (
        (m == 5120 and k == 2880)
        or (m == 2880 and k == 4096)
        or (m == 128 and k == 2880)
        or (m == 640 and k == 2880)
        or (m == 2880 and k == 512)
    )
```
**EN:** Defines function `use_aiter_triton_gemm` with signature `use_aiter_triton_gemm(n, m, k, dtype)`. It mainly works with `n`, `m`, `k`, `dtype`; implements one step of the module control flow. The body uses branching. Key calls include `current_platform.is_fp8_fnuz`, `rocm_aiter_ops.is_triton_gemm_enabled`.
**CN:** 定义函数 `use_aiter_triton_gemm`，其签名为 `use_aiter_triton_gemm(n, m, k, dtype)`。它主要围绕 `n`, `m`, `k`, `dtype` 展开；实现模块控制流中的一个步骤。函数体包含分支判断。关键调用包括 `current_platform.is_fp8_fnuz`, `rocm_aiter_ops.is_triton_gemm_enabled`。

### Function `rocm_unquantized_gemm_impl` (lines 122-193)
```python
def rocm_unquantized_gemm_impl(
    x: torch.Tensor, weight: torch.Tensor, bias: torch.Tensor | None = None
) -> torch.Tensor:
    from vllm.platforms.rocm import on_gfx1x, on_gfx9, on_gfx950

    n = x.numel() // x.size(-1)
    m = weight.shape[0]
    k = weight.shape[1]

    cu_count = num_compute_units()

    # Next ^2 of n
    N_p2 = 1 << (n - 1).bit_length()
    # With 64 Ms per CU (each of 4 SIMDs working on a 16x16 tile),
    # and each working on a 512-shard of K, how many CUs would we need?
    rndup_cus = ((m + 64 - 1) // 64) * ((k + 512 - 1) // 512)
    # How many of 4 waves in a group can work on same 16 Ms at same time?
    # This reduces the Ms each group works on, i.e. increasing the number of CUs needed.
    GrpsShrB = min(N_p2 // 16, 4)
    # Given the above, how many CUs would we need?
    CuNeeded = rndup_cus * GrpsShrB
    # candidate for atomic reduce count splitk?
    fits_wvsplitkrc = (
        N_p2 * m * ((k + 512 - 1) // 512)
    ) <= 128 * 1024 * 12  # deterministic
    fits_wvsplitkrc &= CuNeeded <= cu_count

    use_skinny_reduce_counting = (
        envs.VLLM_ROCM_USE_SKINNY_GEMM
        and on_gfx950()
        and x.dtype in [torch.float16, torch.bfloat16]
        and (
            10 <= n <= 128
            and k % 8 == 0
            and k > 512
            and m % 16 == 0
            and fits_wvsplitkrc
            and weight.is_contiguous()
# ... truncated for analysis ...
        if m > 8 and 0 < n <= 4:
            cu_count = num_compute_units()
            out = ops.wvSplitK(weight, x_view, cu_count, bias)
            return out.reshape(*x.shape[:-1], weight.shape[0])
        elif m % 4 == 0 and n == 1 and k <= 8192 and bias is None:
            out = ops.LLMM1(weight, x_view, 4)
            return out.reshape(*x.shape[:-1], weight.shape[0])

    if rocm_aiter_ops.is_tgemm_enabled():
        from aiter.tuned_gemm import tgemm

        return tgemm.mm(x, weight, bias)

    return torch.nn.functional.linear(x, weight, bias)
```
**EN:** Defines function `rocm_unquantized_gemm_impl` with signature `rocm_unquantized_gemm_impl(x: torch.Tensor, weight: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `x`, `weight`, `bias`; handles quantization-related transformation logic. The body uses branching, tensor/kernel operations. Key calls include `num_compute_units`, `min`, `use_aiter_triton_gemm`, `rocm_aiter_ops.is_tgemm_enabled`, `torch.nn.functional.linear`, `x.numel`.
**CN:** 定义函数 `rocm_unquantized_gemm_impl`，其签名为 `rocm_unquantized_gemm_impl(x: torch.Tensor, weight: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `x`, `weight`, `bias` 展开；处理量化相关的变换逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `num_compute_units`, `min`, `use_aiter_triton_gemm`, `rocm_aiter_ops.is_tgemm_enabled`, `torch.nn.functional.linear`, `x.numel`。

### Function `rocm_unquantized_gemm` (lines 202-208)
```python
def rocm_unquantized_gemm(
    layer: torch.nn.Module,
    x: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor | None = None,
) -> torch.Tensor:
    return torch.ops.vllm.rocm_unquantized_gemm(x, weight, bias)
```
**EN:** Defines function `rocm_unquantized_gemm` with signature `rocm_unquantized_gemm(layer: torch.nn.Module, x: torch.Tensor, weight: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `weight`, `bias`; handles quantization-related transformation logic. The body uses tensor/kernel operations. Key calls include `torch.ops.vllm.rocm_unquantized_gemm`.
**CN:** 定义函数 `rocm_unquantized_gemm`，其签名为 `rocm_unquantized_gemm(layer: torch.nn.Module, x: torch.Tensor, weight: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `weight`, `bias` 展开；处理量化相关的变换逻辑。函数体包含张量或内核操作。关键调用包括 `torch.ops.vllm.rocm_unquantized_gemm`。

### Function `dispatch_cpu_unquantized_gemm` (lines 227-308)
```python
def dispatch_cpu_unquantized_gemm(
    layer: torch.nn.Module,
    remove_weight: bool,
) -> None:
    # skip for missing layers
    if layer.weight.is_meta:
        layer.cpu_linear = torch.nn.functional.linear
        return

    if layer.weight.ndim != 2:
        # this is not a linear layer
        # For now it should be a causal_conv1d op
        if torch.cpu._is_amx_tile_supported():
            # prepack conv weight
            layer.weight.data = ops.causal_conv1d_weight_pack(
                layer.weight.view(
                    layer.weight.size(0),
                    layer.weight.size(2),
                )
            )
        return

    N, K = layer.weight.size()
    dtype = layer.weight.dtype

    # Zen CPU path: zentorch_linear_unary with optional eager weight prepacking.
    if current_platform.is_zen_cpu() and hasattr(
        torch.ops.zentorch, "zentorch_linear_unary"
    ):
        zen_weight = layer.weight.detach()
        is_prepacked = False

        if envs.VLLM_ZENTORCH_WEIGHT_PREPACK and hasattr(
            torch.ops.zentorch, "zentorch_weight_prepack_for_linear"
        ):
            zen_weight = torch.ops.zentorch.zentorch_weight_prepack_for_linear(
                zen_weight
            )
# ... truncated for analysis ...
            layer.cpu_linear = lambda x, weight, bias: ops.onednn_mm(handler, x, bias)
            if remove_weight:
                layer.weight = torch.nn.Parameter(torch.empty(0), requires_grad=False)
            return
        except RuntimeError as e:
            logger.warning_once(
                "Failed to create oneDNN linear, fallback to torch linear."
                f" Exception: {e}"
            )

    # fallback case
    layer.cpu_linear = lambda x, weight, bias: torch.nn.functional.linear(
        x, weight, bias
    )
```
**EN:** Defines function `dispatch_cpu_unquantized_gemm` with signature `dispatch_cpu_unquantized_gemm(layer: torch.nn.Module, remove_weight: bool) -> None`. It mainly works with `layer`, `remove_weight`; handles quantization-related transformation logic. The body uses branching, tensor/kernel operations. Key calls include `layer.weight.size`, `torch.cpu._is_amx_tile_supported`, `current_platform.is_zen_cpu`, `hasattr`, `layer.weight.detach`, `check_cpu_sgl_kernel`.
**CN:** 定义函数 `dispatch_cpu_unquantized_gemm`，其签名为 `dispatch_cpu_unquantized_gemm(layer: torch.nn.Module, remove_weight: bool) -> None`。它主要围绕 `layer`, `remove_weight` 展开；处理量化相关的变换逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `layer.weight.size`, `torch.cpu._is_amx_tile_supported`, `current_platform.is_zen_cpu`, `hasattr`, `layer.weight.detach`, `check_cpu_sgl_kernel`。

## Key Concepts / 关键概念
- **EN:** The file provides a reusable layer/helper inside the model-executor subsystem.
  **CN:** 该文件在模型执行子系统中提供可复用的层或辅助逻辑。
- **EN:** Top-level helpers such as `is_layer_moe_router_gate`, `get_token_bin_counts_and_mask`, `apply_penalties`, `default_unquantized_gemm`, `use_aiter_triton_gemm` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `is_layer_moe_router_gate`, `get_token_bin_counts_and_mask`, `apply_penalties`, `default_unquantized_gemm`, `use_aiter_triton_gemm` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `collections`, `torch`
- **Internal / 内部**: `vllm`, `vllm._aiter_ops`, `vllm.logger`, `vllm.platforms`, `vllm.utils.platform_utils`, `vllm.utils.torch_utils`
