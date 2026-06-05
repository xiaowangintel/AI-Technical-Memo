# test_eagle_step_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/spec_decode/test_eagle_step_kernel.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for the fused EAGLE slot mapping kernel. / 该文件的文档字符串表明其用途：`unit tests for the fused eagle slot mapping kernel`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (line 3)
```python
"""Unit tests for the fused EAGLE slot mapping kernel."""
```
**EN:** Module docstring that declares the scope of the file: Unit tests for the fused EAGLE slot mapping kernel.
**CN:** 模块文档字符串直接说明了文件范围：`unit tests for the fused eagle slot mapping kernel`。

### Imports and setup / 导入与设置 (lines 5-12)
```python
import pytest
import torch

from vllm.platforms import current_platform
from vllm.v1.spec_decode.utils import (
    PADDING_SLOT_ID,
    eagle_step_update_slot_mapping_and_metadata,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.platforms, vllm.v1.spec_decode.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.platforms, vllm.v1.spec_decode.utils`。

### Module state / 模块级状态 (lines 14-19)
```python
DEVICE_TYPE = current_platform.device_type

# Skip if no CUDA - Triton kernel requires GPU
pytest.importorskip("triton")
if not current_platform.is_cuda_alike() and not current_platform.is_xpu():
    pytest.skip("CUDA/XPU required for EAGLE kernel tests", allow_module_level=True)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `DEVICE_TYPE`. Conditional logic is used to adapt the suite to optional dependencies or platform differences. Shared setup calls include `pytest.importorskip, pytest.skip, current_platform.is_cuda_alike, current_platform.is_xpu`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`DEVICE_TYPE`。 条件逻辑用于适配可选依赖或平台差异。 共享初始化调用包括 `pytest.importorskip, pytest.skip, current_platform.is_cuda_alike, current_platform.is_xpu`。

### _reference_eagle_step_slot_mapping (lines 22-48)
```python
def _reference_eagle_step_slot_mapping(
    positions_1d: torch.Tensor,
    block_table_tensor: torch.Tensor,
    seq_lens: torch.Tensor,
    block_size: int,
    max_model_len: int,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    """Python reference for eagle_step_update_slot_mapping_and_metadata."""
    new_positions = positions_1d + 1
    exceeds_max = new_positions >= max_model_len
    clamped_positions = torch.where(
        exceeds_max, torch.zeros_like(positions_1d), new_positions
    )
    block_numbers = (clamped_positions // block_size).clamp(
        max=block_table_tensor.shape[1] - 1
    )
    block_ids = block_table_tensor[
        torch.arange(positions_1d.shape[0], device=positions_1d.device),
        block_numbers.long(),
    ].long()
    slot_mapping = block_ids * block_size + (clamped_positions % block_size)
    slot_mapping = torch.where(
        exceeds_max, torch.full_like(slot_mapping, PADDING_SLOT_ID), slot_mapping
    )
    new_seq_lens = torch.where(exceeds_max, torch.ones_like(seq_lens), seq_lens + 1)
    new_seq_lens = new_seq_lens.clamp(max=max_model_len)
    return clamped_positions, slot_mapping, new_seq_lens
```
**EN:** Helper function `_reference_eagle_step_slot_mapping` encapsulates reusable logic for `reference eagle step slot mapping`. Inputs: `positions_1d, block_table_tensor, seq_lens, block_size, max_model_len`. Key calls include `torch.where, clamp, block_table_tensor.long, new_seq_lens.clamp, torch.zeros_like, torch.full_like`.
**CN:** 辅助函数 `_reference_eagle_step_slot_mapping` 封装了与 `reference eagle step slot mapping` 相关的可复用逻辑。 输入参数：`positions_1d, block_table_tensor, seq_lens, block_size, max_model_len`。 关键调用包括 `torch.where, clamp, block_table_tensor.long, new_seq_lens.clamp, torch.zeros_like, torch.full_like`。

### test_eagle_step_slot_mapping_kernel (lines 51-94)
```python
def test_eagle_step_slot_mapping_kernel():
    """Test fused kernel matches Python reference for slot mapping and metadata."""
    device = torch.device(DEVICE_TYPE)
    batch_size = 32
    block_size = 16
    max_model_len = 4096
    n_blocks_per_req = (max_model_len + block_size - 1) // block_size

    positions_1d = torch.randint(
        0, max_model_len - 10, (batch_size,), dtype=torch.int64, device=device
    )
    block_table_tensor = torch.randint(
        0, 1000, (batch_size, n_blocks_per_req), dtype=torch.int32, device=device
    seq_lens = torch.randint(1, 100, (batch_size,), dtype=torch.int32, device=device)
    ref_clamped, ref_slot, ref_seq_lens = _reference_eagle_step_slot_mapping(
        positions_1d.clone(),
    # ... excerpt omitted for brevity ...
    assert torch.equal(out_clamped, ref_clamped), (
        f"clamped: {out_clamped} vs {ref_clamped}"
    assert torch.equal(out_slot, ref_slot), f"slot: {out_slot} vs {ref_slot}"
    assert torch.equal(seq_lens_copy, ref_seq_lens), (
        f"seq_lens: {seq_lens_copy} vs {ref_seq_lens}"
```
**EN:** Test case covering `eagle step slot mapping kernel`. It exercises `torch.device, torch.randint, _reference_eagle_step_slot_mapping, torch.zeros, seq_lens.clone, eagle_step_update_slot_mapping_and_metadata`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `eagle step slot mapping kernel` 的测试用例。 该测试会调用 `torch.device, torch.randint, _reference_eagle_step_slot_mapping, torch.zeros, seq_lens.clone, eagle_step_update_slot_mapping_and_metadata`。 代码主体包含 3 个显式断言。

### test_eagle_step_slot_mapping_kernel_exceeds_max (lines 97-130)
```python
def test_eagle_step_slot_mapping_kernel_exceeds_max():
    """Test fused kernel when position exceeds max_model_len."""
    device = torch.device(DEVICE_TYPE)
    batch_size = 4
    block_size = 16
    max_model_len = 100
    n_blocks_per_req = (max_model_len + block_size - 1) // block_size

    positions_1d = torch.tensor([50, 98, 99, 100], dtype=torch.int64, device=device)
    block_table_tensor = torch.randint(
        0, 100, (batch_size, n_blocks_per_req), dtype=torch.int32, device=device
    )
    seq_lens = torch.tensor([51, 99, 100, 101], dtype=torch.int32, device=device)
    out_clamped = torch.zeros(batch_size, dtype=torch.int64, device=device)
    out_slot = torch.zeros(batch_size, dtype=torch.int64, device=device)
    eagle_step_update_slot_mapping_and_metadata(
        positions_1d=positions_1d,
    # ... excerpt omitted for brevity ...
    assert out_clamped[0].item() == 51
    assert out_clamped[1].item() == 99
    assert out_clamped[2].item() == 0
    assert out_clamped[3].item() == 0
    assert out_slot[2].item() == PADDING_SLOT_ID
    assert out_slot[3].item() == PADDING_SLOT_ID
    assert seq_lens[2].item() == 1
    assert seq_lens[3].item() == 1
```
**EN:** Test case covering `eagle step slot mapping kernel exceeds max`. It exercises `torch.device, torch.tensor, torch.randint, torch.zeros, eagle_step_update_slot_mapping_and_metadata, out_clamped.item`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `eagle step slot mapping kernel exceeds max` 的测试用例。 该测试会调用 `torch.device, torch.tensor, torch.randint, torch.zeros, eagle_step_update_slot_mapping_and_metadata, out_clamped.item`。 代码主体包含 8 个显式断言。

### test_eagle_step_slot_mapping_kernel_cudagraph_padding (lines 133-178)
```python
def test_eagle_step_slot_mapping_kernel_cudagraph_padding():
    """Test that padding threads write PADDING_SLOT_ID when
    input_batch_size > batch_size (cudagraph padding)."""
    device = torch.device(DEVICE_TYPE)
    batch_size = 4
    input_batch_size = 8
    block_size = 16
    max_model_len = 4096
    n_blocks_per_req = (max_model_len + block_size - 1) // block_size

    positions_1d = torch.tensor([10, 20, 30, 40], dtype=torch.int64, device=device)
    block_table_tensor = torch.randint(
        0, 100, (batch_size, n_blocks_per_req), dtype=torch.int32, device=device
    )
    seq_lens = torch.tensor([11, 21, 31, 41], dtype=torch.int32, device=device)
    ref_clamped, ref_slot, ref_seq_lens = _reference_eagle_step_slot_mapping(
        positions_1d.clone(),
    # ... excerpt omitted for brevity ...
    assert torch.equal(out_clamped, ref_clamped)
    assert torch.equal(out_slot[:batch_size], ref_slot)
    assert torch.equal(seq_lens_copy, ref_seq_lens)
    # Padding slots should be PADDING_SLOT_ID
    for i in range(batch_size, input_batch_size):
        assert out_slot[i].item() == PADDING_SLOT_ID
```
**EN:** Test case covering `eagle step slot mapping kernel cudagraph padding`. It exercises `torch.device, torch.tensor, torch.randint, _reference_eagle_step_slot_mapping, torch.zeros, torch.full`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `eagle step slot mapping kernel cudagraph padding` 的测试用例。 该测试会调用 `torch.device, torch.tensor, torch.randint, _reference_eagle_step_slot_mapping, torch.zeros, torch.full`。 代码主体包含 4 个显式断言。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Speculative execution or decoding paths
- **CN:** 推测执行或解码路径

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.platforms, vllm.v1.spec_decode.utils`.
- **CN:** 被测试的 vLLM 模块：`vllm.platforms, vllm.v1.spec_decode.utils`。
