# test_load_weights_from_remote_instance_npu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/distributed/test_load_weights_from_remote_instance_npu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates load weights from remote instance npu behavior in SGLang's distributed area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 分布式 领域中与 load weights from remote instance npu 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: supporting statements / 辅助语句
```python
"""Test loading weights from remote instance.

This test suite simulates loading weights from a remote instance.
Rank 0 represents the seed instance, while ranks 1 represents the
new instance that needs to loading weights from the seed instance.

Seed instance must be started in `Server` mode, while the dst instance
can be either `Engine` mode or `Server` mode.

Seed instance does not support concurrently serving multiple dst instances.
User has to guarantee that there is only one dst instance trying to load
weights from the seed instance at any time.

"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 16-37: module imports and dependencies / 模块导入与依赖
```python
import gc
import os
import random
import unittest

import numpy as np
import requests
import torch
import torch.multiprocessing as mp

import sglang as sgl
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import (
    DEFAULT_PORT_FOR_SRT_TEST_RUNNER,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
)
from sglang.utils import terminate_process
```
**EN:** This block imports the modules needed by the rest of the file, including `gc`, `os`, `random`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `gc`, `os`, `random`, `unittest`。

### Lines 39-46: CI registration and metadata / CI 注册与元数据
```python
mp.set_start_method("spawn", force=True)

register_npu_ci(
    est_time=400,
    suite="nightly-1-npu-a3",
    nightly=True,
    disabled="run failed",
)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through set_start_method, register_npu_ci.
**CN:** 该代码块通过 set_start_method, register_npu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 49-57: function verify params close / 函数 verify params close
```python
def verify_params_close(params1, params2, error_msg):
    """Verify if two parameter arrays are close enough."""
    try:
        assert np.allclose(np.array(params1), np.array(params2)), error_msg
    except Exception as e:
        print(f"Parameters not close for {error_msg}")
        print("Params1:", np.array(params1))
        print("Params2:", np.array(params2))
        raise e
```
**EN:** Verify if two parameter arrays are close enough. This block implements `verify_params_close` and captures one focused piece of the module's behavior.
**CN:** Verify if two parameter arrays are close enough. 该代码块实现 `verify_params_close`，承担模块行为中的一个聚焦逻辑片段。

### Lines 60-103: function init process / 函数 init process
```python
def init_process(
    rank,
    param_queue,
    truncate_size,
    tp_size,
    model_name,
    backends,
    checking_parameters,
    seed_instance_ip,
    seed_instance_service_port,
    seed_instance_group_base_port,
    event_seed_ready,
    event_dst_ready_list,
    remote_instance_loader_backend,
):
    torch.npu.set_device(rank)

    if rank == 0:
        init_process_seed(
            rank,
            param_queue,
            truncate_size,
            model_name,
            checking_parameters,
            tp_size,
            event_seed_ready,
            event_dst_ready_list,
        )
    elif rank in [1, 2]:
        init_process_dst(
            rank,
            param_queue,
            truncate_size,
            model_name,
            seed_instance_ip,
            seed_instance_service_port,
            seed_instance_group_base_port,
            checking_parameters,
            backends[rank - 1],
            tp_size,
            event_seed_ready,
            event_dst_ready_list,
            remote_instance_loader_backend,
        )
```
**EN:** This block implements `init_process` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `init_process`，承担模块行为中的一个聚焦逻辑片段。

### Lines 106-160: function init process seed / 函数 init process seed
```python
def init_process_seed(
    rank,
    param_queue,
    truncate_size,
    model_name,
    checking_parameters,
    tp_size,
    event_seed_ready,
    event_dst_ready_list,
):
    # These two environment variables are very important
    # to avoid unexpected behaviors of npu and NCCL.
    os.environ["NCCL_CUMEM_ENABLE"] = "0"
    os.environ["NCCL_NVLS_ENABLE"] = "0"

    # Load model and get parameters
    torch.npu.set_device(rank)
    torch.npu.synchronize()

    url = DEFAULT_URL_FOR_TEST
    process = popen_launch_server(
        model_name,
        url,
        timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        other_args=(
            "--attention-backend",
            "ascend",
            "--device",
            "npu",
            "--base-gpu-id",
            str(rank),
            "--tp-size",
            str(tp_size),
        ),
    )
    torch.npu.synchronize()

    seed_params = []
    # Get the weights of seed instance for correctness check.
    for parameter_name in checking_parameters:
        seed_params.append(
            requests.get(
                f"{url}/get_weights_by_name",
                json={
                    "name": parameter_name,
                    "truncate_size": truncate_size,
                },
            ).json()
        )
    param_queue.put((f"seed_params", seed_params))

    event_seed_ready.set()
    for i in range(len(event_dst_ready_list)):
        event_dst_ready_list[i].wait()
    terminate_process(process)
```
**EN:** This block implements `init_process_seed` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `init_process_seed`，承担模块行为中的一个聚焦逻辑片段。

### Lines 163-242: function init process dst (part 1/2) / 函数 init process dst（第 1/2 部分）
```python
def init_process_dst(
    rank,
    param_queue,
    truncate_size,
    model_name,
    seed_instance_ip,
    seed_instance_service_port,
    seed_instance_group_base_port,
    checking_parameters,
    backend,
    tp_size,
    event_seed_ready,
    event_dst_ready_list,
    remote_instance_loader_backend,
):
    torch.npu.set_device(rank * tp_size)
    torch.npu.synchronize()
    base_gpu_id = rank * tp_size

    event_seed_ready.wait()
    print(f"rank {rank}, seed ready")
    for i in range(rank - 1):
        print(f"rank {rank}, wait dst {i}")
        event_dst_ready_list[i].wait()

    ports = []
    for i in range(tp_size):
        ports.append(seed_instance_group_base_port + (rank - 1) * tp_size + i)

    if backend == "Engine":
        print(f"[sgl] rank {rank} init engine")
        engine = sgl.Engine(
            attention_backend="ascend",
            device="npu",
            model_path=model_name,
            base_gpu_id=base_gpu_id,
            tp_size=tp_size,
            cuda_graph_max_bs=2,
            tokenizer_path=model_name,
            remote_instance_weight_loader_seed_instance_ip=seed_instance_ip,
            remote_instance_weight_loader_seed_instance_service_port=seed_instance_service_port,
            remote_instance_weight_loader_send_weights_group_ports=ports,
            load_format="remote_instance",
            remote_instance_weight_loader_backend=remote_instance_loader_backend,
        )
    else:
        host, _, port = DEFAULT_URL_FOR_TEST.rpartition(":")
        url = ":".join([host, str(int(port) + 10000 + rank)])

        print(f"[sgl] rank {rank} init server on url: {url}")
        process = popen_launch_server(
            model_name,
            url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=(
                "--attention-backend",
                "--device",
                "npu",
                "--base-gpu-id",
                str(base_gpu_id),
                "--tp-size",
                str(tp_size),
                "--cuda-graph-max-bs",
                2,
                "--tokenizer-path",
                model_name,
                "--remote-instance-weight-loader-seed-instance-ip",
                seed_instance_ip,
                "--remote-instance-weight-loader-seed-instance-service-port",
                seed_instance_service_port,
                "--remote-instance-weight-loader-send-weights-group-ports",
                f"[{','.join(str(port) for port in ports)}]",
                "--load-format",
                "remote_instance",
                "--remote-instance-weight-loader-backend",
                remote_instance_loader_backend,
            ),
        )
    torch.npu.synchronize()

```
**EN:** This block implements `init_process_dst` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `init_process_dst`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 243-263: function init process dst (part 2/2) / 函数 init process dst（第 2/2 部分）
```python
    event_dst_ready_list[rank - 1].set()

    # Get weights of destination instance loaded from remote instance.
    dst_params = []
    for parameter_name in checking_parameters:
        dst_params.append(
            engine.get_weights_by_name(parameter_name, truncate_size)
            if backend == "Engine"
            else requests.get(
                f"{url}/get_weights_by_name",
                json={"name": parameter_name, "truncate_size": truncate_size},
            ).json()
        )

    param_queue.put((f"sgl_dp_{rank}_dst_params", dst_params))

    # Shutdown the engine or terminate the server process.
    if backend == "Engine":
        engine.shutdown()
    else:
        terminate_process(process)
```
**EN:** This block implements `init_process_dst` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `init_process_dst`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 266-345: test case load weights from remote instance (part 1/2) / 测试用例 load weights from remote instance（第 1/2 部分）
```python
def test_load_weights_from_remote_instance(
    tp_size,
    dp_size,
    model_name,
    backends,
    truncate_size,
    checking_parameters,
    seed_instance_ip,
    seed_instance_service_port,
    seed_instance_group_base_port,
    remote_instance_loader_backend,
):
    print(
        f"Testing model: {model_name} tp_size: {tp_size}, dp_size: {dp_size} backend: {backends} remote_instance_loader_backend: {remote_instance_loader_backend}"
    )
    param_queue = mp.Queue()
    results = {}
    event_seed_ready = mp.Event()
    event_dst_ready_list = []
    for i in range(dp_size):
        event_dst_ready = mp.Event()
        event_dst_ready_list.append(event_dst_ready)

    context = mp.spawn(
        init_process,
        args=(
            param_queue,
            truncate_size,
            tp_size,
            model_name,
            backends,
            checking_parameters,
            seed_instance_ip,
            seed_instance_service_port,
            seed_instance_group_base_port,
            event_seed_ready,
            event_dst_ready_list,
            remote_instance_loader_backend,
        ),
        nprocs=1 + dp_size,
        join=False,
    )

    while len(results) < (1 + dp_size):
        try:
            key, value = param_queue.get(timeout=5)
            results[key] = value
        except Exception as e:
            if all(not p.is_alive() for p in context.processes):
                break

    context.join()

    if len(results) != (1 + dp_size):
        raise RuntimeError(
            f"Expected {(1 + dp_size)} parameters but got {len(results)}"
        )

    params = {
        "seed": results.get("seed_params"),
        "sgl_dp_1_dest": results.get("sgl_dp_1_dst_params"),
    }

    if dp_size == 2:
        dp2_params = {
            "sgl_dp_2_dest": results.get("sgl_dp_2_dst_params"),
        }
        assert all(v is not None for v in dp2_params.values())
        params.update(dp2_params)

    # Check the correctness of weights loaded from remote instance
    # by verifying the weights of seed instance and destination instance.
    for i in range(len(params["seed"])):
        verify_params_close(
            params["seed"][i],
            params["sgl_dp_1_dest"][i],
            f"sgl_dp_1_dst_params rank {i}",
        )

        if dp_size == 2:
```
**EN:** This test exercises `test_load_weights_from_remote_instance` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_load_weights_from_remote_instance`。 这一段对应同一逻辑块的第 1 部分。

### Lines 346-357: test case load weights from remote instance (part 2/2) / 测试用例 load weights from remote instance（第 2/2 部分）
```python
            verify_params_close(
                params["seed"][i],
                params["sgl_dp_2_dest"][i],
                f"sgl_dp_2_dst_params rank {i}",
            )

    # Delete the context and close the parameter queue.
    del context
    param_queue.close()
    param_queue.join_thread()
    gc.collect()
    torch.npu.empty_cache()
```
**EN:** This test exercises `test_load_weights_from_remote_instance` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_load_weights_from_remote_instance`。 这一段对应同一逻辑块的第 2 部分。

### Lines 360-361: class TestLoadWeightsFromRemoteInstance declaration / 类 TestLoadWeightsFromRemoteInstance 声明
```python
class TestLoadWeightsFromRemoteInstance(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 362-439: test case load weights from remote instance / 测试用例 load weights from remote instance
```python
    def test_load_weights_from_remote_instance(self):

        assert torch.npu.device_count() >= 2, "At least 2 GPUs are required"
        # test_suits : tp, dp, model_name, backend, dst_instance_id
        if is_in_ci():
            mode = random.choice(["Engine", "Server"])
            remote_instance_loader_backend = random.choice(["nccl", "nccl"])
            test_suits = [
                (
                    1,
                    1,
                    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
                    [mode],
                    remote_instance_loader_backend,
                ),
            ]
        else:
            test_suits = [
                (1, 1, DEFAULT_SMALL_MODEL_NAME_FOR_TEST, ["Server"], "nccl"),
                (1, 1, DEFAULT_SMALL_MODEL_NAME_FOR_TEST, ["Server"], "nccl"),
                (2, 2, DEFAULT_SMALL_MODEL_NAME_FOR_TEST, ["Server", "Server"], "nccl"),
                (
                    1,
                    1,
                    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
                    ["Server"],
                    "nccl",
                ),
                (
                    1,
                    1,
                    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
                    ["Server"],
                    "nccl",
                ),
                (
                    2,
                    2,
                    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
                    ["Server", "Server"],
                    "nccl",
                ),
            ]

        truncate_size = 10
        checking_parameters = [
            "model.embed_tokens.weight",
            "model.layers.0.input_layernorm.weight",
            "model.layers.1.self_attn.q_proj.weight",
            "model.layers.2.self_attn.k_proj.weight",
            "model.layers.3.self_attn.v_proj.weight",
            "model.layers.4.self_attn.o_proj.weight",
            "model.layers.5.mlp.gate_proj.weight",
            "model.layers.6.mlp.up_proj.weight",
            "model.layers.7.mlp.down_proj.weight",
            "model.layers.8.post_attention_layernorm.weight",
            "model.norm.weight",
        ]

        for (
            tp_size,
            dp_size,
            model_name,
            backends,
            remote_instance_loader_backend,
        ) in test_suits:
            test_load_weights_from_remote_instance(
                tp_size,
                dp_size,
                model_name,
                backends,
                truncate_size,
                checking_parameters,
                "127.0.0.1",
                DEFAULT_PORT_FOR_SRT_TEST_RUNNER + 1000,
                60010,
                remote_instance_loader_backend,
            )
```
**EN:** This test exercises `test_load_weights_from_remote_instance` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_load_weights_from_remote_instance`。

### Lines 442-443: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `verify_params_close`: Verify if two parameter arrays are close enough. / 该代码块实现 `verify_params_close`，承担模块行为中的一个聚焦逻辑片段。
- `init_process`: This block implements `init_process` and captures one focused piece of the module's behavior. / 该代码块实现 `init_process`，承担模块行为中的一个聚焦逻辑片段。
- `init_process_seed`: This block implements `init_process_seed` and captures one focused piece of the module's behavior. / 该代码块实现 `init_process_seed`，承担模块行为中的一个聚焦逻辑片段。
- `init_process_dst`: This block implements `init_process_dst` and captures one focused piece of the module's behavior. / 该代码块实现 `init_process_dst`，承担模块行为中的一个聚焦逻辑片段。
- `test_load_weights_from_remote_instance`: This test exercises `test_load_weights_from_remote_instance` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_load_weights_from_remote_instance`。
- `TestLoadWeightsFromRemoteInstance`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLoadWeightsFromRemoteInstance.test_load_weights_from_remote_instance`: This test exercises `test_load_weights_from_remote_instance` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_load_weights_from_remote_instance`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `gc`, `os`, `random`, `unittest`
- **Third-party modules / 第三方模块**: `numpy`, `requests`, `torch`, `torch.multiprocessing`
- **Internal modules / 内部模块**: `sglang`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`, `sglang.utils`

- **Total lines / 总行数**: 443
