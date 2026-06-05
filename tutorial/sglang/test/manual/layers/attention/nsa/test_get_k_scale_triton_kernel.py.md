# test_get_k_scale_triton_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/layers/attention/nsa/test_get_k_scale_triton_kernel.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `get k scale triton kernel` scenario in `test/manual/layers/attention/nsa`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/layers/attention/nsa` 中的 `get k scale triton kernel` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and shared helpers / 导入与共享辅助项
```python
import torch

from sglang.srt.layers.attention.nsa.index_buf_accessor import (
    _get_k_and_s_triton_kernel,
)
```
**EN:** This range imports `torch` and `sglang.srt.layers.attention.nsa.index_buf_accessor`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 8-32: Helper routines around golden_torch_gen / 辅助例程
```python
def golden_torch_gen(
    seq_len_tensor: torch.Tensor,
    buffer_indexer: torch.Tensor,
    buffer: torch.Tensor,
    index_head_dim,
    page_size,
):
    dim_split = page_size * index_head_dim
    torch_k_out = buffer[:, 0:dim_split]
    torch_s_out = buffer[:, dim_split:]

    torch_k_out = torch_k_out.reshape(-1, 128)
    torch_s_out = torch_s_out.reshape(-1, 4)

    batch = seq_len_tensor.shape[0]
    index_list = []
    for i in range(batch):
        seq_len = seq_len_tensor[i].item()
        buffer_index_ = buffer_indexer[i]
        align_seq_len = ((seq_len + page_size - 1) / page_size) * page_size
        needed_block_num = int((seq_len + page_size - 1) / page_size)
        for j in range(needed_block_num):
            block_idx = buffer_index_[j].item()
            start_idx = block_idx * page_size
            end_idx = 0
```
**EN:** This range implements helper routine(s) `golden_torch_gen` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `reshape` and `item`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 33-51: Scenario logic / 场景逻辑
```python
            if j == (needed_block_num - 1):
                end_idx = block_idx * page_size + (
                    seq_len - (needed_block_num - 1) * page_size
                )
            else:
                end_idx = (block_idx + 1) * page_size

            index_tensor = (
                torch.arange(start=start_idx, end=end_idx, step=1)
                .type(torch.int32)
                .cuda()
            )
            index_list.append(index_tensor)

    index_list_ = torch.cat(index_list, dim=0)
    torch_k_out = torch.index_select(torch_k_out, dim=0, index=index_list_)
    torch_s_out = torch.index_select(torch_s_out, dim=0, index=index_list_)

    return torch_k_out, torch_s_out
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `arange`, `type`, `cuda` and `append`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 54-78: Helper routines around get_k_and_s_triton / 辅助例程
```python
def get_k_and_s_triton():
    index_head_dim = 128
    page_size = 64
    num_page = 128
    s_offset_in_page = page_size * index_head_dim

    seq_len_tensor = torch.tensor(
        [256, 267, 215, 32, 129], dtype=torch.int64, device="cuda"
    )  # 4 + 5 + 3 + 1 + 3 block
    buffer_indexer = torch.tensor(
        [
            [1, 2, 3, 4, 0],
            [7, 6, 5, 8, 9],
            [10, 11, 12, 0, 0],
            [13, 0, 0, 0, 0],
            [14, 15, 16, 0, 0],
        ],
        dtype=torch.int32,
        device="cuda",
    )
    seq_len_sum = seq_len_tensor.sum()
    batch = seq_len_tensor.shape[0]

    triton_k_out = torch.empty(
        (seq_len_sum, index_head_dim), dtype=torch.uint8, device="cuda"
```
**EN:** This range implements helper routine(s) `get_k_and_s_triton` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `tensor` and `empty`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 79-103: Constants and scenario settings / 常量与场景配置
```python
    )
    triton_s_out = torch.empty((seq_len_sum, 4), dtype=torch.uint8, device="cuda")
    buffer = torch.randint(
        0,
        num_page,
        (num_page, page_size * index_head_dim + page_size * 4),
        device="cuda",
    ).type(torch.uint8)

    _, buf_numel_per_page = buffer.shape
    _, page_indice_batch_offset = buffer_indexer.shape
    max_seq_len = seq_len_tensor.max().item()

    BLOCK_SIZE = 256
    BLOCK_SIZE_K = 128

    num_token_blocks = (max_seq_len + BLOCK_SIZE - 1) // BLOCK_SIZE
    num_k_threads = (index_head_dim + BLOCK_SIZE_K - 1) // BLOCK_SIZE_K

    grid = (batch, num_token_blocks, num_k_threads)
    seq_num_pow2 = 1
    while seq_num_pow2 < batch:
        seq_num_pow2 *= 2

    # acc test =====================
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Representative call sites include `empty`, `randint`, `type` and `item`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 104-128: Constants and scenario settings / 常量与场景配置
```python
    _get_k_and_s_triton_kernel[grid](
        buf_ptr=buffer,
        page_indices_ptr=buffer_indexer,
        k_out_ptr=triton_k_out,
        s_out_ptr=triton_s_out,
        seq_len_ptr=seq_len_tensor,
        seq_len_num_pow=seq_num_pow2,
        page_size=page_size,
        buf_numel_per_page=buf_numel_per_page,
        index_head_dim=index_head_dim,
        s_offset_in_page=s_offset_in_page,
        page_indice_batch_offset=page_indice_batch_offset,
        BLOCK_SIZE=BLOCK_SIZE,
        BLOCK_SIZE_K=BLOCK_SIZE_K,
    )

    torch_k_out, torch_s_out = golden_torch_gen(
        seq_len_tensor=seq_len_tensor,
        buffer_indexer=buffer_indexer,
        buffer=buffer,
        index_head_dim=index_head_dim,
        page_size=page_size,
    )

    torch.testing.assert_close(
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Assertions in this block enforce the intended outcome. Representative call sites include `golden_torch_gen` and `assert_close`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 129-153: Constants and scenario settings / 常量与场景配置
```python
        triton_k_out, torch_k_out, rtol=0, atol=0, msg="k outputs differ!"
    )
    torch.testing.assert_close(
        triton_s_out, torch_s_out, rtol=0, atol=0, msg="s outputs differ!"
    )
    print("_get_k_and_s_triton_kernel test pass")

    # perf test =====================
    import time

    torch.cuda.synchronize()
    for _ in range(10):
        _get_k_and_s_triton_kernel[grid](
            buf_ptr=buffer,
            page_indices_ptr=buffer_indexer,
            k_out_ptr=triton_k_out,
            s_out_ptr=triton_s_out,
            seq_len_ptr=seq_len_tensor,
            seq_len_num_pow=seq_num_pow2,
            page_size=page_size,
            buf_numel_per_page=buf_numel_per_page,
            index_head_dim=index_head_dim,
            s_offset_in_page=s_offset_in_page,
            page_indice_batch_offset=page_indice_batch_offset,
            BLOCK_SIZE=BLOCK_SIZE,
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close` and `synchronize`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 154-178: Constants and scenario settings / 常量与场景配置
```python
            BLOCK_SIZE_K=BLOCK_SIZE_K,
        )

    torch.cuda.synchronize()
    start_time = time.perf_counter()

    _get_k_and_s_triton_kernel[grid](
        buf_ptr=buffer,
        page_indices_ptr=buffer_indexer,
        k_out_ptr=triton_k_out,
        s_out_ptr=triton_s_out,
        seq_len_ptr=seq_len_tensor,
        seq_len_num_pow=seq_num_pow2,
        page_size=page_size,
        buf_numel_per_page=buf_numel_per_page,
        index_head_dim=index_head_dim,
        s_offset_in_page=s_offset_in_page,
        page_indice_batch_offset=page_indice_batch_offset,
        BLOCK_SIZE=BLOCK_SIZE,
        BLOCK_SIZE_K=BLOCK_SIZE_K,
    )

    end_time = time.perf_counter()
    print(
        f"_get_k_and_s_triton_kernel triton kernel infer time is {((end_time-start_time)*1000):.4f} ms\n"
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Representative call sites include `synchronize` and `perf_counter`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 179-179: Scenario logic / 场景逻辑
```python
    )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 180-191: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    if not torch.cuda.is_available():
        print("CUDA not available. Skipping tests.")
        exit(0)

    print("Start test cases...\n")

    get_k_and_s_triton()

    print("End test cases...\n")
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `is_available`, `exit` and `get_k_and_s_triton`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `time`
- **Third-party / 第三方库**: `torch`
- **Project Modules / 项目模块**: `sglang.srt.layers.attention.nsa.index_buf_accessor`
