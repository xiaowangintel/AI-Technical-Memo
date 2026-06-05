# test_update_weights_from_disk_blackwell.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/rl/test_update_weights_from_disk_blackwell.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates update weights from disk blackwell behavior in SGLang's rl area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 rl 领域中与 update weights from disk blackwell 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 3-3: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=320, stage="extra-b", runner_config="4-gpu-b200")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 5-15: module imports and dependencies / 模块导入与依赖
```python
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `requests`, `sglang.srt.utils`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `requests`, `sglang.srt.utils`, `sglang.test.test_utils`。

### Lines 18-18: class UpdateWeightsFromDiskBase declaration / 类 UpdateWeightsFromDiskBase 声明
```python
class UpdateWeightsFromDiskBase:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 19-32: class-level constants and configuration for `UpdateWeightsFromDiskBase` / 类级常量与配置
```python
    model = None
    base_url = DEFAULT_URL_FOR_TEST
    request_timeout = 120
    update_timeout = 240
    launch_env = None
    decode_payload = {
        "text": "The capital of France is",
        "sampling_params": {"temperature": 0, "max_new_tokens": 16},
    }
    backend_test_suites = ()
    update_test_suites = (
        {"flush_cache": True, "abort_all_requests": False},
        {"flush_cache": False, "abort_all_requests": False},
    )
```
**EN:** This block defines shared names such as `model`, `base_url`, `request_timeout`, `update_timeout`, `launch_env`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `base_url`, `request_timeout`, `update_timeout`, `launch_env` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 34-42: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        if cls.model is None:
            raise NotImplementedError("Subclass must set 'model' attribute")
        if not cls.backend_test_suites:
            raise NotImplementedError(
                "Subclass must set non-empty 'backend_test_suites'"
            )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 44-55: method launch server / 方法 launch server
```python
    def _launch_server(self, backend_test_suite):
        launch_kwargs = {}
        if self.launch_env is not None:
            launch_kwargs["env"] = self.launch_env
        other_args = backend_test_suite.get("other_args")
        return popen_launch_server(
            self.model,
            self.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
            **launch_kwargs,
        )
```
**EN:** This block implements `_launch_server` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_launch_server`，承担模块行为中的一个聚焦逻辑片段。

### Lines 57-63: method get json / 方法 get json
```python
    def _get_json(self, endpoint, timeout=None):
        response = requests.get(
            f"{self.base_url}{endpoint}",
            timeout=timeout or self.request_timeout,
        )
        response.raise_for_status()
        return response.json()
```
**EN:** This block implements `_get_json` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_get_json`，承担模块行为中的一个聚焦逻辑片段。

### Lines 65-72: method post json / 方法 post json
```python
    def _post_json(self, endpoint, payload, timeout=None):
        response = requests.post(
            f"{self.base_url}{endpoint}",
            json=payload,
            timeout=timeout or self.request_timeout,
        )
        response.raise_for_status()
        return response.json()
```
**EN:** This block implements `_post_json` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_post_json`，承担模块行为中的一个聚焦逻辑片段。

### Lines 74-75: method run decode / 方法 run decode
```python
    def _run_decode(self):
        return self._post_json("/generate", self.decode_payload)["text"]
```
**EN:** This block implements `_run_decode` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_decode`，承担模块行为中的一个聚焦逻辑片段。

### Lines 77-78: method assert non empty decode / 方法 assert non empty decode
```python
    def _assert_non_empty_decode(self):
        self.assertTrue(len(self._run_decode()) > 0)
```
**EN:** This block implements `_assert_non_empty_decode` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_assert_non_empty_decode`，承担模块行为中的一个聚焦逻辑片段。

### Lines 80-96: method get decode logprob signature / 方法 get decode logprob signature
```python
    def _get_decode_logprob_signature(self):
        ret = self._post_json(
            "/generate",
            {**self.decode_payload, "return_logprob": True},
        )
        output_token_logprobs = ret["meta_info"].get("output_token_logprobs")
        self.assertIsNotNone(output_token_logprobs)
        self.assertGreater(
            len(output_token_logprobs),
            0,
            "Expected non-empty output_token_logprobs.",
        )
        return {
            "text": ret["text"],
            "token_ids": [int(x[1]) for x in output_token_logprobs],
            "logprobs": [float(x[0]) for x in output_token_logprobs],
        }
```
**EN:** This block implements `_get_decode_logprob_signature` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_get_decode_logprob_signature`，承担模块行为中的一个聚焦逻辑片段。

### Lines 98-107: method assert decode logprob unchanged / 方法 assert decode logprob unchanged
```python
    def _assert_decode_logprob_unchanged(self, before, after, atol=1e-4):
        self.assertEqual(after["text"], before["text"])
        self.assertEqual(after["token_ids"], before["token_ids"])
        self.assertEqual(len(after["logprobs"]), len(before["logprobs"]))
        for idx, (a, b) in enumerate(zip(after["logprobs"], before["logprobs"])):
            self.assertLessEqual(
                abs(a - b),
                atol,
                f"Output token logprob changed at idx={idx}: before={b}, after={a}",
            )
```
**EN:** This block implements `_assert_decode_logprob_unchanged` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_assert_decode_logprob_unchanged`，承担模块行为中的一个聚焦逻辑片段。

### Lines 109-110: method get model info / 方法 get model info
```python
    def _get_model_info(self):
        return self._get_json("/get_model_info")["model_path"]
```
**EN:** This block implements `_get_model_info` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_get_model_info`，承担模块行为中的一个聚焦逻辑片段。

### Lines 112-126: method run update weights / 方法 run update weights
```python
    def _run_update_weights(
        self,
        model_path,
        flush_cache=True,
        abort_all_requests=False,
    ):
        return self._post_json(
            "/update_weights_from_disk",
            {
                "model_path": model_path,
                "flush_cache": flush_cache,
                "abort_all_requests": abort_all_requests,
            },
            timeout=self.update_timeout,
        )
```
**EN:** This block implements `_run_update_weights` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_update_weights`，承担模块行为中的一个聚焦逻辑片段。

### Lines 128-156: test case parameterized update weights from disk / 测试用例 parameterized update weights from disk
```python
    def test_parameterized_update_weights_from_disk(self):
        for backend_test_suite in self.backend_test_suites:
            case_name = backend_test_suite.get("name", "default")
            with self.subTest(model=self.model, case_name=case_name):
                process = self._launch_server(backend_test_suite)
                try:
                    origin_model_path = self._get_model_info()
                    self.assertEqual(origin_model_path, self.model)
                    self._assert_non_empty_decode()
                    baseline_sig = self._get_decode_logprob_signature()

                    for update_test_suite in self.update_test_suites:
                        with self.subTest(case_name=case_name, **update_test_suite):
                            ret = self._run_update_weights(
                                self.model,
                                flush_cache=update_test_suite["flush_cache"],
                                abort_all_requests=update_test_suite[
                                    "abort_all_requests"
                                ],
                            )
                            self.assertTrue(ret.get("success"), f"{ret=}")
                            self.assertEqual(self._get_model_info(), self.model)
                            self._assert_non_empty_decode()
                            updated_sig = self._get_decode_logprob_signature()
                            self._assert_decode_logprob_unchanged(
                                baseline_sig, updated_sig
                            )
                finally:
                    kill_process_tree(process.pid)
```
**EN:** This test exercises `test_parameterized_update_weights_from_disk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parameterized_update_weights_from_disk`。

### Lines 159-159: class TestServerUpdateWeightsFromDiskMXFP8 declaration / 类 TestServerUpdateWeightsFromDiskMXFP8 声明
```python
class TestServerUpdateWeightsFromDiskMXFP8(UpdateWeightsFromDiskBase, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `UpdateWeightsFromDiskBase`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `UpdateWeightsFromDiskBase`, `CustomTestCase`。

### Lines 160-175: class-level constants and configuration for `TestServerUpdateWeightsFromDiskMXFP8` / 类级常量与配置
```python
    model = "zianglih/Qwen3-30B-A3B-Instruct-2507-MXFP8-last-8-BF16"
    backend_test_suites = (
        {
            "name": "flashinfer_trtllm_routed_mxfp8",
            "other_args": (
                "--base-gpu-id",
                "0",
                "--tp-size",
                "4",
                "--fp8-gemm-backend",
                "flashinfer_trtllm",
                "--moe-runner-backend",
                "flashinfer_trtllm_routed",
            ),
        },
    )
```
**EN:** This block defines shared names such as `model`, `backend_test_suites`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `backend_test_suites` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 178-178: class TestServerUpdateWeightsFromDiskNVFP4 declaration / 类 TestServerUpdateWeightsFromDiskNVFP4 声明
```python
class TestServerUpdateWeightsFromDiskNVFP4(UpdateWeightsFromDiskBase, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `UpdateWeightsFromDiskBase`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `UpdateWeightsFromDiskBase`, `CustomTestCase`。

### Lines 179-194: class-level constants and configuration for `TestServerUpdateWeightsFromDiskNVFP4` / 类级常量与配置
```python
    model = "nvidia/Qwen3-30B-A3B-NVFP4"
    backend_test_suites = (
        {
            "name": "flashinfer_trtllm_nvfp4",
            "other_args": (
                "--base-gpu-id",
                "0",
                "--tp-size",
                "4",
                "--fp4-gemm-backend",
                "flashinfer_trtllm",
                "--moe-runner-backend",
                "flashinfer_trtllm_routed",
            ),
        },
    )
```
**EN:** This block defines shared names such as `model`, `backend_test_suites`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `backend_test_suites` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 197-198: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `UpdateWeightsFromDiskBase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestServerUpdateWeightsFromDiskMXFP8`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestServerUpdateWeightsFromDiskNVFP4`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `UpdateWeightsFromDiskBase.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `UpdateWeightsFromDiskBase._launch_server`: This block implements `_launch_server` and captures one focused piece of the module's behavior. / 该代码块实现 `_launch_server`，承担模块行为中的一个聚焦逻辑片段。
- `UpdateWeightsFromDiskBase._get_json`: This block implements `_get_json` and captures one focused piece of the module's behavior. / 该代码块实现 `_get_json`，承担模块行为中的一个聚焦逻辑片段。
- `UpdateWeightsFromDiskBase._post_json`: This block implements `_post_json` and captures one focused piece of the module's behavior. / 该代码块实现 `_post_json`，承担模块行为中的一个聚焦逻辑片段。
- `UpdateWeightsFromDiskBase._run_decode`: This block implements `_run_decode` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_decode`，承担模块行为中的一个聚焦逻辑片段。
- `UpdateWeightsFromDiskBase._assert_non_empty_decode`: This block implements `_assert_non_empty_decode` and captures one focused piece of the module's behavior. / 该代码块实现 `_assert_non_empty_decode`，承担模块行为中的一个聚焦逻辑片段。
- `UpdateWeightsFromDiskBase._get_decode_logprob_signature`: This block implements `_get_decode_logprob_signature` and captures one focused piece of the module's behavior. / 该代码块实现 `_get_decode_logprob_signature`，承担模块行为中的一个聚焦逻辑片段。
- `UpdateWeightsFromDiskBase._assert_decode_logprob_unchanged`: This block implements `_assert_decode_logprob_unchanged` and captures one focused piece of the module's behavior. / 该代码块实现 `_assert_decode_logprob_unchanged`，承担模块行为中的一个聚焦逻辑片段。
- `UpdateWeightsFromDiskBase._get_model_info`: This block implements `_get_model_info` and captures one focused piece of the module's behavior. / 该代码块实现 `_get_model_info`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.utils`, `sglang.test.test_utils`

- **Total lines / 总行数**: 198
