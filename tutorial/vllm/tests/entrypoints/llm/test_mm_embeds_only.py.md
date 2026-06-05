# test_mm_embeds_only.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/llm/test_mm_embeds_only.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers LLM entrypoint behavior. The file defines 3 test(s), 1 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖LLM 入口行为。它定义了 3 个测试、1 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L10)
```python
import weakref

import pytest

from vllm import LLM, SamplingParams
from vllm.assets.image import ImageAsset
from vllm.distributed import cleanup_dist_env_and_memory
```
**EN:** Imports standard-library modules such as `weakref`, third-party packages like `pytest`, project helpers such as `vllm.LLM`, `vllm.SamplingParams`, `vllm.assets.image.ImageAsset`.
**CN:** 导入标准库模块（如 `weakref`）、第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.LLM`、`vllm.SamplingParams`、`vllm.assets.image.ImageAsset`）。

### Module setup / 模块级配置: MODEL, PROMPT, TEXT_ONLY_PROMPT (L12-L14)
```python
MODEL = "llava-hf/llava-1.5-7b-hf"
PROMPT = "USER: <image>\nDescribe this image briefly.\nASSISTANT:"
TEXT_ONLY_PROMPT = "USER: What is 2 + 2?\nASSISTANT:"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL`, `PROMPT`, `TEXT_ONLY_PROMPT`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL`、`PROMPT`、`TEXT_ONLY_PROMPT`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: llm (L17-L33)
```python
@pytest.fixture(scope="module")
def llm():
    """LLM with enable_mm_embeds=True and all modality limits zeroed out."""
    llm = LLM(
        model=MODEL,
        max_model_len=2048,
        enforce_eager=True,
        gpu_memory_utilization=0.8,
        enable_mm_embeds=True,
        limit_mm_per_prompt={"image": 0},
    )

    yield weakref.proxy(llm)

    del llm

    cleanup_dist_env_and_memory()
```
**EN:** This fixture prepares `llm` for dependent tests. It touches the core vLLM initialization or engine path directly. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `llm`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_generate_with_embedding (L36-L45)
```python
@pytest.mark.skip_global_cleanup
def test_generate_with_embedding(llm: LLM):
    """Pre-computed embedding produces tokens without hanging."""
    embedding = ImageAsset("stop_sign").image_embeds
    outputs = llm.generate(
        {"prompt": PROMPT, "multi_modal_data": {"image": embedding}},
        sampling_params=SamplingParams(max_tokens=32, temperature=0.0),
    )
    assert len(outputs) == 1
    assert len(outputs[0].outputs[0].text) > 0
```
**EN:** This test validates `test_generate_with_embedding`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `len(outputs) == 1` and `len(outputs[0].outputs[0].text) > 0`.
**CN:** 这个测试验证 `test_generate_with_embedding`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `len(outputs) == 1` and `len(outputs[0].outputs[0].text) > 0`。

### Test / 测试: test_raw_image_rejected (L48-L56)
```python
@pytest.mark.skip_global_cleanup
def test_raw_image_rejected(llm: LLM):
    """Raw image input is still rejected when limit=0."""
    raw_image = ImageAsset("stop_sign").pil_image
    with pytest.raises(ValueError, match=r"At most 0 image\(s\)"):
        llm.generate(
            {"prompt": PROMPT, "multi_modal_data": {"image": raw_image}},
            sampling_params=SamplingParams(max_tokens=16),
        )
```
**EN:** This test validates `test_raw_image_rejected`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_raw_image_rejected`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_text_only_prompt (L59-L67)
```python
@pytest.mark.skip_global_cleanup
def test_text_only_prompt(llm: LLM):
    """Text-only prompts still work under this config."""
    outputs = llm.generate(
        TEXT_ONLY_PROMPT,
        sampling_params=SamplingParams(max_tokens=16, temperature=0.0),
    )
    assert len(outputs) == 1
    assert len(outputs[0].outputs[0].text) > 0
```
**EN:** This test validates `test_text_only_prompt`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `len(outputs) == 1` and `len(outputs[0].outputs[0].text) > 0`.
**CN:** 这个测试验证 `test_text_only_prompt`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `len(outputs) == 1` and `len(outputs[0].outputs[0].text) > 0`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `weakref`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.LLM`, `vllm.SamplingParams`, `vllm.assets.image.ImageAsset`, `vllm.distributed.cleanup_dist_env_and_memory`
