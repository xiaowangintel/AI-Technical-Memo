# test_hybrid.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/generation/test_hybrid.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers language-model behavior and model-facing behavior. The file defines 16 test(s), 0 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖语言模型行为与面向模型的行为。它定义了 16 个测试、0 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L16)
```python
from collections.abc import Callable

import pytest

from tests.models.registry import HF_EXAMPLE_MODELS
from tests.utils import multi_gpu_test
from vllm import LLM
from vllm.engine.arg_utils import EngineArgs
from vllm.platforms import current_platform
from vllm.sampling_params import SamplingParams
from vllm.v1.cudagraph_dispatcher import CudagraphDispatcher

from ...utils import check_logprobs_close, check_outputs_equal
```
**EN:** Imports standard-library modules such as `collections.abc.Callable`, third-party packages like `pytest`, project helpers such as `tests.models.registry.HF_EXAMPLE_MODELS`, `tests.utils.multi_gpu_test`, `vllm.LLM`.
**CN:** 导入标准库模块（如 `collections.abc.Callable`）、第三方包（如 `pytest`）、项目内辅助模块（如 `tests.models.registry.HF_EXAMPLE_MODELS`、`tests.utils.multi_gpu_test`、`vllm.LLM`）。

### Module setup / 模块级配置: pytestmark, APC_MULTIPLY_BY, SSM_MODELS (L19-L60)
```python
pytestmark = pytest.mark.hybrid_model

# NOTE: The first model in each list is taken as the primary model,
# meaning that it will be used in all tests in this file
# The rest of the models will only be tested by test_models

APC_MULTIPLY_BY = 300

SSM_MODELS = [
    "state-spaces/mamba-130m-hf",
    "tiiuae/falcon-mamba-tiny-dev",
    # mamba2-codestral in transformers is broken pending:
    # https://github.com/huggingface/transformers/pull/40861
    # "yujiepan/mamba2-codestral-v0.1-tiny-random",
]

# ... 20 lines omitted for brevity ...
]

# Avoid OOM
MAX_NUM_SEQS = 4

ATTN_BACKEND = "TRITON_ATTN" if current_platform.is_rocm() else "auto"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `pytestmark`, `APC_MULTIPLY_BY`, `SSM_MODELS`, `HYBRID_MODELS`, `FULL_CUDA_GRAPH_MODELS`, `FP32_STATE_MODELS`, `MAX_NUM_SEQS`, `ATTN_BACKEND`. Later helpers and tests reuse these values to keep scenarios concise. It also applies module-wide pytest markers.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `pytestmark`、`APC_MULTIPLY_BY`、`SSM_MODELS`、`HYBRID_MODELS`、`FULL_CUDA_GRAPH_MODELS`、`FP32_STATE_MODELS`、`MAX_NUM_SEQS`、`ATTN_BACKEND`。后续辅助函数和测试会复用这些值，以减少重复。 它还会应用模块级的 pytest 标记。

### Helper / 辅助函数: _set_conv_state_layout (L63-L68)
```python
def _set_conv_state_layout(monkeypatch, layout: str) -> None:
    """Set conv state layout env var and clear cache to pick up new value."""
    from vllm.model_executor.layers.mamba import mamba_utils

    monkeypatch.setenv("VLLM_SSM_CONV_STATE_LAYOUT", layout)
    mamba_utils.get_conv_state_layout.cache_clear()
```
**EN:** This helper encapsulates reusable logic in `_set_conv_state_layout`. Key inputs are `monkeypatch`, `layout`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个辅助函数将可复用逻辑封装在 `_set_conv_state_layout` 中。 关键输入包括 `monkeypatch`、`layout`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Test / 测试: test_models (L71-L107)
```python
@pytest.mark.parametrize("model", SSM_MODELS + HYBRID_MODELS)
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("num_logprobs", [5])
def test_models(
    hf_runner,
    vllm_runner,
    example_prompts,
    monkeypatch,
    model: str,
    max_tokens: int,
    num_logprobs: int,
) -> None:
    try:
        model_info = HF_EXAMPLE_MODELS.find_hf_info(model)
        model_info.check_available_online(on_fail="skip")
        model_info.check_transformers_version(on_fail="skip")
    except ValueError:
        pass
# ... 11 lines omitted for brevity ...
        )

    check_logprobs_close(
        outputs_0_lst=hf_outputs,
        outputs_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
    )
```
**EN:** This test validates `test_models`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `monkeypatch`, `model`, `max_tokens`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_models`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`monkeypatch`、`model`、`max_tokens`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Test / 测试: test_batching (L110-L149)
```python
@pytest.mark.parametrize("model", [SSM_MODELS[0], HYBRID_MODELS[0]])
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("num_logprobs", [5])
@pytest.mark.parametrize("conv_state_layout", ["SD", "DS"])
def test_batching(
    vllm_runner,
    example_prompts,
    monkeypatch,
    model: str,
    max_tokens: int,
    num_logprobs: int,
    conv_state_layout: str,
) -> None:
    try:
        model_info = HF_EXAMPLE_MODELS.find_hf_info(model)
        model_info.check_available_online(on_fail="skip")
        model_info.check_transformers_version(on_fail="skip")
    except ValueError:
# ... 14 lines omitted for brevity ...
        )

    check_logprobs_close(
        outputs_0_lst=for_loop_outputs,
        outputs_1_lst=batched_outputs,
        name_0="for_loop_vllm",
        name_1="batched_vllm",
    )
```
**EN:** This test validates `test_batching`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `example_prompts`, `monkeypatch`, `model`, `max_tokens`, `num_logprobs`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_batching`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`example_prompts`、`monkeypatch`、`model`、`max_tokens`、`num_logprobs`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Test / 测试: test_chunked_prefill_with_parallel_sampling (L152-L184)
```python
@pytest.mark.parametrize("model", [SSM_MODELS[0], HYBRID_MODELS[0]])
@pytest.mark.parametrize("max_tokens", [10])
@pytest.mark.parametrize("conv_state_layout", ["SD", "DS"])
def test_chunked_prefill_with_parallel_sampling(
    vllm_runner,
    example_prompts,
    monkeypatch,
    model: str,
    max_tokens: int,
    conv_state_layout: str,
) -> None:
    """
    Tests chunked prefill in conjunction with n > 1.

    In this case, prefill is populated with decoding tokens and
    we test that it doesn't fail.

    This test might fail if cache is not allocated correctly for n > 1
# ... 7 lines omitted for brevity ...
        model,
        enable_chunked_prefill=True,
        # forces prefill chunks with decoding
        max_num_batched_tokens=MAX_NUM_SEQS * 3,
        max_num_seqs=MAX_NUM_SEQS,
        attention_backend=ATTN_BACKEND,
    ) as vllm_model:
        vllm_model.generate(example_prompts, sampling_params)
```
**EN:** This test validates `test_chunked_prefill_with_parallel_sampling`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `example_prompts`, `monkeypatch`, `model`, `max_tokens`, `conv_state_layout`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_chunked_prefill_with_parallel_sampling`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`example_prompts`、`monkeypatch`、`model`、`max_tokens`、`conv_state_layout`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Test / 测试: test_mamba_cache_cg_padding (L187-L224)
```python
@pytest.mark.parametrize("model", [SSM_MODELS[0], HYBRID_MODELS[0]])
@pytest.mark.parametrize("max_tokens", [20])
@pytest.mark.parametrize("conv_state_layout", ["SD", "DS"])
def test_mamba_cache_cg_padding(
    vllm_runner,
    example_prompts,
    monkeypatch,
    model: str,
    max_tokens: int,
    conv_state_layout: str,
) -> None:
    """
    This test is for verifying that mamba cache is padded to CG captured
    batch size. If it's not, a torch RuntimeError will be raised because
    tensor dimensions aren't compatible.
    """
    _set_conv_state_layout(monkeypatch, conv_state_layout)

# ... 12 lines omitted for brevity ...
        with vllm_runner(model) as vllm_model:
            vllm_model.generate_greedy(example_prompts, max_tokens)
    except RuntimeError:
        pytest.fail(
            "Couldn't run batch size which is not equal to a Cuda Graph "
            "captured batch size. "
            "Could be related to mamba cache not padded correctly"
        )
```
**EN:** This test validates `test_mamba_cache_cg_padding`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `example_prompts`, `monkeypatch`, `model`, `max_tokens`, `conv_state_layout`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. It drives client-facing request creation through the API surface under test.
**CN:** 这个测试验证 `test_mamba_cache_cg_padding`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`example_prompts`、`monkeypatch`、`model`、`max_tokens`、`conv_state_layout`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_fail_upon_inc_requests_and_finished_requests_lt_available_blocks (L227-L249)
```python
@pytest.mark.parametrize("model", [SSM_MODELS[0], HYBRID_MODELS[0]])
def test_fail_upon_inc_requests_and_finished_requests_lt_available_blocks(
    vllm_runner,
    example_prompts,
    model: str,
) -> None:
    """
    This test is for verifying that the hybrid inner state management doesn't
    collapse in case where the number of incoming requests and
    finished_requests_ids is larger than the maximum mamba block capacity.

    This could generally happen due to the fact that hybrid does support
    statelessness mechanism where it can clean up new incoming requests in
    a single step.
    """
    try:
        with vllm_runner(model, max_num_seqs=MAX_NUM_SEQS) as vllm_model:
            vllm_model.generate_greedy([example_prompts[0]] * 100, 10)
    except ValueError:
        pytest.fail(
            "Hybrid inner state wasn't cleaned up properly between"
            "steps finished requests registered unnecessarily "
        )
```
**EN:** This test validates `test_fail_upon_inc_requests_and_finished_requests_lt_available_blocks`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `example_prompts`, `model`.
**CN:** 这个测试验证 `test_fail_upon_inc_requests_and_finished_requests_lt_available_blocks`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`example_prompts`、`model`。

### Test / 测试: test_state_cleanup (L252-L272)
```python
@pytest.mark.parametrize("model", [SSM_MODELS[0], HYBRID_MODELS[0]])
def test_state_cleanup(
    vllm_runner,
    example_prompts,
    model: str,
) -> None:
    """
    This test is for verifying that the Hybrid state is cleaned up between
    steps.

    If it's not cleaned, an error would be expected.
    """
    try:
        with vllm_runner(model, max_num_seqs=MAX_NUM_SEQS) as vllm_model:
            for _ in range(10):
                vllm_model.generate_greedy([example_prompts[0]] * 100, 1)
    except ValueError:
        pytest.fail(
            "Hybrid inner state wasn't cleaned up between states, "
            "could be related to finished_requests_ids"
        )
```
**EN:** This test validates `test_state_cleanup`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `example_prompts`, `model`.
**CN:** 这个测试验证 `test_state_cleanup`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`example_prompts`、`model`。

### Test / 测试: test_distributed_correctness (L275-L305)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize("model", [SSM_MODELS[0], HYBRID_MODELS[0]])
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("num_logprobs", [5])
def test_distributed_correctness(
    vllm_runner,
    example_prompts,
    model: str,
    max_tokens: int,
    num_logprobs: int,
) -> None:
    with vllm_runner(
        model, tensor_parallel_size=1, max_num_seqs=MAX_NUM_SEQS
    ) as vllm_model:
        vllm_outputs_tp_1 = vllm_model.generate_greedy_logprobs(
            example_prompts, max_tokens, num_logprobs
        )

# ... 5 lines omitted for brevity ...
        )

    check_logprobs_close(
        outputs_0_lst=vllm_outputs_tp_1,
        outputs_1_lst=vllm_outputs_tp_2,
        name_0="vllm_tp_1",
        name_1="vllm_tp_2",
    )
```
**EN:** This test validates `test_distributed_correctness`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `example_prompts`, `model`, `max_tokens`, `num_logprobs`.
**CN:** 这个测试验证 `test_distributed_correctness`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`example_prompts`、`model`、`max_tokens`、`num_logprobs`。

### Test / 测试: test_full_cuda_graph (L308-L344)
```python
@pytest.mark.parametrize("model", FULL_CUDA_GRAPH_MODELS)
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("num_logprobs", [5])
def test_full_cuda_graph(
    hf_runner,
    vllm_runner,
    example_prompts,
    monkeypatch,
    model: str,
    max_tokens: int,
    num_logprobs: int,
) -> None:
    try:
        model_info = HF_EXAMPLE_MODELS.find_hf_info(model)
        model_info.check_available_online(on_fail="skip")
        model_info.check_transformers_version(on_fail="skip")
    except ValueError:
        pass
# ... 11 lines omitted for brevity ...
        )

    check_logprobs_close(
        outputs_0_lst=hf_outputs,
        outputs_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
    )
```
**EN:** This test validates `test_full_cuda_graph`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `monkeypatch`, `model`, `max_tokens`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_full_cuda_graph`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`monkeypatch`、`model`、`max_tokens`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Test / 测试: test_fp32_cache_state (L347-L387)
```python
@pytest.mark.parametrize("model", FP32_STATE_MODELS)
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("num_logprobs", [5])
@pytest.mark.parametrize(
    "cache_dtype_param", ["mamba_ssm_cache_dtype", "mamba_cache_dtype"]
)
def test_fp32_cache_state(
    hf_runner,
    vllm_runner,
    example_prompts,
    monkeypatch,
    model: str,
    max_tokens: int,
    num_logprobs: int,
    cache_dtype_param: str,
) -> None:
    try:
        model_info = HF_EXAMPLE_MODELS.find_hf_info(model)
# ... 15 lines omitted for brevity ...
        )

    check_logprobs_close(
        outputs_0_lst=hf_outputs,
        outputs_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
    )
```
**EN:** This test validates `test_fp32_cache_state`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `monkeypatch`, `model`, `max_tokens`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_fp32_cache_state`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`monkeypatch`、`model`、`max_tokens`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Helper / 辅助函数: _get_vllm_runner_params (L391-L404)
```python
def _get_vllm_runner_params(
    model: str,
    max_model_len: int,
    tensor_parallel_size: int = 1,
):
    return {
        "model_name": model,
        "enable_chunked_prefill": True,
        "enable_prefix_caching": False,
        "max_model_len": max_model_len,
        "tensor_parallel_size": tensor_parallel_size,
        "gpu_memory_utilization": 0.4,
        "attention_backend": ATTN_BACKEND,
    }
```
**EN:** This helper encapsulates reusable logic in `_get_vllm_runner_params`. Key inputs are `model`, `max_model_len`, `tensor_parallel_size`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_vllm_runner_params` 中。 关键输入包括 `model`、`max_model_len`、`tensor_parallel_size`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _get_vLLM_output (L407-L428)
```python
def _get_vLLM_output(
    vllm_runner,
    kwargs,
    prompts,
    max_tokens,
    num_logprobs,
    num_repetitions=1,
    vllm_model=None,
):
    outs = []
    if vllm_model is None:
        vllm_model = vllm_runner(**kwargs)
    for _ in range(num_repetitions):
        if num_logprobs < 0:
            vllm_output = vllm_model.generate_greedy(prompts, max_tokens)
        else:
            vllm_output = vllm_model.generate_greedy_logprobs(
                prompts, max_tokens, num_logprobs
            )
        outs.append(vllm_output)

    return outs, vllm_model
```
**EN:** This helper encapsulates reusable logic in `_get_vLLM_output`. Key inputs are `vllm_runner`, `kwargs`, `prompts`, `max_tokens`, `num_logprobs`, `num_repetitions`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_vLLM_output` 中。 关键输入包括 `vllm_runner`、`kwargs`、`prompts`、`max_tokens`、`num_logprobs`、`num_repetitions`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_apc_single_prompt (L431-L492)
```python
@pytest.mark.parametrize("model", [HYBRID_MODELS[0], HYBRID_MODELS[3]])
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("n_repetitions", [2])
# If num_logprobs is set to -1, then the stringent version
# of the test is executed using `check_outputs_equal`
# instead of `check_logprobs_close`
@pytest.mark.parametrize("num_logprobs", [5])
@pytest.mark.parametrize("tensor_parallel_size", [1])
def test_apc_single_prompt(
    hf_runner,
    vllm_runner,
    example_prompts,
    monkeypatch,
    model: str,
    max_tokens: int,
    n_repetitions: int,
    num_logprobs: int,
    tensor_parallel_size: int,
# ... 36 lines omitted for brevity ...
        # In the second repetition, these caches are reused

        compare_operator(
            outputs_0_lst=vllm_outputs_no_cache[0],
            outputs_1_lst=vllm_outputs_cache_itn,
            name_0="vllm_no_cache",
            name_1=f"vllm_cache_it_{r_idx + 1}",
        )
```
**EN:** This test validates `test_apc_single_prompt`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `monkeypatch`, `model`, `max_tokens`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_apc_single_prompt`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`monkeypatch`、`model`、`max_tokens`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Test / 测试: test_apc_single_prompt_block_align_alignment (L495-L573)
```python
@pytest.mark.parametrize("model", [HYBRID_MODELS[0], HYBRID_MODELS[3]])
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("n_repetitions", [2])
# If num_logprobs is set to -1, then the stringent version
# of the test is executed using `check_outputs_equal`
# instead of `check_logprobs_close`
@pytest.mark.parametrize("num_logprobs", [5])
@pytest.mark.parametrize("tensor_parallel_size", [1])
def test_apc_single_prompt_block_align_alignment(
    hf_runner,
    vllm_runner,
    example_prompts,
    monkeypatch,
    model: str,
    max_tokens: int,
    n_repetitions: int,
    num_logprobs: int,
    tensor_parallel_size: int,
# ... 53 lines omitted for brevity ...
            # In the second repetition, these caches are reused

            compare_operator(
                outputs_0_lst=vllm_outputs_no_cache[0],
                outputs_1_lst=vllm_outputs_cache_itn,
                name_0="vllm_no_cache",
                name_1=f"vllm_cache_it_{r_idx + 1}",
            )
```
**EN:** This test validates `test_apc_single_prompt_block_align_alignment`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `monkeypatch`, `model`, `max_tokens`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_apc_single_prompt_block_align_alignment`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`monkeypatch`、`model`、`max_tokens`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Test / 测试: test_apc_multiple_prompts_all_cached_outputs (L576-L642)
```python
@pytest.mark.parametrize("model", [HYBRID_MODELS[0], HYBRID_MODELS[3]])
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("n_repetitions", [2])
# If num_logprobs is set to -1, then the stringent version
# of the test is executed using `check_outputs_equal`
# instead of `check_logprobs_close`
@pytest.mark.parametrize("num_logprobs", [5])
@pytest.mark.parametrize("tensor_parallel_size", [1])
def test_apc_multiple_prompts_all_cached_outputs(
    hf_runner,
    vllm_runner,
    example_prompts,
    monkeypatch,
    model: str,
    max_tokens: int,
    n_repetitions: int,
    num_logprobs: int,
    tensor_parallel_size: int,
# ... 41 lines omitted for brevity ...
        # In the second repetition, these caches are reused

        compare_operator(
            outputs_0_lst=vllm_outputs_no_cache[0],
            outputs_1_lst=vllm_outputs_cache_itn,
            name_0="vllm_no_cache",
            name_1=f"vllm_cache_it_{r_idx + 1}",
        )
```
**EN:** This test validates `test_apc_multiple_prompts_all_cached_outputs`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `monkeypatch`, `model`, `max_tokens`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_apc_multiple_prompts_all_cached_outputs`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`monkeypatch`、`model`、`max_tokens`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Test / 测试: test_apc_multiple_prompts_block_align_alignment (L645-L727)
```python
@pytest.mark.parametrize("model", [HYBRID_MODELS[0], HYBRID_MODELS[3]])
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("n_repetitions", [2])
# If num_logprobs is set to -1, then the stringent version
# of the test is executed using `check_outputs_equal`
# instead of `check_logprobs_close`
@pytest.mark.parametrize("num_logprobs", [5])
@pytest.mark.parametrize("tensor_parallel_size", [1])
def test_apc_multiple_prompts_block_align_alignment(
    hf_runner,
    vllm_runner,
    example_prompts,
    monkeypatch,
    model: str,
    max_tokens: int,
    n_repetitions: int,
    num_logprobs: int,
    tensor_parallel_size: int,
# ... 57 lines omitted for brevity ...
            # In the second repetition, these caches are reused

            compare_operator(
                outputs_0_lst=vllm_outputs_no_cache[0],
                outputs_1_lst=vllm_outputs_cache_itn,
                name_0="vllm_no_cache",
                name_1=f"vllm_cache_it_{r_idx + 1}",
            )
```
**EN:** This test validates `test_apc_multiple_prompts_block_align_alignment`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `monkeypatch`, `model`, `max_tokens`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_apc_multiple_prompts_block_align_alignment`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`monkeypatch`、`model`、`max_tokens`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Test / 测试: test_apc_multiple_prompts_partial_cached_outputs (L730-L805)
```python
@pytest.mark.parametrize("model", [HYBRID_MODELS[0], HYBRID_MODELS[3]])
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("n_repetitions", [2])
# If num_logprobs is set to -1, then the stringent version
# of the test is executed using `check_outputs_equal`
# instead of `check_logprobs_close`
@pytest.mark.parametrize("num_logprobs", [5])
@pytest.mark.parametrize("tensor_parallel_size", [1])
def test_apc_multiple_prompts_partial_cached_outputs(
    hf_runner,
    vllm_runner,
    example_prompts,
    monkeypatch,
    model: str,
    max_tokens: int,
    n_repetitions: int,
    num_logprobs: int,
    tensor_parallel_size: int,
# ... 50 lines omitted for brevity ...
        # In the second repetition, these caches are reused

        compare_operator(
            outputs_0_lst=vllm_outputs_no_cache[0],
            outputs_1_lst=vllm_outputs_cache_itn,
            name_0="vllm_no_cache",
            name_1=f"vllm_cache_it_{r_idx + 1}",
        )
```
**EN:** This test validates `test_apc_multiple_prompts_partial_cached_outputs`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `monkeypatch`, `model`, `max_tokens`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_apc_multiple_prompts_partial_cached_outputs`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`monkeypatch`、`model`、`max_tokens`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Test / 测试: test_same_mamba_output_apc_on_vs_off (L809-L859)
```python
@pytest.mark.parametrize("model", ["tiiuae/falcon-mamba-7b"])
def test_same_mamba_output_apc_on_vs_off(
    vllm_runner,
    model: str,
) -> None:
    num_logprobs = 5
    prompts = [
        "hello what is one plus one what is one plus one what is one plus one the answer is",  # noqa: E501
        "hello what is one plus one what is one plus one what is one plus one the answer is",  # noqa: E501
    ]
    max_tokens = 20
    max_model_len = max(len(p) for p in prompts) + max_tokens + 64

    base_kwargs = _get_vllm_runner_params(model, max_model_len)
    base_kwargs.update(
        enforce_eager=True, block_size=16, seed=42, gpu_memory_utilization=0.8
    )

# ... 25 lines omitted for brevity ...
        )

    check_logprobs_close(
        outputs_0_lst=outputs_no_apc[0],
        outputs_1_lst=outputs_with_apc[0],
        name_0="vllm_no_apc",
        name_1="vllm_with_apc",
    )
```
**EN:** This test validates `test_same_mamba_output_apc_on_vs_off`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`.
**CN:** 这个测试验证 `test_same_mamba_output_apc_on_vs_off`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`。

### Test / 测试: test_apc_common_prefix_same_batch (L864-L887)
```python
@pytest.mark.parametrize("model", ["tiiuae/falcon-mamba-7b"])
def test_apc_common_prefix_same_batch(
    model: str,
    monkeypatch,
) -> None:
    # Required to put the two requests in the same batch
    monkeypatch.setenv("VLLM_ENABLE_V1_MULTIPROCESSING", "0")
    llm = LLM(
        model=model,
        enforce_eager=True,
        block_size=16,
        mamba_block_size=16,
        enable_prefix_caching=True,
        seed=42,
        attention_backend=ATTN_BACKEND,
    )
    prompts = [
        "hello what is one plus one what is one plus one what is one plus one the answer is",  # noqa: E501
        "hello what is one plus one what is one plus one what is one plus one the answer is",  # noqa: E501
    ]
    sampling_params = SamplingParams(temperature=0.0, max_tokens=20)
    outputs = llm.generate(prompts, sampling_params)
    for output in outputs:
        assert "two" in output.outputs[0].text
```
**EN:** This test validates `test_apc_common_prefix_same_batch`. It uses parameterization over `model`. Key inputs are `model`, `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. It touches the core vLLM initialization or engine path directly. The main assertion is `'two' in output.outputs[0].text`.
**CN:** 这个测试验证 `test_apc_common_prefix_same_batch`。 它通过参数化组合 `model`。 关键输入包括 `model`、`monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 核心断言是 `'two' in output.outputs[0].text`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Callable`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `tests.models.registry.HF_EXAMPLE_MODELS`, `tests.utils.multi_gpu_test`, `vllm.LLM`, `vllm.engine.arg_utils.EngineArgs`, `vllm.model_executor.layers.mamba.mamba_utils`, `vllm.platforms.current_platform`, `vllm.sampling_params.SamplingParams`, `vllm.v1.cudagraph_dispatcher.CudagraphDispatcher`
- **Local relative imports / 本地相对导入**: `...utils.check_logprobs_close`, `...utils.check_outputs_equal`
