# detokenizer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/engine/detokenizer.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `IncrementalDetokenizer`, `BaseIncrementalDetokenizer`, `FastIncrementalDetokenizer` for the V1 `engine` subsystem. / 为 V1 的 `engine` 子系统实现 `IncrementalDetokenizer`, `BaseIncrementalDetokenizer`, `FastIncrementalDetokenizer`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from abc import ABC, abstractmethod

import tokenizers
import tokenizers.decoders
from packaging import version
from tokenizers import Tokenizer
from transformers import PreTrainedTokenizerFast

from vllm.logger import init_logger
from vllm.tokenizers import TokenizerLike
from vllm.tokenizers.detokenizer_utils import (
    convert_prompt_ids_to_tokens,
    detokenize_incrementally,
)
from vllm.utils import length_from_prompt_token_ids_or_embeds
from vllm.v1.engine import EngineCoreRequest

logger = init_logger(__name__)

# Only tokenizers >= 0.22.0 supports DecodeStream with native prefill
# (ids parameter) used for FastIncrementalDetokenizer.
USE_FAST_DETOKENIZER = version.parse(tokenizers.__version__) >= version.parse("0.22.0")

# Error string from https://github.com/huggingface/tokenizers/blob/909fdde2a4ffedd9295206f705eb612be2a91b12/tokenizers/src/tokenizer/mod.rs#L1042
INVALID_PREFIX_ERR_MSG = "Invalid prefix encountered"
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `USE_FAST_DETOKENIZER`, `INVALID_PREFIX_ERR_MSG`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `USE_FAST_DETOKENIZER`, `INVALID_PREFIX_ERR_MSG`。

### `IncrementalDetokenizer` class / `IncrementalDetokenizer` 类
```python
class IncrementalDetokenizer:
```
**EN:** Introduces the `IncrementalDetokenizer` class. Core methods include `__init__`, `output_token_ids`, `num_output_tokens`, `update`, `get_next_output_text`, `from_new_request`.
**CN:** 这里定义 `IncrementalDetokenizer` 类。核心方法包括 `__init__`, `output_token_ids`, `num_output_tokens`, `update`, `get_next_output_text`, `from_new_request`。

### `IncrementalDetokenizer.__init__` method / `IncrementalDetokenizer.__init__` 方法
```python
    def __init__(self):
        self.token_ids: list[int] = []
```
**EN:** This method initializes the object state within `IncrementalDetokenizer`. It touches state such as `token_ids`.
**CN:** 该方法会初始化对象状态，其作用域位于`IncrementalDetokenizer`。 它会读写 `token_ids` 等状态。

### `IncrementalDetokenizer.output_token_ids` method / `IncrementalDetokenizer.output_token_ids` 方法
```python
    @property
    def output_token_ids(self) -> list[int]:
        return self.token_ids
```
**EN:** This method implements `output_token_ids` within `IncrementalDetokenizer`.
**CN:** 该方法会实现 `output_token_ids`，其作用域位于`IncrementalDetokenizer`。

### `IncrementalDetokenizer.num_output_tokens` method / `IncrementalDetokenizer.num_output_tokens` 方法
```python
    def num_output_tokens(self) -> int:
        return len(self.token_ids)
```
**EN:** This method implements `num_output_tokens` within `IncrementalDetokenizer`. Key calls include `len`.
**CN:** 该方法会实现 `num_output_tokens`，其作用域位于`IncrementalDetokenizer`。 关键调用包括 `len`。

### `IncrementalDetokenizer.update` method / `IncrementalDetokenizer.update` 方法
```python
    def update(self, new_token_ids: list[int], stop_terminated: bool) -> str | None:
        self.token_ids.extend(new_token_ids)
        return None
```
**EN:** This method updates existing state within `IncrementalDetokenizer`. Key calls include `extend`.
**CN:** 该方法会更新现有状态，其作用域位于`IncrementalDetokenizer`。 关键调用包括 `extend`。

### `IncrementalDetokenizer.get_next_output_text` method / `IncrementalDetokenizer.get_next_output_text` 方法
```python
    def get_next_output_text(self, finished: bool, delta: bool) -> str:
        return ""
```
**EN:** This method returns or derives a value within `IncrementalDetokenizer`.
**CN:** 该方法会返回或推导一个值，其作用域位于`IncrementalDetokenizer`。

### `IncrementalDetokenizer.from_new_request` method / `IncrementalDetokenizer.from_new_request` 方法
```python
    @classmethod
    def from_new_request(
        cls,
        tokenizer: TokenizerLike | None,
        request: EngineCoreRequest,
    ) -> "IncrementalDetokenizer":
        assert request.sampling_params is not None

        if tokenizer is None:
            # No tokenizer => skipping detokenization.
            return IncrementalDetokenizer()

        if USE_FAST_DETOKENIZER and isinstance(tokenizer, PreTrainedTokenizerFast):
            # Fast tokenizer => use tokenizers library DecodeStream.
            return FastIncrementalDetokenizer(tokenizer, request)

        # Fall back to slow python-based incremental detokenization.
        return SlowIncrementalDetokenizer(tokenizer, request)
```
**EN:** This method reconstructs data from another representation within `IncrementalDetokenizer`. Key calls include `SlowIncrementalDetokenizer`, `IncrementalDetokenizer`, `isinstance`, `FastIncrementalDetokenizer`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会从另一种表示恢复数据，其作用域位于`IncrementalDetokenizer`。 关键调用包括 `SlowIncrementalDetokenizer`, `IncrementalDetokenizer`, `isinstance`, `FastIncrementalDetokenizer`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `BaseIncrementalDetokenizer` class / `BaseIncrementalDetokenizer` 类
```python
class BaseIncrementalDetokenizer(IncrementalDetokenizer, ABC):
```
**EN:** Declares the `BaseIncrementalDetokenizer` interface. Downstream implementations are expected to provide methods such as `__init__`, `update`, `decode_next`, `get_next_output_text`.
**CN:** `BaseIncrementalDetokenizer` 声明了一组接口约定。下游实现需要提供 `__init__`, `update`, `decode_next`, `get_next_output_text` 等方法。

### `BaseIncrementalDetokenizer.__init__` method / `BaseIncrementalDetokenizer.__init__` 方法
```python
    def __init__(self, request: EngineCoreRequest):
        super().__init__()

        # Stop strings
        params = request.sampling_params
        assert params is not None
        if params.stop is None:
            self.stop = []
        elif isinstance(params.stop, str):
            self.stop = [params.stop]
        else:
            self.stop = params.stop
        self.min_tokens = params.min_tokens
        self.include_stop_str_in_output = params.include_stop_str_in_output

        # Number of chars to hold back when stop strings are to be excluded
        # from streamed output.
        if self.stop and not self.include_stop_str_in_output:
            self.stop_buffer_length = max(len(s) for s in self.stop) - 1
        else:
            self.stop_buffer_length = 0
        self._last_output_text_offset: int = 0

        # Generation data
        self.output_text = ""
```
**EN:** This method initializes the object state within `BaseIncrementalDetokenizer`. Key calls include `__init__`, `isinstance`, `super`, `max`, `len`. It touches state such as `min_tokens`, `include_stop_str_in_output`, `_last_output_text_offset`, `output_text`, `stop`, `stop_buffer_length`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`BaseIncrementalDetokenizer`。 关键调用包括 `__init__`, `isinstance`, `super`, `max`, `len`。 它会读写 `min_tokens`, `include_stop_str_in_output`, `_last_output_text_offset`, `output_text`, `stop`, `stop_buffer_length` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `BaseIncrementalDetokenizer.update` method / `BaseIncrementalDetokenizer.update` 方法
```python
    def update(self, new_token_ids: list[int], stop_terminated: bool) -> str | None:
        """
        Update RequestState for the request_id by:
            1) Detokenize the new token ids incrementally.
            2) Evaluate stop criteria.

        Return matched stop string or None.
        """
        if not new_token_ids:
            # Skip detokenization if no new token ids.
            return None

        if stop_terminated and not self.include_stop_str_in_output:
            # If stop-terminated, exclude last token from detokenization
            # based on include_stop_str_in_output parameter.
            skipped_stop_token_id = new_token_ids[-1]
            new_token_ids = new_token_ids[:-1]
        else:
            skipped_stop_token_id = None

        # 1) Detokenize the new token ids incrementally.
        stop_check_offset = len(self.output_text)
        for new_token_id in new_token_ids:
            self.token_ids.append(new_token_id)
            self.output_text += self.decode_next(new_token_id)
            # Support min_tokens, see https://github.com/vllm-project/vllm/pull/22014
            if self.min_tokens and self.num_output_tokens() <= self.min_tokens:
                stop_check_offset = len(self.output_text)

        if skipped_stop_token_id is not None:
            # Cleanup after skipping detokenization.
            self.token_ids.append(skipped_stop_token_id)

        # 2) Evaluate stop strings.
        stop_string = None
        if self.stop and self.num_output_tokens() > self.min_tokens:
            stop = check_stop_strings(
                output_text=self.output_text,
                new_char_count=len(self.output_text) - stop_check_offset,
                stop=self.stop,
                include_in_output=self.include_stop_str_in_output,
            )
            if stop is not None:
                stop_string, truncate_to = stop
                if truncate_to != -1:
                    self.output_text = self.output_text[:truncate_to]

        return stop_string
```
**EN:** This method updates existing state within `BaseIncrementalDetokenizer`. The docstring frames it as: Update RequestState for the request_id by: 1) Detokenize the new token ids incrementally. Key calls include `len`, `append`, `decode_next`, `check_stop_strings`, `num_output_tokens`. It touches state such as `output_text`. The control flow contains 7 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`BaseIncrementalDetokenizer`。 关键调用包括 `len`, `append`, `decode_next`, `check_stop_strings`, `num_output_tokens`。 它会读写 `output_text` 等状态。 控制流包含 7 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `BaseIncrementalDetokenizer.decode_next` method / `BaseIncrementalDetokenizer.decode_next` 方法
```python
    @abstractmethod
    def decode_next(self, next_token_id: int) -> str:
        raise NotImplementedError
```
**EN:** This method handles decoding logic within `BaseIncrementalDetokenizer`.
**CN:** 该方法会处理解码逻辑，其作用域位于`BaseIncrementalDetokenizer`。

### `BaseIncrementalDetokenizer.get_next_output_text` method / `BaseIncrementalDetokenizer.get_next_output_text` 方法
```python
    def get_next_output_text(self, finished: bool, delta: bool) -> str:
        """If delta is True, only new text since the last call to
        this method is returned"""

        # We return the full output text if the sequence is finished.
        buffer_length = 0 if finished else self.stop_buffer_length
        if not delta:
            if not buffer_length:
                return self.output_text
            return self.output_text[:-buffer_length]

        length = len(self.output_text) - buffer_length
        last_offset = self._last_output_text_offset
        if last_offset < length:
            self._last_output_text_offset = length
            return self.output_text[last_offset:length]
        return ""
```
**EN:** This method returns or derives a value within `BaseIncrementalDetokenizer`. The docstring frames it as: If delta is True, only new text since the last call to this method is returned Key calls include `len`. It touches state such as `_last_output_text_offset`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`BaseIncrementalDetokenizer`。 关键调用包括 `len`。 它会读写 `_last_output_text_offset` 等状态。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FastIncrementalDetokenizer` class / `FastIncrementalDetokenizer` 类
```python
class FastIncrementalDetokenizer(BaseIncrementalDetokenizer):
```
**EN:** Introduces the `FastIncrementalDetokenizer` class on top of `BaseIncrementalDetokenizer`. Core methods include `__init__`, `decode_next`, `_protected_step`.
**CN:** 这里定义 `FastIncrementalDetokenizer` 类，其基类包括 `BaseIncrementalDetokenizer`。核心方法包括 `__init__`, `decode_next`, `_protected_step`。

### `FastIncrementalDetokenizer.__init__` method / `FastIncrementalDetokenizer.__init__` 方法
```python
    def __init__(self, tokenizer: PreTrainedTokenizerFast, request: EngineCoreRequest):
        super().__init__(request)

        sampling_params = request.sampling_params
        assert sampling_params is not None

        self.request_id = request.request_id
        self.skip_special_tokens = sampling_params.skip_special_tokens

        self.tokenizer: Tokenizer = tokenizer._tokenizer

        # Use native prefill to prime the decode stream with prompt tokens.
        # Look up DecodeStream on the module so backend patches (e.g. the
        # fastokens shim that replaces ``tokenizers.decoders.DecodeStream``)
        # are honored regardless of import order.
        self.stream = tokenizers.decoders.DecodeStream(
            ids=request.prompt_token_ids,
            skip_special_tokens=self.skip_special_tokens,
        )

        self.spaces_between_special_tokens = (
            sampling_params.skip_special_tokens
            or sampling_params.spaces_between_special_tokens
        )

        if not self.spaces_between_special_tokens:
            # Store dict of added token ids so that we can suppress
            # the spaces between them.
            added_token_ids = getattr(self.tokenizer, "added_token_ids", None)
            if added_token_ids is None:
                self.tokenizer.added_token_ids = added_token_ids = {
                    tid: tok.content
                    for tid, tok in self.tokenizer.get_added_tokens_decoder().items()
                }

            if added_token_ids:
                self.last_special = False
                self.added_token_ids = added_token_ids
            else:
                # No added tokens.
                self.spaces_between_special_tokens = True
```
**EN:** This method initializes the object state within `FastIncrementalDetokenizer`. Key calls include `__init__`, `DecodeStream`, `getattr`, `super`, `items`, `get_added_tokens_decoder`. It touches state such as `request_id`, `skip_special_tokens`, `tokenizer`, `stream`, `spaces_between_special_tokens`, `last_special`, `added_token_ids`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`FastIncrementalDetokenizer`。 关键调用包括 `__init__`, `DecodeStream`, `getattr`, `super`, `items`, `get_added_tokens_decoder`。 它会读写 `request_id`, `skip_special_tokens`, `tokenizer`, `stream`, `spaces_between_special_tokens`, `last_special`, `added_token_ids` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FastIncrementalDetokenizer.decode_next` method / `FastIncrementalDetokenizer.decode_next` 方法
```python
    def decode_next(self, next_token_id: int) -> str:
        token = self._protected_step(next_token_id)

        if not self.spaces_between_special_tokens:
            special_token = self.added_token_ids.get(next_token_id)
            is_special = special_token is not None
            if is_special and self.last_special:
                # Return raw token string without any prefixed spaces.
                token = special_token
            self.last_special = is_special

        return token or ""
```
**EN:** This method handles decoding logic within `FastIncrementalDetokenizer`. Key calls include `_protected_step`, `get`. It touches state such as `last_special`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会处理解码逻辑，其作用域位于`FastIncrementalDetokenizer`。 关键调用包括 `_protected_step`, `get`。 它会读写 `last_special` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FastIncrementalDetokenizer._protected_step` method / `FastIncrementalDetokenizer._protected_step` 方法
```python
    def _protected_step(self, next_token_id: int) -> str | None:
        try:
            token = self.stream.step(self.tokenizer, next_token_id)
        except (OverflowError, TypeError):
            # Handle rare observed overflow, still to be diagnosed.
            # See https://github.com/vllm-project/vllm/issues/21951.
            logger.exception("Encountered invalid token id: %r", next_token_id)
            token = None
        except Exception as e:
            if not str(e).startswith(INVALID_PREFIX_ERR_MSG):
                raise e
            # Recover from edge case where tokenizer can produce non-monotonic,
            # invalid UTF-8 output, which breaks the internal state of
            # tokenizers' DecodeStream.
            # See https://github.com/vllm-project/vllm/issues/17448.
            logger.warning(
                "Encountered invalid prefix detokenization error"
                " for request %s, resetting decode stream.",
                self.request_id,
            )
            self.stream = tokenizers.decoders.DecodeStream(
                skip_special_tokens=self.skip_special_tokens
            )
            token = self.stream.step(self.tokenizer, next_token_id)
        return token
```
**EN:** This method implements `_protected_step` within `FastIncrementalDetokenizer`. Key calls include `step`, `exception`, `warning`, `DecodeStream`, `startswith`, `str`. It touches state such as `stream`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_protected_step`，其作用域位于`FastIncrementalDetokenizer`。 关键调用包括 `step`, `exception`, `warning`, `DecodeStream`, `startswith`, `str`。 它会读写 `stream` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SlowIncrementalDetokenizer` class / `SlowIncrementalDetokenizer` 类
```python
class SlowIncrementalDetokenizer(BaseIncrementalDetokenizer):
```
**EN:** Introduces the `SlowIncrementalDetokenizer` class on top of `BaseIncrementalDetokenizer`. Core methods include `__init__`, `output_token_ids`, `num_output_tokens`, `decode_next`.
**CN:** 这里定义 `SlowIncrementalDetokenizer` 类，其基类包括 `BaseIncrementalDetokenizer`。核心方法包括 `__init__`, `output_token_ids`, `num_output_tokens`, `decode_next`。

### `SlowIncrementalDetokenizer.__init__` method / `SlowIncrementalDetokenizer.__init__` 方法
```python
    def __init__(self, tokenizer: TokenizerLike, request: EngineCoreRequest):
        super().__init__(request)

        self.tokenizer = tokenizer
        params = request.sampling_params
        assert params is not None

        self.prompt_len = length_from_prompt_token_ids_or_embeds(
            request.prompt_token_ids, request.prompt_embeds
        )

        # Metadata for incremental detokenization.
        if request.prompt_token_ids is not None:
            self.tokens, self.prefix_offset, self.read_offset = (
                convert_prompt_ids_to_tokens(
                    tokenizer=tokenizer,
                    prompt_ids=request.prompt_token_ids,
                    skip_special_tokens=params.skip_special_tokens,
                )
            )
        else:
            # Prompt embedding requests cannot be detokenized, in general.
            self.tokens = [""] * self.prompt_len
            self.prefix_offset = 0
            self.read_offset = 0

        self.token_ids.extend(request.prompt_token_ids or [0] * self.prompt_len)

        self.skip_special_tokens = params.skip_special_tokens
        self.spaces_between_special_tokens = params.spaces_between_special_tokens
```
**EN:** This method initializes the object state within `SlowIncrementalDetokenizer`. Key calls include `__init__`, `length_from_prompt_token_ids_or_embeds`, `extend`, `convert_prompt_ids_to_tokens`, `super`. It touches state such as `tokenizer`, `prompt_len`, `skip_special_tokens`, `spaces_between_special_tokens`, `tokens`, `prefix_offset`, `read_offset`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`SlowIncrementalDetokenizer`。 关键调用包括 `__init__`, `length_from_prompt_token_ids_or_embeds`, `extend`, `convert_prompt_ids_to_tokens`, `super`。 它会读写 `tokenizer`, `prompt_len`, `skip_special_tokens`, `spaces_between_special_tokens`, `tokens`, `prefix_offset`, `read_offset` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SlowIncrementalDetokenizer.output_token_ids` method / `SlowIncrementalDetokenizer.output_token_ids` 方法
```python
    @property
    def output_token_ids(self) -> list[int]:
        if self.prompt_len:
            return self.token_ids[self.prompt_len :]
        return self.token_ids
```
**EN:** This method implements `output_token_ids` within `SlowIncrementalDetokenizer`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `output_token_ids`，其作用域位于`SlowIncrementalDetokenizer`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SlowIncrementalDetokenizer.num_output_tokens` method / `SlowIncrementalDetokenizer.num_output_tokens` 方法
```python
    def num_output_tokens(self) -> int:
        return len(self.token_ids) - self.prompt_len
```
**EN:** This method implements `num_output_tokens` within `SlowIncrementalDetokenizer`. Key calls include `len`.
**CN:** 该方法会实现 `num_output_tokens`，其作用域位于`SlowIncrementalDetokenizer`。 关键调用包括 `len`。

### `SlowIncrementalDetokenizer.decode_next` method / `SlowIncrementalDetokenizer.decode_next` 方法
```python
    def decode_next(self, next_token_id: int) -> str:
        new_tokens, decoded_text, prefix_offset, read_offset = detokenize_incrementally(
            tokenizer=self.tokenizer,
            all_input_ids=self.token_ids,
            prev_tokens=self.tokens,
            prefix_offset=self.prefix_offset,
            read_offset=self.read_offset,
            skip_special_tokens=self.skip_special_tokens,
            spaces_between_special_tokens=self.spaces_between_special_tokens,
        )

        self.tokens.extend(new_tokens)
        self.prefix_offset = prefix_offset
        self.read_offset = read_offset

        return decoded_text
```
**EN:** This method handles decoding logic within `SlowIncrementalDetokenizer`. Key calls include `detokenize_incrementally`, `extend`. It touches state such as `prefix_offset`, `read_offset`.
**CN:** 该方法会处理解码逻辑，其作用域位于`SlowIncrementalDetokenizer`。 关键调用包括 `detokenize_incrementally`, `extend`。 它会读写 `prefix_offset`, `read_offset` 等状态。

### `check_stop_strings` function / `check_stop_strings` 函数
```python
def check_stop_strings(
    output_text: str,
    new_char_count: int,
    stop: list[str],
    include_in_output: bool,
) -> tuple[str, int] | None:
    """Check if any stop strings are matched and truncate sequence
    output text accordingly.

    Returns tuple (stop_string, offset) if matched or else None.

    Where stop_string is the matched stop string and offset is the
    length to which output_text should be truncated, or -1 for no
    truncation.
    """
    if not new_char_count or not stop:
        return None

    for stop_str in stop:
        stop_string_len = len(stop_str)
        # Avoid searching already-searched text.
        stop_index = output_text.find(stop_str, 1 - new_char_count - stop_string_len)
        if stop_index == -1:
            continue

        if include_in_output:
            # Truncate to end of stop string.
            stop_index += stop_string_len
            if stop_index >= len(output_text):
                # No truncation required.
                return stop_str, -1

        # Truncate the output text to either the beginning
        # or end of the stop string.
        return stop_str, stop_index
    return None
```
**EN:** This function validates assumptions or constraints within the module. The docstring frames it as: Check if any stop strings are matched and truncate sequence output text accordingly. Key calls include `len`, `find`. The control flow contains 4 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会校验前提与约束，其作用域位于the module。 关键调用包括 `len`, `find`。 控制流包含 4 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `IncrementalDetokenizer`: central class or interface in this module. / `IncrementalDetokenizer`：本模块中的核心类或接口。
- `BaseIncrementalDetokenizer`: central class or interface in this module. / `BaseIncrementalDetokenizer`：本模块中的核心类或接口。
- `FastIncrementalDetokenizer`: central class or interface in this module. / `FastIncrementalDetokenizer`：本模块中的核心类或接口。
- `SlowIncrementalDetokenizer`: central class or interface in this module. / `SlowIncrementalDetokenizer`：本模块中的核心类或接口。
- `check_stop_strings`: top-level helper or orchestration entry point. / `check_stop_strings`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `abc`
- External / 外部依赖: `tokenizers`, `packaging`, `transformers`
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.tokenizers`, `vllm.tokenizers.detokenizer_utils`, `vllm.utils`, `vllm.v1.engine`
