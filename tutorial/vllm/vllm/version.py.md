# version.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `vllm/version.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Loads generated version metadata when available and provides helpers for “previous minor version” compatibility checks. / 在可用时加载生成的版本元数据，并提供“上一次次版本号”兼容性检查辅助函数。

## Line-by-Line Analysis / 逐行分析
### Generated-version import with dev fallback (lines 4-12)
```python
try:
    from ._version import __version__, __version_tuple__
except Exception as e:
    import warnings

    warnings.warn(f"Failed to read commit hash:\n{e}", RuntimeWarning, stacklevel=2)

    __version__ = "dev"
    __version_tuple__ = (0, 0, __version__)
```
**EN:** The module first tries to import `.__version__` and `.__version_tuple__` from a generated `_version` module. If that fails, it warns once and falls back to a `dev` marker. This keeps source-tree usage working even when build metadata has not been generated yet.
**CN:** 模块会先尝试从生成出来的 `_version` 模块导入 `__version__` 和 `__version_tuple__`。如果失败，就发出警告并回退到 `dev` 标记。这样即使构建阶段尚未生成版本元数据，直接在源码树中使用 vLLM 也不会中断。

### Previous-minor helper functions (lines 15-39)
```python
def _prev_minor_version_was(version_str):
    """Check whether a given version matches the previous minor version.

    Return True if version_str matches the previous minor version.

    For example - return True if the current version if 0.7.4 and the
    supplied version_str is '0.6'.

    Used for --show-hidden-metrics-for-version.
    """
    # Match anything if this is a dev tree
    if __version_tuple__[0:2] == (0, 0):
        return True

    # Note - this won't do the right thing when we release 1.0!
    assert __version_tuple__[0] == 0
    assert isinstance(__version_tuple__[1], int)
    return version_str == f"{__version_tuple__[0]}.{__version_tuple__[1] - 1}"


def _prev_minor_version():
    """For the purpose of testing, return a previous minor version number."""
    # In dev tree, this will return "0.-1", but that will work fine"
    assert isinstance(__version_tuple__[1], int)
    return f"{__version_tuple__[0]}.{__version_tuple__[1] - 1}"
```
**EN:** `_prev_minor_version_was()` checks whether a supplied string matches the immediately previous minor release, with a special case that treats dev trees as matching anything. `_prev_minor_version()` exposes the computed previous minor string directly for tests and hidden-metrics compatibility logic.
**CN:** `_prev_minor_version_was()` 用于判断给定字符串是否等于“当前版本的上一个次版本号”，并对开发树做了特殊处理：在 dev 状态下默认全部匹配。`_prev_minor_version()` 则直接返回这个上一个次版本号字符串，供测试和隐藏指标兼容逻辑使用。

## Key Concepts / 关键概念
- EN: The file separates build-time version generation from runtime fallback behavior.
- CN: 该文件把“构建时生成版本号”和“运行时回退策略”清晰分离开了。
- EN: Compatibility checks are intentionally simple and currently assume the major version is still `0`.
- CN: 兼容性检查逻辑刻意保持简单，目前默认主版本仍处于 `0`。

## Dependencies / 依赖关系
- EN: Depends on a generated sibling module `_version` when present and otherwise uses stdlib `warnings`.
- CN: 在存在时依赖同级生成模块 `_version`，否则只使用标准库 `warnings`。
