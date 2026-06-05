# test_scheduler_rollout_unit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/unit/test_scheduler_rollout_unit.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates scheduler rollout unit with focused assertions and fixtures. Key symbols include `_DummyScheduler`, `TestSchedulerRolloutOdeUnit`, `_flowgrpo_sde_step_with_logprob`. / 该测试模块通过有针对性的断言与夹具，验证 scheduler rollout unit 的实现。 关键符号包括 `_DummyScheduler`, `TestSchedulerRolloutOdeUnit`, `_flowgrpo_sde_step_with_logprob`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Imports and module setup / 导入与模块初始化
```python
import math
import types
import unittest

import torch

import sglang.multimodal_gen.runtime.post_training.scheduler_rl_mixin as rl_mixin_module
from sglang.multimodal_gen.runtime.post_training.scheduler_rl_mixin import (
    SchedulerRLMixin,
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 13-15: Class `_DummyScheduler` / 类 `_DummyScheduler`
```python
class _DummyScheduler(SchedulerRLMixin):
    def __init__(self):
        self.sigmas = torch.tensor([1.0, 0.8, 0.6, 0.4, 0.2, 0.0], dtype=torch.float32)
```
**EN:** This class models `_DummyScheduler` as a specialization of `SchedulerRLMixin`. Important methods include `__init__`.
**CN:** 该类实现 `_DummyScheduler`，并继承/扩展 `SchedulerRLMixin`。 其中较重要的方法包括 `__init__`。

### Lines 18-161: Class `TestSchedulerRolloutOdeUnit` / 类 `TestSchedulerRolloutOdeUnit`
```python
class TestSchedulerRolloutOdeUnit(unittest.TestCase):
    def setUp(self):
        self._orig_get_sp_world_size = rl_mixin_module.get_sp_world_size
        rl_mixin_module.get_sp_world_size = lambda: 1

    def tearDown(self):
        rl_mixin_module.get_sp_world_size = self._orig_get_sp_world_size

    def _build_batch(self, *, debug_mode: bool) -> types.SimpleNamespace:
        return types.SimpleNamespace(
            rollout_log_prob_no_const=True,
            rollout_noise_level=0.5,
            rollout_sde_type="ode",
            rollout_debug_mode=debug_mode,
# ...
        )
        self.assertTrue(
            torch.allclose(variance_noises, torch.zeros_like(variance_noises))
        )
```
**EN:** This class models `TestSchedulerRolloutOdeUnit` as a specialization of `unittest.TestCase`. Important methods include `setUp`, `tearDown`, `_build_batch`, `test_ode_step_does_not_call_variance_noise_sampler`.
**CN:** 该类实现 `TestSchedulerRolloutOdeUnit`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `setUp`, `tearDown`, `_build_batch`, `test_ode_step_does_not_call_variance_noise_sampler`。

### Lines 164-220: Function `_flowgrpo_sde_step_with_logprob` / 函数 `_flowgrpo_sde_step_with_logprob`
```python
def _flowgrpo_sde_step_with_logprob(
    *,
    model_output: torch.Tensor,
    sample: torch.Tensor,
    variance_noise: torch.Tensor,
    sigma: torch.Tensor,
    sigma_prev: torch.Tensor,
    sigma_max: float,
    noise_level: float,
    sde_type: str,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    """Verbatim from FlowGRPO sd3_sde_with_logprob.py ``sde_step_with_logprob``.

    Returns (prev_sample, log_prob, prev_sample_mean, noise_std_dev).
# ...
        raise ValueError(f"Unsupported sde_type: {sde_type}")

    log_prob = log_prob.mean(dim=tuple(range(1, log_prob.ndim)))
    return prev_sample, log_prob, prev_sample_mean, noise_std_dev
```
**EN:** This function drives `_flowgrpo_sde_step_with_logprob` with inputs such as `model_output`, `sample`, `variance_noise`, `sigma`. Verbatim from FlowGRPO sd3_sde_with_logprob.py ``sde_step_with_logprob``.
**CN:** 这个函数负责 `_flowgrpo_sde_step_with_logprob`，主要处理 `model_output`, `sample`, `variance_noise`, `sigma` 等输入。 文档字符串说明：Verbatim from FlowGRPO sd3_sde_with_logprob.py ``sde_step_with_logprob``.

### Lines 221-224: Top-level configuration / 顶层配置
```python


# FlowGRPO convention: SDE uses full Gaussian log-prob, CPS uses no_const.
_FLOWGRPO_LOG_PROB_NO_CONST = {"sde": False, "cps": True}
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 227-578: Class `TestSchedulerFlowGRPOStepAlignmentUnit` / 类 `TestSchedulerFlowGRPOStepAlignmentUnit`
```python
class TestSchedulerFlowGRPOStepAlignmentUnit(unittest.TestCase):
    def setUp(self):
        self._orig_get_sp_world_size = rl_mixin_module.get_sp_world_size
        rl_mixin_module.get_sp_world_size = lambda: 1

    def tearDown(self):
        rl_mixin_module.get_sp_world_size = self._orig_get_sp_world_size

    def _build_batch(
        self, *, sde_type: str, shape: tuple[int, ...]
    ) -> types.SimpleNamespace:
        return types.SimpleNamespace(
            rollout_log_prob_no_const=_FLOWGRPO_LOG_PROB_NO_CONST[sde_type],
            rollout_noise_level=0.5,
# ...
        )
        self.assertEqual(
            len(traj_only_final._rollout_denoising_env_state["step_timesteps"]), 1
        )
```
**EN:** This class models `TestSchedulerFlowGRPOStepAlignmentUnit` as a specialization of `unittest.TestCase`. Important methods include `setUp`, `tearDown`, `_build_batch`, `test_single_step_matches_flowgrpo_reference`.
**CN:** 该类实现 `TestSchedulerFlowGRPOStepAlignmentUnit`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `setUp`, `tearDown`, `_build_batch`, `test_single_step_matches_flowgrpo_reference`。

### Lines 579-582: Top-level configuration / 顶层配置
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
- Sampling parameter control / 采样参数控制
- Automated verification / 自动化验证

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.post_training.scheduler_rl_mixin`, `sglang.multimodal_gen.runtime.post_training.rollout_denoising_mixin`
- **External / 外部**: `unittest`, `torch`
- **Stdlib / 标准库**: `math`, `types`
