# conftest.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/fusions_e2e/conftest.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest fixture module for compile / fusions_e2e / conftest, providing reusable setup, factories, and environment controls. / compile / fusions_e2e / conftest 对应的 pytest fixture 模块，提供可复用的初始化、工厂函数和环境控制。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-12)
```python
import logging
from collections import defaultdict

import pytest
import regex as re

from vllm import LLM, SamplingParams
from vllm.config import CompilationConfig, CompilationMode, CUDAGraphMode

from .common import FUSION_LOG_PATTERNS, AttentionBackendCase, Matches
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as logging, collections, pytest, regex; shared test helpers from .common; and vLLM components like vllm, vllm.config.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 logging、collections、pytest、regex；共享测试辅助模块，例如 .common；vLLM 内部组件，例如 vllm、vllm.config。

### Function `run_model` (lines 15-64)
```python
def run_model(compile_config: int | CompilationConfig, model: str, **model_kwargs):
    """Run a model with the given compilation config for E2E fusion tests."""
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

    # Get the compile ranges endpoints after vllm config post init
    # in order to compute compile ranges correctly
    compilation_config.compile_ranges_endpoints = (
        llm.llm_engine.vllm_config.compilation_config.compile_ranges_endpoints
    )

    # Fetch match table from each worker via RPC and sum across workers.
    worker_tables = llm.llm_engine.engine_core.collective_rpc(
        "get_compilation_match_table"
    )
    combined: defaultdict[str, int] = defaultdict(int)
    for table in worker_tables:
        for k, v in table.items():
            combined[k] += v
    return dict(combined)
```
**EN:** This helper function implements the shared logic for run model. the code exercises an end-to-end vLLM execution path instead of only isolated tensor math.
**CN:** 该辅助函数实现了 run model 所需的共享逻辑。 这里测试的是端到端 vLLM 执行路径，而不只是孤立的张量计算。

### Function `run_e2e_fusion_test` (lines 67-304)
```python
@pytest.fixture
def run_e2e_fusion_test(monkeypatch, caplog_mp_spawn):
    def run(
        model_name: str,
        matches: Matches,
        model_kwargs: dict,
        attn_backend: AttentionBackendCase,
        compilation_config: dict,
        matches_check: list[str],
        use_deepgemm: bool = False,
        use_aiter: bool = False,
        tp_size: int = 1,
    ):
        monkeypatch.setenv("VLLM_USE_DEEP_GEMM", "1" if use_deepgemm else "0")
        monkeypatch.setenv("VLLM_ROCM_USE_AITER", "1" if use_aiter else "0")
        from vllm._aiter_ops import rocm_aiter_ops

        rocm_aiter_ops.refresh_env_variables()

        # Filter here to reduce code duplication
        backend_name = attn_backend.backend.name.lower()
        requires_mla = "deepseek" in model_name.lower()
        is_mla = "mla" in backend_name
        # DeepSeek V3.2 uses sparse MLA
        requires_sparse = "v3.2" in model_name.lower()
        is_sparse = "sparse" in backend_name

        if requires_mla != is_mla or requires_sparse != is_sparse:
            pytest.skip(
                f"Incompatible model '{model_name}' and "
                f"attention backend '{attn_backend.backend.name}'"
            )

        if attn_backend.backend.name == "FLASHINFER":
            from vllm.utils.flashinfer import supports_trtllm_attention

            if not supports_trtllm_attention():
                matches = matches._replace(attn_quant_fusion=0)

        # Disable, compile cache to make sure custom passes run.
# ... excerpt ...
                ) + match_table.get("mla_attn_quant_fusion", 0)
                assert actual_match == expected_matches * n_expected, (
                    f"Could not find {expected_matches * n_expected} "
                    f"{match_name} (found {actual_match})."
                )
            else:
                expected_matches_list = [expected_matches] * n_expected
                assert sorted(log_matches) == expected_matches_list, (
                    f"{match_name} expected: {expected_matches_list}, "
                    f"found: {sorted(log_matches)}"
                )

            if match_name == "ar_rms_fusion" and num_compile_ranges >= 2:
                log_matches = re.findall(
                    r"pass_manager.py:\d+] Skipping "
                    r".*AllReduceFusionPass.* with compile range",
                    log_holder.text,
                )

                n_expected = tp_size * (num_compile_ranges - num_ranges_activated)
                assert len(log_matches) == n_expected, (
                    f'Could not find {n_expected} "Skipping AllReduceFusionPass" '
                    f"(found {len(log_matches)}) in:\n {log_holder.text}"
                )

            if match_name == "sequence_parallel" and num_compile_ranges >= 2:
                log_matches = re.findall(
                    r"pass_manager.py:\d+] Skipping "
                    r".*SequenceParallelismPass.* with compile range",
                    log_holder.text,
                )

                n_expected = tp_size * (num_compile_ranges - num_ranges_activated)
                assert len(log_matches) == n_expected, (
                    f'Could not find {n_expected} "Skipping SequenceParallelismPass" '
                    f"(found {len(log_matches)}) in:\n {log_holder.text}"
                )

    return run
```
**EN:** This fixture prepares reusable state for run e2e fusion test. it consumes fixtures or inputs such as monkeypatch, caplog_mp_spawn. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 fixture 为 run e2e fusion test 准备可复用的测试状态。 它会使用诸如 monkeypatch、caplog_mp_spawn 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。
- **Reusable fixtures / 可复用 fixture:** Fixtures package common setup so each test can focus on the scenario under study. / fixture 封装公共准备逻辑，让每个测试更专注于待验证场景。
- **Shared test utilities / 共享测试工具:** The module factors repeated setup or reference logic out of the individual test files. / 该模块把重复的初始化或参考逻辑从单个测试文件中抽离出来复用。

## Dependencies / 依赖关系
- `logging`
- `collections -> defaultdict`
- `pytest`
- `regex`
- `vllm -> LLM, SamplingParams`
- `vllm.config -> CompilationConfig, CompilationMode, CUDAGraphMode`
- `.common -> FUSION_LOG_PATTERNS, AttentionBackendCase, Matches`
