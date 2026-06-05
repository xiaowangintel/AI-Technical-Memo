# test_update_weights_from_distributed.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/rl/test_update_weights_from_distributed.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates update weights from distributed behavior in SGLang's rl area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 rl 领域中与 update weights from distributed 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 3-16: supporting statements / 辅助语句
```python
"""Test distributed weight updates.

This test suite simulates a distributed training environment to ensure
correct weight synchronization. On rank 0, the instruct model represents
pre-training weights, and the base model represents post-training weights.
The base model's weights are broadcasted to other ranks using the online
weight update API.

On other ranks, an engine is initialized with the instruct model, and its
parameters are verified against the Hugging Face model. After updating
weights from the distributed system, post-training weights are loaded
and verified again to ensure consistency and accuracy across the
distributed setup.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 18-44: module imports and dependencies / 模块导入与依赖
```python
import gc
import os
import random
import time
import unittest
from concurrent.futures import ThreadPoolExecutor

import numpy as np
import requests
import torch
import torch.multiprocessing as mp
from transformers import AutoModelForCausalLM

import sglang as sgl
from sglang.srt.utils import init_custom_process_group
from sglang.srt.weight_sync.tensor_bucket import FlattenedTensorBucket
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_amd_ci,
    is_in_ci,
    popen_launch_server,
)
from sglang.utils import terminate_process
```
**EN:** This block imports the modules needed by the rest of the file, including `gc`, `os`, `random`, `time`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `gc`, `os`, `random`, `time`。

### Lines 46-49: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=137, stage="extra-a", runner_config="2-gpu-large")
register_amd_ci(est_time=400, suite="stage-b-test-2-gpu-large-amd")

mp.set_start_method("spawn", force=True)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci, set_start_method.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci, set_start_method 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 52-60: function verify params close / 函数 verify params close
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

### Lines 63-65: function verify params not close / 函数 verify params not close
```python
def verify_params_not_close(params1, params2, error_msg):
    """Verify if two parameter arrays are different enough."""
    assert not np.allclose(np.array(params1), np.array(params2)), error_msg
```
**EN:** Verify if two parameter arrays are different enough. This block implements `verify_params_not_close` and captures one focused piece of the module's behavior.
**CN:** Verify if two parameter arrays are different enough. 该代码块实现 `verify_params_not_close`，承担模块行为中的一个聚焦逻辑片段。

### Lines 68-93: function warmup broadcast / 函数 warmup broadcast
```python
def _warmup_broadcast(
    hf_base_model,
    state_dict_key_to_shape,
    tie_word_embeddings,
    load_format,
    group,
):
    """Run one broadcast round to warm up RCCL before timing."""
    broadcast_parameters = list(state_dict_key_to_shape.keys())
    if tie_word_embeddings:
        broadcast_parameters.remove("lm_head.weight")

    if load_format == "flattened_bucket":
        named_tensors = [
            (name, hf_base_model.get_parameter(name)) for name in broadcast_parameters
        ]
        bucket = FlattenedTensorBucket(named_tensors=named_tensors)
        flattened_tensor = bucket.get_flattened_tensor()
        torch.distributed.broadcast(flattened_tensor, src=0, group=group)
    else:
        for name in broadcast_parameters:
            torch.distributed.broadcast(
                hf_base_model.get_parameter(name),
                src=0,
                group=group,
            )
```
**EN:** Run one broadcast round to warm up RCCL before timing. This block implements `_warmup_broadcast` and captures one focused piece of the module's behavior.
**CN:** Run one broadcast round to warm up RCCL before timing. 该代码块实现 `_warmup_broadcast`，承担模块行为中的一个聚焦逻辑片段。

### Lines 96-119: function warmup update / 函数 warmup update
```python
def _warmup_update(
    backend, engine, url, names, dtypes, shapes, load_format, pause_generation_mode
):
    """Run one update round to warm up RCCL before timing."""
    if backend == "Engine":
        engine.update_weights_from_distributed(
            names,
            dtypes=dtypes,
            shapes=shapes,
            group_name="test_parameter_update_group",
            load_format=load_format,
        )
    else:
        requests.post(
            f"{url}/update_weights_from_distributed",
            json={
                "names": names,
                "dtypes": dtypes,
                "shapes": shapes,
                "group_name": "test_parameter_update_group",
                "load_format": load_format,
                "flush_cache": not (pause_generation_mode == "in_place"),
            },
        )
```
**EN:** Run one update round to warm up RCCL before timing. This block implements `_warmup_update` and captures one focused piece of the module's behavior.
**CN:** Run one update round to warm up RCCL before timing. 该代码块实现 `_warmup_update`，承担模块行为中的一个聚焦逻辑片段。

### Lines 122-167: function init process / 函数 init process
```python
def init_process(
    rank,
    world_size,
    param_queue,
    truncate_size,
    state_dict_key_to_shape,
    tp_size,
    model_name,
    backend,
    checking_parameters,
    tie_word_embeddings,
    load_format,
    barrier,
    pause_generation_mode,
):
    torch.cuda.set_device(rank)

    if rank == 0:
        init_process_hf(
            rank,
            world_size,
            param_queue,
            truncate_size,
            model_name,
            checking_parameters,
            tie_word_embeddings,
            state_dict_key_to_shape,
            load_format,
            barrier,
        )
    elif rank in [1, 2]:
        init_process_sgl(
            rank,
            world_size,
            param_queue,
            truncate_size,
            model_name,
            checking_parameters,
            tie_word_embeddings,
            state_dict_key_to_shape,
            backend,
            tp_size,
            load_format,
            barrier,
            pause_generation_mode,
        )
```
**EN:** This block implements `init_process` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `init_process`，承担模块行为中的一个聚焦逻辑片段。

### Lines 170-249: function init process hf (part 1/2) / 函数 init process hf（第 1/2 部分）
```python
def init_process_hf(
    rank,
    world_size,
    param_queue,
    truncate_size,
    model_name,
    checking_parameters,
    tie_word_embeddings,
    state_dict_key_to_shape,
    load_format,
    barrier,
):
    # These two environment variables are very important
    # to avoid unexpected behaviors of CUDA and NCCL.
    os.environ["NCCL_CUMEM_ENABLE"] = "0"
    os.environ["NCCL_NVLS_ENABLE"] = "0"

    # Load model and get parameters
    hf_instruct_model = AutoModelForCausalLM.from_pretrained(
        model_name,
        torch_dtype="bfloat16",
        tie_word_embeddings=tie_word_embeddings,
    ).to("cuda:0")
    base_model_name = model_name.replace("-Instruct", "")
    hf_base_model = AutoModelForCausalLM.from_pretrained(
        base_model_name,
        torch_dtype="bfloat16",
        tie_word_embeddings=tie_word_embeddings,
    ).to("cuda:0")

    hf_instruct_params = []
    hf_base_params = []

    print("[hf] get parameter in hf instruct model and base model")
    for parameter_name in checking_parameters:
        hf_instruct_params.append(
            hf_instruct_model.get_parameter(parameter_name)[:truncate_size]
            .cpu()
            .detach()
            .float()
            .numpy()
            .tolist()
        )
        hf_base_params.append(
            hf_base_model.get_parameter(parameter_name)[:truncate_size]
            .cpu()
            .detach()
            .float()
            .numpy()
            .tolist()
        )

    param_queue.put(("hf_instruct_params", hf_instruct_params))
    param_queue.put(("hf_base_params", hf_base_params))

    # Init weight update group for rank 0 (the training engine in RLHF).
    port = 60000 + int(os.environ.get("CUDA_VISIBLE_DEVICES", "0")[0]) * 100
    init_method = f"tcp://localhost:{port}"
    print(f"[hf] {rank=} {world_size=} init custom process group. {init_method=}")
    group = init_custom_process_group(
        backend="nccl",
        init_method=init_method,
        world_size=world_size,
        rank=rank,
        group_name="test_parameter_update_group",
    )
    torch.cuda.synchronize()
    barrier.wait()

    # Warmup: trigger RCCL initialization so it's excluded from timing
    if is_in_amd_ci():
        _warmup_broadcast(
            hf_base_model,
            state_dict_key_to_shape,
            tie_word_embeddings,
            load_format,
            group,
        )
        torch.cuda.synchronize()

```
**EN:** This block implements `init_process_hf` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `init_process_hf`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 250-291: function init process hf (part 2/2) / 函数 init process hf（第 2/2 部分）
```python
    time_begin_broadcast = time.perf_counter()

    # The last parameter is lm_head.weight, which is tied
    # with embed_tokens.weight. Actually, we only need
    # to broadcast embed_tokens.weight once.
    broadcast_parameters = list(state_dict_key_to_shape.keys())
    if tie_word_embeddings:
        broadcast_parameters.remove("lm_head.weight")

    if load_format == "flattened_bucket":
        named_tensors = [
            (parameter_name, hf_base_model.get_parameter(parameter_name))
            for parameter_name in broadcast_parameters
        ]
        bucket = FlattenedTensorBucket(named_tensors=named_tensors)
        flattened_tensor = bucket.get_flattened_tensor()
        torch.distributed.broadcast(flattened_tensor, src=0, group=group)
    else:
        # Broadcast all the weights from the training
        # engine to other ranks (inference engine).
        for parameter_name in broadcast_parameters:
            torch.distributed.broadcast(
                hf_base_model.get_parameter(parameter_name),
                src=0,
                group=group,
            )
    torch.cuda.synchronize()
    time_end_broadcast = time.perf_counter()

    # Measure the latency of broadcasting/weights update.
    broadcast_time = time_end_broadcast - time_begin_broadcast
    print(f"[hf] {rank=} {broadcast_time=:.3f}s")
    param_queue.put(("broadcast_time", broadcast_time))

    # Destroy process group and release related resource
    torch.distributed.destroy_process_group(group)

    # Delete the huggingface models to free up memory.
    del hf_instruct_model
    del hf_base_model
    gc.collect()
    torch.cuda.empty_cache()
```
**EN:** This block implements `init_process_hf` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `init_process_hf`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 294-373: function init process sgl (part 1/3) / 函数 init process sgl（第 1/3 部分）
```python
def init_process_sgl(
    rank,
    world_size,
    param_queue,
    truncate_size,
    model_name,
    checking_parameters,
    tie_word_embeddings,
    state_dict_key_to_shape,
    backend,
    tp_size,
    load_format,
    barrier,
    pause_generation_mode,
):
    torch.cuda.set_device(rank)
    torch.cuda.synchronize()
    base_gpu_id = 1 if rank == 1 else 1 + tp_size
    if backend == "Engine":
        print(f"[sgl] rank {rank} init engine")
        engine = sgl.Engine(
            model_path=model_name,
            base_gpu_id=base_gpu_id,
            tp_size=tp_size,
            cuda_graph_max_bs=2,
        )
    else:
        if rank == 1:
            url = DEFAULT_URL_FOR_TEST
        else:
            host, _, port = DEFAULT_URL_FOR_TEST.rpartition(":")
            url = ":".join([host, str(int(port) + 10000)])

        print(f"[sgl] rank {rank} init server on url: {url}")
        process = popen_launch_server(
            model_name,
            url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=(
                "--base-gpu-id",
                str(base_gpu_id),
                "--tp-size",
                str(tp_size),
                "--cuda-graph-max-bs",
                2,
            ),
        )
    torch.cuda.synchronize()

    # Get weights of instruct model, i.e. pre-training weights.
    instruct_params = []
    for parameter_name in checking_parameters:
        instruct_params.append(
            engine.get_weights_by_name(parameter_name, truncate_size)
            if backend == "Engine"
            else requests.get(
                f"{url}/get_weights_by_name",
                json={"name": parameter_name, "truncate_size": truncate_size},
            ).json()
        )

    param_queue.put((f"sgl_dp_{rank}_instruct_params", instruct_params))

    port = 60000 + int(os.environ.get("CUDA_VISIBLE_DEVICES", "0")[0]) * 100

    # Init weight update group with the training engine.
    if backend == "Engine":
        engine.init_weights_update_group(
            master_address="localhost",
            master_port=str(port),
            rank_offset=base_gpu_id,
            world_size=world_size,
            group_name="test_parameter_update_group",
            backend="nccl",
        )
    else:
        requests.post(
            f"{url}/init_weights_update_group",
            json={
                "master_address": "localhost",
```
**EN:** This block implements `init_process_sgl` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `init_process_sgl`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 374-453: function init process sgl (part 2/3) / 函数 init process sgl（第 2/3 部分）
```python
                "master_port": str(port),
                "rank_offset": base_gpu_id,
                "world_size": world_size,
                "group_name": "test_parameter_update_group",
                "backend": "nccl",
            },
        )

    if pause_generation_mode in ["in_place", "retract"]:

        def run_decode(max_new_tokens=32):
            response = requests.post(
                url + "/generate",
                json={
                    "text": f"Question: {random.randint(0, 100)},The capital of France is",
                    "sampling_params": {
                        "temperature": 0,
                        "max_new_tokens": max_new_tokens,
                        "ignore_eos": True,
                    },
                },
            )
            return response.json()

        with ThreadPoolExecutor(32) as executor:
            futures = [executor.submit(run_decode, 1000) for _ in range(32)]
            time.sleep(2)

    # The last parameter is lm_head.weight, which is tied
    # with embed_tokens.weight. Actually, we only need
    # to update embed_tokens.weight once.
    tie_word_embeddings = (
        True if model_name == DEFAULT_SMALL_MODEL_NAME_FOR_TEST else False
    )
    update_parameters = list(state_dict_key_to_shape.keys())
    if tie_word_embeddings:
        update_parameters.remove("lm_head.weight")

    # Get weights from the training engine and update the inference engine.
    names = [parameter_name for parameter_name in update_parameters]
    dtypes = [torch.bfloat16 if backend == "Engine" else "bfloat16"] * len(names)
    shapes = [state_dict_key_to_shape[parameter_name] for parameter_name in names]

    if pause_generation_mode in ["in_place", "retract"]:
        requests.post(
            url + "/pause_generation",
            json={"mode": pause_generation_mode},
        )
    torch.cuda.synchronize()
    barrier.wait()

    # Warmup: trigger RCCL initialization so it's excluded from timing
    if is_in_amd_ci():
        _warmup_update(
            backend,
            engine if backend == "Engine" else None,
            url if backend != "Engine" else None,
            names,
            dtypes,
            shapes,
            load_format,
            pause_generation_mode,
        )
        torch.cuda.synchronize()

    time_begin_update = time.perf_counter()
    if backend == "Engine":
        engine.update_weights_from_distributed(
            names,
            dtypes=dtypes,
            shapes=shapes,
            group_name="test_parameter_update_group",
            load_format=load_format,
        )
    else:
        requests.post(
            f"{url}/update_weights_from_distributed",
            json={
                "names": names,
                "dtypes": dtypes,
```
**EN:** This block implements `init_process_sgl` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `init_process_sgl`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 454-519: function init process sgl (part 3/3) / 函数 init process sgl（第 3/3 部分）
```python
                "shapes": shapes,
                "group_name": "test_parameter_update_group",
                "load_format": load_format,
                "flush_cache": not (pause_generation_mode == "in_place"),
            },
        )
    torch.cuda.synchronize()
    time_end_update = time.perf_counter()
    if pause_generation_mode in ["in_place", "retract"]:
        requests.post(
            url + "/continue_generation",
            json={},
        )

        # discard unfinished requests to save test overhead
        time.sleep(2)
        requests.post(
            url + "/pause_generation",
            json={"mode": "abort"},
        )

    # Measure the latency of broadcast/weights update.
    update_time = time_end_update - time_begin_update
    print(
        f"[sgl] fully update model_name {model_name} rank {rank} parameter from distributed time: {update_time:.3f}s"
    )
    param_queue.put((f"update_sgl_dp_{rank}_time", update_time))

    # Get the weights of post-training model after weights update for correctness check.
    base_params = []
    for parameter_name in checking_parameters:
        if backend == "Engine":
            base_params.append(
                engine.get_weights_by_name(parameter_name, truncate_size)
            )
        else:
            base_params.append(
                requests.get(
                    f"{url}/get_weights_by_name",
                    json={
                        "name": parameter_name,
                        "truncate_size": truncate_size,
                    },
                ).json()
            )
    param_queue.put((f"sgl_dp_{rank}_base_params", base_params))

    if backend == "Engine":
        success, _ = engine.destroy_weights_update_group(
            group_name="test_parameter_update_group",
        )
        assert success is True
    else:
        response = requests.post(
            f"{url}/destroy_weights_update_group",
            json={
                "group_name": "test_parameter_update_group",
            },
        )
        assert response.status_code == 200

    # Shutdown the engine or terminate the server process.
    if backend == "Engine":
        engine.shutdown()
    else:
        terminate_process(process)
```
**EN:** This block implements `init_process_sgl` and captures one focused piece of the module's behavior. This chunk continues part 3 of the same logical block.
**CN:** 该代码块实现 `init_process_sgl`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 3 部分。

### Lines 522-527: helper routine assert tied weights / 辅助流程 assert tied weights
```python
def assert_tied_weights(params_list, message, should_be_tied):
    for params in params_list:
        if should_be_tied:
            assert np.allclose(params[0], params[-1]), message
        else:
            assert not np.allclose(params[0], params[-1]), message
```
**EN:** This helper encapsulates `assert_tied_weights` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `assert_tied_weights`，以便周围测试复用准备、执行或校验逻辑。

### Lines 530-609: test case update weights from distributed (part 1/3) / 测试用例 update weights from distributed（第 1/3 部分）
```python
def test_update_weights_from_distributed(
    tp_size,
    dp_size,
    model_name,
    backend,
    state_dict_key_to_shape,
    truncate_size,
    checking_parameters,
    load_format=None,
    pause_generation_mode=None,
):
    tie_word_embeddings = (
        True if model_name == DEFAULT_SMALL_MODEL_NAME_FOR_TEST else False
    )

    print(
        f"Testing model: {model_name} tp_size: {tp_size}, dp_size: {dp_size} backend: {backend}"
    )
    param_queue = mp.Queue()
    results = {}
    barrier = mp.Barrier(1 + dp_size)

    context = mp.spawn(
        init_process,
        args=(
            1 + tp_size * dp_size,
            param_queue,
            truncate_size,
            state_dict_key_to_shape,
            tp_size,
            model_name,
            backend,
            checking_parameters,
            tie_word_embeddings,
            load_format,
            barrier,
            pause_generation_mode,
        ),
        nprocs=1 + dp_size,
        join=False,
    )

    while len(results) < 3 * (1 + dp_size):
        try:
            key, value = param_queue.get(timeout=5)
            results[key] = value
        except Exception as e:
            if all(not p.is_alive() for p in context.processes):
                break

    context.join()

    if len(results) != 3 * (1 + dp_size):
        raise RuntimeError(
            f"Expected {3 * (1 + dp_size)} parameters but got {len(results)}"
        )

    params = {
        "hf_instruct": results.get("hf_instruct_params"),
        "hf_base": results.get("hf_base_params"),
        "sgl_dp_1_instruct": results.get("sgl_dp_1_instruct_params"),
        "sgl_dp_1_base": results.get("sgl_dp_1_base_params"),
        "broadcast_time": results.get("broadcast_time"),
        "update_sgl_dp_1_time": results.get("update_sgl_dp_1_time"),
    }

    if dp_size == 2:
        dp2_params = {
            "sgl_dp_2_instruct": results.get("sgl_dp_2_instruct_params"),
            "sgl_dp_2_base": results.get("sgl_dp_2_base_params"),
            "update_sgl_dp_2_time": results.get("update_sgl_dp_2_time"),
        }
        assert all(v is not None for v in dp2_params.values())
        params.update(dp2_params)

    # Check the correctness of weights update by verifying
    # the weights of instruct model and base model.
    for i in range(len(params["hf_instruct"])):
        verify_params_close(
            params["hf_instruct"][i],
```
**EN:** This test exercises `test_update_weights_from_distributed` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_distributed`。 这一段对应同一逻辑块的第 1 部分。

### Lines 610-689: test case update weights from distributed (part 2/3) / 测试用例 update weights from distributed（第 2/3 部分）
```python
            params["sgl_dp_1_instruct"][i],
            f"sgl_dp_1_instruct_params rank {i}",
        )

        verify_params_close(
            params["hf_base"][i],
            params["sgl_dp_1_base"][i],
            f"sgl_dp_1_base_params rank {i}",
        )

        verify_params_not_close(
            params["hf_instruct"][i],
            params["hf_base"][i],
            f"hf_instruct_params rank {i}",
        )

        if dp_size == 2:
            verify_params_close(
                params["hf_base"][i],
                params["sgl_dp_2_base"][i],
                f"sgl_dp_2_base_params rank {i}",
            )
            verify_params_close(
                params["hf_instruct"][i],
                params["sgl_dp_2_instruct"][i],
                f"sgl_dp_2_instruct_params rank {i}",
            )

    assert len(params["hf_instruct"]) == len(
        params["hf_base"]
    ), "hf_instruct_params and hf_base_params have different lengths"

    # Check if the weights of lm_head are tied with embed_tokens.
    params_to_check = [
        (
            params["hf_instruct"],
            "lm_head.weight is not tied with embed_tokens.weight",
        ),
        (
            params["hf_base"],
            "lm_head.weight is not tied with embed_tokens.weight",
        ),
        (
            params["sgl_dp_1_instruct"],
            "lm_head.weight is not tied with embed_tokens.weight",
        ),
        (
            params["sgl_dp_1_base"],
            "lm_head.weight is not tied with embed_tokens.weight",
        ),
    ]

    if dp_size == 2:
        params_to_check.extend(
            [
                (
                    params["sgl_dp_2_instruct"],
                    "lm_head.weight is not tied with embed_tokens.weight",
                ),
                (
                    params["sgl_dp_2_base"],
                    "lm_head.weight is not tied with embed_tokens.weight",
                ),
            ]
        )

    assert_tied_weights(
        [params for params, _ in params_to_check],
        (
            "lm_head.weight is not tied with embed_tokens.weight"
            if tie_word_embeddings
            else "lm_head.weight is tied with embed_tokens.weight"
        ),
        tie_word_embeddings,
    )

    # Time limit for broadcast and update on CI is 3 / 6
    # On local H100, it's 1 / 2
    time_limit = 3 if model_name == DEFAULT_SMALL_MODEL_NAME_FOR_TEST else 6

```
**EN:** This test exercises `test_update_weights_from_distributed` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_distributed`。 这一段对应同一逻辑块的第 2 部分。

### Lines 690-708: test case update weights from distributed (part 3/3) / 测试用例 update weights from distributed（第 3/3 部分）
```python
    assert (
        params["broadcast_time"] < time_limit
    ), f"broadcast_time exceeds time limit {time_limit}s"

    assert (
        params["update_sgl_dp_1_time"] < time_limit
    ), f"update_sgl_dp_one_time exceeds time limit {time_limit}s"

    if dp_size == 2:
        assert (
            params["update_sgl_dp_2_time"] < time_limit
        ), f"update_sgl_dp_two_time exceeds time limit {time_limit}s"

    # Delete the context and close the parameter queue.
    del context
    param_queue.close()
    param_queue.join_thread()
    gc.collect()
    torch.cuda.empty_cache()
```
**EN:** This test exercises `test_update_weights_from_distributed` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 3 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_distributed`。 这一段对应同一逻辑块的第 3 部分。

### Lines 711-712: class TestUpdateWeightsFromDistributed declaration / 类 TestUpdateWeightsFromDistributed 声明
```python
class TestUpdateWeightsFromDistributed(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 713-792: test case update weights from distributed (part 1/2) / 测试用例 update weights from distributed（第 1/2 部分）
```python
    def test_update_weights_from_distributed(self):

        assert torch.cuda.device_count() >= 2, "At least 2 GPUs are required"
        # test_suits : tp, dp, model_name, backend
        if is_in_ci():
            mode = random.choice(["Engine", "Server"])
            if mode == "Server":
                pause_generation_mode = random.choice(["in_place", "retract"])
            else:
                pause_generation_mode = None
            load_format = random.choice(["flattened_bucket", None])
            test_suits = [
                (
                    1,
                    1,
                    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
                    mode,
                    pause_generation_mode,
                    load_format,
                ),
            ]
        else:
            test_suits = [
                (
                    1,
                    1,
                    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
                    "Engine",
                    None,
                    random.choice(["flattened_bucket", None]),
                ),
                (
                    1,
                    1,
                    DEFAULT_MODEL_NAME_FOR_TEST,
                    "Sever",
                    random.choice(["in_place", "retract"]),
                    random.choice(["flattened_bucket", None]),
                ),
            ]

            if torch.cuda.device_count() >= 4:
                test_suits.extend(
                    [
                        (
                            2,
                            1,
                            DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
                            "Engine",
                            None,
                            random.choice(["flattened_bucket", None]),
                        ),
                        (
                            1,
                            2,
                            DEFAULT_MODEL_NAME_FOR_TEST,
                            "Server",
                            random.choice(["in_place", "retract"]),
                            random.choice(["flattened_bucket", None]),
                        ),
                    ]
                )

            if torch.cuda.device_count() >= 5:
                test_suits.extend(
                    [
                        (
                            2,
                            2,
                            DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
                            "Engine",
                            None,
                            random.choice(["flattened_bucket", None]),
                        ),
                        (
                            2,
                            2,
                            DEFAULT_MODEL_NAME_FOR_TEST,
                            "Server",
                            random.choice(["in_place", "retract"]),
```
**EN:** This test exercises `test_update_weights_from_distributed` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_distributed`。 这一段对应同一逻辑块的第 1 部分。

### Lines 793-848: test case update weights from distributed (part 2/2) / 测试用例 update weights from distributed（第 2/2 部分）
```python
                            random.choice(["flattened_bucket", None]),
                        ),
                    ]
                )

        model_state_dict_shapes = {}
        test_models = [test_suit[2] for test_suit in test_suits]

        for model_name in test_models:
            model = AutoModelForCausalLM.from_pretrained(
                model_name, torch_dtype="bfloat16"
            ).to("cuda:0")
            state_dict = model.state_dict()
            state_dict_keys = list(state_dict.keys())
            model_state_dict_shapes[model_name] = {
                key: state_dict[key].shape for key in state_dict_keys
            }
            del model
            gc.collect()
            torch.cuda.empty_cache()

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
            "lm_head.weight",
        ]

        for (
            tp_size,
            dp_size,
            model_name,
            backend,
            pause_generation_mode,
            load_format,
        ) in test_suits:
            test_update_weights_from_distributed(
                tp_size,
                dp_size,
                model_name,
                backend,
                model_state_dict_shapes[model_name],
                truncate_size,
                checking_parameters,
                load_format,
                pause_generation_mode,
            )
```
**EN:** This test exercises `test_update_weights_from_distributed` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_distributed`。 这一段对应同一逻辑块的第 2 部分。

### Lines 851-852: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `verify_params_close`: Verify if two parameter arrays are close enough. / 该代码块实现 `verify_params_close`，承担模块行为中的一个聚焦逻辑片段。
- `verify_params_not_close`: Verify if two parameter arrays are different enough. / 该代码块实现 `verify_params_not_close`，承担模块行为中的一个聚焦逻辑片段。
- `_warmup_broadcast`: Run one broadcast round to warm up RCCL before timing. / 该代码块实现 `_warmup_broadcast`，承担模块行为中的一个聚焦逻辑片段。
- `_warmup_update`: Run one update round to warm up RCCL before timing. / 该代码块实现 `_warmup_update`，承担模块行为中的一个聚焦逻辑片段。
- `init_process`: This block implements `init_process` and captures one focused piece of the module's behavior. / 该代码块实现 `init_process`，承担模块行为中的一个聚焦逻辑片段。
- `init_process_hf`: This block implements `init_process_hf` and captures one focused piece of the module's behavior. / 该代码块实现 `init_process_hf`，承担模块行为中的一个聚焦逻辑片段。
- `init_process_sgl`: This block implements `init_process_sgl` and captures one focused piece of the module's behavior. / 该代码块实现 `init_process_sgl`，承担模块行为中的一个聚焦逻辑片段。
- `assert_tied_weights`: This helper encapsulates `assert_tied_weights` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `assert_tied_weights`，以便周围测试复用准备、执行或校验逻辑。
- `TestUpdateWeightsFromDistributed.test_update_weights_from_distributed`: This test exercises `test_update_weights_from_distributed` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_distributed`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `gc`, `os`, `random`, `time`, `unittest`, `concurrent.futures`
- **Third-party modules / 第三方模块**: `numpy`, `requests`, `torch`, `torch.multiprocessing`, `transformers`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang`, `sglang.srt.utils`, `sglang.srt.weight_sync.tensor_bucket`, `sglang.test.test_utils`, `sglang.utils`

- **Total lines / 总行数**: 852
