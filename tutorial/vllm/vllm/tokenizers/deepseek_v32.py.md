# deepseek_v32.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tokenizers/deepseek_v32.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `deepseek_v32`-related logic centered around `DeepseekV32Tokenizer`, `get_deepseek_v32_tokenizer`. / 实现与 `deepseek_v32` 相关的逻辑，核心符号包括 `DeepseekV32Tokenizer`, `get_deepseek_v32_tokenizer`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-12)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import copy
from typing import Any

from transformers import PreTrainedTokenizerFast

from vllm.entrypoints.chat_utils import ChatCompletionMessageParam

from .deepseek_v32_encoding import encode_messages
from .hf import HfTokenizer, get_cached_tokenizer
from .protocol import TokenizerLike
```
**EN:** Sets up the module with standard-library support such as `copy`, `typing`, external packages such as `transformers`, vLLM modules such as `vllm.entrypoints.chat_utils`, `.deepseek_v32_encoding`, `.hf`. It prepares the symbols later used by `DeepseekV32Tokenizer`, `get_deepseek_v32_tokenizer`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.entrypoints.chat_utils`, `.deepseek_v32_encoding`, `.hf` 等 vLLM 内部依赖。 这些准备工作为后续的 `DeepseekV32Tokenizer`, `get_deepseek_v32_tokenizer` 提供上下文。

### get_deepseek_v32_tokenizer (lines 15-82)
```python
def get_deepseek_v32_tokenizer(tokenizer: HfTokenizer) -> HfTokenizer:
    """
    Wraps a tokenizer to use the custom DeepSeek V3.2 chat template encoding.
    """
    dsv32_tokenizer = copy.copy(tokenizer)

    added_vocab = tokenizer.get_added_vocab()
    added_vocab_size = len(added_vocab)
    tokenizer_vocab_size = tokenizer.vocab_size

    class _DeepseekV32Tokenizer(tokenizer.__class__):  # type: ignore
        def apply_chat_template(
            self,
            messages: list["ChatCompletionMessageParam"],
            tools: list[dict[str, Any]] | None = None,
            **kwargs,
        ) -> str | list[int]:
            thinking = kwargs.get("thinking", False)
            enable_thinking = kwargs.get("enable_thinking", False)
            thinking = thinking or enable_thinking
            thinking_mode = "thinking"
            if not thinking:
                thinking_mode = "chat"
            conversation = kwargs.get("conversation", messages)
    # ...
            return get_deepseek_v32_tokenizer, (tokenizer,)

    _DeepseekV32Tokenizer.__name__ = f"DSV32{tokenizer.__class__.__name__}"

    dsv32_tokenizer.__class__ = _DeepseekV32Tokenizer
    return dsv32_tokenizer
```
**EN:** `get_deepseek_v32_tokenizer`: Wraps a tokenizer to use the custom DeepSeek V3.2 chat template encoding. It mainly works with `tokenizer`. Inside the body, it relies on `copy.copy`, `tokenizer.get_added_vocab`, `kwargs.get` to complete the main steps.
**CN:** `get_deepseek_v32_tokenizer` 负责获取流水线所需的数据或状态。 它主要处理 `tokenizer` 等参数。 实现过程中会调用 `copy.copy`, `tokenizer.get_added_vocab`, `kwargs.get` 等函数完成关键步骤。

### DeepseekV32Tokenizer (lines 85-89)
```python
class DeepseekV32Tokenizer(TokenizerLike):
    @classmethod
    def from_pretrained(cls, *args, **kwargs) -> HfTokenizer:
        tokenizer = PreTrainedTokenizerFast.from_pretrained(*args, **kwargs)
        return get_cached_tokenizer(get_deepseek_v32_tokenizer(tokenizer))
```
**EN:** Defines the `DeepseekV32Tokenizer` class used by this module. It extends `TokenizerLike`. Key methods include `from_pretrained`.
**CN:** `DeepseekV32Tokenizer` 是该文件中的核心类，用于封装与 `DeepseekV32Tokenizer` 相关的状态和行为。 它继承自 `TokenizerLike`。 关键方法包括 `from_pretrained`。

## Key Concepts / 关键概念
- **`DeepseekV32Tokenizer`**: Core class that organizes module behavior. / **`DeepseekV32Tokenizer`**：组织模块行为的核心类。
- **`get_deepseek_v32_tokenizer`**: Key helper or entry point in this file. / **`get_deepseek_v32_tokenizer`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: copy, typing
- **Third-party / 第三方**: transformers
- **Internal vLLM / vLLM 内部依赖**: vllm.entrypoints.chat_utils, .deepseek_v32_encoding, .hf, .protocol
