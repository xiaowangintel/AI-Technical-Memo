# gemm_operation.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/cutlass_library/gemm_operation.py`
- **EN:** Defines GEMM-family operation descriptors and multiple emitters for different CUTLASS GEMM kernel variants.
- **CN:** 定义 GEMM 系列操作描述对象，以及面向不同 CUTLASS GEMM 内核变体的多种生成器。

## Line-by-Line Analysis / 逐行分析

### L1-L33 — Header comments

```python
   1: #################################################################################################
   2: #
   3: # Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
   4: # Copyright (C) 2025 Intel Corporation, All rights reserved.
   5: # SPDX-License-Identifier: BSD-3-Clause
   6: #
   7: # Redistribution and use in source and binary forms, with or without
   8: # modification, are permitted provided that the following conditions are met:
   9: #
  10: # 1. Redistributions of source code must retain the above copyright notice, this
  11: # list of conditions and the following disclaimer.
  12: #
  13: # 2. Redistributions in binary form must reproduce the above copyright notice,
  14: # this list of conditions and the following disclaimer in the documentation
  15: # and/or other materials provided with the distribution.
  16: #
  17: # 3. Neither the name of the copyright holder nor the names of its
  18: # contributors may be used to endorse or promote products derived from
  19: # this software without specific prior written permission.
  20: #
  21: # THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  22: # AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  23: # IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  24: # DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  25: # FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  26: # DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  27: # SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  28: # CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  29: # OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  30: # OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  31: #
  32: #################################################################################################
  33: 
```
**EN:** Contains the license banner and file-level header comments.

**CN:** 包含许可证横幅以及文件级头部注释。

### L34-L62 — Imports and setup

```python
  34: """
  35: Utilities for emitting GEMM kernels
  36: """
  37: 
  38: import collections
  39: import enum
  40: import functools
  41: import logging
  42: import operator
  43: import os.path
  44: import shutil
  45: 
  46: try:
  47:   import builtins
  48:   if hasattr(builtins, "CUTLASS_IGNORE_PACKAGE") and CUTLASS_IGNORE_PACKAGE == True:
  49:     raise ImportError("Disabling attempt to import cutlass_library")
  50:   from cutlass_library.library import *
  51:   from cutlass_library.arch_constants import (
  52:     INTEL_XE_ARCH_MIN, INTEL_XE_ARCH_MAX, CUDA_ARCH_MIN,
  53:     INTEL_XE12, INTEL_XE20, INTEL_XE35
  54:   )
  55: except ImportError:
  56:   from library import *
  57:   from arch_constants import (
  58:     INTEL_XE_ARCH_MIN, INTEL_XE_ARCH_MAX, CUDA_ARCH_MIN,
  59:     INTEL_XE12, INTEL_XE20, INTEL_XE35
  60:   )
  61: 
  62: _LOGGER = logging.getLogger(__name__)
```
**EN:** Imports and/or re-exports modules such as collections, enum, functools, logging, operator, os.path, shutil so later code can reuse shared definitions.

**CN:** 导入或重新导出诸如 collections, enum, functools, logging, operator, os.path, shutil 等模块，使后续代码可以复用共享定义。

### L63-L70 — Comments

```python
  63: 
  64: ###################################################################################################
  65: #
  66: # Data structure modeling a GEMM operation
  67: #
  68: ###################################################################################################
  69: 
  70: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L71-L72 — Class `GemmOperation`

```python
  71: class GemmOperation:
  72:   #
```
**EN:** Introduces class `GemmOperation`, a descriptor for gemm operations.

**CN:** 引入类 `GemmOperation`，它是 gemm 操作的描述对象。

### L73-L134 — Function `__init__`

```python
  73:   def __init__(self, gemm_kind, arch, tile_description, A, B, C, element_epilogue, \
  74:       epilogue_functor = EpilogueFunctor.LinearCombination, swizzling_functor = SwizzlingFunctor.Identity8, D = None,
  75:       kernel_schedule = KernelScheduleType.ScheduleAuto, epilogue_schedule = EpilogueScheduleType.ScheduleAuto,
  76:       tile_scheduler = TileSchedulerType.Default, mixed_input_mode = None, mixed_input_shuffle = False,
  77:       ScaleFactorA = None, ScaleFactorB = None, ScaleFactorD = None, 
  78:       ScaleFactorMVecSize = None, ScaleFactorNVecSize = None, ScaleFactorKVecSize = None):
  79: 
  80:     kinds_3x = {
  81:       GemmKind.Universal3x,
  82:       GemmKind.SparseUniversal3x,
  83:       GemmKind.BlockScaledUniversal3x, 
  84:       GemmKind.GroupedUniversal3x,
  85:       GemmKind.GroupedBlockScaledUniversal3x,
  86:       GemmKind.BlockwiseUniversal3x,
  87:       GemmKind.GroupedBlockwiseUniversal3x,
  88:     }
  89:     self.is_3x = gemm_kind in kinds_3x
  90:     self.prefix = "3x" if self.is_3x else ""
  91:     self.operation_kind = OperationKind.Gemm
  92:     self.arch = arch
  93:     self.tile_description = tile_description
  94:     self.gemm_kind = gemm_kind
  95:     self.A = A
  96:     self.B = B
  97:     self.C = C
  98:     self.D = D
  99:     # Intel Xe architectures: PVC (12), BMG/Xe2 (20), etc.
 100:     self.is_xe = self.arch >= INTEL_XE_ARCH_MIN and self.arch < INTEL_XE_ARCH_MAX
 101: 
 102:     if is_block_scaled(gemm_kind):
 103:       self.ScaleFactorA = ScaleFactorA
 104:       self.ScaleFactorB = ScaleFactorB
 105:       self.ScaleFactorD = ScaleFactorD["tensor"]
 106:       self.ScaleFactorVectorSize = ScaleFactorD["vector_size"]
 107: 
 108:     if is_blockwise(gemm_kind):
 109:       self.ScaleFactorMVecSize = ScaleFactorMVecSize
 110:       self.ScaleFactorNVecSize = ScaleFactorNVecSize
 111:       self.ScaleFactorKVecSize = ScaleFactorKVecSize
 112: 
 113:     if self.D == None:
 114:       self.D = self.C
 115: 
 116:     if not self.is_3x:
 117:       assert(kernel_schedule == KernelScheduleType.ScheduleAuto)
 118:       assert(epilogue_schedule == EpilogueScheduleType.ScheduleAuto)
 119:     self.kernel_schedule = kernel_schedule
 120:     self.epilogue_schedule = epilogue_schedule
 121:     self.element_epilogue = element_epilogue
 122:     self.epilogue_functor = epilogue_functor
 123: 
 124:     if self.is_3x and epilogue_functor == EpilogueFunctor.LinearCombination:
 125:       self.epilogue_functor = EpilogueFunctor3x.LinearCombination
 126: 
 127:     self.swizzling_functor = swizzling_functor
 128:     self.tile_scheduler = tile_scheduler
 129: 
 130:     # Only enable mixed input mode and mixed input shuffle for Hopper
 131:     self.mixed_input_mode = None
 132:     if self.is_mixed_input() and self.arch >= 90 and self.arch < 100:
 133:       self.mixed_input_mode = mixed_input_mode
 134:     self.mixed_input_shuffle = (self.mixed_input_mode is not None) and mixed_input_shuffle
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include is_block_scaled, is_blockwise, is_mixed_input.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 is_block_scaled, is_blockwise, is_mixed_input。

### L135-L136 — Comments

```python
 135: 
 136:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L137-L143 — Function `is_complex`

```python
 137:   def is_complex(self):
 138:     complex_operators = [
 139:       MathOperation.multiply_add_complex,
 140:       MathOperation.multiply_add_complex_gaussian,
 141:       MathOperation.multiply_add_complex_fast_f32
 142:     ]
 143:     return self.tile_description.math_instruction.math_operation in complex_operators
```
**EN:** Defines `is_complex()`, a predicate that checks whether complex.

**CN:** 定义 `is_complex()`，用于判断是否满足 complex 这一条件。

### L144-L145 — Comments

```python
 144: 
 145:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L146-L147 — Function `is_mixed_input`

```python
 146:   def is_mixed_input(self):
 147:     return self.A.element != self.B.element
```
**EN:** Defines `is_mixed_input()`, a predicate that checks whether mixed input.

**CN:** 定义 `is_mixed_input()`，用于判断是否满足 mixed input 这一条件。

### L148-L149 — Comments

```python
 148: 
 149:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L150-L151 — Function `is_planar_complex`

```python
 150:   def is_planar_complex(self):
 151:     return self.gemm_kind in (GemmKind.PlanarComplex, GemmKind.PlanarComplexArray)
```
**EN:** Defines `is_planar_complex()`, a predicate that checks whether planar complex.

**CN:** 定义 `is_planar_complex()`，用于判断是否满足 planar complex 这一条件。

### L152-L153 — Comments

```python
 152: 
 153:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L154-L160 — Function `accumulator_type`

```python
 154:   def accumulator_type(self):
 155:     accum = self.tile_description.math_instruction.element_accumulator
 156: 
 157:     if self.is_complex():
 158:       return get_complex_from_real(accum)
 159: 
 160:     return accum
```
**EN:** Defines `accumulator_type()`, which implements the accumulator type logic. Key helper calls include is_complex, get_complex_from_real.

**CN:** 定义 `accumulator_type()`，用于实现 accumulator type 相关逻辑。 其中会调用的重要辅助函数包括 is_complex, get_complex_from_real。

### L161-L162 — Comments

```python
 161: 
 162:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L163-L166 — Function `short_math_name`

```python
 163:   def short_math_name(self):
 164:     if self.tile_description.math_instruction.math_operation == MathOperation.multiply_add_complex_gaussian:
 165:       return "g%s" % ShortDataTypeNames[self.accumulator_type()]
 166:     return ShortDataTypeNames[self.accumulator_type()]
```
**EN:** Defines `short_math_name()`, which implements the short math name logic. Key helper calls include accumulator_type.

**CN:** 定义 `short_math_name()`，用于实现 short math name 相关逻辑。 其中会调用的重要辅助函数包括 accumulator_type。

### L167-L169 — Comments

```python
 167: 
 168: 
 169:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L170-L207 — Function `core_name`

```python
 170:   def core_name(self):
 171:     ''' The basic operation kind is prefixed with a letter indicating the accumulation type. '''
 172: 
 173:     inst_shape = ''
 174:     inst_operation = ''
 175:     intermediate_type = ''
 176: 
 177:     math_operations_map = {
 178:       MathOperation.xor_popc: 'xor',
 179:       MathOperation.and_popc: 'and',
 180:       MathOperation.multiply_add_fast_accum: 'fastaccum',
 181:     }
 182: 
 183:     tensor_ops = [
 184:       OpcodeClass.TensorOp,
 185:       OpcodeClass.WmmaTensorOp,
 186:       OpcodeClass.SparseTensorOp,
 187:       OpcodeClass.BlockScaledTensorOp, 
 188:     ]
 189: 
 190:     is_tensor_op = self.tile_description.math_instruction.opcode_class in tensor_ops
 191: 
 192:     if is_tensor_op:
 193: 
 194:       math_op = self.tile_description.math_instruction.math_operation
 195:       math_op_string = math_operations_map[math_op] if math_op in math_operations_map.keys() else ''
 196: 
 197:       inst_shape = "{0}{1}{2}".format(*tuple(self.tile_description.math_instruction.instruction_shape)) if not self.is_3x else ""
 198: 
 199:       inst_shape += math_op_string
 200: 
 201:       if self.tile_description.math_instruction.element_a != self.A.element and \
 202:         self.tile_description.math_instruction.element_a != self.tile_description.math_instruction.element_accumulator:
 203:         intermediate_type = DataTypeNames[self.tile_description.math_instruction.element_a]
 204: 
 205:     short_math_name = self.short_math_name() if not self.is_3x else ""
 206: 
 207:     return "%s%s%s%s" % (short_math_name, inst_shape, intermediate_type, GemmKindNames[self.gemm_kind])
```
**EN:** Defines `core_name()`. The docstring says: "The basic operation kind is prefixed with a letter indicating the accumulation type."; this block implements that behavior. Key helper calls include short_math_name, format, keys, tuple.

**CN:** 定义 `core_name()`。文档字符串说明其用途为：“The basic operation kind is prefixed with a letter indicating the accumulation type.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 short_math_name, format, keys, tuple。

### L208-L209 — Comments

```python
 208: 
 209:   # Generates a string representing the MMA instruction.
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L210-L241 — Function `extended_name`

```python
 210:   def extended_name(self):
 211:     ''' Append data types if they differ from compute type. '''
 212:     element_sfa = ""
 213:     element_sfb = ""
 214:     if self.is_complex():
 215:       extended_name = "${core_name}"
 216:     else:
 217:       if self.is_mixed_input():
 218:         extended_name = "${core_name}_${element_a}_${element_b}"
 219:         if self.C.element != self.tile_description.math_instruction.element_accumulator:
 220:           extended_name = "${element_c}_" + extended_name
 221:       elif is_blockwise(self.gemm_kind):
 222:         extended_name = "${core_name}_${element_sfa}x${element_a}_${element_sfb}x${element_b}"
 223:         element_sfa = DataTypeNames[self.accumulator_type()]
 224:         element_sfb = DataTypeNames[self.accumulator_type()]
 225:       else:
 226:         extended_name = "${core_name}"
 227:         if self.C.element != self.tile_description.math_instruction.element_accumulator:
 228:           extended_name = "${element_c}_" + extended_name
 229:         if self.A.element != self.tile_description.math_instruction.element_accumulator:
 230:           extended_name += "_${element_a}"
 231: 
 232:     extended_name = SubstituteTemplate(extended_name, {
 233:       'element_a': DataTypeNames[self.A.element],
 234:       'element_sfa' : element_sfa,
 235:       'element_b': DataTypeNames[self.B.element],
 236:       'element_sfb' : element_sfb,
 237:       'element_c': DataTypeNames[self.C.element],
 238:       'core_name': self.core_name()
 239:       })
 240: 
 241:     return extended_name
```
**EN:** Defines `extended_name()`. The docstring says: "Append data types if they differ from compute type."; this block implements that behavior. Key helper calls include is_complex, SubstituteTemplate, is_mixed_input, is_blockwise, core_name, accumulator_type.

**CN:** 定义 `extended_name()`。文档字符串说明其用途为：“Append data types if they differ from compute type.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 is_complex, SubstituteTemplate, is_mixed_input, is_blockwise, core_name, accumulator_type。

### L242-L243 — Comments

```python
 242: 
 243:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L244-L253 — Function `mixed_input_mode_name`

```python
 244:   def mixed_input_mode_name(self):
 245:     mode_name_mapping = {
 246:       MixedInputMode.ConvertOnly: "_cvt",
 247:       MixedInputMode.ScaleOnly: "_scl",
 248:       MixedInputMode.ScaleWithZeroPoint: "_sclzr"
 249:     }
 250:     mode_name = mode_name_mapping.get(self.mixed_input_mode, "")
 251:     if self.mixed_input_shuffle:
 252:       mode_name = mode_name + "_shfl"
 253:     return mode_name
```
**EN:** Defines `mixed_input_mode_name()`, which implements the mixed input mode name logic. Key helper calls include get.

**CN:** 定义 `mixed_input_mode_name()`，用于实现 mixed input mode name 相关逻辑。 其中会调用的重要辅助函数包括 get。

### L255-L299 — Function `extended_name_3x`

```python
 255:   def extended_name_3x(self):
 256:     '''Generates a string representing the MMA atom. Assumes accumulator type is C type.'''
 257:     extended_name = "{core_name}_{element_a}_{element_b}_{element_acc}_{element_c}_{element_d}".format(
 258:       element_a = DataTypeNames[self.A.element],
 259:       element_b = DataTypeNames[self.B.element],
 260:       element_acc = DataTypeNames[self.accumulator_type()],
 261:       element_c = DataTypeNames[self.C.element],
 262:       element_d = DataTypeNames[self.D.element],
 263:       core_name = self.core_name())
 264: 
 265:     if is_block_scaled(self.gemm_kind):
 266:       d_type_names = DataTypeNames[self.D.element]
 267: 
 268:       if self.ScaleFactorD.element != DataType.void:
 269:         d_type_names = DataTypeNames[self.ScaleFactorD.element] + "x" + d_type_names
 270: 
 271:       extended_name = "{core_name}_{element_sfa}x{element_a}_{element_sfb}x{element_b}_{element_acc}_{element_c}_{element_d}".format(
 272:         element_sfa = DataTypeNames[self.ScaleFactorA],
 273:         element_a = DataTypeNames[self.A.element],
 274:         element_sfb = DataTypeNames[self.ScaleFactorB],
 275:         element_b = DataTypeNames[self.B.element],
 276:         element_acc = DataTypeNames[self.accumulator_type()],
 277:         element_c = DataTypeNames[self.C.element],
 278:         element_d = d_type_names,
 279:         core_name = self.core_name())
 280: 
 281:     if is_blockwise(self.gemm_kind):
 282:       d_type_names = DataTypeNames[self.D.element]
 283: 
 284:       extended_name = "{core_name}_{sfvec_m_size}x{sfvec_k_size}{element_sfa}x{element_a}_{sfvec_n_size}x{sfvec_k_size}{element_sfb}x{element_b}_{element_acc}_{element_c}_{element_d}".format(
 285:         element_sfa = DataTypeNames[self.accumulator_type()],
 286:         element_a = DataTypeNames[self.A.element],
 287:         element_sfb = DataTypeNames[self.accumulator_type()],
 288:         element_b = DataTypeNames[self.B.element],
 289:         element_acc = DataTypeNames[self.accumulator_type()],
 290:         element_c = DataTypeNames[self.C.element],
 291:         element_d = d_type_names,
 292:         sfvec_m_size = self.ScaleFactorMVecSize,
 293:         sfvec_n_size = self.ScaleFactorNVecSize,
 294:         sfvec_k_size = self.ScaleFactorKVecSize,
 295:         core_name = self.core_name())
 296: 
 297:     if self.mixed_input_mode != None:
 298:       extended_name = extended_name + self.mixed_input_mode_name()
 299:     return extended_name
```
**EN:** Defines `extended_name_3x()`. The docstring says: "Generates a string representing the MMA atom."; this block implements that behavior. Key helper calls include format, is_block_scaled, is_blockwise, core_name, mixed_input_mode_name, accumulator_type.

**CN:** 定义 `extended_name_3x()`。文档字符串说明其用途为：“Generates a string representing the MMA atom.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 format, is_block_scaled, is_blockwise, core_name, mixed_input_mode_name, accumulator_type。

### L301-L309 — Function `datatype_name_3x`

```python
 301:   def datatype_name_3x(self):
 302:     '''Generates a string representing the MMA atom. Assumes accumulator type is C type.'''
 303:     datatype_name = "{element_a}_{element_b}_{element_acc}_{element_c}_{element_d}".format(
 304:       element_a = DataTypeNames[self.A.element],
 305:       element_b = DataTypeNames[self.B.element],
 306:       element_acc = DataTypeNames[self.accumulator_type()],
 307:       element_c = DataTypeNames[self.C.element],
 308:       element_d = DataTypeNames[self.D.element])
 309:     return datatype_name
```
**EN:** Defines `datatype_name_3x()`. The docstring says: "Generates a string representing the MMA atom."; this block implements that behavior. Key helper calls include format, accumulator_type.

**CN:** 定义 `datatype_name_3x()`。文档字符串说明其用途为：“Generates a string representing the MMA atom.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 format, accumulator_type。

### L310-L311 — Comments

```python
 310: 
 311:   # Generates a short string representing the AB layout tags (e.g. nt or tn)
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L312-L318 — Function `layout_name`

```python
 312:   def layout_name(self):
 313:     if self.is_complex() or self.is_planar_complex():
 314:       return "%s%s" % (
 315:         ShortComplexLayoutNames[(self.A.layout, self.A.complex_transform)],
 316:         ShortComplexLayoutNames[(self.B.layout, self.B.complex_transform)]
 317:       )
 318:     return "%s%s" % (ShortLayoutTypeNames[self.A.layout], ShortLayoutTypeNames[self.B.layout])
```
**EN:** Defines `layout_name()`, which implements the layout name logic. Key helper calls include is_complex, is_planar_complex.

**CN:** 定义 `layout_name()`，用于实现 layout name 相关逻辑。 其中会调用的重要辅助函数包括 is_complex, is_planar_complex。

### L319-L320 — Comments

```python
 319: 
 320:   # Generates a short string representing the ABC layout tags (e.g. ntn or tnn)
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L321-L331 — Function `layout_name_3x`

```python
 321:   def layout_name_3x(self):
 322:     if self.is_complex() or self.is_planar_complex():
 323:       return "{}{}{}".format(
 324:         ShortComplexLayoutNames[(self.A.layout, self.A.complex_transform)],
 325:         ShortComplexLayoutNames[(self.B.layout, self.B.complex_transform)],
 326:         ShortComplexLayoutNames[(self.C.layout, self.C.complex_transform)])
 327:     else:
 328:       return "{}{}{}".format(
 329:         ShortLayoutTypeNames[self.A.layout],
 330:         ShortLayoutTypeNames[self.B.layout],
 331:         ShortLayoutTypeNames[self.C.layout])
```
**EN:** Defines `layout_name_3x()`, which implements the layout name 3x logic. Key helper calls include is_complex, is_planar_complex, format.

**CN:** 定义 `layout_name_3x()`，用于实现 layout name 3x 相关逻辑。 其中会调用的重要辅助函数包括 is_complex, is_planar_complex, format。

### L332-L333 — Comments

```python
 332: 
 333:   # Generates a short string representing underlying kernel schedule type
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L334-L335 — Function `kernel_schedule_name_3x`

```python
 334:   def kernel_schedule_name_3x(self):
 335:     return KernelScheduleSuffixes[self.kernel_schedule]
```
**EN:** Defines `kernel_schedule_name_3x()`, which implements the kernel schedule name 3x logic.

**CN:** 定义 `kernel_schedule_name_3x()`，用于实现 kernel schedule name 3x 相关逻辑。

### L336-L337 — Comments

```python
 336: 
 337:   # Generates a short string representing underlying epilogue schedule type
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L338-L344 — Function `epilogue_schedule_name_3x`

```python
 338:   def epilogue_schedule_name_3x(self):
 339: 
 340:     if is_block_scaled(self.gemm_kind):
 341:       if self.ScaleFactorD.element != DataType.void:
 342:         return EpilogueScheduleSuffixes[self.epilogue_schedule] + "_epiVs" + str(self.ScaleFactorVectorSize)+ShortLayoutTypeNames[self.ScaleFactorD.layout]
 343:     
 344:     return EpilogueScheduleSuffixes[self.epilogue_schedule]
```
**EN:** Defines `epilogue_schedule_name_3x()`, which implements the epilogue schedule name 3x logic. Key helper calls include is_block_scaled, str.

**CN:** 定义 `epilogue_schedule_name_3x()`，用于实现 epilogue schedule name 3x 相关逻辑。 其中会调用的重要辅助函数包括 is_block_scaled, str。

### L345-L346 — Comments

```python
 345: 
 346:   # Generate a short string representing the operation class
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L347-L348 — Function `opcode_class_name`

```python
 347:   def opcode_class_name(self):
 348:     return OpcodeClassNames[self.tile_description.math_instruction.opcode_class]
```
**EN:** Defines `opcode_class_name()`, which implements the opcode class name logic.

**CN:** 定义 `opcode_class_name()`，用于实现 opcode class name 相关逻辑。

### L350-L365 — Function `get_collective_tile_shape`

```python
 350:   def get_collective_tile_shape(self):
 351:     """
 352:     Get the tile shape passed to the collective builder.
 353:     On Blackwell, this is different than the operation.tile_description.tile_shape.
 354:     """
 355:     is_sm100_kernel = (self.arch == 100 or self.arch == 103)
 356:     if not is_sm100_kernel:
 357:       return self.tile_description.tile_shape
 358: 
 359:     opcode_class_main = self.tile_description.math_instruction.opcode_class
 360:     instruction_shape = self.tile_description.math_instruction.instruction_shape
 361:     tile_shape_m, tile_shape_n, tile_shape_k = self.tile_description.tile_shape
 362:     if opcode_class_main in [OpcodeClass.TensorOp, OpcodeClass.BlockScaledTensorOp, OpcodeClass.SparseTensorOp]:
 363:       tile_shape_m = instruction_shape[0]
 364:       tile_shape_n = instruction_shape[1]
 365:     return (tile_shape_m, tile_shape_n, tile_shape_k)
```
**EN:** Defines `get_collective_tile_shape()`. The docstring says: "Get the tile shape passed to the collective builder."; this block implements that behavior.

**CN:** 定义 `get_collective_tile_shape()`。文档字符串说明其用途为：“Get the tile shape passed to the collective builder.”；这一代码块给出了该行为的具体实现。

### L366-L367 — Comments

```python
 366: 
 367:   # Generates the full kernel function name
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L368-L369 — Function `procedural_name`

```python
 368:   def procedural_name(self):
 369:     return self._procedural_name
```
**EN:** Defines `procedural_name()`, which implements the procedural name logic.

**CN:** 定义 `procedural_name()`，用于实现 procedural name 相关逻辑。

### L370-L371 — Comments

```python
 370: 
 371:   @functools.cached_property
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L372-L442 — Function `_procedural_name`

```python
 372:   def _procedural_name(self):
 373:     ''' The full procedural name indicates architecture, extended name, tile size, and layout. '''
 374:     opcode_class_name = OpcodeClassNames[self.tile_description.math_instruction.opcode_class]
 375:     if self.arch >= 90:
 376:       kernel_name_template = "cutlass{p}_sm{ar}_{op}_{ex}{ct}{cs}_{l}_{s}_align{al}{t}{k}{e}"
 377:       tile_shape = self.get_collective_tile_shape()
 378:       return kernel_name_template.format(
 379:           p = self.prefix,
 380:           ar = self.arch,
 381:           op = opcode_class_name,
 382:           ex = self.extended_name_3x(),
 383:           ct = '_' + 'x'.join([str(i) for i in tile_shape]) if tile_shape[0] > 0 else "",
 384:           cs = '_' + 'x'.join([str(i) for i in self.tile_description.cluster_shape]),
 385:           l = self.tile_description.stages,
 386:           s = self.layout_name_3x(),
 387:           al = str(max(self.A.alignment, self.B.alignment)),
 388:           t = TileSchedulerSuffixes[self.tile_scheduler],
 389:           k = self.kernel_schedule_name_3x(),
 390:           e = self.epilogue_schedule_name_3x())
 391:     elif not self.is_xe:
 392:       threadblock = self.tile_description.procedural_name()
 393:       return "cutlass{p}_{op}_{ex}_{tb}_{l}_align{a}".format(
 394:           p = self.prefix,
 395:           op = opcode_class_name,
 396:           ex = self.extended_name(),
 397:           tb = threadblock,
 398:           l = self.layout_name(),
 399:           a = str(max(self.A.alignment, self.B.alignment)))
 400:     else:
 401:       # Intel Xe architectures use xe{cc} naming with similar detail level as NVIDIA
 402:       # Format: cutlass{p}_xe{ar}_{op}_{ex}{ct}{cs}_{l}_{s}_align{al}{t}{k}{e}
 403:       if self.is_3x:
 404:         # Use 3x naming convention with full details like NVIDIA SM90+
 405:         tile_shape = self.get_collective_tile_shape()
 406:         extended = self.extended_name_3x()
 407:         
 408:         # Add D type suffix if different from C type to distinguish mixed precision variants
 409:         if self.D.element != self.C.element:
 410:           extended += f"_d{DataTypeNames[self.D.element]}"
 411:         
 412:         kernel_name_template = "cutlass{p}_xe{ar}_{op}_{ex}{ct}{cs}_{l}_{s}_align{al}{t}{k}{e}"
 413:         return kernel_name_template.format(
 414:             p = self.prefix,
 415:             ar = self.arch,
 416:             op = opcode_class_name,
 417:             ex = extended,
 418:             ct = '_' + 'x'.join([str(i) for i in tile_shape]) if tile_shape[0] > 0 else "",
 419:             cs = '_' + 'x'.join([str(i) for i in self.tile_description.cluster_shape]),
 420:             l = self.tile_description.stages,
 421:             s = self.layout_name_3x(),
 422:             al = str(max(self.A.alignment, self.B.alignment)),
 423:             t = TileSchedulerSuffixes[self.tile_scheduler],
 424:             k = self.kernel_schedule_name_3x(),
 425:             e = self.epilogue_schedule_name_3x())
 426:       else:
 427:         # Legacy naming for non-3x Intel Xe operations
 428:         threadblock = self.tile_description.procedural_name()
 429:         extended = self.extended_name()
 430:         
 431:         # Add D type suffix if different from C type to distinguish mixed precision variants
 432:         if self.D.element != self.C.element:
 433:           extended += f"_d{DataTypeNames[self.D.element]}"
 434:         
 435:         return "cutlass{p}_xe{ar}_{op}_{ex}_{tb}_{l}_align{a}".format(
 436:             p = self.prefix,
 437:             ar = self.arch,
 438:             op = opcode_class_name,
 439:             ex = extended,
 440:             tb = threadblock,
 441:             l = self.layout_name(),
 442:             a = str(max(self.A.alignment, self.B.alignment)))
```
**EN:** Defines `_procedural_name()`. The docstring says: "The full procedural name indicates architecture, extended name, tile size, and layout."; this block implements that behavior. Key helper calls include get_collective_tile_shape, format, procedural_name, extended_name_3x, layout_name_3x, str.

**CN:** 定义 `_procedural_name()`。文档字符串说明其用途为：“The full procedural name indicates architecture, extended name, tile size, and layout.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 get_collective_tile_shape, format, procedural_name, extended_name_3x, layout_name_3x, str。

### L443-L444 — Comments

```python
 443: 
 444:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L445-L447 — Function `configuration_name`

```python
 445:   def configuration_name(self):
 446:     ''' The full procedural name indicates architecture, extended name, tile size, and layout. '''
 447:     return self.procedural_name()
```
**EN:** Defines `configuration_name()`. The docstring says: "The full procedural name indicates architecture, extended name, tile size, and layout."; this block implements that behavior. Key helper calls include procedural_name.

**CN:** 定义 `configuration_name()`。文档字符串说明其用途为：“The full procedural name indicates architecture, extended name, tile size, and layout.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 procedural_name。

### L449-L450 — Function `__hash__`

```python
 449:   def __hash__(self):
 450:     return hash(self.configuration_name())
```
**EN:** Defines `__hash__()`, which implements the hash logic. Key helper calls include hash, configuration_name.

**CN:** 定义 `__hash__()`，用于实现 hash 相关逻辑。 其中会调用的重要辅助函数包括 hash, configuration_name。

### L452-L453 — Function `__eq__`

```python
 452:   def __eq__(self, other):
 453:     return self.configuration_name() == other.configuration_name()
```
**EN:** Defines `__eq__()`, which implements the eq logic. Key helper calls include configuration_name.

**CN:** 定义 `__eq__()`，用于实现 eq 相关逻辑。 其中会调用的重要辅助函数包括 configuration_name。

### L454-L461 — Comments

```python
 454: 
 455: ###################################################################################################
 456: #
 457: # Data structure modeling a grouped GEMM operation
 458: #
 459: ###################################################################################################
 460: 
 461: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L462-L463 — Class `GroupedGemmOperation`

```python
 462: class GroupedGemmOperation(GemmOperation):
 463:   #
```
**EN:** Introduces class `GroupedGemmOperation`, a descriptor for grouped gemm operations.

**CN:** 引入类 `GroupedGemmOperation`，它是 grouped gemm 操作的描述对象。

### L464-L470 — Function `__init__`

```python
 464:   def __init__(self, gemm_kind, arch, tile_description, A, B, C, element_epilogue, \
 465:       epilogue_functor = EpilogueFunctor.LinearCombination, swizzling_functor = SwizzlingFunctor.Identity8, \
 466:       scheduler_mode = GroupScheduleMode.Device):
 467:     super().__init__(gemm_kind, arch, tile_description, A, B, C, element_epilogue, \
 468:                      epilogue_functor, swizzling_functor)
 469: 
 470:     self.scheduler_mode = scheduler_mode
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include __init__, super.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 __init__, super。

### L471-L472 — Comments

```python
 471: 
 472:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L473-L480 — Function `procedural_name`

```python
 473:   def procedural_name(self):
 474:     ''' The full procedural name indicates architecture, extended name, tile size, and layout. '''
 475:     base = super().procedural_name()
 476:     return SubstituteTemplate(
 477:       base + "_schedule${schedule}",
 478:       {
 479:         'schedule': ShortGroupScheduleModeNames[self.scheduler_mode]
 480:       })
```
**EN:** Defines `procedural_name()`. The docstring says: "The full procedural name indicates architecture, extended name, tile size, and layout."; this block implements that behavior. Key helper calls include procedural_name, SubstituteTemplate, super.

**CN:** 定义 `procedural_name()`。文档字符串说明其用途为：“The full procedural name indicates architecture, extended name, tile size, and layout.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 procedural_name, SubstituteTemplate, super。

### L481-L489 — Comments

```python
 481: 
 482: 
 483: ###################################################################################################
 484: #
 485: # Emits single instances of a CUTLASS device-wide operator
 486: #
 487: ###################################################################################################
 488: 
 489: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L490-L490 — Class `EmitGemmInstance`

```python
 490: class EmitGemmInstance:
```
**EN:** Introduces class `EmitGemmInstance`. The docstring says: "Responsible for emitting a CUTLASS template definition"; the class body implements that role.

**CN:** 引入类 `EmitGemmInstance`。文档字符串说明其用途为：“Responsible for emitting a CUTLASS template definition”；类体负责实现这一职责。

### L491-L491 — Documentation/setup

```python
 491:   ''' Responsible for emitting a CUTLASS template definition'''
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L493-L548 — Function `__init__`

```python
 493:   def __init__(self, operation_suffix = ''):
 494:     self.operation_suffix = operation_suffix
 495:     self.includes = []
 496:     self.gemm_template = """
 497:   // Gemm operator ${operation_name}
 498:   using Operation_${operation_name} = cutlass::gemm::device::Gemm<
 499:     ${element_a}, ${layout_a},
 500:     ${element_b}, ${layout_b},
 501:     ${element_c}, ${layout_c},
 502:     ${element_accumulator},
 503:     ${opcode_class},
 504:     ${arch},
 505:     cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
 506:     cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
 507:     cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
 508:     ${epilogue_functor}<
 509:       ${element_c},
 510:       ${epilogue_vector_length},
 511:       ${element_accumulator},
 512:       ${element_epilogue}
 513:     >,
 514:     ${swizzling_functor},
 515:     ${stages},
 516:     ${align_a},
 517:     ${align_b},
 518:     false,
 519:     ${math_operation}
 520:     ${residual}
 521:   >;
 522: """
 523:     self.gemm_complex_template = """
 524:   // Gemm operator ${operation_name}
 525:   using Operation_${operation_name} = cutlass::gemm::device::GemmComplex<
 526:     ${element_a}, ${layout_a},
 527:     ${element_b}, ${layout_b},
 528:     ${element_c}, ${layout_c},
 529:     ${element_accumulator},
 530:     ${opcode_class},
 531:     ${arch},
 532:     cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
 533:     cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
 534:     cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
 535:     ${epilogue_functor}<
 536:       ${element_c},
 537:       ${epilogue_vector_length},
 538:       ${element_accumulator},
 539:       ${element_epilogue}
 540:     >,
 541:     ${swizzling_functor},
 542:     ${stages},
 543:     ${transform_a},
 544:     ${transform_b},
 545:     ${math_operation}
 546:     ${residual}
 547:   >;
 548: """
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L549-L550 — Comments

```python
 549: 
 550:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L551-L556 — Function `instance_template`

```python
 551:   def instance_template(self):
 552:     return """
 553: ${compile_guard_start}
 554:   manifest.append(new ${gemm_kind}<Operation_${operation_name}>("${operation_name}"));
 555: ${compile_guard_end}
 556: """
```
**EN:** Defines `instance_template()`, which implements the instance template logic.

**CN:** 定义 `instance_template()`，用于实现 instance template 相关逻辑。

### L557-L558 — Comments

```python
 557: 
 558:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L559-L602 — Function `emit`

```python
 559:   def emit(self, operation):
 560: 
 561:     warp_shape = [operation.tile_description.threadblock_shape[idx] // operation.tile_description.warp_count[idx] for idx in range(3)]
 562: 
 563:     epilogue_vector_length = int(min(operation.C.alignment * DataTypeSize[operation.C.element], 128) / DataTypeSize[operation.C.element])
 564: 
 565:     residual = ''
 566: 
 567:     values = {
 568:       'operation_name': operation.procedural_name(),
 569:       'element_a': DataTypeTag[operation.A.element],
 570:       'layout_a': LayoutTag[operation.A.layout],
 571:       'element_b': DataTypeTag[operation.B.element],
 572:       'layout_b': LayoutTag[operation.B.layout],
 573:       'element_c': DataTypeTag[operation.C.element],
 574:       'layout_c': LayoutTag[operation.C.layout],
 575:       'element_accumulator': DataTypeTag[operation.accumulator_type()],
 576:       'opcode_class': OpcodeClassTag[operation.tile_description.math_instruction.opcode_class],
 577:       'arch': "cutlass::arch::Sm%d" % operation.arch,
 578:       'threadblock_shape_m': str(operation.tile_description.threadblock_shape[0]),
 579:       'threadblock_shape_n': str(operation.tile_description.threadblock_shape[1]),
 580:       'threadblock_shape_k': str(operation.tile_description.threadblock_shape[2]),
 581:       'warp_shape_m': str(warp_shape[0]),
 582:       'warp_shape_n': str(warp_shape[1]),
 583:       'warp_shape_k': str(warp_shape[2]),
 584:       'instruction_shape_m': str(operation.tile_description.math_instruction.instruction_shape[0]),
 585:       'instruction_shape_n': str(operation.tile_description.math_instruction.instruction_shape[1]),
 586:       'instruction_shape_k': str(operation.tile_description.math_instruction.instruction_shape[2]),
 587:       'epilogue_vector_length': str(epilogue_vector_length),
 588:       'element_epilogue': str(DataTypeTag[operation.element_epilogue]),
 589:       'epilogue_functor': EpilogueFunctorTag[operation.epilogue_functor],
 590:       'swizzling_functor': SwizzlingFunctorTag[operation.swizzling_functor],
 591:       'stages': str(operation.tile_description.stages),
 592:       'align_a': str(operation.A.alignment),
 593:       'align_b': str(operation.B.alignment),
 594:       'transform_a': ComplexTransformTag[operation.A.complex_transform],
 595:       'transform_b': ComplexTransformTag[operation.B.complex_transform],
 596:       'math_operation': MathOperationTag[operation.tile_description.math_instruction.math_operation],
 597:       'residual': residual
 598:     }
 599: 
 600:     template = self.gemm_complex_template if operation.is_complex() else self.gemm_template
 601: 
 602:     return SubstituteTemplate(template, values)
```
**EN:** Defines `emit()`, which emits . Key helper calls include int, SubstituteTemplate, procedural_name, str, is_complex, range.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 int, SubstituteTemplate, procedural_name, str, is_complex, range。

### L603-L605 — Comments

```python
 603: 
 604: ###################################################################################################
 605: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L606-L606 — Class `EmitSparseGemmInstance`

```python
 606: class EmitSparseGemmInstance:
```
**EN:** Introduces class `EmitSparseGemmInstance`. The docstring says: "Responsible for emitting a CUTLASS template definition"; the class body implements that role.

**CN:** 引入类 `EmitSparseGemmInstance`。文档字符串说明其用途为：“Responsible for emitting a CUTLASS template definition”；类体负责实现这一职责。

### L607-L607 — Documentation/setup

```python
 607:   ''' Responsible for emitting a CUTLASS template definition'''
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L609-L638 — Function `__init__`

```python
 609:   def __init__(self, operation_suffix = ''):
 610:     self.operation_suffix = operation_suffix
 611:     self.includes = []
 612:     self.gemm_template = """
 613:   // Gemm operator ${operation_name}
 614:   using Operation_${operation_name} = cutlass::gemm::device::SparseGemm<
 615:     ${element_a}, ${layout_a},
 616:     ${element_b}, ${layout_b},
 617:     ${element_c}, ${layout_c},
 618:     ${element_accumulator},
 619:     ${opcode_class},
 620:     ${arch},
 621:     cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
 622:     cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
 623:     cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
 624:     ${epilogue_functor}<
 625:       ${element_c},
 626:       ${epilogue_vector_length},
 627:       ${element_accumulator},
 628:       ${element_epilogue}
 629:     >,
 630:     ${swizzling_functor},
 631:     ${stages},
 632:     ${align_a},
 633:     ${align_b},
 634:     false,
 635:     ${math_operation}
 636:     ${residual}
 637:   >;
 638: """
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L639-L640 — Comments

```python
 639: 
 640:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L641-L646 — Function `instance_template`

```python
 641:   def instance_template(self):
 642:     return """
 643: ${compile_guard_start}
 644:   manifest.append(new ${gemm_kind}<Operation_${operation_name}>("${operation_name}"));
 645: ${compile_guard_end}
 646: """
```
**EN:** Defines `instance_template()`, which implements the instance template logic.

**CN:** 定义 `instance_template()`，用于实现 instance template 相关逻辑。

### L647-L648 — Comments

```python
 647: 
 648:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L649-L692 — Function `emit`

```python
 649:   def emit(self, operation):
 650: 
 651:     warp_shape = [operation.tile_description.threadblock_shape[idx] // operation.tile_description.warp_count[idx] for idx in range(3)]
 652: 
 653:     epilogue_vector_length = int(min(operation.C.alignment * DataTypeSize[operation.C.element], 128) / DataTypeSize[operation.C.element])
 654: 
 655:     residual = ''
 656: 
 657:     values = {
 658:       'operation_name': operation.procedural_name(),
 659:       'element_a': DataTypeTag[operation.A.element],
 660:       'layout_a': LayoutTag[operation.A.layout],
 661:       'element_b': DataTypeTag[operation.B.element],
 662:       'layout_b': LayoutTag[operation.B.layout],
 663:       'element_c': DataTypeTag[operation.C.element],
 664:       'layout_c': LayoutTag[operation.C.layout],
 665:       'element_accumulator': DataTypeTag[operation.accumulator_type()],
 666:       'opcode_class': OpcodeClassTag[operation.tile_description.math_instruction.opcode_class],
 667:       'arch': "cutlass::arch::Sm%d" % operation.arch,
 668:       'threadblock_shape_m': str(operation.tile_description.threadblock_shape[0]),
 669:       'threadblock_shape_n': str(operation.tile_description.threadblock_shape[1]),
 670:       'threadblock_shape_k': str(operation.tile_description.threadblock_shape[2]),
 671:       'warp_shape_m': str(warp_shape[0]),
 672:       'warp_shape_n': str(warp_shape[1]),
 673:       'warp_shape_k': str(warp_shape[2]),
 674:       'instruction_shape_m': str(operation.tile_description.math_instruction.instruction_shape[0]),
 675:       'instruction_shape_n': str(operation.tile_description.math_instruction.instruction_shape[1]),
 676:       'instruction_shape_k': str(operation.tile_description.math_instruction.instruction_shape[2]),
 677:       'epilogue_vector_length': str(epilogue_vector_length),
 678:       'element_epilogue': str(DataTypeTag[operation.element_epilogue]),
 679:       'epilogue_functor': EpilogueFunctorTag[operation.epilogue_functor],
 680:       'swizzling_functor': SwizzlingFunctorTag[operation.swizzling_functor],
 681:       'stages': str(operation.tile_description.stages),
 682:       'align_a': str(operation.A.alignment),
 683:       'align_b': str(operation.B.alignment),
 684:       'transform_a': ComplexTransformTag[operation.A.complex_transform],
 685:       'transform_b': ComplexTransformTag[operation.B.complex_transform],
 686:       'math_operation': MathOperationTag[operation.tile_description.math_instruction.math_operation],
 687:       'residual': residual
 688:     }
 689: 
 690:     template = self.gemm_template
 691: 
 692:     return SubstituteTemplate(template, values)
```
**EN:** Defines `emit()`, which emits . Key helper calls include int, SubstituteTemplate, procedural_name, str, range, min.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 int, SubstituteTemplate, procedural_name, str, range, min。

### L693-L697 — Comments

```python
 693: 
 694: ###################################################################################################
 695: 
 696: 
 697: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L698-L698 — Class `EmitGemmUniversalInstance`

```python
 698: class EmitGemmUniversalInstance:
```
**EN:** Introduces class `EmitGemmUniversalInstance`. The docstring says: "Responsible for emitting a CUTLASS template definition"; the class body implements that role.

**CN:** 引入类 `EmitGemmUniversalInstance`。文档字符串说明其用途为：“Responsible for emitting a CUTLASS template definition”；类体负责实现这一职责。

### L699-L699 — Documentation/setup

```python
 699:   ''' Responsible for emitting a CUTLASS template definition'''
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L701-L766 — Function `__init__`

```python
 701:   def __init__(self, operation_suffix = ''):
 702:     self.operation_suffix = operation_suffix
 703:     self.includes = [
 704:       "cutlass/cutlass.h",
 705:       "cutlass/numeric_types.h",
 706:       "cutlass/arch/arch.h",
 707:       "cutlass/arch/mma.h",
 708:       "cutlass/layout/matrix.h",
 709:       "cutlass/gemm/device/gemm.h",
 710:       "cutlass/gemm/device/gemm_universal_adapter.h",
 711:       "cutlass/gemm/kernel/default_gemm_universal.h",
 712:     ]
 713:     self.builtin_epilogue_functor_template = """
 714:     ${epilogue_functor}<
 715:       ${element_c},
 716:       ${epilogue_vector_length},
 717:       ${element_accumulator},
 718:       ${element_epilogue}
 719:     >
 720: """
 721:     self.gemm_template = """
 722: // Gemm operator ${operation_name}
 723: using ${operation_name}_base =
 724:   typename cutlass::gemm::kernel::DefaultGemmUniversal<
 725:     ${element_b}, ${layout_b}, ${transform_b}, ${align_b},    // transposed B operand
 726:     ${element_a}, ${layout_a}, ${transform_a}, ${align_a},    // transposed A operand
 727:     ${element_c}, ${layout_c},
 728:     ${element_accumulator},
 729:     ${opcode_class},
 730:     ${arch},
 731:     cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
 732:     cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
 733:     cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
 734:     ${epilogue_functor},
 735:     ${swizzling_functor},
 736:     ${stages},
 737:     ${math_operation}
 738: >::GemmKernel;
 739: 
 740: // Define named type
 741: struct ${operation_name}${operation_suffix} :
 742:   public ${operation_name}_base { };
 743: """
 744:     self.gemm_template_interleaved = """
 745: // Gemm operator ${operation_name}
 746: using ${operation_name}_base =
 747:   typename cutlass::gemm::kernel::DefaultGemmUniversal<
 748:     ${element_a}, ${layout_a}, ${transform_a}, ${align_a},
 749:     ${element_b}, ${layout_b}, ${transform_b}, ${align_b},
 750:     ${element_c}, ${layout_c},
 751:     ${element_accumulator},
 752:     ${opcode_class},
 753:     ${arch},
 754:     cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
 755:     cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
 756:     cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
 757:     ${epilogue_functor},
 758:     ${swizzling_functor},
 759:     ${stages},
 760:     ${math_operation}
 761: >::GemmKernel;
 762: 
 763: // Define named type
 764: struct ${operation_name}${operation_suffix} :
 765:   public ${operation_name}_base { };
 766: """
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L767-L768 — Comments

```python
 767: 
 768:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L769-L776 — Function `instance_template`

```python
 769:   def instance_template(self):
 770:     return """
 771: ${compile_guard_start}
 772:   manifest.append(new ${gemm_kind}<
 773:       cutlass::gemm::device::GemmUniversalAdapter<${operation_name}>
 774:     >("${operation_name}"));
 775: ${compile_guard_end}
 776: """
```
**EN:** Defines `instance_template()`, which implements the instance template logic.

**CN:** 定义 `instance_template()`，用于实现 instance template 相关逻辑。

### L777-L778 — Comments

```python
 777: 
 778:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L779-L854 — Function `emit`

```python
 779:   def emit(self, operation):
 780: 
 781:     threadblock_shape = operation.tile_description.threadblock_shape
 782:     warp_count = operation.tile_description.warp_count
 783: 
 784:     warp_shape = [threadblock_shape[idx] // warp_count[idx] for idx in range(3)]
 785: 
 786:     transpose_layouts = {
 787:       LayoutType.ColumnMajor: LayoutType.RowMajor,
 788:       LayoutType.RowMajor: LayoutType.ColumnMajor
 789:     }
 790: 
 791:     if operation.A.layout in transpose_layouts.keys() and \
 792:       operation.B.layout in transpose_layouts.keys() and \
 793:       operation.C.layout in transpose_layouts.keys():
 794: 
 795:       instance_layout_A = transpose_layouts[operation.A.layout]
 796:       instance_layout_B = transpose_layouts[operation.B.layout]
 797:       instance_layout_C = transpose_layouts[operation.C.layout]
 798: 
 799:       gemm_template = self.gemm_template
 800:     else:
 801:       instance_layout_A, instance_layout_B, instance_layout_C = \
 802:         (operation.A.layout, operation.B.layout, operation.C.layout)
 803: 
 804:       gemm_template = self.gemm_template_interleaved
 805:     #
 806: 
 807:     # Support built-in epilogue functors or user-defined functions
 808:     if isinstance(operation.epilogue_functor, enum.Enum):
 809: 
 810:       epilogue_vector_length = \
 811:         min(operation.C.alignment * DataTypeSize[operation.C.element], 128) // DataTypeSize[operation.C.element]
 812: 
 813:       values = {
 814:         'epilogue_vector_length': str(epilogue_vector_length),
 815:         'element_epilogue': str(DataTypeTag[operation.element_epilogue]),
 816:         'epilogue_functor': EpilogueFunctorTag[operation.epilogue_functor],
 817:       }
 818:       epilogue_functor = SubstituteTemplate(self.builtin_epilogue_functor_template, values)
 819:     else:
 820:       epilogue_functor = self.epilogue_functor.emit_declaration()
 821:     #
 822: 
 823:     values = {
 824:       'operation_name': operation.procedural_name(),
 825:       'operation_suffix': self.operation_suffix,
 826:       'element_a': DataTypeTag[operation.A.element],
 827:       'layout_a': LayoutTag[instance_layout_A],
 828:       'element_b': DataTypeTag[operation.B.element],
 829:       'layout_b': LayoutTag[instance_layout_B],
 830:       'element_c': DataTypeTag[operation.C.element],
 831:       'layout_c': LayoutTag[instance_layout_C],
 832:       'element_accumulator': DataTypeTag[operation.accumulator_type()],
 833:       'opcode_class': OpcodeClassTag[operation.tile_description.math_instruction.opcode_class],
 834:       'arch': "cutlass::arch::Sm%d" % operation.arch,
 835:       'threadblock_shape_m': str(operation.tile_description.threadblock_shape[0]),
 836:       'threadblock_shape_n': str(operation.tile_description.threadblock_shape[1]),
 837:       'threadblock_shape_k': str(operation.tile_description.threadblock_shape[2]),
 838:       'warp_shape_m': str(warp_shape[0]),
 839:       'warp_shape_n': str(warp_shape[1]),
 840:       'warp_shape_k': str(warp_shape[2]),
 841:       'instruction_shape_m': str(operation.tile_description.math_instruction.instruction_shape[0]),
 842:       'instruction_shape_n': str(operation.tile_description.math_instruction.instruction_shape[1]),
 843:       'instruction_shape_k': str(operation.tile_description.math_instruction.instruction_shape[2]),
 844:       'epilogue_functor': epilogue_functor,
 845:       'swizzling_functor': SwizzlingFunctorTag[operation.swizzling_functor],
 846:       'stages': str(operation.tile_description.stages),
 847:       'align_a': str(operation.A.alignment),
 848:       'align_b': str(operation.B.alignment),
 849:       'transform_a': ComplexTransformTag[operation.A.complex_transform],
 850:       'transform_b': ComplexTransformTag[operation.B.complex_transform],
 851:       'math_operation': MathOperationTag[operation.tile_description.math_instruction.math_operation]
 852:     }
 853: 
 854:     return SubstituteTemplate(gemm_template, values)
```
**EN:** Defines `emit()`, which emits . Key helper calls include isinstance, SubstituteTemplate, emit_declaration, procedural_name, str, range.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 isinstance, SubstituteTemplate, emit_declaration, procedural_name, str, range。

### L855-L858 — Comments

```python
 855: 
 856: 
 857: ###################################################################################################
 858: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L859-L859 — Class `EmitGemmUniversal3xInstance`

```python
 859: class EmitGemmUniversal3xInstance:
```
**EN:** Introduces class `EmitGemmUniversal3xInstance`. The docstring says: "Responsible for emitting a CUTLASS 3.x template definition"; the class body implements that role.

**CN:** 引入类 `EmitGemmUniversal3xInstance`。文档字符串说明其用途为：“Responsible for emitting a CUTLASS 3.x template definition”；类体负责实现这一职责。

### L860-L860 — Documentation/setup

```python
 860:   ''' Responsible for emitting a CUTLASS 3.x template definition'''
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L862-L922 — Function `__init__`

```python
 862:   def __init__(self, operation_suffix = ''):
 863:     self.operation_suffix = operation_suffix
 864:     self.includes = [
 865:       "cutlass/cutlass.h",
 866:       "cutlass/gemm/gemm.h",
 867:       "cutlass/numeric_types.h",
 868:       "cutlass/gemm/kernel/gemm_universal.hpp",
 869:       "cutlass/gemm/collective/collective_builder.hpp",
 870:       "cutlass/epilogue/collective/collective_builder.hpp",
 871:       "cutlass/detail/blockwise_scale_layout.hpp",
 872:     ]
 873:     self.builtin_epilogue_functor_template = \
 874: """${epilogue_functor}<
 875:       ${element_d},
 876:       ${element_epilogue},
 877:       ${element_c},
 878:       ${element_epilogue}
 879:     >"""
 880: 
 881:     self.gemm_template = """
 882: 
 883: using ${operation_name}_epilogue =
 884:   typename cutlass::epilogue::collective::CollectiveBuilder<
 885:     ${arch}, ${opcode_class_epi},
 886:     cute::Shape<cute::_${tile_shape_m}, cute::_${tile_shape_n}, cute::_${tile_shape_k}>,
 887:     cute::Shape<${cluster_shape_m}, ${cluster_shape_n}, ${cluster_shape_k}>,
 888:     ${epi_tile_mn},
 889:     ${element_accumulator}, ${element_epilogue},
 890:     ${element_c}, ${layout_c}, ${align_c},
 891:     ${element_d}, ${layout_d}, ${align_d},
 892:     ${epilogue_schedule},
 893:     ${epilogue_functor}
 894:   >::CollectiveOp;
 895: 
 896: ${mixed_dtype_prepare_code}
 897: ${blockwise_prepare_code}
 898: 
 899: using ${operation_name}_mainloop =
 900:   typename cutlass::gemm::collective::CollectiveBuilder<
 901:     ${arch}, ${opcode_class_main},
 902:     ${element_a}, ${layout_a}, ${align_a},
 903:     ${element_b}, ${layout_b}, ${align_b},
 904:     ${element_accumulator},
 905:     cute::Shape<cute::_${tile_shape_m}, cute::_${tile_shape_n}, cute::_${tile_shape_k}>,
 906:     cute::Shape<${cluster_shape_m}, ${cluster_shape_n}, ${cluster_shape_k}>,
 907:     ${stages},
 908:     ${kernel_schedule}
 909:   >::CollectiveOp;
 910: 
 911: // Gemm operator ${operation_name}
 912: using ${operation_name}_base = cutlass::gemm::kernel::GemmUniversal<
 913:     ${problem_shape},
 914:     ${operation_name}_mainloop,
 915:     ${operation_name}_epilogue,
 916:     ${tile_scheduler}>;
 917: 
 918: // Define named type
 919: struct ${operation_name} :
 920:   public ${operation_name}_base { };
 921: 
 922: """
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L923-L923 — Comments

```python
 923:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L924-L933 — Function `instance_template`

```python
 924:   def instance_template(self):
 925:     return """
 926: ${compile_guard_start}
 927:   {
 928:     using GemmKernel = cutlass::gemm::device::GemmUniversalAdapter<${operation_name}>;
 929:     manifest.append(
 930:       new ${gemm_kind}<GemmKernel>("${operation_name}"));
 931:   }
 932: ${compile_guard_end}
 933: """
```
**EN:** Defines `instance_template()`, which implements the instance template logic.

**CN:** 定义 `instance_template()`，用于实现 instance template 相关逻辑。

### L936-L958 — Function `emit_block_scale_epilogue_functor`

```python
 936:   def emit_block_scale_epilogue_functor(self, operation):
 937:     block_scaled_template = """
 938:       ${epilogue_functor}<
 939:         ${epi_vs},
 940:         ${element_d},
 941:         ${element_accumulator},
 942:         ${element_sfd},
 943:         ${layout_sfd},
 944:         ${element_c},
 945:         ${element_scalar}
 946:       >
 947:     """
 948:     block_scaled_values = {
 949:       'epi_vs'  : str(operation.ScaleFactorVectorSize),
 950:       'element_d': str(DataTypeTag[operation.D.element]),
 951:       'element_sfd': str(DataTypeTag[operation.ScaleFactorD.element]),
 952:       'layout_sfd': LayoutTag[operation.ScaleFactorD.layout],
 953:       'epilogue_functor': EpilogueFunctor3xTag[EpilogueFunctor3x.LinearCombinationBlockScaleFactor],
 954:       'element_accumulator': str(DataTypeTag[operation.accumulator_type()]),
 955:       'element_scalar': str(DataTypeTag[operation.accumulator_type()]),
 956:       'element_c': str(DataTypeTag[operation.C.element]),
 957:     }
 958:     return SubstituteTemplate(block_scaled_template, block_scaled_values)
```
**EN:** Defines `emit_block_scale_epilogue_functor()`, which emits block scale epilogue functor. Key helper calls include SubstituteTemplate, str, accumulator_type.

**CN:** 定义 `emit_block_scale_epilogue_functor()`，用于输出/生成 block scale epilogue functor。 其中会调用的重要辅助函数包括 SubstituteTemplate, str, accumulator_type。

### L959-L961 — Comments

```python
 959:   
 960: 
 961:   @staticmethod
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L962-L963 — Function `pointerize_if_grouped`

```python
 962:   def pointerize_if_grouped(operation, layout):
 963:     return layout if not is_grouped(operation.gemm_kind) else layout + "* "
```
**EN:** Defines `pointerize_if_grouped()`, which implements the pointerize if grouped logic. Key helper calls include is_grouped.

**CN:** 定义 `pointerize_if_grouped()`，用于实现 pointerize if grouped 相关逻辑。 其中会调用的重要辅助函数包括 is_grouped。

### L964-L965 — Comments

```python
 964: 
 965:   @staticmethod
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L966-L969 — Function `transform_layout_A_if_blockwise`

```python
 966:   def transform_layout_A_if_blockwise(operation, layout):
 967:     layout_sfa = f"{operation.procedural_name()}_LayoutSFA"
 968:     layout_sfa = layout_sfa if not is_grouped(operation.gemm_kind) else layout_sfa + "* "
 969:     return layout if not is_blockwise(operation.gemm_kind) else f"cute::tuple<{layout}, {layout_sfa}>"
```
**EN:** Defines `transform_layout_A_if_blockwise()`, which implements the transform layout a if blockwise logic. Key helper calls include procedural_name, is_grouped, is_blockwise.

**CN:** 定义 `transform_layout_A_if_blockwise()`，用于实现 transform layout a if blockwise 相关逻辑。 其中会调用的重要辅助函数包括 procedural_name, is_grouped, is_blockwise。

### L970-L971 — Comments

```python
 970: 
 971:   @staticmethod
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L972-L975 — Function `transform_layout_B_if_blockwise`

```python
 972:   def transform_layout_B_if_blockwise(operation, layout):
 973:     layout_sfb = f"{operation.procedural_name()}_LayoutSFB"
 974:     layout_sfb = layout_sfb if not is_grouped(operation.gemm_kind) else layout_sfb + "* "
 975:     return layout if not is_blockwise(operation.gemm_kind) else f"cute::tuple<{layout}, {layout_sfb}>"
```
**EN:** Defines `transform_layout_B_if_blockwise()`, which implements the transform layout b if blockwise logic. Key helper calls include procedural_name, is_grouped, is_blockwise.

**CN:** 定义 `transform_layout_B_if_blockwise()`，用于实现 transform layout b if blockwise 相关逻辑。 其中会调用的重要辅助函数包括 procedural_name, is_grouped, is_blockwise。

### L976-L977 — Comments

```python
 976: 
 977:   @staticmethod
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L978-L983 — Function `problem_shape`

```python
 978:   def problem_shape(operation):
 979:     gemm_shape_type = "cute::Shape<int,int,int,int>"
 980:     grouped_gemm_shape_type = "cute::Shape<int,int,int>"
 981:     grouped_gemm_shape_type = "cutlass::gemm::GroupProblemShape<" + grouped_gemm_shape_type + ">"
 982: 
 983:     return gemm_shape_type if not is_grouped(operation.gemm_kind) else grouped_gemm_shape_type
```
**EN:** Defines `problem_shape()`, which implements the problem shape logic. Key helper calls include is_grouped.

**CN:** 定义 `problem_shape()`，用于实现 problem shape 相关逻辑。 其中会调用的重要辅助函数包括 is_grouped。

### L985-L1213 — Function `emit`

```python
 985:   def emit(self, operation):
 986:     _LOGGER.debug("*** EmitGemmConfigurationLibrary::emit(operation)")
 987:     _LOGGER.debug("***   operation.procedural_name(): " + operation.procedural_name())
 988:     _LOGGER.debug("***   tile_shape: " + str(operation.tile_description.tile_shape))
 989:     _LOGGER.debug("***   warp_count: " + str(operation.tile_description.warp_count))
 990: 
 991:     opcode_class_main = operation.tile_description.math_instruction.opcode_class
 992:     opcode_class_epi = opcode_class_main
 993:     
 994:     tile_shape = operation.tile_description.tile_shape
 995:     instruction_shape = operation.tile_description.math_instruction.instruction_shape
 996:     cluster_m = operation.tile_description.cluster_shape[0]
 997:     cluster_n = operation.tile_description.cluster_shape[1]
 998:     cta_n = tile_shape[1] // cluster_n if cluster_n > 0 else tile_shape[1]
 999:     tile_shape_m, tile_shape_n, tile_shape_k = operation.get_collective_tile_shape()
1000:  
1001:     # stage count set to zero indicates builder automatic stage selection
1002:     if operation.tile_description.stages > 0:
1003:       stage_count_string = f"cutlass::gemm::collective::StageCount<{str(operation.tile_description.stages)}>"
1004:     elif opcode_class_main == OpcodeClass.SparseTensorOp and operation.arch == 100:
1005:       stage_count_string = f"cutlass::gemm::collective::StageCountAutoCarveoutEpi<{str(operation.procedural_name())}_epilogue>"
1006:     elif operation.is_xe:
1007:       stage_count_string = "cutlass::gemm::collective::StageCountAuto"
1008:     else:
1009:       stage_count_string = f"cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(sizeof(typename {str(operation.procedural_name())}_epilogue::SharedStorage))>"
1010: 
1011:     epi_tile_mn = "cutlass::epilogue::collective::EpilogueTileAuto"
1012: 
1013:     instance_layout_A, instance_layout_B, instance_layout_C , instance_layout_D = \
1014:       (operation.A.layout, operation.B.layout, operation.C.layout, operation.D.layout)
1015: 
1016:     # 3.0 profiler integration only supports trivial epilogues for now
1017:     epilogue_vector_length = 1
1018: 
1019:     # Support built-in epilogue functors or user-defined functions
1020:     if isinstance(operation.epilogue_functor, enum.Enum):
1021:       values = {
1022:         'element_epilogue': str(DataTypeTag[operation.element_epilogue]),
1023:         'epilogue_functor': EpilogueFunctor3xTag[operation.epilogue_functor],
1024:       }
1025:       epilogue_functor = SubstituteTemplate(self.builtin_epilogue_functor_template, values)
1026:       
1027:       if is_block_scaled(operation.gemm_kind) and operation.ScaleFactorD.element != DataType.void:
1028:         epilogue_functor =  self.emit_block_scale_epilogue_functor(operation)
1029: 
1030: 
1031:     else:
1032:       epilogue_functor = self.epilogue_functor.emit_declaration()
1033: 
1034:       if is_block_scaled(operation.gemm_kind) and operation.ScaleFactorD.element != DataType.void:
1035:         epilogue_functor =  self.emit_block_scale_epilogue_functor(operation)
1036: 
1037:     #
1038:     # Cutlass3x complex kernels' ElementA(B) is a tuple in collective mainloop builder, e.g. cute::tuple<Element, Transform>, Transform : cute::identity / cute::conjugate.
1039:     element_a = DataTypeTag[operation.A.element] if not operation.is_complex() else f"cute::tuple<{str(DataTypeTag[operation.A.element])},{str(ComplexTransformTag3x[operation.A.complex_transform])}>"
1040:     element_b = DataTypeTag[operation.B.element] if not operation.is_complex() else f"cute::tuple<{str(DataTypeTag[operation.B.element])},{str(ComplexTransformTag3x[operation.B.complex_transform])}>"
1041:     epilogue_schedule_type = EpilogueScheduleTag[operation.epilogue_schedule]
1042:     
1043:     if opcode_class_main == OpcodeClass.BlockScaledTensorOp:
1044:       grouped = is_grouped(operation.gemm_kind)
1045:       if cta_n == 256 and operation.kernel_schedule == to_grouped_schedule(KernelScheduleType.Nvf4TmaWarpSpecialized1SmSm100, grouped):
1046:         epi_tile_mn = "cute::Shape<cute::_128,cute::_64>"
1047:         if is_tma_epilogue(operation.epilogue_schedule):
1048:           epilogue_schedule_type = EpilogueScheduleTag[to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecialized1Sm, grouped)]
1049:       if cta_n == 256 and operation.kernel_schedule == to_grouped_schedule(KernelScheduleType.Nvf4TmaWarpSpecialized2SmSm100, grouped):
1050:         epi_tile_mn = "cute::Shape<cute::_128,cute::_64>"
1051:         if is_tma_epilogue(operation.epilogue_schedule):
1052:           epilogue_schedule_type = EpilogueScheduleTag[to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecialized2Sm, grouped)]
1053:       # SM103 FP4 Ultra
1054:       is_sm103_fp4_ultra_1sm_kernel_schedule = operation.kernel_schedule in [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103, grouped),
1055:                                                                              to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103, grouped),
1056:                                                                              to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103DisablePrefetch, grouped),
1057:                                                                              to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103DisablePrefetch, grouped),
1058:                                                                              to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103TmaPrefetch, grouped),
1059:                                                                              to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103TmaPrefetch, grouped)
1060:                                                                              ]
1061:       is_sm103_fp4_ultra_2sm_kernel_schedule = operation.kernel_schedule in [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103, grouped),
1062:                                                                              to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103, grouped),
1063:                                                                              to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103DisablePrefetch, grouped),
1064:                                                                              to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103DisablePrefetch, grouped),
1065:                                                                              to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103TmaPrefetch, grouped),
1066:                                                                              to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103TmaPrefetch, grouped)
1067:                                                                              ]
1068:       if cta_n == 256 and is_sm103_fp4_ultra_1sm_kernel_schedule:
1069:         epi_tile_mn = "cute::Shape<cute::_128,cute::_64>"
1070:         if is_tma_epilogue(operation.epilogue_schedule):
1071:           epilogue_schedule_type = EpilogueScheduleTag[to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecialized1Sm, grouped)]
1072:       if cta_n == 256 and is_sm103_fp4_ultra_2sm_kernel_schedule:
1073:         epi_tile_mn = "cute::Shape<cute::_128,cute::_64>"
1074:         if is_tma_epilogue(operation.epilogue_schedule):
1075:           epilogue_schedule_type = EpilogueScheduleTag[to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecialized2Sm, grouped)]
1076: 
1077:       element_a = f'cute::tuple<{str(element_a)},{str(DataTypeTag[operation.ScaleFactorA])}>'
1078:       element_b = f'cute::tuple<{str(element_b)},{str(DataTypeTag[operation.ScaleFactorB])}>'
1079: 
1080:     alignment_c = get_tma_alignment(operation.C.element) \
1081:                   if is_tma_epilogue(operation.epilogue_schedule) and opcode_class_epi != OpcodeClass.Simt \
1082:                   else operation.C.alignment
1083:     alignment_d = get_tma_alignment(operation.D.element) \
1084:                   if is_tma_epilogue(operation.epilogue_schedule) and opcode_class_epi != OpcodeClass.Simt \
1085:                   else operation.D.alignment
1086: 
1087:     operation_name_str = operation.procedural_name()
1088:     layout_a_str = LayoutTag[instance_layout_A]
1089:     layout_b_str = LayoutTag[instance_layout_B]
1090:     mixed_dtype_prepare_code = ""
1091:     if operation.mixed_input_mode != None:
1092:       A_dtype = operation.A.element
1093:       B_dtype = operation.B.element
1094:       A_dtype_bits = DataTypeSize[A_dtype]
1095:       B_dtype_bits = DataTypeSize[B_dtype]
1096:       is_A_dtype_narrow = A_dtype_bits < B_dtype_bits
1097:       if is_A_dtype_narrow:
1098:         narrow_dtype, wide_dtype = (A_dtype, B_dtype)
1099:         narrow_dtype_bits, wide_dtype_bits = (A_dtype_bits, B_dtype_bits)
1100:       else:
1101:         narrow_dtype, wide_dtype = (B_dtype, A_dtype)
1102:         narrow_dtype_bits, wide_dtype_bits = (B_dtype_bits, A_dtype_bits)
1103: 
1104:       narrow_tag = DataTypeTag[narrow_dtype]
1105:       wide_tag   = DataTypeTag[wide_dtype]
1106:       scale_tag  = DataTypeTag[wide_dtype]
1107:       zero_tag   = DataTypeTag[wide_dtype]
1108: 
1109:       do_shuffle = False
1110:       value_shuffle_str = ""
1111:       if narrow_dtype_bits == 4 and wide_dtype_bits == 16:
1112:         value_shuffle_str = "cute::Layout<cute::Shape<cute::_2,cute::_4>, cute::Stride<cute::_4,cute::_1>>"
1113:         do_shuffle = True
1114:       if narrow_dtype_bits == 8 and wide_dtype_bits == 16:
1115:         value_shuffle_str = "cute::Layout<cute::Shape<cute::_2,cute::_2>, cute::Stride<cute::_2,cute::_1>>"
1116:         do_shuffle = True
1117:       do_shuffle = operation.mixed_input_shuffle and do_shuffle
1118: 
1119:       if do_shuffle:
1120:         if is_A_dtype_narrow:
1121:           stride_narrow_str = f"cutlass::detail::TagToStrideA_t<{layout_a_str}>"
1122:           layout_a_str = f"{operation_name_str}_LayoutNarrowReordered"
1123:         else:
1124:           stride_narrow_str = f"cutlass::detail::TagToStrideB_t<{layout_b_str}>"
1125:           layout_b_str = f"{operation_name_str}_LayoutNarrowReordered"
1126:         # The {operation_name_str}_ prefixs in mixed_dtype_prepare_code and
1127:         # layout_{a, b}_str are to prevent errors in Windows platform unity build
1128:         mixed_dtype_prepare_code = f"""
1129: using {operation_name_str}_StrideNarrow = {stride_narrow_str};
1130: using {operation_name_str}_ValueShuffle = {value_shuffle_str};
1131: static constexpr int {operation_name_str}_NumShuffleAtoms = 1;
1132: using {operation_name_str}_MmaAtomShape = cute::Layout<cute::Shape<cute::_1, cute::Int<{operation_name_str}_NumShuffleAtoms>>>;
1133: using {operation_name_str}_LayoutAtomQuant = decltype(cutlass::compute_memory_reordering_atom<{wide_tag}, {operation_name_str}_MmaAtomShape, {operation_name_str}_ValueShuffle>());
1134: using {operation_name_str}_LayoutNarrowReordered = decltype(cute::tile_to_shape({operation_name_str}_LayoutAtomQuant{{}}, cute::Layout<cute::Shape<int,int,int>, {operation_name_str}_StrideNarrow>{{}}));
1135:         """
1136: 
1137:       mixed_input_modes_to_element = {
1138:         MixedInputMode.ConvertOnly: narrow_tag,
1139:         MixedInputMode.ScaleOnly: f"cute::tuple<{narrow_tag}, {scale_tag}>",
1140:         MixedInputMode.ScaleWithZeroPoint: f"cute::tuple<{narrow_tag}, {scale_tag}, {zero_tag}>"
1141:       }
1142:       narrow_element = mixed_input_modes_to_element.get(operation.mixed_input_mode, narrow_tag)
1143: 
1144:       if narrow_dtype == DataType.s4 and (wide_dtype == DataType.e4m3 or wide_dtype == DataType.e5m2):
1145:         narrow_element = f"cute::tuple<{narrow_tag}, cutlass::Array<{scale_tag}, 8>>"
1146: 
1147:       if is_A_dtype_narrow:
1148:         element_a = narrow_element
1149:       else:
1150:         element_b = narrow_element
1151: 
1152:     blockwise_prepare_code = ""
1153:     if is_blockwise(operation.gemm_kind):
1154:       sfm_vec_size = operation.ScaleFactorMVecSize
1155:       sfn_vec_size = operation.ScaleFactorNVecSize
1156:       sfk_vec_size = operation.ScaleFactorKVecSize
1157:       blockwise_prepare_code = f"""
1158: using {operation_name_str}_ScaleConfig = cutlass::detail::Sm{operation.arch}BlockwiseScaleConfig<{sfm_vec_size}, {sfn_vec_size}, {sfk_vec_size}>;
1159: using {operation_name_str}_LayoutSFA = decltype({operation_name_str}_ScaleConfig::deduce_layoutSFA());
1160: using {operation_name_str}_LayoutSFB = decltype({operation_name_str}_ScaleConfig::deduce_layoutSFB());
1161:       """
1162: 
1163:     values = {
1164:       'operation_name': operation_name_str,
1165:       'operation_suffix': self.operation_suffix,
1166:       'problem_shape': self.problem_shape(operation),
1167:       'element_a': element_a,
1168:       'layout_a': self.transform_layout_A_if_blockwise(operation, self.pointerize_if_grouped(operation, layout_a_str)),
1169:       'element_b': element_b,
1170:       'layout_b': self.transform_layout_B_if_blockwise(operation, self.pointerize_if_grouped(operation, layout_b_str)),
1171:       'element_c': DataTypeTag[operation.C.element],
1172:       'layout_c': self.pointerize_if_grouped(operation, LayoutTag[instance_layout_C]),
1173:       'element_d': DataTypeTag[operation.D.element],
1174:       'layout_d': self.pointerize_if_grouped(operation, LayoutTag[instance_layout_D]),
1175:       'element_accumulator': DataTypeTag[operation.accumulator_type()],
1176:       'opcode_class_main': OpcodeClassTag[opcode_class_main],
1177:       'opcode_class_epi': OpcodeClassTag[opcode_class_epi],
1178:       'arch': "cutlass::arch::Sm%d" % operation.arch,
1179:       'tile_shape_m': str(tile_shape_m),
1180:       'tile_shape_n': str(tile_shape_n),
1181:       'tile_shape_k': str(tile_shape_k),
1182:       'cluster_shape_m': 'cute::_' + str(operation.tile_description.cluster_shape[0]) if operation.tile_description.cluster_shape[0] > 0 else "int",
1183:       'cluster_shape_n': 'cute::_' + str(operation.tile_description.cluster_shape[1]) if operation.tile_description.cluster_shape[1] > 0 else "int",
1184:       'cluster_shape_k': 'cute::_' + str(operation.tile_description.cluster_shape[2]) if operation.tile_description.cluster_shape[2] > 0 else "int",
1185:       'instruction_shape_m': str(instruction_shape[0]),
1186:       'instruction_shape_n': str(instruction_shape[1]),
1187:       'instruction_shape_k': str(instruction_shape[2]),
1188:       'kernel_schedule' : str(KernelScheduleTag[operation.kernel_schedule]),
1189:       'epilogue_schedule' : str(epilogue_schedule_type),
1190:       'epi_tile_mn' : epi_tile_mn,
1191:       'epilogue_functor': epilogue_functor,
1192:       'stages': stage_count_string,
1193:       'align_a': str(operation.A.alignment),
1194:       'align_b': str(operation.B.alignment),
1195:       'align_c': str(alignment_c),
1196:       'align_d': str(alignment_d),
1197:       'transform_a': ComplexTransformTag[operation.A.complex_transform],
1198:       'transform_b': ComplexTransformTag[operation.B.complex_transform],
1199:       'math_operation': MathOperationTag[operation.tile_description.math_instruction.math_operation],
1200:       'epilogue_vector_length': str(epilogue_vector_length),
1201:       'element_epilogue': str(DataTypeTag[operation.element_epilogue]),
1202:       'tile_scheduler': str(TileSchedulerTag[operation.tile_scheduler]),
1203:       'mixed_dtype_prepare_code': mixed_dtype_prepare_code,
1204:       'blockwise_prepare_code' : blockwise_prepare_code
1205:     }
1206: 
1207:     # Overriding values for Intel Xe architectures
1208:     if operation.is_xe:
1209:       # Use specific compute capability for Intel Xe GPUs
1210:       # e.g., cutlass::arch::Xe20 for BMG, cutlass::arch::Xe12 for PVC
1211:       values['arch'] = "cutlass::arch::Xe%d" % operation.arch
1212: 
1213:     return SubstituteTemplate(self.gemm_template, values)
```
**EN:** Defines `emit()`, which emits . Key helper calls include debug, get_collective_tile_shape, isinstance, procedural_name, is_blockwise, SubstituteTemplate.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 debug, get_collective_tile_shape, isinstance, procedural_name, is_blockwise, SubstituteTemplate。

### L1214-L1217 — Comments

```python
1214: 
1215: ###################################################################################################
1216: 
1217: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1218-L1218 — Class `EmitGemmPlanarComplexInstance`

```python
1218: class EmitGemmPlanarComplexInstance:
```
**EN:** Introduces class `EmitGemmPlanarComplexInstance`. The docstring says: "Responsible for emitting a CUTLASS template definition"; the class body implements that role.

**CN:** 引入类 `EmitGemmPlanarComplexInstance`。文档字符串说明其用途为：“Responsible for emitting a CUTLASS template definition”；类体负责实现这一职责。

### L1219-L1219 — Documentation/setup

```python
1219:   ''' Responsible for emitting a CUTLASS template definition'''
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L1221-L1249 — Function `__init__`

```python
1221:   def __init__(self, operation_suffix = ''):
1222:     self.operation_suffix = operation_suffix
1223:     self.includes = []
1224:     self.template = """
1225:   // Gemm operator ${operation_name}
1226:   using Operation_${operation_name} = typename cutlass::gemm::kernel::DefaultGemmPlanarComplexUniversal<
1227:     ${element_a}, ${layout_a}, ${transform_a}, ${alignment_a},
1228:     ${element_b}, ${layout_b}, ${transform_b}, ${alignment_b},
1229:     ${element_c}, cutlass::layout::RowMajor,
1230:     ${element_accumulator},
1231:     ${opcode_class},
1232:     ${arch},
1233:     cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
1234:     cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
1235:     cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
1236:     cutlass::epilogue::thread::LinearCombinationPlanarComplex<
1237:       ${element_c},
1238:       ${alignment_c},
1239:       ${element_accumulator},
1240:       ${element_epilogue}
1241:     >,
1242:     cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>,
1243:     ${stages},
1244:     ${math_operator}
1245:   >::GemmKernel;
1246: 
1247:   struct ${operation_name} :
1248:     public Operation_${operation_name} { };
1249: """
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L1250-L1251 — Comments

```python
1250: 
1251:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1252-L1259 — Function `instance_template`

```python
1252:   def instance_template(self):
1253:     return """
1254: ${compile_guard_start}
1255:   manifest.append(new ${gemm_kind}<
1256:     cutlass::gemm::device::GemmUniversalAdapter<${operation_name}>
1257:   >("${operation_name}"));
1258: ${compile_guard_end}
1259: """
```
**EN:** Defines `instance_template()`, which implements the instance template logic.

**CN:** 定义 `instance_template()`，用于实现 instance template 相关逻辑。

### L1260-L1261 — Comments

```python
1260: 
1261:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1262-L1300 — Function `emit`

```python
1262:   def emit(self, operation):
1263: 
1264:     warp_shape = [operation.tile_description.threadblock_shape[idx] // operation.tile_description.warp_count[idx] for idx in range(3)]
1265: 
1266:     # exchange and transpose A and B types, layouts, and complex transforms since the C layout is row-major
1267:     transposed_layout_A = TransposedLayout[operation.A.layout]
1268:     transposed_layout_B = TransposedLayout[operation.B.layout]
1269: 
1270:     values = {
1271:       'operation_name': operation.procedural_name(),
1272:       'element_a': DataTypeTag[operation.B.element],
1273:       'layout_a': LayoutTag[transposed_layout_B],
1274:       'transform_a': ComplexTransformTag[operation.B.complex_transform],
1275:       'alignment_a': str(operation.B.alignment),
1276:       'element_b': DataTypeTag[operation.A.element],
1277:       'layout_b': LayoutTag[transposed_layout_A],
1278:       'transform_b': ComplexTransformTag[operation.A.complex_transform],
1279:       'alignment_b': str(operation.A.alignment),
1280:       'element_c': DataTypeTag[operation.C.element],
1281:       'layout_c': LayoutTag[operation.C.layout],
1282:       'element_accumulator': DataTypeTag[operation.tile_description.math_instruction.element_accumulator],
1283:       'opcode_class': OpcodeClassTag[operation.tile_description.math_instruction.opcode_class],
1284:       'arch': "cutlass::arch::Sm%d" % operation.arch,
1285:       'threadblock_shape_m': str(operation.tile_description.threadblock_shape[0]),
1286:       'threadblock_shape_n': str(operation.tile_description.threadblock_shape[1]),
1287:       'threadblock_shape_k': str(operation.tile_description.threadblock_shape[2]),
1288:       'warp_shape_m': str(warp_shape[0]),
1289:       'warp_shape_n': str(warp_shape[1]),
1290:       'warp_shape_k': str(warp_shape[2]),
1291:       'instruction_shape_m': str(operation.tile_description.math_instruction.instruction_shape[0]),
1292:       'instruction_shape_n': str(operation.tile_description.math_instruction.instruction_shape[1]),
1293:       'instruction_shape_k': str(operation.tile_description.math_instruction.instruction_shape[2]),
1294:       'alignment_c': str(operation.C.alignment),
1295:       'element_epilogue': str(DataTypeTag[operation.element_epilogue]),
1296:       'stages': str(operation.tile_description.stages),
1297:       'math_operator': 'cutlass::arch::OpMultiplyAdd'
1298:     }
1299: 
1300:     return SubstituteTemplate(self.template, values)
```
**EN:** Defines `emit()`, which emits . Key helper calls include SubstituteTemplate, procedural_name, str, range.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 SubstituteTemplate, procedural_name, str, range。

### L1301-L1304 — Comments

```python
1301: 
1302: ###################################################################################################
1303: 
1304: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1305-L1305 — Class `EmitGemmPlanarComplexArrayInstance`

```python
1305: class EmitGemmPlanarComplexArrayInstance:
```
**EN:** Introduces class `EmitGemmPlanarComplexArrayInstance`. The docstring says: "Responsible for emitting a CUTLASS template definition"; the class body implements that role.

**CN:** 引入类 `EmitGemmPlanarComplexArrayInstance`。文档字符串说明其用途为：“Responsible for emitting a CUTLASS template definition”；类体负责实现这一职责。

### L1306-L1306 — Documentation/setup

```python
1306:   ''' Responsible for emitting a CUTLASS template definition'''
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L1308-L1335 — Function `__init__`

```python
1308:   def __init__(self, operation_suffix = ''):
1309:     self.operation_suffix = operation_suffix
1310:     self.includes = []
1311:     self.template = """
1312:   // Gemm operator ${operation_name}
1313:   using Operation_${operation_name} = typename cutlass::gemm::kernel::DefaultGemmPlanarComplexUniversal<
1314:     ${element_a}, ${layout_a}, ${transform_a}, ${alignment_a},
1315:     ${element_b}, ${layout_b}, ${transform_b}, ${alignment_b},
1316:     ${element_c}, cutlass::layout::RowMajor,
1317:     ${element_accumulator},
1318:     ${opcode_class},
1319:     ${arch},
1320:     cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
1321:     cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
1322:     cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
1323:     cutlass::epilogue::thread::LinearCombinationPlanarComplex<
1324:       ${element_c},
1325:       ${alignment_c},
1326:       ${element_accumulator},
1327:       ${element_epilogue}
1328:     >,
1329:     cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>,
1330:     ${stages},
1331:     ${math_operator}
1332:   >::GemmArrayKernel;
1333: 
1334:   struct ${operation_name} : public Operation_${operation_name} { };
1335: """
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L1336-L1337 — Comments

```python
1336: 
1337:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1338-L1345 — Function `instance_template`

```python
1338:   def instance_template(self):
1339:     return """
1340: ${compile_guard_start}
1341:   manifest.append(new ${gemm_kind}<
1342:     cutlass::gemm::device::GemmUniversalAdapter<${operation_name}>
1343:   >("${operation_name}"));
1344: ${compile_guard_end}
1345: """
```
**EN:** Defines `instance_template()`, which implements the instance template logic.

**CN:** 定义 `instance_template()`，用于实现 instance template 相关逻辑。

### L1346-L1347 — Comments

```python
1346: 
1347:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1348-L1386 — Function `emit`

```python
1348:   def emit(self, operation):
1349: 
1350:     warp_shape = [operation.tile_description.threadblock_shape[idx] // operation.tile_description.warp_count[idx] for idx in range(3)]
1351: 
1352:     # exchange and transpose A and B types, layouts, and complex transforms since the C layout is row-major
1353:     transposed_layout_A = TransposedLayout[operation.A.layout]
1354:     transposed_layout_B = TransposedLayout[operation.B.layout]
1355: 
1356:     values = {
1357:       'operation_name': operation.procedural_name(),
1358:       'element_a': DataTypeTag[operation.B.element],
1359:       'layout_a': LayoutTag[transposed_layout_B],
1360:       'transform_a': ComplexTransformTag[operation.B.complex_transform],
1361:       'alignment_a': str(operation.B.alignment),
1362:       'element_b': DataTypeTag[operation.A.element],
1363:       'layout_b': LayoutTag[transposed_layout_A],
1364:       'transform_b': ComplexTransformTag[operation.A.complex_transform],
1365:       'alignment_b': str(operation.A.alignment),
1366:       'element_c': DataTypeTag[operation.C.element],
1367:       'layout_c': LayoutTag[operation.C.layout],
1368:       'element_accumulator': DataTypeTag[operation.tile_description.math_instruction.element_accumulator],
1369:       'opcode_class': OpcodeClassTag[operation.tile_description.math_instruction.opcode_class],
1370:       'arch': "cutlass::arch::Sm%d" % operation.arch,
1371:       'threadblock_shape_m': str(operation.tile_description.threadblock_shape[0]),
1372:       'threadblock_shape_n': str(operation.tile_description.threadblock_shape[1]),
1373:       'threadblock_shape_k': str(operation.tile_description.threadblock_shape[2]),
1374:       'warp_shape_m': str(warp_shape[0]),
1375:       'warp_shape_n': str(warp_shape[1]),
1376:       'warp_shape_k': str(warp_shape[2]),
1377:       'instruction_shape_m': str(operation.tile_description.math_instruction.instruction_shape[0]),
1378:       'instruction_shape_n': str(operation.tile_description.math_instruction.instruction_shape[1]),
1379:       'instruction_shape_k': str(operation.tile_description.math_instruction.instruction_shape[2]),
1380:       'alignment_c': str(operation.C.alignment),
1381:       'element_epilogue': str(DataTypeTag[operation.element_epilogue]),
1382:       'stages': str(operation.tile_description.stages),
1383:       'math_operator': 'cutlass::arch::OpMultiplyAdd'
1384:     }
1385: 
1386:     return SubstituteTemplate(self.template, values)
```
**EN:** Defines `emit()`, which emits . Key helper calls include SubstituteTemplate, procedural_name, str, range.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 SubstituteTemplate, procedural_name, str, range。

### L1387-L1390 — Comments

```python
1387: 
1388: ###################################################################################################
1389: 
1390: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1391-L1391 — Class `EmitGemmGroupedInstance`

```python
1391: class EmitGemmGroupedInstance:
```
**EN:** Introduces class `EmitGemmGroupedInstance`. The docstring says: "Responsible for emitting a CUTLASS template definition"; the class body implements that role.

**CN:** 引入类 `EmitGemmGroupedInstance`。文档字符串说明其用途为：“Responsible for emitting a CUTLASS template definition”；类体负责实现这一职责。

### L1392-L1392 — Documentation/setup

```python
1392:   ''' Responsible for emitting a CUTLASS template definition'''
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L1394-L1438 — Function `__init__`

```python
1394:   def __init__(self, operation_suffix = ''):
1395:     self.operation_suffix = operation_suffix
1396:     self.includes = [
1397:       "cutlass/cutlass.h",
1398:       "cutlass/numeric_types.h",
1399:       "cutlass/arch/arch.h",
1400:       "cutlass/arch/mma.h",
1401:       "cutlass/layout/matrix.h",
1402:       "cutlass/gemm/device/gemm.h",
1403:       "cutlass/gemm/kernel/gemm_grouped.h",
1404:       "cutlass/gemm/kernel/default_gemm_grouped.h",
1405:       "cutlass/gemm/device/gemm_grouped.h"
1406:     ]
1407:     self.builtin_epilogue_functor_template = \
1408: """${epilogue_functor}<
1409:       ${element_c},
1410:       ${epilogue_vector_length},
1411:       ${element_accumulator},
1412:       ${element_epilogue}
1413:     >"""
1414: 
1415:     self.gemm_template = """
1416: // Gemm operator ${operation_name}
1417: using ${operation_name}_base =
1418:   typename cutlass::gemm::kernel::DefaultGemmGrouped<
1419:     ${element_a}, ${layout_a}, ${transform_a}, ${align_a},
1420:     ${element_b}, ${layout_b}, ${transform_b}, ${align_b},
1421:     ${element_c}, ${layout_c},
1422:     ${element_accumulator},
1423:     ${opcode_class},
1424:     ${arch},
1425:     cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
1426:     cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
1427:     cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
1428:     ${epilogue_functor},
1429:     ${swizzling_functor},
1430:     ${stages},
1431:     ${scheduler_mode},
1432:     ${math_operation}
1433: >::GemmKernel;
1434: 
1435: // Define named type
1436: struct ${operation_name}${operation_suffix} :
1437:   public ${operation_name}_base { };
1438: """
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L1439-L1440 — Comments

```python
1439: 
1440:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1441-L1448 — Function `instance_template`

```python
1441:   def instance_template(self):
1442:     return """
1443: ${compile_guard_start}
1444:   manifest.append(new ${gemm_kind}<
1445:     cutlass::gemm::device::GemmGrouped<${operation_name}>
1446:   >("${operation_name}"));
1447: ${compile_guard_end}
1448: """
```
**EN:** Defines `instance_template()`, which implements the instance template logic.

**CN:** 定义 `instance_template()`，用于实现 instance template 相关逻辑。

### L1449-L1450 — Comments

```python
1449: 
1450:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1451-L1515 — Function `emit`

```python
1451:   def emit(self, operation):
1452: 
1453:     threadblock_shape = operation.tile_description.threadblock_shape
1454:     warp_count = operation.tile_description.warp_count
1455: 
1456:     warp_shape = [threadblock_shape[idx] // warp_count[idx] for idx in range(3)]
1457: 
1458:     transpose_layouts = {
1459:       LayoutType.ColumnMajor: LayoutType.RowMajor,
1460:       LayoutType.RowMajor: LayoutType.ColumnMajor
1461:     }
1462: 
1463:     instance_layout_A, instance_layout_B, instance_layout_C = \
1464:       (operation.A.layout, operation.B.layout, operation.C.layout)
1465:     #
1466: 
1467:     # Support built-in epilogue functors or user-defined functions
1468:     if isinstance(operation.epilogue_functor, enum.Enum):
1469: 
1470:       epilogue_vector_length = \
1471:         min(operation.C.alignment * DataTypeSize[operation.C.element], 128) // DataTypeSize[operation.C.element]
1472: 
1473:       values = {
1474:         'epilogue_vector_length': str(epilogue_vector_length),
1475:         'element_epilogue': str(DataTypeTag[operation.element_epilogue]),
1476:         'epilogue_functor': EpilogueFunctorTag[operation.epilogue_functor],
1477:       }
1478:       epilogue_functor = SubstituteTemplate(self.builtin_epilogue_functor_template, values)
1479:     else:
1480:       epilogue_functor = self.epilogue_functor.emit_declaration()
1481:     #
1482: 
1483:     values = {
1484:       'operation_name': operation.procedural_name(),
1485:       'operation_suffix': self.operation_suffix,
1486:       'element_a': DataTypeTag[operation.A.element],
1487:       'layout_a': LayoutTag[instance_layout_A],
1488:       'element_b': DataTypeTag[operation.B.element],
1489:       'layout_b': LayoutTag[instance_layout_B],
1490:       'element_c': DataTypeTag[operation.C.element],
1491:       'layout_c': LayoutTag[instance_layout_C],
1492:       'element_accumulator': DataTypeTag[operation.accumulator_type()],
1493:       'opcode_class': OpcodeClassTag[operation.tile_description.math_instruction.opcode_class],
1494:       'arch': "cutlass::arch::Sm%d" % operation.arch,
1495:       'threadblock_shape_m': str(operation.tile_description.threadblock_shape[0]),
1496:       'threadblock_shape_n': str(operation.tile_description.threadblock_shape[1]),
1497:       'threadblock_shape_k': str(operation.tile_description.threadblock_shape[2]),
1498:       'warp_shape_m': str(warp_shape[0]),
1499:       'warp_shape_n': str(warp_shape[1]),
1500:       'warp_shape_k': str(warp_shape[2]),
1501:       'instruction_shape_m': str(operation.tile_description.math_instruction.instruction_shape[0]),
1502:       'instruction_shape_n': str(operation.tile_description.math_instruction.instruction_shape[1]),
1503:       'instruction_shape_k': str(operation.tile_description.math_instruction.instruction_shape[2]),
1504:       'epilogue_functor': epilogue_functor,
1505:       'swizzling_functor': SwizzlingFunctorTag[operation.swizzling_functor],
1506:       'stages': str(operation.tile_description.stages),
1507:       'align_a': str(operation.A.alignment),
1508:       'align_b': str(operation.B.alignment),
1509:       'transform_a': ComplexTransformTag[operation.A.complex_transform],
1510:       'transform_b': ComplexTransformTag[operation.B.complex_transform],
1511:       'scheduler_mode': GroupScheduleModeTag[operation.scheduler_mode],
1512:       'math_operation': MathOperationTag[operation.tile_description.math_instruction.math_operation]
1513:     }
1514: 
1515:     return SubstituteTemplate(self.gemm_template, values)
```
**EN:** Defines `emit()`, which emits . Key helper calls include isinstance, SubstituteTemplate, emit_declaration, procedural_name, str, range.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 isinstance, SubstituteTemplate, emit_declaration, procedural_name, str, range。

### L1516-L1522 — Comments

```python
1516: 
1517: ###################################################################################################
1518: #
1519: # Emitters functions for all targets
1520: #
1521: ###################################################################################################
1522: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1523-L1523 — Class `EmitGemmConfigurationLibrary`

```python
1523: class EmitGemmConfigurationLibrary:
```
**EN:** Introduces class `EmitGemmConfigurationLibrary`, an emitter that formats generated code for gemm configuration library.

**CN:** 引入类 `EmitGemmConfigurationLibrary`，它是一个用于格式化并生成 gemm configuration library 代码的 emitter。

### L1524-L1602 — Function `__init__`

```python
1524:   def __init__(self, operation_path, configuration_name):
1525:     self.configuration_name = configuration_name
1526:     
1527:     # Determine file extension based on architecture
1528:     # Intel Xe architectures (12=PVC, 20=BMG) use .cpp, CUDA uses .cu
1529:     # Check if operation_path contains /12/, /20/, xe12, or xe20
1530:     is_xe_arch = any(marker in operation_path for marker in ['/12/', '\\12\\', 'xe12', '/20/', '\\20\\', 'xe20'])
1531:     file_extension = "cpp" if is_xe_arch else "cu"
1532:     self.configuration_path = os.path.join(operation_path, "%s.%s" % (configuration_name, file_extension)).replace('\\', '/')
1533: 
1534:     self.instance_emitter = {
1535:       GemmKind.Gemm: EmitGemmInstance,
1536:       GemmKind.Sparse: EmitSparseGemmInstance,
1537:       GemmKind.Universal: EmitGemmUniversalInstance,
1538:       GemmKind.Universal3x: EmitGemmUniversal3xInstance,
1539:       GemmKind.SparseUniversal3x: EmitGemmUniversal3xInstance,
1540:       GemmKind.BlockScaledUniversal3x: EmitGemmUniversal3xInstance,  
1541:       GemmKind.PlanarComplex: EmitGemmPlanarComplexInstance,
1542:       GemmKind.PlanarComplexArray: EmitGemmPlanarComplexArrayInstance,
1543:       GemmKind.Grouped: EmitGemmGroupedInstance,
1544:       GemmKind.GroupedUniversal3x: EmitGemmUniversal3xInstance,
1545:       GemmKind.GroupedBlockScaledUniversal3x: EmitGemmUniversal3xInstance,
1546:       GemmKind.BlockwiseUniversal3x: EmitGemmUniversal3xInstance,
1547:       GemmKind.GroupedBlockwiseUniversal3x: EmitGemmUniversal3xInstance,
1548:     }
1549: 
1550:     self.gemm_kind_wrappers = {
1551:       GemmKind.Gemm: 'GemmOperation',
1552:       GemmKind.Sparse: 'GemmSparseOperation',
1553:       GemmKind.Universal: 'GemmUniversalOperation',
1554:       GemmKind.Universal3x: 'GemmUniversal3xOperation',
1555:       GemmKind.SparseUniversal3x: 'SparseGemmUniversal3xOperation',
1556:       GemmKind.BlockScaledUniversal3x: 'BlockScaledGemmUniversal3xOperation', 
1557:       GemmKind.PlanarComplex: 'GemmPlanarComplexOperation',
1558:       GemmKind.PlanarComplexArray: 'GemmPlanarComplexArrayOperation',
1559:       GemmKind.Grouped: 'GemmGroupedOperation',
1560:       GemmKind.GroupedUniversal3x: 'GroupedGemmUniversal3xOperation',
1561:       GemmKind.GroupedBlockScaledUniversal3x: 'GroupedBlockScaledGemmUniversal3xOperation',
1562:       GemmKind.BlockwiseUniversal3x: 'BlockwiseGemmUniversal3xOperation',
1563:       GemmKind.GroupedBlockwiseUniversal3x: 'GroupedBlockwiseGemmUniversal3xOperation',
1564:     }
1565: 
1566:     self.wmma_guard_start = "#if defined(CUTLASS_ARCH_WMMA_SM${sm_number}_ENABLED)"
1567: 
1568:     self.separator = """
1569: ///////////////////////////////////////////////////////////////////////////////////////////////////
1570: 
1571: """
1572: 
1573:     self.header_template = """
1574: /*
1575:   Generated by gemm_operation.py - Do not edit.
1576: */
1577: """
1578: 
1579:     self.initialize_function_template = """
1580: 
1581: ///////////////////////////////////////////////////////////////////////////////////////////////////
1582: 
1583: namespace cutlass {
1584: namespace library {
1585: 
1586: ///////////////////////////////////////////////////////////////////////////////////////////////////
1587: 
1588: void initialize_${configuration_name}(Manifest &manifest) {
1589: 
1590: """
1591:     self.epilogue_template = """
1592: 
1593: }
1594: 
1595: ///////////////////////////////////////////////////////////////////////////////////////////////////
1596: 
1597: } // namespace library
1598: } // namespace cutlass
1599: 
1600: ///////////////////////////////////////////////////////////////////////////////////////////////////
1601: 
1602: """
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include any, replace, join.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 any, replace, join。

### L1604-L1631 — Function `__enter__`

```python
1604:   def __enter__(self):
1605:     _LOGGER.debug("*** EmitGemmConfigurationLibrary::__enter__")
1606:     _LOGGER.debug("***   configuration_path (file to write): " +
1607:                   str(self.configuration_path))
1608: 
1609:     self.configuration_file = open(self.configuration_path, "w")
1610:     self.configuration_file.write(self.header_template)
1611:     self.configuration_file.write(self.separator)
1612: 
1613:     self.includes = collections.OrderedDict([
1614:       ("cutlass/cutlass.h", None),
1615:       ("cutlass/library/library.h", None),
1616:       ("cutlass/library/manifest.h", None),
1617:       ("library_internal.h", None),
1618:       ("gemm_operation.h", None),
1619:       ("gemm_operation_3x.hpp", None),
1620:       ("grouped_gemm_operation_3x.hpp", None),
1621:       ("sparse_gemm_operation_3x.hpp", None),
1622:       ("block_scaled_gemm_operation_3x.hpp", None),   
1623:       ("blockwise_gemm_operation_3x.hpp", None),   
1624:       ("cutlass/arch/wmma.h", None),
1625:       ("cutlass/numeric_types.h", None)
1626:     ])
1627:     self.instance_definitions = []
1628:     self.instance_wrappers = []
1629: 
1630:     self.operations = []
1631:     return self
```
**EN:** Defines `__enter__()`, which implements the enter logic. Key helper calls include debug, open, write, OrderedDict, str.

**CN:** 定义 `__enter__()`，用于实现 enter 相关逻辑。 其中会调用的重要辅助函数包括 debug, open, write, OrderedDict, str。

### L1633-L1654 — Function `emit`

```python
1633:   def emit(self, operation):
1634:     _LOGGER.debug("*** EmitGemmConfigurationLibrary::emit(operation)")
1635:     _LOGGER.debug("***   operation.gemm_kind: " + str(operation.gemm_kind))
1636: 
1637:     emitter = self.instance_emitter[operation.gemm_kind]()
1638: 
1639:     for incl in emitter.includes:
1640:       self.includes[incl] = None
1641: 
1642:     self.operations.append(operation)
1643: 
1644:     self.instance_definitions.append(emitter.emit(operation))
1645: 
1646:     self.instance_wrappers.append(SubstituteTemplate(emitter.instance_template(), {
1647:       'configuration_name': self.configuration_name,
1648:       'operation_name': operation.procedural_name(),
1649:       'gemm_kind': self.gemm_kind_wrappers[operation.gemm_kind],
1650:       'compile_guard_start': SubstituteTemplate(self.wmma_guard_start, {'sm_number': str(operation.arch)}) \
1651:         if operation.tile_description.math_instruction.opcode_class == OpcodeClass.WmmaTensorOp else "",
1652:       'compile_guard_end': "#endif" \
1653:         if operation.tile_description.math_instruction.opcode_class == OpcodeClass.WmmaTensorOp else ""
1654:       }))
```
**EN:** Defines `emit()`, which emits . Key helper calls include debug, append, emit, SubstituteTemplate, str, instance_template.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 debug, append, emit, SubstituteTemplate, str, instance_template。

### L1656-L1678 — Function `__exit__`

```python
1656:   def __exit__(self, exception_type, exception_value, traceback):
1657: 
1658:     # Write includes
1659:     for incl, _ in self.includes.items():
1660:       include_statement = "#include \"%s\"\n" % incl
1661:       self.configuration_file.write(include_statement)
1662: 
1663:     self.configuration_file.write(self.separator)
1664: 
1665:     # Write instance definitions in top-level namespace
1666:     for instance_definition in self.instance_definitions:
1667:       self.configuration_file.write(instance_definition)
1668: 
1669:     # Add wrapper objects within initialize() function
1670:     self.configuration_file.write(SubstituteTemplate(self.initialize_function_template, {
1671:       'configuration_name': self.configuration_name
1672:       }))
1673: 
1674:     for instance_wrapper in self.instance_wrappers:
1675:       self.configuration_file.write(instance_wrapper)
1676: 
1677:     self.configuration_file.write(self.epilogue_template)
1678:     self.configuration_file.close()
```
**EN:** Defines `__exit__()`, which implements the exit logic. Key helper calls include items, write, close, SubstituteTemplate.

**CN:** 定义 `__exit__()`，用于实现 exit 相关逻辑。 其中会调用的重要辅助函数包括 items, write, close, SubstituteTemplate。

### L1679-L1681 — Comments

```python
1679: 
1680: ###################################################################################################
1681: ###################################################################################################
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

## Key Concepts / 关键概念

- **EN:** File role: Defines GEMM-family operation descriptors and multiple emitters for different CUTLASS GEMM kernel variants.
- **CN:** 文件角色：定义 GEMM 系列操作描述对象，以及面向不同 CUTLASS GEMM 内核变体的多种生成器。
- **EN:** Main classes: `GemmOperation, GroupedGemmOperation, EmitGemmInstance, EmitSparseGemmInstance, EmitGemmUniversalInstance, EmitGemmUniversal3xInstance, EmitGemmPlanarComplexInstance, EmitGemmPlanarComplexArrayInstance, EmitGemmGroupedInstance, EmitGemmConfigurationLibrary`
- **CN:** 主要类：`GemmOperation, GroupedGemmOperation, EmitGemmInstance, EmitSparseGemmInstance, EmitGemmUniversalInstance, EmitGemmUniversal3xInstance, EmitGemmPlanarComplexInstance, EmitGemmPlanarComplexArrayInstance, EmitGemmGroupedInstance, EmitGemmConfigurationLibrary`
- **EN:** Important constants/tables: `_LOGGER`
- **CN:** 重要常量/表：`_LOGGER`

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: `collections, enum, functools, logging, operator, os.path, shutil, builtins`
- **CN:** 标准库依赖：`collections, enum, functools, logging, operator, os.path, shutil, builtins`
- **EN:** Internal CUTLASS dependencies: `cutlass_library.library, cutlass_library.arch_constants, library, arch_constants`
- **CN:** CUTLASS 内部依赖：`cutlass_library.library, cutlass_library.arch_constants, library, arch_constants`
- **EN:** External/non-stdlib dependencies: `None explicitly imported`
- **CN:** 外部/非标准库依赖：`未显式导入`
