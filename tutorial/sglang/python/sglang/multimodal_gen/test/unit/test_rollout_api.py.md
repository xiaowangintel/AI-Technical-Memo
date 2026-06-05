# test_rollout_api.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/unit/test_rollout_api.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates rollout api with focused assertions and fixtures. Key symbols include `TestTensorToBase64Roundtrip`, `TestMaybeSerialize`, `TestSerializeRolloutTrajectory`. / 该测试模块通过有针对性的断言与夹具，验证 rollout api 的实现。 关键符号包括 `TestTensorToBase64Roundtrip`, `TestMaybeSerialize`, `TestSerializeRolloutTrajectory`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Imports and module setup / 导入与模块初始化
```python
"""Unit tests for the rollout generate API (serialization, io_struct, rollout_api)."""

import types
import unittest

import torch

from sglang.multimodal_gen.runtime.entrypoints.post_training.utils import (
    _maybe_deserialize,
    _maybe_serialize,
    base64_to_tensor,
    tensor_to_base64,
)
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import OutputBatch
# ...
    RolloutDenoisingEnv,
    RolloutDitTrajectory,
    RolloutTrajectoryData,
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 23-74: Class `TestTensorToBase64Roundtrip` / 类 `TestTensorToBase64Roundtrip`
```python
class TestTensorToBase64Roundtrip(unittest.TestCase):

    def _roundtrip(self, t: torch.Tensor):
        encoded = tensor_to_base64(t)
        self.assertIsInstance(encoded, str)
        decoded = base64_to_tensor(encoded)
        self.assertTrue(
            torch.equal(t, decoded), f"Mismatch for shape={t.shape} dtype={t.dtype}"
        )

    def test_float32_1d(self):
        self._roundtrip(torch.randn(16))

    def test_float32_nd(self):
# ...
        encoded = tensor_to_base64(t)
        decoded = base64_to_tensor(encoded)
        self.assertFalse(decoded.requires_grad)
        self.assertTrue(torch.equal(t.detach(), decoded))
```
**EN:** This class models `TestTensorToBase64Roundtrip` as a specialization of `unittest.TestCase`. Important methods include `_roundtrip`, `test_float32_1d`, `test_float32_nd`, `test_float16`.
**CN:** 该类实现 `TestTensorToBase64Roundtrip`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `_roundtrip`, `test_float32_1d`, `test_float32_nd`, `test_float16`。

### Lines 77-123: Class `TestMaybeSerialize` / 类 `TestMaybeSerialize`
```python
class TestMaybeSerialize(unittest.TestCase):
    def test_tensor(self):
        t = torch.randn(2, 3)
        result = _maybe_serialize(t)
        self.assertIsInstance(result, dict)
        self.assertTrue(result["__tensor__"])
        self.assertEqual(result["shape"], [2, 3])
        self.assertEqual(result["dtype"], "torch.float32")
        decoded = base64_to_tensor(result["data"])
        self.assertTrue(torch.equal(t, decoded))

    def test_dict_with_tensors(self):
        d = {"a": torch.tensor([1.0]), "b": "hello", "c": 42}
        result = _maybe_serialize(d)
# ...
    def test_tuple_becomes_list(self):
        result = _maybe_serialize((torch.tensor(1.0), 2))
        self.assertIsInstance(result, list)
        self.assertEqual(len(result), 2)
```
**EN:** This class models `TestMaybeSerialize` as a specialization of `unittest.TestCase`. Important methods include `test_tensor`, `test_dict_with_tensors`, `test_list_with_tensors`, `test_nested_structure`.
**CN:** 该类实现 `TestMaybeSerialize`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `test_tensor`, `test_dict_with_tensors`, `test_list_with_tensors`, `test_nested_structure`。

### Lines 124-129: Imports and module setup / 导入与模块初始化
```python


from sglang.multimodal_gen.runtime.entrypoints.post_training.rollout_api import (
    _build_response,
    _serialize_rollout_trajectory,
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 132-223: Class `TestSerializeRolloutTrajectory` / 类 `TestSerializeRolloutTrajectory`
```python
class TestSerializeRolloutTrajectory(unittest.TestCase):
    def test_none_input(self):
        log_probs, debug, env, dit_traj = _serialize_rollout_trajectory(None)
        self.assertIsNone(log_probs)
        self.assertIsNone(debug)
        self.assertIsNone(env)
        self.assertIsNone(dit_traj)

    def test_log_probs_only(self):
        rtd = RolloutTrajectoryData(
            rollout_log_probs=torch.tensor([-1.0, -2.0]),
        )
        log_probs, debug, env, dit_traj = _serialize_rollout_trajectory(rtd)
        self.assertIsNotNone(log_probs)
# ...
        self.assertIn("latents", dit_traj)
        self.assertIn("timesteps", dit_traj)
        self.assertTrue(dit_traj["latents"]["__tensor__"])
        self.assertTrue(dit_traj["timesteps"]["__tensor__"])
```
**EN:** This class models `TestSerializeRolloutTrajectory` as a specialization of `unittest.TestCase`. Important methods include `test_none_input`, `test_log_probs_only`, `test_log_probs_none_in_rtd`, `test_with_debug_tensors`.
**CN:** 该类实现 `TestSerializeRolloutTrajectory`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `test_none_input`, `test_log_probs_only`, `test_log_probs_none_in_rtd`, `test_with_debug_tensors`。

### Lines 226-356: Class `TestBuildResponse` / 类 `TestBuildResponse`
```python
class TestBuildResponse(unittest.TestCase):
    def _make_metrics(self, duration_s: float = 1.0):
        return types.SimpleNamespace(total_duration_s=duration_s)

    def test_minimal_output(self):
        batch = OutputBatch(
            output=torch.randn(1, 3, 1, 64, 64),
            rollout_trajectory_data=RolloutTrajectoryData(
                rollout_log_probs=torch.tensor([0.0]),
            ),
        )
        batch.metrics = self._make_metrics(2.5)
        resps = _build_response("r1", "prompt", 42, True, batch)
        self.assertEqual(len(resps), 1)
# ...
        self.assertEqual(len(resps), 2)
        self.assertIsNone(resps[0].rollout_log_probs)
        self.assertIsNone(resps[1].rollout_log_probs)
        self.assertIsNotNone(resps[0].generated_output)
```
**EN:** This class models `TestBuildResponse` as a specialization of `unittest.TestCase`. Important methods include `_make_metrics`, `test_minimal_output`, `test_full_response`, `test_no_metrics`.
**CN:** 该类实现 `TestBuildResponse`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `_make_metrics`, `test_minimal_output`, `test_full_response`, `test_no_metrics`。

### Lines 359-408: Class `TestBuildSamplingKwargs` / 类 `TestBuildSamplingKwargs`
```python
class TestBuildSamplingKwargs(unittest.TestCase):
    def _make_request(self, **overrides):
        from sglang.multimodal_gen.runtime.entrypoints.post_training.io_struct import (
            RolloutRequest,
        )

        base = dict(prompt="x", num_inference_steps=4, rollout=True)
        base.update(overrides)
        return RolloutRequest(**base)

    def test_step_index_filters_forwarded(self):
        from sglang.multimodal_gen.runtime.entrypoints.post_training.rollout_api import (
            _build_sampling_kwargs,
        )
# ...
        )
        req = Req(sampling_params=sp)
        self.assertEqual(req.rollout_sde_step_indices, [0, 2])
        self.assertEqual(req.rollout_return_step_indices, [1, 3])
```
**EN:** This class models `TestBuildSamplingKwargs` as a specialization of `unittest.TestCase`. Important methods include `_make_request`, `test_step_index_filters_forwarded`, `test_step_index_filters_default_dropped_as_none`, `test_sampling_params_exposes_filters_via_req_getattr`.
**CN:** 该类实现 `TestBuildSamplingKwargs`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `_make_request`, `test_step_index_filters_forwarded`, `test_step_index_filters_default_dropped_as_none`, `test_sampling_params_exposes_filters_via_req_getattr`。

### Lines 409-412: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Sampling parameter control / 采样参数控制
- Automated verification / 自动化验证
- Image generation flow / 图像生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.entrypoints.post_training.utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.post_training.rl_dataclasses`, `sglang.multimodal_gen.runtime.entrypoints.post_training.rollout_api`, `sglang.multimodal_gen.runtime.entrypoints.post_training.io_struct`, `sglang.multimodal_gen.configs.sample.sampling_params`
- **External / 外部**: `unittest`, `torch`
- **Stdlib / 标准库**: `types`
