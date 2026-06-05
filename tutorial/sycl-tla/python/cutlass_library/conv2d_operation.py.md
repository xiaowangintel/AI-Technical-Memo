# conv2d_operation.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/cutlass_library/conv2d_operation.py`
- **EN:** Describes Conv2d operations and emits the C++ kernel/configuration code used by the CUTLASS library generator.
- **CN:** 描述 Conv2d 操作，并生成 CUTLASS 库生成器所需的 C++ 内核与配置代码。

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

### L33-L53 — Imports and setup

```python
 33: """
 34: Utilities for emitting Conv2d kernels
 35: """
 36: 
 37: import enum
 38: import logging
 39: import os.path
 40: import shutil
 41: from string import Template
 42: 
 43: try:
 44:   import builtins
 45:   if hasattr(builtins, "CUTLASS_IGNORE_PACKAGE") and CUTLASS_IGNORE_PACKAGE == True:
 46:     raise ImportError("Disabling attempt to import cutlass_library")
 47:   from cutlass_library.library import *
 48:   from cutlass_library.conv3x_emitter import EmitConv3xInstance, EmitConv3xIncludes
 49: except ImportError:
 50:   from library import *
 51:   from conv3x_emitter import EmitConv3xInstance, EmitConv3xIncludes
 52: 
 53: _LOGGER = logging.getLogger(__name__)
```
**EN:** Imports and/or re-exports modules such as enum, logging, os.path, shutil, string, Template so later code can reuse shared definitions.

**CN:** 导入或重新导出诸如 enum, logging, os.path, shutil, string, Template 等模块，使后续代码可以复用共享定义。

### L54-L57 — Comments

```python
 54: 
 55: ###################################################################################################
 56: 
 57: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L58-L59 — Class `Conv2dOperation`

```python
 58: class Conv2dOperation:
 59:   #
```
**EN:** Introduces class `Conv2dOperation`, a descriptor for conv2d operations.

**CN:** 引入类 `Conv2dOperation`，它是 conv2d 操作的描述对象。

### L60-L76 — Function `__init__`

```python
 60:   def __init__(self, conv_kind, iterator_algorithm, arch, tile_description, A, B, C, element_epilogue, \
 61:     stride_support, epilogue_functor = EpilogueFunctor.LinearCombination, swizzling_functor = SwizzlingFunctor.Identity1, \
 62:     group_mode = GroupMode.NoneGroup):
 63: 
 64:     self.operation_kind = OperationKind.Conv2d
 65:     self.arch = arch
 66:     self.tile_description = tile_description
 67:     self.conv_kind = conv_kind
 68:     self.A = A
 69:     self.B = B
 70:     self.C = C
 71:     self.element_epilogue = element_epilogue
 72:     self.epilogue_functor = epilogue_functor
 73:     self.iterator_algorithm = iterator_algorithm
 74:     self.stride_support = stride_support
 75:     self.swizzling_functor = swizzling_functor
 76:     self.group_mode = group_mode
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L77-L77 — Comments

```python
 77:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L78-L83 — Function `is_complex`

```python
 78:   def is_complex(self):
 79:     complex_operators = [
 80:       MathOperation.multiply_add_complex,
 81:       MathOperation.multiply_add_complex_gaussian
 82:       ]
 83:     return self.tile_description.math_instruction.math_operation in complex_operators
```
**EN:** Defines `is_complex()`, a predicate that checks whether complex.

**CN:** 定义 `is_complex()`，用于判断是否满足 complex 这一条件。

### L84-L85 — Comments

```python
 84: 
 85:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L86-L87 — Function `is_mixed_input`

```python
 86:   def is_mixed_input(self):
 87:     return self.A.element != self.B.element
```
**EN:** Defines `is_mixed_input()`, a predicate that checks whether mixed input.

**CN:** 定义 `is_mixed_input()`，用于判断是否满足 mixed input 这一条件。

### L88-L89 — Comments

```python
 88: 
 89:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L90-L96 — Function `accumulator_type`

```python
 90:   def accumulator_type(self):
 91:     accum = self.tile_description.math_instruction.element_accumulator
 92: 
 93:     if self.is_complex():
 94:       return get_complex_from_real(accum)
 95: 
 96:     return accum
```
**EN:** Defines `accumulator_type()`, which implements the accumulator type logic. Key helper calls include is_complex, get_complex_from_real.

**CN:** 定义 `accumulator_type()`，用于实现 accumulator type 相关逻辑。 其中会调用的重要辅助函数包括 is_complex, get_complex_from_real。

### L97-L98 — Comments

```python
 97: 
 98:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L99-L113 — Function `core_name`

```python
 99:   def core_name(self):
100:     ''' The basic operation kind is prefixed with a letter indicating the accumulation type. '''
101: 
102:     intermediate_type = ''
103: 
104:     if self.tile_description.math_instruction.opcode_class == OpcodeClass.TensorOp:
105:       inst_shape = "%d%d%d" % tuple(self.tile_description.math_instruction.instruction_shape)
106:       if self.tile_description.math_instruction.element_a != self.A.element and \
107:         self.tile_description.math_instruction.element_a != self.accumulator_type():
108:         intermediate_type = DataTypeNames[self.tile_description.math_instruction.element_a]
109:     else:
110:       inst_shape = ''
111: 
112:     return "%s%s%s%s_%s" % (ShortDataTypeNames[self.accumulator_type()], \
113:       inst_shape, intermediate_type, ConvKindNames[self.conv_kind], IteratorAlgorithmNames[self.iterator_algorithm])
```
**EN:** Defines `core_name()`. The docstring says: "The basic operation kind is prefixed with a letter indicating the accumulation type."; this block implements that behavior. Key helper calls include tuple, accumulator_type.

**CN:** 定义 `core_name()`。文档字符串说明其用途为：“The basic operation kind is prefixed with a letter indicating the accumulation type.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 tuple, accumulator_type。

### L114-L115 — Comments

```python
114: 
115:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L116-L133 — Function `extended_name`

```python
116:   def extended_name(self):
117:     ''' Append data types if they differ from compute type. '''
118:     if self.C.element != self.tile_description.math_instruction.element_accumulator and \
119:       self.A.element != self.tile_description.math_instruction.element_accumulator:
120:       extended_name = "${element_c}_${core_name}_${element_a}"
121:     elif self.C.element == self.tile_description.math_instruction.element_accumulator and  \
122:       self.A.element != self.tile_description.math_instruction.element_accumulator:
123:       extended_name = "${core_name}_${element_a}"
124:     else:
125:       extended_name = "${core_name}"
126: 
127:     extended_name = SubstituteTemplate(extended_name, {
128:       'element_a': DataTypeNames[self.A.element],
129:       'element_c': DataTypeNames[self.C.element],
130:       'core_name': self.core_name()
131:       })
132: 
133:     return extended_name
```
**EN:** Defines `extended_name()`. The docstring says: "Append data types if they differ from compute type."; this block implements that behavior. Key helper calls include SubstituteTemplate, core_name.

**CN:** 定义 `extended_name()`。文档字符串说明其用途为：“Append data types if they differ from compute type.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 SubstituteTemplate, core_name。

### L134-L135 — Comments

```python
134: 
135:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L136-L137 — Function `layout_name`

```python
136:   def layout_name(self):
137:     return "%s" % (ShortLayoutTypeNames[self.A.layout])
```
**EN:** Defines `layout_name()`, which implements the layout name logic.

**CN:** 定义 `layout_name()`，用于实现 layout name 相关逻辑。

### L138-L139 — Comments

```python
138: 
139:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L140-L168 — Function `configuration_name`

```python
140:   def configuration_name(self):
141:     ''' The full procedural name indicates architecture, extended name, tile size, and layout. '''
142: 
143:     opcode_class_name = OpcodeClassNames[self.tile_description.math_instruction.opcode_class]
144: 
145:     threadblock = self.tile_description.procedural_name()
146: 
147:     # grouped conv
148:     if self.group_mode != GroupMode.NoneGroup:
149:       group_conv_name = f"{GroupModeNames[self.group_mode]}_"
150:     else:
151:       group_conv_name = ""
152: 
153:     if self.stride_support == StrideSupport.Unity and self.conv_kind == ConvKind.Dgrad:
154:       configuration_name = "cutlass_${opcode_class}_${extended_name}_${threadblock}_${layout}_unity_stride_${group_conv_name}align${alignment}"
155:     else:
156:       configuration_name = "cutlass_${opcode_class}_${extended_name}_${threadblock}_${layout}_${group_conv_name}align${alignment}"
157: 
158:     return SubstituteTemplate(
159:       configuration_name,
160:       {
161:         'opcode_class': opcode_class_name,
162:         'extended_name': self.extended_name(),
163:         'threadblock': threadblock,
164:         'layout': self.layout_name(),
165:         'alignment': "%d" % self.A.alignment,
166:         'group_conv_name': group_conv_name
167:       }
168:     )
```
**EN:** Defines `configuration_name()`. The docstring says: "The full procedural name indicates architecture, extended name, tile size, and layout."; this block implements that behavior. Key helper calls include procedural_name, SubstituteTemplate, extended_name, layout_name.

**CN:** 定义 `configuration_name()`。文档字符串说明其用途为：“The full procedural name indicates architecture, extended name, tile size, and layout.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 procedural_name, SubstituteTemplate, extended_name, layout_name。

### L169-L170 — Comments

```python
169: 
170:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L171-L173 — Function `procedural_name`

```python
171:   def procedural_name(self):
172:     ''' The full procedural name indicates architecture, extended name, tile size, and layout. '''
173:     return self.configuration_name()
```
**EN:** Defines `procedural_name()`. The docstring says: "The full procedural name indicates architecture, extended name, tile size, and layout."; this block implements that behavior. Key helper calls include configuration_name.

**CN:** 定义 `procedural_name()`。文档字符串说明其用途为：“The full procedural name indicates architecture, extended name, tile size, and layout.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 configuration_name。

### L174-L180 — Comments

```python
174: 
175: ###################################################################################################
176: #
177: # Emits single instances of a CUTLASS device-wide operator
178: #
179: ###################################################################################################
180: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L181-L181 — Class `EmitConv2dInstance`

```python
181: class EmitConv2dInstance:
```
**EN:** Introduces class `EmitConv2dInstance`, an emitter that formats generated code for conv2d instance.

**CN:** 引入类 `EmitConv2dInstance`，它是一个用于格式化并生成 conv2d instance 代码的 emitter。

### L182-L286 — Function `__init__`

```python
182:   def __init__(self):
183:     # Emitter for CUTLASS 3 convolution operations
184:     self.conv3x_emitter = EmitConv3xInstance()
185:     self.template = """
186:   // Conv2d${conv_kind_name} ${iterator_algorithm_name} kernel instance "${operation_name}"
187:   using ${operation_name}_base =
188:   typename cutlass::conv::kernel::DefaultConv2d${conv_kind_name}<
189:     ${element_a},
190:     ${layout_a},
191:     ${element_b},
192:     ${layout_b},
193:     ${element_c},
194:     ${layout_c},
195:     ${element_accumulator},
196:     ${opcode_class},
197:     ${arch},
198:     cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
199:     cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k} >,
200:     cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
201:     ${epilogue_functor}<
202:       ${element_c},
203:       ${epilogue_vector_length},
204:       ${element_accumulator},
205:       ${element_epilogue}
206:     >,
207:     ${swizzling_functor}, // cutlass::gemm::threadblock::GemmSplitKIdentityThreadblockSwizzle<>,
208:     ${stages},
209:     ${math_operator},
210:     ${iterator_algorithm},
211:     ${stride_support},
212:     ${align_a},
213:     ${align_b}
214:   >::Kernel;
215: """
216:     self.template_group_conv = """
217:   // Conv2d${conv_kind_name} ${iterator_algorithm_name} kernel instance "${operation_name}"
218:   using ${operation_name}_base =
219:   typename cutlass::conv::kernel::DefaultConv2dGroup${conv_kind_name}<
220:     ${element_a},
221:     ${layout_a},
222:     ${element_b},
223:     ${layout_b},
224:     ${element_c},
225:     ${layout_c},
226:     ${element_accumulator},
227:     ${opcode_class},
228:     ${arch},
229:     cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
230:     cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k} >,
231:     cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
232:     ${epilogue_functor}<
233:       ${element_c},
234:       ${epilogue_vector_length},
235:       ${element_accumulator},
236:       ${element_epilogue}
237:     >,
238:     ${swizzling_functor}, // cutlass::gemm::threadblock::GemmSplitKIdentityThreadblockSwizzle<>,
239:     ${stages},
240:     ${math_operator},
241:     ${group_mode},
242:     ${iterator_algorithm},
243:     ${stride_support},
244:     ${align_a},
245:     ${align_b}
246:   >::Kernel;
247: """
248:     self.template_depthwise_direct_conv = """
249:   // Conv2d${conv_kind_name} ${iterator_algorithm_name} kernel instance "${operation_name}"
250:   using ${operation_name}_base =
251:   typename cutlass::conv::kernel::DefaultDepthwiseDirect2dConv${conv_kind_name}<
252:     ${element_a},
253:     ${layout_a},
254:     ${element_b},
255:     ${layout_b},
256:     ${element_c},
257:     ${layout_c},
258:     ${element_accumulator},
259:     ${opcode_class},
260:     ${arch},
261:     cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
262:     cutlass::conv::TensorNHWCShape<${threadblock_output_shape_n}, ${threadblock_output_shape_p}, ${threadblock_output_shape_q}, ${groups_per_cta}>,
263:     cutlass::MatrixShape<${filter_shape_r}, ${filter_shape_s}>,
264:     cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
265:     cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
266:     ${epilogue_functor}<
267:       ${element_c},
268:       ${epilogue_vector_length},
269:       ${element_accumulator},
270:       ${element_epilogue},
271:       cutlass::epilogue::thread::ScaleType::OnlyAlphaScaling
272:     >,
273: 
274:     cutlass::conv::threadblock::DepthwiseDirect2dConvIdentityThreadblockSwizzle<
275:           1,
276:           ${threadblock_output_shape_n},
277:           ${threadblock_output_shape_p},
278:           ${threadblock_output_shape_q}>,
279:     ${stages},
280:     ${math_operator},
281:     ${iterator_algorithm},
282:     ${stride_support},
283:     cutlass::MatrixShape<${stride_r}, ${stride_s}>,
284:     cutlass::MatrixShape<${dilation_r}, ${dilation_s}>
285:   >::Kernel;
286: """
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include EmitConv3xInstance.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 EmitConv3xInstance。

### L288-L289 — Function `arch_number_to_type`

```python
288:   def arch_number_to_type(self, arch: int):
289:     return f"cutlass::arch::Sm{arch}"
```
**EN:** Defines `arch_number_to_type()`, which implements the arch number to type logic.

**CN:** 定义 `arch_number_to_type()`，用于实现 arch number to type 相关逻辑。

### L291-L369 — Function `emit`

```python
291:   def emit(self, operation):
292:     _LOGGER.debug("*** EmitConv2dInstance::emit")
293:     _LOGGER.debug("***   operation: procedural_name()=" + operation.procedural_name())
294: 
295:     if hasattr(operation, 'is_3x') and operation.is_3x:
296:       _LOGGER.debug("***   CUTLASS 3 operation")
297:       return self.conv3x_emitter.emit(operation)
298: 
299:     _LOGGER.debug("***   CUTLASS 2 operation")
300: 
301:     warp_shape = [int(operation.tile_description.threadblock_shape[idx] / operation.tile_description.warp_count[idx]) for idx in range(3)]
302: 
303:     epilogue_vector_length = int(min(operation.C.alignment * DataTypeSize[operation.C.element], 128) / DataTypeSize[operation.C.element])
304: 
305:     values = {
306:       'operation_name': operation.procedural_name(),
307:       'conv_kind': ConvKindTag[operation.conv_kind],
308:       'conv_kind_name': ConvKindNames[operation.conv_kind].capitalize(),
309:       'element_a': DataTypeTag[operation.A.element],
310:       'layout_a': LayoutTag[operation.A.layout],
311:       'element_b': DataTypeTag[operation.B.element],
312:       'layout_b': LayoutTag[operation.B.layout],
313:       'element_c': DataTypeTag[operation.C.element],
314:       'layout_c': LayoutTag[operation.C.layout],
315:       'element_accumulator': DataTypeTag[operation.accumulator_type()],
316:       'opcode_class': OpcodeClassTag[operation.tile_description.math_instruction.opcode_class],
317:       'arch': "cutlass::arch::Sm%d" % operation.arch,
318:       'threadblock_shape_m': str(operation.tile_description.threadblock_shape[0]),
319:       'threadblock_shape_n': str(operation.tile_description.threadblock_shape[1]),
320:       'threadblock_shape_k': str(operation.tile_description.threadblock_shape[2]),
321:       'warp_shape_m': str(warp_shape[0]),
322:       'warp_shape_n': str(warp_shape[1]),
323:       'warp_shape_k': str(warp_shape[2]),
324:       'instruction_shape_m': str(operation.tile_description.math_instruction.instruction_shape[0]),
325:       'instruction_shape_n': str(operation.tile_description.math_instruction.instruction_shape[1]),
326:       'instruction_shape_k': str(operation.tile_description.math_instruction.instruction_shape[2]),
327:       'epilogue_vector_length': str(epilogue_vector_length),
328:       'epilogue_functor': EpilogueFunctorTag[operation.epilogue_functor],
329:       'element_epilogue': str(DataTypeTag[operation.element_epilogue]),
330:       'swizzling_functor': SwizzlingFunctorTag[operation.swizzling_functor],
331:       'stages': str(operation.tile_description.stages),
332:       'iterator_algorithm': IteratorAlgorithmTag[operation.iterator_algorithm],
333:       'iterator_algorithm_name': IteratorAlgorithmNames[operation.iterator_algorithm].capitalize(),
334:       'stride_support': StrideSupportTag[operation.stride_support],
335:       'math_operator': 'cutlass::arch::OpMultiplyAddComplex' if operation.is_complex() else \
336:       MathOperationTag[operation.tile_description.math_instruction.math_operation],
337:       'align_a': str(operation.A.alignment),
338:       'align_b': str(operation.B.alignment),
339:     }
340: 
341:     if operation.group_mode == GroupMode.NoneGroup:
342:       _LOGGER.debug("***   group_mode=NoneGroup")
343:       return SubstituteTemplate(self.template, values)
344: 
345:     elif operation.group_mode == GroupMode.Depthwise:
346:       _LOGGER.debug("***   group_mode=Depthwise")
347:       values['group_mode'] = GroupModeTag[operation.group_mode]
348:       # Setup other template params
349:       values['threadblock_output_shape_n'] = str(operation.tile_description.threadblock_output_shape[0])
350:       values['threadblock_output_shape_p'] = str(operation.tile_description.threadblock_output_shape[1])
351:       values['threadblock_output_shape_q'] = str(operation.tile_description.threadblock_output_shape[2])
352: 
353:       values['groups_per_cta'] = str(operation.tile_description.threadblock_output_shape[3])
354: 
355:       values['filter_shape_r'] = str(operation.tile_description.filter_shape[0])
356:       values['filter_shape_s'] = str(operation.tile_description.filter_shape[1])
357: 
358:       values['stride_r'] = str(operation.tile_description.stride[0])
359:       values['stride_s'] = str(operation.tile_description.stride[1])
360: 
361:       values['dilation_r'] = str(operation.tile_description.dilation[0])
362:       values['dilation_s'] = str(operation.tile_description.dilation[1])
363: 
364:       return SubstituteTemplate(self.template_depthwise_direct_conv, values)
365: 
366:     else:
367:       _LOGGER.debug("***   group_mode=" + GroupModeTag[operation.group_mode])
368:       values['group_mode'] = GroupModeTag[operation.group_mode]
369:       return SubstituteTemplate(self.template_group_conv, values)
```
**EN:** Defines `emit()`, which emits . Key helper calls include debug, int, hasattr, emit, procedural_name, capitalize.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 debug, int, hasattr, emit, procedural_name, capitalize。

### L370-L377 — Comments

```python
370: 
371: ###################################################################################################
372: #
373: # Generator functions for all layouts
374: #
375: ###################################################################################################
376: 
377: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L378-L396 — Function `GenerateConv2dTensorOp`

```python
378: def GenerateConv2dTensorOp(manifest, tile_descriptions, min_cc, align = 128):
379:   _LOGGER.debug("*** GenerateConv2dTensorOp")
380: 
381:   for tile in tile_descriptions:
382:     for conv_kind in [ConvKind.Fprop, ConvKind.Dgrad, ConvKind.Wgrad]:
383: 
384:       if conv_kind == ConvKind.Fprop or (tile.math_instruction.element_accumulator in [DataType.f16, DataType.f32]):
385: 
386:         #
387:         output_types = [tile.math_instruction.element_a, tile.math_instruction.element_accumulator] \
388:           if DataTypeSize[tile.math_instruction.element_accumulator] == 32 \
389:           else [tile.math_instruction.element_accumulator,]
390: 
391:         for output_type in output_types:
392:           A = TensorDescription(tile.math_instruction.element_a, LayoutType.TensorNHWC, int(align / DataTypeSize[tile.math_instruction.element_a]))
393:           B = TensorDescription(tile.math_instruction.element_b, LayoutType.TensorNHWC, int(align / DataTypeSize[tile.math_instruction.element_b]))
394:           C = TensorDescription(output_type,  LayoutType.TensorNHWC, max(1, int(align / DataTypeSize[output_type])))
395: 
396:           manifest.append(Conv2dOperation(conv_kind, min_cc, tile, A, B, C, tile.math_instruction.element_accumulator))
```
**EN:** Defines `GenerateConv2dTensorOp()`, which generates conv2d tensor op. Key helper calls include debug, TensorDescription, append, int, max, Conv2dOperation.

**CN:** 定义 `GenerateConv2dTensorOp()`，用于生成 conv2d tensor op。 其中会调用的重要辅助函数包括 debug, TensorDescription, append, int, max, Conv2dOperation。

### L398-L398 — Class `EmitConv2dIncludes`

```python
398: class EmitConv2dIncludes:
```
**EN:** Introduces class `EmitConv2dIncludes`. The docstring says: "Emit includes that are specific to the operation."; the class body implements that role.

**CN:** 引入类 `EmitConv2dIncludes`。文档字符串说明其用途为：“Emit includes that are specific to the operation.”；类体负责实现这一职责。

### L399-L399 — Documentation/setup

```python
399:   '''Emit includes that are specific to the operation.'''
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L401-L403 — Function `__init__`

```python
401:   def __init__(self):
402:     self.includes = ['conv2d_operation.h']
403:     self.emitter_3x = EmitConv3xIncludes()
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include EmitConv3xIncludes.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 EmitConv3xIncludes。

### L405-L407 — Function `operation_is_3x`

```python
405:   def operation_is_3x(self, operation) -> bool:
406:     """Whether operation is a CUTLASS 3 convolution (as opposed to CUTLASS 2)"""
407:     return hasattr(operation, 'is_3x') and operation.is_3x
```
**EN:** Defines `operation_is_3x()`. The docstring says: "Whether operation is a CUTLASS 3 convolution (as opposed to CUTLASS 2)"; this block implements that behavior. Key helper calls include hasattr.

**CN:** 定义 `operation_is_3x()`。文档字符串说明其用途为：“Whether operation is a CUTLASS 3 convolution (as opposed to CUTLASS 2)”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 hasattr。

### L409-L414 — Function `emit`

```python
409:   def emit(self, operation) -> str:
410:     if self.operation_is_3x(operation):
411:       return self.emitter_3x.emit(operation)
412: 
413:     return '\n'.join(f"#include \"{incl}\"" for incl in self.includes) + \
414:       "\n\n///////////////////////////////////////////////////////////////////////////////////////////////////"
```
**EN:** Defines `emit()`, which emits . Key helper calls include operation_is_3x, emit, join.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 operation_is_3x, emit, join。

### L415-L421 — Comments

```python
415: 
416: ###################################################################################################
417: #
418: # Emitters functions for all targets
419: #
420: ###################################################################################################
421: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L422-L422 — Class `EmitConv2dConfigurationLibrary`

```python
422: class EmitConv2dConfigurationLibrary:
```
**EN:** Introduces class `EmitConv2dConfigurationLibrary`, an emitter that formats generated code for conv2d configuration library.

**CN:** 引入类 `EmitConv2dConfigurationLibrary`，它是一个用于格式化并生成 conv2d configuration library 代码的 emitter。

### L423-L487 — Function `__init__`

```python
423:   def __init__(self, operation_path, configuration_name):
424:     self.configuration_name = configuration_name
425:     self.configuration_path = os.path.join(operation_path, "%s.cu" % configuration_name)
426: 
427:     self.instance_emitter = EmitConv2dInstance()
428:     self.includes_emitter = EmitConv2dIncludes()
429: 
430:     self.header_template = """
431: /*
432:   Generated by conv2d_operation.py - Do not edit.
433: */
434: 
435: ///////////////////////////////////////////////////////////////////////////////////////////////////
436: 
437: #include "cutlass/cutlass.h"
438: #include "cutlass/library/library.h"
439: #include "cutlass/library/manifest.h"
440: 
441: #include "library_internal.h"
442: """
443: 
444:     self.instance_template = """
445: ${stub_begin}
446: ${operation_instance}
447: // Derived class
448: struct ${operation_name} :
449:   public ${operation_name}_base { };
450: ${stub_end}
451: ///////////////////////////////////////////////////////////////////////////////////////////////////
452: 
453: """
454: 
455:     self.configuration_header = """
456: 
457: namespace cutlass {
458: namespace library {
459: 
460: // Initialize all instances
461: void initialize_${configuration_name}(Manifest &manifest) {
462: """
463: 
464:     self.configuration_instance = """${stub_begin}
465:   using Operation_${operation_name} = cutlass::conv::device::${kernel_name}<
466:     ${operation_name}>;
467: 
468:   manifest.append(new cutlass::library::${operation_wrapper}<
469:       Operation_${operation_name}
470:     >(
471:       "${operation_name}"
472:     ));
473: ${stub_end}
474: """
475: 
476:     self.configuration_epilogue = "}\n"
477: 
478:     self.epilogue_template = """
479: 
480: ///////////////////////////////////////////////////////////////////////////////////////////////////
481: 
482: } // namespace library
483: } // namespace cutlass
484: 
485: ///////////////////////////////////////////////////////////////////////////////////////////////////
486: 
487: """
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include join, EmitConv2dInstance, EmitConv2dIncludes.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 join, EmitConv2dInstance, EmitConv2dIncludes。

### L489-L491 — Function `operation_is_3x`

```python
489:   def operation_is_3x(self, operation):
490:     """Whether operation is a CUTLASS 3 convolution (as opposed to CUTLASS 2)"""
491:     return hasattr(operation, 'is_3x') and operation.is_3x
```
**EN:** Defines `operation_is_3x()`. The docstring says: "Whether operation is a CUTLASS 3 convolution (as opposed to CUTLASS 2)"; this block implements that behavior. Key helper calls include hasattr.

**CN:** 定义 `operation_is_3x()`。文档字符串说明其用途为：“Whether operation is a CUTLASS 3 convolution (as opposed to CUTLASS 2)”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 hasattr。

### L493-L511 — Function `__enter__`

```python
493:   def __enter__(self):
494:     """
495:     Open the configuration_file, and write the "header" C++ code to it.
496: 
497:     The "header" consists of a comment (that this is generated code,
498:     so it should not be edited), and includes that are common
499:     to all kinds of kernels.
500:     """
501:     _LOGGER.debug('*** EmitConv2dConfigurationLibrary::__enter__')
502:     _LOGGER.debug('***   configuration_path (file to write): ' +
503:                   str(self.configuration_path))
504:     _LOGGER.debug('***   configuration_name: ' + self.configuration_name)
505:     self.configuration_file = open(self.configuration_path, "w")
506: 
507:     self.configuration_file.write(SubstituteTemplate(self.header_template, {
508:       'configuration_name': self.configuration_name
509:       }))
510:     self.operations = []
511:     return self
```
**EN:** Defines `__enter__()`. The docstring says: "Open the configuration_file, and write the "header" C++ code to it."; this block implements that behavior. Key helper calls include debug, open, write, SubstituteTemplate, str.

**CN:** 定义 `__enter__()`。文档字符串说明其用途为：“Open the configuration_file, and write the "header" C++ code to it.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 debug, open, write, SubstituteTemplate, str。

### L513-L550 — Function `emit`

```python
513:   def emit(self, operation):
514:     """
515:     Write three pieces of C++ code to the configuration_file
516:     (that was opened by the __enter__ method above):
517: 
518:     1. the header includes that are specific to the operation
519:        (CUTLASS 2 vs. CUTLASS 3);
520: 
521:     2. the "operation instance" (a "using" declaration ending in "_base"); and
522: 
523:     3. the "operation name" (declaration and definition of a derived class
524:        of the above operation instance).
525: 
526:     The "using" declaration turns a C++ class name, possibly namespace-qualified,
527:     possibly also with angle brackets, into a C-style, easily demangled identifier.
528:     """
529:     _LOGGER.debug('*** EmitConv2dConfigurationLibrary::emit')
530:     _LOGGER.debug('***   operation.procedural_name(): ' + operation.procedural_name())
531:     self.operations.append(operation)
532: 
533:     self.configuration_file.write(self.includes_emitter.emit(operation))
534: 
535:     stub_begin = ''
536:     stub_end = ''
537:     # It can be useful to stub (comment) out instantiations for testing.
538:     # In this case, one need only set is_stub to True.
539:     is_stub = False
540:     if is_stub:
541:       stub_begin = "// STUB for now\n#if 0"
542:       stub_end = '#endif // 0'
543: 
544:     self.configuration_file.write(Template(self.instance_template).substitute({
545:       'configuration_name': self.configuration_name,
546:       'operation_name': operation.procedural_name(),
547:       'operation_instance': self.instance_emitter.emit(operation),
548:       'stub_begin': stub_begin,
549:       'stub_end': stub_end
550:       }))
```
**EN:** Defines `emit()`. The docstring says: "Write three pieces of C++ code to the configuration_file (that was opened by the __enter__ method above): 1."; this block implements that behavior. Key helper calls include debug, append, write, emit, substitute, procedural_name.

**CN:** 定义 `emit()`。文档字符串说明其用途为：“Write three pieces of C++ code to the configuration_file (that was opened by the __enter__ method above): 1.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 debug, append, write, emit, substitute, procedural_name。

### L552-L617 — Function `__exit__`

```python
552:   def __exit__(self, exception_type, exception_value, traceback):
553:     """
554:     Write the rest of the C++ code to the configuration_file, and close the file.
555: 
556:     The "rest of the C++ code" has the following components.
557: 
558:     1. Configuration header: Open the namespace(s), and open the definition
559:        of the "initialize_${configuration_name}" registration function
560:        that registers the operation with the Manifest.
561:        ("Registration" helps turn C++ compile-time polymorphism
562:        (via template parameters) into a run-time choice of parameters.)
563: 
564:     2. Configuration instance: In the body of the registration function,
565:        make a "using" declaration Operation_${operation_name} for the
566:        operation type (which uses operation_name as its template argument).
567:        Then, tell the manifest about the operation via a "manifest.append" call.
568:        The argument of the call is a new instance of
569:        "SomethingOperation<Operation_${operation_name}>"
570:        (replace Something with a specific name).
571: 
572:     3. Configuration epilogue: Close the definition of the registration function.
573: 
574:     4. Epilogue template: Close the namespace(s).
575:     """
576: 
577:     _LOGGER.debug('*** EmitConv2dConfigurationLibrary::__exit__')
578:     _LOGGER.debug('***   configuration_path (file to write): ' +
579:                   str(self.configuration_path))
580:     _LOGGER.debug('***   configuration_name: ' + self.configuration_name)
581: 
582:     self.configuration_file.write(SubstituteTemplate(self.configuration_header, {
583:       'configuration_name': self.configuration_name
584:       }))
585: 
586:     for operation in self.operations:
587:       stub_begin = ''
588:       stub_end = ''
589:       # It can be useful to stub (comment) out instantiations for testing.
590:       # In this case, one need only set is_stub to True.
591:       is_stub = False
592:       if is_stub:
593:         stub_begin = "// STUB for now\n#if 0"
594:         stub_end = "#endif // 0"
595: 
596:       if operation.group_mode == GroupMode.Depthwise:
597:         kernel_name = 'DirectConvolution'
598:         operation_wrapper = 'DirectConv2dOperation'
599:       else:
600:         kernel_name = 'ImplicitGemmConvolution'
601:         operation_wrapper = 'Conv2dOperation'
602:       if self.operation_is_3x(operation):
603:         kernel_name = 'ConvUniversalAdapter'
604:         operation_wrapper = 'ConvOperation3x'
605: 
606:       self.configuration_file.write(SubstituteTemplate(self.configuration_instance, {
607:         'configuration_name': self.configuration_name,
608:         'operation_name': operation.procedural_name(),
609:         'kernel_name': kernel_name,
610:         'operation_wrapper': operation_wrapper,
611:         'stub_begin': stub_begin,
612:         'stub_end': stub_end
613:       }))
614: 
615:     self.configuration_file.write(self.configuration_epilogue)
616:     self.configuration_file.write(self.epilogue_template)
617:     self.configuration_file.close()
```
**EN:** Defines `__exit__()`. The docstring says: "Write the rest of the C++ code to the configuration_file, and close the file."; this block implements that behavior. Key helper calls include debug, write, close, SubstituteTemplate, operation_is_3x, str.

**CN:** 定义 `__exit__()`。文档字符串说明其用途为：“Write the rest of the C++ code to the configuration_file, and close the file.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 debug, write, close, SubstituteTemplate, operation_is_3x, str。

### L618-L621 — Comments

```python
618: 
619: 
620: ###################################################################################################
621: ###################################################################################################
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

## Key Concepts / 关键概念

- **EN:** File role: Describes Conv2d operations and emits the C++ kernel/configuration code used by the CUTLASS library generator.
- **CN:** 文件角色：描述 Conv2d 操作，并生成 CUTLASS 库生成器所需的 C++ 内核与配置代码。
- **EN:** Main classes: `Conv2dOperation, EmitConv2dInstance, EmitConv2dIncludes, EmitConv2dConfigurationLibrary`
- **CN:** 主要类：`Conv2dOperation, EmitConv2dInstance, EmitConv2dIncludes, EmitConv2dConfigurationLibrary`
- **EN:** Main functions: `GenerateConv2dTensorOp`
- **CN:** 主要函数：`GenerateConv2dTensorOp`
- **EN:** Important constants/tables: `_LOGGER`
- **CN:** 重要常量/表：`_LOGGER`

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: `enum, logging, os.path, shutil, string, builtins`
- **CN:** 标准库依赖：`enum, logging, os.path, shutil, string, builtins`
- **EN:** Internal CUTLASS dependencies: `cutlass_library.library, cutlass_library.conv3x_emitter, library, conv3x_emitter`
- **CN:** CUTLASS 内部依赖：`cutlass_library.library, cutlass_library.conv3x_emitter, library, conv3x_emitter`
- **EN:** External/non-stdlib dependencies: `None explicitly imported`
- **CN:** 外部/非标准库依赖：`未显式导入`
