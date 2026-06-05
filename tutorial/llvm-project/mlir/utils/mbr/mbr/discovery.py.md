# discovery.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/utils/mbr/mbr/discovery.py`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: This file contains functions for discovering benchmark functions. It works in a similar way to python's unittest library. import configparser import importlib import os import pathlib.
  - **CN**: 提供 MLIR 开发者使用的 MBR 工具脚本与 Python 辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10 / 第 1-10 行

````python
   1 | """This file contains functions for discovering benchmark functions. It works
   2 | in a similar way to python's unittest library.
   3 | """
   4 | import configparser
   5 | import importlib
   6 | import os
   7 | import pathlib
   8 | import re
   9 | import sys
  10 | import types
````
- **L1 EN**: Participates in a module, class, or function docstring: `"""This file contains functions for discovering benchmark functions. It works`.
  **L1 CN**: 参与模块、类或函数的 docstring：`"""This file contains functions for discovering benchmark functions. It works`。
- **L2 EN**: Executes Python statement `in a similar way to python's unittest library.`.
  **L2 CN**: 执行 Python 语句 `in a similar way to python's unittest library.`。
- **L3 EN**: Participates in a module, class, or function docstring: `"""`.
  **L3 CN**: 参与模块、类或函数的 docstring：`"""`。
- **L4 EN**: Imports one or more Python modules: `import configparser`.
  **L4 CN**: 导入一个或多个 Python 模块：`import configparser`。
- **L5 EN**: Imports one or more Python modules: `import importlib`.
  **L5 CN**: 导入一个或多个 Python 模块：`import importlib`。
- **L6 EN**: Imports one or more Python modules: `import os`.
  **L6 CN**: 导入一个或多个 Python 模块：`import os`。
- **L7 EN**: Imports one or more Python modules: `import pathlib`.
  **L7 CN**: 导入一个或多个 Python 模块：`import pathlib`。
- **L8 EN**: Imports one or more Python modules: `import re`.
  **L8 CN**: 导入一个或多个 Python 模块：`import re`。
- **L9 EN**: Imports one or more Python modules: `import sys`.
  **L9 CN**: 导入一个或多个 Python 模块：`import sys`。
- **L10 EN**: Imports one or more Python modules: `import types`.
  **L10 CN**: 导入一个或多个 Python 模块：`import types`。

### Lines 11-20 / 第 11-20 行

````python
  11 | 
  12 | 
  13 | def discover_benchmark_modules(top_level_path):
  14 |     """Starting from the `top_level_path`, discover python files which contains
  15 |     benchmark functions. It looks for files with a specific prefix, which
  16 |     defaults to "benchmark_"
  17 |     """
  18 |     config = configparser.ConfigParser()
  19 |     config.read(os.path.join(os.path.dirname(os.path.realpath(__file__)), "config.ini"))
  20 |     if "discovery" in config.sections():
````
- **L11 EN**: Blank line separating nearby declarations or logic blocks.
  **L11 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Defines function `discover_benchmark_modules`.
  **L13 CN**: 定义函数 `discover_benchmark_modules`。
- **L14 EN**: Participates in a module, class, or function docstring: `"""Starting from the 'top_level_path', discover python files which contains`.
  **L14 CN**: 参与模块、类或函数的 docstring：`"""Starting from the 'top_level_path', discover python files which contains`。
- **L15 EN**: Executes Python statement `benchmark functions. It looks for files with a specific prefix, which`.
  **L15 CN**: 执行 Python 语句 `benchmark functions. It looks for files with a specific prefix, which`。
- **L16 EN**: Executes Python statement `defaults to "benchmark_"`.
  **L16 CN**: 执行 Python 语句 `defaults to "benchmark_"`。
- **L17 EN**: Participates in a module, class, or function docstring: `"""`.
  **L17 CN**: 参与模块、类或函数的 docstring：`"""`。
- **L18 EN**: Assigns or updates `config`.
  **L18 CN**: 对 `config` 进行赋值或更新。
- **L19 EN**: Executes Python statement `config.read(os.path.join(os.path.dirname(os.path.realpath(__file__)), "config.ini"))`.
  **L19 CN**: 执行 Python 语句 `config.read(os.path.join(os.path.dirname(os.path.realpath(__file__)), "config.ini"))`。
- **L20 EN**: Starts a Python control-flow or context-management clause: `if "discovery" in config.sections():`.
  **L20 CN**: 开始一条 Python 控制流或上下文管理子句：`if "discovery" in config.sections():`。

### Lines 21-30 / 第 21-30 行

````python
  21 |         filename_prefix = config["discovery"]["filename_prefix"]
  22 |     else:
  23 |         filename_prefix = "benchmark_"
  24 |     if re.search(rf"{filename_prefix}.*.py$", top_level_path):
  25 |         # A specific python file so just include that.
  26 |         benchmark_files = [top_level_path]
  27 |     else:
  28 |         # A directory so recursively search for all python files.
  29 |         benchmark_files = pathlib.Path(top_level_path).rglob(f"{filename_prefix}*.py")
  30 |     for benchmark_filename in benchmark_files:
````
- **L21 EN**: Assigns or updates `filename_prefix`.
  **L21 CN**: 对 `filename_prefix` 进行赋值或更新。
- **L22 EN**: Starts the fallback branch for the preceding conditional.
  **L22 CN**: 开始前一个条件结构的兜底分支。
- **L23 EN**: Assigns or updates `filename_prefix`.
  **L23 CN**: 对 `filename_prefix` 进行赋值或更新。
- **L24 EN**: Starts a Python control-flow or context-management clause: `if re.search(rf"{filename_prefix}.*.py$", top_level_path):`.
  **L24 CN**: 开始一条 Python 控制流或上下文管理子句：`if re.search(rf"{filename_prefix}.*.py$", top_level_path):`。
- **L25 EN**: Comment documents nearby Python logic: `A specific python file so just include that.`.
  **L25 CN**: 注释说明附近的 Python 逻辑：`A specific python file so just include that.`。
- **L26 EN**: Assigns or updates `benchmark_files`.
  **L26 CN**: 对 `benchmark_files` 进行赋值或更新。
- **L27 EN**: Starts the fallback branch for the preceding conditional.
  **L27 CN**: 开始前一个条件结构的兜底分支。
- **L28 EN**: Comment documents nearby Python logic: `A directory so recursively search for all python files.`.
  **L28 CN**: 注释说明附近的 Python 逻辑：`A directory so recursively search for all python files.`。
- **L29 EN**: Assigns or updates `benchmark_files`.
  **L29 CN**: 对 `benchmark_files` 进行赋值或更新。
- **L30 EN**: Starts a Python control-flow or context-management clause: `for benchmark_filename in benchmark_files:`.
  **L30 CN**: 开始一条 Python 控制流或上下文管理子句：`for benchmark_filename in benchmark_files:`。

### Lines 31-40 / 第 31-40 行

````python
  31 |         benchmark_abs_dir = os.path.abspath(os.path.dirname(benchmark_filename))
  32 |         sys.path.append(benchmark_abs_dir)
  33 |         module_file_name = os.path.basename(benchmark_filename)
  34 |         module_name = module_file_name.replace(".py", "")
  35 |         module = importlib.import_module(module_name)
  36 |         yield module
  37 |         sys.path.pop()
  38 | 
  39 | 
  40 | def get_benchmark_functions(module, benchmark_function_name=None):
````
- **L31 EN**: Assigns or updates `benchmark_abs_dir`.
  **L31 CN**: 对 `benchmark_abs_dir` 进行赋值或更新。
- **L32 EN**: Executes Python statement `sys.path.append(benchmark_abs_dir)`.
  **L32 CN**: 执行 Python 语句 `sys.path.append(benchmark_abs_dir)`。
- **L33 EN**: Assigns or updates `module_file_name`.
  **L33 CN**: 对 `module_file_name` 进行赋值或更新。
- **L34 EN**: Assigns or updates `module_name`.
  **L34 CN**: 对 `module_name` 进行赋值或更新。
- **L35 EN**: Assigns or updates `module`.
  **L35 CN**: 对 `module` 进行赋值或更新。
- **L36 EN**: Executes a Python control statement: `yield module`.
  **L36 CN**: 执行一条 Python 控制语句：`yield module`。
- **L37 EN**: Executes Python statement `sys.path.pop()`.
  **L37 CN**: 执行 Python 语句 `sys.path.pop()`。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L40 EN**: Defines function `get_benchmark_functions`.
  **L40 CN**: 定义函数 `get_benchmark_functions`。

### Lines 41-50 / 第 41-50 行

````python
  41 |     """Discover benchmark functions in python file. It looks for functions with
  42 |     a specific prefix, which defaults to "benchmark_".
  43 |     """
  44 |     config = configparser.ConfigParser()
  45 |     config.read(os.path.join(os.path.dirname(os.path.realpath(__file__)), "config.ini"))
  46 |     if "discovery" in config.sections():
  47 |         function_prefix = config["discovery"].get("function_prefix")
  48 |     else:
  49 |         function_prefix = "benchmark_"
  50 | 
````
- **L41 EN**: Participates in a module, class, or function docstring: `"""Discover benchmark functions in python file. It looks for functions with`.
  **L41 CN**: 参与模块、类或函数的 docstring：`"""Discover benchmark functions in python file. It looks for functions with`。
- **L42 EN**: Executes Python statement `a specific prefix, which defaults to "benchmark_".`.
  **L42 CN**: 执行 Python 语句 `a specific prefix, which defaults to "benchmark_".`。
- **L43 EN**: Participates in a module, class, or function docstring: `"""`.
  **L43 CN**: 参与模块、类或函数的 docstring：`"""`。
- **L44 EN**: Assigns or updates `config`.
  **L44 CN**: 对 `config` 进行赋值或更新。
- **L45 EN**: Executes Python statement `config.read(os.path.join(os.path.dirname(os.path.realpath(__file__)), "config.ini"))`.
  **L45 CN**: 执行 Python 语句 `config.read(os.path.join(os.path.dirname(os.path.realpath(__file__)), "config.ini"))`。
- **L46 EN**: Starts a Python control-flow or context-management clause: `if "discovery" in config.sections():`.
  **L46 CN**: 开始一条 Python 控制流或上下文管理子句：`if "discovery" in config.sections():`。
- **L47 EN**: Assigns or updates `function_prefix`.
  **L47 CN**: 对 `function_prefix` 进行赋值或更新。
- **L48 EN**: Starts the fallback branch for the preceding conditional.
  **L48 CN**: 开始前一个条件结构的兜底分支。
- **L49 EN**: Assigns or updates `function_prefix`.
  **L49 CN**: 对 `function_prefix` 进行赋值或更新。
- **L50 EN**: Blank line separating nearby declarations or logic blocks.
  **L50 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 51-60 / 第 51-60 行

````python
  51 |     module_functions = []
  52 |     for attribute_name in dir(module):
  53 |         attribute = getattr(module, attribute_name)
  54 |         if isinstance(attribute, types.FunctionType) and attribute_name.startswith(
  55 |             function_prefix
  56 |         ):
  57 |             module_functions.append(attribute)
  58 | 
  59 |     if benchmark_function_name:
  60 |         # If benchmark_function_name is present, just yield the corresponding
````
- **L51 EN**: Assigns or updates `module_functions`.
  **L51 CN**: 对 `module_functions` 进行赋值或更新。
- **L52 EN**: Starts a Python control-flow or context-management clause: `for attribute_name in dir(module):`.
  **L52 CN**: 开始一条 Python 控制流或上下文管理子句：`for attribute_name in dir(module):`。
- **L53 EN**: Assigns or updates `attribute`.
  **L53 CN**: 对 `attribute` 进行赋值或更新。
- **L54 EN**: Starts a Python control-flow or context-management clause: `if isinstance(attribute, types.FunctionType) and attribute_name.startswith(`.
  **L54 CN**: 开始一条 Python 控制流或上下文管理子句：`if isinstance(attribute, types.FunctionType) and attribute_name.startswith(`。
- **L55 EN**: Executes Python statement `function_prefix`.
  **L55 CN**: 执行 Python 语句 `function_prefix`。
- **L56 EN**: Executes Python statement `):`.
  **L56 CN**: 执行 Python 语句 `):`。
- **L57 EN**: Executes Python statement `module_functions.append(attribute)`.
  **L57 CN**: 执行 Python 语句 `module_functions.append(attribute)`。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L59 EN**: Starts a Python control-flow or context-management clause: `if benchmark_function_name:`.
  **L59 CN**: 开始一条 Python 控制流或上下文管理子句：`if benchmark_function_name:`。
- **L60 EN**: Comment documents nearby Python logic: `If benchmark_function_name is present, just yield the corresponding`.
  **L60 CN**: 注释说明附近的 Python 逻辑：`If benchmark_function_name is present, just yield the corresponding`。

### Lines 61-68 / 第 61-68 行

````python
  61 |         # function and nothing else.
  62 |         for function in module_functions:
  63 |             if function.__name__ == benchmark_function_name:
  64 |                 yield function
  65 |     else:
  66 |         # If benchmark_function_name is not present, yield all functions.
  67 |         for function in module_functions:
  68 |             yield function
````
- **L61 EN**: Comment documents nearby Python logic: `function and nothing else.`.
  **L61 CN**: 注释说明附近的 Python 逻辑：`function and nothing else.`。
- **L62 EN**: Starts a Python control-flow or context-management clause: `for function in module_functions:`.
  **L62 CN**: 开始一条 Python 控制流或上下文管理子句：`for function in module_functions:`。
- **L63 EN**: Starts a Python control-flow or context-management clause: `if function.__name__ == benchmark_function_name:`.
  **L63 CN**: 开始一条 Python 控制流或上下文管理子句：`if function.__name__ == benchmark_function_name:`。
- **L64 EN**: Executes a Python control statement: `yield function`.
  **L64 CN**: 执行一条 Python 控制语句：`yield function`。
- **L65 EN**: Starts the fallback branch for the preceding conditional.
  **L65 CN**: 开始前一个条件结构的兜底分支。
- **L66 EN**: Comment documents nearby Python logic: `If benchmark_function_name is not present, yield all functions.`.
  **L66 CN**: 注释说明附近的 Python 逻辑：`If benchmark_function_name is not present, yield all functions.`。
- **L67 EN**: Starts a Python control-flow or context-management clause: `for function in module_functions:`.
  **L67 CN**: 开始一条 Python 控制流或上下文管理子句：`for function in module_functions:`。
- **L68 EN**: Executes a Python control statement: `yield function`.
  **L68 CN**: 执行一条 Python 控制语句：`yield function`。

## Key Concepts / 关键概念

- **Developer utilities / 开发者工具**:
  - **EN**: Provides scripts or helpers that support debugging, testing, or developer workflows around MLIR.
  - **CN**: 提供支持 MLIR 调试、测试或开发者工作流的脚本与辅助逻辑。
- **Python automation / Python 自动化**:
  - **EN**: Uses Python to orchestrate MLIR construction, registration, execution, or developer utilities.
  - **CN**: 使用 Python 编排 MLIR 的构造、注册、执行或开发者辅助流程。
- **Library composition / 库组合**:
  - **EN**: Builds behavior by composing generated artifacts, MLIR libraries, or Python modules.
  - **CN**: 通过组合生成工件、MLIR 库或 Python 模块来构建行为。

## Dependencies / 依赖关系

- **Imported modules / 导入模块**: `configparser`, `importlib`, `os`, `pathlib`, `re`, `sys`, `types`
