# layout.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件:** `python/pycute/layout.py`
- **Purpose / 用途:** Defines the core `Layout` abstraction and the algebra used to compose, invert, divide, tile, and slice tensor layouts. / 定义核心 `Layout` 抽象，以及用于组合、求逆、划分、分块和切片张量布局的代数操作。

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
**EN:** This is the shared BSD-3-Clause license header for the file.
**CN:** 这里是该文件共用的 BSD-3-Clause 许可证头。

### Lines 33-40
```python
33: """
34: Definition of CuTe Layouts and functions to manipulate them
35: """
36: 
37: from itertools import chain
38: from typing import Union
39: 
40: from .int_tuple import *
```
**EN:** The module docstring states the file defines CuTe layouts and manipulation utilities. The imports bring in iterator helpers, a type alias, and the full IntTuple toolbox.
**CN:** 模块文档字符串说明本文件定义 CuTe 布局及其操作函数。导入部分引入了迭代工具、类型别名，以及完整的 IntTuple 工具集。

### Lines 43-44
```python
43: class LayoutBase:
44:   pass
```
**EN:** `LayoutBase` is an empty marker base class. It provides a common nominal type for both simple layouts and composed/swizzled layouts.
**CN:** `LayoutBase` 是一个空的标记基类，为普通布局和组合/置换布局提供统一的名义类型。

### Lines 47-48
```python
47: def is_layout(x):
48:   return isinstance(x, LayoutBase)
```
**EN:** `is_layout` centralizes the instance check for layout-like objects.
**CN:** `is_layout` 集中封装了对布局对象的实例判断。

### Lines 51-57
```python
51: class Layout(LayoutBase):
52:   def __init__(self, _shape, _stride=None):
53:     self.shape  = _shape
54:     if _stride is None:
55:       self.stride = prefix_product(self.shape)
56:     else:
57:       self.stride = _stride
```
**EN:** `Layout.__init__` stores the logical shape and either a supplied stride or a default prefix-product stride. This encodes the standard row-major-like linearization used throughout the module.
**CN:** `Layout.__init__` 保存逻辑 shape，并使用传入 stride 或默认的前缀乘积 stride。这样就编码了本模块广泛使用的标准线性化方式。

### Lines 60-61
```python
60:   def __eq__(self, other):
61:     return self.shape == other.shape and self.stride == other.stride
```
**EN:** `__eq__` defines structural equality through both shape and stride.
**CN:** `__eq__` 通过同时比较 shape 和 stride 来定义结构相等。

### Lines 64-68
```python
64:   def __len__(self):
65:     if is_tuple(self.shape):
66:       return len(self.shape)
67:     else:
68:       return 1
```
**EN:** `__len__` reports layout rank. Scalars are treated as rank-1 for convenience, while tuple shapes expose their actual tuple length.
**CN:** `__len__` 返回布局秩。为便于统一处理，标量 shape 被视为秩 1，而元组 shape 返回其真实长度。

### Lines 71-89
```python
71:   def __call__(self, *args):
72:     """
73:     Map a logical coordinate to a linear index (Coord has no Underscore slice operators)
74:     OR
75:     Slice the layout and return the sublayout (Coord has an Underscore slice op)
76: 
77:     Follow the same behavior of `Layout::operator(Coord const&)` in cute C++
78:     """
79:     if has_none(args):
80:       if len(args) == 1:
81:         return Layout(slice_(args[0], self.shape), slice_(args[0], self.stride))
82:       else:
83:         return Layout(slice_(args, self.shape), slice_(args, self.stride))
84:     else:
85:       if len(args) == 1:
86:         return crd2idx(args[0], self.shape, self.stride)
87:       else:
88:         return crd2idx(args, self.shape, self.stride)
89: 
```
**EN:** `__call__` is the heart of the abstraction. If any coordinate terminal is `None`, it slices shape and stride to build a sublayout; otherwise it maps coordinates to a linear index with `crd2idx`.
**CN:** `__call__` 是整个抽象的核心。如果任一坐标终端为 `None`，它会对 shape 和 stride 做切片并构造子布局；否则就调用 `crd2idx` 把坐标映射为线性索引。

### Lines 91-96
```python
91:   def __getitem__(self, i):
92:     if is_tuple(self.shape):
93:       return Layout(self.shape[i], self.stride[i])
94:     else:
95:       assert i == 0
96:       return Layout(self.shape, self.stride)
```
**EN:** `__getitem__` treats a tuple-shaped layout like a tuple of per-mode layouts. Scalar layouts only accept index `0`.
**CN:** `__getitem__` 把元组 shape 的布局视为按模式拆分的布局集合；标量布局则只允许访问索引 `0`。

### Lines 98-104
```python
 98:   # size(layout)   Size of the domain
 99:   def size(self):
100:     return product(self.shape)
101: 
102:   # cosize(layout)   Size of the codomain
103:   def cosize(self):
104:     return self(self.size() - 1) + 1
```
**EN:** `size` returns the size of the logical domain, while `cosize` computes the covered codomain by evaluating the last logical coordinate and adding one.
**CN:** `size` 返回逻辑定义域大小；`cosize` 则通过计算最后一个逻辑坐标对应的位置再加一，得到被覆盖的值域大小。

### Lines 106-112
```python
106:   # print and str
107:   def __str__(self):
108:     return f"{self.shape}:{self.stride}"
109: 
110:   # error msgs and representation
111:   def __repr__(self):
112:     return f"Layout({self.shape},{self.stride})"
```
**EN:** `__str__` and `__repr__` provide human-readable and debugging-oriented string forms of the layout.
**CN:** `__str__` 和 `__repr__` 分别提供面向阅读和面向调试的布局字符串表示。

### Lines 115-121
```python
115: # Make Layout from a list of layouts (each layout it's own mode in the result)
116: def make_layout(*layouts):
117:   if len(layouts) == 1 and not is_layout(layouts[0]):
118:     layouts = layouts[0]
119: 
120:   shape, stride = zip(*((a.shape,a.stride) for a in layouts))
121:   return Layout(shape, stride)
```
**EN:** `make_layout` constructs a higher-rank layout by packaging multiple layouts as separate modes. It also accepts a single iterable of layouts.
**CN:** `make_layout` 把多个布局打包成不同模式，从而构造更高阶布局；同时也支持传入单个可迭代布局集合。

### Lines 124-133
```python
124: # Size of the domain
125: def size(layout):
126:   if is_layout(layout):
127:     return layout.size()
128:   return product(layout)
129: 
130: 
131: # Size of the codomain
132: def cosize(layout):
133:   return layout.cosize()
```
**EN:** These top-level `size` and `cosize` helpers normalize access so callers can pass either `Layout` objects or raw shape-like tuples.
**CN:** 这些顶层 `size` 与 `cosize` 辅助函数统一了访问方式，使调用方既可以传 `Layout` 对象，也可以传原始 shape 样式的元组。

### Lines 136-164
```python
136: # Layout coalesce -- flatten and combine as many modes as possible while preserving the int-to-int function
137: def coalesce(layout, profile=None):
138:   if is_tuple(profile):
139:     assert len(layout) >= len(profile)
140:     return make_layout(chain((coalesce(layout[i], profile[i]) for i in range(           0,len(profile))),
141:                              (layout[i]                       for i in range(len(profile),len(layout)))))
142: 
143:   result_shape  = [1]
144:   result_stride = [0]
145:   for (shape,stride) in zip(flatten(layout.shape),flatten(layout.stride)):
146:     # skip their shape-1s
147:     if shape == 1:
148:       continue
149:     # replace our shape-1 with anything
150:     elif result_shape[-1] == 1:
151:       result_shape[-1]  = shape
152:       result_stride[-1] = stride
153:     # merge modes if the shape*stride match
154:     elif result_shape[-1] * result_stride[-1] == stride:
155:       result_shape[-1] = result_shape[-1] * shape
156:     # append a new mode
157:     else:
158:       result_shape.append(shape)
159:       result_stride.append(stride)
160: 
161:   if len(result_shape) == 1:
162:     return Layout(result_shape[0], result_stride[0])
163:   else:
164:     return Layout(tuple(result_shape), tuple(result_stride))
```
**EN:** `coalesce` flattens and merges adjacent modes whenever the shape/stride relationship preserves the same integer-to-integer mapping. Profile-driven recursion lets callers coalesce only selected substructures.
**CN:** `coalesce` 会在不改变整数到整数映射的前提下，展平并合并相邻模式。借助 profile 递归参数，调用者还能只对选定子结构执行合并。

### Lines 167-185
```python
167: # Layout filter -- replace all stride-0 modes with size-1 and then coalesce to remove them
168: def filter(layout, profile=None):
169:   if is_tuple(profile):
170:     assert len(layout) >= len(profile)
171:     return make_layout(chain((filter(layout[i], profile[i]) for i in range(           0,len(profile))),
172:                              (layout[i]                     for i in range(len(profile),len(layout)))))
173: 
174:   result_shape  = []
175:   result_stride = []
176:   for (shape,stride) in zip(flatten(layout.shape),flatten(layout.stride)):
177:     # skip their shape-1s and stride-0s
178:     if not (shape == 1 or stride == 0):
179:       result_shape.append(shape)
180:       result_stride.append(stride)
181: 
182:   if len(result_shape) == 0:
183:     return Layout(1,0)
184:   else:
185:     return coalesce(Layout(tuple(result_shape), tuple(result_stride)))
```
**EN:** `filter` removes degenerate modes by discarding shape-1 and stride-0 components, then calling `coalesce` to normalize the result.
**CN:** `filter` 通过移除 shape 为 1 或 stride 为 0 的退化模式来精简布局，随后调用 `coalesce` 进一步规范化结果。

### Lines 188-228
```python
188: # Layout composition
189: # Use tuples-of-layouts to perform this operation by-mode and None as no-op
190: def composition(layoutA, layoutB):
191:   if layoutB is None:
192:     return layoutA
193:   elif is_int(layoutB):
194:     return composition(layoutA, Layout(layoutB))
195:   elif is_tuple(layoutB):
196:     assert len(layoutA) >= len(layoutB)
197:     return make_layout(chain((composition(layoutA[i], layoutB[i]) for i in range(           0,len(layoutB))),
198:                              (layoutA[i]                          for i in range(len(layoutB),len(layoutA)))))
199:   elif is_tuple(layoutB.shape):
200:     return make_layout(composition(layoutA, layoutB_i) for layoutB_i in layoutB)
201: 
202:   if layoutB.stride == 0:
203:     return Layout(layoutB.shape, 0)
204:   else:
205:     result_shape  = []
206:     result_stride = []
207:     rest_shape    = layoutB.shape
208:     rest_stride   = layoutB.stride
209:     flat_A = coalesce(layoutA)
210:     for (curr_shape, curr_stride) in zip(flatten(flat_A.shape)[:-1], flatten(flat_A.stride)[:-1]):
211:       assert curr_shape % rest_stride == 0 or rest_stride % curr_shape == 0
212:       new_shape = min(max(1, curr_shape // rest_stride), rest_shape)
213: 
214:       if new_shape != 1:
215:         result_shape.append(new_shape)
216:         result_stride.append(rest_stride * curr_stride)
217: 
218:       rest_shape  = rest_shape // new_shape
219:       rest_stride = -(-rest_stride // curr_shape)  # Python exclusive impl: "//" is always floor div so == ceil_div(abs(rest_stride), curr_shape) * signum(rest_stride)
220: 
221:     if rest_shape != 1 or len(result_shape) == 0:
222:       result_shape.append(rest_shape)
223:       result_stride.append(rest_stride * flatten(flat_A.stride)[-1])
224: 
225:     if len(result_shape) == 1:
226:       return Layout(result_shape[0], result_stride[0])
227:     else:
228:       return Layout(tuple(result_shape), tuple(result_stride))
```
**EN:** `composition` composes two layouts so that one logical mapping feeds the other. The function handles `None`, integers, tuples of layouts, nested layout shapes, and the general scalar-layout case.
**CN:** `composition` 用于组合两个布局，使一个逻辑映射的输出作为另一个映射的输入。它同时处理 `None`、整数、布局元组、嵌套布局 shape，以及一般标量布局的情况。

### Lines 231-256
```python
231: # Layout complement
232: def complement(layout, max_idx=1):
233:   if is_int(layout):
234:     return complement(Layout(layout))
235: 
236:   result_shape  = []
237:   result_stride = []
238:   current_idx = 1
239: 
240:   sorted_DS = sorted(zip(flatten(layout.stride), flatten(layout.shape)))
241:   for (stride, shape) in sorted_DS:
242:     if stride == 0 or shape == 1:
243:       continue
244: 
245:     in_bound = current_idx <= shape * stride
246:     # To support symbolic value which can't be evaluated now
247:     assert (type(in_bound) is not bool) or in_bound
248: 
249:     result_shape.append(stride // current_idx)
250:     result_stride.append(current_idx)
251:     current_idx = shape * stride
252: 
253:   result_shape.append((max_idx + current_idx - 1) // current_idx)  # ceil_div
254:   result_stride.append(current_idx)
255: 
256:   return coalesce(Layout(tuple(result_shape), tuple(result_stride)))
```
**EN:** `complement` builds the missing layout needed to cover the remaining index space up to `max_idx`. It sorts modes by stride, skips trivial cases, and emits a normalized complement layout.
**CN:** `complement` 构造一个补布局，用来覆盖直到 `max_idx` 为止尚未覆盖的索引空间。实现上会按 stride 排序、跳过平凡模式，并输出规范化后的补布局。

### Lines 259-283
```python
259: # Layout right inverse
260: def right_inverse(layout):
261:   if layout is None:
262:     return None
263:   elif is_int(layout):
264:     return Layout(layout)
265: 
266:   result_shape  = []
267:   result_stride = []
268:   current_idx = 1
269: 
270:   flat_shape  = flatten(layout.shape)
271:   flat_stride = flatten(layout.stride)
272:   sorted_DSA = sorted(zip(flat_stride, flat_shape, prefix_product(flat_shape)))
273:   for (stride,shape,rstride) in sorted_DSA:
274:     if shape == 1:
275:       continue
276:     if current_idx != stride:
277:       break
278: 
279:     result_shape.append(shape)
280:     result_stride.append(rstride)
281:     current_idx = shape * stride
282: 
283:   return coalesce(Layout(tuple(result_shape), tuple(result_stride)))
```
**EN:** `right_inverse` computes a right inverse for layouts that begin with a contiguous prefix in stride order. It collects compatible modes and then normalizes them with `coalesce`.
**CN:** `right_inverse` 为那些按 stride 顺序具有连续前缀的布局构造右逆。它会收集满足条件的模式，再通过 `coalesce` 做规范化。

### Lines 286-292
```python
286: # Layout left inverse
287: def left_inverse(layout):
288:   if layout is None:
289:     return None
290:   elif is_int(layout):
291:     return Layout(layout)
292:   return right_inverse(make_layout(layout, complement(layout)))
```
**EN:** `left_inverse` derives the left inverse by building a combined layout from the original layout and its complement, then taking the right inverse of that composition.
**CN:** `left_inverse` 先把原布局与其补布局组合起来，再对组合结果求右逆，从而得到左逆。

### Lines 295-307
```python
295: # Split a layout by the composition of B and the "rest"
296: # Use tuples-of-layouts to perform this operation by-mode and None as no-op
297: def logical_divide(layoutA, layoutB):
298:   if layoutB is None:
299:     return layoutA
300:   elif is_int(layoutB):
301:     return logical_divide(layoutA, Layout(layoutB))
302:   elif is_tuple(layoutB):
303:     assert len(layoutA) >= len(layoutB)
304:     return make_layout(chain((logical_divide(layoutA[i], layoutB[i]) for i in range(           0,len(layoutB))),
305:                              (layoutA[i]                             for i in range(len(layoutB),len(layoutA)))))
306: 
307:   return composition(layoutA, make_layout(layoutB, complement(layoutB, size(layoutA))))
```
**EN:** `logical_divide` splits `layoutA` according to `layoutB` and the residual complement. Tuple inputs are handled mode-by-mode so hierarchical shapes can be divided recursively.
**CN:** `logical_divide` 按照 `layoutB` 及其剩余补空间对 `layoutA` 进行划分。若输入是元组结构，则按模式递归处理层次化 shape。

### Lines 310-322
```python
310: # Reproduce a layoutA over a layoutB
311: # Use tuples-of-layouts to perform this operation by-mode and None as no-op
312: def logical_product(layoutA, layoutB):
313:   if layoutB is None:
314:     return layoutA
315:   elif is_int(layoutB):
316:     return logical_divide(layoutA, Layout(layoutB))
317:   elif is_tuple(layoutB):
318:     assert len(layoutA) >= len(layoutB)
319:     return make_layout(chain((logical_product(layoutA[i], layoutB[i]) for i in range(           0,len(layoutB))),
320:                              (layoutA[i]                              for i in range(len(layoutB),len(layoutA)))))
321: 
322:   return make_layout(layoutA, composition(complement(layoutA, size(layoutA)*cosize(layoutB)), layoutB));
```
**EN:** `logical_product` reproduces `layoutA` over `layoutB`, effectively building a tiled product layout. As with divide, tuple structure is preserved recursively.
**CN:** `logical_product` 在 `layoutB` 上复制 `layoutA`，本质上构造一种分块乘积布局。与 divide 类似，它也会递归保留元组层次结构。

### Lines 325-339
```python
325: # Gather the modes from a hierarchical logical_divide or logical_product
326: def hier_unzip(splitter, layoutA, layoutB):
327:   if layoutB is None:
328:     return make_layout(Layout(1,0), layoutA)
329:   elif is_tuple(layoutB):
330:     assert len(layoutA) >= len(layoutB)
331:     # A layout with shape ((A,a),(B,b),(C,c))
332:     split = make_layout(hier_unzip(splitter, layoutA[i], layoutB[i]) for i in range(0,len(layoutB)))
333:     # Gather to shape ((A,B,C,...),(a,b,c,...,y,z))
334:     return make_layout(make_layout(       split[i][0] for i in range(           0,len(layoutB))),
335:                        make_layout(chain((split[i][1] for i in range(           0,len(layoutB))),
336:                                          (layoutA[i]  for i in range(len(layoutB),len(layoutA))))))
337: 
338:   # splitter must return a rank-2 layout
339:   return splitter(layoutA, layoutB)
```
**EN:** `hier_unzip` is a higher-order helper that applies a splitter hierarchically, then regroups the resulting mode pairs into a two-mode layout structure.
**CN:** `hier_unzip` 是一个高阶辅助函数：它先层次化地应用拆分器，再把得到的模式对重新汇总成一个两模式布局结构。

### Lines 342-350
```python
342: # Apply logical divide hierarchically and gather the split modes into two modes
343: def zipped_divide(layoutA, layoutB):
344:   return hier_unzip(logical_divide, layoutA, layoutB)
345: 
346: 
347: # Perform logical divide hierarchically and gather tiles (B-layouts) into a new mode
348: def tiled_divide(layoutA, layoutB):
349:   result = zipped_divide(layoutA, layoutB)
350:   return make_layout([result[0]] + [result[1][i] for i in range(len(result[1]))])
```
**EN:** `zipped_divide` and `tiled_divide` are convenience wrappers around hierarchical divide. One keeps the two regrouped modes, while the other exposes tiles as a fresh outer mode.
**CN:** `zipped_divide` 与 `tiled_divide` 是层次化 divide 的便捷封装：前者保留重组后的两大模式，后者把 tile 暴露成新的外层模式。

### Lines 353-361
```python
353: # Apply logical product hierarchically and gather the split modes into two modes
354: def zipped_product(layoutA, layoutB):
355:   return hier_unzip(logical_product, layoutA, layoutB)
356: 
357: 
358: # Perform logical product hierarchically and gather tiles (B-layouts) into a new mode
359: def tiled_product(layoutA, layoutB):
360:   result = zipped_product(layoutA, layoutB)
361:   return make_layout([result[0]] + [result[1][i] for i in range(len(result[1]))])
```
**EN:** `zipped_product` and `tiled_product` mirror the previous pair but for logical products instead of divides.
**CN:** `zipped_product` 与 `tiled_product` 与前一组函数一一对应，只是把操作从 divide 换成了 logical product。

### Lines 364-367
```python
364: def slice_and_offset(crd: tuple,
365:                      layout: Layout):
366:   return (Layout(slice_(crd, layout.shape), slice_(crd, layout.stride)),
367:           crd2idx(crd, layout.shape, layout.stride))
```
**EN:** `slice_and_offset` returns both the sliced sublayout and the linear offset of the slice origin. This is useful when slicing a layout but still needing the base address displacement.
**CN:** `slice_and_offset` 同时返回切出的子布局以及切片起点对应的线性偏移量。这个函数适合在切片布局的同时保留基地址位移信息。

## Key Concepts / 关键概念

- **EN:** Shape + stride as a layout representation  
  **CN:** 用 shape + stride 表示布局
- **EN:** Indexing vs slicing through `Layout.__call__`  
  **CN:** 通过 `Layout.__call__` 统一索引与切片
- **EN:** Layout algebra: composition, complement, and inverses  
  **CN:** 布局代数：组合、补布局与逆
- **EN:** Hierarchical divide/product operations for tiling  
  **CN:** 面向分块的层次化 divide/product 操作

## Dependencies / 依赖关系

- **EN:** Uses `chain` from `itertools` and the recursive helpers imported from `.int_tuple`.  
  **CN:** 使用 `itertools.chain` 以及从 `.int_tuple` 导入的递归辅助函数。
- **EN:** Provides the `LayoutBase` API used by `swizzle.py` for composed layouts.  
  **CN:** 提供 `LayoutBase` API，供 `swizzle.py` 中的组合布局复用。
- **EN:** Relies on helpers such as `prefix_product`, `flatten`, `product`, `crd2idx`, `slice_`, and `has_none`.  
  **CN:** 依赖 `prefix_product`、`flatten`、`product`、`crd2idx`、`slice_`、`has_none` 等辅助函数。
