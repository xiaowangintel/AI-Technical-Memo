# test_gptqmodel_dynamic.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/quant/test_gptqmodel_dynamic.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates gptqmodel dynamic behavior in SGLang's quant area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 quant 领域中与 gptqmodel dynamic 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: module imports and dependencies / 模块导入与依赖
```python
import time
import unittest

import requests
import torch

from sglang.srt.server_args import set_global_server_args_for_scheduler
from sglang.srt.utils import get_device, kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `time`, `unittest`, `requests`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `time`, `unittest`, `requests`, `torch`。

### Lines 17-17: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=100, stage="extra-a", runner_config="1-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 20-93: helper routine check quant method / 辅助流程 check quant method
```python
def check_quant_method(model_path: str, use_marlin_kernel: bool):
    from sglang.srt.configs.device_config import DeviceConfig
    from sglang.srt.configs.load_config import LoadConfig
    from sglang.srt.configs.model_config import ModelConfig
    from sglang.srt.distributed import (
        init_distributed_environment,
        initialize_model_parallel,
    )
    from sglang.srt.distributed.parallel_state import monkey_patch_vllm_parallel_state
    from sglang.srt.layers.quantization.utils import get_dynamic_override
    from sglang.srt.model_loader import get_model
    from sglang.srt.server_args import ServerArgs

    try:
        init_distributed_environment(
            backend="nccl",
            world_size=1,
            rank=0,
            local_rank=0,
            distributed_init_method="tcp://127.0.0.1:2646",
        )
        initialize_model_parallel(tensor_model_parallel_size=1)
        monkey_patch_vllm_parallel_state()
    except AssertionError:
        # ignore this error: tensor model parallel group is already initialized
        pass

    server_args = ServerArgs(model_path=model_path, dtype=torch.float16)
    set_global_server_args_for_scheduler(server_args)
    model_config = ModelConfig.from_server_args(server_args)

    load_config = LoadConfig()
    device_config = DeviceConfig(get_device())
    model = get_model(
        model_config=model_config, load_config=load_config, device_config=device_config
    )

    from sglang.srt.layers.quantization.gptq import (
        GPTQLinearMethod,
        GPTQMarlinLinearMethod,
    )
    from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod

    linear_method_cls = (
        GPTQMarlinLinearMethod if use_marlin_kernel else (GPTQLinearMethod)
    )

    for name, submodule in model.named_modules():
        if name == "lm_head":
            assert isinstance(submodule.quant_method, linear_method_cls)
        elif name == "model.layers.0.self_attn.qkv_proj":
            # The first layer is quantized using bits=4, group_size=128
            # desc_act=True
            assert isinstance(submodule.quant_method, linear_method_cls)
            config = submodule.quant_method.quant_config
            assert config.weight_bits == 4
            assert config.group_size == 128
            assert config.desc_act
        elif name == "model.layers.1.self_attn.qkv_proj":
            # The second layer is quantized using bits=8, group_size=32
            # desc_act=False
            assert isinstance(submodule.quant_method, linear_method_cls)
            config = submodule.quant_method.quant_config
            assert get_dynamic_override(config, layer_name=name, key="bits") == 8
            assert get_dynamic_override(config, layer_name=name, key="group_size") == 32
            assert not get_dynamic_override(config, layer_name=name, key="desc_act")
        elif (
            name == "model.layers.2.self_attn.qkv_proj"
            or name == "model.layers.2.mlp.gate_up_proj"
        ):
            # All other layers (layer index >= 2) are not quantized
            assert isinstance(submodule.quant_method, UnquantizedLinearMethod)

    del model
```
**EN:** This helper encapsulates `check_quant_method` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `check_quant_method`，以便周围测试复用准备、执行或校验逻辑。

### Lines 94-98: supporting source context / 辅助源码上下文
```python


# GPTQ with Dynamic Per/Module Quantization Control
# Leverages GPTQModel (pypi) to produce the `dynamic` models
# Test GPTQ fallback kernel that is not Marlin
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 99-99: class TestGPTQModelDynamic declaration / 类 TestGPTQModelDynamic 声明
```python
class TestGPTQModelDynamic(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 100-102: class-level constants and configuration for `TestGPTQModelDynamic` / 类级常量与配置
```python
    MODEL_PATH = (
        "ModelCloud/Qwen1.5-1.8B-Chat-GPTQ-4bits-dynamic-cfg-with-lm_head-symFalse"
    )
```
**EN:** This block defines shared names such as `MODEL_PATH`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `MODEL_PATH` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 104-113: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = cls.MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=["--dtype", "float16"],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 115-117: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 119-130: helper routine run decode / 辅助流程 run decode
```python
    def run_decode(self, max_new_tokens):
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "max_new_tokens": max_new_tokens,
                    "temperature": 0.001,
                },
            },
        )
        return response.json()
```
**EN:** This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 132-145: test case throughput / 测试用例 throughput
```python
    def test_throughput(self):
        max_tokens = 256

        tic = time.perf_counter()
        result = self.run_decode(max_tokens)
        tok = time.perf_counter()

        print(f"result = `{result}`")

        self.assertIn("paris", result["text"].lower())

        throughput = max_tokens / (tok - tic)
        print(f"Throughput: {throughput} tokens/s")
        self.assertGreaterEqual(throughput, 140)
```
**EN:** This test exercises `test_throughput` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_throughput`。

### Lines 147-148: test case gptq module / 测试用例 gptq module
```python
    def test_gptq_module(self):
        check_quant_method(self.MODEL_PATH, use_marlin_kernel=False)
```
**EN:** This test exercises `test_gptq_module` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gptq_module`。

### Lines 149-153: supporting source context / 辅助源码上下文
```python


# GPTQ with Dynamic Per/Module Quantization Control
# Leverages GPTQModel (pypi) to produce the `dynamic` models
# Test Marlin kernel
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 154-154: class TestGPTQModelDynamicWithMarlin declaration / 类 TestGPTQModelDynamicWithMarlin 声明
```python
class TestGPTQModelDynamicWithMarlin(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 155-157: class-level constants and configuration for `TestGPTQModelDynamicWithMarlin` / 类级常量与配置
```python
    MODEL_PATH = (
        "ModelCloud/Qwen1.5-1.8B-Chat-GPTQ-4bits-dynamic-cfg-with-lm_head-symTrue"
    )
```
**EN:** This block defines shared names such as `MODEL_PATH`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `MODEL_PATH` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 159-168: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = cls.MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=["--dtype", "bfloat16"],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 170-172: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 174-185: helper routine run decode / 辅助流程 run decode
```python
    def run_decode(self, max_new_tokens):
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "max_new_tokens": max_new_tokens,
                    "temperature": 0.001,
                },
            },
        )
        return response.json()
```
**EN:** This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 187-200: test case throughput / 测试用例 throughput
```python
    def test_throughput(self):
        max_tokens = 256

        tic = time.perf_counter()
        result = self.run_decode(max_tokens)
        tok = time.perf_counter()

        print(f"result = `{result}`")

        assert "paris" in result["text"].lower()

        throughput = max_tokens / (tok - tic)
        print(f"Throughput: {throughput} tokens/s")
        assert throughput >= 140
```
**EN:** This test exercises `test_throughput` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_throughput`。

### Lines 202-203: test case gptq marlin module / 测试用例 gptq marlin module
```python
    def test_gptq_marlin_module(self):
        check_quant_method(self.MODEL_PATH, use_marlin_kernel=True)
```
**EN:** This test exercises `test_gptq_marlin_module` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gptq_marlin_module`。

### Lines 206-207: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `check_quant_method`: This helper encapsulates `check_quant_method` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `check_quant_method`，以便周围测试复用准备、执行或校验逻辑。
- `TestGPTQModelDynamic`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGPTQModelDynamicWithMarlin`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGPTQModelDynamic.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestGPTQModelDynamic.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestGPTQModelDynamic.run_decode`: This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。
- `TestGPTQModelDynamic.test_throughput`: This test exercises `test_throughput` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_throughput`。
- `TestGPTQModelDynamic.test_gptq_module`: This test exercises `test_gptq_module` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gptq_module`。
- `TestGPTQModelDynamicWithMarlin.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestGPTQModelDynamicWithMarlin.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestGPTQModelDynamicWithMarlin.run_decode`: This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。
- `TestGPTQModelDynamicWithMarlin.test_throughput`: This test exercises `test_throughput` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_throughput`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `time`, `unittest`
- **Third-party modules / 第三方模块**: `requests`, `torch`
- **Internal modules / 内部模块**: `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 207
