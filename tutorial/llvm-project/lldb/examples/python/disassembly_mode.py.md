# disassembly_mode.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/examples/python/disassembly_mode.py`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Adds the 'toggle-disassembly' command to switch you into a disassembly only mode.
  - **CN**: 实现通过 LLDB 脚本 API 自动化调试的 Python 示例脚本。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4
```python
""" Adds the 'toggle-disassembly' command to switch you into a disassembly only mode """
import lldb


```
- **EN**: Imports the Python modules this LLDB helper depends on, including `lldb`.
- **CN**: 导入该 LLDB 辅助脚本依赖的 Python 模块，其中包括 `lldb`。

### Lines 5-11
```python
class DisassemblyMode:
    def __init__(self, debugger, unused):
        self.dbg = debugger
        self.interp = debugger.GetCommandInterpreter()
        self.store_state()
        self.mode_off = True

```
- **EN**: Introduces declarations for `DisassemblyMode`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `DisassemblyMode` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 12-17
```python
    def store_state(self):
        self.dis_count = self.get_string_value("stop-disassembly-count")
        self.dis_display = self.get_string_value("stop-disassembly-display")
        self.before_count = self.get_string_value("stop-line-count-before")
        self.after_count = self.get_string_value("stop-line-count-after")

```
- **EN**: Demonstrates logic around `store_state`, `get_string_value`; this block uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `store_state`, `get_string_value` 的脚本逻辑；该代码块使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 18-23
```python
    def get_string_value(self, setting):
        result = lldb.SBCommandReturnObject()
        self.interp.HandleCommand("settings show " + setting, result)
        value = result.GetOutput().split(" = ")[1].rstrip("\n")
        return value

```
- **EN**: Demonstrates logic around `get_string_value`, `SBCommandReturnObject`, `HandleCommand`, `GetOutput`; this block uses Python scripting hooks to extend or automate LLDB; defines user-visible settings, options, or policy flags.
- **CN**: 演示围绕 `get_string_value`, `SBCommandReturnObject`, `HandleCommand`, `GetOutput` 的脚本逻辑；该代码块使用 Python 脚本钩子扩展或自动化 LLDB，并定义用户可见的设置、选项或策略标志。

### Lines 24-27
```python
    def set_value(self, setting, value):
        result = lldb.SBCommandReturnObject()
        self.interp.HandleCommand("settings set " + setting + " " + value, result)

```
- **EN**: Demonstrates logic around `set_value`, `SBCommandReturnObject`, `HandleCommand`; this block uses Python scripting hooks to extend or automate LLDB; defines user-visible settings, options, or policy flags.
- **CN**: 演示围绕 `set_value`, `SBCommandReturnObject`, `HandleCommand` 的脚本逻辑；该代码块使用 Python 脚本钩子扩展或自动化 LLDB，并定义用户可见的设置、选项或策略标志。

### Lines 28-35
```python
    def __call__(self, debugger, command, exe_ctx, result):
        if self.mode_off:
            self.mode_off = False
            self.store_state()
            self.set_value("stop-disassembly-display", "always")
            self.set_value("stop-disassembly-count", "8")
            self.set_value("stop-line-count-before", "0")
            self.set_value("stop-line-count-after", "0")
```
- **EN**: Demonstrates logic around `__call__`, `store_state`, `set_value`; this block coordinates debugger runtime objects, events, or asynchronous control flow; registers commands, plugins, or interpreter-facing extension points; uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `__call__`, `store_state`, `set_value` 的脚本逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并注册命令、插件或面向解释器的扩展点，并使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 36-43
```python
            result.AppendMessage("Disassembly mode on.")
        else:
            self.mode_off = True
            self.set_value("stop-disassembly-display", self.dis_display)
            self.set_value("stop-disassembly-count", self.dis_count)
            self.set_value("stop-line-count-before", self.before_count)
            self.set_value("stop-line-count-after", self.after_count)
            result.AppendMessage("Disassembly mode off.")
```
- **EN**: Demonstrates logic around `AppendMessage`, `set_value`; this block uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `AppendMessage`, `set_value` 的脚本逻辑；该代码块使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 44-47
```python

    def get_short_help(self):
        return "Toggles between a disassembly only mode and normal source mode\n"

```
- **EN**: Demonstrates logic around `get_short_help`; this block uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `get_short_help` 的脚本逻辑；该代码块使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 48-52
```python

def __lldb_init_module(debugger, unused):
    debugger.HandleCommand(
        "command script add -o -c disassembly_mode.DisassemblyMode toggle-disassembly"
    )
```
- **EN**: Demonstrates logic around `__lldb_init_module`, `HandleCommand`; this block coordinates debugger runtime objects, events, or asynchronous control flow; registers commands, plugins, or interpreter-facing extension points; uses Python scripting hooks to extend or automate LLDB.
- **CN**: 演示围绕 `__lldb_init_module`, `HandleCommand` 的脚本逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并注册命令、插件或面向解释器的扩展点，并使用 Python 脚本钩子扩展或自动化 LLDB。

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

- **Imported modules / 导入模块**: `lldb`
- **Module categories / 模块类别**: LLDB Python scripting APIs / LLDB Python 脚本 API (1)
