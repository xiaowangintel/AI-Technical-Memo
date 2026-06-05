# heuristics_provider.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/cutlass_library/heuristics_provider.py`
- **EN:** Wraps matmul heuristics loading and query logic so the generator can reuse offline tuning results.
- **CN:** 封装矩阵乘法启发式规则的加载与查询逻辑，使生成器可以复用离线调优结果。

## Line-by-Line Analysis / 逐行分析

### L1-L32 — Header comments

```python
  1: #################################################################################################
  2: #
  3: # Copyright (c) 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
```
**EN:** Contains the license banner and file-level header comments.

**CN:** 包含许可证横幅以及文件级头部注释。

### L33-L51 — Imports and setup

```python
 33: """
 34: Providers for kernel selection heuristics
 35: """
 36: 
 37: import sys
 38: import os
 39: import glob
 40: import logging
 41: import ctypes
 42: import functools
 43: 
 44: 
 45: try:
 46:   import builtins
 47:   if hasattr(builtins, "CUTLASS_IGNORE_PACKAGE") and CUTLASS_IGNORE_PACKAGE == True:
 48:     raise ImportError("Disabling attempt to import cutlass_library")
 49:   from cutlass_library.library import DataType, LayoutType
 50: except ImportError:
 51:   from library import DataType, LayoutType
```
**EN:** Imports and/or re-exports modules such as sys, os, glob, logging, ctypes, functools so later code can reuse shared definitions.

**CN:** 导入或重新导出诸如 sys, os, glob, logging, ctypes, functools 等模块，使后续代码可以复用共享定义。

### L53-L54 — Class `MatmulHeuristics`

```python
 53: class MatmulHeuristics:
 54: 
```
**EN:** Introduces class `MatmulHeuristics`, which packages the matmul heuristics logic into a reusable type.

**CN:** 引入类 `MatmulHeuristics`，将 matmul heuristics 相关逻辑封装为可复用类型。

### L55-L71 — Function `__init__`

```python
 55:   def __init__(self, gpu = None):
 56:     import nvMatmulHeuristics
 57:     self.mmh_lib = nvMatmulHeuristics
 58:     self.gpu = gpu
 59: 
 60:     if 'CUTLASS_NVMMH_SO_PATH' in os.environ:
 61:       nvmmhInterfaceEx = functools.partial(self.mmh_lib.NvMatmulHeuristicsInterfaceEx, path=os.environ['CUTLASS_NVMMH_SO_PATH'])
 62:     else:
 63:       nvmmhInterfaceEx = self.mmh_lib.NvMatmulHeuristicsInterfaceEx
 64: 
 65:     self.lh = nvmmhInterfaceEx(
 66:       backend=self.mmh_lib.NvMatmulHeuristicsTarget["CUTLASS3"],
 67:       flags=self.mmh_lib.NvMatmulHeuristicsFlags.PERF_MODEL_BASED_AUTO_TUNING,
 68:       load_discovery_implicitly=True,
 69:       gpu=self.mmh_lib.NvMatmulHeuristicsNvidiaGpu[self.gpu] if self.gpu else None
 70:     )
 71:     self.backend = self.lh.createBackend(self.mmh_lib.NvMatmulHeuristicsTarget["CUTLASS3"])
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include nvmmhInterfaceEx, createBackend, partial.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 nvmmhInterfaceEx, createBackend, partial。

### L73-L78 — Function `_layout_from_cutlass`

```python
 73:   def _layout_from_cutlass(self, layouts):
 74:     assert(len(layouts)==3)
 75:     full_layout_str = ''.join('t' if l == LayoutType.RowMajor else 'n' for l in layouts)
 76:     input_layouts = full_layout_str[:2].upper() 
 77:     lh_layout = input_layouts + '_' + str("ROW_MAJOR" if full_layout_str[-1]=='t' else "COL_MAJOR")
 78:     return self.mmh_lib.NvMatmulHeuristicsMatmulLayout[lh_layout]
```
**EN:** Defines `_layout_from_cutlass()`, which implements the layout from cutlass logic. Key helper calls include join, upper, len, str.

**CN:** 定义 `_layout_from_cutlass()`，用于实现 layout from cutlass 相关逻辑。 其中会调用的重要辅助函数包括 join, upper, len, str。

### L80-L99 — Function `_precision_from_cutlass_dtypes`

```python
 80:   def _precision_from_cutlass_dtypes(self, dtypes):
 81:     dtype_to_cublas = {
 82:       DataType.f64: 'D',
 83:       DataType.f32: 'S',
 84:       DataType.f16: 'H',
 85:       DataType.bf16: 'T',
 86:       DataType.e4m3: 'Q',
 87:       DataType.e5m2: 'R',
 88:       DataType.s32: 'I',
 89:       DataType.s8: 'B',
 90:     }
 91: 
 92:     dtype_a, dtype_b, dtype_compute, dtype_c, dtype_d = dtypes
 93: 
 94:     a_c = dtype_to_cublas[dtype_a]
 95: 
 96:     if a_c.lower() != 'q':
 97:       return a_c + dtype_to_cublas[dtype_compute] + dtype_to_cublas[dtype_d]
 98:     else:
 99:       return a_c + dtype_to_cublas[dtype_b] + dtype_to_cublas[dtype_c] + dtype_to_cublas[dtype_compute] + dtype_to_cublas[dtype_d]
```
**EN:** Defines `_precision_from_cutlass_dtypes()`, which implements the precision from cutlass dtypes logic. Key helper calls include lower.

**CN:** 定义 `_precision_from_cutlass_dtypes()`，用于实现 precision from cutlass dtypes 相关逻辑。 其中会调用的重要辅助函数包括 lower。

### L101-L108 — Function `set_cta_div_n`

```python
101:   def set_cta_div_n(self, div_n):
102:     cta_n_div_requirement = ctypes.c_int(div_n) 
103:     self.lh.setBackendValueProperty(
104:       self.backend,
105:       self.mmh_lib.NvMatmulHeuristicsBackendProperty.CTA_TILE_N_DIV_REQUIREMENT,
106:       ctypes.byref(cta_n_div_requirement),
107:       ctypes.sizeof(cta_n_div_requirement)
108:     )
```
**EN:** Defines `set_cta_div_n()`, which implements the set cta div n logic. Key helper calls include c_int, setBackendValueProperty, byref, sizeof.

**CN:** 定义 `set_cta_div_n()`，用于实现 set cta div n 相关逻辑。 其中会调用的重要辅助函数包括 c_int, setBackendValueProperty, byref, sizeof。

### L110-L117 — Function `set_cta_div_m`

```python
110:   def set_cta_div_m(self, div_m):
111:     cta_m_div_requirement = ctypes.c_int(div_m) 
112:     self.lh.setBackendValueProperty(
113:       self.backend,
114:       self.mmh_lib.NvMatmulHeuristicsBackendProperty.CTA_TILE_M_DIV_REQUIREMENT,
115:       ctypes.byref(cta_m_div_requirement),
116:       ctypes.sizeof(cta_m_div_requirement)
117:     )
```
**EN:** Defines `set_cta_div_m()`, which implements the set cta div m logic. Key helper calls include c_int, setBackendValueProperty, byref, sizeof.

**CN:** 定义 `set_cta_div_m()`，用于实现 set cta div m 相关逻辑。 其中会调用的重要辅助函数包括 c_int, setBackendValueProperty, byref, sizeof。

### L119-L174 — Function `get_configs`

```python
119:   def get_configs(self, m, n, k, batch_count, dtypes, layouts, align_a, align_b, voidC=False, use_fast_acc=True, count=1):
120:     if use_fast_acc:
121:       disable_fast_acc_for_fp8 = ctypes.c_int(0)
122:     else:   
123:       disable_fast_acc_for_fp8 = ctypes.c_int(1)
124:     self.lh.setBackendValueProperty(
125:       self.backend,
126:       self.mmh_lib.NvMatmulHeuristicsBackendProperty.DISABLE_FAST_ACC_FOR_FP8,
127:       ctypes.byref(disable_fast_acc_for_fp8),
128:       ctypes.sizeof(disable_fast_acc_for_fp8)
129:     )
130: 
131:     precision = self._precision_from_cutlass_dtypes(dtypes)
132:     layout = self._layout_from_cutlass(layouts)
133: 
134:     matmul_problem = self.lh.makeNvMatmulHeuristicsProblem(m, n, k, layout, batch_count)
135:     configs = self.lh.getEx(matmul_problem, count, self.backend, precision=precision)
136: 
137:     ret = []
138:     for c in configs:
139:       kernel = c['kernel']
140:       problem = c['problem']
141: 
142:       r = {}
143:       r['estimated_runtime'] = c['runtime']
144:       r['cta_tile_m'] = kernel.cta_tile_m
145:       r['cta_tile_n'] = kernel.cta_tile_n
146:       r['cta_tile_k'] = kernel.cta_tile_k
147:       r['instr_tile_m'] = kernel.instr_tile_m
148:       r['instr_tile_n'] = kernel.instr_tile_n
149:       r['instr_tile_k'] = kernel.instr_tile_k
150:       r['warp_tile_m'] = kernel.warp_tile_m
151:       r['warp_tile_n'] = kernel.warp_tile_n
152:       r['warp_tile_k'] = kernel.warp_tile_k
153:       r['cluster_m'] = kernel.cluster_m
154:       r['cluster_n'] = kernel.cluster_n
155:       r['cluster_k'] = 1
156:       r['layout_a'] = layouts[0]
157:       r['layout_b'] = layouts[1]
158:       r['layout_d'] = layouts[2]
159:       r['dtype_a'] = dtypes[0]
160:       r['dtype_b'] = dtypes[1]
161:       r['dtype_acc'] = dtypes[2]
162:       r['dtype_c'] = dtypes[3]
163:       r['dtype_d'] = dtypes[4]
164:       r['alignment_a'] = align_a
165:       r['alignment_b'] = align_b
166:       r['swizzle_size'] = kernel.swizzle_factor
167:       r['raster_order'] = 'along_m' if kernel.cta_order==0 else 'along_n'
168:       r['split_k_slices'] = kernel.split_k
169:       r['use_fast_acc'] = use_fast_acc
170:       r['voidC'] = voidC
171: 
172:       ret.append(r)
173: 
174:     return ret
```
**EN:** Defines `get_configs()`, which returns or derives configs. Key helper calls include setBackendValueProperty, _precision_from_cutlass_dtypes, _layout_from_cutlass, makeNvMatmulHeuristicsProblem, getEx, c_int.

**CN:** 定义 `get_configs()`，用于返回或推导 configs。 其中会调用的重要辅助函数包括 setBackendValueProperty, _precision_from_cutlass_dtypes, _layout_from_cutlass, makeNvMatmulHeuristicsProblem, getEx, c_int。

## Key Concepts / 关键概念

- **EN:** File role: Wraps matmul heuristics loading and query logic so the generator can reuse offline tuning results.
- **CN:** 文件角色：封装矩阵乘法启发式规则的加载与查询逻辑，使生成器可以复用离线调优结果。
- **EN:** Main classes: `MatmulHeuristics`
- **CN:** 主要类：`MatmulHeuristics`

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: `sys, os, glob, logging, ctypes, functools, builtins`
- **CN:** 标准库依赖：`sys, os, glob, logging, ctypes, functools, builtins`
- **EN:** Internal CUTLASS dependencies: `cutlass_library.library, library`
- **CN:** CUTLASS 内部依赖：`cutlass_library.library, library`
- **EN:** External/non-stdlib dependencies: `nvMatmulHeuristics`
- **CN:** 外部/非标准库依赖：`nvMatmulHeuristics`
