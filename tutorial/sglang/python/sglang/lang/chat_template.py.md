# chat_template.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/lang/chat_template.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements part of SGLang's frontend language layer, such as tracing, intermediate representation, prompting utilities, or chat formatting. / 该文件实现了 SGLang 前端语言层的一部分，例如追踪、中间表示、提示工具或聊天格式化逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module-level supporting statements
```python
import re
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 2-2: Module-level supporting statements
```python
from dataclasses import dataclass
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 3-3: Module-level supporting statements
```python
from enum import Enum, auto
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 4-4: Module-level supporting statements
```python
from typing import Callable, Dict, List, Tuple
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 5-6: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 7-7: ChatTemplateStyle class declaration
```python
class ChatTemplateStyle(Enum):
```
**EN:** This block declares the `ChatTemplateStyle` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `ChatTemplateStyle` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 8-8: Class-level supporting statements
```python
    PLAIN = auto()
```
**EN:** This block contains supporting statements for the `ChatTemplateStyle` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChatTemplateStyle` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 9-9: Class-level supporting statements
```python
    LLAMA2 = auto()
```
**EN:** This block contains supporting statements for the `ChatTemplateStyle` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChatTemplateStyle` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 10-12: Module-level supporting statements
```python


@dataclass
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 13-13: ChatTemplate class declaration
```python
class ChatTemplate:
```
**EN:** This block declares the `ChatTemplate` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `ChatTemplate` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 14-14: Class-level supporting statements
```python
    name: str
```
**EN:** This block contains supporting statements for the `ChatTemplate` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChatTemplate` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 15-15: Class-level supporting statements
```python
    default_system_prompt: str
```
**EN:** This block contains supporting statements for the `ChatTemplate` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChatTemplate` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 16-16: Class-level supporting statements
```python
    role_prefix_and_suffix: Dict[str, Tuple[str, str]]
```
**EN:** This block contains supporting statements for the `ChatTemplate` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChatTemplate` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 17-17: Class-level supporting statements
```python
    stop_str: List[str] = ()
```
**EN:** This block contains supporting statements for the `ChatTemplate` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChatTemplate` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 18-18: Class-level supporting statements
```python
    image_token: str = "<image>"
```
**EN:** This block contains supporting statements for the `ChatTemplate` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChatTemplate` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 19-19: Class-level supporting statements
```python
    audio_token: str = "<audio>"
```
**EN:** This block contains supporting statements for the `ChatTemplate` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChatTemplate` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 20-20: Class-level supporting statements
```python
    style: ChatTemplateStyle = ChatTemplateStyle.PLAIN
```
**EN:** This block contains supporting statements for the `ChatTemplate` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChatTemplate` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 21-21: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ChatTemplate` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChatTemplate` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 22-41: ChatTemplate.get_prefix_and_suffix method
```python
    def get_prefix_and_suffix(
        self, role: str, hist_messages: List[Dict]
    ) -> Tuple[str, str]:
        prefix, suffix = self.role_prefix_and_suffix.get(role, ("", ""))

        if self.style == ChatTemplateStyle.LLAMA2:
            if role == "system" and not hist_messages:
                user_prefix, _ = self.role_prefix_and_suffix.get("user", ("", ""))
                system_prefix, system_suffix = self.role_prefix_and_suffix.get(
                    "system", ("", "")
                )
                return (user_prefix + system_prefix, system_suffix)
            elif (
                role == "user"
                and len(hist_messages) == 1
                and hist_messages[0]["content"] is not None
            ):
                return ("", suffix)

        return prefix, suffix
```
**EN:** This block uses `ChatTemplate.get_prefix_and_suffix` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChatTemplate.get_prefix_and_suffix` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 42-42: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ChatTemplate` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChatTemplate` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 43-54: ChatTemplate.get_prompt method
```python
    def get_prompt(self, messages: List[Dict]) -> str:
        prompt = ""
        for i, message in enumerate(messages):
            role, content = message["role"], message["content"]
            if role == "system" and content is None:
                content = self.default_system_prompt
                if content is None:
                    continue

            prefix, suffix = self.get_prefix_and_suffix(role, messages[:i])
            prompt += f"{prefix}{content}{suffix}"
        return prompt
```
**EN:** This block uses `ChatTemplate.get_prompt` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChatTemplate.get_prompt` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 55-56: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 57-57: Module-level supporting statements
```python
chat_template_registry: Dict[str, ChatTemplate] = {}
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 58-58: Module-level supporting statements
```python
matching_function_registry: List[Callable] = []
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 59-60: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 61-62: register chat template function
```python
def register_chat_template(template):
    chat_template_registry[template.name] = template
```
**EN:** This block uses `register_chat_template` to register components for later lookup. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `register_chat_template` 来注册组件以供后续查找。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 63-64: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 65-66: register chat template matching function function
```python
def register_chat_template_matching_function(func):
    matching_function_registry.append(func)
```
**EN:** This block uses `register_chat_template_matching_function` to register components for later lookup. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `register_chat_template_matching_function` 来注册组件以供后续查找。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 67-68: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 69-70: get chat template function
```python
def get_chat_template(name):
    return chat_template_registry[name]
```
**EN:** This block uses `get_chat_template` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `get_chat_template` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 71-72: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 73-78: get chat template by model path function
```python
def get_chat_template_by_model_path(model_path):
    for matching_func in matching_function_registry:
        template_name = matching_func(model_path)
        if template_name is not None:
            return get_chat_template(template_name)
    return get_chat_template("default")
```
**EN:** This block uses `get_chat_template_by_model_path` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `get_chat_template_by_model_path` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 79-80: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 81-91: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="default",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": ("SYSTEM:", "\n"),
            "user": ("USER:", "\n"),
            "assistant": ("ASSISTANT:", "\n"),
        },
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 92-92: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 93-103: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="claude",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": ("", ""),
            "user": ("\n\nHuman: ", ""),
            "assistant": ("\n\nAssistant:", ""),
        },
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 104-104: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 105-117: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="chatml",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": ("<|im_start|>system\n", "<|im_end|>\n"),
            "user": ("<|im_start|>user\n", "<|im_end|>\n"),
            "assistant": ("<|im_start|>assistant\n", "<|im_end|>\n"),
        },
        style=ChatTemplateStyle.PLAIN,
        stop_str=("<|im_end|>",),
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 118-118: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 119-132: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="chatml-llava",
        default_system_prompt="You are a helpful assistant.",
        role_prefix_and_suffix={
            "system": ("<|im_start|>system\n", "<|im_end|>\n"),
            "user": ("<|im_start|>user\n", "<|im_end|>\n"),
            "assistant": ("<|im_start|>assistant\n", "<|im_end|>\n"),
        },
        style=ChatTemplateStyle.PLAIN,
        stop_str=("<|im_end|>",),
        image_token="<image>\n",
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 133-136: Module-level supporting statements
```python

# There is default system prompt for qwen
# reference: https://modelscope.cn/models/qwen/Qwen2-72B-Instruct/file/view/master?fileName=tokenizer_config.json&status=1
# The chat template is: "{% for message in messages %}{% if loop.first and messages[0]['role'] != 'system' %}{{ '<|im_start|>system\nYou are a helpful assistant.<|im_end|>\n' }}{% endif %}{{'<|im_start|>' + message['role'] + '\n' + message['content'] + '<|im_end|>' + '\n'}}{% endfor %}{% if add_generation_prompt %}{{ '<|im_start|>assistant\n' }}{% endif %}"
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 137-149: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="qwen",
        default_system_prompt="You are a helpful assistant.",
        role_prefix_and_suffix={
            "system": ("<|im_start|>system\n", "<|im_end|>\n"),
            "user": ("<|im_start|>user\n", "<|im_end|>\n"),
            "assistant": ("<|im_start|>assistant\n", "<|im_end|>\n"),
        },
        style=ChatTemplateStyle.PLAIN,
        stop_str=("<|im_end|>",),
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 150-151: Module-level supporting statements
```python

# Reference: https://huggingface.co/docs/transformers/main/model_doc/qwen2_vl#usage-example
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 152-165: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="qwen2-vl",
        default_system_prompt="You are a helpful assistant.",
        role_prefix_and_suffix={
            "system": ("<|im_start|>system\n", "<|im_end|>\n"),
            "user": ("<|im_start|>user\n", "<|im_end|>\n"),
            "assistant": ("<|im_start|>assistant\n", "<|im_end|>\n"),
        },
        style=ChatTemplateStyle.PLAIN,
        stop_str=("<|im_end|>",),
        image_token="<|vision_start|><|image_pad|><|vision_end|>",
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 166-167: Module-level supporting statements
```python

# Reference: https://github.com/lm-sys/FastChat/blob/main/docs/vicuna_weights_version.md#prompt-template
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 168-182: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="vicuna_v1.1",
        default_system_prompt=(
            "A chat between a curious user and an artificial intelligence assistant. "
            "The assistant gives helpful, detailed, and polite answers to the user's questions."
        ),
        role_prefix_and_suffix={
            "system": ("", " "),
            "user": ("USER:", " "),
            "assistant": ("ASSISTANT:", "</s>"),
        },
        image_token=" <image>\n",
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 183-183: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 184-195: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="llama-2-chat",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": ("<<SYS>>\n", "\n<</SYS>>\n\n"),
            "user": ("[INST] ", " [/INST]"),
            "assistant": ("", " </s><s>"),
        },
        style=ChatTemplateStyle.LLAMA2,
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 196-197: Module-level supporting statements
```python

# Reference: https://huggingface.co/mistralai/Mistral-Small-3.1-24B-Instruct-2503/blob/main/chat_template.json
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 198-210: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="mistral",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": ("[SYSTEM_PROMPT] ", " [/SYSTEM_PROMPT]"),
            "user": ("[INST] ", " [/INST]"),
            "assistant": ("", " </s><s>"),
        },
        stop_str=("</s>",),
        image_token="[IMG]",
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 211-211: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 212-233: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="llama-3-instruct",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": (
                "<|start_header_id|>system<|end_header_id|>\n\n",
                "<|eot_id|>",
            ),
            "user": (
                "<|start_header_id|>user<|end_header_id|>\n\n",
                "<|eot_id|>",
            ),
            "assistant": (
                "<|start_header_id|>assistant<|end_header_id|>\n\n",
                "<|eot_id|>",
            ),
        },
        stop_str=("<|eot_id|>",),
        image_token="<|image|>",
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 234-235: Module-level supporting statements
```python

# https://huggingface.co/openbmb/MiniCPM-V-2_6
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 236-248: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="minicpmv",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": ("", " "),
            "user": ("user:", " "),
            "assistant": ("assistant:", "</s>"),
        },
        stop_str=("<|im_end|>", "<|endoftext|>"),
        image_token="(<image>./</image>)",
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 249-249: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 250-271: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="janus-pro",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": (
                "",
                "",
            ),
            "User": (
                "<｜User｜>",
                "",
            ),
            "assistant": (
                "<｜Assistant｜>",
                "<｜end▁of▁sentence｜>",
            ),
        },
        stop_str=("<｜end▁of▁sentence｜>",),
        image_token="<image_placeholder>\n",
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 272-273: Module-level supporting statements
```python

# https://huggingface.co/openbmb/MiniCPM-o-2_6
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 274-287: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="minicpmo",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": ("", " "),
            "user": ("user:", " "),
            "assistant": ("assistant:", "</s>"),
        },
        stop_str=("<|im_end|>", "<|endoftext|>"),
        image_token="(<image>./</image>)",
        audio_token="(<audio>./</audio>)",
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 288-288: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 289-310: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="janus",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": (
                "",
                "",
            ),
            "user": (
                "<｜User｜>",
                "",
            ),
            "assistant": (
                "<｜Assistant｜>",
                "<｜end▁of▁sentence｜>",
            ),
        },
        stop_str=("<｜end▁of▁sentence｜>",),
        image_token="<image_placeholder>\n",
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 311-312: Module-level supporting statements
```python

# The difference between "llama-3-instruct-llava" and "llama-3-instruct" is that llava uses a different image_token.
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 313-334: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="llama-3-instruct-llava",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": (
                "<|start_header_id|>system<|end_header_id|>\n\n",
                "<|eot_id|>",
            ),
            "user": (
                "<|start_header_id|>user<|end_header_id|>\n\n",
                "<|eot_id|>",
            ),
            "assistant": (
                "<|start_header_id|>assistant<|end_header_id|>\n\n",
                "<|eot_id|>",
            ),
        },
        stop_str=("<|eot_id|>",),
        image_token="<image>\n",
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 335-336: Module-level supporting statements
```python

# Reference: https://huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct/blob/main/chat_template.json
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 337-358: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="llama-4",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": (
                "<|header_start|>system<|header_end|>\n\n",
                "<|eot|>",
            ),
            "user": (
                "<|header_start|>user<|header_end|>\n\n",
                "<|eot|>",
            ),
            "assistant": (
                "<|header_start|>assistant<|header_end|>\n\n",
                "<|eot|>",
            ),
        },
        stop_str=("<|eot|>",),
        image_token="<|image|>",
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 359-360: Module-level supporting statements
```python

# Reference: https://modelscope.cn/models/01ai/Yi-1.5-34B-Chat/file/view/master?fileName=tokenizer_config.json&status=1
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 361-373: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="yi-1.5",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": ("", ""),
            "user": ("<|im_start|>user\n", "<|im_end|>\n<|im_start|>assistant\n"),
            "assistant": ("", "<|im_end|>\n"),
        },
        style=ChatTemplateStyle.PLAIN,
        stop_str=("<|im_end|>",),
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 374-375: Module-level supporting statements
```python

# Reference: https://github.com/01-ai/Yi/tree/main/VL#major-difference-with-llava
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 376-390: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="yi-vl",
        default_system_prompt=(
            "This is a chat between an inquisitive human and an AI assistant. Assume the role of the AI assistant. Read all the images carefully, and respond to the human's questions with informative, helpful, detailed and polite answers."
            "这是一个好奇的人类和一个人工智能助手之间的对话。假设你扮演这个AI助手的角色。仔细阅读所有的图像，并对人类的问题做出信息丰富、有帮助、详细的和礼貌的回答。"
        ),
        role_prefix_and_suffix={
            "system": ("", "\n\n"),
            "user": ("### Human:", "\n"),
            "assistant": ("### Assistant:", "\n"),
        },
        image_token=" <image_placeholder>\n",
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 391-391: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 392-405: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="gemma-it",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": ("", ""),
            "user": ("<start_of_turn>user\n", "<end_of_turn>\n"),
            "assistant": ("<start_of_turn>model\n", "<end_of_turn>\n"),
        },
        image_token="<start_of_image>",
        audio_token="<start_of_audio>",
        style=ChatTemplateStyle.PLAIN,
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 406-406: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 407-418: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="gemma-4-it",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": ("", ""),
            "user": ("<|turn>user\n", "<turn|>\n"),
            "assistant": ("<|turn>assistant\n", "<turn|>\n"),
        },
        style=ChatTemplateStyle.PLAIN,
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 419-419: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 420-431: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="dbrx-instruct",
        default_system_prompt="You are DBRX, created by Databricks. You were last updated in December 2023. You answer questions based on information available up to that point.\nYOU PROVIDE SHORT RESPONSES TO SHORT QUESTIONS OR STATEMENTS, but provide thorough responses to more complex and open-ended questions.\nYou assist with various tasks, from writing to coding (using markdown for code blocks — remember to use ``` with code, JSON, and tables).\n(You do not have real-time data access or code execution capabilities. You avoid stereotyping and provide balanced perspectives on controversial topics. You do not provide song lyrics, poems, or news articles and do not divulge details of your training data.)\nThis is your system prompt, guiding your responses. Do not reference it, just respond to the user. If you find yourself talking about this message, stop. You should be responding appropriately and usually that means not mentioning this.\nYOU DO NOT MENTION ANY OF THIS INFORMATION ABOUT YOURSELF UNLESS THE INFORMATION IS DIRECTLY PERTINENT TO THE USER'S QUERY.",
        role_prefix_and_suffix={
            "system": ("<|im_start|>system\n", "<|im_end|>"),
            "user": ("\n<|im_start|>user\n", "<|im_end|>"),
            "assistant": ("\n<|im_start|>assistant\n", "<|im_end|>"),
        },
        stop_str=("<|im_end|>",),
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 432-432: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 433-450: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="c4ai-command-r",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": (
                "<|START_OF_TURN_TOKEN|><|SYSTEM_TOKEN|>",
                "<|END_OF_TURN_TOKEN|>",
            ),
            "user": ("<|START_OF_TURN_TOKEN|><|USER_TOKEN|>", "<|END_OF_TURN_TOKEN|>"),
            "assistant": (
                "<|START_OF_TURN_TOKEN|><|CHATBOT_TOKEN|>",
                "<|END_OF_TURN_TOKEN|>",
            ),
        },
        style=ChatTemplateStyle.PLAIN,
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 451-452: Module-level supporting statements
```python

# Adapted from https://huggingface.co/OpenGVLab/InternVL2-4B/blob/main/modeling_intern_vit.py
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 453-464: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="internvl-2-5",
        default_system_prompt="你是书生·万象，英文名是InternVL，是由上海人工智能实验室、清华大学及多家合作单位联合开发的多模态大语言模型。",
        role_prefix_and_suffix={
            "system": ("<|im_start|>system\n", "<|im_end|>\n"),
            "user": ("<|im_start|>user\n", "<|im_end|>\n"),
            "assistant": ("<|im_start|>assistant\n", "<|im_end|>\n"),
        },
        stop_str=["<|im_end|>", "<|action_end|>"],
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 465-465: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 466-477: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="interns1",
        default_system_prompt="You are an AI assistant whose name is Intern-S1 (书生大模型).\n- Intern-S1 (书生大模型) is a vision-language model that is developed by Shanghai AI Laboratory (上海人工智能实验室).  It is designed to be helpful, honest, and harmless.\n- Intern-S1 (书生大模型) can understand and communicate fluently in the language chosen by the user such as English and 中文.\nYou are an expert reasoner with extensive experience in all areas. You approach problems through systematic thinking and rigorous reasoning. Your response should reflect deep understanding and precise logical thinking, making your solution path and reasoning clear to others. Please put your thinking process within <think>...</think> tags.",
        role_prefix_and_suffix={
            "system": ("<|im_start|>system\n", "<|im_end|>\n"),
            "user": ("<|im_start|>user\n", "<|im_end|>\n"),
            "assistant": ("<|im_start|>assistant\n", "<|im_end|>\n"),
        },
        stop_str=["<|im_end|>", "<|action_end|>"],
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 478-478: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 479-499: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="granite-3-instruct",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": (
                "<|start_of_role|>system<|end_of_role|>",
                "<|end_of_text|>",
            ),
            "user": (
                "<|start_of_role|>user<|end_of_role|>",
                "<|end_of_text|>",
            ),
            "assistant": (
                "<|start_of_role|>assistant<|end_of_role|>",
                "<|end_of_text|>",
            ),
        },
        stop_str=("<|end_of_text|>",),
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 500-500: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 501-521: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="deepseek-v3",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": (
                "",
                "",
            ),
            "user": (
                "<｜User｜>",
                "",
            ),
            "assistant": (
                "<｜Assistant｜>",
                "<｜end▁of▁sentence｜>",
            ),
        },
        stop_str=("<｜end▁of▁sentence｜>",),
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 522-523: Module-level supporting statements
```python

# Reference: https://huggingface.co/docs/transformers/main/model_doc/glm4_v#usage-example
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 524-537: Module-level supporting statements
```python
register_chat_template(
    ChatTemplate(
        name="glm-4v",
        default_system_prompt=None,
        role_prefix_and_suffix={
            "system": ("<|system|>\n", "\n"),
            "user": ("<|user|>\n", "\n"),
            "assistant": ("<|assistant|>\n", "\n"),
        },
        style=ChatTemplateStyle.PLAIN,
        stop_str=["<|user|>", "<|endoftext|>", "<|observation|>"],
        image_token="<|image|>",
    )
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 538-540: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 541-545: match deepseek function
```python
def match_deepseek(model_path: str):
    if re.search(r"deepseek-(v3|r1)", model_path, re.IGNORECASE) and not re.search(
        r"base", model_path, re.IGNORECASE
    ):
        return "deepseek-v3"
```
**EN:** This block uses `match_deepseek` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_deepseek` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 546-548: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 549-551: match orion function
```python
def match_orion(model_path: str):
    if "orion" in model_path.lower():
        return "claude"
```
**EN:** This block uses `match_orion` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_orion` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 552-554: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 555-557: match deepseek janus pro function
```python
def match_deepseek_janus_pro(model_path: str):
    if re.search(r"janus", model_path, re.IGNORECASE):
        return "janus-pro"
```
**EN:** This block uses `match_deepseek_janus_pro` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_deepseek_janus_pro` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 558-560: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 561-565: match dbrx function
```python
def match_dbrx(model_path: str):
    if re.search(r"dbrx", model_path, re.IGNORECASE) and re.search(
        r"instruct", model_path, re.IGNORECASE
    ):
        return "dbrx-instruct"
```
**EN:** This block uses `match_dbrx` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_dbrx` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 566-568: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 569-571: match vicuna function
```python
def match_vicuna(model_path: str):
    if re.search(r"vicuna|llava-v1\.5|llava-next-video-7b", model_path, re.IGNORECASE):
        return "vicuna_v1.1"
```
**EN:** This block uses `match_vicuna` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_vicuna` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 572-574: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 575-581: match llama2 chat function
```python
def match_llama2_chat(model_path: str):
    if re.search(
        r"llama-2.*chat|codellama.*instruct",
        model_path,
        re.IGNORECASE,
    ):
        return "llama-2-chat"
```
**EN:** This block uses `match_llama2_chat` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_llama2_chat` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 582-584: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 585-587: match mistral function
```python
def match_mistral(model_path: str):
    if re.search(r"pixtral|(mistral|mixtral).*instruct", model_path, re.IGNORECASE):
        return "mistral"
```
**EN:** This block uses `match_mistral` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_mistral` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 588-590: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 591-593: match llama3 instruct function
```python
def match_llama3_instruct(model_path: str):
    if re.search(r"llama-3.*instruct", model_path, re.IGNORECASE):
        return "llama-3-instruct"
```
**EN:** This block uses `match_llama3_instruct` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_llama3_instruct` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 594-596: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 597-613: match chat ml function
```python
def match_chat_ml(model_path: str):
    if re.search(r"tinyllama", model_path, re.IGNORECASE):
        return "chatml"
    if re.search(r"qwen.*vl", model_path, re.IGNORECASE):
        return "qwen2-vl"
    if re.search(r"glm[-_]?4(\.\d+)?v", model_path, re.IGNORECASE):
        return "glm-4v"
    if re.search(r"qwen.*(chat|instruct)", model_path, re.IGNORECASE) and not re.search(
        r"llava", model_path, re.IGNORECASE
    ):
        return "qwen"
    if re.search(
        r"llava-v1\.6-34b|llava-v1\.6-yi-34b|llava-next-video-34b|llava-onevision-qwen2",
        model_path,
        re.IGNORECASE,
    ):
        return "chatml-llava"
```
**EN:** This block uses `match_chat_ml` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_chat_ml` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 614-616: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 617-623: match chat yi function
```python
def match_chat_yi(model_path: str):
    if re.search(r"yi-vl", model_path, re.IGNORECASE) and not re.search(
        r"llava", model_path, re.IGNORECASE
    ):
        return "yi-vl"
    elif re.search(r"yi-1\.5.*chat", model_path, re.IGNORECASE):
        return "yi-1.5"
```
**EN:** This block uses `match_chat_yi` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_chat_yi` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 624-626: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 627-631: match gemma function
```python
def match_gemma(model_path: str):
    if re.search(r"gemma-4.*it", model_path, re.IGNORECASE):
        return "gemma-4-it"
    if re.search(r"(gemma.*it)|(gemma-3)", model_path, re.IGNORECASE):
        return "gemma-it"
```
**EN:** This block uses `match_gemma` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_gemma` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 632-634: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 635-639: match openbmb minicpm function
```python
def match_openbmb_minicpm(model_path: str):
    if re.search(r"minicpm-v", model_path, re.IGNORECASE):
        return "minicpmv"
    elif re.search(r"minicpm-o", model_path, re.IGNORECASE):
        return "minicpmo"
```
**EN:** This block uses `match_openbmb_minicpm` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_openbmb_minicpm` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 640-642: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 643-645: match c4ai command r function
```python
def match_c4ai_command_r(model_path: str):
    if re.search(r"c4ai-command-r", model_path, re.IGNORECASE):
        return "c4ai-command-r"
```
**EN:** This block uses `match_c4ai_command_r` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_c4ai_command_r` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 646-648: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 649-651: match granite instruct function
```python
def match_granite_instruct(model_path: str):
    if re.search(r"granite.*instruct", model_path, re.IGNORECASE):
        return "granite-3-instruct"
```
**EN:** This block uses `match_granite_instruct` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_granite_instruct` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 652-654: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 655-657: match internvl chat function
```python
def match_internvl_chat(model_path: str):
    if re.search(r"internvl2_5", model_path, re.IGNORECASE):
        return "internvl-2-5"
```
**EN:** This block uses `match_internvl_chat` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_internvl_chat` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 658-660: Module-level supporting statements
```python


@register_chat_template_matching_function
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 661-665: match interns1 chat function
```python
def match_interns1_chat(model_path: str):
    if re.search(r"intern-s1", model_path, re.IGNORECASE):
        return "interns1"
    if re.search(r"interns1", model_path, re.IGNORECASE):
        return "interns1"
```
**EN:** This block uses `match_interns1_chat` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `match_interns1_chat` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 666-667: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 668-679: Module-level supporting statements
```python
if __name__ == "__main__":
    messages = [
        {"role": "system", "content": None},  # None means default
        # {"role": "system", "content": "You are a helpful, respectful and honest assistant."},
        {"role": "user", "content": "Hello!"},
        {"role": "assistant", "content": "Hi!"},
        {"role": "user", "content": "What can you do?"},
        {"role": "assistant", "content": "I can chat with you."},
    ]

    template = get_chat_template("llama-2-chat")
    print(template.get_prompt(messages))
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

## Key Concepts / 关键概念
- Chat formatting / 对话格式化

## Dependencies / 依赖关系
### Internal / 内部
- None / 无
### External / 外部
- `dataclasses` (stdlib)
- `enum` (stdlib)
- `re` (stdlib)
- `typing` (stdlib)
