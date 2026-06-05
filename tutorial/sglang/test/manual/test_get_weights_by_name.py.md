# test_get_weights_by_name.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_get_weights_by_name.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `get weights by name` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual` 中的 `get weights by name` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Imports and shared helpers / 导入与共享辅助项
```python
import gc
import unittest

import numpy as np
import requests
from transformers import AutoModelForCausalLM

import sglang as sgl
from sglang.srt.utils import get_device
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    empty_gpu_cache,
    get_gpu_count,
    is_in_ci,
    popen_launch_server,
)
from sglang.utils import terminate_process
```
**EN:** This range imports `gc`, `unittest`, `numpy` and `requests`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. Process management appears here to launch, monitor, or stop runtime components.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。

### Lines 24-29: Helper routines around _process_return / 辅助例程
```python
def _process_return(ret):
    if isinstance(ret, list) and len(ret) == 2:
        print(f"running assert_allclose on data parallel")
        np.testing.assert_allclose(ret[0], ret[1])
        return np.array(ret[0])
    return np.array(ret)
```
**EN:** This range implements helper routine(s) `_process_return` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `assert_allclose` and `array`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 32-33: Class definition for TestGetWeightsByName / 类定义
```python
class TestGetWeightsByName(CustomTestCase):
```
**EN:** This range declares `TestGetWeightsByName`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 34-37: Helper routines around init_hf_model / 辅助例程
```python
    def init_hf_model(self, model_name, tie_word_embeddings):
        self.hf_model = AutoModelForCausalLM.from_pretrained(
            model_name, torch_dtype="bfloat16", tie_word_embeddings=tie_word_embeddings
        ).to(get_device())
```
**EN:** This range implements helper routine(s) `init_hf_model` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `from_pretrained`, `to` and `get_device`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 39-56: Helper routines around init_backend / 辅助例程
```python
    def init_backend(self, backend, dp, tp, model_name):
        self.backend = backend
        self.dp = dp
        self.tp = tp
        if backend == "Engine":
            self.engine = sgl.Engine(
                model_path=model_name,
                random_seed=42,
                tp_size=tp,
                dp_size=dp,
            )
        else:
            self.process = popen_launch_server(
                model_name,
                DEFAULT_URL_FOR_TEST,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=(
                    "--tp-size",
```
**EN:** This range implements helper routine(s) `init_backend` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `Engine` and `popen_launch_server`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 57-61: Scenario logic / 场景逻辑
```python
                    str(tp),
                    "--dp-size",
                    str(dp),
                ),
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 63-70: Helper routines around clean_up / 辅助例程
```python
    def clean_up(self):
        del self.hf_model
        gc.collect()
        empty_gpu_cache()
        if self.backend == "Engine":
            self.engine.shutdown()
        else:
            terminate_process(self.process)
```
**EN:** This range implements helper routine(s) `clean_up` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `collect`, `empty_gpu_cache`, `shutdown` and `terminate_process`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 72-81: Helper routines around assert_tie_word_embeddings / 辅助例程
```python
    def assert_tie_word_embeddings(self, truncate_size):
        print("assert_tie_word_embeddings")
        if self.backend == "Engine":
            backend_ret = _process_return(
                self.engine.get_weights_by_name("lm_head.weight", truncate_size)
            )
        else:
            backend_ret = _process_return(
                requests.get(
                    f"{DEFAULT_URL_FOR_TEST}/get_weights_by_name",
```
**EN:** This range implements helper routine(s) `assert_tie_word_embeddings` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `_process_return`, `get_weights_by_name` and `get`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 82-96: Assertions and result checks / 断言与结果检查
```python
                    json={"name": "lm_head.weight", "truncate_size": truncate_size},
                ).json()
            )
        print("assert_tie_word_embeddings of hf and backend")
        assert np.allclose(
            self.hf_model.get_parameter("model.embed_tokens.weight")
            .cpu()
            .detach()
            .float()
            .numpy()[:truncate_size],
            backend_ret,
        )
        assert np.allclose(
            self.hf_model.get_parameter("lm_head.weight")
            .cpu()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `json`, `allclose`, `get_parameter` and `cpu`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 97-105: Scenario logic / 场景逻辑
```python
            .detach()
            .float()
            .numpy()[:truncate_size],
            self.hf_model.get_parameter("model.embed_tokens.weight")
            .cpu()
            .detach()
            .float()
            .numpy()[:truncate_size],
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `detach`, `numpy`, `get_parameter` and `cpu`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 107-125: Helper routines around assert_weights_all_close / 辅助例程
```python
    def assert_weights_all_close(self, param_name, truncate_size):
        print(
            f"param_name: {param_name}, backend: {self.backend}, dp: {self.dp}, tp: {self.tp}"
        )
        param = self.hf_model.get_parameter(param_name)[:truncate_size]
        param_np = param.cpu().detach().float().numpy()

        if self.backend == "Engine":
            engine_ret = self.engine.get_weights_by_name(param_name, truncate_size)
            engine_ret = _process_return(engine_ret)
            np.testing.assert_allclose(engine_ret, param_np, rtol=1e-5, atol=1e-5)

        if self.backend == "Runtime":
            runtime_ret = requests.get(
                f"{DEFAULT_URL_FOR_TEST}/get_weights_by_name",
                json={"name": param_name, "truncate_size": truncate_size},
            ).json()
            runtime_ret = _process_return(runtime_ret)
            np.testing.assert_allclose(runtime_ret, param_np, rtol=1e-5, atol=1e-5)
```
**EN:** This range implements helper routine(s) `assert_weights_all_close` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get_parameter`, `cpu`, `detach` and `numpy`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 127-131: Test routines around test_get_weights_by_name / 测试例程
```python
    def test_get_weights_by_name(self):
        if is_in_ci():
            test_suits = [
                ("Engine", 1, 1, DEFAULT_SMALL_MODEL_NAME_FOR_TEST),
            ]
```
**EN:** This range defines concrete test routine(s) `test_get_weights_by_name`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `is_in_ci`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 132-151: Scenario logic / 场景逻辑
```python
        else:
            test_suits = [
                ("Runtime", 1, 1, DEFAULT_SMALL_MODEL_NAME_FOR_TEST),
                ("Engine", 1, 1, DEFAULT_MODEL_NAME_FOR_TEST),
            ]
            if get_gpu_count() >= 2:
                test_suits.append(("Engine", 1, 2, DEFAULT_SMALL_MODEL_NAME_FOR_TEST))
                test_suits.append(("Runtime", 2, 1, DEFAULT_MODEL_NAME_FOR_TEST))

            if get_gpu_count() >= 4:
                test_suits.extend(
                    [
                        ("Engine", 2, 2, DEFAULT_SMALL_MODEL_NAME_FOR_TEST),
                        ("Runtime", 2, 2, DEFAULT_MODEL_NAME_FOR_TEST),
                    ]
                )

        parameters = [
            "model.embed_tokens.weight",
            "model.layers.0.input_layernorm.weight",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `get_gpu_count`, `append` and `extend`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 152-156: Scenario logic / 场景逻辑
```python
            "model.layers.1.self_attn.q_proj.weight",
            "model.layers.2.self_attn.k_proj.weight",
            "model.layers.3.self_attn.v_proj.weight",
            "model.layers.4.self_attn.o_proj.weight",
            "model.layers.5.mlp.gate_proj.weight",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 157-176: Assertions and result checks / 断言与结果检查
```python
            "model.layers.6.mlp.up_proj.weight",
            "model.layers.7.mlp.down_proj.weight",
            "model.layers.8.post_attention_layernorm.weight",
            "model.norm.weight",
            "lm_head.weight",
        ]

        truncate_size = 100

        for test_suit in test_suits:
            if test_suit[-1] == DEFAULT_MODEL_NAME_FOR_TEST:
                tie_word_embeddings = False
            else:
                tie_word_embeddings = True

            self.init_hf_model(test_suit[-1], tie_word_embeddings)
            self.init_backend(*test_suit)

            for param_name in parameters:
                self.assert_weights_all_close(param_name, truncate_size)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `init_hf_model`, `init_backend` and `assert_weights_all_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 177-181: Assertions and result checks / 断言与结果检查
```python

            if tie_word_embeddings:
                self.assert_tie_word_embeddings(truncate_size)

            self.clean_up()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert_tie_word_embeddings` and `clean_up`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 182-185: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Token-level inspection / Token 级分析
- Prefix caching / 前缀缓存

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `gc`, `unittest`
- **Third-party / 第三方库**: `numpy`, `requests`, `transformers`
- **Project Modules / 项目模块**: `sglang`, `sglang.srt.utils`, `sglang.test.test_utils`, `sglang.utils`
