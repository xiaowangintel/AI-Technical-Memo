# logging_utils.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/logging_utils.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for logging utils, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 logging utils 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
# mypy: ignore-errors

import torch._dynamo.test_case
import unittest.mock
import os
import contextlib
import torch._logging
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch._dynamo.test_case`, `torch._logging`; external imports: `unittest.mock`, `os`, `contextlib`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch._dynamo.test_case`, `torch._logging`；外部导入：`unittest.mock`, `os`, `contextlib`。

### Lines 8-14
```python
import torch._logging._internal
from contextlib import AbstractContextManager
from collections.abc import Callable
from torch._dynamo.utils import LazyString
from torch._inductor import config as inductor_config
import logging
import io
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch._logging._internal`, `torch._dynamo.utils`, `torch._inductor`; external imports: `contextlib`, `collections.abc`, `logging`, `io`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch._logging._internal`, `torch._dynamo.utils`, `torch._inductor`；外部导入：`contextlib`, `collections.abc`, `logging`, `io`。

### Lines 15-25
```python

@contextlib.contextmanager
def preserve_log_state():
    prev_state = torch._logging._internal._get_log_state()
    torch._logging._internal._set_log_state(torch._logging._internal.LogState())
    try:
        yield
    finally:
        torch._logging._internal._set_log_state(prev_state)
        torch._logging._internal._init_logs()

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `preserve_log_state`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`preserve_log_state`。

### Lines 26-32
```python
def log_settings(settings):
    exit_stack = contextlib.ExitStack()
    settings_patch = unittest.mock.patch.dict(os.environ, {"TORCH_LOGS": settings})
    exit_stack.enter_context(preserve_log_state())
    exit_stack.enter_context(settings_patch)
    torch._logging._internal._init_logs()
    return exit_stack
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `log_settings`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`log_settings`。

### Lines 33-39
```python

def log_api(**kwargs):
    exit_stack = contextlib.ExitStack()
    exit_stack.enter_context(preserve_log_state())
    torch._logging.set_logs(**kwargs)
    return exit_stack

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `log_api`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`log_api`。

### Lines 40-46
```python

def kwargs_to_settings(**kwargs):
    INT_TO_VERBOSITY = {10: "+", 20: "", 40: "-"}

    settings = []

    def append_setting(name, level):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `kwargs_to_settings`, `append_setting`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`kwargs_to_settings`, `append_setting`。

### Lines 47-53
```python
        if isinstance(name, str) and isinstance(level, int) and level in INT_TO_VERBOSITY:
            settings.append(INT_TO_VERBOSITY[level] + name)
            return
        else:
            raise ValueError("Invalid value for setting")

    for name, val in kwargs.items():
```
- EN: This block reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 54-63
```python
        if isinstance(val, bool):
            settings.append(name)
        elif isinstance(val, int):
            append_setting(name, val)
        elif isinstance(val, dict) and name == "modules":
            for module_qname, level in val.items():
                append_setting(module_qname, level)
        else:
            raise ValueError("Invalid value for setting")

```
- EN: This block reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 64-77
```python
    return ",".join(settings)


# Note on testing strategy:
# This class does two things:
# 1. Runs two versions of a test:
#    1a. patches the env var log settings to some specific value
#    1b. calls torch._logging.set_logs(..)
# 2. patches the emit method of each setup handler to gather records
# that are emitted to each console stream
# 3. passes a ref to the gathered records to each test case for checking
#
# The goal of this testing in general is to ensure that given some settings env var
# that the logs are setup correctly and capturing the correct records.
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 78-86
```python
def make_logging_test(**kwargs):
    def wrapper(fn):
        @inductor_config.patch({"fx_graph_cache": False})
        def test_fn(self):

            torch._dynamo.reset()
            records = []
            # run with env var
            if len(kwargs) == 0:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `make_logging_test`, `wrapper`, `test_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`make_logging_test`, `wrapper`, `test_fn`。

### Lines 87-98
```python
                with self._handler_watcher(records):
                    fn(self, records)
            else:
                with log_settings(kwargs_to_settings(**kwargs)), self._handler_watcher(records):
                    fn(self, records)

            # run with API
            torch._dynamo.reset()
            records.clear()
            with log_api(**kwargs), self._handler_watcher(records):
                fn(self, records)

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 99-105
```python

        return test_fn

    return wrapper

def make_settings_test(settings):
    def wrapper(fn):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `make_settings_test`, `wrapper`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`make_settings_test`, `wrapper`。

### Lines 106-112
```python
        def test_fn(self):
            torch._dynamo.reset()
            records = []
            # run with env var
            with log_settings(settings), self._handler_watcher(records):
                fn(self, records)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `test_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`test_fn`。

### Lines 113-119
```python
        return test_fn

    return wrapper

class LoggingTestCase(torch._dynamo.test_case.TestCase):
    @classmethod
    def setUpClass(cls):
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `LoggingTestCase`, `setUpClass`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`LoggingTestCase`, `setUpClass`。

### Lines 120-130
```python
        super().setUpClass()
        cls._exit_stack.enter_context(
            unittest.mock.patch.dict(os.environ, {"___LOG_TESTING": ""})
        )
        cls._exit_stack.enter_context(
            unittest.mock.patch("torch._dynamo.config.suppress_errors", True)
        )
        cls._exit_stack.enter_context(
            unittest.mock.patch("torch._dynamo.config.verbose", False)
        )

```
- EN: This block reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 131-137
```python
    @classmethod
    def tearDownClass(cls):
        cls._exit_stack.close()
        torch._logging._internal.log_state.clear()
        torch._logging._init_logs()

    def hasRecord(self, records, m):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `tearDownClass`, `hasRecord`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`tearDownClass`, `hasRecord`。

### Lines 138-145
```python
        return any(m in r.getMessage() for r in records)

    def getRecord(self, records, m):
        record = None
        for r in records:
            # NB: not r.msg because it looks like 3.11 changed how they
            # structure log records
            if m in r.getMessage():
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `getRecord`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`getRecord`。

### Lines 146-153
```python
                self.assertIsNone(
                    record,
                    msg=LazyString(
                        lambda: f"multiple matching records: {record} and {r} among {records}"
                    ),
                )
                record = r
        if record is None:
```
- EN: This block checks invariants or expected outcomes; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理条件控制流。关键符号：无明显局部符号。

### Lines 154-161
```python
            self.fail(f"did not find record with {m} among {records}")
        return record

    # This patches the emit method of each handler to gather records
    # as they are emitted
    def _handler_watcher(self, record_list):
        exit_stack = contextlib.ExitStack()

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_handler_watcher`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_handler_watcher`。

### Lines 162-175
```python
        def emit_post_hook(record):
            nonlocal record_list
            record_list.append(record)

        # registered logs are the only ones with handlers, so patch those
        for log_qname in torch._logging._internal.log_registry.get_log_qnames():
            logger = logging.getLogger(log_qname)
            num_handlers = len(logger.handlers)
            self.assertLessEqual(
                num_handlers,
                2,
                "All pt2 loggers should only have at most two handlers (debug artifacts and messages above debug level).",
            )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `emit_post_hook`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`emit_post_hook`。

### Lines 176-184
```python
            self.assertGreater(num_handlers, 0, "All pt2 loggers should have more than zero handlers")

            for handler in logger.handlers:
                old_emit = handler.emit

                def new_emit(record):
                    old_emit(record)
                    emit_post_hook(record)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `new_emit`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`new_emit`。

### Lines 185-191
```python
                exit_stack.enter_context(
                    unittest.mock.patch.object(handler, "emit", new_emit)
                )

        return exit_stack


```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 192-200
```python
def logs_to_string(module, log_option):
    """Example:
    logs_to_string("torch._inductor.compile_fx", "post_grad_graphs")
    returns the output of TORCH_LOGS="post_grad_graphs" from the
    torch._inductor.compile_fx module.
    """
    log_stream = io.StringIO()
    handler = logging.StreamHandler(stream=log_stream)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `logs_to_string`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`logs_to_string`。

### Lines 201-209
```python
    @contextlib.contextmanager
    def tmp_redirect_logs():
        try:
            logger = torch._logging.getArtifactLogger(module, log_option)
            logger.addHandler(handler)
            yield
        finally:
            logger.removeHandler(handler)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `tmp_redirect_logs`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`tmp_redirect_logs`。

### Lines 210-216
```python
    def ctx_manager():
        exit_stack = log_settings(log_option)
        exit_stack.enter_context(tmp_redirect_logs())
        return exit_stack

    return log_stream, ctx_manager

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `ctx_manager`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`ctx_manager`。

### Lines 217-226
```python

def multiple_logs_to_string(module: str, *log_options: str) -> tuple[list[io.StringIO], Callable[[], AbstractContextManager[None]]]:
    """Example:
    multiple_logs_to_string("torch._inductor.compile_fx", "pre_grad_graphs", "post_grad_graphs")
    returns the output of TORCH_LOGS="pre_graph_graphs, post_grad_graphs" from the
    torch._inductor.compile_fx module.
    """
    log_streams = [io.StringIO() for _ in range(len(log_options))]
    handlers = [logging.StreamHandler(stream=log_stream) for log_stream in log_streams]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `multiple_logs_to_string`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`multiple_logs_to_string`。

### Lines 227-235
```python
    @contextlib.contextmanager
    def tmp_redirect_logs():
        loggers = [torch._logging.getArtifactLogger(module, option) for option in log_options]
        try:
            for logger, handler in zip(loggers, handlers, strict=True):
                logger.addHandler(handler)
            yield
        finally:
            for logger, handler in zip(loggers, handlers, strict=True):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `tmp_redirect_logs`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`tmp_redirect_logs`。

### Lines 236-242
```python
                logger.removeHandler(handler)

    def ctx_manager() -> AbstractContextManager[None]:
        exit_stack = log_settings(", ".join(log_options))
        exit_stack.enter_context(tmp_redirect_logs())
        return exit_stack  # type: ignore[return-value]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `ctx_manager`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`ctx_manager`。

### Lines 243-243
```python
    return log_streams, ctx_manager
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Backend lowering — the code bridges higher-level programs to executable backend artifacts.
  CN: 后端降级——代码把更高层程序桥接到可执行的后端产物。
- EN: Test maintainability — the file helps standardize expectations, fixtures, or generated coverage across PyTorch tests.
  CN: 测试可维护性——该文件有助于在 PyTorch 测试中统一期望、夹具或生成式覆盖。
- EN: Module integration — the implementation coordinates neighboring Python modules and utility layers.
  CN: 模块集成——该实现会协调相邻的 Python 模块与工具层。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch._dynamo.test_case`, `torch._logging`, `torch._logging._internal`, `torch._dynamo.utils`, `torch._inductor`
- External imports / 外部导入: `unittest.mock`, `os`, `contextlib`, `collections.abc`, `logging`, `io`
- Representative symbols / 代表性符号: `preserve_log_state`, `log_settings`, `log_api`, `kwargs_to_settings`, `make_logging_test`, `make_settings_test`, `LoggingTestCase`, `logs_to_string`, `multiple_logs_to_string`
