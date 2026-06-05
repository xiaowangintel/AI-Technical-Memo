# test_flashinfer_fusion_preflight.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/distributed/test_flashinfer_fusion_preflight.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates flashinfer fusion preflight behavior in SGLang's distributed area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 分布式 领域中与 flashinfer fusion preflight 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Distributed tests for FlashInfer allreduce-fusion workspace preflight."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-12: module imports and dependencies / 模块导入与依赖
```python
import multiprocessing as mp
import os
import socket
import unittest

import torch

from sglang.srt.utils import get_cuda_driver_bindings, is_flashinfer_available
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `multiprocessing`, `os`, `socket`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `multiprocessing`, `os`, `socket`, `unittest`。

### Lines 14-16: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=30, stage="base-b", runner_config="2-gpu-large")

WORLD_SIZE = 2
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 19-22: function get free port / 函数 get free port
```python
def _get_free_port():
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
        sock.bind(("127.0.0.1", 0))
        return sock.getsockname()[1]
```
**EN:** This block implements `_get_free_port` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_get_free_port`，承担模块行为中的一个聚焦逻辑片段。

### Lines 25-91: function run rank / 函数 run rank
```python
def _run_rank(rank, world_size, port, scenario, result_q):
    held = None
    cuda_driver = None
    try:
        os.environ["MASTER_ADDR"] = "127.0.0.1"
        os.environ["MASTER_PORT"] = str(port)
        os.environ["RANK"] = str(rank)
        os.environ["WORLD_SIZE"] = str(world_size)
        os.environ["LOCAL_RANK"] = str(rank)

        torch.cuda.set_device(rank)

        import torch.distributed as dist

        dist.init_process_group(
            backend="gloo",
            rank=rank,
            world_size=world_size,
        )
        cpu_group = dist.group.WORLD

        from sglang.srt.layers.flashinfer_comm_fusion import (
            _make_flashinfer_workspace_allocation_prop,
            _preflight_check_workspace_memory,
        )

        probe_kwargs = dict(
            world_size=8,
            max_token_num=2048,
            hidden_dim=12288,
            dtype=torch.bfloat16,
            cpu_group=cpu_group,
        )

        if scenario == "rank0_starved" and rank == 0:
            cuda_driver = get_cuda_driver_bindings()
            prop = _make_flashinfer_workspace_allocation_prop(cuda_driver)

            free, _total = torch.cuda.mem_get_info(rank)
            target = max(free - (1 << 30), 0)
            granularity_flag = (
                cuda_driver.CUmemAllocationGranularity_flags.CU_MEM_ALLOC_GRANULARITY_RECOMMENDED
            )
            err, gran = cuda_driver.cuMemGetAllocationGranularity(
                prop,
                granularity_flag,
            )
            assert err == cuda_driver.CUresult.CUDA_SUCCESS, err
            aligned = (target // gran) * gran
            assert aligned > 0, "not enough free memory to starve the preflight"
            err, held = cuda_driver.cuMemCreate(aligned, prop, 0)
            assert err == cuda_driver.CUresult.CUDA_SUCCESS, (err, aligned)

        decision = _preflight_check_workspace_memory(**probe_kwargs)
        result_q.put((rank, "ok", bool(decision)))
    except Exception as e:  # pragma: no cover - debug path
        result_q.put((rank, "err", repr(e)))
    finally:
        if held is not None:
            cuda_driver.cuMemRelease(held)
        try:
            import torch.distributed as dist

            if dist.is_initialized():
                dist.destroy_process_group()
        except Exception:
            pass
```
**EN:** This block implements `_run_rank` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_rank`，承担模块行为中的一个聚焦逻辑片段。

### Lines 94-122: function spawn and collect / 函数 spawn and collect
```python
def _spawn_and_collect(scenario, world_size=WORLD_SIZE):
    ctx = mp.get_context("spawn")
    q = ctx.Queue()
    port = _get_free_port()
    procs = []
    for rank in range(world_size):
        proc = ctx.Process(
            target=_run_rank,
            args=(rank, world_size, port, scenario, q),
        )
        proc.start()
        procs.append(proc)

    try:
        results = {}
        for _ in range(world_size):
            rank, status, payload = q.get(timeout=300)
            results[rank] = (status, payload)

        for proc in procs:
            proc.join(timeout=60)
            assert proc.exitcode == 0, f"rank exited with {proc.exitcode}"
    finally:
        for proc in procs:
            if proc.is_alive():
                proc.terminate()
                proc.join(timeout=10)

    return results
```
**EN:** This block implements `_spawn_and_collect` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_spawn_and_collect`，承担模块行为中的一个聚焦逻辑片段。

### Lines 125-125: class TestFlashInferPreflightDistributed declaration / 类 TestFlashInferPreflightDistributed 声明
```python
class TestFlashInferPreflightDistributed(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 126-144: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        if not torch.cuda.is_available() or torch.cuda.device_count() < WORLD_SIZE:
            raise unittest.SkipTest(
                f"Need {WORLD_SIZE} CUDA devices, got {torch.cuda.device_count()}"
            )
        if not is_flashinfer_available():
            raise unittest.SkipTest("FlashInfer is not available")
        try:
            from sglang.srt.layers.flashinfer_comm_fusion import (
                _make_flashinfer_workspace_allocation_prop,
            )

            cuda_driver = get_cuda_driver_bindings()
            _make_flashinfer_workspace_allocation_prop(cuda_driver)
        except Exception as e:
            raise unittest.SkipTest(
                f"FlashInfer preflight dependencies unavailable: {e}"
            )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 146-150: test case happy path votes proceed / 测试用例 happy path votes proceed
```python
    def test_happy_path_votes_proceed(self):
        results = _spawn_and_collect("normal")
        for rank, (status, payload) in results.items():
            self.assertEqual(status, "ok", f"rank {rank}: {payload}")
            self.assertTrue(payload, f"rank {rank} voted SKIP unexpectedly")
```
**EN:** This test exercises `test_happy_path_votes_proceed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_happy_path_votes_proceed`。

### Lines 152-159: test case starved rank broadcasts skip / 测试用例 starved rank broadcasts skip
```python
    def test_starved_rank_broadcasts_skip(self):
        results = _spawn_and_collect("rank0_starved")
        for rank, (status, payload) in results.items():
            self.assertEqual(status, "ok", f"rank {rank}: {payload}")
            self.assertFalse(
                payload,
                f"rank {rank} voted PROCEED but rank 0 was starved",
            )
```
**EN:** This test exercises `test_starved_rank_broadcasts_skip` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_starved_rank_broadcasts_skip`。

### Lines 162-163: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_get_free_port`: This block implements `_get_free_port` and captures one focused piece of the module's behavior. / 该代码块实现 `_get_free_port`，承担模块行为中的一个聚焦逻辑片段。
- `_run_rank`: This block implements `_run_rank` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_rank`，承担模块行为中的一个聚焦逻辑片段。
- `_spawn_and_collect`: This block implements `_spawn_and_collect` and captures one focused piece of the module's behavior. / 该代码块实现 `_spawn_and_collect`，承担模块行为中的一个聚焦逻辑片段。
- `TestFlashInferPreflightDistributed`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFlashInferPreflightDistributed.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestFlashInferPreflightDistributed.test_happy_path_votes_proceed`: This test exercises `test_happy_path_votes_proceed` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_happy_path_votes_proceed`。
- `TestFlashInferPreflightDistributed.test_starved_rank_broadcasts_skip`: This test exercises `test_starved_rank_broadcasts_skip` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_starved_rank_broadcasts_skip`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `multiprocessing`, `os`, `socket`, `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 163
