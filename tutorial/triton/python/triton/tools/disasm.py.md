# disasm.py — Code Analysis / 代码分析

## Source / 来源

- **Path:** `./python/triton/tools/disasm.py`
- **EN:** This source file at `./python/triton/tools/disasm.py` defines the main symbols `parseCtrl`, `processSassLines`, `get_sass`, `path_to_cuobjdump` for the surrounding Triton subsystem.
- **CN:** 位于 `./python/triton/tools/disasm.py` 的这个源文件为周边 Triton 子系统定义了主要符号 `parseCtrl`, `processSassLines`, `get_sass`, `path_to_cuobjdump`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1
```python
# MIT License
```
**EN:** At module scope, this comment block explains the surrounding logic or records implementation notes.
**CN:** 在模块级作用域中，这段注释用于说明周围逻辑，或记录实现细节。

### Lines 3-3
```python
# Copyright (c) 2020 Da Yan @ HKUST
```
**EN:** At module scope, this comment block explains the surrounding logic or records implementation notes.
**CN:** 在模块级作用域中，这段注释用于说明周围逻辑，或记录实现细节。

### Lines 5-10
```python
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
```
**EN:** At module scope, this comment block explains the surrounding logic or records implementation notes.
**CN:** 在模块级作用域中，这段注释用于说明周围逻辑，或记录实现细节。

### Lines 12-13
```python
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
```
**EN:** At module scope, this comment block explains the surrounding logic or records implementation notes.
**CN:** 在模块级作用域中，这段注释用于说明周围逻辑，或记录实现细节。

### Lines 15-21
```python
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
```
**EN:** At module scope, this comment block explains the surrounding logic or records implementation notes.
**CN:** 在模块级作用域中，这段注释用于说明周围逻辑，或记录实现细节。

### Lines 23-23
```python
import functools
```
**EN:** At module scope, this block imports functools so later definitions can reuse those modules or symbols.
**CN:** 在模块级作用域中，这段代码导入了 functools，供后续定义复用这些模块或符号。

### Lines 24-24
```python
import os
```
**EN:** At module scope, this block imports os so later definitions can reuse those modules or symbols.
**CN:** 在模块级作用域中，这段代码导入了 os，供后续定义复用这些模块或符号。

### Lines 25-25
```python
import re
```
**EN:** At module scope, this block imports re so later definitions can reuse those modules or symbols.
**CN:** 在模块级作用域中，这段代码导入了 re，供后续定义复用这些模块或符号。

### Lines 26-26
```python
import subprocess
```
**EN:** At module scope, this block imports subprocess so later definitions can reuse those modules or symbols.
**CN:** 在模块级作用域中，这段代码导入了 subprocess，供后续定义复用这些模块或符号。

### Lines 27-27
```python
import tempfile
```
**EN:** At module scope, this block imports tempfile so later definitions can reuse those modules or symbols.
**CN:** 在模块级作用域中，这段代码导入了 tempfile，供后续定义复用这些模块或符号。

### Lines 29-29
```python
FLINE_RE = re.compile(r'\s*/\*\w{4,}\*/\s*([^;]*;)\s*/\* 0x(\w{16}) \*/\s*')
```
**EN:** At module scope, this assignment updates `FLINE_RE` with `re.compile('\\s*/\\*\\w{4,}\\*/\\s*([^;]*;)\\s*/\\* 0x(\\w{16}) \\*/\\s*')`, establishing state, aliases, or configuration used later.
**CN:** 在模块级作用域中，这段赋值把 `re.compile('\\s*/\\*\\w{4,}\\*/\\s*([^;]*;)\\s*/\\* 0x(\\w{16}) \\*/\\s*')` 写入 `FLINE_RE`，为后续逻辑建立状态、别名或配置。

### Lines 30-30
```python
SLINE_RE = re.compile(r'\s*/\* 0x(\w{16}) \*/\s*')
```
**EN:** At module scope, this assignment updates `SLINE_RE` with `re.compile('\\s*/\\* 0x(\\w{16}) \\*/\\s*')`, establishing state, aliases, or configuration used later.
**CN:** 在模块级作用域中，这段赋值把 `re.compile('\\s*/\\* 0x(\\w{16}) \\*/\\s*')` 写入 `SLINE_RE`，为后续逻辑建立状态、别名或配置。

### Lines 31-31
```python
FNAME_RE = re.compile(r'\s*Function : (\w+)\s*')
```
**EN:** At module scope, this assignment updates `FNAME_RE` with `re.compile('\\s*Function : (\\w+)\\s*')`, establishing state, aliases, or configuration used later.
**CN:** 在模块级作用域中，这段赋值把 `re.compile('\\s*Function : (\\w+)\\s*')` 写入 `FNAME_RE`，为后续逻辑建立状态、别名或配置。

### Lines 32-32
```python
BRA_RE = re.compile(r'(.*BRA(?:\.U)? )(0x\w+);')
```
**EN:** At module scope, this assignment updates `BRA_RE` with `re.compile('(.*BRA(?:\\.U)? )(0x\\w+);')`, establishing state, aliases, or configuration used later.
**CN:** 在模块级作用域中，这段赋值把 `re.compile('(.*BRA(?:\\.U)? )(0x\\w+);')` 写入 `BRA_RE`，为后续逻辑建立状态、别名或配置。

### Lines 35-35
```python
def parseCtrl(sline):
```
**EN:** At module scope, this header declares the function `parseCtrl(sline)`, which is responsible for parse ctrl.
**CN:** 在模块级作用域中，这段头部声明了函数 `parseCtrl(sline)`，它负责处理 parse ctrl 相关逻辑。

### Lines 36-36
```python
    enc = int(SLINE_RE.match(sline).group(1), 16)
```
**EN:** Inside function `parseCtrl`, this assignment updates `enc` with `int(SLINE_RE.match(sline).group(1), 16)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `parseCtrl` 内部，这段赋值把 `int(SLINE_RE.match(sline).group(1), 16)` 写入 `enc`，为后续逻辑建立状态、别名或配置。

### Lines 37-37
```python
    stall = (enc >> 41) & 0xf
```
**EN:** Inside function `parseCtrl`, this assignment updates `stall` with `enc >> 41 & 15`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `parseCtrl` 内部，这段赋值把 `enc >> 41 & 15` 写入 `stall`，为后续逻辑建立状态、别名或配置。

### Lines 38-38
```python
    yld = (enc >> 45) & 0x1
```
**EN:** Inside function `parseCtrl`, this assignment updates `yld` with `enc >> 45 & 1`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `parseCtrl` 内部，这段赋值把 `enc >> 45 & 1` 写入 `yld`，为后续逻辑建立状态、别名或配置。

### Lines 39-39
```python
    wrtdb = (enc >> 46) & 0x7
```
**EN:** Inside function `parseCtrl`, this assignment updates `wrtdb` with `enc >> 46 & 7`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `parseCtrl` 内部，这段赋值把 `enc >> 46 & 7` 写入 `wrtdb`，为后续逻辑建立状态、别名或配置。

### Lines 40-40
```python
    readb = (enc >> 49) & 0x7
```
**EN:** Inside function `parseCtrl`, this assignment updates `readb` with `enc >> 49 & 7`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `parseCtrl` 内部，这段赋值把 `enc >> 49 & 7` 写入 `readb`，为后续逻辑建立状态、别名或配置。

### Lines 41-41
```python
    watdb = (enc >> 52) & 0x3f
```
**EN:** Inside function `parseCtrl`, this assignment updates `watdb` with `enc >> 52 & 63`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `parseCtrl` 内部，这段赋值把 `enc >> 52 & 63` 写入 `watdb`，为后续逻辑建立状态、别名或配置。

### Lines 43-43
```python
    yld_str = 'Y' if yld == 0 else '-'
```
**EN:** Inside function `parseCtrl`, this assignment updates `yld_str` with `'Y' if yld == 0 else '-'`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `parseCtrl` 内部，这段赋值把 `'Y' if yld == 0 else '-'` 写入 `yld_str`，为后续逻辑建立状态、别名或配置。

### Lines 44-44
```python
    wrtdb_str = '-' if wrtdb == 7 else str(wrtdb)
```
**EN:** Inside function `parseCtrl`, this assignment updates `wrtdb_str` with `'-' if wrtdb == 7 else str(wrtdb)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `parseCtrl` 内部，这段赋值把 `'-' if wrtdb == 7 else str(wrtdb)` 写入 `wrtdb_str`，为后续逻辑建立状态、别名或配置。

### Lines 45-45
```python
    readb_str = '-' if readb == 7 else str(readb)
```
**EN:** Inside function `parseCtrl`, this assignment updates `readb_str` with `'-' if readb == 7 else str(readb)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `parseCtrl` 内部，这段赋值把 `'-' if readb == 7 else str(readb)` 写入 `readb_str`，为后续逻辑建立状态、别名或配置。

### Lines 46-46
```python
    watdb_str = '--' if watdb == 0 else f'{watdb:02d}'
```
**EN:** Inside function `parseCtrl`, this assignment updates `watdb_str` with `'--' if watdb == 0 else f'{watdb:02d}'`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `parseCtrl` 内部，这段赋值把 `'--' if watdb == 0 else f'{watdb:02d}'` 写入 `watdb_str`，为后续逻辑建立状态、别名或配置。

### Lines 47-47
```python
    return f'{watdb_str}:{readb_str}:{wrtdb_str}:{yld_str}:{stall:x}'
```
**EN:** Inside function `parseCtrl`, this return statement sends `f'{watdb_str}:{readb_str}:{wrtdb_str}:{yld_str}:{stall:x}'` back to the caller as the result of the current routine.
**CN:** 在函数 `parseCtrl` 内部，这条返回语句把 `f'{watdb_str}:{readb_str}:{wrtdb_str}:{yld_str}:{stall:x}'` 作为当前过程的结果返回给调用方。

### Lines 50-50
```python
def processSassLines(fline, sline, labels):
```
**EN:** At module scope, this header declares the function `processSassLines(fline, sline, labels)`, which is responsible for process sass lines.
**CN:** 在模块级作用域中，这段头部声明了函数 `processSassLines(fline, sline, labels)`，它负责处理 process sass lines 相关逻辑。

### Lines 51-51
```python
    asm = FLINE_RE.match(fline).group(1)
```
**EN:** Inside function `processSassLines`, this assignment updates `asm` with `FLINE_RE.match(fline).group(1)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `processSassLines` 内部，这段赋值把 `FLINE_RE.match(fline).group(1)` 写入 `asm`，为后续逻辑建立状态、别名或配置。

### Lines 52-52
```python
    # Remove tailing space
```
**EN:** Inside function `processSassLines`, this comment block explains the surrounding logic or records implementation notes.
**CN:** 在函数 `processSassLines` 内部，这段注释用于说明周围逻辑，或记录实现细节。

### Lines 53-54
```python
    if asm.endswith(" ;"):
        asm = asm[:-2] + ";"
```
**EN:** Inside function `processSassLines`, this conditional checks `asm.endswith(' ;')` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `processSassLines` 内部，这段条件语句检查 `asm.endswith(' ;')`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 55-55
```python
    ctrl = parseCtrl(sline)
```
**EN:** Inside function `processSassLines`, this assignment updates `ctrl` with `parseCtrl(sline)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `processSassLines` 内部，这段赋值把 `parseCtrl(sline)` 写入 `ctrl`，为后续逻辑建立状态、别名或配置。

### Lines 56-56
```python
    # BRA target address
```
**EN:** Inside function `processSassLines`, this comment block explains the surrounding logic or records implementation notes.
**CN:** 在函数 `processSassLines` 内部，这段注释用于说明周围逻辑，或记录实现细节。

### Lines 57-62
```python
    if BRA_RE.match(asm) is not None:
        target = int(BRA_RE.match(asm).group(2), 16)
        if target in labels:
            pass
        else:
            labels[target] = len(labels)
```
**EN:** Inside function `processSassLines`, this conditional checks `BRA_RE.match(asm) is not None` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `processSassLines` 内部，这段条件语句检查 `BRA_RE.match(asm) is not None`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 63-63
```python
    return (f'{ctrl}', f'{asm}')
```
**EN:** Inside function `processSassLines`, this return statement sends `(f'{ctrl}', f'{asm}')` back to the caller as the result of the current routine.
**CN:** 在函数 `processSassLines` 内部，这条返回语句把 `(f'{ctrl}', f'{asm}')` 作为当前过程的结果返回给调用方。

### Lines 66-67
```python
@functools.lru_cache()
def get_sass(cubin_asm, fun=None):
```
**EN:** At module scope, this header declares the function `get_sass(cubin_asm, fun)`, which is responsible for get sass. Decorators: functools.lru_cache().
**CN:** 在模块级作用域中，这段头部声明了函数 `get_sass(cubin_asm, fun)`，它负责处理 get sass 相关逻辑。 装饰器包括：functools.lru_cache()。

### Lines 68-68
```python
    fd, path = tempfile.mkstemp()
```
**EN:** Inside function `get_sass`, this assignment updates `(fd, path)` with `tempfile.mkstemp()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `get_sass` 内部，这段赋值把 `tempfile.mkstemp()` 写入 `(fd, path)`，为后续逻辑建立状态、别名或配置。

### Lines 69-74
```python
    try:
        with open(fd, 'wb') as cubin:
            cubin.write(cubin_asm)
        sass = extract(path, fun)
    finally:
        os.remove(path)
```
**EN:** Inside function `get_sass`, this `try` block wraps fragile work so exceptions, fallback paths, and cleanup handlers can be managed explicitly.
**CN:** 在函数 `get_sass` 内部，这段 `try` 代码把可能出错的逻辑包裹起来，以便显式处理异常、兜底路径和清理动作。

### Lines 75-75
```python
    return sass
```
**EN:** Inside function `get_sass`, this return statement sends `sass` back to the caller as the result of the current routine.
**CN:** 在函数 `get_sass` 内部，这条返回语句把 `sass` 作为当前过程的结果返回给调用方。

### Lines 78-78
```python
def path_to_cuobjdump():
```
**EN:** At module scope, this header declares the function `path_to_cuobjdump()`, which is responsible for path to cuobjdump.
**CN:** 在模块级作用域中，这段头部声明了函数 `path_to_cuobjdump()`，它负责处理 path to cuobjdump 相关逻辑。

### Lines 79-79
```python
    from triton import knobs
```
**EN:** Inside function `path_to_cuobjdump`, this block imports knobs from `triton` to connect this file with nearby APIs and helpers.
**CN:** 在函数 `path_to_cuobjdump` 内部，这段代码从 `triton` 导入 knobs，把当前文件与周边 API 和辅助工具连接起来。

### Lines 80-80
```python
    return knobs.nvidia.cuobjdump.path
```
**EN:** Inside function `path_to_cuobjdump`, this return statement sends `knobs.nvidia.cuobjdump.path` back to the caller as the result of the current routine.
**CN:** 在函数 `path_to_cuobjdump` 内部，这条返回语句把 `knobs.nvidia.cuobjdump.path` 作为当前过程的结果返回给调用方。

### Lines 83-83
```python
def extract(file_path, fun):
```
**EN:** At module scope, this header declares the function `extract(file_path, fun)`, which is responsible for extract.
**CN:** 在模块级作用域中，这段头部声明了函数 `extract(file_path, fun)`，它负责处理 extract 相关逻辑。

### Lines 84-84
```python
    cuobjdump = path_to_cuobjdump()
```
**EN:** Inside function `extract`, this assignment updates `cuobjdump` with `path_to_cuobjdump()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `extract` 内部，这段赋值把 `path_to_cuobjdump()` 写入 `cuobjdump`，为后续逻辑建立状态、别名或配置。

### Lines 85-88
```python
    if fun is None:
        sass_str = subprocess.check_output([cuobjdump, "-sass", file_path])
    else:
        sass_str = subprocess.check_output([cuobjdump, "-fun", fun, "-sass", file_path])
```
**EN:** Inside function `extract`, this conditional checks `fun is None` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `extract` 内部，这段条件语句检查 `fun is None`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 89-89
```python
    sass_lines = sass_str.splitlines()
```
**EN:** Inside function `extract`, this assignment updates `sass_lines` with `sass_str.splitlines()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `extract` 内部，这段赋值把 `sass_str.splitlines()` 写入 `sass_lines`，为后续逻辑建立状态、别名或配置。

### Lines 90-90
```python
    line_idx = 0
```
**EN:** Inside function `extract`, this assignment updates `line_idx` with `0`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `extract` 内部，这段赋值把 `0` 写入 `line_idx`，为后续逻辑建立状态、别名或配置。

### Lines 91-145
```python
    while line_idx < len(sass_lines):
        line = sass_lines[line_idx].decode()
        # format:
        # function : <function_name>
        # .headerflags: ...
        # /*0000*/ asmstr /*0x...*/
        #                 /*0x...*/

        # Looking for new function header (function: <name>)
        while FNAME_RE.match(line) is None:
            line_idx += 1
            if line_idx < len(sass_lines):
                line = sass_lines[line_idx].decode()
            else:
                return

        fname = FNAME_RE.match(line).group(1)
        ret = ''
        ret += f'Function:{fname}\n'
        line_idx += 2  # bypass .headerflags
        line = sass_lines[line_idx].decode()
        # Remapping address to label
        labels = {}  # address -> label_idx
        # store sass asm in buffer and them print them (for labels)
        # (ctrl, asm)
        asm_buffer = []
        while FLINE_RE.match(line) is not None:
            # First line (Offset ASM Encoding)
            fline = sass_lines[line_idx].decode()
            line_idx += 1
            # Second line (Encoding)
            sline = sass_lines[line_idx].decode()
            line_idx += 1
            asm_buffer.append(processSassLines(fline, sline, labels))
            # peek the next line
            if line_idx >= len(sass_lines):
                break
            line = sass_lines[line_idx].decode()
        # Print sass
        # label naming convention: LBB#i
        for idx, (ctrl, asm) in enumerate(asm_buffer):
            # Print label if this is BRA target
            offset = idx * 16
            if offset in labels:
                label_name = f'LBB{labels[offset]}'
                ret += f'{label_name}:\n'
            ret += ctrl + '\t'
            # if this is BRA, remap offset to label
            if BRA_RE.match(asm):
                target = int(BRA_RE.match(asm).group(2), 16)
                target_name = f'LBB{labels[target]}'
                asm = BRA_RE.sub(rf'\1{target_name};', asm)
            ret += asm + '\n'
        ret += '\n'
        return ret
```
**EN:** Inside function `extract`, this loop keeps running while `line_idx < len(sass_lines)` remains true.
**CN:** 在函数 `extract` 内部，这段循环会在 `line_idx < len(sass_lines)` 为真时持续执行。

## Key Concepts / 关键概念

- **EN:** Path theme: `python/triton/tools` places this module in Triton's triton / tools area.
  **CN:** 路径主题：`python/triton/tools` 表明该模块位于 Triton 的 triton / tools 领域。
- **EN:** Primary functions: `parseCtrl`, `processSassLines`, `get_sass`, `path_to_cuobjdump`, `extract`.
  **CN:** 主要函数：`parseCtrl`, `processSassLines`, `get_sass`, `path_to_cuobjdump`, `extract`。
- **EN:** Tooling surface: this file is part of Triton's CLI or code-transformation utilities.
  **CN:** 工具层：该文件属于 Triton 的命令行或代码转换工具。

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: functools, os, re, subprocess, tempfile.
  **CN:** 标准库依赖：functools, os, re, subprocess, tempfile。
- **EN:** Internal Triton modules: triton.
  **CN:** Triton 内部模块：triton。
