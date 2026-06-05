# test_full_cudagraph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/fullgraph/test_full_cudagraph.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / fullgraph / test_full_cudagraph, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / fullgraph / test_full_cudagraph 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-15)
```python
import contextlib
import os
import weakref

import pytest

from tests.utils import wait_for_gpu_memory_to_clear
from tests.v1.attention.utils import full_cg_backend_configs as backend_configs
from vllm import LLM, SamplingParams
from vllm.config import CompilationConfig
from vllm.platforms import current_platform
from vllm.utils.torch_utils import is_torch_equal_or_newer
from vllm.v1.attention.backends.registry import AttentionBackendEnum
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as contextlib, os, weakref, pytest; shared test helpers from tests.utils, tests.v1.attention.utils; and vLLM components like vllm, vllm.config, vllm.platforms, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 contextlib、os、weakref、pytest；共享测试辅助模块，例如 tests.utils、tests.v1.attention.utils；vLLM 内部组件，例如 vllm、vllm.config、vllm.platforms、vllm.utils.torch_utils。

### Function `temporary_environ` (lines 18-34)
```python
@contextlib.contextmanager
def temporary_environ(env_vars):
    """
    Temporarily set environment variables and restore them afterward.
    We have to do this vs monkeypatch because monkeypatch doesn't work
    with "module" scoped fixtures.
    """
    original_env = {k: os.environ.get(k) for k in env_vars}
    try:
        os.environ.update(env_vars)
        yield
    finally:
        for k, v in original_env.items():
            if v is None:
                os.environ.pop(k, None)
            else:
                os.environ[k] = v
```
**EN:** This helper function implements the shared logic for temporary environ. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 temporary environ 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 37-40)
```python
model_backends_full_cudagraph = []

# deepseek-ai/DeepSeek-V2-Lite with MLA
MLA_backends = ["FlashMLA", "FlashAttentionMLA", "CutlassMLA"]
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Top-level block starting at line 41 (lines 41-44)
```python
for mla_backend in MLA_backends:
    model_backends_full_cudagraph.append(
        ("deepseek-ai/DeepSeek-V2-Lite", backend_configs[mla_backend])
    )
```
**EN:** This top-level `For` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `For` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 47-49)
```python
other_backend_configs = [
    backend_configs[c] for c in backend_configs if c not in MLA_backends
]
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Top-level block starting at line 50 (lines 50-51)
```python
for backend_config in other_backend_configs:
    model_backends_full_cudagraph.append(("Qwen/Qwen2-1.5B-Instruct", backend_config))
```
**EN:** This top-level `For` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `For` 代码块执行周边测试所依赖的辅助逻辑。

### Function `llm_pair` (lines 54-113)
```python
@pytest.fixture(scope="class")
def llm_pair(request):
    model, backend_config, use_inductor_graph_partition = request.param
    backend_config.comp_config["use_inductor_graph_partition"] = (
        use_inductor_graph_partition
    )

    if use_inductor_graph_partition and not is_torch_equal_or_newer("2.9.0.dev"):
        pytest.skip("Inductor graph partition only supported in torch>=2.9")

    # Dynamically skip test if GPU capability is not met
    if (
        backend_config.specific_gpu_arch
        and backend_config.specific_gpu_arch != current_platform.get_device_capability()
    ):
        if backend_config.specific_gpu_arch == (9, 0):
            pytest.skip("Only Hopper GPUs support FA3 and FlashMLA")
        elif backend_config.specific_gpu_arch == (10, 0):
            pytest.skip("Only Blackwell GPUs support Cutlass MLA")

    # FlashInfer is not supported on ROCm
    if backend_config == AttentionBackendEnum.FLASHINFER and current_platform.is_rocm():
        pytest.skip("FlashInfer is not supported on ROCm")

    env_vars = {
        # Force native sampler to avoid potential nondeterminism in FlashInfer
        # when per-request generators are not used in V1.
        "VLLM_USE_FLASHINFER_SAMPLER": "0",
    }
    with temporary_environ(env_vars):
        full = LLM(
            model=model,
            gpu_memory_utilization=0.43,
            trust_remote_code=True,
            max_model_len=1024,
            max_num_seqs=128,
            compilation_config=CompilationConfig(**backend_config.comp_config),
            generation_config="vllm",
            seed=42,
        )
        piecewise = LLM(
            model=model,
            gpu_memory_utilization=0.43,
            trust_remote_code=True,
            max_model_len=1024,
            max_num_seqs=128,
            compilation_config=CompilationConfig(cudagraph_mode="PIECEWISE"),
            generation_config="vllm",
            seed=42,
        )

    # PyTest caches the fixture values so we use weakref.proxy to enable GC
    yield weakref.proxy(full), weakref.proxy(piecewise)
    del full
    del piecewise

    wait_for_gpu_memory_to_clear(
        devices=[0],
        threshold_ratio=0.1,
    )
```
**EN:** This fixture prepares reusable state for LLM pair. it consumes fixtures or inputs such as request. unsupported hardware, backend, or configuration combinations are skipped early. the code exercises an end-to-end vLLM execution path instead of only isolated tensor math.
**CN:** 该 fixture 为 LLM pair 准备可复用的测试状态。 它会使用诸如 request 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；这里测试的是端到端 vLLM 执行路径，而不只是孤立的张量计算。

### Class `TestFullCUDAGraph` (lines 116-133)
```python
@pytest.mark.parametrize(
    "llm_pair",
    [
        pytest.param((model, backend_config, use_inductor_graph_partition))
        for model, backend_config in model_backends_full_cudagraph
        for use_inductor_graph_partition in [True, False]
    ],
    indirect=True,
)
class TestFullCUDAGraph:
    """
    Use a class such that an llm pair is constructed once for all
    batch_size/max_tokens combinations and released immediately after.

    Module-scope fixtures would stick around the whole time,
    meaning there would be multiple LLM instances hogging memory simultaneously.
    """
```
**EN:** This helper class groups the state and behavior needed for TestFullCUDAGraph. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestFullCUDAGraph 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestFullCUDAGraph.test_full_cudagraph` (lines 134-172)
```python
    @pytest.mark.parametrize(
        ("batch_size", "max_tokens"),
        [
            (1, 10),
            (7, 10),
            (16, 10),
            (25, 10),
            (32, 10),
            (45, 10),
            (64, 10),
            (123, 10),
            (8, 5),
            (8, 30),
        ],
    )
    def test_full_cudagraph(self, batch_size, max_tokens, llm_pair: tuple[LLM, LLM]):
        """
        Test various batch sizes and max_tokens to ensure that the
        full cudagraph compilation works for padded cases too.
        """

        full_cudagraph_llm, piecewise_llm = llm_pair

        prompts = ["the quick brown fox"] * batch_size
        # Use purely greedy decoding to avoid top-p truncation sensitivity
        # that can amplify tiny numeric differences across runtimes.
        sampling_params = SamplingParams(
            temperature=0.0, max_tokens=max_tokens, top_p=1.0
        )

        piecewise_responses = piecewise_llm.generate(prompts, sampling_params)
        full_responses = full_cudagraph_llm.generate(prompts, sampling_params)

        # Check that all responses are the same
        for piecewise_res, full_res in zip(piecewise_responses, full_responses):
            assert (
                piecewise_res.outputs[0].text.lower()
                == full_res.outputs[0].text.lower()
            )
```
**EN:** This method on `TestFullCUDAGraph` checks full cudagraph. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as batch_size, max_tokens, llm_pair. assertions at the end lock in the intended behavior or graph shape. the code exercises an end-to-end vLLM execution path instead of only isolated tensor math.
**CN:** `TestFullCUDAGraph` 中的这个方法用于检查 full cudagraph。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 batch_size、max_tokens、llm_pair 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态；这里测试的是端到端 vLLM 执行路径，而不只是孤立的张量计算。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。
- **Reusable fixtures / 可复用 fixture:** Fixtures package common setup so each test can focus on the scenario under study. / fixture 封装公共准备逻辑，让每个测试更专注于待验证场景。

## Dependencies / 依赖关系
- `contextlib`
- `os`
- `weakref`
- `pytest`
- `tests.utils -> wait_for_gpu_memory_to_clear`
- `tests.v1.attention.utils -> full_cg_backend_configs`
- `vllm -> LLM, SamplingParams`
- `vllm.config -> CompilationConfig`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> is_torch_equal_or_newer`
- `vllm.v1.attention.backends.registry -> AttentionBackendEnum`
