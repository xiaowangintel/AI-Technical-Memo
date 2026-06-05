# test_ngram.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/spec_decode/test_ngram.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `ngram` behavior and regressions in the v1 stack. / 验证 v1 栈中 `ngram` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-13)
```python
import numpy as np

from vllm.config import (
    ModelConfig,
    SpeculativeConfig,
    VllmConfig,
)
from vllm.v1.spec_decode.ngram_proposer import (
    NgramProposer,
    _find_longest_matched_ngram_and_propose_tokens,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `numpy`. vLLM modules under test include `vllm.config, vllm.v1.spec_decode.ngram_proposer`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `numpy`。 被测试的 vLLM 模块包括 `vllm.config, vllm.v1.spec_decode.ngram_proposer`。

### test_find_longest_matched_ngram_and_propose_tokens (lines 16-62)
```python
def test_find_longest_matched_ngram_and_propose_tokens():
    tokens = np.array([1, 2, 3, 4, 1, 2, 3, 5, 6])
    result = _find_longest_matched_ngram_and_propose_tokens(
        origin_tokens=tokens, min_ngram=2, max_ngram=2, max_model_len=1024, k=2
    )
    assert len(result) == 0

    tokens = np.array([1, 2, 3, 4, 1, 2, 3])
    np.testing.assert_array_equal(
        _find_longest_matched_ngram_and_propose_tokens(
            origin_tokens=tokens, min_ngram=2, max_ngram=2, max_model_len=1024, k=3
        ),
        np.array([4, 1, 2]),
            origin_tokens=tokens, min_ngram=2, max_ngram=2, max_model_len=1024, k=2
    # ... excerpt omitted for brevity ...
            origin_tokens=tokens, min_ngram=1, max_ngram=1, max_model_len=1024, k=2
        np.array([6, 2]),
```
**EN:** Test case covering `find longest matched ngram and propose tokens`. It exercises `np.array, _find_longest_matched_ngram_and_propose_tokens, testing.assert_array_equal, len`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `find longest matched ngram and propose tokens` 的测试用例。 该测试会调用 `np.array, _find_longest_matched_ngram_and_propose_tokens, testing.assert_array_equal, len`。 代码主体包含 1 个显式断言。

### test_ngram_proposer (lines 65-204)
```python
def test_ngram_proposer():
    def get_ngram_proposer(min_n: int, max_n: int, k: int) -> NgramProposer:
        # Dummy model config. Just to set max_model_len.
        model_config = ModelConfig(model="facebook/opt-125m")
        return NgramProposer(
            vllm_config=VllmConfig(
                model_config=model_config,
                speculative_config=SpeculativeConfig(
                    prompt_lookup_min=min_n,
                    prompt_lookup_max=max_n,
                    num_speculative_tokens=k,
                    method="ngram",
                ),
            )
        )

    # No match.
    token_ids_cpu = np.array([[1, 2, 3, 4, 5]])
    # ... excerpt omitted for brevity ...
    assert len(result[0]) == 0
    assert np.array_equal(result, np.array([[4, 1]]))
    assert np.array_equal(result, np.array([[1, 2]]))  # Not [5, 1]]
    assert np.array_equal(result, np.array([[1, 2]]))  # Not [5, 2]]
    assert np.array_equal(result, np.array([[100, 1]]))
        num_tokens_no_spec=np.array([len(input_1), 3]),
        token_ids_cpu=token_ids_cpu,
    )
    assert len(result[0]) == 2
    assert np.array_equal(result[0], np.array([middle_integer + 2, middle_integer + 3]))
    assert np.array_equal(result[1], np.array([]))
```
**EN:** Test case covering `ngram proposer`. It exercises `np.array, get_ngram_proposer.propose, np.array_equal, get_ngram_proposer, np.zeros, proposer.propose`. The body contains 22 explicit assertion(s).
**CN:** 该代码块是覆盖 `ngram proposer` 的测试用例。 该测试会调用 `np.array, get_ngram_proposer.propose, np.array_equal, get_ngram_proposer, np.zeros, proposer.propose`。 代码主体包含 22 个显式断言。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Token-level boundary and decoding checks
- **CN:** token 级边界与解码检查
- **EN:** Speculative execution or decoding paths
- **CN:** 推测执行或解码路径

## Dependencies / 依赖关系
- **EN:** External libraries: `numpy`.
- **CN:** 外部库：`numpy`。
- **EN:** vLLM modules under test: `vllm.config, vllm.v1.spec_decode.ngram_proposer`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.v1.spec_decode.ngram_proposer`。
