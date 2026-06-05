# utils.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `test/python/cutlass/interface/utils.py`
- **EN:** This helper file defines a reusable context manager for interface tests that need to assert whether an exception should occur and, optionally, whether its message matches exactly.
- **CN:** 该辅助文件定义了一个可复用的上下文管理器，供接口测试断言“是否应该抛出异常”，并且可选地校验异常消息是否完全匹配。

## Line-by-Line Analysis / 逐行分析

### Lines 33-35

```python
"""
Helper functions & classes for interface test
"""
```

**EN:** A short module docstring identifies the file as a shared test utility rather than a standalone test suite.

**CN:** 简短的模块文档字符串表明该文件是共享测试工具，而不是独立测试套件。

### Lines 36-56

```python
class ExpectException:
    """
    Utility class to assert that an exception was raised when expected

    Example:

    .. highlight:: python
    .. code-block:: python

        with ExceptionExpected(True, 'Division by zero'):
            x = 1.0 / 0.0

    :param exception_expected: whether an exception is expected to be raised
    :type exception_expected: bool
    :param message: message to print if an exception is raised when not expected or vice versa
    :type message: str
    """
    def __init__(self, exception_expected: bool, message: str = '', verify_msg=False):
        self.exception_expected = exception_expected
        self.message = message
        self.verify_msg = verify_msg
```

**EN:** `ExpectException` stores three pieces of state: whether an exception is expected, the reference message, and whether exact message matching is required. The class docstring also shows its intended `with`-statement usage pattern.

**CN:** `ExpectException` 保存三项状态：是否期望出现异常、参考消息文本，以及是否需要进行精确消息匹配。类文档字符串还展示了它在 `with` 语句中的预期使用方式。

### Lines 58-69

```python
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, traceback):
        exception_raised = exc_type is not None
        assert self.exception_expected == exception_raised, self.message
        if self.verify_msg:
            exc_message = f"{exc_type.__name__}: {exc_val}"
            assert exc_message == self.message, f"expect error message {self.message}, got {exc_message}"

        # Suppress the exception
        return True
```

**EN:** `__enter__` simply returns the context manager itself. `__exit__` checks whether an exception was raised, compares that against the expected boolean, optionally builds a normalized `"TypeName: message"` string for exact matching, and finally returns `True` to suppress the exception after validation.

**CN:** `__enter__` 直接返回上下文管理器本身。`__exit__` 会检查是否真的抛出了异常，将结果与预期布尔值比较；如果启用了精确匹配，还会拼出统一格式的 `"异常类型: 消息"` 字符串再进行比较，最后返回 `True` 以在验证完成后吞掉异常。

## Key Concepts / 关键概念
- **EN:** Python context-manager protocol via `__enter__` and `__exit__`.
  **CN:** 通过 `__enter__` 与 `__exit__` 实现 Python 上下文管理器协议。
- **EN:** Exception-presence assertions are separated from exact-message assertions.
  **CN:** “是否抛异常”的判断与“异常消息是否完全一致”的判断被拆开处理。
- **EN:** Returning `True` from `__exit__` prevents expected failures from terminating the test body.
  **CN:** `__exit__` 返回 `True` 后，符合预期的失败不会中断测试逻辑。

## Dependencies / 依赖关系
- **EN:** No imported modules are required; the helper relies only on Python exception protocol hooks.
  **CN:** 无需额外导入模块；该工具仅依赖 Python 的异常协议钩子。
- **EN:** Used by sibling CUTLASS interface tests such as `conv2d_interface.py`, `evt_interface.py`, and `gemm_interface.py`.
  **CN:** 被同目录下的 CUTLASS 接口测试使用，例如 `conv2d_interface.py`、`evt_interface.py` 和 `gemm_interface.py`。
