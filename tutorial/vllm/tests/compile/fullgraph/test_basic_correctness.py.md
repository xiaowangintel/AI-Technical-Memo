# test_basic_correctness.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/fullgraph/test_basic_correctness.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / fullgraph / test_basic_correctness, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / fullgraph / test_basic_correctness 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-10)
```python
import dataclasses

import pytest

from vllm.config import CompilationMode
from vllm.platforms import current_platform

from ...utils import compare_all_settings
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as dataclasses, pytest; shared test helpers from ...utils; and vLLM components like vllm.config, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 dataclasses、pytest；共享测试辅助模块，例如 ...utils；vLLM 内部组件，例如 vllm.config、vllm.platforms。

### Constants and module state (lines 12-12)
```python
ATTN_BACKEND = "FLASH_ATTN" if not current_platform.is_rocm() else "ROCM_ATTN"
```
**EN:** This block centralizes shared constants and parameter grids, including ATTN_BACKEND. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 ATTN_BACKEND。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `TestSetting` (lines 15-22)
```python
@dataclasses.dataclass
class TestSetting:
    model: str
    model_args: list[str]
    pp_size: int
    tp_size: int
    attn_backend: str
    method: str
```
**EN:** This dataclass packages the fields needed to describe TestSetting. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 TestSetting 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Function `test_compile_correctness` (lines 27-161)
```python
@pytest.mark.parametrize(
    "test_setting",
    [
        # basic llama model
        TestSetting(
            model="meta-llama/Llama-3.2-1B-Instruct",
            model_args=["--max-model-len", "2048"],
            pp_size=2,
            tp_size=2,
            attn_backend=ATTN_BACKEND,
            method="generate",
        ),
        # llama model with quantization
        TestSetting(
            model="TheBloke/TinyLlama-1.1B-Chat-v0.3-GPTQ",
            model_args=["--quantization", "gptq", "--max-model-len", "2048"],
            pp_size=1,
            tp_size=1,
            attn_backend=ATTN_BACKEND,
            method="generate",
        ),
        # MoE model
        TestSetting(
            model="ibm/PowerMoE-3b",
            model_args=["--max-model-len", "2048"],
            pp_size=1,
            tp_size=2,
            attn_backend=ATTN_BACKEND,
            method="generate",
        ),
        # embedding model
        TestSetting(
            model="BAAI/bge-multilingual-gemma2",
            model_args=[
                "--runner",
                "pooling",
                "--dtype",
                "bfloat16",
                "--max-model-len",
                "2048",
# ... excerpt ...
        "-cc.cudagraph_mode=none",
        f"--attention-backend={attn_backend}",
    ]

    all_args: list[list[str]] = []
    all_envs: list[dict[str, str] | None] = []

    for comp_mode in [
        CompilationMode.STOCK_TORCH_COMPILE,
        CompilationMode.DYNAMO_TRACE_ONCE,
        CompilationMode.VLLM_COMPILE,
    ]:
        for mode in [CompilationMode.NONE, comp_mode]:
            all_args.append(
                final_args + [f"-cc.mode={mode.name}", "-cc.backend=inductor"]
            )
            all_envs.append({})

        # inductor will change the output, so we only compare if the output
        # is close, not exactly the same.
        compare_all_settings(
            model,
            all_args,
            all_envs,
            method=method if method != "generate" else "generate_close",
        )
        all_envs.clear()
        all_args.clear()

    for mode in [
        CompilationMode.NONE,
        CompilationMode.STOCK_TORCH_COMPILE,
        CompilationMode.DYNAMO_TRACE_ONCE,
        CompilationMode.VLLM_COMPILE,
    ]:
        all_args.append(final_args + [f"-cc.mode={mode.name}", "-cc.backend=eager"])
        all_envs.append({})

    compare_all_settings(model, all_args, all_envs, method=method)
```
**EN:** This pytest case verifies compile correctness. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as test_setting. unsupported hardware, backend, or configuration combinations are skipped early. it compares behavior across multiple compilation settings. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 compile correctness 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 test_setting 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；它会比较多种编译设置下的行为差异。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `dataclasses`
- `pytest`
- `vllm.config -> CompilationMode`
- `vllm.platforms -> current_platform`
- `...utils -> compare_all_settings`
