# test_beam_search.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/samplers/test_beam_search.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Compare the outputs of HF and vLLM when using beam search. / 该文件主要围绕 Beam Search 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-40)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Compare the outputs of HF and vLLM when using beam search.

Run `pytest tests/samplers/test_beam_search.py`.
"""

import pytest
from transformers import AutoModelForSeq2SeqLM

from vllm.assets.audio import AudioAsset
from vllm.platforms import current_platform

# Extra engine kwargs needed for numerically deterministic beam search.
# On ROCm, floating-point reductions in attention and GEMM kernels are
# non-associative and sensitive to batch geometry, so we:
#   async_scheduling=False      – deterministic batch composition
#   enforce_eager=True          – no CUDA-graph padding changing effective size
#   enable_prefix_caching=False – avoid prefix-sharing side effects
# ... omitted for brevity ...
        max_num_seqs=1,
    )
    if current_platform.is_rocm()
    else dict(async_scheduling=False, max_num_seqs=1)
)

# FIXME(zhuohan): The test can not pass if we:
#   1. Increase max_tokens to 256.
#   2. Increase beam_width to 8.
#   3. Use the model "huggyllama/llama-7b".
MAX_TOKENS = [64]
BEAM_WIDTHS = [4]
MM_BEAM_WIDTHS = [2]
MODELS = ["TinyLlama/TinyLlama-1.1B-Chat-v1.0"]
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `pytest`, `transformers`, `vllm.assets.audio`, `vllm.platforms`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Test: test_beam_search_single_input (lines 43-85)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["half"])
@pytest.mark.parametrize("max_tokens", MAX_TOKENS)
@pytest.mark.parametrize("beam_width", BEAM_WIDTHS)
def test_beam_search_single_input(
    monkeypatch,
    hf_runner,
    vllm_runner,
    example_prompts,
    model: str,
    dtype: str,
    max_tokens: int,
    beam_width: int,
) -> None:
    if current_platform.is_rocm():
        monkeypatch.setenv("VLLM_ROCM_USE_SKINNY_GEMM", "0")

    example_prompts = example_prompts[:1]
    with hf_runner(model, dtype=dtype) as hf_model:
# ... omitted for brevity ...
        hf_output_ids, hf_output_texts = hf_outputs[i]
        vllm_output_ids, vllm_output_texts = vllm_outputs[i]
        for j, (hf_text, vllm_text) in enumerate(
            zip(hf_output_texts, vllm_output_texts)
        ):
            print(f">>>{j}-th hf output:")
            print(hf_text)
            print(f">>>{j}-th vllm output:")
            print(vllm_text)
        assert len(hf_output_ids) == len(vllm_output_ids)
        for j in range(len(hf_output_ids)):
            assert hf_output_ids[j] == vllm_output_ids[j], (
                f"Test{i} output{j}:\nHF: {hf_output_ids}\nvLLM: {vllm_output_ids}"
            )
```
**EN:** Checks Beam Search Single Input under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `current_platform.is_rocm`, `range` before asserting the expected outcome.
**CN:** 该测试用例验证 Beam Search Single Input 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `current_platform.is_rocm`, `range` 驱动目标逻辑，再断言预期结果。

### Test: test_beam_search_with_concurrency_limit (lines 88-147)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["half"])
@pytest.mark.parametrize("max_tokens", MAX_TOKENS)
@pytest.mark.parametrize("beam_width", BEAM_WIDTHS)
def test_beam_search_with_concurrency_limit(
    monkeypatch,
    hf_runner,
    vllm_runner,
    example_prompts,
    model: str,
    dtype: str,
    max_tokens: int,
    beam_width: int,
) -> None:
    if current_platform.is_rocm():
        monkeypatch.setenv("VLLM_ROCM_USE_SKINNY_GEMM", "0")

    # example_prompts[1]&[3]&[7] fails due to unknown reason even without
    # concurrency limit. skip them for now.
# ... omitted for brevity ...
        ):
            print(f">>>{j}-th with limit output:")
            print(text_with_limit)
            print(f">>>{j}-th without limit output:")
            print(text_without_limit)
        assert len(output_ids_with_limit) == len(output_ids_without_limit)
        for j in range(len(output_ids_with_limit)):
            if output_ids_with_limit[j] != output_ids_without_limit[j]:
                print(
                    f"Test{i} output{j}:\n+limit: {output_ids_with_limit}\n"
                    f"-limit: {output_ids_without_limit}"
                )
                correct = False
    assert correct
```
**EN:** Checks Beam Search With Concurrency Limit under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `current_platform.is_rocm`, `range` before asserting the expected outcome.
**CN:** 该测试用例验证 Beam Search With Concurrency Limit 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `current_platform.is_rocm`, `range` 驱动目标逻辑，再断言预期结果。

### Test: test_beam_search_passes_multimodal_data (lines 150-221)
```python
@pytest.mark.parametrize("dtype", ["half"])
@pytest.mark.parametrize("max_tokens", MAX_TOKENS)
@pytest.mark.parametrize("beam_width", MM_BEAM_WIDTHS)
def test_beam_search_passes_multimodal_data(
    monkeypatch,
    hf_runner,
    vllm_runner,
    dtype: str,
    max_tokens: int,
    beam_width: int,
) -> None:
    """Ensure that beam search passes multimodal data through correctly."""
    if current_platform.is_rocm():
        monkeypatch.setenv("VLLM_ROCM_USE_SKINNY_GEMM", "0")

    # NOTE - this test is primarily to check that mm data is passed to beams
    # correctly. As such, we just need to check one extra modality to make
    # sure things pass through properly.
    audios = [AudioAsset("mary_had_lamb").audio_and_sample_rate]
# ... omitted for brevity ...
        for j in range(len(hf_output_ids)):
            # Compare everything except for the audio tokens; we do this since
            # the IDs returned from the transformers helper expands the audio
            # token to match features, while the vLLM helper maintains the
            # single audio token in the input text
            filtered_hf_output_ids = seq_with_no_audio_toks(hf_output_ids[j])
            filtered_vllm_output_ids = seq_with_no_audio_toks(vllm_output_ids[j])

            # HF output IDs may contain the end of sequence
            if len(filtered_hf_output_ids) == len(filtered_vllm_output_ids) + 1:
                assert filtered_hf_output_ids[-1] == eos_token_id
                filtered_hf_output_ids = filtered_hf_output_ids[:-1]

            assert filtered_hf_output_ids == filtered_vllm_output_ids
```
**EN:** Ensure that beam search passes multimodal data through correctly. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `current_platform.is_rocm`, `range` before asserting the expected outcome.
**CN:** 该测试用例验证 Beam Search Passes Multimodal Data 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `current_platform.is_rocm`, `range` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `pytest`, `transformers`
- **vLLM internal / vLLM 内部依赖**: `vllm.assets.audio`, `vllm.platforms`
