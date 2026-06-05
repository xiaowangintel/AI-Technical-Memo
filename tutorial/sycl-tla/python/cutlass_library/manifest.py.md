# manifest.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/cutlass_library/manifest.py`
- **EN:** Implements manifest emission, option parsing, and operation filtering/grouping for generated libraries.
- **CN:** 实现生成库所需的 manifest 输出、选项解析以及操作过滤/分组逻辑。

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

### L34-L75 — Imports and setup

```python
 34: """
 35: Utilities for filtering CUTLASS library kernels and emitting library intitialization
 36: and building code
 37: """
 38: 
 39: import enum
 40: import logging
 41: import os.path
 42: import shutil
 43: 
 44: try:
 45:   import builtins
 46:   if hasattr(builtins, "CUTLASS_IGNORE_PACKAGE") and CUTLASS_IGNORE_PACKAGE == True:
 47:     raise ImportError("Disabling attempt to import cutlass_library")
 48:   from cutlass_library.library import *
 49:   from cutlass_library.gemm_operation import *
 50:   from cutlass_library.rank_k_operation import *
 51:   from cutlass_library.rank_2k_operation import *
 52:   from cutlass_library.trmm_operation import *
 53:   from cutlass_library.symm_operation import *
 54:   from cutlass_library.conv2d_operation import *
 55:   from cutlass_library.conv3d_operation import *
 56: except ImportError:
 57:   from library import *
 58:   from gemm_operation import *
 59:   from rank_k_operation import *
 60:   from rank_2k_operation import *
 61:   from trmm_operation import *
 62:   from symm_operation import *
 63:   from conv2d_operation import *
 64:   from conv3d_operation import *
 65: 
 66: ###################################################################################################
 67: _LOGGER = logging.getLogger(__name__)
 68: 
 69: ###################################################################################################
 70: # Import architecture range constants from shared module
 71: ###################################################################################################
 72: try:
 73:   from cutlass_library.arch_constants import INTEL_XE_ARCH_MIN, INTEL_XE_ARCH_MAX, CUDA_ARCH_MIN, INTEL_XE12, INTEL_XE20
 74: except ImportError:
 75:   from arch_constants import INTEL_XE_ARCH_MIN, INTEL_XE_ARCH_MAX, CUDA_ARCH_MIN, INTEL_XE12, INTEL_XE20
```
**EN:** Imports and/or re-exports modules such as enum, logging, os.path, shutil so later code can reuse shared definitions.

**CN:** 导入或重新导出诸如 enum, logging, os.path, shutil 等模块，使后续代码可以复用共享定义。

### L76-L79 — Comments

```python
 76: 
 77: ###################################################################################################
 78: 
 79: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L80-L80 — Class `EmitOperationKindAll`

```python
 80: class EmitOperationKindAll:
```
**EN:** Introduces class `EmitOperationKindAll`. The docstring says: "Emit the OperationKind-level CUTLASS library initialization code."; the class body implements that role.

**CN:** 引入类 `EmitOperationKindAll`。文档字符串说明其用途为：“Emit the OperationKind-level CUTLASS library initialization code.”；类体负责实现这一职责。

### L81-L99 — Documentation/setup

```python
 81:   """
 82:   Emit the OperationKind-level CUTLASS library initialization code.
 83:   The code is generated in the {generated_path}/{operation_kind} directory
 84:   (e.g., tools/library/generated/gemm in the build directory,
 85:   for OperationKind=Gemm), in the all_{operation_kind}_operations.cu file
 86:   (e.g., all_gemm_operations.cu for OperationKind=Gemm).
 87:   That file declares several functions in namespace cutlass::library.
 88:   The functions all have this form,
 89: 
 90:   void initialize_{configuration_name}(Manifest& manifest);
 91: 
 92:   The file also _defines_ the following function in that namespace.
 93: 
 94:   void initialize_all_{operation_kind}_operations(Manifest& manifest);
 95: 
 96:   That function calls all of the functions declared in this file.
 97:   Those functions are defined in subdirectories
 98:   (which this class does not create).
 99:   """
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L101-L139 — Function `__init__`

```python
101:   def __init__(self, generated_path, kind, args):
102:     self.generated_path = generated_path
103:     self.kind = kind
104:     self.args = args
105: 
106:     self.header_template ="""
107: /*
108:  Generated by manifest.py - Do not edit.
109: */
110: 
111: #include "cutlass/cutlass.h"
112: #include "cutlass/library/library.h"
113: #include "cutlass/library/manifest.h"
114: 
115: namespace cutlass {
116: namespace library {
117: 
118: ///////////////////////////////////////////////////////////////////////////////////////////////////
119: 
120: """
121: 
122:     self.entry_template = """
123: 
124: //
125: // Entry point to construct operations
126: //
127: void initialize_all_${operation_name}_operations(Manifest &manifest) {
128: """
129:     self.configuration_prototype_template = "void initialize_${configuration_name}(Manifest &manifest);\n"
130:     self.configuration_template ="  initialize_${configuration_name}(manifest);\n"
131: 
132:     self.epilogue_template ="""}
133: 
134: ///////////////////////////////////////////////////////////////////////////////////////////////////
135: 
136: } // namespace library
137: } // namespace cutlass
138: 
139: """
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L140-L141 — Comments

```python
140: 
141:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L142-L180 — Function `__enter__`

```python
142:   def __enter__(self):
143:     _LOGGER.debug("*** EmitOperationKindAll::__enter__")
144: 
145:     self.operation_path = os.path.join(self.generated_path, OperationKindNames[self.kind])
146:     _LOGGER.debug('***   operation_path (directory to create): ' +
147:                   str(self.operation_path));
148:     os.makedirs(self.operation_path, exist_ok=True)
149: 
150:     # Determine file extension based on architecture
151:     # Check if any Intel Xe target is present in the architectures
152:     file_extension = "cu"  # Default to CUDA
153:     if self.args and hasattr(self.args, 'architectures'):
154:       archs = self.args.architectures.split(';') if len(self.args.architectures) else []
155:       for arch in archs:
156:         arch_lower = arch.lower()
157:         # Check for Intel Xe targets
158:         if any(xe_target in arch_lower for xe_target in ['pvc', 'bmg', 'intel_gpu']):
159:           file_extension = "cpp"
160:           break
161:         # Check for numeric Xe architecture in the Intel Xe range
162:         try:
163:           arch_num = int(arch.split('a')[0].split('f')[0])
164:           if arch_num >= INTEL_XE_ARCH_MIN and arch_num < INTEL_XE_ARCH_MAX:
165:             file_extension = "cpp"
166:             break
167:         except (ValueError, AttributeError):
168:           pass
169: 
170:     self.top_level_path = os.path.join(self.operation_path, f"all_{OperationKindNames[self.kind]}_operations.{file_extension}")
171:     _LOGGER.debug(f"***   top_level_path (file to write): {str(self.top_level_path)}")
172: 
173:     self.top_level_file = open(self.top_level_path, "w")
174:     self.top_level_file.write(self.header_template)
175: 
176:     self.source_files = [self.top_level_path,]
177: 
178:     self.configurations = []
179: 
180:     return self
```
**EN:** Defines `__enter__()`, which implements the enter logic. Key helper calls include debug, join, makedirs, open, write, hasattr.

**CN:** 定义 `__enter__()`，用于实现 enter 相关逻辑。 其中会调用的重要辅助函数包括 debug, join, makedirs, open, write, hasattr。

### L181-L182 — Comments

```python
181: 
182:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L183-L194 — Function `emit`

```python
183:   def emit(self, operations):
184:     _LOGGER.debug('*** EmitOperationKindAll::emit')
185:     _LOGGER.debug(f"***   len(operations): {len(operations)}")
186:     _LOGGER.debug(f"***   min_cc list: {sorted(min_cc for min_cc, _ in operations.items())}")
187: 
188:     for min_cc, configurations in sorted(operations.items()):
189:       _LOGGER.debug(f"***   min_cc={min_cc}")
190: 
191:       for configuration_name, _ in configurations.items():
192:         _LOGGER.debug(f"***     configuration_name={configuration_name}")
193:         self.configurations.append(configuration_name)
194:         self.top_level_file.write(SubstituteTemplate(self.configuration_prototype_template, {'configuration_name': configuration_name} ))
```
**EN:** Defines `emit()`, which emits . Key helper calls include debug, sorted, items, append, write, len.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 debug, sorted, items, append, write, len。

### L195-L196 — Comments

```python
195: 
196:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L197-L206 — Function `__exit__`

```python
197:   def __exit__(self, exception_type, exception_value, traceback):
198:     _LOGGER.debug("*** EmitOperationKindAll::__exit__")
199: 
200:     self.top_level_file.write(SubstituteTemplate(self.entry_template, {'operation_name': OperationKindNames[self.kind]}))
201: 
202:     for configuration_name in self.configurations:
203:       self.top_level_file.write(SubstituteTemplate(self.configuration_template, {'configuration_name': configuration_name}))
204: 
205:     self.top_level_file.write(self.epilogue_template)
206:     self.top_level_file.close()
```
**EN:** Defines `__exit__()`, which implements the exit logic. Key helper calls include debug, write, close, SubstituteTemplate.

**CN:** 定义 `__exit__()`，用于实现 exit 相关逻辑。 其中会调用的重要辅助函数包括 debug, write, close, SubstituteTemplate。

### L209-L209 — Class `EmitOperationKindLibrary`

```python
209: class EmitOperationKindLibrary:
```
**EN:** Introduces class `EmitOperationKindLibrary`. The docstring says: "Emit the CUTLASS library initialization code for each OperationKind."; the class body implements that role.

**CN:** 引入类 `EmitOperationKindLibrary`。文档字符串说明其用途为：“Emit the CUTLASS library initialization code for each OperationKind.”；类体负责实现这一职责。

### L210-L240 — Documentation/setup

```python
210:   """
211:   Emit the CUTLASS library initialization code for each OperationKind.
212:   The code is generated in the directory
213:   {generated_path}/{operation_kind}/{min_cc}
214:   (e.g., tools/library/generated/gemm/90 in the build directory,
215:   for min_cc=90 and OperationKind=Gemm), in the file
216:   all_sm{min_cc}_{operation_kind}_operations.cu
217:   (e.g., all_sm90_gemm_operations.cu for min_cc=90 and OperationKind=Gemm).
218:   For Intel Xe targets, uses xe{min_cc} prefix instead of sm{min_cc}.
219:   The min_cc variable here indicates the minimum GPU architecture version
220:   that the things to be initialized require.
221:   For example, min_cc=90 indicates sm90 for CUDA, min_cc=20 indicates Xe2/BMG for Intel.
222: 
223:   That file declares several functions in namespace cutlass::library.
224:   The functions all have this form,
225: 
226:   void initialize_all_sm{min_cc}_{subclass_name}_{extended_name}_operations(Manifest& manifest);
227: 
228:   where extended_name is operation.extended_name() for all the operations
229:   given to the emit method (which see below).  (All operations for a given
230:   configuration_name are guaranteed to have the same extended_name().)
231: 
232:   The file also _defines_ the following function in that namespace.
233: 
234:   void initialize_all_sm{min_cc}__{operation_kind}_operations(Manifest& manifest);
235: 
236:   That function calls all of the functions declared in this file.
237:   Those functions are defined in subdirectories.
238:   The mapping from OperationKind to emitter handles the details
239:   of what happens in each of those subdirectories.
240:   """
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L241-L242 — Comments

```python
241: 
242:   @staticmethod
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L243-L251 — Function `get_arch_prefix`

```python
243:   def get_arch_prefix(min_cc):
244:     """Get architecture prefix based on compute capability.
245:     Returns 'sm' for CUDA architectures, 'xe' for Intel Xe architectures.
246:     Intel Xe: 12 (PVC), 20 (BMG)
247:     CUDA: 50+ for CUDA architectures"""
248:     if min_cc >= INTEL_XE_ARCH_MIN and min_cc < INTEL_XE_ARCH_MAX:
249:       return 'xe'
250:     else:
251:       return 'sm'
```
**EN:** Defines `get_arch_prefix()`. The docstring says: "Get architecture prefix based on compute capability."; this block implements that behavior.

**CN:** 定义 `get_arch_prefix()`。文档字符串说明其用途为：“Get architecture prefix based on compute capability.”；这一代码块给出了该行为的具体实现。

### L253-L302 — Function `__init__`

```python
253:   def __init__(self, generated_path, min_cc, kind, args):
254:     self.generated_path = generated_path
255:     self.min_cc = min_cc
256:     self.kind = kind
257:     self.args = args
258:     self.arch_prefix = self.get_arch_prefix(min_cc)
259:     self.emitters = {
260:       OperationKind.Gemm: EmitGemmConfigurationLibrary,
261:       OperationKind.Conv2d: EmitConv2dConfigurationLibrary,
262:       OperationKind.Conv3d: EmitConv3dConfigurationLibrary,
263:       OperationKind.RankK: EmitRankKConfigurationLibrary,
264:       OperationKind.Rank2K: EmitRank2KConfigurationLibrary,
265:       OperationKind.Trmm: EmitTrmmConfigurationLibrary,
266:       OperationKind.Symm: EmitSymmConfigurationLibrary
267:     }
268: 
269:     self.header_template ="""
270: /*
271:  Generated by manifest.py - Do not edit.
272: */
273: 
274: #include "cutlass/cutlass.h"
275: #include "cutlass/library/library.h"
276: #include "cutlass/library/manifest.h"
277: 
278: namespace cutlass {
279: namespace library {
280: 
281: ///////////////////////////////////////////////////////////////////////////////////////////////////
282: 
283: """
284:     self.entry_template = """
285: 
286: //
287: // Entry point to construct operations
288: //
289: void initialize_all_${arch_prefix}${min_cc}_${subclass_name}_${operation_name}_operations(Manifest &manifest) {
290: """
291:     self.configuration_prototype_template = "void initialize_${configuration_name}(Manifest &manifest);\n"
292:     self.configuration_template = "  initialize_${configuration_name}(manifest);\n"
293:     self.subclass_call_template = "  initialize_all_${arch_prefix}${min_cc}_${subclass_name}_${operation_name}_operations(manifest);\n"
294:     self.subclass_prototype_template = "void initialize_all_${arch_prefix}${min_cc}_${subclass_name}_${operation_name}_operations(Manifest &manifest);\n"
295:     self.epilogue_template ="""}
296: 
297: ///////////////////////////////////////////////////////////////////////////////////////////////////
298: 
299: } // namespace library
300: } // namespace cutlass
301: 
302: """
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include get_arch_prefix.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 get_arch_prefix。

### L303-L304 — Comments

```python
303: 
304:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L305-L333 — Function `__enter__`

```python
305:   def __enter__(self):
306:     _LOGGER.debug("*** EmitOperationKindLibrary::__enter__")
307:     _LOGGER.debug(f"***   generated_path: {str(self.generated_path)}")
308:     _LOGGER.debug(f"***   OperationKindNames[kind]: {OperationKindNames[self.kind]}")
309:     _LOGGER.debug(f"***   min_cc: {self.min_cc}")
310: 
311:     self.operation_path = os.path.join(self.generated_path, OperationKindNames[self.kind], str(self.min_cc))
312:     _LOGGER.debug(f"***   operation_path (directory to make): {str(self.operation_path)}")
313:     os.makedirs(self.operation_path)
314: 
315:     # Use .cpp extension for Intel Xe architectures, .cu for CUDA
316:     file_extension = "cpp" if (self.min_cc >= INTEL_XE_ARCH_MIN and self.min_cc < INTEL_XE_ARCH_MAX) else "cu"
317:     self.top_level_path = os.path.join(self.operation_path, f"all_{self.arch_prefix}{self.min_cc}_{OperationKindNames[self.kind]}_operations.{file_extension}")
318:     _LOGGER.debug(f"***   top_level_path (file to write): {str(self.top_level_path)}")
319: 
320:     self.top_level_file = open(self.top_level_path, "w")
321:     self.top_level_file.write(self.header_template)
322: 
323:     self.source_files = {}
324: 
325:     # Each {operation_kind x cc} combination is further decomposed by the instruction
326:     # types used. This dictionary used to track the file handles for the top-level
327:     # files of each subclass
328:     self.subclass_files = {}
329: 
330:     # Configurations in each sub class
331:     self.subclass_configurations = {}
332: 
333:     return self
```
**EN:** Defines `__enter__()`, which implements the enter logic. Key helper calls include debug, join, makedirs, open, write, str.

**CN:** 定义 `__enter__()`，用于实现 enter 相关逻辑。 其中会调用的重要辅助函数包括 debug, join, makedirs, open, write, str。

### L334-L335 — Comments

```python
334: 
335:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L336-L381 — Function `emit`

```python
336:   def emit(self, configuration_name, operations):
337:     _LOGGER.debug("*** EmitOperationKindLibrary::emit")
338:     _LOGGER.debug(f"***   configuration_name: {configuration_name}")
339: 
340:     assert len(operations) > 0
341: 
342:     # The extended name for all operations of a given configuration_name is guaranteed
343:     # to be the same because extended_name() is used in defining configuration_name. Thus,
344:     # we can safely use the extended_name() of the first operation.
345:     extended_name = operations[0].extended_name()
346:     _LOGGER.debug('***   extended_name (for all ops): ' + extended_name)
347: 
348:     # Create a directory for operations with this subclass if it does not exist
349:     if extended_name not in self.subclass_files:
350:       subclass_path = os.path.join(self.operation_path, extended_name)
351:       _LOGGER.debug(f"***     subclass_path: {str(subclass_path)}")
352:       os.mkdir(subclass_path)
353: 
354:       self.subclass_configurations[extended_name] = []
355: 
356:       # Use .cpp extension for Intel Xe architectures, .cu for CUDA
357:       file_extension = "cpp" if (self.min_cc >= INTEL_XE_ARCH_MIN and self.min_cc < INTEL_XE_ARCH_MAX) else "cu"
358:       # Open a new top-level file for this sub class
359:       subclass_top_level_path = os.path.join(
360:         subclass_path, f"all_{self.arch_prefix}{self.min_cc}_{extended_name}_{OperationKindNames[self.kind]}_operations.{file_extension}")
361:       _LOGGER.debug('***     subclass_top_level_path (min_cc, extended_name, ' +
362:                     'OperationKind): ' + str(subclass_top_level_path))
363: 
364:       self.subclass_files[extended_name] = open(subclass_top_level_path, "w")
365:       self.subclass_files[extended_name].write(self.header_template)
366: 
367:       self.source_files[extended_name] = [subclass_top_level_path]
368: 
369:     subclass_dir = os.path.dirname(self.subclass_files[extended_name].name)
370:     _LOGGER.debug('***   subclass_dir: ' + str(subclass_dir))
371: 
372:     with self.emitters[self.kind](subclass_dir, configuration_name) as configuration_emitter:
373:       for operation in operations:
374:         configuration_emitter.emit(operation)
375: 
376:       _LOGGER.debug('***   configuration_emitter.configuration_path: ' +
377:                     str(configuration_emitter.configuration_path))
378:       self.source_files[extended_name].append(configuration_emitter.configuration_path)
379: 
380:     self.subclass_configurations[extended_name].append(configuration_name)
381:     self.subclass_files[extended_name].write(SubstituteTemplate(self.configuration_prototype_template, {'configuration_name': configuration_name} ))
```
**EN:** Defines `emit()`, which emits . Key helper calls include debug, extended_name, dirname, append, write, len.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 debug, extended_name, dirname, append, write, len。

### L382-L383 — Comments

```python
382: 
383:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L384-L426 — Function `__exit__`

```python
384:   def __exit__(self, exception_type, exception_value, traceback):
385:     _LOGGER.debug("*** EmitOperationKindLibrary::__exit__")    
386:     for subclass_name, subclass_file in sorted(self.subclass_files.items()):
387:       subclass_cfg = {
388:         'arch_prefix': self.arch_prefix,
389:         'min_cc': str(self.min_cc),
390:         'subclass_name': subclass_name,
391:         'operation_name': OperationKindNames[self.kind]
392:       }
393:       self.top_level_file.write(SubstituteTemplate(self.subclass_prototype_template, subclass_cfg))
394: 
395:     self.top_level_file.write(
396:       SubstituteTemplate(self.entry_template, {
397:         'arch_prefix': self.arch_prefix,
398:         'min_cc': str(self.min_cc),
399:         'subclass_name': '',
400:         'operation_name': OperationKindNames[self.kind]
401:       }))
402: 
403:     # Finish and close all subclass files
404:     for subclass_name, subclass_file in sorted(self.subclass_files.items()):
405:       subclass_cfg = {
406:         'arch_prefix': self.arch_prefix,
407:         'min_cc': str(self.min_cc),
408:         'subclass_name': subclass_name,
409:         'operation_name': OperationKindNames[self.kind]
410:       }
411:       subclass_file.write(SubstituteTemplate(self.entry_template, subclass_cfg))
412: 
413:       for configuration in self.subclass_configurations[subclass_name]:
414:         subclass_file.write(
415:           SubstituteTemplate(self.configuration_template, {
416:             'configuration_name': configuration
417:           }))
418: 
419:       subclass_file.write(self.epilogue_template)
420:       subclass_file.close()
421: 
422:       # Write the call to initialize_all for this subclass to the top-level file
423:       self.top_level_file.write(SubstituteTemplate(self.subclass_call_template, subclass_cfg))
424: 
425:     self.top_level_file.write(self.epilogue_template)
426:     self.top_level_file.close()
```
**EN:** Defines `__exit__()`, which implements the exit logic. Key helper calls include debug, sorted, write, close, items, SubstituteTemplate.

**CN:** 定义 `__exit__()`，用于实现 exit 相关逻辑。 其中会调用的重要辅助函数包括 debug, sorted, write, close, items, SubstituteTemplate。

### L428-L428 — Class `EmitInterfaceLibrary`

```python
428: class EmitInterfaceLibrary:
```
**EN:** Introduces class `EmitInterfaceLibrary`. The docstring says: "Emit the topmost-level CUTLASS library initialization code."; the class body implements that role.

**CN:** 引入类 `EmitInterfaceLibrary`。文档字符串说明其用途为：“Emit the topmost-level CUTLASS library initialization code.”；类体负责实现这一职责。

### L429-L450 — Documentation/setup

```python
429:   """
430:   Emit the topmost-level CUTLASS library initialization code.
431:   The code is generated in the generated_path directory
432:   (e.g., tools/library/generated in the build directory),
433:   in the initialize_all.cpp file.
434:   That file declares several functions in namespace cutlass::library.
435:   The functions all have this form,
436: 
437:   void initialize_all_{operation_kind}_operations(Manifest& manifest);
438: 
439:   where {operation_kind} abbreviates the "kind" of operation
440:   (e.g., gemm for matrix-matrix multiply, conv2d for 2-d convolution,
441:   or trmm for triangular solve with multiple right-hand sides).
442:   The definitions of these functions live in subdirectories.
443: 
444:   The file also _defines_ the following function in that namespace.
445: 
446:   void initialize_all(Manifest& manifest);
447: 
448:   That function first prepares the manifest, and then
449:   calls all of the functions declared in this file.
450:   """
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L452-L493 — Function `__init__`

```python
452:   def __init__(self, generated_path, operation_count, args):
453:     self.generated_path = generated_path
454:     self.args = args
455: 
456:     self.prototypes = []
457:     self.fn_calls = []
458:     self.operation_count = str(operation_count)
459: 
460:     self.top_level_hdr_template = '''
461: /*
462:  Generated by manifest.py - Do not edit.
463: */
464: '''
465:     self.top_level_prologue = '''
466: 
467: #include "cutlass/library/library.h"
468: #include "cutlass/library/manifest.h"
469: 
470: namespace cutlass {
471: \tnamespace library {
472: 
473: ${prototypes}
474: '''
475: 
476:     self.top_level_initialize_kind = '''
477: \t\tvoid initialize_all_${kind}_operations(Manifest &manifest) {
478: ${fn_calls}
479: \t\t}
480: '''
481: 
482:     self.top_level_initialize = '''
483: \t\tvoid initialize_all(Manifest &manifest) {
484: \t\t\tmanifest.reserve(${operation_count});\n
485: ${fn_calls}
486: \t\t}
487: '''
488: 
489:     self.top_level_suffix = '''
490: \t} // namespace library
491: } // namespace cutlass
492: 
493: '''
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include str.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 str。

### L494-L495 — Comments

```python
494: 
495:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L496-L507 — Function `__enter__`

```python
496:   def __enter__(self):
497:     _LOGGER.debug("*** EmitInterfaceLibrary::__enter__")
498: 
499:     self.top_level_path = os.path.join(self.generated_path, 'initialize_all.cpp')
500:     _LOGGER.debug("***   top_level_path: " + str(self.top_level_path))
501: 
502:     self.top_level_file = open(self.top_level_path, "w")
503:     self.top_level_file.write(self.top_level_hdr_template)
504: 
505:     self.source_files = [self.top_level_path,]
506: 
507:     return self
```
**EN:** Defines `__enter__()`, which implements the enter logic. Key helper calls include debug, join, open, write, str.

**CN:** 定义 `__enter__()`，用于实现 enter 相关逻辑。 其中会调用的重要辅助函数包括 debug, join, open, write, str。

### L508-L509 — Comments

```python
508: 
509:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L510-L520 — Function `emit`

```python
510:   def emit(self, operation_name):
511:     _LOGGER.debug("*** EmitInterfaceLibrary::emit")
512:     _LOGGER.debug("***   operation_name: " + operation_name)
513: 
514:     self.prototypes.append(SubstituteTemplate(
515:        "\t\tvoid initialize_all_${operation_kind}_operations(Manifest &manifest);",
516:        {'operation_kind': operation_name}))
517: 
518:     self.fn_calls.append(SubstituteTemplate(
519:       "\t\t\tinitialize_all_${operation_kind}_operations(manifest);",
520:       {'operation_kind': operation_name}))
```
**EN:** Defines `emit()`, which emits . Key helper calls include debug, append, SubstituteTemplate.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 debug, append, SubstituteTemplate。

### L521-L522 — Comments

```python
521: 
522:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L523-L533 — Function `__exit__`

```python
523:   def __exit__(self, exception_type, exception_value, traceback):
524:     _LOGGER.debug("*** EmitInterfaceLibrary::__exit__")
525: 
526:     self.top_level_file.write(SubstituteTemplate(self.top_level_prologue, {'prototypes':"\n".join(self.prototypes)}))
527: 
528:     # Write out initialize_all method
529:     self.top_level_file.write(SubstituteTemplate(self.top_level_initialize,
530:                               {'operation_count': self.operation_count, 'fn_calls':"\n".join(self.fn_calls)}))
531: 
532:     self.top_level_file.write(self.top_level_suffix)
533:     self.top_level_file.close()
```
**EN:** Defines `__exit__()`, which implements the exit logic. Key helper calls include debug, write, close, SubstituteTemplate, join.

**CN:** 定义 `__exit__()`，用于实现 exit 相关逻辑。 其中会调用的重要辅助函数包括 debug, write, close, SubstituteTemplate, join。

### L534-L537 — Comments

```python
534: 
535: ###################################################################################################
536: ###################################################################################################
537: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L538-L538 — Class `Options`

```python
538: class Options:
```
**EN:** Introduces class `Options`, which stores and normalizes command-line configuration.

**CN:** 引入类 `Options`，用于保存并规范化命令行配置。

### L539-L540 — Function `__init__`

```python
539:   def __init__(self):
540:     pass
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L541-L544 — Comments

```python
541: 
542: ###################################################################################################
543: 
544: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L545-L547 — Class `Manifest`

```python
545: class Manifest:
546: 
547:   #
```
**EN:** Introduces class `Manifest`, which manages generated-operation manifests and related output.

**CN:** 引入类 `Manifest`，用于管理生成操作的 manifest 及相关输出。

### L548-L651 — Function `__init__`

```python
548:   def __init__(self, args = None):
549:     self.operations = {}
550:     self.args = args
551:     self.operation_count = 0
552:     self.operations_by_name = {}
553: 
554:     self.kernel_filter = ''
555:     self.kernel_filter_list = []
556:     self.kernel_names = []
557:     self.operations_enabled = []
558:     self.selected_kernels = []
559:     self.ignore_kernel_names = []
560:     self.exclude_kernel_names = []
561:     self.compute_capabilities_baseline = [50,]
562:     self.compute_capabilities_feature_set = ['50',]
563:     self.curr_build_dir = '.'
564:     self.filter_by_cc = True
565:     self.is_xe_target = False  # Track if building for Intel Xe
566: 
567:     if self.args:
568:       self.kernel_filter = self.args.kernels
569:       self.curr_build_dir = args.curr_build_dir
570: 
571:       # A common user error is to use commas instead of semicolons.
572:       if ',' in args.architectures:
573:         raise RuntimeError("The list of architectures (CMake option CUTLASS_NVCC_ARCHS or DPCPP_SYCL_TARGET) must be semicolon-delimited.\nDon't use commas to separate the architectures; use semicolons.\nYou specified the list as: " + args.architectures)
574:       
575:       self.compute_capabilities_feature_set = args.architectures.split(';') if len(args.architectures) else ['50',]
576:       
577:       # Parse architecture identifiers - support both CUDA SM and Intel Xe targets
578:       baseline_archs = []
579:       for arch in self.compute_capabilities_feature_set:
580:         # Check if this is an Intel Xe target (pvc, bmg, etc.)
581:         # Support both string names ('pvc', 'bmg') and numeric values
582:         arch_lower = arch.lower()
583:         is_xe_named = any(xe_target in arch_lower for xe_target in ['pvc', 'bmg', 'intel_gpu'])
584:         
585:         # Also check if it's a numeric Xe architecture in the Intel Xe range
586:         try:
587:           arch_num = int(arch.split('a')[0].split('f')[0])
588:           is_xe_numeric = (arch_num >= INTEL_XE_ARCH_MIN and arch_num < INTEL_XE_ARCH_MAX)
589:         except (ValueError, AttributeError):
590:           arch_num = None
591:           is_xe_numeric = False
592:         
593:         if is_xe_named or is_xe_numeric:
594:           self.is_xe_target = True
595:           # Map Intel Xe architectures to numeric identifiers for compatibility
596:           # PVC (Ponte Vecchio) -> 12
597:           # BMG (Battlemage/Xe2) -> 20  
598:           if 'pvc' in arch_lower or arch_num == INTEL_XE12:
599:             baseline_archs.append(INTEL_XE12)
600:           elif 'bmg' in arch_lower or 'xe2' in arch_lower or arch_num == INTEL_XE20:
601:             baseline_archs.append(INTEL_XE20)
602:           else:
603:             # Generic Intel GPU target - default to BMG
604:             baseline_archs.append(INTEL_XE20)
605:         else:
606:           # CUDA SM architecture
607:           baseline_archs.append(arch_num if arch_num is not None else int(arch.split('a')[0].split('f')[0]))
608:       
609:       self.compute_capabilities_baseline = sorted(set(baseline_archs))
610: 
611:       if args.filter_by_cc in ['false', 'False', '0']:
612:         self.filter_by_cc = False
613: 
614:       if args.operations == 'all':
615:         self.operations_enabled = []
616:       else:
617:         operations_list = [
618:           OperationKind.Gemm
619:           , OperationKind.Conv2d
620:           , OperationKind.Conv3d
621:             , OperationKind.RankK
622:             , OperationKind.Trmm
623:             , OperationKind.Symm
624:         ]
625:         self.operations_enabled = [x for x in operations_list if OperationKindNames[x] in args.operations.split(',')]
626: 
627:       if args.kernels == 'all':
628:         self.kernel_names = []
629:       else:
630:         self.kernel_names = [x for x in args.kernels.split(',') if x != '']
631: 
632:       self.ignore_kernel_names = [x for x in args.ignore_kernels.split(',') if x != '']
633:       self.exclude_kernel_names = [x for x in args.exclude_kernels.split(',') if x != '']
634: 
635:       if args.kernel_filter_file is None:
636:           self.kernel_filter_list = []
637:       else:
638:           self.kernel_filter_list = self.get_kernel_filters(args.kernel_filter_file)
639:           _LOGGER.debug("Using {filter_count} kernel filters from {filter_file}".format(
640:               filter_count = len(self.kernel_filter_list),
641:               filter_file = args.kernel_filter_file))
642: 
643:       self.operation_count = 0
644:       self.operations_by_name = {}
645:       self.disable_full_archs_compilation = args.disable_full_archs_compilation
646:       self.is_kernel_filter_set_to_all = args.instantiation_level == "max" and args.kernels != ''
647:       self.instantiation_level = 0
648:       try:
649:           self.instantiation_level = int(args.instantiation_level)
650:       except ValueError:
651:           self.instantiation_level = 0
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include sorted, RuntimeError, len, split, lower, any.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 sorted, RuntimeError, len, split, lower, any。

### L653-L656 — Function `add_kernel_filter`

```python
653:   def add_kernel_filter(self, filter_str):
654:     filter_re = re.compile(filter_str)
655: 
656:     self.kernel_filter_list.append(filter_re)
```
**EN:** Defines `add_kernel_filter()`, which implements the add kernel filter logic. Key helper calls include compile, append.

**CN:** 定义 `add_kernel_filter()`，用于实现 add kernel filter 相关逻辑。 其中会调用的重要辅助函数包括 compile, append。

### L658-L676 — Function `get_instantiation_level`

```python
658:   def get_instantiation_level(self, pruned_level=0, default_level=111, exhaustive_level=9992):
659:     # Non-negative integer which determines how many kernels are instantiated.
660:     # 0 = 0000 generates the fewest kernels, 9999 generates all possible combinations.
661:     # increasing first digit reduces schedule / mixed type pruning,
662:     # increasing second digit generates more cluster sizes,
663:     # increasing third digit generates more MMA multipliers,
664:     # increasing fourth digit generates more instruction shapes.
665: 
666:     if self.instantiation_level > 0:
667:         return self.instantiation_level
668: 
669:     elif self.is_kernel_filter_set_to_all:
670:         return exhaustive_level
671: 
672:     elif self.kernel_filter == '':
673:         return pruned_level
674: 
675:     else:
676:         return default_level
```
**EN:** Defines `get_instantiation_level()`, which returns or derives instantiation level.

**CN:** 定义 `get_instantiation_level()`，用于返回或推导 instantiation level。

### L679-L687 — Function `get_kernel_filters`

```python
679:   def get_kernel_filters(self, kernelListFile):
680:     if os.path.isfile(kernelListFile):
681:         with open(kernelListFile, 'r') as fileReader:
682:             lines = [line.rstrip() for line in fileReader if not line.startswith("#")]
683: 
684:         lines = [re.compile(line) for line in lines if line]
685:         return lines
686:     else:
687:         return []
```
**EN:** Defines `get_kernel_filters()`, which returns or derives kernel filters. Key helper calls include isfile, open, compile, rstrip, startswith.

**CN:** 定义 `get_kernel_filters()`，用于返回或推导 kernel filters。 其中会调用的重要辅助函数包括 isfile, open, compile, rstrip, startswith。

### L688-L689 — Comments

```python
688: 
689:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L690-L696 — Function `filter_out_kernels`

```python
690:   def filter_out_kernels(self, kernel_name, kernel_filter_list):
691: 
692:     for kernel_filter_re in kernel_filter_list:
693:         if kernel_filter_re.search(kernel_name) is not None:
694:             return True
695: 
696:     return False
```
**EN:** Defines `filter_out_kernels()`, which filters out kernels. Key helper calls include search.

**CN:** 定义 `filter_out_kernels()`，用于过滤 out kernels。 其中会调用的重要辅助函数包括 search。

### L697-L699 — Comments

```python
697: 
698: 
699:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L700-L708 — Function `_filter_string_matches`

```python
700:   def _filter_string_matches(self, filter_string, haystack):
701:     ''' Returns true if all substrings appear in the haystack in order'''
702:     substrings = filter_string.split('*')
703:     for sub in substrings:
704:       idx = haystack.find(sub)
705:       if idx < 0:
706:         return False
707:       haystack = haystack[idx + len(sub):]
708:     return True
```
**EN:** Defines `_filter_string_matches()`. The docstring says: "Returns true if all substrings appear in the haystack in order"; this block implements that behavior. Key helper calls include split, find, len.

**CN:** 定义 `_filter_string_matches()`。文档字符串说明其用途为：“Returns true if all substrings appear in the haystack in order”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 split, find, len。

### L709-L710 — Comments

```python
709: 
710:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L711-L782 — Function `filter`

```python
711:   def filter(self, operation):
712:     ''' Filtering operations based on various criteria'''
713: 
714:     # filter based on compute capability
715:     enabled = not (self.filter_by_cc)
716: 
717:     for cc in self.compute_capabilities_baseline:
718: 
719:       if cc >= operation.tile_description.minimum_compute_capability and \
720:          cc <= operation.tile_description.maximum_compute_capability and \
721:          (cc not in SharedMemPerCC or SharedMemPerCC[cc] >= CalculateSmemUsage(operation)):
722: 
723:         enabled = True
724:         break
725: 
726:     if not enabled:
727:       return False
728: 
729:     if len(self.operations_enabled) and not operation.operation_kind in self.operations_enabled:
730:       return False
731: 
732:     name = operation.procedural_name()
733: 
734:     # eliminate duplicates
735:     if name in self.operations_by_name.keys():
736:       return False
737: 
738:     # Filter based on list of valid substrings
739:     if len(self.kernel_names):
740:       enabled = False
741: 
742:       # compare against the include list
743:       for name_substr in self.kernel_names:
744:         if self._filter_string_matches(name_substr, name):
745:           _LOGGER.debug(f"Kernel {name} included due to filter string '{name_substr}'.")
746:           enabled = True
747:           break
748:         else:
749:           _LOGGER.debug(f"Kernel {name} NOT included due to not matching '{name_substr}'.")
750: 
751:       # compare against the exclude list
752:       for name_substr in self.ignore_kernel_names:
753:         if self._filter_string_matches(name_substr, name):
754:           _LOGGER.debug(f"Kernel {name} ignored due to filter string '{name_substr}'.")
755:           enabled = False
756:           break
757:         else:
758:           _LOGGER.debug(f"Kernel {name} NOT ignored due to not matching '{name_substr}'.")
759: 
760:     if len(self.kernel_filter_list) > 0:
761:       if self.filter_out_kernels(name, self.kernel_filter_list):
762:         _LOGGER.debug(f"Kernel {name} matched via kernel filter file.")
763:         enabled = True
764:       else:
765:         _LOGGER.debug(f"Kernel {name} culled due to no match in kernel filter file.")
766:         enabled = False
767: 
768:     # CUTLASS_LIBRARY_IGNORE_KERNELS ("ignore" list) only takes effect
769:     # if CUTLASS_LIBRARY_KERNELS was specified.
770:     # Changing that would break backwards compatibility.
771:     # Thus, CUTLASS has introduced the new CMake option CUTLASS_LIBRARY_EXCLUDE_KERNELS,
772:     # that always takes effect, whether or not CUTLASS_LIBRARY_KERNELS was specified.
773:     for name_substr in self.exclude_kernel_names:
774:       if self._filter_string_matches(name_substr, name):
775:         _LOGGER.debug(f"Kernel {name} excluded due to filter string '{name_substr}'.")
776:         enabled = False
777:         break
778:       else:
779:         _LOGGER.debug(f"Kernel {name} NOT excluded due to not matching '{name_substr}'.")
780: 
781:     # TODO: filter based on compute data type
782:     return enabled
```
**EN:** Defines `filter()`. The docstring says: "Filtering operations based on various criteria"; this block implements that behavior. Key helper calls include procedural_name, len, keys, filter_out_kernels, _filter_string_matches, debug.

**CN:** 定义 `filter()`。文档字符串说明其用途为：“Filtering operations based on various criteria”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 procedural_name, len, keys, filter_out_kernels, _filter_string_matches, debug。

### L783-L785 — Comments

```python
783:   #
784: 
785:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L786-L817 — Function `append`

```python
786:   def append(self, operation):
787:     '''
788:       Inserts the operation.
789: 
790:       operation_kind -> configuration_name -> []
791:     '''
792: 
793:     if self.filter(operation):
794: 
795:       self.selected_kernels.append(operation.procedural_name())
796: 
797:       self.operations_by_name[operation.procedural_name()] = operation
798: 
799:       # add the configuration
800:       configuration_name = operation.configuration_name()
801: 
802:       # Split operations by minimum CC
803:       min_cc = operation.arch
804: 
805:       if operation.operation_kind not in self.operations.keys():
806:         self.operations[operation.operation_kind] = {}
807: 
808:       if min_cc not in self.operations[operation.operation_kind]:
809:         self.operations[operation.operation_kind][min_cc] = {}
810: 
811:       if configuration_name not in self.operations[operation.operation_kind][min_cc].keys():
812:         self.operations[operation.operation_kind][min_cc][configuration_name] = []
813: 
814:       self.operations[operation.operation_kind][min_cc][configuration_name].append(operation)
815:       self.operation_count += 1
816:     else:
817:       _LOGGER.debug("Culled {} from manifest".format(operation.procedural_name()))
```
**EN:** Defines `append()`. The docstring says: "Inserts the operation."; this block implements that behavior. Key helper calls include filter, append, configuration_name, debug, procedural_name, keys.

**CN:** 定义 `append()`。文档字符串说明其用途为：“Inserts the operation.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 filter, append, configuration_name, debug, procedural_name, keys。

### L818-L819 — Comments

```python
818:   #
819: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L820-L855 — Function `emit_manifest_cmake`

```python
820:   def emit_manifest_cmake(self, manifest_path, top_level_path, source_files):
821:     with open(manifest_path, "w") as manifest_file:
822: 
823:       target_text = SubstituteTemplate("""cutlass_target_sources(cutlass_library_objs PRIVATE
824:       """, { })
825:       manifest_file.write(target_text + '\n\n')
826:       manifest_file.write("    %s\n" % str(top_level_path.replace('\\', '/')))
827:       generated_path = os.path.join(self.curr_build_dir, 'generated')
828:       
829:       # Determine file extension based on whether we're targeting Intel Xe
830:       file_extension = "cpp" if self.is_xe_target else "cu"
831:       
832:       for kind in self.operations.keys():
833:         kind_str = OperationKindNames[kind]
834:         all_kind_file = os.path.join(generated_path, kind_str, f"all_{kind_str}_operations.{file_extension}").replace('\\', '/')
835:         manifest_file.write(f"    {all_kind_file}\n")
836:       manifest_file.write(')\n\n')
837: 
838:       for kind in self.operations.keys():
839:         for min_cc in sorted(self.operations[kind].keys()):
840:           for subclass in sorted(source_files[kind][min_cc].keys()):
841:             # Use appropriate prefix (sm for CUDA, xe for Intel)
842:             arch_prefix = 'xe' if (min_cc >= INTEL_XE_ARCH_MIN and min_cc < INTEL_XE_ARCH_MAX) else 'sm'
843:             target_text = SubstituteTemplate("""cutlass_add_cutlass_library(
844:       SUFFIX ${kind}_${arch_prefix}${min_cc}_${subclass}
845: """, { 'arch_prefix': arch_prefix, 'min_cc': str(min_cc), 'kind': OperationKindNames[kind], 'subclass': subclass })
846:             manifest_file.write(target_text + '\n\n')
847: 
848:             for source_file in source_files[kind][min_cc][subclass]:
849:               manifest_file.write("    %s\n" % str(source_file.replace('\\', '/')))
850: 
851:             manifest_file.write(")\n")
852: 
853:           # Only apply CUDA-specific arch compilation settings for CUDA targets
854:           if self.disable_full_archs_compilation and min_cc < INTEL_XE_ARCH_MIN:
855:             self.emit_disable_full_archs_compilation(manifest_file, source_files)
```
**EN:** Defines `emit_manifest_cmake()`, which emits manifest cmake. Key helper calls include open, SubstituteTemplate, write, join, keys, replace.

**CN:** 定义 `emit_manifest_cmake()`，用于输出/生成 manifest cmake。 其中会调用的重要辅助函数包括 open, SubstituteTemplate, write, join, keys, replace。

### L857-L902 — Function `emit_disable_full_archs_compilation`

```python
857:   def emit_disable_full_archs_compilation(manifest_file, source_files):
858:       def for_hopper(name):
859:           pass
860: 
861:       def for_ampere(name):
862:           return "16816" in name or \
863:                   "16832" in name or \
864:                   "16864" in name or \
865:                   ("1688" in name and "tf32" in name)
866: 
867:       def for_turing(name):
868:           return ("1688" in name and "tf32" not in name) or \
869:                   "8816" in name
870: 
871:       def for_volta(name):
872:           return "884" in name
873: 
874:       def is_cpp(name):
875:           return name.endswith(".cpp")
876: 
877:       def get_src_archs_str_given_requested_cuda_archs(archs, source_file):
878:           intersected_archs = archs & set(self.compute_capabilities_baseline)
879:           if intersected_archs == set():
880:               raise RuntimeError(
881:                     """
882:                     Empty archs set for file {} after taking
883:                     the intersection of {} (global requested archs) and
884:                     {} (per file requested archs)
885:                     """.format(source_file, set(self.compute_capabilities_baseline), archs))
886:           else:
887:               return " ".join(map(str, intersected_archs))
888: 
889:       for min_cc in sorted(source_files.keys()):
890:         for source_file in source_files[min_cc]:
891:             if is_cpp(source_file):
892:                 continue # skip because source is cpp
893:             elif for_ampere(source_file):
894:                 archs_str = get_src_archs_str_given_requested_cuda_archs({80, 87, 90}, source_file)
895:             elif for_turing(source_file):
896:                 archs_str = get_src_archs_str_given_requested_cuda_archs({75}, source_file)
897:             elif for_volta(source_file):
898:                 archs_str = get_src_archs_str_given_requested_cuda_archs({70, 72}, source_file)
899:             else:
900:                 raise RuntimeError("Per file archs are not set {}, as there is no rule specified for this file pattern".format(source_file))
901: 
902:             manifest_file.write("cutlass_apply_cuda_gencode_flags({} SM_ARCHS {})\n".format(str(source_file.replace('\\', '/')), archs_str))
```
**EN:** Defines `emit_disable_full_archs_compilation()`, which emits disable full archs compilation. Key helper calls include sorted, endswith, keys, set, RuntimeError, join.

**CN:** 定义 `emit_disable_full_archs_compilation()`，用于输出/生成 disable full archs compilation。 其中会调用的重要辅助函数包括 sorted, endswith, keys, set, RuntimeError, join。

### L903-L904 — Comments

```python
903: 
904:   #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L905-L958 — Function `emit`

```python
905:   def emit(self, target = GeneratorTarget.Library):
906: 
907:     operation_emitters = {
908:       GeneratorTarget.Library: EmitOperationKindLibrary
909:     }
910: 
911:     # Emitters for all operations that fall under a particular kind (e.g., GEMM, Conv2d)
912:     kind_emitters = {
913:       GeneratorTarget.Library: EmitOperationKindAll
914:     }
915: 
916:     interface_emitters = {
917:       GeneratorTarget.Library: EmitInterfaceLibrary
918:     }
919: 
920:     generated_path = os.path.join(self.curr_build_dir, 'generated')
921: 
922:     # create generated/
923:     if os.path.exists(generated_path):
924:       shutil.rmtree(generated_path)
925: 
926:     os.mkdir(generated_path)
927: 
928:     with interface_emitters[target](generated_path, self.operation_count, self.args) as iface_emitter:
929:       top_level_path = iface_emitter.top_level_path
930:       for operation_kind in self.operations.keys():
931:         iface_emitter.emit(OperationKindNames[operation_kind])
932: 
933:     source_files = {}
934:     for kind in self.operations.keys():
935:       source_files[kind] = {}
936:       for min_cc in self.operations[kind].keys():
937:         source_files[kind][min_cc] = {}
938: 
939:     for operation_kind, ops in self.operations.items():
940:       for min_cc, configurations in sorted(ops.items()):
941:         with operation_emitters[target](generated_path, min_cc, operation_kind, self.args) as operation_kind_emitter:
942:           for configuration_name, operations in configurations.items():
943:             _LOGGER.info(f"Emitting {configuration_name} with {len(operations)} operation{'' if len(operations) == 1 else 's'}.")
944:             operation_kind_emitter.emit(configuration_name, operations)
945: 
946:           for subclass, files in operation_kind_emitter.source_files.items():
947:             if subclass not in source_files[operation_kind][min_cc]:
948:               source_files[operation_kind][min_cc][subclass] = []
949:             source_files[operation_kind][min_cc][subclass].extend(operation_kind_emitter.source_files[subclass])
950: 
951:       # Emit top level all_{gemm, conv2d, ...}_operations.cu files
952:       with kind_emitters[target](generated_path, operation_kind, self.args) as operation_kind_emitter:
953:         operation_kind_emitter.emit(ops)
954: 
955:     # write the manifest.cmake file containing paths from all targets
956:     manifest_path = os.path.join(generated_path, "manifest.cmake")
957: 
958:     self.emit_manifest_cmake(manifest_path, top_level_path, source_files)
```
**EN:** Defines `emit()`, which emits . Key helper calls include join, exists, mkdir, keys, items, emit_manifest_cmake.

**CN:** 定义 `emit()`，用于输出/生成 。 其中会调用的重要辅助函数包括 join, exists, mkdir, keys, items, emit_manifest_cmake。

### L959-L960 — Comments

```python
959: 
960: ###################################################################################################
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

## Key Concepts / 关键概念

- **EN:** File role: Implements manifest emission, option parsing, and operation filtering/grouping for generated libraries.
- **CN:** 文件角色：实现生成库所需的 manifest 输出、选项解析以及操作过滤/分组逻辑。
- **EN:** Main classes: `EmitOperationKindAll, EmitOperationKindLibrary, EmitInterfaceLibrary, Options, Manifest`
- **CN:** 主要类：`EmitOperationKindAll, EmitOperationKindLibrary, EmitInterfaceLibrary, Options, Manifest`
- **EN:** Important constants/tables: `_LOGGER`
- **CN:** 重要常量/表：`_LOGGER`

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: `enum, logging, os.path, shutil, builtins`
- **CN:** 标准库依赖：`enum, logging, os.path, shutil, builtins`
- **EN:** Internal CUTLASS dependencies: `cutlass_library.library, cutlass_library.gemm_operation, cutlass_library.rank_k_operation, cutlass_library.rank_2k_operation, cutlass_library.trmm_operation, cutlass_library.symm_operation, cutlass_library.conv2d_operation, cutlass_library.conv3d_operation, cutlass_library.arch_constants, library, gemm_operation, rank_k_operation, rank_2k_operation, trmm_operation, symm_operation, conv2d_operation, conv3d_operation, arch_constants`
- **CN:** CUTLASS 内部依赖：`cutlass_library.library, cutlass_library.gemm_operation, cutlass_library.rank_k_operation, cutlass_library.rank_2k_operation, cutlass_library.trmm_operation, cutlass_library.symm_operation, cutlass_library.conv2d_operation, cutlass_library.conv3d_operation, cutlass_library.arch_constants, library, gemm_operation, rank_k_operation, rank_2k_operation, trmm_operation, symm_operation, conv2d_operation, conv3d_operation, arch_constants`
- **EN:** External/non-stdlib dependencies: `None explicitly imported`
- **CN:** 外部/非标准库依赖：`未显式导入`
