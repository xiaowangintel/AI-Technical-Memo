# conv3x_emitter.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/cutlass_library/conv3x_emitter.py`
- **EN:** Provides reusable emitters for CUTLASS 3.x convolution instances and include directives.
- **CN:** 提供可复用的 CUTLASS 3.x 卷积实例与头文件包含语句生成器。

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

### L33-L51 — Imports and setup

```python
 33: """
 34: Utilities for emitting CUTLASS >= 3 convolution kernels
 35: """
 36: 
 37: import enum
 38: import os.path
 39: import shutil
 40: import logging
 41: from string import Template
 42: 
 43: try:
 44:   import builtins
 45:   if hasattr(builtins, "CUTLASS_IGNORE_PACKAGE") and CUTLASS_IGNORE_PACKAGE == True:
 46:     raise ImportError("Disabling attempt to import cutlass_library")
 47:   from cutlass_library.library import *
 48: except ImportError:
 49:   from library import *
 50: 
 51: _LOGGER = logging.getLogger(__name__)
```
**EN:** Imports and/or re-exports modules such as enum, os.path, shutil, logging, string, Template so later code can reuse shared definitions.

**CN:** 导入或重新导出诸如 enum, os.path, shutil, logging, string, Template 等模块，使后续代码可以复用共享定义。

### L52-L58 — Comments

```python
 52: 
 53: ###################################################################################################
 54: #
 55: # Emits single instances of a CUTLASS device-wide operator
 56: #
 57: ###################################################################################################
 58: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L59-L59 — Class `EmitConv3xInstance`

```python
 59: class EmitConv3xInstance:
```
**EN:** Introduces class `EmitConv3xInstance`, an emitter that formats generated code for conv3x instance.

**CN:** 引入类 `EmitConv3xInstance`，它是一个用于格式化并生成 conv3x instance 代码的 emitter。

### L60-L107 — Function `__init__`

```python
 60:   def __init__(self):
 61:     _LOGGER.debug("*** EmitConv3xInstance::__init__")
 62: 
 63:     # Define epilogue type first, so that the mainloop type
 64:     # can use it with StageCountAutoCarveout.
 65:     self.template = """
 66: 
 67: // CUTLASS >= 3 convolution ${conv_kind_name} kernel instance "${operation_name}"
 68: using ${operation_name}_epilogue =
 69:   typename cutlass::epilogue::collective::CollectiveBuilder<
 70:     ${arch},
 71:     ${opcode_class_epi},
 72:     ${mma_tile_shape},               // mma tile shape
 73:     ${cluster_shape},                // cluster shape
 74:     ${epi_tile_mn},
 75:     ${element_accumulator},
 76:     ${element_compute},
 77:     ${element_c}, ${layout_c}, 128 / cute::sizeof_bits_v<${element_c}>,
 78:     ${element_d}, ${layout_d}, 128 / cute::sizeof_bits_v<${element_d}>,
 79:     ${epilogue_schedule}
 80:     // , class FusionOpOrCallbacks = cutlass::epilogue::fusion::LinearCombination<ElementD,ElementCompute>
 81:   >::CollectiveOp;
 82: 
 83: using ${operation_name}_mainloop =
 84:   typename cutlass::conv::collective::CollectiveBuilder<
 85:     ${arch},
 86:     ${opcode_class_main},
 87:     ${conv_kind},         // kFprop, kDgrad, or kWgrad
 88:     ${element_a}, ${layout_a}, 128 / cute::sizeof_bits_v<${element_a}>,
 89:     ${element_b}, ${layout_b}, 128 / cute::sizeof_bits_v<${element_b}>,
 90:     ${element_accumulator},
 91:     ${mma_tile_shape},        // mma tile shape
 92:     ${cluster_shape},         // cluster shape
 93:     ${stages},
 94:     ${kernel_schedule}
 95:   >::CollectiveOp;
 96: 
 97: using ${operation_name}_problem_shape = cutlass::conv::ConvProblemShape<${conv_kind}, ${operation_name}_mainloop::NumSpatialDimensions>;
 98: 
 99: // Unit tests call this "ConvKernel".
100: // Conv operator ${operation_name}
101: using ${operation_name}_base = cutlass::conv::kernel::ConvUniversal<
102:     ${operation_name}_problem_shape,
103:     ${operation_name}_mainloop,
104:     ${operation_name}_epilogue,
105:     ${tile_scheduler}
106:   >;
107: """
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include debug.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 debug。

### L109-L110 — Function `arch_number_to_type`

```python
109:   def arch_number_to_type(self, arch: int) -> str:
110:     return f"cutlass::arch::Sm{arch}"
```
**EN:** Defines `arch_number_to_type()`, which implements the arch number to type logic.

**CN:** 定义 `arch_number_to_type()`，用于实现 arch number to type 相关逻辑。

### L112-L148 — Function `mma_tile_shape`

```python
112:   def mma_tile_shape(self, operation, cta_m, cta_n, cta_k) -> str:
113:     mma_m = cta_m
114:     mma_n = cta_n
115:     mma_k = cta_k
116: 
117:     if operation.arch >= 100:
118:       # MmaTileShape (mma_m, mma_n, mma_k) is passed to kernel mainloop where
119:       # mma_m = cta_m for 1sm version and mma_m = cta_m * 2 for 2sm version.
120:       # If schedule is auto and cluster size is static and cta_m % 64 == 0 and cluster_m % 2 == 0, 2sm kernel version is allocated,
121:       # otherwise 1sm kernel is allocated.
122:       cta_m_per_mma_instruction = 1
123:       if "2sm" in operation.procedural_name() :
124:         cta_m_per_mma_instruction = 2
125:       elif "1sm" in operation.procedural_name() :
126:         cta_m_per_mma_instruction = 1
127:       elif operation.tile_description.cluster_shape[0] > 0 and operation.tile_description.cluster_shape[0] % 2 == 0 and cta_m % 64 == 0 :
128:         cta_m_per_mma_instruction = 2
129:       mma_m = cta_m * cta_m_per_mma_instruction
130: 
131:     # For all three kinds of convolutions, the tile shape's K mode
132:     # differs from GEMM in that needs to be wrapped in a Shape.
133:     # For Wgrad convolutions specifically,
134:     # the N tile shape also needs to be wrapped in a Shape.
135:     m_template = 'cute::_${mma_m}'
136:     if operation.conv_kind == ConvKind.Wgrad:
137:       n_template = 'cute::Shape<cute::_${mma_n}>'
138:     else:
139:       n_template = 'cute::_${mma_n}'
140:     k_template = 'cute::Shape<cute::_${mma_k}>'
141: 
142:     mma_tile_shape_template = f'cute::Shape<{m_template}, {n_template}, {k_template}>'
143:     values = {
144:       'mma_m': mma_m,
145:       'mma_n': mma_n,
146:       'mma_k': mma_k
147:     }
148:     return Template(mma_tile_shape_template).substitute(values)
```
**EN:** Defines `mma_tile_shape()`, which implements the mma tile shape logic. Key helper calls include substitute, procedural_name, Template.

**CN:** 定义 `mma_tile_shape()`，用于实现 mma tile shape 相关逻辑。 其中会调用的重要辅助函数包括 substitute, procedural_name, Template。

### L150-L160 — Function `cluster_shape`

```python
150:   def cluster_shape(self, operation) -> str:
151:     m_template = 'cute::_${cluster_shape_m}' if operation.tile_description.cluster_shape[0] > 0 else 'int(0)'
152:     n_template = 'cute::_${cluster_shape_n}' if operation.tile_description.cluster_shape[1] > 0 else 'int(0)'
153:     k_template = 'cute::_${cluster_shape_k}' if operation.tile_description.cluster_shape[2] > 0 else 'int(0)'
154:     cluster_shape_template = f'cute::Shape<{m_template}, {n_template}, {k_template}>'
155:     values = {
156:       'cluster_shape_m': operation.tile_description.cluster_shape[0],
157:       'cluster_shape_n': operation.tile_description.cluster_shape[1],
158:       'cluster_shape_k': operation.tile_description.cluster_shape[2],
159:     }
160:     return Template(cluster_shape_template).substitute(values)
```
**EN:** Defines `cluster_shape()`, which implements the cluster shape logic. Key helper calls include substitute, Template.

**CN:** 定义 `cluster_shape()`，用于实现 cluster shape 相关逻辑。 其中会调用的重要辅助函数包括 substitute, Template。

### L162-L168 — Function `stage_count`

```python
162:   def stage_count(self, operation) -> str:
163:     # stages == 0 tells builder to pick the number of stages automatically
164:     namespace_prefix = 'cutlass::conv::collective::'
165:     if operation.tile_description.stages > 0:
166:       return f"{namespace_prefix}StageCount<{str(operation.tile_description.stages)}>"
167:     else:
168:       return f"{namespace_prefix}StageCountAutoCarveout<sizeof(typename {operation.procedural_name()}_epilogue::SharedStorage)>"
```
**EN:** Defines `stage_count()`, which implements the stage count logic. Key helper calls include str, procedural_name.

**CN:** 定义 `stage_count()`，用于实现 stage count 相关逻辑。 其中会调用的重要辅助函数包括 str, procedural_name。

### L170-L236 — Function `emit`

```python
170:   def emit(self, operation) -> str:
171:     _LOGGER.debug("*** EmitConv3xInstance::emit")
172:     _LOGGER.debug("***   operation: procedural_name()=" + operation.procedural_name())
173: 
174:     # Identify the operation as CUTLASS 3 by its is_3x field
175:     if (not hasattr(operation, 'is_3x')) or (not operation.is_3x):
176:       raise RuntimeError("operation must be a CUTLASS 3 operation")
177: 
178:     epi_tile_mn = "cutlass::epilogue::collective::EpilogueTileAuto"
179:     opcode_class_main = OpcodeClassTag[operation.tile_description.math_instruction.opcode_class]
180:     opcode_class_epi = opcode_class_main
181: 
182:     tile_shape = operation.tile_description.tile_shape
183:     cluster_m = operation.tile_description.cluster_shape[0]
184:     cluster_n = operation.tile_description.cluster_shape[1]
185: 
186:     cta_m, cta_n, cta_k = tile_shape
187:     # account for static/dynamic cluster shapes
188:     if operation.arch >= 100:
189:       cta_m = cta_m // cluster_m if cluster_m > 0 else cta_m
190:       cta_n = cta_n // cluster_n if cluster_n > 0 else cta_n
191: 
192:     warp_count = operation.tile_description.warp_count
193:     epilogue_schedule = EpilogueScheduleTag[operation.epilogue_schedule]
194: 
195:     # KernelScheduleTag and TileSchedulerTag both hard-code the
196:     # namespace qualification of KernelScheduleAuto as
197:     # "cutlass::gemm::collective::" (unless the tag is 'void').
198:     #
199:     # For TileSchedulerTag, this namespace is fine, since CUTLASS 3
200:     # convolutions use the same tile schedulers (from the same
201:     # cutlass::gemm::collective namespace) as GEMMs.
202:     kernel_schedule = KernelScheduleTag[operation.kernel_schedule].replace('gemm::', 'conv::')
203:     tile_scheduler = TileSchedulerTag[operation.tile_scheduler]
204:     opcode_class = OpcodeClassTag[operation.tile_description.math_instruction.opcode_class]
205: 
206:     values = {
207:       'operation_name': operation.procedural_name(),
208:       'conv_kind':      ConvKindTag[operation.conv_kind],
209:       'conv_kind_name': ConvKindNames[operation.conv_kind].capitalize(),
210:       'element_a':      DataTypeTag[operation.A.element],
211:       'layout_a':       LayoutTag[operation.A.layout],
212:       'align_a':        int(operation.A.alignment),
213:       'element_b':      DataTypeTag[operation.B.element],
214:       'layout_b':       LayoutTag[operation.B.layout],
215:       'align_b':        int(operation.B.alignment),
216:       'element_c':      DataTypeTag[operation.C.element],
217:       'layout_c':       LayoutTag[operation.C.layout],
218:       'align_c':        int(operation.C.alignment),
219:       'element_d':      DataTypeTag[operation.D.element],
220:       'layout_d':       LayoutTag[operation.D.layout],
221:       'align_d':        int(operation.D.alignment),
222:       'element_accumulator':   DataTypeTag[operation.accumulator_type()],
223:       'opcode_class':          opcode_class,
224:       'arch':                  self.arch_number_to_type(operation.arch),
225:       'mma_tile_shape':        self.mma_tile_shape(operation, cta_m, cta_n, cta_k),
226:       'cluster_shape':         self.cluster_shape(operation),
227:       'opcode_class_epi':      opcode_class_epi,
228:       'opcode_class_main':     opcode_class_main,
229:       'epi_tile_mn':           epi_tile_mn,
230:       'stages':                self.stage_count(operation),
231:       'kernel_schedule':       kernel_schedule,
232:       'epilogue_schedule':     epilogue_schedule,
233:       'tile_scheduler':        tile_scheduler,
234:       'element_compute':       DataTypeTag[operation.element_compute]
235:     }
236:     return Template(self.template).substitute(values)
```
**EN:** Defines `emit()`, which emits . Key helper calls include debug, replace, substitute, RuntimeError, procedural_name, capitalize.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 debug, replace, substitute, RuntimeError, procedural_name, capitalize。

### L238-L238 — Class `EmitConv3xIncludes`

```python
238: class EmitConv3xIncludes:
```
**EN:** Introduces class `EmitConv3xIncludes`, an emitter that formats generated code for conv3x includes.

**CN:** 引入类 `EmitConv3xIncludes`，它是一个用于格式化并生成 conv3x includes 代码的 emitter。

### L239-L245 — Function `__init__`

```python
239:   def __init__(self):
240:     _LOGGER.debug("*** EmitConv3xIncludes::__init__")
241:     self.includes = ['conv_operation_3x.hpp',
242:                      'cutlass/conv/device/conv_universal_adapter.hpp',
243:                      'cutlass/conv/kernel/conv_universal.hpp',
244:                      'cutlass/conv/collective/collective_builder.hpp',
245:                      'cutlass/epilogue/collective/collective_builder.hpp']
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include debug.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 debug。

### L247-L250 — Function `emit`

```python
247:   def emit(self, operation) -> str:
248:     _LOGGER.debug("*** EmitConv3xIncludes::emit")
249:     return '\n'.join(f"#include \"{incl}\"" for incl in self.includes) + \
250:       "\n\n///////////////////////////////////////////////////////////////////////////////////////////////////"
```
**EN:** Defines `emit()`, which emits . Key helper calls include debug, join.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 debug, join。

## Key Concepts / 关键概念

- **EN:** File role: Provides reusable emitters for CUTLASS 3.x convolution instances and include directives.
- **CN:** 文件角色：提供可复用的 CUTLASS 3.x 卷积实例与头文件包含语句生成器。
- **EN:** Main classes: `EmitConv3xInstance, EmitConv3xIncludes`
- **CN:** 主要类：`EmitConv3xInstance, EmitConv3xIncludes`
- **EN:** Important constants/tables: `_LOGGER`
- **CN:** 重要常量/表：`_LOGGER`

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: `enum, os.path, shutil, logging, string, builtins`
- **CN:** 标准库依赖：`enum, os.path, shutil, logging, string, builtins`
- **EN:** Internal CUTLASS dependencies: `cutlass_library.library, library`
- **CN:** CUTLASS 内部依赖：`cutlass_library.library, library`
- **EN:** External/non-stdlib dependencies: `None explicitly imported`
- **CN:** 外部/非标准库依赖：`未显式导入`
