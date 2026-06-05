# update_test_body.py — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/utils/update_test_body.py` | `llvm/utils/update_test_body.py` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Generate test body using split-file and a custom script. The script will prepare extra files with `split-file`, invoke `gen`, and then rewrite the part after `gen` with its stdout. https://llvm.org/docs/TestingGuide.h... | 实现与 `update_test_body` 相关的 Python 工具逻辑、命令处理或辅助流程。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

````python
#!/usr/bin/env python3
"""Generate test body using split-file and a custom script.

The script will prepare extra files with `split-file`, invoke `gen`, and then
rewrite the part after `gen` with its stdout.

https://llvm.org/docs/TestingGuide.html#elaborated-tests

````
- **L1 EN**: Shebang selects the interpreter used to execute this script.
  **L1 CN**: Shebang 指定了执行该脚本时使用的解释器。
- **L2 EN**: Executes Python statement `"""Generate test body using split-file and a custom script.`.
  **L2 CN**: 执行 Python 语句 `"""Generate test body using split-file and a custom script.`。
- **L3 EN**: Blank line separates nearby declarations or logic blocks.
  **L3 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4 EN**: Executes Python statement `The script will prepare extra files with \`split-file\`, invoke \`gen\`, and then`.
  **L4 CN**: 执行 Python 语句 `The script will prepare extra files with \`split-file\`, invoke \`gen\`, and then`。
- **L5 EN**: Executes Python statement `rewrite the part after \`gen\` with its stdout.`.
  **L5 CN**: 执行 Python 语句 `rewrite the part after \`gen\` with its stdout.`。
- **L6 EN**: Blank line separates nearby declarations or logic blocks.
  **L6 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L7 EN**: Executes Python statement `https://llvm.org/docs/TestingGuide.html#elaborated-tests`.
  **L7 CN**: 执行 Python 语句 `https://llvm.org/docs/TestingGuide.html#elaborated-tests`。
- **L8 EN**: Blank line separates nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 9-19

````python
Example:
PATH=/path/to/clang_build/bin:$PATH llvm/utils/update_test_body.py path/to/test.s
"""
import argparse
import contextlib
import os
import re
import subprocess
import sys
import tempfile

````
- **L9 EN**: Executes Python statement `Example:`.
  **L9 CN**: 执行 Python 语句 `Example:`。
- **L10 EN**: Assigns or updates `PATH`.
  **L10 CN**: 对 `PATH` 进行赋值或更新。
- **L11 EN**: Executes Python statement `"""`.
  **L11 CN**: 执行 Python 语句 `"""`。
- **L12 EN**: Imports Python module(s) `argparse` for supporting functionality.
  **L12 CN**: 导入 Python 模块 `argparse` 以提供辅助功能。
- **L13 EN**: Imports Python module(s) `contextlib` for supporting functionality.
  **L13 CN**: 导入 Python 模块 `contextlib` 以提供辅助功能。
- **L14 EN**: Imports Python module(s) `os` for supporting functionality.
  **L14 CN**: 导入 Python 模块 `os` 以提供辅助功能。
- **L15 EN**: Imports Python module(s) `re` for supporting functionality.
  **L15 CN**: 导入 Python 模块 `re` 以提供辅助功能。
- **L16 EN**: Imports Python module(s) `subprocess` for supporting functionality.
  **L16 CN**: 导入 Python 模块 `subprocess` 以提供辅助功能。
- **L17 EN**: Imports Python module(s) `sys` for supporting functionality.
  **L17 CN**: 导入 Python 模块 `sys` 以提供辅助功能。
- **L18 EN**: Imports Python module(s) `tempfile` for supporting functionality.
  **L18 CN**: 导入 Python 模块 `tempfile` 以提供辅助功能。
- **L19 EN**: Blank line separates nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 20-29

````python

@contextlib.contextmanager
def cd(directory):
    cwd = os.getcwd()
    os.chdir(directory)
    try:
        yield
    finally:
        os.chdir(cwd)

````
- **L20 EN**: Blank line separates nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L21 EN**: Applies decorator `@contextlib.contextmanager` to customize the following declaration.
  **L21 CN**: 应用装饰器 `@contextlib.contextmanager`，以定制后续声明的行为。
- **L22 EN**: Declares function `cd`.
  **L22 CN**: 声明函数 `cd`。
- **L23 EN**: Assigns or updates `cwd`.
  **L23 CN**: 对 `cwd` 进行赋值或更新。
- **L24 EN**: Executes Python statement `os.chdir(directory)`.
  **L24 CN**: 执行 Python 语句 `os.chdir(directory)`。
- **L25 EN**: Controls Python flow with `try` logic.
  **L25 CN**: 使用 `try` 逻辑控制 Python 执行流程。
- **L26 EN**: Executes Python statement `yield`.
  **L26 CN**: 执行 Python 语句 `yield`。
- **L27 EN**: Controls Python flow with `finally` logic.
  **L27 CN**: 使用 `finally` 逻辑控制 Python 执行流程。
- **L28 EN**: Executes Python statement `os.chdir(cwd)`.
  **L28 CN**: 执行 Python 语句 `os.chdir(cwd)`。
- **L29 EN**: Blank line separates nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 30-45

````python

def process(args, path):
    prolog = []
    seen_gen = False
    with open(path) as f:
        for line in f.readlines():
            line = line.rstrip()
            prolog.append(line)
            if (seen_gen and re.match(r"(.|//)---", line)) or line.startswith(".endif"):
                break
            if re.match(r"(.|//)--- gen", line):
                seen_gen = True
        else:
            print(
                "'gen' should be followed by another part (---) or .endif",
                file=sys.stderr,
````
- **L30 EN**: Blank line separates nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L31 EN**: Declares function `process`.
  **L31 CN**: 声明函数 `process`。
- **L32 EN**: Assigns or updates `prolog`.
  **L32 CN**: 对 `prolog` 进行赋值或更新。
- **L33 EN**: Assigns or updates `seen_gen`.
  **L33 CN**: 对 `seen_gen` 进行赋值或更新。
- **L34 EN**: Controls Python flow with `with` logic.
  **L34 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L35 EN**: Controls Python flow with `for` logic.
  **L35 CN**: 使用 `for` 逻辑控制 Python 执行流程。
- **L36 EN**: Assigns or updates `line`.
  **L36 CN**: 对 `line` 进行赋值或更新。
- **L37 EN**: Executes Python statement `prolog.append(line)`.
  **L37 CN**: 执行 Python 语句 `prolog.append(line)`。
- **L38 EN**: Controls Python flow with `if` logic.
  **L38 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L39 EN**: Executes Python statement `break`.
  **L39 CN**: 执行 Python 语句 `break`。
- **L40 EN**: Controls Python flow with `if` logic.
  **L40 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L41 EN**: Assigns or updates `seen_gen`.
  **L41 CN**: 对 `seen_gen` 进行赋值或更新。
- **L42 EN**: Controls Python flow with `else` logic.
  **L42 CN**: 使用 `else` 逻辑控制 Python 执行流程。
- **L43 EN**: Executes Python statement `print(`.
  **L43 CN**: 执行 Python 语句 `print(`。
- **L44 EN**: Executes Python statement `"'gen' should be followed by another part (---) or .endif",`.
  **L44 CN**: 执行 Python 语句 `"'gen' should be followed by another part (---) or .endif",`。
- **L45 EN**: Assigns or updates `file`.
  **L45 CN**: 对 `file` 进行赋值或更新。

### Lines 46-61

````python
            )
            return 1

    if not seen_gen:
        print("'gen' does not exist", file=sys.stderr)
        return 1
    with tempfile.TemporaryDirectory(prefix="update_test_body_") as dir:
        try:
            # If the last line starts with ".endif", remove it.
            sub = subprocess.run(
                ["split-file", "-", dir],
                input="\n".join(
                    prolog[:-1] if prolog[-1].startswith(".endif") else prolog
                ).encode(),
                capture_output=True,
                check=True,
````
- **L46 EN**: Executes Python statement `)`.
  **L46 CN**: 执行 Python 语句 `)`。
- **L47 EN**: Returns a value or exits the current function.
  **L47 CN**: 返回一个值或结束当前函数。
- **L48 EN**: Blank line separates nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L49 EN**: Controls Python flow with `if` logic.
  **L49 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L50 EN**: Assigns or updates `print("'gen' does not exist", file`.
  **L50 CN**: 对 `print("'gen' does not exist", file` 进行赋值或更新。
- **L51 EN**: Returns a value or exits the current function.
  **L51 CN**: 返回一个值或结束当前函数。
- **L52 EN**: Controls Python flow with `with` logic.
  **L52 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L53 EN**: Controls Python flow with `try` logic.
  **L53 CN**: 使用 `try` 逻辑控制 Python 执行流程。
- **L54 EN**: Comment documents nearby script behavior: `If the last line starts with ".endif", remove it.`.
  **L54 CN**: 注释说明了附近脚本逻辑：`If the last line starts with ".endif", remove it.`。
- **L55 EN**: Assigns or updates `sub`.
  **L55 CN**: 对 `sub` 进行赋值或更新。
- **L56 EN**: Executes Python statement `["split-file", "-", dir],`.
  **L56 CN**: 执行 Python 语句 `["split-file", "-", dir],`。
- **L57 EN**: Assigns or updates `input`.
  **L57 CN**: 对 `input` 进行赋值或更新。
- **L58 EN**: Executes Python statement `prolog[:-1] if prolog[-1].startswith(".endif") else prolog`.
  **L58 CN**: 执行 Python 语句 `prolog[:-1] if prolog[-1].startswith(".endif") else prolog`。
- **L59 EN**: Executes Python statement `).encode(),`.
  **L59 CN**: 执行 Python 语句 `).encode(),`。
- **L60 EN**: Assigns or updates `capture_output`.
  **L60 CN**: 对 `capture_output` 进行赋值或更新。
- **L61 EN**: Assigns or updates `check`.
  **L61 CN**: 对 `check` 进行赋值或更新。

### Lines 62-71

````python
            )
        except subprocess.CalledProcessError as ex:
            sys.stderr.write(ex.stderr.decode())
            return 1
        with cd(dir):
            if args.shell:
                print(f"invoke shell in the temporary directory '{dir}'")
                subprocess.run([os.environ.get("SHELL", "sh")])
                return 0

````
- **L62 EN**: Executes Python statement `)`.
  **L62 CN**: 执行 Python 语句 `)`。
- **L63 EN**: Controls Python flow with `except` logic.
  **L63 CN**: 使用 `except` 逻辑控制 Python 执行流程。
- **L64 EN**: Executes Python statement `sys.stderr.write(ex.stderr.decode())`.
  **L64 CN**: 执行 Python 语句 `sys.stderr.write(ex.stderr.decode())`。
- **L65 EN**: Returns a value or exits the current function.
  **L65 CN**: 返回一个值或结束当前函数。
- **L66 EN**: Controls Python flow with `with` logic.
  **L66 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L67 EN**: Controls Python flow with `if` logic.
  **L67 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L68 EN**: Executes Python statement `print(f"invoke shell in the temporary directory '{dir}'")`.
  **L68 CN**: 执行 Python 语句 `print(f"invoke shell in the temporary directory '{dir}'")`。
- **L69 EN**: Executes Python statement `subprocess.run([os.environ.get("SHELL", "sh")])`.
  **L69 CN**: 执行 Python 语句 `subprocess.run([os.environ.get("SHELL", "sh")])`。
- **L70 EN**: Returns a value or exits the current function.
  **L70 CN**: 返回一个值或结束当前函数。
- **L71 EN**: Blank line separates nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 72-87

````python
            sub = subprocess.run(
                ["sh", "-eu", "gen"],
                capture_output=True,
                # Don't encode the directory information to the Clang output.
                # Remove unneeded details (.ident) as well.
                env=dict(
                    os.environ,
                    CCC_OVERRIDE_OPTIONS="#^-fno-ident",
                    PWD="/proc/self/cwd",
                ),
            )
            sys.stderr.write(sub.stderr.decode())
            if sub.returncode != 0:
                print("'gen' failed", file=sys.stderr)
                return sub.returncode
            if not sub.stdout:
````
- **L72 EN**: Assigns or updates `sub`.
  **L72 CN**: 对 `sub` 进行赋值或更新。
- **L73 EN**: Executes Python statement `["sh", "-eu", "gen"],`.
  **L73 CN**: 执行 Python 语句 `["sh", "-eu", "gen"],`。
- **L74 EN**: Assigns or updates `capture_output`.
  **L74 CN**: 对 `capture_output` 进行赋值或更新。
- **L75 EN**: Comment documents nearby script behavior: `Don't encode the directory information to the Clang output.`.
  **L75 CN**: 注释说明了附近脚本逻辑：`Don't encode the directory information to the Clang output.`。
- **L76 EN**: Comment documents nearby script behavior: `Remove unneeded details (.ident) as well.`.
  **L76 CN**: 注释说明了附近脚本逻辑：`Remove unneeded details (.ident) as well.`。
- **L77 EN**: Assigns or updates `env`.
  **L77 CN**: 对 `env` 进行赋值或更新。
- **L78 EN**: Executes Python statement `os.environ,`.
  **L78 CN**: 执行 Python 语句 `os.environ,`。
- **L79 EN**: Assigns or updates `CCC_OVERRIDE_OPTIONS`.
  **L79 CN**: 对 `CCC_OVERRIDE_OPTIONS` 进行赋值或更新。
- **L80 EN**: Assigns or updates `PWD`.
  **L80 CN**: 对 `PWD` 进行赋值或更新。
- **L81 EN**: Executes Python statement `),`.
  **L81 CN**: 执行 Python 语句 `),`。
- **L82 EN**: Executes Python statement `)`.
  **L82 CN**: 执行 Python 语句 `)`。
- **L83 EN**: Executes Python statement `sys.stderr.write(sub.stderr.decode())`.
  **L83 CN**: 执行 Python 语句 `sys.stderr.write(sub.stderr.decode())`。
- **L84 EN**: Controls Python flow with `if` logic.
  **L84 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L85 EN**: Assigns or updates `print("'gen' failed", file`.
  **L85 CN**: 对 `print("'gen' failed", file` 进行赋值或更新。
- **L86 EN**: Returns a value or exits the current function.
  **L86 CN**: 返回一个值或结束当前函数。
- **L87 EN**: Controls Python flow with `if` logic.
  **L87 CN**: 使用 `if` 逻辑控制 Python 执行流程。

### Lines 88-97

````python
                print("stdout is empty; forgot -o - ?", file=sys.stderr)
                return 1
            content = sub.stdout.decode()

    with open(path, "w") as f:
        # Print lines up to '.endif'.
        print("\n".join(prolog), file=f)
        # Then print the stdout of 'gen'.
        f.write(content)

````
- **L88 EN**: Assigns or updates `print("stdout is empty; forgot -o - ?", file`.
  **L88 CN**: 对 `print("stdout is empty; forgot -o - ?", file` 进行赋值或更新。
- **L89 EN**: Returns a value or exits the current function.
  **L89 CN**: 返回一个值或结束当前函数。
- **L90 EN**: Assigns or updates `content`.
  **L90 CN**: 对 `content` 进行赋值或更新。
- **L91 EN**: Blank line separates nearby declarations or logic blocks.
  **L91 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L92 EN**: Controls Python flow with `with` logic.
  **L92 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L93 EN**: Comment documents nearby script behavior: `Print lines up to '.endif'.`.
  **L93 CN**: 注释说明了附近脚本逻辑：`Print lines up to '.endif'.`。
- **L94 EN**: Assigns or updates `print("\n".join(prolog), file`.
  **L94 CN**: 对 `print("\n".join(prolog), file` 进行赋值或更新。
- **L95 EN**: Comment documents nearby script behavior: `Then print the stdout of 'gen'.`.
  **L95 CN**: 注释说明了附近脚本逻辑：`Then print the stdout of 'gen'.`。
- **L96 EN**: Executes Python statement `f.write(content)`.
  **L96 CN**: 执行 Python 语句 `f.write(content)`。
- **L97 EN**: Blank line separates nearby declarations or logic blocks.
  **L97 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 98-110

````python

parser = argparse.ArgumentParser(
    description="Generate test body using split-file and a custom script"
)
parser.add_argument("files", nargs="+")
parser.add_argument(
    "--shell", action="store_true", help="invoke shell instead of 'gen'"
)
args = parser.parse_args()
for path in args.files:
    retcode = process(args, path)
    if retcode != 0:
        sys.exit(retcode)
````
- **L98 EN**: Blank line separates nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L99 EN**: Assigns or updates `parser`.
  **L99 CN**: 对 `parser` 进行赋值或更新。
- **L100 EN**: Assigns or updates `description`.
  **L100 CN**: 对 `description` 进行赋值或更新。
- **L101 EN**: Executes Python statement `)`.
  **L101 CN**: 执行 Python 语句 `)`。
- **L102 EN**: Assigns or updates `parser.add_argument("files", nargs`.
  **L102 CN**: 对 `parser.add_argument("files", nargs` 进行赋值或更新。
- **L103 EN**: Executes Python statement `parser.add_argument(`.
  **L103 CN**: 执行 Python 语句 `parser.add_argument(`。
- **L104 EN**: Assigns or updates `"--shell", action`.
  **L104 CN**: 对 `"--shell", action` 进行赋值或更新。
- **L105 EN**: Executes Python statement `)`.
  **L105 CN**: 执行 Python 语句 `)`。
- **L106 EN**: Assigns or updates `args`.
  **L106 CN**: 对 `args` 进行赋值或更新。
- **L107 EN**: Controls Python flow with `for` logic.
  **L107 CN**: 使用 `for` 逻辑控制 Python 执行流程。
- **L108 EN**: Assigns or updates `retcode`.
  **L108 CN**: 对 `retcode` 进行赋值或更新。
- **L109 EN**: Controls Python flow with `if` logic.
  **L109 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L110 EN**: Executes Python statement `sys.exit(retcode)`.
  **L110 CN**: 执行 Python 语句 `sys.exit(retcode)`。

## Key Concepts / 关键概念

- EN: lit test runner infrastructure
  - CN: lit 测试运行基础设施
- EN: test harness behavior
  - CN: 测试框架行为
- EN: subprocess management
  - CN: 子进程管理
- EN: command-line parsing
  - CN: 命令行解析
- EN: Python utility scripting
  - CN: Python 工具脚本

## Dependencies / 依赖关系

- EN: `argparse` supplies command-line argument parsing.
  - CN: `argparse` 提供了命令行参数解析。
- EN: `contextlib` supplies supporting Python helpers.
  - CN: `contextlib` 提供了辅助性的 Python 模块。
- EN: `os` supplies operating-system services.
  - CN: `os` 提供了操作系统服务。
- EN: `re` supplies regular-expression support.
  - CN: `re` 提供了正则表达式支持。
- EN: `subprocess` supplies subprocess spawning and control.
  - CN: `subprocess` 提供了子进程创建与控制。
- EN: `sys` supplies Python runtime state and argv access.
  - CN: `sys` 提供了Python 运行时状态与 argv 访问。
- EN: `tempfile` supplies temporary-file helpers.
  - CN: `tempfile` 提供了临时文件辅助工具。
