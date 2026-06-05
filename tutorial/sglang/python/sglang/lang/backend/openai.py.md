# openai.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/lang/backend/openai.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements an SGLang language backend adapter, translating high-level requests into the protocol of an external model provider or runtime endpoint. / 该文件实现了 SGLang 语言层的后端适配器，将高层请求转换为外部模型提供方或运行时端点可理解的协议。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module-level supporting statements
```python
import dataclasses
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 2-2: Module-level supporting statements
```python
import logging
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 3-3: Module-level supporting statements
```python
import time
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 4-4: Module-level supporting statements
```python
import warnings
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 5-5: Module-level supporting statements
```python
from typing import List, Optional, Union
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 6-6: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 7-7: Module-level supporting statements
```python
import numpy as np
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 8-8: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 9-9: Module-level supporting statements
```python
from sglang.lang.backend.base_backend import BaseBackend
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 10-10: Module-level supporting statements
```python
from sglang.lang.chat_template import ChatTemplate, get_chat_template_by_model_path
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 11-11: Module-level supporting statements
```python
from sglang.lang.choices import ChoicesDecision, ChoicesSamplingMethod
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 12-12: Module-level supporting statements
```python
from sglang.lang.interpreter import StreamExecutor
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 13-13: Module-level supporting statements
```python
from sglang.lang.ir import SglSamplingParams
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 14-14: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 15-19: Module-level supporting statements
```python
try:
    import openai
    import tiktoken
except ImportError as e:
    openai = tiktoken = e
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 20-21: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 22-22: Module-level supporting statements
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 23-24: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 25-39: create logit bias int function
```python
def create_logit_bias_int(tokenizer):
    """Get logit bias for integer numbers."""
    int_token_ids = []

    tokens = tokenizer._mergeable_ranks
    for token, token_id in tokens.items():
        s = tokenizer.decode([token_id])
        if all([c.isdigit() for c in s]) or s in [" "]:
            int_token_ids.append(token_id)
            if len(int_token_ids) >= 300:  # OpenAI API limit
                break
    special_tokens = tokenizer._special_tokens
    mask = {t: 100 for t in int_token_ids[:299]}
    mask[special_tokens["<|endoftext|>"]] = 100
    return mask
```
**EN:** This block uses `create_logit_bias_int` to create runtime objects or artifacts. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `create_logit_bias_int` 来创建运行时对象或产物。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 40-41: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 42-44: Module-level supporting statements
```python
INSTRUCT_MODEL_NAMES = [
    "gpt-3.5-turbo-instruct",
]
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 45-47: Module-level supporting statements
```python


@dataclasses.dataclass
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 48-48: TokenUsage class declaration
```python
class TokenUsage:
```
**EN:** This block declares the `TokenUsage` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `TokenUsage` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 49-49: Class-level supporting statements
```python
    prompt_tokens: int
```
**EN:** This block contains supporting statements for the `TokenUsage` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`TokenUsage` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 50-50: Class-level supporting statements
```python
    completion_tokens: int
```
**EN:** This block contains supporting statements for the `TokenUsage` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`TokenUsage` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 51-51: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `TokenUsage` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`TokenUsage` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 52-53: TokenUsage.reset method
```python
    def reset(self):
        self.prompt_tokens = self.completion_tokens = 0
```
**EN:** This block uses `TokenUsage.reset` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TokenUsage.reset` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 54-55: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 56-56: OpenAI class declaration
```python
class OpenAI(BaseBackend):
```
**EN:** This block declares the `OpenAI` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `OpenAI` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 57-104: OpenAI initializer
```python
    def __init__(
        self,
        model_name: str,
        is_chat_model: Optional[bool] = None,
        chat_template: Optional[ChatTemplate] = None,
        is_azure: bool = False,
        *args,
        **kwargs,
    ):
        super().__init__()

        if isinstance(openai, Exception):
            raise openai

        if is_azure:
            self.client = openai.AzureOpenAI(*args, **kwargs)
        else:
            self.client = openai.OpenAI(*args, **kwargs)

        self.model_name = model_name
        try:
            self.tokenizer = tiktoken.encoding_for_model(model_name)
        except KeyError:
            self.tokenizer = tiktoken.get_encoding("cl100k_base")
        self.logit_bias_int = create_logit_bias_int(self.tokenizer)

        self.chat_template = chat_template or get_chat_template_by_model_path(
            model_name
        )

        if is_chat_model is not None:
            self.is_chat_model = is_chat_model
        else:
            if model_name in INSTRUCT_MODEL_NAMES:
                self.is_chat_model = False
            else:
                self.is_chat_model = True

        self.chat_prefix = self.chat_template.role_prefix_and_suffix["assistant"][0]

        # Usage
        self.token_usage = TokenUsage(0, 0)

        # API speculative execution
        # TODO(ying): This does not support multi-threading (run_batch)
        self.spec_kwargs = {}
        self.spec_format = []
        self.spec_max_num_tries = 3
```
**EN:** This block initializes the `OpenAI` object, wiring together the state and dependencies used by later methods.
**CN:** 该代码块初始化 `OpenAI` 对象，连接后续方法使用的状态与依赖。

### Lines 105-105: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `OpenAI` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`OpenAI` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 106-107: OpenAI.get_chat_template method
```python
    def get_chat_template(self):
        return self.chat_template
```
**EN:** This block uses `OpenAI.get_chat_template` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `OpenAI.get_chat_template` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 108-108: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `OpenAI` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`OpenAI` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 109-138: OpenAI._prepare_spec_execution method
```python
    def _prepare_spec_execution(
        self,
        sampling_params: SglSamplingParams,
        num_api_spec_tokens: int,
        spec_var_name: str,
    ):
        if "max_tokens" not in self.spec_kwargs:
            self.spec_kwargs["max_tokens"] = num_api_spec_tokens
        else:
            assert self.spec_kwargs["max_tokens"] == num_api_spec_tokens

        params = sampling_params.to_openai_kwargs()
        for key, value in params.items():
            if key in ["stop"]:
                continue
            if key in ["max_tokens"]:
                warnings.warn(
                    "The parameter max_tokens will be overwritten by speculated number of tokens."
                )
                continue
            if key not in self.spec_kwargs:
                self.spec_kwargs[key] = value
            else:
                assert (
                    value == self.spec_kwargs[key]
                ), "sampling parameters should be consistent if turn on api speculative execution."
        self.spec_format.append(
            {"text": "", "stop": params["stop"], "name": spec_var_name}
        )
        return "", {}
```
**EN:** This block uses `OpenAI._prepare_spec_execution` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `OpenAI._prepare_spec_execution` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 139-139: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `OpenAI` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`OpenAI` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 140-199: OpenAI.generate method (part 1/2)
```python
    def generate(
        self,
        s: StreamExecutor,
        sampling_params: SglSamplingParams,
        spec_var_name: str = None,
    ):
        if sampling_params.dtype is None:
            if self.is_chat_model:
                if s.num_api_spec_tokens is None:
                    if not s.text_.endswith(self.chat_prefix):
                        raise RuntimeError(
                            "This use case is not supported if api speculative execution is off. "
                            "For OpenAI chat models, sgl.gen must be right after sgl.assistant. "
                            "Example of adding api speculative execution: @function(num_api_spec_tokens=128)."
                        )
                    prompt = s.messages_
                else:
                    return self._prepare_spec_execution(
                        sampling_params, s.num_api_spec_tokens, spec_var_name
                    )
            else:
                prompt = s.text_

            kwargs = sampling_params.to_openai_kwargs()
            if (
                self.model_name.startswith("o1")
                or self.model_name.startswith("o3")
                or "o1" in self.model_name
            ):
                kwargs.pop("max_tokens", None)
            else:
                kwargs.pop("max_completion_tokens", None)

            comp = openai_completion(
                client=self.client,
                token_usage=self.token_usage,
                is_chat=self.is_chat_model,
                model=self.model_name,
                prompt=prompt,
                **kwargs,
            )
            # Keep the returned list (or string) as is.
        elif sampling_params.dtype in [str, "str", "string"]:
            assert (
                not self.is_chat_model
            ), "constrained type not supported on chat model"
            kwargs = sampling_params.to_openai_kwargs()
            kwargs.pop("stop")
            comp = openai_completion(
                client=self.client,
                token_usage=self.token_usage,
                is_chat=self.is_chat_model,
                model=self.model_name,
                prompt=s.text_ + '"',
                stop='"',
                **kwargs,
            )
            # Wrap each element in quotes if we have a list.
            if isinstance(comp, list):
                comp = ['"' + x + '"' for x in comp]
```
**EN:** This block uses `OpenAI.generate` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `OpenAI.generate` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 200-222: OpenAI.generate method (part 2/2)
```python
            else:
                comp = '"' + comp + '"'
        elif sampling_params.dtype in [int, "int"]:
            assert (
                not self.is_chat_model
            ), "constrained type not supported on chat model"
            kwargs = sampling_params.to_openai_kwargs()
            kwargs.pop("stop")
            comp = openai_completion(
                client=self.client,
                token_usage=self.token_usage,
                is_chat=self.is_chat_model,
                model=self.model_name,
                prompt=s.text_,
                logit_bias=self.logit_bias_int,
                stop=[" "],
                **kwargs,
            )
            # Leave as a list if that's what is returned.
        else:
            raise ValueError(f"Unknown dtype: {sampling_params.dtype}")

        return comp, {}
```
**EN:** This block uses `OpenAI.generate` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `OpenAI.generate` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 223-223: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `OpenAI` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`OpenAI` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 224-226: OpenAI.spec_fill method
```python
    def spec_fill(self, value: str):
        assert self.is_chat_model
        self.spec_format.append({"text": value, "stop": None, "name": None})
```
**EN:** This block uses `OpenAI.spec_fill` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `OpenAI.spec_fill` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 227-227: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `OpenAI` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`OpenAI` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 228-246: OpenAI.spec_pattern_match method
```python
    def spec_pattern_match(self, comp):
        for i, term in enumerate(self.spec_format):
            text = term["text"]
            if text != "":
                if comp.startswith(text):
                    comp = comp[len(text) :]
                else:
                    return False
            else:
                pos = comp.find(term["stop"])
                if pos != -1:
                    term["text"] = comp[:pos]
                    comp = comp[pos:]
                else:
                    if i == len(self.spec_format) - 1:
                        term["text"] = comp
                    else:
                        return False
        return True
```
**EN:** This block uses `OpenAI.spec_pattern_match` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `OpenAI.spec_pattern_match` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 247-247: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `OpenAI` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`OpenAI` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 248-281: OpenAI.role_end_generate method
```python
    def role_end_generate(
        self,
        s: StreamExecutor,
    ):
        if s.num_api_spec_tokens is None or not s.text_.endswith(self.chat_prefix):
            return

        comp = ""
        if not all(x["name"] is None for x in self.spec_format):
            # TODO(ying): throw errors or warnings
            for i in range(self.spec_max_num_tries):
                comp = openai_completion(
                    client=self.client,
                    token_usage=self.token_usage,
                    is_chat=self.is_chat_model,
                    model=self.model_name,
                    prompt=s.messages_,
                    **self.spec_kwargs,
                )
                # Use a string for pattern matching.
                comp_for_match = comp[0] if isinstance(comp, list) else comp
                if self.spec_pattern_match(comp_for_match):
                    break

        for term in self.spec_format:
            s.text_ += term["text"]
            name = term["name"]
            if name is not None:
                s.variables[name] = term["text"]
                s.meta_info[name] = {}
                s.variable_event[name].set()

        self.spec_kwargs = {}
        self.spec_format = []
```
**EN:** This block uses `OpenAI.role_end_generate` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `OpenAI.role_end_generate` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 282-282: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `OpenAI` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`OpenAI` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 283-310: OpenAI.generate_stream method
```python
    def generate_stream(
        self,
        s: StreamExecutor,
        sampling_params: SglSamplingParams,
    ):
        if sampling_params.dtype is None:
            if self.is_chat_model:
                if not s.text_.endswith(self.chat_prefix):
                    raise RuntimeError(
                        "This use case is not supported. "
                        "For OpenAI chat models, sgl.gen must be right after sgl.assistant"
                    )
                prompt = s.messages_
            else:
                prompt = s.text_

            kwargs = sampling_params.to_openai_kwargs()
            generator = openai_completion_stream(
                client=self.client,
                token_usage=self.token_usage,
                is_chat=self.is_chat_model,
                model=self.model_name,
                prompt=prompt,
                **kwargs,
            )
            return generator
        else:
            raise ValueError(f"Unknown dtype: {sampling_params.dtype}")
```
**EN:** This block uses `OpenAI.generate_stream` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `OpenAI.generate_stream` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 311-311: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `OpenAI` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`OpenAI` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 312-371: OpenAI.select method (part 1/2)
```python
    def select(
        self,
        s: StreamExecutor,
        choices: List[str],
        temperature: float,
        choices_method: ChoicesSamplingMethod,
    ) -> ChoicesDecision:
        """Note: `choices_method` is not used by the OpenAI backend."""
        if self.is_chat_model:
            raise NotImplementedError(
                "select/choices is not supported for chat models. "
                "Please try to use a non-chat model such as gpt-3.5-turbo-instruct"
            )

        n_choices = len(choices)
        token_ids = [self.tokenizer.encode(x) for x in choices]
        scores = [0] * n_choices
        valid = [len(x) > 0 for x in token_ids]
        prompt_tokens = self.tokenizer.encode(s.text_)

        max_len = max([len(x) for x in token_ids])
        for step in range(max_len):
            # Build logit bias
            logit_bias = {}
            for i in range(n_choices):
                if valid[i]:
                    logit_bias[token_ids[i][step]] = 100

            # Call API
            ret = self.client.completions.create(
                model=self.model_name,
                prompt=prompt_tokens,
                logit_bias=logit_bias,
                max_tokens=1,
                temperature=temperature,
            )
            ret_str = ret.choices[0].text
            ret_token = self.tokenizer.encode(ret_str)[0]
            self.token_usage.prompt_tokens += ret.usage.prompt_tokens
            self.token_usage.completion_tokens = ret.usage.completion_tokens

            # TODO:
            # 1. return logits as the scores
            # 2. compute logits of the full choice
            # 3. consider chunk-based decoding

            # Update valid
            hit = False
            for i in range(n_choices):
                if valid[i]:
                    if step == len(token_ids[i]) - 1:
                        valid[i] = False

                    if ret_token == token_ids[i][step]:
                        scores[i] += 1
                        hit = True
                    else:
                        valid[i] = False
            assert hit
```
**EN:** This block uses `OpenAI.select` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `OpenAI.select` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 372-380: OpenAI.select method (part 2/2)
```python
            if np.sum(valid) <= 1:
                break

            prompt_tokens.append(ret_token)

        return ChoicesDecision(
            decision=choices[np.argmax(scores)],
            meta_info={"scores": scores},
        )
```
**EN:** This block uses `OpenAI.select` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `OpenAI.select` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 381-382: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 383-422: openai completion function
```python
def openai_completion(
    client, token_usage, is_chat=None, retries=3, prompt=None, **kwargs
) -> Union[str, List[str]]:
    # if "ebnf" is in kwargs, warn and remove
    if "ebnf" in kwargs:
        warnings.warn("EBNF is not officially supported by OpenAI endpoints. Ignoring.")
        del kwargs["ebnf"]

    for attempt in range(retries):
        try:
            if is_chat:
                if "stop" in kwargs and kwargs["stop"] is None:
                    kwargs.pop("stop")
                ret = client.chat.completions.create(messages=prompt, **kwargs)
                if len(ret.choices) == 1:
                    comp = ret.choices[0].message.content
                else:
                    comp = [c.message.content for c in ret.choices]
            else:
                ret = client.completions.create(prompt=prompt, **kwargs)
                if isinstance(prompt, (list, tuple)):
                    comp = [c.text for c in ret.choices]
                else:
                    comp = ret.choices[0].text
                    if len(ret.choices) > 1:
                        comp = [c.text for c in ret.choices]

            token_usage.prompt_tokens += ret.usage.prompt_tokens
            token_usage.completion_tokens += ret.usage.completion_tokens
            break
        except (openai.APIError, openai.APIConnectionError, openai.RateLimitError) as e:
            logger.error(f"OpenAI Error: {e}. Waiting 5 seconds...")
            time.sleep(5)
            if attempt == retries - 1:
                raise e
        except Exception as e:
            logger.error(f"RuntimeError {e}.")
            raise e

    return comp
```
**EN:** This block uses `openai_completion` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `openai_completion` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 423-424: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 425-475: openai completion stream function
```python
def openai_completion_stream(
    client, token_usage, is_chat=None, retries=3, prompt=None, **kwargs
):
    # if "ebnf" is in kwargs, warn and remove
    if "ebnf" in kwargs:
        warnings.warn("EBNF is not officially supported by OpenAI endpoints. Ignoring.")
        del kwargs["ebnf"]

    for attempt in range(retries):
        try:
            if is_chat:
                if "stop" in kwargs and kwargs["stop"] is None:
                    kwargs.pop("stop")
                generator = client.chat.completions.create(
                    messages=prompt,
                    stream=True,
                    stream_options={"include_usage": True},
                    **kwargs,
                )
                for ret in generator:
                    if len(ret.choices) == 0:
                        continue
                    try:
                        content = ret.choices[0].delta.content
                    except IndexError:
                        content = None
                    yield content or "", {}
            else:
                generator = client.completions.create(
                    prompt=prompt,
                    stream=True,
                    stream_options={"include_usage": True},
                    **kwargs,
                )
                for ret in generator:
                    if len(ret.choices) == 0:
                        continue
                    content = ret.choices[0].text
                    yield content or "", {}

            token_usage.prompt_tokens += ret.usage.prompt_tokens
            token_usage.completion_tokens += ret.usage.completion_tokens
            break
        except (openai.APIError, openai.APIConnectionError, openai.RateLimitError) as e:
            logger.error(f"OpenAI Error: {e}. Waiting 5 seconds...")
            time.sleep(5)
            if attempt == retries - 1:
                raise e
        except Exception as e:
            logger.error(f"RuntimeError {e}.")
            raise e
```
**EN:** This block uses `openai_completion_stream` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `openai_completion_stream` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Backend integration / 后端集成
- External API integration / 外部 API 集成
- Numerical data processing / 数值数据处理
- Prompt and template handling / 提示词与模板处理

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.lang.backend.base_backend`
- `sglang.lang.chat_template`
- `sglang.lang.choices`
- `sglang.lang.interpreter`
- `sglang.lang.ir`
### External / 外部
- `numpy`
- `openai`
- `tiktoken`
- `dataclasses` (stdlib)
- `logging` (stdlib)
- `time` (stdlib)
- `typing` (stdlib)
- `warnings` (stdlib)
