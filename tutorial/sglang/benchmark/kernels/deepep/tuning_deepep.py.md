# tuning_deepep.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/deepep/tuning_deepep.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels deepep tuning. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. / 该 Python 模块聚焦于 kernels deepep tuning 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

## Line-by-Line Analysis / 逐行分析
### Lines 3-27: Imports and setup / 导入与初始化
```python
"""
Example usage:
python tuning_deepep.py --nnodes 4 --node-rank $MY_NODE_RANK --master-addr 1.2.3.4
Then check `deepep_tuned.json`
"""

import argparse
import json
import time
from copy import deepcopy
from pathlib import Path

# noinspection PyUnresolvedReferences
import deep_ep
import torch
import torch.distributed as dist
from deepep_utils import (
    bench,
    calc_diff,
    create_grouped_scores,
    init_dist,
    inplace_unique,
    per_token_cast_back,
    per_token_cast_to_fp8,
)
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。

### Lines 30-418: Function `test_main` / 函数 `test_main`
```python
def test_main(
    num_sms: int,
    local_rank: int,
    num_local_ranks: int,
    num_ranks: int,
    num_nodes: int,
    rank: int,
    buffer: deep_ep.Buffer,
    group: dist.ProcessGroup,
    args,
):
    # Settings
    num_tokens, hidden, num_topk_groups, num_topk, num_experts = (
        args.num_tokens,
        args.hidden,
        min(num_nodes, 4),
        args.num_topk,
        (args.num_experts // num_ranks) * num_ranks,
    )
    assert num_experts % num_ranks == 0 and num_local_ranks == 8
    if local_rank == 0:
        print(
            f"[config] num_tokens={num_tokens}, hidden={hidden}, num_topk_groups={num_topk_groups}, num_topk={num_topk}",
            flush=True,
        )

    # Random data
    x = torch.ones((num_tokens, hidden), dtype=torch.bfloat16, device="cuda") * rank
    x_pure_rand = torch.randn((num_tokens, hidden), dtype=torch.bfloat16, device="cuda")
    x_e4m3 = per_token_cast_to_fp8(x)
    scores = (
        torch.randn((num_tokens, num_experts), dtype=torch.float32, device="cuda").abs()
        + 1
    )
    group_scores = scores.view(num_tokens, num_nodes, -1).amax(dim=-1)
    group_idx = torch.topk(
        group_scores, k=num_topk_groups, dim=-1, sorted=False
    ).indices
    masked_scores = create_grouped_scores(scores, group_idx, num_nodes)
    topk_idx = torch.topk(masked_scores, num_topk, dim=-1, largest=True, sorted=False)[
        1
    ]
    topk_weights = (
        torch.ones((num_tokens, num_topk), dtype=torch.float32, device="cuda") * rank
    )
    topk_weights_pure_rand = torch.randn(
        (num_tokens, num_topk), dtype=torch.float32, device="cuda"
    )
    rank_idx = topk_idx // (num_experts // num_ranks)
    rank_idx.masked_fill_(topk_idx == -1, -1)
    inplace_unique(rank_idx, num_ranks)
    rdma_rank_idx = rank_idx // num_local_ranks
    rdma_rank_idx.masked_fill_(rank_idx == -1, -1)
    inplace_unique(rdma_rank_idx, num_nodes)

    # RDMA dispatch counts
    rdma_idx = topk_idx // (num_experts // num_nodes)
    rdma_idx.masked_fill_(topk_idx == -1, -1)
    inplace_unique(rdma_idx, num_nodes)
    num_rdma_token_sent = rdma_idx.ne(-1).sum().item()

    # Expert meta
    num_tokens_per_expert = torch.zeros((num_experts,), dtype=torch.int, device="cuda")
    for i in range(num_experts):
        num_tokens_per_expert[i] = (topk_idx == i).sum()
    gbl_num_tokens_per_expert = num_tokens_per_expert.clone()
    dist.all_reduce(gbl_num_tokens_per_expert, group=group)

    # Rank layout meta
    num_tokens_per_rank = torch.empty((num_ranks,), dtype=torch.int, device="cuda")
    num_tokens_per_rdma_rank = torch.empty((num_nodes,), dtype=torch.int, device="cuda")
    token_idx_in_rank = torch.full(
        (num_ranks, num_tokens), -1, dtype=torch.long, device="cuda"
    )
    for i in range(num_ranks):
        num_tokens_per_rank[i] = (rank_idx == i).sum()
        token_sel = (rank_idx == i).max(dim=-1)[0]
        count = token_sel.sum().item()
        tokens = torch.sort(token_sel.to(torch.int), descending=True)[1]
        tokens[:count] = torch.sort(tokens[:count])[0]
        token_idx_in_rank[i][tokens[:count]] = torch.arange(
            count, dtype=torch.long, device="cuda"
        )
    for i in range(num_nodes):
        num_tokens_per_rdma_rank[i] = (rdma_rank_idx == i).sum()
    token_idx_in_rank = token_idx_in_rank.T.contiguous().to(torch.int)
    is_token_in_rank = token_idx_in_rank >= 0
    gbl_num_tokens_per_rank = num_tokens_per_rank.clone()
    dist.all_reduce(gbl_num_tokens_per_rank, group=group)

    (
        ref_num_tokens_per_rank,
        ref_num_tokens_per_rdma_rank,
        ref_num_tokens_per_expert,
        ref_is_token_in_rank,
        _,
    ) = buffer.get_dispatch_layout(topk_idx, num_experts)
    assert torch.allclose(ref_num_tokens_per_rank, num_tokens_per_rank)
    assert torch.allclose(ref_num_tokens_per_rdma_rank, num_tokens_per_rdma_rank)
    assert torch.allclose(ref_num_tokens_per_expert, num_tokens_per_expert)
    assert torch.allclose(ref_is_token_in_rank, is_token_in_rank)
    t = bench(lambda: buffer.get_dispatch_layout(topk_idx, num_experts))[0]
    if local_rank == 0:
        print(f"[layout] Kernel performance: {t * 1000:.3f} ms", flush=True)
        print("", flush=True)
    group.barrier()
    time.sleep(1)

    # Config
    rdma_buffer_size, nvl_buffer_size = 128, (720 if num_ranks in (144, 160) else 512)
    config = deep_ep.Config(num_sms, 8, nvl_buffer_size, 16, rdma_buffer_size)

    # Test dispatch
    # noinspection PyShadowingNames
    def check_data(check_x, recv_gbl_rank_prefix_sum):
        assert torch.allclose(check_x.amin(dim=1), check_x.amax(dim=1))
        check_start = 0
        for i in range(num_ranks):
            check_end = recv_gbl_rank_prefix_sum[i].item()
            assert (check_x[check_start:check_end, :].int() - i).sum().item() == 0
            check_start = check_end

    for previous_mode in (False, True):
        for async_mode in (False, True):
            for current_x in (x_pure_rand, x, x_e4m3):
                for with_topk in (False, True):
                    if local_rank == 0:
                        print(
                            f'[testing] Running with {"FP8" if isinstance(current_x, tuple) else "BF16"}, {"with" if with_topk else "without"} top-k (async={async_mode}, previous={previous_mode}) ...',
                            flush=True,
                            end="",
                        )
                    dispatch_args = {
                        "x": current_x,
                        "num_tokens_per_rank": num_tokens_per_rank,
                        "num_tokens_per_rdma_rank": num_tokens_per_rdma_rank,
                        "is_token_in_rank": is_token_in_rank,
                        "num_tokens_per_expert": num_tokens_per_expert,
                        "config": config,
                        "async_finish": async_mode,
                    }
                    if with_topk:
                        dispatch_args.update(
                            {
                                "topk_idx": topk_idx,
                                "topk_weights": (
                                    topk_weights_pure_rand
                                    if current_x is x_pure_rand
                                    else topk_weights
                                ),
                            }
                        )
                    if previous_mode:
                        dispatch_args.update({"previous_event": buffer.capture()})
                    (
                        recv_x,
                        recv_topk_idx,
                        recv_topk_weights,
                        recv_num_tokens_per_expert_list,
                        handle,
                        event,
                    ) = buffer.dispatch(**dispatch_args)
                    event.current_stream_wait() if async_mode else ()
                    recv_x = (
                        per_token_cast_back(*recv_x)
                        if isinstance(recv_x, tuple)
                        else recv_x
                    )

                    # Checks
                    recv_gbl_rank_prefix_sum = handle[-4]
                    assert gbl_num_tokens_per_rank[rank].item() == recv_x.size(
                        0
                    ), f"{gbl_num_tokens_per_rank[rank].item()} != {recv_x.size(0)}"
                    assert (
                        gbl_num_tokens_per_expert.view(num_ranks, -1)[rank].tolist()
                        == recv_num_tokens_per_expert_list
                    )
                    if current_x is not x_pure_rand:
                        check_data(recv_x, recv_gbl_rank_prefix_sum)
                    if with_topk:
                        # Check `topk_idx`
                        assert (
                            recv_topk_idx.eq(-1)
                            | (
                                (recv_topk_idx >= 0)
                                & (recv_topk_idx < (num_experts // num_ranks))
                            )
                        ).sum().item() == recv_topk_idx.numel()
                        for i, count in enumerate(recv_num_tokens_per_expert_list):
                            assert recv_topk_idx.eq(i).sum().item() == count

                        # Check `topk_weights`
                        if current_x is not x_pure_rand:
                            recv_topk_weights[recv_topk_idx.eq(-1)] = (
                                recv_topk_weights.amax(dim=1, keepdim=True).expand_as(
                                    recv_topk_weights
                                )[recv_topk_idx.eq(-1)]
                            )
                            check_data(recv_topk_weights, recv_gbl_rank_prefix_sum)

                    # Test cached dispatch (must without top-k staffs)
                    if not with_topk:
                        dispatch_args = {
                            "x": current_x,
                            "handle": handle,
                            "config": config,
                            "async_finish": async_mode,
                        }
                        if previous_mode:
                            dispatch_args.update({"previous_event": buffer.capture()})
                        recv_x, _, _, _, _, event = buffer.dispatch(**dispatch_args)
                        event.current_stream_wait() if async_mode else ()
                        recv_x = (
                            per_token_cast_back(*recv_x)
                            if isinstance(recv_x, tuple)
                            else recv_x
                        )
                        if current_x is not x_pure_rand:
                            check_data(recv_x, recv_gbl_rank_prefix_sum)

                    # Test combine
                    combine_args = {
                        "x": recv_x,
                        "handle": handle,
                        "config": config,
                        "async_finish": async_mode,
                    }
                    if with_topk:
                        combine_args.update({"topk_weights": recv_topk_weights})
                    if previous_mode:
                        combine_args.update({"previous_event": buffer.capture()})
                    combined_x, combined_topk_weights, event = buffer.combine(
                        **combine_args
                    )
                    event.current_stream_wait() if async_mode else ()
                    check_x = combined_x.float() / is_token_in_rank.sum(
                        dim=1
                    ).unsqueeze(1)
                    ref_x = x_pure_rand if current_x is x_pure_rand else x
                    assert calc_diff(check_x, ref_x) < 5e-6
                    if with_topk:
                        check_topk_weights = (
                            combined_topk_weights
                            if (current_x is x_pure_rand)
                            else (
                                combined_topk_weights
                                / is_token_in_rank.sum(dim=1).unsqueeze(1)
                            )
                        )
                        ref_topk_weights = (
                            topk_weights_pure_rand
                            if current_x is x_pure_rand
                            else topk_weights
                        )
                        assert calc_diff(check_topk_weights, ref_topk_weights) < 1e-9

                    # For later tuning
                    dispatch_bf16_rdma_send_bytes = num_rdma_token_sent * hidden * 2
                    dispatch_bf16_nvl_recv_bytes = recv_x.numel() * 2
                    combine_bf16_nvl_send_bytes = dispatch_bf16_nvl_recv_bytes
                    combine_bf16_rdma_recv_bytes = dispatch_bf16_rdma_send_bytes

                    if local_rank == 0:
                        print(" passed", flush=True)
    if local_rank == 0:
        print("", flush=True)

    output_data = {}

    # Tune dispatch performance
    best_dispatch_results = None
    fp8_factor = (1 + 4 / 128) / 2
    for current_x in (x_e4m3, x):
        best_time, best_results = 1e10, None
        rdma_send_bytes = (
            (dispatch_bf16_rdma_send_bytes * fp8_factor)
            if isinstance(current_x, tuple)
            else dispatch_bf16_rdma_send_bytes
        )
        nvl_recv_bytes = (
            (dispatch_bf16_nvl_recv_bytes * fp8_factor)
            if isinstance(current_x, tuple)
            else dispatch_bf16_nvl_recv_bytes
        )
        for nvl_chunk_size in range(4, 33, 4):
            for rdma_chunk_size in range(4, 33, 4):
                config_kwargs = {
                    "num_sms": num_sms,
                    "num_max_nvl_chunked_send_tokens": nvl_chunk_size,
                    "num_max_nvl_chunked_recv_tokens": nvl_buffer_size,
                    "num_max_rdma_chunked_send_tokens": rdma_chunk_size,
                    "num_max_rdma_chunked_recv_tokens": rdma_buffer_size,
                }
                config = deep_ep.Config(**config_kwargs)
                tune_args = {"x": current_x, "handle": handle, "config": config}
                t = bench(lambda: buffer.dispatch(**tune_args))[0]
                if t < best_time:
                    best_time, best_results = t, (
                        num_sms,
                        nvl_chunk_size,
                        rdma_chunk_size,
                        config_kwargs,
                    )
                if local_rank == 0:
                    print(
                        f"[tuning] SMs {num_sms}, NVL chunk {nvl_chunk_size}, RDMA chunk {rdma_chunk_size}: {rdma_send_bytes / 1e9 / t:.2f} GB/s (RDMA), {nvl_recv_bytes / 1e9 / t:.2f} GB/s (NVL) ",
                        flush=True,
                    )
        if local_rank == 0:
            print(
                f'[tuning] Best dispatch ({"FP8" if isinstance(current_x, tuple) else "BF16"}): SMs {best_results[0]}, NVL chunk {best_results[1]}, RDMA chunk {best_results[2]}: {rdma_send_bytes / 1e9 / best_time:.2f} GB/s (RDMA), {nvl_recv_bytes / 1e9 / best_time:.2f} GB/s (NVL)',
                flush=True,
            )
            print("", flush=True)
            is_fp8 = isinstance(current_x, tuple)
            if is_fp8:
                output_data["normal_dispatch"] = deepcopy(best_results[3])

        if isinstance(current_x, tuple):
            # Gather FP8 the best config from rank 0
            best_dispatch_results = torch.tensor(
                [best_results[0], best_results[1], best_results[2]],
                dtype=torch.int32,
                device="cuda",
            )
            all_best_fp8_results_list = [
                torch.zeros_like(best_dispatch_results)
                for _ in range(torch.distributed.get_world_size())
            ]
            dist.all_gather(
                all_best_fp8_results_list, best_dispatch_results, group=group
            )
            best_dispatch_results = all_best_fp8_results_list[0].tolist()
    dispatch_config = deep_ep.Config(
        best_dispatch_results[0],
        best_dispatch_results[1],
        nvl_buffer_size,
        best_dispatch_results[2],
        rdma_buffer_size,
    )

    dispatch_args = {
        "x": x,
        "num_tokens_per_rank": num_tokens_per_rank,
        "num_tokens_per_rdma_rank": num_tokens_per_rdma_rank,
        "is_token_in_rank": is_token_in_rank,
        "num_tokens_per_expert": num_tokens_per_expert,
        "config": dispatch_config if dispatch_config is not None else config,
    }
    recv_x, _, _, _, handle, _ = buffer.dispatch(**dispatch_args)

    # Tune combine performance
    best_time, best_results = 1e10, None
    for nvl_chunk_size in range(1, 8, 1):
        for rdma_chunk_size in range(12 if num_nodes == 2 else 8, 33, 4):
            config_kwargs = {
                "num_sms": num_sms,
                "num_max_nvl_chunked_send_tokens": nvl_chunk_size,
                "num_max_nvl_chunked_recv_tokens": nvl_buffer_size,
                "num_max_rdma_chunked_send_tokens": rdma_chunk_size,
                "num_max_rdma_chunked_recv_tokens": rdma_buffer_size,
            }
            config = deep_ep.Config(**config_kwargs)
            tune_args = {"x": recv_x, "handle": handle, "config": config}
            t = bench(lambda: buffer.combine(**tune_args))[0]
            if local_rank == 0:
                print(
                    f"[tuning] SMs {num_sms}, NVL chunk {nvl_chunk_size}, RDMA chunk {rdma_chunk_size}: {combine_bf16_rdma_recv_bytes / 1e9 / t:.2f} GB/s (RDMA), {combine_bf16_nvl_send_bytes / 1e9 / t:.2f} GB/s (NVL) ",
                    flush=True,
                )
                if t < best_time:
                    best_time, best_results = t, (
                        num_sms,
                        nvl_chunk_size,
                        rdma_chunk_size,
                        config_kwargs,
                    )

    if local_rank == 0:
        print(
            f"[tuning] Best combine: SMs {best_results[0]}, NVL chunk {best_results[1]}, RDMA chunk {best_results[2]}: {combine_bf16_rdma_recv_bytes / 1e9 / best_time:.2f} GB/s (RDMA), {combine_bf16_nvl_send_bytes / 1e9 / best_time:.2f} GB/s (NVL)",
            flush=True,
        )
        print("", flush=True)
        output_data["normal_combine"] = deepcopy(best_results[3])

    if rank == 0 and local_rank == 0:
        _write_output(args, output_data)
```
**EN:** `test_main` is a function that measures runtime latency, throughput, or other benchmark metrics, coordinates asynchronous or parallel execution, and prepares tensors and invokes GPU kernels. Notable calls include `torch.randn`, `per_token_cast_to_fp8`, `scores.view(num_tokens, num_nodes, -1).amax`.
**CN:** `test_main` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行、准备张量并调用 GPU 内核。其中较关键的调用包括 `torch.randn`, `per_token_cast_to_fp8`, `scores.view(num_tokens, num_nodes, -1).amax`。

### Lines 421-425: Function `_write_output` / 函数 `_write_output`
```python
def _write_output(args, output_data):
    text = json.dumps(output_data, indent=4)
    output_path = args.output_path
    print(f"Write to {output_path} with {text}")
    Path(output_path).write_text(text)
```
**EN:** `_write_output` is a function that loads, filters, or serializes benchmark datasets and reads or writes local files and intermediate artifacts. Notable calls include `json.dumps`, `print`, `Path(output_path).write_text`.
**CN:** `_write_output` 是一个函数，用于加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。其中较关键的调用包括 `json.dumps`, `print`, `Path(output_path).write_text`。

### Lines 429-459: Function `test_loop` / 函数 `test_loop`
```python
def test_loop(local_rank: int, num_local_ranks: int, args):
    num_nodes = args.nnodes
    rank, num_ranks, group = init_dist(local_rank, num_local_ranks, args)

    num_sms = args.num_sms
    num_qps_per_rank = num_sms // 2

    buffer = deep_ep.Buffer(
        group,
        int(1e9),
        int(1e9),
        low_latency_mode=False,
        num_qps_per_rank=num_qps_per_rank,
    )
    assert num_local_ranks == 8 and num_ranks > 8
    torch.manual_seed(rank)

    for i in (num_sms,):
        test_main(
            i,
            local_rank,
            num_local_ranks,
            num_ranks,
            num_nodes,
            rank,
            buffer,
            group,
            args,
        )
        if local_rank == 0:
            print("", flush=True)
```
**EN:** `test_loop` is a function that measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels. Notable calls include `init_dist`, `deep_ep.Buffer`, `torch.manual_seed`.
**CN:** `test_loop` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。其中较关键的调用包括 `init_dist`, `deep_ep.Buffer`, `torch.manual_seed`。

### Lines 462-480: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--num-sms", type=int, default=24)
    parser.add_argument("--num-tokens", type=int, default=4096)
    parser.add_argument("--hidden", type=int, default=7168)
    parser.add_argument("--num-topk", type=int, default=8)
    parser.add_argument("--num-experts", type=int, default=256)
    parser.add_argument("--output-path", type=str, default="deepep_tuned.json")
    parser.add_argument("--nnodes", type=int, default=1)
    parser.add_argument("--node-rank", type=int, default=0)
    parser.add_argument("--master-addr", type=str, default="127.0.0.1")
    parser.add_argument("--master-port", type=int, default=8361)
    args = parser.parse_args()
    print(f"Start system with {args=}")

    num_processes = 8
    torch.multiprocessing.spawn(
        test_loop, args=(num_processes, args), nprocs=num_processes
    )
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `time`, `copy`, `pathlib`
- **Third-party / 第三方依赖**: `deep_ep`, `torch`, `torch.distributed`, `deepep_utils`
