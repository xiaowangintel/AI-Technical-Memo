# symm_operation.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/cutlass_library/symm_operation.py`
- **EN:** Models symmetric matrix-multiply operations and emits the corresponding kernel/configuration code.
- **CN:** 建模对称矩阵乘法操作，并生成对应的内核/配置代码。

## Line-by-Line Analysis / 逐行分析

### L1-L32 — Header comments

```python
  1: #################################################################################################
  2: #
  3: # Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### L33-L49 — Imports and setup

```python
 33: """
 34: Utilities for emitting Symm kernels
 35: """
 36: 
 37: import enum
 38: import functools
 39: import operator
 40: import os.path
 41: import shutil
 42: 
 43: try:
 44:   import builtins
 45:   if hasattr(builtins, "CUTLASS_IGNORE_PACKAGE") and CUTLASS_IGNORE_PACKAGE == True:
 46:     raise ImportError("Disabling attempt to import cutlass_library")
 47:   from cutlass_library.library import *
 48: except ImportError:
 49:   from library import *
```
**EN:** Imports and/or re-exports modules such as enum, functools, operator, os.path, shutil so later code can reuse shared definitions.

**CN:** 导入或重新导出诸如 enum, functools, operator, os.path, shutil 等模块，使后续代码可以复用共享定义。

### L50-L58 — Comments

```python
 50: 
 51: 
 52: ###################################################################################################
 53: #
 54: # Data structure modeling a Symm update operation
 55: #
 56: ###################################################################################################
 57: 
 58: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L59-L60 — Class `SymmOperation`

```python
 59: class SymmOperation:
 60:   #
```
**EN:** Introduces class `SymmOperation`, a descriptor for symm operations.

**CN:** 引入类 `SymmOperation`，它是 symm 操作的描述对象。

### L61-L76 — Function `__init__`

```python
 61:   def __init__(self, symm_kind, arch, tile_description, A, B, C, element_epilogue, \
 62:       epilogue_functor = EpilogueFunctor.LinearCombination, swizzling_functor = SwizzlingFunctor.Identity8, \
 63:       blas_mode = BlasMode.symmetric):
 64: 
 65:     self.blas_mode = blas_mode
 66:     self.operation_kind = OperationKind.Symm
 67:     self.arch = arch
 68:     self.tile_description = tile_description
 69:     self.symm_kind = symm_kind
 70:     # tensor A and B have same data type and layout
 71:     self.A = A
 72:     self.B = B
 73:     self.C = C
 74:     self.element_epilogue = element_epilogue
 75:     self.epilogue_functor = epilogue_functor
 76:     self.swizzling_functor = swizzling_functor
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L77-L78 — Comments

```python
 77: 
 78:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L79-L86 — Function `is_complex`

```python
 79:   def is_complex(self):
 80:     complex_operators = [
 81:       MathOperation.multiply_add_complex,
 82:       MathOperation.multiply_add_complex_gaussian,
 83:       MathOperation.multiply_add_complex_fast_f32
 84:     ]
 85:     return self.tile_description.math_instruction.math_operation in complex_operators
 86:     return False
```
**EN:** Defines `is_complex()`, a predicate that checks whether complex.

**CN:** 定义 `is_complex()`，用于判断是否满足 complex 这一条件。

### L87-L88 — Comments

```python
 87: 
 88:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L89-L90 — Function `is_mixed_input`

```python
 89:   def is_mixed_input(self):
 90:     return self.A.element != self.B.element
```
**EN:** Defines `is_mixed_input()`, a predicate that checks whether mixed input.

**CN:** 定义 `is_mixed_input()`，用于判断是否满足 mixed input 这一条件。

### L91-L92 — Comments

```python
 91: 
 92:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L93-L94 — Function `is_planar_complex`

```python
 93:   def is_planar_complex(self):
 94:     return False
```
**EN:** Defines `is_planar_complex()`, a predicate that checks whether planar complex.

**CN:** 定义 `is_planar_complex()`，用于判断是否满足 planar complex 这一条件。

### L95-L96 — Comments

```python
 95: 
 96:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L97-L103 — Function `accumulator_type`

```python
 97:   def accumulator_type(self):
 98:     accum = self.tile_description.math_instruction.element_accumulator
 99: 
100:     if self.is_complex():
101:       return get_complex_from_real(accum)
102: 
103:     return accum
```
**EN:** Defines `accumulator_type()`, which implements the accumulator type logic. Key helper calls include is_complex, get_complex_from_real.

**CN:** 定义 `accumulator_type()`，用于实现 accumulator type 相关逻辑。 其中会调用的重要辅助函数包括 is_complex, get_complex_from_real。

### L104-L105 — Comments

```python
104: 
105:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L106-L109 — Function `short_math_name`

```python
106:   def short_math_name(self):
107:     if self.tile_description.math_instruction.math_operation == MathOperation.multiply_add_complex_gaussian:
108:       return "g%s" % ShortDataTypeNames[self.accumulator_type()]
109:     return ShortDataTypeNames[self.accumulator_type()]
```
**EN:** Defines `short_math_name()`, which implements the short math name logic. Key helper calls include accumulator_type.

**CN:** 定义 `short_math_name()`，用于实现 short math name 相关逻辑。 其中会调用的重要辅助函数包括 accumulator_type。

### L110-L112 — Comments

```python
110: 
111: 
112:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L113-L140 — Function `core_name`

```python
113:   def core_name(self):
114:     ''' The basic operation kind is prefixed with a letter indicating the accumulation type. '''
115: 
116:     inst_shape = ''
117:     inst_operation = ''
118:     intermediate_type = ''
119: 
120:     math_operations_map = {
121:       MathOperation.xor_popc: 'xor',
122:       MathOperation.and_popc: 'and'
123:     }
124: 
125:     if self.tile_description.math_instruction.opcode_class == OpcodeClass.TensorOp or \
126:       self.tile_description.math_instruction.opcode_class == OpcodeClass.WmmaTensorOp:
127: 
128:       math_op = self.tile_description.math_instruction.math_operation
129:       math_op_string = math_operations_map[math_op] if math_op in math_operations_map.keys() else ''
130: 
131:       inst_shape = "%d%d%d" % tuple(self.tile_description.math_instruction.instruction_shape)
132:       inst_shape += math_op_string
133: 
134:       if self.tile_description.math_instruction.element_a != self.A.element and \
135:         self.tile_description.math_instruction.element_a != self.tile_description.math_instruction.element_accumulator:
136:         intermediate_type = DataTypeNames[self.tile_description.math_instruction.element_a]
137: 
138:     operation_name = 'symm' if self.blas_mode == BlasMode.symmetric else 'hemm'
139: 
140:     return "%s%s%s%s" % (self.short_math_name(), inst_shape, intermediate_type, operation_name)
```
**EN:** Defines `core_name()`. The docstring says: "The basic operation kind is prefixed with a letter indicating the accumulation type."; this block implements that behavior. Key helper calls include tuple, short_math_name, keys.

**CN:** 定义 `core_name()`。文档字符串说明其用途为：“The basic operation kind is prefixed with a letter indicating the accumulation type.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 tuple, short_math_name, keys。

### L141-L142 — Comments

```python
141: 
142:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L143-L163 — Function `extended_name`

```python
143:   def extended_name(self):
144:     ''' Append data types if they differ from compute type. '''
145:     if self.is_complex():
146:       extended_name = "${core_name}"
147:     else:
148:       if self.C.element != self.tile_description.math_instruction.element_accumulator and \
149:         self.A.element != self.tile_description.math_instruction.element_accumulator:
150:         extended_name = "${element_c}_${core_name}_${element_a}"
151:       elif self.C.element == self.tile_description.math_instruction.element_accumulator and  \
152:         self.A.element != self.tile_description.math_instruction.element_accumulator:
153:         extended_name = "${core_name}_${element_a}"
154:       else:
155:         extended_name = "${core_name}"
156: 
157:     extended_name = SubstituteTemplate(extended_name, {
158:       'element_a': DataTypeNames[self.A.element],
159:       'element_c': DataTypeNames[self.C.element],
160:       'core_name': self.core_name()
161:       })
162: 
163:     return extended_name
```
**EN:** Defines `extended_name()`. The docstring says: "Append data types if they differ from compute type."; this block implements that behavior. Key helper calls include is_complex, SubstituteTemplate, core_name.

**CN:** 定义 `extended_name()`。文档字符串说明其用途为：“Append data types if they differ from compute type.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 is_complex, SubstituteTemplate, core_name。

### L164-L165 — Comments

```python
164: 
165:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L166-L171 — Function `layout_name`

```python
166:   def layout_name(self):
167:     if self.is_complex() or self.is_planar_complex():
168:       return "%s" % (
169:         ShortComplexLayoutNames[(self.A.layout, self.A.complex_transform)]
170:       )
171:     return "%s" % (ShortLayoutTypeNames[self.A.layout])
```
**EN:** Defines `layout_name()`, which implements the layout name logic. Key helper calls include is_complex, is_planar_complex.

**CN:** 定义 `layout_name()`，用于实现 layout name 相关逻辑。 其中会调用的重要辅助函数包括 is_complex, is_planar_complex。

### L172-L173 — Comments

```python
172: 
173:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L174-L175 — Function `side_mode_name`

```python
174:   def side_mode_name(self):
175:     return "%s" % (ShortSideModeNames[self.A.side_mode])
```
**EN:** Defines `side_mode_name()`, which implements the side mode name logic.

**CN:** 定义 `side_mode_name()`，用于实现 side mode name 相关逻辑。

### L176-L177 — Comments

```python
176: 
177:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L178-L179 — Function `fill_mode_name`

```python
178:   def fill_mode_name(self):
179:     return "%s" % (ShortFillModeNames[self.A.fill_mode])
```
**EN:** Defines `fill_mode_name()`, which implements the fill mode name logic.

**CN:** 定义 `fill_mode_name()`，用于实现 fill mode name 相关逻辑。

### L180-L181 — Comments

```python
180: 
181:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L182-L201 — Function `procedural_name`

```python
182:   def procedural_name(self):
183:     ''' The full procedural name indicates architecture, extended name, tile size, and layout. '''
184:     threadblock = self.tile_description.procedural_name()
185: 
186:     opcode_class_name = OpcodeClassNames[self.tile_description.math_instruction.opcode_class]
187: 
188:     alignment = self.C.alignment
189: 
190:     return SubstituteTemplate(
191:       "cutlass_${opcode_class}_${extended_name}_${threadblock}_${layout}_${side_mode}_${fill_mode}_align${alignment}",
192:       {
193:         'opcode_class': opcode_class_name,
194:         'extended_name': self.extended_name(),
195:         'threadblock': threadblock,
196:         'layout': self.layout_name(),
197:         'side_mode': self.side_mode_name(),
198:         'fill_mode': self.fill_mode_name(),
199:         'alignment': "%d" % alignment,
200:       }
201:     )
```
**EN:** Defines `procedural_name()`. The docstring says: "The full procedural name indicates architecture, extended name, tile size, and layout."; this block implements that behavior. Key helper calls include procedural_name, SubstituteTemplate, extended_name, layout_name, side_mode_name, fill_mode_name.

**CN:** 定义 `procedural_name()`。文档字符串说明其用途为：“The full procedural name indicates architecture, extended name, tile size, and layout.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 procedural_name, SubstituteTemplate, extended_name, layout_name, side_mode_name, fill_mode_name。

### L202-L203 — Comments

```python
202: 
203:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L204-L206 — Function `configuration_name`

```python
204:   def configuration_name(self):
205:     ''' The full procedural name indicates architecture, extended name, tile size, and layout. '''
206:     return self.procedural_name()
```
**EN:** Defines `configuration_name()`. The docstring says: "The full procedural name indicates architecture, extended name, tile size, and layout."; this block implements that behavior. Key helper calls include procedural_name.

**CN:** 定义 `configuration_name()`。文档字符串说明其用途为：“The full procedural name indicates architecture, extended name, tile size, and layout.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 procedural_name。

### L207-L214 — Comments

```python
207: 
208: ###################################################################################################
209: #
210: # Emits single instances of a CUTLASS device-wide operator
211: #
212: ###################################################################################################
213: 
214: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L215-L215 — Class `EmitSymmUniversalInstance`

```python
215: class EmitSymmUniversalInstance:
```
**EN:** Introduces class `EmitSymmUniversalInstance`. The docstring says: "Responsible for emitting a CUTLASS template definition"; the class body implements that role.

**CN:** 引入类 `EmitSymmUniversalInstance`。文档字符串说明其用途为：“Responsible for emitting a CUTLASS template definition”；类体负责实现这一职责。

### L216-L216 — Documentation/setup

```python
216:   ''' Responsible for emitting a CUTLASS template definition'''
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L218-L273 — Function `__init__`

```python
218:   def __init__(self):
219:     self.symm_template = """
220: // Symm operator ${operation_name}
221: using Operation_${operation_name} =
222:   typename cutlass::gemm::device::Symm<
223:     ${element_a}, ${layout_a}, ${side_mode}, ${fill_mode},
224:     ${element_b}, ${layout_b},
225:     ${element_c}, ${layout_c},
226:     ${element_accumulator},
227:     ${opcode_class},
228:     ${arch},
229:     cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
230:     cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
231:     cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
232:     ${epilogue_functor}<
233:       ${element_c},
234:       ${epilogue_vector_length},
235:       ${element_accumulator},
236:       ${element_epilogue}
237:     >,
238:     ${swizzling_functor},
239:     ${stages},
240:     ${align_a},
241:     ${align_b},
242:     ${split_k_serial},
243:     ${math_operation}
244: >;
245: """
246:     self.symm_complex_template = """
247: // Symm operator ${operation_name}
248: using Operation_${operation_name} =
249:   typename cutlass::gemm::device::Symm<
250:     ${element_a}, ${layout_a}, ${side_mode}, ${fill_mode},
251:     ${element_b}, ${layout_b},
252:     ${element_c}, ${layout_c},
253:     ${element_accumulator},
254:     ${opcode_class},
255:     ${arch},
256:     cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
257:     cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
258:     cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
259:     ${epilogue_functor}<
260:       ${element_c},
261:       ${epilogue_vector_length},
262:       ${element_accumulator},
263:       ${element_epilogue}
264:     >,
265:     ${swizzling_functor},
266:     ${stages},
267:     ${align_a},
268:     ${align_b},
269:     ${split_k_serial},
270:     ${math_operation},
271:     ${blas_mode}
272: >;
273: """
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L275-L320 — Function `emit`

```python
275:   def emit(self, operation):
276: 
277:     threadblock_shape = operation.tile_description.threadblock_shape
278: 
279:     warp_count = operation.tile_description.warp_count
280:     warp_shape = [threadblock_shape[idx] // warp_count[idx] for idx in range(3)]
281: 
282:     epilogue_vector_length = int(min(operation.C.alignment * DataTypeSize[operation.C.element], 128) / DataTypeSize[operation.C.element])
283: 
284:     values = {
285:       'operation_name': operation.procedural_name(),
286:       'element_a': DataTypeTag[operation.A.element],
287:       'layout_a': LayoutTag[operation.A.layout],
288:       'side_mode': SideModeTag[operation.A.side_mode],
289:       'fill_mode': FillModeTag[operation.A.fill_mode],
290:       'element_b': DataTypeTag[operation.B.element],
291:       'layout_b': LayoutTag[operation.B.layout],
292:       'element_c': DataTypeTag[operation.C.element],
293:       'layout_c': LayoutTag[operation.C.layout],
294:       'element_accumulator': DataTypeTag[operation.accumulator_type()],
295:       'opcode_class': OpcodeClassTag[operation.tile_description.math_instruction.opcode_class],
296:       'arch': "cutlass::arch::Sm%d" % operation.arch,
297:       'threadblock_shape_m': str(operation.tile_description.threadblock_shape[0]),
298:       'threadblock_shape_n': str(operation.tile_description.threadblock_shape[1]),
299:       'threadblock_shape_k': str(operation.tile_description.threadblock_shape[2]),
300:       'warp_shape_m': str(warp_shape[0]),
301:       'warp_shape_n': str(warp_shape[1]),
302:       'warp_shape_k': str(warp_shape[2]),
303:       'instruction_shape_m': str(operation.tile_description.math_instruction.instruction_shape[0]),
304:       'instruction_shape_n': str(operation.tile_description.math_instruction.instruction_shape[1]),
305:       'instruction_shape_k': str(operation.tile_description.math_instruction.instruction_shape[2]),
306:       'epilogue_vector_length': str(epilogue_vector_length),
307:       'element_epilogue': str(DataTypeTag[operation.element_epilogue]),
308:       'epilogue_functor': EpilogueFunctorTag[operation.epilogue_functor],
309:       'swizzling_functor': SwizzlingFunctorTag[operation.swizzling_functor],
310:       'stages': str(operation.tile_description.stages),
311:       'align_a': str(operation.A.alignment),
312:       'align_b': str(operation.B.alignment),
313:       'split_k_serial': 'false',
314:       'math_operation': MathOperationTag[operation.tile_description.math_instruction.math_operation],
315:       'blas_mode': BlasModeTag[operation.blas_mode]
316:     }
317: 
318:     symm_template = self.symm_complex_template if operation.is_complex() else self.symm_template
319: 
320:     return SubstituteTemplate(symm_template, values)
```
**EN:** Defines `emit()`, which emits . Key helper calls include int, SubstituteTemplate, procedural_name, str, is_complex, range.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 int, SubstituteTemplate, procedural_name, str, is_complex, range。

### L321-L330 — Comments

```python
321: 
322: ###################################################################################################
323: 
324: 
325: ###################################################################################################
326: #
327: # Emitters functions for all targets
328: #
329: ###################################################################################################
330: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L331-L331 — Class `EmitSymmConfigurationLibrary`

```python
331: class EmitSymmConfigurationLibrary:
```
**EN:** Introduces class `EmitSymmConfigurationLibrary`, an emitter that formats generated code for symm configuration library.

**CN:** 引入类 `EmitSymmConfigurationLibrary`，它是一个用于格式化并生成 symm configuration library 代码的 emitter。

### L332-L394 — Function `__init__`

```python
332:   def __init__(self, operation_path, configuration_name):
333:     self.configuration_name = configuration_name
334:     self.configuration_path = os.path.join(operation_path, "%s.cu" % configuration_name).replace('\\', '/')
335: 
336:     self.instance_emitter = {
337:       SymmKind.Universal: EmitSymmUniversalInstance,
338:     }
339: 
340:     self.symm_kind_wrappers = {
341:       SymmKind.Universal: 'SymmOperation',
342:     }
343: 
344:     self.instance_template = {
345:       SymmKind.Universal: """
346: ${compile_guard_start}
347:   manifest.append(new ${symm_kind}<
348:     Operation_${operation_name}
349:   >("${operation_name}"));
350: ${compile_guard_end}
351: """
352:     }
353: 
354:     self.header_template = """
355: /*
356:   Generated by symm_operation.py - Do not edit.
357: */
358: 
359: ///////////////////////////////////////////////////////////////////////////////////////////////////
360: #include "cutlass/cutlass.h"
361: #include "cutlass/library/library.h"
362: #include "cutlass/library/manifest.h"
363: 
364: #include "library_internal.h"
365: #include "symm_operation.h"
366: 
367: ///////////////////////////////////////////////////////////////////////////////////////////////////
368: 
369: """
370: 
371:     self.initialize_function_template = """
372: 
373: ///////////////////////////////////////////////////////////////////////////////////////////////////
374: 
375: namespace cutlass {
376: namespace library {
377: 
378: ///////////////////////////////////////////////////////////////////////////////////////////////////
379: 
380: void initialize_${configuration_name}(Manifest &manifest) {
381: 
382: """
383:     self.epilogue_template = """
384: 
385: }
386: 
387: ///////////////////////////////////////////////////////////////////////////////////////////////////
388: 
389: } // namespace library
390: } // namespace cutlass
391: 
392: ///////////////////////////////////////////////////////////////////////////////////////////////////
393: 
394: """
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include replace, join.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 replace, join。

### L396-L404 — Function `__enter__`

```python
396:   def __enter__(self):
397:     self.configuration_file = open(self.configuration_path, "w")
398:     self.configuration_file.write(self.header_template)
399: 
400:     self.instance_definitions = []
401:     self.instance_wrappers = []
402: 
403:     self.operations = []
404:     return self
```
**EN:** Defines `__enter__()`, which implements the enter logic. Key helper calls include open, write.

**CN:** 定义 `__enter__()`，用于实现 enter 相关逻辑。 其中会调用的重要辅助函数包括 open, write。

### L406-L421 — Function `emit`

```python
406:   def emit(self, operation):
407:     emitter = self.instance_emitter[operation.symm_kind]()
408: 
409:     self.operations.append(operation)
410: 
411:     self.instance_definitions.append(emitter.emit(operation))
412: 
413:     self.instance_wrappers.append(SubstituteTemplate(self.instance_template[operation.symm_kind], {
414:       'configuration_name': self.configuration_name,
415:       'operation_name': operation.procedural_name(),
416:       'symm_kind': self.symm_kind_wrappers[operation.symm_kind],
417:       'compile_guard_start': SubstituteTemplate(self.wmma_guard_start, {'sm_number': str(operation.arch)}) \
418:         if operation.tile_description.math_instruction.opcode_class == OpcodeClass.WmmaTensorOp else "",
419:       'compile_guard_end': "#endif" \
420:         if operation.tile_description.math_instruction.opcode_class == OpcodeClass.WmmaTensorOp else ""
421:       }))
```
**EN:** Defines `emit()`, which emits . Key helper calls include append, emit, SubstituteTemplate, procedural_name, str.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 append, emit, SubstituteTemplate, procedural_name, str。

### L423-L438 — Function `__exit__`

```python
423:   def __exit__(self, exception_type, exception_value, traceback):
424: 
425:     # Write instance definitions in top-level namespace
426:     for instance_definition in self.instance_definitions:
427:       self.configuration_file.write(instance_definition)
428: 
429:     # Add wrapper objects within initialize() function
430:     self.configuration_file.write(SubstituteTemplate(self.initialize_function_template, {
431:       'configuration_name': self.configuration_name
432:       }))
433: 
434:     for instance_wrapper in self.instance_wrappers:
435:       self.configuration_file.write(instance_wrapper)
436: 
437:     self.configuration_file.write(self.epilogue_template)
438:     self.configuration_file.close()
```
**EN:** Defines `__exit__()`, which implements the exit logic. Key helper calls include write, close, SubstituteTemplate.

**CN:** 定义 `__exit__()`，用于实现 exit 相关逻辑。 其中会调用的重要辅助函数包括 write, close, SubstituteTemplate。

### L439-L440 — Comments

```python
439: 
440: ###################################################################################################
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

## Key Concepts / 关键概念

- **EN:** File role: Models symmetric matrix-multiply operations and emits the corresponding kernel/configuration code.
- **CN:** 文件角色：建模对称矩阵乘法操作，并生成对应的内核/配置代码。
- **EN:** Main classes: `SymmOperation, EmitSymmUniversalInstance, EmitSymmConfigurationLibrary`
- **CN:** 主要类：`SymmOperation, EmitSymmUniversalInstance, EmitSymmConfigurationLibrary`

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: `enum, functools, operator, os.path, shutil, builtins`
- **CN:** 标准库依赖：`enum, functools, operator, os.path, shutil, builtins`
- **EN:** Internal CUTLASS dependencies: `cutlass_library.library, library`
- **CN:** CUTLASS 内部依赖：`cutlass_library.library, library`
- **EN:** External/non-stdlib dependencies: `None explicitly imported`
- **CN:** 外部/非标准库依赖：`未显式导入`
