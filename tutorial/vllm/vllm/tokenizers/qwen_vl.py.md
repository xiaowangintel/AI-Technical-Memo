# qwen_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tokenizers/qwen_vl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `qwen_vl`-related logic centered around `QwenVLTokenizer`, `get_qwen_vl_tokenizer`. / 实现与 `qwen_vl` 相关的逻辑，核心符号包括 `QwenVLTokenizer`, `get_qwen_vl_tokenizer`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-10)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import copy
import unicodedata
from collections.abc import Collection, Set

from transformers import AutoTokenizer

from .hf import HfTokenizer, get_cached_tokenizer
from .protocol import TokenizerLike
```
**EN:** Sets up the module with standard-library support such as `copy`, `unicodedata`, `collections.abc`, external packages such as `transformers`, vLLM modules such as `.hf`, `.protocol`. It prepares the symbols later used by `QwenVLTokenizer`, `get_qwen_vl_tokenizer`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `.hf`, `.protocol` 等 vLLM 内部依赖。 这些准备工作为后续的 `QwenVLTokenizer`, `get_qwen_vl_tokenizer` 提供上下文。

### get_qwen_vl_tokenizer (lines 13-60)
```python
def get_qwen_vl_tokenizer(tokenizer: HfTokenizer) -> HfTokenizer:
    """
    The logic of adding image pad tokens should only be applied in
    `QwenVLProcessor`, so they are patched out here.

    The definition of the wrapped tokenizer can be found here:
    https://huggingface.co/Qwen/Qwen-VL/blob/main/tokenization_qwen.py
    """
    new_tokenizer = copy.copy(tokenizer)

    class TokenizerWithoutImagePad(tokenizer.__class__):  # type: ignore
        def tokenize(
            self,
            text: str,
            allowed_special: Set[str] | str = "all",
            disallowed_special: Collection[str] | str = (),
            **kwargs,
        ) -> list[bytes | str]:
            text = unicodedata.normalize("NFC", text)

            return [
                self.decoder[t]
                for t in self.tokenizer.encode(
                    text,
    # ...
            )

    TokenizerWithoutImagePad.__name__ = f"{tokenizer.__class__.__name__}WithoutImagePad"

    new_tokenizer.__class__ = TokenizerWithoutImagePad
    return new_tokenizer
```
**EN:** `get_qwen_vl_tokenizer`: The logic of adding image pad tokens should only be applied in `QwenVLProcessor`, so they are patched out here. It mainly works with `tokenizer`. Inside the body, it relies on `copy.copy`, `unicodedata.normalize`, `self.tokenizer.decode` to complete the main steps.
**CN:** `get_qwen_vl_tokenizer` 负责获取流水线所需的数据或状态。 它主要处理 `tokenizer` 等参数。 实现过程中会调用 `copy.copy`, `unicodedata.normalize`, `self.tokenizer.decode` 等函数完成关键步骤。

### QwenVLTokenizer (lines 63-71)
```python
class QwenVLTokenizer(TokenizerLike):
    image_start_tag: str
    image_end_tag: str
    image_pad_tag: str

    @classmethod
    def from_pretrained(cls, *args, **kwargs) -> HfTokenizer:
        tokenizer = AutoTokenizer.from_pretrained(*args, **kwargs)
        return get_cached_tokenizer(get_qwen_vl_tokenizer(tokenizer))
```
**EN:** Defines the `QwenVLTokenizer` class used by this module. It extends `TokenizerLike`. Key methods include `from_pretrained`.
**CN:** `QwenVLTokenizer` 是该文件中的核心类，用于封装与 `QwenVLTokenizer` 相关的状态和行为。 它继承自 `TokenizerLike`。 关键方法包括 `from_pretrained`。

## Key Concepts / 关键概念
- **`QwenVLTokenizer`**: Core class that organizes module behavior. / **`QwenVLTokenizer`**：组织模块行为的核心类。
- **`get_qwen_vl_tokenizer`**: Key helper or entry point in this file. / **`get_qwen_vl_tokenizer`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: copy, unicodedata, collections.abc
- **Third-party / 第三方**: transformers
- **Internal vLLM / vLLM 内部依赖**: .hf, .protocol
