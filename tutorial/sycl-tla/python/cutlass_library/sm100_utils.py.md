# sm100_utils.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/cutlass_library/sm100_utils.py`
- **EN:** Provides helper routines that derive SM100 math instructions, clusters, tiles, schedules, and pruning behavior.
- **CN:** 提供推导 SM100 数学指令、集群、tile、调度与裁剪行为的辅助例程。

## Line-by-Line Analysis / 逐行分析

### L1-L32 — Header comments

```python
  1: #################################################################################################
  2: #
  3: # Copyright (c) 2025 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
 34: Utilities for enumerating CUTLASS library SM100 kernels
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
 46: from typing import Any, Optional, Sequence, Tuple, List, Union, Callable
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

### L55-L74 — Comments

```python
 55: 
 56: #### Step 0: define levels
 57: 
 58: # One integer level controls multiple "generators" and how many
 59: # combinations they generate. That is the "global" level.
 60: # "Generators" are WGMMA shapes, MMA multipliers, cluster sizes, and
 61: # anything that is eventually involved in the Cartesian product
 62: # which yields our kernel configurations.
 63: # For simplicity, each generator defines their own levels, 
 64: # starting from 0. As a rule we assume 10 or fewer levels, making
 65: # their level a digit.
 66: # The "global" level simply stacks these digits and represents them
 67: # as a single integer.
 68: # 
 69: # For example, level 500 indicates cluster sizes are at level 5, MMA
 70: # multipliers are at level 0, and WGMMA shapes are at level 0 as well.
 71: #
 72: # Here we define the global level to generator level mappings.
 73: 
 74: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L75-L76 — Function `get_tcgen05_level_from_global_level`

```python
 75: def get_tcgen05_level_from_global_level(global_level: int):
 76:     return global_level % 10
```
**EN:** Defines `get_tcgen05_level_from_global_level()`, which returns or derives tcgen05 level from global level.

**CN:** 定义 `get_tcgen05_level_from_global_level()`，用于返回或推导 tcgen05 level from global level。

### L78-L79 — Function `get_mma_level_from_global_level`

```python
 78: def get_mma_level_from_global_level(global_level: int):
 79:     return (global_level // 10) % 10
```
**EN:** Defines `get_mma_level_from_global_level()`, which returns or derives mma level from global level.

**CN:** 定义 `get_mma_level_from_global_level()`，用于返回或推导 mma level from global level。

### L82-L83 — Function `get_cluster_level_from_global_level`

```python
 82: def get_cluster_level_from_global_level(global_level: int):
 83:     return (global_level // 100) % 10
```
**EN:** Defines `get_cluster_level_from_global_level()`, which returns or derives cluster level from global level.

**CN:** 定义 `get_cluster_level_from_global_level()`，用于返回或推导 cluster level from global level。

### L86-L87 — Function `get_pruning_level_from_global_level`

```python
 86: def get_pruning_level_from_global_level(global_level: int):
 87:     return (global_level // 1000) % 10
```
**EN:** Defines `get_pruning_level_from_global_level()`, which returns or derives pruning level from global level.

**CN:** 定义 `get_pruning_level_from_global_level()`，用于返回或推导 pruning level from global level。

### L88-L91 — Comments

```python
 88: 
 89: 
 90: #### Step 1: generate MMA instruction shapes based on levels
 91: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L92-L95 — Import fallback

```python
 92: try:
 93:     from .sm100_shapes import *
 94: except:
 95:     from sm100_shapes import *
```
**EN:** Uses `try`/`except` to prefer package imports and fall back to local imports, keeping both installed-package and script execution modes working.

**CN:** 通过 `try`/`except` 优先使用包导入，并在失败时回退到本地导入，从而同时兼容已安装包与脚本运行模式。

### L96-L98 — Comments

```python
 96: 
 97: ###########
 98: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L99-L139 — Function `generate_tf32_math_instructions_sm100`

```python
 99: def generate_tf32_math_instructions_sm100(level: int):
100:     """
101:     Generate all TensorOp math instructions for TF32 MMA that are supported by SM100 at or above the given level.
102: 
103:     Args:
104:         level: The global level to generate math instructions for.
105: 
106:     Returns:
107:         A tuple of two lists of MathInstruction objects. 
108:         The first list contains the math instructions for 1SM, and the second list contains the math instructions for 2SM.
109:     """
110:     tcgen05_level = get_tcgen05_level_from_global_level(level)
111:     math_instructions_1sm = []
112:     math_instructions_2sm = []
113: 
114:     shapes_1sm = [
115:         shape for shape, min_level in SM100_MMA_SHAPES_TF32_DENSE_1SM.items() if tcgen05_level >= min_level
116:     ]
117:     shapes_2sm = [
118:         shape for shape, min_level in SM100_MMA_SHAPES_TF32_DENSE_2SM.items() if tcgen05_level >= min_level
119:     ]
120: 
121:     for shape in shapes_1sm:
122:         math_instructions_1sm.append(
123:           MathInstruction(
124:               shape,
125:               DataType.tf32, DataType.tf32, DataType.f32,
126:               OpcodeClass.TensorOp,
127:               MathOperation.multiply_add)
128:         )
129: 
130:     for shape in shapes_2sm:
131:         math_instructions_2sm.append(
132:           MathInstruction(
133:               shape,
134:               DataType.tf32, DataType.tf32, DataType.f32,
135:               OpcodeClass.TensorOp,
136:               MathOperation.multiply_add)
137:         )
138:  
139:     return math_instructions_1sm, math_instructions_2sm
```
**EN:** Defines `generate_tf32_math_instructions_sm100()`. The docstring says: "Generate all TensorOp math instructions for TF32 MMA that are supported by SM100 at or above the given level."; this block implements that behavior. Key helper calls include get_tcgen05_level_from_global_level, append, items, MathInstruction.

**CN:** 定义 `generate_tf32_math_instructions_sm100()`。文档字符串说明其用途为：“Generate all TensorOp math instructions for TF32 MMA that are supported by SM100 at or above the given level.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 get_tcgen05_level_from_global_level, append, items, MathInstruction。

### L141-L210 — Function `generate_16b_math_instructions_sm100`

```python
141: def generate_16b_math_instructions_sm100(level: int):
142:     """
143:     Generate all TensorOp math instructions for 16b MMA that are supported by SM100 at or above the given level.
144: 
145:     Args:
146:         level: The global level to generate math instructions for.
147: 
148:     Returns:
149:         A tuple of two lists of MathInstruction objects. 
150:         The first list contains the math instructions for 1SM, and the second list contains the math instructions for 2SM.
151:     """
152:     tcgen05_level = get_tcgen05_level_from_global_level(level)
153:     math_instructions_1sm = []
154:     math_instructions_2sm = []
155: 
156:     shapes_1sm = [
157:         shape for shape, min_level in SM100_MMA_SHAPES_16b_DENSE_1SM.items() if tcgen05_level >= min_level
158:     ]
159:     shapes_2sm = [
160:         shape for shape, min_level in SM100_MMA_SHAPES_16b_DENSE_2SM.items() if tcgen05_level >= min_level
161:     ]
162: 
163:     for shape in shapes_1sm:
164:         math_instructions_1sm.append(
165:           MathInstruction(
166:               shape,
167:               DataType.f16, DataType.f16, DataType.f16,
168:               OpcodeClass.TensorOp,
169:               MathOperation.multiply_add)
170:         )
171:         math_instructions_1sm.append(
172:           MathInstruction(
173:               shape,
174:               DataType.f16, DataType.f16, DataType.f32,
175:               OpcodeClass.TensorOp,
176:               MathOperation.multiply_add)
177:         )
178:         math_instructions_1sm.append(
179:           MathInstruction(
180:               shape,
181:               DataType.bf16, DataType.bf16, DataType.f32,
182:               OpcodeClass.TensorOp,
183:               MathOperation.multiply_add)
184:         )
185: 
186: 
187:     for shape in shapes_2sm:
188:         math_instructions_2sm.append(
189:           MathInstruction(
190:               shape,
191:               DataType.f16, DataType.f16, DataType.f16,
192:               OpcodeClass.TensorOp,
193:               MathOperation.multiply_add)
194:         )
195:         math_instructions_2sm.append(
196:           MathInstruction(
197:               shape,
198:               DataType.f16, DataType.f16, DataType.f32,
199:               OpcodeClass.TensorOp,
200:               MathOperation.multiply_add)
201:         )
202:         math_instructions_2sm.append(
203:           MathInstruction(
204:               shape,
205:               DataType.bf16, DataType.bf16, DataType.f32,
206:               OpcodeClass.TensorOp,
207:               MathOperation.multiply_add)
208:         )
209:  
210:     return math_instructions_1sm, math_instructions_2sm
```
**EN:** Defines `generate_16b_math_instructions_sm100()`. The docstring says: "Generate all TensorOp math instructions for 16b MMA that are supported by SM100 at or above the given level."; this block implements that behavior. Key helper calls include get_tcgen05_level_from_global_level, append, items, MathInstruction.

**CN:** 定义 `generate_16b_math_instructions_sm100()`。文档字符串说明其用途为：“Generate all TensorOp math instructions for 16b MMA that are supported by SM100 at or above the given level.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 get_tcgen05_level_from_global_level, append, items, MathInstruction。

### L213-L319 — Function `generate_fp8_math_instructions_sm100`

```python
213: def generate_fp8_math_instructions_sm100(level: int, enable_runtime_dtype = True, enable_compile_time_dtype = True):
214:     """
215:     Generate all TensorOp math instructions for FP8 MMA that are supported by SM100 at or above the given level.
216: 
217:     Args:
218:         level: The global level to generate math instructions for.
219:         enable_runtime_dtype: Whether to generate runtime dtype math instructions.
220:         enable_compile_time_dtype: Whether to generate compile time dtype math instructions.
221: 
222:     Returns:
223:         A tuple of two lists of MathInstruction objects. 
224:         The first list contains the math instructions for 1SM, and the second list contains the math instructions for 2SM.
225:     """
226: 
227:     tcgen05_level = get_tcgen05_level_from_global_level(level)
228:     pruning_level = get_pruning_level_from_global_level(level)
229:     math_instructions_1sm = []
230:     math_instructions_2sm = []
231: 
232:     shapes_1sm = [
233:         shape for shape, min_level in SM100_MMA_SHAPES_F8F6F4_DENSE_1SM.items() if tcgen05_level >= min_level
234:     ]
235:     shapes_2sm = [
236:         shape for shape, min_level in SM100_MMA_SHAPES_F8F6F4_DENSE_2SM.items() if tcgen05_level >= min_level
237:     ]
238: 
239:     for shape in shapes_1sm:
240:         if enable_runtime_dtype:
241:             math_instructions_1sm.append(
242:               MathInstruction(
243:                   shape,
244:                   DataType.f8, DataType.f8, DataType.f32,
245:                   OpcodeClass.TensorOp,
246:                   MathOperation.multiply_add)
247:             )
248:         if enable_compile_time_dtype:    
249:             math_instructions_1sm.append(
250:               MathInstruction(
251:                   shape,
252:                   DataType.e4m3, DataType.e4m3, DataType.f32,
253:                   OpcodeClass.TensorOp,
254:                   MathOperation.multiply_add)
255:             )
256:             math_instructions_1sm.append(
257:               MathInstruction(
258:                   shape,
259:                   DataType.e5m2, DataType.e4m3, DataType.f32,
260:                   OpcodeClass.TensorOp,
261:                   MathOperation.multiply_add)
262:             )
263:             math_instructions_1sm.append(
264:               MathInstruction(
265:                   shape,
266:                   DataType.e4m3, DataType.e5m2, DataType.f32,
267:                   OpcodeClass.TensorOp,
268:                   MathOperation.multiply_add)
269:             )
270:             if pruning_level >= 2:
271:                 math_instructions_1sm.append(
272:                   MathInstruction(
273:                       shape,
274:                       DataType.e5m2, DataType.e5m2, DataType.f32,
275:                       OpcodeClass.TensorOp,
276:                       MathOperation.multiply_add)
277:                 )
278: 
279:     for shape in shapes_2sm:
280:         if enable_runtime_dtype:
281:             math_instructions_2sm.append(
282:               MathInstruction(
283:                   shape,
284:                   DataType.f8, DataType.f8, DataType.f32,
285:                   OpcodeClass.TensorOp,
286:                   MathOperation.multiply_add)
287:             )
288:         if enable_compile_time_dtype:    
289:             math_instructions_2sm.append(
290:               MathInstruction(
291:                   shape,
292:                   DataType.e4m3, DataType.e4m3, DataType.f32,
293:                   OpcodeClass.TensorOp,
294:                   MathOperation.multiply_add)
295:             )
296:             math_instructions_2sm.append(
297:               MathInstruction(
298:                   shape,
299:                   DataType.e5m2, DataType.e4m3, DataType.f32,
300:                   OpcodeClass.TensorOp,
301:                   MathOperation.multiply_add)
302:             )
303:             math_instructions_2sm.append(
304:               MathInstruction(
305:                   shape,
306:                   DataType.e4m3, DataType.e5m2, DataType.f32,
307:                   OpcodeClass.TensorOp,
308:                   MathOperation.multiply_add)
309:             )
310:             if pruning_level >= 2:
311:                 math_instructions_2sm.append(
312:                   MathInstruction(
313:                       shape,
314:                       DataType.e5m2, DataType.e5m2, DataType.f32,
315:                       OpcodeClass.TensorOp,
316:                       MathOperation.multiply_add)
317:                 )
318: 
319:     return math_instructions_1sm, math_instructions_2sm
```
**EN:** Defines `generate_fp8_math_instructions_sm100()`. The docstring says: "Generate all TensorOp math instructions for FP8 MMA that are supported by SM100 at or above the given level."; this block implements that behavior. Key helper calls include get_tcgen05_level_from_global_level, get_pruning_level_from_global_level, items, append, MathInstruction.

**CN:** 定义 `generate_fp8_math_instructions_sm100()`。文档字符串说明其用途为：“Generate all TensorOp math instructions for FP8 MMA that are supported by SM100 at or above the given level.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 get_tcgen05_level_from_global_level, get_pruning_level_from_global_level, items, append, MathInstruction。

### L321-L399 — Function `generate_f8f6f4_math_instructions_sm100`

```python
321: def generate_f8f6f4_math_instructions_sm100(level: int, enable_runtime_dtype = True, enable_compile_time_dtype = True):
322:     """
323:     Generate all TensorOp math instructions for FP8 FP6 and FP4 MMA that are supported by SM100 at or above the given level.
324: 
325:     Args:
326:         level: The global level to generate math instructions for.
327:         enable_runtime_dtype: Whether to generate runtime dtype math instructions.
328:         enable_compile_time_dtype: Whether to generate compile time dtype math instructions.
329: 
330:     Returns:
331:         A tuple of two lists of MathInstruction objects. 
332:         The first list contains the math instructions for 1SM, and the second list contains the math instructions for 2SM.
333:     """
334: 
335:     tcgen05_level = get_tcgen05_level_from_global_level(level)
336:     math_instructions_1sm = []
337:     math_instructions_2sm = []
338: 
339:     shapes_1sm = [
340:         shape for shape, min_level in SM100_MMA_SHAPES_F8F6F4_DENSE_1SM.items() if tcgen05_level >= min_level
341:     ]
342:     shapes_2sm = [
343:         shape for shape, min_level in SM100_MMA_SHAPES_F8F6F4_DENSE_2SM.items() if tcgen05_level >= min_level
344:     ]
345: 
346:     for shape in shapes_1sm:
347:         if enable_runtime_dtype:
348: 
349:             runtime_types = [ DataType.f8, DataType.f6, DataType.f4 ]
350: 
351:             for a_type, b_type in product(runtime_types, repeat=2):
352:                 math_instructions_1sm.append(
353:                   MathInstruction(
354:                       shape,
355:                       a_type, b_type, DataType.f32,
356:                       OpcodeClass.TensorOp,
357:                       MathOperation.multiply_add)
358:                 )
359: 
360:         if enable_compile_time_dtype:
361:             compile_time_types = [ DataType.e4m3, DataType.e5m2, DataType.e3m2, DataType.e2m1 ]
362: 
363:             for a_type, b_type in product(compile_time_types, repeat=2):
364:                 math_instructions_1sm.append(
365:                   MathInstruction(
366:                       shape,
367:                       a_type, b_type, DataType.f32,
368:                       OpcodeClass.TensorOp,
369:                       MathOperation.multiply_add)
370:                 )
371: 
372: 
373:     for shape in shapes_2sm:
374:         if enable_runtime_dtype:
375: 
376:             runtime_types = [ DataType.f8, DataType.f6, DataType.f4 ]
377: 
378:             for a_type, b_type in product(runtime_types, repeat=2):
379:                 math_instructions_2sm.append(
380:                   MathInstruction(
381:                       shape,
382:                       a_type, b_type, DataType.f32,
383:                       OpcodeClass.TensorOp,
384:                       MathOperation.multiply_add)
385:                 )
386: 
387:         if enable_compile_time_dtype:
388:             compile_time_types = [ DataType.e4m3, DataType.e5m2, DataType.e3m2, DataType.e2m1 ]
389: 
390:             for a_type, b_type in product(compile_time_types, repeat=2):
391:                 math_instructions_2sm.append(
392:                   MathInstruction(
393:                       shape,
394:                       a_type, b_type, DataType.f32,
395:                       OpcodeClass.TensorOp,
396:                       MathOperation.multiply_add)
397:                 )
398: 
399:     return math_instructions_1sm, math_instructions_2sm
```
**EN:** Defines `generate_f8f6f4_math_instructions_sm100()`. The docstring says: "Generate all TensorOp math instructions for FP8 FP6 and FP4 MMA that are supported by SM100 at or above the given level."; this block implements that behavior. Key helper calls include get_tcgen05_level_from_global_level, items, product, append, MathInstruction.

**CN:** 定义 `generate_f8f6f4_math_instructions_sm100()`。文档字符串说明其用途为：“Generate all TensorOp math instructions for FP8 FP6 and FP4 MMA that are supported by SM100 at or above the given level.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 get_tcgen05_level_from_global_level, items, product, append, MathInstruction。

### L401-L501 — Function `generate_mxf8f6f4_math_instructions_sm100`

```python
401: def generate_mxf8f6f4_math_instructions_sm100(level: int, enable_runtime_dtype = True, enable_compile_time_dtype = True):
402:     """
403:     Generate all BlockScaledTensorOp math instructions for MXFP8, MXFP6, and MXFP4 MMA that are supported by SM100 at or above the given level.
404: 
405:     Args:
406:         level: The global level to generate math instructions for.
407:         enable_runtime_dtype: Whether to generate runtime dtype math instructions.
408:         enable_compile_time_dtype: Whether to generate compile time dtype math instructions.
409: 
410:     Returns:
411:         A tuple of two lists of MathInstruction objects. 
412:         The first list contains the math instructions for 1SM, and the second list contains the math instructions for 2SM.
413:     """
414: 
415:     tcgen05_level = get_tcgen05_level_from_global_level(level)
416:     pruning_level = get_pruning_level_from_global_level(level)
417: 
418:     math_instructions_1sm = []
419:     math_instructions_2sm = []
420: 
421:     shapes_1sm = [
422:         shape for shape, min_level in SM100_MMA_SHAPES_MXF8F6F4_DENSE_1SM.items() if tcgen05_level >= min_level
423:     ]
424:     shapes_2sm = [
425:         shape for shape, min_level in SM100_MMA_SHAPES_MXF8F6F4_DENSE_2SM.items() if tcgen05_level >= min_level
426:     ]
427: 
428:     for shape in shapes_1sm:
429:         if enable_runtime_dtype:
430: 
431:             runtime_types = [ DataType.f8, DataType.f6, DataType.f4 ]
432: 
433:             for a_type, b_type in product(runtime_types, repeat=2):
434: 
435:                 if pruning_level < 2 and ((a_type == DataType.f8 or b_type == DataType.f8)):
436:                     continue
437: 
438:                 math_instructions_1sm.append(
439:                   MathInstruction(
440:                       shape,
441:                       a_type, b_type, DataType.f32,
442:                       OpcodeClass.BlockScaledTensorOp,
443:                       MathOperation.multiply_add,
444:                       DataType.ue8m0)
445:                 )
446: 
447:         if enable_compile_time_dtype:
448:             compile_time_types = [ DataType.e4m3, 
449:                                    DataType.e5m2, 
450:                                    DataType.e3m2, 
451:                                    DataType.e2m3,
452:                                    DataType.e2m1 ]
453: 
454:             for a_type, b_type in product(compile_time_types, repeat=2):
455:                 math_instructions_1sm.append(
456:                   MathInstruction(
457:                       shape,
458:                       a_type, b_type, DataType.f32,
459:                       OpcodeClass.BlockScaledTensorOp,
460:                       MathOperation.multiply_add,
461:                       DataType.ue8m0)
462:                 )
463: 
464: 
465:     for shape in shapes_2sm:
466:         if enable_runtime_dtype:
467: 
468:             runtime_types = [ DataType.f8, DataType.f6, DataType.f4 ]
469: 
470:             for a_type, b_type in product(runtime_types, repeat=2):
471: 
472:                 if pruning_level < 2 and ((a_type == DataType.f8 or b_type == DataType.f8)):
473:                     continue
474: 
475:                 math_instructions_2sm.append(
476:                   MathInstruction(
477:                       shape,
478:                       a_type, b_type, DataType.f32,
479:                       OpcodeClass.BlockScaledTensorOp,
480:                       MathOperation.multiply_add,
481:                       DataType.ue8m0)
482:                 )
483: 
484:         if enable_compile_time_dtype:
485:             compile_time_types = [ DataType.e4m3, 
486:                                    DataType.e5m2, 
487:                                    DataType.e3m2, 
488:                                    DataType.e2m3,
489:                                    DataType.e2m1 ]
490: 
491:             for a_type, b_type in product(compile_time_types, repeat=2):
492:                 math_instructions_2sm.append(
493:                   MathInstruction(
494:                       shape,
495:                       a_type, b_type, DataType.f32,
496:                       OpcodeClass.BlockScaledTensorOp,
497:                       MathOperation.multiply_add,
498:                       DataType.ue8m0)
499:                 )
500: 
501:     return math_instructions_1sm, math_instructions_2sm
```
**EN:** Defines `generate_mxf8f6f4_math_instructions_sm100()`. The docstring says: "Generate all BlockScaledTensorOp math instructions for MXFP8, MXFP6, and MXFP4 MMA that are supported by SM100 at or above the given level."; this block implements that behavior. Key helper calls include get_tcgen05_level_from_global_level, get_pruning_level_from_global_level, items, product, append, MathInstruction.

**CN:** 定义 `generate_mxf8f6f4_math_instructions_sm100()`。文档字符串说明其用途为：“Generate all BlockScaledTensorOp math instructions for MXFP8, MXFP6, and MXFP4 MMA that are supported by SM100 at or above the given level.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 get_tcgen05_level_from_global_level, get_pruning_level_from_global_level, items, product, append, MathInstruction。

### L503-L621 — Function `generate_mxf4nvf4_math_instructions_sm100`

```python
503: def generate_mxf4nvf4_math_instructions_sm100(level: int, enable_runtime_dtype = True, enable_compile_time_dtype = True):
504:     """
505:     Generate all BlockScaledTensorOp math instructions for MXFP4 and MXFP4 MMA that are supported by SM100 at or above the given level.
506: 
507:     Args:
508:         level: The global level to generate math instructions for.
509:         enable_runtime_dtype: Whether to generate runtime dtype math instructions.
510:         enable_compile_time_dtype: Whether to generate compile time dtype math instructions.
511: 
512:     Returns:
513:         A tuple of two lists of MathInstruction objects. 
514:         The first list contains the math instructions for 1SM, and the second list contains the math instructions for 2SM.
515:     """
516:     tcgen05_level = get_tcgen05_level_from_global_level(level)
517:     math_instructions_1sm = []
518:     math_instructions_2sm = []
519: 
520:     shapes_1sm = [
521:         shape for shape, min_level in SM100_MMA_SHAPES_MXF4NVF4_DENSE_1SM.items() if tcgen05_level >= min_level
522:     ]
523:     shapes_2sm = [
524:         shape for shape, min_level in SM100_MMA_SHAPES_MXF4NVF4_DENSE_2SM.items() if tcgen05_level >= min_level
525:     ]
526: 
527:     for shape in shapes_1sm:
528:         if enable_runtime_dtype:
529: 
530:             runtime_types = [ DataType.f4 ]
531: 
532:             for a_type, b_type in product(runtime_types, repeat=2):
533:                 math_instructions_1sm.append(
534:                   MathInstruction(
535:                       shape,
536:                       a_type, b_type, DataType.f32,
537:                       OpcodeClass.BlockScaledTensorOp,
538:                       MathOperation.multiply_add,
539:                       DataType.ue8m0)
540:                 )
541:                 math_instructions_1sm.append(
542:                   MathInstruction(
543:                       shape,
544:                       a_type, b_type, DataType.f32,
545:                       OpcodeClass.BlockScaledTensorOp,
546:                       MathOperation.multiply_add,
547:                       DataType.ue4m3)
548:                 )
549: 
550: 
551:         if enable_compile_time_dtype:
552:             compile_time_types = [ DataType.e2m1, 
553:                                  ]
554: 
555:             for a_type, b_type in product(compile_time_types, repeat=2):
556:                 math_instructions_1sm.append(
557:                   MathInstruction(
558:                       shape,
559:                       a_type, b_type, DataType.f32,
560:                       OpcodeClass.BlockScaledTensorOp,
561:                       MathOperation.multiply_add,
562:                       DataType.ue8m0)
563:                 )
564:                 math_instructions_1sm.append(
565:                   MathInstruction(
566:                       shape,
567:                       a_type, b_type, DataType.f32,
568:                       OpcodeClass.BlockScaledTensorOp,
569:                       MathOperation.multiply_add,
570:                       DataType.ue4m3)
571:                 )
572: 
573: 
574:     for shape in shapes_2sm:
575:         if enable_runtime_dtype:
576: 
577:             runtime_types = [ DataType.f4 ]
578: 
579:             for a_type, b_type in product(runtime_types, repeat=2):
580:                 math_instructions_2sm.append(
581:                   MathInstruction(
582:                       shape,
583:                       a_type, b_type, DataType.f32,
584:                       OpcodeClass.BlockScaledTensorOp,
585:                       MathOperation.multiply_add,
586:                       DataType.ue8m0)
587:                 )
588:                 math_instructions_2sm.append(
589:                   MathInstruction(
590:                       shape,
591:                       a_type, b_type, DataType.f32,
592:                       OpcodeClass.BlockScaledTensorOp,
593:                       MathOperation.multiply_add,
594:                       DataType.ue4m3)
595:                 )
596: 
597: 
598:         if enable_compile_time_dtype:
599:             compile_time_types = [ DataType.e2m1, 
600:                                  ]
601: 
602:             for a_type, b_type in product(compile_time_types, repeat=2):
603:                 math_instructions_2sm.append(
604:                   MathInstruction(
605:                       shape,
606:                       a_type, b_type, DataType.f32,
607:                       OpcodeClass.BlockScaledTensorOp,
608:                       MathOperation.multiply_add,
609:                       DataType.ue8m0)
610:                 )
611:                 math_instructions_2sm.append(
612:                   MathInstruction(
613:                       shape,
614:                       a_type, b_type, DataType.f32,
615:                       OpcodeClass.BlockScaledTensorOp,
616:                       MathOperation.multiply_add,
617:                       DataType.ue4m3)
618:                 )
619: 
620: 
621:     return math_instructions_1sm, math_instructions_2sm
```
**EN:** Defines `generate_mxf4nvf4_math_instructions_sm100()`. The docstring says: "Generate all BlockScaledTensorOp math instructions for MXFP4 and MXFP4 MMA that are supported by SM100 at or above the given level."; this block implements that behavior. Key helper calls include get_tcgen05_level_from_global_level, items, product, append, MathInstruction.

**CN:** 定义 `generate_mxf4nvf4_math_instructions_sm100()`。文档字符串说明其用途为：“Generate all BlockScaledTensorOp math instructions for MXFP4 and MXFP4 MMA that are supported by SM100 at or above the given level.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 get_tcgen05_level_from_global_level, items, product, append, MathInstruction。

### L624-L661 — Function `generate_cluster_shapes_sm100`

```python
624: def generate_cluster_shapes_sm100(level: int, change_priority_func : Union[Callable, None] = None):
625:     """
626:     Generate all cluster shapes for SM100 at or above the given level.
627: 
628:     Args:
629:         level: The global level to generate cluster shapes for.
630: 
631:     Returns:
632:         A tuple of two lists of cluster shapes. 
633:         The first list contains the cluster shapes for 1SM, and the second list contains the cluster shapes for 2SM.
634:     """
635:     cluster_level = get_cluster_level_from_global_level(level)
636: 
637:     assert cluster_level >= 4
638: 
639:     if change_priority_func is not None:
640:         SM100_CLUSTER_SHAPES_1SM_CPY = copy.deepcopy(SM100_CLUSTER_SHAPES_1SM)
641:         SM100_CLUSTER_SHAPES_2SM_CPY = copy.deepcopy(SM100_CLUSTER_SHAPES_2SM)
642:         change_priority_func(SM100_CLUSTER_SHAPES_1SM_CPY, SM100_CLUSTER_SHAPES_2SM_CPY)
643:         shapes_1sm = [
644:             list(shape) for shape, min_level in SM100_CLUSTER_SHAPES_1SM_CPY.items() if cluster_level >= min_level
645:         ]
646:         shapes_2sm = [
647:             list(shape) for shape, min_level in SM100_CLUSTER_SHAPES_2SM_CPY.items() if cluster_level >= min_level
648:         ]
649: 
650:         return shapes_1sm, shapes_2sm
651:    
652:     else:
653: 
654:         shapes_1sm = [
655:             list(shape) for shape, min_level in SM100_CLUSTER_SHAPES_1SM.items() if cluster_level >= min_level
656:         ]
657:         shapes_2sm = [
658:             list(shape) for shape, min_level in SM100_CLUSTER_SHAPES_2SM.items() if cluster_level >= min_level
659:         ]
660: 
661:         return shapes_1sm, shapes_2sm
```
**EN:** Defines `generate_cluster_shapes_sm100()`. The docstring says: "Generate all cluster shapes for SM100 at or above the given level."; this block implements that behavior. Key helper calls include get_cluster_level_from_global_level, deepcopy, change_priority_func, list, items.

**CN:** 定义 `generate_cluster_shapes_sm100()`。文档字符串说明其用途为：“Generate all cluster shapes for SM100 at or above the given level.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 get_cluster_level_from_global_level, deepcopy, change_priority_func, list, items。

## Key Concepts / 关键概念

- **EN:** File role: Provides helper routines that derive SM100 math instructions, clusters, tiles, schedules, and pruning behavior.
- **CN:** 文件角色：提供推导 SM100 数学指令、集群、tile、调度与裁剪行为的辅助例程。
- **EN:** Main functions: `get_tcgen05_level_from_global_level, get_mma_level_from_global_level, get_cluster_level_from_global_level, get_pruning_level_from_global_level, generate_tf32_math_instructions_sm100, generate_16b_math_instructions_sm100, generate_fp8_math_instructions_sm100, generate_f8f6f4_math_instructions_sm100, generate_mxf8f6f4_math_instructions_sm100, generate_mxf4nvf4_math_instructions_sm100, generate_cluster_shapes_sm100`
- **CN:** 主要函数：`get_tcgen05_level_from_global_level, get_mma_level_from_global_level, get_cluster_level_from_global_level, get_pruning_level_from_global_level, generate_tf32_math_instructions_sm100, generate_16b_math_instructions_sm100, generate_fp8_math_instructions_sm100, generate_f8f6f4_math_instructions_sm100, generate_mxf8f6f4_math_instructions_sm100, generate_mxf4nvf4_math_instructions_sm100, generate_cluster_shapes_sm100`

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: `argparse, enum, itertools, math, logging, os.path, shutil, sys, copy, typing, builtins`
- **CN:** 标准库依赖：`argparse, enum, itertools, math, logging, os.path, shutil, sys, copy, typing, builtins`
- **EN:** Internal CUTLASS dependencies: `cutlass_library.library, .sm100_shapes, library, sm100_shapes`
- **CN:** CUTLASS 内部依赖：`cutlass_library.library, .sm100_shapes, library, sm100_shapes`
- **EN:** External/non-stdlib dependencies: `None explicitly imported`
- **CN:** 外部/非标准库依赖：`未显式导入`
