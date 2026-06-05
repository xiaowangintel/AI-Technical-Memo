# test_tp2_async_tp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/fusions_e2e/test_tp2_async_tp.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / fusions_e2e / test_tp2_async_tp, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / fusions_e2e / test_tp2_async_tp 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-26)
```python
from collections.abc import Callable

import pytest

from vllm.config import PassConfig
from vllm.platforms import current_platform

from ...utils import multi_gpu_test
from .common import (
    INDUCTOR_GRAPH_PARTITION,
    AttentionBackendCase,
    Matches,
    custom_ops_combos,
    is_blackwell,
)
from .models import (
    FLASHINFER_ATTN,
    TRITON_ATTN,
    llama3_8b,
    llama3_8b_fp4,
    llama3_8b_fp8,
    llama4_scout_fp8,
    qwen3_a3b,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as collections.abc, pytest; shared test helpers from ...utils, .common, .models; and vLLM components like vllm.config, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 collections.abc、pytest；共享测试辅助模块，例如 ...utils、.common、.models；vLLM 内部组件，例如 vllm.config、vllm.platforms。

### Constants and module state (lines 28-28)
```python
pytestmark = pytest.mark.skipif(not current_platform.is_cuda(), reason="Only test CUDA")
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Function `test_tp2_async_tp_fp8_fusions` (lines 31-92)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize(
    "model_name, matches_fn, model_kwargs, hf_overrides",
    [llama3_8b_fp8, llama4_scout_fp8],
)
@pytest.mark.parametrize("attn_backend", [TRITON_ATTN, FLASHINFER_ATTN])
@pytest.mark.parametrize("n_layers", [4])
@pytest.mark.parametrize("custom_ops", custom_ops_combos("quant_fp8", "rms_norm"))
@pytest.mark.parametrize("inductor_graph_partition", INDUCTOR_GRAPH_PARTITION)
def test_tp2_async_tp_fp8_fusions(
    model_name: str,
    matches_fn: Callable[[int], Matches],
    model_kwargs: dict,
    hf_overrides: Callable[[int], dict],
    attn_backend: AttentionBackendCase,
    n_layers: int,
    custom_ops: str,
    inductor_graph_partition: bool,
    run_e2e_fusion_test,
):
    matches = matches_fn(n_layers)

    # Reduce size of model and skip weight loading time
    model_kwargs["hf_overrides"] = hf_overrides(n_layers)
    model_kwargs["load_format"] = "dummy"
    model_kwargs["max_model_len"] = 1024
    model_kwargs["kernel_config"] = {"enable_flashinfer_autotune": False}

    compilation_config = dict(
        use_inductor_graph_partition=inductor_graph_partition,
        custom_ops=custom_ops.split(","),
        pass_config=PassConfig(
            fuse_norm_quant=True,
            fuse_act_quant=True,
            fuse_attn_quant=True,
            enable_qk_norm_rope_fusion=True,
            enable_sp=True,
            fuse_gemm_comms=True,
            fuse_allreduce_rms=False,
            # Override threshold for testing (models have small hidden_size)
            sp_min_token_num=512,
        ),
    )

    matches_check = [
        "rms_quant_fusion",
        "act_quant_fusion",
        "norm_rope_fusion",
        "attn_quant_fusion",
        "sequence_parallel",
        "async_tp",
    ]

    run_e2e_fusion_test(
        model_name,
        matches,
        model_kwargs,
        attn_backend,
        compilation_config,
        matches_check,
        tp_size=2,
    )
```
**EN:** This pytest case verifies tp2 async tp FP8 fusions. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as model_name, matches_fn, model_kwargs, hf_overrides.
**CN:** 该 pytest 用例验证 tp2 async tp FP8 fusions 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 model_name、matches_fn、model_kwargs、hf_overrides 等 fixture 或输入。

### Function `test_tp2_async_tp_nvfp4_fusions` (lines 95-155)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize(
    "model_name, matches_fn, model_kwargs, hf_overrides",
    [llama3_8b_fp4],
)
@pytest.mark.parametrize("attn_backend", [FLASHINFER_ATTN])
@pytest.mark.parametrize("n_layers", [4])
@pytest.mark.parametrize("custom_ops", custom_ops_combos("rms_norm"))
@pytest.mark.parametrize("inductor_graph_partition", INDUCTOR_GRAPH_PARTITION)
@pytest.mark.skipif(not is_blackwell(), reason="Blackwell required for fp4")
@pytest.mark.skipif(not current_platform.is_cuda(), reason="Only test CUDA")
def test_tp2_async_tp_nvfp4_fusions(
    model_name: str,
    matches_fn: Callable[[int], Matches],
    model_kwargs: dict,
    hf_overrides: Callable[[int], dict],
    attn_backend: AttentionBackendCase,
    n_layers: int,
    custom_ops: str,
    inductor_graph_partition: bool,
    run_e2e_fusion_test,
):
    # NVFP4 currently wires the all-gather + GEMM path only.
    matches = matches_fn(n_layers)._replace(async_tp=n_layers * 2)

    # Reduce size of model and skip weight loading time
    model_kwargs["hf_overrides"] = hf_overrides(n_layers)
    model_kwargs["load_format"] = "dummy"
    model_kwargs["max_model_len"] = 1024
    model_kwargs["kernel_config"] = {"enable_flashinfer_autotune": False}

    compilation_config = dict(
        use_inductor_graph_partition=inductor_graph_partition,
        custom_ops=custom_ops.split(","),
        pass_config=PassConfig(
            fuse_act_quant=True,
            fuse_attn_quant=True,
            enable_sp=True,
            fuse_gemm_comms=True,
            fuse_allreduce_rms=False,
            # Override threshold for testing (models have small hidden_size)
            sp_min_token_num=512,
        ),
    )

    matches_check = [
        "act_quant_fusion",
        "attn_quant_fusion",
        "sequence_parallel",
        "async_tp",
    ]

    run_e2e_fusion_test(
        model_name,
        matches,
        model_kwargs,
        attn_backend,
        compilation_config,
        matches_check,
        tp_size=2,
    )
```
**EN:** This pytest case verifies tp2 async tp nvfp4 fusions. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as model_name, matches_fn, model_kwargs, hf_overrides. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 tp2 async tp nvfp4 fusions 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 model_name、matches_fn、model_kwargs、hf_overrides 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_tp2_async_tp_fusions` (lines 158-213)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize(
    "model_name, matches_fn, model_kwargs, hf_overrides",
    [llama3_8b, qwen3_a3b],
)
@pytest.mark.parametrize("attn_backend", [TRITON_ATTN])
@pytest.mark.parametrize("n_layers", [4])
@pytest.mark.parametrize("custom_ops", custom_ops_combos("rms_norm"))
@pytest.mark.parametrize("inductor_graph_partition", INDUCTOR_GRAPH_PARTITION)
def test_tp2_async_tp_fusions(
    model_name: str,
    matches_fn: Callable[[int], Matches],
    model_kwargs: dict,
    hf_overrides: Callable[[int], dict],
    attn_backend: AttentionBackendCase,
    n_layers: int,
    custom_ops: str,
    inductor_graph_partition: bool,
    run_e2e_fusion_test,
):
    matches = matches_fn(n_layers)

    # Reduce size of model and skip weight loading time
    model_kwargs["hf_overrides"] = hf_overrides(n_layers)
    model_kwargs["load_format"] = "dummy"
    model_kwargs["max_model_len"] = 1024
    model_kwargs["kernel_config"] = {"enable_flashinfer_autotune": False}

    compilation_config = dict(
        use_inductor_graph_partition=inductor_graph_partition,
        custom_ops=custom_ops.split(","),
        pass_config=PassConfig(
            enable_qk_norm_rope_fusion=True,
            enable_sp=True,
            fuse_gemm_comms=True,
            fuse_allreduce_rms=False,
            # Override threshold for testing (models have small hidden_size)
            sp_min_token_num=512,
        ),
    )

    matches_check = [
        "norm_rope_fusion",
        "sequence_parallel",
        "async_tp",
    ]

    run_e2e_fusion_test(
        model_name,
        matches,
        model_kwargs,
        attn_backend,
        compilation_config,
        matches_check,
        tp_size=2,
    )
```
**EN:** This pytest case verifies tp2 async tp fusions. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as model_name, matches_fn, model_kwargs, hf_overrides.
**CN:** 该 pytest 用例验证 tp2 async tp fusions 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 model_name、matches_fn、model_kwargs、hf_overrides 等 fixture 或输入。

### Function `test_tp2_sp_ar_rms_fp8_fusions` (lines 216-277)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize(
    "model_name, matches_fn, model_kwargs, hf_overrides",
    [llama3_8b_fp8, llama4_scout_fp8],
)
@pytest.mark.parametrize("attn_backend", [TRITON_ATTN, FLASHINFER_ATTN])
@pytest.mark.parametrize("n_layers", [4])
@pytest.mark.parametrize("custom_ops", custom_ops_combos("quant_fp8", "rms_norm"))
@pytest.mark.parametrize("inductor_graph_partition", INDUCTOR_GRAPH_PARTITION)
def test_tp2_sp_ar_rms_fp8_fusions(
    model_name: str,
    matches_fn: Callable[[int], Matches],
    model_kwargs: dict,
    hf_overrides: Callable[[int], dict],
    attn_backend: AttentionBackendCase,
    n_layers: int,
    custom_ops: str,
    inductor_graph_partition: bool,
    run_e2e_fusion_test,
):
    matches = matches_fn(n_layers)

    # Reduce size of model and skip weight loading time
    model_kwargs["hf_overrides"] = hf_overrides(n_layers)
    model_kwargs["load_format"] = "dummy"
    model_kwargs["max_model_len"] = 1024

    compilation_config = dict(
        use_inductor_graph_partition=inductor_graph_partition,
        custom_ops=custom_ops.split(","),
        pass_config=PassConfig(
            fuse_norm_quant=True,
            fuse_act_quant=True,
            fuse_attn_quant=True,
            enable_qk_norm_rope_fusion=True,
            enable_sp=True,
            fuse_gemm_comms=True,
            fuse_allreduce_rms=True,
            # Override threshold for testing (models have small hidden_size)
            sp_min_token_num=512,
        ),
    )

    matches_check = [
        "rms_quant_fusion",
        "act_quant_fusion",
        "norm_rope_fusion",
        "attn_quant_fusion",
        "ar_rms_fusion",
        "sequence_parallel",
        "async_tp",
    ]

    run_e2e_fusion_test(
        model_name,
        matches,
        model_kwargs,
        attn_backend,
        compilation_config,
        matches_check,
        tp_size=2,
    )
```
**EN:** This pytest case verifies tp2 sp ar rms FP8 fusions. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as model_name, matches_fn, model_kwargs, hf_overrides.
**CN:** 该 pytest 用例验证 tp2 sp ar rms FP8 fusions 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 model_name、matches_fn、model_kwargs、hf_overrides 等 fixture 或输入。

### Function `test_tp2_sp_ar_rms_fusions` (lines 280-335)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize(
    "model_name, matches_fn, model_kwargs, hf_overrides",
    [llama3_8b, qwen3_a3b],
)
@pytest.mark.parametrize("attn_backend", [TRITON_ATTN])
@pytest.mark.parametrize("n_layers", [4])
@pytest.mark.parametrize("custom_ops", custom_ops_combos("rms_norm"))
@pytest.mark.parametrize("inductor_graph_partition", INDUCTOR_GRAPH_PARTITION)
def test_tp2_sp_ar_rms_fusions(
    model_name: str,
    matches_fn: Callable[[int], Matches],
    model_kwargs: dict,
    hf_overrides: Callable[[int], dict],
    attn_backend: AttentionBackendCase,
    n_layers: int,
    custom_ops: str,
    inductor_graph_partition: bool,
    run_e2e_fusion_test,
):
    matches = matches_fn(n_layers)

    # Reduce size of model and skip weight loading time
    model_kwargs["hf_overrides"] = hf_overrides(n_layers)
    model_kwargs["load_format"] = "dummy"
    model_kwargs["max_model_len"] = 1024

    compilation_config = dict(
        use_inductor_graph_partition=inductor_graph_partition,
        custom_ops=custom_ops.split(","),
        pass_config=PassConfig(
            enable_qk_norm_rope_fusion=True,
            enable_sp=True,
            fuse_gemm_comms=True,
            fuse_allreduce_rms=True,
            # Override threshold for testing (models have small hidden_size)
            sp_min_token_num=512,
        ),
    )

    matches_check = [
        "norm_rope_fusion",
        "ar_rms_fusion",
        "sequence_parallel",
        "async_tp",
    ]

    run_e2e_fusion_test(
        model_name,
        matches,
        model_kwargs,
        attn_backend,
        compilation_config,
        matches_check,
        tp_size=2,
    )
```
**EN:** This pytest case verifies tp2 sp ar rms fusions. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as model_name, matches_fn, model_kwargs, hf_overrides.
**CN:** 该 pytest 用例验证 tp2 sp ar rms fusions 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 model_name、matches_fn、model_kwargs、hf_overrides 等 fixture 或输入。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `collections.abc -> Callable`
- `pytest`
- `vllm.config -> PassConfig`
- `vllm.platforms -> current_platform`
- `...utils -> multi_gpu_test`
- `.common -> INDUCTOR_GRAPH_PARTITION, AttentionBackendCase, Matches, custom_ops_combos, is_blackwell`
- `.models -> FLASHINFER_ATTN, TRITON_ATTN, llama3_8b, llama3_8b_fp4, llama3_8b_fp8, llama4_scout_fp8, qwen3_a3b`
