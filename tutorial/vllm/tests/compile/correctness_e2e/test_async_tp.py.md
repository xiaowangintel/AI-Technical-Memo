# test_async_tp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/correctness_e2e/test_async_tp.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / correctness_e2e / test_async_tp, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / correctness_e2e / test_async_tp 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-17)
```python
import json

import pytest

from tests.models.registry import HF_EXAMPLE_MODELS
from tests.utils import (
    compare_two_settings,
    create_new_process_for_each_test,
)
from vllm.config import (
    CompilationMode,
)
from vllm.platforms import current_platform
from vllm.utils.flashinfer import has_flashinfer
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as json, pytest; shared test helpers from tests.models.registry, tests.utils; and vLLM components like vllm.config, vllm.platforms, vllm.utils.flashinfer.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 json、pytest；共享测试辅助模块，例如 tests.models.registry、tests.utils；vLLM 内部组件，例如 vllm.config、vllm.platforms、vllm.utils.flashinfer。

### Constants and module state (lines 19-26)
```python
NVFP4_MODEL_ID = "nvidia/Llama-3.1-8B-Instruct-NVFP4"
NVFP4_HF_OVERRIDES = {
    "num_hidden_layers": 4,
    "hidden_size": 512,
    "intermediate_size": 800,
    "num_attention_heads": 4,
    "num_key_value_heads": 1,
}
```
**EN:** This block centralizes shared constants and parameter grids, including NVFP4_MODEL_ID, NVFP4_HF_OVERRIDES. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 NVFP4_MODEL_ID、NVFP4_HF_OVERRIDES。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_async_tp_pass_correctness` (lines 29-95)
```python
@create_new_process_for_each_test()
@pytest.mark.parametrize(
    "model_id",
    ["meta-llama/Llama-3.2-1B-Instruct", "RedHatAI/Meta-Llama-3.1-8B-Instruct-FP8"],
)
@pytest.mark.parametrize("tp_size", [2])
@pytest.mark.parametrize("async_tp_enabled", [True])
@pytest.mark.parametrize("distributed_backend", ["mp"])
@pytest.mark.parametrize("eager_mode", [False, True])
def test_async_tp_pass_correctness(
    model_id: str,
    tp_size: int,
    async_tp_enabled: bool,
    distributed_backend: str,
    eager_mode: bool,
    num_gpus_available: int,
    monkeypatch,
):
    # Disable FlashInfer FP8 scaled_mm kernel as it is incompatible with
    # async TP patterns. No-op on H100 (kernel requires CC >= 100).
    monkeypatch.setenv("VLLM_DISABLED_KERNELS", "FlashInferFP8ScaledMMLinearKernel")

    model_info = HF_EXAMPLE_MODELS.find_hf_info(model_id)
    model_info.check_transformers_version(on_fail="skip")
    model_info.check_available_online(on_fail="skip")

    pp_size = 1
    if num_gpus_available < tp_size:
        pytest.skip(f"Need at least {tp_size} x {pp_size} GPUs")

    common_args = [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "2048",
        "--max-num-seqs",
        "8",
    ]
    if eager_mode:
        common_args.append("--enforce-eager")

    compilation_config = {
        "mode": CompilationMode.VLLM_COMPILE,
        "compile_sizes": [2, 4, 8],
        "splitting_ops": [],
        "pass_config": {"fuse_gemm_comms": async_tp_enabled},
    }

    async_tp_args = [
        *common_args,
        "--tensor-parallel-size",
        str(tp_size),
        "--distributed-executor-backend",
        distributed_backend,
        "--compilation_config",
        json.dumps(compilation_config),
    ]

    tp_args = [
        *common_args,
        "--tensor-parallel-size",
        str(tp_size),
        "--distributed-executor-backend",
        "mp",
    ]

    compare_two_settings(model_id, async_tp_args, tp_args, method="generate")
```
**EN:** This pytest case verifies async tp pass correctness. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as model_id, tp_size, async_tp_enabled, distributed_backend. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 async tp pass correctness 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 model_id、tp_size、async_tp_enabled、distributed_backend 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_async_tp_pass_nvfp4_correctness` (lines 98-157)
```python
@create_new_process_for_each_test()
def test_async_tp_pass_nvfp4_correctness(num_gpus_available: int, monkeypatch):
    if (
        not current_platform.is_cuda()
        or not current_platform.is_device_capability_family(100)
    ):
        pytest.skip("NVFP4 requires Blackwell")
    if not has_flashinfer():
        pytest.skip("FlashInfer is required for the NVFP4 AsyncTP path")

    monkeypatch.setenv("VLLM_NVFP4_GEMM_BACKEND", "flashinfer-cutlass")

    tp_size = 2
    if num_gpus_available < tp_size:
        pytest.skip(f"Need at least {tp_size} GPUs")

    common_args = [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "2048",
        "--max-num-seqs",
        "8",
        "--load-format",
        "dummy",
        "--hf-overrides",
        json.dumps(NVFP4_HF_OVERRIDES),
    ]

    compilation_config = {
        "mode": CompilationMode.VLLM_COMPILE,
        "compile_sizes": [2, 4, 8],
        "splitting_ops": [],
        "pass_config": {
            "enable_sp": True,
            "fuse_gemm_comms": True,
            "fuse_allreduce_rms": False,
            "sp_min_token_num": 1,
        },
    }

    async_tp_args = [
        *common_args,
        "--tensor-parallel-size",
        str(tp_size),
        "--distributed-executor-backend",
        "mp",
        "--compilation_config",
        json.dumps(compilation_config),
    ]

    tp_args = [
        *common_args,
        "--tensor-parallel-size",
        str(tp_size),
        "--distributed-executor-backend",
        "mp",
    ]

    compare_two_settings(NVFP4_MODEL_ID, async_tp_args, tp_args, method="generate")
```
**EN:** This pytest case verifies async tp pass nvfp4 correctness. it consumes fixtures or inputs such as num_gpus_available, monkeypatch. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 async tp pass nvfp4 correctness 的行为。 它会使用诸如 num_gpus_available、monkeypatch 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `json`
- `pytest`
- `tests.models.registry -> HF_EXAMPLE_MODELS`
- `tests.utils -> compare_two_settings, create_new_process_for_each_test`
- `vllm.config -> CompilationMode`
- `vllm.platforms -> current_platform`
- `vllm.utils.flashinfer -> has_flashinfer`
