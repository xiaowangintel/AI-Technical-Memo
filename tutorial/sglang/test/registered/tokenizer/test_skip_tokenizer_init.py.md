# test_skip_tokenizer_init.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/tokenizer/test_skip_tokenizer_init.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates skip tokenizer init behavior in SGLang's tokenizer area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 tokenizer 领域中与 skip tokenizer init 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: supporting statements / 辅助语句
```python
"""
python3 -m unittest test_skip_tokenizer_init.TestSkipTokenizerInit.test_parallel_sample
python3 -m unittest test_skip_tokenizer_init.TestSkipTokenizerInit.run_decode_stream
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 6-24: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest

import requests
from transformers import AutoProcessor, AutoTokenizer

from sglang.lang.chat_template import get_chat_template_by_model_path
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_IMAGE_URL,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_SMALL_VLM_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    download_image_with_retry,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `requests`, `transformers`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `requests`, `transformers`。

### Lines 26-27: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=79, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=117, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 30-30: class TestSkipTokenizerInit declaration / 类 TestSkipTokenizerInit 声明
```python
class TestSkipTokenizerInit(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 31-44: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=["--skip-tokenizer-init", "--incremental-streaming-output"],
        )
        cls.eos_token_id = [119690]
        cls.tokenizer = AutoTokenizer.from_pretrained(
            DEFAULT_SMALL_MODEL_NAME_FOR_TEST, use_fast=False
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 46-48: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 50-110: helper routine run decode / 辅助流程 run decode
```python
    def run_decode(
        self,
        prompt_text="The capital of France is",
        max_new_tokens=32,
        return_logprob=False,
        top_logprobs_num=0,
        n=1,
    ):
        input_ids = self.get_input_ids(prompt_text)

        request = self.get_request_json(
            input_ids=input_ids,
            return_logprob=return_logprob,
            top_logprobs_num=top_logprobs_num,
            max_new_tokens=max_new_tokens,
            stream=False,
            n=n,
        )
        response = requests.post(
            self.base_url + "/generate",
            json=request,
        )
        ret = response.json()
        print(json.dumps(ret, indent=2))

        def assert_one_item(item):
            if item["meta_info"]["finish_reason"]["type"] == "stop":
                self.assertEqual(
                    item["meta_info"]["finish_reason"]["matched"],
                    self.tokenizer.eos_token_id,
                )
            elif item["meta_info"]["finish_reason"]["type"] == "length":
                self.assertEqual(
                    len(item["output_ids"]), item["meta_info"]["completion_tokens"]
                )
                self.assertEqual(len(item["output_ids"]), max_new_tokens)
                self.assertEqual(item["meta_info"]["prompt_tokens"], len(input_ids))

                if return_logprob:
                    num_input_logprobs = len(input_ids) - request["logprob_start_len"]
                    if num_input_logprobs > len(input_ids):
                        num_input_logprobs -= len(input_ids)
                    self.assertEqual(
                        len(item["meta_info"]["input_token_logprobs"]),
                        num_input_logprobs,
                        f'{len(item["meta_info"]["input_token_logprobs"])} mismatch with {len(input_ids)}',
                    )
                    self.assertEqual(
                        len(item["meta_info"]["output_token_logprobs"]),
                        max_new_tokens,
                    )

        # Determine whether to assert a single item or multiple items based on n
        if n == 1:
            assert_one_item(ret)
        else:
            self.assertEqual(len(ret), n)
            for i in range(n):
                assert_one_item(ret[i])

        print("=" * 100)
```
**EN:** This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 112-158: helper routine run decode stream / 辅助流程 run decode stream
```python
    def run_decode_stream(self, return_logprob=False, top_logprobs_num=0, n=1):
        max_new_tokens = 32
        input_ids = self.get_input_ids("The capital of France is")
        requests.post(self.base_url + "/flush_cache")
        response = requests.post(
            self.base_url + "/generate",
            json=self.get_request_json(
                input_ids=input_ids,
                max_new_tokens=max_new_tokens,
                return_logprob=return_logprob,
                top_logprobs_num=top_logprobs_num,
                stream=False,
                n=n,
            ),
        )
        ret = response.json()
        print(json.dumps(ret))
        output_ids = ret["output_ids"]
        print("output from non-streaming request:")
        print(output_ids)
        print(self.tokenizer.decode(output_ids, skip_special_tokens=True))

        requests.post(self.base_url + "/flush_cache")
        response_stream = requests.post(
            self.base_url + "/generate",
            json=self.get_request_json(
                input_ids=input_ids,
                return_logprob=return_logprob,
                top_logprobs_num=top_logprobs_num,
                stream=True,
                n=n,
            ),
        )

        response_stream_json = []
        for line in response_stream.iter_lines():
            print(line)
            if line.startswith(b"data: ") and line[6:] != b"[DONE]":
                response_stream_json.append(json.loads(line[6:]))
        out_stream_ids = []
        for x in response_stream_json:
            out_stream_ids += x["output_ids"]
        print("output from streaming request:")
        print(out_stream_ids)
        print(self.tokenizer.decode(out_stream_ids, skip_special_tokens=True))

        assert output_ids == out_stream_ids
```
**EN:** This helper encapsulates `run_decode_stream` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_decode_stream`，以便周围测试复用准备、执行或校验逻辑。

### Lines 160-161: test case simple decode / 测试用例 simple decode
```python
    def test_simple_decode(self):
        self.run_decode()
```
**EN:** This test exercises `test_simple_decode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_simple_decode`。

### Lines 163-164: test case parallel sample / 测试用例 parallel sample
```python
    def test_parallel_sample(self):
        self.run_decode(n=3)
```
**EN:** This test exercises `test_parallel_sample` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_sample`。

### Lines 166-168: test case logprob / 测试用例 logprob
```python
    def test_logprob(self):
        for top_logprobs_num in [0, 3]:
            self.run_decode(return_logprob=True, top_logprobs_num=top_logprobs_num)
```
**EN:** This test exercises `test_logprob` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logprob`。

### Lines 170-171: test case eos behavior / 测试用例 eos behavior
```python
    def test_eos_behavior(self):
        self.run_decode(max_new_tokens=256)
```
**EN:** This test exercises `test_eos_behavior` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_eos_behavior`。

### Lines 173-174: test case simple decode stream / 测试用例 simple decode stream
```python
    def test_simple_decode_stream(self):
        self.run_decode_stream()
```
**EN:** This test exercises `test_simple_decode_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_simple_decode_stream`。

### Lines 176-180: helper routine get input ids / 辅助流程 get input ids
```python
    def get_input_ids(self, prompt_text) -> list[int]:
        input_ids = self.tokenizer(prompt_text, return_tensors="pt")["input_ids"][
            0
        ].tolist()
        return input_ids
```
**EN:** This helper encapsulates `get_input_ids` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_input_ids`，以便周围测试复用准备、执行或校验逻辑。

### Lines 182-203: helper routine get request json / 辅助流程 get request json
```python
    def get_request_json(
        self,
        input_ids,
        max_new_tokens=32,
        return_logprob=False,
        top_logprobs_num=0,
        stream=False,
        n=1,
    ):
        return {
            "input_ids": input_ids,
            "sampling_params": {
                "temperature": 0 if n == 1 else 0.5,
                "max_new_tokens": max_new_tokens,
                "n": n,
                "stop_token_ids": self.eos_token_id,
            },
            "stream": stream,
            "return_logprob": return_logprob,
            "top_logprobs_num": top_logprobs_num,
            "logprob_start_len": 0,
        }
```
**EN:** This helper encapsulates `get_request_json` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_request_json`，以便周围测试复用准备、执行或校验逻辑。

### Lines 206-206: class TestSkipTokenizerInitVLM declaration / 类 TestSkipTokenizerInitVLM 声明
```python
class TestSkipTokenizerInitVLM(TestSkipTokenizerInit):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestSkipTokenizerInit`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestSkipTokenizerInit`。

### Lines 207-221: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.image_url = DEFAULT_IMAGE_URL
        cls.image = download_image_with_retry(cls.image_url)
        cls.model = DEFAULT_SMALL_VLM_MODEL_NAME_FOR_TEST
        cls.tokenizer = AutoTokenizer.from_pretrained(cls.model, use_fast=False)
        cls.processor = AutoProcessor.from_pretrained(cls.model, trust_remote_code=True)
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=["--skip-tokenizer-init"],
        )
        cls.eos_token_id = [cls.tokenizer.eos_token_id]
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 223-232: helper routine get input ids / 辅助流程 get input ids
```python
    def get_input_ids(self, _prompt_text) -> list[int]:
        chat_template = get_chat_template_by_model_path(self.model)
        text = f"{chat_template.image_token}What is in this picture?"
        inputs = self.processor(
            text=[text],
            images=[self.image],
            return_tensors="pt",
        )

        return inputs.input_ids[0].tolist()
```
**EN:** This helper encapsulates `get_input_ids` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_input_ids`，以便周围测试复用准备、执行或校验逻辑。

### Lines 234-240: helper routine get request json / 辅助流程 get request json
```python
    def get_request_json(self, *args, **kwargs):
        ret = super().get_request_json(*args, **kwargs)
        ret["image_data"] = [self.image_url]
        ret["logprob_start_len"] = (
            -1
        )  # Do not try to calculate logprobs of image embeddings.
        return ret
```
**EN:** This helper encapsulates `get_request_json` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_request_json`，以便周围测试复用准备、执行或校验逻辑。

### Lines 242-244: test case simple decode stream / 测试用例 simple decode stream
```python
    def test_simple_decode_stream(self):
        # TODO mick
        pass
```
**EN:** This test exercises `test_simple_decode_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_simple_decode_stream`。

### Lines 247-248: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestSkipTokenizerInit`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSkipTokenizerInitVLM`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSkipTokenizerInit.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestSkipTokenizerInit.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestSkipTokenizerInit.run_decode`: This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。
- `TestSkipTokenizerInit.run_decode_stream`: This helper encapsulates `run_decode_stream` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_decode_stream`，以便周围测试复用准备、执行或校验逻辑。
- `TestSkipTokenizerInit.test_simple_decode`: This test exercises `test_simple_decode` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_simple_decode`。
- `TestSkipTokenizerInit.test_parallel_sample`: This test exercises `test_parallel_sample` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_sample`。
- `TestSkipTokenizerInit.test_logprob`: This test exercises `test_logprob` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logprob`。
- `TestSkipTokenizerInit.test_eos_behavior`: This test exercises `test_eos_behavior` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_eos_behavior`。
- `TestSkipTokenizerInit.test_simple_decode_stream`: This test exercises `test_simple_decode_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_simple_decode_stream`。
- `TestSkipTokenizerInit.get_input_ids`: This helper encapsulates `get_input_ids` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `get_input_ids`，以便周围测试复用准备、执行或校验逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`
- **Third-party modules / 第三方模块**: `requests`, `transformers`
- **Internal modules / 内部模块**: `sglang.lang.chat_template`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 248
