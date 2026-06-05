# detokenizer_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tokenizers/detokenizer_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `detokenizer_utils`-related logic centered around `_replace_none_with_empty`, `_convert_tokens_to_string_with_added_encoders`. / 实现与 `detokenizer_utils` 相关的逻辑，核心符号包括 `_replace_none_with_empty`, `_convert_tokens_to_string_with_added_encoders`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-5)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project


from vllm.tokenizers import TokenizerLike
```
**EN:** Sets up the module with vLLM modules such as `vllm.tokenizers`. It prepares the symbols later used by `_replace_none_with_empty`, `_convert_tokens_to_string_with_added_encoders`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.tokenizers` 等 vLLM 内部依赖。 这些准备工作为后续的 `_replace_none_with_empty`, `_convert_tokens_to_string_with_added_encoders` 提供上下文。

### _replace_none_with_empty (lines 8-11)
```python
def _replace_none_with_empty(tokens: list[str | None]):
    for i, token in enumerate(tokens):
        if token is None:
            tokens[i] = ""
```
**EN:** `_replace_none_with_empty` implements helper logic used by this module. It mainly works with `tokens`.
**CN:** `_replace_none_with_empty` 负责实现本模块使用的辅助逻辑。 它主要处理 `tokens` 等参数。

### _convert_tokens_to_string_with_added_encoders (lines 14-51)
```python
def _convert_tokens_to_string_with_added_encoders(
    tokenizer: TokenizerLike,
    output_tokens: list[str],
    skip_special_tokens: bool,
    spaces_between_special_tokens: bool,
) -> str:
    # Adapted from
    # https://github.com/huggingface/transformers/blob/v4.28.0/src/transformers/tokenization_utils.py#L921
    # NOTE(woosuk): The following code is slow because it runs a for loop over
    # the output_tokens. In Python, running a for loop over a list can be slow
    # even when the loop body is very simple.
    # Performance improvements: avoid repeated attribute and function lookups;
    # localize frequently used objects;

    sub_texts: list[str] = []
    current_sub_text: list[str] = []
    convert_tokens_to_string = tokenizer.convert_tokens_to_string
    added_vocab_set = set(tokenizer.get_added_vocab())
    all_special_tokens = (
        set(tokenizer.all_special_tokens) if skip_special_tokens else ()
    )

    for token in output_tokens:
        # Use precomputed set for skip-special check
    # ...
            current_sub_text.append(token)
    if current_sub_text:
        sub_texts.append(convert_tokens_to_string(current_sub_text))
    if spaces_between_special_tokens:
        return " ".join(sub_texts)
    return "".join(sub_texts)
```
**EN:** `_convert_tokens_to_string_with_added_encoders` implements helper logic used by this module. It mainly works with `tokenizer`, `output_tokens`, `skip_special_tokens`, `spaces_between_special_tokens`. Inside the body, it relies on `join`, `tokenizer.get_added_vocab`, `sub_texts.append` to complete the main steps.
**CN:** `_convert_tokens_to_string_with_added_encoders` 负责实现本模块使用的辅助逻辑。 它主要处理 `tokenizer`, `output_tokens`, `skip_special_tokens`, `spaces_between_special_tokens` 等参数。 实现过程中会调用 `join`, `tokenizer.get_added_vocab`, `sub_texts.append` 等函数完成关键步骤。

### convert_prompt_ids_to_tokens (lines 59-80)
```python
def convert_prompt_ids_to_tokens(
    tokenizer: TokenizerLike,
    prompt_ids: list[int],
    skip_special_tokens: bool = False,
) -> tuple[list[str], int, int]:
    """Converts the prompt ids to tokens and returns the tokens and offsets
    for incremental detokenization.

    Note that not all tokens are converted to strings. Only the tokens that
    are necessary for incremental detokenization are converted to strings.
    """
    # We do not need to convert the whole prompt to tokens.
    # Offset a little more in case we have special tokens.
    new_tokens = tokenizer.convert_ids_to_tokens(
        prompt_ids[-INITIAL_INCREMENTAL_DETOKENIZATION_OFFSET - 2 :],
        skip_special_tokens=skip_special_tokens,
    )
    read_offset = len(new_tokens)
    prefix_offset = max(read_offset - INITIAL_INCREMENTAL_DETOKENIZATION_OFFSET, 0)
    # This is required to guard against out-of-vocab prompt token ids
    _replace_none_with_empty(new_tokens)  # type: ignore[arg-type]
    return new_tokens, prefix_offset, read_offset
```
**EN:** `convert_prompt_ids_to_tokens`: Converts the prompt ids to tokens and returns the tokens and offsets for incremental detokenization. It mainly works with `tokenizer`, `prompt_ids`, `skip_special_tokens`. Inside the body, it relies on `tokenizer.convert_ids_to_tokens`, `max`, `_replace_none_with_empty` to complete the main steps.
**CN:** `convert_prompt_ids_to_tokens` 负责实现本模块使用的辅助逻辑。 它主要处理 `tokenizer`, `prompt_ids`, `skip_special_tokens` 等参数。 实现过程中会调用 `tokenizer.convert_ids_to_tokens`, `max`, `_replace_none_with_empty` 等函数完成关键步骤。

### convert_ids_list_to_tokens (lines 83-104)
```python
def convert_ids_list_to_tokens(
    tokenizer: TokenizerLike,
    token_ids: list[int],
) -> list[str]:
    """Detokenize the input ids individually.

    Args:
      tokenizer: tokenizer used by model under test
      token_ids: convert these tokens (Python list form)

    Returns:
      Python list of token string representations

    """
    token_str_lst = []
    for token_id in token_ids:
        # use default skip_special_tokens.
        token_str = tokenizer.decode([token_id])
        if token_str is None:
            token_str = ""
        token_str_lst.append(token_str)
    return token_str_lst
```
**EN:** `convert_ids_list_to_tokens`: Detokenize the input ids individually. It mainly works with `tokenizer`, `token_ids`. Inside the body, it relies on `tokenizer.decode`, `token_str_lst.append` to complete the main steps.
**CN:** `convert_ids_list_to_tokens` 负责实现本模块使用的辅助逻辑。 它主要处理 `tokenizer`, `token_ids` 等参数。 实现过程中会调用 `tokenizer.decode`, `token_str_lst.append` 等函数完成关键步骤。

### detokenize_incrementally (lines 110-202)
```python
def detokenize_incrementally(
    tokenizer: TokenizerLike,
    all_input_ids: list[int],
    prev_tokens: list[str] | None,
    prefix_offset: int,
    read_offset: int,
    skip_special_tokens: bool = False,
    spaces_between_special_tokens: bool = True,
) -> tuple[list[str], str, int, int]:
    """Detokenizes the input ids incrementally and returns the new tokens
    and the new text.

    If `prev_tokens` is None, this function will convert the input ids to
    tokens and return the tokens and the new text. Otherwise, it will return the
    new tokens and the new text.

    This function will also return the new prefix offset and the new read
    offset to be used in the next iteration.

    The offsets are necessary to defeat cleanup algorithms in the decode which
    decide to add a space or not depending on the surrounding ids.

    Args:
        tokenizer: The tokenizer to use.
    # ...
        # If it's in the middle, it's probably a real invalid id generated
        # by the model
        return new_tokens, "", prefix_offset, read_offset

    new_text = new_text[len(prefix_text) :]
    return new_tokens, new_text, read_offset, len(output_tokens)
```
**EN:** `detokenize_incrementally`: Detokenizes the input ids incrementally and returns the new tokens and the new text. It mainly works with `tokenizer`, `all_input_ids`, `prev_tokens`, `prefix_offset`. Inside the body, it relies on `convert_prompt_ids_to_tokens`, `tokenizer.convert_ids_to_tokens`, `tokenizer.convert_tokens_to_string` to complete the main steps.
**CN:** `detokenize_incrementally` 负责实现本模块使用的辅助逻辑。 它主要处理 `tokenizer`, `all_input_ids`, `prev_tokens`, `prefix_offset` 等参数。 实现过程中会调用 `convert_prompt_ids_to_tokens`, `tokenizer.convert_ids_to_tokens`, `tokenizer.convert_tokens_to_string` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`_replace_none_with_empty`**: Key helper or entry point in this file. / **`_replace_none_with_empty`**：本文件中的关键辅助函数或入口。
- **`_convert_tokens_to_string_with_added_encoders`**: Key helper or entry point in this file. / **`_convert_tokens_to_string_with_added_encoders`**：本文件中的关键辅助函数或入口。
- **`convert_prompt_ids_to_tokens`**: Key helper or entry point in this file. / **`convert_prompt_ids_to_tokens`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: None / 无
- **Third-party / 第三方**: None / 无
- **Internal vLLM / vLLM 内部依赖**: vllm.tokenizers
