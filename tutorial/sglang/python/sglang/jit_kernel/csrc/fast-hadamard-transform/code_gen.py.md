# code_gen.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/csrc/fast-hadamard-transform/code_gen.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from pathlib import Path". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from pathlib import Path”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-160: Imports and module setup
```python
from pathlib import Path

import numpy as np

# From https://en.wikipedia.org/wiki/Paley_construction (construction II for q = 5)

had_12_paley = """
+-++++++++++
--+-+-+-+-+-
+++-++----++
+---+--+-++-
+++++-++----
+-+---+--+-+
++--+++-++--
+--++---+--+
++----+++-++
+--+-++---+-
++++----+++-
+-+--+-++---
"""

# From http://neilsloane.com/hadamard/

had_12 = """
+-----------
++-+---+++-+
+++-+---+++-
+-++-+---+++
++-++-+---++
+++-++-+---+
++++-++-+---
+-+++-++-+--
+--+++-++-+-
+---+++-++-+
++---+++-++-
+-+---+++-++
"""

had_20_will = """
+----+----++--++-++-
-+----+---+++---+-++
--+----+---+++-+-+-+
---+----+---+++++-+-
----+----++--++-++-+
-+++++-----+--+++--+
+-+++-+---+-+--+++--
++-++--+---+-+--+++-
+++-+---+---+-+--+++
++++-----++--+-+--++
--++-+-++-+-----++++
---++-+-++-+---+-+++
+---++-+-+--+--++-++
++---++-+----+-+++-+
-++---++-+----+++++-
-+--+--++-+----+----
+-+-----++-+----+---
-+-+-+---+--+----+--
--+-+++------+----+-
+--+--++------+----+
"""


had_28_will = """
+------++----++-+--+-+--++--
-+-----+++-----+-+--+-+--++-
--+-----+++---+-+-+----+--++
---+-----+++---+-+-+-+--+--+
----+-----+++---+-+-+++--+--
-----+-----++++--+-+--++--+-
------++----++-+--+-+--++--+
--++++-+-------++--+++-+--+-
---++++-+-----+-++--+-+-+--+
+---+++--+----++-++--+-+-+--
++---++---+----++-++--+-+-+-
+++---+----+----++-++--+-+-+
++++--------+-+--++-++--+-+-
-++++--------+++--++--+--+-+
-+-++-++--++--+--------++++-
+-+-++--+--++--+--------++++
-+-+-++--+--++--+----+---+++
# ...
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 161-171: Function `string_to_array`
```python
def string_to_array(string):
    # Convert strings of + and - to bool arrays
    string = string.strip().replace("+", "1").replace("-", "-1").split()
    return np.stack(
        [
            np.fromstring(" ".join(string[i]), dtype=np.int32, sep=" ")
            for i in range(len(string))
        ]
    )
```
**EN:** This block defines `string_to_array`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `string_to_array`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 172-184: Function `array_code_gen`
```python
def array_code_gen(arr):
    N = arr.shape[0]
    assert arr.shape[0] == arr.shape[1]
    out = []
    for i in range(N):
        out.append(
            f"out[{i}] = "
            + " ".join([f"{'+' if arr[i, j] == 1 else '-'} x[{j}]" for j in range(N)])
            + ";"
        )
    return template.replace("{N}", str(N)).replace("{code}", "\n    ".join(out))
```
**EN:** This block defines `array_code_gen`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `array_code_gen`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 185-197: Function `main`
```python
def main():
    output_dir = Path(__file__).parent / "fast_hadamard_transform_special.h"
    output_dir.write_text(
        header
        + array_code_gen(string_to_array(had_12_paley))
        + array_code_gen(string_to_array(had_20_will))
        + array_code_gen(string_to_array(had_28_will))
        + array_code_gen(string_to_array(had_40_tpal))
    )


if __name__ == "__main__":
    main()
```
**EN:** This block defines `main`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `main`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `pathlib -> Path`
- `numpy as np`
