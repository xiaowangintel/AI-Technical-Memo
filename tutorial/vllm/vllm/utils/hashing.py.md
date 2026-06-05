# hashing.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/hashing.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides reusable utility helpers shared across vLLM components. / 提供 vLLM 各组件复用的通用工具函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-23)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from __future__ import annotations

import hashlib
import pickle
from _hashlib import HASH, UnsupportedDigestmodError
from collections.abc import Callable
from typing import Any

import cbor2

try:
    # It is important that this remains an optional dependency.
    # It would not be allowed in environments with strict security controls,
    # so it's best not to have it installed when not in use.
    import xxhash as _xxhash

    if not hasattr(_xxhash, "xxh3_128_digest"):
        _xxhash = None
except ImportError:  # pragma: no cover
    _xxhash = None
```
**EN:** Sets up the module with standard-library support such as `__future__`, `hashlib`, `pickle`, external packages such as `cbor2`, `xxhash`. It prepares the symbols later used by `sha256`, `sha256_cbor`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义。 这些准备工作为后续的 `sha256`, `sha256_cbor` 提供上下文。

### sha256 (lines 26-40)
```python
def sha256(input: Any) -> bytes:
    """Hash any picklable Python object using SHA-256.

    The input is serialized using pickle before hashing, which allows
    arbitrary Python objects to be used. Note that this function does
    not use a hash seed—if you need one, prepend it explicitly to the input.

    Args:
        input: Any picklable Python object.

    Returns:
        Bytes representing the SHA-256 hash of the serialized input.
    """
    input_bytes = pickle.dumps(input, protocol=pickle.HIGHEST_PROTOCOL)
    return hashlib.sha256(input_bytes).digest()
```
**EN:** `sha256`: Hash any picklable Python object using SHA-256. It mainly works with `input`. Inside the body, it relies on `pickle.dumps`, `hashlib.sha256.digest`, `hashlib.sha256` to complete the main steps.
**CN:** `sha256` 负责实现本模块使用的辅助逻辑。 它主要处理 `input` 等参数。 实现过程中会调用 `pickle.dumps`, `hashlib.sha256.digest`, `hashlib.sha256` 等函数完成关键步骤。

### sha256_cbor (lines 43-58)
```python
def sha256_cbor(input: Any) -> bytes:
    """Hash objects using CBOR serialization and SHA-256.

    This option is useful for non-Python-dependent serialization and hashing.

    Args:
        input: Object to be serialized and hashed. Supported types include
            basic Python types and complex structures like lists, tuples, and
            dictionaries.
            Custom classes must implement CBOR serialization methods.

    Returns:
        Bytes representing the SHA-256 hash of the CBOR serialized input.
    """
    input_bytes = cbor2.dumps(input, canonical=True)
    return hashlib.sha256(input_bytes).digest()
```
**EN:** `sha256_cbor`: Hash objects using CBOR serialization and SHA-256. It mainly works with `input`. Inside the body, it relies on `cbor2.dumps`, `hashlib.sha256.digest`, `hashlib.sha256` to complete the main steps.
**CN:** `sha256_cbor` 负责实现本模块使用的辅助逻辑。 它主要处理 `input` 等参数。 实现过程中会调用 `cbor2.dumps`, `hashlib.sha256.digest`, `hashlib.sha256` 等函数完成关键步骤。

### _xxhash_digest (lines 61-67)
```python
def _xxhash_digest(input_bytes: bytes) -> bytes:
    if _xxhash is None:
        raise ModuleNotFoundError(
            "xxhash is required for the 'xxhash' prefix caching hash algorithms. "
            "Install it via `pip install xxhash`."
        )
    return _xxhash.xxh3_128_digest(input_bytes)
```
**EN:** `_xxhash_digest` implements helper logic used by this module. It mainly works with `input_bytes`. Inside the body, it relies on `_xxhash.xxh3_128_digest`, `ModuleNotFoundError` to complete the main steps.
**CN:** `_xxhash_digest` 负责实现本模块使用的辅助逻辑。 它主要处理 `input_bytes` 等参数。 实现过程中会调用 `_xxhash.xxh3_128_digest`, `ModuleNotFoundError` 等函数完成关键步骤。

### xxhash (lines 70-73)
```python
def xxhash(input: Any) -> bytes:
    """Hash picklable objects using xxHash."""
    input_bytes = pickle.dumps(input, protocol=pickle.HIGHEST_PROTOCOL)
    return _xxhash_digest(input_bytes)
```
**EN:** `xxhash`: Hash picklable objects using xxHash. It mainly works with `input`. Inside the body, it relies on `pickle.dumps`, `_xxhash_digest` to complete the main steps.
**CN:** `xxhash` 负责实现本模块使用的辅助逻辑。 它主要处理 `input` 等参数。 实现过程中会调用 `pickle.dumps`, `_xxhash_digest` 等函数完成关键步骤。

### xxhash_cbor (lines 76-79)
```python
def xxhash_cbor(input: Any) -> bytes:
    """Hash objects serialized with CBOR using xxHash."""
    input_bytes = cbor2.dumps(input, canonical=True)
    return _xxhash_digest(input_bytes)
```
**EN:** `xxhash_cbor`: Hash objects serialized with CBOR using xxHash. It mainly works with `input`. Inside the body, it relies on `cbor2.dumps`, `_xxhash_digest` to complete the main steps.
**CN:** `xxhash_cbor` 负责实现本模块使用的辅助逻辑。 它主要处理 `input` 等参数。 实现过程中会调用 `cbor2.dumps`, `_xxhash_digest` 等函数完成关键步骤。

### get_hash_fn_by_name (lines 82-100)
```python
def get_hash_fn_by_name(hash_fn_name: str) -> Callable[[Any], bytes]:
    """Get a hash function by name, or raise an error if the function is not found.

    Args:
        hash_fn_name: Name of the hash function.

    Returns:
        A hash function.
    """
    if hash_fn_name == "sha256":
        return sha256
    if hash_fn_name == "sha256_cbor":
        return sha256_cbor
    if hash_fn_name == "xxhash":
        return xxhash
    if hash_fn_name == "xxhash_cbor":
        return xxhash_cbor

    raise ValueError(f"Unsupported hash function: {hash_fn_name}")
```
**EN:** `get_hash_fn_by_name`: Get a hash function by name, or raise an error if the function is not found. It mainly works with `hash_fn_name`. Inside the body, it relies on `ValueError` to complete the main steps.
**CN:** `get_hash_fn_by_name` 负责获取流水线所需的数据或状态。 它主要处理 `hash_fn_name` 等参数。 实现过程中会调用 `ValueError` 等函数完成关键步骤。

### safe_hash (lines 103-117)
```python
def safe_hash(data: bytes, usedforsecurity: bool = True) -> HASH:
    """Hash for configs, defaulting to md5 but falling back to sha256
    in FIPS constrained environments.

    Args:
        data: bytes
        usedforsecurity: Whether the hash is used for security purposes

    Returns:
        Hash object
    """
    try:
        return hashlib.md5(data, usedforsecurity=usedforsecurity)
    except (UnsupportedDigestmodError, ValueError):
        return hashlib.sha256(data)
```
**EN:** `safe_hash`: Hash for configs, defaulting to md5 but falling back to sha256 in FIPS constrained environments. It mainly works with `data`, `usedforsecurity`. Inside the body, it relies on `hashlib.md5`, `hashlib.sha256` to complete the main steps.
**CN:** `safe_hash` 负责实现本模块使用的辅助逻辑。 它主要处理 `data`, `usedforsecurity` 等参数。 实现过程中会调用 `hashlib.md5`, `hashlib.sha256` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`sha256`**: Key helper or entry point in this file. / **`sha256`**：本文件中的关键辅助函数或入口。
- **`sha256_cbor`**: Key helper or entry point in this file. / **`sha256_cbor`**：本文件中的关键辅助函数或入口。
- **`_xxhash_digest`**: Key helper or entry point in this file. / **`_xxhash_digest`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: __future__, hashlib, pickle, _hashlib, collections.abc, typing
- **Third-party / 第三方**: cbor2, xxhash
- **Internal vLLM / vLLM 内部依赖**: None / 无
