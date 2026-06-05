# test_spec_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/spec/test_spec_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `spec utils` scenario in `test/manual/spec`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/spec` 中的 `spec utils` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and shared helpers / 导入与共享辅助项
```python
import unittest

import numpy as np
import torch

from sglang.srt.mem_cache.memory_pool import copy_all_layer_kv_cache_tiled
from sglang.srt.speculative.spec_utils import assign_draft_cache_locs
from sglang.srt.utils import next_power_of_2

BYTES_PER_TILE = 128
```
**EN:** This range imports `unittest`, `numpy`, `torch` and `sglang.srt.mem_cache.memory_pool`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 13-14: Class definition for TestSpecUtils / 类定义
```python
class TestSpecUtils(unittest.TestCase):
```
**EN:** This range declares `TestSpecUtils`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 15-37: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUp(self):
        self.device = "cuda" if torch.cuda.is_available() else "cpu"
        self.data_ptrs = torch.zeros(2, 1, dtype=torch.uint64, device=self.device)
        self.k_cache = [
            torch.zeros((100, 1, 1), dtype=torch.float32, device=self.device)
        ]
        self.v_cache = [
            torch.zeros((100, 1, 1), dtype=torch.float32, device=self.device)
        ]
        self.k_cache[0][:11, 0, 0] = torch.tensor(
            [0.0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0],
            dtype=torch.float32,
            device=self.device,
        )
        self.v_cache[0][:11, 0, 0] = torch.tensor(
            [-0.0, -0.1, -0.2, -0.3, -0.4, -0.5, -0.6, -0.7, -0.8, -0.9, -1.0],
            dtype=torch.float32,
            device=self.device,
        )
        self.data_ptrs[0, 0] = self.k_cache[0].data_ptr()
        self.data_ptrs[1, 0] = self.v_cache[0].data_ptr()

        self.data_strides = torch.tensor(
```
**EN:** This range implements lifecycle helper(s) `setUp` to prepare or clean up shared resources across the test flow. Representative call sites include `is_available`, `zeros`, `tensor` and `data_ptr`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 38-39: Scenario logic / 场景逻辑
```python
            [
                np.prod(x.shape[1:]) * x.dtype.itemsize
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `prod`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 40-44: Scenario logic / 场景逻辑
```python
                for x in self.k_cache + self.v_cache
            ],
            device=self.device,
            dtype=torch.int64,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 46-62: Test routines around test_assign_draft_cache_locs_single_seq / 测试例程
```python
    def test_assign_draft_cache_locs_single_seq(self):
        # Testing Setup: req_to_token starting from 4
        # 4,5,6,7,{8,9,10}, 8,9,10 is the last partial page, 3 tokens < page_size=4
        # next kv cache will be stored starting 11,12,13...
        device = self.device
        num_seqs = 1
        page_size = 4
        speculative_num_steps = 5
        topk = 8
        seq_lens_num = 7
        extend_lens_num = 61  # includes the duplicated last page
        req_pool_indices = torch.arange(num_seqs, dtype=torch.int32, device=device)
        req_to_token = torch.zeros((num_seqs, 100), dtype=torch.int32, device=device)
        req_to_token[0, :seq_lens_num] = torch.tensor(
            [4, 5, 6, 7, 8, 9, 10], device=device
        )
        seq_lens = torch.tensor([seq_lens_num], dtype=torch.int32, device=device)
```
**EN:** This range defines concrete test routine(s) `test_assign_draft_cache_locs_single_seq`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `arange`, `zeros` and `tensor`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 63-70: Scenario logic / 场景逻辑
```python
        extend_lens = torch.tensor([extend_lens_num], dtype=torch.int32, device=device)
        num_new_pages_per_topk = torch.tensor([2], dtype=torch.int32, device=device)
        out_cache_loc = torch.arange(11, 11 + extend_lens_num, device=device)
        last_page_lens = torch.tensor([3], dtype=torch.int32, device=device)
        last_page_lens_cumsum = torch.cumsum(last_page_lens, dim=0)
        duplicate_cache_len = last_page_lens.sum().item() * (topk - 1)
        target_cache_loc = torch.zeros(
            duplicate_cache_len, dtype=torch.int32, device=device
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor`, `arange`, `cumsum` and `item`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 71-87: Scenario logic / 场景逻辑
```python
        )
        source_cache_loc = torch.zeros(
            duplicate_cache_len, dtype=torch.int32, device=device
        )
        assign_draft_cache_locs[(num_seqs,)](
            req_pool_indices,
            req_to_token,
            seq_lens,
            extend_lens,
            num_new_pages_per_topk,
            out_cache_loc,
            source_cache_loc,
            target_cache_loc,
            last_page_lens_cumsum,
            duplicate_cache_len,
            req_to_token.shape[1],
            topk,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 88-95: Scenario logic / 场景逻辑
```python
            speculative_num_steps,
            page_size,
            next_power_of_2(num_seqs),
            next_power_of_2(speculative_num_steps + page_size),
        )

        out_cache_loc = out_cache_loc[: num_seqs * topk * speculative_num_steps]
        expected_source_cache_loc = torch.tensor(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `next_power_of_2` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 96-112: Assertions and result checks / 断言与结果检查
```python
            [8, 9, 10] * (topk - 1), device=device, dtype=torch.int32
        )
        assert torch.allclose(source_cache_loc, expected_source_cache_loc)

        copy_all_layer_kv_cache_tiled[(len(self.data_ptrs),)](
            self.data_ptrs,
            self.data_strides,
            target_cache_loc,
            source_cache_loc,
            len(target_cache_loc),
            next_power_of_2(len(target_cache_loc)),
            BYTES_PER_TILE,
        )
        assert torch.allclose(
            self.k_cache[0][16:19, 0, 0],
            torch.tensor(
                [0.8, 0.9, 1.0],
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `allclose`, `next_power_of_2` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 113-120: Assertions and result checks / 断言与结果检查
```python
                dtype=torch.float32,
                device=device,
            ),
        )
        assert torch.allclose(
            self.v_cache[0][16:19, 0, 0],
            torch.tensor(
                [-0.8, -0.9, -1.0],
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `allclose` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 121-124: Scenario logic / 场景逻辑
```python
                dtype=torch.float32,
                device=device,
            ),
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 126-137: Test routines around test_assign_draft_cache_locs_multi_seq / 测试例程
```python
    def test_assign_draft_cache_locs_multi_seq(self):
        device = self.device
        num_seqs = 3
        page_size = 4
        speculative_num_steps = 5
        topk = 8
        req_pool_indices = torch.arange(num_seqs, dtype=torch.int32, device=device)
        req_to_token = torch.zeros((num_seqs, 100), dtype=torch.int32, device=device)
        seq_lens = torch.tensor([8, 7, 5], dtype=torch.int32, device=device)
        extend_lens = torch.tensor([64, 64, 64], dtype=torch.int32, device=device)
        num_new_pages_per_topk = torch.tensor(
            [2, 2, 2], dtype=torch.int32, device=device
```
**EN:** This range defines concrete test routine(s) `test_assign_draft_cache_locs_multi_seq`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `arange`, `zeros` and `tensor`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 138-150: Scenario logic / 场景逻辑
```python
        )
        req_to_token = torch.zeros((num_seqs, 100), dtype=torch.int32, device=device)
        req_to_token[0, :8] = torch.tensor([4, 5, 6, 7, 8, 9, 10, 11], device=device)
        req_to_token[1, :7] = torch.tensor([4, 5, 6, 7, 8, 9, 10], device=device)
        req_to_token[2, :5] = torch.tensor([4, 5, 6, 7, 8], device=device)
        last_page_lens = torch.tensor([0, 3, 1], dtype=torch.int32, device=device)
        last_page_lens_cumsum = torch.cumsum(last_page_lens, dim=0)
        duplicate_cache_len = last_page_lens.sum().item() * (topk - 1)
        out_cache_loc = torch.arange(
            12, 12 + torch.sum(extend_lens), dtype=torch.int32, device=device
        )
        target_cache_loc = torch.zeros(
            duplicate_cache_len, dtype=torch.int32, device=device
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `zeros`, `tensor`, `cumsum` and `item`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 151-162: Scenario logic / 场景逻辑
```python
        )
        source_cache_loc = torch.zeros(
            duplicate_cache_len, dtype=torch.int32, device=device
        )
        assign_draft_cache_locs[(num_seqs,)](
            req_pool_indices,
            req_to_token,
            seq_lens,
            extend_lens,
            num_new_pages_per_topk,
            out_cache_loc,
            source_cache_loc,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 163-175: Scenario logic / 场景逻辑
```python
            target_cache_loc,
            last_page_lens_cumsum,
            duplicate_cache_len,
            req_to_token.shape[1],
            topk,
            speculative_num_steps,
            page_size,
            next_power_of_2(num_seqs),
            next_power_of_2(speculative_num_steps + page_size),
        )
        out_cache_loc = out_cache_loc[: num_seqs * topk * speculative_num_steps]
        # fmt: off
        expected_out_cache_loc = torch.tensor([
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `next_power_of_2` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 176-187: Scenario logic / 场景逻辑
```python
            12,  13,  14,  15,  16,
            20,  21,  22,  23,  24,
            28,  29,  30,  31,  32,
            36,  37,  38,  39,  40,
            44,  45,  46,  47,  48,
            52,  53,  54,  55,  56,
            60,  61,  62,  63,  64,
            68,  69,  70,  71,  72,
            76,  77,  78,  79,  80,
            84,  85,  86,  87,  88,
            92,  93,  94,  95,  96,
            100, 101, 102, 103, 104,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 188-200: Scenario logic / 场景逻辑
```python
            108, 109, 110, 111, 112,
            116, 117, 118, 119, 120,
            124, 125, 126, 127, 128,
            132, 133, 134, 135, 136,
            140, 141, 142, 143, 144,
            148, 149, 150, 151, 152,
            156, 157, 158, 159, 160,
            164, 165, 166, 167, 168,
            172, 173, 174, 175, 176,
            180, 181, 182, 183, 184,
            188, 189, 190, 191, 192,
            196, 197, 198, 199, 200
        ], device=device, dtype=torch.int32)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 201-212: Assertions and result checks / 断言与结果检查
```python
        expected_source_cache_loc = torch.tensor([8, 9, 10] * 7 + [8] * 7, device=device, dtype=torch.int32)
        expected_target_cache_loc = torch.tensor([
            81,  82,  83,  89,  90,  91,  97,  98,  99, 105, 106, 107, 113, 114,
           115, 121, 122, 123, 129, 130, 131, 147, 155, 163, 171, 179, 187, 195
        ], device=device, dtype=torch.int32)
        # fmt: on
        assert torch.allclose(out_cache_loc, expected_out_cache_loc)
        assert torch.allclose(source_cache_loc, expected_source_cache_loc)
        assert torch.allclose(target_cache_loc, expected_target_cache_loc)
        copy_all_layer_kv_cache_tiled[(len(self.data_ptrs),)](
            self.data_ptrs,
            self.data_strides,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `tensor` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 213-225: Assertions and result checks / 断言与结果检查
```python
            target_cache_loc,
            source_cache_loc,
            len(target_cache_loc),
            next_power_of_2(len(target_cache_loc)),
            BYTES_PER_TILE,
        )
        assert torch.allclose(
            self.k_cache[0][81:84, 0, 0],
            torch.tensor(
                [0.8, 0.9, 1.0],
                dtype=torch.float32,
                device=device,
            ),
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `next_power_of_2`, `allclose` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 226-234: Assertions and result checks / 断言与结果检查
```python
        )
        assert torch.allclose(
            self.v_cache[0][81:84, 0, 0],
            torch.tensor(
                [-0.8, -0.9, -1.0],
                dtype=torch.float32,
                device=device,
            ),
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `allclose` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 236-237: Test routines around test_assign_draft_cache_locs_page_size_1 / 测试例程
```python
    def test_assign_draft_cache_locs_page_size_1(self):
        # Test to make sure page_size=1 not affected
```
**EN:** This range defines concrete test routine(s) `test_assign_draft_cache_locs_page_size_1`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 238-260: Scenario logic / 场景逻辑
```python
        device = self.device
        num_seqs = 1
        page_size = 1
        speculative_num_steps = 5
        topk = 8
        seq_lens_num = 7
        extend_lens_num = topk * speculative_num_steps
        req_pool_indices = torch.arange(num_seqs, dtype=torch.int32, device=device)
        req_to_token = torch.zeros((num_seqs, 100), dtype=torch.int32, device=device)
        req_to_token[0, :seq_lens_num] = torch.tensor(
            [4, 5, 6, 7, 8, 9, 10], device=device
        )
        seq_lens = torch.tensor([seq_lens_num], dtype=torch.int32, device=device)
        extend_lens = torch.tensor([extend_lens_num], dtype=torch.int32, device=device)
        num_new_pages_per_topk = torch.tensor([2], dtype=torch.int32, device=device)
        out_cache_loc = torch.arange(11, 11 + extend_lens_num, device=device)
        last_page_lens = torch.tensor([3], dtype=torch.int32, device=device)
        duplicate_cache_len = 0
        target_cache_loc = None
        source_cache_loc = None
        last_page_lens_cumsum = None
        assign_draft_cache_locs[(num_seqs,)](
            req_pool_indices,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `arange`, `zeros` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 261-262: Scenario logic / 场景逻辑
```python
            req_to_token,
            seq_lens,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 263-279: Assertions and result checks / 断言与结果检查
```python
            extend_lens,
            num_new_pages_per_topk,
            out_cache_loc,
            source_cache_loc,
            target_cache_loc,
            last_page_lens_cumsum,
            duplicate_cache_len,
            req_to_token.shape[1],
            topk,
            speculative_num_steps,
            page_size,
            next_power_of_2(num_seqs),
            next_power_of_2(speculative_num_steps + page_size),
        )
        out_cache_loc = out_cache_loc[: num_seqs * topk * speculative_num_steps]
        expected_out_cache_loc = torch.arange(11, 11 + extend_lens_num, device=device)
        assert torch.allclose(out_cache_loc, expected_out_cache_loc)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `next_power_of_2`, `arange` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 281-287: Test routines around test_assign_draft_cache_locs_page_size_gt_spec_steps / 测试例程
```python
    def test_assign_draft_cache_locs_page_size_gt_spec_steps(self):
        device = self.device
        num_seqs = 1
        page_size = 16
        speculative_num_steps = 4
        topk = 3
        seq_lens_num = 12
```
**EN:** This range defines concrete test routine(s) `test_assign_draft_cache_locs_page_size_gt_spec_steps`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 288-305: Scenario logic / 场景逻辑
```python
        pool_len = 256
        req_pool_indices = torch.arange(num_seqs, dtype=torch.int32, device=device)
        req_to_token = torch.zeros(
            (num_seqs, pool_len), dtype=torch.int32, device=device
        )
        req_to_token[0, :seq_lens_num] = torch.arange(
            seq_lens_num, dtype=torch.int32, device=device
        )
        seq_lens = torch.tensor([seq_lens_num], dtype=torch.int32, device=device)
        last_page_len = seq_lens_num % page_size
        last_page_lens = torch.tensor([last_page_len], dtype=torch.int32, device=device)
        last_page_lens_cumsum = torch.cumsum(last_page_lens, dim=0)
        num_new_pages_per_topk_val = (
            last_page_len + speculative_num_steps + page_size - 1
        ) // page_size
        num_new_pages_per_topk = torch.tensor(
            [num_new_pages_per_topk_val], dtype=torch.int32, device=device
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `arange`, `zeros`, `tensor` and `cumsum`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 306-312: Scenario logic / 场景逻辑
```python
        extend_lens_num = num_new_pages_per_topk_val * page_size * topk
        extend_lens = torch.tensor([extend_lens_num], dtype=torch.int32, device=device)
        out_cache_loc = torch.arange(
            2000, 2000 + extend_lens_num, dtype=torch.int32, device=device
        )
        duplicate_cache_len = last_page_lens.sum().item() * (topk - 1)
        target_cache_loc = torch.zeros(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor`, `arange`, `item` and `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 313-330: Scenario logic / 场景逻辑
```python
            duplicate_cache_len, dtype=torch.int32, device=device
        )
        source_cache_loc = torch.zeros(
            duplicate_cache_len, dtype=torch.int32, device=device
        )
        assign_draft_cache_locs[(num_seqs,)](
            req_pool_indices,
            req_to_token,
            seq_lens,
            extend_lens,
            num_new_pages_per_topk,
            out_cache_loc,
            source_cache_loc,
            target_cache_loc,
            last_page_lens_cumsum,
            duplicate_cache_len,
            req_to_token.shape[1],
            topk,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 331-337: Scenario logic / 场景逻辑
```python
            speculative_num_steps,
            page_size,
            next_power_of_2(num_seqs),
            next_power_of_2(speculative_num_steps + page_size),
        )
        trimmed = out_cache_loc[: num_seqs * topk * speculative_num_steps]
        expected = []
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `next_power_of_2`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 338-344: Assertions and result checks / 断言与结果检查
```python
        for topk_id in range(topk):
            start = seq_lens_num + topk_id * num_new_pages_per_topk_val * page_size
            expected.append(
                req_to_token[0, start : start + speculative_num_steps].clone()
            )
        expected_out_cache_loc = torch.cat(expected)
        assert torch.allclose(trimmed, expected_out_cache_loc)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `append`, `clone`, `cat` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 345-348: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Token-level inspection / Token 级分析
- Prefix caching / 前缀缓存

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: `numpy`, `torch`
- **Project Modules / 项目模块**: `sglang.srt.mem_cache.memory_pool`, `sglang.srt.speculative.spec_utils`, `sglang.srt.utils`
