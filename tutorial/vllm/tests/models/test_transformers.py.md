# test_transformers.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/test_transformers.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers model-facing behavior. The file defines 6 test(s), 0 fixture(s), and 2 helper/class block(s) to validate this area. / [CN] 该文件覆盖面向模型的行为。它定义了 6 个测试、0 个 fixture，以及 2 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L12)
```python
from typing import Any

import pytest

from ..conftest import HfRunner, VllmRunner
from ..utils import multi_gpu_test, prep_prompts
from .registry import HF_EXAMPLE_MODELS
from .utils import check_embeddings_close, check_logprobs_close
```
**EN:** Imports standard-library modules such as `typing.Any`, third-party packages like `packaging.version.Version`, `pytest`, `transformers`, project helpers such as `..conftest.HfRunner`, `..conftest.VllmRunner`, `..utils.multi_gpu_test`.
**CN:** 导入标准库模块（如 `typing.Any`）、第三方包（如 `packaging.version.Version`、`pytest`、`transformers`）、项目内辅助模块（如 `..conftest.HfRunner`、`..conftest.VllmRunner`、`..utils.multi_gpu_test`）。

### Helper / 辅助函数: get_model (L15-L18)
```python
def get_model(arch: str) -> str:
    model_info = HF_EXAMPLE_MODELS.get_hf_info(arch)
    model_info.check_transformers_version(on_fail="skip")
    return model_info.default
```
**EN:** This helper encapsulates reusable logic in `get_model`. Key inputs are `arch`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_model` 中。 关键输入包括 `arch`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: check_implementation (L21-L57)
```python
def check_implementation(
    runner_ref: type[HfRunner | VllmRunner],
    runner_test: type[VllmRunner],
    example_prompts: list[str],
    model: str,
    kwargs_ref: dict[str, Any] | None = None,
    kwargs_test: dict[str, Any] | None = None,
    **kwargs,
):
    if kwargs_ref is None:
        kwargs_ref = {}
    if kwargs_test is None:
        kwargs_test = {}

    max_tokens = 32
    num_logprobs = 5

    args = (example_prompts, max_tokens, num_logprobs)
# ... 11 lines omitted for brevity ...
            outputs_ref = model_ref.generate_greedy_logprobs_limit(*args)

    check_logprobs_close(
        outputs_0_lst=outputs_ref,
        outputs_1_lst=outputs_test,
        name_0="ref",
        name_1="test",
    )
```
**EN:** This helper encapsulates reusable logic in `check_implementation`. Key inputs are `runner_ref`, `runner_test`, `example_prompts`, `model`, `kwargs_ref`, `kwargs_test`. The main assertion is `model_config.using_transformers_backend()`.
**CN:** 这个辅助函数将可复用逻辑封装在 `check_implementation` 中。 关键输入包括 `runner_ref`、`runner_test`、`example_prompts`、`model`、`kwargs_ref`、`kwargs_test`。 核心断言是 `model_config.using_transformers_backend()`。

### Test / 测试: test_models (L60-L88)
```python
@pytest.mark.parametrize(
    "model,model_impl",
    [
        ("meta-llama/Llama-3.2-1B-Instruct", "transformers"),
        ("hmellor/Ilama-3.2-1B", "auto"),  # CUSTOM CODE
        ("allenai/OLMoE-1B-7B-0924", "transformers"),  # MoE
    ],
)  # trust_remote_code=True by default
def test_models(
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    example_prompts: list[str],
    model: str,
    model_impl: str,
) -> None:
    import transformers
    from packaging.version import Version

    installed = Version(transformers.__version__)
    required = Version("5.0.0")
    if model == "allenai/OLMoE-1B-7B-0924" and installed < required:
        pytest.skip(
            "MoE models with the Transformers modeling backend require "
            f"transformers>={required}, but got {installed}"
        )

    check_implementation(
        hf_runner, vllm_runner, example_prompts, model, model_impl=model_impl
    )
```
**EN:** This test validates `test_models`. It uses parameterization over `model`, `model_impl`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model`, `model_impl`. The logic can skip unsupported environments when prerequisites are not satisfied.
**CN:** 这个测试验证 `test_models`。 它通过参数化组合 `model`、`model_impl`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model`、`model_impl`。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。

### Test / 测试: test_hybrid_attention (L91-L102)
```python
def test_hybrid_attention(vllm_runner: type[VllmRunner]) -> None:
    prompts, _, _ = prep_prompts(4, (800, 801))
    kwargs_ref = {"max_model_len": 8192, "enforce_eager": True}
    kwargs_test = {"model_impl": "transformers", **kwargs_ref}
    check_implementation(
        vllm_runner,
        vllm_runner,
        prompts,
        model="hmellor/tiny-random-Gemma2ForCausalLM",
        kwargs_ref=kwargs_ref,
        kwargs_test=kwargs_test,
    )
```
**EN:** This test validates `test_hybrid_attention`. Key inputs are `vllm_runner`.
**CN:** 这个测试验证 `test_hybrid_attention`。 关键输入包括 `vllm_runner`。

### Test / 测试: test_distributed (L105-L118)
```python
@multi_gpu_test(num_gpus=2)
def test_distributed(
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    example_prompts,
):
    kwargs = {"model_impl": "transformers", "tensor_parallel_size": 2}
    check_implementation(
        hf_runner,
        vllm_runner,
        example_prompts,
        "meta-llama/Llama-3.2-1B-Instruct",
        kwargs_test=kwargs,
    )
```
**EN:** This test validates `test_distributed`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`.
**CN:** 这个测试验证 `test_distributed`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`。

### Test / 测试: test_quantization (L121-L173)
```python
@pytest.mark.parametrize(
    "model, quantization_kwargs",
    [
        ("TheBloke/TinyLlama-1.1B-Chat-v0.3-AWQ", {}),
        ("TheBloke/TinyLlama-1.1B-Chat-v0.3-GPTQ", {}),
        (
            "meta-llama/Llama-3.2-1B-Instruct",
            {
                "quantization": "bitsandbytes",
            },
        ),
        ("unsloth/tinyllama-bnb-4bit", {}),
    ],
)
@pytest.mark.parametrize("max_tokens", [32])
@pytest.mark.parametrize("num_logprobs", [5])
def test_quantization(
    vllm_runner: type[VllmRunner],
# ... 27 lines omitted for brevity ...
        )

    check_logprobs_close(
        outputs_0_lst=transformers_outputs,
        outputs_1_lst=vllm_outputs,
        name_0="transformers",
        name_1="vllm",
    )
```
**EN:** This test validates `test_quantization`. It uses parameterization over `model`, `quantization_kwargs`. Key inputs are `vllm_runner`, `example_prompts`, `model`, `quantization_kwargs`, `max_tokens`, `num_logprobs`. The main assertion is `model_config.using_transformers_backend()`.
**CN:** 这个测试验证 `test_quantization`。 它通过参数化组合 `model`、`quantization_kwargs`。 关键输入包括 `vllm_runner`、`example_prompts`、`model`、`quantization_kwargs`、`max_tokens`、`num_logprobs`。 核心断言是 `model_config.using_transformers_backend()`。

### Test / 测试: test_embed_loading (L176-L194)
```python
@pytest.mark.parametrize(
    "model",
    [
        # Layers live in `layers`
        "Qwen/Qwen3-Embedding-0.6B",
        # Layers live in `model.layers`
        "meta-llama/Llama-3.2-1B-Instruct",
    ],
)
def test_embed_loading(vllm_runner, model):
    with vllm_runner(
        model,
        max_model_len=1024,
        enforce_eager=True,
        runner="pooling",
        model_impl="transformers",
    ) as model_test:
        model_config = model_test.llm.llm_engine.model_config
        assert model_config.using_transformers_backend()
```
**EN:** This test validates `test_embed_loading`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`. The main assertion is `model_config.using_transformers_backend()`.
**CN:** 这个测试验证 `test_embed_loading`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`。 核心断言是 `model_config.using_transformers_backend()`。

### Test / 测试: test_pooling (L197-L240)
```python
@pytest.mark.parametrize(
    "arch", ["TransformersEmbeddingModel", "TransformersForSequenceClassification"]
)
def test_pooling(hf_runner, vllm_runner, example_prompts, arch):
    model = get_model(arch)

    vllm_kwargs = dict(max_model_len=None, model_impl="transformers")

    hf_kwargs = dict()
    if arch == "TransformersEmbeddingModel":
        hf_kwargs["is_sentence_transformer"] = True
    elif arch == "TransformersForSequenceClassification":
        from transformers import AutoModelForSequenceClassification

        hf_kwargs["auto_cls"] = AutoModelForSequenceClassification

    # The example_prompts has ending "\n", for example:
    # "Write a short story about a robot that dreams for the first time.\n"
# ... 18 lines omitted for brevity ...
            hf_outputs = hf_model.classify(example_prompts)

    check_embeddings_close(
        embeddings_0_lst=hf_outputs,
        embeddings_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
    )
```
**EN:** This test validates `test_pooling`. It uses parameterization over `arch`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `arch`. The main assertion is `model_config.using_transformers_backend()`.
**CN:** 这个测试验证 `test_pooling`。 它通过参数化组合 `arch`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`arch`。 核心断言是 `model_config.using_transformers_backend()`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing.Any`
- **Third-party / 第三方**: `packaging.version.Version`, `pytest`, `transformers`, `transformers.AutoModelForSequenceClassification`
- **Local relative imports / 本地相对导入**: `..conftest.HfRunner`, `..conftest.VllmRunner`, `..utils.multi_gpu_test`, `..utils.prep_prompts`, `.registry.HF_EXAMPLE_MODELS`, `.utils.check_embeddings_close`, `.utils.check_logprobs_close`
