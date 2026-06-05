# test_update_weights_from_tensor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/rl/test_update_weights_from_tensor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates update weights from tensor behavior in SGLang's rl area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 rl 领域中与 update weights from tensor 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 3-4: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=147, stage="extra-a", runner_config="1-gpu-small")
register_amd_ci(est_time=195, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 6-25: module imports and dependencies / 模块导入与依赖
```python
import gc
import json
import random
import time
import unittest
from concurrent.futures import ThreadPoolExecutor, as_completed

import requests
import torch

import sglang as sgl
from sglang.srt.utils import MultiprocessingSerializer, kill_process_tree
from sglang.srt.weight_sync.tensor_bucket import FlattenedTensorBucket
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `gc`, `json`, `random`, `time`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `gc`, `json`, `random`, `time`。

### Lines 28-57: test case update weights from tensor / 测试用例 update weights from tensor
```python
def test_update_weights_from_tensor(tp_size):
    assert torch.cuda.device_count() >= tp_size, f"At least {tp_size} GPUs are required"
    torch.cuda.empty_cache()

    engine = sgl.Engine(model_path=DEFAULT_SMALL_MODEL_NAME_FOR_TEST, tp_size=tp_size)

    param_names = [f"model.layers.{i}.mlp.up_proj.weight" for i in range(6, 16)]

    _check_param(engine, param_names[0], [0.0087, -0.0214, -0.0004, 0.0039, 0.0110])

    memory_before = torch.cuda.memory_allocated()
    new_tensor = torch.full((16384, 2048), 1.5, device="cuda")

    time_start = time.perf_counter()
    engine.update_weights_from_tensor([(x, new_tensor) for x in param_names])
    print(f"Time delta: {time.perf_counter() - time_start:.03f}")

    for param_name in param_names[:3]:
        _check_param(engine, param_name, [1.5] * 5)

    engine.shutdown()

    del new_tensor
    gc.collect()
    torch.cuda.ipc_collect()
    torch.cuda.empty_cache()
    memory_after = torch.cuda.memory_allocated()
    assert (
        memory_after <= memory_before + 1024
    ), f"Memory leak detected: {memory_after - memory_before} bytes"
```
**EN:** This test exercises `test_update_weights_from_tensor` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_tensor`。

### Lines 60-60: class TestUpdateWeightsFromTensor declaration / 类 TestUpdateWeightsFromTensor 声明
```python
class TestUpdateWeightsFromTensor(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 61-68: test case update weights from tensor / 测试用例 update weights from tensor
```python
    def test_update_weights_from_tensor(self):
        tp_sizes = [1, 2]
        for tp_size in tp_sizes:
            if torch.cuda.device_count() < tp_size:
                continue

            with self.subTest(tp_size=tp_size):
                test_update_weights_from_tensor(tp_size)
```
**EN:** This test exercises `test_update_weights_from_tensor` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_tensor`。

### Lines 70-96: test case update weights from tensor load format direct / 测试用例 update weights from tensor load format direct
```python
    def test_update_weights_from_tensor_load_format_direct(self):
        engine = sgl.Engine(model_path=DEFAULT_SMALL_MODEL_NAME_FOR_TEST)

        write_param_names = [
            f"model.layers.{i}.self_attn.qkv_proj.weight" for i in range(6, 16)
        ]
        read_param_names = [
            f"model.layers.{i}.self_attn.k_proj.weight" for i in range(6, 16)
        ]

        _check_param(
            engine, read_param_names[0], [-0.0198, 0.0227, 0.0168, 0.0232, -0.0178]
        )

        new_tensor = torch.full((3072, 2048), 1.5)
        engine.update_weights_from_tensor(
            [
                (write_param_name, new_tensor.clone())
                for write_param_name in write_param_names
            ],
            load_format="direct",
        )

        for read_param_name in read_param_names[:3]:
            _check_param(engine, read_param_name, [1.5] * 5)

        engine.shutdown()
```
**EN:** This test exercises `test_update_weights_from_tensor_load_format_direct` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_tensor_load_format_direct`。

### Lines 98-130: test case update weights from tensor load format custom / 测试用例 update weights from tensor load format custom
```python
    def test_update_weights_from_tensor_load_format_custom(self):
        custom_loader_name = (
            "sglang.srt.model_executor.model_runner._model_load_weights_direct"
        )
        engine = sgl.Engine(
            model_path=DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
            custom_weight_loader=[custom_loader_name],
        )

        write_param_names = [
            f"model.layers.{i}.self_attn.qkv_proj.weight" for i in range(6, 16)
        ]
        read_param_names = [
            f"model.layers.{i}.self_attn.k_proj.weight" for i in range(6, 16)
        ]

        _check_param(
            engine, read_param_names[0], [-0.0198, 0.0227, 0.0168, 0.0232, -0.0178]
        )

        new_tensor = torch.full((3072, 2048), 1.5)
        engine.update_weights_from_tensor(
            [
                (write_param_name, new_tensor.clone())
                for write_param_name in write_param_names
            ],
            load_format=custom_loader_name,
        )

        for read_param_name in read_param_names[:3]:
            _check_param(engine, read_param_name, [1.5] * 5)

        engine.shutdown()
```
**EN:** This test exercises `test_update_weights_from_tensor_load_format_custom` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_tensor_load_format_custom`。

### Lines 132-183: test case update weights from tensor load format flattened bucket / 测试用例 update weights from tensor load format flattened bucket
```python
    def test_update_weights_from_tensor_load_format_flattened_bucket(self):
        """Test updating weights using flattened_bucket format"""
        engine = sgl.Engine(model_path=DEFAULT_SMALL_MODEL_NAME_FOR_TEST)

        # Create a small set of parameters for testing
        param_names = [f"model.layers.{i}.mlp.up_proj.weight" for i in range(6, 10)]

        # Check original values
        _check_param(engine, param_names[0], [0.0087, -0.0214, -0.0004, 0.0039, 0.0110])

        # Create new tensors with different values
        new_tensors = []
        for _, name in enumerate(param_names):
            # Create tensors with different values for each parameter
            value = 2.0  # Different value for each parameter
            new_tensor = torch.full((16384, 2048), value, device="cuda")
            new_tensors.append((name, new_tensor))

        # Create a flattened bucket
        flattened_bucket = FlattenedTensorBucket(named_tensors=new_tensors)

        # Extract the flattened tensor and metadata in the format expected by model_runner
        flattened_tensor = flattened_bucket.get_flattened_tensor()
        metadata = flattened_bucket.get_metadata()

        # Create the dict format expected by _update_weights_from_flattened_bucket
        bucket_dict = {"flattened_tensor": flattened_tensor, "metadata": metadata}

        # Serialize the bucket data
        from sglang.srt.utils import MultiprocessingSerializer

        serialized_bucket = MultiprocessingSerializer.serialize(
            bucket_dict, output_str=True
        )

        # Create a list where each rank contains the same serialized data
        # This simulates the distributed environment where each rank has the same data
        serialized_bucket_list = [serialized_bucket]

        # Update weights using flattened_bucket format
        time_start = time.perf_counter()
        engine.update_weights_from_tensor(
            named_tensors=serialized_bucket_list, load_format="flattened_bucket"
        )
        update_time = time.perf_counter() - time_start
        print(f"Flattened bucket update time: {update_time:.03f}")

        # Verify the weights were updated correctly
        for i, param_name in enumerate(param_names):
            _check_param(engine, param_name, [2.0] * 5)

        engine.shutdown()
```
**EN:** Test updating weights using flattened_bucket format This test exercises `test_update_weights_from_tensor_load_format_flattened_bucket` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test updating weights using flattened_bucket format 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_tensor_load_format_flattened_bucket`。

### Lines 186-186: class TestServerUpdateWeightsFromTensorNonBlocking declaration / 类 TestServerUpdateWeightsFromTensorNonBlocking 声明
```python
class TestServerUpdateWeightsFromTensorNonBlocking(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 187-196: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=["--max-running-requests", 8],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 198-200: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 202-214: helper routine run decode / 辅助流程 run decode
```python
    def run_decode(self, max_new_tokens=32):
        response = requests.post(
            self.base_url + "/generate",
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
```
**EN:** This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 216-220: helper routine get model info / 辅助流程 get model info
```python
    def get_model_info(self):
        response = requests.get(self.base_url + "/get_model_info")
        model_path = response.json()["model_path"]
        print(json.dumps(response.json()))
        return model_path
```
**EN:** This helper encapsulates `get_model_info` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_model_info`，以便周围测试复用准备、执行或校验逻辑。

### Lines 222-228: method pause generation / 方法 pause generation
```python
    def pause_generation(self, mode):
        response = requests.post(
            self.base_url + "/pause_generation",
            json={"mode": mode},
        )
        ret = response.json()
        return ret
```
**EN:** This block implements `pause_generation` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `pause_generation`，承担模块行为中的一个聚焦逻辑片段。

### Lines 230-236: method continue generation / 方法 continue generation
```python
    def continue_generation(self):
        response = requests.post(
            self.base_url + "/continue_generation",
            json={},
        )
        ret = response.json()
        return ret
```
**EN:** This block implements `continue_generation` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `continue_generation`，承担模块行为中的一个聚焦逻辑片段。

### Lines 238-249: helper routine run update weights / 辅助流程 run update weights
```python
    def run_update_weights(self, named_tensors, flush_cache=True):
        response = requests.post(
            self.base_url + "/update_weights_from_tensor",
            json={
                "serialized_named_tensors": [
                    MultiprocessingSerializer.serialize(named_tensors, output_str=True)
                ],
                "flush_cache": flush_cache,
            },
        )
        ret = response.json()
        return ret
```
**EN:** This helper encapsulates `run_update_weights` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_update_weights`，以便周围测试复用准备、执行或校验逻辑。

### Lines 251-283: test case update weights / 测试用例 update weights
```python
    def test_update_weights(self):
        num_requests = 32
        with ThreadPoolExecutor(num_requests) as executor:
            futures = [
                executor.submit(self.run_decode, 3000) for _ in range(num_requests)
            ]

            # ensure the decode has been started
            time.sleep(2)

            param_names = [f"model.layers.{i}.mlp.up_proj.weight" for i in range(6, 16)]
            new_tensor = torch.full((16384, 2048), 1.5, device="cuda")
            named_tensors = [(x, new_tensor) for x in param_names]

            # abort mode ensures server is totally idle before returning
            ret = self.pause_generation("abort")
            ret = self.run_update_weights(named_tensors, flush_cache=True)
            self.assertTrue(ret["success"])
            ret = self.continue_generation()

            # requests were aborted by pause_generation("abort")
            for future in as_completed(futures):
                future.result()

            for param_name in param_names[:3]:
                response = requests.post(
                    self.base_url + "/get_weights_by_name",
                    json={"name": param_name},
                )
                actual_values = torch.tensor(response.json())[0, :5]
                assert torch.allclose(
                    actual_values, torch.tensor([1.5] * 5), atol=0.002
                ), f"{actual_values=}"
```
**EN:** This test exercises `test_update_weights` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights`。

### Lines 286-290: function check param / 函数 check param
```python
def _check_param(engine, param_name, expect_values):
    actual_values = torch.tensor(engine.get_weights_by_name(param_name))[0, :5]
    assert torch.allclose(
        actual_values, torch.tensor(expect_values), atol=0.002
    ), f"{actual_values=}"
```
**EN:** This block implements `_check_param` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_check_param`，承担模块行为中的一个聚焦逻辑片段。

### Lines 293-294: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `test_update_weights_from_tensor`: This test exercises `test_update_weights_from_tensor` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_tensor`。
- `TestUpdateWeightsFromTensor`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestServerUpdateWeightsFromTensorNonBlocking`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_check_param`: This block implements `_check_param` and captures one focused piece of the module's behavior. / 该代码块实现 `_check_param`，承担模块行为中的一个聚焦逻辑片段。
- `TestUpdateWeightsFromTensor.test_update_weights_from_tensor`: This test exercises `test_update_weights_from_tensor` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_tensor`。
- `TestUpdateWeightsFromTensor.test_update_weights_from_tensor_load_format_direct`: This test exercises `test_update_weights_from_tensor_load_format_direct` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_tensor_load_format_direct`。
- `TestUpdateWeightsFromTensor.test_update_weights_from_tensor_load_format_custom`: This test exercises `test_update_weights_from_tensor_load_format_custom` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_tensor_load_format_custom`。
- `TestUpdateWeightsFromTensor.test_update_weights_from_tensor_load_format_flattened_bucket`: Test updating weights using flattened_bucket format / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_from_tensor_load_format_flattened_bucket`。
- `TestServerUpdateWeightsFromTensorNonBlocking.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestServerUpdateWeightsFromTensorNonBlocking.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestServerUpdateWeightsFromTensorNonBlocking.run_decode`: This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。
- `TestServerUpdateWeightsFromTensorNonBlocking.get_model_info`: This helper encapsulates `get_model_info` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `get_model_info`，以便周围测试复用准备、执行或校验逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `gc`, `json`, `random`, `time`, `unittest`, `concurrent.futures`
- **Third-party modules / 第三方模块**: `requests`, `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang`, `sglang.srt.utils`, `sglang.srt.weight_sync.tensor_bucket`, `sglang.test.test_utils`

- **Total lines / 总行数**: 294
