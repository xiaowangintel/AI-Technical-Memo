# utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/examples/customization/pwd-cd-and-system/utils.py`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Utility for changing directories and execution of commands in a subshell.
  - **CN**: 演示 LLDB 命令定制与 shell 集成技术。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6
```python
"""Utility for changing directories and execution of commands in a subshell."""

import os
import shlex
import subprocess

```
- **EN**: Imports the Python modules this LLDB helper depends on, including `os`, `shlex`, `subprocess`.
- **CN**: 导入该 LLDB 辅助脚本依赖的 Python 模块，其中包括 `os`, `shlex`, `subprocess`。

### Lines 7-10
```python
# Store the previous working directory for the 'cd -' command.


class Holder:
```
- **EN**: Introduces declarations for `Holder`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `Holder` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 11-14
```python
    """Holds the _prev_dir_ class attribute for chdir() function."""

    _prev_dir_ = None

```
- **EN**: Demonstrates logic around `chdir`; this block uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `chdir` 的脚本逻辑；该代码块使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 15-18
```python
    @classmethod
    def prev_dir(cls):
        return cls._prev_dir_

```
- **EN**: Demonstrates logic around `prev_dir`; this block uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `prev_dir` 的脚本逻辑；该代码块使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 19-22
```python
    @classmethod
    def swap(cls, dir):
        cls._prev_dir_ = dir

```
- **EN**: Demonstrates logic around `swap`; this block uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `swap` 的脚本逻辑；该代码块使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 23-30
```python

def chdir(debugger, args, result, dict):
    """Change the working directory, or cd to ${HOME}.
    You can also issue 'cd -' to change to the previous working directory."""
    new_dir = args.strip()
    if not new_dir:
        new_dir = os.path.expanduser("~")
    elif new_dir == "-":
```
- **EN**: Demonstrates logic around `chdir`, `strip`, `expanduser`; this block coordinates debugger runtime objects, events, or asynchronous control flow; uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `chdir`, `strip`, `expanduser` 的脚本逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 31-37
```python
        if not Holder.prev_dir():
            # Bad directory, not changing.
            print("bad directory, not changing")
            return
        else:
            new_dir = Holder.prev_dir()

```
- **EN**: Demonstrates logic around `prev_dir`; this block uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `prev_dir` 的脚本逻辑；该代码块使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 38-41
```python
    Holder.swap(os.getcwd())
    os.chdir(new_dir)
    print("Current working directory: %s" % os.getcwd())

```
- **EN**: Demonstrates logic around `swap`, `chdir`, `getcwd`; this block uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `swap`, `chdir`, `getcwd` 的脚本逻辑；该代码块使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 42-49
```python

def system(debugger, command_line, result, dict):
    """Execute the command (a string) in a subshell."""
    args = shlex.split(command_line)
    process = subprocess.Popen(args, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    output, error = process.communicate()
    retcode = process.poll()
    if output and error:
```
- **EN**: Demonstrates logic around `system`, `command`, `split`, `Popen`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms; registers commands, plugins, or interpreter-facing extension points; uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `system`, `command`, `split`, `Popen`, and 2 more symbols 的脚本逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源，并注册命令、插件或面向解释器的扩展点，并使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 50-56
```python
        print("stdout=>\n", output)
        print("stderr=>\n", error)
    elif output:
        print(output)
    elif error:
        print(error)
    print("retcode:", retcode)
```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
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

- **Imported modules / 导入模块**: `os`, `shlex`, `subprocess`
- **Module categories / 模块类别**: Python standard-library OS interfaces / Python 标准库操作系统接口 (1), Python standard library or local helper / Python 标准库或本地辅助模块 (1), Python standard-library subprocess control / Python 标准库子进程控制 (1)
