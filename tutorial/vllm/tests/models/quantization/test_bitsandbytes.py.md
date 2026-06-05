# test_bitsandbytes.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/quantization/test_bitsandbytes.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers quantized model support and model-facing behavior. The file defines 7 test(s), 0 fixture(s), and 2 helper/class block(s) to validate this area. / [CN] 该文件覆盖量化模型支持与面向模型的行为。它定义了 7 个测试、0 个 fixture，以及 2 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L8-L17)
```python
import pytest
from packaging.version import Version
from transformers import BitsAndBytesConfig
from transformers import __version__ as TRANSFORMERS_VERSION

from tests.quantization.utils import is_quant_method_supported
from vllm.platforms import current_platform

from ...utils import compare_two_settings, multi_gpu_test
from ..utils import check_embeddings_close, check_logprobs_close
```
**EN:** Imports third-party packages like `packaging.version.Version`, `pytest`, `transformers.BitsAndBytesConfig`, project helpers such as `tests.quantization.utils.is_quant_method_supported`, `vllm.platforms.current_platform`, `vllm.platforms.rocm.on_gfx9`.
**CN:** 导入第三方包（如 `packaging.version.Version`、`pytest`、`transformers.BitsAndBytesConfig`）、项目内辅助模块（如 `tests.quantization.utils.is_quant_method_supported`、`vllm.platforms.current_platform`、`vllm.platforms.rocm.on_gfx9`）。

### Module setup / 模块级配置: models_4bit_to_test, models_4bit_to_embedding_test, models_4bit_to_moe_test (L19-L54)
```python
if current_platform.is_rocm():
    from vllm.platforms.rocm import on_gfx9

    pytestmark = pytest.mark.skipif(
        on_gfx9(),
        reason="bitsandbytes not supported on gfx9 (warp size 64 limitation)",
    )

models_4bit_to_test = [
    ("facebook/opt-125m", "quantize opt model inflight"),
    (
        "mistralai/Mistral-7B-Instruct-v0.3",
        "quantize inflight model with both HF and Mistral format weights",
    ),
]

# ... 14 lines omitted for brevity ...
]

models_pre_quant_8bit_to_test = [
    ("meta-llama/Llama-Guard-3-8B-INT8", "read pre-quantized llama 8-bit model"),
    ("yec019/fbopt-350m-8bit", "read pre-quantized 8-bit opt model"),
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `models_4bit_to_test`, `models_4bit_to_embedding_test`, `models_4bit_to_moe_test`, `models_pre_qaunt_4bit_to_test`, `models_pre_quant_8bit_to_test`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `models_4bit_to_test`、`models_4bit_to_embedding_test`、`models_4bit_to_moe_test`、`models_pre_qaunt_4bit_to_test`、`models_pre_quant_8bit_to_test`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_load_4bit_bnb_model (L57-L68)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("bitsandbytes"),
    reason="bitsandbytes is not supported on this GPU type.",
)
@pytest.mark.parametrize("model_name, description", models_4bit_to_test)
def test_load_4bit_bnb_model(
    hf_runner, vllm_runner, example_prompts, model_name, description
) -> None:
    hf_model_kwargs = dict(quantization_config=BitsAndBytesConfig(load_in_4bit=True))
    validate_generated_texts(
        hf_runner, vllm_runner, example_prompts[:1], model_name, False, hf_model_kwargs
    )
```
**EN:** This test validates `test_load_4bit_bnb_model`. It uses parameterization over `model_name`, `description`. Relevant pytest markers include `skipif`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model_name`, `description`.
**CN:** 这个测试验证 `test_load_4bit_bnb_model`。 它通过参数化组合 `model_name`、`description`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model_name`、`description`。

### Test / 测试: test_load_pre_quant_4bit_bnb_model (L71-L81)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("bitsandbytes"),
    reason="bitsandbytes is not supported on this GPU type.",
)
@pytest.mark.parametrize("model_name, description", models_pre_qaunt_4bit_to_test)
def test_load_pre_quant_4bit_bnb_model(
    hf_runner, vllm_runner, example_prompts, model_name, description
) -> None:
    validate_generated_texts(
        hf_runner, vllm_runner, example_prompts[:1], model_name, True
    )
```
**EN:** This test validates `test_load_pre_quant_4bit_bnb_model`. It uses parameterization over `model_name`, `description`. Relevant pytest markers include `skipif`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model_name`, `description`.
**CN:** 这个测试验证 `test_load_pre_quant_4bit_bnb_model`。 它通过参数化组合 `model_name`、`description`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model_name`、`description`。

### Test / 测试: test_load_8bit_bnb_model (L84-L94)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("bitsandbytes"),
    reason="bitsandbytes is not supported on this GPU type.",
)
@pytest.mark.parametrize("model_name, description", models_pre_quant_8bit_to_test)
def test_load_8bit_bnb_model(
    hf_runner, vllm_runner, example_prompts, model_name, description
) -> None:
    validate_generated_texts(
        hf_runner, vllm_runner, example_prompts[:1], model_name, True
    )
```
**EN:** This test validates `test_load_8bit_bnb_model`. It uses parameterization over `model_name`, `description`. Relevant pytest markers include `skipif`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model_name`, `description`.
**CN:** 这个测试验证 `test_load_8bit_bnb_model`。 它通过参数化组合 `model_name`、`description`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model_name`、`description`。

### Test / 测试: test_load_tp_4bit_bnb_model (L97-L115)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("bitsandbytes"),
    reason="bitsandbytes is not supported on this GPU type.",
)
@pytest.mark.parametrize("model_name, description", models_4bit_to_test)
@multi_gpu_test(num_gpus=2)
def test_load_tp_4bit_bnb_model(
    hf_runner, vllm_runner, example_prompts, model_name, description
) -> None:
    hf_model_kwargs = dict(quantization_config=BitsAndBytesConfig(load_in_4bit=True))
    validate_generated_texts(
        hf_runner,
        vllm_runner,
        example_prompts[:1],
        model_name,
        False,
        hf_model_kwargs,
        vllm_tp_size=2,
    )
```
**EN:** This test validates `test_load_tp_4bit_bnb_model`. It uses parameterization over `model_name`, `description`. Relevant pytest markers include `skipif`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model_name`, `description`.
**CN:** 这个测试验证 `test_load_tp_4bit_bnb_model`。 它通过参数化组合 `model_name`、`description`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model_name`、`description`。

### Test / 测试: test_load_pp_4bit_bnb_model (L118-L140)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("bitsandbytes"),
    reason="bitsandbytes is not supported on this GPU type.",
)
@pytest.mark.parametrize("model_name, description", models_4bit_to_test)
@multi_gpu_test(num_gpus=2)
def test_load_pp_4bit_bnb_model(model_name, description) -> None:
    common_args = [
        "--disable-log-stats",
        "--dtype",
        "bfloat16",
        "--enable-prefix-caching",
        "--quantization",
        "bitsandbytes",
        "--gpu-memory-utilization",
        "0.7",
    ]
    pp_args = [
        *common_args,
        "--pipeline-parallel-size",
        "2",
    ]
    compare_two_settings(model_name, common_args, pp_args)
```
**EN:** This test validates `test_load_pp_4bit_bnb_model`. It uses parameterization over `model_name`, `description`. Relevant pytest markers include `skipif`. Key inputs are `model_name`, `description`.
**CN:** 这个测试验证 `test_load_pp_4bit_bnb_model`。 它通过参数化组合 `model_name`、`description`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `model_name`、`description`。

### Test / 测试: test_4bit_bnb_moe_model (L143-L185)
```python
@pytest.mark.skipif(
    Version(TRANSFORMERS_VERSION) >= Version("5.0.0"),
    reason="Need to add support for quantizing MoE experts with bnb"
    " in transformers v5. See"
    " https://github.com/bitsandbytes-foundation/bitsandbytes/issues/1849",
)
@pytest.mark.skipif(
    not is_quant_method_supported("bitsandbytes"),
    reason="bitsandbytes is not supported on this GPU type.",
)
@pytest.mark.parametrize("model_name, description", models_4bit_to_moe_test)
def test_4bit_bnb_moe_model(
    hf_runner, vllm_runner, example_prompts, model_name, description
) -> None:
    hf_model_kwargs = dict(
        quantization_config=BitsAndBytesConfig(
            load_in_4bit=True,
            bnb_4bit_quant_type="nf4",
# ... 17 lines omitted for brevity ...
            example_prompts, max_tokens=32, num_logprobs=5
        )
    check_logprobs_close(
        outputs_0_lst=transformers_outputs,
        outputs_1_lst=vllm_outputs,
        name_0="transformers",
        name_1="vllm",
    )
```
**EN:** This test validates `test_4bit_bnb_moe_model`. It uses parameterization over `model_name`, `description`. Relevant pytest markers include `skipif`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model_name`, `description`.
**CN:** 这个测试验证 `test_4bit_bnb_moe_model`。 它通过参数化组合 `model_name`、`description`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model_name`、`description`。

### Test / 测试: test_4bit_bnb_embedding_model (L188-L237)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("bitsandbytes"),
    reason="bitsandbytes is not supported on this GPU type.",
)
@pytest.mark.parametrize("model_name, description", models_4bit_to_embedding_test)
@pytest.mark.parametrize("dtype", ["half"])
def test_4bit_bnb_embedding_model(
    model_name,
    description,
    hf_runner,
    vllm_runner,
    example_prompts,
    dtype: str,
) -> None:
    # The example_prompts has ending "\n", for example:
    # "Write a short story about a robot that dreams for the first time.\n"
    # sentence_transformers will strip the input texts, see:
    # https://github.com/UKPLab/sentence-transformers/blob/v3.1.1/sentence_transformers/models/Transformer.py#L159
# ... 24 lines omitted for brevity ...

    check_embeddings_close(
        embeddings_0_lst=hf_outputs,
        embeddings_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
        tol=5e-2,
    )
```
**EN:** This test validates `test_4bit_bnb_embedding_model`. It uses parameterization over `model_name`, `description`. Relevant pytest markers include `skipif`. Key inputs are `model_name`, `description`, `hf_runner`, `vllm_runner`, `example_prompts`, `dtype`.
**CN:** 这个测试验证 `test_4bit_bnb_embedding_model`。 它通过参数化组合 `model_name`、`description`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `model_name`、`description`、`hf_runner`、`vllm_runner`、`example_prompts`、`dtype`。

### Helper / 辅助函数: log_generated_texts (L240-L249)
```python
def log_generated_texts(prompts, outputs, runner_name):
    logged_texts = []
    for i, (_, generated_text) in enumerate(outputs):
        log_entry = {
            "prompt": prompts[i],
            "runner_name": runner_name,
            "generated_text": generated_text,
        }
        logged_texts.append(log_entry)
    return logged_texts
```
**EN:** This helper encapsulates reusable logic in `log_generated_texts`. Key inputs are `prompts`, `outputs`, `runner_name`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `log_generated_texts` 中。 关键输入包括 `prompts`、`outputs`、`runner_name`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: validate_generated_texts (L252-L298)
```python
def validate_generated_texts(
    hf_runner,
    vllm_runner,
    prompts,
    model_name,
    pre_quant=False,
    hf_model_kwargs=None,
    vllm_tp_size=1,
    max_tokens=8,
):
    # NOTE: run vLLM first, as it requires a clean process
    # when using distributed inference
    with vllm_runner(
        model_name,
        quantization=None if pre_quant else "bitsandbytes",
        tensor_parallel_size=vllm_tp_size,
        enforce_eager=False,
        default_torch_num_threads=1,
# ... 21 lines omitted for brevity ...
        prompt = hf_log["prompt"]
        assert hf_str == vllm_str, (
            f"Model: {model_name}"
            f"Mismatch between HF and vLLM outputs:\n"
            f"Prompt: {prompt}\n"
            f"HF Output: '{hf_str}'\n"
            f"vLLM Output: '{vllm_str}'"
        )
```
**EN:** This helper encapsulates reusable logic in `validate_generated_texts`. Key inputs are `hf_runner`, `vllm_runner`, `prompts`, `model_name`, `pre_quant`, `hf_model_kwargs`. The main assertion is `hf_str == vllm_str`.
**CN:** 这个辅助函数将可复用逻辑封装在 `validate_generated_texts` 中。 关键输入包括 `hf_runner`、`vllm_runner`、`prompts`、`model_name`、`pre_quant`、`hf_model_kwargs`。 核心断言是 `hf_str == vllm_str`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `packaging.version.Version`, `pytest`, `transformers.BitsAndBytesConfig`, `transformers.__version__`
- **Project / 项目内**: `tests.quantization.utils.is_quant_method_supported`, `vllm.platforms.current_platform`, `vllm.platforms.rocm.on_gfx9`
- **Local relative imports / 本地相对导入**: `...utils.compare_two_settings`, `...utils.multi_gpu_test`, `..utils.check_embeddings_close`, `..utils.check_logprobs_close`
