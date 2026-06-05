# swizzle.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件:** `python/pycute/swizzle.py`
- **Purpose / 用途:** Implements bit-level swizzling and layout composition helpers for remapping offsets. / 实现基于位操作的 swizzle 以及用于偏移重映射的布局组合辅助类。

## Line-by-Line Analysis / 逐行分析

### Lines 1-37
```python
 1: #################################################################################################
 2: #
 3: # Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 4: # SPDX-License-Identifier: BSD-3-Clause
 5: #
 6: # Redistribution and use in source and binary forms, with or without
 7: # modification, are permitted provided that the following conditions are met:
 8: #
 9: # 1. Redistributions of source code must retain the above copyright notice, this
10: # list of conditions and the following disclaimer.
11: #
12: # 2. Redistributions in binary form must reproduce the above copyright notice,
13: # this list of conditions and the following disclaimer in the documentation
14: # and/or other materials provided with the distribution.
15: #
16: # 3. Neither the name of the copyright holder nor the names of its
17: # contributors may be used to endorse or promote products derived from
18: # this software without specific prior written permission.
19: #
20: # THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
21: # AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
22: # IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
23: # DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
24: # FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
25: # DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
26: # SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
27: # CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
28: # OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
29: # OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
30: #
31: #################################################################################################
32: 
33: """
34: Methods for layout swizzling
35: """
36: 
37: from .layout import *
```
**EN:** The header contains the standard license text, a short module docstring, and an import of all layout utilities. This module builds directly on the abstractions defined in `layout.py`.
**CN:** 开头包含标准许可证、简短的模块说明，以及对布局工具的整体导入。本模块直接建立在 `layout.py` 定义的抽象之上。

### Lines 40-45
```python
40: def shiftr(a, s):
41:   return a >> s if s > 0 else shiftl(a, -s)
42: 
43: 
44: def shiftl(a, s):
45:   return a << s if s > 0 else shiftr(a, -s)
```
**EN:** `shiftr` and `shiftl` are symmetric helpers that treat negative shift amounts by delegating to the opposite operation. This avoids duplicating sign-handling logic elsewhere.
**CN:** `shiftr` 和 `shiftl` 是对称的辅助函数：当移位量为负时，它们会委托给相反方向的移位函数，从而避免重复处理符号逻辑。

### Lines 48-59
```python
48: ## A generic Swizzle functor
49:  # 0bxxxxxxxxxxxxxxxYYYxxxxxxxZZZxxxx
50:  #                               ^--^  Base is the number of least-sig bits to keep constant
51:  #                  ^-^       ^-^      Bits is the number of bits in the mask
52:  #                    ^---------^      Shift is the distance to shift the YYY mask
53:  #                                       (pos shifts YYY to the right, neg shifts YYY to the left)
54:  #
55:  # e.g. Given
56:  # 0bxxxxxxxxxxxxxxxxYYxxxxxxxxxZZxxx
57:  # the result is
58:  # 0bxxxxxxxxxxxxxxxxYYxxxxxxxxxAAxxx where AA = ZZ xor YY
59:  #
```
**EN:** This comment block explains the swizzle bit pattern. It shows how a mask of source bits (`YYY`) is XORed into another bit region (`ZZZ`) while a low-bit base region stays fixed.
**CN:** 这段注释解释了 swizzle 的位模式：源位段 `YYY` 会通过 XOR 混入另一个位段 `ZZZ`，而低位的 base 区域保持不变。

### Lines 60-70
```python
60: class Swizzle:
61:   def __init__(self, bits, base, shift):
62:     assert bits >= 0
63:     assert base >= 0
64:     assert abs(shift) >= bits
65:     self.bits = bits
66:     self.base = base
67:     self.shift = shift
68:     bit_msk = (1 << bits) - 1
69:     self.yyy_msk = bit_msk << (base + max(0,shift))
70:     self.zzz_msk = bit_msk << (base - min(0,shift))
```
**EN:** `Swizzle.__init__` validates the swizzle parameters and precomputes the `YYY` and `ZZZ` bit masks. The `shift` constraint ensures the two bit regions do not overlap incorrectly.
**CN:** `Swizzle.__init__` 会校验 swizzle 参数，并预先计算 `YYY` 与 `ZZZ` 的位掩码。对 `shift` 的约束保证两个位区域不会发生错误重叠。

### Lines 72-74
```python
72:   # operator ()    (transform integer)
73:   def __call__(self, offset):
74:     return offset ^ shiftr(offset & self.yyy_msk, self.shift)
```
**EN:** `Swizzle.__call__` applies the transformation by XORing the shifted `YYY` mask into the input offset.
**CN:** `Swizzle.__call__` 通过把移位后的 `YYY` 掩码与输入偏移量做 XOR 来应用该变换。

### Lines 76-82
```python
76:   # Size of the domain
77:   def size(self):
78:     return 1 << (self.bits + self.base + abs(self.shift))
79: 
80:   # Size of the codomain
81:   def cosize(self):
82:     return self.size()
```
**EN:** `size` and `cosize` describe the swizzle domain and codomain. Because the transform is a permutation over the relevant bit window, the two values are identical.
**CN:** `size` 与 `cosize` 描述 swizzle 的定义域和值域大小。由于该变换在相关位窗口上本质是一个置换，所以二者相同。

### Lines 84-90
```python
84:   # print and str
85:   def __str__(self):
86:     return f"SW_{self.bits}_{self.base}_{self.shift}"
87: 
88:   # error msgs and representation
89:   def __repr__(self):
90:     return f"Swizzle({self.bits},{self.base},{self.shift})"
```
**EN:** `__str__` and `__repr__` provide compact textual encodings of the swizzle parameters.
**CN:** `__str__` 和 `__repr__` 提供了 swizzle 参数的紧凑文本表示。

### Lines 93-97
```python
93: class ComposedLayout(LayoutBase):
94:   def __init__(self, layoutB, offset, layoutA):
95:     self.layoutB = layoutB
96:     self.offset  = offset
97:     self.layoutA = layoutA
```
**EN:** `ComposedLayout.__init__` stores three parts: an outer layout or transform (`layoutB`), an offset, and an inner layout (`layoutA`).
**CN:** `ComposedLayout.__init__` 保存三个组成部分：外层布局或变换 `layoutB`、一个偏移量，以及内层布局 `layoutA`。

### Lines 99-101
```python
 99:   # operator ==
100:   def __eq__(self, other):
101:     return self.layoutB == other.layoutB and self.offset == other.offset and self.layoutA == other.layoutA
```
**EN:** `__eq__` defines equality for composed layouts by comparing all three stored components.
**CN:** `__eq__` 通过比较保存的三个组成部分来定义组合布局的相等性。

### Lines 103-113
```python
103:   # operator len(L)  (len [rank] like tuples)
104:   def __len__(self):
105:     return len(self.layoutA)
106: 
107:   # operator ()    (map coord to idx)
108:   def __call__(self, *args):
109:     return self.layoutB(self.offset + self.layoutA(*args))
110: 
111:   # operator []    (get-i like tuples)
112:   def __getitem__(self, i):
113:     return ComposedLayout(self.layoutB, self.offset, self.layoutA[i])
```
**EN:** `__len__`, `__call__`, and `__getitem__` forward rank, coordinate mapping, and mode extraction behavior to the inner layout while applying the outer transform to produced offsets.
**CN:** `__len__`、`__call__` 与 `__getitem__` 把秩、坐标映射和模式提取等行为委托给内层布局，同时对得到的偏移量施加外层变换。

### Lines 115-121
```python
115:   # size(layout)   Size of the domain
116:   def size(self):
117:     return size(self.layoutA)
118: 
119:   # cosize(layout)   Size of the codomain
120:   def cosize(self):
121:     return cosize(self.layoutB)
```
**EN:** `size` exposes the logical domain of the inner layout, while `cosize` reports the codomain size of the outer transform.
**CN:** `size` 暴露内层布局的逻辑定义域大小，而 `cosize` 则返回外层变换的值域大小。

### Lines 123-129
```python
123:   # print and str
124:   def __str__(self):
125:     return f"{self.layoutB} o {self.offset} o {self.layoutA}"
126: 
127:   # error msgs and representation
128:   def __repr__(self):
129:     return f"ComposedLayout({repr(self.layoutB)},{repr(self.offset)},{repr(self.layoutA)})"
```
**EN:** The string methods make composed layouts readable during debugging by displaying the composition order explicitly.
**CN:** 这些字符串方法在调试时能清晰展示组合顺序，从而提升可读性。

## Key Concepts / 关键概念

- **EN:** Bit-mask based address swizzling  
  **CN:** 基于位掩码的地址 swizzle
- **EN:** Shift normalization for signed offsets  
  **CN:** 面向正负位移的移位归一化
- **EN:** Composition of logical layouts with outer transforms  
  **CN:** 逻辑布局与外层变换的组合

## Dependencies / 依赖关系

- **EN:** Imports all layout helpers from `.layout`.  
  **CN:** 从 `.layout` 导入全部布局辅助工具。
- **EN:** `ComposedLayout` inherits from `LayoutBase` defined in `layout.py`.  
  **CN:** `ComposedLayout` 继承自 `layout.py` 中定义的 `LayoutBase`。
- **EN:** Relies on top-level `size` and `cosize` helpers for polymorphic behavior.  
  **CN:** 依赖顶层 `size` 和 `cosize` 辅助函数实现多态行为。
