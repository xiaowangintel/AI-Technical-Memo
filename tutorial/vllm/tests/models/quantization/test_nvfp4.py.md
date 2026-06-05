# test_nvfp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/quantization/test_nvfp4.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers quantized model support and model-facing behavior. The file defines 3 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖量化模型支持与面向模型的行为。它定义了 3 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L8-L17)
```python
import os
from typing import List

import pytest
from transformers import AutoTokenizer

from tests.quantization.utils import is_quant_method_supported
from vllm import LLM, SamplingParams

from vllm.platforms import current_platform
```
**EN:** Imports standard-library modules such as `os`, `typing.List`, third-party packages like `pytest`, `transformers.AutoTokenizer`, project helpers such as `tests.quantization.utils.is_quant_method_supported`, `vllm.LLM`, `vllm.SamplingParams`.
**CN:** 导入标准库模块（如 `os`、`typing.List`）、第三方包（如 `pytest`、`transformers.AutoTokenizer`）、项目内辅助模块（如 `tests.quantization.utils.is_quant_method_supported`、`vllm.LLM`、`vllm.SamplingParams`）。

### Module setup / 模块级配置: MAX_MODEL_LEN, MODELS, EXPECTED_STRS_MAP (L19-L36)
```python
os.environ["TOKENIZERS_PARALLELISM"] = "true"

MAX_MODEL_LEN = 1024

MODELS = ["nvidia/Llama-3.3-70B-Instruct-FP4"]

EXPECTED_STRS_MAP = {
    "nvidia/Llama-3.3-70B-Instruct-FP4": [
        "vLLM (Vectorized Large Language Model) is indeed a high-throughput and memory-efficient inference",
        "Here are the major milestones in the development of artificial intelligence (AI) from 1950 to ",
        "Artificial intelligence (AI) and human intelligence (HI) are two distinct forms of intelligence that process",
        "A neural network is a type of machine learning model inspired by the structure and function of the human brain",
        "In the heart of a cutting-edge robotics lab, a team of engineers had been working tirelessly to push",
        "The COVID-19 pandemic has had a profound impact on global economic structures and future business models, leading",
        "The Mona Lisa, painted by Leonardo da Vinci in the early 16th century, is one of",
        "Here are the translations:\n\n* Japanese: (Sasuga no tori ga miwa o ts",
    ]
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MAX_MODEL_LEN`, `MODELS`, `EXPECTED_STRS_MAP`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MAX_MODEL_LEN`、`MODELS`、`EXPECTED_STRS_MAP`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_models (L44-L87)
```python
@pytest.mark.skip(
    reason="Prevent unstable test based on golden strings from breaking the build "
    " and test input model being too large and hanging the system."
)
@pytest.mark.skipif(
    not is_quant_method_supported("modelopt_fp4"),
    reason="modelopt_fp4 is not supported on this GPU type.",
)
@pytest.mark.parametrize("model_name", MODELS)
def test_models(example_prompts, model_name) -> None:
    llm = LLM(
        model=model_name,
        max_model_len=MAX_MODEL_LEN,
        trust_remote_code=True,
        enforce_eager=True,
        quantization="modelopt_fp4",
    )

# ... 18 lines omitted for brevity ...
    print(model_name, generations)
    expected_strs = EXPECTED_STRS_MAP[model_name]
    for i in range(len(example_prompts)):
        generated_str = generations[i]
        expected_str = expected_strs[i]
        assert expected_str == generated_str, (
            f"Test{i}:\nExpected: {expected_str!r}\nvLLM: {generated_str!r}"
        )
```
**EN:** This test validates `test_models`. It uses parameterization over `model_name`. Relevant pytest markers include `skip`, `skipif`. Key inputs are `example_prompts`, `model_name`. It touches the core vLLM initialization or engine path directly. The main assertion is `expected_str == generated_str`.
**CN:** 这个测试验证 `test_models`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `skip`、`skipif`。 关键输入包括 `example_prompts`、`model_name`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 核心断言是 `expected_str == generated_str`。

### Module setup / 模块级配置: EAGER, SM_100_NVFP4_BACKENDS (L90-L96)
```python
EAGER = [True, False]

SM_100_NVFP4_BACKENDS = [
    "flashinfer_cudnn",
    "flashinfer_trtllm",
    "flashinfer_cutlass",
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `EAGER`, `SM_100_NVFP4_BACKENDS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `EAGER`、`SM_100_NVFP4_BACKENDS`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_nvfp4 (L99-L121)
```python
@pytest.mark.parametrize("model", ["nvidia/Llama-3.1-8B-Instruct-NVFP4"])
@pytest.mark.parametrize("eager", EAGER)
@pytest.mark.parametrize(
    "backend",
    [
        "emulation",
        "flashinfer_cudnn",
        "flashinfer_trtllm",  # the small seq_len ensures trtllm_8x4_layout backend is used
        "flashinfer_cutlass",
    ],
)
def test_nvfp4(vllm_runner, model, eager, backend):
    if (
        not current_platform.has_device_capability(100)
        and backend in SM_100_NVFP4_BACKENDS
    ):
        pytest.skip(
            f"The backend {backend} is not supported with current_platform.has_device_capability(100) == False"
        )

    with vllm_runner(model, enforce_eager=eager, linear_backend=backend) as llm:
        output = llm.generate_greedy(["1 2 3 4 5"], max_tokens=2)
    assert output[0][1] == "1 2 3 4 5 6"
```
**EN:** This test validates `test_nvfp4`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`, `eager`, `backend`. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `output[0][1] == '1 2 3 4 5 6'`.
**CN:** 这个测试验证 `test_nvfp4`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`、`eager`、`backend`。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `output[0][1] == '1 2 3 4 5 6'`。

### Test / 测试: test_nvfp4_moe (L124-L144)
```python
@pytest.mark.parametrize(
    "model",
    [
        "nvidia/Qwen3-30B-A3B-NVFP4",
        "RedHatAI/Qwen3-30B-A3B-NVFP4",
    ],
)
@pytest.mark.parametrize("backend", ["emulation"])
@pytest.mark.skipif(
    not current_platform.is_rocm(),
    reason="NVFP4 MOE emulation is only useful on AMD Instinct MI3xx",
)
def test_nvfp4_moe(vllm_runner, model, backend, monkeypatch):
    monkeypatch.setenv("VLLM_NVFP4_GEMM_BACKEND", backend)
    with vllm_runner(
        model,
        moe_backend=backend,
        load_format="dummy",
        hf_overrides={"num_hidden_layers": 2},
    ) as llm:
        _ = llm.generate_greedy(["1 2 3 4 5"], max_tokens=2)
```
**EN:** This test validates `test_nvfp4_moe`. It uses parameterization over `model`. Relevant pytest markers include `skipif`. Key inputs are `vllm_runner`, `model`, `backend`, `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_nvfp4_moe`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `vllm_runner`、`model`、`backend`、`monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `os`, `typing.List`
- **Third-party / 第三方**: `pytest`, `transformers.AutoTokenizer`
- **Project / 项目内**: `tests.quantization.utils.is_quant_method_supported`, `vllm.LLM`, `vllm.SamplingParams`, `vllm.platforms.current_platform`
