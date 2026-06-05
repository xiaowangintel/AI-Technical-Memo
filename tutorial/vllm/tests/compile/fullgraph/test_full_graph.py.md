# test_full_graph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/fullgraph/test_full_graph.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / fullgraph / test_full_graph, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / fullgraph / test_full_graph 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-18)
```python
import tempfile
from pathlib import Path
from typing import Any

import pytest
import torch

from tests.quantization.utils import is_quant_method_supported
from vllm import LLM, SamplingParams
from vllm.config import CompilationConfig, CompilationMode, CUDAGraphMode, PassConfig
from vllm.platforms import current_platform
from vllm.utils.torch_utils import is_torch_equal_or_newer
from vllm.v1.attention.backends.registry import AttentionBackendEnum

from ...utils import create_new_process_for_each_test
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as tempfile, pathlib, typing, pytest; shared test helpers from tests.quantization.utils, ...utils; and vLLM components like vllm, vllm.config, vllm.platforms, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 tempfile、pathlib、typing、pytest；共享测试辅助模块，例如 tests.quantization.utils、...utils；vLLM 内部组件，例如 vllm、vllm.config、vllm.platforms、vllm.utils.torch_utils。

### Function `models_list` (lines 21-71)
```python
def models_list(*, all: bool = True, keywords: list[str] | None = None):
    TEST_MODELS: list[tuple[str, dict[str, Any]]] = [
        ("facebook/opt-125m", {}),
        (
            "neuralmagic/Llama-3.2-1B-Instruct-FP8-dynamic",
            {"dtype": torch.float16},
        ),
        ("meta-llama/Llama-3.2-1B-Instruct", {}),
    ]

    if all:
        TEST_MODELS.extend(
            [
                ("neuralmagic/Llama-3.2-1B-Instruct-quantized.w8a8", {}),
                (
                    "nm-testing/tinyllama-oneshot-w8w8-test-static-shape-change",
                    {"dtype": torch.float16},
                ),
            ]
        )

        # TODO: figure out why this fails.
        if False and is_quant_method_supported("gguf"):  # noqa: SIM223
            TEST_MODELS.append(
                ("TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF", {"quantization": "gguf"})
            )

        if is_quant_method_supported("gptq"):
            TEST_MODELS.append(
                ("TheBloke/TinyLlama-1.1B-Chat-v0.3-GPTQ", {"quantization": "gptq"})
            )

        if is_quant_method_supported("gptq_marlin"):
            TEST_MODELS.append(
                (
                    "TheBloke/TinyLlama-1.1B-Chat-v1.0-GPTQ",
                    {"quantization": "gptq_marlin"},
                )
            )

        if not current_platform.is_rocm() and is_quant_method_supported("awq"):
            TEST_MODELS.append(
                ("TheBloke/TinyLlama-1.1B-Chat-v0.3-AWQ", {"quantization": "AWQ"})
            )

    if keywords is None:
        return TEST_MODELS

    # filter by keywords
    pred = lambda model: any(keyword in model[0] for keyword in keywords)
    return list(filter(pred, TEST_MODELS))
```
**EN:** This helper function implements the shared logic for models list. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 models list 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_full_graph` (lines 74-97)
```python
@pytest.mark.parametrize(
    "compilation_mode",
    [CompilationMode.DYNAMO_TRACE_ONCE, CompilationMode.VLLM_COMPILE],
)
@pytest.mark.parametrize("model, model_kwargs", models_list(all=True))
@create_new_process_for_each_test()
def test_full_graph(
    monkeypatch: pytest.MonkeyPatch,
    model: str,
    model_kwargs: dict[str, Any],
    compilation_mode: int,
):
    if (
        "w8a8" in model
        or "w8w8" in model
        and current_platform.has_device_capability((10, 0))
    ):
        # int8 removed on Blackwell:
        pytest.skip("int8 support removed on Blackwell")

    with monkeypatch.context():
        print(f"MODEL={model}")

        run_model(compilation_mode, model, **model_kwargs)
```
**EN:** This pytest case verifies full graph. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as monkeypatch, model, model_kwargs, compilation_mode. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 full graph 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 monkeypatch、model、model_kwargs、compilation_mode 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_custom_compile_config` (lines 101-188)
```python
@pytest.mark.parametrize(
    "compilation_config, model, model_kwargs",
    [
        # additional compile sizes, only some of the models
        (
            CompilationConfig(mode=CompilationMode.VLLM_COMPILE, compile_sizes=[1, 2]),
            *model_info,
        )
        for model_info in models_list(all=False)
    ]
    + [
        # RMSNorm + quant fusion, only 8-bit quant models
        (
            CompilationConfig(
                mode=CompilationMode.VLLM_COMPILE,
                custom_ops=["+rms_norm"],
                pass_config=PassConfig(
                    fuse_norm_quant=True, fuse_act_quant=True, eliminate_noops=True
                ),
            ),
            *model_info,
        )
        for model_info in models_list(keywords=["FP8-dynamic", "quantized.w8a8"])
    ]
    + [
        # Test depyf integration works
        (
            CompilationConfig(
                mode=CompilationMode.VLLM_COMPILE,
                debug_dump_path=Path(tempfile.gettempdir()),
            ),
            "facebook/opt-125m",
            {},
        ),
    ]
    + [
        # graph inductor partition
        (
            CompilationConfig(
                mode=CompilationMode.VLLM_COMPILE,
# ... excerpt ...
        if is_torch_equal_or_newer("2.9.0.dev")
    ]
    + [
        # Test get_raw_stream patch with compile_sizes
        # This tests that TorchInductor autotune works correctly with get_raw_stream
        # patch in torch 2.9 and without patch in torch 2.10+
        (
            CompilationConfig(
                mode=CompilationMode.VLLM_COMPILE,
                compile_sizes=[1, 2],  # Triggers autotune which uses get_raw_stream
                cudagraph_mode=CUDAGraphMode.NONE,
            ),
            "facebook/opt-125m",
            {},
        ),
    ],
)
# only test some of the models
@create_new_process_for_each_test()
def test_custom_compile_config(
    compilation_config: CompilationConfig,
    model: str,
    model_kwargs: dict[str, Any],
):
    if (
        "w8a8" in model
        or "w8w8" in model
        and current_platform.has_device_capability((10, 0))
    ):
        # int8 removed on Blackwell:
        pytest.skip("int8 support removed on Blackwell")

    if compilation_config.use_inductor_graph_partition and not is_torch_equal_or_newer(
        "2.9.0.dev"
    ):
        pytest.skip("inductor graph partition is only available in PyTorch 2.9+")

    print(f"MODEL={model}")
    run_model(compilation_config, model, **model_kwargs)
```
**EN:** This pytest case verifies custom compile config. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as compilation_config, model, model_kwargs. unsupported hardware, backend, or configuration combinations are skipped early. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 custom compile config 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 compilation_config、model、model_kwargs 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。 由于该代码块较大，这里只展示关键片段。

### Function `test_fp8_kv_scale_compile` (lines 191-219)
```python
@pytest.mark.parametrize(
    "compilation_mode",
    [CompilationMode.NONE, CompilationMode.VLLM_COMPILE],
)
@pytest.mark.parametrize(
    "model, backend",
    [
        ("Qwen/Qwen2-0.5B", None),  # Standard attention model
        (
            "deepseek-ai/DeepSeek-V2-Lite",
            AttentionBackendEnum.FLASHINFER_MLA,
        ),  # MLA (Multi-head Latent Attention) model
    ],
)
def test_fp8_kv_scale_compile(
    compilation_mode: int,
    model: str,
    backend: AttentionBackendEnum | None,
):
    model_kwargs = {
        "quantization": "fp8",
        "kv_cache_dtype": "fp8_e4m3",
        "calculate_kv_scales": True,
        "max_model_len": 512,
    }
    if backend:
        model_kwargs["attention_config"] = {"backend": backend.name}

    run_model(compilation_mode, model, **model_kwargs)
```
**EN:** This pytest case verifies FP8 KV scale compile. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as compilation_mode, model, backend.
**CN:** 该 pytest 用例验证 FP8 KV scale compile 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 compilation_mode、model、backend 等 fixture 或输入。

### Function `run_model` (lines 222-255)
```python
def run_model(compile_config: int | CompilationConfig, model: str, **model_kwargs):
    compilation_config = (
        compile_config
        if isinstance(compile_config, CompilationConfig)
        else CompilationConfig(mode=compile_config)
    )

    prompts = [
        "Hello, my name is",
        "The president of the United States is",
        "The capital of France is",
        "The future of AI is",
    ]
    sampling_params = SamplingParams(temperature=0)
    # Allow override from model_kwargs
    model_kwargs = {"tensor_parallel_size": 1, **model_kwargs}
    model_kwargs = {"disable_custom_all_reduce": True, **model_kwargs}

    # No cudagraphs by default
    if compilation_config.cudagraph_mode is None:
        compilation_config.cudagraph_mode = CUDAGraphMode.NONE

    llm = LLM(
        model=model,
        compilation_config=compilation_config,
        **model_kwargs,
    )
    outputs = llm.generate(prompts, sampling_params)

    # Print the outputs.
    for output in outputs:
        prompt = output.prompt
        generated_text = output.outputs[0].text
        print(f"Prompt: {prompt!r}, Generated text: {generated_text!r}")
```
**EN:** This helper function implements the shared logic for run model. the code exercises an end-to-end vLLM execution path instead of only isolated tensor math.
**CN:** 该辅助函数实现了 run model 所需的共享逻辑。 这里测试的是端到端 vLLM 执行路径，而不只是孤立的张量计算。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `tempfile`
- `pathlib -> Path`
- `typing -> Any`
- `pytest`
- `torch`
- `tests.quantization.utils -> is_quant_method_supported`
- `vllm -> LLM, SamplingParams`
- `vllm.config -> CompilationConfig, CompilationMode, CUDAGraphMode, PassConfig`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> is_torch_equal_or_newer`
- `vllm.v1.attention.backends.registry -> AttentionBackendEnum`
- `...utils -> create_new_process_for_each_test`
