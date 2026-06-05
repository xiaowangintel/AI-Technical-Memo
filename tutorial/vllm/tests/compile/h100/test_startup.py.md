# test_startup.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/h100/test_startup.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / h100 / test_startup, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / h100 / test_startup 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-8)
```python
"""Cold start and warm start tests for vLLM-compile.

Cold start runs in a forked child (must fork before CUDA init) which
populates on-disk caches and asserts cold-start counters.  Warm start
then runs in the parent with clean in-memory state but populated caches.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 10-21)
```python
import multiprocessing as mp
from typing import NamedTuple

import pytest
from torch._dynamo.utils import counters

import vllm.envs as envs
from vllm.compilation.counter import compilation_counter
from vllm.config import CompilationConfig, CompilationMode, CUDAGraphMode, PassConfig
from vllm.utils.torch_utils import is_torch_equal_or_newer

from ...utils import fork_new_process_for_each_test
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as multiprocessing, typing, pytest, torch._dynamo.utils; shared test helpers from ...utils; and vLLM components like vllm.envs, vllm.compilation.counter, vllm.config, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 multiprocessing、typing、pytest、torch._dynamo.utils；共享测试辅助模块，例如 ...utils；vLLM 内部组件，例如 vllm.envs、vllm.compilation.counter、vllm.config、vllm.utils.torch_utils。

### Constants and module state (lines 23-23)
```python
MODEL = "microsoft/Phi-tiny-MoE-instruct"
```
**EN:** This block centralizes shared constants and parameter grids, including MODEL. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 MODEL。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `_run_vllm` (lines 26-42)
```python
def _run_vllm(vllm_runner):
    with vllm_runner(
        MODEL,
        trust_remote_code=False,
        max_model_len=256,
        max_num_batched_tokens=1024,
        load_format="dummy",
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            cudagraph_mode=CUDAGraphMode.NONE,
        ),
        # Phi-tiny-MoE uses SWA, whose admission cap is `cdiv(L, block_size) + 1`
        # at default block_size=16 — i.e. 17 blocks for max_model_len=256. Use
        # 32 for headroom.
        num_gpu_blocks_override=32,
    ):
        pass
```
**EN:** This helper function implements the shared logic for run vllm. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 run vllm 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_cold_start` (lines 45-54)
```python
def _cold_start(vllm_runner):
    counters.clear()
    with compilation_counter.expect(
        num_compiled_artifacts_saved=3,
        num_compiled_artifacts_loaded=0,
    ):
        _run_vllm(vllm_runner)
    assert counters["aot_autograd"]["total"] == 33
    assert counters["aot_autograd"]["autograd_cache_miss"] == 3
    assert counters["aot_autograd"]["autograd_cache_hit"] == 0
```
**EN:** This helper function implements the shared logic for cold start. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 cold start 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_moe_startup` (lines 57-92)
```python
@fork_new_process_for_each_test
@pytest.mark.parametrize("mega_aot_artifact", ["0", "1"])
def test_moe_startup(monkeypatch, vllm_runner, fresh_vllm_cache, mega_aot_artifact):
    monkeypatch.setenv("VLLM_ENABLE_V1_MULTIPROCESSING", "0")
    monkeypatch.setenv("VLLM_USE_MEGA_AOT_ARTIFACT", mega_aot_artifact)
    monkeypatch.setenv("VLLM_DEEP_GEMM_WARMUP", "skip")

    # Cold start in a forked child (must fork before CUDA init).
    # This model has 32 identical transformer layers which produce
    # 33 subgraphs after splitting on attention — only 3 are unique.
    ctx = mp.get_context("fork")
    p = ctx.Process(target=_cold_start, args=(vllm_runner,))
    p.start()
    p.join()
    assert p.exitcode == 0, "Cold-start child failed"

    # Warm start — compiled artifacts loaded from disk cache.
    counters.clear()
    with compilation_counter.expect(
        num_compiled_artifacts_loaded=3,
        num_compiled_artifacts_saved=0,
    ):
        _run_vllm(vllm_runner)
    mega_aot_active = envs.VLLM_USE_MEGA_AOT_ARTIFACT and is_torch_equal_or_newer(
        "2.10.0"
    )
    if mega_aot_active:
        # MEGA_AOT_ARTIFACT is enabled, so we expect no aot_autograd running on
        # subgraphs.
        assert counters["aot_autograd"]["total"] == 0
    else:
        assert counters["aot_autograd"]["total"] == 30
    assert counters["aot_autograd"]["autograd_cache_miss"] == 0
    assert (
        counters["aot_autograd"]["autograd_cache_hit"] == 0
    )  # No miss at aot_autograd level causing disk I/O.
```
**EN:** This pytest case verifies MoE startup. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as monkeypatch, vllm_runner, fresh_vllm_cache, mega_aot_artifact. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 MoE startup 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 monkeypatch、vllm_runner、fresh_vllm_cache、mega_aot_artifact 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Class `ModelStartupSpec` (lines 100-105)
```python
class ModelStartupSpec(NamedTuple):
    model: str
    hf_overrides: dict
    cold_artifacts_saved: int
    warm_artifacts_saved: int
    warm_artifacts_loaded: int
```
**EN:** This class defines a container for ModelStartupSpec.
**CN:** 该类定义了 ModelStartupSpec 对应的容器。

### Constants and module state (lines 108-179)
```python
_SMALL_MOE_OVERRIDES = {
    "num_hidden_layers": 8,
    "hidden_size": 256,
    "intermediate_size": 512,
    "num_attention_heads": 8,
    "num_key_value_heads": 1,
    "n_routed_experts": 8,
}

MODEL_SPECS = [
    pytest.param(
        ModelStartupSpec(
            model="openai/gpt-oss-120b",
            hf_overrides={
                "num_hidden_layers": 8,
                "hidden_size": 256,
                "intermediate_size": 512,
                "num_attention_heads": 8,
                "num_key_value_heads": 1,
                "num_local_experts": 8,
            },
            cold_artifacts_saved=3,
            warm_artifacts_saved=0,
            warm_artifacts_loaded=3,
        ),
        id="gpt_oss_120b",
    ),
    # NOTE: DeepSeek-V3.2 requires sparse MLA (index_topk) which needs
    # Hopper+ GPUs. This test must run on H100 (see pytorch.yaml).
    pytest.param(
        ModelStartupSpec(
            model="deepseek-ai/DeepSeek-V3.2",
            hf_overrides=_SMALL_MOE_OVERRIDES,
            cold_artifacts_saved=4,
            # https://github.com/vllm-project/vllm/issues/38051
            warm_artifacts_saved=0 if is_torch_equal_or_newer("2.12.0") else 4,
            warm_artifacts_loaded=4 if is_torch_equal_or_newer("2.12.0") else 0,
        ),
        id="deepseek_v3.2",
    ),
    pytest.param(
        ModelStartupSpec(
            model="moonshotai/Kimi-K2.5",
            hf_overrides={"text_config": _SMALL_MOE_OVERRIDES},
            cold_artifacts_saved=4,
            # https://github.com/vllm-project/vllm/issues/38051
            warm_artifacts_saved=0 if is_torch_equal_or_newer("2.12.0") else 4,
            warm_artifacts_loaded=4 if is_torch_equal_or_newer("2.12.0") else 0,
        ),
        id="kimi_k2.5",
    ),
    pytest.param(
        ModelStartupSpec(
            model="zai-org/GLM-4.5",
            hf_overrides=_SMALL_MOE_OVERRIDES,
            cold_artifacts_saved=4,
            warm_artifacts_saved=0,
            warm_artifacts_loaded=4,
        ),
        id="glm_4.5",
    ),
    pytest.param(
        ModelStartupSpec(
            model="MiniMaxAI/MiniMax-M2.5",
            hf_overrides=_SMALL_MOE_OVERRIDES,
            cold_artifacts_saved=3,
            warm_artifacts_saved=0,
            warm_artifacts_loaded=3,
        ),
        id="minimax_m2.5",
    ),
]
```
**EN:** This block centralizes shared constants and parameter grids, including _SMALL_MOE_OVERRIDES, MODEL_SPECS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 _SMALL_MOE_OVERRIDES、MODEL_SPECS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `_run_model` (lines 182-198)
```python
def _run_model(vllm_runner, spec: ModelStartupSpec):
    with vllm_runner(
        spec.model,
        trust_remote_code=True,
        max_model_len=256,
        max_num_batched_tokens=1024,
        block_size=64,
        load_format="dummy",
        hf_overrides=spec.hf_overrides,
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            cudagraph_mode=CUDAGraphMode.NONE,
            pass_config=PassConfig(fuse_allreduce_rms=False),
        ),
        num_gpu_blocks_override=16,
    ):
        pass
```
**EN:** This helper function implements the shared logic for run model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 run model 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_check_model_run` (lines 201-231)
```python
def _check_model_run(vllm_runner, spec: ModelStartupSpec, is_cold_start: bool):
    """Runs a model and checks the number of compiled artifacts."""
    old = compilation_counter.clone()
    _run_model(vllm_runner, spec)
    saved = (
        compilation_counter.num_compiled_artifacts_saved
        - old.num_compiled_artifacts_saved
    )
    loaded = (
        compilation_counter.num_compiled_artifacts_loaded
        - old.num_compiled_artifacts_loaded
    )

    start_type = "COLD" if is_cold_start else "WARM"
    # Print actual values for debugging — intentional, helps diagnose
    # failures and calibrate expected counts when adding new models.
    print(f"\n=== {start_type} START for {spec.model} ===")
    print(f"  num_compiled_artifacts_saved={saved}")
    print(f"  num_compiled_artifacts_loaded={loaded}")

    if is_cold_start:
        expected_saved = spec.cold_artifacts_saved
        expected_loaded = 0
    else:
        expected_saved = spec.warm_artifacts_saved
        expected_loaded = spec.warm_artifacts_loaded

    assert saved == expected_saved, f"{start_type.lower()}_artifacts_saved: got {saved}"
    assert loaded == expected_loaded, (
        f"{start_type.lower()}_artifacts_loaded: got {loaded}"
    )
```
**EN:** This helper function implements the shared logic for check model run. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 check model run 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `_cold_start_model` (lines 234-235)
```python
def _cold_start_model(vllm_runner, spec: ModelStartupSpec):
    _check_model_run(vllm_runner, spec, is_cold_start=True)
```
**EN:** This helper function implements the shared logic for cold start model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 cold start model 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_model_startup` (lines 238-252)
```python
@pytest.mark.parametrize("spec", MODEL_SPECS)
@fork_new_process_for_each_test
def test_model_startup(monkeypatch, vllm_runner, fresh_vllm_cache, spec):
    monkeypatch.setenv("VLLM_ENABLE_V1_MULTIPROCESSING", "0")
    monkeypatch.setenv("VLLM_DEEP_GEMM_WARMUP", "skip")

    # Cold start in a forked child (must fork before CUDA init).
    ctx = mp.get_context("fork")
    p = ctx.Process(target=_cold_start_model, args=(vllm_runner, spec))
    p.start()
    p.join()
    assert p.exitcode == 0, "Cold-start child failed"

    # Warm start — compiled artifacts loaded from disk cache.
    _check_model_run(vllm_runner, spec, is_cold_start=False)
```
**EN:** This pytest case verifies model startup. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as monkeypatch, vllm_runner, fresh_vllm_cache, spec. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 model startup 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 monkeypatch、vllm_runner、fresh_vllm_cache、spec 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `multiprocessing`
- `typing -> NamedTuple`
- `pytest`
- `torch._dynamo.utils -> counters`
- `vllm.envs`
- `vllm.compilation.counter -> compilation_counter`
- `vllm.config -> CompilationConfig, CompilationMode, CUDAGraphMode, PassConfig`
- `vllm.utils.torch_utils -> is_torch_equal_or_newer`
- `...utils -> fork_new_process_for_each_test`
