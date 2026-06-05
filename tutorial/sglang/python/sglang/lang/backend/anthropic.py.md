# anthropic.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/lang/backend/anthropic.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements an SGLang language backend adapter, translating high-level requests into the protocol of an external model provider or runtime endpoint. / 该文件实现了 SGLang 语言层的后端适配器，将高层请求转换为外部模型提供方或运行时端点可理解的协议。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module-level supporting statements
```python
from sglang.lang.backend.base_backend import BaseBackend
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 2-2: Module-level supporting statements
```python
from sglang.lang.chat_template import get_chat_template
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 3-3: Module-level supporting statements
```python
from sglang.lang.interpreter import StreamExecutor
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 4-4: Module-level supporting statements
```python
from sglang.lang.ir import SglSamplingParams
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 5-5: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 6-9: Module-level supporting statements
```python
try:
    import anthropic
except ImportError as e:
    anthropic = e
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 10-11: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 12-12: Anthropic class declaration
```python
class Anthropic(BaseBackend):
```
**EN:** This block declares the `Anthropic` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `Anthropic` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 13-21: Anthropic initializer
```python
    def __init__(self, model_name, *args, **kwargs):
        super().__init__()

        if isinstance(anthropic, Exception):
            raise anthropic

        self.model_name = model_name
        self.chat_template = get_chat_template("claude")
        self.client = anthropic.Anthropic(*args, **kwargs)
```
**EN:** This block initializes the `Anthropic` object, wiring together the state and dependencies used by later methods.
**CN:** 该代码块初始化 `Anthropic` 对象，连接后续方法使用的状态与依赖。

### Lines 22-22: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Anthropic` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Anthropic` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 23-24: Anthropic.get_chat_template method
```python
    def get_chat_template(self):
        return self.chat_template
```
**EN:** This block uses `Anthropic.get_chat_template` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `Anthropic.get_chat_template` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 25-25: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Anthropic` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Anthropic` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 26-49: Anthropic.generate method
```python
    def generate(
        self,
        s: StreamExecutor,
        sampling_params: SglSamplingParams,
    ):
        if s.messages_:
            messages = s.messages_
        else:
            messages = [{"role": "user", "content": s.text_}]

        if messages and messages[0]["role"] == "system":
            system = messages.pop(0)["content"]
        else:
            system = ""

        ret = self.client.messages.create(
            model=self.model_name,
            system=system,
            messages=messages,
            **sampling_params.to_anthropic_kwargs(),
        )
        comp = ret.content[0].text

        return comp, {}
```
**EN:** This block uses `Anthropic.generate` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `Anthropic.generate` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 50-50: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `Anthropic` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`Anthropic` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 51-73: Anthropic.generate_stream method
```python
    def generate_stream(
        self,
        s: StreamExecutor,
        sampling_params: SglSamplingParams,
    ):
        if s.messages_:
            messages = s.messages_
        else:
            messages = [{"role": "user", "content": s.text_}]

        if messages and messages[0]["role"] == "system":
            system = messages.pop(0)["content"]
        else:
            system = ""

        with self.client.messages.stream(
            model=self.model_name,
            system=system,
            messages=messages,
            **sampling_params.to_anthropic_kwargs(),
        ) as stream:
            for text in stream.text_stream:
                yield text, {}
```
**EN:** This block uses `Anthropic.generate_stream` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `Anthropic.generate_stream` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

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
- `anthropic`
