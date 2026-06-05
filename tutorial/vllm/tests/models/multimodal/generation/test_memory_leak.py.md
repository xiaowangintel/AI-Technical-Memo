# test_memory_leak.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/test_memory_leak.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 1 test(s), 1 fixture(s), and 5 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 1 个测试、1 个 fixture，以及 5 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L18)
```python
import gc
import random
import string
import sys
import weakref

import pytest
import torch

from tests.models.registry import HF_EXAMPLE_MODELS
from vllm import LLM, SamplingParams
from vllm.distributed import cleanup_dist_env_and_memory
from vllm.entrypoints.chat_utils import ChatCompletionMessageParam
from vllm.platforms import current_platform
from vllm.utils.mem_utils import KiB_bytes, MiB_bytes, format_mib
```
**EN:** Imports standard-library modules such as `gc`, `random`, `resource`, third-party packages like `pytest`, `torch`, project helpers such as `tests.models.registry.HF_EXAMPLE_MODELS`, `vllm.LLM`, `vllm.SamplingParams`.
**CN:** 导入标准库模块（如 `gc`、`random`、`resource`）、第三方包（如 `pytest`、`torch`）、项目内辅助模块（如 `tests.models.registry.HF_EXAMPLE_MODELS`、`vllm.LLM`、`vllm.SamplingParams`）。

### Module setup / 模块级配置: MODEL_NAME, RANDOM_PREFIX_LEN, TEST_IMAGE_NAMES (L20-L36)
```python
MODEL_NAME = "Qwen/Qwen3-VL-4B-Instruct"
RANDOM_PREFIX_LEN = 100
TEST_IMAGE_NAMES = [
    "2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",
    "Grayscale_8bits_palette_sample_image.png",
]
MAX_MODEL_LEN = 8192
REQUESTS_PER_ROUND = 4
WARMUP_ROUNDS = 1
MEASURED_ROUNDS = 16
GPU_GROWTH_THRESHOLD_MIB = 0
CPU_PEAK_GROWTH_THRESHOLD_MIB = 0

SAMPLING_PARAMS = SamplingParams(
    temperature=0.0,
    max_tokens=16,
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `RANDOM_PREFIX_LEN`, `TEST_IMAGE_NAMES`, `MAX_MODEL_LEN`, `REQUESTS_PER_ROUND`, `WARMUP_ROUNDS`, `MEASURED_ROUNDS`, `GPU_GROWTH_THRESHOLD_MIB`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`RANDOM_PREFIX_LEN`、`TEST_IMAGE_NAMES`、`MAX_MODEL_LEN`、`REQUESTS_PER_ROUND`、`WARMUP_ROUNDS`、`MEASURED_ROUNDS`、`GPU_GROWTH_THRESHOLD_MIB`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _make_messages (L39-L58)
```python
def _make_messages(image_url: str) -> list[ChatCompletionMessageParam]:
    # Avoid obscuring memory leaks because of prefix caching
    random_text = "".join(random.choices(string.ascii_uppercase, k=RANDOM_PREFIX_LEN))

    return [
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": f"Ignore this random string: {random_text}",
                },
                {"type": "image_url", "image_url": {"url": image_url}},
                {
                    "type": "text",
                    "text": "Describe this image in one short sentence.",
                },
            ],
        }
    ]
```
**EN:** This helper encapsulates reusable logic in `_make_messages`. Key inputs are `image_url`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_messages` 中。 关键输入包括 `image_url`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _build_request_batch (L61-L67)
```python
def _build_request_batch(
    image_urls: list[str],
) -> list[list[ChatCompletionMessageParam]]:
    return [
        _make_messages(image_urls[i % len(image_urls)])
        for i in range(REQUESTS_PER_ROUND)
    ]
```
**EN:** This helper encapsulates reusable logic in `_build_request_batch`. Key inputs are `image_urls`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_build_request_batch` 中。 关键输入包括 `image_urls`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _ru_maxrss_bytes (L70-L81)
```python
def _ru_maxrss_bytes() -> int | None:
    try:
        import resource
    except ImportError:
        return None

    rss = resource.getrusage(resource.RUSAGE_SELF).ru_maxrss
    if rss <= 0:
        return 0

    # Linux reports kilobytes, macOS reports bytes.
    return rss if sys.platform == "darwin" else rss * KiB_bytes
```
**EN:** This helper encapsulates reusable logic in `_ru_maxrss_bytes`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_ru_maxrss_bytes` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _gpu_used_bytes (L84-L87)
```python
def _gpu_used_bytes() -> int:
    torch.accelerator.synchronize()
    free_bytes, total_bytes = current_platform.mem_get_info()
    return int(total_bytes - free_bytes)
```
**EN:** This helper encapsulates reusable logic in `_gpu_used_bytes`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_gpu_used_bytes` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _format_mib (L90-L94)
```python
def _format_mib(num_bytes: int | None) -> str:
    if num_bytes is None:
        return "n/a"

    return f"{format_mib(num_bytes)} MiB"
```
**EN:** This helper encapsulates reusable logic in `_format_mib`. Key inputs are `num_bytes`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_format_mib` 中。 关键输入包括 `num_bytes`。 它把计算得到的状态或辅助对象返回给调用方。

### Fixture / 夹具: llm (L97-L122)
```python
@pytest.fixture(scope="function")
def llm(monkeypatch):
    monkeypatch.setenv("VLLM_ENABLE_V1_MULTIPROCESSING", "0")

    # pytest caches the fixture so we use weakref.proxy to
    # enable garbage collection
    llm_kwargs = dict(
        model=MODEL_NAME,
        enforce_eager=True,
        max_model_len=MAX_MODEL_LEN,
        max_num_seqs=REQUESTS_PER_ROUND,
        limit_mm_per_prompt={"image": 1},
        seed=0,
        disable_log_stats=True,
        gpu_memory_utilization=0.8,
    )
    if current_platform.is_rocm():
        llm_kwargs["attention_backend"] = "TRITON_ATTN"

    llm = LLM(**llm_kwargs)

    yield weakref.proxy(llm)

    del llm

    cleanup_dist_env_and_memory()
```
**EN:** This fixture prepares `llm` for dependent tests. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. It touches the core vLLM initialization or engine path directly. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `llm`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_no_memory_leak (L125-L182)
```python
@pytest.mark.core_model
@pytest.mark.parametrize("image_urls", [TEST_IMAGE_NAMES], indirect=True)
def test_no_memory_leak(llm, image_urls: list[str]) -> None:
    model_info = HF_EXAMPLE_MODELS.find_hf_info(MODEL_NAME)
    model_info.check_available_online(on_fail="skip")
    model_info.check_transformers_version(on_fail="skip")

    request_batch = _build_request_batch(image_urls)

    # Establish a warmup baseline after model load and the first multimodal
    # requests complete. Later rounds should remain near this steady state.
    for _ in range(WARMUP_ROUNDS):
        outputs = llm.chat(request_batch, sampling_params=SAMPLING_PARAMS)
        assert len(outputs) == len(request_batch)
        assert llm.llm_engine.get_num_unfinished_requests() == 0
        del outputs

    gc.collect()
# ... 32 lines omitted for brevity ...

        assert cpu_peak_growth <= cpu_threshold, (
            "Qwen3-VL CPU peak RSS kept growing after warmup. "
            f"warmup_ru_maxrss={_format_mib(warmup_cpu_peak)}, "
            f"post_warmup_ru_maxrss={[_format_mib(x) for x in post_warmup_cpu_peak_samples]}, "  # noqa: E501
            f"cpu_peak_growth={_format_mib(cpu_peak_growth)}, "
            f"cpu_peak_threshold={CPU_PEAK_GROWTH_THRESHOLD_MIB} MiB"
        )
```
**EN:** This test validates `test_no_memory_leak`. It uses parameterization over `image_urls`. Relevant pytest markers include `core_model`. Key inputs are `llm`, `image_urls`. The main assertion is `gpu_growth <= gpu_threshold` and `len(outputs) == len(request_batch)`.
**CN:** 这个测试验证 `test_no_memory_leak`。 它通过参数化组合 `image_urls`。 相关的 pytest 标记包括 `core_model`。 关键输入包括 `llm`、`image_urls`。 核心断言是 `gpu_growth <= gpu_threshold` and `len(outputs) == len(request_batch)`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `gc`, `random`, `resource`, `string`, `sys`, `weakref`
- **Third-party / 第三方**: `pytest`, `torch`
- **Project / 项目内**: `tests.models.registry.HF_EXAMPLE_MODELS`, `vllm.LLM`, `vllm.SamplingParams`, `vllm.distributed.cleanup_dist_env_and_memory`, `vllm.entrypoints.chat_utils.ChatCompletionMessageParam`, `vllm.platforms.current_platform`, `vllm.utils.mem_utils.KiB_bytes`, `vllm.utils.mem_utils.MiB_bytes`, `vllm.utils.mem_utils.format_mib`
