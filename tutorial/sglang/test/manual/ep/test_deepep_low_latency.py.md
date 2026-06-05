# test_deepep_low_latency.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/ep/test_deepep_low_latency.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `deepep low latency` scenario in `test/manual/ep`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/ep` 中的 `deepep low latency` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Scenario logic / 场景逻辑
```python
# Copy from deepseek-ai/DeepEP/tests/test_low_latency.py

import random
from functools import partial

import deep_ep
import torch
import torch.distributed as dist

from sglang.test.test_deepep_utils import (
    bench,
    bench_kineto,
    calc_diff,
    hash_tensor,
    init_dist,
    per_token_cast_back,
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 20-44: Test routines around test_main / 测试例程
```python
def test_main(
    num_tokens: int,
    hidden: int,
    num_experts: int,
    num_topk: int,
    rank: int,
    num_ranks: int,
    group: dist.ProcessGroup,
    buffer: deep_ep.Buffer,
    seed: int = 0,
):
    torch.manual_seed(seed + rank)
    random.seed(seed + rank)

    assert num_experts % num_ranks == 0
    num_local_experts = num_experts // num_ranks

    # NOTES: the integers greater than 256 exceeds the BF16 precision limit
    rank_offset = 128
    assert (
        num_ranks - rank_offset < 257
    ), "Too many ranks (exceeding test precision limit)"

    x = torch.ones((num_tokens, hidden), dtype=torch.bfloat16, device="cuda") * (
        rank - rank_offset
```
**EN:** This range defines concrete test routine(s) `test_main`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `manual_seed`, `seed`, `assert` and `ranks`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 45-69: Scenario logic / 场景逻辑
```python
    )
    x[:, -128:] = torch.arange(num_tokens, device="cuda").to(torch.bfloat16).view(-1, 1)
    scores = (
        torch.randn((num_tokens, num_experts), dtype=torch.float32, device="cuda").abs()
        + 1
    )
    topk_idx = torch.topk(scores, num_topk, dim=-1, largest=True, sorted=True)[1]
    topk_weights = torch.randn(
        (num_tokens, num_topk), dtype=torch.float32, device="cuda"
    ).abs()

    # Randomly mask some positions
    for i in range(10):
        topk_idx[random.randint(0, num_tokens - 1), random.randint(0, num_topk - 1)] = (
            -1
        )

    # Check dispatch correctness
    do_check = True
    hash_value, num_times = 0, 0
    for return_recv_hook in (False, True):
        for dispatch_use_fp8 in (False, True):
            num_times += 1
            for i in range((num_times % 2) + 1):
                packed_recv_x, packed_recv_count, handle, event, hook = (
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `arange`, `to`, `view` and `randn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 70-94: Scenario logic / 场景逻辑
```python
                    buffer.low_latency_dispatch(
                        x,
                        topk_idx,
                        num_tokens,
                        num_experts,
                        use_fp8=dispatch_use_fp8,
                        async_finish=not return_recv_hook,
                        return_recv_hook=return_recv_hook,
                    )
                )
                hook() if return_recv_hook else event.current_stream_wait()
            packed_recv_x = (
                (packed_recv_x[0], packed_recv_x[1].contiguous())
                if dispatch_use_fp8
                else packed_recv_x
            )
            simulated_gemm_x = (
                per_token_cast_back(
                    packed_recv_x[0].view(-1, hidden),
                    packed_recv_x[1].view(-1, hidden // 128),
                ).view(packed_recv_x[0].shape)
                if dispatch_use_fp8
                else packed_recv_x.clone()
            )
            all_topk_idx = torch.empty(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `low_latency_dispatch`, `hook`, `current_stream_wait` and `contiguous`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 95-119: Assertions and result checks / 断言与结果检查
```python
                (num_ranks, num_tokens, num_topk), dtype=topk_idx.dtype, device="cuda"
            )
            dist.all_gather_into_tensor(all_topk_idx, topk_idx, group=group)
            for i in range(num_local_experts if do_check else 0):
                expert_id = rank * num_local_experts + i
                recv_x = (
                    per_token_cast_back(packed_recv_x[0][i], packed_recv_x[1][i])
                    if dispatch_use_fp8
                    else packed_recv_x[i]
                )
                recv_count, recv_src_info, recv_layout_range = (
                    packed_recv_count[i],
                    handle[0][i],
                    handle[1][i],
                )

                # Check expert indices
                int_mask = (2**32) - 1
                num_valid_tokens = recv_count.item()
                assert (
                    num_valid_tokens == (recv_layout_range & int_mask).sum().item()
                ), f"{num_valid_tokens} != {recv_layout_range & int_mask}.sum().item()"
                assert (
                    num_valid_tokens == (all_topk_idx == expert_id).sum().item()
                ), f"{num_valid_tokens} != {(all_topk_idx == expert_id).sum().item()}"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `all_gather_into_tensor`, `per_token_cast_back`, `item` and `assert`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 120-144: Assertions and result checks / 断言与结果检查
```python

                # Check received data
                recv_x = recv_x[:num_valid_tokens]
                recv_x_amin = recv_x[:, :-128].amin(dim=-1)
                recv_src_info = recv_src_info[:num_valid_tokens]
                assert torch.equal(recv_x_amin, recv_x[:, :-128].amax(dim=-1))
                assert (
                    recv_x[:, -128:] - recv_src_info.view(-1, 1) % num_tokens
                ).sum().item() == 0
                for j in range(num_ranks):
                    begin_idx, count = (recv_layout_range[j] >> 32).item(), (
                        recv_layout_range[j] & int_mask
                    ).item()
                    assert (recv_x_amin == j - rank_offset).sum().item() == (
                        all_topk_idx[j] == expert_id
                    ).sum().item()
                    assert (
                        recv_x[begin_idx : begin_idx + count][:-128] - j
                    ).sum().item() == 0
                if dispatch_use_fp8:
                    hash_value ^= hash_tensor(packed_recv_x[0][i, :num_valid_tokens])
                    hash_value ^= hash_tensor(packed_recv_x[1][i, :num_valid_tokens])
                else:
                    hash_value ^= hash_tensor(packed_recv_x[i, :num_valid_tokens])
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `amin`, `equal`, `amax` and `assert`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 145-169: Scenario logic / 场景逻辑
```python
            # Check combine correctness
            for zero_copy in (False, True):
                if zero_copy:
                    buffer.get_next_low_latency_combine_buffer(handle)[
                        :, :, :
                    ] = simulated_gemm_x
                out = torch.empty(
                    (num_tokens, hidden), dtype=torch.bfloat16, device="cuda"
                )
                combined_x, event, hook = buffer.low_latency_combine(
                    simulated_gemm_x,
                    topk_idx,
                    topk_weights,
                    handle,
                    async_finish=not return_recv_hook,
                    zero_copy=zero_copy,
                    return_recv_hook=return_recv_hook,
                    out=out,
                )
                hook() if return_recv_hook else event.current_stream_wait()
                if do_check:
                    diff = calc_diff(
                        x
                        * topk_weights.masked_fill(topk_idx == -1, 0)
                        .sum(dim=1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `in`, `get_next_low_latency_combine_buffer`, `empty` and `low_latency_combine`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 170-194: Helper routines around create_test_cast_with_outliers / 辅助例程
```python
                        .view(-1, 1),
                        combined_x,
                    )
                    assert torch.isnan(combined_x).sum().item() == 0
                    assert diff < 1e-5, f"Error: {diff=}, {zero_copy=}"
                    hash_value ^= hash_tensor(combined_x)

    def create_test_cast_with_outliers(num_outliers):
        tmp = torch.randn((num_tokens, hidden), dtype=torch.bfloat16, device="cuda")
        tmp /= tmp.abs().amax(dim=1).view(-1, 1)
        assert tmp.abs().amax().item() <= 1

        # Create some amax outliers
        for i in range(num_outliers):
            tmp[random.randint(0, num_tokens - 1)] *= 1e3
        return tmp

    # noinspection PyShadowingNames
    def large_gemm_with_hook(hook):
        mat_0 = torch.randn((8192, 8192), dtype=torch.float)
        mat_1 = torch.randn((8192, 8192), dtype=torch.float)
        mat_0 @ mat_1
        hook()

    # noinspection PyShadowingNames
```
**EN:** This range implements helper routine(s) `create_test_cast_with_outliers` and `large_gemm_with_hook` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `view`, `isnan`, `item` and `hash_tensor`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 195-219: Test routines around test_func / 测试例程
```python
    def test_func(zero_copy: bool, return_recv_hook: bool):
        recv_x, recv_count, handle, event, hook = buffer.low_latency_dispatch(
            x,
            topk_idx,
            num_tokens,
            num_experts,
            async_finish=False,
            return_recv_hook=return_recv_hook,
        )
        large_gemm_with_hook(hook) if return_recv_hook else None
        if zero_copy:
            buffer.get_next_low_latency_combine_buffer(handle)[
                :, :, :
            ] = simulated_gemm_x
        combined_x, event, hook = buffer.low_latency_combine(
            simulated_gemm_x,
            topk_idx,
            topk_weights,
            handle,
            zero_copy=zero_copy,
            return_recv_hook=return_recv_hook,
        )
        large_gemm_with_hook(hook) if return_recv_hook else None

    # Calculate bandwidth
```
**EN:** This range defines concrete test routine(s) `test_func`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `low_latency_dispatch`, `large_gemm_with_hook`, `get_next_low_latency_combine_buffer` and `low_latency_combine`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 220-244: Scenario logic / 场景逻辑
```python
    num_fp8_bytes, num_bf16_bytes = (hidden + hidden / 128 * 4 + 16), hidden * 2
    num_dispatch_comm_bytes, num_combine_comm_bytes = 0, 0
    for i in range(num_tokens):
        num_selections = (topk_idx[i] != -1).sum().item()
        num_dispatch_comm_bytes += num_fp8_bytes * num_selections
        num_combine_comm_bytes += num_bf16_bytes * num_selections

    # Dispatch + combine testing
    avg_t, min_t, max_t = bench(
        partial(test_func, zero_copy=False, return_recv_hook=False)
    )
    print(
        f"[rank {rank}] Dispatch + combine bandwidth: {(num_dispatch_comm_bytes + num_combine_comm_bytes) / 1e9 / avg_t:.2f} GB/s, "
        f"avg_t={avg_t * 1e6:.2f} us, min_t={min_t * 1e6:.2f} us, max_t={max_t * 1e6:.2f} us",
        flush=True,
    )

    # Separate profiling
    for return_recv_hook in (False, True):
        group.barrier()
        dispatch_t, combine_t = bench_kineto(
            partial(test_func, zero_copy=True, return_recv_hook=return_recv_hook),
            kernel_names=("dispatch", "combine"),
            barrier_comm_profiling=True,
            suppress_kineto_output=True,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `item`, `bench`, `partial` and `in`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 245-259: Scenario logic / 场景逻辑
```python
        )
        if not return_recv_hook:
            print(
                f"[rank {rank}] Dispatch bandwidth: {num_dispatch_comm_bytes / 1e9 / dispatch_t:.2f} GB/s, avg_t={dispatch_t * 1e6:.2f} us | "
                f"Combine bandwidth: {num_combine_comm_bytes / 1e9 / combine_t:.2f} GB/s, avg_t={combine_t * 1e6:.2f} us",
                flush=True,
            )
        else:
            print(
                f"[rank {rank}] Dispatch send/recv time: {dispatch_t * 2 * 1e6:.2f} us | "
                f"Combine send/recv time: {combine_t * 2 * 1e6:.2f} us",
                flush=True,
            )

    return hash_value
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 260-262: Scenario logic / 场景逻辑
```python


# noinspection PyUnboundLocalVariable
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 263-287: Test routines around test_loop / 测试例程
```python
def test_loop(local_rank: int, num_local_ranks: int):
    rank, num_ranks, group = init_dist(local_rank, num_local_ranks)
    num_tokens, hidden, num_topk, num_experts = 128, 7168, 8, 288

    num_rdma_bytes = deep_ep.Buffer.get_low_latency_rdma_size_hint(
        num_tokens, hidden, num_ranks, num_experts
    )
    if local_rank == 0:
        print(f"Allocating buffer size: {num_rdma_bytes / 1e6} MB ...", flush=True)
    buffer = deep_ep.Buffer(
        group,
        num_rdma_bytes=num_rdma_bytes,
        low_latency_mode=True,
        num_qps_per_rank=num_experts // num_ranks,
    )
    test_main(
        num_tokens,
        hidden,
        num_experts,
        num_topk,
        rank,
        num_ranks,
        group,
        buffer,
        seed=1,
```
**EN:** This range defines concrete test routine(s) `test_loop`. The logic drives the target scenario and encodes the expected acceptance criteria. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `init_dist`, `get_low_latency_rdma_size_hint`, `Buffer` and `test_main`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 288-312: Assertions and result checks / 断言与结果检查
```python
    )

    do_pressure_test = False
    for seed in range(int(1e9) if do_pressure_test else 0):
        if local_rank == 0:
            print(f"Testing with seed {seed} ...", flush=True)
        ref_hash = test_main(
            num_tokens,
            hidden,
            num_experts,
            num_topk,
            rank,
            num_ranks,
            group,
            buffer,
            seed=seed,
        )
        for i in range(20):
            assert (
                test_main(
                    num_tokens,
                    hidden,
                    num_experts,
                    num_topk,
                    rank,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `test_main` and `assert`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 313-319: Scenario logic / 场景逻辑
```python
                    num_ranks,
                    group,
                    buffer,
                    seed=seed,
                )
                == ref_hash
            ), f"Error: seed={seed}"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 320-325: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    # TODO: you may modify NUMA binding for less CPU overhead
    num_processes = 8
    torch.multiprocessing.spawn(test_loop, args=(num_processes,), nprocs=num_processes)
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `spawn`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Accuracy evaluation / 精度评测
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Environment-aware configuration / 环境感知配置
- Streaming responses / 流式响应

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `functools`, `random`
- **Third-party / 第三方库**: `deep_ep`, `torch`, `torch.distributed`
- **Project Modules / 项目模块**: `sglang.test.test_deepep_utils`
