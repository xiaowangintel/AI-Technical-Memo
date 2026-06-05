# logging_tensor.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/logging_tensor.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for logging tensor, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 logging tensor 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```python
# mypy: ignore-errors

import torch
from torch.utils._pytree import tree_map
from collections.abc import Iterator
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.utils._pytree`; external imports: `collections.abc`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.utils._pytree`；外部导入：`collections.abc`。

### Lines 6-10
```python
import logging
import contextlib
import itertools
from torch.utils._dtype_abbrs import dtype_abbrs as _dtype_abbrs
from torch.utils._python_dispatch import TorchDispatchMode
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.utils._dtype_abbrs`, `torch.utils._python_dispatch`; external imports: `logging`, `contextlib`, `itertools`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.utils._dtype_abbrs`, `torch.utils._python_dispatch`；外部导入：`logging`, `contextlib`, `itertools`。

### Lines 11-16
```python
from torch.utils.weak import WeakTensorKeyDictionary
import functools
from torch._C._profiler import gather_traceback, symbolize_tracebacks

logger = logging.getLogger("LoggingTensor")

```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.utils.weak`, `torch._C._profiler`; external imports: `functools`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.utils.weak`, `torch._C._profiler`；外部导入：`functools`。

### Lines 17-22
```python
# How the chain of calls works for LoggingTensor:
# 1. Call torch.sin
# 2. Attempt __torch_function__. In LoggingTensor torch function is disabled so we bypass it entirely
# 3. Enter dispatcher, wind your way through Autograd
# 4. Hit Python dispatch key, call __torch_dispatch__

```
- EN: Documents intent, assumptions, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、假设或兼容性约束。

### Lines 23-30
```python
# This Tensor can work with autograd in two ways:
#  - The wrapped Tensor does not require gradients. In that case, the LoggingTensor
#    can require gradients if the user asks for it as a constructor kwarg.
#  - The wrapped Tensor can require gradients. In that case autograd will be tracked
#    for the wrapped Tensor and the LoggingTensor itself cannot require gradients.
# WARNING: We allow these two possibilities for testing purposes. You should NEVER use both in a single
# test or you might get surprising behavior.

```
- EN: Documents intent, assumptions, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、假设或兼容性约束。

### Lines 31-36
```python
# TODO: TensorBase should work
class LoggingTensor(torch.Tensor):
    elem: torch.Tensor

    __slots__ = ['elem']

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `LoggingTensor`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`LoggingTensor`。

### Lines 37-46
```python
    context = contextlib.nullcontext

    @staticmethod
    def __new__(cls, elem, *args, **kwargs):
        # The wrapping tensor (LoggingTensor) shouldn't hold any
        # memory for the class in question, but it should still
        # advertise the same device as before
        r = torch.Tensor._make_wrapper_subclass(
            cls, elem.size(),
            strides=elem.stride(), storage_offset=elem.storage_offset(),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__new__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__new__`。

### Lines 47-53
```python
            # TODO: clone storage aliasing
            dtype=elem.dtype, layout=elem.layout,
            device=elem.device, requires_grad=kwargs.get("requires_grad", False)
        )
        # ...the real tensor is held as an element on the tensor.
        r.elem = elem.detach() if r.requires_grad else elem
        return r
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 54-58
```python

    def __repr__(self):
        return super().__repr__(tensor_contents=f"{self.elem}")

    @classmethod
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__repr__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__repr__`。

### Lines 59-63
```python
    def __torch_dispatch__(cls, func, types, args=(), kwargs=None):
        def unwrap(e):
            return e.elem if isinstance(e, cls) else e

        def wrap(e):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__torch_dispatch__`, `unwrap`, `wrap`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__torch_dispatch__`, `unwrap`, `wrap`。

### Lines 64-69
```python
            return cls(e) if isinstance(e, torch.Tensor) else e

        with cls.context():
            rs = tree_map(wrap, func(*tree_map(unwrap, args), **tree_map(unwrap, kwargs)))
        logging.getLogger("LoggingTensor").info(f"{func.__module__}.{func.__name__}", args, kwargs, rs)  # noqa: G004
        return rs
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 70-77
```python

class LoggingTensorMode(TorchDispatchMode):
    def __torch_dispatch__(self, func, types, args=(), kwargs=None):
        if kwargs is None:
            kwargs = {}
        rs = func(*args, **kwargs)
        logging.getLogger("LoggingTensor").info(f"{func.__module__}.{func.__name__}", args, kwargs, rs)  # noqa: G004
        return rs
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `LoggingTensorMode`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`LoggingTensorMode`。

### Lines 78-83
```python

class LoggingTensorReentrant(LoggingTensor):
    context = torch.overrides.enable_reentrant_dispatch

# https://stackoverflow.com/questions/36408496/python-logging-handler-to-append-to-list
class LoggingTensorHandler(logging.Handler):
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `LoggingTensorReentrant`, `LoggingTensorHandler`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`LoggingTensorReentrant`, `LoggingTensorHandler`。

### Lines 84-93
```python
    def __init__(
            self, log_list: list[str], use_shortid_for_all_tensors: bool,
            with_type: bool, tracebacks_list: list | None) -> None:
        logging.Handler.__init__(self)
        self.log_list = log_list
        self.use_shortid_for_all_tensors = use_shortid_for_all_tensors
        self.tracebacks_list = tracebacks_list
        self.memo = WeakTensorKeyDictionary()
        self.next_id = 0
        self.with_type = with_type
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 94-99
```python

    def _shortid(self, t: torch.Tensor) -> int:
        if t not in self.memo:
            self.memo[t] = self.next_id
            self.next_id += 1
        return self.memo[t]
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_shortid`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_shortid`。

### Lines 100-105
```python

    def _fmt(self, a: object, with_type: bool = False) -> str:
        cond_cls = torch.Tensor if self.use_shortid_for_all_tensors else LoggingTensor
        if isinstance(a, cond_cls):
            maybe_type = ""
            if with_type and self.with_type:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_fmt`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_fmt`。

### Lines 106-110
```python
                maybe_type = f": {_dtype_abbrs[a.dtype]}[{', '.join(map(str, a.shape))}]"
            x = f"${self._shortid(a)}{maybe_type}"
            return x
        else:
            return repr(a)
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 111-120
```python

    def emit(self, record):
        fmt_args = ", ".join(
            itertools.chain(
                (str(tree_map(self._fmt, a)) for a in record.args[0]),
                (f"{k}={str(tree_map(self._fmt, v))}" for k, v in record.args[1].items()),
            )
        )
        fmt_rets = tree_map(functools.partial(self._fmt, with_type=True), record.args[2])
        self.log_list.append(f'{fmt_rets} = {record.msg}({fmt_args})')
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `emit`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`emit`。

### Lines 121-126
```python
        if self.tracebacks_list is not None:
            self.tracebacks_list.append(record.traceback)

def log_input(name: str, var: object) -> None:
    logger.info("input", (name,), {}, var)  # noqa: PLE1205

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `log_input`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`log_input`。

### Lines 127-132
```python
class GatherTraceback(logging.Filter):
    def __init__(self, python=True, script=True, cpp=False):
        self.python = python
        self.script = script
        self.cpp = cpp

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `GatherTraceback`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`GatherTraceback`。

### Lines 133-137
```python
    def filter(self, record):
        record.traceback = gather_traceback(python=self.python, script=self.script, cpp=self.cpp)
        return True

@contextlib.contextmanager
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `filter`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`filter`。

### Lines 138-147
```python
def capture_logs(is_mode=False, python_tb=False, script_tb=False, cpp_tb=False) -> Iterator[list[str]]:
    collect_traceback = python_tb or script_tb or cpp_tb
    log_list: list[str] = []
    tracebacks_list: list[str] = []
    handler = LoggingTensorHandler(
        log_list,
        with_type=True,
        use_shortid_for_all_tensors=is_mode,
        tracebacks_list=tracebacks_list if collect_traceback else None
    )
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `capture_logs`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`capture_logs`。

### Lines 148-154
```python
    logger.addHandler(handler)
    logger.setLevel(logging.INFO)
    logger.propagate = False
    if collect_traceback:
        logger.addFilter(GatherTraceback(python=python_tb, script=script_tb, cpp=cpp_tb))
    try:
        if collect_traceback:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 155-163
```python
            yield log_list, tracebacks_list
        else:
            yield log_list
    finally:
        symbolized_tracebacks = symbolize_tracebacks(tracebacks_list)
        tracebacks_list.clear()
        tracebacks_list.extend(symbolized_tracebacks)
        logger.removeHandler(handler)

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 164-167
```python
@contextlib.contextmanager
def capture_logs_with_logging_tensor_mode(python_tb=False, script_tb=False, cpp_tb=False):
    with LoggingTensorMode(), capture_logs(True, python_tb, script_tb, cpp_tb) as logs:
        yield logs
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `capture_logs_with_logging_tensor_mode`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`capture_logs_with_logging_tensor_mode`。


## Key Concepts / 关键概念
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Test maintainability — the file helps standardize expectations, fixtures, or generated coverage across PyTorch tests.
  CN: 测试可维护性——该文件有助于在 PyTorch 测试中统一期望、夹具或生成式覆盖。
- EN: Module integration — the implementation coordinates neighboring Python modules and utility layers.
  CN: 模块集成——该实现会协调相邻的 Python 模块与工具层。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.utils._pytree`, `torch.utils._dtype_abbrs`, `torch.utils._python_dispatch`, `torch.utils.weak`, `torch._C._profiler`
- External imports / 外部导入: `collections.abc`, `logging`, `contextlib`, `itertools`, `functools`
- Representative symbols / 代表性符号: `LoggingTensor`, `LoggingTensorMode`, `LoggingTensorReentrant`, `LoggingTensorHandler`, `log_input`, `GatherTraceback`, `capture_logs`, `capture_logs_with_logging_tensor_mode`
