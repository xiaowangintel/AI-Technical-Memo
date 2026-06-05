# formatter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/logging_utils/formatter.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements formatter support for the `logging_utils` portion of vLLM. / 为 vLLM 的 `logging_utils` 子目录实现与 formatter 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-7)
```python
import logging

from pathlib import Path

from vllm import envs
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Class `NewLineFormatter` (lines 10-75)
```python
class NewLineFormatter(logging.Formatter):
    """Adds logging prefix to newlines to align multi-line messages."""

    def __init__(self, fmt, datefmt=None, style="%"):
        super().__init__(fmt, datefmt, style)

        self.use_relpath = envs.VLLM_LOGGING_LEVEL == "DEBUG"
        if self.use_relpath:
            self.root_dir = Path(__file__).resolve().parent.parent.parent

    def format(self, record):
        def shrink_path(relpath: Path) -> str:
            """
            Shortens a file path for logging display:
            - Removes leading 'vllm' folder if present.
            - If path starts with 'v1',
            keeps the first two and last two levels,
            collapsing the middle as '...'.
            - Otherwise, keeps the first and last two levels,
            collapsing the middle as '...'.
            - If the path is short, returns it as-is.
            - Examples:
            vllm/model_executor/layers/quantization/utils/fp8_utils.py ->
    # ... omitted for brevity ...
            msg = msg.replace("\n", "\r\n" + parts[0])
        return msg
```
**EN:** Class `NewLineFormatter` is a structured building block in this module. It inherits from `logging.Formatter`. Key methods include `__init__`, `format`, which define initialization, validation, transformation, or access patterns. The class docstring says: Adds logging prefix to newlines to align multi-line messages.
**CN:** 类 `NewLineFormatter` 是该模块中的结构化构件，继承自 `logging.Formatter`。 关键方法包括 `__init__`, `format`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Adds logging prefix to newlines to align multi-line messages.

### Method `NewLineFormatter.__init__` (lines 13-18)
```python
    def __init__(self, fmt, datefmt=None, style="%"):
        super().__init__(fmt, datefmt, style)

        self.use_relpath = envs.VLLM_LOGGING_LEVEL == "DEBUG"
        if self.use_relpath:
            self.root_dir = Path(__file__).resolve().parent.parent.parent
```
**EN:** Method `NewLineFormatter.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super`, `Path(__file__).resolve`, `Path` show the concrete execution path.
**CN:** Method `NewLineFormatter.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super`, `Path(__file__).resolve`, `Path` 这样的关键调用展示了该代码块的具体执行路径。

### Method `NewLineFormatter.format` (lines 20-75)
```python
    def format(self, record):
        def shrink_path(relpath: Path) -> str:
            """
            Shortens a file path for logging display:
            - Removes leading 'vllm' folder if present.
            - If path starts with 'v1',
            keeps the first two and last two levels,
            collapsing the middle as '...'.
            - Otherwise, keeps the first and last two levels,
            collapsing the middle as '...'.
            - If the path is short, returns it as-is.
            - Examples:
            vllm/model_executor/layers/quantization/utils/fp8_utils.py ->
            model_executor/.../quantization/utils/fp8_utils.py
            vllm/model_executor/layers/quantization/awq.py ->
            model_executor/layers/quantization/awq.py

            Args:
                relpath (Path): The relative path to be shortened.
    # ... omitted for brevity ...
            msg = msg.replace("\n", "\r\n" + parts[0])
        return msg
```
**EN:** Method `NewLineFormatter.format` provides a reusable helper around the module's main workflow. Key calls such as `list`, `len`, `'/'.join`, `getattr`, `Path(abs_path).resolve().relative_to` show the concrete execution path.
**CN:** Method `NewLineFormatter.format` 为模块主流程提供可复用的辅助逻辑。 像 `list`, `len`, `'/'.join`, `getattr`, `Path(abs_path).resolve().relative_to` 这样的关键调用展示了该代码块的具体执行路径。

### Class `ColoredFormatter` (lines 78-125)
```python
class ColoredFormatter(NewLineFormatter):
    """Adds ANSI color codes to log levels for terminal output.

    This formatter adds colors by injecting them into the format string for
    static elements (timestamp, filename, line number) and modifying the
    levelname attribute for dynamic color selection.
    """

    # ANSI color codes
    COLORS = {
        "DEBUG": "\033[37m",  # White
        "INFO": "\033[32m",  # Green
        "WARNING": "\033[33m",  # Yellow
        "ERROR": "\033[31m",  # Red
        "CRITICAL": "\033[35m",  # Magenta
    }
    GREY = "\033[90m"  # Grey for timestamp and file info
    RESET = "\033[0m"

    def __init__(self, fmt, datefmt=None, style="%"):
        # Inject grey color codes into format string for timestamp and file info
        if fmt:
            # Wrap %(asctime)s with grey
    # ... omitted for brevity ...

        return msg
```
**EN:** Class `ColoredFormatter` is a structured building block in this module. It inherits from `NewLineFormatter`. Key methods include `__init__`, `format`, which define initialization, validation, transformation, or access patterns. The class docstring says: Adds ANSI color codes to log levels for terminal output.
**CN:** 类 `ColoredFormatter` 是该模块中的结构化构件，继承自 `NewLineFormatter`。 关键方法包括 `__init__`, `format`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Adds ANSI color codes to log levels for terminal output.

### Method `ColoredFormatter.__init__` (lines 97-109)
```python
    def __init__(self, fmt, datefmt=None, style="%"):
        # Inject grey color codes into format string for timestamp and file info
        if fmt:
            # Wrap %(asctime)s with grey
            fmt = fmt.replace("%(asctime)s", f"{self.GREY}%(asctime)s{self.RESET}")
            # Wrap [%(fileinfo)s:%(lineno)d] with grey
            fmt = fmt.replace(
                "[%(fileinfo)s:%(lineno)d]",
                f"{self.GREY}[%(fileinfo)s:%(lineno)d]{self.RESET}",
            )

        # Call parent __init__ with potentially modified format string
        super().__init__(fmt, datefmt, style)
```
**EN:** Method `ColoredFormatter.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `fmt.replace`, `super().__init__`, `super` show the concrete execution path.
**CN:** Method `ColoredFormatter.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `fmt.replace`, `super().__init__`, `super` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ColoredFormatter.format` (lines 111-125)
```python
    def format(self, record):
        # Store original levelname to restore later (in case record is reused)
        orig_levelname = record.levelname

        # Only modify levelname - it needs dynamic color based on severity
        if (color_code := self.COLORS.get(record.levelname)) is not None:
            record.levelname = f"{color_code}{record.levelname}{self.RESET}"

        # Call parent format which will handle everything else
        msg = super().format(record)

        # Restore original levelname
        record.levelname = orig_levelname

        return msg
```
**EN:** Method `ColoredFormatter.format` provides a reusable helper around the module's main workflow. Key calls such as `self.COLORS.get`, `super().format`, `super` show the concrete execution path.
**CN:** Method `ColoredFormatter.format` 为模块主流程提供可复用的辅助逻辑。 像 `self.COLORS.get`, `super().format`, `super` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Module organization / 模块组织**
  - **EN:** The file mainly groups reusable helpers and definitions behind a coherent interface.
  - **CN:** 该文件主要把可复用的辅助逻辑和定义组织到一致接口之后。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import logging`, `from pathlib import Path`
- **vLLM internal / vLLM 内部依赖**: `from vllm import envs`
