# runtime_endpoint.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/lang/backend/runtime_endpoint.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements an SGLang language backend adapter, translating high-level requests into the protocol of an external model provider or runtime endpoint. / 该文件实现了 SGLang 语言层的后端适配器，将高层请求转换为外部模型提供方或运行时端点可理解的协议。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module-level supporting statements
```python
import atexit
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 2-2: Module-level supporting statements
```python
import json
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 3-3: Module-level supporting statements
```python
import multiprocessing
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 4-4: Module-level supporting statements
```python
import time
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 5-5: Module-level supporting statements
```python
import warnings
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 6-6: Module-level supporting statements
```python
from typing import Dict, List, Optional, Union
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 7-7: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 8-8: Module-level supporting statements
```python
import aiohttp
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 9-9: Module-level supporting statements
```python
import requests
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 10-10: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 11-11: Module-level supporting statements
```python
from sglang.global_config import global_config
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 12-12: Module-level supporting statements
```python
from sglang.lang.backend.base_backend import BaseBackend
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 13-13: Module-level supporting statements
```python
from sglang.lang.chat_template import get_chat_template, get_chat_template_by_model_path
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 14-14: Module-level supporting statements
```python
from sglang.lang.choices import ChoicesDecision, ChoicesSamplingMethod
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 15-15: Module-level supporting statements
```python
from sglang.lang.interpreter import StreamExecutor
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 16-22: Module-level supporting statements
```python
from sglang.lang.ir import (
    REGEX_BOOL,
    REGEX_FLOAT,
    REGEX_INT,
    REGEX_STR,
    SglSamplingParams,
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 23-23: Module-level supporting statements
```python
from sglang.utils import http_request
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 24-25: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 26-26: RuntimeEndpoint class declaration
```python
class RuntimeEndpoint(BaseBackend):
```
**EN:** This block declares the `RuntimeEndpoint` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `RuntimeEndpoint` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 27-54: RuntimeEndpoint initializer
```python
    def __init__(
        self,
        base_url: str,
        api_key: Optional[str] = None,
        verify: Optional[str] = None,
        chat_template_name: Optional[str] = None,
    ):
        super().__init__()
        self.support_concate_and_append = True

        self.base_url = base_url
        self.api_key = api_key
        self.verify = verify

        res = http_request(
            self.base_url + "/get_model_info",
            api_key=self.api_key,
            verify=self.verify,
        )
        self._assert_success(res)
        self.model_info = res.json()

        if chat_template_name:
            self.chat_template = get_chat_template(chat_template_name)
        else:
            self.chat_template = get_chat_template_by_model_path(
                self.model_info["model_path"]
            )
```
**EN:** This block initializes the `RuntimeEndpoint` object, wiring together the state and dependencies used by later methods.
**CN:** 该代码块初始化 `RuntimeEndpoint` 对象，连接后续方法使用的状态与依赖。

### Lines 55-55: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 56-57: RuntimeEndpoint.get_model_name method
```python
    def get_model_name(self):
        return self.model_info["model_path"]
```
**EN:** This block uses `RuntimeEndpoint.get_model_name` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint.get_model_name` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 58-58: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 59-66: RuntimeEndpoint.flush_cache method
```python
    def flush_cache(self):
        res = http_request(
            self.base_url + "/flush_cache",
            api_key=self.api_key,
            verify=self.verify,
            method="POST",
        )
        self._assert_success(res)
```
**EN:** This block uses `RuntimeEndpoint.flush_cache` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint.flush_cache` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 67-67: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 68-75: RuntimeEndpoint.get_server_info method
```python
    def get_server_info(self):
        res = http_request(
            self.base_url + "/server_info",
            api_key=self.api_key,
            verify=self.verify,
        )
        self._assert_success(res)
        return res.json()
```
**EN:** This block uses `RuntimeEndpoint.get_server_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint.get_server_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 76-76: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 77-78: RuntimeEndpoint.get_chat_template method
```python
    def get_chat_template(self):
        return self.chat_template
```
**EN:** This block uses `RuntimeEndpoint.get_chat_template` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint.get_chat_template` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 79-79: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 80-87: RuntimeEndpoint.cache_prefix method
```python
    def cache_prefix(self, prefix_str: str):
        res = http_request(
            self.base_url + "/generate",
            json={"text": prefix_str, "sampling_params": {"max_new_tokens": 0}},
            api_key=self.api_key,
            verify=self.verify,
        )
        self._assert_success(res)
```
**EN:** This block uses `RuntimeEndpoint.cache_prefix` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint.cache_prefix` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 88-88: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 89-95: RuntimeEndpoint.start_profile method
```python
    def start_profile(self):
        res = http_request(
            self.base_url + "/start_profile",
            api_key=self.api_key,
            verify=self.verify,
        )
        self._assert_success(res)
```
**EN:** This block uses `RuntimeEndpoint.start_profile` to collect profiling data. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint.start_profile` 来收集剖析数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 96-96: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 97-103: RuntimeEndpoint.stop_profile method
```python
    def stop_profile(self):
        res = http_request(
            self.base_url + "/stop_profile",
            api_key=self.api_key,
            verify=self.verify,
        )
        self._assert_success(res)
```
**EN:** This block uses `RuntimeEndpoint.stop_profile` to collect profiling data. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint.stop_profile` 来收集剖析数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 104-104: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 105-114: RuntimeEndpoint.commit_lazy_operations method
```python
    def commit_lazy_operations(self, s: StreamExecutor):
        data = {"text": s.text_, "sampling_params": {"max_new_tokens": 0}}
        self._add_images(s, data)
        res = http_request(
            self.base_url + "/generate",
            json=data,
            api_key=self.api_key,
            verify=self.verify,
        )
        self._assert_success(res)
```
**EN:** This block uses `RuntimeEndpoint.commit_lazy_operations` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint.commit_lazy_operations` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 115-115: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 116-125: RuntimeEndpoint.fill_image method
```python
    def fill_image(self, s: StreamExecutor):
        data = {"text": s.text_, "sampling_params": {"max_new_tokens": 0}}
        self._add_images(s, data)
        res = http_request(
            self.base_url + "/generate",
            json=data,
            api_key=self.api_key,
            verify=self.verify,
        )
        self._assert_success(res)
```
**EN:** This block uses `RuntimeEndpoint.fill_image` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint.fill_image` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 126-126: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 127-157: RuntimeEndpoint._handle_dtype_to_regex method
```python
    def _handle_dtype_to_regex(self, sampling_params: SglSamplingParams):
        if sampling_params.dtype is None:
            return

        if sampling_params.stop == ():
            sampling_params.stop = []

        dtype_regex = None
        if sampling_params.dtype in ["int", int]:

            dtype_regex = REGEX_INT
            sampling_params.stop.extend([" ", "\n"])
        elif sampling_params.dtype in ["float", float]:

            dtype_regex = REGEX_FLOAT
            sampling_params.stop.extend([" ", "\n"])
        elif sampling_params.dtype in ["str", str]:

            dtype_regex = REGEX_STR
        elif sampling_params.dtype in ["bool", bool]:

            dtype_regex = REGEX_BOOL
        else:
            raise RuntimeError(f"Invalid dtype: {sampling_params.dtype}")

        if dtype_regex is not None and sampling_params.regex is not None:
            warnings.warn(
                f"Both dtype and regex are set. Only dtype will be used. dtype: {sampling_params.dtype}, regex: {sampling_params.regex}"
            )

        sampling_params.regex = dtype_regex
```
**EN:** This block uses `RuntimeEndpoint._handle_dtype_to_regex` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint._handle_dtype_to_regex` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 158-158: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 159-196: RuntimeEndpoint.generate method
```python
    def generate(
        self,
        s: StreamExecutor,
        sampling_params: SglSamplingParams,
    ):
        self._handle_dtype_to_regex(sampling_params)
        data = {
            "text": s.text_,
            "sampling_params": {
                "skip_special_tokens": global_config.skip_special_tokens_in_output,
                "spaces_between_special_tokens": global_config.spaces_between_special_tokens_in_out,
                **sampling_params.to_srt_kwargs(),
            },
        }

        for item in [
            "return_logprob",
            "logprob_start_len",
            "top_logprobs_num",
            "return_text_in_logprobs",
        ]:
            value = getattr(sampling_params, item, None)
            if value is not None:
                data[item] = value

        self._add_images(s, data)

        res = http_request(
            self.base_url + "/generate",
            json=data,
            api_key=self.api_key,
            verify=self.verify,
        )
        self._assert_success(res)

        obj = res.json()
        comp = obj["text"]
        return comp, obj["meta_info"]
```
**EN:** This block uses `RuntimeEndpoint.generate` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint.generate` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 197-197: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 198-246: RuntimeEndpoint.generate_stream method
```python
    def generate_stream(
        self,
        s: StreamExecutor,
        sampling_params: SglSamplingParams,
    ):
        self._handle_dtype_to_regex(sampling_params)

        data = {
            "text": s.text_,
            "sampling_params": {
                "skip_special_tokens": global_config.skip_special_tokens_in_output,
                "spaces_between_special_tokens": global_config.spaces_between_special_tokens_in_out,
                **sampling_params.to_srt_kwargs(),
            },
        }

        for item in [
            "return_logprob",
            "logprob_start_len",
            "top_logprobs_num",
            "return_text_in_logprobs",
        ]:
            value = getattr(sampling_params, item, None)
            if value is not None:
                data[item] = value

        data["stream"] = True
        self._add_images(s, data)

        res = http_request(
            self.base_url + "/generate",
            json=data,
            stream=True,
            api_key=self.api_key,
            verify=self.verify,
        )
        self._assert_success(res)
        pos = 0

        for chunk in res.iter_lines(decode_unicode=False):
            chunk = chunk.decode("utf-8")
            if chunk and chunk.startswith("data:"):
                if chunk == "data: [DONE]":
                    break
                data = json.loads(chunk[5:].strip("\n"))
                chunk_text = data["text"][pos:]
                meta_info = data["meta_info"]
                pos += len(chunk_text)
                yield chunk_text, meta_info
```
**EN:** This block uses `RuntimeEndpoint.generate_stream` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint.generate_stream` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 247-247: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 248-307: RuntimeEndpoint.select method (part 1/2)
```python
    def select(
        self,
        s: StreamExecutor,
        choices: List[str],
        temperature: float,
        choices_method: ChoicesSamplingMethod,
    ) -> ChoicesDecision:
        assert temperature <= 1e-5

        # Cache common prefix
        data = {"text": s.text_, "sampling_params": {"max_new_tokens": 0}}
        obj = self._generate_http_request(s, data)
        prompt_len = obj["meta_info"]["prompt_tokens"]
        logprob_start_len = max(prompt_len - 2, 0)  # For token healing

        # Compute logprob
        data = {
            "text": [s.text_ + c for c in choices],
            "sampling_params": {
                "max_new_tokens": 0,
                "temperature": 0,
            },
            "return_logprob": True,
            "return_text_in_logprobs": True,
            "logprob_start_len": logprob_start_len,
        }
        obj = self._generate_http_request(s, data)

        input_token_logprobs = [r["meta_info"]["input_token_logprobs"] for r in obj]
        output_token_logprobs = [r["meta_info"]["output_token_logprobs"] for r in obj]
        normalized_prompt_logprobs = [
            compute_normalized_prompt_logprobs(r["meta_info"]["input_token_logprobs"])
            for r in obj
        ]

        # Remove extra token if no token healing occurred
        for i in range(len(input_token_logprobs)):
            healed_token_str = input_token_logprobs[i][0][-1]
            if s.text_.endswith(healed_token_str):
                healed_token_logprob = input_token_logprobs[i][0][0]
                normalized_prompt_logprobs[i] = (
                    normalized_prompt_logprobs[i] * len(input_token_logprobs[i])
                    - healed_token_logprob
                ) / (len(input_token_logprobs[i]) - 1)
                input_token_logprobs[i] = input_token_logprobs[i][1:]

        # Compute unconditional logprobs if required
        if choices_method.requires_unconditional_logprobs:
            input_ids = [[el[1] for el in subl] for subl in input_token_logprobs]
            data = {
                "input_ids": input_ids,
                "sampling_params": {"max_new_tokens": 0},
                "return_logprob": True,
            }
            obj = self._generate_http_request(s, data)
            unconditional_token_logprobs = [
                r["meta_info"]["input_token_logprobs"] for r in obj
            ]
        else:
            unconditional_token_logprobs = None
```
**EN:** This block uses `RuntimeEndpoint.select` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `RuntimeEndpoint.select` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 308-315: RuntimeEndpoint.select method (part 2/2)
```python

        return choices_method(
            choices=choices,
            normalized_prompt_logprobs=normalized_prompt_logprobs,
            input_token_logprobs=input_token_logprobs,
            output_token_logprobs=output_token_logprobs,
            unconditional_token_logprobs=unconditional_token_logprobs,
        )
```
**EN:** This block uses `RuntimeEndpoint.select` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `RuntimeEndpoint.select` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 316-316: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 317-324: RuntimeEndpoint.concatenate_and_append method
```python
    def concatenate_and_append(self, src_rids: List[str], dst_rid: str):
        res = http_request(
            self.base_url + "/concate_and_append_request",
            json={"src_rids": src_rids, "dst_rid": dst_rid},
            api_key=self.api_key,
            verify=self.verify,
        )
        self._assert_success(res)
```
**EN:** This block uses `RuntimeEndpoint.concatenate_and_append` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint.concatenate_and_append` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 325-325: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 326-335: RuntimeEndpoint._generate_http_request method
```python
    def _generate_http_request(self, s: StreamExecutor, data):
        self._add_images(s, data)
        res = http_request(
            self.base_url + "/generate",
            json=data,
            api_key=self.api_key,
            verify=self.verify,
        )
        self._assert_success(res)
        return res.json()
```
**EN:** This block uses `RuntimeEndpoint._generate_http_request` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint._generate_http_request` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 336-336: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 337-340: RuntimeEndpoint._add_images method
```python
    def _add_images(self, s: StreamExecutor, data):
        if s.images_:
            assert len(s.images_) == 1, "Only support one image."
            data["image_data"] = s.images_[0][1]
```
**EN:** This block uses `RuntimeEndpoint._add_images` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint._add_images` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 341-341: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `RuntimeEndpoint` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`RuntimeEndpoint` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 342-348: RuntimeEndpoint._assert_success method
```python
    def _assert_success(self, res):
        if res.status_code != 200:
            try:
                content = res.json()
            except json.JSONDecodeError:
                content = res.text
            raise RuntimeError(content)
```
**EN:** This block uses `RuntimeEndpoint._assert_success` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RuntimeEndpoint._assert_success` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 349-350: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 351-353: compute normalized prompt logprobs function
```python
def compute_normalized_prompt_logprobs(input_logprobs):
    values = [x[0] for x in input_logprobs if x[0]]
    return sum(values) / len(values)
```
**EN:** This block uses `compute_normalized_prompt_logprobs` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `compute_normalized_prompt_logprobs` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 354-355: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 356-356: Runtime class declaration
```python
class Runtime:
```
**EN:** This block declares the `Runtime` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `Runtime` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 357-364: Class-level supporting statements
```python
    """
    A wrapper for the HTTP server.
    This is used for launching the server in a python program without
    using the command line interface.

    It is mainly used for the frontend language.
    You should use the Engine class if you want to do normal offline processing without the frontend language.
    """
```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 365-365: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 366-425: Runtime initializer (part 1/2)
```python
    def __init__(
        self,
        log_level: str = "error",
        launch_timeout: float = 300.0,
        *args,
        **kwargs,
    ):
        """See the arguments in server_args.py::ServerArgs

        Args:
            log_level: Log level for the server.
            timeout: Timeout in seconds for waiting for the server to start.
            *args: Additional arguments passed to ServerArgs.
            **kwargs: Additional keyword arguments passed to ServerArgs.
        """
        # We delay the import of any `sglang.srt` components in `sglang.lang`, so users can run
        # client code without installing SRT server and its dependency if they want.
        from sglang.srt.entrypoints.http_server import launch_server
        from sglang.srt.server_args import ServerArgs
        from sglang.srt.utils.network import is_port_available

        self.server_args = ServerArgs(*args, log_level=log_level, **kwargs)

        # Pre-allocate ports
        for port in range(self.server_args.port, 40000):
            if is_port_available(port):
                break
        self.server_args.port = port

        self.url = self.server_args.url()
        self.generate_url = self.url + "/generate"

        # NOTE: We store pid instead of proc to fix some issues during __delete__
        self.pid = None

        ctx = multiprocessing.get_context("spawn")
        proc = ctx.Process(
            target=launch_server,
            args=(self.server_args,),
        )
        proc.start()
        self.pid = proc.pid

        # Before python program terminates, call shutdown implicitly. Therefore, users don't have to explicitly call .shutdown()
        atexit.register(self.shutdown)

        # Wait for server to be ready by polling /health_generate
        start_time = time.time()
        with requests.Session() as session:
            while time.time() - start_time < launch_timeout:
                try:
                    response = session.get(f"{self.url}/health_generate")
                    if response.status_code == 200:
                        break
                except requests.RequestException:
                    pass

                if not proc.is_alive():
                    self.shutdown()
                    raise RuntimeError(
```
**EN:** This block initializes the `Runtime` object, wiring together the state and dependencies used by later methods. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块初始化 `Runtime` 对象，连接后续方法使用的状态与依赖。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 426-434: Runtime initializer (part 2/2)
```python
                        "Initialization failed. Please see the error messages above."
                    )

                time.sleep(2)
            else:
                self.shutdown()
                raise TimeoutError("Server failed to start within the timeout period.")

        self.endpoint = RuntimeEndpoint(self.url)
```
**EN:** This block initializes the `Runtime` object, wiring together the state and dependencies used by later methods. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块初始化 `Runtime` 对象，连接后续方法使用的状态与依赖。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 435-435: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 436-441: Runtime.shutdown method
```python
    def shutdown(self):
        from sglang.srt.utils import kill_process_tree

        if self.pid is not None:
            kill_process_tree(self.pid)
            self.pid = None
```
**EN:** This block uses `Runtime.shutdown` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `Runtime.shutdown` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 442-442: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 443-444: Runtime.start_profile method
```python
    def start_profile(self):
        self.endpoint.start_profile()
```
**EN:** This block uses `Runtime.start_profile` to collect profiling data. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `Runtime.start_profile` 来收集剖析数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 445-445: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 446-447: Runtime.stop_profile method
```python
    def stop_profile(self):
        self.endpoint.stop_profile()
```
**EN:** This block uses `Runtime.stop_profile` to collect profiling data. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `Runtime.stop_profile` 来收集剖析数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 448-448: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 449-450: Runtime.cache_prefix method
```python
    def cache_prefix(self, prefix: str):
        self.endpoint.cache_prefix(prefix)
```
**EN:** This block uses `Runtime.cache_prefix` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `Runtime.cache_prefix` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 451-451: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 452-460: Runtime.get_tokenizer method
```python
    def get_tokenizer(self):
        from sglang.srt.utils.hf_transformers_utils import get_tokenizer

        return get_tokenizer(
            self.server_args.tokenizer_path,
            tokenizer_mode=self.server_args.tokenizer_mode,
            trust_remote_code=self.server_args.trust_remote_code,
            revision=self.server_args.revision,
        )
```
**EN:** This block uses `Runtime.get_tokenizer` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `Runtime.get_tokenizer` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 461-461: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 462-496: Runtime.async_generate method
```python
    async def async_generate(
        self,
        prompt: str,
        sampling_params: Optional[Dict] = None,
    ):
        if self.server_args.skip_tokenizer_init:
            json_data = {
                "input_ids": prompt,
                "sampling_params": sampling_params,
                "stream": True,
            }
        else:
            json_data = {
                "text": prompt,
                "sampling_params": sampling_params,
                "stream": True,
            }
        pos = 0

        timeout = aiohttp.ClientTimeout(total=3 * 3600)
        async with aiohttp.ClientSession(timeout=timeout, trust_env=True) as session:
            async with session.post(self.generate_url, json=json_data) as response:
                async for chunk, _ in response.content.iter_chunks():
                    chunk = chunk.decode("utf-8")
                    if chunk and chunk.startswith("data:"):
                        if chunk == "data: [DONE]\n\n":
                            break
                        data = json.loads(chunk[5:].strip("\n"))
                        if "text" in data:
                            cur = data["text"][pos:]
                            if cur:
                                yield cur
                            pos += len(cur)
                        else:
                            yield data
```
**EN:** This block uses `Runtime.async_generate` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `Runtime.async_generate` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 497-497: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 498-498: Class-level supporting statements
```python
    add_request = async_generate
```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 499-499: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 500-522: Runtime.generate method
```python
    def generate(
        self,
        prompt: Union[str, List[str]],
        sampling_params: Optional[Dict] = None,
        return_logprob: Optional[Union[List[bool], bool]] = False,
        logprob_start_len: Optional[Union[List[int], int]] = None,
        top_logprobs_num: Optional[Union[List[int], int]] = None,
        lora_path: Optional[List[Optional[str]]] = None,
    ):
        json_data = {
            "text": prompt,
            "sampling_params": sampling_params,
            "return_logprob": return_logprob,
            "logprob_start_len": logprob_start_len,
            "top_logprobs_num": top_logprobs_num,
            "lora_path": lora_path,
        }
        assert not isinstance(lora_path, list) or len(lora_path) == len(prompt)
        response = requests.post(
            self.url + "/generate",
            json=json_data,
        )
        return json.dumps(response.json())
```
**EN:** This block uses `Runtime.generate` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `Runtime.generate` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 523-523: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 524-530: Runtime.encode method
```python
    def encode(
        self,
        prompt: Union[str, List[str], List[Dict], List[List[Dict]]],
    ):
        json_data = {"text": prompt}
        response = requests.post(self.url + "/encode", json=json_data)
        return json.dumps(response.json())
```
**EN:** This block uses `Runtime.encode` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `Runtime.encode` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 531-531: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 532-541: Runtime.get_server_info method
```python
    async def get_server_info(self):
        async with aiohttp.ClientSession() as session:
            async with session.get(f"{self.url}/server_info") as response:
                if response.status == 200:
                    return await response.json()
                else:
                    error_data = await response.json()
                    raise RuntimeError(
                        f"Failed to get server info. {error_data['error']['message']}"
                    )
```
**EN:** This block uses `Runtime.get_server_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `Runtime.get_server_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 542-542: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Runtime` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Runtime` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 543-544: Runtime.__del__ method
```python
    def __del__(self):
        self.shutdown()
```
**EN:** This block uses `Runtime.__del__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `Runtime.__del__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Backend integration / 后端集成
- External API integration / 外部 API 集成
- Prompt and template handling / 提示词与模板处理
- Runtime coordination / 运行时协同

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.global_config`
- `sglang.lang.backend.base_backend`
- `sglang.lang.chat_template`
- `sglang.lang.choices`
- `sglang.lang.interpreter`
- `sglang.lang.ir`
- `sglang.srt.entrypoints.http_server`
- `sglang.srt.server_args`
- `sglang.srt.utils`
- `sglang.srt.utils.hf_transformers_utils`
- `sglang.srt.utils.network`
- `sglang.utils`
### External / 外部
- `aiohttp`
- `atexit`
- `multiprocessing`
- `requests`
- `json` (stdlib)
- `time` (stdlib)
- `typing` (stdlib)
- `warnings` (stdlib)
