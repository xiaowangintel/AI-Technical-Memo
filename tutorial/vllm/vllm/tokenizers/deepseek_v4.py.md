# deepseek_v4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tokenizers/deepseek_v4.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `deepseek_v4`-related logic centered around `DeepseekV4Tokenizer`, `get_deepseek_v4_tokenizer`. / 实现与 `deepseek_v4` 相关的逻辑，核心符号包括 `DeepseekV4Tokenizer`, `get_deepseek_v4_tokenizer`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-12)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import copy
from typing import Any

from transformers import PreTrainedTokenizerFast

from vllm.entrypoints.chat_utils import ChatCompletionMessageParam

from .deepseek_v4_encoding import encode_messages
from .hf import HfTokenizer, get_cached_tokenizer
from .protocol import TokenizerLike
```
**EN:** Sets up the module with standard-library support such as `copy`, `typing`, external packages such as `transformers`, vLLM modules such as `vllm.entrypoints.chat_utils`, `.deepseek_v4_encoding`, `.hf`. It prepares the symbols later used by `DeepseekV4Tokenizer`, `get_deepseek_v4_tokenizer`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.entrypoints.chat_utils`, `.deepseek_v4_encoding`, `.hf` 等 vLLM 内部依赖。 这些准备工作为后续的 `DeepseekV4Tokenizer`, `get_deepseek_v4_tokenizer` 提供上下文。

### get_deepseek_v4_tokenizer (lines 15-89)
```python
def get_deepseek_v4_tokenizer(tokenizer: HfTokenizer) -> HfTokenizer:
    """
    Wraps a tokenizer to use the custom DeepSeek V4 chat template encoding.
    """
    dsv4_tokenizer = copy.copy(tokenizer)

    added_vocab = tokenizer.get_added_vocab()
    added_vocab_size = len(added_vocab)
    tokenizer_vocab_size = tokenizer.vocab_size

    class _DeepseekV4Tokenizer(tokenizer.__class__):  # type: ignore
        def apply_chat_template(
            self,
            messages: list["ChatCompletionMessageParam"],
            tools: list[dict[str, Any]] | None = None,
            **kwargs,
        ) -> str | list[int]:
            thinking = kwargs.get("thinking", False)
            enable_thinking = kwargs.get("enable_thinking", False)
            thinking = thinking or enable_thinking
            thinking_mode = "thinking" if thinking else "chat"

            conversation = kwargs.get("conversation", messages)
            messages = conversation.copy()
    # ...
            return get_deepseek_v4_tokenizer, (tokenizer,)

    _DeepseekV4Tokenizer.__name__ = f"DSV4{tokenizer.__class__.__name__}"

    dsv4_tokenizer.__class__ = _DeepseekV4Tokenizer
    return dsv4_tokenizer
```
**EN:** `get_deepseek_v4_tokenizer`: Wraps a tokenizer to use the custom DeepSeek V4 chat template encoding. It mainly works with `tokenizer`. Inside the body, it relies on `copy.copy`, `tokenizer.get_added_vocab`, `kwargs.get` to complete the main steps.
**CN:** `get_deepseek_v4_tokenizer` 负责获取流水线所需的数据或状态。 它主要处理 `tokenizer` 等参数。 实现过程中会调用 `copy.copy`, `tokenizer.get_added_vocab`, `kwargs.get` 等函数完成关键步骤。

### DeepseekV4Tokenizer (lines 92-96)
```python
class DeepseekV4Tokenizer(TokenizerLike):
    @classmethod
    def from_pretrained(cls, *args, **kwargs) -> HfTokenizer:
        tokenizer = PreTrainedTokenizerFast.from_pretrained(*args, **kwargs)
        return get_cached_tokenizer(get_deepseek_v4_tokenizer(tokenizer))
```
**EN:** Defines the `DeepseekV4Tokenizer` class used by this module. It extends `TokenizerLike`. Key methods include `from_pretrained`.
**CN:** `DeepseekV4Tokenizer` 是该文件中的核心类，用于封装与 `DeepseekV4Tokenizer` 相关的状态和行为。 它继承自 `TokenizerLike`。 关键方法包括 `from_pretrained`。

## Key Concepts / 关键概念
- **`DeepseekV4Tokenizer`**: Core class that organizes module behavior. / **`DeepseekV4Tokenizer`**：组织模块行为的核心类。
- **`get_deepseek_v4_tokenizer`**: Key helper or entry point in this file. / **`get_deepseek_v4_tokenizer`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: copy, typing
- **Third-party / 第三方**: transformers
- **Internal vLLM / vLLM 内部依赖**: vllm.entrypoints.chat_utils, .deepseek_v4_encoding, .hf, .protocol
