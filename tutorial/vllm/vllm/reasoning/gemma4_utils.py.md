# gemma4_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/gemma4_utils.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Collects Gemma 4 specific helper routines shared by the reasoning parser. / 收集 Gemma 4 推理解析器复用的辅助函数。

## Line-by-Line Analysis / 逐行分析
### Docstring (lines 5-32)
```python
"""Gemma4 thinking/reasoning output parsing utilities for offline inference.

Standalone functions that parse decoded model text to extract structured
thinking content from Gemma4 models. These are pure-Python utilities with
zero heavy dependencies — they work on raw decoded strings from any
inference backend (vLLM, HuggingFace, TGI, etc.).

For the OpenAI-compatible API reasoning parser (streaming +
non-streaming), see ``vllm.reasoning.gemma4_reasoning_parser``.
For tool call parsing, see ``vllm.tool_parsers.gemma4_utils``.

Usage with vLLM offline inference::

    from vllm import LLM, SamplingParams
    from vllm.reasoning.gemma4_utils import parse_thinking_output

    llm = LLM(model="google/gemma-4-it")
    outputs = llm.generate(prompt, SamplingParams(...))
    text = tokenizer.decode(outputs[0].outputs[0].token_ids, skip_special_tokens=False)

    # Extract thinking / answer (works with or without enable_thinking)
    result = parse_thinking_output(text)
    print(result["thinking"])  # chain-of-thought or None
    print(result["answer"])  # final answer

Ported from ``transformers.models.gemma4.utils_gemma4`` so that vLLM users
do not need a transformers dependency for output parsing.
"""
```
**EN:** Gemma4 thinking/reasoning output parsing utilities for offline inference.
**CN:** 模块文档字符串直接说明了该文件的用途。

### Module constants/configuration 1 (lines 38-39)
```python
_THINKING_START_TAG = "<|channel>"
_THINKING_END_TAG = "<channel|>"
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Module constants/configuration 2 (lines 42-42)
```python
_TURN_END_TAG = "<turn|>"
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Function `parse_thinking_output` (lines 45-103)
```python
def parse_thinking_output(text: str) -> dict[str, str | None]:
    """Parse decoded Gemma4 model output.

    Use this on **all** Gemma4 output regardless of whether thinking mode
    was enabled.  It handles three cases:

    1. **Thinking enabled, tags present** — splits on ``<|channel>``/
       ``<channel|>`` to separate chain-of-thought from the answer and
       strips the ``thought\\n`` role label.
    2. **Thinking disabled, spurious label** — strips the bare
       ``thought\\n`` prefix that some Gemma4 models emit even
       without thinking mode.
    3. **Clean output** — returns the text unchanged.

    The answer text is always cleaned of trailing sentinel tokens
    (``<turn|>``, ``<eos>``, etc.).

    Args:
        text: Decoded model output text (from ``tokenizer.decode(...)``).

    Returns:
        A dict with keys:
            - ``"thinking"``: The chain-of-thought text, or ``None`` if no
              thinking delimiters were found.
            - ``"answer"``: The final answer text.

    Example::

        >>> from vllm.reasoning.gemma4_utils import parse_thinking_output
        >>> output_text = tokenizer.decode(outputs[0], skip_special_tokens=False)
        >>> result = parse_thinking_output(output_text)
        >>> print(result["thinking"])  # chain-of-thought reasoning or None
        >>> print(result["answer"])    # final answer
    """
    if _THINKING_END_TAG in text:
        parts = text.split(_THINKING_END_TAG, 1)
        thinking_block = parts[0]
        answer = _clean_answer(parts[1])

        # Extract thinking content: strip the start tag if present
        if _THINKING_START_TAG in thinking_block:
            thinking = thinking_block.split(_THINKING_START_TAG, 1)[1]
        else:
            thinking = thinking_block

        # Strip the "thought\n" channel role label the model emits inside
        # <|channel>thought\n...<channel|> (analogous to "user\n" in
        # <|turn>user\n...<turn|>).
        thinking = _strip_thought_label(thinking.strip())
        thinking = thinking.strip()

        return {"thinking": thinking, "answer": answer}

    # No thinking delimiters found.
    # Strip spurious "thought\n" role label that some Gemma4 models sometimes
    # emit even without thinking mode enabled, then clean trailing tokens.
    answer = _strip_thought_label(text)
    answer = _clean_answer(answer)
    return {"thinking": None, "answer": answer}
```
**EN:** Parse decoded Gemma4 model output.
**CN:** `parse_thinking_output` 函数实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Function `_strip_thought_label` (lines 106-114)
```python
def _strip_thought_label(text: str) -> str:
    """Strip the spurious ``thought\\n`` label from the start of text.

    Only strips when ``thought`` appears as the very first word followed by
    a newline — preserving the word ``thought`` in any other context.
    """
    if text.startswith("thought\n"):
        return text[len("thought\n") :]
    return text
```
**EN:** Strip the spurious ``thought\n`` label from the start of text.
**CN:** `_strip_thought_label` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Function `_clean_answer` (lines 117-130)
```python
def _clean_answer(text: str) -> str:
    """Clean trailing sentinel tokens from the answer text.

    Strips ``<turn|>``, ``<eos>``, and surrounding whitespace that the
    model appends at the end of its response.
    """
    text = text.strip()
    # Strip trailing <turn|> (Gemma4 turn-end marker)
    if text.endswith(_TURN_END_TAG):
        text = text[: -len(_TURN_END_TAG)].rstrip()
    # Strip trailing <eos> if present
    if text.endswith("<eos>"):
        text = text[:-5].rstrip()
    return text
```
**EN:** Clean trailing sentinel tokens from the answer text.
**CN:** `_clean_answer` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: None / 无
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: None / 无
