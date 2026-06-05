# main.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/utils/mbr/mbr/main.py`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: This file contains the main function that's called by the CLI of the library.
  - **CN**: 提供 MLIR 开发者使用的 MBR 工具脚本与 Python 辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

````python
   1 | """This file contains the main function that's called by the CLI of the library.
   2 | """
   3 | 
   4 | import os
   5 | import sys
   6 | import time
   7 | 
   8 | import numpy as np
   9 | 
  10 | from discovery import discover_benchmark_modules, get_benchmark_functions
  11 | from stats import has_enough_measurements
  12 | 
````
- **L1 EN**: Participates in a module, class, or function docstring: `"""This file contains the main function that's called by the CLI of the library.`.
  **L1 CN**: 参与模块、类或函数的 docstring：`"""This file contains the main function that's called by the CLI of the library.`。
- **L2 EN**: Participates in a module, class, or function docstring: `"""`.
  **L2 CN**: 参与模块、类或函数的 docstring：`"""`。
- **L3 EN**: Blank line separating nearby declarations or logic blocks.
  **L3 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L4 EN**: Imports one or more Python modules: `import os`.
  **L4 CN**: 导入一个或多个 Python 模块：`import os`。
- **L5 EN**: Imports one or more Python modules: `import sys`.
  **L5 CN**: 导入一个或多个 Python 模块：`import sys`。
- **L6 EN**: Imports one or more Python modules: `import time`.
  **L6 CN**: 导入一个或多个 Python 模块：`import time`。
- **L7 EN**: Blank line separating nearby declarations or logic blocks.
  **L7 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L8 EN**: Imports one or more Python modules: `import numpy as np`.
  **L8 CN**: 导入一个或多个 Python 模块：`import numpy as np`。
- **L9 EN**: Blank line separating nearby declarations or logic blocks.
  **L9 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L10 EN**: Imports selected names from module `discovery`.
  **L10 CN**: 从模块 `discovery` 中导入指定名称。
- **L11 EN**: Imports selected names from module `stats`.
  **L11 CN**: 从模块 `stats` 中导入指定名称。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 13-24 / 第 13-24 行

````python
  13 | 
  14 | def main(top_level_path, stop_on_error):
  15 |     """Top level function called when the CLI is invoked."""
  16 |     if "::" in top_level_path:
  17 |         if top_level_path.count("::") > 1:
  18 |             raise AssertionError(f"Invalid path {top_level_path}")
  19 |         top_level_path, benchmark_function_name = top_level_path.split("::")
  20 |     else:
  21 |         benchmark_function_name = None
  22 | 
  23 |     if not os.path.exists(top_level_path):
  24 |         raise AssertionError(f"The top-level path {top_level_path} doesn't exist")
````
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Defines function `main`.
  **L14 CN**: 定义函数 `main`。
- **L15 EN**: Participates in a module, class, or function docstring: `"""Top level function called when the CLI is invoked."""`.
  **L15 CN**: 参与模块、类或函数的 docstring：`"""Top level function called when the CLI is invoked."""`。
- **L16 EN**: Starts a Python control-flow or context-management clause: `if "::" in top_level_path:`.
  **L16 CN**: 开始一条 Python 控制流或上下文管理子句：`if "::" in top_level_path:`。
- **L17 EN**: Starts a Python control-flow or context-management clause: `if top_level_path.count("::") > 1:`.
  **L17 CN**: 开始一条 Python 控制流或上下文管理子句：`if top_level_path.count("::") > 1:`。
- **L18 EN**: Executes a Python control statement: `raise AssertionError(f"Invalid path {top_level_path}")`.
  **L18 CN**: 执行一条 Python 控制语句：`raise AssertionError(f"Invalid path {top_level_path}")`。
- **L19 EN**: Assigns or updates `top_level_path`.
  **L19 CN**: 对 `top_level_path` 进行赋值或更新。
- **L20 EN**: Starts the fallback branch for the preceding conditional.
  **L20 CN**: 开始前一个条件结构的兜底分支。
- **L21 EN**: Assigns or updates `benchmark_function_name`.
  **L21 CN**: 对 `benchmark_function_name` 进行赋值或更新。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L23 EN**: Starts a Python control-flow or context-management clause: `if not os.path.exists(top_level_path):`.
  **L23 CN**: 开始一条 Python 控制流或上下文管理子句：`if not os.path.exists(top_level_path):`。
- **L24 EN**: Executes a Python control statement: `raise AssertionError(f"The top-level path {top_level_path} doesn't exist")`.
  **L24 CN**: 执行一条 Python 控制语句：`raise AssertionError(f"The top-level path {top_level_path} doesn't exist")`。

### Lines 25-36 / 第 25-36 行

````python
  25 | 
  26 |     modules = [module for module in discover_benchmark_modules(top_level_path)]
  27 |     benchmark_dicts = []
  28 |     for module in modules:
  29 |         benchmark_functions = [
  30 |             function
  31 |             for function in get_benchmark_functions(module, benchmark_function_name)
  32 |         ]
  33 |         for benchmark_function in benchmark_functions:
  34 |             try:
  35 |                 compiler, runner = benchmark_function()
  36 |             except (TypeError, ValueError) as e:
````
- **L25 EN**: Blank line separating nearby declarations or logic blocks.
  **L25 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L26 EN**: Assigns or updates `modules`.
  **L26 CN**: 对 `modules` 进行赋值或更新。
- **L27 EN**: Assigns or updates `benchmark_dicts`.
  **L27 CN**: 对 `benchmark_dicts` 进行赋值或更新。
- **L28 EN**: Starts a Python control-flow or context-management clause: `for module in modules:`.
  **L28 CN**: 开始一条 Python 控制流或上下文管理子句：`for module in modules:`。
- **L29 EN**: Assigns or updates `benchmark_functions`.
  **L29 CN**: 对 `benchmark_functions` 进行赋值或更新。
- **L30 EN**: Executes Python statement `function`.
  **L30 CN**: 执行 Python 语句 `function`。
- **L31 EN**: Starts a Python control-flow or context-management clause: `for function in get_benchmark_functions(module, benchmark_function_name)`.
  **L31 CN**: 开始一条 Python 控制流或上下文管理子句：`for function in get_benchmark_functions(module, benchmark_function_name)`。
- **L32 EN**: Executes Python statement `]`.
  **L32 CN**: 执行 Python 语句 `]`。
- **L33 EN**: Starts a Python control-flow or context-management clause: `for benchmark_function in benchmark_functions:`.
  **L33 CN**: 开始一条 Python 控制流或上下文管理子句：`for benchmark_function in benchmark_functions:`。
- **L34 EN**: Starts a Python control-flow or context-management clause: `try:`.
  **L34 CN**: 开始一条 Python 控制流或上下文管理子句：`try:`。
- **L35 EN**: Assigns or updates `compiler`.
  **L35 CN**: 对 `compiler` 进行赋值或更新。
- **L36 EN**: Starts a Python control-flow or context-management clause: `except (TypeError, ValueError) as e:`.
  **L36 CN**: 开始一条 Python 控制流或上下文管理子句：`except (TypeError, ValueError) as e:`。

### Lines 37-48 / 第 37-48 行

````python
  37 |                 error_message = (
  38 |                     f"Obtaining compiler and runner failed because of {e}."
  39 |                     f" Benchmark function '{benchmark_function.__name__}'"
  40 |                     f" must return a two-tuple value (compiler, runner)."
  41 |                 )
  42 |                 if stop_on_error is False:
  43 |                     print(error_message, file=sys.stderr)
  44 |                     continue
  45 |                 else:
  46 |                     raise AssertionError(error_message) from e
  47 |             measurements_ns = np.array([])
  48 |             if compiler:
````
- **L37 EN**: Assigns or updates `error_message`.
  **L37 CN**: 对 `error_message` 进行赋值或更新。
- **L38 EN**: Executes Python statement `f"Obtaining compiler and runner failed because of {e}."`.
  **L38 CN**: 执行 Python 语句 `f"Obtaining compiler and runner failed because of {e}."`。
- **L39 EN**: Executes Python statement `f" Benchmark function '{benchmark_function.__name__}'"`.
  **L39 CN**: 执行 Python 语句 `f" Benchmark function '{benchmark_function.__name__}'"`。
- **L40 EN**: Executes Python statement `f" must return a two-tuple value (compiler, runner)."`.
  **L40 CN**: 执行 Python 语句 `f" must return a two-tuple value (compiler, runner)."`。
- **L41 EN**: Executes Python statement `)`.
  **L41 CN**: 执行 Python 语句 `)`。
- **L42 EN**: Starts a Python control-flow or context-management clause: `if stop_on_error is False:`.
  **L42 CN**: 开始一条 Python 控制流或上下文管理子句：`if stop_on_error is False:`。
- **L43 EN**: Executes Python statement `print(error_message, file=sys.stderr)`.
  **L43 CN**: 执行 Python 语句 `print(error_message, file=sys.stderr)`。
- **L44 EN**: Executes Python statement `continue`.
  **L44 CN**: 执行 Python 语句 `continue`。
- **L45 EN**: Starts the fallback branch for the preceding conditional.
  **L45 CN**: 开始前一个条件结构的兜底分支。
- **L46 EN**: Executes a Python control statement: `raise AssertionError(error_message) from e`.
  **L46 CN**: 执行一条 Python 控制语句：`raise AssertionError(error_message) from e`。
- **L47 EN**: Assigns or updates `measurements_ns`.
  **L47 CN**: 对 `measurements_ns` 进行赋值或更新。
- **L48 EN**: Starts a Python control-flow or context-management clause: `if compiler:`.
  **L48 CN**: 开始一条 Python 控制流或上下文管理子句：`if compiler:`。

### Lines 49-60 / 第 49-60 行

````python
  49 |                 start_compile_time_s = time.time()
  50 |                 try:
  51 |                     compiled_callable = compiler()
  52 |                 except Exception as e:
  53 |                     error_message = (
  54 |                         f"Compilation of {benchmark_function.__name__} failed"
  55 |                         f" because of {e}"
  56 |                     )
  57 |                     if stop_on_error is False:
  58 |                         print(error_message, file=sys.stderr)
  59 |                         continue
  60 |                     else:
````
- **L49 EN**: Assigns or updates `start_compile_time_s`.
  **L49 CN**: 对 `start_compile_time_s` 进行赋值或更新。
- **L50 EN**: Starts a Python control-flow or context-management clause: `try:`.
  **L50 CN**: 开始一条 Python 控制流或上下文管理子句：`try:`。
- **L51 EN**: Assigns or updates `compiled_callable`.
  **L51 CN**: 对 `compiled_callable` 进行赋值或更新。
- **L52 EN**: Starts a Python control-flow or context-management clause: `except Exception as e:`.
  **L52 CN**: 开始一条 Python 控制流或上下文管理子句：`except Exception as e:`。
- **L53 EN**: Assigns or updates `error_message`.
  **L53 CN**: 对 `error_message` 进行赋值或更新。
- **L54 EN**: Executes Python statement `f"Compilation of {benchmark_function.__name__} failed"`.
  **L54 CN**: 执行 Python 语句 `f"Compilation of {benchmark_function.__name__} failed"`。
- **L55 EN**: Executes Python statement `f" because of {e}"`.
  **L55 CN**: 执行 Python 语句 `f" because of {e}"`。
- **L56 EN**: Executes Python statement `)`.
  **L56 CN**: 执行 Python 语句 `)`。
- **L57 EN**: Starts a Python control-flow or context-management clause: `if stop_on_error is False:`.
  **L57 CN**: 开始一条 Python 控制流或上下文管理子句：`if stop_on_error is False:`。
- **L58 EN**: Executes Python statement `print(error_message, file=sys.stderr)`.
  **L58 CN**: 执行 Python 语句 `print(error_message, file=sys.stderr)`。
- **L59 EN**: Executes Python statement `continue`.
  **L59 CN**: 执行 Python 语句 `continue`。
- **L60 EN**: Starts the fallback branch for the preceding conditional.
  **L60 CN**: 开始前一个条件结构的兜底分支。

### Lines 61-72 / 第 61-72 行

````python
  61 |                         raise AssertionError(error_message) from e
  62 |                 total_compile_time_s = time.time() - start_compile_time_s
  63 |                 runner_args = (compiled_callable,)
  64 |             else:
  65 |                 total_compile_time_s = 0
  66 |                 runner_args = ()
  67 |             while not has_enough_measurements(measurements_ns):
  68 |                 try:
  69 |                     measurement_ns = runner(*runner_args)
  70 |                 except Exception as e:
  71 |                     error_message = (
  72 |                         f"Runner of {benchmark_function.__name__} failed"
````
- **L61 EN**: Executes a Python control statement: `raise AssertionError(error_message) from e`.
  **L61 CN**: 执行一条 Python 控制语句：`raise AssertionError(error_message) from e`。
- **L62 EN**: Assigns or updates `total_compile_time_s`.
  **L62 CN**: 对 `total_compile_time_s` 进行赋值或更新。
- **L63 EN**: Assigns or updates `runner_args`.
  **L63 CN**: 对 `runner_args` 进行赋值或更新。
- **L64 EN**: Starts the fallback branch for the preceding conditional.
  **L64 CN**: 开始前一个条件结构的兜底分支。
- **L65 EN**: Assigns or updates `total_compile_time_s`.
  **L65 CN**: 对 `total_compile_time_s` 进行赋值或更新。
- **L66 EN**: Assigns or updates `runner_args`.
  **L66 CN**: 对 `runner_args` 进行赋值或更新。
- **L67 EN**: Starts a Python control-flow or context-management clause: `while not has_enough_measurements(measurements_ns):`.
  **L67 CN**: 开始一条 Python 控制流或上下文管理子句：`while not has_enough_measurements(measurements_ns):`。
- **L68 EN**: Starts a Python control-flow or context-management clause: `try:`.
  **L68 CN**: 开始一条 Python 控制流或上下文管理子句：`try:`。
- **L69 EN**: Assigns or updates `measurement_ns`.
  **L69 CN**: 对 `measurement_ns` 进行赋值或更新。
- **L70 EN**: Starts a Python control-flow or context-management clause: `except Exception as e:`.
  **L70 CN**: 开始一条 Python 控制流或上下文管理子句：`except Exception as e:`。
- **L71 EN**: Assigns or updates `error_message`.
  **L71 CN**: 对 `error_message` 进行赋值或更新。
- **L72 EN**: Executes Python statement `f"Runner of {benchmark_function.__name__} failed"`.
  **L72 CN**: 执行 Python 语句 `f"Runner of {benchmark_function.__name__} failed"`。

### Lines 73-84 / 第 73-84 行

````python
  73 |                         f" because of {e}"
  74 |                     )
  75 |                     if stop_on_error is False:
  76 |                         print(error_message, file=sys.stderr)
  77 |                         # Recover from runner error by breaking out of this loop
  78 |                         # and continuing forward.
  79 |                         break
  80 |                     else:
  81 |                         raise AssertionError(error_message) from e
  82 |                 if not isinstance(measurement_ns, int):
  83 |                     error_message = (
  84 |                         f"Expected benchmark runner function"
````
- **L73 EN**: Executes Python statement `f" because of {e}"`.
  **L73 CN**: 执行 Python 语句 `f" because of {e}"`。
- **L74 EN**: Executes Python statement `)`.
  **L74 CN**: 执行 Python 语句 `)`。
- **L75 EN**: Starts a Python control-flow or context-management clause: `if stop_on_error is False:`.
  **L75 CN**: 开始一条 Python 控制流或上下文管理子句：`if stop_on_error is False:`。
- **L76 EN**: Executes Python statement `print(error_message, file=sys.stderr)`.
  **L76 CN**: 执行 Python 语句 `print(error_message, file=sys.stderr)`。
- **L77 EN**: Comment documents nearby Python logic: `Recover from runner error by breaking out of this loop`.
  **L77 CN**: 注释说明附近的 Python 逻辑：`Recover from runner error by breaking out of this loop`。
- **L78 EN**: Comment documents nearby Python logic: `and continuing forward.`.
  **L78 CN**: 注释说明附近的 Python 逻辑：`and continuing forward.`。
- **L79 EN**: Executes Python statement `break`.
  **L79 CN**: 执行 Python 语句 `break`。
- **L80 EN**: Starts the fallback branch for the preceding conditional.
  **L80 CN**: 开始前一个条件结构的兜底分支。
- **L81 EN**: Executes a Python control statement: `raise AssertionError(error_message) from e`.
  **L81 CN**: 执行一条 Python 控制语句：`raise AssertionError(error_message) from e`。
- **L82 EN**: Starts a Python control-flow or context-management clause: `if not isinstance(measurement_ns, int):`.
  **L82 CN**: 开始一条 Python 控制流或上下文管理子句：`if not isinstance(measurement_ns, int):`。
- **L83 EN**: Assigns or updates `error_message`.
  **L83 CN**: 对 `error_message` 进行赋值或更新。
- **L84 EN**: Executes Python statement `f"Expected benchmark runner function"`.
  **L84 CN**: 执行 Python 语句 `f"Expected benchmark runner function"`。

### Lines 85-96 / 第 85-96 行

````python
  85 |                         f" to return an int, got {measurement_ns}"
  86 |                     )
  87 |                     if stop_on_error is False:
  88 |                         print(error_message, file=sys.stderr)
  89 |                         continue
  90 |                     else:
  91 |                         raise AssertionError(error_message)
  92 |                 measurements_ns = np.append(measurements_ns, measurement_ns)
  93 | 
  94 |             if len(measurements_ns) > 0:
  95 |                 measurements_s = [t * 1e-9 for t in measurements_ns]
  96 |                 benchmark_identifier = ":".join(
````
- **L85 EN**: Executes Python statement `f" to return an int, got {measurement_ns}"`.
  **L85 CN**: 执行 Python 语句 `f" to return an int, got {measurement_ns}"`。
- **L86 EN**: Executes Python statement `)`.
  **L86 CN**: 执行 Python 语句 `)`。
- **L87 EN**: Starts a Python control-flow or context-management clause: `if stop_on_error is False:`.
  **L87 CN**: 开始一条 Python 控制流或上下文管理子句：`if stop_on_error is False:`。
- **L88 EN**: Executes Python statement `print(error_message, file=sys.stderr)`.
  **L88 CN**: 执行 Python 语句 `print(error_message, file=sys.stderr)`。
- **L89 EN**: Executes Python statement `continue`.
  **L89 CN**: 执行 Python 语句 `continue`。
- **L90 EN**: Starts the fallback branch for the preceding conditional.
  **L90 CN**: 开始前一个条件结构的兜底分支。
- **L91 EN**: Executes a Python control statement: `raise AssertionError(error_message)`.
  **L91 CN**: 执行一条 Python 控制语句：`raise AssertionError(error_message)`。
- **L92 EN**: Assigns or updates `measurements_ns`.
  **L92 CN**: 对 `measurements_ns` 进行赋值或更新。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L94 EN**: Starts a Python control-flow or context-management clause: `if len(measurements_ns) > 0:`.
  **L94 CN**: 开始一条 Python 控制流或上下文管理子句：`if len(measurements_ns) > 0:`。
- **L95 EN**: Assigns or updates `measurements_s`.
  **L95 CN**: 对 `measurements_s` 进行赋值或更新。
- **L96 EN**: Assigns or updates `benchmark_identifier`.
  **L96 CN**: 对 `benchmark_identifier` 进行赋值或更新。

### Lines 97-107 / 第 97-107 行

````python
  97 |                     [module.__name__, benchmark_function.__name__]
  98 |                 )
  99 |                 benchmark_dicts.append(
 100 |                     {
 101 |                         "name": benchmark_identifier,
 102 |                         "compile_time": total_compile_time_s,
 103 |                         "execution_time": list(measurements_s),
 104 |                     }
 105 |                 )
 106 | 
 107 |     return benchmark_dicts
````
- **L97 EN**: Executes Python statement `[module.__name__, benchmark_function.__name__]`.
  **L97 CN**: 执行 Python 语句 `[module.__name__, benchmark_function.__name__]`。
- **L98 EN**: Executes Python statement `)`.
  **L98 CN**: 执行 Python 语句 `)`。
- **L99 EN**: Executes Python statement `benchmark_dicts.append(`.
  **L99 CN**: 执行 Python 语句 `benchmark_dicts.append(`。
- **L100 EN**: Executes Python statement `{`.
  **L100 CN**: 执行 Python 语句 `{`。
- **L101 EN**: Executes Python statement `"name": benchmark_identifier,`.
  **L101 CN**: 执行 Python 语句 `"name": benchmark_identifier,`。
- **L102 EN**: Executes Python statement `"compile_time": total_compile_time_s,`.
  **L102 CN**: 执行 Python 语句 `"compile_time": total_compile_time_s,`。
- **L103 EN**: Executes Python statement `"execution_time": list(measurements_s),`.
  **L103 CN**: 执行 Python 语句 `"execution_time": list(measurements_s),`。
- **L104 EN**: Executes Python statement `}`.
  **L104 CN**: 执行 Python 语句 `}`。
- **L105 EN**: Executes Python statement `)`.
  **L105 CN**: 执行 Python 语句 `)`。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L107 EN**: Returns from the current Python function: `return benchmark_dicts`.
  **L107 CN**: 从当前 Python 函数返回：`return benchmark_dicts`。

## Key Concepts / 关键概念

- **Developer utilities / 开发者工具**:
  - **EN**: Provides scripts or helpers that support debugging, testing, or developer workflows around MLIR.
  - **CN**: 提供支持 MLIR 调试、测试或开发者工作流的脚本与辅助逻辑。
- **Execution support / 执行支持**:
  - **EN**: Connects MLIR IR to JIT execution or runtime invocation paths.
  - **CN**: 将 MLIR IR 连接到 JIT 执行或运行时调用路径。
- **Python automation / Python 自动化**:
  - **EN**: Uses Python to orchestrate MLIR construction, registration, execution, or developer utilities.
  - **CN**: 使用 Python 编排 MLIR 的构造、注册、执行或开发者辅助流程。
- **Library composition / 库组合**:
  - **EN**: Builds behavior by composing generated artifacts, MLIR libraries, or Python modules.
  - **CN**: 通过组合生成工件、MLIR 库或 Python 模块来构建行为。
- **Executable startup flow / 可执行启动流程**:
  - **EN**: Defines the process entry point and drives the surrounding MLIR workflow from there.
  - **CN**: 定义进程入口，并从该入口驱动周边 MLIR 工作流。

## Dependencies / 依赖关系

- **Imported modules / 导入模块**: `os`, `sys`, `time`, `numpy`, `discovery`, `stats`
