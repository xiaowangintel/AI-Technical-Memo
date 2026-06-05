# test_weight_checker_e2e.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/rl/test_weight_checker_e2e.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates weight checker e2e behavior in SGLang's rl area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 rl 领域中与 weight checker e2e 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: supporting source context / 辅助源码上下文
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 14-20: supporting statements / 辅助语句
```python
"""End-to-end test for the /weights_checker HTTP endpoint.

Exercises the full HTTP -> tokenizer_manager -> scheduler -> model_runner ->
WeightChecker chain on a real engine. Unit tests in
test/registered/unit/utils/test_weight_checker.py cover the in-module
logic; this file is the thin integration cover plus interaction with
update_weights_from_tensor."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 22-35: module imports and dependencies / 模块导入与依赖
```python
import unittest
from typing import List, Tuple

import requests
import torch

from sglang.srt.utils import MultiprocessingSerializer, kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `typing`, `requests`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `typing`, `requests`, `torch`。

### Lines 37-45: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=150, suite="nightly-1-gpu", nightly=True)

_MODEL_NAME = "Qwen/Qwen3-0.6B"
# We address the up half via the HF-style unfused name "up_proj.weight". sglang's
# stacked_params_mapping rewrites this to "gate_up_proj.weight" with shard_id=1,
# so the upload writes only the up half of the fused tensor. Sending the fused
# name directly hits a name.replace() collision (gate_up_proj contains up_proj),
# producing a malformed key like "gate_gate_up_proj.weight" and crashing load.
_UP_PROJ_SHAPE = (3072, 1024)  # intermediate_size, hidden_size for Qwen3-0.6B
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 48-48: class TestWeightCheckerE2E declaration / 类 TestWeightCheckerE2E 声明
```python
class TestWeightCheckerE2E(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 49-53: supporting statements / 辅助语句
```python
    """All cases share one launched server (setUpClass).

    The reset case mutates weights to random; it is named to sort last so any
    case that needs intact weights runs first. The server is torn down right
    after, so leaving the engine in a corrupted state is harmless."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 55-69: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.url = DEFAULT_URL_FOR_TEST
        # --mem-fraction-static 0.7 leaves enough free GPU for _check_tensors's
        # CPU->GPU round trip: snapshot lives on CPU, then _compare moves each
        # snapshot tensor back to GPU for byte equality. With the default 0.88,
        # sglang holds ~29GB on a 32GB GPU and only ~200MB is free, so the
        # vocab-embedding round-trip (~600MB) OOMs the snapshot/reset/compare
        # cycle in test_z_*.
        cls.process = popen_launch_server(
            _MODEL_NAME,
            cls.url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=["--mem-fraction-static", "0.7"],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 71-73: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 75-78: method post / 方法 post
```python
    def _post(self, action: str) -> requests.Response:
        return requests.post(
            f"{self.url}/weights_checker", json={"action": action}, timeout=120
        )
```
**EN:** This block implements `_post` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_post`，承担模块行为中的一个聚焦逻辑片段。

### Lines 80-92: method update weights / 方法 update weights
```python
    def _update_weights(
        self, named_tensors: List[Tuple[str, torch.Tensor]]
    ) -> requests.Response:
        return requests.post(
            f"{self.url}/update_weights_from_tensor",
            json={
                "serialized_named_tensors": [
                    MultiprocessingSerializer.serialize(named_tensors, output_str=True)
                ],
                "flush_cache": True,
            },
            timeout=120,
        )
```
**EN:** This block implements `_update_weights` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_update_weights`，承担模块行为中的一个聚焦逻辑片段。

### Lines 94-101: test case a snapshot then compare unchanged succeeds / 测试用例 a snapshot then compare unchanged succeeds
```python
    def test_a_snapshot_then_compare_unchanged_succeeds(self):
        resp = self._post("snapshot")
        self.assertEqual(resp.status_code, 200)
        self.assertTrue(resp.json()["success"])

        resp = self._post("compare")
        self.assertEqual(resp.status_code, 200)
        self.assertTrue(resp.json()["success"])
```
**EN:** This test exercises `test_a_snapshot_then_compare_unchanged_succeeds` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_a_snapshot_then_compare_unchanged_succeeds`。

### Lines 103-106: test case b unknown action returns 400 / 测试用例 b unknown action returns 400
```python
    def test_b_unknown_action_returns_400(self):
        resp = self._post("nonsense_action")
        self.assertEqual(resp.status_code, 400)
        self.assertIn("Unsupported", resp.json()["message"])
```
**EN:** This test exercises `test_b_unknown_action_returns_400` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_b_unknown_action_returns_400`。

### Lines 108-126: test case c update with diff tensor makes compare fail / 测试用例 c update with diff tensor makes compare fail
```python
    def test_c_update_with_diff_tensor_makes_compare_fail(self):
        """A snapshot then an update with new bytes must make compare fail."""
        self.assertEqual(self._post("snapshot").status_code, 200)

        # The unfused HF name "up_proj" is what update_weights_from_tensor accepts;
        # sglang's loader rewrites it onto the fused gate_up_proj tensor.
        upload_name = "model.layers.5.mlp.up_proj.weight"
        new_tensor = torch.full(_UP_PROJ_SHAPE, 1.5, device="cuda")
        update_resp = self._update_weights([(upload_name, new_tensor)])
        self.assertEqual(update_resp.status_code, 200)
        self.assertTrue(update_resp.json()["success"])

        resp = self._post("compare")
        self.assertEqual(resp.status_code, 400)
        body = resp.json()
        self.assertFalse(body["success"])
        # The error references the fused on-device parameter name, not the upload alias.
        self.assertIn("model.layers.5.mlp.gate_up_proj.weight", body["message"])
        self.assertIn("max_abs_err", body["message"])
```
**EN:** A snapshot then an update with new bytes must make compare fail. This test exercises `test_c_update_with_diff_tensor_makes_compare_fail` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** A snapshot then an update with new bytes must make compare fail. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_c_update_with_diff_tensor_makes_compare_fail`。

### Lines 128-146: test case d update with same tensor keeps compare passing / 测试用例 d update with same tensor keeps compare passing
```python
    def test_d_update_with_same_tensor_keeps_compare_passing(self):
        """Prime a param, snapshot, push the same bytes again, compare must pass."""
        param_name = "model.layers.6.mlp.up_proj.weight"
        same_tensor = torch.full(_UP_PROJ_SHAPE, 0.25, device="cuda")

        # Step 1: prime the param to a known value.
        self.assertTrue(
            self._update_weights([(param_name, same_tensor)]).json()["success"]
        )
        # Step 2: snapshot the now-primed state.
        self.assertEqual(self._post("snapshot").status_code, 200)
        # Step 3: push the exact same bytes again — should be a byte-perfect no-op.
        self.assertTrue(
            self._update_weights([(param_name, same_tensor)]).json()["success"]
        )
        # Step 4: compare passes.
        resp = self._post("compare")
        self.assertEqual(resp.status_code, 200)
        self.assertTrue(resp.json()["success"])
```
**EN:** Prime a param, snapshot, push the same bytes again, compare must pass. This test exercises `test_d_update_with_same_tensor_keeps_compare_passing` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Prime a param, snapshot, push the same bytes again, compare must pass. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_d_update_with_same_tensor_keeps_compare_passing`。

### Lines 148-181: test case e checksum returns ranks with hashes / 测试用例 e checksum returns ranks with hashes
```python
    def test_e_checksum_returns_ranks_with_hashes(self):
        """checksum action must yield a ranks list with hex hashes per rank."""
        resp = self._post("checksum")
        self.assertEqual(resp.status_code, 200)
        body = resp.json()
        self.assertTrue(body["success"])
        self.assertIn("ranks", body)
        ranks = body["ranks"]
        self.assertIsInstance(ranks, list)
        self.assertGreaterEqual(len(ranks), 1)

        first = ranks[0]
        self.assertIn("checksums", first)
        self.assertIn("parallelism_info", first)

        info = first["parallelism_info"]
        for key in (
            "tp_rank",
            "tp_size",
            "dp_rank",
            "dp_size",
            "pp_rank",
            "pp_size",
            "rank",
            "size",
        ):
            self.assertIn(key, info)

        checksums = first["checksums"]
        self.assertGreater(len(checksums), 0)
        for name, h in checksums.items():
            self.assertIsInstance(h, str)
            self.assertEqual(len(h), 16, f"unexpected hash length for {name!r}: {h!r}")
            int(h, 16)
```
**EN:** checksum action must yield a ranks list with hex hashes per rank. This test exercises `test_e_checksum_returns_ranks_with_hashes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** checksum action must yield a ranks list with hex hashes per rank. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_e_checksum_returns_ranks_with_hashes`。

### Lines 183-187: test case e checksum is stable across calls / 测试用例 e checksum is stable across calls
```python
    def test_e_checksum_is_stable_across_calls(self):
        """Two consecutive checksum calls with no weight update must match."""
        first = self._post("checksum").json()["ranks"]
        second = self._post("checksum").json()["ranks"]
        self.assertEqual(first, second)
```
**EN:** Two consecutive checksum calls with no weight update must match. This test exercises `test_e_checksum_is_stable_across_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Two consecutive checksum calls with no weight update must match. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_e_checksum_is_stable_across_calls`。

### Lines 189-204: test case e checksum changes after weight update / 测试用例 e checksum changes after weight update
```python
    def test_e_checksum_changes_after_weight_update(self):
        """Updating a tensor must change its corresponding hash."""
        param_name = "model.layers.7.mlp.up_proj.weight"
        fused_name = "model.layers.7.mlp.gate_up_proj.weight"

        before = self._post("checksum").json()["ranks"][0]["checksums"]
        before_hash = before.get(fused_name)
        self.assertIsNotNone(before_hash, f"missing {fused_name!r} in checksum keys")

        new_tensor = torch.full(_UP_PROJ_SHAPE, 0.5, device="cuda")
        self.assertTrue(
            self._update_weights([(param_name, new_tensor)]).json()["success"]
        )

        after = self._post("checksum").json()["ranks"][0]["checksums"]
        self.assertNotEqual(after[fused_name], before_hash)
```
**EN:** Updating a tensor must change its corresponding hash. This test exercises `test_e_checksum_changes_after_weight_update` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Updating a tensor must change its corresponding hash. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_e_checksum_changes_after_weight_update`。

### Lines 206-214: test case e checksum skips non persistent buffers / 测试用例 e checksum skips non persistent buffers
```python
    def test_e_checksum_skips_non_persistent_buffers(self):
        """No checksum entry should contain a non-persistent-buffer substring."""
        ranks = self._post("checksum").json()["ranks"]
        for rank in ranks:
            for name in rank["checksums"]:
                self.assertNotIn("cos_sin_cache", name)
                self.assertNotIn("inv_freq", name)
                self.assertNotIn("freqs_cis", name)
                self.assertNotIn("_weight_fp32", name)
```
**EN:** No checksum entry should contain a non-persistent-buffer substring. This test exercises `test_e_checksum_skips_non_persistent_buffers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** No checksum entry should contain a non-persistent-buffer substring. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_e_checksum_skips_non_persistent_buffers`。

### Lines 216-225: test case z snapshot reset compare detects diff / 测试用例 z snapshot reset compare detects diff
```python
    def test_z_snapshot_reset_compare_detects_diff(self):
        """Destructive: leaves weights randomized. Named test_z_* so it runs last."""
        self.assertEqual(self._post("snapshot").status_code, 200)
        self.assertEqual(self._post("reset_tensors").status_code, 200)

        resp = self._post("compare")
        self.assertEqual(resp.status_code, 400)
        body = resp.json()
        self.assertFalse(body["success"])
        self.assertIn("max_abs_err", body["message"])
```
**EN:** Destructive: leaves weights randomized. This test exercises `test_z_snapshot_reset_compare_detects_diff` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Destructive: leaves weights randomized. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_z_snapshot_reset_compare_detects_diff`。

### Lines 228-229: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestWeightCheckerE2E`: All cases share one launched server (setUpClass). / 用于组织相关测试、夹具或辅助方法。
- `TestWeightCheckerE2E.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestWeightCheckerE2E.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestWeightCheckerE2E._post`: This block implements `_post` and captures one focused piece of the module's behavior. / 该代码块实现 `_post`，承担模块行为中的一个聚焦逻辑片段。
- `TestWeightCheckerE2E._update_weights`: This block implements `_update_weights` and captures one focused piece of the module's behavior. / 该代码块实现 `_update_weights`，承担模块行为中的一个聚焦逻辑片段。
- `TestWeightCheckerE2E.test_a_snapshot_then_compare_unchanged_succeeds`: This test exercises `test_a_snapshot_then_compare_unchanged_succeeds` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_a_snapshot_then_compare_unchanged_succeeds`。
- `TestWeightCheckerE2E.test_b_unknown_action_returns_400`: This test exercises `test_b_unknown_action_returns_400` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_b_unknown_action_returns_400`。
- `TestWeightCheckerE2E.test_c_update_with_diff_tensor_makes_compare_fail`: A snapshot then an update with new bytes must make compare fail. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_c_update_with_diff_tensor_makes_compare_fail`。
- `TestWeightCheckerE2E.test_d_update_with_same_tensor_keeps_compare_passing`: Prime a param, snapshot, push the same bytes again, compare must pass. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_d_update_with_same_tensor_keeps_compare_passing`。
- `TestWeightCheckerE2E.test_e_checksum_returns_ranks_with_hashes`: checksum action must yield a ranks list with hex hashes per rank. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_e_checksum_returns_ranks_with_hashes`。
- `TestWeightCheckerE2E.test_e_checksum_is_stable_across_calls`: Two consecutive checksum calls with no weight update must match. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_e_checksum_is_stable_across_calls`。
- `TestWeightCheckerE2E.test_e_checksum_changes_after_weight_update`: Updating a tensor must change its corresponding hash. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_e_checksum_changes_after_weight_update`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `typing`
- **Third-party modules / 第三方模块**: `requests`, `torch`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 229
