# vertexai.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/lang/backend/vertexai.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements an SGLang language backend adapter, translating high-level requests into the protocol of an external model provider or runtime endpoint. / 该文件实现了 SGLang 语言层的后端适配器，将高层请求转换为外部模型提供方或运行时端点可理解的协议。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module-level supporting statements
```python
import os
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 2-2: Module-level supporting statements
```python
import warnings
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 3-3: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 4-4: Module-level supporting statements
```python
from sglang.lang.backend.base_backend import BaseBackend
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 5-5: Module-level supporting statements
```python
from sglang.lang.chat_template import get_chat_template
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 6-6: Module-level supporting statements
```python
from sglang.lang.interpreter import StreamExecutor
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 7-7: Module-level supporting statements
```python
from sglang.lang.ir import SglSamplingParams
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 8-8: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 9-17: Module-level supporting statements
```python
try:
    import vertexai
    from vertexai.preview.generative_models import (
        GenerationConfig,
        GenerativeModel,
        Image,
    )
except ImportError as e:
    GenerativeModel = e
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 18-19: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 20-20: VertexAI class declaration
```python
class VertexAI(BaseBackend):
```
**EN:** This block declares the `VertexAI` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `VertexAI` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 21-33: VertexAI initializer
```python
    def __init__(self, model_name, safety_settings=None):
        super().__init__()

        if isinstance(GenerativeModel, Exception):
            raise GenerativeModel

        project_id = os.environ["GCP_PROJECT_ID"]
        location = os.environ.get("GCP_LOCATION")
        vertexai.init(project=project_id, location=location)

        self.model_name = model_name
        self.chat_template = get_chat_template("default")
        self.safety_settings = safety_settings
```
**EN:** This block initializes the `VertexAI` object, wiring together the state and dependencies used by later methods.
**CN:** 该代码块初始化 `VertexAI` 对象，连接后续方法使用的状态与依赖。

### Lines 34-34: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `VertexAI` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`VertexAI` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 35-36: VertexAI.get_chat_template method
```python
    def get_chat_template(self):
        return self.chat_template
```
**EN:** This block uses `VertexAI.get_chat_template` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `VertexAI.get_chat_template` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 37-37: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `VertexAI` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`VertexAI` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 38-60: VertexAI.generate method
```python
    def generate(
        self,
        s: StreamExecutor,
        sampling_params: SglSamplingParams,
    ):
        if s.messages_:
            prompt = self.messages_to_vertexai_input(s.messages_)
        else:
            # single-turn
            prompt = (
                self.text_to_vertexai_input(s.text_, s.cur_images)
                if s.cur_images
                else s.text_
            )
        ret = GenerativeModel(self.model_name).generate_content(
            prompt,
            generation_config=GenerationConfig(**sampling_params.to_vertexai_kwargs()),
            safety_settings=self.safety_settings,
        )

        comp = ret.text

        return comp, {}
```
**EN:** This block uses `VertexAI.generate` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `VertexAI.generate` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 61-61: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `VertexAI` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`VertexAI` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 62-83: VertexAI.generate_stream method
```python
    def generate_stream(
        self,
        s: StreamExecutor,
        sampling_params: SglSamplingParams,
    ):
        if s.messages_:
            prompt = self.messages_to_vertexai_input(s.messages_)
        else:
            # single-turn
            prompt = (
                self.text_to_vertexai_input(s.text_, s.cur_images)
                if s.cur_images
                else s.text_
            )
        generator = GenerativeModel(self.model_name).generate_content(
            prompt,
            stream=True,
            generation_config=GenerationConfig(**sampling_params.to_vertexai_kwargs()),
            safety_settings=self.safety_settings,
        )
        for ret in generator:
            yield ret.text, {}
```
**EN:** This block uses `VertexAI.generate_stream` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `VertexAI.generate_stream` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 84-84: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `VertexAI` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`VertexAI` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 85-97: VertexAI.text_to_vertexai_input method
```python
    def text_to_vertexai_input(self, text, images):
        input = []
        # split with image token
        text_segs = text.split(self.chat_template.image_token)
        for image_path, image_base64_data in images:
            text_seg = text_segs.pop(0)
            if text_seg != "":
                input.append(text_seg)
            input.append(Image.from_bytes(image_base64_data))
        text_seg = text_segs.pop(0)
        if text_seg != "":
            input.append(text_seg)
        return input
```
**EN:** This block uses `VertexAI.text_to_vertexai_input` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `VertexAI.text_to_vertexai_input` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 98-98: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `VertexAI` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`VertexAI` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 99-148: VertexAI.messages_to_vertexai_input method
```python
    def messages_to_vertexai_input(self, messages):
        vertexai_message = []
        # from openai message format to vertexai message format
        for msg in messages:
            if isinstance(msg["content"], str):
                text = msg["content"]
            else:
                text = msg["content"][0]["text"]

            if msg["role"] == "system":
                warnings.warn("Warning: system prompt is not supported in VertexAI.")
                vertexai_message.append(
                    {
                        "role": "user",
                        "parts": [{"text": "System prompt: " + text}],
                    }
                )
                vertexai_message.append(
                    {
                        "role": "model",
                        "parts": [{"text": "Understood."}],
                    }
                )
                continue
            if msg["role"] == "user":
                vertexai_msg = {
                    "role": "user",
                    "parts": [{"text": text}],
                }
            elif msg["role"] == "assistant":
                vertexai_msg = {
                    "role": "model",
                    "parts": [{"text": text}],
                }

            # images
            if isinstance(msg["content"], list) and len(msg["content"]) > 1:
                for image in msg["content"][1:]:
                    assert image["type"] == "image_url"
                    vertexai_msg["parts"].append(
                        {
                            "inline_data": {
                                "data": image["image_url"]["url"].split(",")[1],
                                "mime_type": "image/jpeg",
                            }
                        }
                    )

            vertexai_message.append(vertexai_msg)
        return vertexai_message
```
**EN:** This block uses `VertexAI.messages_to_vertexai_input` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `VertexAI.messages_to_vertexai_input` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Backend integration / 后端集成
- Prompt and template handling / 提示词与模板处理

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.lang.backend.base_backend`
- `sglang.lang.chat_template`
- `sglang.lang.interpreter`
- `sglang.lang.ir`
### External / 外部
- `vertexai`
- `os` (stdlib)
- `warnings` (stdlib)
