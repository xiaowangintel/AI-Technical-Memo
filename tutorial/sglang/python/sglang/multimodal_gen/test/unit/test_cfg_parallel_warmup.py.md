# test_cfg_parallel_warmup.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/unit/test_cfg_parallel_warmup.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates cfg parallel warmup with focused assertions and fixtures. Key symbols include `_make_bare_scheduler`, `_make_input_validation_stage`, `_make_validation_server_args`. / 该测试模块通过有针对性的断言与夹具，验证 cfg parallel warmup 的实现。 关键符号包括 `_make_bare_scheduler`, `_make_input_validation_stage`, `_make_validation_server_args`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: Imports and module setup / 导入与模块初始化
```python
"""Unit tests for the --enable-cfg-parallel warmup fix and guard.

Covers two code paths introduced alongside this file:
- Scheduler.prepare_server_warmup_reqs synthesizes warmup Reqs that
  actually enable classifier-free guidance when cfg-parallel is on.
- InputValidationStage.forward rejects non-CFG requests when the server
  has cfg-parallel on.

All tests are CPU-only; no model loading, no distributed init.
"""

import unittest
from collections import deque
from unittest.mock import MagicMock, patch
# ...
# Patch path for get_global_server_args used by Stage.__init__
_GLOBAL_ARGS_PATCH = (
    "sglang.multimodal_gen.runtime.pipelines_core.stages.base.get_global_server_args"
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 32-58: Function `_make_bare_scheduler` / 函数 `_make_bare_scheduler`
```python
def _make_bare_scheduler(enable_cfg_parallel: bool) -> Scheduler:
    """
    Build a minimal Scheduler without calling __init__ (which requires
    distributed init, ZMQ sockets, pipeline load, etc.). Populates only
    the attributes prepare_server_warmup_reqs reads/writes for a
    text-only task so _prepare_shared_warmup_image_path is skipped.
    """
    scheduler = object.__new__(Scheduler)

    server_args = MagicMock()
    server_args.warmup = True
    server_args.warmup_steps = 1
    server_args.warmup_resolutions = ["512x512"]
    server_args.enable_cfg_parallel = enable_cfg_parallel
# ...
    scheduler.server_args = server_args
    scheduler.warmed_up = False
    scheduler.waiting_queue = deque()
    return scheduler
```
**EN:** This function drives `_make_bare_scheduler` with inputs such as `enable_cfg_parallel`. Build a minimal Scheduler without calling __init__ (which requires
**CN:** 这个函数负责 `_make_bare_scheduler`，主要处理 `enable_cfg_parallel` 等输入。 文档字符串说明：Build a minimal Scheduler without calling __init__ (which requires

### Lines 61-66: Function `_make_input_validation_stage` / 函数 `_make_input_validation_stage`
```python
def _make_input_validation_stage() -> InputValidationStage:
    """Construct InputValidationStage with the global server-args patch
    that existing tests in this suite use (see test_input_validation.py)."""
    with patch(_GLOBAL_ARGS_PATCH) as m:
        m.return_value = MagicMock()
        return InputValidationStage()
```
**EN:** This function drives `_make_input_validation_stage`. Construct InputValidationStage with the global server-args patch
**CN:** 这个函数负责 `_make_input_validation_stage`。 文档字符串说明：Construct InputValidationStage with the global server-args patch

### Lines 69-73: Function `_make_validation_server_args` / 函数 `_make_validation_server_args`
```python
def _make_validation_server_args(enable_cfg_parallel: bool) -> MagicMock:
    sa = MagicMock()
    sa.enable_cfg_parallel = enable_cfg_parallel
    sa.pipeline_config.task_type = ModelTaskType.T2I
    return sa
```
**EN:** This function drives `_make_validation_server_args` with inputs such as `enable_cfg_parallel`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_make_validation_server_args`，主要处理 `enable_cfg_parallel` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 76-100: Class `TestWarmupReqCfgParallel` / 类 `TestWarmupReqCfgParallel`
```python
class TestWarmupReqCfgParallel(unittest.TestCase):
    """Commit 1 regression: prepare_server_warmup_reqs."""

    def test_warmup_req_cfg_parallel_sets_do_cfg(self):
        scheduler = _make_bare_scheduler(enable_cfg_parallel=True)
        scheduler.prepare_server_warmup_reqs()

        self.assertEqual(len(scheduler.waiting_queue), 1)
        _, req, _ = scheduler.waiting_queue[0]
        self.assertIs(req.do_classifier_free_guidance, True)
        self.assertEqual(req.negative_prompt, DEFAULT_PLACEHOLDER_PROMPT)

    def test_warmup_req_no_cfg_parallel_unchanged(self):
        # Regression guard: the cfg-parallel=on fix must not bleed into
# ...
        self.assertEqual(len(scheduler.waiting_queue), 1)
        _, req, _ = scheduler.waiting_queue[0]
        self.assertIs(req.do_classifier_free_guidance, False)
        self.assertNotEqual(req.negative_prompt, DEFAULT_PLACEHOLDER_PROMPT)
```
**EN:** This class models `TestWarmupReqCfgParallel` as a specialization of `unittest.TestCase`. Commit 1 regression: prepare_server_warmup_reqs. Important methods include `test_warmup_req_cfg_parallel_sets_do_cfg`, `test_warmup_req_no_cfg_parallel_unchanged`.
**CN:** 该类实现 `TestWarmupReqCfgParallel`，并继承/扩展 `unittest.TestCase`。 文档字符串指出：Commit 1 regression: prepare_server_warmup_reqs. 其中较重要的方法包括 `test_warmup_req_cfg_parallel_sets_do_cfg`, `test_warmup_req_no_cfg_parallel_unchanged`。

### Lines 103-180: Class `TestInputValidationCfgParallelGuard` / 类 `TestInputValidationCfgParallelGuard`
```python
class TestInputValidationCfgParallelGuard(unittest.TestCase):
    """Commit 2: per-request cfg-parallel check.

    Both tests patch _generate_seeds (the first statement of
    InputValidationStage.forward, input_validation.py:274) to sidestep
    its device-lookup / generator-creation code which pulls in torch
    CUDA bindings — keeps the suite strictly CPU-only. We still need
    num_inference_steps on the Req because the stage's
    "num_inference_steps <= 0" check at L305-308 raises TypeError on
    None before the new commit-2 check is reached.
    """

    def test_input_validation_rejects_cfg_parallel_without_cfg(self):
        # negative_prompt="" (non-None) ensures the existing
# ...
            try:
                stage.forward(req, server_args)
            except ValueError as e:
                self.fail(f"forward() raised ValueError on a valid CFG request: {e}")
```
**EN:** This class models `TestInputValidationCfgParallelGuard` as a specialization of `unittest.TestCase`. Commit 2: per-request cfg-parallel check. Important methods include `test_input_validation_rejects_cfg_parallel_without_cfg`, `test_input_validation_passes_cfg_parallel_with_cfg`.
**CN:** 该类实现 `TestInputValidationCfgParallelGuard`，并继承/扩展 `unittest.TestCase`。 文档字符串指出：Commit 2: per-request cfg-parallel check. 其中较重要的方法包括 `test_input_validation_rejects_cfg_parallel_without_cfg`, `test_input_validation_passes_cfg_parallel_with_cfg`。

### Lines 181-184: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Distributed execution / 分布式执行
- Automated verification / 自动化验证

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.runtime.managers.scheduler`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.input_validation`
- **External / 外部**: `unittest`, `unittest.mock`
- **Stdlib / 标准库**: `collections`
