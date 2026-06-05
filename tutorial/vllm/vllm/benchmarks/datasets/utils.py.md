# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/datasets/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Shared utilities for benchmark dataset sampling. / 该文件的核心目的为：Shared utilities for benchmark dataset sampling.

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-5)
```python
"""
Shared utilities for benchmark dataset sampling.
"""
```
**EN:** This opening docstring states the module's intent, expected usage, and any important assumptions before the implementation details begin.
**CN:** 开头的文档字符串先说明模块意图、预期用法以及实现展开前需要了解的重要假设。

### Imports (lines 7-12)
```python
import logging

import math

import numpy as np

from vllm.tokenizers import TokenizerLike
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 14-18)
```python
logger = logging.getLogger(__name__)

RangeRatio = float | dict[str, float]
```
**EN:** This constant/configuration block defines `logger`, `RangeRatio`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`, `RangeRatio`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Function `_resolve_range_ratios` (lines 21-38)
```python
def _resolve_range_ratios(
    range_ratio: RangeRatio,
) -> tuple[float, float]:
    """Return ``(input_range_ratio, output_range_ratio)`` from *range_ratio*.

    *range_ratio* is either a single float (used for both input and output)
    or a dict with ``"input"`` and ``"output"`` keys.
    """
    if isinstance(range_ratio, dict):
        try:
            return float(range_ratio["input"]), float(range_ratio["output"])
        except KeyError as exc:
            raise ValueError(
                "When range_ratio is a dict it must contain 'input' and "
                f"'output' keys, got: {sorted(range_ratio)}"
            ) from exc
    ratio = float(range_ratio)
    return ratio, ratio
```
**EN:** Function `_resolve_range_ratios` provides a reusable helper around the module's main workflow. The docstring highlights: Return ``(input_range_ratio, output_range_ratio)`` from *range_ratio*. Key calls such as `isinstance`, `float`, `ValueError`, `sorted` show the concrete execution path.
**CN:** Function `_resolve_range_ratios` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Return ``(input_range_ratio, output_range_ratio)`` from *range_ratio*. 像 `isinstance`, `float`, `ValueError`, `sorted` 这样的关键调用展示了该代码块的具体执行路径。

### Function `get_sampling_params` (lines 41-101)
```python
def get_sampling_params(
    rng: np.random.Generator,
    num_requests: int,
    range_ratio: RangeRatio,
    input_len: int,
    output_len: int,
    tokenizer: TokenizerLike,
) -> tuple[np.ndarray, np.ndarray, np.ndarray]:
    """
    Sample per-request input/output token lengths and vocab offsets.

    Lengths are drawn uniformly from integer ranges around the configured
    means, controlled by *range_ratio*.  It may be a single ``float``
    (applied to both input and output) or a ``dict`` with ``"input"`` and
    ``"output"`` keys for independent control.

    Tokenizer special tokens are subtracted from ``input_len`` before
    computing the sampling interval.

    Returns:
        (input_lens, output_lens, offsets) – three 1-D ``np.ndarray`` of
        shape ``(num_requests,)``.
    """
    input_range_ratio, output_range_ratio = _resolve_range_ratios(range_ratio)

    # ... omitted for brevity ...
    offsets = rng.integers(0, tokenizer.vocab_size, size=num_requests)
    return input_lens, output_lens, offsets
```
**EN:** Function `get_sampling_params` provides a reusable helper around the module's main workflow. The docstring highlights: Sample per-request input/output token lengths and vocab offsets. Key calls such as `_resolve_range_ratios`, `ValueError`, `int`, `tokenizer.num_special_tokens_to_add`, `max` show the concrete execution path.
**CN:** Function `get_sampling_params` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Sample per-request input/output token lengths and vocab offsets. 像 `_resolve_range_ratios`, `ValueError`, `int`, `tokenizer.num_special_tokens_to_add`, `max` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Benchmarking/data prep / 基准测试/数据准备**
  - **EN:** The file includes utilities for preparing benchmark inputs or measuring runtime behavior.
  - **CN:** 该文件包含用于准备基准测试输入或测量运行时行为的工具。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import logging`, `import math`
- **Third-party / 第三方**: `import numpy as np`
- **vLLM internal / vLLM 内部依赖**: `from vllm.tokenizers import TokenizerLike`
