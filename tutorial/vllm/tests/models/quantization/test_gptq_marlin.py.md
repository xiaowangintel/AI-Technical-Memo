# test_gptq_marlin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/quantization/test_gptq_marlin.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers quantized model support and model-facing behavior. The file defines 1 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖量化模型支持与面向模型的行为。它定义了 1 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L10-L18)
```python
import os

import pytest

from tests.quantization.utils import is_quant_method_supported
from vllm.model_executor.layers.rotary_embedding import _ROPE_DICT
from vllm.platforms import current_platform

from ..utils import check_logprobs_close
```
**EN:** Imports standard-library modules such as `os`, third-party packages like `pytest`, project helpers such as `tests.quantization.utils.is_quant_method_supported`, `vllm.model_executor.layers.rotary_embedding._ROPE_DICT`, `vllm.platforms.current_platform`.
**CN:** 导入标准库模块（如 `os`）、第三方包（如 `pytest`）、项目内辅助模块（如 `tests.quantization.utils.is_quant_method_supported`、`vllm.model_executor.layers.rotary_embedding._ROPE_DICT`、`vllm.platforms.current_platform`）。

### Module setup / 模块级配置: MAX_MODEL_LEN, MODELS (L20-L31)
```python
os.environ["TOKENIZERS_PARALLELISM"] = "true"

MAX_MODEL_LEN = 1024

MODELS = [
    # act_order==True, group_size=128
    ("TheBloke/TinyLlama-1.1B-Chat-v1.0-GPTQ", "main"),
    # 8-bit, act_order==True, group_size=channelwise
    ("TheBloke/TinyLlama-1.1B-Chat-v1.0-GPTQ", "gptq-8bit--1g-actorder_True"),
    # 4-bit, act_order==True, group_size=128
    ("TechxGenus/gemma-1.1-2b-it-GPTQ", "main"),
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MAX_MODEL_LEN`, `MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MAX_MODEL_LEN`、`MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_models (L34-L90)
```python
@pytest.mark.flaky(reruns=3)
@pytest.mark.skipif(
    not is_quant_method_supported("auto_gptq")
    or current_platform.is_rocm()
    or not current_platform.is_cuda(),
    reason="auto_gptq is not supported on this GPU type.",
)
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["half", "bfloat16"])
@pytest.mark.parametrize("max_tokens", [32])
@pytest.mark.parametrize("num_logprobs", [5])
def test_models(
    vllm_runner,
    example_prompts,
    model,
    dtype: str,
    max_tokens: int,
    num_logprobs: int,
# ... 31 lines omitted for brevity ...
        )

    check_logprobs_close(
        outputs_0_lst=gptq_outputs,
        outputs_1_lst=gptq_marlin_outputs,
        name_0="gptq",
        name_1="gptq_marlin",
    )
```
**EN:** This test validates `test_models`. It uses parameterization over `model`. Relevant pytest markers include `flaky`, `skipif`. Key inputs are `vllm_runner`, `example_prompts`, `model`, `dtype`, `max_tokens`, `num_logprobs`.
**CN:** 这个测试验证 `test_models`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `flaky`、`skipif`。 关键输入包括 `vllm_runner`、`example_prompts`、`model`、`dtype`、`max_tokens`、`num_logprobs`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `os`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `tests.quantization.utils.is_quant_method_supported`, `vllm.model_executor.layers.rotary_embedding._ROPE_DICT`, `vllm.platforms.current_platform`
- **Local relative imports / 本地相对导入**: `..utils.check_logprobs_close`
