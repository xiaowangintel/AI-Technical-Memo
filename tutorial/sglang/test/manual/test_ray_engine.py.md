# test_ray_engine.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_ray_engine.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `ray engine` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual` 中的 `ray engine` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-74: Request and response handling / 请求与响应处理
```python
"""Integration tests for RayEngine and Ray HTTP server (requires GPU + Ray).

Tests the Ray actor scheduler backend:
  - Offline inference via Engine(use_ray=True) inside a Ray actor on a placement group
  - Data parallel (DP) and DP attention support
  - Error paths in RayEngine._launch_scheduler_processes()
  - HTTP server launched via --use-ray flag

Usage:
    # 1-GPU tests
    python -m pytest test/manual/test_ray_engine.py::TestRayEngineOfflineTP1 -v -s
    python -m pytest test/manual/test_ray_engine.py::TestRayEngineErrors -v -s
    python -m pytest test/manual/test_ray_engine.py::TestRayHTTPServerTP1 -v -s

    # 2-GPU tests
    python -m pytest test/manual/test_ray_engine.py::TestRayEngineOfflineTP2 -v -s
    python -m pytest test/manual/test_ray_engine.py::TestRayEngineOfflinePP2 -v -s
    python -m pytest test/manual/test_ray_engine.py::TestRayEngineOfflineDP2 -v -s
    python -m pytest test/manual/test_ray_engine.py::TestRayEngineOfflineDPAttention -v -s
"""

from __future__ import annotations

import os
import time
import unittest

import torch

from sglang.test.test_utils import DEFAULT_SMALL_MODEL_NAME_FOR_TEST

# Allow overriding the model via env var for environments without gated access
_MODEL = os.environ.get("SGLANG_TEST_MODEL", DEFAULT_SMALL_MODEL_NAME_FOR_TEST)

# DP attention requires a model whose num_kv_heads divides evenly across the
# attention-TP dimension.  Qwen2.5-0.5B (kv_heads=2, attn_heads=14) hits a
# shape mismatch in the KV cache, so we use a larger model here.
_DP_ATTN_MODEL = os.environ.get("SGLANG_TEST_DP_ATTN_MODEL", "Qwen/Qwen3-8B")

try:
    import ray
    from ray.runtime_env import RuntimeEnv
    from ray.util.placement_group import placement_group
    from ray.util.scheduling_strategies import PlacementGroupSchedulingStrategy

    # Prevent Ray from overriding CUDA_VISIBLE_DEVICES so that all GPUs
    # remain visible inside actors regardless of num_gpus allocation.
    _env_vars = {"RAY_EXPERIMENTAL_NOSET_CUDA_VISIBLE_DEVICES": "1"}
    if os.environ.get("HF_TOKEN"):
        _env_vars["HF_TOKEN"] = os.environ["HF_TOKEN"]
    _RAY_RUNTIME_ENV = RuntimeEnv(env_vars=_env_vars)
    _has_ray = True
except ImportError:
    _has_ray = False
    _RAY_RUNTIME_ENV = None


_NUM_GPUS = torch.cuda.device_count()

_SAMPLING_PARAMS = {"max_new_tokens": 32, "temperature": 0.0}

_PROMPTS = [
    "The capital of France is",
    "Explain quantum computing in simple terms:",
    "Write a haiku about programming:",
    "What is 2 + 2?",
]


# ---------------------------------------------------------------------------
# Helpers
# ---------------------------------------------------------------------------
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `server`, `Engine`, `parallel` and `_launch_scheduler_processes`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 75-99: Class definition for EngineActor / 类定义
```python
def _create_engine_on_pg(
    tp_size, pp_size=1, dp_size=1, model=_MODEL, extra_kwargs=None
):
    """Create an EngineActor on a placement group and wait for it to be ready.

    Returns (engine_actor, placement_group).
    """

    @ray.remote
    class EngineActor:
        def __init__(self, **kwargs):
            from sglang.srt.ray.engine import RayEngine

            self.engine = RayEngine(**kwargs)

        def is_ready(self):
            return True

        def generate(self, prompt, sampling_params):
            return self.engine.generate(prompt=prompt, sampling_params=sampling_params)

        def shutdown(self):
            if self.engine:
                self.engine.shutdown()
                self.engine = None
```
**EN:** This range declares `EngineActor`, which organizes the scenario as a reusable test-oriented class. Representative call sites include `Returns` and `RayEngine`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 100-124: Request and response handling / 请求与响应处理
```python

    enable_dp_attention = (extra_kwargs or {}).get("enable_dp_attention", False)
    if enable_dp_attention:
        # DP attention folds DP into TP — total GPUs = tp_size * pp_size
        total_gpus = tp_size * pp_size
    else:
        total_gpus = dp_size * tp_size * pp_size
    pg = placement_group(
        [{"CPU": 1, "GPU": total_gpus}],
        strategy="STRICT_PACK",
    )
    ray.get(pg.ready())

    kwargs = dict(
        model_path=model,
        tp_size=tp_size,
        pp_size=pp_size,
        dp_size=dp_size,
    )
    if extra_kwargs:
        kwargs.update(extra_kwargs)

    actor = EngineActor.options(
        num_cpus=1,
        num_gpus=0,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get`, `placement_group`, `ready` and `update`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 125-132: Request and response handling / 请求与响应处理
```python
        scheduling_strategy=PlacementGroupSchedulingStrategy(
            placement_group=pg,
            placement_group_bundle_index=0,
        ),
    ).remote(**kwargs)

    ray.get(actor.is_ready.remote(), timeout=600)
    return actor, pg
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Representative call sites include `PlacementGroupSchedulingStrategy`, `remote` and `get`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 135-144: Helper routines around _cleanup / 辅助例程
```python
def _cleanup(actor, pg):
    """Shutdown engine actor and remove placement group."""
    try:
        ray.get(actor.shutdown.remote(), timeout=60)
    except Exception:
        pass
    try:
        ray.util.remove_placement_group(pg)
    except Exception:
        pass
```
**EN:** This range implements helper routine(s) `_cleanup` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get`, `remote` and `remove_placement_group`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 145-153: Scenario logic / 场景逻辑
```python


# ---------------------------------------------------------------------------
# Tests: Offline TP=1
# ---------------------------------------------------------------------------


@unittest.skipUnless(_has_ray, "ray is not installed")
@unittest.skipUnless(_NUM_GPUS >= 1, "requires at least 1 GPU")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipUnless`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 154-156: Class definition for TestRayEngineOfflineTP1 / 类定义
```python
class TestRayEngineOfflineTP1(unittest.TestCase):

    @classmethod
```
**EN:** This range declares `TestRayEngineOfflineTP1`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 157-160: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not ray.is_initialized():
            ray.init(log_to_driver=True, runtime_env=_RAY_RUNTIME_ENV)
        cls.actor, cls.pg = _create_engine_on_pg(tp_size=1)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_initialized`, `init` and `_create_engine_on_pg`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 161-162: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 163-165: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        _cleanup(cls.actor, cls.pg)
        ray.shutdown()
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Representative call sites include `_cleanup` and `shutdown`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 167-173: Test routines around test_offline_generate / 测试例程
```python
    def test_offline_generate(self):
        result = ray.get(
            self.actor.generate.remote("The capital of France is", _SAMPLING_PARAMS)
        )
        self.assertIn("text", result)
        self.assertGreater(len(result["text"]), 0)
        print(f"Generated: {result['text'][:200]}")
```
**EN:** This range defines concrete test routine(s) `test_offline_generate`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `remote`, `assertIn` and `assertGreater`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 175-178: Test routines around test_batch_generate / 测试例程
```python
    def test_batch_generate(self):
        for prompt in _PROMPTS:
            result = ray.get(self.actor.generate.remote(prompt, _SAMPLING_PARAMS))
            self.assertIn("text", result)
```
**EN:** This range defines concrete test routine(s) `test_batch_generate`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `remote` and `assertIn`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 179-179: Assertions and result checks / 断言与结果检查
```python
            self.assertGreater(len(result["text"]), 0, f"Empty output for: {prompt}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 181-185: Test routines around test_deterministic / 测试例程
```python
    def test_deterministic(self):
        prompt = "The meaning of life is"
        r1 = ray.get(self.actor.generate.remote(prompt, _SAMPLING_PARAMS))
        r2 = ray.get(self.actor.generate.remote(prompt, _SAMPLING_PARAMS))
        self.assertEqual(r1["text"], r2["text"])
```
**EN:** This range defines concrete test routine(s) `test_deterministic`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `remote` and `assertEqual`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 186-194: Scenario logic / 场景逻辑
```python


# ---------------------------------------------------------------------------
# Tests: Offline TP=2
# ---------------------------------------------------------------------------


@unittest.skipUnless(_has_ray, "ray is not installed")
@unittest.skipUnless(_NUM_GPUS >= 2, "requires at least 2 GPUs")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipUnless`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 195-197: Class definition for TestRayEngineOfflineTP2 / 类定义
```python
class TestRayEngineOfflineTP2(unittest.TestCase):

    @classmethod
```
**EN:** This range declares `TestRayEngineOfflineTP2`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 198-201: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not ray.is_initialized():
            ray.init(log_to_driver=True, runtime_env=_RAY_RUNTIME_ENV)
        cls.actor, cls.pg = _create_engine_on_pg(tp_size=2)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_initialized`, `init` and `_create_engine_on_pg`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 202-203: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 204-206: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        _cleanup(cls.actor, cls.pg)
        ray.shutdown()
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Representative call sites include `_cleanup` and `shutdown`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 208-214: Test routines around test_offline_generate_tp2 / 测试例程
```python
    def test_offline_generate_tp2(self):
        result = ray.get(
            self.actor.generate.remote("The capital of France is", _SAMPLING_PARAMS)
        )
        self.assertIn("text", result)
        self.assertGreater(len(result["text"]), 0)
        print(f"Generated (TP=2): {result['text'][:200]}")
```
**EN:** This range defines concrete test routine(s) `test_offline_generate_tp2`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `remote`, `assertIn` and `assertGreater`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 216-219: Test routines around test_batch_generate_tp2 / 测试例程
```python
    def test_batch_generate_tp2(self):
        for prompt in _PROMPTS:
            result = ray.get(self.actor.generate.remote(prompt, _SAMPLING_PARAMS))
            self.assertIn("text", result)
```
**EN:** This range defines concrete test routine(s) `test_batch_generate_tp2`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `remote` and `assertIn`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 220-220: Assertions and result checks / 断言与结果检查
```python
            self.assertGreater(len(result["text"]), 0, f"Empty output for: {prompt}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 221-229: Scenario logic / 场景逻辑
```python


# ---------------------------------------------------------------------------
# Tests: Offline PP=2
# ---------------------------------------------------------------------------


@unittest.skipUnless(_has_ray, "ray is not installed")
@unittest.skipUnless(_NUM_GPUS >= 2, "requires at least 2 GPUs")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipUnless`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 230-232: Class definition for TestRayEngineOfflinePP2 / 类定义
```python
class TestRayEngineOfflinePP2(unittest.TestCase):

    @classmethod
```
**EN:** This range declares `TestRayEngineOfflinePP2`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 233-236: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not ray.is_initialized():
            ray.init(log_to_driver=True, runtime_env=_RAY_RUNTIME_ENV)
        cls.actor, cls.pg = _create_engine_on_pg(tp_size=1, pp_size=2)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_initialized`, `init` and `_create_engine_on_pg`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 237-238: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 239-241: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        _cleanup(cls.actor, cls.pg)
        ray.shutdown()
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Representative call sites include `_cleanup` and `shutdown`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 243-249: Test routines around test_offline_generate_pp2 / 测试例程
```python
    def test_offline_generate_pp2(self):
        result = ray.get(
            self.actor.generate.remote("The capital of France is", _SAMPLING_PARAMS)
        )
        self.assertIn("text", result)
        self.assertGreater(len(result["text"]), 0)
        print(f"Generated (PP=2): {result['text'][:200]}")
```
**EN:** This range defines concrete test routine(s) `test_offline_generate_pp2`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `remote`, `assertIn` and `assertGreater`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 251-254: Test routines around test_batch_generate_pp2 / 测试例程
```python
    def test_batch_generate_pp2(self):
        for prompt in _PROMPTS:
            result = ray.get(self.actor.generate.remote(prompt, _SAMPLING_PARAMS))
            self.assertIn("text", result)
```
**EN:** This range defines concrete test routine(s) `test_batch_generate_pp2`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `remote` and `assertIn`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 255-255: Assertions and result checks / 断言与结果检查
```python
            self.assertGreater(len(result["text"]), 0, f"Empty output for: {prompt}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 256-264: Scenario logic / 场景逻辑
```python


# ---------------------------------------------------------------------------
# Tests: Error paths
# ---------------------------------------------------------------------------


@unittest.skipUnless(_has_ray, "ray is not installed")
@unittest.skipUnless(_NUM_GPUS >= 2, "requires at least 2 GPUs")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipUnless`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 265-268: Class definition for TestRayEngineOfflineDP2 / 类定义
```python
class TestRayEngineOfflineDP2(unittest.TestCase):
    """Test Ray engine with dp_size=2, tp_size=1."""

    @classmethod
```
**EN:** This range declares `TestRayEngineOfflineDP2`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 269-272: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not ray.is_initialized():
            ray.init(log_to_driver=True, runtime_env=_RAY_RUNTIME_ENV)
        cls.actor, cls.pg = _create_engine_on_pg(tp_size=1, dp_size=2)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_initialized`, `init` and `_create_engine_on_pg`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 273-274: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 275-277: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        _cleanup(cls.actor, cls.pg)
        ray.shutdown()
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Representative call sites include `_cleanup` and `shutdown`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 279-285: Test routines around test_offline_generate_dp2 / 测试例程
```python
    def test_offline_generate_dp2(self):
        result = ray.get(
            self.actor.generate.remote("The capital of France is", _SAMPLING_PARAMS)
        )
        self.assertIn("text", result)
        self.assertGreater(len(result["text"]), 0)
        print(f"Generated (DP=2): {result['text'][:200]}")
```
**EN:** This range defines concrete test routine(s) `test_offline_generate_dp2`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `remote`, `assertIn` and `assertGreater`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 287-289: Test routines around test_batch_generate_dp2 / 测试例程
```python
    def test_batch_generate_dp2(self):
        for prompt in _PROMPTS:
            result = ray.get(self.actor.generate.remote(prompt, _SAMPLING_PARAMS))
```
**EN:** This range defines concrete test routine(s) `test_batch_generate_dp2`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get` and `remote`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 290-291: Assertions and result checks / 断言与结果检查
```python
            self.assertIn("text", result)
            self.assertGreater(len(result["text"]), 0, f"Empty output for: {prompt}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIn` and `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 292-300: Scenario logic / 场景逻辑
```python


# ---------------------------------------------------------------------------
# Tests: Offline DP Attention (dp=2, tp=2)
# ---------------------------------------------------------------------------


@unittest.skipUnless(_has_ray, "ray is not installed")
@unittest.skipUnless(_NUM_GPUS >= 2, "requires at least 2 GPUs")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `Attention` and `skipUnless`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 301-304: Class definition for TestRayEngineOfflineDPAttention / 类定义
```python
class TestRayEngineOfflineDPAttention(unittest.TestCase):
    """Test Ray engine with dp_size=2, tp_size=2, enable_dp_attention=True."""

    @classmethod
```
**EN:** This range declares `TestRayEngineOfflineDPAttention`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 305-317: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not ray.is_initialized():
            ray.init(log_to_driver=True, runtime_env=_RAY_RUNTIME_ENV)
        cls.actor, cls.pg = _create_engine_on_pg(
            tp_size=2,
            dp_size=2,
            model=_DP_ATTN_MODEL,
            extra_kwargs={
                "enable_dp_attention": True,
                "disable_cuda_graph": True,
                "port": 31500,
            },
        )
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_initialized`, `init` and `_create_engine_on_pg`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 318-319: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 320-322: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        _cleanup(cls.actor, cls.pg)
        ray.shutdown()
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Representative call sites include `_cleanup` and `shutdown`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 324-325: Test routines around test_offline_generate_dp_attention / 测试例程
```python
    def test_offline_generate_dp_attention(self):
        result = ray.get(
```
**EN:** This range defines concrete test routine(s) `test_offline_generate_dp_attention`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 326-330: Assertions and result checks / 断言与结果检查
```python
            self.actor.generate.remote("The capital of France is", _SAMPLING_PARAMS)
        )
        self.assertIn("text", result)
        self.assertGreater(len(result["text"]), 0)
        print(f"Generated (DP-Attention): {result['text'][:200]}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `remote`, `assertIn`, `assertGreater` and `Generated`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 332-336: Test routines around test_batch_generate_dp_attention / 测试例程
```python
    def test_batch_generate_dp_attention(self):
        for prompt in _PROMPTS:
            result = ray.get(self.actor.generate.remote(prompt, _SAMPLING_PARAMS))
            self.assertIn("text", result)
            self.assertGreater(len(result["text"]), 0, f"Empty output for: {prompt}")
```
**EN:** This range defines concrete test routine(s) `test_batch_generate_dp_attention`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `remote`, `assertIn` and `assertGreater`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 337-345: Scenario logic / 场景逻辑
```python


# ---------------------------------------------------------------------------
# Tests: Error paths
# ---------------------------------------------------------------------------


@unittest.skipUnless(_has_ray, "ray is not installed")
@unittest.skipUnless(_NUM_GPUS >= 1, "requires at least 1 GPU")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipUnless`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 346-348: Class definition for TestRayEngineErrors / 类定义
```python
class TestRayEngineErrors(unittest.TestCase):

    @classmethod
```
**EN:** This range declares `TestRayEngineErrors`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 349-351: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not ray.is_initialized():
            ray.init(log_to_driver=True, runtime_env=_RAY_RUNTIME_ENV)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_initialized` and `init`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 352-353: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 354-355: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        ray.shutdown()
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Representative call sites include `shutdown`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 357-370: Test routines around test_missing_placement_group_raises / 测试例程
```python
    def test_missing_placement_group_raises(self):
        """RayEngine without a placement group should raise RuntimeError."""

        @ray.remote(num_gpus=1)
        def _try_create_without_pg():
            from sglang.srt.ray.engine import RayEngine

            try:
                RayEngine(
                    model_path=_MODEL,
                    tp_size=1,
                    use_ray=True,
                )
                return None
```
**EN:** This range defines concrete test routine(s) `test_missing_placement_group_raises` and `_try_create_without_pg`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `remote` and `RayEngine`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 371-378: Assertions and result checks / 断言与结果检查
```python
            except RuntimeError as e:
                return str(e)

        error_msg = ray.get(_try_create_without_pg.remote(), timeout=120)
        self.assertIsNotNone(
            error_msg, "Expected RuntimeError but RayEngine created OK"
        )
        self.assertIn("placement group", error_msg.lower())
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `remote`, `assertIsNotNone` and `assertIn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 379-387: Scenario logic / 场景逻辑
```python


# ---------------------------------------------------------------------------
# Tests: HTTP server
# ---------------------------------------------------------------------------


@unittest.skipUnless(_has_ray, "ray is not installed")
@unittest.skipUnless(_NUM_GPUS >= 1, "requires at least 1 GPU")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipUnless`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 388-395: Class definition for TestRayHTTPServerTP1 / 类定义
```python
class TestRayHTTPServerTP1(unittest.TestCase):
    """Test the Ray HTTP server path (launch_server.py --use-ray).

    Launches the server inside a Ray task on a placement group (mirrors
    examples/anyscale/driver_online.py) and sends HTTP requests to it.
    """

    @classmethod
```
**EN:** This range declares `TestRayHTTPServerTP1`, which organizes the scenario as a reusable test-oriented class. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `path` and `group`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 396-412: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        import requests as req_lib

        if not ray.is_initialized():
            ray.init(log_to_driver=True, runtime_env=_RAY_RUNTIME_ENV)

        cls.port = 30100
        cls.pg = placement_group(
            [{"CPU": 1, "GPU": 1}],
            strategy="STRICT_PACK",
        )
        ray.get(cls.pg.ready())

        pg_strategy = PlacementGroupSchedulingStrategy(
            placement_group=cls.pg,
            placement_group_bundle_index=0,
        )
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Representative call sites include `is_initialized`, `init`, `placement_group` and `get`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 413-420: Helper routines around _get_ip / 辅助例程
```python

        # Resolve the node IP where the server will run
        @ray.remote(num_cpus=0, num_gpus=0)
        def _get_ip():
            return ray.util.get_node_ip_address()

        cls.node_ip = ray.get(_get_ip.options(scheduling_strategy=pg_strategy).remote())
        cls.base_url = f"http://{cls.node_ip}:{cls.port}"
```
**EN:** This range implements helper routine(s) `_get_ip` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `remote`, `get_node_ip_address`, `get` and `options`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 421-437: Helper routines around _launch / 辅助例程
```python

        # Launch server as a Ray task (blocks until server exits)
        @ray.remote
        def _launch(**kwargs):
            from sglang.srt.ray.http_server import launch_server
            from sglang.srt.server_args import ServerArgs

            launch_server(ServerArgs(**kwargs))

        cls.server_ref = _launch.options(
            num_cpus=1,
            num_gpus=0,
            scheduling_strategy=pg_strategy,
        ).remote(
            model_path=_MODEL,
            tp_size=1,
            port=cls.port,
```
**EN:** This range implements helper routine(s) `_launch` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `task`, `launch_server`, `ServerArgs` and `options`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 438-445: Scenario logic / 场景逻辑
```python
            host="0.0.0.0",
            use_ray=True,
        )

        # Wait for health check
        t0 = time.time()
        timeout = 600
        healthy = False
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `time`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 446-462: Request and response handling / 请求与响应处理
```python
        while time.time() - t0 < timeout:
            ready, _ = ray.wait([cls.server_ref], timeout=0)
            if ready:
                try:
                    ray.get(cls.server_ref)
                except Exception as e:
                    raise RuntimeError(f"Server task crashed: {e}") from e
                raise RuntimeError("Server task exited before becoming healthy")
            try:
                if req_lib.get(f"{cls.base_url}/health", timeout=5).status_code == 200:
                    healthy = True
                    break
            except req_lib.exceptions.RequestException:
                pass
            time.sleep(3)

        if not healthy:
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Representative call sites include `time`, `wait`, `get` and `RuntimeError`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 463-464: Scenario logic / 场景逻辑
```python
            ray.cancel(cls.server_ref, force=True)
            raise RuntimeError(f"Server did not become healthy within {timeout}s")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `cancel` and `RuntimeError`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 465-466: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 467-476: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        try:
            ray.cancel(cls.server_ref, force=True)
        except Exception:
            pass
        try:
            ray.util.remove_placement_group(cls.pg)
        except Exception:
            pass
        ray.shutdown()
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Representative call sites include `cancel`, `remove_placement_group` and `shutdown`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 478-482: Test routines around test_health_endpoint / 测试例程
```python
    def test_health_endpoint(self):
        import requests

        resp = requests.get(f"{self.base_url}/health", timeout=10)
        self.assertEqual(resp.status_code, 200)
```
**EN:** This range defines concrete test routine(s) `test_health_endpoint`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get` and `assertEqual`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 484-487: Imports and shared helpers / 导入与共享辅助项
```python
    def test_generate_endpoint(self):
        import requests

        resp = requests.post(
```
**EN:** This range imports `requests`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. It interacts with request/response APIs to observe live model behavior. Representative call sites include `post`.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 488-499: Assertions and result checks / 断言与结果检查
```python
            f"{self.base_url}/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": _SAMPLING_PARAMS,
            },
            timeout=60,
        )
        resp.raise_for_status()
        data = resp.json()
        self.assertIn("text", data)
        self.assertGreater(len(data["text"]), 0)
        print(f"HTTP response: {data['text'][:200]}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `raise_for_status`, `json`, `assertIn` and `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 501-512: Test routines around test_generate_multiple / 测试例程
```python
    def test_generate_multiple(self):
        import requests

        for prompt in _PROMPTS:
            resp = requests.post(
                f"{self.base_url}/generate",
                json={
                    "text": prompt,
                    "sampling_params": _SAMPLING_PARAMS,
                },
                timeout=60,
            )
```
**EN:** This range defines concrete test routine(s) `test_generate_multiple`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `post`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 513-516: Assertions and result checks / 断言与结果检查
```python
            resp.raise_for_status()
            data = resp.json()
            self.assertIn("text", data)
            self.assertGreater(len(data["text"]), 0, f"Empty output for: {prompt}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `raise_for_status`, `json`, `assertIn` and `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 517-520: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Pytest markers / Pytest 标记
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `__future__`, `os`, `time`, `unittest`
- **Third-party / 第三方库**: `ray`, `ray.runtime_env`, `ray.util.placement_group`, `ray.util.scheduling_strategies`, `requests`, `torch`
- **Project Modules / 项目模块**: `sglang.srt.ray.engine`, `sglang.srt.ray.http_server`, `sglang.srt.server_args`, `sglang.test.test_utils`
