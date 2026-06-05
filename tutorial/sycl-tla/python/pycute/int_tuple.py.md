# int_tuple.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件:** `python/pycute/int_tuple.py`
- **Purpose / 用途:** Recursive utilities for nested integer/tuple structures used to represent shapes, strides, coordinates, and slicing behavior. / 提供递归工具函数，用于处理表示形状、步长、坐标和切片行为的嵌套整数/元组结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
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
```
**EN:** The file starts with the BSD-3-Clause license notice shared across the project.
**CN:** 文件开头是项目统一使用的 BSD-3-Clause 许可证声明。

### Lines 33-40
```python
33: """
34: Functions for manipulating IntTuples
35: """
36: 
37: from functools import reduce
38: from itertools import chain
39: from typing import Union
40: from .typing import Integer
```
**EN:** The module docstring states that this file manipulates “IntTuples”. It imports `reduce`, `chain`, `Union`, and the custom `Integer` ABC used to distinguish integer-like values from tuples.
**CN:** 模块文档字符串说明本文件用于处理 “IntTuple”。随后导入 `reduce`、`chain`、`Union` 以及自定义的 `Integer` 抽象基类，用于区分整数样对象与元组。

### Lines 43-44
```python
43: def is_int(x):
44:   return isinstance(x, Integer)
```
**EN:** `is_int` checks whether a value behaves like the project’s notion of an integer. It delegates the decision to the `Integer` abstract base class defined in `typing.py`.
**CN:** `is_int` 用来判断一个值是否符合项目定义的整数语义，实际判定逻辑委托给 `typing.py` 中定义的 `Integer` 抽象基类。

### Lines 47-48
```python
47: def is_tuple(x):
48:   return isinstance(x, tuple)
```
**EN:** `is_tuple` is a thin helper that recognizes Python tuples. Many later functions branch on tuple vs scalar structure.
**CN:** `is_tuple` 是一个轻量辅助函数，用于识别 Python 元组。后续很多函数都会根据“元组还是标量”走不同分支。

### Lines 51-58
```python
51: def flatten(t):
52:   if is_tuple(t):
53:     if len(t) == 0:
54:       return ()
55:     else:
56:       return tuple(i for a in t for i in flatten(a))
57:   else:
58:     return (t,)
```
**EN:** `flatten` recursively walks nested tuples and returns a flat tuple of terminal elements. Empty tuples stay empty, while scalars are wrapped as one-element tuples.
**CN:** `flatten` 递归展开嵌套元组，并返回由叶子元素组成的扁平元组。空元组保持为空，而标量会被包装成单元素元组。

### Lines 61-62
```python
61: def signum(a):
62:   return bool(a > 0) - bool(a < 0)
```
**EN:** `signum` computes the sign of a value using Boolean comparisons. The expression returns `1`, `0`, or `-1`.
**CN:** `signum` 通过布尔比较计算数值符号，结果分别为 `1`、`0` 或 `-1`。

### Lines 65-69
```python
65: def product(a):
66:   if is_tuple(a):
67:     return reduce(lambda val,elem : val*product(elem), a, 1)
68:   else:
69:     return a
```
**EN:** `product` multiplies all leaves of a nested tuple structure. For a scalar, it simply returns the scalar itself.
**CN:** `product` 会把嵌套元组中所有叶子元素相乘；若输入本身是标量，则直接返回该标量。

### Lines 72-79
```python
72: def inner_product(a, b):
73:   if is_tuple(a):                      # tuple tuple
74:     assert len(a) == len(b)
75:     return sum(inner_product(x,y) for x,y in zip(a,b))
76:   else:                                # "int" "int"
77:     assert not is_tuple(b)
78:     return a * b
79: 
```
**EN:** `inner_product` recursively computes a structural dot product. Tuple inputs must have matching shapes; scalar inputs are multiplied directly.
**CN:** `inner_product` 递归计算结构化点积。若输入是元组，结构必须匹配；若输入是标量，则直接相乘。

### Lines 81-85
```python
81: def tuple_max(a):
82:   if is_tuple(a):
83:     return max(tuple_max(x) for x in a)
84:   else:
85:     return a
```
**EN:** `tuple_max` finds the maximum leaf value in a nested tuple tree. Scalars are returned unchanged.
**CN:** `tuple_max` 在嵌套元组树中寻找最大叶子值；若输入是标量，则原样返回。

### Lines 88-99
```python
88: def elem_scale(a, b):
89:   if is_tuple(a):
90:     if is_tuple(b):                     # tuple tuple
91:       assert len(a) == len(b)
92:       return tuple(elem_scale(x,y) for x,y in zip(a,b))
93:     else:                               # tuple "int"
94:       assert False           # Error
95:   else:
96:     if is_tuple(b):                     # "int" tuple
97:       return elem_scale(a, product(b))
98:     else:                               # "int" "int"
99:       return a * b
```
**EN:** `elem_scale` performs element-wise style scaling with asymmetric rules. Tuple–tuple shapes must align, scalar–tuple falls back to multiplying by the tuple product, and tuple–scalar is rejected.
**CN:** `elem_scale` 实现一种带有不对称规则的逐元素缩放。元组对元组时结构必须一致；标量对元组会退化为乘以该元组乘积；元组对标量则被显式拒绝。

### Lines 102-120
```python
102: # Inclusive prefix ceil div with output congruent to input a
103: def shape_div(a, b):
104:   if is_tuple(a):
105:     if is_tuple(b):                    # tuple tuple
106:       assert len(a) == len(b)
107:       return tuple(shape_div(x,y) for x,y in zip(a,b))
108:     else:                              # tuple "int"
109:       #r = [shape_div(a[0],b)] + [shape_div(a[i],b := shape_div(b, product(a[i-1]))) for i in range(1,len(a))]
110:       r = []
111:       for v in a:
112:         r.append(shape_div(v,b))
113:         b = shape_div(b,product(v))
114:       return tuple(r)
115:   else:
116:     if is_tuple(b):                    # "int" tuple
117:       return shape_div(a, product(b))
118:     else:                              # "int" "int"
119:       assert a % b == 0 or b % a == 0
120:       return (a + b - 1) // b
```
**EN:** `shape_div` performs recursive ceil-division while preserving the structural “shape” of the left operand. Its assertions enforce divisibility relationships expected by layout arithmetic.
**CN:** `shape_div` 在保留左操作数结构形态的前提下执行递归向上整除。内部断言保证布局运算所依赖的可整除关系成立。

### Lines 123-139
```python
123: def prefix_product(a, init=1):
124:   if is_tuple(a):
125:     if is_tuple(init):                 # tuple tuple
126:       assert len(a) == len(init)
127:       return tuple(prefix_product(x,i) for x,i in zip(a,init))
128:     else:                              # tuple "int"
129:       #r = [prefix_product(a[0],init)] + [prefix_product(a[i],init := init * product(a[i-1])) for i in range(1,len(a))]
130:       r = []
131:       for v in a:
132:         r.append(prefix_product(v,init))
133:         init = init * product(v)
134:       return tuple(r)
135:   else:
136:     if is_tuple(init):                 # "int" tuple
137:       assert False           # Error
138:     else:                              # "int" "int"
139:       return init
```
**EN:** `prefix_product` computes exclusive prefix products. For tuples, it propagates an accumulator through each mode so that every leaf receives the product of all previous extents.
**CN:** `prefix_product` 计算排他前缀乘积。对于元组输入，它会逐模式传递累积值，使每个叶子得到其之前所有维度大小的乘积。

### Lines 142-158
```python
142: def idx2crd(idx, shape, stride=None):
143:   if stride is None:
144:     stride = prefix_product(shape)
145: 
146:   if is_tuple(idx):
147:     if is_tuple(shape):                # tuple tuple tuple
148:       assert len(idx) == len(shape) and len(idx) == len(stride)
149:       return tuple(idx2crd(i, s, d) for i, s, d in zip(idx,shape,stride))
150:     else:                              # tuple "int" "int"
151:       assert False           # Error
152:   else:
153:     if is_tuple(shape):                # "int" tuple tuple
154:       assert len(shape) == len(stride)
155:       return tuple(idx2crd(idx, s, d) for s,d in zip(shape,stride))
156:     else:                              # "int" "int" "int"
157:       return (idx // stride) % shape
158: 
```
**EN:** `idx2crd` converts a linear index into coordinates under a given shape and optional stride. If no stride is provided, a canonical prefix-product stride is synthesized.
**CN:** `idx2crd` 根据给定 shape 和可选 stride，把线性索引转换为坐标；若未提供 stride，则自动使用前缀乘积生成标准步长。

### Lines 160-182
```python
160: def crd2idx(crd, shape, stride=None):
161:   if stride is None:
162:     stride = prefix_product(shape)
163: 
164:   if is_tuple(crd):
165:     if is_tuple(shape):                # tuple tuple tuple
166:       assert len(crd) == len(shape) and len(crd) == len(stride)
167:       return sum(crd2idx(c, s, d) for c, s, d in zip(crd, shape, stride))
168:     else:                              # tuple "int" "int"
169:       assert False, f"crd={crd}, shape={shape}"           # Error
170:   else:
171:     if crd is None:
172:       crd = 0
173: 
174:     if is_tuple(shape):                # "int" tuple tuple
175:       assert len(shape) == len(stride)
176:       result = 0
177:       for i in range(len(shape)-1):
178:         result += crd2idx(crd % product(shape[i]), shape[i], stride[i])
179:         crd = crd // product(shape[i])
180:       return result + crd2idx(crd, shape[-1], stride[-1])
181:     else:                              # "int" "int" "int"
182:       return crd * stride
```
**EN:** `crd2idx` is the inverse-style mapping from coordinates to a linear index. It also supports flattened coordinates, nested shapes, and `None` terminals used by slicing logic.
**CN:** `crd2idx` 实现从坐标到线性索引的反向映射，同时支持扁平坐标、嵌套 shape，以及切片逻辑中使用的 `None` 终端值。

### Lines 185-200
```python
185: # Transform crd into the dst_shape's iteration space
186: def crd2crd(crd, dst_shape, src_shape=None):
187:   if is_tuple(crd):
188:     if is_tuple(dst_shape):            # tuple tuple
189:       assert len(crd) == len(dst_shape)
190:       return tuple(crd2crd(x, y) for x, y in zip(crd,dst_shape))
191:     else:                              # tuple "int"
192:       # Ambiguous unless we have src_shape
193:       assert src_shape is not None
194:       return crd2idx(crd, src_shape)
195:   else:
196:     if is_tuple(dst_shape):            # "int" tuple
197:       return idx2crd(crd, dst_shape)
198:     else:                              # "int" "int"
199:       assert crd < dst_shape
200:       return crd
```
**EN:** `crd2crd` remaps a coordinate into another iteration space. Depending on scalar/tuple structure, it either packs coordinates into an index or unpacks an index into coordinates.
**CN:** `crd2crd` 用于把坐标重新映射到另一种迭代空间。根据输入和目标的标量/元组结构，它会在“坐标打包为索引”和“索引解包为坐标”之间切换。

### Lines 203-217
```python
203: # Filter trg according to crd: keep only elements of trg that are paired with None
204: def slice_(crd: Union[None, tuple, int],
205:            trg: Union[tuple, int]):
206:   if is_tuple(crd):
207:     if is_tuple(trg):                  # tuple tuple
208:       assert len(crd) == len(trg)
209:       # match C++ behavior of `filter_tuple` using `tuple_cat(...)`
210:       return tuple(chain(*filter(lambda x: x != (), [slice_(c, s) for c, s in zip(crd, trg)])))
211:     else:
212:       assert False                     # tuple "int" : Error
213:   elif crd is None:
214:     # match C++ behavior `return cute::tuple<B>{b};`
215:     return (trg,)
216:   else:
217:     return ()
```
**EN:** `slice_` filters a target tuple structure according to a coordinate pattern. Terminals paired with `None` are kept, which mirrors the C++ CuTe slice semantics used for sublayout extraction.
**CN:** `slice_` 根据坐标模式过滤目标元组结构。与 `None` 配对的终端会被保留下来，这与 C++ CuTe 在提取子布局时的切片语义保持一致。

### Lines 220-225
```python
220: # Determine if None appears at any of an int_tuples' terminals
221: def has_none(a: Union[None, tuple, int]):
222:   if is_tuple(a):
223:     return any(has_none(v) for v in a)
224:   else:
225:     return a is None
```
**EN:** `has_none` recursively checks whether any terminal element is `None`. `Layout.__call__` later uses this to decide between indexing and slicing behavior.
**CN:** `has_none` 递归检查任意终端元素是否为 `None`。后面的 `Layout.__call__` 就用它来判断应执行索引还是切片。

## Key Concepts / 关键概念

- **EN:** Recursive structural programming over nested tuples  
  **CN:** 针对嵌套元组的递归结构化编程
- **EN:** Shape/stride arithmetic for tensor layouts  
  **CN:** 面向张量布局的 shape/stride 算术
- **EN:** Coordinate-index conversion utilities  
  **CN:** 坐标与索引互转工具
- **EN:** `None`-driven slicing semantics  
  **CN:** 基于 `None` 的切片语义

## Dependencies / 依赖关系

- **EN:** Uses `functools.reduce`, `itertools.chain`, and `typing.Union` from the standard library.  
  **CN:** 使用标准库中的 `functools.reduce`、`itertools.chain` 和 `typing.Union`。
- **EN:** Depends on `.typing.Integer` for integer-like type checks.  
  **CN:** 依赖 `.typing.Integer` 进行整数类型语义判断。
- **EN:** Provides core helpers consumed heavily by `layout.py`.  
  **CN:** 为 `layout.py` 提供大量核心基础工具。
