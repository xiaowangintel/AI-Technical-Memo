# test_mrope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/core/test_mrope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / core / test_mrope, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / core / test_mrope 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-11)
```python
from typing import NamedTuple

import pytest
import torch

from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.platforms import current_platform
from vllm.transformers_utils.config import get_config
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as typing, pytest, torch; and vLLM components like vllm.model_executor.layers.rotary_embedding, vllm.platforms, vllm.transformers_utils.config, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 typing、pytest、torch；vLLM 内部组件，例如 vllm.model_executor.layers.rotary_embedding、vllm.platforms、vllm.transformers_utils.config、vllm.utils.torch_utils。

### Constants and module state (lines 13-13)
```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Function `generate_test_data` (lines 16-36)
```python
def generate_test_data(
    num_tokens: int,
    num_q_heads: int,
    num_kv_heads: int,
    head_size: int,
    max_position_embeddings: int,
    dtype: torch.dtype,
    device: torch.device,
):
    """Generate test data for given configuration."""
    set_random_seed(42)
    # Create 2D positions (3, num_tokens) for multimodal case
    positions = torch.randint(
        0, max_position_embeddings // 4, (3, num_tokens), device=device
    )

    # Create query and key tensors
    query = torch.randn(num_tokens, num_q_heads * head_size, dtype=dtype, device=device)
    key = torch.randn(num_tokens, num_kv_heads * head_size, dtype=dtype, device=device)

    return positions, query, key
```
**EN:** This helper function implements the shared logic for generate test data. randomness is controlled so the scenario remains reproducible.
**CN:** 该辅助函数实现了 generate test data 所需的共享逻辑。 代码会控制随机性以保证场景可复现。

### Class `MRoPETestInfo` (lines 39-44)
```python
class MRoPETestInfo(NamedTuple):
    model_name: str
    # https://github.com/pytorch/pytorch/blob/main/torch/testing/_comparison.py#L1317
    atol: float = 1e-2
    rtol: float = 1.6e-2
    marks: list[pytest.MarkDecorator] = []
```
**EN:** This class defines a container for MRoPETestInfo.
**CN:** 该类定义了 MRoPETestInfo 对应的容器。

### Constants and module state (lines 47-56)
```python
MODELS_TO_TEST = [
    MRoPETestInfo(model_name="zai-org/GLM-4.1V-9B-Thinking"),
    MRoPETestInfo(model_name="Qwen/Qwen2-VL-7B-Instruct"),
    MRoPETestInfo(model_name="Qwen/Qwen2-VL-72B-Instruct"),
    MRoPETestInfo(model_name="Qwen/Qwen2.5-VL-72B-Instruct"),
    MRoPETestInfo(model_name="Qwen/Qwen3-VL-4B-Instruct"),
    MRoPETestInfo(model_name="Qwen/Qwen3-VL-30B-A3B-Instruct"),
]

num_tokens_list = [11, 8192]
```
**EN:** This block centralizes shared constants and parameter grids, including MODELS_TO_TEST. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 MODELS_TO_TEST。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_mrope` (lines 59-126)
```python
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(), reason="Skipping CUDA/ROCm only tests."
)
@pytest.mark.parametrize(
    "model_info, model_name",
    [
        pytest.param(test_config, test_config.model_name, marks=test_config.marks)
        for test_config in MODELS_TO_TEST
    ],
)
@pytest.mark.parametrize("tp_size", [1, 2])
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("num_tokens", num_tokens_list)
def test_mrope(
    default_vllm_config,
    model_name: str,
    model_info: MRoPETestInfo,
    tp_size: int,
    dtype: torch.dtype,
    num_tokens: int,
):
    atol = model_info.atol
    rtol = model_info.rtol

    config = get_config(model_name, False).get_text_config()

    # get the model config
    total_num_kv_heads = config.num_key_value_heads
    total_num_heads = config.num_attention_heads
    num_heads = total_num_heads // tp_size
    num_kv_heads = max(1, total_num_kv_heads // tp_size)
    head_dim = (
        config.head_dim
        if hasattr(config, "head_dim")
        else config.hidden_size // total_num_heads
    )
    is_neox_style = True

    max_position = config.max_position_embeddings

    mrope_helper_class = get_rope(
        head_size=head_dim,
        max_position=max_position,
        is_neox_style=is_neox_style,
        rope_parameters=config.rope_parameters,
        dtype=dtype,
    ).to(device=device)

    # create q k v input tensors
    # create rotary pos emb input tensors
    positions, query, key = generate_test_data(
        num_tokens, num_heads, num_kv_heads, head_dim, max_position, dtype, device
    )

    query_native, key_native = mrope_helper_class.forward_native(
        positions,
        query.clone(),
        key.clone(),
    )

    query_cuda, key_cuda = mrope_helper_class.forward_cuda(
        positions,
        query.clone(),
        key.clone(),
    )

    torch.testing.assert_close(query_native, query_cuda, atol=atol, rtol=rtol)
    torch.testing.assert_close(key_native, key_cuda, atol=atol, rtol=rtol)
```
**EN:** This pytest case verifies mrope. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as default_vllm_config, model_name, model_info, tp_size. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 mrope 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 default_vllm_config、model_name、model_info、tp_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_mrope_torch_compile_tracing` (lines 129-236)
```python
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(), reason="Skipping CUDA/ROCm only tests."
)
@pytest.mark.parametrize(
    "model_info, model_name",
    [
        pytest.param(test_config, test_config.model_name, marks=test_config.marks)
        for test_config in MODELS_TO_TEST
    ],
)
@pytest.mark.parametrize("tp_size", [1, 2])
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("num_tokens", num_tokens_list)
def test_mrope_torch_compile_tracing(
    default_vllm_config,
    model_name: str,
    model_info: MRoPETestInfo,
    tp_size: int,
    dtype: torch.dtype,
    num_tokens: int,
):
    atol = model_info.atol
    rtol = model_info.rtol

    config = get_config(model_name, False).get_text_config()

    # get the model config
    total_num_kv_heads = config.num_key_value_heads
    total_num_heads = config.num_attention_heads
    num_heads = total_num_heads // tp_size
    num_kv_heads = max(1, total_num_kv_heads // tp_size)
    head_dim = (
        config.head_dim
        if hasattr(config, "head_dim")
        else config.hidden_size // total_num_heads
    )
    is_neox_style = True
    max_position = config.max_position_embeddings

    mrope_helper_class = get_rope(
# ... excerpt ...
        query.clone(),
        key.clone(),
    )

    try:
        compiled_forward_cuda = torch.compile(
            functional_forward_cuda,
            fullgraph=True,
            backend="inductor",
            mode="reduce-overhead",
            dynamic=False,
        )

        # Run compiled version
        query_compiled_cuda, key_compiled_cuda = compiled_forward_cuda(
            positions,
            query,
            key,
        )

        # Run original version for comparison
        query_cuda = query.clone()
        key_cuda = key.clone()
        mrope_helper_class.forward_cuda(positions, query_cuda, key_cuda)

        # Verify results
        torch.testing.assert_close(
            query_compiled_cuda, query_cuda, atol=atol, rtol=rtol
        )
        torch.testing.assert_close(key_compiled_cuda, key_cuda, atol=atol, rtol=rtol)
        torch.testing.assert_close(
            query_compiled_cuda, query_native, atol=atol, rtol=rtol
        )
        torch.testing.assert_close(key_compiled_cuda, key_native, atol=atol, rtol=rtol)

        print("✓ forward_cuda successfully traced with torch.compile inductor")

    except Exception as e:
        pytest.fail(f"forward_cuda failed to trace with torch.compile inductor: {e}")
```
**EN:** This pytest case verifies mrope torch compile tracing. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as default_vllm_config, model_name, model_info, tp_size. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 mrope torch compile tracing 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 default_vllm_config、model_name、model_info、tp_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `typing -> NamedTuple`
- `pytest`
- `torch`
- `vllm.model_executor.layers.rotary_embedding -> get_rope`
- `vllm.platforms -> current_platform`
- `vllm.transformers_utils.config -> get_config`
- `vllm.utils.torch_utils -> set_random_seed`
