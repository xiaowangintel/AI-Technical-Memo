# sm90_utils.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/cutlass_library/sm90_utils.py`
- **EN:** Provides helper routines that derive SM90 math instructions, clusters, tiles, schedules, and pruning behavior.
- **CN:** 提供推导 SM90 数学指令、集群、tile、调度与裁剪行为的辅助例程。

## Line-by-Line Analysis / 逐行分析

### L1-L32 — Header comments

```python
  1: #################################################################################################
  2: #
  3: # Copyright (c) 2024 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### L33-L54 — Imports and setup

```python
 33: """
 34: Utilities for enumerating CUTLASS library SM90 kernels
 35: """
 36: 
 37: import argparse
 38: import enum
 39: from itertools import product
 40: import math
 41: import logging
 42: import os.path
 43: import shutil
 44: import sys
 45: import copy
 46: from typing import Any, Optional, Sequence, Tuple, List
 47: 
 48: try:
 49:   import builtins
 50:   if hasattr(builtins, "CUTLASS_IGNORE_PACKAGE") and CUTLASS_IGNORE_PACKAGE == True:
 51:     raise ImportError("Disabling attempt to import cutlass_library")
 52:   from cutlass_library.library import *
 53: except ImportError:
 54:   from library import *
```
**EN:** Imports and/or re-exports modules such as argparse, enum, itertools, product, math, logging, os.path, shutil, ... so later code can reuse shared definitions.

**CN:** 导入或重新导出诸如 argparse, enum, itertools, product, math, logging, os.path, shutil, ... 等模块，使后续代码可以复用共享定义。

### L55-L56 — Comments

```python
 55: 
 56: # NOTE: this is a duplicate of CudaToolkitVersionSatisfies in generator.py
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L57-L69 — Function `CudaToolkitVersionSatisfies`

```python
 57: def CudaToolkitVersionSatisfies(semantic_ver_string, major, minor, patch = 0):
 58: 
 59:   # by default, use the latest CUDA Toolkit version
 60:   cuda_version = [11, 0, 132]
 61: 
 62:   # Update cuda_version based on parsed string
 63:   if semantic_ver_string != '':
 64:     for i, x in enumerate([int(x) for x in semantic_ver_string.split('.')[:3]]):
 65:       if i < len(cuda_version):
 66:         cuda_version[i] = x
 67:       else:
 68:         cuda_version.append(x)
 69:   return cuda_version >= [major, minor, patch]
```
**EN:** Defines `CudaToolkitVersionSatisfies()`, which implements the cuda toolkit version satisfies logic. Key helper calls include enumerate, int, len, append, split.

**CN:** 定义 `CudaToolkitVersionSatisfies()`，用于实现 cuda toolkit version satisfies 相关逻辑。 其中会调用的重要辅助函数包括 enumerate, int, len, append, split。

### L70-L89 — Comments

```python
 70: 
 71: #### Step 0: define levels
 72: 
 73: # One integer level controls multiple "generators" and how many
 74: # combinations they generate. That is the "global" level.
 75: # "Generators" are WGMMA shapes, MMA multipliers, cluster sizes, and
 76: # anything that is eventually involved in the Cartesian product
 77: # which yields our kernel configurations.
 78: # For simplicity, each generator defines their own levels, 
 79: # starting from 0. As a rule we assume 10 or fewer levels, making
 80: # their level a digit.
 81: # The "global" level simply stacks these digits and represents them
 82: # as a single integer.
 83: # 
 84: # For example, level 500 indicates cluster sizes are at level 5, MMA
 85: # multipliers are at level 0, and WGMMA shapes are at level 0 as well.
 86: #
 87: # Here we define the global level to generator level mappings.
 88: 
 89: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L90-L91 — Function `get_wgmma_level_from_global_level`

```python
 90: def get_wgmma_level_from_global_level(global_level: int):
 91:     return global_level % 10
```
**EN:** Defines `get_wgmma_level_from_global_level()`, which returns or derives wgmma level from global level.

**CN:** 定义 `get_wgmma_level_from_global_level()`，用于返回或推导 wgmma level from global level。

### L94-L95 — Function `get_mma_level_from_global_level`

```python
 94: def get_mma_level_from_global_level(global_level: int):
 95:     return (global_level // 10) % 10
```
**EN:** Defines `get_mma_level_from_global_level()`, which returns or derives mma level from global level.

**CN:** 定义 `get_mma_level_from_global_level()`，用于返回或推导 mma level from global level。

### L98-L99 — Function `get_cluster_level_from_global_level`

```python
 98: def get_cluster_level_from_global_level(global_level: int):
 99:     return (global_level // 100) % 10
```
**EN:** Defines `get_cluster_level_from_global_level()`, which returns or derives cluster level from global level.

**CN:** 定义 `get_cluster_level_from_global_level()`，用于返回或推导 cluster level from global level。

### L102-L103 — Function `get_pruning_level_from_global_level`

```python
102: def get_pruning_level_from_global_level(global_level: int):
103:     return (global_level // 1000) % 10
```
**EN:** Defines `get_pruning_level_from_global_level()`, which returns or derives pruning level from global level.

**CN:** 定义 `get_pruning_level_from_global_level()`，用于返回或推导 pruning level from global level。

### L104-L107 — Comments

```python
104: 
105: 
106: #### Step 1: generate MMA instruction shapes based on levels
107: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L108-L125 — Import fallback

```python
108: try:
109:     from .sm90_shapes import (
110:         SM90_MMA_MULTIPLIERS,
111:         SM90_CLUSTER_SIZES,
112:         SM90_WGMMA_SHAPES_TF32_DENSE,
113:         SM90_WGMMA_SHAPES_FP16_BF16_DENSE,
114:         SM90_WGMMA_SHAPES_FP8_DENSE,
115:         SM90_WGMMA_SHAPES_INT8_DENSE,
116:     )
117: except:
118:     from sm90_shapes import (
119:         SM90_MMA_MULTIPLIERS,
120:         SM90_CLUSTER_SIZES,
121:         SM90_WGMMA_SHAPES_TF32_DENSE,
122:         SM90_WGMMA_SHAPES_FP16_BF16_DENSE,
123:         SM90_WGMMA_SHAPES_FP8_DENSE,
124:         SM90_WGMMA_SHAPES_INT8_DENSE,
125:     )
```
**EN:** Uses `try`/`except` to prefer package imports and fall back to local imports, keeping both installed-package and script execution modes working.

**CN:** 通过 `try`/`except` 优先使用包导入，并在失败时回退到本地导入，从而同时兼容已安装包与脚本运行模式。

### L128-L133 — Function `generate_tf32_math_instruction_shapes_sm90`

```python
128: def generate_tf32_math_instruction_shapes_sm90(level: int):
129:     assert isinstance(level, int) and level >= 0
130:     filtered_list_of_wgmma_shapes = [
131:         wgmma_shape for wgmma_shape, min_level in SM90_WGMMA_SHAPES_TF32_DENSE.items() if level >= min_level
132:     ]
133:     return filtered_list_of_wgmma_shapes
```
**EN:** Defines `generate_tf32_math_instruction_shapes_sm90()`, which generates tf32 math instruction shapes sm90. Key helper calls include isinstance, items.

**CN:** 定义 `generate_tf32_math_instruction_shapes_sm90()`，用于生成 tf32 math instruction shapes sm90。 其中会调用的重要辅助函数包括 isinstance, items。

### L135-L140 — Function `generate_fp16_bf16_math_instruction_shapes_sm90`

```python
135: def generate_fp16_bf16_math_instruction_shapes_sm90(level: int):
136:     assert isinstance(level, int) and level >= 0
137:     filtered_list_of_wgmma_shapes = [
138:         wgmma_shape for wgmma_shape, min_level in SM90_WGMMA_SHAPES_FP16_BF16_DENSE.items() if level >= min_level
139:     ]
140:     return filtered_list_of_wgmma_shapes
```
**EN:** Defines `generate_fp16_bf16_math_instruction_shapes_sm90()`, which generates fp16 bf16 math instruction shapes sm90. Key helper calls include isinstance, items.

**CN:** 定义 `generate_fp16_bf16_math_instruction_shapes_sm90()`，用于生成 fp16 bf16 math instruction shapes sm90。 其中会调用的重要辅助函数包括 isinstance, items。

### L142-L147 — Function `generate_fp8_math_instruction_shapes_sm90`

```python
142: def generate_fp8_math_instruction_shapes_sm90(level: int):
143:     assert isinstance(level, int) and level >= 0
144:     filtered_list_of_wgmma_shapes = [
145:         wgmma_shape for wgmma_shape, min_level in SM90_WGMMA_SHAPES_FP8_DENSE.items() if level >= min_level
146:     ]
147:     return filtered_list_of_wgmma_shapes
```
**EN:** Defines `generate_fp8_math_instruction_shapes_sm90()`, which generates fp8 math instruction shapes sm90. Key helper calls include isinstance, items.

**CN:** 定义 `generate_fp8_math_instruction_shapes_sm90()`，用于生成 fp8 math instruction shapes sm90。 其中会调用的重要辅助函数包括 isinstance, items。

### L149-L154 — Function `generate_int8_math_instruction_shapes_sm90`

```python
149: def generate_int8_math_instruction_shapes_sm90(level: int):
150:     assert isinstance(level, int) and level >= 0
151:     filtered_list_of_wgmma_shapes = [
152:         wgmma_shape for wgmma_shape, min_level in SM90_WGMMA_SHAPES_INT8_DENSE.items() if level >= min_level
153:     ]
154:     return filtered_list_of_wgmma_shapes
```
**EN:** Defines `generate_int8_math_instruction_shapes_sm90()`, which generates int8 math instruction shapes sm90. Key helper calls include isinstance, items.

**CN:** 定义 `generate_int8_math_instruction_shapes_sm90()`，用于生成 int8 math instruction shapes sm90。 其中会调用的重要辅助函数包括 isinstance, items。

### L156-L165 — Function `generate_mixed_dtype_math_instructions_shapes_sm90`

```python
156: def generate_mixed_dtype_math_instructions_shapes_sm90(wgmma_level: int, a_type: DataType, b_type: DataType):
157:     # DataTypeSize are in the unit of bits
158:     a_bytes = DataTypeSize[a_type] // 8
159:     b_bytes = DataTypeSize[b_type] // 8
160:     if a_bytes == 4 or b_bytes == 4:
161:         return generate_tf32_math_instruction_shapes_sm90(wgmma_level)
162:     elif a_bytes == 2 or b_bytes == 2:
163:         return generate_fp16_bf16_math_instruction_shapes_sm90(wgmma_level)
164:     else:
165:         return generate_fp8_math_instruction_shapes_sm90(wgmma_level)
```
**EN:** Defines `generate_mixed_dtype_math_instructions_shapes_sm90()`, which generates mixed dtype math instructions shapes sm90. Key helper calls include generate_tf32_math_instruction_shapes_sm90, generate_fp16_bf16_math_instruction_shapes_sm90, generate_fp8_math_instruction_shapes_sm90.

**CN:** 定义 `generate_mixed_dtype_math_instructions_shapes_sm90()`，用于生成 mixed dtype math instructions shapes sm90。 其中会调用的重要辅助函数包括 generate_tf32_math_instruction_shapes_sm90, generate_fp16_bf16_math_instruction_shapes_sm90, generate_fp8_math_instruction_shapes_sm90。

### L166-L168 — Comments

```python
166: 
167: ###########
168: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L169-L180 — Function `generate_tf32_math_instructions_sm90`

```python
169: def generate_tf32_math_instructions_sm90(level: int):
170:     wgmma_level = get_wgmma_level_from_global_level(level)
171:     math_instructions = []
172:     for math_instruction_shape in generate_tf32_math_instruction_shapes_sm90(wgmma_level):
173:         math_instructions.append(
174:           MathInstruction(
175:               math_instruction_shape,
176:               DataType.tf32, DataType.tf32, DataType.f32,
177:               OpcodeClass.TensorOp,
178:               MathOperation.multiply_add)
179:         )
180:     return math_instructions
```
**EN:** Defines `generate_tf32_math_instructions_sm90()`, which generates tf32 math instructions sm90. Key helper calls include get_wgmma_level_from_global_level, generate_tf32_math_instruction_shapes_sm90, append, MathInstruction.

**CN:** 定义 `generate_tf32_math_instructions_sm90()`，用于生成 tf32 math instructions sm90。 其中会调用的重要辅助函数包括 get_wgmma_level_from_global_level, generate_tf32_math_instruction_shapes_sm90, append, MathInstruction。

### L182-L203 — Function `generate_fp16_bf16_math_instructions_sm90`

```python
182: def generate_fp16_bf16_math_instructions_sm90(level: int):
183:     wgmma_level = get_wgmma_level_from_global_level(level)
184:     math_instructions = []
185:     for math_instruction_shape in generate_fp16_bf16_math_instruction_shapes_sm90(wgmma_level):
186:         math_instructions += [
187:           MathInstruction(
188:               math_instruction_shape,
189:               DataType.f16, DataType.f16, DataType.f16,
190:               OpcodeClass.TensorOp,
191:               MathOperation.multiply_add),
192:           MathInstruction(
193:               math_instruction_shape,
194:               DataType.f16, DataType.f16, DataType.f32,
195:               OpcodeClass.TensorOp,
196:               MathOperation.multiply_add),
197:           MathInstruction(
198:               math_instruction_shape,
199:               DataType.bf16, DataType.bf16, DataType.f32,
200:               OpcodeClass.TensorOp,
201:               MathOperation.multiply_add),
202:         ]
203:     return math_instructions
```
**EN:** Defines `generate_fp16_bf16_math_instructions_sm90()`, which generates fp16 bf16 math instructions sm90. Key helper calls include get_wgmma_level_from_global_level, generate_fp16_bf16_math_instruction_shapes_sm90, MathInstruction.

**CN:** 定义 `generate_fp16_bf16_math_instructions_sm90()`，用于生成 fp16 bf16 math instructions sm90。 其中会调用的重要辅助函数包括 get_wgmma_level_from_global_level, generate_fp16_bf16_math_instruction_shapes_sm90, MathInstruction。

### L205-L231 — Function `generate_fp8_math_instructions_sm90`

```python
205: def generate_fp8_math_instructions_sm90(level: int):
206:     wgmma_level = get_wgmma_level_from_global_level(level)
207:     math_instructions = []
208:     for math_instruction_shape in generate_fp8_math_instruction_shapes_sm90(wgmma_level):
209:         math_instructions += [
210:           MathInstruction(
211:               math_instruction_shape,
212:               DataType.e4m3, DataType.e4m3, DataType.f32,
213:               OpcodeClass.TensorOp,
214:               MathOperation.multiply_add),
215:           MathInstruction(
216:               math_instruction_shape,
217:               DataType.e4m3, DataType.e5m2, DataType.f32,
218:               OpcodeClass.TensorOp,
219:               MathOperation.multiply_add),
220:           MathInstruction(
221:               math_instruction_shape,
222:               DataType.e5m2, DataType.e4m3, DataType.f32,
223:               OpcodeClass.TensorOp,
224:               MathOperation.multiply_add),
225:           MathInstruction(
226:               math_instruction_shape,
227:               DataType.e5m2, DataType.e5m2, DataType.f32,
228:               OpcodeClass.TensorOp,
229:               MathOperation.multiply_add),
230:         ]
231:     return math_instructions
```
**EN:** Defines `generate_fp8_math_instructions_sm90()`, which generates fp8 math instructions sm90. Key helper calls include get_wgmma_level_from_global_level, generate_fp8_math_instruction_shapes_sm90, MathInstruction.

**CN:** 定义 `generate_fp8_math_instructions_sm90()`，用于生成 fp8 math instructions sm90。 其中会调用的重要辅助函数包括 get_wgmma_level_from_global_level, generate_fp8_math_instruction_shapes_sm90, MathInstruction。

### L233-L247 — Function `generate_mixed_dtype_math_instructions_sm90`

```python
233: def generate_mixed_dtype_math_instructions_sm90(level: int, types_of_a_b_acc: List[Tuple[DataType, DataType, DataType]]):
234:     wgmma_level = get_wgmma_level_from_global_level(level)
235:     math_instructions = []
236:     for a_type, b_type, acc_type in types_of_a_b_acc:
237:         math_instruction_shapes = generate_mixed_dtype_math_instructions_shapes_sm90(wgmma_level, a_type, b_type)
238:         for math_instruction_shape in math_instruction_shapes:
239:             math_instructions += [
240:                 MathInstruction(
241:                     math_instruction_shape,
242:                     a_type, b_type, acc_type,
243:                     OpcodeClass.TensorOp,
244:                     MathOperation.multiply_add
245:                 ),
246:             ]
247:     return math_instructions
```
**EN:** Defines `generate_mixed_dtype_math_instructions_sm90()`, which generates mixed dtype math instructions sm90. Key helper calls include get_wgmma_level_from_global_level, generate_mixed_dtype_math_instructions_shapes_sm90, MathInstruction.

**CN:** 定义 `generate_mixed_dtype_math_instructions_sm90()`，用于生成 mixed dtype math instructions sm90。 其中会调用的重要辅助函数包括 get_wgmma_level_from_global_level, generate_mixed_dtype_math_instructions_shapes_sm90, MathInstruction。

### L249-L265 — Function `generate_int8_math_instructions_sm90`

```python
249: def generate_int8_math_instructions_sm90(level: int):
250:     wgmma_level = get_wgmma_level_from_global_level(level)
251:     math_instructions = []
252:     for math_instruction_shape in generate_int8_math_instruction_shapes_sm90(wgmma_level):
253:         math_instructions += [
254:           MathInstruction(
255:               math_instruction_shape,
256:               DataType.s8, DataType.s8, DataType.s32,
257:               OpcodeClass.TensorOp,
258:               MathOperation.multiply_add),
259:           MathInstruction(
260:               math_instruction_shape,
261:               DataType.u8, DataType.u8, DataType.s32,
262:               OpcodeClass.TensorOp,
263:               MathOperation.multiply_add),
264:         ]
265:     return math_instructions
```
**EN:** Defines `generate_int8_math_instructions_sm90()`, which generates int8 math instructions sm90. Key helper calls include get_wgmma_level_from_global_level, generate_int8_math_instruction_shapes_sm90, MathInstruction.

**CN:** 定义 `generate_int8_math_instructions_sm90()`，用于生成 int8 math instructions sm90。 其中会调用的重要辅助函数包括 get_wgmma_level_from_global_level, generate_int8_math_instruction_shapes_sm90, MathInstruction。

### L267-L276 — Function `make_sparse_math_instructions`

```python
267: def make_sparse_math_instructions(math_instructions):
268:     sparse_instructions = []
269:     for inst in math_instructions:
270:         if inst.opcode_class == OpcodeClass.TensorOp:
271:             sparse_instructions.append(MathInstruction(
272:                 (inst.instruction_shape[0], inst.instruction_shape[1], inst.instruction_shape[2] * 2),
273:                 inst.element_a, inst.element_b, inst.element_accumulator,
274:                 OpcodeClass.SparseTensorOp,
275:                 inst.math_operation),)
276:     return sparse_instructions
```
**EN:** Defines `make_sparse_math_instructions()`, which implements the make sparse math instructions logic. Key helper calls include append, MathInstruction.

**CN:** 定义 `make_sparse_math_instructions()`，用于实现 make sparse math instructions 相关逻辑。 其中会调用的重要辅助函数包括 append, MathInstruction。

### L277-L280 — Comments

```python
277: 
278: 
279: #### Step 2: generate tile descriptions from math instruction shapes
280: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L281-L355 — Function `is_tile_desc_valid`

```python
281: def is_tile_desc_valid(tile_description):
282:     if tile_description.minimum_compute_capability != 90 or tile_description.maximum_compute_capability != 90:
283:         return False
284: 
285:     element_a, element_b, element_accum = (
286:         tile_description.math_instruction.element_a,
287:         tile_description.math_instruction.element_b,
288:         tile_description.math_instruction.element_accumulator
289:     )
290: 
291:     cluster_size, cta_shape = (
292:         tile_description.cluster_shape,
293:         tile_description.threadblock_shape,
294:     )
295:     grid_size = (
296:         cta_shape[0] * cluster_size[0] +
297:         cta_shape[1] * cluster_size[1] +
298:         cta_shape[2] * cluster_size[2]
299:     )
300:     num_ctas_in_cluster = cluster_size[0] * cluster_size[1] * cluster_size[2]
301:     cluster_shape = (
302:         cluster_size[0] * cta_shape[0],
303:         cluster_size[1] * cta_shape[1],
304:         cluster_size[2] * cta_shape[2]
305:     )
306: 
307:     FP32_TYPES = [DataType.f32, DataType.tf32]
308:     FP16_TYPES = [DataType.f16, DataType.bf16]
309:     is_fp32 = element_a in FP32_TYPES and element_b in FP32_TYPES
310:     is_fp16 = element_a in FP16_TYPES and element_b in FP16_TYPES
311: 
312:     # Maximum number of CTAs per cluster is 8 for Hopper, but up to 16 is
313:     # allowed for non portable clusters.
314:     if num_ctas_in_cluster > 16 or num_ctas_in_cluster < 1:
315:         return False
316: 
317:     if grid_size < 1:
318:         return False
319: 
320:     # SM90 WGMMA shapes are always 64 across M, therefore
321:     # CTA shape across M must always be a multiple of 64.
322:     if cta_shape[0] < 64 or cta_shape[0] % 64 != 0:
323:         return False
324: 
325:     # The minimum WGMMA shape across N is 8, and increments
326:     # vary across different dtypes, but they're never smaller
327:     # than 8. The minimum CTA shape allowed across N though is 16.
328:     if cta_shape[1] < 16 or cta_shape[1] % 8 != 0:
329:         return False
330: 
331:     # SM90 WGMMA shapes across K are always 8 for 32 bit dense
332:     # operations, 16 for 16 bit, and 32 for 8 bit. In any case,
333:     # the CTA shape across K should be a multiple of 8 and at least
334:     # twice the WGMMA shape across K.
335:     if cta_shape[2] < 16 or cta_shape[2] % 8 != 0:
336:         return False
337: 
338:     # Minimum of 2 stages (very rough heuristic that may filter out valid kernel configs)
339:     if (cluster_shape[0] >= 128 or cluster_shape[1] >= 128) and cluster_shape[2] >= 256:
340:         return False
341: 
342:     if is_fp32 and (cluster_shape[0] >= 128 or cluster_shape[1] >= 128) and cluster_shape[2] >= 128:
343:         return False
344: 
345:     if is_fp32 and cluster_shape[0] >= 256 and cluster_shape[1] >= 256 and cluster_shape[2] >= 64:
346:         return False
347: 
348:     if is_fp16 and cluster_shape[0] >= 256 and cluster_shape[1] >= 256 and cluster_shape[2] >= 128:
349:         return False
350: 
351:     # CTA shape upper bound: <256, 256, 256>
352:     if cta_shape[0] > 256 or cta_shape[1] > 256 or cta_shape[2] > 256:
353:         return False
354: 
355:     return True
```
**EN:** Defines `is_tile_desc_valid()`, a predicate that checks whether tile desc valid.

**CN:** 定义 `is_tile_desc_valid()`，用于判断是否满足 tile desc valid 这一条件。

### L357-L362 — Function `get_mma_multipliers`

```python
357: def get_mma_multipliers(level: int):
358:     assert isinstance(level, int) and level >= 0
359:     mma_level = get_mma_level_from_global_level(level)
360:     return [
361:         mma_mul for mma_mul, mma_min_level in SM90_MMA_MULTIPLIERS.items() if mma_level >= mma_min_level
362:     ]
```
**EN:** Defines `get_mma_multipliers()`, which returns or derives mma multipliers. Key helper calls include get_mma_level_from_global_level, isinstance, items.

**CN:** 定义 `get_mma_multipliers()`，用于返回或推导 mma multipliers。 其中会调用的重要辅助函数包括 get_mma_level_from_global_level, isinstance, items。

### L364-L371 — Function `get_cluster_sizes`

```python
364: def get_cluster_sizes(level: int, is_aligned: bool):
365:     if not is_aligned:
366:         return [(1, 1, 1)]
367:     assert isinstance(level, int) and level >= 0
368:     cluster_level = get_cluster_level_from_global_level(level)
369:     return [
370:         cluster_size for cluster_size, cluster_min_level in SM90_CLUSTER_SIZES.items() if cluster_level >= cluster_min_level
371:     ]
```
**EN:** Defines `get_cluster_sizes()`, which returns or derives cluster sizes. Key helper calls include get_cluster_level_from_global_level, isinstance, items.

**CN:** 定义 `get_cluster_sizes()`，用于返回或推导 cluster sizes。 其中会调用的重要辅助函数包括 get_cluster_level_from_global_level, isinstance, items。

### L373-L404 — Function `generate_tile_descriptions_sm90`

```python
373: def generate_tile_descriptions_sm90(math_instructions, is_aligned: bool, level: int):
374:     tile_descriptions = set()
375:     mma_multipliers, cluster_sizes = get_mma_multipliers(level), get_cluster_sizes(level, is_aligned)
376:     for math_inst, mma_mul, cluster_size in product(math_instructions, mma_multipliers, cluster_sizes):
377: 
378:         # generator can stamp out duplicate kernels, because it doesn't explicitly set instruction
379:         # shape for SM90 kernels, and the 3.X collective API doesn't directly expose them when using
380:         # the auto kernel schedule.
381: 
382:         math_inst_stub = copy.deepcopy(math_inst)
383:         math_inst_stub.instruction_shape = [0, 0, 0]
384: 
385:         tile_desc = TileDescription(
386:             threadblock_shape=[
387:                 math_inst.instruction_shape[0] * mma_mul[0],
388:                 math_inst.instruction_shape[1] * mma_mul[1],
389:                 math_inst.instruction_shape[2] * mma_mul[2]
390:             ],
391:             stages=0,
392:             warp_count=[4, 1, 1],
393:             math_instruction=math_inst_stub,
394:             min_compute=90,
395:             max_compute=90,
396:             cluster_shape=cluster_size)
397:         # For sparse kernels K-tile is twice as large (due to 2x MMA-K size)
398:         # Reduce it to same size as dense to afford more smem stages
399:         if math_inst.opcode_class == OpcodeClass.SparseTensorOp:
400:             tile_desc.threadblock_shape[2] = tile_desc.threadblock_shape[2] // 2
401:         if is_tile_desc_valid(tile_desc):
402:             tile_descriptions.add(tile_desc)
403: 
404:     return tile_descriptions
```
**EN:** Defines `generate_tile_descriptions_sm90()`, which generates tile descriptions sm90. Key helper calls include set, product, get_mma_multipliers, get_cluster_sizes, deepcopy, TileDescription.

**CN:** 定义 `generate_tile_descriptions_sm90()`，用于生成 tile descriptions sm90。 其中会调用的重要辅助函数包括 set, product, get_mma_multipliers, get_cluster_sizes, deepcopy, TileDescription。

### L405-L407 — Comments

```python
405: 
406: #### Step 3: map tile description to valid schedules
407: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L408-L410 — Function `is_tile_desc_compatible_with_cooperative`

```python
408: def is_tile_desc_compatible_with_cooperative(tile_description):
409:     # Cooperative kernels require a minimum CTA-M of 128
410:     return tile_description.threadblock_shape[0] % 128 == 0
```
**EN:** Defines `is_tile_desc_compatible_with_cooperative()`, a predicate that checks whether tile desc compatible with cooperative.

**CN:** 定义 `is_tile_desc_compatible_with_cooperative()`，用于判断是否满足 tile desc compatible with cooperative 这一条件。

### L413-L440 — Function `can_tile_desc_use_shmem_in_epilogue`

```python
413: def can_tile_desc_use_shmem_in_epilogue(tile_description, data_types):
414:     dtype_a, dtype_b, dtype_c, dtype_d, dtype_acc, dtype_epi = (
415:         data_types["a_type"],
416:         data_types["b_type"],
417:         data_types["c_type"],
418:         data_types["d_type"],
419:         data_types["acc_type"],
420:         data_types["epi_type"]
421:     )
422:     mn = tile_description.threadblock_shape[0] * tile_description.threadblock_shape[1]
423:     bitsize_c, bitsize_d = DataTypeSize[dtype_c], DataTypeSize[dtype_d]
424: 
425:     shmem_bits_c, shmem_bits_d = bitsize_c * mn, bitsize_d * mn
426:     shmem_bits_total = shmem_bits_c + shmem_bits_d
427:     # Magic number: 2^20
428:     # Existing logic suggested that tile shape 256x128 (or 128x256)
429:     # would run out of shmem if D is FP32, and source is needed.
430:     # That would be 256 * 128 * 32 == 2^21 (~262 KB), which is over the limit.
431:     # Hopper's max shmem size is 228 KB, and 2^20 ~= 131 KB.
432:     # Since epilogue can't possibly use ALL of the shmem available
433:     # we can just settle on 2^20 bits (~ 131 KB) being the upper bound
434:     # we would allow for epilogue.
435:     # This can be different for non-persistent kernels where epilogue and
436:     # mainloop shmem is shared.
437:     if shmem_bits_total > 2 ** 20:
438:         return False
439: 
440:     return True
```
**EN:** Defines `can_tile_desc_use_shmem_in_epilogue()`, which implements the can tile desc use shmem in epilogue logic.

**CN:** 定义 `can_tile_desc_use_shmem_in_epilogue()`，用于实现 can tile desc use shmem in epilogue 相关逻辑。

### L443-L713 — Function `get_valid_schedules`

```python
443: def get_valid_schedules(tile_description, cuda_version, is_aligned, data_types, layout,
444:                         instantiation_level, enable_fp8_fast_acc=True, gemm_kind=GemmKind.Universal3x):
445:     # Level 0: prune according to existing generator.py behavior
446:     # Level >= 1: no pruning
447:     level = get_pruning_level_from_global_level(instantiation_level)
448:     schedules = []
449:     stream_k_schedules = []
450: 
451:     if not is_tile_desc_valid(tile_description):
452:         return schedules, stream_k_schedules
453: 
454:     FP16_TYPES = [DataType.f16, DataType.bf16]
455:     is_fp16 = data_types["a_type"] in FP16_TYPES and data_types["b_type"] in FP16_TYPES
456: 
457:     FP8_TYPES = [DataType.e4m3, DataType.e5m2]
458:     is_fp8 = data_types["a_type"] in FP8_TYPES and data_types["b_type"] in FP8_TYPES
459:     can_do_fp8_fast_accum = is_fp8 and enable_fp8_fast_acc
460: 
461:     FP32_TYPES = [DataType.f32, DataType.tf32]
462:     is_fp32 = data_types["a_type"] in FP32_TYPES and data_types["b_type"] in FP32_TYPES
463:     requires_transposed_epilogue = is_fp32 and layout[0][0] == LayoutType.RowMajor and layout[1][0] == LayoutType.RowMajor
464: 
465:     can_do_cooperative = is_tile_desc_compatible_with_cooperative(tile_description)
466:     can_do_tma_epilogue = is_aligned and not requires_transposed_epilogue and can_tile_desc_use_shmem_in_epilogue(tile_description, data_types)
467: 
468:     default_epilogue = EpilogueScheduleType.NoSmemWarpSpecialized if not requires_transposed_epilogue else EpilogueScheduleType.EpilogueTransposed
469:     auto_epilogue = EpilogueScheduleType.ScheduleAuto if not requires_transposed_epilogue else EpilogueScheduleType.EpilogueTransposed
470: 
471:     cta_m, cta_n, cta_k = (
472:         tile_description.threadblock_shape[0],
473:         tile_description.threadblock_shape[1],
474:         tile_description.threadblock_shape[2]
475:     )
476:     c_type = data_types["c_type"]
477:     d_type = data_types["d_type"]
478:     is_void_c = c_type == DataType.void
479: 
480:     # Filter out invalid kernels
481:     is_nt = layout[0][0] == LayoutType.ColumnMajor and layout[1][0] == LayoutType.RowMajor
482:     is_tn = layout[0][0] == LayoutType.RowMajor and layout[1][0] == LayoutType.ColumnMajor
483:     is_nn = layout[0][0] == LayoutType.ColumnMajor and layout[1][0] == LayoutType.ColumnMajor
484: 
485:     # static_assert(size<0>(SmemLayoutB{}) % WarpgroupTileSize == 0,
486:     #   "Copy size must evenly divide SMEM tile.");
487:     if is_fp32 and is_nt and (cta_n % cta_k != 0):
488:         return [], []
489: 
490:     # static_assert(!TransposeB || (cutlass::bits_to_bytes((size<1>(SmemLayoutB{}) * sizeof_bits<InternalElementB>::value))) == 128,
491:     # "SmemLayoutB K must be 128bytes to be transposed.")
492:     if is_fp32 and is_nt and cta_k != 32:
493:         return [], []
494: 
495:     # Static assert failure when instantiating SmemLayoutB
496:     if is_fp32 and (is_tn or is_nn) and (cta_n % cta_k != 0):
497:         return [], []
498: 
499:     grouped = is_grouped(gemm_kind)
500:     if grouped:
501:         # the following cases are unsupported by grouped GEMM
502:         if not is_aligned:
503:             return [], []
504:         if requires_transposed_epilogue:
505:             return [], []
506: 
507:     # Early pruning
508:     if level < 1:
509:         # Don't stamp out FP16/BF16 kernels smaller than or equal to 64x128x64
510:         if is_fp16 and cta_m <= 64 and cta_n <= 128 and cta_k <= 64:
511:             return [], []
512: 
513:         # FP8 configs with CTA tile larger than or equal to 256x128x128 limit data types and schedules
514:         is_large_fp8_tile = is_fp8 and cta_m >= 256 and cta_n >= 128 and cta_k >= 128
515:         if is_large_fp8_tile:
516:             # Only void-C, and only FP8 outputs allowed
517:             if not is_void_c or d_type not in FP8_TYPES:
518:                 return [], []
519:             if CudaToolkitVersionSatisfies(cuda_version, 12, 1) and can_do_cooperative and can_do_tma_epilogue:
520:                 schedules = []
521:                 if is_blockwise(gemm_kind):
522:                     schedules.append(
523:                         [
524:                             to_grouped_schedule(KernelScheduleType.BlockwiseTmaWarpSpecializedCooperative, grouped),
525:                             to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecializedCooperative, grouped)
526:                         ])
527:                 else:
528:                     schedules.append(
529:                         [
530:                             to_grouped_schedule(KernelScheduleType.TmaWarpSpecializedCooperative, grouped),
531:                             to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecializedCooperative, grouped)
532:                         ])
533:                     schedules.append(
534:                         [
535:                             to_grouped_schedule(KernelScheduleType.TmaWarpSpecializedCooperativeFP8FastAccum, grouped),
536:                             to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecializedCooperative, grouped)
537:                         ])
538:                 return schedules, []
539:             return [], []
540: 
541:         if is_fp8 and not is_large_fp8_tile:
542:             valid_dtypes_for_c = [DataType.f32, DataType.bf16, DataType.f16, DataType.void]
543:             # Prune all configs with fp8 source, and all configs with non-fp8 output
544:             # that have different dtypes for source and output.
545:             if c_type not in valid_dtypes_for_c or (d_type not in FP8_TYPES and c_type != d_type):
546:                 return [], []
547: 
548:         # FP32/TF32 kernels don't stamp out void-C
549:         if is_fp32 and is_void_c:
550:             return [], []
551: 
552:     # Void-c only makes a difference for TMA epilogues
553:     if is_void_c and not can_do_tma_epilogue:
554:         return [], []
555: 
556:     # For mixed input data types
557:     a_type_size = DataTypeSize[data_types["a_type"]]
558:     b_type_size = DataTypeSize[data_types["b_type"]]
559:     if a_type_size != b_type_size and CudaToolkitVersionSatisfies(cuda_version, 12, 1):
560:         schedules = []
561:         stream_k_schedules = []
562:         epilogue_schedule = EpilogueScheduleType.TmaWarpSpecialized
563:         if a_type_size > b_type_size:
564:             epilogue_schedule = EpilogueScheduleType.EpilogueTransposed
565:         
566:         if not is_blockwise(gemm_kind):
567:             schedules.append([
568:                 KernelScheduleType.TmaWarpSpecialized,
569:                 epilogue_schedule
570:             ])
571:             schedules.append([
572:                 KernelScheduleType.TmaWarpSpecializedPingpong,
573:                 epilogue_schedule
574:             ])
575:         if cta_m >= 128:
576:             if a_type_size > b_type_size:
577:                 epilogue_schedule = EpilogueScheduleType.EpilogueTransposed
578:             else:
579:                 epilogue_schedule = EpilogueScheduleType.TmaWarpSpecializedCooperative
580:             if is_blockwise(gemm_kind):
581:                 schedules.append([
582:                     KernelScheduleType.BlockwiseTmaWarpSpecializedCooperative,
583:                     epilogue_schedule
584:                 ])
585:             else:
586:                 schedules.append([
587:                     KernelScheduleType.TmaWarpSpecializedCooperative,
588:                     epilogue_schedule
589:                 ])
590:                 stream_k_schedules.append([
591:                     KernelScheduleType.TmaWarpSpecializedCooperative,
592:                     epilogue_schedule
593:                 ])
594:         return schedules, stream_k_schedules
595: 
596:     if not is_aligned and not is_blockwise(gemm_kind):
597:         schedules = [[KernelScheduleType.CpAsyncWarpSpecialized,
598:                     default_epilogue]]
599:         stream_k_schedules = []
600: 
601:         if CudaToolkitVersionSatisfies(cuda_version, 12, 1) and can_do_cooperative:
602:             schedules.append([
603:                 KernelScheduleType.CpAsyncWarpSpecializedCooperative,
604:                 default_epilogue
605:             ])
606:             stream_k_schedules.append([
607:                 KernelScheduleType.CpAsyncWarpSpecializedCooperative,
608:                 default_epilogue
609:             ])
610: 
611:         return schedules, stream_k_schedules
612: 
613:     schedules = []
614:     # Pruning: emit Void-C and Grouped kernels with persistent kernels only
615:     if (level >= 1 or not is_void_c) and not grouped and not is_blockwise(gemm_kind):
616:         # Pruning: don't stamp out fp8 kernels with auto schedule
617:         if not is_fp8:
618:             schedules.append([KernelScheduleType.ScheduleAuto, auto_epilogue])
619:         schedules.append([KernelScheduleType.TmaWarpSpecialized, default_epilogue])
620:     stream_k_schedules = []
621:     
622:     if CudaToolkitVersionSatisfies(cuda_version, 12, 0):
623:         if can_do_tma_epilogue:
624:             assert not requires_transposed_epilogue
625:             # Inconsistency: fp8 pingpong only gets stamped out with fast accum
626:             if (not is_fp8 or level >= 1) and not is_blockwise(gemm_kind):
627:                 schedules.append([
628:                     to_grouped_schedule(KernelScheduleType.TmaWarpSpecializedPingpong, grouped),
629:                     to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecialized, grouped)
630:                 ])
631:             if can_do_fp8_fast_accum:
632:                 schedules.append([
633:                     to_grouped_schedule(KernelScheduleType.TmaWarpSpecializedPingpongFP8FastAccum, grouped),
634:                     to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecialized, grouped)
635:                 ])
636: 
637:     if CudaToolkitVersionSatisfies(cuda_version, 12, 1):
638:         # Pruning: don't stamp out fp8 ping-pong kernel with non-tma epilogue
639:         if not is_fp8 or level >= 1:
640:             if not is_blockwise(gemm_kind):
641:                 schedules.append([to_grouped_schedule(KernelScheduleType.TmaWarpSpecializedPingpong, grouped), to_grouped_schedule(default_epilogue, grouped)])
642:             else:
643:                 schedules.append([to_grouped_schedule(KernelScheduleType.BlockwiseTmaWarpSpecializedPingpong, grouped), to_grouped_schedule(default_epilogue, grouped)])
644: 
645:         if can_do_fp8_fast_accum:
646:             if not grouped:
647:                 schedules.append([KernelScheduleType.TmaWarpSpecializedFP8FastAccum, default_epilogue])
648:             schedules.append([to_grouped_schedule(KernelScheduleType.TmaWarpSpecializedPingpongFP8FastAccum, grouped), to_grouped_schedule(default_epilogue, grouped)])
649: 
650:         if can_do_cooperative:
651:             if is_blockwise(gemm_kind):
652:                 schedules.append([
653:                     to_grouped_schedule(KernelScheduleType.BlockwiseTmaWarpSpecializedCooperative, grouped),
654:                     to_grouped_schedule(default_epilogue, grouped)
655:                 ])
656:                 stream_k_schedules.append([
657:                     KernelScheduleType.BlockwiseTmaWarpSpecializedCooperative,
658:                     default_epilogue
659:                 ])
660:             else:
661:                 schedules.append([
662:                     to_grouped_schedule(KernelScheduleType.TmaWarpSpecializedCooperative, grouped),
663:                     to_grouped_schedule(default_epilogue, grouped)
664:                 ])
665:                 stream_k_schedules.append([
666:                     KernelScheduleType.TmaWarpSpecializedCooperative,
667:                     default_epilogue
668:                 ])
669:             if can_do_fp8_fast_accum:
670:                 schedules.append([
671:                     to_grouped_schedule(KernelScheduleType.TmaWarpSpecializedCooperativeFP8FastAccum, grouped),
672:                     to_grouped_schedule(default_epilogue, grouped)
673:                 ])
674:                 stream_k_schedules.append([
675:                     KernelScheduleType.TmaWarpSpecializedCooperativeFP8FastAccum,
676:                     default_epilogue
677:                 ])
678: 
679:         # persistent kernels with TMA epilogues
680:         if can_do_tma_epilogue:
681:             assert not requires_transposed_epilogue
682:             if can_do_cooperative:
683:                 if is_blockwise(gemm_kind):
684:                     schedules.append([
685:                         to_grouped_schedule(KernelScheduleType.BlockwiseTmaWarpSpecializedCooperative, grouped),
686:                         to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecializedCooperative, grouped)
687:                     ])
688:                     stream_k_schedules.append([
689:                         KernelScheduleType.BlockwiseTmaWarpSpecializedCooperative,
690:                         EpilogueScheduleType.TmaWarpSpecializedCooperative
691:                     ])
692:                 else:
693:                     schedules.append([
694:                         to_grouped_schedule(KernelScheduleType.TmaWarpSpecializedCooperative, grouped),
695:                         to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecializedCooperative, grouped)
696:                     ])
697:                     stream_k_schedules.append([
698:                         KernelScheduleType.TmaWarpSpecializedCooperative,
699:                         EpilogueScheduleType.TmaWarpSpecializedCooperative
700:                     ])
701:                 if can_do_fp8_fast_accum:
702:                     schedules.append([
703:                         to_grouped_schedule(KernelScheduleType.TmaWarpSpecializedCooperativeFP8FastAccum, grouped),
704:                         to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecializedCooperative, grouped)
705:                     ])
706:                     stream_k_schedules.append([
707:                         KernelScheduleType.TmaWarpSpecializedCooperativeFP8FastAccum,
708:                         EpilogueScheduleType.TmaWarpSpecializedCooperative
709:                     ])
710:     # Grouped GEMM do not support Stream-K scheduler
711:     if grouped:
712:         return schedules, []
713:     return schedules, stream_k_schedules
```
**EN:** Defines `get_valid_schedules()`, which returns or derives valid schedules. Key helper calls include get_pruning_level_from_global_level, is_tile_desc_compatible_with_cooperative, is_grouped, CudaToolkitVersionSatisfies, is_tile_desc_valid, can_tile_desc_use_shmem_in_epilogue.

**CN:** 定义 `get_valid_schedules()`，用于返回或推导 valid schedules。 其中会调用的重要辅助函数包括 get_pruning_level_from_global_level, is_tile_desc_compatible_with_cooperative, is_grouped, CudaToolkitVersionSatisfies, is_tile_desc_valid, can_tile_desc_use_shmem_in_epilogue。

### L714-L717 — Comments

```python
714: 
715: 
716: #### Misc: helpers
717: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L718-L732 — Function `generate_data_types_from_math_instruction`

```python
718: def generate_data_types_from_math_instruction(math_instruction, element_source = None, element_dest = None, element_epilogue = None):
719:     element_a, element_b = math_instruction.element_a, math_instruction.element_b
720:     element_accumulator = math_instruction.element_accumulator
721:     element_c = element_source or element_accumulator
722:     element_d = element_dest or element_accumulator
723:     element_epilogue = element_epilogue or element_accumulator
724:     data_types = {
725:         "a_type"   : element_a,
726:         "b_type"   : element_b,
727:         "c_type"   : element_c,
728:         "d_type"   : element_d,
729:         "acc_type" : element_accumulator,
730:         "epi_type" : element_epilogue
731:     }
732:     return data_types
```
**EN:** Defines `generate_data_types_from_math_instruction()`, which generates data types from math instruction.

**CN:** 定义 `generate_data_types_from_math_instruction()`，用于生成 data types from math instruction。

### L734-L753 — Function `fix_alignments`

```python
734: def fix_alignments(data_types, layout, alignment_bits = 128):
735:     operand_keys = ["a_type", "b_type", "c_type"]
736:     operands_to_fix = ["c_type"]
737:     new_layout = []
738:     assert len(layout) == len(operand_keys)
739:     for i, k in enumerate(operand_keys):
740:         assert k in data_types and data_types[k] in DataTypeSize
741:         dtype = data_types[k]
742:         dtype_size_bits = DataTypeSize[dtype]
743: 
744:         layout_type = layout[i][0]
745:         layout_alignment = layout[i][1]
746: 
747:         # Don't modify alignment if dtype's been changed to void
748:         if k in operands_to_fix and dtype_size_bits >= 1:
749:             layout_alignment = alignment_bits // dtype_size_bits
750: 
751:         new_layout.append([layout_type, layout_alignment])
752: 
753:     return new_layout
```
**EN:** Defines `fix_alignments()`, which implements the fix alignments logic. Key helper calls include enumerate, len, append.

**CN:** 定义 `fix_alignments()`，用于实现 fix alignments 相关逻辑。 其中会调用的重要辅助函数包括 enumerate, len, append。

## Key Concepts / 关键概念

- **EN:** File role: Provides helper routines that derive SM90 math instructions, clusters, tiles, schedules, and pruning behavior.
- **CN:** 文件角色：提供推导 SM90 数学指令、集群、tile、调度与裁剪行为的辅助例程。
- **EN:** Main functions: `CudaToolkitVersionSatisfies, get_wgmma_level_from_global_level, get_mma_level_from_global_level, get_cluster_level_from_global_level, get_pruning_level_from_global_level, generate_tf32_math_instruction_shapes_sm90, generate_fp16_bf16_math_instruction_shapes_sm90, generate_fp8_math_instruction_shapes_sm90, generate_int8_math_instruction_shapes_sm90, generate_mixed_dtype_math_instructions_shapes_sm90, generate_tf32_math_instructions_sm90, generate_fp16_bf16_math_instructions_sm90, ...`
- **CN:** 主要函数：`CudaToolkitVersionSatisfies, get_wgmma_level_from_global_level, get_mma_level_from_global_level, get_cluster_level_from_global_level, get_pruning_level_from_global_level, generate_tf32_math_instruction_shapes_sm90, generate_fp16_bf16_math_instruction_shapes_sm90, generate_fp8_math_instruction_shapes_sm90, generate_int8_math_instruction_shapes_sm90, generate_mixed_dtype_math_instructions_shapes_sm90, generate_tf32_math_instructions_sm90, generate_fp16_bf16_math_instructions_sm90, ...`

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: `argparse, enum, itertools, math, logging, os.path, shutil, sys, copy, typing, builtins`
- **CN:** 标准库依赖：`argparse, enum, itertools, math, logging, os.path, shutil, sys, copy, typing, builtins`
- **EN:** Internal CUTLASS dependencies: `cutlass_library.library, .sm90_shapes, library, sm90_shapes`
- **CN:** CUTLASS 内部依赖：`cutlass_library.library, .sm90_shapes, library, sm90_shapes`
- **EN:** External/non-stdlib dependencies: `None explicitly imported`
- **CN:** 外部/非标准库依赖：`未显式导入`
