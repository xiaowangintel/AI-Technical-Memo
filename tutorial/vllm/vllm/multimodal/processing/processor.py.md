# processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/processing/processor.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides preprocessing abstractions that turn multimodal inputs into model-ready data structures. / 提供把多模态输入转换为模型可用数据结构的预处理抽象。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-66)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from abc import ABC, abstractmethod
from collections import defaultdict
from collections.abc import Callable, Generator, ItemsView, Iterable, Mapping, Sequence
from dataclasses import dataclass, field, replace
from enum import Enum
from functools import lru_cache, partial
from typing import (
    TYPE_CHECKING,
    Generic,
    NamedTuple,
    Protocol,
    TypeAlias,
    cast,
)

import regex as re
import torch
from typing_extensions import TypeVar, assert_never

from vllm.inputs import (
    MultiModalEncDecInput,
    MultiModalHashes,
    # ...

_S = TypeVar("_S", str, list[int])


PromptSeq: TypeAlias = str | list[int]
"""A token sequence (list of token IDs) or text."""
```
**EN:** Sets up the module with standard-library support such as `abc`, `collections`, `collections.abc`, external packages such as `regex`, `torch`, `typing_extensions`, vLLM modules such as `vllm.inputs`, `vllm.logger`, `vllm.tokenizers`. It prepares the symbols later used by `_GetMatchIndex`, `PromptIndex`, `_cached_encode`, `_cached_decode`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.inputs`, `vllm.logger`, `vllm.tokenizers` 等 vLLM 内部依赖。 这些准备工作为后续的 `_GetMatchIndex`, `PromptIndex`, `_cached_encode`, `_cached_decode` 提供上下文。

### PromptIndexTargets overview (lines 141-186)
```python
class PromptIndexTargets:
    @staticmethod
    def start() -> PromptIndex:
        """
        Resolves to the start of the prompt (before the first token).

        This results in a match even if the prompt is empty.
        """
        return PromptIndex(lambda tokenizer, prompt, start_idx=0: 0)

    @staticmethod
    def prefix(seq: PromptSeq) -> PromptIndex:
        """
        Resolves to a location in the prompt after the given prefix.
        """

        def get_match_index(
            tokenizer: TokenizerLike | None,
            prompt: PromptSeq,
            start_idx: int = 0,
        ) -> int | None:
            if start_idx != 0:
                return None

            prefix = seq

            if isinstance(prompt, str):
    # ...
```
**EN:** Defines the `PromptIndexTargets` class used by this module. Key methods include `start`, `prefix`, `end`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `PromptIndexTargets` 是该文件中的核心类，用于封装与 `PromptIndexTargets` 相关的状态和行为。 关键方法包括 `start`, `prefix`, `end`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### PromptIndexTargets.prefix (lines 152-177)
```python
    def prefix(seq: PromptSeq) -> PromptIndex:
        """
        Resolves to a location in the prompt after the given prefix.
        """

        def get_match_index(
            tokenizer: TokenizerLike | None,
            prompt: PromptSeq,
            start_idx: int = 0,
        ) -> int | None:
            if start_idx != 0:
                return None

            prefix = seq

            if isinstance(prompt, str):
                # Make both `str`
                prefix = _seq2text(tokenizer, prefix, use_cache=False)
            else:
                # Make both `list[int]`
                prefix = _seq2tokens(tokenizer, prefix, use_cache=False)

            match_idx = len(prefix)
            return match_idx if prompt[:match_idx] == prefix else None

        return PromptIndex(get_match_index)
```
**EN:** `prefix`: Resolves to a location in the prompt after the given prefix. It mainly works with `seq`. Inside the body, it relies on `PromptIndex`, `_seq2text`, `_seq2tokens` to complete the main steps.
**CN:** `prefix` 负责实现本模块使用的辅助逻辑。 它主要处理 `seq` 等参数。 实现过程中会调用 `PromptIndex`, `_seq2text`, `_seq2tokens` 等函数完成关键步骤。

### PromptIndexTargets.start (lines 143-149)
```python
    def start() -> PromptIndex:
        """
        Resolves to the start of the prompt (before the first token).

        This results in a match even if the prompt is empty.
        """
        return PromptIndex(lambda tokenizer, prompt, start_idx=0: 0)
```
**EN:** `start`: Resolves to the start of the prompt (before the first token). Inside the body, it relies on `PromptIndex` to complete the main steps.
**CN:** `start` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `PromptIndex` 等函数完成关键步骤。

### PromptIndexTargets.end (lines 180-186)
```python
    def end() -> PromptIndex:
        """
        Resolves to the end of the prompt (after the last token).

        This results in a match even if the prompt is empty.
        """
        return PromptIndex(lambda tokenizer, prompt, start_idx=0: len(prompt))
```
**EN:** `end`: Resolves to the end of the prompt (after the last token). Inside the body, it relies on `PromptIndex` to complete the main steps.
**CN:** `end` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `PromptIndex` 等函数完成关键步骤。

### PromptUpdateDetails overview (lines 206-269)
```python
class PromptUpdateDetails(Generic[_S]):
    """Details about the token sequence or text that are part of the update."""

    full: _S
    """The full content."""

    is_embed: Callable[[TokenizerLike | None, PromptSeq], torch.Tensor] | None = None
    """
    Given [`full`][vllm.multimodal.processing.PromptUpdateDetails.full],
    return a boolean mask of shape `(len(full),)` indicating which positions
    of `full` to assign embeddings to.

    `None` (default) means to assign embeddings to all positions of `full`.

    The embeddings are obtained by calling
    [`SupportsMultiModal.embed_multimodal`][vllm.model_executor.models.interfaces.SupportsMultiModal.embed_multimodal].
    """

    @staticmethod
    def from_seq(seq: _S) -> "PromptUpdateDetails[_S]":
        return PromptUpdateDetails(full=seq)

    @staticmethod
    def select_text(
        seq: _S,
        embed_text: str,
    ) -> "PromptUpdateDetails[_S]":
    # ...
```
**EN:** `PromptUpdateDetails`: Details about the token sequence or text that are part of the update. It extends `Generic`. Key methods include `from_seq`, `select_text`, `select_token_id`, `select_token_ids`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `PromptUpdateDetails` 是该文件中的核心类，用于封装与 `PromptUpdateDetails` 相关的状态和行为。 它继承自 `Generic`。 关键方法包括 `from_seq`, `select_text`, `select_token_id`, `select_token_ids`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### PromptUpdateDetails.select_text (lines 229-242)
```python
    def select_text(
        seq: _S,
        embed_text: str,
    ) -> "PromptUpdateDetails[_S]":
        def is_embed(tokenizer: TokenizerLike | None, full: PromptSeq) -> torch.Tensor:
            embed_token_ids = _seq2tokens(tokenizer, embed_text, use_cache=False)
            token_ids = _seq2tokens(tokenizer, full)

            return torch.isin(
                torch.tensor(token_ids),
                torch.tensor(embed_token_ids),
            )

        return PromptUpdateDetails(full=seq, is_embed=is_embed)
```
**EN:** `select_text` implements helper logic used by this module. It mainly works with `seq`, `embed_text`. Inside the body, it relies on `PromptUpdateDetails`, `_seq2tokens`, `torch.isin` to complete the main steps.
**CN:** `select_text` 负责实现本模块使用的辅助逻辑。 它主要处理 `seq`, `embed_text` 等参数。 实现过程中会调用 `PromptUpdateDetails`, `_seq2tokens`, `torch.isin` 等函数完成关键步骤。

### PromptUpdateDetails.select_token_ids (lines 257-269)
```python
    def select_token_ids(
        seq: _S,
        embed_token_ids: list[int],
    ) -> "PromptUpdateDetails[_S]":
        def is_embed(tokenizer: TokenizerLike | None, full: PromptSeq) -> torch.Tensor:
            token_ids = _seq2tokens(tokenizer, full)

            return torch.isin(
                torch.tensor(token_ids),
                torch.tensor(embed_token_ids),
            )

        return PromptUpdateDetails(full=seq, is_embed=is_embed)
```
**EN:** `select_token_ids` implements helper logic used by this module. It mainly works with `seq`, `embed_token_ids`. Inside the body, it relies on `PromptUpdateDetails`, `_seq2tokens`, `torch.isin` to complete the main steps.
**CN:** `select_token_ids` 负责实现本模块使用的辅助逻辑。 它主要处理 `seq`, `embed_token_ids` 等参数。 实现过程中会调用 `PromptUpdateDetails`, `_seq2tokens`, `torch.isin` 等函数完成关键步骤。

### PromptUpdateDetails.select_token_id (lines 245-254)
```python
    def select_token_id(
        seq: _S,
        embed_token_id: int,
    ) -> "PromptUpdateDetails[_S]":
        def is_embed(tokenizer: TokenizerLike | None, full: PromptSeq) -> torch.Tensor:
            token_ids = _seq2tokens(tokenizer, full)

            return torch.tensor(token_ids) == embed_token_id

        return PromptUpdateDetails(full=seq, is_embed=is_embed)
```
**EN:** `select_token_id` implements helper logic used by this module. It mainly works with `seq`, `embed_token_id`. Inside the body, it relies on `PromptUpdateDetails`, `_seq2tokens`, `torch.tensor` to complete the main steps.
**CN:** `select_token_id` 负责实现本模块使用的辅助逻辑。 它主要处理 `seq`, `embed_token_id` 等参数。 实现过程中会调用 `PromptUpdateDetails`, `_seq2tokens`, `torch.tensor` 等函数完成关键步骤。

### PromptUpdateDetails.from_seq (lines 225-226)
```python
    def from_seq(seq: _S) -> "PromptUpdateDetails[_S]":
        return PromptUpdateDetails(full=seq)
```
**EN:** `from_seq` implements helper logic used by this module. It mainly works with `seq`. Inside the body, it relies on `PromptUpdateDetails` to complete the main steps.
**CN:** `from_seq` 负责实现本模块使用的辅助逻辑。 它主要处理 `seq` 等参数。 实现过程中会调用 `PromptUpdateDetails` 等函数完成关键步骤。

### Additional repeated patterns / 其他重复模式
```python
# Large file omitted for brevity
# ...
```
**EN:** The remaining code mostly expands the same abstractions with more wrappers, constants, or schema variants. The analysis above focuses on the control points that shape overall behavior.
**CN:** 剩余代码主要在相同抽象之上继续展开，补充更多包装函数、常量或模式变体。上面的分析聚焦于决定整体行为的关键控制点。

## Key Concepts / 关键概念
- **`_GetMatchIndex`**: Core class that organizes module behavior. / **`_GetMatchIndex`**：组织模块行为的核心类。
- **`PromptIndex`**: Core class that organizes module behavior. / **`PromptIndex`**：组织模块行为的核心类。
- **`PromptIndexTargets`**: Core class that organizes module behavior. / **`PromptIndexTargets`**：组织模块行为的核心类。
- **`_cached_encode`**: Key helper or entry point in this file. / **`_cached_encode`**：本文件中的关键辅助函数或入口。
- **`_cached_decode`**: Key helper or entry point in this file. / **`_cached_decode`**：本文件中的关键辅助函数或入口。
- **`_seq2text`**: Key helper or entry point in this file. / **`_seq2text`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: abc, collections, collections.abc, dataclasses, enum, functools, typing
- **Third-party / 第三方**: regex, torch, typing_extensions, transformers.feature_extraction_utils
- **Internal vLLM / vLLM 内部依赖**: vllm.inputs, vllm.logger, vllm.tokenizers, vllm.transformers_utils.processor, vllm.utils.collection_utils, ..inputs, ..parse, .context, .dummy_inputs, .inputs, ..cache
