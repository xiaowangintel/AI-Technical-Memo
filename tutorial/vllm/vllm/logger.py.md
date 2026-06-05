# logger.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `vllm/logger.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Builds vLLM's logging configuration, adds once-only helpers, and exposes tracing utilities for debugging. / 构建 vLLM 的日志配置，提供“只打印一次”的辅助方法，并暴露用于调试的函数调用追踪工具。

## Line-by-Line Analysis / 逐行分析
### Formatting, color policy, and default config (lines 22-72)
```python
_FORMAT = (
    f"{envs.VLLM_LOGGING_PREFIX}%(levelname)s %(asctime)s "
    "[%(fileinfo)s:%(lineno)d] %(message)s"
)
_DATE_FORMAT = "%m-%d %H:%M:%S"


def _use_color() -> bool:
    if envs.NO_COLOR or envs.VLLM_LOGGING_COLOR == "0":
        return False
    if envs.VLLM_LOGGING_COLOR == "1":
        return True
    if envs.VLLM_LOGGING_STREAM == "ext://sys.stdout":  # stdout
        return hasattr(sys.stdout, "isatty") and sys.stdout.isatty()
    elif envs.VLLM_LOGGING_STREAM == "ext://sys.stderr":  # stderr
        return hasattr(sys.stderr, "isatty") and sys.stderr.isatty()
    return False


DEFAULT_LOGGING_CONFIG: dict[str, dict[str, Any] | Any] = {
    "formatters": {
        "vllm": {
            "class": "vllm.logging_utils.NewLineFormatter",
            "datefmt": _DATE_FORMAT,
            "format": _FORMAT,
        },
        "vllm_color": {
            "class": "vllm.logging_utils.ColoredFormatter",
            "datefmt": _DATE_FORMAT,
            "format": _FORMAT,
        },
    },
    "handlers": {
        "vllm": {
            "class": "logging.StreamHandler",
            # Choose formatter based on color setting.
            "formatter": "vllm_color" if _use_color() else "vllm",
            "level": envs.VLLM_LOGGING_LEVEL,
            "stream": envs.VLLM_LOGGING_STREAM,
        },
    },
    "loggers": {
        "vllm": {
            "handlers": ["vllm"],
            "level": envs.VLLM_LOGGING_LEVEL,
            "propagate": False,
        },
    },
    "version": 1,
    "disable_existing_loggers": False,
}
```
**EN:** `_FORMAT` includes a configurable prefix plus custom `fileinfo`, and `_use_color()` decides whether ANSI coloring is appropriate based on `NO_COLOR`, explicit overrides, and whether stdout/stderr is a TTY. `DEFAULT_LOGGING_CONFIG` then wires those decisions into a `dictConfig`-compatible structure centered on a single `vllm` stream handler.
**CN:** `_FORMAT` 包含可配置前缀以及自定义的 `fileinfo` 字段，`_use_color()` 则根据 `NO_COLOR`、显式覆盖值以及 stdout/stderr 是否为 TTY 来决定是否启用 ANSI 颜色。随后 `DEFAULT_LOGGING_CONFIG` 会把这些决策组织成 `dictConfig` 兼容结构，其核心是一个名为 `vllm` 的流处理器。

### Once-only logging and scope gating (lines 75-145)
```python
@lru_cache
def _print_debug_once(logger: Logger, msg: str, *args: Hashable) -> None:
    # Set the stacklevel to 3 to print the original caller's line info
    logger.debug(msg, *args, stacklevel=3)


@lru_cache
def _print_info_once(logger: Logger, msg: str, *args: Hashable) -> None:
    # Set the stacklevel to 3 to print the original caller's line info
    logger.info(msg, *args, stacklevel=3)


@lru_cache
def _print_warning_once(logger: Logger, msg: str, *args: Hashable) -> None:
    # Set the stacklevel to 3 to print the original caller's line info
    logger.warning(msg, *args, stacklevel=3)


LogScope = Literal["process", "global", "local"]


def _should_log_with_scope(scope: LogScope) -> bool:
    """Decide whether to log based on scope"""
    if scope == "global":
        from vllm.distributed.parallel_state import is_global_first_rank

        return is_global_first_rank()
    if scope == "local":
        from vllm.distributed.parallel_state import is_local_first_rank

        return is_local_first_rank()
    return True


class _VllmLogger(Logger):
    """
    Note:
        This class is just to provide type information.
        We actually patch the methods directly on the [`logging.Logger`][]
        instance to avoid conflicting with other libraries such as
        `intel_extension_for_pytorch.utils._logger`.
    """

    def debug_once(self, msg: str, *args: Hashable, scope: LogScope = "local") -> None:
        """
        As [`debug`][logging.Logger.debug], but subsequent calls with
        the same message are silently dropped.
        """
        if not _should_log_with_scope(scope):
            return
        _print_debug_once(self, msg, *args)

    def info_once(self, msg: str, *args: Hashable, scope: LogScope = "local") -> None:
        """
        As [`info`][logging.Logger.info], but subsequent calls with
        the same message are silently dropped.
        """
        if not _should_log_with_scope(scope):
            return
        _print_info_once(self, msg, *args)

    def warning_once(
        self, msg: str, *args: Hashable, scope: LogScope = "local"
    ) -> None:
        """
        As [`warning`][logging.Logger.warning], but subsequent calls with
        the same message are silently dropped.
        """
        if not _should_log_with_scope(scope):
            return
        _print_warning_once(self, msg, *args)
```
**EN:** The `*_once` helpers are implemented with `@lru_cache`, so identical messages are emitted only once per process. `_should_log_with_scope()` adds distributed-awareness: `global` means only rank 0 across the whole job logs, while `local` means only the first rank on each node logs. This is how vLLM reduces log spam in distributed deployments.
**CN:** `*_once` 系列辅助函数通过 `@lru_cache` 实现，因此同样的消息在同一进程中只会输出一次。`_should_log_with_scope()` 又加入了分布式语义：`global` 表示整个作业只有全局 rank 0 打印，`local` 表示每台机器只有本地第一个 rank 打印。这正是 vLLM 在分布式部署中减少日志刷屏的方式。

### Root logger configuration and logger patching (lines 156-214)
```python
def _configure_vllm_root_logger() -> None:
    logging_config: dict[str, dict[str, Any] | Any] = {}

    if not envs.VLLM_CONFIGURE_LOGGING and envs.VLLM_LOGGING_CONFIG_PATH:
        raise RuntimeError(
            "VLLM_CONFIGURE_LOGGING evaluated to false, but "
            "VLLM_LOGGING_CONFIG_PATH was given. VLLM_LOGGING_CONFIG_PATH "
            "implies VLLM_CONFIGURE_LOGGING. Please enable "
            "VLLM_CONFIGURE_LOGGING or unset VLLM_LOGGING_CONFIG_PATH."
        )

    if envs.VLLM_CONFIGURE_LOGGING:
        logging_config = DEFAULT_LOGGING_CONFIG

        vllm_handler = logging_config["handlers"]["vllm"]
        # Refresh these values in case env vars have changed.
        vllm_handler["level"] = envs.VLLM_LOGGING_LEVEL
        vllm_handler["stream"] = envs.VLLM_LOGGING_STREAM
        vllm_handler["formatter"] = "vllm_color" if _use_color() else "vllm"

        vllm_loggers = logging_config["loggers"]["vllm"]
        vllm_loggers["level"] = envs.VLLM_LOGGING_LEVEL

    if envs.VLLM_LOGGING_CONFIG_PATH:
        if not path.exists(envs.VLLM_LOGGING_CONFIG_PATH):
            raise RuntimeError(
                "Could not load logging config. File does not exist: %s",
                envs.VLLM_LOGGING_CONFIG_PATH,
            )
        with open(envs.VLLM_LOGGING_CONFIG_PATH, encoding="utf-8") as file:
            custom_config = json.loads(file.read())

        if not isinstance(custom_config, dict):
            raise ValueError(
                "Invalid logging config. Expected dict, got %s.",
                type(custom_config).__name__,
            )
        logging_config = custom_config

    for formatter in logging_config.get("formatters", {}).values():
        # This provides backwards compatibility after #10134.
        if formatter.get("class") == "vllm.logging.NewLineFormatter":
            formatter["class"] = "vllm.logging_utils.NewLineFormatter"

    if logging_config:
        dictConfig(logging_config)


def init_logger(name: str) -> _VllmLogger:
    """The main purpose of this function is to ensure that loggers are
    retrieved in such a way that we can be sure the root vllm logger has
    already been configured."""

    logger = logging.getLogger(name)

    for method_name, method in _METHODS_TO_PATCH.items():
        setattr(logger, method_name, MethodType(method, logger))

    return cast(_VllmLogger, logger)
```
**EN:** `_configure_vllm_root_logger()` enforces a consistent policy between `VLLM_CONFIGURE_LOGGING` and `VLLM_LOGGING_CONFIG_PATH`, refreshes env-driven handler fields at import time, and allows a custom JSON config file. `init_logger()` then monkey-patches `debug_once`, `info_once`, and `warning_once` directly onto the stdlib logger instance instead of subclass-instantiating a custom logger class, which avoids conflicts with other frameworks that also replace `logging.Logger`.
**CN:** `_configure_vllm_root_logger()` 会校验 `VLLM_CONFIGURE_LOGGING` 与 `VLLM_LOGGING_CONFIG_PATH` 之间的一致性，在导入时刷新由环境变量驱动的 handler 字段，并支持加载自定义 JSON 日志配置。`init_logger()` 随后把 `debug_once`、`info_once`、`warning_once` 直接 monkey-patch 到标准库 logger 实例上，而不是强制实例化自定义子类，从而避免与其他同样会替换 `logging.Logger` 的框架冲突。

### Temporary suppression and import-time side effects (lines 217-246)
```python
@contextmanager
def suppress_logging(level: int = logging.INFO) -> Generator[None, Any, None]:
    current_level = logging.root.manager.disable
    logging.disable(level)
    yield
    logging.disable(current_level)


def current_formatter_type(logger: Logger) -> Literal["color", "newline", None]:
    lgr: Logger | None = logger
    while lgr is not None:
        if lgr.handlers and len(lgr.handlers) == 1 and lgr.handlers[0].name == "vllm":
            formatter = lgr.handlers[0].formatter
            if isinstance(formatter, ColoredFormatter):
                return "color"
            if isinstance(formatter, NewLineFormatter):
                return "newline"
        lgr = lgr.parent
    return None


# The root logger is initialized when the module is imported.
# This is thread-safe as the module is only imported once,
# guaranteed by the Python GIL.
_configure_vllm_root_logger()

# Transformers uses httpx to access the Hugging Face Hub. httpx is quite verbose,
# so we set its logging level to WARNING when vLLM's logging level is INFO.
if envs.VLLM_LOGGING_LEVEL == "INFO":
    logging.getLogger("httpx").setLevel(logging.WARNING)
```
**EN:** `suppress_logging()` temporarily raises the global logging disable level and restores it afterward. `current_formatter_type()` walks logger parents to inspect whether the effective handler is colored or newline-aware. At module import time, vLLM immediately configures the root logger and also downgrades `httpx` noise to WARNING when vLLM itself runs at INFO.
**CN:** `suppress_logging()` 会临时提高全局日志禁用级别，并在退出后恢复。`current_formatter_type()` 会沿着 logger 父链向上查找，判断当前生效的 formatter 是彩色还是换行感知型。模块导入时，vLLM 会立刻配置根日志器；如果 vLLM 自身处于 INFO 级别，还会把 `httpx` 的噪声日志下调到 WARNING。

### Function-call tracing (lines 251-315)
```python
def _trace_calls(log_path, root_dir, frame, event, arg=None):
    if event in ["call", "return"]:
        # Extract the filename, line number, function name, and the code object
        filename = frame.f_code.co_filename
        lineno = frame.f_lineno
        func_name = frame.f_code.co_name
        if not filename.startswith(root_dir):
            # only log the functions in the vllm root_dir
            return
        # Log every function call or return
        try:
            last_frame = frame.f_back
            if last_frame is not None:
                last_filename = last_frame.f_code.co_filename
                last_lineno = last_frame.f_lineno
                last_func_name = last_frame.f_code.co_name
            else:
                # initial frame
                last_filename = ""
                last_lineno = 0
                last_func_name = ""
            with open(log_path, "a") as f:
                ts = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S.%f")
                if event == "call":
                    f.write(
                        f"{ts} Call to"
                        f" {func_name} in {filename}:{lineno}"
                        f" from {last_func_name} in {last_filename}:"
                        f"{last_lineno}\n"
                    )
                else:
                    f.write(
                        f"{ts} Return from"
                        f" {func_name} in {filename}:{lineno}"
                        f" to {last_func_name} in {last_filename}:"
                        f"{last_lineno}\n"
                    )
        except NameError:
            # modules are deleted during shutdown
            pass
    return partial(_trace_calls, log_path, root_dir)


def enable_trace_function_call(log_file_path: str, root_dir: str | None = None):
    """
    Enable tracing of every function call in code under `root_dir`.
    This is useful for debugging hangs or crashes.
    `log_file_path` is the path to the log file.
    `root_dir` is the root directory of the code to trace. If None, it is the
    vllm root directory.

    Note that this call is thread-level, any threads calling this function
    will have the trace enabled. Other threads will not be affected.
    """
    logger.warning(
        "VLLM_TRACE_FUNCTION is enabled. It will record every"
        " function executed by Python. This will slow down the code. It "
        "is suggested to be used for debugging hang or crashes only."
    )
    logger.info("Trace frame log is saved to %s", log_file_path)
    if root_dir is None:
        # by default, this is the vllm root directory
        root_dir = os.path.dirname(os.path.dirname(__file__))
    sys.settrace(partial(_trace_calls, log_file_path, root_dir))
```
**EN:** `_trace_calls()` is a low-level `sys.settrace` hook that appends per-call and per-return events to a log file, but only for files under a selected root directory. `enable_trace_function_call()` wraps that with warnings about overhead and defaults the traced root to the vLLM package directory, making it useful for diagnosing hangs or startup deadlocks.
**CN:** `_trace_calls()` 是一个底层 `sys.settrace` 钩子，会把函数调用和返回事件追加到日志文件里，但仅限于指定根目录下的文件。`enable_trace_function_call()` 在其外层增加了性能开销警告，并把默认追踪根目录设为 vLLM 包目录，因此很适合排查卡死或启动死锁。

## Key Concepts / 关键概念
- EN: vLLM logging is env-driven end to end: formatting, color, stream, and even whether logging should be configured at all.
- CN: vLLM 的日志体系是端到端由环境变量驱动的：格式、颜色、输出流，甚至是否由 vLLM 负责配置日志，都可以通过环境变量控制。
- EN: The once-only helpers solve a distributed-systems problem: repeated warnings from many ranks quickly become unreadable.
- CN: “只打印一次”辅助函数解决的是分布式系统中的可读性问题：大量 rank 重复输出同一警告会让日志几乎无法阅读。
- EN: Tracing uses Python's interpreter hook, so it is powerful but intentionally opt-in due to high overhead.
- CN: 函数追踪使用 Python 解释器级钩子，因此能力很强，但也因为开销较高而被设计成显式 opt-in。

## Dependencies / 依赖关系
- EN: Consumes environment settings from `vllm.envs` and formatter classes from `vllm.logging_utils`.
- CN: 从 `vllm.envs` 读取环境设置，并依赖 `vllm.logging_utils` 中的 formatter 类。
- EN: For distributed scope filtering it imports rank helpers from `vllm.distributed.parallel_state` on demand.
- CN: 在分布式作用域过滤时，会按需导入 `vllm.distributed.parallel_state` 中的 rank 辅助函数。
- EN: Uses stdlib `logging`, `json`, `sys.settrace`, and file IO for configuration and tracing output.
- CN: 使用标准库 `logging`、`json`、`sys.settrace` 以及文件 IO 完成配置和追踪输出。
