# test_nsa_pool_host_unit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/mem_cache/test_nsa_pool_host_unit.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates nsa pool host unit behavior in SGLang's unit / mem cache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 内存缓存 领域中与 nsa pool host unit 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang.srt.mem_cache.memory_pool import NSATokenToKVPool
from sglang.srt.mem_cache.memory_pool_host import (
    ALLOC_MEMORY_FUNCS,
    MLATokenToKVPoolHost,
    NSAIndexerPoolHost,
    alloc_with_pin_memory,
)
from sglang.srt.utils import is_cuda, is_hip, is_npu, is_xpu
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.memory_pool_host`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.memory_pool_host`。

### Lines 15-15: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=9, stage="base-b", runner_config="1-gpu-small")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 18-18: class TestNSAHiCacheTransfer declaration / 类 TestNSAHiCacheTransfer 声明
```python
class TestNSAHiCacheTransfer(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 19-25: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        if not torch.cuda.is_available():
            self.skipTest("CUDA is required for NSA host transfer tests.")
        if is_npu() or is_xpu():
            self.skipTest("NSA host transfer tests only support CUDA/ROCm.")
        if not (is_cuda() or is_hip()):
            self.skipTest("CUDA/ROCm not available.")
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 27-38: method token indices for pages / 方法 token indices for pages
```python
    @staticmethod
    def _token_indices_for_pages(pages: torch.Tensor, page_size: int, device: str):
        parts = [
            torch.arange(
                int(page_id) * page_size,
                (int(page_id) + 1) * page_size,
                device=device,
                dtype=torch.int64,
            )
            for page_id in pages.tolist()
        ]
        return torch.cat(parts, dim=0)
```
**EN:** This block implements `_token_indices_for_pages` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_token_indices_for_pages`，承担模块行为中的一个聚焦逻辑片段。

### Lines 40-119: method run device to host indexer copy (part 1/2) / 方法 run device to host indexer copy（第 1/2 部分）
```python
    def _run_device_to_host_indexer_copy(self, io_backend: str):
        page_size = 1 if is_hip() else 64
        layer_num = 2
        size = page_size * 4

        device_pool = NSATokenToKVPool(
            size=size,
            page_size=page_size,
            kv_lora_rank=128,
            dtype=torch.bfloat16,
            qk_rope_head_dim=32,
            layer_num=layer_num,
            device="cuda",
            enable_memory_saver=False,
            kv_cache_dim=576,
            index_head_dim=128,
        )
        pin_memory = io_backend == "kernel"
        original_alloc = ALLOC_MEMORY_FUNCS["cuda"]
        if pin_memory:
            ALLOC_MEMORY_FUNCS["cuda"] = alloc_with_pin_memory
        try:
            mla_host = MLATokenToKVPoolHost(
                device_pool=device_pool,
                host_to_device_ratio=2.0,
                host_size=0,
                page_size=page_size,
                layout="layer_first",
                pin_memory=pin_memory,
                device="cpu",
                allocator_type="default",
                override_kv_cache_dim=device_pool.kv_cache_dim,
            )
            indexer_host = NSAIndexerPoolHost(
                device_pool=device_pool,
                anchor_host=mla_host,
                layout="layer_first",
                pin_memory=pin_memory,
                device="cpu",
                allocator_type="default",
            )
        finally:
            ALLOC_MEMORY_FUNCS["cuda"] = original_alloc

        for layer_id in range(layer_num):
            buf = device_pool.index_k_with_scale_buffer[layer_id]
            data = torch.arange(
                buf.numel(), device=buf.device, dtype=torch.uint8
            ).view_as(buf)
            buf.copy_((data + layer_id) % 256)
            kv_buf = device_pool.kv_buffer[layer_id]
            kv_data = torch.arange(
                kv_buf.numel(), device=kv_buf.device, dtype=kv_buf.dtype
            ).view_as(kv_buf)
            kv_buf.copy_(kv_data + layer_id)

        device_pages = torch.tensor([1, 2, 3], device="cuda", dtype=torch.int64)
        host_pages = torch.tensor(
            [0, 1, 2],
            device="cuda" if io_backend == "kernel" else "cpu",
            dtype=torch.int64,
        )
        device_indices = self._token_indices_for_pages(
            device_pages, page_size, device="cuda"
        )
        host_indices = self._token_indices_for_pages(
            host_pages,
            page_size,
            device="cuda" if io_backend == "kernel" else "cpu",
        )

        mla_host.backup_from_device_all_layer(
            device_pool, host_indices, device_indices, io_backend
        )
        indexer_host.backup_from_device_all_layer(
            device_pool, host_indices, device_indices, io_backend
        )

        for layer_id in range(layer_num):
            for host_page, device_page in zip(
```
**EN:** This block implements `_run_device_to_host_indexer_copy` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `_run_device_to_host_indexer_copy`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 120-135: method run device to host indexer copy (part 2/2) / 方法 run device to host indexer copy（第 2/2 部分）
```python
                host_pages.tolist(), device_pages.tolist()
            ):
                got = indexer_host.index_k_with_scale_buffer[layer_id][host_page].cpu()
                expected = device_pool.index_k_with_scale_buffer[layer_id][
                    device_page
                ].cpu()
                self.assertTrue(torch.equal(got, expected))
                host_start = host_page * page_size
                device_start = device_page * page_size
                got_kv = mla_host.kv_buffer[layer_id][
                    host_start : host_start + page_size
                ].cpu()
                expected_kv = device_pool.kv_buffer[layer_id][
                    device_start : device_start + page_size
                ].cpu()
                self.assertTrue(torch.equal(got_kv, expected_kv))
```
**EN:** This block implements `_run_device_to_host_indexer_copy` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `_run_device_to_host_indexer_copy`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 137-138: test case device to host indexer kernel / 测试用例 device to host indexer kernel
```python
    def test_device_to_host_indexer_kernel(self):
        self._run_device_to_host_indexer_copy(io_backend="kernel")
```
**EN:** This test exercises `test_device_to_host_indexer_kernel` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_device_to_host_indexer_kernel`。

### Lines 140-141: test case device to host indexer direct / 测试用例 device to host indexer direct
```python
    def test_device_to_host_indexer_direct(self):
        self._run_device_to_host_indexer_copy(io_backend="direct")
```
**EN:** This test exercises `test_device_to_host_indexer_direct` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_device_to_host_indexer_direct`。

### Lines 144-145: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestNSAHiCacheTransfer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNSAHiCacheTransfer.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestNSAHiCacheTransfer._token_indices_for_pages`: This block implements `_token_indices_for_pages` and captures one focused piece of the module's behavior. / 该代码块实现 `_token_indices_for_pages`，承担模块行为中的一个聚焦逻辑片段。
- `TestNSAHiCacheTransfer._run_device_to_host_indexer_copy`: This block implements `_run_device_to_host_indexer_copy` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_device_to_host_indexer_copy`，承担模块行为中的一个聚焦逻辑片段。
- `TestNSAHiCacheTransfer.test_device_to_host_indexer_kernel`: This test exercises `test_device_to_host_indexer_kernel` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_device_to_host_indexer_kernel`。
- `TestNSAHiCacheTransfer.test_device_to_host_indexer_direct`: This test exercises `test_device_to_host_indexer_direct` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_device_to_host_indexer_direct`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.memory_pool_host`, `sglang.srt.utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 145
