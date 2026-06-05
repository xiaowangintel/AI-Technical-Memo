# filter_disasm.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/examples/python/filter_disasm.py`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines a command, fdis, that does filtered disassembly. The command does the lldb disassemble command with -b and any other arguments passed in, and pipes that through a provided filter program. The intention is to support disassembly of RISC-V proprietary instructions. This is handled with llvm-objdump by piping the output of llvm-objdump through a filter program. This script is intended to mimic that workflow.
  - **CN**: 实现通过 LLDB 脚本 API 自动化调试的 Python 示例脚本。

## Line-by-Line Analysis / 逐行分析

### Lines 1-5
```python
"""
Defines a command, fdis, that does filtered disassembly. The command does the
lldb disassemble command with -b and any other arguments passed in, and
pipes that through a provided filter program.

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 6-10
```python
The intention is to support disassembly of RISC-V proprietary instructions.
This is handled with llvm-objdump by piping the output of llvm-objdump through
a filter program. This script is intended to mimic that workflow.
"""

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 11-15
```python
import lldb
import subprocess


class Program(list):
```
- **EN**: Imports the Python modules this LLDB helper depends on, including `lldb`, `subprocess`.
- **CN**: 导入该 LLDB 辅助脚本依赖的 Python 模块，其中包括 `lldb`, `subprocess`。

### Lines 16-21
```python
    def __str__(self):
        return " ".join(self)


filter_program = Program(["crustfilt"])

```
- **EN**: Demonstrates logic around `__str__`, `join`, `Program`; this block uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `__str__`, `join`, `Program` 的脚本逻辑；该代码块使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 22-26
```python
def __lldb_init_module(debugger, dict):
    debugger.HandleCommand("command script add -f filter_disasm.fdis fdis")
    print("Disassembly filter command (fdis) loaded")
    print("Filter program set to %s" % filter_program)

```
- **EN**: Demonstrates logic around `__lldb_init_module`, `HandleCommand`, `command`; this block coordinates debugger runtime objects, events, or asynchronous control flow; registers commands, plugins, or interpreter-facing extension points; uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `__lldb_init_module`, `HandleCommand`, `command` 的脚本逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并注册命令、插件或面向解释器的扩展点，并使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 27-36
```python

def fdis(debugger, args, exe_ctx, result, dict):
    """
  Call the built in disassembler, then pass its output to a filter program
  to add in disassembly for hidden opcodes.
  Except for get and set, use the fdis command like the disassemble command.
  By default, the filter program is crustfilt, from
  https://github.com/quic/crustfilt . This can be changed by changing
  the global variable filter_program.

```
- **EN**: Demonstrates logic around `fdis`; this block coordinates debugger runtime objects, events, or asynchronous control flow; registers commands, plugins, or interpreter-facing extension points; uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `fdis` 的脚本逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并注册命令、插件或面向解释器的扩展点，并使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 37-43
```python
  Usage:
    fdis [[get] [set <program>] [<disassembly options>]]

    Choose one of the following:
        get
            Gets the current filter program

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 44-51
```python
        set <program>
            Sets the current filter program. This can be an executable, which
            will be found on PATH, or an absolute path.

        <disassembly options>
            If the first argument is not get or set, the args will be passed
            to the disassemble command as is.

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 52-57
```python
    """

    global filter_program
    args_list = args.split(" ")
    result.Clear()

```
- **EN**: Demonstrates logic around `split`, `Clear`; this block uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `split`, `Clear` 的脚本逻辑；该代码块使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 58-62
```python
    if len(args_list) == 1 and args_list[0] == "get":
        result.PutCString(str(filter_program))
        result.SetStatus(lldb.eReturnStatusSuccessFinishResult)
        return

```
- **EN**: Demonstrates logic around `len`, `PutCString`, `SetStatus`; this block uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `len`, `PutCString`, `SetStatus` 的脚本逻辑；该代码块使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 63-70
```python
    if args_list[0] == "set":
        # Assume the rest is a program to run and any arguments to be passed to
        # it.
        if len(args_list) <= 1:
            result.PutCString('"set" command requires a program argument')
            result.SetStatus(lldb.eReturnStatusFailed)
            return

```
- **EN**: Demonstrates logic around `len`, `PutCString`, `SetStatus`; this block registers commands, plugins, or interpreter-facing extension points; uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `len`, `PutCString`, `SetStatus` 的脚本逻辑；该代码块注册命令、插件或面向解释器的扩展点，并使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 71-75
```python
        filter_program = Program(args_list[1:])
        result.PutCString('Filter program set to "{}"'.format(filter_program))
        result.SetStatus(lldb.eReturnStatusSuccessFinishResult)
        return

```
- **EN**: Demonstrates logic around `Program`, `PutCString`, `SetStatus`; this block controls debugger-side formatting or synthetic presentation of values; uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `Program`, `PutCString`, `SetStatus` 的脚本逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示，并使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 76-83
```python
    res = lldb.SBCommandReturnObject()
    debugger.GetCommandInterpreter().HandleCommand("disassemble -b " + args, exe_ctx, res)
    if len(res.GetError()) > 0:
        result.SetError(res.GetError())
        result.SetStatus(lldb.eReturnStatusFailed)
        return
    output = res.GetOutput()

```
- **EN**: Demonstrates logic around `SBCommandReturnObject`, `GetCommandInterpreter`, `len`, `SetError`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `SBCommandReturnObject`, `GetCommandInterpreter`, `len`, `SetError`, and 2 more symbols 的脚本逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 84-93
```python
    try:
        proc = subprocess.run(
            filter_program, capture_output=True, text=True, input=output
        )
    except (subprocess.SubprocessError, OSError) as e:
        result.PutCString("Error occurred. Original disassembly:\n\n" + output)
        result.SetError(str(e))
        result.SetStatus(lldb.eReturnStatusFailed)
        return

```
- **EN**: Demonstrates logic around `run`, `except`, `PutCString`, `SetError`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `run`, `except`, `PutCString`, `SetError`, and 1 more symbols 的脚本逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 94-98
```python
    if proc.returncode:
        result.PutCString("warning: {} returned non-zero value {}".format(filter_program, proc.returncode))

    result.PutCString(proc.stdout)
    result.SetStatus(lldb.eReturnStatusSuccessFinishResult)
```
- **EN**: Demonstrates logic around `PutCString`, `SetStatus`; this block controls debugger-side formatting or synthetic presentation of values; uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `PutCString`, `SetStatus` 的脚本逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示，并使用 Python 脚本钩子扩展或自动化 LLDB。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Python scripting / Python 脚本化**:
  - **EN**: Demonstrates how LLDB exposes automation hooks and debugger extensions through Python.
  - **CN**: 演示 LLDB 如何通过 Python 暴露自动化钩子与调试器扩展。
- **Symbolication / 符号化**:
  - **EN**: Relates raw addresses back to modules, sections, symbols, and source locations.
  - **CN**: 将原始地址关联回模块、节区、符号与源码位置。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。
- **Python automation / Python 自动化**:
  - **EN**: Uses Python entry points, modules, and callbacks to script debugger behavior.
  - **CN**: 使用 Python 入口、模块与回调来脚本化调试器行为。
- **Example workflows / 示例工作流**:
  - **EN**: Provides runnable examples that teach extension points or debugger usage patterns.
  - **CN**: 提供可运行的示例，用于讲解扩展点或调试器使用模式。

## Dependencies / 依赖关系

- **Imported modules / 导入模块**: `lldb`, `subprocess`
- **Module categories / 模块类别**: LLDB Python scripting APIs / LLDB Python 脚本 API (1), Python standard-library subprocess control / Python 标准库子进程控制 (1)
