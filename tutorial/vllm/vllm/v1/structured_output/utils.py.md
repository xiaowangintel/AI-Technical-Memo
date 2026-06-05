# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/structured_output/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `apply_grammar_bitmask`, `OutlinesVocabulary`, `get_outlines_cache_path` for the V1 `structured_output` subsystem. / 为 V1 的 `structured_output` 子系统实现 `apply_grammar_bitmask`, `OutlinesVocabulary`, `get_outlines_cache_path`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from __future__ import annotations

import hashlib
import importlib.metadata
import os
import tempfile
from typing import TYPE_CHECKING

import numpy as np
import regex as re
import torch
from cachetools import LRUCache

import vllm.envs as envs
from vllm.logger import init_logger
from vllm.utils.import_utils import LazyLoader
from vllm.utils.platform_utils import is_pin_memory_available
from vllm.v1.core.sched.output import GrammarOutput, SchedulerOutput

if TYPE_CHECKING:
    import outlines_core as oc
    import transformers.convert_slow_tokenizer as convert_slow_tokenizer
    import transformers.file_utils as file_utils
    import xgrammar as xgr

    from vllm.tokenizers import TokenizerLike
    from vllm.v1.worker.gpu_input_batch import InputBatch
else:
    xgr = LazyLoader("xgr", globals(), "xgrammar")
    oc = LazyLoader("oc", globals(), "outlines_core")
    file_utils = LazyLoader("file_utils", globals(), "transformers.file_utils")
    convert_slow_tokenizer = LazyLoader(
        "convert_slow_tokenizer", globals(), "transformers.convert_slow_tokenizer"
    )


logger = init_logger(__name__)

CACHE = None
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `CACHE`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `CACHE`。

### `apply_grammar_bitmask` function / `apply_grammar_bitmask` 函数
```python
def apply_grammar_bitmask(
    scheduler_output: SchedulerOutput,
    grammar_output: GrammarOutput,
    input_batch: InputBatch,
    logits: torch.Tensor,
) -> None:
    """
    Apply grammar bitmask to output logits of the model with xgrammar function.

    Args:
        scheduler_output (SchedulerOutput): The result of engine scheduling.
        input_batch (InputBatch): The input of model runner.
        logits (torch.Tensor): The output logits of model forward.
    """
    # Serialization of np.ndarray is much more efficient than a tensor,
    # so we receive it in that format.
    grammar_bitmask = grammar_output.grammar_bitmask

    # We receive the structured output bitmask from the scheduler,
    # compacted to contain bitmasks only for structured output requests.
    # The order of the requests in the bitmask is not guaranteed to be the
    # same as the order of the requests in the gpu runner's batch. We need
    # to sort the bitmask to match the order of the requests used here.

    # Get the batch indices of the structured output requests.
    # Keep track of the number of speculative tokens scheduled for every
    # request in the batch, as the logit indices are offset by this amount.
    struct_out_req_batch_indices: dict[str, int] = {}
    cumulative_offset = 0
    spec_tokens = scheduler_output.scheduled_spec_decode_tokens
    struct_out_req_ids = set(grammar_output.structured_output_request_ids)
    for batch_index, req_id in enumerate(input_batch.req_ids):
        logit_index = batch_index + cumulative_offset
        cumulative_offset += len(spec_tokens.get(req_id, ()))
        if req_id in struct_out_req_ids:
            struct_out_req_batch_indices[req_id] = logit_index

    out_indices = []

    # Reorder the bitmask to match the order of the requests in the batch.
    sorted_bitmask = np.full(
        shape=(logits.shape[0], grammar_bitmask.shape[1]),
        fill_value=-1,
        dtype=grammar_bitmask.dtype,
    )
    cumulative_index = 0
    for req_id in grammar_output.structured_output_request_ids:
        num_spec_tokens = len(spec_tokens.get(req_id, ()))
        if (logit_idx := struct_out_req_batch_indices.get(req_id)) is not None:
            for i in range(1 + num_spec_tokens):
# ... omitted for brevity ...
        if not skip_out_indices:
            # xgrammar expects a python list of indices but it will actually work with
            # a tensor. If we copy the tensor ourselves here we can do it in a
            # non_blocking manner and there should be no cpu sync within xgrammar.
            pin_memory = is_pin_memory_available()
            index_tensor = torch.tensor(
                out_indices, dtype=torch.int32, device="cpu", pin_memory=pin_memory
            )
            index_tensor = index_tensor.to(logits.device, non_blocking=True)

        xgr.apply_token_bitmask_inplace(logits, grammar_bitmask, indices=index_tensor)
        return

    # CPU case, use list for indices.
    indices = None if skip_out_indices else out_indices
    # Handle dtype conversion for CPU (older xgrammar CPU kernels require float32)
    # See: https://github.com/vllm-project/vllm/issues/31901
    if logits.dtype != torch.float32:
        # Convert to float32, apply bitmask, then convert back
        logits_fp32 = logits.to(torch.float32)
        xgr.apply_token_bitmask_inplace(logits_fp32, grammar_bitmask, indices=indices)
        # Copy the modified values back to the original tensor
        logits.copy_(logits_fp32.to(logits.dtype))
    else:
        xgr.apply_token_bitmask_inplace(logits, grammar_bitmask, indices=indices)
```
**EN:** This function implements `apply_grammar_bitmask` within the module. The docstring frames it as: Apply grammar bitmask to output logits of the model with xgrammar function. Key calls include `set`, `enumerate`, `full`, `to`, `len`, `apply_token_bitmask_inplace`. The control flow contains 6 branch(es) and 3 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `apply_grammar_bitmask`，其作用域位于the module。 关键调用包括 `set`, `enumerate`, `full`, `to`, `len`, `apply_token_bitmask_inplace`。 控制流包含 6 个分支和 3 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `OutlinesVocabulary` class / `OutlinesVocabulary` 类
```python
class OutlinesVocabulary:
    """
    Wrapper class for `outlines_core.Vocabulary`,
    which allows us to store a hash with the vocabulary
    """
```
**EN:** Introduces the `OutlinesVocabulary` class. Core methods include `__init__`. Docstring signal: Wrapper class for `outlines_core.Vocabulary`, which allows us to store a hash with the vocabulary
**CN:** 这里定义 `OutlinesVocabulary` 类。核心方法包括 `__init__`。

### `OutlinesVocabulary.__init__` method / `OutlinesVocabulary.__init__` 方法
```python
    def __init__(self, vocabulary: oc.Vocabulary) -> None:
        # Actual vocabulary object
        self.inner = vocabulary
        # Have to do abs(hash()) because python hashes can
        # be negative, and we are using hash as a cache key.
        hex_str = hashlib.sha256(vocabulary.__repr__().encode("utf-8")).hexdigest()
        hash_int = int(hex_str, 16)
        self._hash = hash_int
```
**EN:** This method initializes the object state within `OutlinesVocabulary`. Key calls include `hexdigest`, `int`, `sha256`, `encode`, `__repr__`. It touches state such as `inner`, `_hash`.
**CN:** 该方法会初始化对象状态，其作用域位于`OutlinesVocabulary`。 关键调用包括 `hexdigest`, `int`, `sha256`, `encode`, `__repr__`。 它会读写 `inner`, `_hash` 等状态。

### `get_outlines_cache_path` function / `get_outlines_cache_path` 函数
```python
def get_outlines_cache_path() -> str:
    """Get the context object that contains previously-computed return values"""
    outlines_cache_dir = os.getenv("OUTLINES_CACHE_DIR")
    xdg_cache_home = os.getenv("XDG_CACHE_HOME")
    home_dir = os.path.expanduser("~")

    if outlines_cache_dir:
        # OUTLINES_CACHE_DIR takes precedence
        return outlines_cache_dir
    if xdg_cache_home:
        return os.path.join(xdg_cache_home, ".cache", "outlines")
    # If homedir is "/", we may be inside a container, and thus writing to
    # root would be problematic, so we fall back to using a tempfile.
    # Also validate the path exists, since os.path.expanduser does
    # not guarantee existence.
    if os.path.isdir(home_dir) and home_dir != "/":
        # Default Unix fallback: ~/.cache/outlines
        return os.path.join(home_dir, ".cache", "outlines")

    # home_dir may be / inside a docker container without existing user
    tempdir = tempfile.gettempdir()
    return os.path.join(tempdir, ".cache", "outlines")
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Get the context object that contains previously-computed return values Key calls include `getenv`, `expanduser`, `gettempdir`, `join`, `isdir`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `getenv`, `expanduser`, `gettempdir`, `join`, `isdir`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `get_outlines_cache` function / `get_outlines_cache` 函数
```python
def get_outlines_cache():
    """Get the Cache instance to be used for index caching"""

    cache_dir = get_outlines_cache_path()
    if envs.VLLM_V1_USE_OUTLINES_CACHE:
        from diskcache import Cache

        logger.warning(
            "Enabling outlines cache. This is an unbounded on-disk "
            "cache. It may consume a lot of disk space and should "
            "not be used with untrusted clients."
        )
        cache = Cache(cache_dir, eviction_policy="none", cull_limit=0)
        outlines_version = importlib.metadata.version("outlines_core")

        cached_version = cache.get("__version__", None)
        if cached_version != outlines_version:
            cache.clear()
        cache.set("__version__", outlines_version)
        return cache

    return LRUCache(maxsize=128)
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Get the Cache instance to be used for index caching Key calls include `get_outlines_cache_path`, `LRUCache`, `warning`, `Cache`, `version`, `get`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `get_outlines_cache_path`, `LRUCache`, `warning`, `Cache`, `version`, `get`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### Module constants / 模块常量
```python
re_llama_byte_token = re.compile(r"^<0x[0-9A-F]{2}>$")
re_replacement_seq = re.compile(r"^.{0,6}�+.{0,6}$")
```
**EN:** Defines module-level constants or aliases such as `re_llama_byte_token`, `re_replacement_seq`, which are reused by later definitions.
**CN:** 定义 `re_llama_byte_token`, `re_replacement_seq` 等模块级常量或别名，供后续定义复用。

### `_reduced_vocabulary` function / `_reduced_vocabulary` 函数
```python
def _reduced_vocabulary(tokenizer: TokenizerLike) -> dict[bytes, list[int]]:
    """Create a map from vocabulary tokens to lists of equivalent token ids.

    Returns:
        A Dict of token string -> equivalent token ids
    """
    eos_token_id = tokenizer.eos_token_id

    unicode_to_bytes = {
        v: k for k, v in convert_slow_tokenizer.bytes_to_unicode().items()
    }

    def convert_token_to_string(token: str) -> str:
        string = tokenizer.convert_tokens_to_string([token])

        # A hack to handle missing spaces to HF's Llama tokenizers
        if (
            type(token) is str
            and token.startswith(file_utils.SPIECE_UNDERLINE)
            or token == "<0x20>"
        ):
            return " " + string

        return string

    vocabulary: dict[bytes, list[int]] = {}
    empty_token_ids: list[int] = []
    for token, token_idx in tokenizer.get_vocab().items():
        if token in tokenizer.all_special_tokens:
            continue

        token_str = convert_token_to_string(token)
        if token_str:
            if isinstance(token, (bytes, bytearray)):
                # For BPE tokenizers where tokens are stored as bytes.

                # safe to ignore since token_str is of type (bytearray, bytes)
                # by this point.
                token_bytes = bytes(token_str)  # type: ignore[arg-type]

            elif (token_str == "\ufffd" and token != "\ufffd") or (
                "\ufffd" in token_str and not re_replacement_seq.match(token_str)
            ):
                # Handle tokens with invalid UTF-8 sequences.
                if re_llama_byte_token.match(token):
                    # Llama-like tokenizers use <0xXX> for incomplete sequences.
                    token_bytes = bytes([int(token[3:5], 16)])
                else:
                    # GPT2 tokenizers: map each byte back using unicode_to_bytes
                    byte_vals = [unicode_to_bytes.get(c) for c in token]
                    if None in byte_vals:
                        raise RuntimeError(
                            f"Cannot convert token `{token}`"
                            f" ({token_idx}) to bytes: {token_str}"
                        )
                    # safe to ignore, since if None in byte_vals,
                    # an error is thrown.
                    token_bytes = bytes(byte_vals)  # type: ignore[arg-type]
            else:
                token_bytes = token_str.encode("utf-8")

            if token_idx != eos_token_id:
                vocabulary.setdefault(token_bytes, []).append(token_idx)
        else:
            empty_token_ids.append(token_idx)

    return vocabulary
```
**EN:** This function implements `_reduced_vocabulary` within the module. The docstring frames it as: Create a map from vocabulary tokens to lists of equivalent token ids. Key calls include `items`, `convert_tokens_to_string`, `convert_token_to_string`, `get_vocab`, `isinstance`, `append`. The control flow contains 8 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_reduced_vocabulary`，其作用域位于the module。 关键调用包括 `items`, `convert_tokens_to_string`, `convert_token_to_string`, `get_vocab`, `isinstance`, `append`。 控制流包含 8 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `get_outlines_vocabulary` function / `get_outlines_vocabulary` 函数
```python
def get_outlines_vocabulary(tokenizer: TokenizerLike) -> oc.Vocabulary:
    """Get the `Vocabulary` object for a given tokenizer."""
    if hasattr(tokenizer, "_outlines_vocabulary"):
        return tokenizer._outlines_vocabulary  # type: ignore

    reduced_vocab = _reduced_vocabulary(tokenizer)
    vocabulary = OutlinesVocabulary(
        oc.Vocabulary(tokenizer.eos_token_id, reduced_vocab)
    )
    tokenizer._outlines_vocabulary = vocabulary  # type: ignore

    return vocabulary
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Get the `Vocabulary` object for a given tokenizer. Key calls include `hasattr`, `_reduced_vocabulary`, `OutlinesVocabulary`, `Vocabulary`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `hasattr`, `_reduced_vocabulary`, `OutlinesVocabulary`, `Vocabulary`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `grammar_is_likely_lark` function / `grammar_is_likely_lark` 函数
```python
def grammar_is_likely_lark(grammar_str: str) -> bool:
    """
    Check if grammar appears to use Lark syntax.

    Args:
        grammar_str: Input grammar string

    Returns:
        bool: True if grammar appears to be in Lark format, False otherwise

    Examples:
        >>> grammar_is_likely_lark("rule: 'abc'")
        True
        >>> grammar_is_likely_lark("rule ::= 'abc'")
        False
    """
    if not grammar_str or not isinstance(grammar_str, str):
        return False

    for line in grammar_str.split("\n"):
        # Remove both comment styles
        line = re.sub(r"(#|//).*$", "", line).strip()
        if not line:
            continue

        # Look for EBNF rule definition
        if "::=" in line:
            return False

    return True
```
**EN:** This function implements `grammar_is_likely_lark` within the module. The docstring frames it as: Check if grammar appears to use Lark syntax. Key calls include `split`, `strip`, `isinstance`, `sub`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `grammar_is_likely_lark`，其作用域位于the module。 关键调用包括 `split`, `strip`, `isinstance`, `sub`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `convert_lark_to_ebnf` function / `convert_lark_to_ebnf` 函数
```python
def convert_lark_to_ebnf(grammar_str: str) -> str:
    """
    Convert a Lark grammar string to EBNF format.

    EBNF reference:
    https://github.com/ggerganov/llama.cpp/blob/master/grammars/README.md
    Lark grammar reference:
    https://lark-parser.readthedocs.io/en/latest/grammar.html

    Args:
        grammar_str: Input grammar in Lark format

    Returns:
        str: Converted grammar in EBNF format

    Examples:
        >>> print(convert_lark_to_ebnf("rule: 'hello'"))
        root ::= rule
        rule ::= "hello"
    """
    if not isinstance(grammar_str, str):
        raise ValueError(f"Grammar must be a string, got {type(grammar_str)}")
    if not grammar_str.strip():
        raise ValueError("Grammar string cannot be empty")

    defined_rules = set()
    referenced_rules = set()
    output_lines = []

    def clean_line(line: str) -> str:
        """Remove comments and whitespace from line."""
        return re.sub(r"(#|//).*$", "", line).strip()

    def check_quotes(text: str, rule_name: str, line_num: int) -> None:
        """Validate quote matching in text."""
        if text.count("'") % 2 != 0 or text.count('"') % 2 != 0:
            raise ValueError(f"Mismatched quotes in {rule_name} on line {line_num}")

    def extract_references(text: str) -> set[str]:
        """Extract rule references from text."""
        # Remove quoted strings and special characters
        text = re.sub(r'"[^"]*"', "", text)
        text = re.sub(r"[+*?()|\[\]{}]", " ", text)
        return set(re.findall(r"\b[a-zA-Z_][a-zA-Z0-9_]*\b", text))

    # First pass: Find root rule and validate rule definitions
    lines = [clean_line(line) for line in grammar_str.split("\n")]
    first_rule = None

    for line_num, line in enumerate(lines, 1):
# ... omitted for brevity ...
                    )

                alt_def = line[1:].strip()
                check_quotes(
                    alt_def, f"alternative for rule '{current_rule}'", line_num
                )
                alt_def = re.sub(r"'([^']*)'", r'"\1"', alt_def)
                referenced_rules.update(extract_references(alt_def))
                current_definition.append(alt_def)

        except ValueError as e:
            raise ValueError(f"Error on line {line_num}: {str(e)}") from e

    # Add final rule if exists
    if current_rule:
        output_lines.append(f"{current_rule} ::= {' | '.join(current_definition)}")

    # Validate all rules are defined
    undefined_rules = referenced_rules - defined_rules - {"root"}
    if undefined_rules:
        raise ValueError(
            f"Referenced rules are not defined: {', '.join(sorted(undefined_rules))}"
        )

    return "\n".join(output_lines)
```
**EN:** This function converts data between representations within the module. The docstring frames it as: Convert a Lark grammar string to EBNF format. Key calls include `set`, `enumerate`, `append`, `join`, `isinstance`, `ValueError`. The control flow contains 17 branch(es) and 2 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会在不同表示之间转换数据，其作用域位于the module。 关键调用包括 `set`, `enumerate`, `append`, `join`, `isinstance`, `ValueError`。 控制流包含 17 个分支和 2 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `choice_as_grammar` function / `choice_as_grammar` 函数
```python
def choice_as_grammar(choice: list[str]) -> str:
    def escape_ebnf_string(s: str) -> str:
        """Escape special characters in a EBNF string."""
        # Escape double quotes and backslashes
        return re.sub(r'(["\\])', r"\\\1", s)

    escaped_choices = (escape_ebnf_string(c) for c in choice)
    grammar = "root ::= " + " | ".join(f'"{c}"' for c in escaped_choices)
    return grammar
```
**EN:** This function implements `choice_as_grammar` within the module. Key calls include `sub`, `escape_ebnf_string`, `join`.
**CN:** 该函数会实现 `choice_as_grammar`，其作用域位于the module。 关键调用包括 `sub`, `escape_ebnf_string`, `join`。

## Key Concepts / 关键概念
- `apply_grammar_bitmask`: top-level helper or orchestration entry point. / `apply_grammar_bitmask`：顶层辅助函数或编排入口。
- `OutlinesVocabulary`: central class or interface in this module. / `OutlinesVocabulary`：本模块中的核心类或接口。
- `get_outlines_cache_path`: top-level helper or orchestration entry point. / `get_outlines_cache_path`：顶层辅助函数或编排入口。
- `get_outlines_cache`: top-level helper or orchestration entry point. / `get_outlines_cache`：顶层辅助函数或编排入口。
- `_reduced_vocabulary`: top-level helper or orchestration entry point. / `_reduced_vocabulary`：顶层辅助函数或编排入口。
- `get_outlines_vocabulary`: top-level helper or orchestration entry point. / `get_outlines_vocabulary`：顶层辅助函数或编排入口。
- `grammar_is_likely_lark`: top-level helper or orchestration entry point. / `grammar_is_likely_lark`：顶层辅助函数或编排入口。
- `convert_lark_to_ebnf`: top-level helper or orchestration entry point. / `convert_lark_to_ebnf`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `__future__`, `hashlib`, `importlib`, `os`, `tempfile`, `typing`
- External / 外部依赖: `numpy`, `regex`, `torch`, `cachetools`, `outlines_core`, `transformers`, `xgrammar`, `diskcache`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.logger`, `vllm.utils.import_utils`, `vllm.utils.platform_utils`, `vllm.v1.core.sched.output`, `vllm.tokenizers`, `vllm.v1.worker.gpu_input_batch`
