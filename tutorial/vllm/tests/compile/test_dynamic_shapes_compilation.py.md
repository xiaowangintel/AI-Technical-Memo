# test_dynamic_shapes_compilation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/test_dynamic_shapes_compilation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / test_dynamic_shapes_compilation, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / test_dynamic_shapes_compilation 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-21)
```python
import gc
import tempfile
from contextlib import contextmanager

import pytest
import torch

from tests.models.utils import check_logprobs_close
from vllm import LLM, SamplingParams
from vllm.compilation.decorators import support_torch_compile
from vllm.config import CompilationConfig, VllmConfig, set_current_vllm_config
from vllm.config.compilation import (
    CompilationMode,
    DynamicShapesConfig,
    DynamicShapesType,
)
from vllm.forward_context import set_forward_context
from vllm.utils.torch_utils import is_torch_equal_or_newer
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as gc, tempfile, contextlib, pytest; shared test helpers from tests.models.utils; and vLLM components like vllm, vllm.compilation.decorators, vllm.config, vllm.config.compilation.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 gc、tempfile、contextlib、pytest；共享测试辅助模块，例如 tests.models.utils；vLLM 内部组件，例如 vllm、vllm.compilation.decorators、vllm.config、vllm.config.compilation。

### Function `get_test_models` (lines 24-33)
```python
def get_test_models():
    """Get list of models to test based on PyTorch version"""
    models = [
        "gpt2",
        "Qwen/Qwen2-7B-Instruct",
        "meta-llama/Llama-3.1-8B",
    ]
    if is_torch_equal_or_newer("2.12.0.dev"):
        models.append("Qwen/Qwen3-4B-Instruct-2507")
    return models
```
**EN:** This helper function implements the shared logic for test models. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 test models 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_dynamic_shapes_compilation` (lines 36-115)
```python
@pytest.mark.parametrize("model_name", get_test_models())
@pytest.mark.parametrize(
    "shapes_type",
    [
        DynamicShapesType.BACKED,
        DynamicShapesType.UNBACKED,
        DynamicShapesType.BACKED_SIZE_OBLIVIOUS,
    ],
)
@pytest.mark.parametrize("use_aot_compile", ["0", "1"])
@pytest.mark.parametrize("use_bytecode_hook", [True, False])
@pytest.mark.parametrize("evaluate_guards", [False, True])
@pytest.mark.skipif(not is_torch_equal_or_newer("2.10.0"), reason="requires torch 2.10")
def test_dynamic_shapes_compilation(
    monkeypatch,
    model_name,
    shapes_type,
    use_aot_compile,
    use_bytecode_hook,
    evaluate_guards,
):
    """Test that all dynamic shapes types compile successfully"""
    if evaluate_guards and shapes_type == DynamicShapesType.UNBACKED:
        pytest.skip("unbacked dynamic shapes do not add guards")

    # TODO is this still a requirement?
    if evaluate_guards and use_aot_compile:
        pytest.skip("evaluate_guards requires use_aot_compile=0")

    monkeypatch.setenv("VLLM_USE_AOT_COMPILE", use_aot_compile)
    monkeypatch.setenv("VLLM_USE_BYTECODE_HOOK", "1" if use_bytecode_hook else "0")

    prompt = "Hello, my name is"

    print(f"Testing {shapes_type.name} dynamic shapes...")

    # Initialize the model with specific dynamic shapes configuration
    model = LLM(
        model=model_name,
        compilation_config={
            "mode": CompilationMode.VLLM_COMPILE,
            "dynamic_shapes_config": {
                "type": shapes_type.value,
                "evaluate_guards": evaluate_guards,
            },
        },
        max_model_len=1024,
    )

    sampling_params = SamplingParams(max_tokens=5, temperature=0, logprobs=10)
    test_prompts = [prompt, "The capital of France is"]

    compiled_outputs = []
    for p in test_prompts:
        output = model.generate(p, sampling_params)[0].outputs[0]
        assert len(output.text.strip()) > 0, "Compiled model produced empty output"
        compiled_outputs.append((output.token_ids, output.text, output.logprobs))

    del model
    gc.collect()
    torch.accelerator.empty_cache()
    torch.accelerator.synchronize()

    eager_model = LLM(model=model_name, enforce_eager=True, max_model_len=1024)
    eager_outputs = []
    for p in test_prompts:
        output = eager_model.generate(p, sampling_params)[0].outputs[0]
        assert len(output.text.strip()) > 0, "Eager model produced empty output"
        eager_outputs.append((output.token_ids, output.text, output.logprobs))
    del eager_model
    gc.collect()
    torch.accelerator.empty_cache()
    torch.accelerator.synchronize()

    check_logprobs_close(
        outputs_0_lst=eager_outputs,
        outputs_1_lst=compiled_outputs,
        name_0="eager",
        name_1="compiled",
    )
```
**EN:** This pytest case verifies dynamic shapes compilation. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as monkeypatch, model_name, shapes_type, use_aot_compile. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 dynamic shapes compilation 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 monkeypatch、model_name、shapes_type、use_aot_compile 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_model_specialization_with_evaluate_guards` (lines 118-229)
```python
@pytest.mark.parametrize("use_aot_compile", ["0", "1"])
@pytest.mark.parametrize(
    "dynamic_shapes_type",
    [
        DynamicShapesType.BACKED,
        DynamicShapesType.BACKED_SIZE_OBLIVIOUS,
    ],
)
@pytest.mark.parametrize("evaluate_guards", [False, True])
def test_model_specialization_with_evaluate_guards(
    monkeypatch, use_aot_compile, dynamic_shapes_type, evaluate_guards
):
    """Test that evaluate_guards correctly detects shape specialization
    violations.
    """

    if (
        use_aot_compile == "1"
        and dynamic_shapes_type == DynamicShapesType.BACKED
        and evaluate_guards
    ):
        pytest.skip("evaluate_guards for backed does not work with aot_compile=1")

    @support_torch_compile
    class ModelWithSizeCheck(torch.nn.Module):
        def __init__(self, **kwargs):
            super().__init__()

        def forward(self, x: torch.Tensor):
            # This will cause specialization - torch.compile will guard on
            # sx.shape[0]
            if x.shape[0] >= 10:
                return x * 10
            else:
                return x * 10

    @support_torch_compile
    class ModelWithOneSizeCheck(torch.nn.Module):
        def __init__(self, **kwargs):
            super().__init__()
# ... excerpt ...
        with (
            torch.no_grad(),
            use_vllm_config(vllm_config),
            tempfile.TemporaryDirectory() as tmpdirname,
        ):
            monkeypatch.setenv("VLLM_CACHE_ROOT", tmpdirname)

            model = model_class(vllm_config=vllm_config).cuda()

            model(input1)

            if evaluate_guards and (
                not (
                    is_01_specialization
                    and dynamic_shapes_type == DynamicShapesType.BACKED
                )
            ):
                # This should fail because guards were added.
                with pytest.raises(RuntimeError) as excinfo:
                    model(input2)

                # Expected failure - guard was violated
                error_msg = str(excinfo.value)
                assert (
                    "GuardManager check failed" in error_msg
                    or "Detected recompile when torch.compile stance" in error_msg
                ), error_msg

            else:
                model(input2)

    test(ModelWithSizeCheck, torch.randn(20, 10).cuda(), torch.randn(5, 10).cuda())
    test(ModelWithSizeCheck, torch.randn(5, 10).cuda(), torch.randn(20, 10).cuda())
    test(
        ModelWithOneSizeCheck,
        torch.randn(20, 10).cuda(),
        torch.randn(1, 10).cuda(),
        is_01_specialization=True,
    )
```
**EN:** This pytest case verifies model specialization with evaluate guards. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as monkeypatch, use_aot_compile, dynamic_shapes_type, evaluate_guards. unsupported hardware, backend, or configuration combinations are skipped early. the expected failure path is asserted explicitly. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 model specialization with evaluate guards 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 monkeypatch、use_aot_compile、dynamic_shapes_type、evaluate_guards 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；代码会显式断言预期的失败路径。 由于该代码块较大，这里只展示关键片段。

### Function `test_piecewise_backend_empty_sym_shape_indices` (lines 232-273)
```python
@pytest.mark.skipif(not is_torch_equal_or_newer("2.10.0"), reason="requires torch 2.10")
def test_piecewise_backend_empty_sym_shape_indices():
    """Test that PiecewiseBackend handles empty sym_shape_indices correctly.

    When all inputs have static shapes (no torch.SymInt), sym_shape_indices
    will be empty. The fix in PiecewiseBackend.__call__ handles this case
    by using the first compiled range_entry.
    """
    gc.collect()
    torch.accelerator.empty_cache()
    torch.accelerator.synchronize()

    # Use small max_model_len and max_num_batched_tokens to encourage
    # static shape compilation with empty sym_shape_indices
    llm = LLM(
        model="Qwen/Qwen3-0.6B",
        max_model_len=512,
        max_num_batched_tokens=1,
        compilation_config={
            "mode": CompilationMode.VLLM_COMPILE,
            "dynamic_shapes_config": {
                "type": DynamicShapesType.BACKED.value,
            },
        },
    )

    sampling_params = SamplingParams(temperature=0, top_p=0.95, max_tokens=10)

    # Generate with static shape inputs
    output = llm.generate("Hello, my name is", sampling_params=sampling_params)
    result = output[0].outputs[0].text
    assert len(result) > 0, "Should generate non-empty output"

    # Generate again to verify compilation works with empty sym_shape_indices
    output = llm.generate("The capital of France is", sampling_params=sampling_params)
    result = output[0].outputs[0].text
    assert len(result) > 0, "Should generate non-empty output on second run"

    del llm
    gc.collect()
    torch.accelerator.empty_cache()
    torch.accelerator.synchronize()
```
**EN:** This pytest case verifies piecewise backend empty sym shape indices. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape. the code exercises an end-to-end vLLM execution path instead of only isolated tensor math.
**CN:** 该 pytest 用例验证 piecewise backend empty sym shape indices 的行为。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态；这里测试的是端到端 vLLM 执行路径，而不只是孤立的张量计算。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `gc`
- `tempfile`
- `contextlib -> contextmanager`
- `pytest`
- `torch`
- `tests.models.utils -> check_logprobs_close`
- `vllm -> LLM, SamplingParams`
- `vllm.compilation.decorators -> support_torch_compile`
- `vllm.config -> CompilationConfig, VllmConfig, set_current_vllm_config`
- `vllm.config.compilation -> CompilationMode, DynamicShapesConfig, DynamicShapesType`
- `vllm.forward_context -> set_forward_context`
- `vllm.utils.torch_utils -> is_torch_equal_or_newer`
