# test_offline_mode.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/offline_mode/test_offline_mode.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for offline-mode execution. The file exposes 3 helper/class block(s) used by nearby tests. / [CN] 为离线模式执行提供共享测试工具。该文件暴露了 3 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L13)
```python
import importlib
import sys

import pytest
import regex as re
import urllib3

from vllm import LLM
from vllm.distributed import cleanup_dist_env_and_memory
```
**EN:** Imports standard-library modules such as `importlib`, `sys`, third-party packages like `pytest`, `regex`, `urllib3`, project helpers such as `vllm.LLM`, `vllm.distributed.cleanup_dist_env_and_memory`.
**CN:** 导入标准库模块（如 `importlib`、`sys`）、第三方包（如 `pytest`、`regex`、`urllib3`）、项目内辅助模块（如 `vllm.LLM`、`vllm.distributed.cleanup_dist_env_and_memory`）。

### Module setup / 模块级配置: MODEL_CONFIGS (L15-L55)
```python
MODEL_CONFIGS = [
    {
        "model": "facebook/opt-125m",
        "enforce_eager": True,
        "gpu_memory_utilization": 0.20,
        "max_model_len": 64,
        "max_num_batched_tokens": 64,
        "max_num_seqs": 64,
        "tensor_parallel_size": 1,
    },
    {
        "model": "Qwen/Qwen3-0.6B",
        "enforce_eager": True,
        "gpu_memory_utilization": 0.50,
        "max_model_len": 64,
        "max_num_batched_tokens": 64,
# ... 19 lines omitted for brevity ...
    #     "max_model_len": 64,
    #     "max_num_batched_tokens": 64,
    #     "max_num_seqs": 64,
    #     "tensor_parallel_size": 1,
    # },
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_CONFIGS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_CONFIGS`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: cache_models (L58-L65)
```python
@pytest.fixture(scope="module")
def cache_models():
    # Cache model files first
    for model_config in MODEL_CONFIGS:
        LLM(**model_config)
        cleanup_dist_env_and_memory()

    yield
```
**EN:** This fixture prepares `cache_models` for dependent tests. It touches the core vLLM initialization or engine path directly. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `cache_models`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: test_offline_mode (L68-L100)
```python
@pytest.mark.skip_global_cleanup
@pytest.mark.usefixtures("cache_models")
def test_offline_mode(monkeypatch: pytest.MonkeyPatch):
    # Set HF to offline mode and ensure we can still construct an LLM
    with monkeypatch.context() as m:
        try:
            m.setenv("HF_HUB_OFFLINE", "1")
            m.setenv("VLLM_NO_USAGE_STATS", "1")

            def disable_connect(*args, **kwargs):
                raise RuntimeError("No http calls allowed")

            m.setattr(
                urllib3.connection.HTTPConnection,
                "connect",
                disable_connect,
            )
            m.setattr(
# ... 7 lines omitted for brevity ...
            _re_import_modules()
            # Cached model files should be used in offline mode
            for model_config in MODEL_CONFIGS:
                LLM(**model_config)
        finally:
            # Reset the environment after the test
            # NB: Assuming tests are run in online mode
            _re_import_modules()
```
**EN:** This fixture prepares `test_offline_mode` for dependent tests. Relevant pytest markers include `skip_global_cleanup`, `usefixtures`. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. It touches the core vLLM initialization or engine path directly.
**CN:** 这个 fixture 为依赖它的测试准备 `test_offline_mode`。 相关的 pytest 标记包括 `skip_global_cleanup`、`usefixtures`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。

### Helper / 辅助函数: _re_import_modules (L103-L134)
```python
def _re_import_modules():
    hf_hub_module_names = [k for k in sys.modules if k.startswith("huggingface_hub")]
    transformers_module_names = [
        k
        for k in sys.modules
        if k.startswith("transformers") and not k.startswith("transformers_modules")
    ]

    # These modules are aliased in Transformers v5 and so cannot be reloaded directly
    aliased_module_patterns = [
        r".+\.tokenization_utils$",
        r".+\.tokenization_utils_fast$",
        r".+\.image_processing_utils_fast$",
        r".+\.models\..+\.image_processing_.+_fast$",
    ]

    reload_exception = None
    for module_name in hf_hub_module_names + transformers_module_names:
# ... 6 lines omitted for brevity ...
        except Exception as e:
            reload_exception = e
            # Try to continue clean up so that other tests are less likely to
            # be affected

    # Error this test if reloading a module failed
    if reload_exception is not None:
        raise reload_exception
```
**EN:** This helper encapsulates reusable logic in `_re_import_modules`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_re_import_modules` 中。

### Fixture / 夹具: test_model_from_huggingface_offline (L137-L166)
```python
@pytest.mark.skip_global_cleanup
@pytest.mark.usefixtures("cache_models")
def test_model_from_huggingface_offline(monkeypatch: pytest.MonkeyPatch):
    # Set HF to offline mode and ensure we can still construct an LLM
    with monkeypatch.context() as m:
        try:
            m.setenv("HF_HUB_OFFLINE", "1")
            m.setenv("VLLM_NO_USAGE_STATS", "1")

            def disable_connect(*args, **kwargs):
                raise RuntimeError("No http calls allowed")

            m.setattr(
                urllib3.connection.HTTPConnection,
                "connect",
                disable_connect,
            )
            m.setattr(
                urllib3.connection.HTTPSConnection,
                "connect",
                disable_connect,
            )
            # Need to re-import huggingface_hub
            # and friends to set up offline mode
            _re_import_modules()
            LLM(model="facebook/opt-125m")
        finally:
            # Reset the environment after the test
            # NB: Assuming tests are run in online mode
            _re_import_modules()
```
**EN:** This fixture prepares `test_model_from_huggingface_offline` for dependent tests. Relevant pytest markers include `skip_global_cleanup`, `usefixtures`. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. It touches the core vLLM initialization or engine path directly.
**CN:** 这个 fixture 为依赖它的测试准备 `test_model_from_huggingface_offline`。 相关的 pytest 标记包括 `skip_global_cleanup`、`usefixtures`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `importlib`, `sys`
- **Third-party / 第三方**: `pytest`, `regex`, `urllib3`
- **Project / 项目内**: `vllm.LLM`, `vllm.distributed.cleanup_dist_env_and_memory`
