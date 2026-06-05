# logging_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/utils/logging_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for logging utils in the multimodal generation stack. Key symbols include `ColoredFormatter`, `SortedHelpFormatter`, `_print_info_once`. / 该模块包含多模态生成体系中与 logging utils 相关的运行时支持代码。 关键符号包括 `ColoredFormatter`, `SortedHelpFormatter`, `_print_info_once`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-72: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# adapted from vllm: https://github.com/vllm-project/vllm/blob/v0.7.3/vllm/logger.py
"""Logging configuration for sglang.multimodal_gen."""

import argparse
import contextlib
import dataclasses
import datetime
import inspect
import logging
import os
import sys
# ...
    },
    "version": 1,
    "disable_existing_loggers": False,
}
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 75-92: Class `ColoredFormatter` / 类 `ColoredFormatter`
```python
class ColoredFormatter(logging.Formatter):
    """A logging formatter that adds color to log levels."""

    LEVEL_COLORS = {
        logging.ERROR: RED,
        logging.WARNING: YELLOW,
    }

    def format(self, record: logging.LogRecord) -> str:
        """Adds color to the log"""

        formatted_message = super().format(record)

        color = self.LEVEL_COLORS.get(record.levelno)
        if color:
            formatted_message = f"{color}{formatted_message}{RESET}"

        return formatted_message
```
**EN:** This class models `ColoredFormatter` as a specialization of `logging.Formatter`. A logging formatter that adds color to log levels. Important methods include `format`.
**CN:** 该类实现 `ColoredFormatter`，并继承/扩展 `logging.Formatter`。 文档字符串指出：A logging formatter that adds color to log levels. 其中较重要的方法包括 `format`。

### Lines 95-100: Class `SortedHelpFormatter` / 类 `SortedHelpFormatter`
```python
class SortedHelpFormatter(argparse.HelpFormatter):
    """SortedHelpFormatter that sorts arguments by their option strings."""

    def add_arguments(self, actions):
        actions = sorted(actions, key=lambda x: x.option_strings)
        super().add_arguments(actions)
```
**EN:** This class models `SortedHelpFormatter` as a specialization of `argparse.HelpFormatter`. SortedHelpFormatter that sorts arguments by their option strings. Important methods include `add_arguments`.
**CN:** 该类实现 `SortedHelpFormatter`，并继承/扩展 `argparse.HelpFormatter`。 文档字符串指出：SortedHelpFormatter that sorts arguments by their option strings. 其中较重要的方法包括 `add_arguments`。

### Lines 103-106: Function `_print_info_once` / 函数 `_print_info_once`
```python
@lru_cache
def _print_info_once(logger: Logger, msg: str) -> None:
    # Set the stacklevel to 2 to print the original caller's line info
    logger.info(msg, stacklevel=2)
```
**EN:** This function drives `_print_info_once` with inputs such as `logger`, `msg`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_print_info_once`，主要处理 `logger`, `msg` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 109-112: Function `_print_warning_once` / 函数 `_print_warning_once`
```python
@lru_cache
def _print_warning_once(logger: Logger, msg: str) -> None:
    # Set the stacklevel to 2 to print the original caller's line info
    logger.warning(msg, stacklevel=2)
```
**EN:** This function drives `_print_warning_once` with inputs such as `logger`, `msg`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_print_warning_once`，主要处理 `logger`, `msg` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 115-120: Function `get_is_main_process` / 函数 `get_is_main_process`
```python
def get_is_main_process():
    try:
        rank = int(os.environ["RANK"])
    except (KeyError, ValueError):
        rank = 0
    return rank == 0
```
**EN:** This function drives `get_is_main_process`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_is_main_process`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 123-128: Function `get_is_local_main_process` / 函数 `get_is_local_main_process`
```python
def get_is_local_main_process():
    try:
        rank = int(os.environ["LOCAL_RANK"])
    except (KeyError, ValueError):
        rank = 0
    return rank == 0
```
**EN:** This function drives `get_is_local_main_process`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_is_local_main_process`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 131-158: Function `_log_process_aware` / 函数 `_log_process_aware`
```python
def _log_process_aware(
    server_log_level: int,
    level: int,
    logger_self: Logger,
    msg: object,
    *args: Any,
    main_process_only: bool,
    local_main_process_only: bool,
    **kwargs: Any,
) -> None:
    """Helper function to log a message if the process rank matches the criteria."""
    is_main_process = get_is_main_process()
    is_local_main_process = get_is_local_main_process()
    should_log = (
# ...
        if "stacklevel" in kwargs:
            logger_self.log(level, msg, *args, **kwargs)
        else:
            logger_self.log(level, msg, *args, stacklevel=3, **kwargs)
```
**EN:** This function drives `_log_process_aware` with inputs such as `server_log_level`, `level`, `logger_self`, `msg`. Helper function to log a message if the process rank matches the criteria.
**CN:** 这个函数负责 `_log_process_aware`，主要处理 `server_log_level`, `level`, `logger_self`, `msg` 等输入。 文档字符串说明：Helper function to log a message if the process rank matches the criteria.

### Lines 161-218: Class `_SGLDiffusionLogger` / 类 `_SGLDiffusionLogger`
```python
class _SGLDiffusionLogger(Logger):
    """
    Note:
        This class is just to provide type information.
        We actually patch the methods directly on the :class:`logging.Logger`
        instance to avoid conflicting with other libraries such as
        `intel_extension_for_pytorch.utils._logger`.
    """

    def info_once(self, msg: str) -> None:
        """
        As :meth:`info`, but subsequent calls with the same message
        are silently dropped.
        """
# ...
        main_process_only: bool = False,
        local_main_process_only: bool = True,
        **kwargs: Any,
    ) -> None: ...
```
**EN:** This class models `_SGLDiffusionLogger` as a specialization of `Logger`. Note: Important methods include `info_once`, `warning_once`, `info`, `debug`.
**CN:** 该类实现 `_SGLDiffusionLogger`，并继承/扩展 `Logger`。 文档字符串指出：Note: 其中较重要的方法包括 `info_once`, `warning_once`, `info`, `debug`。

### Lines 221-281: Function `init_logger` / 函数 `init_logger`
```python
def init_logger(name: str) -> _SGLDiffusionLogger:
    """The main purpose of this function is to ensure that loggers are
    retrieved in such a way that we can be sure the root sgl_diffusion logger has
    already been configured."""

    logger = logging.getLogger(name)

    server_log_level = logger.getEffectiveLevel()

    # Patch instance methods
    setattr(logger, "info_once", MethodType(_print_info_once, logger))
    setattr(logger, "warning_once", MethodType(_print_warning_once, logger))

    def _create_patched_method(
# ...
        MethodType(_create_patched_method(logging.ERROR, False, False), logger),
    )

    return cast(_SGLDiffusionLogger, logger)
```
**EN:** This function drives `init_logger` with inputs such as `name`. The main purpose of this function is to ensure that loggers are
**CN:** 这个函数负责 `init_logger`，主要处理 `name` 等输入。 文档字符串说明：The main purpose of this function is to ensure that loggers are

### Lines 282-284: Top-level configuration / 顶层配置
```python


logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 287-294: Function `_is_torch_tensor` / 函数 `_is_torch_tensor`
```python
def _is_torch_tensor(obj: Any) -> tuple[bool, Any]:
    """Return (is_tensor, torch_module_or_None) without importing torch at module import time."""
    try:
        import torch  # type: ignore

        return isinstance(obj, torch.Tensor), torch
    except Exception:
        return False, None
```
**EN:** This function drives `_is_torch_tensor` with inputs such as `obj`. Return (is_tensor, torch_module_or_None) without importing torch at module import time.
**CN:** 这个函数负责 `_is_torch_tensor`，主要处理 `obj` 等输入。 文档字符串说明：Return (is_tensor, torch_module_or_None) without importing torch at module import time.

### Lines 297-385: Function `_sanitize_for_logging` / 函数 `_sanitize_for_logging`
```python
def _sanitize_for_logging(obj: Any, key_hint: str | None = None) -> Any:
    """Recursively convert objects to JSON-serializable forms for concise logging.

    Rules:
    - Drop any field/dict key named 'param_names_mapping'.
    - Render Enums using their value.
    - Render torch.Tensor as a compact summary; if key name is 'scaling_factor', include stats.
    - Dataclasses are expanded to dicts and sanitized recursively.
    - Callables/functions are rendered as their qualified name.
    - Redact sensitive fields like 'prompt' and 'negative_prompt' (only show length).
    - Fallback to str(...) for unknown types.
    """
    if obj is None or isinstance(obj, (str, int, float, bool)):
        if key_hint in ("prompt", "negative_prompt"):
# ...
    try:
        return str(obj)
    except Exception:
        return "<unserializable>"
```
**EN:** This function drives `_sanitize_for_logging` with inputs such as `obj`, `key_hint`. Recursively convert objects to JSON-serializable forms for concise logging.
**CN:** 这个函数负责 `_sanitize_for_logging`，主要处理 `obj`, `key_hint` 等输入。 文档字符串说明：Recursively convert objects to JSON-serializable forms for concise logging.

### Lines 388-428: Function `_trace_calls` / 函数 `_trace_calls`
```python
def _trace_calls(log_path, root_dir, frame, event, arg=None):
    if event in ["call", "return"]:
        # Extract the filename, line number, function name, and the code object
        filename = frame.f_code.co_filename
        lineno = frame.f_lineno
        func_name = frame.f_code.co_name
        if not filename.startswith(root_dir):
            # only log the functions in the sgl_diffusion root_dir
            return
        # Log every function call or return
        try:
            last_frame = frame.f_back
            if last_frame is not None:
                last_filename = last_frame.f_code.co_filename
# ...
        except NameError:
            # modules are deleted during shutdown
            pass
    return partial(_trace_calls, log_path, root_dir)
```
**EN:** This function drives `_trace_calls` with inputs such as `log_path`, `root_dir`, `frame`, `event`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_trace_calls`，主要处理 `log_path`, `root_dir`, `frame`, `event` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 431-451: Function `enable_trace_function_call` / 函数 `enable_trace_function_call`
```python
def enable_trace_function_call(log_file_path: str, root_dir: str | None = None):
    """
    Enable tracing of every function call in code under `root_dir`.
    This is useful for debugging hangs or crashes.
    `log_file_path` is the path to the log file.
    `root_dir` is the root directory of the code to trace. If None, it is the
    sgl_diffusion root directory.

    Note that this call is thread-level, any threads calling this function
    will have the trace enabled. Other threads will not be affected.
    """
    logger.warning(
        "SGLANG_DIFFUSION_TRACE_FUNCTION is enabled. It will record every"
        " function executed by Python. This will slow down the code. It "
# ...
    if root_dir is None:
        # by default, this is the sgl_diffusion root directory
        root_dir = os.path.dirname(os.path.dirname(__file__))
    sys.settrace(partial(_trace_calls, log_file_path, root_dir))
```
**EN:** This function drives `enable_trace_function_call` with inputs such as `log_file_path`, `root_dir`. Enable tracing of every function call in code under `root_dir`.
**CN:** 这个函数负责 `enable_trace_function_call`，主要处理 `log_file_path`, `root_dir` 等输入。 文档字符串说明：Enable tracing of every function call in code under `root_dir`.

### Lines 454-470: Function `set_uvicorn_logging_configs` / 函数 `set_uvicorn_logging_configs`
```python
def set_uvicorn_logging_configs(server_args=None):
    from uvicorn.config import LOGGING_CONFIG

    LOGGING_CONFIG["formatters"]["default"][
        "fmt"
    ] = "[%(asctime)s] %(levelprefix)s %(message)s"
    LOGGING_CONFIG["formatters"]["default"]["datefmt"] = "%Y-%m-%d %H:%M:%S"
    LOGGING_CONFIG["formatters"]["access"][
        "fmt"
    ] = '[%(asctime)s] %(levelprefix)s %(client_addr)s - "%(request_line)s" %(status_code)s'
    LOGGING_CONFIG["formatters"]["access"]["datefmt"] = "%Y-%m-%d %H:%M:%S"

    # Install access log path filter into LOGGING_CONFIG so it survives
    # uvicorn's internal dictConfig() call during startup.
    prefixes = getattr(server_args, "uvicorn_access_log_exclude_prefixes", None)
    if prefixes:
        _install_access_log_filter(LOGGING_CONFIG, prefixes)
```
**EN:** This function drives `set_uvicorn_logging_configs` with inputs such as `server_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `set_uvicorn_logging_configs`，主要处理 `server_args` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 473-495: Function `_install_access_log_filter` / 函数 `_install_access_log_filter`
```python
def _install_access_log_filter(config: dict, prefixes: list[str]):
    """Register a path-based access log filter into uvicorn's LOGGING_CONFIG dict.

    Only attaches to the ``access`` handler (not the ``uvicorn.access`` logger)
    to avoid filtering the same record twice.
    """
    # Sanitize: drop empty strings (would match all paths) and deduplicate.
    prefixes = [str(p) for p in prefixes if p]
    prefixes = list(dict.fromkeys(prefixes))
    if not prefixes:
        return

    name = "sglang_diffusion_path_filter"
    config.setdefault("filters", {})[name] = {
# ...
    if handler_cfg is not None:
        fl = handler_cfg.setdefault("filters", [])
        if name not in fl:
            fl.append(name)
```
**EN:** This function drives `_install_access_log_filter` with inputs such as `config`, `prefixes`. Register a path-based access log filter into uvicorn's LOGGING_CONFIG dict.
**CN:** 这个函数负责 `_install_access_log_filter`，主要处理 `config`, `prefixes` 等输入。 文档字符串说明：Register a path-based access log filter into uvicorn's LOGGING_CONFIG dict.

### Lines 498-517: Class `_UvicornAccessLogFilter` / 类 `_UvicornAccessLogFilter`
```python
class _UvicornAccessLogFilter(logging.Filter):
    """Suppress uvicorn access logs whose path starts with an excluded prefix.

    uvicorn's ``AccessFormatter`` injects ``request_line`` during ``format()``,
    which runs *after* filters.  We therefore extract the path from
    ``record.args`` which uvicorn populates as::

        (client_addr, method, full_path, http_version, status_code)
    """

    def __init__(self, prefixes: list[str] | None = None):
        super().__init__()
        self.prefixes = tuple(str(p) for p in (prefixes or ()) if p)

# ...
        if isinstance(args, tuple) and len(args) >= 3:
            path = str(args[2]).split("?", 1)[0]
            return not path.startswith(self.prefixes)
        return True
```
**EN:** This class models `_UvicornAccessLogFilter` as a specialization of `logging.Filter`. Suppress uvicorn access logs whose path starts with an excluded prefix. Important methods include `__init__`, `filter`.
**CN:** 该类实现 `_UvicornAccessLogFilter`，并继承/扩展 `logging.Filter`。 文档字符串指出：Suppress uvicorn access logs whose path starts with an excluded prefix. 其中较重要的方法包括 `__init__`, `filter`。

### Lines 520-533: Function `configure_logger` / 函数 `configure_logger`
```python
def configure_logger(server_args, prefix: str = ""):
    log_format = f"[%(asctime)s{prefix}] %(message)s"
    datefmt = "%m-%d %H:%M:%S"

    formatter = ColoredFormatter(log_format, datefmt=datefmt)
    handler = logging.StreamHandler(sys.stdout)
    handler.setFormatter(formatter)

    root = logging.getLogger()
    root.handlers.clear()
    root.addHandler(handler)
    root.setLevel(getattr(logging, server_args.log_level.upper()))

    set_uvicorn_logging_configs(server_args)
```
**EN:** This function drives `configure_logger` with inputs such as `server_args`, `prefix`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `configure_logger`，主要处理 `server_args`, `prefix` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 536-539: Function `get_log_level` / 函数 `get_log_level`
```python
@lru_cache(maxsize=1)
def get_log_level() -> int:
    root = logging.getLogger()
    return root.level
```
**EN:** This function drives `get_log_level`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_log_level`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 542-550: Function `suppress_loggers` / 函数 `suppress_loggers`
```python
def suppress_loggers(loggers_to_suppress: list[str], level: int = logging.WARNING):
    original_levels = {}

    for logger_name in loggers_to_suppress:
        logger = logging.getLogger(logger_name)
        original_levels[logger_name] = logger.level
        logger.setLevel(level)

    return original_levels
```
**EN:** This function drives `suppress_loggers` with inputs such as `loggers_to_suppress`, `level`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `suppress_loggers`，主要处理 `loggers_to_suppress`, `level` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 553-569: Function `globally_suppress_loggers` / 函数 `globally_suppress_loggers`
```python
def globally_suppress_loggers():
    # globally suppress some obsessive loggers
    target_names = [
        "imageio",
        "imageio_ffmpeg",
        "PIL",
        "PIL_Image",
        "python_multipart.multipart",
        "filelock",
        "urllib3",
        "httpx",
        "httpcore",
        "flash_attn.cute.cache_utils",
    ]

    for name in target_names:
        logging.getLogger(name).setLevel(logging.ERROR)
```
**EN:** This function drives `globally_suppress_loggers`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `globally_suppress_loggers`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 573-598: Function `suppress_stdout` / 函数 `suppress_stdout`
```python
@contextlib.contextmanager
def suppress_stdout():
    """
    Suppress stdout from C libraries at the file descriptor level.

    Only suppresses stdout, not stderr, to preserve error messages.
    Example:
        with suppress_stdout():
            # C library calls that would normally print to stdout
            torch.distributed.new_group(ranks, backend="gloo")
    """
    # Don't suppress if logging level is DEBUG

    stdout_fd = sys.stdout.fileno()
# ...
        sys.stdout.flush()
        os.dup2(stdout_dup, stdout_fd)
        os.close(stdout_dup)
        os.close(devnull_fd)
```
**EN:** This function drives `suppress_stdout`. Suppress stdout from C libraries at the file descriptor level.
**CN:** 这个函数负责 `suppress_stdout`。 文档字符串说明：Suppress stdout from C libraries at the file descriptor level.

### Lines 601-605: Class `GenerationTimer` / 类 `GenerationTimer`
```python
class GenerationTimer:
    def __init__(self):
        self.start_time = 0.0
        self.end_time = 0.0
        self.duration = 0.0
```
**EN:** This class models `GenerationTimer`. Important methods include `__init__`.
**CN:** 该类实现 `GenerationTimer`。 其中较重要的方法包括 `__init__`。

### Lines 608-646: Function `log_generation_timer` / 函数 `log_generation_timer`
```python
@contextmanager
def log_generation_timer(
    logger: logging.Logger,
    prompt: str,
    request_idx: int | None = None,
    total_requests: int | None = None,
):
    if request_idx is not None and total_requests is not None:
        logger.info(
            "Processing prompt %d/%d: %s",
            request_idx,
            total_requests,
            _sanitize_for_logging(prompt, key_hint="prompt"),
        )
# ...
                f"Failed to generate output for prompt: {e}",
                exc_info=True,
            )
        raise
```
**EN:** This function drives `log_generation_timer` with inputs such as `logger`, `prompt`, `request_idx`, `total_requests`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `log_generation_timer`，主要处理 `logger`, `prompt`, `request_idx`, `total_requests` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 649-656: Function `log_batch_completion` / 函数 `log_batch_completion`
```python
def log_batch_completion(
    logger: logging.Logger, num_outputs: int, total_time: float
) -> None:
    logger.info(
        f"Completed batch processing. Generated %d outputs in {GREEN}%.2f{RESET} seconds",
        num_outputs,
        total_time,
    )
```
**EN:** This function drives `log_batch_completion` with inputs such as `logger`, `num_outputs`, `total_time`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `log_batch_completion`，主要处理 `logger`, `num_outputs`, `total_time` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行
- Command-line interface / 命令行接口
- Image generation flow / 图像生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.envs`
- **External / 外部**: `uvicorn.config`, `torch`
- **Stdlib / 标准库**: `argparse`, `contextlib`, `dataclasses`, `datetime`, `inspect`, `logging`, `os`, `sys`
