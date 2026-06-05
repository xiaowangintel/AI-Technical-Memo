# util.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/ir/util.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Utility helpers for source hashing and weak-reference caches. / 源码哈希与弱引用缓存的辅助工具。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-9)
```python
import functools
import hashlib
import inspect
import types
import weakref
from pathlib import Path
from typing import Any
```
**EN:** This import block brings in PyTorch operator-registration utilities, inspection helpers, and vLLM IR support modules used to define or dispatch custom IR ops inside vLLM's custom IR registry and provider system.
**CN:** 这一组导入语句引入了 PyTorch 算子注册工具、代码检查辅助模块以及 vLLM IR 支撑组件，用于在vLLM 自定义 IR 注册与 provider 系统中定义或派发自定义 IR 算子。

### Function `hash_source` (lines 12-33)
```python
def hash_source(*srcs: str | Any) -> str:
    """
    Utility method to hash the sources of functions or objects.
    :param srcs: strings or objects to add to the hash.
    Objects and functions have their source inspected.
    :return:
    """
    hasher = hashlib.sha256()
    for src in srcs:
        if src is None:
            src_str = "None"
        elif isinstance(src, str):
            src_str = src
        elif isinstance(src, Path):
            src_str = src.read_text()
        elif isinstance(src, (types.FunctionType, type)):
...
```
**EN:** Function `hash_source` computes a digest or summary used for cache identity and reproducibility. It fits into vLLM's custom IR registry and provider system. Utility method to hash the sources of functions or objects.
**CN:** 函数 `hash_source` 计算用于缓存身份与可复现性的摘要或哈希，并嵌入vLLM 自定义 IR 注册与 provider 系统。 文档字符串说明：Utility method to hash the sources of functions or objects.

### Function `weak_lru_cache` (lines 36-56)
```python
def weak_lru_cache(maxsize: int | None = 128, typed: bool = False):
    """
    LRU Cache decorator that keeps a weak reference to 'self'.
    This avoids memory leakage, which happens when functools.lru_cache
    stores a reference to self in the global cache.

    Taken from: https://stackoverflow.com/a/68052994/5082708
    """

    def wrapper(func):
        @functools.lru_cache(maxsize, typed)
        def _func(_self, *args, **kwargs):
            return func(_self(), *args, **kwargs)

        @functools.wraps(func)
        def inner(self, *args, **kwargs):
...
```
**EN:** Function `weak_lru_cache` provides a focused helper used by the surrounding module. It fits into vLLM's custom IR registry and provider system. LRU Cache decorator that keeps a weak reference to 'self'.
**CN:** 函数 `weak_lru_cache` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 自定义 IR 注册与 provider 系统。 文档字符串说明：LRU Cache decorator that keeps a weak reference to 'self'.

### Function `weak_cache` (lines 59-61)
```python
def weak_cache(user_function, /):
    """Simple weak equivalent to functools.cache"""
    return weak_lru_cache(maxsize=None)(user_function)
```
**EN:** Function `weak_cache` provides a focused helper used by the surrounding module. It fits into vLLM's custom IR registry and provider system. Simple weak equivalent to functools.cache
**CN:** 函数 `weak_cache` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 自定义 IR 注册与 provider 系统。 文档字符串说明：Simple weak equivalent to functools.cache

## Key Concepts / 关键概念
- **Custom IR ops / 自定义 IR 算子**
  - **EN:** The code models vLLM-specific IR operations or lowers them to concrete providers.
  - **CN:** 这些代码建模了 vLLM 特有的 IR 算子，或将其降级到具体 provider。
- **Source hashing and weak caches / 源码哈希与弱缓存**
  - **EN:** Hashing and weak-reference caches help memoize lightweight metadata without creating ownership leaks.
  - **CN:** 哈希与弱引用缓存有助于记忆化轻量元数据，同时避免所有权泄漏。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `import functools`, `import hashlib`, `import inspect`, `import types`, `import weakref`, `from pathlib import Path`, `from typing import Any`
