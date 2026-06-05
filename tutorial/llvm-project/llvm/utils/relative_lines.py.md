# relative_lines.py — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/utils/relative_lines.py` | `llvm/utils/relative_lines.py` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Replaces absolute line numbers in lit-tests with relative line numbers. Writing line numbers like 152 in 'RUN: or CHECK:' makes tests hard to maintain: inserting lines in the middle of the test means updating all the... | 实现与 `relative_lines` 相关的 Python 工具逻辑、命令处理或辅助流程。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-11

````python
#!/usr/bin/env python3

"""Replaces absolute line numbers in lit-tests with relative line numbers.

Writing line numbers like 152 in 'RUN: or CHECK:' makes tests hard to maintain:
inserting lines in the middle of the test means updating all the line numbers.

Encoding them relative to the current line helps, and tools support it:
    Lit will substitute %(line+2) with the actual line number
    FileCheck supports [[@LINE+2]]

````
- **L1 EN**: Shebang selects the interpreter used to execute this script.
  **L1 CN**: Shebang 指定了执行该脚本时使用的解释器。
- **L2 EN**: Blank line separates nearby declarations or logic blocks.
  **L2 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3 EN**: Executes Python statement `"""Replaces absolute line numbers in lit-tests with relative line numbers.`.
  **L3 CN**: 执行 Python 语句 `"""Replaces absolute line numbers in lit-tests with relative line numbers.`。
- **L4 EN**: Blank line separates nearby declarations or logic blocks.
  **L4 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L5 EN**: Executes Python statement `Writing line numbers like 152 in 'RUN: or CHECK:' makes tests hard to maintain:`.
  **L5 CN**: 执行 Python 语句 `Writing line numbers like 152 in 'RUN: or CHECK:' makes tests hard to maintain:`。
- **L6 EN**: Executes Python statement `inserting lines in the middle of the test means updating all the line numbers.`.
  **L6 CN**: 执行 Python 语句 `inserting lines in the middle of the test means updating all the line numbers.`。
- **L7 EN**: Blank line separates nearby declarations or logic blocks.
  **L7 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L8 EN**: Executes Python statement `Encoding them relative to the current line helps, and tools support it:`.
  **L8 CN**: 执行 Python 语句 `Encoding them relative to the current line helps, and tools support it:`。
- **L9 EN**: Executes Python statement `Lit will substitute %(line+2) with the actual line number`.
  **L9 CN**: 执行 Python 语句 `Lit will substitute %(line+2) with the actual line number`。
- **L10 EN**: Executes Python statement `FileCheck supports [[@LINE+2]]`.
  **L10 CN**: 执行 Python 语句 `FileCheck supports [[@LINE+2]]`。
- **L11 EN**: Blank line separates nearby declarations or logic blocks.
  **L11 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 12-23

````python
This tool takes a regex which captures a line number, and a list of test files.
It searches for line numbers in the files and replaces them with a relative
line number reference.
"""

USAGE = """Example usage:
    find -type f clang/test/CodeCompletion | grep -v /Inputs/ | \\
    xargs relative_lines.py --dry-run --verbose --near=100 \\
    --pattern='-code-completion-at[ =]%s:(\d+)' \\
    --pattern='requires fix-it: {(\d+):\d+-(\d+):\d+}'
"""

````
- **L12 EN**: Executes Python statement `This tool takes a regex which captures a line number, and a list of test files.`.
  **L12 CN**: 执行 Python 语句 `This tool takes a regex which captures a line number, and a list of test files.`。
- **L13 EN**: Executes Python statement `It searches for line numbers in the files and replaces them with a relative`.
  **L13 CN**: 执行 Python 语句 `It searches for line numbers in the files and replaces them with a relative`。
- **L14 EN**: Executes Python statement `line number reference.`.
  **L14 CN**: 执行 Python 语句 `line number reference.`。
- **L15 EN**: Executes Python statement `"""`.
  **L15 CN**: 执行 Python 语句 `"""`。
- **L16 EN**: Blank line separates nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L17 EN**: Assigns or updates `USAGE`.
  **L17 CN**: 对 `USAGE` 进行赋值或更新。
- **L18 EN**: Executes Python statement `find -type f clang/test/CodeCompletion | grep -v /Inputs/ | \\`.
  **L18 CN**: 执行 Python 语句 `find -type f clang/test/CodeCompletion | grep -v /Inputs/ | \\`。
- **L19 EN**: Assigns or updates `xargs relative_lines.py --dry-run --verbose --near`.
  **L19 CN**: 对 `xargs relative_lines.py --dry-run --verbose --near` 进行赋值或更新。
- **L20 EN**: Assigns or updates `--pattern`.
  **L20 CN**: 对 `--pattern` 进行赋值或更新。
- **L21 EN**: Assigns or updates `--pattern`.
  **L21 CN**: 对 `--pattern` 进行赋值或更新。
- **L22 EN**: Executes Python statement `"""`.
  **L22 CN**: 执行 Python 语句 `"""`。
- **L23 EN**: Blank line separates nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 24-31

````python
import argparse
import re
import sys


def b(x):
    return bytes(x, encoding="utf-8")

````
- **L24 EN**: Imports Python module(s) `argparse` for supporting functionality.
  **L24 CN**: 导入 Python 模块 `argparse` 以提供辅助功能。
- **L25 EN**: Imports Python module(s) `re` for supporting functionality.
  **L25 CN**: 导入 Python 模块 `re` 以提供辅助功能。
- **L26 EN**: Imports Python module(s) `sys` for supporting functionality.
  **L26 CN**: 导入 Python 模块 `sys` 以提供辅助功能。
- **L27 EN**: Blank line separates nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Blank line separates nearby declarations or logic blocks.
  **L28 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L29 EN**: Declares function `b`.
  **L29 CN**: 声明函数 `b`。
- **L30 EN**: Returns a value or exits the current function.
  **L30 CN**: 返回一个值或结束当前函数。
- **L31 EN**: Blank line separates nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 32-47

````python

parser = argparse.ArgumentParser(
    prog="relative_lines",
    description=__doc__,
    epilog=USAGE,
    formatter_class=argparse.RawTextHelpFormatter,
)
parser.add_argument(
    "--near", type=int, default=20, help="maximum line distance to make relative"
)
parser.add_argument(
    "--partial",
    action="store_true",
    default=False,
    help="apply replacements to files even if others failed",
)
````
- **L32 EN**: Blank line separates nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L33 EN**: Assigns or updates `parser`.
  **L33 CN**: 对 `parser` 进行赋值或更新。
- **L34 EN**: Assigns or updates `prog`.
  **L34 CN**: 对 `prog` 进行赋值或更新。
- **L35 EN**: Assigns or updates `description`.
  **L35 CN**: 对 `description` 进行赋值或更新。
- **L36 EN**: Assigns or updates `epilog`.
  **L36 CN**: 对 `epilog` 进行赋值或更新。
- **L37 EN**: Assigns or updates `formatter_class`.
  **L37 CN**: 对 `formatter_class` 进行赋值或更新。
- **L38 EN**: Executes Python statement `)`.
  **L38 CN**: 执行 Python 语句 `)`。
- **L39 EN**: Executes Python statement `parser.add_argument(`.
  **L39 CN**: 执行 Python 语句 `parser.add_argument(`。
- **L40 EN**: Assigns or updates `"--near", type`.
  **L40 CN**: 对 `"--near", type` 进行赋值或更新。
- **L41 EN**: Executes Python statement `)`.
  **L41 CN**: 执行 Python 语句 `)`。
- **L42 EN**: Executes Python statement `parser.add_argument(`.
  **L42 CN**: 执行 Python 语句 `parser.add_argument(`。
- **L43 EN**: Executes Python statement `"--partial",`.
  **L43 CN**: 执行 Python 语句 `"--partial",`。
- **L44 EN**: Assigns or updates `action`.
  **L44 CN**: 对 `action` 进行赋值或更新。
- **L45 EN**: Assigns or updates `default`.
  **L45 CN**: 对 `default` 进行赋值或更新。
- **L46 EN**: Assigns or updates `help`.
  **L46 CN**: 对 `help` 进行赋值或更新。
- **L47 EN**: Executes Python statement `)`.
  **L47 CN**: 执行 Python 语句 `)`。

### Lines 48-63

````python
parser.add_argument(
    "--pattern",
    default=[],
    action="append",
    type=lambda x: re.compile(b(x)),
    help="regex to match, with line numbers captured in ().",
)
parser.add_argument(
    "--verbose", action="store_true", default=False, help="print matches applied"
)
parser.add_argument(
    "--dry-run",
    action="store_true",
    default=False,
    help="don't apply replacements. Best with --verbose.",
)
````
- **L48 EN**: Executes Python statement `parser.add_argument(`.
  **L48 CN**: 执行 Python 语句 `parser.add_argument(`。
- **L49 EN**: Executes Python statement `"--pattern",`.
  **L49 CN**: 执行 Python 语句 `"--pattern",`。
- **L50 EN**: Assigns or updates `default`.
  **L50 CN**: 对 `default` 进行赋值或更新。
- **L51 EN**: Assigns or updates `action`.
  **L51 CN**: 对 `action` 进行赋值或更新。
- **L52 EN**: Assigns or updates `type`.
  **L52 CN**: 对 `type` 进行赋值或更新。
- **L53 EN**: Assigns or updates `help`.
  **L53 CN**: 对 `help` 进行赋值或更新。
- **L54 EN**: Executes Python statement `)`.
  **L54 CN**: 执行 Python 语句 `)`。
- **L55 EN**: Executes Python statement `parser.add_argument(`.
  **L55 CN**: 执行 Python 语句 `parser.add_argument(`。
- **L56 EN**: Assigns or updates `"--verbose", action`.
  **L56 CN**: 对 `"--verbose", action` 进行赋值或更新。
- **L57 EN**: Executes Python statement `)`.
  **L57 CN**: 执行 Python 语句 `)`。
- **L58 EN**: Executes Python statement `parser.add_argument(`.
  **L58 CN**: 执行 Python 语句 `parser.add_argument(`。
- **L59 EN**: Executes Python statement `"--dry-run",`.
  **L59 CN**: 执行 Python 语句 `"--dry-run",`。
- **L60 EN**: Assigns or updates `action`.
  **L60 CN**: 对 `action` 进行赋值或更新。
- **L61 EN**: Assigns or updates `default`.
  **L61 CN**: 对 `default` 进行赋值或更新。
- **L62 EN**: Assigns or updates `help`.
  **L62 CN**: 对 `help` 进行赋值或更新。
- **L63 EN**: Executes Python statement `)`.
  **L63 CN**: 执行 Python 语句 `)`。

### Lines 64-73

````python
parser.add_argument("files", nargs="+")
args = parser.parse_args()

for file in args.files:
    try:
        contents = open(file, "rb").read()
    except UnicodeDecodeError as e:
        print(f"{file}: not valid UTF-8 - {e}", file=sys.stderr)
    failures = 0

````
- **L64 EN**: Assigns or updates `parser.add_argument("files", nargs`.
  **L64 CN**: 对 `parser.add_argument("files", nargs` 进行赋值或更新。
- **L65 EN**: Assigns or updates `args`.
  **L65 CN**: 对 `args` 进行赋值或更新。
- **L66 EN**: Blank line separates nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L67 EN**: Controls Python flow with `for` logic.
  **L67 CN**: 使用 `for` 逻辑控制 Python 执行流程。
- **L68 EN**: Controls Python flow with `try` logic.
  **L68 CN**: 使用 `try` 逻辑控制 Python 执行流程。
- **L69 EN**: Assigns or updates `contents`.
  **L69 CN**: 对 `contents` 进行赋值或更新。
- **L70 EN**: Controls Python flow with `except` logic.
  **L70 CN**: 使用 `except` 逻辑控制 Python 执行流程。
- **L71 EN**: Assigns or updates `print(f"{file}: not valid UTF-8 - {e}", file`.
  **L71 CN**: 对 `print(f"{file}: not valid UTF-8 - {e}", file` 进行赋值或更新。
- **L72 EN**: Assigns or updates `failures`.
  **L72 CN**: 对 `failures` 进行赋值或更新。
- **L73 EN**: Blank line separates nearby declarations or logic blocks.
  **L73 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 74-84

````python
    def line_number(offset):
        return 1 + contents[:offset].count(b"\n")

    def replace_one(capture, line, offset):
        """Text to replace a capture group, e.g. 42 => %(line+1)"""
        try:
            target = int(capture)
        except ValueError:
            print(f"{file}:{line}: matched non-number '{capture}'", file=sys.stderr)
            return capture

````
- **L74 EN**: Declares function `line_number`.
  **L74 CN**: 声明函数 `line_number`。
- **L75 EN**: Returns a value or exits the current function.
  **L75 CN**: 返回一个值或结束当前函数。
- **L76 EN**: Blank line separates nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L77 EN**: Declares function `replace_one`.
  **L77 CN**: 声明函数 `replace_one`。
- **L78 EN**: Assigns or updates `"""Text to replace a capture group, e.g. 42`.
  **L78 CN**: 对 `"""Text to replace a capture group, e.g. 42` 进行赋值或更新。
- **L79 EN**: Controls Python flow with `try` logic.
  **L79 CN**: 使用 `try` 逻辑控制 Python 执行流程。
- **L80 EN**: Assigns or updates `target`.
  **L80 CN**: 对 `target` 进行赋值或更新。
- **L81 EN**: Controls Python flow with `except` logic.
  **L81 CN**: 使用 `except` 逻辑控制 Python 执行流程。
- **L82 EN**: Assigns or updates `print(f"{file}:{line}: matched non-number '{capture}'", file`.
  **L82 CN**: 对 `print(f"{file}:{line}: matched non-number '{capture}'", file` 进行赋值或更新。
- **L83 EN**: Returns a value or exits the current function.
  **L83 CN**: 返回一个值或结束当前函数。
- **L84 EN**: Blank line separates nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 85-97

````python
        if args.near > 0 and abs(target - line) > args.near:
            print(
                f"{file}:{line}: target line {target} is farther than {args.near}",
                file=sys.stderr,
            )
            return capture
        if target > line:
            delta = "+" + str(target - line)
        elif target < line:
            delta = "-" + str(line - target)
        else:
            delta = ""

````
- **L85 EN**: Controls Python flow with `if` logic.
  **L85 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L86 EN**: Executes Python statement `print(`.
  **L86 CN**: 执行 Python 语句 `print(`。
- **L87 EN**: Executes Python statement `f"{file}:{line}: target line {target} is farther than {args.near}",`.
  **L87 CN**: 执行 Python 语句 `f"{file}:{line}: target line {target} is farther than {args.near}",`。
- **L88 EN**: Assigns or updates `file`.
  **L88 CN**: 对 `file` 进行赋值或更新。
- **L89 EN**: Executes Python statement `)`.
  **L89 CN**: 执行 Python 语句 `)`。
- **L90 EN**: Returns a value or exits the current function.
  **L90 CN**: 返回一个值或结束当前函数。
- **L91 EN**: Controls Python flow with `if` logic.
  **L91 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L92 EN**: Assigns or updates `delta`.
  **L92 CN**: 对 `delta` 进行赋值或更新。
- **L93 EN**: Controls Python flow with `elif` logic.
  **L93 CN**: 使用 `elif` 逻辑控制 Python 执行流程。
- **L94 EN**: Assigns or updates `delta`.
  **L94 CN**: 对 `delta` 进行赋值或更新。
- **L95 EN**: Controls Python flow with `else` logic.
  **L95 CN**: 使用 `else` 逻辑控制 Python 执行流程。
- **L96 EN**: Assigns or updates `delta`.
  **L96 CN**: 对 `delta` 进行赋值或更新。
- **L97 EN**: Blank line separates nearby declarations or logic blocks.
  **L97 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 98-105

````python
        prefix = contents[:offset].rsplit(b"\n")[-1]
        is_lit = b"RUN" in prefix or b"DEFINE" in prefix
        text = ("%(line{0})" if is_lit else "[[@LINE{0}]]").format(delta)
        if args.verbose:
            print(f"{file}:{line}: {0} ==> {text}")
        return b(text)

    def replace_match(m):
````
- **L98 EN**: Assigns or updates `prefix`.
  **L98 CN**: 对 `prefix` 进行赋值或更新。
- **L99 EN**: Assigns or updates `is_lit`.
  **L99 CN**: 对 `is_lit` 进行赋值或更新。
- **L100 EN**: Assigns or updates `text`.
  **L100 CN**: 对 `text` 进行赋值或更新。
- **L101 EN**: Controls Python flow with `if` logic.
  **L101 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L102 EN**: Executes Python statement `print(f"{file}:{line}: {0} ==> {text}")`.
  **L102 CN**: 执行 Python 语句 `print(f"{file}:{line}: {0} ==> {text}")`。
- **L103 EN**: Returns a value or exits the current function.
  **L103 CN**: 返回一个值或结束当前函数。
- **L104 EN**: Blank line separates nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L105 EN**: Declares function `replace_match`.
  **L105 CN**: 声明函数 `replace_match`。

### Lines 106-121

````python
        """Text to replace a whole match, e.g. --at=42:3 => --at=%(line+2):3"""
        line = 1 + contents[: m.start()].count(b"\n")
        result = b""
        pos = m.start()
        for index, capture in enumerate(m.groups()):
            index += 1  # re groups are conventionally 1-indexed
            result += contents[pos : m.start(index)]
            replacement = replace_one(capture, line, m.start(index))
            result += replacement
            if replacement == capture:
                global failures
                failures += 1
            pos = m.end(index)
        result += contents[pos : m.end()]
        return result

````
- **L106 EN**: Assigns or updates `"""Text to replace a whole match, e.g. --at`.
  **L106 CN**: 对 `"""Text to replace a whole match, e.g. --at` 进行赋值或更新。
- **L107 EN**: Assigns or updates `line`.
  **L107 CN**: 对 `line` 进行赋值或更新。
- **L108 EN**: Assigns or updates `result`.
  **L108 CN**: 对 `result` 进行赋值或更新。
- **L109 EN**: Assigns or updates `pos`.
  **L109 CN**: 对 `pos` 进行赋值或更新。
- **L110 EN**: Controls Python flow with `for` logic.
  **L110 CN**: 使用 `for` 逻辑控制 Python 执行流程。
- **L111 EN**: Assigns or updates `index +`.
  **L111 CN**: 对 `index +` 进行赋值或更新。
- **L112 EN**: Assigns or updates `result +`.
  **L112 CN**: 对 `result +` 进行赋值或更新。
- **L113 EN**: Assigns or updates `replacement`.
  **L113 CN**: 对 `replacement` 进行赋值或更新。
- **L114 EN**: Assigns or updates `result +`.
  **L114 CN**: 对 `result +` 进行赋值或更新。
- **L115 EN**: Controls Python flow with `if` logic.
  **L115 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L116 EN**: Executes Python statement `global failures`.
  **L116 CN**: 执行 Python 语句 `global failures`。
- **L117 EN**: Assigns or updates `failures +`.
  **L117 CN**: 对 `failures +` 进行赋值或更新。
- **L118 EN**: Assigns or updates `pos`.
  **L118 CN**: 对 `pos` 进行赋值或更新。
- **L119 EN**: Assigns or updates `result +`.
  **L119 CN**: 对 `result +` 进行赋值或更新。
- **L120 EN**: Returns a value or exits the current function.
  **L120 CN**: 返回一个值或结束当前函数。
- **L121 EN**: Blank line separates nearby declarations or logic blocks.
  **L121 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 122-128

````python
    for pattern in args.pattern:
        contents = re.sub(pattern, replace_match, contents)
    if failures > 0 and not args.partial:
        print(f"{file}: leaving unchanged (some failed, --partial not given)")
        continue
    if not args.dry_run:
        open(file, "wb").write(contents)
````
- **L122 EN**: Controls Python flow with `for` logic.
  **L122 CN**: 使用 `for` 逻辑控制 Python 执行流程。
- **L123 EN**: Assigns or updates `contents`.
  **L123 CN**: 对 `contents` 进行赋值或更新。
- **L124 EN**: Controls Python flow with `if` logic.
  **L124 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L125 EN**: Executes Python statement `print(f"{file}: leaving unchanged (some failed, --partial not given)")`.
  **L125 CN**: 执行 Python 语句 `print(f"{file}: leaving unchanged (some failed, --partial not given)")`。
- **L126 EN**: Executes Python statement `continue`.
  **L126 CN**: 执行 Python 语句 `continue`。
- **L127 EN**: Controls Python flow with `if` logic.
  **L127 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L128 EN**: Executes Python statement `open(file, "wb").write(contents)`.
  **L128 CN**: 执行 Python 语句 `open(file, "wb").write(contents)`。

## Key Concepts / 关键概念

- EN: lit test runner infrastructure
  - CN: lit 测试运行基础设施
- EN: pattern-based output verification
  - CN: 基于模式的输出校验
- EN: test harness behavior
  - CN: 测试框架行为
- EN: command-line parsing
  - CN: 命令行解析
- EN: Python utility scripting
  - CN: Python 工具脚本

## Dependencies / 依赖关系

- EN: `argparse` supplies command-line argument parsing.
  - CN: `argparse` 提供了命令行参数解析。
- EN: `re` supplies regular-expression support.
  - CN: `re` 提供了正则表达式支持。
- EN: `sys` supplies Python runtime state and argv access.
  - CN: `sys` 提供了Python 运行时状态与 argv 访问。
