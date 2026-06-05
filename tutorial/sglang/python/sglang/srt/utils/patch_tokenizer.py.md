# patch_tokenizer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/patch_tokenizer.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for token processing helpers. / 为 SGLang 运行时提供面向Token 处理辅助逻辑的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Module setup and shared state / 模块设置与共享状态
```python
import logging

from sglang.srt.environ import envs

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `logging`, `sglang.srt.environ`. It also defines symbols such as `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `logging`, `sglang.srt.environ`。 同时定义了 `logger` 等符号，供后续逻辑使用。

### Lines 8-18: Function `patch_tokenizer` / 函数 `patch_tokenizer`
```python
def patch_tokenizer(tokenizer):
    if not envs.SGLANG_PATCH_TOKENIZER.get():
        return tokenizer

    if _is_kimi_tiktoken_tokenizer(tokenizer):
        logger.info(
            f"Applying special tokens cache patch for Kimi tokenizer: {type(tokenizer)}"
        )
        return _SpecialTokensCachePatcher.patch(tokenizer)

    return tokenizer
```
**EN:** This function implements `patch_tokenizer`. It primarily calls `_is_kimi_tiktoken_tokenizer`, `envs.SGLANG_PATCH_TOKENIZER.get`, `logger.info`, `_SpecialTokensCachePatcher.patch`, `type` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `patch_tokenizer`。 它主要通过调用 `_is_kimi_tiktoken_tokenizer`, `envs.SGLANG_PATCH_TOKENIZER.get`, `logger.info`, `_SpecialTokensCachePatcher.patch`, `type` 来完成任务。 实现中使用了条件分支。

### Lines 21-22: Function `unpatch_tokenizer` / 函数 `unpatch_tokenizer`
```python
def unpatch_tokenizer(tokenizer):
    return _SpecialTokensCachePatcher.unpatch(tokenizer)
```
**EN:** This function implements `unpatch_tokenizer`. It primarily calls `_SpecialTokensCachePatcher.unpatch` to complete its work.
**CN:** 该函数实现了 `unpatch_tokenizer`。 它主要通过调用 `_SpecialTokensCachePatcher.unpatch` 来完成任务。

### Lines 25-29: Function `_is_kimi_tiktoken_tokenizer` / 函数 `_is_kimi_tiktoken_tokenizer`
```python
def _is_kimi_tiktoken_tokenizer(tokenizer):
    cls = type(tokenizer)
    class_name = cls.__name__
    module_name = cls.__module__ or ""
    return class_name == "TikTokenTokenizer" and "tokenization_kimi" in module_name
```
**EN:** This function implements `_is_kimi_tiktoken_tokenizer`. It primarily calls `type` to complete its work. State updates are written into `cls`, `class_name`, `module_name`.
**CN:** 该函数实现了 `_is_kimi_tiktoken_tokenizer`。 它主要通过调用 `type` 来完成任务。 状态更新主要写入 `cls`, `class_name`, `module_name`。

### Lines 32-38: Function `decode_without_hf_kwargs` / 函数 `decode_without_hf_kwargs`
```python
def decode_without_hf_kwargs(tokenizer, token_ids, skip_special_tokens):
    if skip_special_tokens:
        special_ids = getattr(tokenizer, "all_special_ids_set", None)
        if special_ids is None:
            special_ids = set(tokenizer.all_special_ids)
        token_ids = [tid for tid in token_ids if tid not in special_ids]
    return tokenizer.decode(token_ids)
```
**EN:** This function implements `decode_without_hf_kwargs`. It primarily calls `tokenizer.decode`, `getattr`, `set` to complete its work. State updates are written into `special_ids`, `token_ids`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `decode_without_hf_kwargs`。 它主要通过调用 `tokenizer.decode`, `getattr`, `set` 来完成任务。 状态更新主要写入 `special_ids`, `token_ids`。 实现中使用了条件分支。

### Lines 41-44: Class `_SpecialTokensCachePatcher` declaration / 类 `_SpecialTokensCachePatcher` 声明
```python
class _SpecialTokensCachePatcher:
    _PATCHED_FLAG = "_sglang_special_tokens_patched"
    _CACHED_TOKENS_ATTR = "_sglang_cached_special_tokens"
    _CACHED_IDS_ATTR = "_sglang_cached_special_ids"
```
**EN:** This class establishes `_SpecialTokensCachePatcher` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `patch`, `unpatch`.
**CN:** 该类将 `_SpecialTokensCachePatcher` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `patch`, `unpatch` 等方法。

### Lines 46-86: Method `_SpecialTokensCachePatcher.patch` / 方法 `_SpecialTokensCachePatcher.patch`
```python
    @classmethod
    def patch(cls, tokenizer):
        tokenizer_cls = type(tokenizer)

        if getattr(tokenizer_cls, cls._PATCHED_FLAG, False):
            return tokenizer

        tokenizer_cls._original_all_special_tokens = (
            tokenizer_cls.all_special_tokens.fget
        )
        tokenizer_cls._original_all_special_ids = tokenizer_cls.all_special_ids.fget
        tokenizer_cls._original_add_special_tokens = tokenizer_cls.add_special_tokens
        tokenizer_cls._original_add_tokens = tokenizer_cls.add_tokens

        patched_all_special_tokens = _make_cached_property(
            cls._CACHED_TOKENS_ATTR, tokenizer_cls._original_all_special_tokens
        )
        patched_all_special_ids = _make_cached_property(
            cls._CACHED_IDS_ATTR, tokenizer_cls._original_all_special_ids
        )

        def patched_add_special_tokens(self, *args, **kwargs):
            assert (
                False
            ), "Cannot modify special tokens after patch. Call unpatch_tokenizer first."

        def patched_add_tokens(self, new_tokens, special_tokens=False):
            assert (
# ... omitted for brevity ...
        tokenizer_cls.all_special_ids = patched_all_special_ids
        tokenizer_cls.add_special_tokens = patched_add_special_tokens
        tokenizer_cls.add_tokens = patched_add_tokens
        setattr(tokenizer_cls, cls._PATCHED_FLAG, True)

        return tokenizer
```
**EN:** This method implements `patch` on `_SpecialTokensCachePatcher`. It primarily calls `type`, `getattr`, `_make_cached_property`, `setattr`, `tokenizer_cls._original_add_tokens` to complete its work. State updates are written into `tokenizer_cls`, `tokenizer_cls._original_all_special_tokens`, `tokenizer_cls._original_all_special_ids`, `tokenizer_cls._original_add_special_tokens`, `tokenizer_cls._original_add_tokens`, `patched_all_special_tokens`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_SpecialTokensCachePatcher`）实现了 `patch`。 它主要通过调用 `type`, `getattr`, `_make_cached_property`, `setattr`, `tokenizer_cls._original_add_tokens` 来完成任务。 状态更新主要写入 `tokenizer_cls`, `tokenizer_cls._original_all_special_tokens`, `tokenizer_cls._original_all_special_ids`, `tokenizer_cls._original_add_special_tokens`, `tokenizer_cls._original_add_tokens`, `patched_all_special_tokens`。 实现中使用了条件分支。

### Lines 88-115: Method `_SpecialTokensCachePatcher.unpatch` / 方法 `_SpecialTokensCachePatcher.unpatch`
```python
    @classmethod
    def unpatch(cls, tokenizer):
        tokenizer_cls = type(tokenizer)

        if not getattr(tokenizer_cls, cls._PATCHED_FLAG, False):
            return tokenizer

        tokenizer_cls.all_special_tokens = property(
            tokenizer_cls._original_all_special_tokens
        )
        tokenizer_cls.all_special_ids = property(
            tokenizer_cls._original_all_special_ids
        )
        tokenizer_cls.add_special_tokens = tokenizer_cls._original_add_special_tokens
        tokenizer_cls.add_tokens = tokenizer_cls._original_add_tokens

        del tokenizer_cls._original_all_special_tokens
        del tokenizer_cls._original_all_special_ids
        del tokenizer_cls._original_add_special_tokens
        del tokenizer_cls._original_add_tokens
        delattr(tokenizer_cls, cls._PATCHED_FLAG)

        for attr in [cls._CACHED_TOKENS_ATTR, cls._CACHED_IDS_ATTR]:
            if hasattr(tokenizer, attr):
                delattr(tokenizer, attr)

        logger.info(f"Unpatched special tokens cache for {tokenizer_cls.__name__}")
        return tokenizer
```
**EN:** This method implements `unpatch` on `_SpecialTokensCachePatcher`. It primarily calls `type`, `property`, `delattr`, `logger.info`, `getattr`, `hasattr` to complete its work. State updates are written into `tokenizer_cls`, `tokenizer_cls.all_special_tokens`, `tokenizer_cls.all_special_ids`, `tokenizer_cls.add_special_tokens`, `tokenizer_cls.add_tokens`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `_SpecialTokensCachePatcher`）实现了 `unpatch`。 它主要通过调用 `type`, `property`, `delattr`, `logger.info`, `getattr`, `hasattr` 来完成任务。 状态更新主要写入 `tokenizer_cls`, `tokenizer_cls.all_special_tokens`, `tokenizer_cls.all_special_ids`, `tokenizer_cls.add_special_tokens`, `tokenizer_cls.add_tokens`。 实现中使用了条件分支、迭代逻辑。

### Lines 118-125: Function `_make_cached_property` / 函数 `_make_cached_property`
```python
def _make_cached_property(cache_attr, original_fn):
    @property
    def cached_prop(self):
        if getattr(self, cache_attr, None) is None:
            setattr(self, cache_attr, original_fn(self))
        return getattr(self, cache_attr)

    return cached_prop
```
**EN:** This function implements `_make_cached_property`. It primarily calls `getattr`, `setattr`, `original_fn` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_make_cached_property`。 它主要通过调用 `getattr`, `setattr`, `original_fn` 来完成任务。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `_SpecialTokensCachePatcher`
- **Functions / 函数**: `patch_tokenizer`, `unpatch_tokenizer`, `_is_kimi_tiktoken_tokenizer`, `decode_without_hf_kwargs`, `_make_cached_property`, `patch`, `unpatch`, `cached_prop`
- **Themes / 主题**: `patch`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.environ`
- **External / 外部依赖**: None / 无
- **Standard library / 标准库**: `logging`
