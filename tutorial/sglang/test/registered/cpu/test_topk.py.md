# test_topk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_topk.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu topk in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 cpu topk 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import unittest

import torch

from sglang.srt.eplb.expert_location_dispatch import ExpertLocationDispatchInfo
from sglang.srt.layers.moe.topk import (
    biased_grouped_topk_impl as native_biased_grouped_topk,
)
from sglang.srt.layers.moe.topk import biased_topk_impl as native_biased_topk
from sglang.srt.layers.moe.topk import fused_topk_torch_native as native_fused_topk
from sglang.srt.layers.moe.topk import grouped_topk_gpu as native_grouped_topk
from sglang.srt.models.llama4 import Llama4MoE
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 16-16: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-b-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 18-18: Implement expr logic
```python
torch.manual_seed(1234)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 22-22: Define class TestGroupedTopK
```python
class TestGroupedTopK(CustomTestCase):
```
**EN:** This declaration introduces the `TestGroupedTopK` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGroupedTopK` 测试类，并说明它通过继承承担的职责。

### Lines 23-56: Define helper: run single test
```python
    def _run_single_test(self, M, E, G, topk, topk_group, renormalize, dtype):
        torch.manual_seed(1234)

        # expand gating_output by M, otherwise bfloat16 fall into same value aftering truncating
        hidden_states = torch.randn(M, 100, dtype=dtype)
        gating_output = torch.randn(M, E, dtype=dtype) * 2 * M

        ref_topk_weights, ref_topk_ids = native_grouped_topk(
            hidden_states.float(),
            gating_output.float(),
            topk,
            renormalize,
            G,
            topk_group,
        )

        # fused version
        topk_weights, topk_ids = torch.ops.sgl_kernel.grouped_topk_cpu(
            hidden_states,
            gating_output,
            topk,
            renormalize,
            G,
            topk_group,
            0,
            None,
            None,
        )

        res = torch.zeros(M, E, dtype=torch.float)
        ref = torch.zeros(M, E, dtype=torch.float)
        res.scatter_(1, topk_ids.long(), topk_weights)
        ref.scatter_(1, ref_topk_ids.long(), ref_topk_weights)
        torch.testing.assert_close(res, ref)
```
**EN:** This helper function encapsulates reusable logic inside `TestGroupedTopK` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGroupedTopK` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 58-66: Run test: grouped topk
```python
    def test_grouped_topk(self):
        for renormalize in [True, False]:
            self._run_single_test(123, 8, 2, 2, 1, renormalize, torch.bfloat16)
            self._run_single_test(123, 16, 4, 3, 2, renormalize, torch.bfloat16)
            self._run_single_test(123, 32, 4, 3, 2, renormalize, torch.bfloat16)
            self._run_single_test(1123, 32, 4, 3, 2, renormalize, torch.bfloat16)
            self._run_single_test(123, 64, 1, 6, 1, renormalize, torch.bfloat16)
            self._run_single_test(123, 256, 8, 4, 8, renormalize, torch.bfloat16)
            self._run_single_test(123, 160, 8, 6, 2, renormalize, torch.bfloat16)
```
**EN:** This test method exercises grouped topk and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 grouped topk 场景，并验证观测到的行为是否符合预期契约。

### Lines 70-70: Define class TestBiasedGroupedTopK
```python
class TestBiasedGroupedTopK(CustomTestCase):
```
**EN:** This declaration introduces the `TestBiasedGroupedTopK` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBiasedGroupedTopK` 测试类，并说明它通过继承承担的职责。

### Lines 71-122: Define helper: run single test
```python
    def _run_single_test(
        self,
        M,
        E,
        G,
        topk,
        topk_group,
        renormalize,
        gating_dtype,
        bias_dtype,
        routed_scaling_factor,
    ):
        torch.manual_seed(1024)

        # expand gating_output by M, otherwise bfloat16 fall into same value aftering truncating
        hidden_states = torch.randn(M, 100, dtype=torch.bfloat16)
        gating_output = torch.randn(M, E, dtype=gating_dtype) * 2 * M
        correction_bias = torch.randn(E, dtype=bias_dtype)

        ref_topk_weights, ref_topk_ids = native_biased_grouped_topk(
            hidden_states.float(),
            gating_output.float(),
            correction_bias.float(),
            topk,
            renormalize,
            G,
            topk_group,
        )
        ref_topk_weights = (
            ref_topk_weights * routed_scaling_factor
            if routed_scaling_factor is not None
            else ref_topk_weights
        )
        # fused version
        topk_weights, topk_ids = torch.ops.sgl_kernel.biased_grouped_topk_cpu(
            hidden_states,
            gating_output,
            correction_bias,
            topk,
            renormalize,
            G,
            topk_group,
            0,
            routed_scaling_factor,
            None,
        )

        res = torch.zeros(M, E, dtype=torch.float)
        ref = torch.zeros(M, E, dtype=torch.float)
        res.scatter_(1, topk_ids.long(), topk_weights)
        ref.scatter_(1, ref_topk_ids.long(), ref_topk_weights)
        torch.testing.assert_close(res, ref)
```
**EN:** This helper function encapsulates reusable logic inside `TestBiasedGroupedTopK` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestBiasedGroupedTopK` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 124-140: Run test: biased grouped topk
```python
    def test_biased_grouped_topk(self):
        for renormalize in [False]:
            for bias_dtype in [torch.float32, torch.bfloat16]:
                for gating_dtype in [torch.float32, torch.bfloat16]:
                    for routed_scaling_factor in [None, 1.125]:
                        for E_num in [128, 192, 256, 384]:
                            self._run_single_test(
                                34,
                                E_num,
                                8,
                                8,
                                2,
                                renormalize,
                                gating_dtype,
                                bias_dtype,
                                routed_scaling_factor,
                            )
```
**EN:** This test method exercises biased grouped topk and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 biased grouped topk 场景，并验证观测到的行为是否符合预期契约。

### Lines 143-143: Define class TestBiasedTopK
```python
class TestBiasedTopK(CustomTestCase):
```
**EN:** This declaration introduces the `TestBiasedTopK` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBiasedTopK` 测试类，并说明它通过继承承担的职责。

### Lines 144-172: Run test: biased topk returns logical ids with eplb info
```python
    def test_biased_topk_returns_logical_ids_with_eplb_info(self):
        hidden_states = torch.ones(1, 4)
        gating_output = torch.tensor([[10.0, 9.0, 1.0, 0.0]])
        correction_bias = torch.zeros(4)
        dispatch_info = ExpertLocationDispatchInfo(
            ep_dispatch_algorithm="static",
            partial_logical_to_rank_dispatch_physical_map=torch.tensor(
                [2, 3, 0, 1], dtype=torch.int64
            ),
            partial_logical_to_all_physical_map=torch.tensor(
                [[2], [3], [0], [1]], dtype=torch.int64
            ),
            partial_logical_to_all_physical_map_num_valid=torch.ones(
                4, dtype=torch.int64
            ),
            num_physical_experts=4,
        )

        _, topk_ids = native_biased_topk(
            hidden_states=hidden_states,
            gating_output=gating_output,
            correction_bias=correction_bias,
            topk=2,
            renormalize=False,
            scoring_func="sqrtsoftplus",
            expert_location_dispatch_info=dispatch_info,
        )

        torch.testing.assert_close(topk_ids, torch.tensor([[0, 1]], dtype=torch.int32))
```
**EN:** This test method exercises biased topk returns logical ids with eplb info and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 biased topk returns logical ids with eplb info 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 175-175: Define class TestTopK
```python
class TestTopK(CustomTestCase):
```
**EN:** This declaration introduces the `TestTopK` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestTopK` 测试类，并说明它通过继承承担的职责。

### Lines 176-199: Define helper: run single test
```python
    def _run_single_test(self, M, E, topk, renormalize, dtype):
        torch.manual_seed(1998)

        # expand gating_output by M, otherwise bfloat16 fall into same value aftering truncating
        hidden_states = torch.randn(M, 100, dtype=dtype)
        gating_output = torch.randn(M, E, dtype=dtype) * 2 * M

        ref_topk_weights, ref_topk_ids = native_fused_topk(
            hidden_states.float(),
            gating_output.float(),
            topk,
            renormalize,
        )

        # fused version
        topk_weights, topk_ids = torch.ops.sgl_kernel.topk_softmax_cpu(
            hidden_states, gating_output, topk, renormalize
        )

        res = torch.zeros(M, E, dtype=torch.float)
        ref = torch.zeros(M, E, dtype=torch.float)
        res.scatter_(1, topk_ids.long(), topk_weights)
        ref.scatter_(1, ref_topk_ids.long(), ref_topk_weights)
        torch.testing.assert_close(res, ref)
```
**EN:** This helper function encapsulates reusable logic inside `TestTopK` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestTopK` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 201-209: Run test: topk
```python
    def test_topk(self):
        for renormalize in [True, False]:
            self._run_single_test(123, 8, 2, renormalize, torch.bfloat16)
            self._run_single_test(123, 16, 3, renormalize, torch.bfloat16)
            self._run_single_test(123, 32, 3, renormalize, torch.bfloat16)
            self._run_single_test(123, 32, 3, renormalize, torch.bfloat16)
            self._run_single_test(123, 64, 6, renormalize, torch.bfloat16)
            self._run_single_test(123, 256, 4, renormalize, torch.bfloat16)
            self._run_single_test(123, 160, 6, renormalize, torch.bfloat16)
```
**EN:** This test method exercises topk and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 topk 场景，并验证观测到的行为是否符合预期契约。

### Lines 212-212: Define class TestCustomTopK
```python
class TestCustomTopK(CustomTestCase):
```
**EN:** This declaration introduces the `TestCustomTopK` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCustomTopK` 测试类，并说明它通过继承承担的职责。

### Lines 213-238: Define helper: run single test
```python
    def _run_single_test(
        self, M, E, topk, renormalize, dtype, native_custom_f, fused_custom_f
    ):
        torch.manual_seed(16)

        # expand gating_output by M, otherwise bfloat16 fall into same value aftering truncating
        hidden_states = torch.randn(M, 100, dtype=dtype)
        gating_output = torch.randn(M, E, dtype=dtype) * 2 * M

        ref_topk_weights, ref_topk_ids = native_custom_f(
            hidden_states.float(),
            gating_output.float(),
            topk,
            renormalize,
        )

        # fused version
        topk_weights, topk_ids = fused_custom_f(
            hidden_states, gating_output, topk, renormalize
        )

        res = torch.zeros(M, E, dtype=torch.float)
        ref = torch.zeros(M, E, dtype=torch.float)
        res.scatter_(1, topk_ids.long(), topk_weights)
        ref.scatter_(1, ref_topk_ids.long(), ref_topk_weights)
        torch.testing.assert_close(res, ref)
```
**EN:** This helper function encapsulates reusable logic inside `TestCustomTopK` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestCustomTopK` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 240-253: Run test: custom topk
```python
    def test_custom_topk(self):
        test_custom_functions = [
            (Llama4MoE.custom_routing_function, torch.ops.sgl_kernel.topk_sigmoid_cpu)
        ]
        for native_custom_f, fused_custom_f in test_custom_functions:
            self._run_single_test(
                123, 8, 1, False, torch.bfloat16, native_custom_f, fused_custom_f
            )
            self._run_single_test(
                123, 16, 1, False, torch.bfloat16, native_custom_f, fused_custom_f
            )
            self._run_single_test(
                123, 32, 1, False, torch.bfloat16, native_custom_f, fused_custom_f
            )
```
**EN:** This test method exercises custom topk and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 custom topk 场景，并验证观测到的行为是否符合预期契约。

### Lines 256-257: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.eplb.expert_location_dispatch`, `sglang.srt.layers.moe.topk`, `sglang.srt.models.llama4`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `torch`, `unittest`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
