# update.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/checkpoint_engine/update.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example coordinates live checkpoint registration and weight updates for a running SGLang service. / 该示例演示了如何为运行中的 SGLang 服务协调在线检查点注册与权重更新。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Module overview and usage
````python
"""
Usage:
1) Launch the server with wait-for-initial-weights option in one terminal:
   python -m sglang.launch_server --model-path /workspace/Qwen/Qwen3-4B/ --tensor-parallel-size 2 --port 19730 --load-format dummy --checkpoint-engine-wait-weights-before-ready --mem-fraction-static 0.7

2) Torchrun this script in another terminal:
    torchrun --nproc-per-node 2 update.py --update-method broadcast --checkpoint-path /workspace/Qwen/Qwen3-4B/  --inference-parallel-size 2
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 10-25: Import dependencies and runtime symbols
````python
import argparse
import json
import os
import pickle
import time
from collections import defaultdict
from collections.abc import Callable
from contextlib import contextmanager
from typing import Literal

import httpx
import torch
import torch.distributed as dist
from checkpoint_engine.ps import ParameterServer
from loguru import logger
from safetensors import safe_open
````
**EN:** These imports combine distributed PyTorch, HTTP helpers, safetensor readers, and checkpoint-engine utilities required for online weight updates.
**CN:** 这些导入组合了分布式 PyTorch、HTTP 工具、safetensors 读取器以及在线权重更新所需的 checkpoint-engine 组件。

### Lines 28-33: Handle
````python
@contextmanager
def timer(msg: str):
    start = time.perf_counter()
    yield
    end = time.perf_counter()
    logger.info(f"{msg} duration: {end - start:.2f} seconds")
````
**EN:** This function encapsulates the “Handle” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 36-57: Check SGLang ready
````python
def check_sglang_ready(
    endpoint: str, inference_parallel_size: int, uds: str | None = None
):
    if rank != rank // inference_parallel_size * inference_parallel_size:
        return
    retry_num = 0
    transport = None
    if uds is not None:
        transport = httpx.HTTPTransport(uds=uds)
    with httpx.Client(transport=transport) as client:
        while True:
            try:
                response = client.get(f"{endpoint}/ping", timeout=10)
                response.raise_for_status()
                break
            except (httpx.ConnectError, httpx.HTTPStatusError) as e:
                if retry_num % 10 == 0:
                    logger.warning(
                        f"fail to check sglang ready, retry {retry_num} times, error: {e}"
                    )
                retry_num += 1
                time.sleep(0.1)
````
**EN:** This function encapsulates the “Check SGLang ready” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Check SGLang ready”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 60-70: Coordinate checkpoint updates
````python
def split_checkpoint_files(
    checkpoint_path: str, rank: int, world_size: int
) -> list[str]:
    checkpoint_files = [
        os.path.join(checkpoint_path, f)
        for f in filter(
            lambda x: x.endswith(".safetensors"), os.listdir(checkpoint_path)
        )
    ]
    files_per_rank = (len(checkpoint_files) + world_size - 1) // world_size
    return checkpoint_files[rank * files_per_rank : (rank + 1) * files_per_rank]
````
**EN:** This function coordinates registration, synchronization, metadata exchange, and the final weight-transfer step for a distributed checkpoint update.
**CN:** 该函数负责协同分布式检查点更新中的注册、同步、元数据交换以及最终的权重传输步骤。

### Lines 73-91: Coordinate checkpoint updates
````python
def split_tensors(
    checkpoint_path: str, rank: int, world_size: int
) -> dict[str, torch.Tensor]:
    index_fn = os.path.join(checkpoint_path, "model.safetensors.index.json")
    with open(index_fn) as f:
        weight_map: dict[str, str] = json.load(f)["weight_map"]
    weights_per_rank = (len(weight_map) + world_size - 1) // world_size
    fn_tensors: dict[str, list[str]] = defaultdict(list)
    weight_keys = list(weight_map.items())
    for name, file in weight_keys[
        rank * weights_per_rank : (rank + 1) * weights_per_rank
    ]:
        fn_tensors[file].append(name)
    named_tensors = {}
    for file, names in fn_tensors.items():
        with safe_open(os.path.join(checkpoint_path, file), framework="pt") as f:
            for name in names:
                named_tensors[name] = f.get_tensor(name)
    return named_tensors
````
**EN:** This function coordinates registration, synchronization, metadata exchange, and the final weight-transfer step for a distributed checkpoint update.
**CN:** 该函数负责协同分布式检查点更新中的注册、同步、元数据交换以及最终的权重传输步骤。

### Lines 94-120: Create request helper inference
````python
def req_inference(
    endpoint: str,
    inference_parallel_size: int,
    timeout: float = 300.0,
    uds: str | None = None,
    weight_version: str | None = None,
) -> Callable[[list[tuple[str, str]]], None]:
    rank = int(os.getenv("RANK", 0))
    src = rank // inference_parallel_size * inference_parallel_size

    def req_func(socket_paths: list[tuple[str, str]]):
        if rank == src:
            with httpx.Client(transport=httpx.HTTPTransport(uds=uds)) as client:
                resp = client.post(
                    f"{endpoint}/update_weights_from_ipc",
                    json={
                        "zmq_handles": dict(
                            socket_paths[src : src + inference_parallel_size]
                        ),
                        "flush_cache": True,
                        "weight_version": weight_version,
                    },
                    timeout=timeout,
                )
                resp.raise_for_status()

    return req_func
````
**EN:** This function encapsulates the “Create request helper inference” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Create request helper inference”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 123-158: Coordinate checkpoint updates
````python
def update_weights(
    ps: ParameterServer,
    checkpoint_name: str,
    checkpoint_files: list[str],
    named_tensors: dict[str, torch.Tensor],
    req_func: Callable[[list[tuple[str, str]]], None],
    inference_parallel_size: int,
    endpoint: str,
    save_metas_file: str | None = None,
    update_method: Literal["broadcast", "p2p", "all"] = "broadcast",
    uds: str | None = None,
):
    ps.register_checkpoint(
        checkpoint_name, files=checkpoint_files, named_tensors=named_tensors
    )
    ps.init_process_group()
    check_sglang_ready(endpoint, inference_parallel_size, uds)
    dist.barrier()
    with timer("Gather metas"):
        ps.gather_metas(checkpoint_name)
    if save_metas_file and int(os.getenv("RANK")) == 0:
        with open(save_metas_file, "wb") as f:
            pickle.dump(ps.get_metas(), f)

    if update_method == "broadcast" or update_method == "all":
        with timer("Update weights without setting ranks"):
            ps.update(checkpoint_name, req_func)

    if update_method == "p2p" or update_method == "all":
        if update_method:
            # sleep 2s to wait destroy process group
            time.sleep(2)
        with timer("Update weights with setting ranks"):
            ps.update(
                checkpoint_name, req_func, ranks=list(range(inference_parallel_size))
            )
````
**EN:** This function coordinates registration, synchronization, metadata exchange, and the final weight-transfer step for a distributed checkpoint update.
**CN:** 该函数负责协同分布式检查点更新中的注册、同步、元数据交换以及最终的权重传输步骤。

### Lines 161-182: Coordinate checkpoint updates
````python
def join(
    ps: ParameterServer,
    checkpoint_name: str,
    load_metas_file: str,
    req_func: Callable[[list[tuple[str, str]]], None],
    inference_parallel_size: int,
    endpoint: str,
    uds: str | None = None,
):
    assert load_metas_file, "load_metas_file is required"
    with open(load_metas_file, "rb") as f:
        metas = pickle.load(f)
    ps.init_process_group()
    check_sglang_ready(endpoint, inference_parallel_size, uds)
    dist.barrier()
    with timer("Gather metas before join"):
        ps.gather_metas(checkpoint_name)
    ps.load_metas(metas)
    with timer(
        f"Update weights with setting ranks as range(0, {inference_parallel_size}) by using p2p"
    ):
        ps.update(checkpoint_name, req_func, ranks=list(range(inference_parallel_size)))
````
**EN:** This function coordinates registration, synchronization, metadata exchange, and the final weight-transfer step for a distributed checkpoint update.
**CN:** 该函数负责协同分布式检查点更新中的注册、同步、元数据交换以及最终的权重传输步骤。

### Lines 185-241: Program entry point
````python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Update weights example")
    parser.add_argument("--checkpoint-path", type=str, default=None)
    parser.add_argument("--save-metas-file", type=str, default=None)
    parser.add_argument("--load-metas-file", type=str, default=None)
    parser.add_argument("--sleep-time", type=int, default=0)
    parser.add_argument("--endpoint", type=str, default="http://localhost:19730")
    parser.add_argument("--inference-parallel-size", type=int, default=8)
    parser.add_argument("--checkpoint-name", type=str, default="my-checkpoint-iter-0")
    parser.add_argument("--update-method", type=str, default="broadcast")
    parser.add_argument("--uds", type=str, default=None)
    parser.add_argument("--weight-version", type=str, default=None)
    args = parser.parse_args()
    rank = int(os.getenv("RANK"))
    world_size = int(os.getenv("WORLD_SIZE"))
    req_func = req_inference(
        args.endpoint,
        args.inference_parallel_size,
        uds=args.uds,
        weight_version=args.weight_version,
    )
    ps = ParameterServer(auto_pg=True)
    ps._p2p_store = None
    if args.load_metas_file:
        join(
            ps,
            args.checkpoint_name,
            args.load_metas_file,
            req_func,
            args.inference_parallel_size,
            args.endpoint,
            args.uds,
        )
    else:
        if os.path.exists(
            os.path.join(args.checkpoint_path, "model.safetensors.index.json")
        ):
            named_tensors = split_tensors(args.checkpoint_path, rank, world_size)
            checkpoint_files = []
        else:
            checkpoint_files = split_checkpoint_files(
                args.checkpoint_path, rank, world_size
            )
            named_tensors = {}
        update_weights(
            ps,
            args.checkpoint_name,
            checkpoint_files,
            named_tensors,
            req_func,
            args.inference_parallel_size,
            args.endpoint,
            args.save_metas_file,
            args.update_method,
            args.uds,
        )
    time.sleep(args.sleep_time)
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Distributed coordination / 分布式协同**: Multiple workers synchronize state, data, or updates. / 多个工作进程会同步状态、数据或更新。
- **Checkpoint management / 检查点管理**: Weights or runtime state are saved, loaded, or updated explicitly. / 权重或运行状态会被显式保存、加载或更新。

## Dependencies / 依赖关系
- **Standard library / 标准库**: argparse, collections.abc.Callable, collections.defaultdict, contextlib.contextmanager, json, os, pickle, time, typing.Literal
- **Third-party / 第三方**: httpx, loguru.logger, safetensors.safe_open, torch, torch.distributed
- **Project-specific / 项目相关**: checkpoint_engine.ps.ParameterServer
