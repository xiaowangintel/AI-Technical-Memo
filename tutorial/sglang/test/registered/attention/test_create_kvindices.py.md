# test_create_kvindices.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/attention/test_create_kvindices.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on attention create kvindices in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 attention create kvindices 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Import dependencies
```python
import unittest

import numpy as np
import torch

from sglang.srt.layers.attention.utils import create_flashinfer_kv_indices_triton
from sglang.srt.utils import get_device
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 12-13: Register CI metadata
```python
register_cuda_ci(est_time=7, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=10, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 16-16: Define class TestCreateKvIndices
```python
class TestCreateKvIndices(CustomTestCase):
```
**EN:** This declaration introduces the `TestCreateKvIndices` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCreateKvIndices` 测试类，并说明它通过继承承担的职责。

### Lines 17-19: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        torch.set_default_device(get_device())
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 21-69: Define helper: run test
```python
    def _run_test(self, batch, max_batch, max_context_len):
        req_to_token = torch.arange(
            max_batch * max_context_len, dtype=torch.int32, device=get_device()
        ).reshape((max_batch, max_context_len))
        req_pool_indices = torch.tensor(
            torch.from_numpy(
                np.random.choice(range(max_batch), size=batch, replace=False)
            ),
            dtype=torch.int32,
            device=get_device(),
        )
        paged_kernel_lens = torch.tensor(
            torch.from_numpy(
                np.random.choice(range(max_context_len), size=batch, replace=False)
            ),
            dtype=torch.int32,
            device=get_device(),
        )

        kv_indptr = torch.zeros((batch + 1,), dtype=torch.int32, device=get_device())
        kv_indptr[1:] = torch.cumsum(paged_kernel_lens, dim=0)

        # ref
        req_pool_indices_cpu = req_pool_indices.cpu().numpy()
        paged_kernel_lens_cpu = paged_kernel_lens.cpu().numpy()
        kv_indices_ref = torch.cat(
            [
                req_to_token[req_pool_indices_cpu[i], : paged_kernel_lens_cpu[i]]
                for i in range(batch)
            ],
            dim=0,
        ).contiguous()

        # triton
        kv_indices_triton = torch.empty(
            kv_indptr[-1], dtype=torch.int32, device=get_device()
        )
        create_flashinfer_kv_indices_triton[(batch,)](
            req_to_token,
            req_pool_indices,
            paged_kernel_lens,
            kv_indptr,
            None,
            kv_indices_triton,
            req_to_token.size(1),
        )

        # Check
        self.assertTrue(torch.equal(kv_indices_ref, kv_indices_triton))
```
**EN:** This helper function encapsulates reusable logic inside `TestCreateKvIndices` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestCreateKvIndices` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 71-76: Run test: create kvindices
```python
    def test_create_kvindices(self):
        BATCH = [1, 37, 1786]
        MAX_BATCH = 4096
        MAX_CONTEXT_LEN = 4096
        for batch in BATCH:
            self._run_test(batch, MAX_BATCH, MAX_CONTEXT_LEN)
```
**EN:** This test method exercises create kvindices and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 create kvindices 场景，并验证观测到的行为是否符合预期契约。

### Lines 79-80: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.layers.attention.utils`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `numpy`, `torch`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `CustomTestCase`, `unittest.main`
