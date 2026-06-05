# generator.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/cutlass_library/generator.py`
- **EN:** Acts as the main kernel-library generator: it parses options, creates operation families, and populates manifests.
- **CN:** 作为主内核库生成器：解析选项、创建各类操作族，并填充 manifest。

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

### L34-L49 — Imports and setup

```python
   34: """
   35: Utilities for enumerating CUTLASS library kernels
   36: """
   37: 
   38: import argparse
   39: import enum
   40: from itertools import chain, product
   41: import logging
   42: import os.path
   43: import shutil
   44: import sys
   45: import copy
   46: from typing import Any, Dict, Optional, Sequence, Tuple
   47: import json
   48: 
   49: _LOGGER = logging.getLogger(__name__)
```
**EN:** Imports and/or re-exports modules such as argparse, enum, itertools, chain, product, logging, os.path, shutil, ... so later code can reuse shared definitions.

**CN:** 导入或重新导出诸如 argparse, enum, itertools, chain, product, logging, os.path, shutil, ... 等模块，使后续代码可以复用共享定义。

### L51-L55 — Function `logging_prefix`

```python
   51: def logging_prefix(indent_level: int = 0) -> str:
   52:   """String prefix for start of each debug log entry"""
   53:   prefix = '*** '
   54:   indent = '  '
   55:   return f"{prefix}{indent_level * indent}"
```
**EN:** Defines `logging_prefix()`. The docstring says: "String prefix for start of each debug log entry"; this block implements that behavior.

**CN:** 定义 `logging_prefix()`。文档字符串说明其用途为：“String prefix for start of each debug log entry”；这一代码块给出了该行为的具体实现。

### L57-L60 — Function `log_debug_line`

```python
   57: def log_debug_line(line: str, indent_level: int = 0) -> None:
   58:   """Log one line of debug output"""
   59:   prefix = logging_prefix(indent_level)
   60:   _LOGGER.debug(prefix + line)
```
**EN:** Defines `log_debug_line()`. The docstring says: "Log one line of debug output"; this block implements that behavior. Key helper calls include logging_prefix, debug.

**CN:** 定义 `log_debug_line()`。文档字符串说明其用途为：“Log one line of debug output”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 logging_prefix, debug。

### L61-L71 — Comments

```python
   61: 
   62: # Certain usecases of cutlass_library nearly always prefer to run as scripts with
   63: # relative imports, rather than via an installed Python package. An example of this
   64: # is using CUTLASS's CMake system to generate a library of kernels to be profiled.
   65: # To make it easy to use these use cases when an existing installation of cutlass_library
   66: # exists, this global flag can be set to true (via command-line arguments) to ensure
   67: # that package-based installations are not used.
   68: 
   69: # Create a temporary argument parser to check only for the availability of the
   70: # --disable-cutlass-package-imports argument, which controls whether package-based
   71: # imports are disabled.
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L72-L74 — Function `_add_package_disablement_flag`

```python
   72: def _add_package_disablement_flag(argparser):
   73:   argparser.add_argument("--disable-cutlass-package-imports", action='store_true', required=False,
   74:                      help="Disable use of cutlass_library from Python package")
```
**EN:** Defines `_add_package_disablement_flag()`, which implements the add package disablement flag logic. Key helper calls include add_argument.

**CN:** 定义 `_add_package_disablement_flag()`，用于实现 add package disablement flag 相关逻辑。 其中会调用的重要辅助函数包括 add_argument。

### L76-L102 — Imports and setup

```python
   76: _parser = argparse.ArgumentParser()
   77: _add_package_disablement_flag(_parser)
   78: _args, _ = _parser.parse_known_args()
   79: 
   80: # Add `CUTLASS_IGNORE_PACKAGE` to `builtins` so that it is visible for gating future
   81: # imports without requiring importing another module. Ideally, we would just place this
   82: # as a global variable in a module to that could be imported and checked (e.g.,
   83: # utils.CUTLASS_IGNORE_PACKAGE). However, this raises the issue of determining
   84: # where this module should be sourced (from the cutlass_library package or from
   85: # a relative import), which is the problem this variable is being used to solve in the
   86: # first place.
   87: import builtins
   88: builtins.CUTLASS_IGNORE_PACKAGE = _args.disable_cutlass_package_imports
   89: 
   90: try:
   91:   if CUTLASS_IGNORE_PACKAGE:
   92:     raise ImportError("Disabling attempt to import cutlass_library")
   93:   from cutlass_library.library import *
   94:   from cutlass_library.manifest import *
   95:   from cutlass_library.heuristics import *
   96:   from cutlass_library.emit_kernel_listing import emit_gemm_kernel_testlist 
   97:   from cutlass_library.arch_constants import INTEL_XE12, INTEL_XE20, INTEL_XE35
   98: except ImportError:
   99:   from library import *
  100:   from manifest import *
  101:   from heuristics import *
  102:   from emit_kernel_listing import emit_gemm_kernel_testlist 
```
**EN:** Imports and/or re-exports modules such as builtins so later code can reuse shared definitions.

**CN:** 导入或重新导出诸如 builtins 等模块，使后续代码可以复用共享定义。

### L103-L105 — Comments

```python
  103: ###################################################################################################
  104: 
  105: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L106-L118 — Function `CudaToolkitVersionSatisfies`

```python
  106: def CudaToolkitVersionSatisfies(semantic_ver_string, major, minor, patch = 0):
  107: 
  108:   # by default, use the latest CUDA Toolkit version
  109:   cuda_version = [11, 0, 132]
  110: 
  111:   # Update cuda_version based on parsed string
  112:   if semantic_ver_string != '':
  113:     for i, x in enumerate([int(x) for x in semantic_ver_string.split('.')[:3]]):
  114:       if i < len(cuda_version):
  115:         cuda_version[i] = x
  116:       else:
  117:         cuda_version.append(x)
  118:   return cuda_version >= [major, minor, patch]
```
**EN:** Defines `CudaToolkitVersionSatisfies()`, which implements the cuda toolkit version satisfies logic. Key helper calls include enumerate, int, len, append, split.

**CN:** 定义 `CudaToolkitVersionSatisfies()`，用于实现 cuda toolkit version satisfies 相关逻辑。 其中会调用的重要辅助函数包括 enumerate, int, len, append, split。

### L119-L120 — Comments

```python
  119: 
  120: # From cuda 13.0, Thor SM is renumbered from 101 to 110
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L121-L122 — Function `ThorSMRenumbering`

```python
  121: def ThorSMRenumbering(cuda_version):
  122:   return 110 if CudaToolkitVersionSatisfies(cuda_version, 13, 0) else 101
```
**EN:** Defines `ThorSMRenumbering()`, which implements the thor smrenumbering logic. Key helper calls include CudaToolkitVersionSatisfies.

**CN:** 定义 `ThorSMRenumbering()`，用于实现 thor smrenumbering 相关逻辑。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies。

### L123-L127 — Comments

```python
  123: 
  124: ###################################################################################################
  125: ###################################################################################################
  126: 
  127: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L128-L138 — Function `EpilogueAlignment`

```python
  128: def EpilogueAlignment(max_alignment, tile, epilogue_steps = 8):
  129:   ''' Helper to compute the maximum alignment of the epilogue '''
  130: 
  131:   def product(X, identity = 1):
  132:     result = identity
  133:     for item in X:
  134:       result *= item
  135:     return result
  136: 
  137:   elements_per_thread = product(tile.threadblock_shape[:-1]) // product(tile.warp_count) // 32 // epilogue_steps
  138:   return min(max_alignment, elements_per_thread)
```
**EN:** Defines `EpilogueAlignment()`. The docstring says: "Helper to compute the maximum alignment of the epilogue"; this block implements that behavior. Key helper calls include min, product.

**CN:** 定义 `EpilogueAlignment()`。文档字符串说明其用途为：“Helper to compute the maximum alignment of the epilogue”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 min, product。

### L140-L141 — Function `DefaultSwizzlingFunctor`

```python
  140: def DefaultSwizzlingFunctor():
  141:     return SwizzlingFunctor.Identity8
```
**EN:** Defines `DefaultSwizzlingFunctor()`, which implements the default swizzling functor logic.

**CN:** 定义 `DefaultSwizzlingFunctor()`，用于实现 default swizzling functor 相关逻辑。

### L142-L144 — Comments

```python
  142:     # To use StreamK decomposition for basic GEMMs, set `swizzling_functor = SwizzlingFunctor.StreamK`
  143: 
  144: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L145-L181 — Function `CreateGemmOperator`

```python
  145: def CreateGemmOperator(manifest, layouts, tile_descriptions, data_type, \
  146:   alignment_constraints, complex_transforms = None, epilogue_functor = EpilogueFunctor.LinearCombination, \
  147:   swizzling_functor = DefaultSwizzlingFunctor()):
  148: 
  149:   if complex_transforms is None:
  150:     complex_transforms = [(ComplexTransform.none, ComplexTransform.none),]
  151: 
  152:   element_a, element_b, element_c, element_epilogue = data_type
  153: 
  154:   operations = []
  155: 
  156:   # by default, only generate the largest tile and largest alignment
  157:   if manifest.kernel_filter == '':
  158:     tile_descriptions = [tile_descriptions[0],]
  159:     alignment_constraints = [alignment_constraints[0],]
  160: 
  161:   for layout in layouts:
  162:     for tile_description in tile_descriptions:
  163:       for alignment in alignment_constraints:
  164:         for complex_transform in complex_transforms:
  165: 
  166:             # If alignment is a tuple or a list, then we have different alignments for A and B
  167:             alignment_a = alignment if isinstance(alignment, int) else alignment[0]
  168:             alignment_b = alignment if isinstance(alignment, int) else alignment[1]
  169:             alignment_c = min(8, alignment_a) if isinstance(alignment, int) else alignment[2]
  170: 
  171:             A = TensorDescription(element_a, layout[0], alignment_a, complex_transform[0])
  172:             B = TensorDescription(element_b, layout[1], alignment_b, complex_transform[1])
  173:             C = TensorDescription(element_c, layout[2], alignment_c)
  174: 
  175:             new_operation = GemmOperation(GemmKind.Universal, tile_description.minimum_compute_capability, \
  176:               tile_description, A, B, C, element_epilogue, epilogue_functor, swizzling_functor)
  177: 
  178:             manifest.append(new_operation)
  179:             operations.append(new_operation)
  180: 
  181:   return operations
```
**EN:** Defines `CreateGemmOperator()`, which creates gemm operator and appends it to the surrounding workflow. Key helper calls include DefaultSwizzlingFunctor, TensorDescription, GemmOperation, append, isinstance, min.

**CN:** 定义 `CreateGemmOperator()`，用于创建 gemm operator 并将其接入周边流程。 其中会调用的重要辅助函数包括 DefaultSwizzlingFunctor, TensorDescription, GemmOperation, append, isinstance, min。

### L182-L183 — Comments

```python
  182: 
  183: # Generates 3.0 API based GemmUniversal API kernels. Alignment constraints are folded in with layouts
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L184-L271 — Function `CreateGemmUniversal3xOperator`

```python
  184: def CreateGemmUniversal3xOperator(
  185:     manifest, layouts, tile_descriptions, data_types,
  186:     schedules = [[KernelScheduleType.ScheduleAuto, EpilogueScheduleType.ScheduleAuto]],
  187:     complex_transforms=None,
  188:     epilogue_functor=EpilogueFunctor.LinearCombination,
  189:     swizzling_functor=SwizzlingFunctor.Identity1,
  190:     tile_schedulers=[TileSchedulerType.Default],
  191:     gemm_kind=GemmKind.Universal3x):
  192: 
  193:   if type(data_types) is dict:
  194:     data_types = [data_types]
  195: 
  196:   for s in schedules:
  197:     assert(len(s) == 2)
  198: 
  199:   if complex_transforms is None:
  200:     complex_transforms = [(ComplexTransform.none, ComplexTransform.none), ]
  201: 
  202:   operations = []
  203: 
  204:   # generate all tiles when --kernels=all is specified
  205:   if manifest.kernel_filter == '' or manifest.kernel_filter == 'all':
  206:     if len(tile_descriptions) == 0:
  207:       return operations
  208:   
  209:   combinations = product(layouts, tile_descriptions, data_types, complex_transforms, schedules, tile_schedulers)
  210:   for layout, tile_description, data_type, complex_transform, schedules, tile_scheduler in combinations:
  211:     kernel_schedule, epilogue_schedule = schedules
  212:     A = TensorDescription(
  213:         data_type["a_type"], layout[0][0], layout[0][1], complex_transform[0])
  214:     B = TensorDescription(
  215:         data_type["b_type"], layout[1][0], layout[1][1], complex_transform[1])
  216: 
  217:     C = TensorDescription(data_type["c_type"], layout[2][0], layout[2][1])
  218:     D = TensorDescription(data_type["d_type"], layout[2][0], layout[2][1])
  219: 
  220:     gemm_op_extra_args = {}
  221:     element_compute = data_type.get("epi_type", data_type["acc_type"])
  222: 
  223:     if "sf_type" in data_type:
  224:       gemm_op_extra_args["ScaleFactorA"] = data_type["sf_type"]
  225:       gemm_op_extra_args["ScaleFactorB"] = data_type["sf_type"]
  226:       gemm_op_extra_args["ScaleFactorD"] = { "tensor": TensorDescription(data_type["sfd_type"]["type"], data_type["sfd_type"]["layout"]),
  227:                                              "vector_size" : data_type["sfd_type"]["vector_size"]}
  228:       assert is_block_scaled(gemm_kind)
  229:     
  230:     if tile_description.explicit_vector_sizes != None:
  231:       assert len(tile_description.explicit_vector_sizes) == 3
  232:       gemm_op_extra_args["ScaleFactorMVecSize"] = tile_description.explicit_vector_sizes[0]
  233:       gemm_op_extra_args["ScaleFactorNVecSize"] = tile_description.explicit_vector_sizes[1]
  234:       gemm_op_extra_args["ScaleFactorKVecSize"] = tile_description.explicit_vector_sizes[2]
  235:       assert is_blockwise(gemm_kind)
  236:     else:
  237:       assert not is_blockwise(gemm_kind)
  238: 
  239:     A_dtype = data_type["a_type"]
  240:     B_dtype = data_type["b_type"]
  241:     A_dtype_bits = DataTypeSize[A_dtype]
  242:     B_dtype_bits = DataTypeSize[B_dtype]
  243:     is_A_dtype_narrow = A_dtype_bits < B_dtype_bits
  244:     if is_A_dtype_narrow:
  245:       narrow_dtype, wide_dtype = (A_dtype, B_dtype)
  246:       narrow_dtype_bits, wide_dtype_bits = (A_dtype_bits, B_dtype_bits)
  247:     else:
  248:       narrow_dtype, wide_dtype = (B_dtype, A_dtype)
  249:       narrow_dtype_bits, wide_dtype_bits = (B_dtype_bits, A_dtype_bits)
  250: 
  251:     mixed_input_modes = [None]
  252:     if narrow_dtype_bits != wide_dtype_bits:
  253:       if narrow_dtype == DataType.s4 and (wide_dtype == DataType.e4m3 or wide_dtype == DataType.e5m2):
  254:         mixed_input_modes = [MixedInputMode.ScaleOnly]
  255:       else:
  256:         mixed_input_modes = [MixedInputMode.ConvertOnly, MixedInputMode.ScaleOnly, MixedInputMode.ScaleWithZeroPoint]
  257: 
  258:     mixed_input_shuffle_options = [False]
  259:     if (mixed_input_modes[0] is not None) and (wide_dtype_bits == 16) and (narrow_dtype_bits == 4 or narrow_dtype_bits == 8):
  260:       mixed_input_shuffle_options = [False, True]
  261: 
  262:     for mixed_input_mode, mixed_input_shuffle in product(mixed_input_modes, mixed_input_shuffle_options):
  263:       operation = GemmOperation(
  264:           gemm_kind, tile_description.minimum_compute_capability,
  265:           tile_description, A, B, C, element_compute, epilogue_functor, swizzling_functor, D,
  266:           kernel_schedule, epilogue_schedule, tile_scheduler,
  267:           mixed_input_mode=mixed_input_mode, mixed_input_shuffle=mixed_input_shuffle, **gemm_op_extra_args)
  268:       manifest.append(operation)
  269:       operations.append(operation)
  270: 
  271:   return operations
```
**EN:** Defines `CreateGemmUniversal3xOperator()`, which creates gemm universal3x operator and appends it to the surrounding workflow. Key helper calls include product, type, TensorDescription, get, len, is_block_scaled.

**CN:** 定义 `CreateGemmUniversal3xOperator()`，用于创建 gemm universal3x operator 并将其接入周边流程。 其中会调用的重要辅助函数包括 product, type, TensorDescription, get, len, is_block_scaled。

### L272-L273 — Comments

```python
  272: 
  273: # Generates 3.0 API based GemmUniversal API kernels. Alignment constraints are folded in with layouts
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L274-L319 — Function `CreateSparseGemmUniversal3xOperator`

```python
  274: def CreateSparseGemmUniversal3xOperator(
  275:     manifest, layouts, tile_descriptions, data_types,
  276:     schedules = [[KernelScheduleType.ScheduleAuto, EpilogueScheduleType.ScheduleAuto]],
  277:     complex_transforms=None,
  278:     epilogue_functor=EpilogueFunctor.LinearCombination,
  279:     swizzling_functor=SwizzlingFunctor.Identity1,
  280:     tile_schedulers=[TileSchedulerType.Default]):
  281: 
  282:   if type(data_types) is dict:
  283:     data_types = [data_types]
  284: 
  285:   for s in schedules:
  286:     assert(len(s) == 2)
  287: 
  288:   if complex_transforms is None:
  289:     complex_transforms = [(ComplexTransform.none, ComplexTransform.none), ]
  290: 
  291:   operations = []
  292: 
  293:   # by default, only generate the largest tile and largest alignment
  294:   if manifest.kernel_filter == '':
  295:     tile_descriptions = [tile_descriptions[0]]
  296: 
  297:   combinations = product(layouts, tile_descriptions, data_types, complex_transforms, schedules, tile_schedulers)
  298:   for layout, tile_description, data_type, complex_transform, schedules, tile_scheduler in combinations:
  299:     kernel_schedule, epilogue_schedule = schedules
  300:     A = TensorDescription(
  301:         data_type["a_type"], layout[0][0], layout[0][1], complex_transform[0])
  302:     B = TensorDescription(
  303:         data_type["b_type"], layout[1][0], layout[1][1], complex_transform[1])
  304: 
  305:     # Currently assume tensor C/D have same layout requirement.
  306:     C = TensorDescription(data_type["c_type"], layout[2][0], layout[2][1])
  307:     D = TensorDescription(data_type["d_type"], layout[2][0], layout[2][1])
  308: 
  309:     element_compute = data_type.get("epi_type", data_type["acc_type"])
  310: 
  311:     operation = GemmOperation(
  312:         GemmKind.SparseUniversal3x, tile_description.minimum_compute_capability,
  313:         tile_description, A, B, C, element_compute, epilogue_functor, swizzling_functor, D,
  314:         kernel_schedule, epilogue_schedule, tile_scheduler)
  315: 
  316:     manifest.append(operation)
  317:     operations.append(operation)
  318: 
  319:   return operations
```
**EN:** Defines `CreateSparseGemmUniversal3xOperator()`, which creates sparse gemm universal3x operator and appends it to the surrounding workflow. Key helper calls include product, type, TensorDescription, get, GemmOperation, append.

**CN:** 定义 `CreateSparseGemmUniversal3xOperator()`，用于创建 sparse gemm universal3x operator 并将其接入周边流程。 其中会调用的重要辅助函数包括 product, type, TensorDescription, get, GemmOperation, append。

### L320-L321 — Comments

```python
  320: 
  321: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L322-L357 — Function `CreateSparseGemmOperator`

```python
  322: def CreateSparseGemmOperator(manifest, layouts, tile_descriptions, data_type, \
  323:   alignment_constraints, complex_transforms = None, epilogue_functor = EpilogueFunctor.LinearCombination, \
  324:   swizzling_functor = SwizzlingFunctor.Identity8):
  325: 
  326:   if complex_transforms is None:
  327:     complex_transforms = [(ComplexTransform.none, ComplexTransform.none),]
  328: 
  329:   element_a, element_b, element_c, element_epilogue = data_type
  330: 
  331:   gemm_kinds = [GemmKind.Sparse]
  332: 
  333:   operations = []
  334: 
  335:   # by default, only generate the largest tile and largest alignment
  336:   if manifest.kernel_filter == '':
  337:     tile_descriptions = [tile_descriptions[0],]
  338:     alignment_constraints = [alignment_constraints[0],]
  339: 
  340:   for layout in layouts:
  341:     for tile_description in tile_descriptions:
  342:       for alignment in alignment_constraints:
  343:         for complex_transform in complex_transforms:
  344: 
  345:             alignment_c = min(8, alignment)
  346: 
  347:             A = TensorDescription(element_a, layout[0], alignment, complex_transform[0])
  348:             B = TensorDescription(element_b, layout[1], alignment, complex_transform[1])
  349:             C = TensorDescription(element_c, layout[2], alignment_c)
  350: 
  351:             new_operation = GemmOperation(GemmKind.Sparse, tile_description.minimum_compute_capability, \
  352:               tile_description, A, B, C, element_epilogue, epilogue_functor, swizzling_functor)
  353: 
  354:             manifest.append(new_operation)
  355:             operations.append(new_operation)
  356: 
  357:   return operations
```
**EN:** Defines `CreateSparseGemmOperator()`, which creates sparse gemm operator and appends it to the surrounding workflow. Key helper calls include min, TensorDescription, GemmOperation, append.

**CN:** 定义 `CreateSparseGemmOperator()`，用于创建 sparse gemm operator 并将其接入周边流程。 其中会调用的重要辅助函数包括 min, TensorDescription, GemmOperation, append。

### L358-L359 — Comments

```python
  358: 
  359: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L360-L390 — Function `CreateGemmPlanarComplexOperator`

```python
  360: def CreateGemmPlanarComplexOperator(manifest, layouts, tile_descriptions, data_type, \
  361:   alignment_constraints, complex_transforms):
  362: 
  363:   if complex_transforms is None:
  364:     complex_transforms = [(ComplexTransform.none, ComplexTransform.none),]
  365: 
  366:   element_a, element_b, element_c, element_epilogue = data_type
  367: 
  368:   gemm_kinds = [GemmKind.PlanarComplex, GemmKind.PlanarComplexArray]
  369: 
  370:   # by default, only generate the largest tile and largest alignment
  371:   if manifest.kernel_filter == '':
  372:     tile_descriptions = [tile_descriptions[0],]
  373:     alignment_constraints = [alignment_constraints[0],]
  374: 
  375:   for gemm_kind in gemm_kinds:
  376:     for layout in layouts:
  377:       for tile_description in tile_descriptions:
  378:         for alignment in alignment_constraints:
  379:           for complex_transform in complex_transforms:
  380: 
  381:             alignment_c = min(8, alignment)
  382: 
  383:             A = TensorDescription(element_a, layout[0], alignment, complex_transform[0])
  384:             B = TensorDescription(element_b, layout[1], alignment, complex_transform[1])
  385:             C = TensorDescription(element_c, layout[2], alignment_c)
  386: 
  387:             manifest.append(GemmOperation(gemm_kind, \
  388:               tile_description.minimum_compute_capability, \
  389:               tile_description, A, B, C, element_epilogue))
  390:   return
```
**EN:** Defines `CreateGemmPlanarComplexOperator()`, which creates gemm planar complex operator and appends it to the surrounding workflow. Key helper calls include min, TensorDescription, append, GemmOperation.

**CN:** 定义 `CreateGemmPlanarComplexOperator()`，用于创建 gemm planar complex operator 并将其接入周边流程。 其中会调用的重要辅助函数包括 min, TensorDescription, append, GemmOperation。

### L391-L392 — Comments

```python
  391: 
  392: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L393-L426 — Function `CreateGemmGroupedOperator`

```python
  393: def CreateGemmGroupedOperator(manifest, layouts, tile_descriptions, data_type, \
  394:   alignment_constraints, complex_transforms = None, epilogue_functor = EpilogueFunctor.LinearCombination, \
  395:   swizzling_functor = SwizzlingFunctor.Identity8):
  396: 
  397:   if complex_transforms is None:
  398:     complex_transforms = [(ComplexTransform.none, ComplexTransform.none),]
  399: 
  400:   element_a, element_b, element_c, element_epilogue = data_type
  401: 
  402:   operations = []
  403: 
  404:   # by default, only generate the largest tile and largest alignment
  405:   if manifest.kernel_filter == '':
  406:     tile_descriptions = [tile_descriptions[0],]
  407:     alignment_constraints = [alignment_constraints[0],]
  408: 
  409:   for layout in layouts:
  410:     for tile_description in tile_descriptions:
  411:       for alignment in alignment_constraints:
  412:         for complex_transform in complex_transforms:
  413: 
  414:             alignment_c = min(8, alignment)
  415: 
  416:             A = TensorDescription(element_a, layout[0], alignment, complex_transform[0])
  417:             B = TensorDescription(element_b, layout[1], alignment, complex_transform[1])
  418:             C = TensorDescription(element_c, layout[2], alignment_c)
  419: 
  420:             new_operation = GroupedGemmOperation(GemmKind.Grouped, tile_description.minimum_compute_capability, \
  421:               tile_description, A, B, C, element_epilogue, epilogue_functor, swizzling_functor)
  422: 
  423:             manifest.append(new_operation)
  424:             operations.append(new_operation)
  425: 
  426:   return operations
```
**EN:** Defines `CreateGemmGroupedOperator()`, which creates gemm grouped operator and appends it to the surrounding workflow. Key helper calls include min, TensorDescription, GroupedGemmOperation, append.

**CN:** 定义 `CreateGemmGroupedOperator()`，用于创建 gemm grouped operator 并将其接入周边流程。 其中会调用的重要辅助函数包括 min, TensorDescription, GroupedGemmOperation, append。

### L427-L428 — Comments

```python
  427: 
  428: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L429-L473 — Function `CreateRankKOperator`

```python
  429: def CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, data_type, \
  430:   alignment_constraints, blas_mode, epilogue_functor = EpilogueFunctor.LinearCombination, \
  431:   swizzling_functor = SwizzlingFunctor.Identity8):
  432: 
  433:   element_a, element_c, element_epilogue = data_type
  434: 
  435:   operations = []
  436: 
  437:   # by default, only generate the largest tile and largest alignment
  438:   if manifest.kernel_filter == '':
  439:     tile_descriptions = [tile_descriptions[0],]
  440:     alignment_constraints = [alignment_constraints[0],]
  441: 
  442:   for layout in layouts:
  443:     for fill_mode in fill_modes:
  444:       for tile_description in tile_descriptions:
  445:         for alignment in alignment_constraints:
  446: 
  447:           # SERK supported layouts (RowMajor, ColumnMajor) with no conjugation
  448:           complex_transform = ComplexTransform.none
  449: 
  450:           # HERK supported layouts (RowMajor + conj, ColumnMajor)
  451:           if blas_mode == BlasMode.hermitian and layout[0] == LayoutType.RowMajor:
  452:             complex_transform = ComplexTransform.conj
  453: 
  454:           alignment_c = 1 # Alignment only applies to A in SYRK
  455: 
  456:           A = TensorDescription(element_a, layout[0], alignment, complex_transform)
  457:           C = SymmetricTensorDescription(element_c, layout[1], fill_mode, alignment_c)
  458: 
  459:           # Rank-K update
  460:           new_operation = RankKOperation(RankKKind.Universal, tile_description.minimum_compute_capability, \
  461:             tile_description, A, C, element_epilogue, epilogue_functor, swizzling_functor, blas_mode)
  462: 
  463:           manifest.append(new_operation)
  464:           operations.append(new_operation)
  465: 
  466:           # Rank-2K update
  467:           new_operation = Rank2KOperation(RankKKind.Universal, tile_description.minimum_compute_capability, \
  468:             tile_description, A, C, element_epilogue, epilogue_functor, swizzling_functor, blas_mode)
  469: 
  470:           manifest.append(new_operation)
  471:           operations.append(new_operation)
  472: 
  473:   return operations
```
**EN:** Defines `CreateRankKOperator()`, which creates rank koperator and appends it to the surrounding workflow. Key helper calls include TensorDescription, SymmetricTensorDescription, RankKOperation, append, Rank2KOperation.

**CN:** 定义 `CreateRankKOperator()`，用于创建 rank koperator 并将其接入周边流程。 其中会调用的重要辅助函数包括 TensorDescription, SymmetricTensorDescription, RankKOperation, append, Rank2KOperation。

### L474-L475 — Comments

```python
  474: 
  475: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L476-L513 — Function `CreateTrmmOperator`

```python
  476: def CreateTrmmOperator(manifest, layouts, side_modes, fill_modes, diag_types, tile_descriptions, data_type, \
  477:   alignment_constraints, complex_transforms = None, epilogue_functor = EpilogueFunctor.LinearCombination, \
  478:   swizzling_functor = SwizzlingFunctor.Identity8):
  479: 
  480:   if complex_transforms is None:
  481:     complex_transforms = [(ComplexTransform.none),]
  482: 
  483:   element_a, element_b, element_c, element_epilogue = data_type
  484: 
  485:   operations = []
  486: 
  487:   # by default, only generate the largest tile and largest alignment
  488:   if manifest.kernel_filter == '':
  489:     tile_descriptions = [tile_descriptions[0],]
  490:     alignment_constraints = [alignment_constraints[0],]
  491: 
  492:   for layout in layouts:
  493:     for side_mode in side_modes:
  494:       for fill_mode in fill_modes:
  495:         for diag_type in diag_types:
  496:           for tile_description in tile_descriptions:
  497:             for alignment in alignment_constraints:
  498:               for complex_transform in complex_transforms:
  499: 
  500:                   alignment_c = min(8, alignment)
  501: 
  502:                   A = TriangularTensorDescription(element_a, layout[0], side_mode, fill_mode, diag_type,
  503:                                                   alignment, complex_transform)
  504:                   B = TensorDescription(element_b, layout[1], alignment)
  505:                   C = TensorDescription(element_c, layout[2], alignment_c)
  506: 
  507:                   new_operation = TrmmOperation(TrmmKind.Universal, tile_description.minimum_compute_capability, \
  508:                     tile_description, A, B, C, element_epilogue, epilogue_functor, swizzling_functor)
  509: 
  510:                   manifest.append(new_operation)
  511:                   operations.append(new_operation)
  512: 
  513:   return operations
```
**EN:** Defines `CreateTrmmOperator()`, which creates trmm operator and appends it to the surrounding workflow. Key helper calls include min, TriangularTensorDescription, TensorDescription, TrmmOperation, append.

**CN:** 定义 `CreateTrmmOperator()`，用于创建 trmm operator 并将其接入周边流程。 其中会调用的重要辅助函数包括 min, TriangularTensorDescription, TensorDescription, TrmmOperation, append。

### L514-L515 — Comments

```python
  514: 
  515: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L516-L560 — Function `CreateSymmOperator`

```python
  516: def CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, data_type, \
  517:   alignment_constraints, blas_mode, epilogue_functor = EpilogueFunctor.LinearCombination, \
  518:   swizzling_functor = SwizzlingFunctor.Identity8):
  519: 
  520:   element_a, element_b, element_c, element_epilogue = data_type
  521: 
  522:   operations = []
  523: 
  524:   # by default, only generate the largest tile and largest alignment
  525:   if manifest.kernel_filter == '':
  526:     tile_descriptions = [tile_descriptions[0],]
  527:     alignment_constraints = [alignment_constraints[0],]
  528: 
  529:   for layout in layouts:
  530:     for side_mode in side_modes:
  531:       for fill_mode in fill_modes:
  532:         for tile_description in tile_descriptions:
  533:           for alignment in alignment_constraints:
  534: 
  535:             # SYMM supported layouts (RowMajor, ColumnMajor) with no conjugation
  536:             complex_transform = ComplexTransform.none
  537: 
  538:             alignment_a = 1 # No vectorized access for the triangular matrix
  539:             alignment_c = min(8, alignment)
  540: 
  541:             A = SymmetricTensorDescription(element_a, layout[0], fill_mode, alignment_a, complex_transform, side_mode)
  542:             # tensor A and B have same data type and layout
  543:             B = TensorDescription(element_b, layout[0], alignment)
  544:             C = TensorDescription(element_c, layout[1], alignment_c)
  545: 
  546:             # SYMM/HEMM update
  547:             new_operation = SymmOperation(SymmKind.Universal, tile_description.minimum_compute_capability, \
  548:               tile_description, A, B, C, element_epilogue, epilogue_functor, swizzling_functor, blas_mode)
  549: 
  550:             manifest.append(new_operation)
  551:             operations.append(new_operation)
  552: 
  553:             # SYMM/HEMM update
  554:             new_operation = SymmOperation(SymmKind.Universal, tile_description.minimum_compute_capability, \
  555:               tile_description, A, B, C, element_epilogue, epilogue_functor, swizzling_functor, blas_mode)
  556: 
  557:             manifest.append(new_operation)
  558:             operations.append(new_operation)
  559: 
  560:   return operations
```
**EN:** Defines `CreateSymmOperator()`, which creates symm operator and appends it to the surrounding workflow. Key helper calls include min, SymmetricTensorDescription, TensorDescription, SymmOperation, append.

**CN:** 定义 `CreateSymmOperator()`，用于创建 symm operator 并将其接入周边流程。 其中会调用的重要辅助函数包括 min, SymmetricTensorDescription, TensorDescription, SymmOperation, append。

### L561-L574 — Comments

```python
  561: 
  562: ###########################################################################################################
  563: #   ConvolutionOperator support variations
  564: #        ____________________________________________________________________
  565: #         ConvolutionalOperator |      Analytic          |    Optimized
  566: #        ____________________________________________________________________
  567: #        |       Fprop          |     (strided)          |    (strided)
  568: #        |       Dgrad          |     (strided, unity*)  |    (strided, unity)
  569: #        |       Wgrad          |     (strided)          |    (strided)
  570: #        ____________________________________________________________________
  571: #
  572: # Note :  Operator marked (*) are supported but not generated to keep the instantiated kernel count low
  573: ###########################################################################################################
  574: # Convolution for 2D operations
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L575-L681 — Function `CreateConv2dOperator`

```python
  575: def CreateConv2dOperator(manifest, layout, tile_descriptions, data_type, alignment_constraints, \
  576:   conv_kinds = [ConvKind.Fprop, ConvKind.Dgrad, ConvKind.Wgrad], \
  577:   epilogue_functor = EpilogueFunctor.LinearCombination, swizzling_functor = SwizzlingFunctor.Identity4):
  578: 
  579:   element_a, element_b, element_c, element_epilogue = data_type
  580: 
  581:   # one exceptional case
  582: 
  583:   # iterator algorithm (analytic and optimized)
  584:   iterator_algorithms = [IteratorAlgorithm.Analytic, IteratorAlgorithm.Optimized]
  585: 
  586:   # by default, only generate the largest tile size, largest alignment, and optimized iterator
  587:   if manifest.kernel_filter == '':
  588:     tile_descriptions = [tile_descriptions[0],]
  589:     alignment_constraints = [alignment_constraints[0],]
  590:     iterator_algorithms = [IteratorAlgorithm.Optimized]
  591: 
  592:   operations = []
  593: 
  594:   for tile in tile_descriptions:
  595:     for alignment in alignment_constraints:
  596: 
  597:       alignment_c = min(8, alignment)
  598: 
  599:       A = TensorDescription(element_a, layout[0], alignment)
  600:       B = TensorDescription(element_b, layout[1], alignment)
  601:       C = TensorDescription(element_c, layout[2], alignment_c)
  602: 
  603:       swizzling_functor_ = swizzling_functor
  604: 
  605:       #
  606:       # Conv2d Fprop
  607:       #
  608:       if ConvKind.Fprop in conv_kinds:
  609: 
  610:         # Strided support for Analytic and Optimized Fprop
  611:         for iterator_algorithm in iterator_algorithms:
  612:           new_operations = [
  613:             # None grouped kernel
  614:             Conv2dOperation(ConvKind.Fprop, iterator_algorithm, tile.minimum_compute_capability, tile,\
  615:               A, B, C, element_epilogue, StrideSupport.Unity, epilogue_functor, swizzling_functor_),
  616:           ]
  617: 
  618:           # Instance group conv kernel
  619:           if tile.math_instruction.opcode_class == OpcodeClass.TensorOp and A.layout == LayoutType.TensorNHWC and \
  620:             tile.minimum_compute_capability >= 80:
  621:             # SingleGroup kernel
  622:             new_operations.append(Conv2dOperation(ConvKind.Fprop, iterator_algorithm, tile.minimum_compute_capability, tile,\
  623:               A, B, C, element_epilogue, StrideSupport.Unity, epilogue_functor, swizzling_functor_, group_mode=GroupMode.SingleGroup))
  624: 
  625:             # Analytic iterator supports MultipleGroup mode
  626:             if iterator_algorithm == IteratorAlgorithm.Analytic:
  627:               new_operations.append(Conv2dOperation(ConvKind.Fprop, iterator_algorithm, tile.minimum_compute_capability, tile,\
  628:                 A, B, C, element_epilogue, StrideSupport.Unity, epilogue_functor, swizzling_functor_, group_mode=GroupMode.MultipleGroup))
  629: 
  630:           for new_operation in new_operations:
  631:             manifest.append(new_operation)
  632:             operations.append(new_operation)
  633: 
  634:       #
  635:       # Conv2d Dgrad
  636:       #
  637:       if ConvKind.Dgrad in conv_kinds:
  638: 
  639:         # Unity stride for Analytic and Optimized Dgrad
  640:         for iterator_algorithm in iterator_algorithms:
  641:           new_operation = Conv2dOperation(ConvKind.Dgrad, iterator_algorithm, tile.minimum_compute_capability, tile,\
  642:             A, B, C, element_epilogue, StrideSupport.Unity, epilogue_functor, swizzling_functor_)
  643: 
  644:           manifest.append(new_operation)
  645:           operations.append(new_operation)
  646: 
  647:         # Strided support for Analytic Dgrad
  648:         # strided dgrad uses a special threadblock swizzle
  649:         # note that SwizzlingFunctor.StridedDgradHorizontal might be
  650:         # better for problem sizes with large activation channel count
  651:         swizzling_functor_strided_dgrad_ = SwizzlingFunctor.StridedDgradIdentity1
  652: 
  653:         if IteratorAlgorithm.Analytic in iterator_algorithms:
  654:           new_operation = Conv2dOperation(ConvKind.Dgrad, IteratorAlgorithm.Analytic, tile.minimum_compute_capability, tile,\
  655:             A, B, C, element_epilogue, StrideSupport.Strided, epilogue_functor, swizzling_functor_strided_dgrad_)
  656: 
  657:           manifest.append(new_operation)
  658:           operations.append(new_operation)
  659: 
  660:         # Strided support for Optimized Dgrad
  661:         if IteratorAlgorithm.Optimized in iterator_algorithms:
  662:           new_operation = Conv2dOperation(ConvKind.Dgrad, IteratorAlgorithm.Optimized, tile.minimum_compute_capability, tile,\
  663:             A, B, C, element_epilogue, StrideSupport.Strided, epilogue_functor, swizzling_functor_strided_dgrad_)
  664: 
  665:           manifest.append(new_operation)
  666:           operations.append(new_operation)
  667: 
  668:       #
  669:       # Conv2d Wgrad
  670:       #
  671:       if ConvKind.Wgrad in conv_kinds:
  672: 
  673:         # Strided support for Analytic and Optimized Wgrad
  674:         for iterator_algorithm in iterator_algorithms:
  675:           new_operation = Conv2dOperation(ConvKind.Wgrad, iterator_algorithm, tile.minimum_compute_capability, tile,\
  676:             A, B, C, element_epilogue, StrideSupport.Strided, epilogue_functor, swizzling_functor_)
  677: 
  678:           manifest.append(new_operation)
  679:           operations.append(new_operation)
  680: 
  681:   return operations
```
**EN:** Defines `CreateConv2dOperator()`, which creates conv2d operator and appends it to the surrounding workflow. Key helper calls include min, TensorDescription, Conv2dOperation, append.

**CN:** 定义 `CreateConv2dOperator()`，用于创建 conv2d operator 并将其接入周边流程。 其中会调用的重要辅助函数包括 min, TensorDescription, Conv2dOperation, append。

### L682-L683 — Comments

```python
  682: 
  683: # Convolution for 2D operations specialized for few channels
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L684-L728 — Function `CreateConv2dFixedChannelsOperator`

```python
  684: def CreateConv2dFixedChannelsOperator(manifest, layout, tile_descriptions, data_type, channel_counts, \
  685:   conv_kinds = [ConvKind.Fprop, ConvKind.Dgrad, ConvKind.Wgrad], \
  686:   epilogue_functor = EpilogueFunctor.LinearCombination, swizzling_functor = SwizzlingFunctor.Identity4):
  687: 
  688:   element_a, element_b, element_c, element_epilogue = data_type
  689: 
  690:   # one exceptional case
  691: 
  692:   # iterator algorithm (analytic and optimized)
  693:   iterator_algorithms = [IteratorAlgorithm.FixedChannels,]
  694: 
  695:   # by default, only generate the largest tile size, largest alignment, and optimized iterator
  696:   if manifest.kernel_filter == '':
  697:     tile_descriptions = [tile_descriptions[0],]
  698:     channel_counts = [channel_counts[0],]
  699: 
  700:   operations = []
  701: 
  702: 
  703: 
  704:   for tile in tile_descriptions:
  705:     for channel_count in channel_counts:
  706: 
  707:       alignment_c = EpilogueAlignment(channel_count, tile)
  708: 
  709:       A = TensorDescription(element_a, layout[0], channel_count)
  710:       B = TensorDescription(element_b, layout[1], channel_count)
  711:       C = TensorDescription(element_c, layout[2], alignment_c)
  712: 
  713:       swizzling_functor_ = swizzling_functor
  714: 
  715:       #
  716:       # Conv2d Fprop
  717:       #
  718:       if ConvKind.Fprop in conv_kinds:
  719: 
  720:         # Strided support for Analytic and Optimized Fprop
  721:         for iterator_algorithm in iterator_algorithms:
  722:           new_operation = Conv2dOperation(ConvKind.Fprop, iterator_algorithm, tile.minimum_compute_capability, tile,\
  723:             A, B, C, element_epilogue, StrideSupport.Strided, epilogue_functor, swizzling_functor_)
  724: 
  725:           manifest.append(new_operation)
  726:           operations.append(new_operation)
  727: 
  728:   return operations
```
**EN:** Defines `CreateConv2dFixedChannelsOperator()`, which creates conv2d fixed channels operator and appends it to the surrounding workflow. Key helper calls include EpilogueAlignment, TensorDescription, Conv2dOperation, append.

**CN:** 定义 `CreateConv2dFixedChannelsOperator()`，用于创建 conv2d fixed channels operator 并将其接入周边流程。 其中会调用的重要辅助函数包括 EpilogueAlignment, TensorDescription, Conv2dOperation, append。

### L729-L730 — Comments

```python
  729: 
  730: # Convolution for 2D operations specialized for few channels
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L731-L773 — Function `CreateConv2dFewChannelsOperator`

```python
  731: def CreateConv2dFewChannelsOperator(manifest, layout, tile_descriptions, data_type, channel_counts, \
  732:   conv_kinds = [ConvKind.Fprop, ConvKind.Dgrad, ConvKind.Wgrad], \
  733:   epilogue_functor = EpilogueFunctor.LinearCombination, swizzling_functor = SwizzlingFunctor.Identity4):
  734: 
  735:   element_a, element_b, element_c, element_epilogue = data_type
  736: 
  737:   # one exceptional case
  738: 
  739:   # iterator algorithm (analytic and optimized)
  740:   iterator_algorithms = [IteratorAlgorithm.FewChannels,]
  741: 
  742:   # by default, only generate the largest tile size, largest alignment, and optimized iterator
  743:   if manifest.kernel_filter == '':
  744:     tile_descriptions = [tile_descriptions[0],]
  745:     channel_counts = [channel_counts[0],]
  746: 
  747:   operations = []
  748: 
  749:   for tile in tile_descriptions:
  750:     for channel_count in channel_counts:
  751: 
  752:       alignment_c = EpilogueAlignment(channel_count, tile)
  753: 
  754:       A = TensorDescription(element_a, layout[0], channel_count)
  755:       B = TensorDescription(element_b, layout[1], channel_count)
  756:       C = TensorDescription(element_c, layout[2], alignment_c)
  757: 
  758:       swizzling_functor_ = swizzling_functor
  759: 
  760:       #
  761:       # Conv2d Fprop
  762:       #
  763:       if ConvKind.Fprop in conv_kinds:
  764: 
  765:         # Strided support for Analytic and Optimized Fprop
  766:         for iterator_algorithm in iterator_algorithms:
  767:           new_operation = Conv2dOperation(ConvKind.Fprop, iterator_algorithm, tile.minimum_compute_capability, tile,\
  768:             A, B, C, element_epilogue, StrideSupport.Strided, epilogue_functor, swizzling_functor_)
  769: 
  770:           manifest.append(new_operation)
  771:           operations.append(new_operation)
  772: 
  773:   return operations
```
**EN:** Defines `CreateConv2dFewChannelsOperator()`, which creates conv2d few channels operator and appends it to the surrounding workflow. Key helper calls include EpilogueAlignment, TensorDescription, Conv2dOperation, append.

**CN:** 定义 `CreateConv2dFewChannelsOperator()`，用于创建 conv2d few channels operator 并将其接入周边流程。 其中会调用的重要辅助函数包括 EpilogueAlignment, TensorDescription, Conv2dOperation, append。

### L774-L775 — Comments

```python
  774: 
  775: # Convolution for 3D operations
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L776-L848 — Function `CreateConv3dOperator`

```python
  776: def CreateConv3dOperator(manifest, layout, tile_descriptions, data_type, alignment, \
  777:   conv_kinds = [ConvKind.Fprop, ConvKind.Dgrad, ConvKind.Wgrad], epilogue_functor = EpilogueFunctor.LinearCombination):
  778: 
  779:   element_a, element_b, element_c, element_epilogue = data_type
  780: 
  781:   # one exceptional case
  782:   alignment_c = min(8, alignment)
  783: 
  784:   # iterator algorithm (analytic and optimized)
  785:   iterator_algorithms = [IteratorAlgorithm.Analytic, IteratorAlgorithm.Optimized]
  786: 
  787:   # by default, only generate the largest tile size and optimized iterators
  788:   if manifest.kernel_filter == '':
  789:     tile_descriptions = [tile_descriptions[0],]
  790:     iterator_algorithms = [IteratorAlgorithm.Optimized]
  791: 
  792:   operations = []
  793: 
  794:   # All tile sizes for Conv3dFprop and Conv3dWgrad
  795:   for tile in tile_descriptions:
  796:     A = TensorDescription(element_a, layout, alignment)
  797:     B = TensorDescription(element_b, layout, alignment)
  798:     C = TensorDescription(element_c, layout, alignment_c)
  799: 
  800:     #
  801:     # Conv3d Fprop
  802:     #
  803:     if ConvKind.Fprop in conv_kinds:
  804:       # Strided support for Analytic and Optimized Fprop
  805:       for iterator_algorithm in iterator_algorithms:
  806:         new_operation = Conv3dOperation(ConvKind.Fprop, iterator_algorithm, tile.minimum_compute_capability, tile,\
  807:                                         A, B, C, element_epilogue, StrideSupport.Strided)
  808:         manifest.append(new_operation)
  809:         operations.append(new_operation)
  810:     #
  811:     # Conv3d Wgrad
  812:     #
  813:     if ConvKind.Wgrad in conv_kinds:
  814: 
  815:       # Strided support for Analytic and Optimized Wgrad
  816:       for iterator_algorithm in iterator_algorithms:
  817:         new_operation = Conv3dOperation(ConvKind.Wgrad, iterator_algorithm, tile.minimum_compute_capability, tile,\
  818:           A, B, C, element_epilogue, StrideSupport.Strided, epilogue_functor)
  819:         manifest.append(new_operation)
  820:         operations.append(new_operation)
  821: 
  822:   # All tile sizes for Conv3dDgrad
  823:   for tile in tile_descriptions:
  824: 
  825:     A = TensorDescription(element_a, layout, alignment)
  826:     B = TensorDescription(element_b, layout, alignment)
  827:     C = TensorDescription(element_c, layout, alignment_c)
  828: 
  829:     #
  830:     # Conv3d Dgrad
  831:     #
  832:     if ConvKind.Dgrad in conv_kinds:
  833:       # Unity stride for Optimized Dgrad
  834:       new_operation = Conv3dOperation(ConvKind.Dgrad, IteratorAlgorithm.Optimized, tile.minimum_compute_capability, tile,\
  835:         A, B, C, element_epilogue, StrideSupport.Unity, epilogue_functor)
  836: 
  837:       manifest.append(new_operation)
  838:       operations.append(new_operation)
  839: 
  840:       # Strided support for Analytic Dgrad
  841:       # Conv3dDgrad has a naive strided support which does not cut down redundant MMAs
  842:       new_operation = Conv3dOperation(ConvKind.Dgrad, IteratorAlgorithm.Analytic, tile.minimum_compute_capability, tile,\
  843:         A, B, C, element_epilogue, StrideSupport.Strided, epilogue_functor)
  844: 
  845:       manifest.append(new_operation)
  846:       operations.append(new_operation)
  847: 
  848:   return operations
```
**EN:** Defines `CreateConv3dOperator()`, which creates conv3d operator and appends it to the surrounding workflow. Key helper calls include min, TensorDescription, Conv3dOperation, append.

**CN:** 定义 `CreateConv3dOperator()`，用于创建 conv3d operator 并将其接入周边流程。 其中会调用的重要辅助函数包括 min, TensorDescription, Conv3dOperation, append。

### L849-L850 — Comments

```python
  849: 
  850: # Convolution for Depthwise 2d conv
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L851-L905 — Function `CreateDepthwiseConv2dOperator`

```python
  851: def CreateDepthwiseConv2dOperator(manifest, layout, tile_descriptions, data_type, alignment_constraints, \
  852:   conv_kinds = [ConvKind.Fprop, ConvKind.Dgrad, ConvKind.Wgrad], \
  853:   epilogue_functor = EpilogueFunctor.LinearCombination, swizzling_functor = SwizzlingFunctor.Identity4):
  854: 
  855:   element_a, element_b, element_c, element_epilogue = data_type
  856: 
  857:   # iterator algorithm (FixedStrideDilation, Optimized)
  858:   iterator_algorithms = [IteratorAlgorithm.FixedStrideDilation, IteratorAlgorithm.Optimized]
  859: 
  860:   # by default, only generate the largest tile size, largest alignment, and optimized iterator
  861:   if manifest.kernel_filter == '':
  862:     tile_descriptions = [tile_descriptions[0],]
  863:     alignment_constraints = [alignment_constraints[0],]
  864: 
  865:   operations = []
  866: 
  867:   for tile in tile_descriptions:
  868:     for alignment in alignment_constraints:
  869: 
  870:       alignment_c = min(8, alignment)
  871: 
  872:       A = TensorDescription(element_a, layout[0], alignment)
  873:       B = TensorDescription(element_b, layout[1], alignment)
  874:       C = TensorDescription(element_c, layout[2], alignment_c)
  875: 
  876:       swizzling_functor_ = swizzling_functor
  877: 
  878:       if ConvKind.Fprop in conv_kinds:
  879: 
  880:         # Strided support for Optimized and FixedStridedDilation Depthwise Conv
  881:         for iterator_algorithm in iterator_algorithms:
  882:           stride_support = StrideSupport.Strided
  883:           if iterator_algorithm == IteratorAlgorithm.FixedStrideDilation:
  884:               if tile.stride == [-1, -1] or tile.dilation == [-1,-1]:
  885:                 continue
  886:               stride_support = StrideSupport.Fixed
  887: 
  888:           if iterator_algorithm == IteratorAlgorithm.Optimized:
  889:               if tile.stride != [-1, -1] or tile.dilation != [-1,-1]:
  890:                 continue
  891:           new_operation = Conv2dOperation(ConvKind.Fprop,
  892:                                           iterator_algorithm,
  893:                                           tile.minimum_compute_capability,
  894:                                           tile,
  895:                                           A, B, C,
  896:                                           element_epilogue,
  897:                                           stride_support,
  898:                                           epilogue_functor,
  899:                                           swizzling_functor_,
  900:                                           group_mode=GroupMode.Depthwise)
  901: 
  902:           manifest.append(new_operation)
  903:           operations.append(new_operation)
  904: 
  905:   return operations
```
**EN:** Defines `CreateDepthwiseConv2dOperator()`, which creates depthwise conv2d operator and appends it to the surrounding workflow. Key helper calls include min, TensorDescription, Conv2dOperation, append.

**CN:** 定义 `CreateDepthwiseConv2dOperator()`，用于创建 depthwise conv2d operator 并将其接入周边流程。 其中会调用的重要辅助函数包括 min, TensorDescription, Conv2dOperation, append。

### L907-L907 — Class `ConvOperation3x`

```python
  907: class ConvOperation3x:
```
**EN:** Introduces class `ConvOperation3x`. The docstring says: "All parameters of a CUTLASS 3 convolution operation."; the class body implements that role.

**CN:** 引入类 `ConvOperation3x`。文档字符串说明其用途为：“All parameters of a CUTLASS 3 convolution operation.”；类体负责实现这一职责。

### L908-L916 — Documentation/setup

```python
  908:   """All parameters of a CUTLASS 3 convolution operation.
  909: 
  910:   Unlike CUTLASS 2 convolutions, CUTLASS 3 convolutions do not
  911:   distinguish between 2-D and 3-D convolutions by kernel class name.
  912:   Instead, for CUTLASS 3 convolutions, the tensor layouts encode
  913:   whether the convolution is 2-D or 3-D.  Thus, this class deduces
  914:   the OperationKind (either Conv2d or Conv3d) from the layouts,
  915:   rather than taking it as a constructor parameter.
  916:   """
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L917-L960 — Function `__init__`

```python
  917:   def __init__(self,
  918:                conv_kind: ConvKind,
  919:                tile_description: TileDescription,
  920:                A: TensorDescription,
  921:                B: TensorDescription,
  922:                C: TensorDescription,
  923:                element_compute: Optional[DataType] = None,
  924:                D: Optional[TensorDescription] = None,
  925:                kernel_schedule: KernelScheduleType = KernelScheduleType.ScheduleAuto,
  926:                epilogue_schedule: EpilogueScheduleType = EpilogueScheduleType.ScheduleAuto,
  927:                tile_scheduler: TileSchedulerType = TileSchedulerType.Default,
  928:                log_indent_level: int = 1):
  929:     log_debug_line(f'ConvOperation3x::init: conv_kind: {conv_kind}', log_indent_level)
  930:     log_indent_level = log_indent_level + 1
  931: 
  932:     self.conv_kind = conv_kind
  933:     self.tile_description = tile_description
  934:     self.A = A
  935:     self.B = B
  936:     self.C = C
  937:     self.element_compute = C.element if element_compute is None else element_compute
  938:     self.kernel_schedule = kernel_schedule
  939:     self.epilogue_schedule = epilogue_schedule
  940: 
  941:     self.arch = tile_description.minimum_compute_capability
  942:     self.tile_scheduler = tile_scheduler
  943:     if D == None:
  944:       self.D = C
  945:     else:
  946:       self.D = D
  947: 
  948:     self.is_3x = True
  949:     self.group_mode = GroupMode.NoneGroup # CUTLASS 3 convolutions currently aren't grouped
  950: 
  951:     operation_kind = None
  952:     for layout in (A.layout, B.layout, C.layout):
  953:       assert(isinstance(layout, LayoutType))
  954:       new_operation_kind = convolution_tensor_layout_type_to_operation_kind(layout)
  955:       if operation_kind is None:
  956:         operation_kind = new_operation_kind
  957:       else: # CUTLASS 3 convolutions don't permit mixing 2-D and 3-D layouts.
  958:         assert(operation_kind == new_operation_kind)
  959:     assert(operation_kind is not None)
  960:     self.operation_kind = operation_kind
```
**EN:** Defines `__init__()`, which implements the init logic. Key helper calls include log_debug_line, isinstance, convolution_tensor_layout_type_to_operation_kind.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。 其中会调用的重要辅助函数包括 log_debug_line, isinstance, convolution_tensor_layout_type_to_operation_kind。

### L962-L963 — Function `__str__`

```python
  962:   def __str__(self):
  963:     return f"ConvOperation3x: operation_kind={self.operation_kind}, conv_kind={self.conv_kind}, tile_description={self.tile_description}"
```
**EN:** Defines `__str__()`, which implements the str logic.

**CN:** 定义 `__str__()`，用于实现 str 相关逻辑。

### L965-L971 — Function `is_complex`

```python
  965:   def is_complex(self):
  966:     complex_operators = [
  967:       MathOperation.multiply_add_complex,
  968:       MathOperation.multiply_add_complex_gaussian,
  969:       MathOperation.multiply_add_complex_fast_f32
  970:     ]
  971:     return self.tile_description.math_instruction.math_operation in complex_operators
```
**EN:** Defines `is_complex()`, a predicate that checks whether complex.

**CN:** 定义 `is_complex()`，用于判断是否满足 complex 这一条件。

### L973-L974 — Function `is_mixed_input`

```python
  973:   def is_mixed_input(self):
  974:     return self.A.element != self.B.element
```
**EN:** Defines `is_mixed_input()`, a predicate that checks whether mixed input.

**CN:** 定义 `is_mixed_input()`，用于判断是否满足 mixed input 这一条件。

### L976-L980 — Function `accumulator_type`

```python
  976:   def accumulator_type(self):
  977:     accum = self.tile_description.math_instruction.element_accumulator
  978:     if self.is_complex():
  979:       return get_complex_from_real(accum)
  980:     return accum
```
**EN:** Defines `accumulator_type()`, which implements the accumulator type logic. Key helper calls include is_complex, get_complex_from_real.

**CN:** 定义 `accumulator_type()`，用于实现 accumulator type 相关逻辑。 其中会调用的重要辅助函数包括 is_complex, get_complex_from_real。

### L982-L985 — Function `short_math_name`

```python
  982:   def short_math_name(self):
  983:     if self.tile_description.math_instruction.math_operation == MathOperation.multiply_add_complex_gaussian:
  984:       return "g%s" % ShortDataTypeNames[self.accumulator_type()]
  985:     return ShortDataTypeNames[self.accumulator_type()]
```
**EN:** Defines `short_math_name()`, which implements the short math name logic. Key helper calls include accumulator_type.

**CN:** 定义 `short_math_name()`，用于实现 short math name 相关逻辑。 其中会调用的重要辅助函数包括 accumulator_type。

### L987-L1017 — Function `core_name`

```python
  987:   def core_name(self):
  988:     ''' The basic operation kind is prefixed with a letter indicating the accumulation type. '''
  989: 
  990:     inst_shape = ''
  991:     inst_operation = ''
  992:     intermediate_type = ''
  993: 
  994:     math_operations_map = {
  995:       MathOperation.xor_popc: 'xor',
  996:       MathOperation.and_popc: 'and',
  997:     }
  998: 
  999:     tensor_ops = [
 1000:       OpcodeClass.TensorOp,
 1001:       OpcodeClass.WmmaTensorOp,
 1002:       OpcodeClass.SparseTensorOp,
 1003:       OpcodeClass.BlockScaledTensorOp, 
 1004:     ]
 1005: 
 1006:     is_tensor_op = self.tile_description.math_instruction.opcode_class in tensor_ops
 1007: 
 1008:     if is_tensor_op:
 1009: 
 1010:       math_op = self.tile_description.math_instruction.math_operation
 1011:       math_op_string = math_operations_map[math_op] if math_op in math_operations_map.keys() else ''
 1012: 
 1013:       if self.tile_description.math_instruction.element_a != self.A.element and \
 1014:         self.tile_description.math_instruction.element_a != self.tile_description.math_instruction.element_accumulator:
 1015:         intermediate_type = DataTypeNames[self.tile_description.math_instruction.element_a]
 1016: 
 1017:     return "%s%s%s" % (math_op_string, intermediate_type, ConvKindNames[self.conv_kind])
```
**EN:** Defines `core_name()`. The docstring says: "The basic operation kind is prefixed with a letter indicating the accumulation type."; this block implements that behavior. Key helper calls include keys.

**CN:** 定义 `core_name()`。文档字符串说明其用途为：“The basic operation kind is prefixed with a letter indicating the accumulation type.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 keys。

### L1019-L1032 — Function `extended_name`

```python
 1019:   def extended_name(self):
 1020:     '''Generates a string representing the MMA atom. Assumes accumulator type is C type.'''
 1021:     extended_name = "{core_name}_{element_a}{layout_a}_{element_b}{layout_b}_{element_acc}_{element_c}_{element_d}{layout_c}".format(
 1022:       element_a = DataTypeNames[self.A.element],
 1023:       layout_a = ShortLayoutTypeNames[self.A.layout],
 1024:       element_b = DataTypeNames[self.B.element],
 1025:       layout_b = ShortLayoutTypeNames[self.B.layout],
 1026:       element_acc = DataTypeNames[self.accumulator_type()],
 1027:       element_c = DataTypeNames[self.C.element],
 1028:       layout_c = ShortLayoutTypeNames[self.C.layout],
 1029:       element_d = DataTypeNames[self.D.element],
 1030:       core_name = self.core_name())
 1031: 
 1032:     return extended_name
```
**EN:** Defines `extended_name()`. The docstring says: "Generates a string representing the MMA atom."; this block implements that behavior. Key helper calls include format, core_name, accumulator_type.

**CN:** 定义 `extended_name()`。文档字符串说明其用途为：“Generates a string representing the MMA atom.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 format, core_name, accumulator_type。

### L1033-L1034 — Comments

```python
 1033: 
 1034:   # Generates a short string representing underlying kernel schedule type
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1035-L1036 — Function `kernel_schedule_name`

```python
 1035:   def kernel_schedule_name(self):
 1036:     return KernelScheduleSuffixes[self.kernel_schedule]
```
**EN:** Defines `kernel_schedule_name()`, which implements the kernel schedule name logic.

**CN:** 定义 `kernel_schedule_name()`，用于实现 kernel schedule name 相关逻辑。

### L1037-L1038 — Comments

```python
 1037: 
 1038:   # Generates a short string representing underlying epilogue schedule type
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1039-L1040 — Function `epilogue_schedule_name`

```python
 1039:   def epilogue_schedule_name(self):
 1040:     return EpilogueScheduleSuffixes[self.epilogue_schedule]
```
**EN:** Defines `epilogue_schedule_name()`, which implements the epilogue schedule name logic.

**CN:** 定义 `epilogue_schedule_name()`，用于实现 epilogue schedule name 相关逻辑。

### L1041-L1042 — Comments

```python
 1041:   
 1042:   # Generate a short string representing the operation class
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1043-L1044 — Function `opcode_class_name`

```python
 1043:   def opcode_class_name(self):
 1044:     return OpcodeClassNames[self.tile_description.math_instruction.opcode_class]
```
**EN:** Defines `opcode_class_name()`, which implements the opcode class name logic.

**CN:** 定义 `opcode_class_name()`，用于实现 opcode class name 相关逻辑。

### L1045-L1046 — Comments

```python
 1045: 
 1046:   # Generates the full kernel function name
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1047-L1060 — Function `configuration_name`

```python
 1047:   def configuration_name(self):
 1048:     ''' The full function name indicates architecture, extended name, tile size, and layout. '''
 1049:     kernel_name_template = "cutlass3x_sm{ar}_{op}_{ex}{ct}{cs}_{l}_align{al}{t}{k}{e}"
 1050:     return kernel_name_template.format(
 1051:         ar = self.arch,
 1052:         op = self.opcode_class_name(),
 1053:         ex = self.extended_name(),
 1054:         ct = '_' + 'x'.join([str(i) for i in self.tile_description.tile_shape]) if self.tile_description.tile_shape[0] > 0 else "",
 1055:         cs = '_' + 'x'.join([str(i) for i in self.tile_description.cluster_shape]),
 1056:         l = self.tile_description.stages,
 1057:         al = str(max(self.A.alignment, self.B.alignment)),
 1058:         t = TileSchedulerSuffixes[self.tile_scheduler],
 1059:         k = self.kernel_schedule_name(),
 1060:         e = self.epilogue_schedule_name())
```
**EN:** Defines `configuration_name()`. The docstring says: "The full function name indicates architecture, extended name, tile size, and layout."; this block implements that behavior. Key helper calls include format, opcode_class_name, extended_name, str, kernel_schedule_name, epilogue_schedule_name.

**CN:** 定义 `configuration_name()`。文档字符串说明其用途为：“The full function name indicates architecture, extended name, tile size, and layout.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 format, opcode_class_name, extended_name, str, kernel_schedule_name, epilogue_schedule_name。

### L1062-L1063 — Function `procedural_name`

```python
 1062:   def procedural_name(self):
 1063:     return self.configuration_name()
```
**EN:** Defines `procedural_name()`, which implements the procedural name logic. Key helper calls include configuration_name.

**CN:** 定义 `procedural_name()`，用于实现 procedural name 相关逻辑。 其中会调用的重要辅助函数包括 configuration_name。

### L1065-L1071 — Function `convolution_tensor_layout_type_to_operation_kind`

```python
 1065: def convolution_tensor_layout_type_to_operation_kind(layout: LayoutType) -> OperationKind:
 1066:   if layout == LayoutType.TensorNHWC or layout == LayoutType.TensorKCSR:
 1067:     return OperationKind.Conv2d
 1068:   elif layout == LayoutType.TensorNDHWC or layout == LayoutType.TensorKCSRT:
 1069:     return OperationKind.Conv3d
 1070:   else:
 1071:     raise RuntimeError(f'LayoutType {layout} does not have a corresponding OperationKind')
```
**EN:** Defines `convolution_tensor_layout_type_to_operation_kind()`, which implements the convolution tensor layout type to operation kind logic. Key helper calls include RuntimeError.

**CN:** 定义 `convolution_tensor_layout_type_to_operation_kind()`，用于实现 convolution tensor layout type to operation kind 相关逻辑。 其中会调用的重要辅助函数包括 RuntimeError。

### L1073-L1240 — Function `CreateConvOperator3x`

```python
 1073: def CreateConvOperator3x(manifest: Manifest,
 1074:                          dims_and_alignments: Sequence[Tuple[Tuple[int, int], Tuple[int, int], Tuple[int, int]]],
 1075:                          tile_descriptions: Sequence[Sequence[TileDescription]],
 1076:                          data_types,
 1077:                          schedule_pairs: Sequence[Tuple[KernelScheduleType, KernelScheduleType]] = \
 1078:                            [(KernelScheduleType.ScheduleAuto, EpilogueScheduleType.ScheduleAuto)],
 1079:                          complex_transforms: Optional[Sequence[ComplexTransform]] = None,
 1080:                          tile_schedulers: Sequence[TileSchedulerType] = [TileSchedulerType.Default],
 1081:                          conv_kind: ConvKind = ConvKind.Fprop,
 1082:                          log_indent_level: int = 1):
 1083:   """
 1084:   Create zero or more CUTLASS 3 two-dimensional convolution operators.
 1085: 
 1086:   Create a CUTLASS 3 two-dimensional convolution operator
 1087:   for all feasible combinations of the input parameters.
 1088:   Add the operators to the manifest.
 1089: 
 1090:   dims_and_alignments: 3-level list.  Each outer list term is a list [A, B, C].
 1091:     Each inner list (A, B, or C) has the form [num_spatial_dimensions, alignment].
 1092:     Both are integers; the first is the number of spatial dimensions
 1093:     (currently, only 2 or 3 are supported), and the second is the byte alignment.
 1094:     We deduce the operation_kind (either OperationKind.Conv2d or OperationKind.Conv3d)
 1095:     from num_spatial_dimensions.
 1096: 
 1097:   This function doesn't take layouts, unlike the GEMM functions.
 1098:   CUTLASS 3 convolutions currently support three input layouts:
 1099: 
 1100:   * TensorNWC for 1-D convolutions,
 1101:   * TensorNHWC for 2-D convolutions, and
 1102:   * TensorNDHWC for 3-D convolutions.
 1103: 
 1104:   Output (C and D) layouts are the same as input layouts,
 1105:   except for Wgrad convolutions, where the layouts are
 1106: 
 1107:   * TensorKCS for 1-D convolutions,
 1108:   * TensorKCSR for 2-D convolutions, and
 1109:   * TensorKCSRT for 3-D convolutions.
 1110: 
 1111:   The output layouts are completely constrained by the input layouts
 1112:   and the convolution kind.
 1113: 
 1114:   tile_descriptions: 2-level list.
 1115:     Outer level has one list per math instruction.
 1116:     Inner level has one TileDescription for each cluster shape.
 1117: 
 1118:   data_types: Either a single data_type dictionary, or a list of them.
 1119:     Keys: 'a_type', 'b_type', 'c_type', 'd_type', 'acc_type', 'epi_type'
 1120: 
 1121:   complex_transforms: Optional list of pairs.
 1122:     First element of each pair is the complex transform for A, and
 1123:     second element of each pair is the complex transform for B.
 1124: 
 1125:   schedule_pairs: [(kernel_schedule, epilogue_schedule), ...]
 1126: 
 1127:   conv_kind: Convolution kind (Fprop, Dgrad, or Wgrad).
 1128:   """
 1129:   log_debug_line('CreateConvOperator3x', log_indent_level)
 1130:   log_indent_level = log_indent_level + 1
 1131:   log_debug_line(f'conv_kind: {conv_kind}', log_indent_level)
 1132: 
 1133:   for triple in dims_and_alignments:
 1134:     assert(isinstance(triple, tuple) or isinstance(triple, list))
 1135:     assert(len(triple) == 3)
 1136: 
 1137:     spatial_dimensionality = None # to be determined by loop below
 1138: 
 1139:     for entry in triple: # [A, B, C]
 1140:       assert(len(entry) == 2)
 1141:       [dim, alignment] = entry
 1142:       assert(type(dim) is int)
 1143:       assert(dim == 2 or dim == 3)
 1144:       assert(type(alignment) is int)
 1145:       assert(alignment > 0)
 1146:       if spatial_dimensionality is None:
 1147:         spatial_dimensionality = dim
 1148:       else:
 1149:         # A, B, and C need to have the same spatial dimensionality
 1150:         assert(spatial_dimensionality == dim)
 1151: 
 1152:   def input_and_output_layouts(spatial_dim: int, kind: ConvKind) -> Tuple[LayoutType, LayoutType]:
 1153:     if spatial_dim == 1:
 1154:       input_layout = LayoutType.TensorNWC
 1155:       if kind == ConvKind.Wgrad:
 1156:         output_layout = LayoutType.TensorKCS
 1157:       else:
 1158:         output_layout = input_layout
 1159:     elif spatial_dim == 2:
 1160:       input_layout = LayoutType.TensorNHWC
 1161:       if kind == ConvKind.Wgrad:
 1162:         output_layout = LayoutType.TensorKCSR
 1163:       else:
 1164:         output_layout = input_layout
 1165:     elif spatial_dim == 3:
 1166:       input_layout = LayoutType.TensorNDHWC
 1167:       if kind == ConvKind.Wgrad:
 1168:         output_layout = LayoutType.TensorKCSRT
 1169:       else:
 1170:         output_layout = input_layout
 1171:     else:
 1172:       assert(False)
 1173:     return (input_layout, output_layout)
 1174: 
 1175:   def dims_to_layouts(A_B_C: Tuple[Tuple[int, int], Tuple[int, int], Tuple[int, int]]) -> \
 1176:       Tuple[Tuple[LayoutType, int], Tuple[LayoutType, int], Tuple[LayoutType, int]]:
 1177:     [A, B, C] = A_B_C
 1178:     [spatial_dim, alignment] = A
 1179:     [input_layout, output_layout] = input_and_output_layouts(spatial_dim, conv_kind)
 1180:     return ((input_layout, A[1]),
 1181:             (input_layout, B[1]),
 1182:             (output_layout, C[1]))
 1183: 
 1184:   # layouts: list of triples (A, B, C).
 1185:   # Each of A, B, and C has the form [layout, alignment].
 1186:   layouts = [dims_to_layouts(A_B_C) for A_B_C in dims_and_alignments]
 1187: 
 1188:   if type(data_types) is dict:
 1189:     data_types = [data_types]
 1190: 
 1191:   for s in schedule_pairs:
 1192:     assert(len(s) == 2)
 1193: 
 1194:   if complex_transforms is None:
 1195:     complex_transforms = [(ComplexTransform.none, ComplexTransform.none)]
 1196: 
 1197:   # product produces a one-pass generator, so the loop must call it anew each time.
 1198:   def make_combinations():
 1199:     return product(
 1200:       layouts,
 1201:       tile_descriptions,
 1202:       data_types,
 1203:       complex_transforms,
 1204:       schedule_pairs,
 1205:       tile_schedulers
 1206:     )
 1207: 
 1208:   operations = []
 1209:   for layout_triple, tile_description, data_type, complex_transform_pair, schedule_pair, tile_scheduler in make_combinations():
 1210:     A_layout, A_alignment = layout_triple[0]
 1211:     A_xform = complex_transform_pair[0]
 1212:     B_layout, B_alignment = layout_triple[1]
 1213:     B_xform = complex_transform_pair[1]
 1214:     C_layout, C_alignment = layout_triple[2]
 1215:     D_layout = C_layout
 1216:     D_alignment = C_alignment
 1217: 
 1218:     A = TensorDescription(data_type["a_type"], A_layout, A_alignment, A_xform)
 1219:     B = TensorDescription(data_type["b_type"], B_layout, B_alignment, B_xform)
 1220:     C = TensorDescription(data_type["c_type"], C_layout, C_alignment)
 1221:     D = TensorDescription(data_type["d_type"], D_layout, D_alignment)
 1222:     element_compute = data_type.get("epi_type", data_type["acc_type"])
 1223:     kernel_schedule, epilogue_schedule = schedule_pair
 1224: 
 1225:     operation = ConvOperation3x(conv_kind=conv_kind,
 1226:                                 tile_description=tile_description,
 1227:                                 A=A,
 1228:                                 B=B,
 1229:                                 C=C,
 1230:                                 element_compute=element_compute,
 1231:                                 D=D,
 1232:                                 kernel_schedule=kernel_schedule,
 1233:                                 epilogue_schedule=epilogue_schedule,
 1234:                                 tile_scheduler=tile_scheduler,
 1235:                                 log_indent_level=log_indent_level)
 1236:     log_debug_line(f'Created ConvOperation3x: {str(operation)}', log_indent_level)
 1237:     manifest.append(operation)
 1238:     operations.append(operation)
 1239: 
 1240:   return operations
```
**EN:** Defines `CreateConvOperator3x()`. The docstring says: "Create zero or more CUTLASS 3 two-dimensional convolution operators."; this block implements that behavior. Key helper calls include log_debug_line, make_combinations, input_and_output_layouts, dims_to_layouts, type, product.

**CN:** 定义 `CreateConvOperator3x()`。文档字符串说明其用途为：“Create zero or more CUTLASS 3 two-dimensional convolution operators.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 log_debug_line, make_combinations, input_and_output_layouts, dims_to_layouts, type, product。

### L1241-L1245 — Comments

```python
 1241: 
 1242: ###################################################################################################
 1243: ###################################################################################################
 1244: 
 1245: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1246-L1294 — Function `GenerateSM50_Simt`

```python
 1246: def GenerateSM50_Simt(manifest, cuda_version):
 1247:   layouts = [
 1248:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1249:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1250:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1251:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1252:   ]
 1253: 
 1254:   math_instructions = [
 1255:     MathInstruction(                                  \
 1256:       [1, 1, 1],                                      \
 1257:       DataType.f32, DataType.f32, DataType.f32,       \
 1258:       OpcodeClass.Simt,                               \
 1259:       MathOperation.multiply_add),
 1260:     MathInstruction(                                  \
 1261:       [1, 1, 1],                                      \
 1262:       DataType.f64, DataType.f64, DataType.f64,       \
 1263:       OpcodeClass.Simt,                               \
 1264:       MathOperation.multiply_add),
 1265:   ]
 1266: 
 1267:   min_cc = 50
 1268:   max_cc = 1024
 1269: 
 1270:   alignment_constraints = [1,]
 1271: 
 1272:   for math_inst in math_instructions:
 1273:     tile_descriptions = [
 1274:       TileDescription([128, 128, 8], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1275:       TileDescription([128,  64, 8], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1276:       TileDescription([ 64, 128, 8], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1277:       TileDescription([ 64,  64, 8], 2, [2, 1, 1], math_inst, min_cc, max_cc),
 1278:       TileDescription([128,  32, 8], 2, [2, 1, 1], math_inst, min_cc, max_cc),
 1279:       TileDescription([ 32, 128, 8], 2, [1, 2, 1], math_inst, min_cc, max_cc),
 1280:     ]
 1281: 
 1282:     data_type = [
 1283:       math_inst.element_a,
 1284:       math_inst.element_b,
 1285:       math_inst.element_accumulator,
 1286:       math_inst.element_accumulator,
 1287:     ]
 1288: 
 1289:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 1290:       data_type, alignment_constraints)
 1291: 
 1292:     if math_inst.element_a == DataType.f32:
 1293:       conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 1294:       CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM50_Simt()`, which generates sm50 simt. Key helper calls include MathInstruction, CreateGemmOperator, TileDescription, CreateConv2dOperator.

**CN:** 定义 `GenerateSM50_Simt()`，用于生成 sm50 simt。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, TileDescription, CreateConv2dOperator。

### L1295-L1297 — Comments

```python
 1295: #
 1296: 
 1297: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1298-L1341 — Function `GenerateSM50_Simt_complex`

```python
 1298: def GenerateSM50_Simt_complex(manifest, cuda_version):
 1299:   layouts = [
 1300:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1301:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1302:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1303:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1304:   ]
 1305: 
 1306:   math_instructions = [
 1307:     MathInstruction(                                  \
 1308:       [1, 1, 1],                                      \
 1309:       DataType.f32, DataType.f32, DataType.f32,       \
 1310:       OpcodeClass.Simt,                               \
 1311:       MathOperation.multiply_add_complex),
 1312:   ]
 1313: 
 1314:   min_cc = 50
 1315:   max_cc = 1024
 1316: 
 1317:   alignment_constraints = [1,]
 1318: 
 1319:   for math_inst in math_instructions:
 1320:     tile_descriptions = [
 1321:       TileDescription([128,  64, 8], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1322:       TileDescription([ 64, 128, 8], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1323:       TileDescription([ 64,  64, 8], 2, [2, 1, 1], math_inst, min_cc, max_cc),
 1324:       TileDescription([128,  32, 8], 2, [2, 1, 1], math_inst, min_cc, max_cc),
 1325:       TileDescription([ 32, 128, 8], 2, [1, 2, 1], math_inst, min_cc, max_cc),
 1326:       TileDescription([128, 128, 8], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1327:     ]
 1328: 
 1329:     data_type = [
 1330:       DataType.cf32,
 1331:       DataType.cf32,
 1332:       DataType.cf32,
 1333:       DataType.cf32,
 1334:     ]
 1335: 
 1336: 
 1337:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 1338:       data_type, alignment_constraints)
 1339: 
 1340:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 1341:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM50_Simt_complex()`, which generates sm50 simt complex. Key helper calls include MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription.

**CN:** 定义 `GenerateSM50_Simt_complex()`，用于生成 sm50 simt complex。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription。

### L1342-L1344 — Comments

```python
 1342: #
 1343: 
 1344: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1345-L1347 — Function `GenerateSM50`

```python
 1345: def GenerateSM50(manifest, cuda_version):
 1346:   GenerateSM50_Simt(manifest, cuda_version)
 1347:   GenerateSM50_Simt_complex(manifest, cuda_version)
```
**EN:** Defines `GenerateSM50()`, which generates sm50. Key helper calls include GenerateSM50_Simt, GenerateSM50_Simt_complex.

**CN:** 定义 `GenerateSM50()`，用于生成 sm50。 其中会调用的重要辅助函数包括 GenerateSM50_Simt, GenerateSM50_Simt_complex。

### L1348-L1352 — Comments

```python
 1348: 
 1349: ###################################################################################################
 1350: ###################################################################################################
 1351: 
 1352: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1353-L1394 — Function `GenerateSM60_Simt`

```python
 1353: def GenerateSM60_Simt(manifest, cuda_version):
 1354:   layouts = [
 1355:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1356:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1357:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1358:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1359:   ]
 1360: 
 1361:   math_instructions = [
 1362:     MathInstruction(                                  \
 1363:       [1, 1, 1],                                      \
 1364:       DataType.f16, DataType.f16, DataType.f16,       \
 1365:       OpcodeClass.Simt,                               \
 1366:       MathOperation.multiply_add),
 1367:   ]
 1368: 
 1369:   min_cc = 60
 1370:   max_cc = 1024
 1371: 
 1372:   alignment_constraints = [1,]
 1373: 
 1374:   for math_inst in math_instructions:
 1375:     tile_descriptions = [
 1376:       TileDescription([256, 128, 8], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1377:       TileDescription([128, 256, 8], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1378:       TileDescription([128, 128, 8], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1379:       TileDescription([128,  64, 8], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1380:       TileDescription([ 64, 128, 8], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1381:       TileDescription([ 64,  64, 8], 2, [2, 1, 1], math_inst, min_cc, max_cc),
 1382:       TileDescription([128,  32, 8], 2, [2, 1, 1], math_inst, min_cc, max_cc),
 1383:       TileDescription([ 32, 128, 8], 2, [1, 2, 1], math_inst, min_cc, max_cc),
 1384:     ]
 1385: 
 1386:     data_type = [
 1387:       math_inst.element_a,
 1388:       math_inst.element_b,
 1389:       math_inst.element_accumulator,
 1390:       math_inst.element_accumulator,
 1391:     ]
 1392: 
 1393:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 1394:       data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM60_Simt()`, which generates sm60 simt. Key helper calls include MathInstruction, CreateGemmOperator, TileDescription.

**CN:** 定义 `GenerateSM60_Simt()`，用于生成 sm60 simt。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, TileDescription。

### L1395-L1395 — Comments

```python
 1395: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1396-L1466 — Function `GenerateSM60_Simt_DepthwiseConv2d`

```python
 1396: def GenerateSM60_Simt_DepthwiseConv2d(manifest, cuda_version):
 1397: 
 1398:   math_instructions = [
 1399:     MathInstruction(                                  \
 1400:       [1, 1, 1],                                      \
 1401:       DataType.f16, DataType.f16, DataType.f16,       \
 1402:       OpcodeClass.Simt,                               \
 1403:       MathOperation.multiply_add),
 1404:   ]
 1405: 
 1406:   min_cc = 60
 1407:   max_cc = 1024
 1408: 
 1409:   alignment_constraints = [8,]
 1410: 
 1411:   filter_3x3 = [3, 3]
 1412:   filter_5x5 = [5, 5]
 1413: 
 1414:   # [stride_h, stride_w]
 1415:   # [-1, -1] means all stride size.
 1416:   strides = [[-1,-1], [1, 1], [2, 2]]
 1417:   # [dilation_h, dilation_w]
 1418:   # [-1, -1] means all dilation size.
 1419:   dilations = [[-1,-1], [1, 1], [2, 2]]
 1420: 
 1421:   #groups per thread block
 1422:   g16 = 16
 1423:   g32 = 32
 1424:   g64 = 64
 1425: 
 1426:   #output shape per thread block
 1427:   npq_1x4x4 = [1, 4, 4]
 1428:   npq_1x8x8 = [1, 8, 8]
 1429:   npq_1x10x10 = [1, 10, 10]
 1430: 
 1431:   tile_descriptions = []
 1432:   for math_inst in math_instructions:
 1433:     for stride, dilation in product(strides, dilations):
 1434:       tile_descriptions.extend([
 1435:         # filter3x3               ThreadBlock_output, filter, stage, warp
 1436:         Direct2dConvFixedStrideDilationTileDescription(npq_1x8x8+[g32], filter_3x3, 3, stride, dilation,[4, 1, 1],math_inst, min_cc, max_cc),
 1437:         Direct2dConvFixedStrideDilationTileDescription(npq_1x8x8+[g64], filter_3x3, 3, stride, dilation,[4, 1, 1],math_inst, min_cc, max_cc),
 1438:         Direct2dConvFixedStrideDilationTileDescription(npq_1x8x8+[g16], filter_3x3, 3, stride, dilation,[4, 1, 1],math_inst, min_cc, max_cc),
 1439: 
 1440:         Direct2dConvFixedStrideDilationTileDescription(npq_1x10x10+[g64], filter_3x3, 2, stride, dilation,[4, 1, 1],math_inst, min_cc, max_cc),
 1441: 
 1442:         Direct2dConvFixedStrideDilationTileDescription(npq_1x4x4+[g32], filter_3x3, 4, stride, dilation, [4, 1, 1],  math_inst, min_cc, max_cc),
 1443:         Direct2dConvFixedStrideDilationTileDescription(npq_1x4x4+[g64], filter_3x3, 4,  stride, dilation,[4, 1, 1], math_inst, min_cc, max_cc),
 1444:         Direct2dConvFixedStrideDilationTileDescription(npq_1x4x4+[g16], filter_3x3, 4, stride, dilation, [4, 1, 1],  math_inst, min_cc, max_cc),
 1445: 
 1446:         # filter5x5               ThreadBlock_output, filter, stage, warp
 1447:         Direct2dConvFixedStrideDilationTileDescription(npq_1x8x8+[g32], filter_5x5, 3, stride, dilation,[4, 1, 1],math_inst, min_cc, max_cc),
 1448:         Direct2dConvFixedStrideDilationTileDescription(npq_1x8x8+[g64], filter_5x5, 3, stride, dilation,[4, 1, 1],math_inst, min_cc, max_cc),
 1449:         Direct2dConvFixedStrideDilationTileDescription(npq_1x8x8+[g16], filter_5x5, 3, stride, dilation,[4, 1, 1],math_inst, min_cc, max_cc),
 1450: 
 1451:         Direct2dConvFixedStrideDilationTileDescription(npq_1x10x10+[g64], filter_5x5, 2, stride, dilation,[4, 1, 1],math_inst, min_cc, max_cc),
 1452: 
 1453:         Direct2dConvFixedStrideDilationTileDescription(npq_1x4x4+[g32], filter_5x5, 4, stride, dilation,[4, 1, 1],math_inst, min_cc, max_cc),
 1454:         Direct2dConvFixedStrideDilationTileDescription(npq_1x4x4+[g64], filter_5x5, 4, stride, dilation,[4, 1, 1],math_inst, min_cc, max_cc),
 1455:         Direct2dConvFixedStrideDilationTileDescription(npq_1x4x4+[g16], filter_5x5, 4, stride, dilation,[4, 1, 1],math_inst, min_cc, max_cc)
 1456:       ])
 1457: 
 1458:     data_type = [
 1459:       math_inst.element_a,
 1460:       math_inst.element_b,
 1461:       math_inst.element_accumulator,
 1462:       math_inst.element_accumulator,
 1463:     ]
 1464: 
 1465:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 1466:     CreateDepthwiseConv2dOperator(manifest, conv_layout, tile_descriptions, data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM60_Simt_DepthwiseConv2d()`, which generates sm60 simt depthwise conv2d. Key helper calls include MathInstruction, product, CreateDepthwiseConv2dOperator, extend, Direct2dConvFixedStrideDilationTileDescription.

**CN:** 定义 `GenerateSM60_Simt_DepthwiseConv2d()`，用于生成 sm60 simt depthwise conv2d。 其中会调用的重要辅助函数包括 MathInstruction, product, CreateDepthwiseConv2dOperator, extend, Direct2dConvFixedStrideDilationTileDescription。

### L1467-L1469 — Comments

```python
 1467: #
 1468: 
 1469: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1470-L1472 — Function `GenerateSM60`

```python
 1470: def GenerateSM60(manifest, cuda_version):
 1471:   GenerateSM60_Simt(manifest, cuda_version)
 1472:   GenerateSM60_Simt_DepthwiseConv2d(manifest, cuda_version)
```
**EN:** Defines `GenerateSM60()`, which generates sm60. Key helper calls include GenerateSM60_Simt, GenerateSM60_Simt_DepthwiseConv2d.

**CN:** 定义 `GenerateSM60()`，用于生成 sm60。 其中会调用的重要辅助函数包括 GenerateSM60_Simt, GenerateSM60_Simt_DepthwiseConv2d。

### L1473-L1477 — Comments

```python
 1473: 
 1474: ###################################################################################################
 1475: ###################################################################################################
 1476: 
 1477: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1478-L1526 — Function `GenerateSM61_Simt`

```python
 1478: def GenerateSM61_Simt(manifest, cuda_version):
 1479:   layouts = [
 1480:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1481:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1482:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1483:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1484:   ]
 1485: 
 1486:   math_instructions = [
 1487:     MathInstruction(                                  \
 1488:       [1, 1, 4],                                      \
 1489:       DataType.s8, DataType.s8, DataType.s32,         \
 1490:       OpcodeClass.Simt,                               \
 1491:       MathOperation.multiply_add),
 1492:   ]
 1493: 
 1494:   min_cc = 61
 1495:   max_cc = 1024
 1496: 
 1497:   alignment_constraints = [1,]
 1498: 
 1499:   for math_inst in math_instructions:
 1500:     tile_descriptions = [
 1501:       TileDescription([128, 128, 32], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1502:       TileDescription([128,  64, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1503:       TileDescription([ 64, 128, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1504:       TileDescription([ 64,  64, 32], 2, [2, 1, 1], math_inst, min_cc, max_cc),
 1505:       TileDescription([128,  32, 32], 2, [2, 1, 1], math_inst, min_cc, max_cc),
 1506:       TileDescription([ 32, 128, 32], 2, [1, 2, 1], math_inst, min_cc, max_cc),
 1507:     ]
 1508: 
 1509:     data_type = [
 1510:       math_inst.element_a,
 1511:       math_inst.element_b,
 1512:       math_inst.element_accumulator,
 1513:       math_inst.element_accumulator,
 1514:     ]
 1515:     data_type_mixed = [
 1516:       math_inst.element_a,
 1517:       math_inst.element_b,
 1518:       math_inst.element_a,
 1519:       math_inst.element_accumulator,
 1520:     ]
 1521: 
 1522:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 1523:       data_type, alignment_constraints)
 1524: 
 1525:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 1526:       data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombinationClamp)
```
**EN:** Defines `GenerateSM61_Simt()`, which generates sm61 simt. Key helper calls include MathInstruction, CreateGemmOperator, TileDescription.

**CN:** 定义 `GenerateSM61_Simt()`，用于生成 sm61 simt。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, TileDescription。

### L1527-L1529 — Comments

```python
 1527: #
 1528: 
 1529: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1530-L1531 — Function `GenerateSM61`

```python
 1530: def GenerateSM61(manifest, cuda_version):
 1531:   GenerateSM61_Simt(manifest, cuda_version)
```
**EN:** Defines `GenerateSM61()`, which generates sm61. Key helper calls include GenerateSM61_Simt.

**CN:** 定义 `GenerateSM61()`，用于生成 sm61。 其中会调用的重要辅助函数包括 GenerateSM61_Simt。

### L1532-L1536 — Comments

```python
 1532: 
 1533: ###################################################################################################
 1534: ###################################################################################################
 1535: 
 1536: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1537-L1605 — Function `GenerateSM70_TensorOp_884`

```python
 1537: def GenerateSM70_TensorOp_884(manifest, cuda_version):
 1538: 
 1539:   if not CudaToolkitVersionSatisfies(cuda_version, 10, 1):
 1540:     return
 1541: 
 1542:   layouts = [
 1543:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1544:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1545:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1546:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1547:   ]
 1548: 
 1549:   math_instructions = [
 1550:     MathInstruction(                                  \
 1551:       [8, 8, 4],                                      \
 1552:       DataType.f16, DataType.f16, DataType.f32,       \
 1553:       OpcodeClass.TensorOp,                           \
 1554:       MathOperation.multiply_add),
 1555:     MathInstruction(                                  \
 1556:       [8, 8, 4],                                      \
 1557:       DataType.f16, DataType.f16, DataType.f16,       \
 1558:       OpcodeClass.TensorOp,                           \
 1559:       MathOperation.multiply_add),
 1560:   ]
 1561: 
 1562:   min_cc = 70
 1563:   max_cc = 75
 1564: 
 1565:   alignment_constraints = [8, 4, 2, 1]
 1566: 
 1567:   for math_inst in math_instructions:
 1568:     tile_descriptions = [
 1569:       TileDescription([256, 128, 32], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1570:       TileDescription([128, 256, 32], 2, [2, 4, 1], math_inst, min_cc, max_cc),
 1571:       TileDescription([128, 128, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1572:       TileDescription([256,  64, 32], 2, [4, 1, 1], math_inst, min_cc, max_cc),
 1573:       TileDescription([ 64, 256, 32], 2, [1, 4, 1], math_inst, min_cc, max_cc),
 1574:       TileDescription([ 64, 128, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1575:       TileDescription([128,  64, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1576:       TileDescription([ 64,  64, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1577:     ]
 1578: 
 1579:     data_type = [
 1580:       math_inst.element_a,
 1581:       math_inst.element_b,
 1582:       math_inst.element_accumulator,
 1583:       math_inst.element_accumulator,
 1584:     ]
 1585: 
 1586:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 1587:       data_type, alignment_constraints)
 1588: 
 1589:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 1590:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type, alignment_constraints)
 1591: 
 1592:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 1593:     if math_inst.element_a != math_inst.element_accumulator:
 1594: 
 1595:       data_type_mixed = [
 1596:         math_inst.element_a,
 1597:         math_inst.element_b,
 1598:         math_inst.element_a,
 1599:         math_inst.element_accumulator,
 1600:       ]
 1601: 
 1602:       CreateGemmOperator(manifest, layouts, tile_descriptions, \
 1603:         data_type_mixed, alignment_constraints)
 1604: 
 1605:       CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type_mixed, alignment_constraints)
```
**EN:** Defines `GenerateSM70_TensorOp_884()`, which generates sm70 tensor op 884. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription.

**CN:** 定义 `GenerateSM70_TensorOp_884()`，用于生成 sm70 tensor op 884。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription。

### L1606-L1607 — Comments

```python
 1606: 
 1607: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1608-L1671 — Function `GenerateSM70_PlanarComplexTensorOp_884`

```python
 1608: def GenerateSM70_PlanarComplexTensorOp_884(manifest, cuda_version):
 1609: 
 1610:   if not CudaToolkitVersionSatisfies(cuda_version, 10, 1):
 1611:     return
 1612: 
 1613:   layouts = [
 1614:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1615:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1616:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1617:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1618:   ]
 1619: 
 1620:   complex_transforms = [
 1621:     (ComplexTransform.none, ComplexTransform.none),
 1622:     (ComplexTransform.conj, ComplexTransform.none),
 1623:     (ComplexTransform.none, ComplexTransform.conj),
 1624:     (ComplexTransform.conj, ComplexTransform.conj)
 1625:   ]
 1626: 
 1627:   math_instructions = [
 1628:     MathInstruction(                                  \
 1629:       [8, 8, 4],                                      \
 1630:       DataType.f16, DataType.f16, DataType.f32,       \
 1631:       OpcodeClass.TensorOp,                           \
 1632:       MathOperation.multiply_add),
 1633:     MathInstruction(                                  \
 1634:       [8, 8, 4],                                      \
 1635:       DataType.f16, DataType.f16, DataType.f16,       \
 1636:       OpcodeClass.TensorOp,                           \
 1637:       MathOperation.multiply_add),
 1638:   ]
 1639: 
 1640:   min_cc = 70
 1641:   max_cc = 75
 1642: 
 1643:   alignment_constraints = [8, 2, 1]
 1644: 
 1645:   for math_inst in math_instructions:
 1646:     tile_descriptions = [
 1647:       TileDescription([ 64,  64, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1648:     ]
 1649: 
 1650:     data_type = [
 1651:       math_inst.element_a,
 1652:       math_inst.element_b,
 1653:       math_inst.element_accumulator,
 1654:       math_inst.element_accumulator,
 1655:     ]
 1656: 
 1657:     CreateGemmPlanarComplexOperator(manifest, layouts, tile_descriptions, \
 1658:       data_type, alignment_constraints, complex_transforms)
 1659: 
 1660:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 1661:     if math_inst.element_a != math_inst.element_accumulator:
 1662: 
 1663:       data_type_mixed = [
 1664:         math_inst.element_a,
 1665:         math_inst.element_b,
 1666:         math_inst.element_a,
 1667:         math_inst.element_accumulator,
 1668:       ]
 1669: 
 1670:       CreateGemmPlanarComplexOperator(manifest, layouts, tile_descriptions, \
 1671:         data_type_mixed, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM70_PlanarComplexTensorOp_884()`, which generates sm70 planar complex tensor op 884. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmPlanarComplexOperator, TileDescription.

**CN:** 定义 `GenerateSM70_PlanarComplexTensorOp_884()`，用于生成 sm70 planar complex tensor op 884。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmPlanarComplexOperator, TileDescription。

### L1672-L1674 — Comments

```python
 1672: 
 1673: 
 1674: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1675-L1731 — Function `GenerateSM70_WmmaTensorOp_161616`

```python
 1675: def GenerateSM70_WmmaTensorOp_161616(manifest, cuda_version):
 1676: 
 1677:   layouts = [
 1678:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1679:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1680:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1681:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1682:   ]
 1683: 
 1684:   math_instructions = [
 1685:     MathInstruction(                                  \
 1686:       [16, 16, 16],                                   \
 1687:       DataType.f16, DataType.f16, DataType.f32,       \
 1688:       OpcodeClass.WmmaTensorOp,                       \
 1689:       MathOperation.multiply_add),
 1690:     MathInstruction(                                  \
 1691:       [16, 16, 16],                                   \
 1692:       DataType.f16, DataType.f16, DataType.f16,       \
 1693:       OpcodeClass.WmmaTensorOp,                       \
 1694:       MathOperation.multiply_add),
 1695:   ]
 1696: 
 1697:   min_cc = 70
 1698:   max_cc = 1024
 1699: 
 1700:   alignment_constraints = [8,]
 1701: 
 1702:   for math_inst in math_instructions:
 1703:     tile_descriptions = [
 1704:       TileDescription([128, 128, 32], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1705:       TileDescription([ 64, 128, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1706:       TileDescription([128,  64, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1707:       TileDescription([ 64,  64, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1708:     ]
 1709: 
 1710:     data_type = [
 1711:       math_inst.element_a,
 1712:       math_inst.element_b,
 1713:       math_inst.element_accumulator,
 1714:       math_inst.element_accumulator,
 1715:     ]
 1716: 
 1717:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 1718:       data_type, alignment_constraints)
 1719: 
 1720:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 1721:     if math_inst.element_a != math_inst.element_accumulator:
 1722: 
 1723:       data_type_mixed = [
 1724:         math_inst.element_a,
 1725:         math_inst.element_b,
 1726:         math_inst.element_a,
 1727:         math_inst.element_accumulator,
 1728:       ]
 1729: 
 1730:       CreateGemmOperator(manifest, layouts, tile_descriptions, \
 1731:         data_type_mixed, alignment_constraints)
```
**EN:** Defines `GenerateSM70_WmmaTensorOp_161616()`, which generates sm70 wmma tensor op 161616. Key helper calls include MathInstruction, CreateGemmOperator, TileDescription.

**CN:** 定义 `GenerateSM70_WmmaTensorOp_161616()`，用于生成 sm70 wmma tensor op 161616。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, TileDescription。

### L1732-L1736 — Comments

```python
 1732: 
 1733: #
 1734: ##################################################################################################
 1735: #
 1736: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1737-L1739 — Function `GenerateSM70`

```python
 1737: def GenerateSM70(manifest, cuda_version):
 1738:   GenerateSM70_TensorOp_884(manifest, cuda_version)
 1739:   GenerateSM70_PlanarComplexTensorOp_884(manifest, cuda_version)
```
**EN:** Defines `GenerateSM70()`, which generates sm70. Key helper calls include GenerateSM70_TensorOp_884, GenerateSM70_PlanarComplexTensorOp_884.

**CN:** 定义 `GenerateSM70()`，用于生成 sm70。 其中会调用的重要辅助函数包括 GenerateSM70_TensorOp_884, GenerateSM70_PlanarComplexTensorOp_884。

### L1740-L1748 — Comments

```python
 1740: 
 1741:   # To limit build size, WMMA GEMMs are disabled for now.
 1742:   #
 1743:   #GenerateSM70_WmmaTensorOp_161616(manifest, cuda_version)
 1744: 
 1745: ###################################################################################################
 1746: ###################################################################################################
 1747: 
 1748: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1749-L1787 — Function `GenerateSM75_TensorOp_1688_FewChannels`

```python
 1749: def GenerateSM75_TensorOp_1688_FewChannels(manifest, cuda_version, math_inst):
 1750: 
 1751:   min_cc = 75
 1752:   max_cc = 1024
 1753: 
 1754:   tile_descriptions = [
 1755:     TileDescription([128,  64, 32], 2, [2, 4, 1], math_inst, min_cc, max_cc),
 1756:     TileDescription([256,  64, 32], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1757:     TileDescription([128, 128, 32], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1758:     TileDescription([ 64, 256, 32], 2, [2, 4, 1], math_inst, min_cc, max_cc),
 1759:     TileDescription([ 64, 128, 32], 2, [2, 4, 1], math_inst, min_cc, max_cc),
 1760:     TileDescription([ 64,  64, 32], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1761:     TileDescription([ 64, 128, 64], 2, [2, 2, 2], math_inst, min_cc, max_cc),
 1762:   ]
 1763: 
 1764:   data_type = [
 1765:     math_inst.element_a,
 1766:     math_inst.element_b,
 1767:     math_inst.element_accumulator,
 1768:     math_inst.element_accumulator,
 1769:   ]
 1770: 
 1771:   conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 1772: 
 1773:   CreateConv2dFixedChannelsOperator(manifest, conv_layout, tile_descriptions, data_type, [4, 8])
 1774:   CreateConv2dFewChannelsOperator(manifest, conv_layout, tile_descriptions, data_type, [1, 2, 4])
 1775: 
 1776:   # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 1777:   if math_inst.element_a != math_inst.element_accumulator:
 1778: 
 1779:     data_type_mixed = [
 1780:       math_inst.element_a,
 1781:       math_inst.element_b,
 1782:       math_inst.element_a,
 1783:       math_inst.element_accumulator,
 1784:     ]
 1785: 
 1786:     CreateConv2dFixedChannelsOperator(manifest, conv_layout, tile_descriptions, data_type_mixed, [4, 8])
 1787:     CreateConv2dFewChannelsOperator(manifest, conv_layout, tile_descriptions, data_type_mixed, [1, 2, 4])
```
**EN:** Defines `GenerateSM75_TensorOp_1688_FewChannels()`, which generates sm75 tensor op 1688 few channels. Key helper calls include CreateConv2dFixedChannelsOperator, CreateConv2dFewChannelsOperator, TileDescription.

**CN:** 定义 `GenerateSM75_TensorOp_1688_FewChannels()`，用于生成 sm75 tensor op 1688 few channels。 其中会调用的重要辅助函数包括 CreateConv2dFixedChannelsOperator, CreateConv2dFewChannelsOperator, TileDescription。

### L1788-L1789 — Comments

```python
 1788: 
 1789: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1790-L1863 — Function `GenerateSM75_TensorOp_1688`

```python
 1790: def GenerateSM75_TensorOp_1688(manifest, cuda_version):
 1791: 
 1792:   if not CudaToolkitVersionSatisfies(cuda_version, 10, 2):
 1793:     return
 1794: 
 1795:   layouts = [
 1796:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1797:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1798:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1799:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1800:   ]
 1801: 
 1802:   math_instructions = [
 1803:     MathInstruction(                                  \
 1804:       [16, 8, 8],                                     \
 1805:       DataType.f16, DataType.f16, DataType.f32,       \
 1806:       OpcodeClass.TensorOp,                           \
 1807:       MathOperation.multiply_add),
 1808:     MathInstruction(                                  \
 1809:       [16, 8, 8],                                     \
 1810:       DataType.f16, DataType.f16, DataType.f16,       \
 1811:       OpcodeClass.TensorOp,                           \
 1812:       MathOperation.multiply_add),
 1813:   ]
 1814: 
 1815:   min_cc = 75
 1816:   max_cc = 1024
 1817: 
 1818:   alignment_constraints = [8, 4, 2, 1]
 1819: 
 1820:   for math_inst in math_instructions:
 1821:     tile_descriptions = [
 1822:       TileDescription([256, 128, 32], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1823:       TileDescription([128, 256, 32], 2, [2, 4, 1], math_inst, min_cc, max_cc),
 1824:       TileDescription([128, 128, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1825:       TileDescription([ 64, 256, 32], 2, [1, 4, 1], math_inst, min_cc, max_cc),
 1826:       TileDescription([256,  64, 32], 2, [4, 1, 1], math_inst, min_cc, max_cc),
 1827:       TileDescription([ 64, 128, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1828:       TileDescription([128,  64, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1829:       TileDescription([ 64,  64, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1830:       TileDescription([ 64, 128, 64], 2, [1, 2, 2], math_inst, min_cc, max_cc),
 1831:     ]
 1832: 
 1833:     data_type = [
 1834:       math_inst.element_a,
 1835:       math_inst.element_b,
 1836:       math_inst.element_accumulator,
 1837:       math_inst.element_accumulator,
 1838:     ]
 1839: 
 1840:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 1841:       data_type, alignment_constraints)
 1842: 
 1843:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 1844: 
 1845:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type, alignment_constraints)
 1846: 
 1847:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 1848:     if math_inst.element_a != math_inst.element_accumulator:
 1849: 
 1850:       data_type_mixed = [
 1851:         math_inst.element_a,
 1852:         math_inst.element_b,
 1853:         math_inst.element_a,
 1854:         math_inst.element_accumulator,
 1855:       ]
 1856: 
 1857:       CreateGemmOperator(manifest, layouts, tile_descriptions, \
 1858:         data_type_mixed, alignment_constraints)
 1859: 
 1860:       CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type_mixed, alignment_constraints)
 1861: 
 1862:     # Separate generator for 'few channels' specializations
 1863:     GenerateSM75_TensorOp_1688_FewChannels(manifest, cuda_version, math_inst)
```
**EN:** Defines `GenerateSM75_TensorOp_1688()`, which generates sm75 tensor op 1688. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, GenerateSM75_TensorOp_1688_FewChannels, TileDescription.

**CN:** 定义 `GenerateSM75_TensorOp_1688()`，用于生成 sm75 tensor op 1688。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, GenerateSM75_TensorOp_1688_FewChannels, TileDescription。

### L1864-L1867 — Comments

```python
 1864: 
 1865: #
 1866: 
 1867: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1868-L1933 — Function `GenerateSM75_PlanarComplexTensorOp_1688`

```python
 1868: def GenerateSM75_PlanarComplexTensorOp_1688(manifest, cuda_version):
 1869: 
 1870:   if not CudaToolkitVersionSatisfies(cuda_version, 10, 2):
 1871:     return
 1872: 
 1873:   layouts = [
 1874:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1875:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1876:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1877:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 1878:   ]
 1879: 
 1880:   complex_transforms = [
 1881:     (ComplexTransform.none, ComplexTransform.none),
 1882:     (ComplexTransform.conj, ComplexTransform.none),
 1883:     (ComplexTransform.none, ComplexTransform.conj),
 1884:     (ComplexTransform.conj, ComplexTransform.conj)
 1885:   ]
 1886: 
 1887:   math_instructions = [
 1888:     MathInstruction(                                  \
 1889:       [16, 8, 8],                                     \
 1890:       DataType.f16, DataType.f16, DataType.f32,       \
 1891:       OpcodeClass.TensorOp,                           \
 1892:       MathOperation.multiply_add),
 1893:     MathInstruction(                                  \
 1894:       [16, 8, 8],                                     \
 1895:       DataType.f16, DataType.f16, DataType.f16,       \
 1896:       OpcodeClass.TensorOp,                           \
 1897:       MathOperation.multiply_add),
 1898:   ]
 1899: 
 1900:   min_cc = 75
 1901:   max_cc = 1024
 1902: 
 1903:   alignment_constraints = [8, 2, 1]
 1904: 
 1905:   for math_inst in math_instructions:
 1906:     tile_descriptions = [
 1907:       TileDescription([ 64, 128, 32], 2, [2, 4, 1], math_inst, min_cc, max_cc),
 1908:       TileDescription([128,  64, 32], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1909:       TileDescription([ 64,  64, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1910:     ]
 1911: 
 1912:     data_type = [
 1913:       math_inst.element_a,
 1914:       math_inst.element_b,
 1915:       math_inst.element_accumulator,
 1916:       math_inst.element_accumulator,
 1917:     ]
 1918: 
 1919:     CreateGemmPlanarComplexOperator(manifest, layouts, tile_descriptions, \
 1920:       data_type, alignment_constraints, complex_transforms)
 1921: 
 1922:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 1923:     if math_inst.element_a != math_inst.element_accumulator:
 1924: 
 1925:       data_type_mixed = [
 1926:         math_inst.element_a,
 1927:         math_inst.element_b,
 1928:         math_inst.element_a,
 1929:         math_inst.element_accumulator,
 1930:       ]
 1931: 
 1932:       CreateGemmPlanarComplexOperator(manifest, layouts, tile_descriptions, \
 1933:         data_type_mixed, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM75_PlanarComplexTensorOp_1688()`, which generates sm75 planar complex tensor op 1688. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmPlanarComplexOperator, TileDescription.

**CN:** 定义 `GenerateSM75_PlanarComplexTensorOp_1688()`，用于生成 sm75 planar complex tensor op 1688。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmPlanarComplexOperator, TileDescription。

### L1934-L1935 — Comments

```python
 1934: 
 1935: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1936-L2033 — Function `GenerateSM75_TensorOp_8816_TN`

```python
 1936: def GenerateSM75_TensorOp_8816_TN(manifest, cuda_version):
 1937: 
 1938:   if not CudaToolkitVersionSatisfies(cuda_version, 10, 2):
 1939:     return
 1940: 
 1941:   layouts = [
 1942:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 1943:   ]
 1944: 
 1945:   math_instructions = [
 1946:     MathInstruction(                                  \
 1947:       [8, 8, 16],                                     \
 1948:       DataType.s8, DataType.s8, DataType.s32,         \
 1949:       OpcodeClass.TensorOp,                           \
 1950:       MathOperation.multiply_add_saturate),
 1951:     MathInstruction(                                  \
 1952:       [8, 8, 16],                                     \
 1953:       DataType.u8, DataType.u8, DataType.s32,         \
 1954:       OpcodeClass.TensorOp,                           \
 1955:       MathOperation.multiply_add_saturate),
 1956:   ]
 1957: 
 1958:   min_cc = 75
 1959:   max_cc = 90
 1960: 
 1961:   alignment_constraints = [16,]
 1962:   alignment_constraints_small_channels = [16, 8, 4]
 1963: 
 1964:   for math_inst in math_instructions:
 1965:     tile_descriptions = [
 1966:       TileDescription([256, 128, 64], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1967:       TileDescription([128, 256, 64], 2, [2, 4, 1], math_inst, min_cc, max_cc),
 1968:       TileDescription([128, 128, 64], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1969:       TileDescription([ 64, 256, 64], 2, [1, 4, 1], math_inst, min_cc, max_cc),
 1970:       TileDescription([256,  64, 64], 2, [4, 1, 1], math_inst, min_cc, max_cc),
 1971:       TileDescription([ 64, 128, 64], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1972:       TileDescription([128,  64, 64], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1973:       TileDescription([ 64,  64, 64], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1974:       TileDescription([256,  32, 64], 2, [4, 1, 1], math_inst, min_cc, max_cc),
 1975:       TileDescription([ 32, 256, 64], 2, [1, 4, 1], math_inst, min_cc, max_cc),
 1976:       TileDescription([128,  32, 64], 2, [4, 1, 1], math_inst, min_cc, max_cc),
 1977:       TileDescription([ 64,  32, 64], 2, [2, 1, 1], math_inst, min_cc, max_cc),
 1978: 
 1979:       TileDescription([256, 128, 32], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 1980:       TileDescription([128, 256, 32], 2, [2, 4, 1], math_inst, min_cc, max_cc),
 1981:       TileDescription([128, 128, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1982:       TileDescription([ 64, 256, 32], 2, [1, 4, 1], math_inst, min_cc, max_cc),
 1983:       TileDescription([256,  64, 32], 2, [4, 1, 1], math_inst, min_cc, max_cc),
 1984:       TileDescription([ 64, 128, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1985:       TileDescription([128,  64, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1986:       TileDescription([ 64,  64, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 1987:       TileDescription([128,  32, 32], 2, [2, 1, 1], math_inst, min_cc, max_cc),
 1988:       TileDescription([ 64,  32, 32], 2, [2, 1, 1], math_inst, min_cc, max_cc),
 1989:     ]
 1990: 
 1991:     data_type = [
 1992:       math_inst.element_a,
 1993:       math_inst.element_b,
 1994:       math_inst.element_accumulator,
 1995:       DataType.s32,
 1996:     ]
 1997: 
 1998:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 1999:       data_type, alignment_constraints, None, EpilogueFunctor.LinearCombination)
 2000: 
 2001:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 2002:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions,
 2003:       data_type, alignment_constraints, [ConvKind.Fprop], EpilogueFunctor.LinearCombination)
 2004: 
 2005:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 2006:     if math_inst.element_a != math_inst.element_accumulator:
 2007: 
 2008:       data_type_mixed = [
 2009:         math_inst.element_a,
 2010:         math_inst.element_b,
 2011:         math_inst.element_a,
 2012:         DataType.f32,
 2013:       ]
 2014: 
 2015:       operations = []
 2016: 
 2017:       operations += CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2018:         data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombinationClamp)
 2019: 
 2020:       operations += CreateConv2dOperator(manifest, conv_layout, tile_descriptions,
 2021:         data_type_mixed, alignment_constraints, [ConvKind.Fprop], EpilogueFunctor.LinearCombinationClamp)
 2022: 
 2023:       operations += CreateConv2dFixedChannelsOperator(manifest, conv_layout, tile_descriptions,
 2024:         data_type_mixed, alignment_constraints_small_channels, [ConvKind.Fprop], EpilogueFunctor.LinearCombinationClamp)
 2025: 
 2026:       operations += CreateConv2dFewChannelsOperator(manifest, conv_layout, tile_descriptions,
 2027:         data_type_mixed, alignment_constraints_small_channels, [ConvKind.Fprop], EpilogueFunctor.LinearCombinationClamp)
 2028: 
 2029:       for op in operations:
 2030:         if op.tile_description.threadblock_shape[1] >= 128:
 2031:           op.C.alignment = 16
 2032:         else:
 2033:           op.C.alignment = 8
```
**EN:** Defines `GenerateSM75_TensorOp_8816_TN()`, which generates sm75 tensor op 8816 tn. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription, CreateConv2dFixedChannelsOperator.

**CN:** 定义 `GenerateSM75_TensorOp_8816_TN()`，用于生成 sm75 tensor op 8816 tn。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription, CreateConv2dFixedChannelsOperator。

### L2034-L2037 — Comments

```python
 2034: 
 2035: #
 2036: 
 2037: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2038-L2093 — Function `GenerateSM75_TensorOp_8816_Interleaved`

```python
 2038: def GenerateSM75_TensorOp_8816_Interleaved(manifest, cuda_version):
 2039: 
 2040:   if not CudaToolkitVersionSatisfies(cuda_version, 10, 2):
 2041:     return
 2042: 
 2043:   layouts = [
 2044:     (LayoutType.ColumnMajorInterleaved32, LayoutType.RowMajorInterleaved32, LayoutType.ColumnMajorInterleaved32),
 2045:   ]
 2046: 
 2047:   math_instructions = [
 2048:     MathInstruction(                                  \
 2049:       [8, 8, 16],                                     \
 2050:       DataType.s8, DataType.s8, DataType.s32,         \
 2051:       OpcodeClass.TensorOp,                           \
 2052:       MathOperation.multiply_add_saturate),
 2053:     MathInstruction(                                  \
 2054:       [8, 8, 16],                                     \
 2055:       DataType.u8, DataType.u8, DataType.s32,         \
 2056:       OpcodeClass.TensorOp,                           \
 2057:       MathOperation.multiply_add_saturate),
 2058:   ]
 2059: 
 2060:   min_cc = 75
 2061:   max_cc = 90
 2062: 
 2063:   alignment_constraints = [16,]
 2064: 
 2065:   for math_inst in math_instructions:
 2066:     tile_descriptions = [
 2067:       TileDescription([256, 128, 64], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 2068:       TileDescription([128, 256, 64], 2, [2, 4, 1], math_inst, min_cc, max_cc),
 2069:       TileDescription([128, 128, 64], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 2070:       TileDescription([256,  64, 64], 2, [4, 1, 1], math_inst, min_cc, max_cc),
 2071:       TileDescription([ 64, 256, 64], 2, [1, 4, 1], math_inst, min_cc, max_cc),
 2072:       TileDescription([ 64, 128, 64], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 2073:       TileDescription([128,  64, 64], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 2074:       TileDescription([ 64,  64, 64], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 2075:     ]
 2076: 
 2077:     data_type_mixed = [
 2078:       math_inst.element_a,
 2079:       math_inst.element_b,
 2080:       math_inst.element_a,
 2081:       DataType.f32,
 2082:     ]
 2083: 
 2084:     operations = CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2085:       data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombinationClamp)
 2086: 
 2087:     conv_layout = (LayoutType.TensorNC32HW32, LayoutType.TensorC32RSK32, LayoutType.TensorNC32HW32)
 2088: 
 2089:     operations += CreateConv2dOperator(manifest, conv_layout, tile_descriptions,
 2090:       data_type_mixed, alignment_constraints, [ConvKind.Fprop], EpilogueFunctor.LinearCombinationClamp)
 2091: 
 2092:     for op in operations:
 2093:       op.C.alignment = 8
```
**EN:** Defines `GenerateSM75_TensorOp_8816_Interleaved()`, which generates sm75 tensor op 8816 interleaved. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription.

**CN:** 定义 `GenerateSM75_TensorOp_8816_Interleaved()`，用于生成 sm75 tensor op 8816 interleaved。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription。

### L2094-L2096 — Comments

```python
 2094: #
 2095: 
 2096: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2097-L2174 — Function `GenerateSM75_TensorOp_8832_TN`

```python
 2097: def GenerateSM75_TensorOp_8832_TN(manifest, cuda_version):
 2098: 
 2099:   if not CudaToolkitVersionSatisfies(cuda_version, 10, 2):
 2100:     return
 2101: 
 2102:   layouts = [
 2103:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 2104:   ]
 2105: 
 2106:   math_instructions = [
 2107:     MathInstruction(                                  \
 2108:       [8, 8, 32],                                     \
 2109:       DataType.s4, DataType.s4, DataType.s32,         \
 2110:       OpcodeClass.TensorOp,                           \
 2111:       MathOperation.multiply_add_saturate),
 2112:     MathInstruction(                                  \
 2113:       [8, 8, 32],                                     \
 2114:       DataType.u4, DataType.u4, DataType.s32,         \
 2115:       OpcodeClass.TensorOp,                           \
 2116:       MathOperation.multiply_add_saturate),
 2117:   ]
 2118: 
 2119:   min_cc = 75
 2120:   max_cc = 89
 2121: 
 2122:   alignment_constraints = [32,]
 2123: 
 2124:   for math_inst in math_instructions:
 2125:     tile_descriptions = [
 2126:       TileDescription([256, 128, 128], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 2127:       TileDescription([128, 256, 128], 2, [2, 4, 1], math_inst, min_cc, max_cc),
 2128:       TileDescription([128, 128, 128], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 2129:       TileDescription([256,  64, 128], 2, [4, 1, 1], math_inst, min_cc, max_cc),
 2130:       TileDescription([ 64, 256, 128], 2, [1, 4, 1], math_inst, min_cc, max_cc),
 2131:       TileDescription([ 64, 128, 128], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 2132:       TileDescription([128,  64, 128], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 2133:       TileDescription([ 64,  64, 128], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 2134:     ]
 2135: 
 2136:     data_type = [
 2137:       math_inst.element_a,
 2138:       math_inst.element_b,
 2139:       math_inst.element_accumulator,
 2140:       DataType.s32,
 2141:     ]
 2142: 
 2143:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2144:       data_type, alignment_constraints, None, EpilogueFunctor.LinearCombination)
 2145: 
 2146:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 2147:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions,
 2148:       data_type, alignment_constraints, [ConvKind.Fprop], EpilogueFunctor.LinearCombination)
 2149: 
 2150:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 2151:     if math_inst.element_a != math_inst.element_accumulator:
 2152: 
 2153:       data_type_mixed = [
 2154:         math_inst.element_a,
 2155:         math_inst.element_b,
 2156:         math_inst.element_a,
 2157:         DataType.f32,
 2158:       ]
 2159: 
 2160:       operations = []
 2161: 
 2162:       operations += CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2163:         data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombinationClamp)
 2164: 
 2165:       operations += CreateConv2dOperator(manifest, conv_layout, tile_descriptions,
 2166:         data_type_mixed, alignment_constraints, [ConvKind.Fprop], EpilogueFunctor.LinearCombinationClamp)
 2167: 
 2168:       for op in operations:
 2169:         if op.tile_description.threadblock_shape[1] >= 128:
 2170:           op.C.alignment = 16
 2171:         elif op.tile_description.threadblock_shape[1] == 64:
 2172:           op.C.alignment = 8
 2173:         else:
 2174:           op.C.alignment = 8
```
**EN:** Defines `GenerateSM75_TensorOp_8832_TN()`, which generates sm75 tensor op 8832 tn. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription.

**CN:** 定义 `GenerateSM75_TensorOp_8832_TN()`，用于生成 sm75 tensor op 8832 tn。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription。

### L2175-L2178 — Comments

```python
 2175: 
 2176: #
 2177: 
 2178: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2179-L2235 — Function `GenerateSM75_TensorOp_8832_Interleaved`

```python
 2179: def GenerateSM75_TensorOp_8832_Interleaved(manifest, cuda_version):
 2180: 
 2181:   if not CudaToolkitVersionSatisfies(cuda_version, 10, 2):
 2182:     return
 2183: 
 2184:   layouts = [
 2185:     (LayoutType.ColumnMajorInterleaved64, LayoutType.RowMajorInterleaved64, LayoutType.ColumnMajorInterleaved64),
 2186:   ]
 2187: 
 2188:   math_instructions = [
 2189:     MathInstruction(                                  \
 2190:       [8, 8, 32],                                     \
 2191:       DataType.s4, DataType.s4, DataType.s32,         \
 2192:       OpcodeClass.TensorOp,                           \
 2193:       MathOperation.multiply_add_saturate),
 2194:     MathInstruction(                                  \
 2195:       [8, 8, 32],                                     \
 2196:       DataType.u4, DataType.u4, DataType.s32,         \
 2197:       OpcodeClass.TensorOp,                           \
 2198:       MathOperation.multiply_add_saturate),
 2199:   ]
 2200: 
 2201:   min_cc = 75
 2202:   max_cc = 89
 2203: 
 2204:   alignment_constraints = [32,]
 2205: 
 2206:   for math_inst in math_instructions:
 2207:     tile_descriptions = [
 2208:       TileDescription([256, 128, 128], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 2209:       TileDescription([128, 256, 128], 2, [2, 4, 1], math_inst, min_cc, max_cc),
 2210:       TileDescription([128, 128, 128], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 2211:       TileDescription([256,  64, 128], 2, [4, 1, 1], math_inst, min_cc, max_cc),
 2212:       TileDescription([ 64, 256, 128], 2, [1, 4, 1], math_inst, min_cc, max_cc),
 2213:       TileDescription([ 64, 128, 128], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 2214:     ]
 2215: 
 2216:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 2217:     if math_inst.element_a != math_inst.element_accumulator:
 2218: 
 2219:       data_type_mixed = [
 2220:         math_inst.element_a,
 2221:         math_inst.element_b,
 2222:         math_inst.element_a,
 2223:         DataType.f32,
 2224:       ]
 2225: 
 2226:       operations = CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2227:         data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombinationClamp)
 2228: 
 2229:       conv_layout = (LayoutType.TensorNC64HW64, LayoutType.TensorC64RSK64, LayoutType.TensorNC64HW64)
 2230: 
 2231:       operations += CreateConv2dOperator(manifest, conv_layout, tile_descriptions,
 2232:         data_type_mixed, alignment_constraints, [ConvKind.Fprop], EpilogueFunctor.LinearCombinationClamp)
 2233: 
 2234:       for op in operations:
 2235:         op.C.alignment = 16
```
**EN:** Defines `GenerateSM75_TensorOp_8832_Interleaved()`, which generates sm75 tensor op 8832 interleaved. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, TileDescription, CreateGemmOperator, CreateConv2dOperator.

**CN:** 定义 `GenerateSM75_TensorOp_8832_Interleaved()`，用于生成 sm75 tensor op 8832 interleaved。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, TileDescription, CreateGemmOperator, CreateConv2dOperator。

### L2236-L2238 — Comments

```python
 2236: #
 2237: 
 2238: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2239-L2279 — Function `GenerateSM75_TensorOp_88128`

```python
 2239: def GenerateSM75_TensorOp_88128(manifest, cuda_version):
 2240: 
 2241:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 2242:     return
 2243: 
 2244:   layouts = [
 2245:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 2246:   ]
 2247: 
 2248:   math_instructions = [
 2249:     MathInstruction(                                  \
 2250:       [8, 8, 128],                                   \
 2251:       DataType.b1, DataType.b1, DataType.s32,         \
 2252:       OpcodeClass.TensorOp,                           \
 2253:       MathOperation.xor_popc),
 2254:   ]
 2255: 
 2256:   min_cc = 75
 2257:   max_cc = {
 2258:     MathOperation.xor_popc: 89,
 2259:     MathOperation.and_popc: 90
 2260:   }
 2261: 
 2262:   alignment_constraints = [128,]
 2263: 
 2264:   for math_inst in math_instructions:
 2265:     tile_descriptions = [
 2266:       TileDescription([256, 128, 512], 2, [4, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 2267:       TileDescription([128, 256, 512], 2, [2, 4, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 2268:       TileDescription([128, 128, 512], 2, [2, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 2269:       TileDescription([ 64, 256, 512], 2, [1, 4, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 2270:       TileDescription([256,  64, 512], 2, [4, 1, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 2271:       TileDescription([ 64, 128, 512], 2, [2, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 2272:       TileDescription([128,  64, 512], 2, [2, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 2273:       TileDescription([ 64,  64, 512], 2, [2, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 2274:     ]
 2275: 
 2276:     data_type = [DataType.b1, DataType.b1, DataType.s32, DataType.s32]
 2277: 
 2278:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2279:       data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM75_TensorOp_88128()`, which generates sm75 tensor op 88128. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription.

**CN:** 定义 `GenerateSM75_TensorOp_88128()`，用于生成 sm75 tensor op 88128。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription。

### L2280-L2283 — Comments

```python
 2280: 
 2281: #
 2282: 
 2283: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2284-L2338 — Function `GenerateSM75_WmmaTensorOp_161616`

```python
 2284: def GenerateSM75_WmmaTensorOp_161616(manifest, cuda_version):
 2285: 
 2286:   if not CudaToolkitVersionSatisfies(cuda_version, 10, 0):
 2287:     return
 2288: 
 2289:   layouts = [
 2290:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 2291:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 2292:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 2293:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 2294:   ]
 2295: 
 2296:   math_instructions = [
 2297:     MathInstruction(                                  \
 2298:       [16, 16, 16],                                   \
 2299:       DataType.s8, DataType.s8, DataType.s32,         \
 2300:       OpcodeClass.WmmaTensorOp,                       \
 2301:       MathOperation.multiply_add),
 2302:   ]
 2303: 
 2304:   min_cc = 75
 2305:   max_cc = 1024
 2306: 
 2307:   alignment_constraints = [16,]
 2308: 
 2309:   for math_inst in math_instructions:
 2310:     tile_descriptions = [
 2311:       TileDescription([128, 128, 32], 2, [4, 2, 1], math_inst, min_cc, max_cc),
 2312:       TileDescription([ 64, 128, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 2313:       TileDescription([128,  64, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 2314:       TileDescription([ 64,  64, 32], 2, [2, 2, 1], math_inst, min_cc, max_cc),
 2315:     ]
 2316: 
 2317:     data_type = [
 2318:       math_inst.element_a,
 2319:       math_inst.element_b,
 2320:       math_inst.element_accumulator,
 2321:       DataType.f32,
 2322:     ]
 2323: 
 2324:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2325:       data_type, alignment_constraints)
 2326: 
 2327:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 2328:     if math_inst.element_a != math_inst.element_accumulator:
 2329: 
 2330:       data_type_mixed = [
 2331:         math_inst.element_a,
 2332:         math_inst.element_b,
 2333:         math_inst.element_a,
 2334:         DataType.f32,
 2335:       ]
 2336: 
 2337:       CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2338:         data_type_mixed, alignment_constraints)
```
**EN:** Defines `GenerateSM75_WmmaTensorOp_161616()`, which generates sm75 wmma tensor op 161616. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription.

**CN:** 定义 `GenerateSM75_WmmaTensorOp_161616()`，用于生成 sm75 wmma tensor op 161616。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription。

### L2339-L2341 — Comments

```python
 2339: #
 2340: 
 2341: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2342-L2375 — Function `GenerateSM75_Simt_complex`

```python
 2342: def GenerateSM75_Simt_complex(manifest, cuda_version):
 2343:   math_instructions = [
 2344:     MathInstruction(                                  \
 2345:       [1, 1, 1],                                      \
 2346:       DataType.f32, DataType.f32, DataType.f32,       \
 2347:       OpcodeClass.Simt,                               \
 2348:       MathOperation.multiply_add_complex),
 2349:   ]
 2350: 
 2351:   min_cc = 75
 2352:   max_cc = 1024
 2353: 
 2354:   alignment_constraints = [1,]
 2355: 
 2356:   for math_inst in math_instructions:
 2357:     tile_descriptions = [
 2358:       TileDescription([128, 128, 8], 5, [4, 2, 1], math_inst, min_cc, max_cc)
 2359:     ]
 2360:     data_type = [
 2361:       DataType.cf32,
 2362:       DataType.cf32,
 2363:       DataType.cf32,
 2364:       DataType.cf32
 2365:     ]
 2366: 
 2367:     complex_transforms = [
 2368:       (ComplexTransform.none, ComplexTransform.none),
 2369:       (ComplexTransform.conj, ComplexTransform.none),
 2370:       (ComplexTransform.none, ComplexTransform.conj),
 2371:       (ComplexTransform.conj, ComplexTransform.conj)
 2372:     ]
 2373: 
 2374:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 2375:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM75_Simt_complex()`, which generates sm75 simt complex. Key helper calls include MathInstruction, CreateConv2dOperator, TileDescription.

**CN:** 定义 `GenerateSM75_Simt_complex()`，用于生成 sm75 simt complex。 其中会调用的重要辅助函数包括 MathInstruction, CreateConv2dOperator, TileDescription。

### L2376-L2377 — Comments

```python
 2376: #
 2377: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2378-L2387 — Function `GenerateSM75`

```python
 2378: def GenerateSM75(manifest, cuda_version):
 2379:   GenerateSM75_TensorOp_1688(manifest, cuda_version)
 2380:   GenerateSM75_PlanarComplexTensorOp_1688(manifest, cuda_version)
 2381:   GenerateSM75_TensorOp_8816_TN(manifest, cuda_version)
 2382:   GenerateSM75_TensorOp_8816_Interleaved(manifest, cuda_version)
 2383:   GenerateSM75_TensorOp_8832_TN(manifest, cuda_version)
 2384:   GenerateSM75_TensorOp_8832_Interleaved(manifest, cuda_version)
 2385:   GenerateSM75_TensorOp_88128(manifest, cuda_version)
 2386:   #GenerateSM75_WmmaTensorOp_161616(manifest, cuda_version)
 2387:   GenerateSM75_Simt_complex(manifest, cuda_version)
```
**EN:** Defines `GenerateSM75()`, which generates sm75. Key helper calls include GenerateSM75_TensorOp_1688, GenerateSM75_PlanarComplexTensorOp_1688, GenerateSM75_TensorOp_8816_TN, GenerateSM75_TensorOp_8816_Interleaved, GenerateSM75_TensorOp_8832_TN, GenerateSM75_TensorOp_8832_Interleaved.

**CN:** 定义 `GenerateSM75()`，用于生成 sm75。 其中会调用的重要辅助函数包括 GenerateSM75_TensorOp_1688, GenerateSM75_PlanarComplexTensorOp_1688, GenerateSM75_TensorOp_8816_TN, GenerateSM75_TensorOp_8816_Interleaved, GenerateSM75_TensorOp_8832_TN, GenerateSM75_TensorOp_8832_Interleaved。

### L2388-L2393 — Comments

```python
 2388: 
 2389: 
 2390: ###################################################################################################
 2391: ###################################################################################################
 2392: 
 2393: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2394-L2487 — Function `GenerateSM80_TensorOp_16816`

```python
 2394: def GenerateSM80_TensorOp_16816(manifest, cuda_version):
 2395: 
 2396:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 2397:     return
 2398: 
 2399:   layouts = [
 2400:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 2401:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 2402:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 2403:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 2404:   ]
 2405: 
 2406:   math_instructions = [
 2407:     MathInstruction(                                  \
 2408:       [16, 8, 16],                                    \
 2409:       DataType.f16, DataType.f16, DataType.f32,       \
 2410:       OpcodeClass.TensorOp,                           \
 2411:       MathOperation.multiply_add),
 2412:     MathInstruction(                                  \
 2413:       [16, 8, 16],                                    \
 2414:       DataType.f16, DataType.f16, DataType.f16,       \
 2415:       OpcodeClass.TensorOp,                           \
 2416:       MathOperation.multiply_add),
 2417:     MathInstruction(                                  \
 2418:       [16, 8, 16],                                    \
 2419:       DataType.bf16, DataType.bf16, DataType.f32,     \
 2420:       OpcodeClass.TensorOp,                           \
 2421:       MathOperation.multiply_add),
 2422:   ]
 2423: 
 2424:   min_cc = 80
 2425:   max_cc = 1024
 2426: 
 2427:   alignment_constraints = [8, 4, 2]
 2428: 
 2429:   for math_inst in math_instructions:
 2430:     tile_descriptions = [
 2431:       TileDescription([256, 128, 32],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 2432:       TileDescription([128, 256, 32],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 2433:       TileDescription([256,  64, 32],  3, [4, 1, 1], math_inst, min_cc, max_cc),
 2434:       TileDescription([256,  64, 32],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 2435:       TileDescription([ 64, 256, 32],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 2436:       TileDescription([128, 128, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2437:       TileDescription([128, 128, 32],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 2438:       TileDescription([128, 128, 32],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 2439:       TileDescription([128,  64, 32],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 2440:       TileDescription([ 64, 128, 32],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 2441:       TileDescription([ 64,  64, 32], 10, [2, 2, 1], math_inst, min_cc, max_cc),
 2442:       TileDescription([256, 128, 64],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 2443:       TileDescription([128, 256, 64],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 2444:       TileDescription([256,  64, 64],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 2445:       TileDescription([ 64, 256, 64],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 2446:       TileDescription([128, 128, 64],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 2447:       TileDescription([256,  64, 64],  3, [4, 1, 1], math_inst, min_cc, max_cc),
 2448:       TileDescription([ 64, 256, 64],  3, [1, 4, 1], math_inst, min_cc, max_cc),
 2449:       TileDescription([128, 128, 64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2450:       TileDescription([128,  64, 64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2451:       TileDescription([ 64, 128, 64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2452:       TileDescription([ 64,  64, 64],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 2453:     ]
 2454: 
 2455:     data_type = [
 2456:       math_inst.element_a,
 2457:       math_inst.element_b,
 2458:       math_inst.element_accumulator,
 2459:       math_inst.element_accumulator,
 2460:     ]
 2461: 
 2462:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2463:       data_type, alignment_constraints)
 2464: 
 2465:     CreateGemmGroupedOperator(manifest, layouts, tile_descriptions, data_type, alignment_constraints)
 2466: 
 2467:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 2468:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type, alignment_constraints)
 2469:     CreateConv2dFixedChannelsOperator(manifest, conv_layout, tile_descriptions, data_type, [4, 8])
 2470:     CreateConv3dOperator(manifest, LayoutType.TensorNDHWC, tile_descriptions, data_type, 8)
 2471: 
 2472:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 2473:     if math_inst.element_a != math_inst.element_accumulator:
 2474: 
 2475:       data_type_mixed = [
 2476:         math_inst.element_a,
 2477:         math_inst.element_b,
 2478:         math_inst.element_a,
 2479:         math_inst.element_accumulator,
 2480:       ]
 2481: 
 2482:       CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2483:         data_type_mixed, alignment_constraints)
 2484: 
 2485:       CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type_mixed, alignment_constraints)
 2486:       CreateConv2dFixedChannelsOperator(manifest, conv_layout, tile_descriptions, data_type_mixed, [4, 8])
 2487:       CreateConv3dOperator(manifest, LayoutType.TensorNDHWC, tile_descriptions, data_type_mixed, 8)
```
**EN:** Defines `GenerateSM80_TensorOp_16816()`, which generates sm80 tensor op 16816. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateGemmGroupedOperator, CreateConv2dOperator, CreateConv2dFixedChannelsOperator.

**CN:** 定义 `GenerateSM80_TensorOp_16816()`，用于生成 sm80 tensor op 16816。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateGemmGroupedOperator, CreateConv2dOperator, CreateConv2dFixedChannelsOperator。

### L2488-L2490 — Comments

```python
 2488: #
 2489: 
 2490: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2491-L2564 — Function `GenerateSM80_SparseTensorOp_16832`

```python
 2491: def GenerateSM80_SparseTensorOp_16832(manifest, cuda_version):
 2492: 
 2493:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 1):
 2494:     return
 2495: 
 2496:   layouts = [
 2497:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.RowMajor),
 2498:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.RowMajor),
 2499:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.RowMajor),
 2500:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.RowMajor),
 2501:   ]
 2502: 
 2503:   math_instructions = [
 2504:     MathInstruction(                                  \
 2505:       [16, 8, 32],                                    \
 2506:       DataType.f16, DataType.f16, DataType.f32,       \
 2507:       OpcodeClass.TensorOp,                           \
 2508:       MathOperation.multiply_add),
 2509:     MathInstruction(                                  \
 2510:       [16, 8, 32],                                    \
 2511:       DataType.f16, DataType.f16, DataType.f16,       \
 2512:       OpcodeClass.TensorOp,                           \
 2513:       MathOperation.multiply_add),
 2514:     MathInstruction(                                  \
 2515:       [16, 8, 32],                                    \
 2516:       DataType.bf16, DataType.bf16, DataType.f32,     \
 2517:       OpcodeClass.TensorOp,                           \
 2518:       MathOperation.multiply_add),
 2519:   ]
 2520: 
 2521:   min_cc = 80
 2522:   max_cc = 1024
 2523: 
 2524:   alignment_constraints = [8]
 2525: 
 2526:   for math_inst in math_instructions:
 2527:     tile_descriptions = [
 2528:       TileDescription([ 64, 128,  64],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 2529:       TileDescription([256, 128,  64],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 2530:       TileDescription([128, 256,  64],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 2531:       TileDescription([128, 128,  64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2532:       TileDescription([256,  64,  64],  3, [4, 1, 1], math_inst, min_cc, max_cc),
 2533:       TileDescription([ 64, 256,  64],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 2534:       TileDescription([128,  64,  64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2535:       TileDescription([ 64,  64,  64],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 2536:       TileDescription([128, 128, 128],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 2537:       TileDescription([256,  64, 128],  3, [4, 1, 1], math_inst, min_cc, max_cc),
 2538:       TileDescription([128,  64, 128],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 2539:       TileDescription([ 64, 128, 128],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2540:       TileDescription([ 64,  64, 128],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2541:     ]
 2542: 
 2543:     data_type = [
 2544:       math_inst.element_a,
 2545:       math_inst.element_b,
 2546:       math_inst.element_accumulator,
 2547:       math_inst.element_accumulator,
 2548:     ]
 2549: 
 2550:     CreateSparseGemmOperator(manifest, layouts, tile_descriptions, \
 2551:       data_type, alignment_constraints)
 2552: 
 2553:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 2554:     if math_inst.element_a != math_inst.element_accumulator:
 2555: 
 2556:       data_type_mixed = [
 2557:         math_inst.element_a,
 2558:         math_inst.element_b,
 2559:         math_inst.element_a,
 2560:         math_inst.element_accumulator,
 2561:       ]
 2562: 
 2563:       CreateSparseGemmOperator(manifest, layouts, tile_descriptions, \
 2564:         data_type_mixed, alignment_constraints)
```
**EN:** Defines `GenerateSM80_SparseTensorOp_16832()`, which generates sm80 sparse tensor op 16832. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateSparseGemmOperator, TileDescription.

**CN:** 定义 `GenerateSM80_SparseTensorOp_16832()`，用于生成 sm80 sparse tensor op 16832。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateSparseGemmOperator, TileDescription。

### L2565-L2568 — Comments

```python
 2565: 
 2566: #
 2567: 
 2568: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2569-L2639 — Function `GenerateSM80_PlanarComplexTensorOp_16816`

```python
 2569: def GenerateSM80_PlanarComplexTensorOp_16816(manifest, cuda_version):
 2570: 
 2571:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 2572:     return
 2573: 
 2574:   layouts = [
 2575:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 2576:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 2577:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 2578:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 2579:   ]
 2580: 
 2581:   complex_transforms = [
 2582:     (ComplexTransform.none, ComplexTransform.none),
 2583:     (ComplexTransform.conj, ComplexTransform.none),
 2584:     (ComplexTransform.none, ComplexTransform.conj),
 2585:     (ComplexTransform.conj, ComplexTransform.conj)
 2586:   ]
 2587: 
 2588:   math_instructions = [
 2589:     MathInstruction(                                  \
 2590:       [16, 8, 16],                                    \
 2591:       DataType.f16, DataType.f16, DataType.f32,       \
 2592:       OpcodeClass.TensorOp,                           \
 2593:       MathOperation.multiply_add),
 2594:     MathInstruction(                                  \
 2595:       [16, 8, 16],                                    \
 2596:       DataType.bf16, DataType.bf16, DataType.f32,     \
 2597:       OpcodeClass.TensorOp,                           \
 2598:       MathOperation.multiply_add),
 2599:     MathInstruction(                                  \
 2600:       [16, 8, 16],                                    \
 2601:       DataType.f16, DataType.f16, DataType.f16,       \
 2602:       OpcodeClass.TensorOp,                           \
 2603:       MathOperation.multiply_add),
 2604:   ]
 2605: 
 2606:   min_cc = 80
 2607:   max_cc = 1024
 2608: 
 2609:   alignment_constraints = [8, ]
 2610: 
 2611:   for math_inst in math_instructions:
 2612:     tile_descriptions = [
 2613:       TileDescription([ 64, 128, 32], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 2614:       TileDescription([128,  64, 32], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 2615:       TileDescription([ 64,  64, 32], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 2616:     ]
 2617: 
 2618:     data_type = [
 2619:       math_inst.element_a,
 2620:       math_inst.element_b,
 2621:       math_inst.element_accumulator,
 2622:       math_inst.element_accumulator,
 2623:     ]
 2624: 
 2625:     CreateGemmPlanarComplexOperator(manifest, layouts, tile_descriptions, \
 2626:       data_type, alignment_constraints, complex_transforms)
 2627: 
 2628:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 2629:     if math_inst.element_a != math_inst.element_accumulator:
 2630: 
 2631:       data_type_mixed = [
 2632:         math_inst.element_a,
 2633:         math_inst.element_b,
 2634:         math_inst.element_a,
 2635:         math_inst.element_accumulator,
 2636:       ]
 2637: 
 2638:       CreateGemmPlanarComplexOperator(manifest, layouts, tile_descriptions, \
 2639:         data_type_mixed, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM80_PlanarComplexTensorOp_16816()`, which generates sm80 planar complex tensor op 16816. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmPlanarComplexOperator, TileDescription.

**CN:** 定义 `GenerateSM80_PlanarComplexTensorOp_16816()`，用于生成 sm80 planar complex tensor op 16816。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmPlanarComplexOperator, TileDescription。

### L2640-L2641 — Comments

```python
 2640: 
 2641: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2642-L2737 — Function `GenerateSM80_TensorOp_16816_mixed_input_upcast_a`

```python
 2642: def GenerateSM80_TensorOp_16816_mixed_input_upcast_a(manifest, cuda_version):
 2643: 
 2644:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 2645:     return
 2646: 
 2647:   layouts = [
 2648:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 2649:   ]
 2650: 
 2651:   # Upcast on Operand A
 2652:   math_instructions = [
 2653:     MathInstruction(                                  \
 2654:       [16, 8, 16],                                    \
 2655:       DataType.s8, DataType.f16, DataType.f32,        \
 2656:       OpcodeClass.TensorOp,                           \
 2657:       MathOperation.multiply_add_mixed_input_upcast),
 2658:     MathInstruction(                                  \
 2659:       [16, 8, 16],                                    \
 2660:       DataType.u8, DataType.f16, DataType.f32,        \
 2661:       OpcodeClass.TensorOp,                           \
 2662:       MathOperation.multiply_add_mixed_input_upcast),
 2663:     MathInstruction(                                  \
 2664:       [16, 8, 16],                                    \
 2665:       DataType.s8, DataType.bf16, DataType.f32,       \
 2666:       OpcodeClass.TensorOp,                           \
 2667:       MathOperation.multiply_add_mixed_input_upcast),
 2668:     MathInstruction(                                  \
 2669:       [16, 8, 16],                                    \
 2670:       DataType.u8, DataType.bf16, DataType.f32,       \
 2671:       OpcodeClass.TensorOp,                           \
 2672:       MathOperation.multiply_add_mixed_input_upcast),
 2673:     MathInstruction(                                  \
 2674:       [16, 8, 16],                                    \
 2675:       DataType.s8, DataType.f16, DataType.f16,        \
 2676:       OpcodeClass.TensorOp,                           \
 2677:       MathOperation.multiply_add_mixed_input_upcast),
 2678:     MathInstruction(                                  \
 2679:       [16, 8, 16],                                    \
 2680:       DataType.u8, DataType.f16, DataType.f16,        \
 2681:       OpcodeClass.TensorOp,                           \
 2682:       MathOperation.multiply_add_mixed_input_upcast),
 2683:   ]
 2684: 
 2685:   min_cc = 80
 2686:   max_cc = 1024
 2687: 
 2688:   # For mixed-input alignment constraints are a list of lists, where the
 2689:   # inner list contains the alignment constraints for operands/matrices
 2690:   # [[alignA, alignB, alignC],..]
 2691:   alignment_constraints = [[16, 8, 8],]
 2692: 
 2693:   for math_inst in math_instructions:
 2694:     tile_descriptions = [
 2695:       # 128x128
 2696:       TileDescription([128, 128, 64],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 2697:       TileDescription([128, 128, 64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2698:       # 128x64
 2699:       TileDescription([128, 64, 64],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 2700:       TileDescription([128, 64, 64],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 2701:       TileDescription([128, 64, 64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2702:       # 128x32
 2703:       TileDescription([128, 32, 64],  9, [2, 2, 1], math_inst, min_cc, max_cc),
 2704:       TileDescription([128, 32, 64],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 2705:       # 128x16
 2706:       TileDescription([128, 16, 64],  5, [2, 1, 1], math_inst, min_cc, max_cc),
 2707:       TileDescription([128, 16, 64],  3, [2, 1, 1], math_inst, min_cc, max_cc),
 2708:     ]
 2709: 
 2710:     data_type = [
 2711:       math_inst.element_a,
 2712:       math_inst.element_b,
 2713:       math_inst.element_accumulator,
 2714:       math_inst.element_accumulator,
 2715:     ]
 2716: 
 2717:     # streamk uses more regs which can cause spill for the biggest warp tile size when the accumulators are 32bit.
 2718:     operations = CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2719:       data_type, alignment_constraints, None, EpilogueFunctor.LinearCombination, SwizzlingFunctor.Identity8)
 2720: 
 2721:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 2722:     if math_inst.element_b != math_inst.element_accumulator:
 2723: 
 2724:       data_type_mixed = [
 2725:         math_inst.element_a,
 2726:         math_inst.element_b,
 2727:         math_inst.element_b,
 2728:         math_inst.element_accumulator,
 2729:       ]
 2730: 
 2731:       operations += CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2732:         data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombination, SwizzlingFunctor.Identity8)
 2733: 
 2734:     for op in operations:
 2735:       if (DataTypeSize[op.C.element] == 16) and \
 2736:          (op.tile_description.threadblock_shape[1] <= 32):
 2737:         op.C.alignment = 4
```
**EN:** Defines `GenerateSM80_TensorOp_16816_mixed_input_upcast_a()`, which generates sm80 tensor op 16816 mixed input upcast a. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_16816_mixed_input_upcast_a()`，用于生成 sm80 tensor op 16816 mixed input upcast a。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription。

### L2738-L2739 — Comments

```python
 2738: 
 2739: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2740-L2841 — Function `GenerateSM80_TensorOp_16816_mixed_input_upcast_b`

```python
 2740: def GenerateSM80_TensorOp_16816_mixed_input_upcast_b(manifest, cuda_version):
 2741: 
 2742:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 2743:     return
 2744: 
 2745:   layouts = [
 2746:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 2747:   ]
 2748: 
 2749:   math_instructions = [
 2750:     MathInstruction(                                  \
 2751:       [16, 8, 16],                                    \
 2752:       DataType.f16, DataType.s8, DataType.f32,        \
 2753:       OpcodeClass.TensorOp,                           \
 2754:       MathOperation.multiply_add_mixed_input_upcast),
 2755:     MathInstruction(                                  \
 2756:       [16, 8, 16],                                    \
 2757:       DataType.f16, DataType.u8, DataType.f32,        \
 2758:       OpcodeClass.TensorOp,                           \
 2759:       MathOperation.multiply_add_mixed_input_upcast),
 2760:     MathInstruction(                                  \
 2761:       [16, 8, 16],                                    \
 2762:       DataType.bf16, DataType.s8, DataType.f32,       \
 2763:       OpcodeClass.TensorOp,                           \
 2764:       MathOperation.multiply_add_mixed_input_upcast),
 2765:     MathInstruction(                                  \
 2766:       [16, 8, 16],                                    \
 2767:       DataType.bf16, DataType.u8, DataType.f32,       \
 2768:       OpcodeClass.TensorOp,                           \
 2769:       MathOperation.multiply_add_mixed_input_upcast),
 2770:     MathInstruction(                                  \
 2771:       [16, 8, 16],                                    \
 2772:       DataType.f16, DataType.s8, DataType.f16,        \
 2773:       OpcodeClass.TensorOp,                           \
 2774:       MathOperation.multiply_add_mixed_input_upcast),
 2775:     MathInstruction(                                  \
 2776:       [16, 8, 16],                                    \
 2777:       DataType.f16, DataType.u8, DataType.f16,        \
 2778:       OpcodeClass.TensorOp,                           \
 2779:       MathOperation.multiply_add_mixed_input_upcast),
 2780:   ]
 2781: 
 2782:   min_cc = 80
 2783:   max_cc = 1024
 2784: 
 2785:   # For mixed-input alignment constraints are a list of lists, where the
 2786:   # inner list contains the alignment constraints for operands/matrices
 2787:   # [[alignA, alignB, alignC],..]
 2788:   alignment_constraints = [[8, 16, 8],]
 2789: 
 2790:   for math_inst in math_instructions:
 2791:     tile_descriptions = [
 2792:       # 128x128
 2793:       TileDescription([128, 128, 64],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 2794:       TileDescription([128, 128, 64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2795:       # 128x64
 2796:       TileDescription([128, 64, 64],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 2797:       TileDescription([128, 64, 64],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 2798:       TileDescription([128, 64, 64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2799:       # 128x32
 2800:       TileDescription([128, 32, 64],  9, [2, 2, 1], math_inst, min_cc, max_cc),
 2801:       TileDescription([128, 32, 64],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 2802:       TileDescription([128, 32, 32],  9, [2, 2, 1], math_inst, min_cc, max_cc),
 2803:       TileDescription([128, 32, 32],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 2804:       # 128x16
 2805:       TileDescription([128, 16, 64],  5, [2, 1, 1], math_inst, min_cc, max_cc),
 2806:       TileDescription([128, 16, 64],  3, [2, 1, 1], math_inst, min_cc, max_cc),
 2807:       TileDescription([128, 16, 32],  9, [2, 1, 1], math_inst, min_cc, max_cc),
 2808:       TileDescription([128, 16, 32],  5, [2, 1, 1], math_inst, min_cc, max_cc),
 2809:       TileDescription([128, 16, 32],  3, [2, 1, 1], math_inst, min_cc, max_cc),
 2810:       # 256x16
 2811:       TileDescription([256, 16, 32],  5, [2, 1, 1], math_inst, min_cc, max_cc),
 2812:       TileDescription([256, 16, 32],  3, [2, 1, 1], math_inst, min_cc, max_cc),
 2813:     ]
 2814: 
 2815:     data_type = [
 2816:       math_inst.element_a,
 2817:       math_inst.element_b,
 2818:       math_inst.element_accumulator,
 2819:       math_inst.element_accumulator,
 2820:     ]
 2821: 
 2822:     # streamk uses more regs which can cause spill for the biggest warp tile size when the accumulators are 32bit.
 2823:     operations = CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2824:       data_type, alignment_constraints, None, EpilogueFunctor.LinearCombination, SwizzlingFunctor.Identity8)
 2825: 
 2826:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 2827:     if math_inst.element_a != math_inst.element_accumulator:
 2828: 
 2829:       data_type_mixed = [
 2830:         math_inst.element_a,
 2831:         math_inst.element_b,
 2832:         math_inst.element_a,
 2833:         math_inst.element_accumulator,
 2834:       ]
 2835: 
 2836:       operations += CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2837:         data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombination, SwizzlingFunctor.Identity8)
 2838: 
 2839:     for op in operations:
 2840:       if op.tile_description.threadblock_shape[1] <= 32:
 2841:         op.C.alignment = 4
```
**EN:** Defines `GenerateSM80_TensorOp_16816_mixed_input_upcast_b()`, which generates sm80 tensor op 16816 mixed input upcast b. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_16816_mixed_input_upcast_b()`，用于生成 sm80 tensor op 16816 mixed input upcast b。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription。

### L2842-L2843 — Comments

```python
 2842: 
 2843: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2844-L2932 — Function `GenerateSM80_TensorOp_16832_TN`

```python
 2844: def GenerateSM80_TensorOp_16832_TN(manifest, cuda_version):
 2845: 
 2846:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 2847:     return
 2848: 
 2849:   layouts = [
 2850:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 2851:   ]
 2852: 
 2853:   math_instructions = [
 2854:     MathInstruction(                                  \
 2855:       [16, 8, 32],                                    \
 2856:       DataType.s8, DataType.s8, DataType.s32,         \
 2857:       OpcodeClass.TensorOp,                           \
 2858:       MathOperation.multiply_add_saturate),
 2859:     MathInstruction(                                  \
 2860:       [16, 8, 32],                                    \
 2861:       DataType.u8, DataType.u8, DataType.s32,         \
 2862:       OpcodeClass.TensorOp,                           \
 2863:       MathOperation.multiply_add_saturate),
 2864:   ]
 2865: 
 2866:   min_cc = 80
 2867:   max_cc = 1024
 2868:   smem_usage = 164
 2869: 
 2870:   alignment_constraints = [16,]
 2871:   alignment_constraints_small_channels = [16, 8, 4]
 2872: 
 2873:   for math_inst in math_instructions:
 2874:     tile_descriptions = [
 2875:       TileDescription([256, 128,  64],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 2876:       TileDescription([128, 256,  64],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 2877:       TileDescription([256,  64,  64],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 2878:       TileDescription([ 64, 256,  64],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 2879:       TileDescription([256,  32,  64],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 2880:       TileDescription([ 32, 256,  64],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 2881:       TileDescription([128, 128,  64],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 2882:       TileDescription([128,  64,  64],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 2883:       TileDescription([ 64, 128,  64],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 2884:       TileDescription([128,  32,  64],  6, [4, 1, 1], math_inst, min_cc, max_cc),
 2885:       TileDescription([ 32, 128,  64],  6, [1, 4, 1], math_inst, min_cc, max_cc),
 2886:       TileDescription([ 64,  64,  64], 10, [2, 2, 1], math_inst, min_cc, max_cc),
 2887:       TileDescription([256, 128, 128],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 2888:       TileDescription([128, 256, 128],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 2889:       TileDescription([256,  64, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 2890:       TileDescription([ 64, 256, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 2891:       TileDescription([256,  32, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 2892:       TileDescription([ 32, 256, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 2893:       TileDescription([128, 128, 128],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 2894:       TileDescription([128,  64, 128],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2895:       TileDescription([ 64, 128, 128],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2896:       TileDescription([128,  32, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 2897:       TileDescription([ 32, 128, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 2898:       TileDescription([ 64,  64, 128],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 2899:     ]
 2900: 
 2901:     data_type = [math_inst.element_a, math_inst.element_b, math_inst.element_accumulator, DataType.s32]
 2902:     data_type_mixed = [math_inst.element_a, math_inst.element_b, math_inst.element_a, DataType.f32]
 2903: 
 2904:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2905:       data_type, alignment_constraints, None, EpilogueFunctor.LinearCombination)
 2906: 
 2907:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 2908:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions,
 2909:       data_type, alignment_constraints, [ConvKind.Fprop], EpilogueFunctor.LinearCombination)
 2910: 
 2911:     operations = []
 2912: 
 2913:     operations += CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2914:       data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombinationClamp)
 2915: 
 2916:     operations += CreateConv2dOperator(manifest, conv_layout, tile_descriptions,
 2917:       data_type_mixed, alignment_constraints, [ConvKind.Fprop], EpilogueFunctor.LinearCombinationClamp)
 2918: 
 2919:     operations += CreateConv2dFixedChannelsOperator(manifest, conv_layout, tile_descriptions,
 2920:       data_type_mixed, alignment_constraints_small_channels, [ConvKind.Fprop], EpilogueFunctor.LinearCombinationClamp)
 2921: 
 2922:     operations += CreateConv2dFewChannelsOperator(manifest, conv_layout, tile_descriptions,
 2923:       data_type_mixed, alignment_constraints_small_channels, [ConvKind.Fprop], EpilogueFunctor.LinearCombinationClamp)
 2924: 
 2925:     for op in operations:
 2926:       if op.tile_description.threadblock_shape[1] >= 128:
 2927:         if op.tile_description.threadblock_shape[0] == 32:
 2928:           op.C.alignment = 8
 2929:         else:
 2930:           op.C.alignment = 16
 2931:       else:
 2932:         op.C.alignment = 8
```
**EN:** Defines `GenerateSM80_TensorOp_16832_TN()`, which generates sm80 tensor op 16832 tn. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, CreateConv2dFixedChannelsOperator, CreateConv2dFewChannelsOperator.

**CN:** 定义 `GenerateSM80_TensorOp_16832_TN()`，用于生成 sm80 tensor op 16832 tn。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, CreateConv2dFixedChannelsOperator, CreateConv2dFewChannelsOperator。

### L2933-L2935 — Comments

```python
 2933: 
 2934: #
 2935: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L2936-L3014 — Function `GenerateSM80_TensorOp_16832_TN_mixed_input_upcast_a`

```python
 2936: def GenerateSM80_TensorOp_16832_TN_mixed_input_upcast_a(manifest, cuda_version):
 2937: 
 2938:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 2939:     return
 2940: 
 2941:   layouts = [
 2942:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 2943:   ]
 2944: 
 2945:   # Upcast on Operand A
 2946:   math_instructions = [
 2947:     MathInstruction(                                  \
 2948:       [16, 8, 32],                                    \
 2949:       DataType.s4, DataType.s8, DataType.s32,         \
 2950:       OpcodeClass.TensorOp,                           \
 2951:       MathOperation.multiply_add_mixed_input_upcast),
 2952:   ]
 2953: 
 2954:   min_cc = 80
 2955:   max_cc = 1024
 2956: 
 2957:   # For mixed-input alignment constraints are a list of lists, where the 
 2958:   # inner list contains the alignment constraints for operands/matrices 
 2959:   # [[alignA, alignB, alignC],..]
 2960:   alignment_constraints = [[32, 16, 4],]
 2961: 
 2962:   for math_inst in math_instructions:
 2963:     tile_descriptions = [
 2964:       TileDescription([256, 128,  64],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 2965:       TileDescription([128, 256,  64],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 2966:       TileDescription([256,  64,  64],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 2967:       TileDescription([ 64, 256,  64],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 2968:       TileDescription([ 32, 256,  64],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 2969:       TileDescription([128, 128,  64],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 2970:       TileDescription([ 64, 128,  64],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 2971:       TileDescription([256, 128, 128],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 2972:       TileDescription([128, 256, 128],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 2973:       TileDescription([256,  64, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 2974:       TileDescription([ 64, 256, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 2975:       TileDescription([256,  32, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 2976:       TileDescription([ 32, 256, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 2977:       TileDescription([128, 128, 128],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 2978:       TileDescription([ 64, 128, 128],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 2979:       TileDescription([128,  32, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 2980:     ]
 2981: 
 2982:     data_type = [
 2983:       math_inst.element_a,
 2984:       math_inst.element_b,
 2985:       math_inst.element_accumulator,
 2986:       math_inst.element_accumulator,
 2987:     ]
 2988: 
 2989:     # streamk uses more regs which can cause spill for the biggest warp tile size when the accumulators are 32bit.
 2990:     operations = CreateGemmOperator(manifest, layouts, tile_descriptions, \
 2991:       data_type, alignment_constraints, None, EpilogueFunctor.LinearCombination, SwizzlingFunctor.Identity8)
 2992: 
 2993:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. S8 accumulation)
 2994:     if math_inst.element_a != math_inst.element_accumulator:
 2995:       alignment_constraints = [[32, 16, 16],]
 2996: 
 2997:       data_type_mixed = [
 2998:         math_inst.element_a,
 2999:         math_inst.element_b,
 3000:         math_inst.element_b,
 3001:         DataType.f32
 3002:       ]
 3003: 
 3004:       operations += CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3005:         data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombinationClamp, SwizzlingFunctor.Identity8)
 3006: 
 3007:     for op in operations:
 3008:       if op.tile_description.threadblock_shape[1] >= 128:
 3009:         if op.tile_description.threadblock_shape[0] == 32:
 3010:           op.C.alignment = 8
 3011:         else:
 3012:           op.C.alignment = 16
 3013:       else:
 3014:         op.C.alignment = 8
```
**EN:** Defines `GenerateSM80_TensorOp_16832_TN_mixed_input_upcast_a()`, which generates sm80 tensor op 16832 tn mixed input upcast a. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_16832_TN_mixed_input_upcast_a()`，用于生成 sm80 tensor op 16832 tn mixed input upcast a。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription。

### L3015-L3017 — Comments

```python
 3015: #
 3016: 
 3017: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3018-L3097 — Function `GenerateSM80_TensorOp_16832_TN_mixed_input_upcast_b`

```python
 3018: def GenerateSM80_TensorOp_16832_TN_mixed_input_upcast_b(manifest, cuda_version):
 3019: 
 3020:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3021:     return
 3022: 
 3023:   layouts = [
 3024:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3025:   ]
 3026: 
 3027:   # Upcast on Operand B
 3028:   math_instructions = [
 3029:     MathInstruction(                                  \
 3030:       [16, 8, 32],                                    \
 3031:       DataType.s8, DataType.s4, DataType.s32,         \
 3032:       OpcodeClass.TensorOp,                           \
 3033:       MathOperation.multiply_add_mixed_input_upcast),
 3034:   ]
 3035: 
 3036:   min_cc = 80
 3037:   max_cc = 1024
 3038: 
 3039:   # For mixed-input alignment constraints are a list of lists, where the 
 3040:   # inner list contains the alignment constraints for operands/matrices 
 3041:   # [[alignA, alignB, alignC],..]
 3042:   alignment_constraints = [[16, 32, 4],]
 3043: 
 3044:   for math_inst in math_instructions:
 3045:     tile_descriptions = [
 3046:       TileDescription([256, 128,  64],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3047:       TileDescription([128, 256,  64],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3048:       TileDescription([256,  64,  64],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3049:       TileDescription([ 64, 256,  64],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3050:       TileDescription([256,  32,  64],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3051:       TileDescription([128, 128,  64],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 3052:       TileDescription([ 64, 128,  64],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3053:       TileDescription([128,  32,  64],  6, [4, 1, 1], math_inst, min_cc, max_cc),
 3054:       TileDescription([256, 128, 128],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3055:       TileDescription([128, 256, 128],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3056:       TileDescription([256,  64, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3057:       TileDescription([ 64, 256, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3058:       TileDescription([256,  32, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3059:       TileDescription([ 32, 256, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3060:       TileDescription([128, 128, 128],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3061:       TileDescription([ 64, 128, 128],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3062:       TileDescription([128,  32, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3063:     ]
 3064: 
 3065:     data_type = [
 3066:       math_inst.element_a,
 3067:       math_inst.element_b,
 3068:       math_inst.element_accumulator,
 3069:       math_inst.element_accumulator,
 3070:     ]
 3071: 
 3072:     # streamk uses more regs which can cause spill for the biggest warp tile size when the accumulators are 32bit.
 3073:     operations = CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3074:       data_type, alignment_constraints, None, EpilogueFunctor.LinearCombination, SwizzlingFunctor.Identity8)
 3075: 
 3076:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. S8 accumulation)
 3077:     if math_inst.element_a != math_inst.element_accumulator:
 3078:       alignment_constraints = [[16, 32, 16],]
 3079: 
 3080:       data_type_mixed = [
 3081:         math_inst.element_a,
 3082:         math_inst.element_b,
 3083:         math_inst.element_a,
 3084:         DataType.f32,
 3085:       ]
 3086: 
 3087:       operations += CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3088:         data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombinationClamp, SwizzlingFunctor.Identity8)
 3089: 
 3090:     for op in operations:
 3091:       if op.tile_description.threadblock_shape[1] >= 128:
 3092:         if op.tile_description.threadblock_shape[0] == 32:
 3093:           op.C.alignment = 8
 3094:         else:
 3095:           op.C.alignment = 16
 3096:       else:
 3097:         op.C.alignment = 8
```
**EN:** Defines `GenerateSM80_TensorOp_16832_TN_mixed_input_upcast_b()`, which generates sm80 tensor op 16832 tn mixed input upcast b. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_16832_TN_mixed_input_upcast_b()`，用于生成 sm80 tensor op 16832 tn mixed input upcast b。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription。

### L3098-L3100 — Comments

```python
 3098: #
 3099: 
 3100: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3101-L3152 — Function `GenerateSM80_SparseTensorOp_16864_TN`

```python
 3101: def GenerateSM80_SparseTensorOp_16864_TN(manifest, cuda_version):
 3102: 
 3103:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 1):
 3104:     return
 3105: 
 3106:   layouts = [
 3107:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.RowMajor),
 3108:   ]
 3109: 
 3110:   math_inst =                                         \
 3111:     MathInstruction(                                  \
 3112:       [16, 8, 64],                                    \
 3113:       DataType.s8, DataType.s8, DataType.s32,         \
 3114:       OpcodeClass.TensorOp,                           \
 3115:       MathOperation.multiply_add_saturate)
 3116: 
 3117:   min_cc = 80
 3118:   max_cc = 1024
 3119: 
 3120:   alignment_constraints = [16,]
 3121: 
 3122:   tile_descriptions = [
 3123:     TileDescription([128,  64, 128],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3124:     TileDescription([256, 128, 128],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3125:     TileDescription([128, 256, 128],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3126:     TileDescription([128, 128, 128],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3127:     TileDescription([256,  64, 128],  3, [4, 1, 1], math_inst, min_cc, max_cc),
 3128:     TileDescription([ 64, 256, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3129:     TileDescription([ 64, 128, 128],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3130:     TileDescription([ 64,  64, 128],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3131:     TileDescription([128, 128, 256],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3132:     TileDescription([128,  64, 256],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3133:     TileDescription([ 64, 128, 256],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3134:     TileDescription([ 64,  64, 256],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3135:   ]
 3136: 
 3137:   data_type = [DataType.s8, DataType.s8, DataType.s32, DataType.s32]
 3138:   data_type_mixed = [DataType.s8, DataType.s8, DataType.s8, DataType.f32]
 3139: 
 3140:   CreateSparseGemmOperator(manifest, layouts, tile_descriptions, \
 3141:     data_type, alignment_constraints, None, EpilogueFunctor.LinearCombination)
 3142: 
 3143:   operations = []
 3144: 
 3145:   operations += CreateSparseGemmOperator(manifest, layouts, tile_descriptions, \
 3146:     data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombinationClamp)
 3147: 
 3148:   for op in operations:
 3149:     if op.tile_description.threadblock_shape[1] >= 128:
 3150:       op.C.alignment = 16
 3151:     else:
 3152:       op.C.alignment = 8
```
**EN:** Defines `GenerateSM80_SparseTensorOp_16864_TN()`, which generates sm80 sparse tensor op 16864 tn. Key helper calls include MathInstruction, CreateSparseGemmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_SparseTensorOp_16864_TN()`，用于生成 sm80 sparse tensor op 16864 tn。 其中会调用的重要辅助函数包括 MathInstruction, CreateSparseGemmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L3153-L3155 — Comments

```python
 3153: #
 3154: 
 3155: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3156-L3206 — Function `GenerateSM80_TensorOp_16832_Interleaved`

```python
 3156: def GenerateSM80_TensorOp_16832_Interleaved(manifest, cuda_version):
 3157: 
 3158:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3159:     return
 3160: 
 3161:   layouts = [
 3162:     (LayoutType.ColumnMajorInterleaved32, LayoutType.RowMajorInterleaved32, LayoutType.ColumnMajorInterleaved32),
 3163:   ]
 3164: 
 3165:   math_instructions = [
 3166:     MathInstruction(                                  \
 3167:       [16, 8, 32],                                    \
 3168:       DataType.s8, DataType.s8, DataType.s32,         \
 3169:       OpcodeClass.TensorOp,                           \
 3170:       MathOperation.multiply_add_saturate),
 3171:     MathInstruction(                                  \
 3172:       [16, 8, 32],                                    \
 3173:       DataType.u8, DataType.u8, DataType.s32,         \
 3174:       OpcodeClass.TensorOp,                           \
 3175:       MathOperation.multiply_add_saturate),
 3176:   ]
 3177: 
 3178:   min_cc = 80
 3179:   max_cc = 1024
 3180: 
 3181:   alignment_constraints = [16,]
 3182: 
 3183:   for math_inst in math_instructions:
 3184:     tile_descriptions = [
 3185:       TileDescription([256, 128, 64],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3186:       TileDescription([128, 256, 64],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3187:       TileDescription([256,  64, 64],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3188:       TileDescription([ 64, 256, 64],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3189:       TileDescription([128, 128, 64],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 3190:       TileDescription([128,  64, 64],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3191:       TileDescription([ 64, 128, 64],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3192:       TileDescription([ 64,  64, 64], 10, [2, 2, 1], math_inst, min_cc, max_cc),
 3193:     ]
 3194: 
 3195:     data_type_mixed = [math_inst.element_a, math_inst.element_b, math_inst.element_a, DataType.f32]
 3196: 
 3197:     operations = CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3198:       data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombinationClamp)
 3199: 
 3200:     conv_layout = (LayoutType.TensorNC32HW32, LayoutType.TensorC32RSK32, LayoutType.TensorNC32HW32)
 3201: 
 3202:     operations += CreateConv2dOperator(manifest, conv_layout, tile_descriptions,
 3203:       data_type_mixed, alignment_constraints, [ConvKind.Fprop], EpilogueFunctor.LinearCombinationClamp)
 3204: 
 3205:     for op in operations:
 3206:       op.C.alignment = 8
```
**EN:** Defines `GenerateSM80_TensorOp_16832_Interleaved()`, which generates sm80 tensor op 16832 interleaved. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_16832_Interleaved()`，用于生成 sm80 tensor op 16832 interleaved。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription。

### L3207-L3209 — Comments

```python
 3207: #
 3208: 
 3209: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3210-L3281 — Function `GenerateSM80_TensorOp_16864_TN`

```python
 3210: def GenerateSM80_TensorOp_16864_TN(manifest, cuda_version):
 3211: 
 3212:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3213:     return
 3214: 
 3215:   layouts = [
 3216:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3217:   ]
 3218: 
 3219:   math_instructions = [
 3220:     MathInstruction(                                  \
 3221:       [16, 8, 64],                                    \
 3222:       DataType.s4, DataType.s4, DataType.s32,         \
 3223:       OpcodeClass.TensorOp,                           \
 3224:       MathOperation.multiply_add_saturate),
 3225:     MathInstruction(                                  \
 3226:       [16, 8, 64],                                    \
 3227:       DataType.u4, DataType.u4, DataType.s32,         \
 3228:       OpcodeClass.TensorOp,                           \
 3229:       MathOperation.multiply_add_saturate),
 3230:   ]
 3231: 
 3232:   min_cc = 80
 3233:   max_cc = 1024
 3234:   alignment_constraints = [32,]
 3235: 
 3236:   for math_inst in math_instructions:
 3237:     tile_descriptions = [
 3238:       TileDescription([256, 128, 128],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3239:       TileDescription([128, 256, 128],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3240:       TileDescription([256,  64, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3241:       TileDescription([ 64, 256, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3242:       TileDescription([128, 128, 128],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 3243:       TileDescription([128,  64, 128],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3244:       TileDescription([ 64, 128, 128],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3245:       TileDescription([ 64,  64, 128], 10, [2, 2, 1], math_inst, min_cc, max_cc),
 3246:       TileDescription([256, 128, 256],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3247:       TileDescription([128, 256, 256],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3248:       TileDescription([256,  64, 256],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3249:       TileDescription([ 64, 256, 256],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3250:       TileDescription([128, 128, 256],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3251:       TileDescription([128, 128, 256],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3252:       TileDescription([128,  64, 256],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3253:       TileDescription([ 64, 128, 256],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3254:       TileDescription([ 64,  64, 256],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 3255:     ]
 3256: 
 3257:     data_type = [math_inst.element_a, math_inst.element_b, math_inst.element_accumulator, DataType.s32]
 3258:     data_type_mixed = [math_inst.element_a, math_inst.element_b, math_inst.element_a, DataType.f32]
 3259: 
 3260:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3261:       data_type, alignment_constraints, None, EpilogueFunctor.LinearCombination)
 3262: 
 3263:     operations = []
 3264: 
 3265:     operations += CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3266:       data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombinationClamp)
 3267: 
 3268:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 3269:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions,
 3270:       data_type, alignment_constraints, [ConvKind.Fprop], EpilogueFunctor.LinearCombination)
 3271: 
 3272:     operations += CreateConv2dOperator(manifest, conv_layout, tile_descriptions,
 3273:       data_type_mixed, alignment_constraints, [ConvKind.Fprop], EpilogueFunctor.LinearCombinationClamp)
 3274: 
 3275:     for op in operations:
 3276:       if op.tile_description.threadblock_shape[1] >= 128:
 3277:         op.C.alignment = 16
 3278:       elif op.tile_description.threadblock_shape[1] == 64:
 3279:         op.C.alignment = 8
 3280:       else:
 3281:         op.C.alignment = 8
```
**EN:** Defines `GenerateSM80_TensorOp_16864_TN()`, which generates sm80 tensor op 16864 tn. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_16864_TN()`，用于生成 sm80 tensor op 16864 tn。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription。

### L3282-L3284 — Comments

```python
 3282: #
 3283: 
 3284: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3285-L3335 — Function `GenerateSM80_SparseTensorOp_168128_TN`

```python
 3285: def GenerateSM80_SparseTensorOp_168128_TN(manifest, cuda_version):
 3286: 
 3287:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 1):
 3288:     return
 3289: 
 3290:   layouts = [
 3291:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.RowMajor),
 3292:   ]
 3293: 
 3294:   math_inst =                                         \
 3295:     MathInstruction(                                  \
 3296:       [16, 8, 128],                                    \
 3297:       DataType.s4, DataType.s4, DataType.s32,         \
 3298:       OpcodeClass.TensorOp,                           \
 3299:       MathOperation.multiply_add_saturate)
 3300: 
 3301:   min_cc = 80
 3302:   max_cc = 1024
 3303:   alignment_constraints = [32,]
 3304: 
 3305:   tile_descriptions = [
 3306:     TileDescription([ 64,  64, 256],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3307:     TileDescription([256,  64, 256],  3, [4, 1, 1], math_inst, min_cc, max_cc),
 3308:     TileDescription([256, 128, 256],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3309:     TileDescription([128, 256, 256],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3310:     TileDescription([128, 128, 256],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3311:     TileDescription([ 64, 256, 256],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3312:     TileDescription([128,  64, 256],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3313:     TileDescription([ 64, 128, 256],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3314:     TileDescription([128, 128, 512],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3315:     TileDescription([128,  64, 512],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3316:     TileDescription([ 64, 128, 512],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3317:     TileDescription([ 64,  64, 512],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3318:   ]
 3319: 
 3320:   data_type = [DataType.s4, DataType.s4, DataType.s32, DataType.s32]
 3321:   data_type_mixed = [DataType.s4, DataType.s4, DataType.s4, DataType.f32]
 3322: 
 3323:   CreateSparseGemmOperator(manifest, layouts, tile_descriptions, \
 3324:     data_type, alignment_constraints, None, EpilogueFunctor.LinearCombination)
 3325: 
 3326:   operations = []
 3327: 
 3328:   operations += CreateSparseGemmOperator(manifest, layouts, tile_descriptions, \
 3329:     data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombinationClamp)
 3330: 
 3331:   for op in operations:
 3332:     if op.tile_description.threadblock_shape[1] > 128:
 3333:       op.C.alignment = 16
 3334:     else:
 3335:       op.C.alignment = 8
```
**EN:** Defines `GenerateSM80_SparseTensorOp_168128_TN()`, which generates sm80 sparse tensor op 168128 tn. Key helper calls include MathInstruction, CreateSparseGemmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_SparseTensorOp_168128_TN()`，用于生成 sm80 sparse tensor op 168128 tn。 其中会调用的重要辅助函数包括 MathInstruction, CreateSparseGemmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L3336-L3338 — Comments

```python
 3336: #
 3337: 
 3338: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3339-L3388 — Function `GenerateSM80_TensorOp_16864_Interleaved`

```python
 3339: def GenerateSM80_TensorOp_16864_Interleaved(manifest, cuda_version):
 3340: 
 3341:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3342:     return
 3343: 
 3344:   layouts = [
 3345:       (LayoutType.ColumnMajorInterleaved64, LayoutType.RowMajorInterleaved64, LayoutType.ColumnMajorInterleaved64),
 3346:   ]
 3347: 
 3348:   math_instructions = [
 3349:     MathInstruction(                                  \
 3350:       [16, 8, 64],                                    \
 3351:       DataType.s4, DataType.s4, DataType.s32,         \
 3352:       OpcodeClass.TensorOp,                           \
 3353:       MathOperation.multiply_add_saturate),
 3354:     MathInstruction(                                  \
 3355:       [16, 8, 64],                                    \
 3356:       DataType.u4, DataType.u4, DataType.s32,         \
 3357:       OpcodeClass.TensorOp,                           \
 3358:       MathOperation.multiply_add_saturate),
 3359:   ]
 3360: 
 3361:   min_cc = 80
 3362:   max_cc = 1024
 3363:   alignment_constraints = [32,]
 3364: 
 3365:   for math_inst in math_instructions:
 3366:     tile_descriptions = [
 3367:       TileDescription([256, 128, 128],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3368:       TileDescription([128, 256, 128],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3369:       TileDescription([256,  64, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3370:       TileDescription([ 64, 256, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3371:       TileDescription([128, 128, 128],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 3372:       TileDescription([ 64, 128, 128],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3373:     ]
 3374: 
 3375:     data_type_mixed = [math_inst.element_a, math_inst.element_b, math_inst.element_a, DataType.f32]
 3376: 
 3377:     operations = []
 3378: 
 3379:     operations += CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3380:       data_type_mixed, alignment_constraints, None, EpilogueFunctor.LinearCombinationClamp)
 3381: 
 3382:     conv_layout = (LayoutType.TensorNC64HW64, LayoutType.TensorC64RSK64, LayoutType.TensorNC64HW64)
 3383: 
 3384:     operations += CreateConv2dOperator(manifest, conv_layout, tile_descriptions,
 3385:       data_type_mixed, alignment_constraints, [ConvKind.Fprop], EpilogueFunctor.LinearCombinationClamp)
 3386: 
 3387:     for op in operations:
 3388:       op.C.alignment = 16
```
**EN:** Defines `GenerateSM80_TensorOp_16864_Interleaved()`, which generates sm80 tensor op 16864 interleaved. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_16864_Interleaved()`，用于生成 sm80 tensor op 16864 interleaved。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription。

### L3389-L3391 — Comments

```python
 3389: #
 3390: 
 3391: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3392-L3445 — Function `GenerateSM80_TensorOp_168256`

```python
 3392: def GenerateSM80_TensorOp_168256(manifest, cuda_version):
 3393: 
 3394:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3395:     return
 3396: 
 3397:   layouts = [
 3398:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3399:   ]
 3400: 
 3401:   math_instructions = [
 3402:     MathInstruction(                                  \
 3403:       [16, 8, 256],                                   \
 3404:       DataType.b1, DataType.b1, DataType.s32,         \
 3405:       OpcodeClass.TensorOp,                           \
 3406:       MathOperation.xor_popc),
 3407:     MathInstruction(                                  \
 3408:       [16, 8, 256],                                   \
 3409:       DataType.b1, DataType.b1, DataType.s32,         \
 3410:       OpcodeClass.TensorOp,                           \
 3411:       MathOperation.and_popc),
 3412:   ]
 3413: 
 3414:   min_cc = 80
 3415:   max_cc = {
 3416:     MathOperation.xor_popc: 89,
 3417:     MathOperation.and_popc: 90
 3418:   }
 3419: 
 3420:   alignment_constraints = [128,]
 3421: 
 3422:   for math_inst in math_instructions:
 3423:     tile_descriptions = [
 3424:       TileDescription([256, 128,  512],  3, [4, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3425:       TileDescription([128, 256,  512],  3, [2, 4, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3426:       TileDescription([256,  64,  512],  4, [4, 1, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3427:       TileDescription([ 64, 256,  512],  4, [1, 4, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3428:       TileDescription([128, 128,  512],  5, [2, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3429:       TileDescription([128,  64,  512],  6, [2, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3430:       TileDescription([ 64, 128,  512],  6, [2, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3431:       TileDescription([ 64,  64,  512], 10, [2, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3432:       TileDescription([256, 128, 1024],  3, [4, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3433:       TileDescription([128, 256, 1024],  3, [2, 4, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3434:       TileDescription([256,  64, 1024],  4, [4, 1, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3435:       TileDescription([ 64, 256, 1024],  4, [1, 4, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3436:       TileDescription([128, 128, 1024],  4, [2, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3437:       TileDescription([128,  64, 1024],  3, [2, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3438:       TileDescription([ 64, 128, 1024],  3, [2, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3439:       TileDescription([ 64,  64, 1024],  5, [2, 2, 1], math_inst, min_cc, max_cc[math_inst.math_operation]),
 3440:     ]
 3441: 
 3442:     data_type = [DataType.b1, DataType.b1, DataType.s32, DataType.s32]
 3443: 
 3444:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3445:       data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM80_TensorOp_168256()`, which generates sm80 tensor op 168256. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_168256()`，用于生成 sm80 tensor op 168256。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, TileDescription。

### L3446-L3449 — Comments

```python
 3446: 
 3447: #
 3448: 
 3449: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3450-L3521 — Function `GenerateSM80_TensorOp_1688`

```python
 3450: def GenerateSM80_TensorOp_1688(manifest, cuda_version):
 3451: 
 3452:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3453:     return
 3454: 
 3455:   layouts = [
 3456:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3457:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 3458:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3459:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 3460:   ]
 3461: 
 3462:   math_instructions = [
 3463:     MathInstruction(                                      \
 3464:       [16, 8, 8],                                         \
 3465:       DataType.tf32, DataType.tf32, DataType.f32,     \
 3466:       OpcodeClass.TensorOp,                               \
 3467:       MathOperation.multiply_add)
 3468:   ]
 3469: 
 3470:   min_cc = 80
 3471:   max_cc = 1024
 3472: 
 3473:   alignment_constraints = [4, 2, 1]
 3474: 
 3475:   for math_inst in math_instructions:
 3476:     tile_descriptions = [
 3477:       TileDescription([256, 128, 16],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3478:       TileDescription([128, 256, 16],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3479:       TileDescription([256,  64, 16],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3480:       TileDescription([ 64, 256, 16],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3481:       TileDescription([128, 128, 16],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 3482:       TileDescription([128, 128, 16],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3483:       TileDescription([128, 128, 16],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3484:       TileDescription([128,  64, 16],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3485:       TileDescription([ 64, 128, 16],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3486:       TileDescription([ 64,  64, 16], 10, [2, 2, 1], math_inst, min_cc, max_cc),
 3487:       TileDescription([256, 128, 32],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3488:       TileDescription([128, 256, 32],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3489:       TileDescription([256,  64, 32],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3490:       TileDescription([ 64, 256, 32],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3491:       TileDescription([128, 128, 32],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3492:       TileDescription([128, 128, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3493:       TileDescription([128,  64, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3494:       TileDescription([64,  128, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3495:       TileDescription([ 64,  64, 32],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 3496:     ]
 3497: 
 3498:     data_type = [
 3499:       math_inst.element_a,
 3500:       math_inst.element_b,
 3501:       math_inst.element_accumulator,
 3502:       math_inst.element_accumulator,
 3503:     ]
 3504: 
 3505:     data_type_mixed = [
 3506:       math_inst.element_a,
 3507:       math_inst.element_b,
 3508:       math_inst.element_a,
 3509:       math_inst.element_accumulator,
 3510:     ]
 3511: 
 3512:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3513:       data_type, alignment_constraints)
 3514: 
 3515:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3516:       data_type_mixed, alignment_constraints)
 3517: 
 3518:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 3519: 
 3520:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type, alignment_constraints)
 3521:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type_mixed, alignment_constraints)
```
**EN:** Defines `GenerateSM80_TensorOp_1688()`, which generates sm80 tensor op 1688. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_1688()`，用于生成 sm80 tensor op 1688。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription。

### L3522-L3524 — Comments

```python
 3522: #
 3523: 
 3524: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3525-L3589 — Function `GenerateSM80_TensorOp_1688_fast_math`

```python
 3525: def GenerateSM80_TensorOp_1688_fast_math(manifest, cuda_version):
 3526: 
 3527:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3528:     return
 3529: 
 3530:   layouts = [
 3531:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3532:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 3533:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3534:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 3535:   ]
 3536: 
 3537:   math_instructions = [
 3538:     MathInstruction(                                      \
 3539:       [16, 8, 8],                                         \
 3540:       DataType.tf32, DataType.tf32, DataType.f32,     \
 3541:       OpcodeClass.TensorOp,                               \
 3542:       MathOperation.multiply_add),
 3543:     MathInstruction(                                      \
 3544:       [16, 8, 8],                                         \
 3545:       DataType.f16, DataType.f16, DataType.f32,           \
 3546:       OpcodeClass.TensorOp,                               \
 3547:       MathOperation.multiply_add_fast_f16),
 3548:     MathInstruction(                                      \
 3549:       [16, 8, 8],                                         \
 3550:       DataType.bf16, DataType.bf16, DataType.f32,       \
 3551:       OpcodeClass.TensorOp,                               \
 3552:       MathOperation.multiply_add_fast_bf16),
 3553:   ]
 3554: 
 3555:   min_cc = 80
 3556:   max_cc = 1024
 3557: 
 3558:   alignment_constraints = [4, 2, 1]
 3559: 
 3560:   for math_inst in math_instructions:
 3561:     tile_descriptions = [
 3562:       TileDescription([256, 128, 16],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3563:       TileDescription([128, 256, 16],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3564:       TileDescription([256,  64, 16],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3565:       TileDescription([ 64, 256, 16],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3566:       TileDescription([128, 128, 16],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 3567:       TileDescription([128, 128, 16],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3568:       TileDescription([128, 128, 16],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3569:       TileDescription([128,  64, 16],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3570:       TileDescription([ 64, 128, 16],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3571:       TileDescription([ 64,  64, 16], 10, [2, 2, 1], math_inst, min_cc, max_cc),
 3572:       TileDescription([256, 128, 32],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3573:       TileDescription([128, 256, 32],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3574:       TileDescription([256,  64, 32],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3575:       TileDescription([ 64, 256, 32],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3576:       TileDescription([128, 128, 32],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3577:       TileDescription([128, 128, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3578:       TileDescription([128,  64, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3579:       TileDescription([ 64, 128, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3580:       TileDescription([ 64,  64, 32],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 3581:     ]
 3582: 
 3583:     data_type = [DataType.f32, DataType.f32, DataType.f32, DataType.f32]
 3584: 
 3585:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3586:       data_type, alignment_constraints)
 3587: 
 3588:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 3589:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM80_TensorOp_1688_fast_math()`, which generates sm80 tensor op 1688 fast math. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_1688_fast_math()`，用于生成 sm80 tensor op 1688 fast math。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription。

### L3590-L3592 — Comments

```python
 3590: #
 3591: 
 3592: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3593-L3641 — Function `GenerateSM80_TensorOp_1688_fast_fp32_math`

```python
 3593: def GenerateSM80_TensorOp_1688_fast_fp32_math(manifest, cuda_version):
 3594: 
 3595:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3596:     return
 3597: 
 3598:   layouts = [
 3599:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3600:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 3601:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3602:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 3603:   ]
 3604: 
 3605:   math_instructions = [
 3606:     MathInstruction(                                      \
 3607:       [16, 8, 8],                                         \
 3608:       DataType.f32, DataType.f32, DataType.f32,       \
 3609:       OpcodeClass.TensorOp,                               \
 3610:       MathOperation.multiply_add_fast_f32),
 3611:   ]
 3612: 
 3613:   min_cc = 80
 3614:   max_cc = 1024
 3615: 
 3616:   alignment_constraints = [4, 2, 1]
 3617: 
 3618:   for math_inst in math_instructions:
 3619:     tile_descriptions = [
 3620:       TileDescription([128, 128, 16],  4, [4, 2, 1], math_inst, min_cc, max_cc),
 3621:       TileDescription([128, 128, 16],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3622:       TileDescription([256,  64, 16],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3623:       TileDescription([ 64, 256, 16],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3624:       TileDescription([128,  64, 16],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3625:       TileDescription([ 64, 128, 16],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3626:       TileDescription([ 64,  64, 16],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3627:       TileDescription([128, 128, 32],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3628:       TileDescription([256,  64, 32],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3629:       TileDescription([ 64, 256, 32],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3630:       TileDescription([128,  64, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3631:       TileDescription([ 64, 128, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3632:       TileDescription([ 64,  64, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3633:     ]
 3634: 
 3635:     data_type = [DataType.f32, DataType.f32, DataType.f32, DataType.f32]
 3636: 
 3637:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3638:       data_type, alignment_constraints)
 3639: 
 3640:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 3641:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM80_TensorOp_1688_fast_fp32_math()`, which generates sm80 tensor op 1688 fast fp32 math. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_1688_fast_fp32_math()`，用于生成 sm80 tensor op 1688 fast fp32 math。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription。

### L3642-L3643 — Comments

```python
 3642: #
 3643: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3644-L3688 — Function `GenerateSM80_TensorOp_1688_fast_fp32_math_complex`

```python
 3644: def GenerateSM80_TensorOp_1688_fast_fp32_math_complex(manifest, cuda_version):
 3645: 
 3646:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3647:     return
 3648: 
 3649:   layouts = [
 3650:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3651:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 3652:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3653:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 3654:   ]
 3655: 
 3656:   math_inst = MathInstruction(                            \
 3657:       [16, 8, 8],                                         \
 3658:       DataType.f32, DataType.f32, DataType.f32,           \
 3659:       OpcodeClass.TensorOp,                               \
 3660:       MathOperation.multiply_add_complex_fast_f32)
 3661: 
 3662:   min_cc = 80
 3663:   max_cc = 1024
 3664: 
 3665:   tile_descriptions = [
 3666:     TileDescription([128, 64, 16], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 3667:     TileDescription([64, 128, 16], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 3668:     TileDescription([64, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 3669:     TileDescription([64, 32, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 3670:     TileDescription([32, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 3671:     TileDescription([32, 32, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 3672:   ]
 3673: 
 3674:   data_type = [
 3675:     DataType.cf32, DataType.cf32, DataType.cf32, DataType.cf32
 3676:   ]
 3677: 
 3678:   alignment_constraints = [1,]
 3679: 
 3680:   complex_transforms = [
 3681:     (ComplexTransform.none, ComplexTransform.none),
 3682:     (ComplexTransform.conj, ComplexTransform.none),
 3683:     (ComplexTransform.none, ComplexTransform.conj),
 3684:     (ComplexTransform.conj, ComplexTransform.conj)
 3685:   ]
 3686: 
 3687:   CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3688:     data_type, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM80_TensorOp_1688_fast_fp32_math_complex()`, which generates sm80 tensor op 1688 fast fp32 math complex. Key helper calls include MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_1688_fast_fp32_math_complex()`，用于生成 sm80 tensor op 1688 fast fp32 math complex。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L3689-L3691 — Comments

```python
 3689: 
 3690: 
 3691: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3692-L3737 — Function `GenerateSM80_SparseTensorOp_16816_fast_math`

```python
 3692: def GenerateSM80_SparseTensorOp_16816_fast_math(manifest, cuda_version):
 3693: 
 3694:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 1):
 3695:     return
 3696: 
 3697:   layouts = [
 3698:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.RowMajor),
 3699:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.RowMajor),
 3700:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.RowMajor),
 3701:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.RowMajor),
 3702:   ]
 3703: 
 3704:   math_instructions = [
 3705:     MathInstruction(                                      \
 3706:       [16, 8, 16],                                         \
 3707:       DataType.tf32, DataType.tf32, DataType.f32,     \
 3708:       OpcodeClass.TensorOp,                               \
 3709:       MathOperation.multiply_add),
 3710:   ]
 3711: 
 3712:   min_cc = 80
 3713:   max_cc = 1024
 3714: 
 3715:   alignment_constraints = [4]
 3716: 
 3717:   for math_inst in math_instructions:
 3718:     tile_descriptions = [
 3719:       TileDescription([128,  64, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3720:       TileDescription([128, 128, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3721:       TileDescription([256, 128, 32],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3722:       TileDescription([128, 256, 32],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3723:       TileDescription([256,  64, 32],  3, [4, 1, 1], math_inst, min_cc, max_cc),
 3724:       TileDescription([ 64, 256, 32],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3725:       TileDescription([ 64, 128, 32],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3726:       TileDescription([ 64,  64, 32],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3727:       TileDescription([128, 128, 64],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3728:       TileDescription([256,  64, 64],  3, [4, 1, 1], math_inst, min_cc, max_cc),
 3729:       TileDescription([128,  64, 64],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3730:       TileDescription([ 64, 128, 64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3731:       TileDescription([ 64,  64, 64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3732:     ]
 3733: 
 3734:     data_type = [DataType.f32, DataType.f32, DataType.f32, DataType.f32]
 3735: 
 3736:     CreateSparseGemmOperator(manifest, layouts, tile_descriptions, \
 3737:       data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM80_SparseTensorOp_16816_fast_math()`, which generates sm80 sparse tensor op 16816 fast math. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateSparseGemmOperator, TileDescription.

**CN:** 定义 `GenerateSM80_SparseTensorOp_16816_fast_math()`，用于生成 sm80 sparse tensor op 16816 fast math。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateSparseGemmOperator, TileDescription。

### L3738-L3740 — Comments

```python
 3738: #
 3739: 
 3740: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3741-L3786 — Function `GenerateSM80_TensorOp_1688_complex`

```python
 3741: def GenerateSM80_TensorOp_1688_complex(manifest, cuda_version):
 3742: 
 3743:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3744:     return
 3745: 
 3746:   layouts = [
 3747:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3748:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 3749:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3750:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 3751:   ]
 3752: 
 3753:   math_inst = MathInstruction(                  \
 3754:     [16, 8, 8],                                 \
 3755:     DataType.tf32, DataType.tf32, DataType.f32,   \
 3756:     OpcodeClass.TensorOp,                       \
 3757:     MathOperation.multiply_add_complex)
 3758: 
 3759:   min_cc = 80
 3760:   max_cc = 1024
 3761: 
 3762:   tile_descriptions = [
 3763:     TileDescription([128, 128, 16], 4, [2, 4, 1], math_inst, min_cc, max_cc),
 3764:     TileDescription([128, 64, 16], 4, [4, 2, 1], math_inst, min_cc, max_cc),
 3765:     TileDescription([64, 128, 16], 4, [2, 4, 1], math_inst, min_cc, max_cc),
 3766:     TileDescription([64, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 3767:     TileDescription([64, 32, 16], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 3768:     TileDescription([32, 64, 16], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 3769:     TileDescription([32, 32, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 3770:   ]
 3771: 
 3772:   data_type = [
 3773:     DataType.cf32, DataType.cf32, DataType.cf32, DataType.cf32
 3774:   ]
 3775: 
 3776:   alignment_constraints = [1,]
 3777: 
 3778:   complex_transforms = [
 3779:     (ComplexTransform.none, ComplexTransform.none),
 3780:     (ComplexTransform.conj, ComplexTransform.none),
 3781:     (ComplexTransform.none, ComplexTransform.conj),
 3782:     (ComplexTransform.conj, ComplexTransform.conj)
 3783:   ]
 3784: 
 3785:   CreateGemmOperator(manifest, layouts, tile_descriptions, \
 3786:     data_type, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM80_TensorOp_1688_complex()`, which generates sm80 tensor op 1688 complex. Key helper calls include MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_1688_complex()`，用于生成 sm80 tensor op 1688 complex。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L3787-L3789 — Comments

```python
 3787: #
 3788: 
 3789: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3790-L3845 — Function `GenerateSM80_TensorOp_1688_rank_k`

```python
 3790: def GenerateSM80_TensorOp_1688_rank_k(manifest, cuda_version):
 3791: 
 3792:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3793:     return
 3794: 
 3795:   layouts = [
 3796:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3797:     (LayoutType.RowMajor, LayoutType.ColumnMajor),
 3798:   ]
 3799: 
 3800:   fill_modes = [
 3801:     FillMode.Lower, FillMode.Upper,
 3802:   ]
 3803: 
 3804:   math_instructions = [
 3805:       MathInstruction(                                    \
 3806:       [16, 8, 8],                                         \
 3807:       DataType.tf32, DataType.tf32, DataType.f32,         \
 3808:       OpcodeClass.TensorOp,                               \
 3809:       MathOperation.multiply_add),
 3810:       MathInstruction(                                    \
 3811:       [16, 8, 8],                                         \
 3812:       DataType.f32, DataType.f32, DataType.f32,           \
 3813:       OpcodeClass.TensorOp,                               \
 3814:       MathOperation.multiply_add_fast_f32),
 3815:   ]
 3816: 
 3817:   min_cc = 80
 3818:   max_cc = 1024
 3819: 
 3820:   alignment_constraints = [1, 2, 4]  # Alignment only applies to A in SYRK
 3821: 
 3822:   for math_inst in math_instructions:
 3823:     tile_descriptions = [
 3824:       TileDescription([256, 128, 16],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3825:       TileDescription([128, 256, 16],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3826:       #TileDescription([256,  64, 16],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3827:       #TileDescription([ 64, 256, 16],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3828:       TileDescription([128, 128, 16],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 3829:       #TileDescription([128,  64, 16],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3830:       #TileDescription([ 64, 128, 16],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3831:       #TileDescription([ 64,  64, 16], 10, [2, 2, 1], math_inst, min_cc, max_cc),
 3832:       TileDescription([256, 128, 32],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3833:       TileDescription([128, 256, 32],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3834:       #TileDescription([256,  64, 32],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3835:       #TileDescription([ 64, 256, 32],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3836:       TileDescription([128, 128, 32],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3837:       #TileDescription([128,  64, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3838:       #TileDescription([ 64, 128, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3839:       #TileDescription([ 64,  64, 32],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 3840:     ]
 3841: 
 3842:     data_type = [DataType.f32, DataType.f32, DataType.f32]
 3843: 
 3844:     CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, \
 3845:       data_type, alignment_constraints, BlasMode.symmetric)
```
**EN:** Defines `GenerateSM80_TensorOp_1688_rank_k()`, which generates sm80 tensor op 1688 rank k. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateRankKOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_1688_rank_k()`，用于生成 sm80 tensor op 1688 rank k。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateRankKOperator, TileDescription。

### L3846-L3848 — Comments

```python
 3846: #
 3847: 
 3848: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3849-L3900 — Function `GenerateSM80_TensorOp_1688_rank_k_complex`

```python
 3849: def GenerateSM80_TensorOp_1688_rank_k_complex(manifest, cuda_version):
 3850: 
 3851:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3852:     return
 3853: 
 3854:   layouts = [
 3855:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3856:     (LayoutType.RowMajor, LayoutType.ColumnMajor),
 3857:   ]
 3858: 
 3859:   fill_modes = [
 3860:     FillMode.Lower, FillMode.Upper,
 3861:   ]
 3862: 
 3863:   math_instructions = [
 3864:       MathInstruction(                                    \
 3865:       [16, 8, 8],                                         \
 3866:       DataType.tf32, DataType.tf32, DataType.f32,         \
 3867:       OpcodeClass.TensorOp,                               \
 3868:       MathOperation.multiply_add_complex),
 3869:       MathInstruction(                                    \
 3870:       [16, 8, 8],                                         \
 3871:       DataType.f32, DataType.f32, DataType.f32,           \
 3872:       OpcodeClass.TensorOp,                               \
 3873:       MathOperation.multiply_add_complex_fast_f32),
 3874:   ]
 3875: 
 3876:   min_cc = 80
 3877:   max_cc = 1024
 3878: 
 3879:   for math_inst in math_instructions:
 3880:     tile_descriptions = [
 3881:       TileDescription([128, 64, 16], 4, [4, 2, 1], math_inst, min_cc, max_cc),
 3882:       TileDescription([64, 128, 16], 4, [2, 4, 1], math_inst, min_cc, max_cc),
 3883:       TileDescription([64, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 3884:       #TileDescription([64, 32, 16], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 3885:       #TileDescription([32, 32, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 3886:     ]
 3887: 
 3888:     data_type = [
 3889:       DataType.cf32, DataType.cf32, DataType.cf32
 3890:     ]
 3891: 
 3892:     alignment_constraints = [1,]
 3893: 
 3894:     # SYRK
 3895:     CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, \
 3896:       data_type, alignment_constraints, BlasMode.symmetric)
 3897: 
 3898:     # HERK
 3899:     CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, \
 3900:       data_type, alignment_constraints, BlasMode.hermitian)
```
**EN:** Defines `GenerateSM80_TensorOp_1688_rank_k_complex()`, which generates sm80 tensor op 1688 rank k complex. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateRankKOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_1688_rank_k_complex()`，用于生成 sm80 tensor op 1688 rank k complex。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateRankKOperator, TileDescription。

### L3901-L3903 — Comments

```python
 3901: #
 3902: 
 3903: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3904-L3967 — Function `GenerateSM80_TensorOp_1688_trmm`

```python
 3904: def GenerateSM80_TensorOp_1688_trmm(manifest, cuda_version):
 3905: 
 3906:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3907:     return
 3908: 
 3909:   layouts = [
 3910:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3911:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3912:   ]
 3913: 
 3914:   side_modes = [
 3915:     SideMode.Left, SideMode.Right,
 3916:   ]
 3917: 
 3918:   fill_modes = [
 3919:     FillMode.Lower, FillMode.Upper,
 3920:   ]
 3921: 
 3922:   diag_types = [
 3923:     DiagType.NonUnit, DiagType.Unit,
 3924:   ]
 3925: 
 3926:   math_instructions = [
 3927:       MathInstruction(                                    \
 3928:       [16, 8, 8],                                         \
 3929:       DataType.tf32, DataType.tf32, DataType.f32,         \
 3930:       OpcodeClass.TensorOp,                               \
 3931:       MathOperation.multiply_add),
 3932:       MathInstruction(                                    \
 3933:       [16, 8, 8],                                         \
 3934:       DataType.f32, DataType.f32, DataType.f32,           \
 3935:       OpcodeClass.TensorOp,                               \
 3936:       MathOperation.multiply_add_fast_f32),
 3937:   ]
 3938: 
 3939:   min_cc = 80
 3940:   max_cc = 1024
 3941: 
 3942:   alignment_constraints = [1, 2, 4]
 3943: 
 3944:   for math_inst in math_instructions:
 3945:     tile_descriptions = [
 3946:       TileDescription([256, 128, 16],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3947:       TileDescription([128, 256, 16],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3948:       TileDescription([256,  64, 16],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3949:       TileDescription([ 64, 256, 16],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3950:       TileDescription([128, 128, 16],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 3951:       TileDescription([128,  64, 16],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3952:       #TileDescription([ 64, 128, 16],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 3953:       TileDescription([ 64,  64, 16], 10, [2, 2, 1], math_inst, min_cc, max_cc),
 3954:       TileDescription([256, 128, 32],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 3955:       TileDescription([128, 256, 32],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 3956:       #TileDescription([256,  64, 32],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 3957:       #TileDescription([ 64, 256, 32],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 3958:       TileDescription([128, 128, 32],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 3959:       #TileDescription([128,  64, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3960:       #TileDescription([ 64, 128, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 3961:       #TileDescription([ 64,  64, 32],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 3962:     ]
 3963: 
 3964:     data_type = [DataType.f32, DataType.f32, DataType.f32, DataType.f32]
 3965: 
 3966:     CreateTrmmOperator(manifest, layouts, side_modes, fill_modes, diag_types, tile_descriptions, \
 3967:       data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM80_TensorOp_1688_trmm()`, which generates sm80 tensor op 1688 trmm. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateTrmmOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_1688_trmm()`，用于生成 sm80 tensor op 1688 trmm。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateTrmmOperator, TileDescription。

### L3968-L3970 — Comments

```python
 3968: #
 3969: 
 3970: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L3971-L4029 — Function `GenerateSM80_TensorOp_1688_trmm_complex`

```python
 3971: def GenerateSM80_TensorOp_1688_trmm_complex(manifest, cuda_version):
 3972: 
 3973:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 3974:     return
 3975: 
 3976:   layouts = [
 3977:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3978:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 3979:   ]
 3980: 
 3981:   side_modes = [
 3982:     SideMode.Left, SideMode.Right,
 3983:   ]
 3984: 
 3985:   fill_modes = [
 3986:     FillMode.Lower, FillMode.Upper,
 3987:   ]
 3988: 
 3989:   diag_types = [
 3990:     DiagType.NonUnit, DiagType.Unit,
 3991:   ]
 3992: 
 3993:   math_instructions = [
 3994:       MathInstruction(                                    \
 3995:       [16, 8, 8],                                         \
 3996:       DataType.tf32, DataType.tf32, DataType.f32,         \
 3997:       OpcodeClass.TensorOp,                               \
 3998:       MathOperation.multiply_add_complex),
 3999:       MathInstruction(                                    \
 4000:       [16, 8, 8],                                         \
 4001:       DataType.f32, DataType.f32, DataType.f32,           \
 4002:       OpcodeClass.TensorOp,                               \
 4003:       MathOperation.multiply_add_complex_fast_f32),
 4004:   ]
 4005: 
 4006:   min_cc = 80
 4007:   max_cc = 1024
 4008: 
 4009:   for math_inst in math_instructions:
 4010:     tile_descriptions = [
 4011:       TileDescription([128, 64, 16], 4, [4, 2, 1], math_inst, min_cc, max_cc),
 4012:       TileDescription([64, 128, 16], 4, [2, 4, 1], math_inst, min_cc, max_cc),
 4013:       TileDescription([64, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4014:       TileDescription([64, 32, 16], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 4015:       TileDescription([32, 32, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4016:     ]
 4017: 
 4018:     data_type = [
 4019:       DataType.cf32, DataType.cf32, DataType.cf32, DataType.cf32
 4020:     ]
 4021: 
 4022:     alignment_constraints = [1,]
 4023: 
 4024:     complex_transforms = [
 4025:       ComplexTransform.none, ComplexTransform.conj,
 4026:     ]
 4027: 
 4028:     CreateTrmmOperator(manifest, layouts, side_modes, fill_modes, diag_types, tile_descriptions, \
 4029:       data_type, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM80_TensorOp_1688_trmm_complex()`, which generates sm80 tensor op 1688 trmm complex. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateTrmmOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_1688_trmm_complex()`，用于生成 sm80 tensor op 1688 trmm complex。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateTrmmOperator, TileDescription。

### L4030-L4032 — Comments

```python
 4030: #
 4031: 
 4032: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4033-L4094 — Function `GenerateSM80_TensorOp_1688_symm`

```python
 4033: def GenerateSM80_TensorOp_1688_symm(manifest, cuda_version):
 4034: 
 4035:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4036:     return
 4037: 
 4038:   # A and B have same layouts
 4039:   layouts = [
 4040:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4041:   ]
 4042: 
 4043:   side_modes = [
 4044:     SideMode.Left, SideMode.Right,
 4045:   ]
 4046: 
 4047:   fill_modes = [
 4048:     FillMode.Lower, FillMode.Upper,
 4049:   ]
 4050: 
 4051:   math_instructions = [
 4052:       MathInstruction(                                    \
 4053:       [16, 8, 8],                                         \
 4054:       DataType.tf32, DataType.tf32, DataType.f32,         \
 4055:       OpcodeClass.TensorOp,                               \
 4056:       MathOperation.multiply_add),
 4057:       MathInstruction(                                    \
 4058:       [16, 8, 8],                                         \
 4059:       DataType.f32, DataType.f32, DataType.f32,           \
 4060:       OpcodeClass.TensorOp,                               \
 4061:       MathOperation.multiply_add_fast_f32),
 4062:   ]
 4063: 
 4064:   min_cc = 80
 4065:   max_cc = 1024
 4066: 
 4067:   alignment_constraints = [
 4068:     1, 2, 4
 4069:   ]
 4070: 
 4071:   for math_inst in math_instructions:
 4072:     tile_descriptions = [
 4073:       TileDescription([256, 128, 16],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 4074:       TileDescription([128, 256, 16],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 4075:       #TileDescription([256,  64, 16],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 4076:       #TileDescription([ 64, 256, 16],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 4077:       TileDescription([128, 128, 16],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 4078:       #TileDescription([128,  64, 16],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 4079:       #TileDescription([ 64, 128, 16],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 4080:       #TileDescription([ 64,  64, 16], 10, [2, 2, 1], math_inst, min_cc, max_cc),
 4081:       TileDescription([256, 128, 32],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 4082:       TileDescription([128, 256, 32],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 4083:       #TileDescription([256,  64, 32],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 4084:       #TileDescription([ 64, 256, 32],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 4085:       TileDescription([128, 128, 32],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 4086:       #TileDescription([128,  64, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 4087:       #TileDescription([ 64, 128, 32],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 4088:       #TileDescription([ 64,  64, 32],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 4089:     ]
 4090: 
 4091:     data_type = [DataType.f32, DataType.f32, DataType.f32, DataType.f32]
 4092: 
 4093:     CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, \
 4094:       data_type, alignment_constraints, BlasMode.symmetric)
```
**EN:** Defines `GenerateSM80_TensorOp_1688_symm()`, which generates sm80 tensor op 1688 symm. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateSymmOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_1688_symm()`，用于生成 sm80 tensor op 1688 symm。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateSymmOperator, TileDescription。

### L4095-L4097 — Comments

```python
 4095: #
 4096: 
 4097: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4098-L4152 — Function `GenerateSM80_TensorOp_1688_symm_complex`

```python
 4098: def GenerateSM80_TensorOp_1688_symm_complex(manifest, cuda_version):
 4099: 
 4100:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4101:     return
 4102: 
 4103:   layouts = [
 4104:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4105:   ]
 4106: 
 4107:   side_modes = [
 4108:     SideMode.Left, SideMode.Right,
 4109:   ]
 4110: 
 4111:   fill_modes = [
 4112:     FillMode.Lower, FillMode.Upper,
 4113:   ]
 4114: 
 4115:   math_instructions = [
 4116:       MathInstruction(                                    \
 4117:       [16, 8, 8],                                         \
 4118:       DataType.tf32, DataType.tf32, DataType.f32,         \
 4119:       OpcodeClass.TensorOp,                               \
 4120:       MathOperation.multiply_add_complex),
 4121:       MathInstruction(                                    \
 4122:       [16, 8, 8],                                         \
 4123:       DataType.f32, DataType.f32, DataType.f32,           \
 4124:       OpcodeClass.TensorOp,                               \
 4125:       MathOperation.multiply_add_complex_fast_f32),
 4126:   ]
 4127: 
 4128:   min_cc = 80
 4129:   max_cc = 1024
 4130: 
 4131:   for math_inst in math_instructions:
 4132:     tile_descriptions = [
 4133:       TileDescription([128, 64, 16], 4, [4, 2, 1], math_inst, min_cc, max_cc),
 4134:       TileDescription([64, 128, 16], 4, [2, 4, 1], math_inst, min_cc, max_cc),
 4135:       TileDescription([64, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4136:       #TileDescription([64, 32, 16], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 4137:       #TileDescription([32, 32, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4138:     ]
 4139: 
 4140:     data_type = [
 4141:       DataType.cf32, DataType.cf32, DataType.cf32, DataType.cf32
 4142:     ]
 4143: 
 4144:     alignment_constraints = [1,]
 4145: 
 4146:     # SYMM
 4147:     CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, \
 4148:       data_type, alignment_constraints, BlasMode.symmetric)
 4149: 
 4150:     # HEMM
 4151:     CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, \
 4152:       data_type, alignment_constraints, BlasMode.hermitian)
```
**EN:** Defines `GenerateSM80_TensorOp_1688_symm_complex()`, which generates sm80 tensor op 1688 symm complex. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, CreateSymmOperator, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_1688_symm_complex()`，用于生成 sm80 tensor op 1688 symm complex。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, CreateSymmOperator, TileDescription。

### L4153-L4155 — Comments

```python
 4153: #
 4154: 
 4155: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4156-L4199 — Function `GenerateSM80_TensorOp_884`

```python
 4156: def GenerateSM80_TensorOp_884(manifest, cuda_version):
 4157: 
 4158:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4159:     return
 4160: 
 4161:   layouts = [
 4162:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4163:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 4164:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4165:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 4166:   ]
 4167: 
 4168:   math_inst =                                             \
 4169:     MathInstruction(                                      \
 4170:       [8, 8, 4],                                          \
 4171:       DataType.f64, DataType.f64, DataType.f64,           \
 4172:       OpcodeClass.TensorOp,                               \
 4173:       MathOperation.multiply_add)
 4174: 
 4175:   min_cc = 80
 4176:   max_cc = 1024
 4177: 
 4178:   alignment_constraints = [1,]
 4179: 
 4180:   tile_descriptions = [
 4181:     TileDescription([128, 128, 16], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4182:     TileDescription([256, 64, 16], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4183:     TileDescription([64, 256, 16], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 4184:     TileDescription([256, 32, 16], 3, [4, 1, 1], math_inst, min_cc, max_cc),
 4185:     TileDescription([32, 256, 16], 3, [1, 4, 1], math_inst, min_cc, max_cc),
 4186:     TileDescription([128, 64, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4187:     TileDescription([64, 128, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4188:     TileDescription([64, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4189:     TileDescription([64, 32, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4190:     TileDescription([32, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4191:     TileDescription([32, 32, 16], 5, [2, 2, 1], math_inst, min_cc, max_cc),
 4192:     TileDescription([16, 32, 16], 5, [1, 2, 1], math_inst, min_cc, max_cc),
 4193:     TileDescription([32, 16, 16], 5, [2, 1, 1], math_inst, min_cc, max_cc),
 4194:   ]
 4195: 
 4196:   data_type = [DataType.f64, DataType.f64, DataType.f64, DataType.f64]
 4197: 
 4198:   CreateGemmOperator(manifest, layouts, tile_descriptions, \
 4199:     data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM80_TensorOp_884()`, which generates sm80 tensor op 884. Key helper calls include MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_884()`，用于生成 sm80 tensor op 884。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L4200-L4202 — Comments

```python
 4200: #
 4201: 
 4202: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4203-L4256 — Function `GenerateSM80_TensorOp_884_complex`

```python
 4203: def GenerateSM80_TensorOp_884_complex(manifest, cuda_version):
 4204: 
 4205:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4206:     return
 4207: 
 4208:   layouts = [
 4209:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4210:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 4211:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4212:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 4213:   ]
 4214: 
 4215:   math_inst =                                             \
 4216:     MathInstruction(                                      \
 4217:       [8, 8, 4],                                          \
 4218:       DataType.f64, DataType.f64, DataType.f64,           \
 4219:       OpcodeClass.TensorOp,                               \
 4220:       MathOperation.multiply_add_complex)
 4221: 
 4222:   min_cc = 80
 4223:   max_cc = 1024
 4224: 
 4225:   alignment_constraints = [1,]
 4226: 
 4227:   tile_descriptions = [
 4228:     TileDescription([128, 64,  8 ], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4229:     TileDescription([64,  128, 8 ], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 4230:     TileDescription([64,  64,  8 ], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4231:     TileDescription([64,  32,  8 ], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4232:     TileDescription([32,  64,  8 ], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4233:     TileDescription([32,  32,  8 ], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4234:     TileDescription([16,  32,  8 ], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 4235:     TileDescription([32,  16,  8 ], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 4236:     TileDescription([128, 64,  16], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4237:     TileDescription([64,  128, 16], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 4238:     TileDescription([64,  64,  16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4239:     TileDescription([64,  32,  16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4240:     TileDescription([32,  64,  16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4241:     TileDescription([32,  32,  16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4242:     TileDescription([16,  32,  16], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 4243:     TileDescription([32,  16,  16], 3, [2, 1, 1], math_inst, min_cc, max_cc),
 4244:   ]
 4245: 
 4246:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64, DataType.cf64]
 4247: 
 4248:   complex_transforms = [
 4249:     (ComplexTransform.none, ComplexTransform.none),
 4250:     (ComplexTransform.conj, ComplexTransform.none),
 4251:     (ComplexTransform.none, ComplexTransform.conj),
 4252:     (ComplexTransform.conj, ComplexTransform.conj)
 4253:   ]
 4254: 
 4255:   CreateGemmOperator(manifest, layouts, tile_descriptions, \
 4256:     data_type, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM80_TensorOp_884_complex()`, which generates sm80 tensor op 884 complex. Key helper calls include MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_884_complex()`，用于生成 sm80 tensor op 884 complex。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L4257-L4258 — Comments

```python
 4257: 
 4258: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4259-L4302 — Function `GenerateSM80_TensorOp_884_complex_gaussian`

```python
 4259: def GenerateSM80_TensorOp_884_complex_gaussian(manifest, cuda_version):
 4260: 
 4261:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4262:     return
 4263: 
 4264:   layouts = [
 4265:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4266:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 4267:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4268:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 4269:   ]
 4270: 
 4271:   math_inst =                                             \
 4272:     MathInstruction(                                      \
 4273:       [8, 8, 4],                                          \
 4274:       DataType.f64, DataType.f64, DataType.f64,           \
 4275:       OpcodeClass.TensorOp,                               \
 4276:       MathOperation.multiply_add_complex_gaussian)
 4277: 
 4278:   min_cc = 80
 4279:   max_cc = 1024
 4280: 
 4281:   alignment_constraints = [1,]
 4282: 
 4283:   tile_descriptions = [
 4284:     TileDescription([64, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4285:     TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4286:     TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4287:     TileDescription([32, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4288:     TileDescription([16, 32, 8], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 4289:     TileDescription([32, 16, 8], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 4290:   ]
 4291: 
 4292:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64, DataType.cf64]
 4293: 
 4294:   complex_transforms = [
 4295:     (ComplexTransform.none, ComplexTransform.none),
 4296:     (ComplexTransform.conj, ComplexTransform.none),
 4297:     (ComplexTransform.none, ComplexTransform.conj),
 4298:     (ComplexTransform.conj, ComplexTransform.conj)
 4299:   ]
 4300: 
 4301:   CreateGemmOperator(manifest, layouts, tile_descriptions, \
 4302:     data_type, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM80_TensorOp_884_complex_gaussian()`, which generates sm80 tensor op 884 complex gaussian. Key helper calls include MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_884_complex_gaussian()`，用于生成 sm80 tensor op 884 complex gaussian。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L4303-L4305 — Comments

```python
 4303: #
 4304: 
 4305: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4306-L4347 — Function `GenerateSM80_TensorOp_884_rank_k`

```python
 4306: def GenerateSM80_TensorOp_884_rank_k(manifest, cuda_version):
 4307: 
 4308:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4309:     return
 4310: 
 4311:   layouts = [
 4312:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4313:     (LayoutType.RowMajor, LayoutType.ColumnMajor),
 4314:   ]
 4315: 
 4316:   fill_modes = [
 4317:     FillMode.Lower, FillMode.Upper,
 4318:   ]
 4319: 
 4320:   math_inst =                                             \
 4321:     MathInstruction(                                      \
 4322:       [8, 8, 4],                                          \
 4323:       DataType.f64, DataType.f64, DataType.f64,           \
 4324:       OpcodeClass.TensorOp,                               \
 4325:       MathOperation.multiply_add)
 4326: 
 4327:   min_cc = 80
 4328:   max_cc = 1024
 4329: 
 4330:   alignment_constraints = [1,]
 4331: 
 4332:   tile_descriptions = [
 4333:     TileDescription([128, 128, 16], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4334:     TileDescription([64, 128, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4335:     TileDescription([128, 64, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4336:     TileDescription([64, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4337:     TileDescription([64, 32, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4338:     TileDescription([32, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4339:     TileDescription([32, 32, 16], 5, [2, 2, 1], math_inst, min_cc, max_cc),
 4340:     TileDescription([16, 32, 16], 5, [1, 2, 1], math_inst, min_cc, max_cc),
 4341:     TileDescription([32, 16, 16], 5, [2, 1, 1], math_inst, min_cc, max_cc),
 4342:   ]
 4343: 
 4344:   data_type = [DataType.f64, DataType.f64, DataType.f64]
 4345: 
 4346:   CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, \
 4347:     data_type, alignment_constraints, BlasMode.symmetric)
```
**EN:** Defines `GenerateSM80_TensorOp_884_rank_k()`, which generates sm80 tensor op 884 rank k. Key helper calls include MathInstruction, CreateRankKOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_884_rank_k()`，用于生成 sm80 tensor op 884 rank k。 其中会调用的重要辅助函数包括 MathInstruction, CreateRankKOperator, CudaToolkitVersionSatisfies, TileDescription。

### L4348-L4350 — Comments

```python
 4348: #
 4349: 
 4350: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4351-L4396 — Function `GenerateSM80_TensorOp_884_rank_k_complex`

```python
 4351: def GenerateSM80_TensorOp_884_rank_k_complex(manifest, cuda_version):
 4352: 
 4353:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4354:     return
 4355: 
 4356:   layouts = [
 4357:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4358:     (LayoutType.RowMajor, LayoutType.ColumnMajor),
 4359:   ]
 4360: 
 4361:   fill_modes = [
 4362:     FillMode.Lower, FillMode.Upper,
 4363:   ]
 4364: 
 4365:   math_inst =                                             \
 4366:     MathInstruction(                                      \
 4367:       [8, 8, 4],                                          \
 4368:       DataType.f64, DataType.f64, DataType.f64,           \
 4369:       OpcodeClass.TensorOp,                               \
 4370:       MathOperation.multiply_add_complex)
 4371: 
 4372:   min_cc = 80
 4373:   max_cc = 1024
 4374: 
 4375:   alignment_constraints = [1,]
 4376: 
 4377:   tile_descriptions = [
 4378:     TileDescription([128, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4379:     TileDescription([64, 128, 8], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 4380:     TileDescription([64, 64, 8], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4381:     #TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4382:     #TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4383:     #TileDescription([32, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4384:     #TileDescription([16, 32, 8], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 4385:     #TileDescription([32, 16, 8], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 4386:   ]
 4387: 
 4388:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64]
 4389: 
 4390:   # SYRK computation
 4391:   CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, \
 4392:     data_type, alignment_constraints, BlasMode.symmetric)
 4393: 
 4394:   # HERK computation
 4395:   CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, \
 4396:     data_type, alignment_constraints, BlasMode.hermitian)
```
**EN:** Defines `GenerateSM80_TensorOp_884_rank_k_complex()`, which generates sm80 tensor op 884 rank k complex. Key helper calls include MathInstruction, CreateRankKOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_884_rank_k_complex()`，用于生成 sm80 tensor op 884 rank k complex。 其中会调用的重要辅助函数包括 MathInstruction, CreateRankKOperator, CudaToolkitVersionSatisfies, TileDescription。

### L4397-L4400 — Comments

```python
 4397: 
 4398: #
 4399: 
 4400: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4401-L4446 — Function `GenerateSM80_TensorOp_884_rank_k_complex_gaussian`

```python
 4401: def GenerateSM80_TensorOp_884_rank_k_complex_gaussian(manifest, cuda_version):
 4402: 
 4403:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4404:     return
 4405: 
 4406:   layouts = [
 4407:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4408:     (LayoutType.RowMajor, LayoutType.ColumnMajor),
 4409:   ]
 4410: 
 4411:   fill_modes = [
 4412:     FillMode.Lower, FillMode.Upper,
 4413:   ]
 4414: 
 4415:   math_inst =                                             \
 4416:     MathInstruction(                                      \
 4417:       [8, 8, 4],                                          \
 4418:       DataType.f64, DataType.f64, DataType.f64,           \
 4419:       OpcodeClass.TensorOp,                               \
 4420:       MathOperation.multiply_add_complex_gaussian)
 4421: 
 4422:   min_cc = 80
 4423:   max_cc = 1024
 4424: 
 4425:   alignment_constraints = [1,]
 4426: 
 4427:   tile_descriptions = [
 4428:     TileDescription([64, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4429:     TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4430:     TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4431:     #TileDescription([32, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4432:     #TileDescription([16, 32, 8], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 4433:     #TileDescription([32, 16, 8], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 4434:   ]
 4435: 
 4436:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64]
 4437: 
 4438:   complex_transforms = [ComplexTransform.none,]
 4439: 
 4440:   # SYRK computation
 4441:   CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, \
 4442:     data_type, alignment_constraints, BlasMode.symmetric)
 4443: 
 4444:   # HERK computation
 4445:   CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, \
 4446:     data_type, alignment_constraints, BlasMode.hermitian)
```
**EN:** Defines `GenerateSM80_TensorOp_884_rank_k_complex_gaussian()`, which generates sm80 tensor op 884 rank k complex gaussian. Key helper calls include MathInstruction, CreateRankKOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_884_rank_k_complex_gaussian()`，用于生成 sm80 tensor op 884 rank k complex gaussian。 其中会调用的重要辅助函数包括 MathInstruction, CreateRankKOperator, CudaToolkitVersionSatisfies, TileDescription。

### L4447-L4449 — Comments

```python
 4447: #
 4448: 
 4449: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4450-L4494 — Function `GenerateSM80_TensorOp_884_trmm`

```python
 4450: def GenerateSM80_TensorOp_884_trmm(manifest, cuda_version):
 4451: 
 4452:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4453:     return
 4454: 
 4455:   layouts = [
 4456:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4457:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4458:   ]
 4459: 
 4460:   side_modes = [
 4461:     SideMode.Left, SideMode.Right,
 4462:   ]
 4463: 
 4464:   fill_modes = [
 4465:     FillMode.Lower, FillMode.Upper,
 4466:   ]
 4467: 
 4468:   diag_types = [
 4469:     DiagType.NonUnit, DiagType.Unit,
 4470:   ]
 4471: 
 4472:   math_inst =                                             \
 4473:     MathInstruction(                                      \
 4474:       [8, 8, 4],                                          \
 4475:       DataType.f64, DataType.f64, DataType.f64,           \
 4476:       OpcodeClass.TensorOp,                               \
 4477:       MathOperation.multiply_add)
 4478: 
 4479:   min_cc = 80
 4480:   max_cc = 1024
 4481: 
 4482:   alignment_constraints = [1,]
 4483: 
 4484:   tile_descriptions = [
 4485:     TileDescription([128, 128, 16], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4486:     TileDescription([64, 128, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4487:     TileDescription([128, 64, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4488:     TileDescription([64, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4489:   ]
 4490: 
 4491:   data_type = [DataType.f64, DataType.f64, DataType.f64, DataType.f64]
 4492: 
 4493:   CreateTrmmOperator(manifest, layouts, side_modes, fill_modes, diag_types, tile_descriptions, \
 4494:     data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM80_TensorOp_884_trmm()`, which generates sm80 tensor op 884 trmm. Key helper calls include MathInstruction, CreateTrmmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_884_trmm()`，用于生成 sm80 tensor op 884 trmm。 其中会调用的重要辅助函数包括 MathInstruction, CreateTrmmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L4495-L4497 — Comments

```python
 4495: #
 4496: 
 4497: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4498-L4547 — Function `GenerateSM80_TensorOp_884_trmm_complex`

```python
 4498: def GenerateSM80_TensorOp_884_trmm_complex(manifest, cuda_version):
 4499: 
 4500:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4501:     return
 4502: 
 4503:   layouts = [
 4504:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4505:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4506:   ]
 4507: 
 4508:   side_modes = [
 4509:     SideMode.Left, SideMode.Right,
 4510:   ]
 4511: 
 4512:   fill_modes = [
 4513:     FillMode.Lower, FillMode.Upper,
 4514:   ]
 4515: 
 4516:   diag_types = [
 4517:     DiagType.NonUnit, DiagType.Unit,
 4518:   ]
 4519: 
 4520:   math_inst =                                             \
 4521:     MathInstruction(                                      \
 4522:       [8, 8, 4],                                          \
 4523:       DataType.f64, DataType.f64, DataType.f64,           \
 4524:       OpcodeClass.TensorOp,                               \
 4525:       MathOperation.multiply_add_complex)
 4526: 
 4527:   min_cc = 80
 4528:   max_cc = 1024
 4529: 
 4530:   alignment_constraints = [1,]
 4531: 
 4532:   tile_descriptions = [
 4533:     TileDescription([128, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4534:     TileDescription([64, 128, 8], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 4535:     TileDescription([64, 64, 8], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4536:     TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4537:     TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4538:   ]
 4539: 
 4540:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64, DataType.cf64]
 4541: 
 4542:   complex_transforms = [
 4543:     ComplexTransform.none, ComplexTransform.conj,
 4544:   ]
 4545: 
 4546:   CreateTrmmOperator(manifest, layouts, side_modes, fill_modes, diag_types, tile_descriptions, \
 4547:     data_type, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM80_TensorOp_884_trmm_complex()`, which generates sm80 tensor op 884 trmm complex. Key helper calls include MathInstruction, CreateTrmmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_884_trmm_complex()`，用于生成 sm80 tensor op 884 trmm complex。 其中会调用的重要辅助函数包括 MathInstruction, CreateTrmmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L4548-L4551 — Comments

```python
 4548: #
 4549: 
 4550: 
 4551: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4552-L4599 — Function `GenerateSM80_TensorOp_884_trmm_complex_gaussian`

```python
 4552: def GenerateSM80_TensorOp_884_trmm_complex_gaussian(manifest, cuda_version):
 4553: 
 4554:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4555:     return
 4556: 
 4557:   layouts = [
 4558:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4559:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4560:   ]
 4561: 
 4562:   side_modes = [
 4563:     SideMode.Left, SideMode.Right,
 4564:   ]
 4565: 
 4566:   fill_modes = [
 4567:     FillMode.Lower, FillMode.Upper,
 4568:   ]
 4569: 
 4570:   diag_types = [
 4571:     DiagType.NonUnit, DiagType.Unit,
 4572:   ]
 4573: 
 4574:   math_inst =                                             \
 4575:     MathInstruction(                                      \
 4576:       [8, 8, 4],                                          \
 4577:       DataType.f64, DataType.f64, DataType.f64,           \
 4578:       OpcodeClass.TensorOp,                               \
 4579:       MathOperation.multiply_add_complex_gaussian)
 4580: 
 4581:   min_cc = 80
 4582:   max_cc = 1024
 4583: 
 4584:   alignment_constraints = [1,]
 4585: 
 4586:   tile_descriptions = [
 4587:     TileDescription([64, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4588:     TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4589:     TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4590:   ]
 4591: 
 4592:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64, DataType.cf64]
 4593: 
 4594:   complex_transforms = [
 4595:     ComplexTransform.none, ComplexTransform.conj,
 4596:   ]
 4597: 
 4598:   CreateTrmmOperator(manifest, layouts, side_modes, fill_modes, diag_types, tile_descriptions, \
 4599:     data_type, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM80_TensorOp_884_trmm_complex_gaussian()`, which generates sm80 tensor op 884 trmm complex gaussian. Key helper calls include MathInstruction, CreateTrmmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_884_trmm_complex_gaussian()`，用于生成 sm80 tensor op 884 trmm complex gaussian。 其中会调用的重要辅助函数包括 MathInstruction, CreateTrmmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L4600-L4602 — Comments

```python
 4600: #
 4601: 
 4602: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4603-L4647 — Function `GenerateSM80_TensorOp_884_symm`

```python
 4603: def GenerateSM80_TensorOp_884_symm(manifest, cuda_version):
 4604: 
 4605:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4606:     return
 4607: 
 4608:   layouts = [
 4609:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4610:   ]
 4611: 
 4612:   side_modes = [
 4613:     SideMode.Left, SideMode.Right,
 4614:   ]
 4615: 
 4616:   fill_modes = [
 4617:     FillMode.Lower, FillMode.Upper,
 4618:   ]
 4619: 
 4620:   math_inst =                                             \
 4621:     MathInstruction(                                      \
 4622:       [8, 8, 4],                                          \
 4623:       DataType.f64, DataType.f64, DataType.f64,           \
 4624:       OpcodeClass.TensorOp,                               \
 4625:       MathOperation.multiply_add)
 4626: 
 4627:   min_cc = 80
 4628:   max_cc = 1024
 4629: 
 4630:   alignment_constraints = [1,]
 4631: 
 4632:   tile_descriptions = [
 4633:     TileDescription([128, 128, 16], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4634:     TileDescription([64, 128, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4635:     TileDescription([128, 64, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4636:     TileDescription([64, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4637:     TileDescription([64, 32, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4638:     TileDescription([32, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4639:     TileDescription([32, 32, 16], 5, [2, 2, 1], math_inst, min_cc, max_cc),
 4640:     TileDescription([16, 32, 16], 5, [1, 2, 1], math_inst, min_cc, max_cc),
 4641:     TileDescription([32, 16, 16], 5, [2, 1, 1], math_inst, min_cc, max_cc),
 4642:   ]
 4643: 
 4644:   data_type = [DataType.f64, DataType.f64, DataType.f64, DataType.f64]
 4645: 
 4646:   CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, \
 4647:     data_type, alignment_constraints, BlasMode.symmetric)
```
**EN:** Defines `GenerateSM80_TensorOp_884_symm()`, which generates sm80 tensor op 884 symm. Key helper calls include MathInstruction, CreateSymmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_884_symm()`，用于生成 sm80 tensor op 884 symm。 其中会调用的重要辅助函数包括 MathInstruction, CreateSymmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L4648-L4650 — Comments

```python
 4648: #
 4649: 
 4650: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4651-L4699 — Function `GenerateSM80_TensorOp_884_symm_complex`

```python
 4651: def GenerateSM80_TensorOp_884_symm_complex(manifest, cuda_version):
 4652: 
 4653:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4654:     return
 4655: 
 4656:   layouts = [
 4657:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4658:   ]
 4659: 
 4660:   side_modes = [
 4661:     SideMode.Left, SideMode.Right,
 4662:   ]
 4663: 
 4664:   fill_modes = [
 4665:     FillMode.Lower, FillMode.Upper,
 4666:   ]
 4667: 
 4668:   math_inst =                                             \
 4669:     MathInstruction(                                      \
 4670:       [8, 8, 4],                                          \
 4671:       DataType.f64, DataType.f64, DataType.f64,           \
 4672:       OpcodeClass.TensorOp,                               \
 4673:       MathOperation.multiply_add_complex)
 4674: 
 4675:   min_cc = 80
 4676:   max_cc = 1024
 4677: 
 4678:   alignment_constraints = [1,]
 4679: 
 4680:   tile_descriptions = [
 4681:     TileDescription([128, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4682:     TileDescription([64, 128, 8], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 4683:     TileDescription([64, 64, 8], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 4684:     #TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4685:     #TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4686:     #TileDescription([32, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4687:     #TileDescription([16, 32, 8], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 4688:     #TileDescription([32, 16, 8], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 4689:   ]
 4690: 
 4691:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64, DataType.cf64]
 4692: 
 4693:   # SYMM computation
 4694:   CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, \
 4695:     data_type, alignment_constraints, BlasMode.symmetric)
 4696: 
 4697:   # HEMM computation
 4698:   CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, \
 4699:     data_type, alignment_constraints, BlasMode.hermitian)
```
**EN:** Defines `GenerateSM80_TensorOp_884_symm_complex()`, which generates sm80 tensor op 884 symm complex. Key helper calls include MathInstruction, CreateSymmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_884_symm_complex()`，用于生成 sm80 tensor op 884 symm complex。 其中会调用的重要辅助函数包括 MathInstruction, CreateSymmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L4700-L4702 — Comments

```python
 4700: #
 4701: 
 4702: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4703-L4751 — Function `GenerateSM80_TensorOp_884_symm_complex_gaussian`

```python
 4703: def GenerateSM80_TensorOp_884_symm_complex_gaussian(manifest, cuda_version):
 4704: 
 4705:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 0):
 4706:     return
 4707: 
 4708:   layouts = [
 4709:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4710:   ]
 4711: 
 4712:   side_modes = [
 4713:     SideMode.Left, SideMode.Right,
 4714:   ]
 4715: 
 4716:   fill_modes = [
 4717:     FillMode.Lower, FillMode.Upper,
 4718:   ]
 4719: 
 4720:   math_inst =                                             \
 4721:     MathInstruction(                                      \
 4722:       [8, 8, 4],                                          \
 4723:       DataType.f64, DataType.f64, DataType.f64,           \
 4724:       OpcodeClass.TensorOp,                               \
 4725:       MathOperation.multiply_add_complex_gaussian)
 4726: 
 4727:   min_cc = 80
 4728:   max_cc = 1024
 4729: 
 4730:   alignment_constraints = [1,]
 4731: 
 4732:   tile_descriptions = [
 4733:     TileDescription([64, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4734:     TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4735:     TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4736:     #TileDescription([32, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4737:     #TileDescription([16, 32, 8], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 4738:     #TileDescription([32, 16, 8], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 4739:   ]
 4740: 
 4741:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64, DataType.cf64]
 4742: 
 4743:   complex_transforms = [ComplexTransform.none,]
 4744: 
 4745:   # SYMM computation
 4746:   CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, \
 4747:     data_type, alignment_constraints, BlasMode.symmetric)
 4748: 
 4749:   # HEMM computation
 4750:   CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, \
 4751:     data_type, alignment_constraints, BlasMode.hermitian)
```
**EN:** Defines `GenerateSM80_TensorOp_884_symm_complex_gaussian()`, which generates sm80 tensor op 884 symm complex gaussian. Key helper calls include MathInstruction, CreateSymmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM80_TensorOp_884_symm_complex_gaussian()`，用于生成 sm80 tensor op 884 symm complex gaussian。 其中会调用的重要辅助函数包括 MathInstruction, CreateSymmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L4752-L4756 — Comments

```python
 4752: #
 4753: 
 4754: ###################################################################################################
 4755: 
 4756: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4757-L4804 — Function `GenerateSM80_Simt_f32`

```python
 4757: def GenerateSM80_Simt_f32(manifest, cuda_version):
 4758:   layouts = [
 4759:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4760:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 4761:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4762:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 4763:   ]
 4764: 
 4765:   math_instructions = [
 4766:     MathInstruction(                                  \
 4767:       [1, 1, 1],                                      \
 4768:       DataType.f32, DataType.f32, DataType.f32,       \
 4769:       OpcodeClass.Simt,                               \
 4770:       MathOperation.multiply_add),
 4771:   ]
 4772: 
 4773:   min_cc = 80
 4774:   max_cc = 1024
 4775: 
 4776:   alignment_constraints = [1,]
 4777: 
 4778:   for math_inst in math_instructions:
 4779:     tile_descriptions = [
 4780:       TileDescription([256, 128, 8], 5, [4, 2, 1], math_inst, min_cc, max_cc),
 4781:       TileDescription([128, 256, 8], 5, [2, 4, 1], math_inst, min_cc, max_cc),
 4782:       TileDescription([128, 128, 8], 5, [4, 2, 1], math_inst, min_cc, max_cc),
 4783:       TileDescription([256, 128, 8], 4, [4, 2, 1], math_inst, min_cc, max_cc),
 4784:       TileDescription([128, 256, 8], 4, [2, 4, 1], math_inst, min_cc, max_cc),
 4785:       TileDescription([128, 128, 8], 4, [4, 2, 1], math_inst, min_cc, max_cc),
 4786:       TileDescription([128,  64, 8], 5, [2, 2, 1], math_inst, min_cc, max_cc),
 4787:       TileDescription([ 64, 128, 8], 5, [2, 2, 1], math_inst, min_cc, max_cc),
 4788:       TileDescription([ 64,  64, 8], 5, [2, 1, 1], math_inst, min_cc, max_cc),
 4789:       TileDescription([128,  32, 8], 5, [2, 1, 1], math_inst, min_cc, max_cc),
 4790:       TileDescription([ 32, 128, 8], 5, [1, 2, 1], math_inst, min_cc, max_cc),
 4791:     ]
 4792: 
 4793:     data_type = [
 4794:       math_inst.element_a,
 4795:       math_inst.element_b,
 4796:       math_inst.element_accumulator,
 4797:       math_inst.element_accumulator,
 4798:     ]
 4799: 
 4800:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 4801:       data_type, alignment_constraints)
 4802: 
 4803:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 4804:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM80_Simt_f32()`, which generates sm80 simt f32. Key helper calls include MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription.

**CN:** 定义 `GenerateSM80_Simt_f32()`，用于生成 sm80 simt f32。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription。

### L4805-L4808 — Comments

```python
 4805: #
 4806: 
 4807: 
 4808: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4809-L4848 — Function `GenerateSM80_Simt_f64`

```python
 4809: def GenerateSM80_Simt_f64(manifest, cuda_version):
 4810:   layouts = [
 4811:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4812:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 4813:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4814:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 4815:   ]
 4816: 
 4817:   math_instructions = [
 4818:     MathInstruction(                                  \
 4819:       [1, 1, 1],                                      \
 4820:       DataType.f64, DataType.f64, DataType.f64,       \
 4821:       OpcodeClass.Simt,                               \
 4822:       MathOperation.multiply_add),
 4823:   ]
 4824: 
 4825:   min_cc = 80
 4826:   max_cc = 1024
 4827: 
 4828:   alignment_constraints = [1,]
 4829: 
 4830:   for math_inst in math_instructions:
 4831:     tile_descriptions = [
 4832:       TileDescription([128, 128, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4833:       TileDescription([128,  64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4834:       TileDescription([ 64, 128, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4835:       TileDescription([ 64,  64, 8], 5, [2, 1, 1], math_inst, min_cc, max_cc),
 4836:       TileDescription([128,  32, 8], 5, [2, 1, 1], math_inst, min_cc, max_cc),
 4837:       TileDescription([ 32, 128, 8], 5, [1, 2, 1], math_inst, min_cc, max_cc),
 4838:     ]
 4839: 
 4840:     data_type = [
 4841:       math_inst.element_a,
 4842:       math_inst.element_b,
 4843:       math_inst.element_accumulator,
 4844:       math_inst.element_accumulator,
 4845:     ]
 4846: 
 4847:     CreateGemmOperator(manifest, layouts, tile_descriptions, \
 4848:       data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM80_Simt_f64()`, which generates sm80 simt f64. Key helper calls include MathInstruction, CreateGemmOperator, TileDescription.

**CN:** 定义 `GenerateSM80_Simt_f64()`，用于生成 sm80 simt f64。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, TileDescription。

### L4849-L4853 — Comments

```python
 4849: #
 4850: 
 4851: 
 4852: ##################################################################################################
 4853: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4854-L4905 — Function `GenerateSM80_Simt_complex`

```python
 4854: def GenerateSM80_Simt_complex(manifest, cuda_version):
 4855:   math_instructions = [
 4856:     MathInstruction(                                  \
 4857:       [1, 1, 1],                                      \
 4858:       DataType.f32, DataType.f32, DataType.f32,       \
 4859:       OpcodeClass.Simt,                               \
 4860:       MathOperation.multiply_add_complex),
 4861:   ]
 4862: 
 4863:   min_cc = 80
 4864:   max_cc = 1024
 4865: 
 4866:   alignment_constraints = [1,]
 4867: 
 4868:   data_type = [
 4869:     DataType.cf32,
 4870:     DataType.cf32,
 4871:     DataType.cf32,
 4872:     DataType.cf32
 4873:   ]
 4874: 
 4875:   layouts = [
 4876:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4877:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 4878:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4879:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 4880:   ]
 4881: 
 4882:   complex_transforms = [
 4883:     (ComplexTransform.none, ComplexTransform.none),
 4884:     (ComplexTransform.conj, ComplexTransform.none),
 4885:     (ComplexTransform.none, ComplexTransform.conj),
 4886:     (ComplexTransform.conj, ComplexTransform.conj)
 4887:   ]
 4888: 
 4889:   for math_inst in math_instructions:
 4890: 
 4891:     tile_descriptions = [
 4892:       TileDescription([128, 128, 8], 5, [4, 2, 1], math_inst, min_cc, max_cc),
 4893:       TileDescription([128, 128, 8], 4, [4, 2, 1], math_inst, min_cc, max_cc),
 4894:       TileDescription([64, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 4895:       TileDescription([ 64, 128, 16],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 4896:       TileDescription([128,  64, 16],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 4897:       TileDescription([64, 32, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4898:       TileDescription([32, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 4899:       TileDescription([32, 32, 16], 5, [2, 2, 1], math_inst, min_cc, max_cc),
 4900:     ]
 4901: 
 4902:     CreateGemmOperator(manifest, layouts, tile_descriptions, data_type, alignment_constraints, complex_transforms)
 4903: 
 4904:     conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 4905:     CreateConv2dOperator(manifest, conv_layout, tile_descriptions, data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM80_Simt_complex()`, which generates sm80 simt complex. Key helper calls include MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription.

**CN:** 定义 `GenerateSM80_Simt_complex()`，用于生成 sm80 simt complex。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, CreateConv2dOperator, TileDescription。

### L4906-L4910 — Comments

```python
 4906: #
 4907: 
 4908: ###################################################################################################
 4909: 
 4910: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4911-L4953 — Function `GenerateSM80`

```python
 4911: def GenerateSM80(manifest, cuda_version):
 4912:   GenerateSM80_TensorOp_16816(manifest, cuda_version)
 4913:   GenerateSM80_SparseTensorOp_16832(manifest, cuda_version)
 4914:   GenerateSM80_PlanarComplexTensorOp_16816(manifest, cuda_version)
 4915:   GenerateSM80_TensorOp_1688(manifest, cuda_version)
 4916:   GenerateSM80_TensorOp_1688_fast_math(manifest, cuda_version)
 4917:   GenerateSM80_SparseTensorOp_16816_fast_math(manifest, cuda_version)
 4918:   GenerateSM80_TensorOp_1688_complex(manifest, cuda_version)
 4919:   # 3xTF32
 4920:   GenerateSM80_TensorOp_1688_fast_fp32_math(manifest, cuda_version)
 4921:   GenerateSM80_TensorOp_1688_fast_fp32_math_complex(manifest, cuda_version)
 4922:   GenerateSM80_TensorOp_1688_rank_k(manifest, cuda_version)
 4923:   GenerateSM80_TensorOp_1688_rank_k_complex(manifest, cuda_version)
 4924:   GenerateSM80_TensorOp_1688_trmm(manifest, cuda_version)
 4925:   GenerateSM80_TensorOp_1688_trmm_complex(manifest, cuda_version)
 4926:   GenerateSM80_TensorOp_1688_symm(manifest, cuda_version)
 4927:   GenerateSM80_TensorOp_1688_symm_complex(manifest, cuda_version)
 4928:   GenerateSM80_TensorOp_884(manifest, cuda_version)
 4929:   GenerateSM80_TensorOp_884_complex(manifest, cuda_version)
 4930:   GenerateSM80_TensorOp_884_complex_gaussian(manifest, cuda_version)
 4931:   GenerateSM80_TensorOp_884_rank_k(manifest, cuda_version)
 4932:   GenerateSM80_TensorOp_884_rank_k_complex(manifest, cuda_version)
 4933:   GenerateSM80_TensorOp_884_rank_k_complex_gaussian(manifest, cuda_version)
 4934:   GenerateSM80_TensorOp_884_trmm(manifest, cuda_version)
 4935:   GenerateSM80_TensorOp_884_trmm_complex(manifest, cuda_version)
 4936:   GenerateSM80_TensorOp_884_trmm_complex_gaussian(manifest, cuda_version)
 4937:   GenerateSM80_TensorOp_884_symm(manifest, cuda_version)
 4938:   GenerateSM80_TensorOp_884_symm_complex(manifest, cuda_version)
 4939:   GenerateSM80_TensorOp_884_symm_complex_gaussian(manifest, cuda_version)
 4940:   GenerateSM80_TensorOp_16816_mixed_input_upcast_a(manifest, cuda_version)
 4941:   GenerateSM80_TensorOp_16816_mixed_input_upcast_b(manifest, cuda_version)
 4942:   GenerateSM80_TensorOp_16832_TN(manifest, cuda_version)
 4943:   GenerateSM80_TensorOp_16832_TN_mixed_input_upcast_a(manifest, cuda_version)
 4944:   GenerateSM80_TensorOp_16832_TN_mixed_input_upcast_b(manifest, cuda_version)
 4945:   GenerateSM80_SparseTensorOp_16864_TN(manifest, cuda_version)
 4946:   GenerateSM80_TensorOp_16832_Interleaved(manifest, cuda_version)
 4947:   GenerateSM80_TensorOp_16864_TN(manifest, cuda_version)
 4948:   GenerateSM80_SparseTensorOp_168128_TN(manifest, cuda_version)
 4949:   GenerateSM80_TensorOp_16864_Interleaved(manifest, cuda_version)
 4950:   GenerateSM80_TensorOp_168256(manifest, cuda_version)
 4951:   GenerateSM80_Simt_f32(manifest, cuda_version)
 4952:   GenerateSM80_Simt_f64(manifest, cuda_version)
 4953:   GenerateSM80_Simt_complex(manifest, cuda_version)
```
**EN:** Defines `GenerateSM80()`, which generates sm80. Key helper calls include GenerateSM80_TensorOp_16816, GenerateSM80_SparseTensorOp_16832, GenerateSM80_PlanarComplexTensorOp_16816, GenerateSM80_TensorOp_1688, GenerateSM80_TensorOp_1688_fast_math, GenerateSM80_SparseTensorOp_16816_fast_math.

**CN:** 定义 `GenerateSM80()`，用于生成 sm80。 其中会调用的重要辅助函数包括 GenerateSM80_TensorOp_16816, GenerateSM80_SparseTensorOp_16832, GenerateSM80_PlanarComplexTensorOp_16816, GenerateSM80_TensorOp_1688, GenerateSM80_TensorOp_1688_fast_math, GenerateSM80_SparseTensorOp_16816_fast_math。

### L4954-L4956 — Comments

```python
 4954: 
 4955: ###################################################################################################
 4956: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L4957-L5092 — Function `GenerateSM89_TensorOp_16832_fp8`

```python
 4957: def GenerateSM89_TensorOp_16832_fp8(manifest, element_acc):
 4958:   layouts = [
 4959:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 4960:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.RowMajor)
 4961:   ]
 4962: 
 4963:   math_instructions = [
 4964:     MathInstruction(
 4965:       [16, 8, 32],
 4966:       DataType.e4m3, DataType.e4m3, element_acc,
 4967:       OpcodeClass.TensorOp,
 4968:       MathOperation.multiply_add),
 4969:     MathInstruction(
 4970:       [16, 8, 32],
 4971:       DataType.e4m3, DataType.e5m2, element_acc,
 4972:       OpcodeClass.TensorOp,
 4973:       MathOperation.multiply_add),
 4974:     MathInstruction(
 4975:       [16, 8, 32],
 4976:       DataType.e5m2, DataType.e4m3, element_acc,
 4977:       OpcodeClass.TensorOp,
 4978:       MathOperation.multiply_add),
 4979:     MathInstruction(
 4980:       [16, 8, 32],
 4981:       DataType.e5m2, DataType.e5m2, element_acc,
 4982:       OpcodeClass.TensorOp,
 4983:       MathOperation.multiply_add),
 4984:     MathInstruction(
 4985:       [16, 8, 32],
 4986:       DataType.e4m3, DataType.e4m3, element_acc,
 4987:       OpcodeClass.TensorOp,
 4988:       MathOperation.multiply_add_fast_accum),
 4989:     MathInstruction(
 4990:       [16, 8, 32],
 4991:       DataType.e4m3, DataType.e5m2, element_acc,
 4992:       OpcodeClass.TensorOp,
 4993:       MathOperation.multiply_add_fast_accum),
 4994:     MathInstruction(
 4995:       [16, 8, 32],
 4996:       DataType.e5m2, DataType.e4m3, element_acc,
 4997:       OpcodeClass.TensorOp,
 4998:       MathOperation.multiply_add_fast_accum),
 4999:     MathInstruction(
 5000:       [16, 8, 32],
 5001:       DataType.e5m2, DataType.e5m2, element_acc,
 5002:       OpcodeClass.TensorOp,
 5003:       MathOperation.multiply_add_fast_accum),
 5004:   ]
 5005: 
 5006:   min_cc = 89
 5007:   max_cc = 100
 5008:   alignment_constraints = [16,]
 5009:   alignment_constraints_small_channels = [16, 8, 4]
 5010: 
 5011:   for math_inst in math_instructions:
 5012:     tile_descriptions = [
 5013:       TileDescription([256, 128,  64],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 5014:       TileDescription([256, 128, 128],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 5015:       TileDescription([256, 128,  64],  6, [4, 2, 1], math_inst, min_cc, max_cc),
 5016:       TileDescription([128, 256, 128],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 5017:       TileDescription([128, 256,  64],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 5018:       TileDescription([128, 256,  64],  6, [2, 4, 1], math_inst, min_cc, max_cc),
 5019:       TileDescription([256,  64, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 5020:       TileDescription([256,  64,  64],  3, [4, 1, 1], math_inst, min_cc, max_cc),
 5021:       TileDescription([256,  64,  64],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 5022:       TileDescription([ 64, 256, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 5023:       TileDescription([ 64, 256,  64],  3, [1, 4, 1], math_inst, min_cc, max_cc),
 5024:       TileDescription([ 64, 256,  64],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 5025:       TileDescription([256,  32, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 5026:       TileDescription([256,  32,  64],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 5027:       TileDescription([ 32, 256, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 5028:       TileDescription([ 32, 256,  64],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 5029:       TileDescription([128, 128, 128],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 5030:       TileDescription([128, 128, 128],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 5031:       TileDescription([128, 128, 128],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 5032:       TileDescription([128, 128,  64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 5033:       TileDescription([128, 128,  64],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 5034:       TileDescription([128, 128,  64],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 5035:       TileDescription([128, 128,  64],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 5036:       TileDescription([128,  64, 128],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 5037:       TileDescription([128,  64, 128],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 5038:       TileDescription([ 64, 128, 128],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 5039:       TileDescription([ 64, 128, 128],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 5040:       TileDescription([128,  64,  64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 5041:       TileDescription([128,  64,  64],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 5042:       TileDescription([128,  64,  64],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 5043:       TileDescription([128,  64,  64],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 5044:       TileDescription([ 64, 128,  64],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 5045:       TileDescription([ 64, 128,  64],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 5046:       TileDescription([ 64, 128,  64],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 5047:       TileDescription([ 64, 128,  64],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 5048:       TileDescription([128,  32, 128],  4, [4, 1, 1], math_inst, min_cc, max_cc),
 5049:       TileDescription([128,  32,  64],  6, [4, 1, 1], math_inst, min_cc, max_cc),
 5050:       TileDescription([ 32, 128, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 5051:       TileDescription([ 32, 128,  64],  6, [1, 4, 1], math_inst, min_cc, max_cc),
 5052:       TileDescription([ 64,  64, 128],  5, [2, 2, 1], math_inst, min_cc, max_cc),
 5053:       TileDescription([ 64,  64, 128],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 5054:       TileDescription([ 64,  64,  64],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 5055:       TileDescription([ 64,  64,  64], 10, [2, 2, 1], math_inst, min_cc, max_cc),
 5056:     ]
 5057: 
 5058:     data_types = [
 5059:       [
 5060:         math_inst.element_a,
 5061:         math_inst.element_b,
 5062:         DataType.f32,
 5063:         math_inst.element_accumulator
 5064:       ],
 5065:       [
 5066:         math_inst.element_a,
 5067:         math_inst.element_b,
 5068:         DataType.bf16,
 5069:         math_inst.element_accumulator
 5070:       ],
 5071:     ]
 5072: 
 5073:     operations = []
 5074:     for data_type in data_types:
 5075:       operations += CreateGemmOperator(manifest, layouts, tile_descriptions, data_type,
 5076:         alignment_constraints, None, EpilogueFunctor.LinearCombination)
 5077: 
 5078:       conv_layout = (LayoutType.TensorNHWC, LayoutType.TensorNHWC, LayoutType.TensorNHWC)
 5079:       operations += CreateConv2dOperator(manifest, conv_layout, tile_descriptions,
 5080:         data_type, alignment_constraints, [ConvKind.Fprop], EpilogueFunctor.LinearCombination)
 5081: 
 5082:       operations += CreateConv2dFixedChannelsOperator(manifest, conv_layout, tile_descriptions,
 5083:         data_type, alignment_constraints_small_channels, [ConvKind.Fprop], EpilogueFunctor.LinearCombination)
 5084: 
 5085:     for op in operations:
 5086:       if op.tile_description.threadblock_shape[1] >= 128:
 5087:         if op.tile_description.threadblock_shape[0] == 32:
 5088:           op.C.alignment = 8
 5089:         else:
 5090:           op.C.alignment = 16
 5091:       else:
 5092:         op.C.alignment = 8
```
**EN:** Defines `GenerateSM89_TensorOp_16832_fp8()`, which generates sm89 tensor op 16832 fp8. Key helper calls include MathInstruction, TileDescription, CreateGemmOperator, CreateConv2dOperator, CreateConv2dFixedChannelsOperator.

**CN:** 定义 `GenerateSM89_TensorOp_16832_fp8()`，用于生成 sm89 tensor op 16832 fp8。 其中会调用的重要辅助函数包括 MathInstruction, TileDescription, CreateGemmOperator, CreateConv2dOperator, CreateConv2dFixedChannelsOperator。

### L5094-L5098 — Function `GenerateSM89_TensorOp_16832_fp8_fp32acc`

```python
 5094: def GenerateSM89_TensorOp_16832_fp8_fp32acc(manifest, cuda_version):
 5095:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 4):
 5096:     return
 5097: 
 5098:   GenerateSM89_TensorOp_16832_fp8(manifest, DataType.f32)
```
**EN:** Defines `GenerateSM89_TensorOp_16832_fp8_fp32acc()`, which generates sm89 tensor op 16832 fp8 fp32acc. Key helper calls include GenerateSM89_TensorOp_16832_fp8, CudaToolkitVersionSatisfies.

**CN:** 定义 `GenerateSM89_TensorOp_16832_fp8_fp32acc()`，用于生成 sm89 tensor op 16832 fp8 fp32acc。 其中会调用的重要辅助函数包括 GenerateSM89_TensorOp_16832_fp8, CudaToolkitVersionSatisfies。

### L5100-L5104 — Function `GenerateSM89_TensorOp_16832_fp8_fp16acc`

```python
 5100: def GenerateSM89_TensorOp_16832_fp8_fp16acc(manifest, cuda_version):
 5101:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 8):
 5102:     return
 5103: 
 5104:   GenerateSM89_TensorOp_16832_fp8(manifest, DataType.f16)
```
**EN:** Defines `GenerateSM89_TensorOp_16832_fp8_fp16acc()`, which generates sm89 tensor op 16832 fp8 fp16acc. Key helper calls include GenerateSM89_TensorOp_16832_fp8, CudaToolkitVersionSatisfies.

**CN:** 定义 `GenerateSM89_TensorOp_16832_fp8_fp16acc()`，用于生成 sm89 tensor op 16832 fp8 fp16acc。 其中会调用的重要辅助函数包括 GenerateSM89_TensorOp_16832_fp8, CudaToolkitVersionSatisfies。

### L5105-L5106 — Comments

```python
 5105: 
 5106: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L5107-L5200 — Function `GenerateSM89_SparseTensorOp_16864_fp8`

```python
 5107: def GenerateSM89_SparseTensorOp_16864_fp8(manifest, cuda_version):
 5108: 
 5109:   if (
 5110:     not CudaToolkitVersionSatisfies(cuda_version, 12, 4)
 5111:   ):
 5112:     return
 5113: 
 5114:   layouts = [
 5115:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.RowMajor)
 5116:   ]
 5117: 
 5118:   math_instructions = [
 5119:     MathInstruction(
 5120:       [16, 8, 64],
 5121:       DataType.e4m3, DataType.e4m3, DataType.f32,
 5122:       OpcodeClass.TensorOp,
 5123:       MathOperation.multiply_add),
 5124:     MathInstruction(
 5125:       [16, 8, 64],
 5126:       DataType.e4m3, DataType.e5m2, DataType.f32,
 5127:       OpcodeClass.TensorOp,
 5128:       MathOperation.multiply_add),
 5129:     MathInstruction(
 5130:       [16, 8, 64],
 5131:       DataType.e5m2, DataType.e4m3, DataType.f32,
 5132:       OpcodeClass.TensorOp,
 5133:       MathOperation.multiply_add),
 5134:     MathInstruction(
 5135:       [16, 8, 64],
 5136:       DataType.e5m2, DataType.e5m2, DataType.f32,
 5137:       OpcodeClass.TensorOp,
 5138:       MathOperation.multiply_add),
 5139:     MathInstruction(
 5140:       [16, 8, 64],
 5141:       DataType.e4m3, DataType.e4m3, DataType.f32,
 5142:       OpcodeClass.TensorOp,
 5143:       MathOperation.multiply_add_fast_accum),
 5144:     MathInstruction(
 5145:       [16, 8, 64],
 5146:       DataType.e4m3, DataType.e5m2, DataType.f32,
 5147:       OpcodeClass.TensorOp,
 5148:       MathOperation.multiply_add_fast_accum),
 5149:     MathInstruction(
 5150:       [16, 8, 64],
 5151:       DataType.e5m2, DataType.e4m3, DataType.f32,
 5152:       OpcodeClass.TensorOp,
 5153:       MathOperation.multiply_add_fast_accum),
 5154:     MathInstruction(
 5155:       [16, 8, 64],
 5156:       DataType.e5m2, DataType.e5m2, DataType.f32,
 5157:       OpcodeClass.TensorOp,
 5158:       MathOperation.multiply_add_fast_accum),
 5159:   ]
 5160: 
 5161:   min_cc = 89
 5162:   max_cc = 89
 5163: 
 5164:   alignment_constraints = [16,]
 5165: 
 5166:   for math_inst in math_instructions:
 5167:     tile_descriptions = [
 5168:       TileDescription([128,  64, 128],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 5169:       TileDescription([256, 128, 128],  3, [4, 2, 1], math_inst, min_cc, max_cc),
 5170:       TileDescription([128, 256, 128],  3, [2, 4, 1], math_inst, min_cc, max_cc),
 5171:       TileDescription([128, 128, 128],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 5172:       TileDescription([256,  64, 128],  3, [4, 1, 1], math_inst, min_cc, max_cc),
 5173:       TileDescription([ 64, 256, 128],  4, [1, 4, 1], math_inst, min_cc, max_cc),
 5174:       TileDescription([ 64, 128, 128],  6, [2, 2, 1], math_inst, min_cc, max_cc),
 5175:       TileDescription([ 64,  64, 128],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 5176:       TileDescription([128, 128, 256],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 5177:       TileDescription([128,  64, 256],  4, [2, 2, 1], math_inst, min_cc, max_cc),
 5178:       TileDescription([ 64, 128, 256],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 5179:       TileDescription([ 64,  64, 256],  3, [2, 2, 1], math_inst, min_cc, max_cc),
 5180:     ]
 5181: 
 5182:     data_types = [
 5183:       [
 5184:         math_inst.element_a,
 5185:         math_inst.element_b,
 5186:         DataType.f32,
 5187:         math_inst.element_accumulator
 5188:       ],
 5189:     ]
 5190: 
 5191:     operations = []
 5192:     for data_type in data_types:
 5193:       operations += CreateSparseGemmOperator(manifest, layouts, tile_descriptions, data_type,
 5194:         alignment_constraints, None, EpilogueFunctor.LinearCombination)
 5195: 
 5196:     for op in operations:
 5197:       if op.tile_description.threadblock_shape[1] >= 128:
 5198:         op.C.alignment = 16
 5199:       else:
 5200:         op.C.alignment = 8
```
**EN:** Defines `GenerateSM89_SparseTensorOp_16864_fp8()`, which generates sm89 sparse tensor op 16864 fp8. Key helper calls include CudaToolkitVersionSatisfies, MathInstruction, TileDescription, CreateSparseGemmOperator.

**CN:** 定义 `GenerateSM89_SparseTensorOp_16864_fp8()`，用于生成 sm89 sparse tensor op 16864 fp8。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, MathInstruction, TileDescription, CreateSparseGemmOperator。

### L5201-L5204 — Comments

```python
 5201: 
 5202: ###################################################################################################
 5203: 
 5204: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L5205-L5208 — Function `GenerateSM89`

```python
 5205: def GenerateSM89(manifest, cuda_version):
 5206:   GenerateSM89_TensorOp_16832_fp8_fp32acc(manifest, cuda_version)
 5207:   GenerateSM89_TensorOp_16832_fp8_fp16acc(manifest, cuda_version)
 5208:   GenerateSM89_SparseTensorOp_16864_fp8(manifest, cuda_version)
```
**EN:** Defines `GenerateSM89()`, which generates sm89. Key helper calls include GenerateSM89_TensorOp_16832_fp8_fp32acc, GenerateSM89_TensorOp_16832_fp8_fp16acc, GenerateSM89_SparseTensorOp_16864_fp8.

**CN:** 定义 `GenerateSM89()`，用于生成 sm89。 其中会调用的重要辅助函数包括 GenerateSM89_TensorOp_16832_fp8_fp32acc, GenerateSM89_TensorOp_16832_fp8_fp16acc, GenerateSM89_SparseTensorOp_16864_fp8。

### L5209-L5212 — Comments

```python
 5209: 
 5210: ###################################################################################################
 5211: 
 5212: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L5213-L5238 — Import fallback

```python
 5213: try:
 5214:     from .sm90_utils import (
 5215:         generate_fp16_bf16_math_instructions_sm90,
 5216:         generate_tf32_math_instructions_sm90,
 5217:         generate_int8_math_instructions_sm90,
 5218:         generate_fp8_math_instructions_sm90,
 5219:         generate_mixed_dtype_math_instructions_sm90,
 5220:         make_sparse_math_instructions,
 5221:         generate_tile_descriptions_sm90,
 5222:         get_valid_schedules,
 5223:         generate_data_types_from_math_instruction,
 5224:         fix_alignments,
 5225:     )
 5226: except ImportError:
 5227:     from sm90_utils import (
 5228:         generate_fp16_bf16_math_instructions_sm90,
 5229:         generate_tf32_math_instructions_sm90,
 5230:         generate_int8_math_instructions_sm90,
 5231:         generate_fp8_math_instructions_sm90,
 5232:         generate_mixed_dtype_math_instructions_sm90,
 5233:         make_sparse_math_instructions,
 5234:         generate_tile_descriptions_sm90,
 5235:         get_valid_schedules,
 5236:         generate_data_types_from_math_instruction,
 5237:         fix_alignments,
 5238:     )
```
**EN:** Uses `try`/`except` to prefer package imports and fall back to local imports, keeping both installed-package and script execution modes working.

**CN:** 通过 `try`/`except` 优先使用包导入，并在失败时回退到本地导入，从而同时兼容已安装包与脚本运行模式。

### L5240-L5303 — Function `GenerateSM90_TensorOp_16b_WGMMA_gemm`

```python
 5240: def GenerateSM90_TensorOp_16b_WGMMA_gemm(manifest, cuda_version, gemm_kind=GemmKind.Universal3x):
 5241:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 3 if is_grouped(gemm_kind) else 0):
 5242:     return
 5243: 
 5244:   instantiation_level = manifest.get_instantiation_level(pruned_level=100, default_level=131, exhaustive_level=9992)
 5245:   is_aligned = True
 5246: 
 5247:   # layouts for ABC and their alignments.
 5248:   layouts = [
 5249:     [[LayoutType.ColumnMajor, 8], [LayoutType.ColumnMajor, 8], [LayoutType.ColumnMajor, 1]],
 5250:     [[LayoutType.ColumnMajor, 8], [LayoutType.RowMajor,    8], [LayoutType.ColumnMajor, 1]],
 5251:     [[LayoutType.RowMajor,    8], [LayoutType.ColumnMajor, 8], [LayoutType.ColumnMajor, 1]],
 5252:     [[LayoutType.RowMajor,    8], [LayoutType.RowMajor,    8], [LayoutType.ColumnMajor, 1]],
 5253:   ]
 5254: 
 5255:   math_instructions = generate_fp16_bf16_math_instructions_sm90(instantiation_level)
 5256:   tile_descriptions = generate_tile_descriptions_sm90(
 5257:       math_instructions=math_instructions,
 5258:       is_aligned=is_aligned,
 5259:       level=instantiation_level)
 5260: 
 5261:   for tile_desc in tile_descriptions:
 5262:     math_inst = tile_desc.math_instruction
 5263:     data_type_w_source = generate_data_types_from_math_instruction(math_inst)
 5264:     data_type_wo_source = generate_data_types_from_math_instruction(math_inst, element_source=DataType.void)
 5265:     data_types = [data_type_w_source, data_type_wo_source]
 5266: 
 5267:     # for mixed precision kernels, also generate kernels that write output matrix in the A/B format
 5268:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 5269:     if math_inst.element_a != math_inst.element_accumulator:
 5270:         data_type_mixed_w_source = generate_data_types_from_math_instruction(
 5271:             math_inst,
 5272:             element_source=math_inst.element_a,
 5273:             element_dest=math_inst.element_a
 5274:         )
 5275:         data_type_mixed_wo_source = generate_data_types_from_math_instruction(
 5276:             math_inst,
 5277:             element_source=DataType.void,
 5278:             element_dest=math_inst.element_a
 5279:         )
 5280:         data_types.append(data_type_mixed_w_source)
 5281:         data_types.append(data_type_mixed_wo_source)
 5282: 
 5283:     for layout in layouts:
 5284:         for data_type in data_types:
 5285:             layout = fix_alignments(data_type, layout, alignment_bits=128)
 5286: 
 5287:             schedules, stream_k_schedules = get_valid_schedules(
 5288:               tile_description=tile_desc,
 5289:               cuda_version=cuda_version,
 5290:               is_aligned=is_aligned,
 5291:               data_types=data_type,
 5292:               instantiation_level=instantiation_level,
 5293:               layout=layout,
 5294:               gemm_kind=gemm_kind,
 5295:             )
 5296: 
 5297:             if len(schedules):
 5298:               CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules, gemm_kind=gemm_kind)
 5299:               if len(stream_k_schedules):
 5300:                 assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 5301:                 CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 5302:                                               stream_k_schedules,
 5303:                                               tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateSM90_TensorOp_16b_WGMMA_gemm()`, which generates sm90 tensor op 16b wgmma gemm. Key helper calls include get_instantiation_level, generate_fp16_bf16_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_data_types_from_math_instruction, append.

**CN:** 定义 `GenerateSM90_TensorOp_16b_WGMMA_gemm()`，用于生成 sm90 tensor op 16b wgmma gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, generate_fp16_bf16_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_data_types_from_math_instruction, append。

### L5306-L5365 — Function `GenerateSM90_TensorOp_16b_WGMMA_alignx_gemm`

```python
 5306: def GenerateSM90_TensorOp_16b_WGMMA_alignx_gemm(manifest, cuda_version):
 5307:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 5308:     return
 5309: 
 5310:   instantiation_level = manifest.get_instantiation_level(pruned_level=100, default_level=101, exhaustive_level=9992)
 5311:   is_aligned = False
 5312: 
 5313:   # layouts for ABC and their alignments.
 5314:   layouts = [
 5315:     [[LayoutType.RowMajor,    4], [LayoutType.ColumnMajor, 4], [LayoutType.ColumnMajor, 1]],
 5316:     [[LayoutType.RowMajor,    4], [LayoutType.RowMajor,    4], [LayoutType.ColumnMajor, 1]],
 5317:     [[LayoutType.ColumnMajor, 4], [LayoutType.ColumnMajor, 4], [LayoutType.ColumnMajor, 1]],
 5318:     [[LayoutType.ColumnMajor, 4], [LayoutType.RowMajor,    4], [LayoutType.ColumnMajor, 1]],
 5319:     [[LayoutType.RowMajor,    2], [LayoutType.ColumnMajor, 2], [LayoutType.ColumnMajor, 1]],
 5320:     [[LayoutType.RowMajor,    2], [LayoutType.RowMajor,    2], [LayoutType.ColumnMajor, 1]],
 5321:     [[LayoutType.ColumnMajor, 2], [LayoutType.ColumnMajor, 2], [LayoutType.ColumnMajor, 1]],
 5322:     [[LayoutType.ColumnMajor, 2], [LayoutType.RowMajor,    2], [LayoutType.ColumnMajor, 1]],
 5323:   ]
 5324: 
 5325:   math_instructions = generate_fp16_bf16_math_instructions_sm90(instantiation_level)
 5326:   tile_descriptions = generate_tile_descriptions_sm90(
 5327:       math_instructions=math_instructions,
 5328:       is_aligned=is_aligned,
 5329:       level=instantiation_level)
 5330: 
 5331:   for tile_desc in tile_descriptions:
 5332:     math_inst = tile_desc.math_instruction
 5333:     data_type_w_source = generate_data_types_from_math_instruction(math_inst)
 5334:     data_types = [data_type_w_source]
 5335: 
 5336:     # for mixed precision kernels, also generate kernels that write output matrix in the A/B format
 5337:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 5338:     if math_inst.element_a != math_inst.element_accumulator:
 5339:         data_type_mixed_w_source = generate_data_types_from_math_instruction(
 5340:             math_inst,
 5341:             element_source=math_inst.element_a,
 5342:             element_dest=math_inst.element_a
 5343:         )
 5344:         data_types.append(data_type_mixed_w_source)
 5345: 
 5346:     for layout in layouts:
 5347:         for data_type in data_types:
 5348:             layout = fix_alignments(data_type, layout, alignment_bits=128)
 5349: 
 5350:             schedules, stream_k_schedules = get_valid_schedules(
 5351:               tile_description=tile_desc,
 5352:               cuda_version=cuda_version,
 5353:               is_aligned=is_aligned,
 5354:               data_types=data_type,
 5355:               instantiation_level=instantiation_level,
 5356:               layout=layout,
 5357:             )
 5358: 
 5359:             if len(schedules):
 5360:               CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules)
 5361:               if len(stream_k_schedules):
 5362:                 assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 5363:                 CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 5364:                                               stream_k_schedules,
 5365:                                               tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateSM90_TensorOp_16b_WGMMA_alignx_gemm()`, which generates sm90 tensor op 16b wgmma alignx gemm. Key helper calls include get_instantiation_level, generate_fp16_bf16_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_data_types_from_math_instruction, append.

**CN:** 定义 `GenerateSM90_TensorOp_16b_WGMMA_alignx_gemm()`，用于生成 sm90 tensor op 16b wgmma alignx gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, generate_fp16_bf16_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_data_types_from_math_instruction, append。

### L5367-L5429 — Function `GenerateSM90_SparseTensorOp_16b_WGMMA_gemm`

```python
 5367: def GenerateSM90_SparseTensorOp_16b_WGMMA_gemm(manifest, cuda_version):
 5368:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 2):
 5369:     return
 5370: 
 5371:   instantiation_level = manifest.get_instantiation_level(pruned_level=100, default_level=131, exhaustive_level=9992)
 5372:   is_aligned = True
 5373: 
 5374:   # layouts for ABC and their alignments.
 5375:   layouts = [
 5376:     [[LayoutType.ColumnMajor, 8], [LayoutType.ColumnMajor, 8], [LayoutType.ColumnMajor, 1]],
 5377:     [[LayoutType.ColumnMajor, 8], [LayoutType.RowMajor,    8], [LayoutType.ColumnMajor, 1]],
 5378:     [[LayoutType.RowMajor,   16], [LayoutType.ColumnMajor, 8], [LayoutType.ColumnMajor, 1]],
 5379:     [[LayoutType.RowMajor,   16], [LayoutType.RowMajor,    8], [LayoutType.ColumnMajor, 1]],
 5380:   ]
 5381: 
 5382:   math_instructions = make_sparse_math_instructions(generate_fp16_bf16_math_instructions_sm90(instantiation_level))
 5383:   tile_descriptions = generate_tile_descriptions_sm90(
 5384:       math_instructions=math_instructions,
 5385:       is_aligned=is_aligned,
 5386:       level=instantiation_level)
 5387: 
 5388:   for tile_desc in tile_descriptions:
 5389:     math_inst = tile_desc.math_instruction
 5390:     data_type_w_source = generate_data_types_from_math_instruction(math_inst)
 5391:     data_type_wo_source = generate_data_types_from_math_instruction(math_inst, element_source=DataType.void)
 5392:     data_types = [data_type_w_source, data_type_wo_source]
 5393: 
 5394:     # for mixed precision kernels, also generate kernels that write output matrix in the A/B format
 5395:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 5396:     if math_inst.element_a != math_inst.element_accumulator:
 5397:         data_type_mixed_w_source = generate_data_types_from_math_instruction(
 5398:             math_inst,
 5399:             element_source=math_inst.element_a,
 5400:             element_dest=math_inst.element_a
 5401:         )
 5402:         data_type_mixed_wo_source = generate_data_types_from_math_instruction(
 5403:             math_inst,
 5404:             element_source=DataType.void,
 5405:             element_dest=math_inst.element_a
 5406:         )
 5407:         data_types.append(data_type_mixed_w_source)
 5408:         data_types.append(data_type_mixed_wo_source)
 5409: 
 5410:     for layout in layouts:
 5411:         for data_type in data_types:
 5412:             layout = fix_alignments(data_type, layout, alignment_bits=128)
 5413: 
 5414:             schedules, stream_k_schedules = get_valid_schedules(
 5415:               tile_description=tile_desc,
 5416:               cuda_version=cuda_version,
 5417:               is_aligned=is_aligned,
 5418:               data_types=data_type,
 5419:               instantiation_level=instantiation_level,
 5420:               layout=layout,
 5421:             )
 5422: 
 5423:             if len(schedules):
 5424:               CreateSparseGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules)
 5425:               if len(stream_k_schedules):
 5426:                 assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 5427:                 CreateSparseGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 5428:                                                     stream_k_schedules,
 5429:                                                     tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateSM90_SparseTensorOp_16b_WGMMA_gemm()`, which generates sm90 sparse tensor op 16b wgmma gemm. Key helper calls include get_instantiation_level, make_sparse_math_instructions, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_fp16_bf16_math_instructions_sm90, generate_data_types_from_math_instruction.

**CN:** 定义 `GenerateSM90_SparseTensorOp_16b_WGMMA_gemm()`，用于生成 sm90 sparse tensor op 16b wgmma gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, make_sparse_math_instructions, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_fp16_bf16_math_instructions_sm90, generate_data_types_from_math_instruction。

### L5432-L5487 — Function `GenerateSM90_TensorOp_tf32_WGMMA_gemm`

```python
 5432: def GenerateSM90_TensorOp_tf32_WGMMA_gemm(manifest, cuda_version):
 5433:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 5434:     return
 5435: 
 5436:   instantiation_level = manifest.get_instantiation_level(pruned_level=120, default_level=121, exhaustive_level=9992)
 5437:   is_aligned = True
 5438: 
 5439:   # layouts for ABC and their alignments
 5440:   layouts = [
 5441:     [[LayoutType.RowMajor,    4], [LayoutType.ColumnMajor, 4], [LayoutType.ColumnMajor, 4]],
 5442:     [[LayoutType.RowMajor,    4], [LayoutType.RowMajor,    4], [LayoutType.ColumnMajor, 4]],
 5443:     [[LayoutType.ColumnMajor, 4], [LayoutType.ColumnMajor, 4], [LayoutType.ColumnMajor, 4]],
 5444:     [[LayoutType.ColumnMajor, 4], [LayoutType.RowMajor,    4], [LayoutType.ColumnMajor, 4]],
 5445:   ]
 5446: 
 5447:   math_instructions = generate_tf32_math_instructions_sm90(instantiation_level)
 5448:   tile_descriptions = generate_tile_descriptions_sm90(
 5449:       math_instructions=math_instructions,
 5450:       is_aligned=is_aligned,
 5451:       level=instantiation_level)
 5452: 
 5453:   for tile_desc in tile_descriptions:
 5454:     math_inst = tile_desc.math_instruction
 5455: 
 5456:     for layout in layouts:
 5457:         data_type_tf32 = generate_data_types_from_math_instruction(math_inst)
 5458:         data_type_tf32_wo_source = generate_data_types_from_math_instruction(math_inst, element_source=DataType.void)
 5459:         data_type_f32 = copy.deepcopy(data_type_tf32)
 5460:         data_type_f32_wo_source = copy.deepcopy(data_type_tf32_wo_source)
 5461:         data_type_f32["a_type"] = DataType.f32
 5462:         data_type_f32["b_type"] = DataType.f32
 5463:         data_type_f32["epi_type"] = DataType.f32
 5464:         data_type_f32_wo_source["a_type"] = DataType.f32
 5465:         data_type_f32_wo_source["b_type"] = DataType.f32
 5466:         data_type_f32_wo_source["epi_type"] = DataType.f32
 5467:         data_types = [data_type_tf32, data_type_f32, data_type_tf32_wo_source, data_type_f32_wo_source]
 5468: 
 5469:         for data_type in data_types:
 5470:             layout = fix_alignments(data_type, layout, alignment_bits=128)
 5471: 
 5472:             schedules, stream_k_schedules = get_valid_schedules(
 5473:               tile_description=tile_desc,
 5474:               cuda_version=cuda_version,
 5475:               is_aligned=is_aligned,
 5476:               data_types=data_type,
 5477:               instantiation_level=instantiation_level,
 5478:               layout=layout,
 5479:             )
 5480: 
 5481:             if len(schedules):
 5482:               CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules)
 5483:               if len(stream_k_schedules):
 5484:                 assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 5485:                 CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 5486:                                               stream_k_schedules,
 5487:                                               tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateSM90_TensorOp_tf32_WGMMA_gemm()`, which generates sm90 tensor op tf32 wgmma gemm. Key helper calls include get_instantiation_level, generate_tf32_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_data_types_from_math_instruction, deepcopy.

**CN:** 定义 `GenerateSM90_TensorOp_tf32_WGMMA_gemm()`，用于生成 sm90 tensor op tf32 wgmma gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, generate_tf32_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_data_types_from_math_instruction, deepcopy。

### L5490-L5544 — Function `GenerateSM90_TensorOp_tf32_WGMMA_alignx_gemm`

```python
 5490: def GenerateSM90_TensorOp_tf32_WGMMA_alignx_gemm(manifest, cuda_version):
 5491:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 5492:     return
 5493: 
 5494:   instantiation_level = manifest.get_instantiation_level(pruned_level=100, default_level=101, exhaustive_level=9992)
 5495:   is_aligned = False
 5496: 
 5497:   # layouts for ABC and their alignments.
 5498:   layouts = [
 5499:     [[LayoutType.RowMajor,    2], [LayoutType.ColumnMajor, 2], [LayoutType.ColumnMajor, 1]],
 5500:     [[LayoutType.RowMajor,    2], [LayoutType.RowMajor,    2], [LayoutType.ColumnMajor, 1]],
 5501:     [[LayoutType.ColumnMajor, 2], [LayoutType.ColumnMajor, 2], [LayoutType.ColumnMajor, 1]],
 5502:     [[LayoutType.ColumnMajor, 2], [LayoutType.RowMajor,    2], [LayoutType.ColumnMajor, 1]],
 5503:     [[LayoutType.RowMajor,    1], [LayoutType.ColumnMajor, 1], [LayoutType.ColumnMajor, 1]],
 5504:     [[LayoutType.RowMajor,    1], [LayoutType.RowMajor,    1], [LayoutType.ColumnMajor, 1]],
 5505:     [[LayoutType.ColumnMajor, 1], [LayoutType.ColumnMajor, 1], [LayoutType.ColumnMajor, 1]],
 5506:     [[LayoutType.ColumnMajor, 1], [LayoutType.RowMajor,    1], [LayoutType.ColumnMajor, 1]],
 5507:   ]
 5508: 
 5509:   math_instructions = generate_tf32_math_instructions_sm90(instantiation_level)
 5510:   tile_descriptions = generate_tile_descriptions_sm90(
 5511:       math_instructions=math_instructions,
 5512:       is_aligned=is_aligned,
 5513:       level=instantiation_level)
 5514: 
 5515:   for tile_desc in tile_descriptions:
 5516:     math_inst = tile_desc.math_instruction
 5517: 
 5518:     for layout in layouts:
 5519:         # Inconsistency: TF32 does not stamp out void-C
 5520:         data_type_tf32 = generate_data_types_from_math_instruction(math_inst)
 5521:         data_type_f32 = copy.deepcopy(data_type_tf32)
 5522:         data_type_f32["a_type"] = DataType.f32
 5523:         data_type_f32["b_type"] = DataType.f32
 5524:         data_type_f32["epi_type"] = DataType.f32
 5525:         for data_type in [data_type_tf32, data_type_f32]:
 5526:             # Inconsistency: alignments aren't fixed in TF32 / alignx
 5527:             # layout = fix_alignments(data_type, layout, alignment_bits=128)
 5528: 
 5529:             schedules, stream_k_schedules = get_valid_schedules(
 5530:               tile_description=tile_desc,
 5531:               cuda_version=cuda_version,
 5532:               is_aligned=is_aligned,
 5533:               data_types=data_type,
 5534:               instantiation_level=instantiation_level,
 5535:               layout=layout,
 5536:             )
 5537: 
 5538:             if len(schedules):
 5539:               CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules)
 5540:               if len(stream_k_schedules):
 5541:                 assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 5542:                 CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 5543:                                               stream_k_schedules,
 5544:                                               tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateSM90_TensorOp_tf32_WGMMA_alignx_gemm()`, which generates sm90 tensor op tf32 wgmma alignx gemm. Key helper calls include get_instantiation_level, generate_tf32_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_data_types_from_math_instruction, deepcopy.

**CN:** 定义 `GenerateSM90_TensorOp_tf32_WGMMA_alignx_gemm()`，用于生成 sm90 tensor op tf32 wgmma alignx gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, generate_tf32_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_data_types_from_math_instruction, deepcopy。

### L5547-L5599 — Function `GenerateSM90_SparseTensorOp_tf32_WGMMA_gemm`

```python
 5547: def GenerateSM90_SparseTensorOp_tf32_WGMMA_gemm(manifest, cuda_version):
 5548:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 2):
 5549:     return
 5550: 
 5551:   instantiation_level = manifest.get_instantiation_level(pruned_level=120, default_level=121, exhaustive_level=9992)
 5552:   is_aligned = True
 5553: 
 5554:   # layouts for ABC and their alignments
 5555:   layouts = [
 5556:     [[LayoutType.RowMajor,    8], [LayoutType.ColumnMajor, 4], [LayoutType.ColumnMajor, 4]],
 5557:   ]
 5558: 
 5559:   math_instructions = make_sparse_math_instructions(generate_tf32_math_instructions_sm90(instantiation_level))
 5560:   tile_descriptions = generate_tile_descriptions_sm90(
 5561:       math_instructions=math_instructions,
 5562:       is_aligned=is_aligned,
 5563:       level=instantiation_level)
 5564: 
 5565:   for tile_desc in tile_descriptions:
 5566:     math_inst = tile_desc.math_instruction
 5567: 
 5568:     for layout in layouts:
 5569:         data_type_tf32 = generate_data_types_from_math_instruction(math_inst)
 5570:         data_type_tf32_wo_source = generate_data_types_from_math_instruction(math_inst, element_source=DataType.void)
 5571:         data_type_f32 = copy.deepcopy(data_type_tf32)
 5572:         data_type_f32_wo_source = copy.deepcopy(data_type_tf32_wo_source)
 5573:         data_type_f32["a_type"] = DataType.f32
 5574:         data_type_f32["b_type"] = DataType.f32
 5575:         data_type_f32["epi_type"] = DataType.f32
 5576:         data_type_f32_wo_source["a_type"] = DataType.f32
 5577:         data_type_f32_wo_source["b_type"] = DataType.f32
 5578:         data_type_f32_wo_source["epi_type"] = DataType.f32
 5579:         data_types = [data_type_tf32, data_type_f32, data_type_tf32_wo_source, data_type_f32_wo_source]
 5580: 
 5581:         for data_type in data_types:
 5582:             layout = fix_alignments(data_type, layout, alignment_bits=128)
 5583: 
 5584:             schedules, stream_k_schedules = get_valid_schedules(
 5585:               tile_description=tile_desc,
 5586:               cuda_version=cuda_version,
 5587:               is_aligned=is_aligned,
 5588:               data_types=data_type,
 5589:               instantiation_level=instantiation_level,
 5590:               layout=layout,
 5591:             )
 5592: 
 5593:             if len(schedules):
 5594:               CreateSparseGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules)
 5595:               if len(stream_k_schedules):
 5596:                 assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 5597:                 CreateSparseGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 5598:                                                     stream_k_schedules,
 5599:                                                     tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateSM90_SparseTensorOp_tf32_WGMMA_gemm()`, which generates sm90 sparse tensor op tf32 wgmma gemm. Key helper calls include get_instantiation_level, make_sparse_math_instructions, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_tf32_math_instructions_sm90, generate_data_types_from_math_instruction.

**CN:** 定义 `GenerateSM90_SparseTensorOp_tf32_WGMMA_gemm()`，用于生成 sm90 sparse tensor op tf32 wgmma gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, make_sparse_math_instructions, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_tf32_math_instructions_sm90, generate_data_types_from_math_instruction。

### L5602-L5651 — Function `GenerateSM90_TensorOp_int8_WGMMA_gemm`

```python
 5602: def GenerateSM90_TensorOp_int8_WGMMA_gemm(manifest, cuda_version):
 5603:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 5604:     return
 5605: 
 5606:   instantiation_level = manifest.get_instantiation_level(pruned_level=100, default_level=111, exhaustive_level=9992)
 5607:   is_aligned = True
 5608: 
 5609:   # layouts for ABC and their alignments
 5610:   layouts = [
 5611:     [[LayoutType.RowMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 16]],
 5612:   ]
 5613: 
 5614:   math_instructions = generate_int8_math_instructions_sm90(instantiation_level)
 5615:   tile_descriptions = generate_tile_descriptions_sm90(
 5616:       math_instructions=math_instructions,
 5617:       is_aligned=is_aligned,
 5618:       level=instantiation_level)
 5619: 
 5620:   for tile_desc in tile_descriptions:
 5621:     math_inst = tile_desc.math_instruction
 5622:     data_type_w_source = generate_data_types_from_math_instruction(math_inst)
 5623:     data_type_wo_source = generate_data_types_from_math_instruction(math_inst, element_source=DataType.void)
 5624:     data_type_int8_output = generate_data_types_from_math_instruction(
 5625:         math_inst,
 5626:         element_source=DataType.s8,
 5627:         element_dest=math_inst.element_a,
 5628:         element_epilogue=DataType.f32
 5629:     )
 5630:     data_types = [data_type_w_source, data_type_wo_source, data_type_int8_output]
 5631: 
 5632:     for layout in layouts:
 5633:         for data_type in data_types:
 5634:             layout = fix_alignments(data_type, layout, alignment_bits=128)
 5635: 
 5636:             schedules, stream_k_schedules = get_valid_schedules(
 5637:               tile_description=tile_desc,
 5638:               cuda_version=cuda_version,
 5639:               is_aligned=is_aligned,
 5640:               data_types=data_type,
 5641:               instantiation_level=instantiation_level,
 5642:               layout=layout,
 5643:             )
 5644: 
 5645:             if len(schedules):
 5646:               CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules)
 5647:               if len(stream_k_schedules):
 5648:                 assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 5649:                 CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 5650:                                               stream_k_schedules,
 5651:                                               tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateSM90_TensorOp_int8_WGMMA_gemm()`, which generates sm90 tensor op int8 wgmma gemm. Key helper calls include get_instantiation_level, generate_int8_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_data_types_from_math_instruction, fix_alignments.

**CN:** 定义 `GenerateSM90_TensorOp_int8_WGMMA_gemm()`，用于生成 sm90 tensor op int8 wgmma gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, generate_int8_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_data_types_from_math_instruction, fix_alignments。

### L5654-L5703 — Function `GenerateSM90_TensorOp_int8_WGMMA_alignx_gemm`

```python
 5654: def GenerateSM90_TensorOp_int8_WGMMA_alignx_gemm(manifest, cuda_version):
 5655:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 5656:     return
 5657: 
 5658:   instantiation_level = manifest.get_instantiation_level(pruned_level=100, default_level=111, exhaustive_level=9992)
 5659:   is_aligned = False
 5660: 
 5661:   # layouts for ABC and their alignments
 5662:   layouts = [
 5663:     [[LayoutType.RowMajor,  8], [LayoutType.ColumnMajor,  8], [LayoutType.ColumnMajor, 1]],
 5664:     [[LayoutType.RowMajor,  4], [LayoutType.ColumnMajor,  4], [LayoutType.ColumnMajor, 1]],
 5665:   ]
 5666: 
 5667:   math_instructions = generate_int8_math_instructions_sm90(instantiation_level)
 5668:   tile_descriptions = generate_tile_descriptions_sm90(
 5669:       math_instructions=math_instructions,
 5670:       is_aligned=is_aligned,
 5671:       level=instantiation_level)
 5672: 
 5673:   for tile_desc in tile_descriptions:
 5674:     math_inst = tile_desc.math_instruction
 5675:     data_type_w_source = generate_data_types_from_math_instruction(math_inst)
 5676:     data_type_int8_output = generate_data_types_from_math_instruction(
 5677:         math_inst,
 5678:         element_source=DataType.s8,
 5679:         element_dest=math_inst.element_a,
 5680:         element_epilogue=DataType.f32
 5681:     )
 5682:     data_types = [data_type_w_source, data_type_int8_output]
 5683: 
 5684:     for layout in layouts:
 5685:         for data_type in data_types:
 5686:             layout = fix_alignments(data_type, layout, alignment_bits=128)
 5687: 
 5688:             schedules, stream_k_schedules = get_valid_schedules(
 5689:               tile_description=tile_desc,
 5690:               cuda_version=cuda_version,
 5691:               is_aligned=is_aligned,
 5692:               data_types=data_type,
 5693:               instantiation_level=instantiation_level,
 5694:               layout=layout,
 5695:             )
 5696: 
 5697:             if len(schedules):
 5698:               CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules)
 5699:               if len(stream_k_schedules):
 5700:                 assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 5701:                 CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 5702:                                               stream_k_schedules,
 5703:                                               tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateSM90_TensorOp_int8_WGMMA_alignx_gemm()`, which generates sm90 tensor op int8 wgmma alignx gemm. Key helper calls include get_instantiation_level, generate_int8_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_data_types_from_math_instruction, fix_alignments.

**CN:** 定义 `GenerateSM90_TensorOp_int8_WGMMA_alignx_gemm()`，用于生成 sm90 tensor op int8 wgmma alignx gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, generate_int8_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_data_types_from_math_instruction, fix_alignments。

### L5706-L5758 — Function `GenerateSM90_SparseTensorOp_int8_WGMMA_gemm`

```python
 5706: def GenerateSM90_SparseTensorOp_int8_WGMMA_gemm(manifest, cuda_version):
 5707:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 2):
 5708:     return
 5709: 
 5710:   instantiation_level = manifest.get_instantiation_level(pruned_level=100, default_level=111, exhaustive_level=9992)
 5711:   is_aligned = True
 5712: 
 5713:   # layouts for ABC and their alignments
 5714:   layouts = [
 5715:     [[LayoutType.RowMajor, 32], [LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 16]],
 5716:   ]
 5717: 
 5718:   math_instructions = make_sparse_math_instructions(generate_int8_math_instructions_sm90(instantiation_level))
 5719:   tile_descriptions = generate_tile_descriptions_sm90(
 5720:       math_instructions=math_instructions,
 5721:       is_aligned=is_aligned,
 5722:       level=instantiation_level)
 5723: 
 5724:   for tile_desc in tile_descriptions:
 5725:     math_inst = tile_desc.math_instruction
 5726:     # s8.u8 and u8.s8 wgmma variants require PTX 8.4
 5727:     if math_inst.element_a != math_inst.element_b and not CudaToolkitVersionSatisfies(cuda_version, 12, 4):
 5728:       continue
 5729:     data_type_w_source = generate_data_types_from_math_instruction(math_inst)
 5730:     data_type_wo_source = generate_data_types_from_math_instruction(math_inst, element_source=DataType.void)
 5731:     data_type_int8_output = generate_data_types_from_math_instruction(
 5732:         math_inst,
 5733:         element_source=DataType.s8,
 5734:         element_dest=math_inst.element_a,
 5735:         element_epilogue=DataType.f32
 5736:     )
 5737:     data_types = [data_type_w_source, data_type_wo_source, data_type_int8_output]
 5738: 
 5739:     for layout in layouts:
 5740:         for data_type in data_types:
 5741:             layout = fix_alignments(data_type, layout, alignment_bits=128)
 5742: 
 5743:             schedules, stream_k_schedules = get_valid_schedules(
 5744:               tile_description=tile_desc,
 5745:               cuda_version=cuda_version,
 5746:               is_aligned=is_aligned,
 5747:               data_types=data_type,
 5748:               instantiation_level=instantiation_level,
 5749:               layout=layout,
 5750:             )
 5751: 
 5752:             if len(schedules):
 5753:               CreateSparseGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules)
 5754:               if len(stream_k_schedules):
 5755:                 assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 5756:                 CreateSparseGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 5757:                                                     stream_k_schedules,
 5758:                                                     tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateSM90_SparseTensorOp_int8_WGMMA_gemm()`, which generates sm90 sparse tensor op int8 wgmma gemm. Key helper calls include get_instantiation_level, make_sparse_math_instructions, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_int8_math_instructions_sm90, generate_data_types_from_math_instruction.

**CN:** 定义 `GenerateSM90_SparseTensorOp_int8_WGMMA_gemm()`，用于生成 sm90 sparse tensor op int8 wgmma gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, make_sparse_math_instructions, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_int8_math_instructions_sm90, generate_data_types_from_math_instruction。

### L5761-L5825 — Function `GenerateSM90_TensorOp_fp8_WGMMA_gemm`

```python
 5761: def GenerateSM90_TensorOp_fp8_WGMMA_gemm(manifest, cuda_version, gemm_kind=GemmKind.Universal3x):
 5762:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 3 if is_grouped(gemm_kind) else 0):
 5763:     return
 5764: 
 5765:   instantiation_level = manifest.get_instantiation_level(pruned_level=20, default_level=121, exhaustive_level=9992)
 5766:   is_aligned = True
 5767: 
 5768:   # layouts for ABC and their alignments
 5769:   layouts = [
 5770:     [[LayoutType.RowMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 1]],  # TN Layout
 5771:   ]
 5772: 
 5773:   math_instructions = generate_fp8_math_instructions_sm90(instantiation_level)
 5774:   tile_descriptions = generate_tile_descriptions_sm90(
 5775:       math_instructions=math_instructions,
 5776:       is_aligned=is_aligned,
 5777:       level=instantiation_level)
 5778: 
 5779:   for tile_desc in tile_descriptions:
 5780:     math_inst = tile_desc.math_instruction
 5781:     data_types = []
 5782:     fp8_types = [DataType.e4m3, DataType.e5m2]
 5783:     valid_types_for_d = [DataType.f32, DataType.bf16, DataType.f16, DataType.e4m3, DataType.e5m2]
 5784:     valid_types_for_c = copy.deepcopy(valid_types_for_d)
 5785:     valid_types_for_c.append(DataType.void)
 5786:     for c_type, d_type in product(valid_types_for_c, valid_types_for_d):
 5787:         data_types.append(
 5788:             generate_data_types_from_math_instruction(
 5789:                 math_inst,
 5790:                 element_source=c_type,
 5791:                 element_dest=d_type,
 5792:             )
 5793:         )
 5794:     else:
 5795:         for d_type in valid_types_for_d:
 5796:             data_types.append(
 5797:                 generate_data_types_from_math_instruction(
 5798:                     math_inst,
 5799:                     element_source=DataType.void,
 5800:                     element_dest=d_type,
 5801:                 )
 5802:             )
 5803: 
 5804:     for layout in layouts:
 5805:         for data_type in data_types:
 5806:             # Inconsistency: alignments aren't fixed in FP8
 5807:             # layout = fix_alignments(data_type, layout, alignment_bits=128)
 5808: 
 5809:             schedules, stream_k_schedules = get_valid_schedules(
 5810:               tile_description=tile_desc,
 5811:               cuda_version=cuda_version,
 5812:               is_aligned=is_aligned,
 5813:               data_types=data_type,
 5814:               instantiation_level=instantiation_level,
 5815:               layout=layout,
 5816:               gemm_kind=gemm_kind,
 5817:             )
 5818: 
 5819:             if len(schedules):
 5820:               CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules, gemm_kind=gemm_kind)
 5821:               if len(stream_k_schedules):
 5822:                 assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 5823:                 CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 5824:                                               stream_k_schedules,
 5825:                                               tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateSM90_TensorOp_fp8_WGMMA_gemm()`, which generates sm90 tensor op fp8 wgmma gemm. Key helper calls include get_instantiation_level, generate_fp8_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, deepcopy, append.

**CN:** 定义 `GenerateSM90_TensorOp_fp8_WGMMA_gemm()`，用于生成 sm90 tensor op fp8 wgmma gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, generate_fp8_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, deepcopy, append。

### L5827-L5905 — Function `GenerateSM90_TensorOp_fp8_WGMMA_gemm_with_blockwise`

```python
 5827: def GenerateSM90_TensorOp_fp8_WGMMA_gemm_with_blockwise(manifest, cuda_version, gemm_kind=GemmKind.BlockwiseUniversal3x):
 5828:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 3 if is_grouped(gemm_kind) else 0):
 5829:     return
 5830: 
 5831:   instantiation_level = manifest.get_instantiation_level(pruned_level=20, default_level=121, exhaustive_level=9992)
 5832:   is_aligned = True
 5833: 
 5834:   # layouts for ABC and their alignments
 5835:   layouts = [
 5836:     [[LayoutType.RowMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 1]],  # TN Layout
 5837:   ]
 5838: 
 5839:   math_instructions = generate_fp8_math_instructions_sm90(instantiation_level)
 5840:   tile_descriptions_ = generate_tile_descriptions_sm90(
 5841:       math_instructions=math_instructions,
 5842:       is_aligned=is_aligned,
 5843:       level=instantiation_level)
 5844: 
 5845:   tile_descriptions = list()
 5846: 
 5847:   for desc in tile_descriptions_:
 5848:     desc.explicit_vector_sizes = [1, desc.tile_shape[1], desc.tile_shape[2]]
 5849:     tile_descriptions.append(copy.deepcopy(desc))
 5850:     desc.explicit_vector_sizes = [desc.tile_shape[0], desc.tile_shape[1], desc.tile_shape[2]]
 5851:     tile_descriptions.append(copy.deepcopy(desc))
 5852:     desc.explicit_vector_sizes = [desc.tile_shape[0], desc.tile_shape[1], desc.tile_shape[2]]
 5853:     tile_descriptions.append(copy.deepcopy(desc))
 5854:     desc.explicit_vector_sizes = [1, 1, desc.tile_shape[2]]
 5855:     tile_descriptions.append(copy.deepcopy(desc))
 5856: 
 5857:   for tile_desc in tile_descriptions:
 5858:     math_inst = tile_desc.math_instruction
 5859:     data_types = []
 5860:     fp8_types = [DataType.e4m3, DataType.e5m2]
 5861:     valid_types_for_d = [DataType.f32, DataType.bf16, DataType.f16, DataType.e4m3, DataType.e5m2]
 5862:     valid_types_for_c = copy.deepcopy(valid_types_for_d)
 5863:     valid_types_for_c.append(DataType.void)
 5864:     for c_type, d_type in product(valid_types_for_c, valid_types_for_d):
 5865:         data_types.append(
 5866:             generate_data_types_from_math_instruction(
 5867:                 math_inst,
 5868:                 element_source=c_type,
 5869:                 element_dest=d_type,
 5870:             )
 5871:         )
 5872:     else:
 5873:         for d_type in valid_types_for_d:
 5874:             data_types.append(
 5875:                 generate_data_types_from_math_instruction(
 5876:                     math_inst,
 5877:                     element_source=DataType.void,
 5878:                     element_dest=d_type,
 5879:                 )
 5880:             )
 5881: 
 5882:     for layout in layouts:
 5883:         for data_type in data_types:
 5884:             # Inconsistency: alignments aren't fixed in FP8
 5885:             # layout = fix_alignments(data_type, layout, alignment_bits=128)
 5886: 
 5887:             schedules, stream_k_schedules = get_valid_schedules(
 5888:               tile_description=tile_desc,
 5889:               cuda_version=cuda_version,
 5890:               is_aligned=is_aligned,
 5891:               data_types=data_type,
 5892:               instantiation_level=instantiation_level,
 5893:               layout=layout,
 5894:               gemm_kind=gemm_kind,
 5895:               enable_fp8_fast_acc=False,
 5896:             )
 5897: 
 5898:             if len(schedules):
 5899:               CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules, gemm_kind=gemm_kind)
 5900:               if len(stream_k_schedules):
 5901:                 assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 5902:                 CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 5903:                                               stream_k_schedules,
 5904:                                               tile_schedulers=[TileSchedulerType.StreamK],
 5905:                                               gemm_kind=gemm_kind)
```
**EN:** Defines `GenerateSM90_TensorOp_fp8_WGMMA_gemm_with_blockwise()`, which generates sm90 tensor op fp8 wgmma gemm with blockwise. Key helper calls include get_instantiation_level, generate_fp8_math_instructions_sm90, generate_tile_descriptions_sm90, list, CudaToolkitVersionSatisfies, append.

**CN:** 定义 `GenerateSM90_TensorOp_fp8_WGMMA_gemm_with_blockwise()`，用于生成 sm90 tensor op fp8 wgmma gemm with blockwise。 其中会调用的重要辅助函数包括 get_instantiation_level, generate_fp8_math_instructions_sm90, generate_tile_descriptions_sm90, list, CudaToolkitVersionSatisfies, append。

### L5909-L5964 — Function `GenerateSM90_TensorOp_fp8_WGMMA_alignx_gemm`

```python
 5909: def GenerateSM90_TensorOp_fp8_WGMMA_alignx_gemm(manifest, cuda_version):
 5910:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 5911:     return
 5912: 
 5913:   instantiation_level = manifest.get_instantiation_level(pruned_level=0, default_level=101, exhaustive_level=9992)
 5914:   is_aligned = False
 5915: 
 5916:   # layouts for ABC and their alignments
 5917:   layouts = [
 5918:     [[LayoutType.RowMajor, 8], [LayoutType.ColumnMajor, 8], [LayoutType.ColumnMajor, 1]],  # TN Layout
 5919:     [[LayoutType.RowMajor, 4], [LayoutType.ColumnMajor, 4], [LayoutType.ColumnMajor, 1]],  # TN Layout
 5920:   ]
 5921: 
 5922:   math_instructions = generate_fp8_math_instructions_sm90(instantiation_level)
 5923:   tile_descriptions = generate_tile_descriptions_sm90(
 5924:       math_instructions=math_instructions,
 5925:       is_aligned=is_aligned,
 5926:       level=instantiation_level)
 5927: 
 5928:   for tile_desc in tile_descriptions:
 5929:     math_inst = tile_desc.math_instruction
 5930:     data_types = [generate_data_types_from_math_instruction(math_inst)]
 5931:     fp8_types = [DataType.e4m3, DataType.e5m2]
 5932:     valid_types_for_d = [DataType.f32, DataType.bf16, DataType.f16, DataType.e4m3, DataType.e5m2]
 5933:     valid_types_for_c = copy.deepcopy(valid_types_for_d)
 5934:     valid_types_for_c.append(DataType.void)
 5935:     for c_type, d_type in product(valid_types_for_c, valid_types_for_d):
 5936:         data_types.append(
 5937:             generate_data_types_from_math_instruction(
 5938:                 math_inst,
 5939:                 element_source=c_type,
 5940:                 element_dest=d_type,
 5941:             )
 5942:         )
 5943: 
 5944:     for layout in layouts:
 5945:         for data_type in data_types:
 5946:             # Inconsistency: alignments aren't fixed in FP8
 5947:             # layout = fix_alignments(data_type, layout, alignment_bits=128)
 5948: 
 5949:             schedules, stream_k_schedules = get_valid_schedules(
 5950:               tile_description=tile_desc,
 5951:               cuda_version=cuda_version,
 5952:               is_aligned=is_aligned,
 5953:               data_types=data_type,
 5954:               instantiation_level=instantiation_level,
 5955:               layout=layout,
 5956:             )
 5957: 
 5958:             if len(schedules):
 5959:               CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules)
 5960:               if len(stream_k_schedules):
 5961:                 assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 5962:                 CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 5963:                                               stream_k_schedules,
 5964:                                               tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateSM90_TensorOp_fp8_WGMMA_alignx_gemm()`, which generates sm90 tensor op fp8 wgmma alignx gemm. Key helper calls include get_instantiation_level, generate_fp8_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, deepcopy, append.

**CN:** 定义 `GenerateSM90_TensorOp_fp8_WGMMA_alignx_gemm()`，用于生成 sm90 tensor op fp8 wgmma alignx gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, generate_fp8_math_instructions_sm90, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, deepcopy, append。

### L5966-L6054 — Function `GenerateSM90_TensorOp_mixed_dtype_WGMMA_gemm`

```python
 5966: def GenerateSM90_TensorOp_mixed_dtype_WGMMA_gemm(manifest, cuda_version):
 5967:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 1):
 5968:     return
 5969: 
 5970:   instantiation_level = manifest.get_instantiation_level(pruned_level=20, default_level=121, exhaustive_level=9999)
 5971:   is_aligned = True
 5972: 
 5973:   # layouts for ABC, their alignments will be fixed later based on the data type
 5974:   layouts = [
 5975:     [[LayoutType.RowMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 16]],
 5976:   ]
 5977: 
 5978:   valid_types_for_a_b_acc = [
 5979:     (DataType.e4m3, DataType.f16, DataType.f32),
 5980:     (DataType.e4m3, DataType.bf16, DataType.f32),
 5981:     (DataType.e5m2, DataType.f16, DataType.f32),
 5982:     (DataType.e5m2, DataType.bf16, DataType.f32),
 5983:     (DataType.s8, DataType.f16, DataType.f32),
 5984:     (DataType.s8, DataType.bf16, DataType.f32),
 5985:     (DataType.u8, DataType.f16, DataType.f32),
 5986:     (DataType.u8, DataType.bf16, DataType.f32),
 5987:     (DataType.s4, DataType.f16, DataType.f32),
 5988:     (DataType.s4, DataType.bf16, DataType.f32),
 5989:     (DataType.s4, DataType.e4m3, DataType.f32),
 5990:     (DataType.s4, DataType.e5m2, DataType.f32),
 5991:     (DataType.u4, DataType.f16, DataType.f32),
 5992:     (DataType.u4, DataType.bf16, DataType.f32),
 5993:     (DataType.u2, DataType.f16, DataType.f32),
 5994:     (DataType.u2, DataType.bf16, DataType.f32),
 5995:     (DataType.s2, DataType.f16, DataType.f32),
 5996:     (DataType.s2, DataType.bf16, DataType.f32),
 5997:   ]
 5998:   # Note: For sizeof(a_type) > sizeof(b_type), some generated kernels might crash due to a compiler bug. Disable it for now.
 5999:   #swapped_valid_types_for_a_b_acc = [(b_type, a_type, acc_type) for a_type, b_type, acc_type in valid_types_for_a_b_acc]
 6000:   #valid_types_for_a_b_acc = valid_types_for_a_b_acc + swapped_valid_types_for_a_b_acc
 6001: 
 6002:   math_instructions = generate_mixed_dtype_math_instructions_sm90(instantiation_level, valid_types_for_a_b_acc)
 6003: 
 6004:   valid_types_for_d = [DataType.f32, DataType.bf16, DataType.f16, DataType.e4m3, DataType.e5m2]
 6005:   valid_types_for_c = copy.deepcopy(valid_types_for_d)
 6006: 
 6007:   tile_descriptions = generate_tile_descriptions_sm90(
 6008:     math_instructions=math_instructions,
 6009:     is_aligned=is_aligned,
 6010:     level=instantiation_level)
 6011: 
 6012:   for tile_desc in tile_descriptions:
 6013:     math_inst = tile_desc.math_instruction
 6014:     data_types = []
 6015: 
 6016:     # Limit C/D types to avoid a giant number of instantiations.
 6017:     # A typical use case for mixed dtype in DL is weight quantization (tensor A),
 6018:     # therefore we can limit the output type to that of activation (tensor B).
 6019:     valid_types_for_c = [math_inst.element_b]
 6020:     valid_types_for_d = [math_inst.element_b]
 6021: 
 6022:     for c_type, d_type in product(valid_types_for_c, valid_types_for_d):
 6023:       data_types.append(
 6024:         generate_data_types_from_math_instruction(
 6025:           math_inst,
 6026:           element_source=c_type,
 6027:           element_dest=d_type,
 6028:         )
 6029:       )
 6030: 
 6031:     for layout in layouts:
 6032:       for data_type in data_types:
 6033:         # Fix alignments, DataTypeSize are in the unit of bits
 6034:         alignment_bits = 128
 6035:         layout[0][1] = alignment_bits // DataTypeSize[data_type['a_type']]
 6036:         layout[1][1] = alignment_bits // DataTypeSize[data_type['b_type']]
 6037:         layout[2][1] = alignment_bits // DataTypeSize[data_type['c_type']]
 6038: 
 6039:         schedules, stream_k_schedules = get_valid_schedules(
 6040:           tile_description=tile_desc,
 6041:           cuda_version=cuda_version,
 6042:           is_aligned=is_aligned,
 6043:           data_types=data_type,
 6044:           instantiation_level=instantiation_level,
 6045:           layout=layout,
 6046:         )
 6047: 
 6048:         if len(schedules):
 6049:           CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules)
 6050:           if len(stream_k_schedules):
 6051:             assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 6052:             CreateGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 6053:                                           stream_k_schedules,
 6054:                                           tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateSM90_TensorOp_mixed_dtype_WGMMA_gemm()`, which generates sm90 tensor op mixed dtype wgmma gemm. Key helper calls include get_instantiation_level, generate_mixed_dtype_math_instructions_sm90, deepcopy, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, product.

**CN:** 定义 `GenerateSM90_TensorOp_mixed_dtype_WGMMA_gemm()`，用于生成 sm90 tensor op mixed dtype wgmma gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, generate_mixed_dtype_math_instructions_sm90, deepcopy, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, product。

### L6057-L6120 — Function `GenerateSM90_SparseTensorOp_fp8_WGMMA_gemm`

```python
 6057: def GenerateSM90_SparseTensorOp_fp8_WGMMA_gemm(manifest, cuda_version):
 6058:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 2):
 6059:     return
 6060: 
 6061:   instantiation_level = manifest.get_instantiation_level(pruned_level=20, default_level=121, exhaustive_level=9992)
 6062:   is_aligned = True
 6063: 
 6064:   # layouts for ABC and their alignments
 6065:   layouts = [
 6066:     [[LayoutType.RowMajor, 32], [LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 1]],  # TN Layout
 6067:   ]
 6068: 
 6069:   math_instructions = make_sparse_math_instructions(generate_fp8_math_instructions_sm90(instantiation_level))
 6070:   tile_descriptions = generate_tile_descriptions_sm90(
 6071:       math_instructions=math_instructions,
 6072:       is_aligned=is_aligned,
 6073:       level=instantiation_level)
 6074: 
 6075:   for tile_desc in tile_descriptions:
 6076:     math_inst = tile_desc.math_instruction
 6077:     data_types = []
 6078:     fp8_types = [DataType.e4m3, DataType.e5m2]
 6079:     valid_types_for_d = [DataType.f32, DataType.bf16, DataType.f16, DataType.e4m3, DataType.e5m2]
 6080:     valid_types_for_c = copy.deepcopy(valid_types_for_d)
 6081:     valid_types_for_c.append(DataType.void)
 6082:     for c_type, d_type in product(valid_types_for_c, valid_types_for_d):
 6083:         data_types.append(
 6084:             generate_data_types_from_math_instruction(
 6085:                 math_inst,
 6086:                 element_source=c_type,
 6087:                 element_dest=d_type,
 6088:             )
 6089:         )
 6090:     else:
 6091:         for d_type in valid_types_for_d:
 6092:             data_types.append(
 6093:                 generate_data_types_from_math_instruction(
 6094:                     math_inst,
 6095:                     element_source=DataType.void,
 6096:                     element_dest=d_type,
 6097:                 )
 6098:             )
 6099: 
 6100:     for layout in layouts:
 6101:         for data_type in data_types:
 6102:             # Inconsistency: alignments aren't fixed in FP8
 6103:             # layout = fix_alignments(data_type, layout, alignment_bits=128)
 6104: 
 6105:             schedules, stream_k_schedules = get_valid_schedules(
 6106:               tile_description=tile_desc,
 6107:               cuda_version=cuda_version,
 6108:               is_aligned=is_aligned,
 6109:               data_types=data_type,
 6110:               instantiation_level=instantiation_level,
 6111:               layout=layout,
 6112:             )
 6113: 
 6114:             if len(schedules):
 6115:               CreateSparseGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type, schedules)
 6116:               if len(stream_k_schedules):
 6117:                 assert CudaToolkitVersionSatisfies(cuda_version, 12, 1)
 6118:                 CreateSparseGemmUniversal3xOperator(manifest, [layout], [tile_desc], data_type,
 6119:                                                     stream_k_schedules,
 6120:                                                     tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateSM90_SparseTensorOp_fp8_WGMMA_gemm()`, which generates sm90 sparse tensor op fp8 wgmma gemm. Key helper calls include get_instantiation_level, make_sparse_math_instructions, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_fp8_math_instructions_sm90, deepcopy.

**CN:** 定义 `GenerateSM90_SparseTensorOp_fp8_WGMMA_gemm()`，用于生成 sm90 sparse tensor op fp8 wgmma gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, make_sparse_math_instructions, generate_tile_descriptions_sm90, CudaToolkitVersionSatisfies, generate_fp8_math_instructions_sm90, deepcopy。

### L6123-L6165 — Function `GenerateSM90_TensorOp_1684`

```python
 6123: def GenerateSM90_TensorOp_1684(manifest, cuda_version):
 6124: 
 6125:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 8):
 6126:     return
 6127: 
 6128:   layouts = [
 6129:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6130:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 6131:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6132:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 6133:   ]
 6134: 
 6135:   math_inst = MathInstruction(
 6136:       [16, 8, 4],
 6137:       DataType.f64, DataType.f64, DataType.f64,
 6138:       OpcodeClass.TensorOp,
 6139:       MathOperation.multiply_add)
 6140: 
 6141:   min_cc = 90
 6142:   max_cc = 90
 6143: 
 6144:   alignment_constraints = [1,]
 6145: 
 6146:   tile_descriptions = [
 6147:     TileDescription([128, 128, 16], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6148:     TileDescription([256, 64, 16], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6149:     TileDescription([64, 256, 16], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 6150:     TileDescription([256, 32, 16], 3, [4, 1, 1], math_inst, min_cc, max_cc),
 6151:     TileDescription([32, 256, 16], 3, [1, 4, 1], math_inst, min_cc, max_cc),
 6152:     TileDescription([128, 64, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6153:     TileDescription([64, 128, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6154:     TileDescription([64, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6155:     TileDescription([64, 32, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6156:     TileDescription([32, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6157:     TileDescription([32, 32, 16], 5, [2, 2, 1], math_inst, min_cc, max_cc),
 6158:     TileDescription([16, 32, 16], 5, [1, 2, 1], math_inst, min_cc, max_cc),
 6159:     TileDescription([32, 16, 16], 5, [2, 1, 1], math_inst, min_cc, max_cc),
 6160:   ]
 6161: 
 6162:   data_type = [DataType.f64, DataType.f64, DataType.f64, DataType.f64]
 6163: 
 6164:   CreateGemmOperator(manifest, layouts, tile_descriptions,
 6165:     data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM90_TensorOp_1684()`, which generates sm90 tensor op 1684. Key helper calls include MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM90_TensorOp_1684()`，用于生成 sm90 tensor op 1684。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L6166-L6169 — Comments

```python
 6166: 
 6167: #
 6168: 
 6169: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L6170-L6223 — Function `GenerateSM90_TensorOp_1684_complex`

```python
 6170: def GenerateSM90_TensorOp_1684_complex(manifest, cuda_version):
 6171: 
 6172:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 8):
 6173:     return
 6174: 
 6175:   layouts = [
 6176:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6177:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 6178:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6179:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 6180:   ]
 6181: 
 6182:   math_inst =                                             \
 6183:     MathInstruction(                                      \
 6184:       [16, 8, 4],                                          \
 6185:       DataType.f64, DataType.f64, DataType.f64,           \
 6186:       OpcodeClass.TensorOp,                               \
 6187:       MathOperation.multiply_add_complex)
 6188: 
 6189:   min_cc = 90
 6190:   max_cc = 90
 6191: 
 6192:   alignment_constraints = [1,]
 6193: 
 6194:   tile_descriptions = [
 6195:     TileDescription([128, 64,  8 ], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6196:     TileDescription([64,  128, 8 ], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 6197:     TileDescription([64,  64,  8 ], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6198:     TileDescription([64,  32,  8 ], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6199:     TileDescription([32,  64,  8 ], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6200:     TileDescription([32,  32,  8 ], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6201:     TileDescription([16,  32,  8 ], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 6202:     TileDescription([32,  16,  8 ], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 6203:     TileDescription([128, 64,  16], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6204:     TileDescription([64,  128, 16], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 6205:     TileDescription([64,  64,  16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6206:     TileDescription([64,  32,  16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6207:     TileDescription([32,  64,  16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6208:     TileDescription([32,  32,  16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6209:     TileDescription([16,  32,  16], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 6210:     TileDescription([32,  16,  16], 3, [2, 1, 1], math_inst, min_cc, max_cc),
 6211:   ]
 6212: 
 6213:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64, DataType.cf64]
 6214: 
 6215:   complex_transforms = [
 6216:     (ComplexTransform.none, ComplexTransform.none),
 6217:     (ComplexTransform.conj, ComplexTransform.none),
 6218:     (ComplexTransform.none, ComplexTransform.conj),
 6219:     (ComplexTransform.conj, ComplexTransform.conj)
 6220:   ]
 6221: 
 6222:   CreateGemmOperator(manifest, layouts, tile_descriptions, \
 6223:     data_type, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM90_TensorOp_1684_complex()`, which generates sm90 tensor op 1684 complex. Key helper calls include MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM90_TensorOp_1684_complex()`，用于生成 sm90 tensor op 1684 complex。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L6224-L6226 — Comments

```python
 6224: #
 6225: 
 6226: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L6227-L6270 — Function `GenerateSM90_TensorOp_1684_complex_gaussian`

```python
 6227: def GenerateSM90_TensorOp_1684_complex_gaussian(manifest, cuda_version):
 6228: 
 6229:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 8):
 6230:     return
 6231: 
 6232:   layouts = [
 6233:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6234:     (LayoutType.ColumnMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 6235:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6236:     (LayoutType.RowMajor, LayoutType.RowMajor, LayoutType.ColumnMajor),
 6237:   ]
 6238: 
 6239:   math_inst =                                             \
 6240:     MathInstruction(                                      \
 6241:       [16, 8, 4],                                          \
 6242:       DataType.f64, DataType.f64, DataType.f64,           \
 6243:       OpcodeClass.TensorOp,                               \
 6244:       MathOperation.multiply_add_complex_gaussian)
 6245: 
 6246:   min_cc = 90
 6247:   max_cc = 90
 6248: 
 6249:   alignment_constraints = [1,]
 6250: 
 6251:   tile_descriptions = [
 6252:     TileDescription([64, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6253:     TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6254:     TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6255:     TileDescription([32, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6256:     TileDescription([16, 32, 8], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 6257:     TileDescription([32, 16, 8], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 6258:   ]
 6259: 
 6260:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64, DataType.cf64]
 6261: 
 6262:   complex_transforms = [
 6263:     (ComplexTransform.none, ComplexTransform.none),
 6264:     (ComplexTransform.conj, ComplexTransform.none),
 6265:     (ComplexTransform.none, ComplexTransform.conj),
 6266:     (ComplexTransform.conj, ComplexTransform.conj)
 6267:   ]
 6268: 
 6269:   CreateGemmOperator(manifest, layouts, tile_descriptions, \
 6270:     data_type, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM90_TensorOp_1684_complex_gaussian()`, which generates sm90 tensor op 1684 complex gaussian. Key helper calls include MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM90_TensorOp_1684_complex_gaussian()`，用于生成 sm90 tensor op 1684 complex gaussian。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L6271-L6273 — Comments

```python
 6271: #
 6272: 
 6273: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L6274-L6315 — Function `GenerateSM90_TensorOp_1684_rank_k`

```python
 6274: def GenerateSM90_TensorOp_1684_rank_k(manifest, cuda_version):
 6275: 
 6276:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 8):
 6277:     return
 6278: 
 6279:   layouts = [
 6280:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6281:     (LayoutType.RowMajor, LayoutType.ColumnMajor),
 6282:   ]
 6283: 
 6284:   fill_modes = [
 6285:     FillMode.Lower, FillMode.Upper,
 6286:   ]
 6287: 
 6288:   math_inst =                                             \
 6289:     MathInstruction(                                      \
 6290:       [16, 8, 4],                                          \
 6291:       DataType.f64, DataType.f64, DataType.f64,           \
 6292:       OpcodeClass.TensorOp,                               \
 6293:       MathOperation.multiply_add)
 6294: 
 6295:   min_cc = 90
 6296:   max_cc = 90
 6297: 
 6298:   alignment_constraints = [1,]
 6299: 
 6300:   tile_descriptions = [
 6301:     TileDescription([128, 128, 16], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6302:     TileDescription([64, 128, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6303:     TileDescription([128, 64, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6304:     TileDescription([64, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6305:     TileDescription([64, 32, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6306:     TileDescription([32, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6307:     TileDescription([32, 32, 16], 5, [2, 2, 1], math_inst, min_cc, max_cc),
 6308:     TileDescription([16, 32, 16], 5, [1, 2, 1], math_inst, min_cc, max_cc),
 6309:     TileDescription([32, 16, 16], 5, [2, 1, 1], math_inst, min_cc, max_cc),
 6310:   ]
 6311: 
 6312:   data_type = [DataType.f64, DataType.f64, DataType.f64]
 6313: 
 6314:   CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, \
 6315:     data_type, alignment_constraints, BlasMode.symmetric)
```
**EN:** Defines `GenerateSM90_TensorOp_1684_rank_k()`, which generates sm90 tensor op 1684 rank k. Key helper calls include MathInstruction, CreateRankKOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM90_TensorOp_1684_rank_k()`，用于生成 sm90 tensor op 1684 rank k。 其中会调用的重要辅助函数包括 MathInstruction, CreateRankKOperator, CudaToolkitVersionSatisfies, TileDescription。

### L6316-L6318 — Comments

```python
 6316: #
 6317: 
 6318: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L6319-L6364 — Function `GenerateSM90_TensorOp_1684_rank_k_complex`

```python
 6319: def GenerateSM90_TensorOp_1684_rank_k_complex(manifest, cuda_version):
 6320: 
 6321:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 8):
 6322:     return
 6323: 
 6324:   layouts = [
 6325:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6326:     (LayoutType.RowMajor, LayoutType.ColumnMajor),
 6327:   ]
 6328: 
 6329:   fill_modes = [
 6330:     FillMode.Lower, FillMode.Upper,
 6331:   ]
 6332: 
 6333:   math_inst =                                             \
 6334:     MathInstruction(                                      \
 6335:       [16, 8, 4],                                          \
 6336:       DataType.f64, DataType.f64, DataType.f64,           \
 6337:       OpcodeClass.TensorOp,                               \
 6338:       MathOperation.multiply_add_complex)
 6339: 
 6340:   min_cc = 90
 6341:   max_cc = 90
 6342: 
 6343:   alignment_constraints = [1,]
 6344: 
 6345:   tile_descriptions = [
 6346:     TileDescription([128, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6347:     TileDescription([64, 128, 8], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 6348:     TileDescription([64, 64, 8], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6349:     #TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6350:     #TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6351:     #TileDescription([32, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6352:     #TileDescription([16, 32, 8], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 6353:     #TileDescription([32, 16, 8], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 6354:   ]
 6355: 
 6356:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64]
 6357: 
 6358:   # SYRK computation
 6359:   CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, \
 6360:     data_type, alignment_constraints, BlasMode.symmetric)
 6361: 
 6362:   # HERK computation
 6363:   CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, \
 6364:     data_type, alignment_constraints, BlasMode.hermitian)
```
**EN:** Defines `GenerateSM90_TensorOp_1684_rank_k_complex()`, which generates sm90 tensor op 1684 rank k complex. Key helper calls include MathInstruction, CreateRankKOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM90_TensorOp_1684_rank_k_complex()`，用于生成 sm90 tensor op 1684 rank k complex。 其中会调用的重要辅助函数包括 MathInstruction, CreateRankKOperator, CudaToolkitVersionSatisfies, TileDescription。

### L6365-L6368 — Comments

```python
 6365: 
 6366: #
 6367: 
 6368: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L6369-L6414 — Function `GenerateSM90_TensorOp_1684_rank_k_complex_gaussian`

```python
 6369: def GenerateSM90_TensorOp_1684_rank_k_complex_gaussian(manifest, cuda_version):
 6370: 
 6371:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 8):
 6372:     return
 6373: 
 6374:   layouts = [
 6375:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6376:     (LayoutType.RowMajor, LayoutType.ColumnMajor),
 6377:   ]
 6378: 
 6379:   fill_modes = [
 6380:     FillMode.Lower, FillMode.Upper,
 6381:   ]
 6382: 
 6383:   math_inst =                                             \
 6384:     MathInstruction(                                      \
 6385:       [16, 8, 4],                                          \
 6386:       DataType.f64, DataType.f64, DataType.f64,           \
 6387:       OpcodeClass.TensorOp,                               \
 6388:       MathOperation.multiply_add_complex_gaussian)
 6389: 
 6390:   min_cc = 90
 6391:   max_cc = 90
 6392: 
 6393:   alignment_constraints = [1,]
 6394: 
 6395:   tile_descriptions = [
 6396:     TileDescription([64, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6397:     TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6398:     TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6399:     #TileDescription([32, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6400:     #TileDescription([16, 32, 8], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 6401:     #TileDescription([32, 16, 8], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 6402:   ]
 6403: 
 6404:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64]
 6405: 
 6406:   complex_transforms = [ComplexTransform.none,]
 6407: 
 6408:   # SYRK computation
 6409:   CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, \
 6410:     data_type, alignment_constraints, BlasMode.symmetric)
 6411: 
 6412:   # HERK computation
 6413:   CreateRankKOperator(manifest, layouts, fill_modes, tile_descriptions, \
 6414:     data_type, alignment_constraints, BlasMode.hermitian)
```
**EN:** Defines `GenerateSM90_TensorOp_1684_rank_k_complex_gaussian()`, which generates sm90 tensor op 1684 rank k complex gaussian. Key helper calls include MathInstruction, CreateRankKOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM90_TensorOp_1684_rank_k_complex_gaussian()`，用于生成 sm90 tensor op 1684 rank k complex gaussian。 其中会调用的重要辅助函数包括 MathInstruction, CreateRankKOperator, CudaToolkitVersionSatisfies, TileDescription。

### L6415-L6417 — Comments

```python
 6415: #
 6416: 
 6417: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L6418-L6462 — Function `GenerateSM90_TensorOp_1684_trmm`

```python
 6418: def GenerateSM90_TensorOp_1684_trmm(manifest, cuda_version):
 6419: 
 6420:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 8):
 6421:     return
 6422: 
 6423:   layouts = [
 6424:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6425:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6426:   ]
 6427: 
 6428:   side_modes = [
 6429:     SideMode.Left, SideMode.Right,
 6430:   ]
 6431: 
 6432:   fill_modes = [
 6433:     FillMode.Lower, FillMode.Upper,
 6434:   ]
 6435: 
 6436:   diag_types = [
 6437:     DiagType.NonUnit, DiagType.Unit,
 6438:   ]
 6439: 
 6440:   math_inst =                                             \
 6441:     MathInstruction(                                      \
 6442:       [16, 8, 4],                                          \
 6443:       DataType.f64, DataType.f64, DataType.f64,           \
 6444:       OpcodeClass.TensorOp,                               \
 6445:       MathOperation.multiply_add)
 6446: 
 6447:   min_cc = 90
 6448:   max_cc = 90
 6449: 
 6450:   alignment_constraints = [1,]
 6451: 
 6452:   tile_descriptions = [
 6453:     TileDescription([128, 128, 16], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6454:     TileDescription([64, 128, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6455:     TileDescription([128, 64, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6456:     TileDescription([64, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6457:   ]
 6458: 
 6459:   data_type = [DataType.f64, DataType.f64, DataType.f64, DataType.f64]
 6460: 
 6461:   CreateTrmmOperator(manifest, layouts, side_modes, fill_modes, diag_types, tile_descriptions, \
 6462:     data_type, alignment_constraints)
```
**EN:** Defines `GenerateSM90_TensorOp_1684_trmm()`, which generates sm90 tensor op 1684 trmm. Key helper calls include MathInstruction, CreateTrmmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM90_TensorOp_1684_trmm()`，用于生成 sm90 tensor op 1684 trmm。 其中会调用的重要辅助函数包括 MathInstruction, CreateTrmmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L6463-L6465 — Comments

```python
 6463: #
 6464: 
 6465: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L6466-L6515 — Function `GenerateSM90_TensorOp_1684_trmm_complex`

```python
 6466: def GenerateSM90_TensorOp_1684_trmm_complex(manifest, cuda_version):
 6467: 
 6468:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 8):
 6469:     return
 6470: 
 6471:   layouts = [
 6472:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6473:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6474:   ]
 6475: 
 6476:   side_modes = [
 6477:     SideMode.Left, SideMode.Right,
 6478:   ]
 6479: 
 6480:   fill_modes = [
 6481:     FillMode.Lower, FillMode.Upper,
 6482:   ]
 6483: 
 6484:   diag_types = [
 6485:     DiagType.NonUnit, DiagType.Unit,
 6486:   ]
 6487: 
 6488:   math_inst =                                             \
 6489:     MathInstruction(                                      \
 6490:       [16, 8, 4],                                          \
 6491:       DataType.f64, DataType.f64, DataType.f64,           \
 6492:       OpcodeClass.TensorOp,                               \
 6493:       MathOperation.multiply_add_complex)
 6494: 
 6495:   min_cc = 90
 6496:   max_cc = 90
 6497: 
 6498:   alignment_constraints = [1,]
 6499: 
 6500:   tile_descriptions = [
 6501:     TileDescription([128, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6502:     TileDescription([64, 128, 8], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 6503:     TileDescription([64, 64, 8], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6504:     TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6505:     TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6506:   ]
 6507: 
 6508:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64, DataType.cf64]
 6509: 
 6510:   complex_transforms = [
 6511:     ComplexTransform.none, ComplexTransform.conj,
 6512:   ]
 6513: 
 6514:   CreateTrmmOperator(manifest, layouts, side_modes, fill_modes, diag_types, tile_descriptions, \
 6515:     data_type, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM90_TensorOp_1684_trmm_complex()`, which generates sm90 tensor op 1684 trmm complex. Key helper calls include MathInstruction, CreateTrmmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM90_TensorOp_1684_trmm_complex()`，用于生成 sm90 tensor op 1684 trmm complex。 其中会调用的重要辅助函数包括 MathInstruction, CreateTrmmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L6516-L6519 — Comments

```python
 6516: #
 6517: 
 6518: 
 6519: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L6520-L6567 — Function `GenerateSM90_TensorOp_1684_trmm_complex_gaussian`

```python
 6520: def GenerateSM90_TensorOp_1684_trmm_complex_gaussian(manifest, cuda_version):
 6521: 
 6522:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 8):
 6523:     return
 6524: 
 6525:   layouts = [
 6526:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6527:     (LayoutType.RowMajor, LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6528:   ]
 6529: 
 6530:   side_modes = [
 6531:     SideMode.Left, SideMode.Right,
 6532:   ]
 6533: 
 6534:   fill_modes = [
 6535:     FillMode.Lower, FillMode.Upper,
 6536:   ]
 6537: 
 6538:   diag_types = [
 6539:     DiagType.NonUnit, DiagType.Unit,
 6540:   ]
 6541: 
 6542:   math_inst =                                             \
 6543:     MathInstruction(                                      \
 6544:       [16, 8, 4],                                          \
 6545:       DataType.f64, DataType.f64, DataType.f64,           \
 6546:       OpcodeClass.TensorOp,                               \
 6547:       MathOperation.multiply_add_complex_gaussian)
 6548: 
 6549:   min_cc = 90
 6550:   max_cc = 90
 6551: 
 6552:   alignment_constraints = [1,]
 6553: 
 6554:   tile_descriptions = [
 6555:     TileDescription([64, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6556:     TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6557:     TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6558:   ]
 6559: 
 6560:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64, DataType.cf64]
 6561: 
 6562:   complex_transforms = [
 6563:     ComplexTransform.none, ComplexTransform.conj,
 6564:   ]
 6565: 
 6566:   CreateTrmmOperator(manifest, layouts, side_modes, fill_modes, diag_types, tile_descriptions, \
 6567:     data_type, alignment_constraints, complex_transforms)
```
**EN:** Defines `GenerateSM90_TensorOp_1684_trmm_complex_gaussian()`, which generates sm90 tensor op 1684 trmm complex gaussian. Key helper calls include MathInstruction, CreateTrmmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM90_TensorOp_1684_trmm_complex_gaussian()`，用于生成 sm90 tensor op 1684 trmm complex gaussian。 其中会调用的重要辅助函数包括 MathInstruction, CreateTrmmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L6568-L6570 — Comments

```python
 6568: #
 6569: 
 6570: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L6571-L6615 — Function `GenerateSM90_TensorOp_1684_symm`

```python
 6571: def GenerateSM90_TensorOp_1684_symm(manifest, cuda_version):
 6572: 
 6573:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 8):
 6574:     return
 6575: 
 6576:   layouts = [
 6577:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6578:   ]
 6579: 
 6580:   side_modes = [
 6581:     SideMode.Left, SideMode.Right,
 6582:   ]
 6583: 
 6584:   fill_modes = [
 6585:     FillMode.Lower, FillMode.Upper,
 6586:   ]
 6587: 
 6588:   math_inst =                                             \
 6589:     MathInstruction(                                      \
 6590:       [16, 8, 4],                                          \
 6591:       DataType.f64, DataType.f64, DataType.f64,           \
 6592:       OpcodeClass.TensorOp,                               \
 6593:       MathOperation.multiply_add)
 6594: 
 6595:   min_cc = 90
 6596:   max_cc = 90
 6597: 
 6598:   alignment_constraints = [1,]
 6599: 
 6600:   tile_descriptions = [
 6601:     TileDescription([128, 128, 16], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6602:     TileDescription([64, 128, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6603:     TileDescription([128, 64, 16], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6604:     TileDescription([64, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6605:     TileDescription([64, 32, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6606:     TileDescription([32, 64, 16], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6607:     TileDescription([32, 32, 16], 5, [2, 2, 1], math_inst, min_cc, max_cc),
 6608:     TileDescription([16, 32, 16], 5, [1, 2, 1], math_inst, min_cc, max_cc),
 6609:     TileDescription([32, 16, 16], 5, [2, 1, 1], math_inst, min_cc, max_cc),
 6610:   ]
 6611: 
 6612:   data_type = [DataType.f64, DataType.f64, DataType.f64, DataType.f64]
 6613: 
 6614:   CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, \
 6615:     data_type, alignment_constraints, BlasMode.symmetric)
```
**EN:** Defines `GenerateSM90_TensorOp_1684_symm()`, which generates sm90 tensor op 1684 symm. Key helper calls include MathInstruction, CreateSymmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM90_TensorOp_1684_symm()`，用于生成 sm90 tensor op 1684 symm。 其中会调用的重要辅助函数包括 MathInstruction, CreateSymmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L6616-L6618 — Comments

```python
 6616: #
 6617: 
 6618: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L6619-L6667 — Function `GenerateSM90_TensorOp_1684_symm_complex`

```python
 6619: def GenerateSM90_TensorOp_1684_symm_complex(manifest, cuda_version):
 6620: 
 6621:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 8):
 6622:     return
 6623: 
 6624:   layouts = [
 6625:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6626:   ]
 6627: 
 6628:   side_modes = [
 6629:     SideMode.Left, SideMode.Right,
 6630:   ]
 6631: 
 6632:   fill_modes = [
 6633:     FillMode.Lower, FillMode.Upper,
 6634:   ]
 6635: 
 6636:   math_inst =                                             \
 6637:     MathInstruction(                                      \
 6638:       [16, 8, 4],                                          \
 6639:       DataType.f64, DataType.f64, DataType.f64,           \
 6640:       OpcodeClass.TensorOp,                               \
 6641:       MathOperation.multiply_add_complex)
 6642: 
 6643:   min_cc = 90
 6644:   max_cc = 90
 6645: 
 6646:   alignment_constraints = [1,]
 6647: 
 6648:   tile_descriptions = [
 6649:     TileDescription([128, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6650:     TileDescription([64, 128, 8], 3, [2, 4, 1], math_inst, min_cc, max_cc),
 6651:     TileDescription([64, 64, 8], 3, [2, 2, 1], math_inst, min_cc, max_cc),
 6652:     #TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6653:     #TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6654:     #TileDescription([32, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6655:     #TileDescription([16, 32, 8], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 6656:     #TileDescription([32, 16, 8], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 6657:   ]
 6658: 
 6659:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64, DataType.cf64]
 6660: 
 6661:   # SYMM computation
 6662:   CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, \
 6663:     data_type, alignment_constraints, BlasMode.symmetric)
 6664: 
 6665:   # HEMM computation
 6666:   CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, \
 6667:     data_type, alignment_constraints, BlasMode.hermitian)
```
**EN:** Defines `GenerateSM90_TensorOp_1684_symm_complex()`, which generates sm90 tensor op 1684 symm complex. Key helper calls include MathInstruction, CreateSymmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM90_TensorOp_1684_symm_complex()`，用于生成 sm90 tensor op 1684 symm complex。 其中会调用的重要辅助函数包括 MathInstruction, CreateSymmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L6668-L6670 — Comments

```python
 6668: #
 6669: 
 6670: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L6671-L6719 — Function `GenerateSM90_TensorOp_1684_symm_complex_gaussian`

```python
 6671: def GenerateSM90_TensorOp_1684_symm_complex_gaussian(manifest, cuda_version):
 6672: 
 6673:   if not CudaToolkitVersionSatisfies(cuda_version, 11, 8):
 6674:     return
 6675: 
 6676:   layouts = [
 6677:     (LayoutType.ColumnMajor, LayoutType.ColumnMajor),
 6678:   ]
 6679: 
 6680:   side_modes = [
 6681:     SideMode.Left, SideMode.Right,
 6682:   ]
 6683: 
 6684:   fill_modes = [
 6685:     FillMode.Lower, FillMode.Upper,
 6686:   ]
 6687: 
 6688:   math_inst =                                             \
 6689:     MathInstruction(                                      \
 6690:       [16, 8, 4],                                          \
 6691:       DataType.f64, DataType.f64, DataType.f64,           \
 6692:       OpcodeClass.TensorOp,                               \
 6693:       MathOperation.multiply_add_complex_gaussian)
 6694: 
 6695:   min_cc = 90
 6696:   max_cc = 90
 6697: 
 6698:   alignment_constraints = [1,]
 6699: 
 6700:   tile_descriptions = [
 6701:     TileDescription([64, 64, 8], 3, [4, 2, 1], math_inst, min_cc, max_cc),
 6702:     TileDescription([64, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6703:     TileDescription([32, 64, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6704:     #TileDescription([32, 32, 8], 4, [2, 2, 1], math_inst, min_cc, max_cc),
 6705:     #TileDescription([16, 32, 8], 4, [1, 2, 1], math_inst, min_cc, max_cc),
 6706:     #TileDescription([32, 16, 8], 4, [2, 1, 1], math_inst, min_cc, max_cc),
 6707:   ]
 6708: 
 6709:   data_type = [DataType.cf64, DataType.cf64, DataType.cf64, DataType.cf64]
 6710: 
 6711:   complex_transforms = [ComplexTransform.none,]
 6712: 
 6713:   # SYMM computation
 6714:   CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, \
 6715:     data_type, alignment_constraints, BlasMode.symmetric)
 6716: 
 6717:   # HEMM computation
 6718:   CreateSymmOperator(manifest, layouts, side_modes, fill_modes, tile_descriptions, \
 6719:     data_type, alignment_constraints, BlasMode.hermitian)
```
**EN:** Defines `GenerateSM90_TensorOp_1684_symm_complex_gaussian()`, which generates sm90 tensor op 1684 symm complex gaussian. Key helper calls include MathInstruction, CreateSymmOperator, CudaToolkitVersionSatisfies, TileDescription.

**CN:** 定义 `GenerateSM90_TensorOp_1684_symm_complex_gaussian()`，用于生成 sm90 tensor op 1684 symm complex gaussian。 其中会调用的重要辅助函数包括 MathInstruction, CreateSymmOperator, CudaToolkitVersionSatisfies, TileDescription。

### L6720-L6725 — Comments

```python
 6720: #
 6721: 
 6722: 
 6723: 
 6724: # Blackwell SM 100 generators
 6725: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L6726-L6749 — Import fallback

```python
 6726: try:
 6727:     import cutlass_library.sm100_utils
 6728:     from cutlass_library.sm100_utils import (
 6729:       generate_tf32_math_instructions_sm100,
 6730:       generate_16b_math_instructions_sm100,
 6731:       generate_f8f6f4_math_instructions_sm100,
 6732:       generate_mxf8f6f4_math_instructions_sm100,
 6733:       generate_mxf4nvf4_math_instructions_sm100,
 6734:       generate_fp8_math_instructions_sm100,
 6735:       generate_cluster_shapes_sm100,
 6736:       get_pruning_level_from_global_level
 6737:     )
 6738: except ImportError:
 6739:     import sm100_utils
 6740:     from sm100_utils import (
 6741:       generate_tf32_math_instructions_sm100,
 6742:       generate_16b_math_instructions_sm100,
 6743:       generate_f8f6f4_math_instructions_sm100,
 6744:       generate_mxf8f6f4_math_instructions_sm100,
 6745:       generate_mxf4nvf4_math_instructions_sm100,
 6746:       generate_fp8_math_instructions_sm100,
 6747:       generate_cluster_shapes_sm100,
 6748:       get_pruning_level_from_global_level
 6749:     )
```
**EN:** Uses `try`/`except` to prefer package imports and fall back to local imports, keeping both installed-package and script execution modes working.

**CN:** 通过 `try`/`except` 优先使用包导入，并在失败时回退到本地导入，从而同时兼容已安装包与脚本运行模式。

### L6750-L6752 — Comments

```python
 6750: 
 6751: ###################################################################################################
 6752: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L6753-L6756 — Function `get_tma_alignment_elt`

```python
 6753: def get_tma_alignment_elt(data_type : DataType, is_f8f6f4 : bool = True ) -> int:
 6754:   if DataTypeSize[data_type] < 8 and is_f8f6f4:
 6755:     return int(128)
 6756:   return int(16 * 8 / DataTypeSize[data_type])
```
**EN:** Defines `get_tma_alignment_elt()`, which returns or derives tma alignment elt. Key helper calls include int.

**CN:** 定义 `get_tma_alignment_elt()`，用于返回或推导 tma alignment elt。 其中会调用的重要辅助函数包括 int。

### L6758-L6767 — Data definitions

```python
 6758: sm100_cluster_shape_1sm = [
 6759:   [4,4,1]
 6760:   , DynamicClusterShape
 6761: ]
 6762: 
 6763: sm100_cluster_shape_2sm = [
 6764:   # cluster_m % 2 == 0 for 2sm
 6765:   [4,4,1]
 6766:   , DynamicClusterShape
 6767: ]
```
**EN:** Defines or updates module/class-level data such as sm100_cluster_shape_1sm, sm100_cluster_shape_2sm; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 sm100_cluster_shape_1sm, sm100_cluster_shape_2sm；这些值会被后续生成器与 emitter 引用。

### L6769-L6859 — Function `GenerateSM100_TensorOp_32b_UMMA_gemm`

```python
 6769: def GenerateSM100_TensorOp_32b_UMMA_gemm(manifest, cuda_version):
 6770:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 8):
 6771:     return
 6772: 
 6773:   instantiation_level = manifest.get_instantiation_level(pruned_level=490, default_level=490, exhaustive_level=9999)
 6774: 
 6775:   # layouts for ABC and their alignments.
 6776:   layouts = [
 6777:     [[LayoutType.ColumnMajor, 4], [LayoutType.ColumnMajor, 4], [LayoutType.ColumnMajor, 4]],
 6778:     [[LayoutType.ColumnMajor, 4], [LayoutType.RowMajor,    4], [LayoutType.ColumnMajor, 4]],
 6779:     [[LayoutType.RowMajor,    4], [LayoutType.ColumnMajor, 4], [LayoutType.ColumnMajor, 4]],
 6780:     [[LayoutType.RowMajor,    4], [LayoutType.RowMajor,    4], [LayoutType.ColumnMajor, 4]],
 6781:     [[LayoutType.ColumnMajor, 4], [LayoutType.ColumnMajor, 4], [LayoutType.RowMajor,    4]],
 6782:     [[LayoutType.ColumnMajor, 4], [LayoutType.RowMajor,    4], [LayoutType.RowMajor,    4]],
 6783:     [[LayoutType.RowMajor,    4], [LayoutType.ColumnMajor, 4], [LayoutType.RowMajor,    4]],
 6784:     [[LayoutType.RowMajor,    4], [LayoutType.RowMajor,    4], [LayoutType.RowMajor,    4]],
 6785:   ]
 6786: 
 6787:   data_types = [
 6788:     {
 6789:       "a_type"   : DataType.f32,
 6790:       "b_type"   : DataType.f32,
 6791:       "c_type"   : DataType.f32,
 6792:       "d_type"   : DataType.f32,
 6793:       "acc_type" : DataType.f32,
 6794:       "epi_type" : DataType.f32,
 6795:     },
 6796:     {
 6797:       "a_type"   : DataType.f32,
 6798:       "b_type"   : DataType.f32,
 6799:       "c_type"   : DataType.void,
 6800:       "d_type"   : DataType.f32,
 6801:       "acc_type" : DataType.f32,
 6802:       "epi_type" : DataType.f32,
 6803:     },
 6804:   ]
 6805: 
 6806:   thor_sm = ThorSMRenumbering(cuda_version)
 6807: 
 6808:   min_cc = 100
 6809:   max_cc = thor_sm
 6810: 
 6811:   math_instructions_1sm, math_instructions_2sm = generate_tf32_math_instructions_sm100(instantiation_level)
 6812: 
 6813:   cluster_shapes_1sm, cluster_shapes_2sm = generate_cluster_shapes_sm100(instantiation_level)
 6814: 
 6815:   if thor_sm in manifest.compute_capabilities_baseline :
 6816:     if [4,4,1] in cluster_shapes_1sm :
 6817:       cluster_shapes_1sm.remove([4,4,1])
 6818:     if [4,4,1] in cluster_shapes_2sm :
 6819:       cluster_shapes_2sm.remove([4,4,1])
 6820: 
 6821:   tile_schedulers = [
 6822:     TileSchedulerType.Default, TileSchedulerType.StreamK
 6823:   ]
 6824: 
 6825:   # 1xSM MMA kernels
 6826:   for math_inst in math_instructions_1sm:
 6827:     tile_descriptions = []
 6828:     for cluster_shape in cluster_shapes_1sm:
 6829:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 6830:       tile_descriptions.append(
 6831:         TileDescription([
 6832:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 6833:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 6834:           math_inst.instruction_shape[2] * 4 * multiplier_1sm[2]],
 6835:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 6836: 
 6837:     CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_types,
 6838:       [[KernelScheduleType.TmaWarpSpecialized1SmSm100, EpilogueScheduleType.TmaWarpSpecialized1Sm]],
 6839:       tile_schedulers=tile_schedulers)
 6840: 
 6841:   # 2xSM MMA kernels
 6842:   for math_inst in math_instructions_2sm:
 6843:     tile_descriptions = []
 6844:     for cluster_shape in cluster_shapes_2sm:
 6845:       multiplier_2sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 6846:       tile_descriptions.append(
 6847:         TileDescription([
 6848:           math_inst.instruction_shape[0]     * multiplier_2sm[0],
 6849:           math_inst.instruction_shape[1]     * multiplier_2sm[1],
 6850:           math_inst.instruction_shape[2] * 4 * multiplier_2sm[2]],
 6851:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 6852: 
 6853:     if math_inst.instruction_shape[0] == 128:
 6854:       epi_schedule = EpilogueScheduleType.TmaWarpSpecialized2Sm
 6855:     else:
 6856:       epi_schedule = EpilogueScheduleType.ScheduleAuto
 6857: 
 6858:     CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_types,
 6859:       [[KernelScheduleType.TmaWarpSpecialized2SmSm100, epi_schedule]], tile_schedulers=tile_schedulers)
```
**EN:** Defines `GenerateSM100_TensorOp_32b_UMMA_gemm()`, which generates sm100 tensor op 32b umma gemm. Key helper calls include get_instantiation_level, ThorSMRenumbering, generate_tf32_math_instructions_sm100, generate_cluster_shapes_sm100, CudaToolkitVersionSatisfies, CreateGemmUniversal3xOperator.

**CN:** 定义 `GenerateSM100_TensorOp_32b_UMMA_gemm()`，用于生成 sm100 tensor op 32b umma gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, ThorSMRenumbering, generate_tf32_math_instructions_sm100, generate_cluster_shapes_sm100, CudaToolkitVersionSatisfies, CreateGemmUniversal3xOperator。

### L6861-L7039 — Function `GenerateSM100_TensorOp_16b_UMMA_gemm`

```python
 6861: def GenerateSM100_TensorOp_16b_UMMA_gemm(manifest, cuda_version, gemm_kind=GemmKind.Universal3x):
 6862:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 8):
 6863:     return
 6864: 
 6865:   instantiation_level = manifest.get_instantiation_level(pruned_level=490, default_level=490, exhaustive_level=9999)
 6866: 
 6867:   # layouts for ABC and their alignments. C alignment will be set later based on output type
 6868:   layouts = [
 6869:     [[LayoutType.ColumnMajor, 8], [LayoutType.ColumnMajor, 8], [LayoutType.ColumnMajor, 0]],
 6870:     [[LayoutType.ColumnMajor, 8], [LayoutType.RowMajor,    8], [LayoutType.ColumnMajor, 0]],
 6871:     [[LayoutType.RowMajor,    8], [LayoutType.ColumnMajor, 8], [LayoutType.ColumnMajor, 0]],
 6872:     [[LayoutType.RowMajor,    8], [LayoutType.RowMajor,    8], [LayoutType.ColumnMajor, 0]],
 6873:     [[LayoutType.ColumnMajor, 8], [LayoutType.ColumnMajor, 8], [LayoutType.RowMajor,    0]],
 6874:     [[LayoutType.ColumnMajor, 8], [LayoutType.RowMajor,    8], [LayoutType.RowMajor,    0]],
 6875:     [[LayoutType.RowMajor,    8], [LayoutType.ColumnMajor, 8], [LayoutType.RowMajor,    0]],
 6876:     [[LayoutType.RowMajor,    8], [LayoutType.RowMajor,    8], [LayoutType.RowMajor,    0]],
 6877:   ]
 6878: 
 6879:   thor_sm = ThorSMRenumbering(cuda_version)
 6880: 
 6881:   math_instructions_1sm, math_instructions_2sm = generate_16b_math_instructions_sm100(instantiation_level)
 6882:   
 6883:   min_cc = 100
 6884:   max_cc = thor_sm
 6885:   grouped = is_grouped(gemm_kind)
 6886: 
 6887:   cluster_shapes_1sm, cluster_shapes_2sm = generate_cluster_shapes_sm100(instantiation_level)
 6888: 
 6889:   if thor_sm in manifest.compute_capabilities_baseline :
 6890:     if [4,4,1] in cluster_shapes_1sm :
 6891:       cluster_shapes_1sm.remove([4,4,1])
 6892:     if [4,4,1] in cluster_shapes_2sm :
 6893:       cluster_shapes_2sm.remove([4,4,1])
 6894: 
 6895:   tile_schedulers = [
 6896:     TileSchedulerType.Default, TileSchedulerType.StreamK
 6897:   ]
 6898: 
 6899:   # 1xSM MMA kernels
 6900:   for math_inst in math_instructions_1sm:
 6901:     tile_descriptions = []
 6902:     for cluster_shape in cluster_shapes_1sm:
 6903:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 6904:       tile_descriptions.append(
 6905:         TileDescription([
 6906:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 6907:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 6908:           math_inst.instruction_shape[2] * 4 * multiplier_1sm[2]],
 6909:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 6910: 
 6911:     data_types = [
 6912:       {
 6913:         "a_type"   : math_inst.element_a,
 6914:         "b_type"   : math_inst.element_b,
 6915:         "c_type"   : math_inst.element_accumulator,
 6916:         "d_type"   : math_inst.element_accumulator,
 6917:         "acc_type" : math_inst.element_accumulator,
 6918:         "epi_type" : math_inst.element_accumulator,
 6919:       },
 6920:       {
 6921:         "a_type"   : math_inst.element_a,
 6922:         "b_type"   : math_inst.element_b,
 6923:         "c_type"   : DataType.void,
 6924:         "d_type"   : math_inst.element_accumulator,
 6925:         "acc_type" : math_inst.element_accumulator,
 6926:         "epi_type" : math_inst.element_accumulator,
 6927:       },
 6928:     ]
 6929:     # Set alignment d based on Destination format.
 6930:     for layout in layouts:
 6931:       layout[2][1] = 128 // DataTypeSize[data_types[0]["d_type"]]
 6932: 
 6933:     kernel_schedule = KernelScheduleType.TmaWarpSpecialized1SmSm100 if not grouped else KernelScheduleType.PtrArrayTmaWarpSpecialized1SmSm100
 6934:     epi_schedule = EpilogueScheduleType.TmaWarpSpecialized1Sm if not grouped else EpilogueScheduleType.PtrArrayTmaWarpSpecialized1Sm
 6935:     CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_types,
 6936:       [[kernel_schedule, epi_schedule]],
 6937:       tile_schedulers=tile_schedulers, gemm_kind=gemm_kind)
 6938: 
 6939:     # for mixed precision kernels, also generate kernels that write output matrix in the A/B format
 6940:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 6941:     if math_inst.element_a != math_inst.element_accumulator:
 6942:       data_types_mixed = [
 6943:         {
 6944:           "a_type"   : math_inst.element_a,
 6945:           "b_type"   : math_inst.element_b,
 6946:           "c_type"   : math_inst.element_a,
 6947:           "d_type"   : math_inst.element_a,
 6948:           "acc_type" : math_inst.element_accumulator,
 6949:           "epi_type" : math_inst.element_accumulator,
 6950:         },
 6951:         {
 6952:           "a_type"   : math_inst.element_a,
 6953:           "b_type"   : math_inst.element_b,
 6954:           "c_type"   : DataType.void,
 6955:           "d_type"   : math_inst.element_a,
 6956:           "acc_type" : math_inst.element_accumulator,
 6957:           "epi_type" : math_inst.element_accumulator,
 6958:         },
 6959:       ]
 6960:       # Set alignment d based on Destination format.
 6961:       for layout in layouts:
 6962:         layout[2][1] = 128 // DataTypeSize[data_types_mixed[0]["d_type"]]
 6963: 
 6964:       CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_types_mixed,
 6965:         [[kernel_schedule, epi_schedule]],
 6966:         tile_schedulers=tile_schedulers, gemm_kind=gemm_kind)
 6967: 
 6968:   # 2xSM MMA kernels
 6969:   for math_inst in math_instructions_2sm:
 6970:     tile_descriptions = []
 6971:     for cluster_shape in cluster_shapes_2sm:
 6972:       multiplier_2sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 6973:       tile_descriptions.append(
 6974:         TileDescription([
 6975:           math_inst.instruction_shape[0]     * multiplier_2sm[0],
 6976:           math_inst.instruction_shape[1]     * multiplier_2sm[1],
 6977:           math_inst.instruction_shape[2] * 4 * multiplier_2sm[2]],
 6978:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 6979: 
 6980:     data_types = [
 6981:       {
 6982:         "a_type"   : math_inst.element_a,
 6983:         "b_type"   : math_inst.element_b,
 6984:         "c_type"   : math_inst.element_accumulator,
 6985:         "d_type"   : math_inst.element_accumulator,
 6986:         "acc_type" : math_inst.element_accumulator,
 6987:         "epi_type" : math_inst.element_accumulator,
 6988:       },
 6989:       {
 6990:         "a_type"   : math_inst.element_a,
 6991:         "b_type"   : math_inst.element_b,
 6992:         "c_type"   : DataType.void,
 6993:         "d_type"   : math_inst.element_accumulator,
 6994:         "acc_type" : math_inst.element_accumulator,
 6995:         "epi_type" : math_inst.element_accumulator,
 6996:       },
 6997:     ]
 6998:     # Set alignment d based on Destination format.
 6999:     for layout in layouts:
 7000:       layout[2][1] = 128 // DataTypeSize[data_types[0]["d_type"]]
 7001: 
 7002:     if grouped:
 7003:       epi_schedule = EpilogueScheduleType.PtrArrayTmaWarpSpecialized2Sm
 7004:     elif math_inst.instruction_shape[0] == 128:
 7005:       epi_schedule = EpilogueScheduleType.TmaWarpSpecialized2Sm
 7006:     else:
 7007:       epi_schedule = EpilogueScheduleType.ScheduleAuto
 7008:     kernel_schedule = to_grouped_schedule(KernelScheduleType.TmaWarpSpecialized2SmSm100, grouped)
 7009: 
 7010:     CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_types,
 7011:       [[kernel_schedule, epi_schedule]], tile_schedulers=tile_schedulers, gemm_kind=gemm_kind)
 7012: 
 7013:     # for mixed precision kernels, also generate kernels that write output matrix in the A/B format
 7014:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 7015:     if math_inst.element_a != math_inst.element_accumulator:
 7016:       data_types_mixed = [
 7017:         {
 7018:           "a_type"   : math_inst.element_a,
 7019:           "b_type"   : math_inst.element_b,
 7020:           "c_type"   : math_inst.element_a,
 7021:           "d_type"   : math_inst.element_a,
 7022:           "acc_type" : math_inst.element_accumulator,
 7023:           "epi_type" : math_inst.element_accumulator,
 7024:         },
 7025:         {
 7026:           "a_type"   : math_inst.element_a,
 7027:           "b_type"   : math_inst.element_b,
 7028:           "c_type"   : DataType.void,
 7029:           "d_type"   : math_inst.element_a,
 7030:           "acc_type" : math_inst.element_accumulator,
 7031:           "epi_type" : math_inst.element_accumulator,
 7032:         },
 7033:       ]
 7034:       # Set alignment d based on Destination format.
 7035:       for layout in layouts:
 7036:         layout[2][1] = 128 // DataTypeSize[data_types_mixed[0]["d_type"]]
 7037: 
 7038:       CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_types_mixed,
 7039:         [[kernel_schedule, epi_schedule]], tile_schedulers=tile_schedulers, gemm_kind=gemm_kind)
```
**EN:** Defines `GenerateSM100_TensorOp_16b_UMMA_gemm()`, which generates sm100 tensor op 16b umma gemm. Key helper calls include get_instantiation_level, ThorSMRenumbering, generate_16b_math_instructions_sm100, is_grouped, generate_cluster_shapes_sm100, CudaToolkitVersionSatisfies.

**CN:** 定义 `GenerateSM100_TensorOp_16b_UMMA_gemm()`，用于生成 sm100 tensor op 16b umma gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, ThorSMRenumbering, generate_16b_math_instructions_sm100, is_grouped, generate_cluster_shapes_sm100, CudaToolkitVersionSatisfies。

### L7041-L7336 — Function `GenerateSM100_TensorOp_fp8_UMMA_gemm`

```python
 7041: def GenerateSM100_TensorOp_fp8_UMMA_gemm(manifest, cuda_version, gemm_kind=GemmKind.Universal3x):
 7042:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 8):
 7043:     return
 7044: 
 7045:   instantiation_level = manifest.get_instantiation_level(pruned_level=591 , default_level=591 , exhaustive_level=9999)
 7046: 
 7047:   # layouts for ABC and their alignments.
 7048:   layouts = [
 7049:     [[LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 0]],
 7050:     [[LayoutType.ColumnMajor, 16], [LayoutType.RowMajor,    16], [LayoutType.ColumnMajor, 0]], 
 7051:     [[LayoutType.RowMajor,    16], [LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 0]],
 7052:     [[LayoutType.RowMajor,    16], [LayoutType.RowMajor,    16], [LayoutType.ColumnMajor, 0]],
 7053:     [[LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.RowMajor,    0]],
 7054:     [[LayoutType.ColumnMajor, 16], [LayoutType.RowMajor,    16], [LayoutType.RowMajor,    0]],
 7055:     [[LayoutType.RowMajor,    16], [LayoutType.ColumnMajor, 16], [LayoutType.RowMajor,    0]],
 7056:     [[LayoutType.RowMajor,    16], [LayoutType.RowMajor,    16], [LayoutType.RowMajor,    0]],
 7057:   ]
 7058: 
 7059:   thor_sm = ThorSMRenumbering(cuda_version)
 7060: 
 7061:   min_cc = 100
 7062:   max_cc = thor_sm
 7063: 
 7064:   epi_type = DataType.f32
 7065:   grouped = is_grouped(gemm_kind)
 7066: 
 7067:   math_instructions_1sm, math_instructions_2sm = generate_fp8_math_instructions_sm100(instantiation_level, enable_runtime_dtype=not grouped)
 7068: 
 7069:   cluster_shapes_1sm, cluster_shapes_2sm = generate_cluster_shapes_sm100(instantiation_level)
 7070: 
 7071:   if thor_sm in manifest.compute_capabilities_baseline :
 7072:     if [4,4,1] in cluster_shapes_1sm :
 7073:       cluster_shapes_1sm.remove([4,4,1])
 7074:     if [4,4,1] in cluster_shapes_2sm :
 7075:       cluster_shapes_2sm.remove([4,4,1])
 7076: 
 7077:   tile_schedulers = [
 7078:     TileSchedulerType.Default, TileSchedulerType.StreamK
 7079:   ]
 7080: 
 7081:   # 1xSM MMA kernels
 7082:   for math_inst in math_instructions_1sm:
 7083:     tile_descriptions = []
 7084:     for cluster_shape in cluster_shapes_1sm:
 7085:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 7086:       tile_descriptions.append(
 7087:         TileDescription([
 7088:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 7089:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 7090:           math_inst.instruction_shape[2] * 4 * multiplier_1sm[2]],
 7091:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 7092: 
 7093:     data_types = [
 7094:       {
 7095:         "a_type"   : math_inst.element_a,
 7096:         "b_type"   : math_inst.element_b,
 7097:         "c_type"   : DataType.f16,
 7098:         "d_type"   : DataType.f16,
 7099:         "acc_type" : math_inst.element_accumulator,
 7100:         "epi_type" : epi_type,
 7101:       },
 7102:       {
 7103:         "a_type"   : math_inst.element_a,
 7104:         "b_type"   : math_inst.element_b,
 7105:         "c_type"   : DataType.f16,
 7106:         "d_type"   : DataType.e4m3,
 7107:         "acc_type" : math_inst.element_accumulator,
 7108:         "epi_type" : epi_type,
 7109:       },
 7110:       {
 7111:         "a_type"   : math_inst.element_a,
 7112:         "b_type"   : math_inst.element_b,
 7113:         "c_type"   : DataType.f16,
 7114:         "d_type"   : DataType.e5m2,
 7115:         "acc_type" : math_inst.element_accumulator,
 7116:         "epi_type" : epi_type,
 7117:       },
 7118:       {
 7119:         "a_type"   : math_inst.element_a,
 7120:         "b_type"   : math_inst.element_b,
 7121:         "c_type"   : DataType.bf16,
 7122:         "d_type"   : DataType.bf16,
 7123:         "acc_type" : math_inst.element_accumulator,
 7124:         "epi_type" : epi_type,
 7125:       },
 7126:       {
 7127:         "a_type"   : math_inst.element_a,
 7128:         "b_type"   : math_inst.element_b,
 7129:         "c_type"   : DataType.bf16,
 7130:         "d_type"   : DataType.e4m3,
 7131:         "acc_type" : math_inst.element_accumulator,
 7132:         "epi_type" : epi_type,
 7133:       },
 7134:       {
 7135:         "a_type"   : math_inst.element_a,
 7136:         "b_type"   : math_inst.element_b,
 7137:         "c_type"   : DataType.bf16,
 7138:         "d_type"   : DataType.e5m2,
 7139:         "acc_type" : math_inst.element_accumulator,
 7140:         "epi_type" : epi_type,
 7141:       },
 7142:       {
 7143:         "a_type"   : math_inst.element_a,
 7144:         "b_type"   : math_inst.element_b,
 7145:         "c_type"   : DataType.f32,
 7146:         "d_type"   : DataType.f32,
 7147:         "acc_type" : math_inst.element_accumulator,
 7148:         "epi_type" : epi_type,
 7149:       },
 7150:       {
 7151:         "a_type"   : math_inst.element_a,
 7152:         "b_type"   : math_inst.element_b,
 7153:         "c_type"   : DataType.void,
 7154:         "d_type"   : DataType.f16,
 7155:         "acc_type" : math_inst.element_accumulator,
 7156:         "epi_type" : epi_type,
 7157:       },
 7158:       {
 7159:         "a_type"   : math_inst.element_a,
 7160:         "b_type"   : math_inst.element_b,
 7161:         "c_type"   : DataType.void,
 7162:         "d_type"   : DataType.bf16,
 7163:         "acc_type" : math_inst.element_accumulator,
 7164:         "epi_type" : epi_type,
 7165:       },
 7166:       {
 7167:         "a_type"   : math_inst.element_a,
 7168:         "b_type"   : math_inst.element_b,
 7169:         "c_type"   : DataType.void,
 7170:         "d_type"   : DataType.f32,
 7171:         "acc_type" : math_inst.element_accumulator,
 7172:         "epi_type" : epi_type,
 7173:       },
 7174:       {
 7175:         "a_type"   : math_inst.element_a,
 7176:         "b_type"   : math_inst.element_b,
 7177:         "c_type"   : DataType.void,
 7178:         "d_type"   : DataType.e4m3,
 7179:         "acc_type" : math_inst.element_accumulator,
 7180:         "epi_type" : epi_type,
 7181:       },
 7182:       {
 7183:         "a_type"   : math_inst.element_a,
 7184:         "b_type"   : math_inst.element_b,
 7185:         "c_type"   : DataType.void,
 7186:         "d_type"   : DataType.e5m2,
 7187:         "acc_type" : math_inst.element_accumulator,
 7188:         "epi_type" : epi_type,
 7189:       }
 7190:     ]
 7191: 
 7192:     # Set alignment d based on Destination format.
 7193:     for layout in layouts:
 7194:       layout[2][1] = 128 // DataTypeSize[data_types[0]["d_type"]]
 7195: 
 7196:     for data_type in data_types:
 7197:       if ( data_type["a_type"] == DataType.e4m3 ) and ( data_type["b_type"] == DataType.e4m3 ) and\
 7198:          ( data_type["d_type"] == DataType.e5m2 ):
 7199:         continue
 7200:       kernel_schedule = to_grouped_schedule(KernelScheduleType.TmaWarpSpecialized1SmSm100, grouped)
 7201:       epi_schedule = to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecialized1Sm, grouped)
 7202:       CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_type,
 7203:         [[kernel_schedule, epi_schedule]],
 7204:         tile_schedulers=tile_schedulers, gemm_kind=gemm_kind)
 7205: 
 7206:   # 2xSM MMA kernels
 7207: 
 7208:   for math_inst in math_instructions_2sm:
 7209:     tile_descriptions = []
 7210:     for cluster_shape in cluster_shapes_2sm:
 7211:       multiplier_2sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 7212:       tile_descriptions.append(
 7213:         TileDescription([
 7214:           math_inst.instruction_shape[0]     * multiplier_2sm[0],
 7215:           math_inst.instruction_shape[1]     * multiplier_2sm[1],
 7216:           math_inst.instruction_shape[2] * 4 * multiplier_2sm[2]],
 7217:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 7218: 
 7219:     data_types = [
 7220:       {
 7221:         "a_type"   : math_inst.element_a,
 7222:         "b_type"   : math_inst.element_b,
 7223:         "c_type"   : DataType.f16,
 7224:         "d_type"   : DataType.f16,
 7225:         "acc_type" : math_inst.element_accumulator,
 7226:         "epi_type" : epi_type,
 7227:       },
 7228:       {
 7229:         "a_type"   : math_inst.element_a,
 7230:         "b_type"   : math_inst.element_b,
 7231:         "c_type"   : DataType.f16,
 7232:         "d_type"   : DataType.e4m3,
 7233:         "acc_type" : math_inst.element_accumulator,
 7234:         "epi_type" : epi_type,
 7235:       },
 7236:       {
 7237:         "a_type"   : math_inst.element_a,
 7238:         "b_type"   : math_inst.element_b,
 7239:         "c_type"   : DataType.f16,
 7240:         "d_type"   : DataType.e5m2,
 7241:         "acc_type" : math_inst.element_accumulator,
 7242:         "epi_type" : epi_type,
 7243:       },
 7244:       {
 7245:         "a_type"   : math_inst.element_a,
 7246:         "b_type"   : math_inst.element_b,
 7247:         "c_type"   : DataType.bf16,
 7248:         "d_type"   : DataType.bf16,
 7249:         "acc_type" : math_inst.element_accumulator,
 7250:         "epi_type" : epi_type,
 7251:       },
 7252:       {
 7253:         "a_type"   : math_inst.element_a,
 7254:         "b_type"   : math_inst.element_b,
 7255:         "c_type"   : DataType.bf16,
 7256:         "d_type"   : DataType.e4m3,
 7257:         "acc_type" : math_inst.element_accumulator,
 7258:         "epi_type" : epi_type,
 7259:       },
 7260:       {
 7261:         "a_type"   : math_inst.element_a,
 7262:         "b_type"   : math_inst.element_b,
 7263:         "c_type"   : DataType.bf16,
 7264:         "d_type"   : DataType.e5m2,
 7265:         "acc_type" : math_inst.element_accumulator,
 7266:         "epi_type" : epi_type,
 7267:       },
 7268:       {
 7269:         "a_type"   : math_inst.element_a,
 7270:         "b_type"   : math_inst.element_b,
 7271:         "c_type"   : DataType.f32,
 7272:         "d_type"   : DataType.f32,
 7273:         "acc_type" : math_inst.element_accumulator,
 7274:         "epi_type" : epi_type,
 7275:       },
 7276:       {
 7277:         "a_type"   : math_inst.element_a,
 7278:         "b_type"   : math_inst.element_b,
 7279:         "c_type"   : DataType.void,
 7280:         "d_type"   : DataType.f16,
 7281:         "acc_type" : math_inst.element_accumulator,
 7282:         "epi_type" : epi_type,
 7283:       },
 7284:       {
 7285:         "a_type"   : math_inst.element_a,
 7286:         "b_type"   : math_inst.element_b,
 7287:         "c_type"   : DataType.void,
 7288:         "d_type"   : DataType.bf16,
 7289:         "acc_type" : math_inst.element_accumulator,
 7290:         "epi_type" : epi_type,
 7291:       },
 7292:       {
 7293:         "a_type"   : math_inst.element_a,
 7294:         "b_type"   : math_inst.element_b,
 7295:         "c_type"   : DataType.void,
 7296:         "d_type"   : DataType.f32,
 7297:         "acc_type" : math_inst.element_accumulator,
 7298:         "epi_type" : epi_type,
 7299:       },
 7300:       {
 7301:         "a_type"   : math_inst.element_a,
 7302:         "b_type"   : math_inst.element_b,
 7303:         "c_type"   : DataType.void,
 7304:         "d_type"   : DataType.e4m3,
 7305:         "acc_type" : math_inst.element_accumulator,
 7306:         "epi_type" : epi_type,
 7307:       },
 7308:       {
 7309:         "a_type"   : math_inst.element_a,
 7310:         "b_type"   : math_inst.element_b,
 7311:         "c_type"   : DataType.void,
 7312:         "d_type"   : DataType.e5m2,
 7313:         "acc_type" : math_inst.element_accumulator,
 7314:         "epi_type" : epi_type,
 7315:       }
 7316:     ]
 7317: 
 7318:     # Set alignment d based on Destination format.
 7319:     for layout in layouts:
 7320:       layout[2][1] = 128 // DataTypeSize[data_types[0]["d_type"]]
 7321: 
 7322:     for data_type in data_types:
 7323:       if ( data_type["a_type"] == DataType.e4m3 ) and ( data_type["b_type"] == DataType.e4m3 ) and\
 7324:          ( data_type["d_type"] == DataType.e5m2 ):
 7325:         continue
 7326: 
 7327:       if grouped:
 7328:         epi_schedule = EpilogueScheduleType.PtrArrayTmaWarpSpecialized2Sm
 7329:       elif math_inst.instruction_shape[0] == 128:
 7330:         epi_schedule = EpilogueScheduleType.TmaWarpSpecialized2Sm
 7331:       else:
 7332:         epi_schedule = EpilogueScheduleType.ScheduleAuto
 7333:       kernel_schedule = to_grouped_schedule(KernelScheduleType.TmaWarpSpecialized2SmSm100, grouped)
 7334: 
 7335:       CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_type,
 7336:       [[kernel_schedule, epi_schedule]], tile_schedulers=tile_schedulers, gemm_kind=gemm_kind)
```
**EN:** Defines `GenerateSM100_TensorOp_fp8_UMMA_gemm()`, which generates sm100 tensor op fp8 umma gemm. Key helper calls include get_instantiation_level, ThorSMRenumbering, is_grouped, generate_fp8_math_instructions_sm100, generate_cluster_shapes_sm100, CudaToolkitVersionSatisfies.

**CN:** 定义 `GenerateSM100_TensorOp_fp8_UMMA_gemm()`，用于生成 sm100 tensor op fp8 umma gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, ThorSMRenumbering, is_grouped, generate_fp8_math_instructions_sm100, generate_cluster_shapes_sm100, CudaToolkitVersionSatisfies。

### L7338-L7475 — Function `GenerateSM100_TensorOp_fp8_UMMA_gemm_with_blockwise`

```python
 7338: def GenerateSM100_TensorOp_fp8_UMMA_gemm_with_blockwise(manifest, cuda_version, gemm_kind=GemmKind.BlockwiseUniversal3x):
 7339:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 8):
 7340:     return
 7341: 
 7342:   instantiation_level = manifest.get_instantiation_level(pruned_level=593, default_level=593, exhaustive_level=9999)
 7343: 
 7344:   grouped = is_grouped(gemm_kind)
 7345: 
 7346:   # layouts for ABC and their alignments.
 7347:   layouts = [
 7348:     [[LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 0]],
 7349:     [[LayoutType.ColumnMajor, 16], [LayoutType.RowMajor,    16], [LayoutType.ColumnMajor, 0]], 
 7350:     [[LayoutType.RowMajor,    16], [LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 0]],
 7351:     [[LayoutType.RowMajor,    16], [LayoutType.RowMajor,    16], [LayoutType.ColumnMajor, 0]],
 7352:     [[LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.RowMajor,    0]],
 7353:     [[LayoutType.ColumnMajor, 16], [LayoutType.RowMajor,    16], [LayoutType.RowMajor,    0]],
 7354:     [[LayoutType.RowMajor,    16], [LayoutType.ColumnMajor, 16], [LayoutType.RowMajor,    0]],
 7355:     [[LayoutType.RowMajor,    16], [LayoutType.RowMajor,    16], [LayoutType.RowMajor,    0]],
 7356:   ]
 7357: 
 7358:   min_cc = 100
 7359:   max_cc = 100
 7360:   epi_type = DataType.f32
 7361: 
 7362:   pruning_level = get_pruning_level_from_global_level(instantiation_level)
 7363: 
 7364:   math_instructions_1sm, math_instructions_2sm = generate_fp8_math_instructions_sm100(instantiation_level, enable_compile_time_dtype=grouped or pruning_level >= 1, enable_runtime_dtype=not grouped)
 7365: 
 7366:   cluster_shapes_1sm, cluster_shapes_2sm = generate_cluster_shapes_sm100(instantiation_level)
 7367: 
 7368:   tile_schedulers = [
 7369:     TileSchedulerType.Default,
 7370:   ]
 7371: 
 7372:   # 1xSM MMA kernels
 7373:   for math_inst in math_instructions_1sm:
 7374:     tile_descriptions = []
 7375:     for cluster_shape in cluster_shapes_1sm:
 7376:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 7377:       tile_descriptions.append(
 7378:         TileDescription([
 7379:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 7380:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 7381:           math_inst.instruction_shape[2] * 4 * multiplier_1sm[2]],
 7382:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape,
 7383:           [math_inst.instruction_shape[0], math_inst.instruction_shape[1], 
 7384:            math_inst.instruction_shape[2] * 4]))
 7385:       tile_descriptions.append(
 7386:         TileDescription([
 7387:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 7388:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 7389:           math_inst.instruction_shape[2] * 4 * multiplier_1sm[2]],
 7390:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape,
 7391:           [1, math_inst.instruction_shape[1], 
 7392:            math_inst.instruction_shape[2] * 4]))
 7393:       tile_descriptions.append(
 7394:         TileDescription([
 7395:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 7396:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 7397:           math_inst.instruction_shape[2] * 4 * multiplier_1sm[2]],
 7398:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape,
 7399:           [math_inst.instruction_shape[0], 1, 
 7400:            math_inst.instruction_shape[2] * 4]))
 7401: 
 7402:     data_types = [
 7403:       {
 7404:         "a_type"   : math_inst.element_a,
 7405:         "b_type"   : math_inst.element_b,
 7406:         "c_type"   : DataType.f16,
 7407:         "d_type"   : DataType.f16,
 7408:         "acc_type" : math_inst.element_accumulator,
 7409:         "epi_type" : epi_type,
 7410:       },
 7411:       {
 7412:         "a_type"   : math_inst.element_a,
 7413:         "b_type"   : math_inst.element_b,
 7414:         "c_type"   : DataType.bf16,
 7415:         "d_type"   : DataType.bf16,
 7416:         "acc_type" : math_inst.element_accumulator,
 7417:         "epi_type" : epi_type,
 7418:       },
 7419:       {
 7420:         "a_type"   : math_inst.element_a,
 7421:         "b_type"   : math_inst.element_b,
 7422:         "c_type"   : DataType.f32,
 7423:         "d_type"   : DataType.f32,
 7424:         "acc_type" : math_inst.element_accumulator,
 7425:         "epi_type" : epi_type,
 7426:       },
 7427:       {
 7428:         "a_type"   : math_inst.element_a,
 7429:         "b_type"   : math_inst.element_b,
 7430:         "c_type"   : DataType.void,
 7431:         "d_type"   : DataType.f16,
 7432:         "acc_type" : math_inst.element_accumulator,
 7433:         "epi_type" : epi_type,
 7434:       },
 7435:       {
 7436:         "a_type"   : math_inst.element_a,
 7437:         "b_type"   : math_inst.element_b,
 7438:         "c_type"   : DataType.void,
 7439:         "d_type"   : DataType.bf16,
 7440:         "acc_type" : math_inst.element_accumulator,
 7441:         "epi_type" : epi_type,
 7442:       },
 7443:       {
 7444:         "a_type"   : math_inst.element_a,
 7445:         "b_type"   : math_inst.element_b,
 7446:         "c_type"   : DataType.void,
 7447:         "d_type"   : DataType.f32,
 7448:         "acc_type" : math_inst.element_accumulator,
 7449:         "epi_type" : epi_type,
 7450:       },
 7451:     ]
 7452: 
 7453:     # Set alignment d based on Destination format.
 7454:     for layout in layouts:
 7455:       layout[2][1] = 128 // DataTypeSize[data_types[0]["d_type"]]
 7456: 
 7457:     is_runtime_datatype = lambda runtime_datatype: runtime_datatype in (DataType.f4, DataType.f6, DataType.f8)
 7458:     for data_type in data_types:
 7459:       if ( data_type["a_type"] == DataType.e4m3 ) and ( data_type["b_type"] == DataType.e4m3 ) and\
 7460:          ( data_type["d_type"] == DataType.e5m2 ):
 7461:         continue
 7462: 
 7463:       is_runtime_datatype_a = is_runtime_datatype(data_type["a_type"])
 7464:       is_runtime_datatype_b = is_runtime_datatype(data_type["d_type"])
 7465: 
 7466:       # A/B datatypes should be both static or dynamic
 7467:       if (is_runtime_datatype_a != is_runtime_datatype_b):
 7468:         continue
 7469: 
 7470:       kernel_schedule = to_grouped_schedule(KernelScheduleType.BlockwiseTmaWarpSpecialized1SmSm100, grouped)
 7471:       epi_schedule = to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecialized1Sm, grouped)
 7472:       epi_schedule_nosmem = to_grouped_schedule(EpilogueScheduleType.BlockwiseNoSmemWarpSpecialized1Sm, grouped)
 7473:       CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_type,
 7474:         [[kernel_schedule, epi_schedule], [kernel_schedule, epi_schedule_nosmem]],
 7475:         tile_schedulers=tile_schedulers, gemm_kind=gemm_kind)
```
**EN:** Defines `GenerateSM100_TensorOp_fp8_UMMA_gemm_with_blockwise()`, which generates sm100 tensor op fp8 umma gemm with blockwise. Key helper calls include get_instantiation_level, is_grouped, get_pruning_level_from_global_level, generate_fp8_math_instructions_sm100, generate_cluster_shapes_sm100, CudaToolkitVersionSatisfies.

**CN:** 定义 `GenerateSM100_TensorOp_fp8_UMMA_gemm_with_blockwise()`，用于生成 sm100 tensor op fp8 umma gemm with blockwise。 其中会调用的重要辅助函数包括 get_instantiation_level, is_grouped, get_pruning_level_from_global_level, generate_fp8_math_instructions_sm100, generate_cluster_shapes_sm100, CudaToolkitVersionSatisfies。

### L7477-L7641 — Function `GenerateSM100_TensorOp_mixed_8bits_UMMA_gemm`

```python
 7477: def GenerateSM100_TensorOp_mixed_8bits_UMMA_gemm(manifest, cuda_version, gemm_kind=GemmKind.Universal3x):
 7478: 
 7479:   # SM100 MMA with mixed F4/F6/F8 inputs + without block scale
 7480:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 7481:     return
 7482: 
 7483:   instantiation_level = manifest.get_instantiation_level(pruned_level=590, default_level=590, exhaustive_level=9999)
 7484: 
 7485:   grouped = is_grouped(gemm_kind)
 7486: 
 7487:   # layouts for ABC and their alignments.
 7488:   layouts = [
 7489:     [[LayoutType.RowMajor,    -1], [LayoutType.ColumnMajor, -1], [LayoutType.RowMajor, -1]],
 7490:   ]
 7491: 
 7492:   math_instructions_1sm, math_instructions_2sm = generate_f8f6f4_math_instructions_sm100(instantiation_level, enable_runtime_dtype=not grouped)
 7493: 
 7494:   def change_priority_func(shapes_1sm, shapes_2sm):
 7495:     shapes_1sm[(1,2,1)] = 6
 7496:     shapes_1sm[(1,4,1)] = 6
 7497:     shapes_2sm[(2,2,1)] = 6
 7498:     shapes_2sm[(2,4,1)] = 6
 7499:     shapes_2sm[(4,2,1)] = 6
 7500: 
 7501:   cluster_shapes_1sm, cluster_shapes_2sm = generate_cluster_shapes_sm100(instantiation_level, change_priority_func)
 7502: 
 7503:   tile_schedulers = [
 7504:     TileSchedulerType.Default, TileSchedulerType.StreamK
 7505:   ]
 7506: 
 7507:   thor_sm = ThorSMRenumbering(cuda_version)
 7508: 
 7509:   min_cc = 100
 7510:   max_cc = thor_sm
 7511: 
 7512:   epi_type = DataType.f32
 7513: 
 7514:   is_runtime_datatype = lambda runtime_datatype: runtime_datatype in (DataType.f4, DataType.f6, DataType.f8)
 7515: 
 7516:   if thor_sm in manifest.compute_capabilities_baseline :
 7517:     if [4,4,1] in cluster_shapes_1sm :
 7518:       cluster_shapes_1sm.remove([4,4,1])
 7519:     if [4,4,1] in cluster_shapes_2sm :
 7520:       cluster_shapes_2sm.remove([4,4,1])
 7521: 
 7522:   # 1xSM MMA kernels
 7523:   for math_inst in math_instructions_1sm:
 7524:     tile_descriptions = []
 7525:     for cluster_shape in cluster_shapes_1sm:
 7526:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 7527:       tile_descriptions.append(
 7528:         TileDescription([
 7529:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 7530:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 7531:           math_inst.instruction_shape[2] * 4 * multiplier_1sm[2]],
 7532:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 7533: 
 7534:     kernel_data_types = [
 7535:       {
 7536:         "a_type"   : math_inst.element_a,
 7537:         "b_type"   : math_inst.element_b,
 7538:         "c_type"   : DataType.f32,
 7539:         "d_type"   : DataType.f32,
 7540:         "acc_type" : math_inst.element_accumulator,
 7541:         "epi_type" : epi_type,
 7542:       },
 7543:       {
 7544:         "a_type"   : math_inst.element_a,
 7545:         "b_type"   : math_inst.element_b,
 7546:         "c_type"   : DataType.void,
 7547:         "d_type"   : DataType.f32,
 7548:         "acc_type" : math_inst.element_accumulator,
 7549:         "epi_type" : epi_type,
 7550:       },
 7551:       {
 7552:         "a_type"   : math_inst.element_a,
 7553:         "b_type"   : math_inst.element_b,
 7554:         "c_type"   : DataType.void,
 7555:         "d_type"   : DataType.e5m2,
 7556:         "acc_type" : math_inst.element_accumulator,
 7557:         "epi_type" : epi_type,
 7558:       }
 7559:       ]
 7560: 
 7561:     for kernel_data_type in kernel_data_types:
 7562:       # Filter out some kernel
 7563:       if ( kernel_data_type["a_type"] == DataType.e4m3 ) and ( kernel_data_type["b_type"] == DataType.e4m3 ) and\
 7564:          ( kernel_data_type["d_type"] == DataType.e5m2 ):
 7565:         continue
 7566: 
 7567:       # Update layout alignment
 7568:       # alignment for d might be different for each kernel_data_type
 7569:       layouts_copy = copy.deepcopy(layouts)
 7570:       for layout in layouts_copy:
 7571:         # alignment for a
 7572:         layout[0][1] = get_tma_alignment_elt(kernel_data_type["a_type"])
 7573:         # alignment for b
 7574:         layout[1][1] = get_tma_alignment_elt(kernel_data_type["b_type"])
 7575:         # alignment for d
 7576:         layout[2][1] = get_tma_alignment_elt(kernel_data_type["d_type"])
 7577: 
 7578:       CreateGemmUniversal3xOperator(manifest, layouts_copy, tile_descriptions, [kernel_data_type],
 7579:         [[KernelScheduleType.TmaWarpSpecialized1SmSm100, EpilogueScheduleType.TmaWarpSpecialized1Sm]], tile_schedulers=tile_schedulers)
 7580: 
 7581:   for math_inst in math_instructions_2sm:
 7582:     tile_descriptions = []
 7583:     for cluster_shape in cluster_shapes_2sm:
 7584:       multiplier_2sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 7585:       tile_descriptions.append(
 7586:         TileDescription([
 7587:           math_inst.instruction_shape[0]     * multiplier_2sm[0],
 7588:           math_inst.instruction_shape[1]     * multiplier_2sm[1],
 7589:           math_inst.instruction_shape[2] * 4 * multiplier_2sm[2]],
 7590:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 7591: 
 7592:     kernel_data_types = [
 7593:       {
 7594:         "a_type"   : math_inst.element_a,
 7595:         "b_type"   : math_inst.element_b,
 7596:         "c_type"   : DataType.f32,
 7597:         "d_type"   : DataType.f32,
 7598:         "acc_type" : math_inst.element_accumulator,
 7599:         "epi_type" : epi_type,
 7600:       },
 7601:       {
 7602:         "a_type"   : math_inst.element_a,
 7603:         "b_type"   : math_inst.element_b,
 7604:         "c_type"   : DataType.void,
 7605:         "d_type"   : DataType.f32,
 7606:         "acc_type" : math_inst.element_accumulator,
 7607:         "epi_type" : epi_type,
 7608:       },
 7609:       {
 7610:         "a_type"   : math_inst.element_a,
 7611:         "b_type"   : math_inst.element_b,
 7612:         "c_type"   : DataType.void,
 7613:         "d_type"   : DataType.e5m2,
 7614:         "acc_type" : math_inst.element_accumulator,
 7615:         "epi_type" : epi_type,
 7616:       }
 7617:       ]
 7618: 
 7619:     for kernel_data_type in kernel_data_types:
 7620:       # Filter some kernel
 7621:       if ( kernel_data_type["a_type"] == DataType.e4m3 ) and ( kernel_data_type["b_type"] == DataType.e4m3 ) and\
 7622:          ( kernel_data_type["d_type"] == DataType.e5m2 ):
 7623:         continue
 7624: 
 7625:       # Update layout alignment
 7626:       # alignment for d might be different for each kernel_data_type
 7627:       layouts_copy = copy.deepcopy(layouts)
 7628:       for layout in layouts_copy:
 7629:         # alignment for a
 7630:         layout[0][1] = get_tma_alignment_elt(kernel_data_type["a_type"])
 7631:         # alignment for b
 7632:         layout[1][1] = get_tma_alignment_elt(kernel_data_type["b_type"])
 7633:         # alignment for d
 7634:         layout[2][1] = get_tma_alignment_elt(kernel_data_type["d_type"])
 7635: 
 7636:       if math_inst.instruction_shape[0] == 128:
 7637:         CreateGemmUniversal3xOperator(manifest, layouts_copy, tile_descriptions, [kernel_data_type],
 7638:           [[KernelScheduleType.TmaWarpSpecialized2SmSm100, EpilogueScheduleType.TmaWarpSpecialized2Sm]], tile_schedulers=tile_schedulers)
 7639:       else:
 7640:         CreateGemmUniversal3xOperator(manifest, layouts_copy, tile_descriptions, [kernel_data_type],
 7641:           [[KernelScheduleType.TmaWarpSpecialized2SmSm100, EpilogueScheduleType.ScheduleAuto]], tile_schedulers=tile_schedulers)
```
**EN:** Defines `GenerateSM100_TensorOp_mixed_8bits_UMMA_gemm()`, which generates sm100 tensor op mixed 8bits umma gemm. Key helper calls include get_instantiation_level, is_grouped, generate_f8f6f4_math_instructions_sm100, generate_cluster_shapes_sm100, ThorSMRenumbering, CudaToolkitVersionSatisfies.

**CN:** 定义 `GenerateSM100_TensorOp_mixed_8bits_UMMA_gemm()`，用于生成 sm100 tensor op mixed 8bits umma gemm。 其中会调用的重要辅助函数包括 get_instantiation_level, is_grouped, generate_f8f6f4_math_instructions_sm100, generate_cluster_shapes_sm100, ThorSMRenumbering, CudaToolkitVersionSatisfies。

### L7643-L7880 — Function `GenerateSM100_TensorOp_mixed_8bits_UMMA_gemm_with_block_scaled`

```python
 7643: def GenerateSM100_TensorOp_mixed_8bits_UMMA_gemm_with_block_scaled(manifest, cuda_version, gemm_kind=GemmKind.BlockScaledUniversal3x):
 7644: 
 7645:   # SM100 MMA with mixed F4/F6/F8 inputs + block scale
 7646:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 8):
 7647:     return
 7648: 
 7649:   instantiation_level = manifest.get_instantiation_level(pruned_level=590, default_level=590, exhaustive_level=9999)
 7650: 
 7651:   grouped = is_grouped(gemm_kind)
 7652: 
 7653:   layouts = [
 7654:     [[LayoutType.RowMajor,    128], [LayoutType.ColumnMajor, 128], [LayoutType.RowMajor,    0]],
 7655:     [[LayoutType.RowMajor,    128], [LayoutType.ColumnMajor, 128], [LayoutType.ColumnMajor, 0]],
 7656:     [[LayoutType.ColumnMajor, 128], [LayoutType.RowMajor,    128], [LayoutType.RowMajor,    0]],
 7657:   ]
 7658: 
 7659:   math_instructions_1sm, math_instructions_2sm = generate_mxf8f6f4_math_instructions_sm100(instantiation_level, enable_runtime_dtype=not grouped)
 7660: 
 7661:   def change_priority_func(shapes_1sm, shapes_2sm):
 7662:     shapes_1sm[(1,2,1)] = 6
 7663:     shapes_1sm[(1,4,1)] = 6
 7664:     shapes_2sm[(2,2,1)] = 6
 7665:     shapes_2sm[(2,4,1)] = 6
 7666:     shapes_2sm[(4,2,1)] = 6
 7667: 
 7668:   cluster_shapes_1sm, cluster_shapes_2sm = generate_cluster_shapes_sm100(instantiation_level, change_priority_func)
 7669: 
 7670:   ab_types  = [
 7671:     DataType.f4, DataType.f6,
 7672:     DataType.e2m1, 
 7673:     DataType.e2m3, 
 7674:     DataType.e3m2,
 7675:     DataType.e5m2,
 7676:     DataType.e4m3,
 7677:   ]
 7678: 
 7679:   acc_types = [ DataType.f32 ]
 7680: 
 7681:   def tile_schedulers(sfdtype):
 7682:     # Only use the stream-K scheduler for non-void SFD to limit kernel count. When SFD is void,
 7683:     # the epilogue is the traditional linear combination, for which we already have tests with stream-K.
 7684:     if sfdtype["type"] == DataType.void or grouped:
 7685:       return [TileSchedulerType.Default]
 7686:     else:
 7687:       return [TileSchedulerType.Default, TileSchedulerType.StreamK]
 7688: 
 7689:   thor_sm = ThorSMRenumbering(cuda_version)
 7690: 
 7691:   min_cc = 100
 7692:   max_cc = thor_sm
 7693: 
 7694:   epi_type = DataType.f32
 7695: 
 7696:   is_runtime_datatype = lambda runtime_datatype: runtime_datatype in (DataType.f4, DataType.f6, DataType.f8)
 7697: 
 7698:   if thor_sm in manifest.compute_capabilities_baseline :
 7699:     if [4,4,1] in cluster_shapes_1sm :
 7700:       cluster_shapes_1sm.remove([4,4,1])
 7701:     if [4,4,1] in cluster_shapes_2sm :
 7702:       cluster_shapes_2sm.remove([4,4,1])
 7703: 
 7704:   # 1xSM MMA kernels
 7705:   for math_inst in math_instructions_1sm:
 7706:     assert math_inst.opcode_class == OpcodeClass.BlockScaledTensorOp
 7707:     tile_descriptions = []
 7708:     for cluster_shape in cluster_shapes_1sm:
 7709:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 7710:       tile_descriptions.append(
 7711:         TileDescription([
 7712:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 7713:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 7714:           math_inst.instruction_shape[2] * 4 * multiplier_1sm[2]],
 7715:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 7716: 
 7717:     data_types = [
 7718:       {
 7719:         "a_type"   : math_inst.element_a,
 7720:         "b_type"   : math_inst.element_b,
 7721:         "c_type"   : DataType.void,
 7722:         "d_type"   : DataType.f32,
 7723:         "acc_type" : math_inst.element_accumulator,
 7724:         "epi_type" : epi_type,
 7725:         "sf_type"  : math_inst.element_scale_factor,
 7726:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 7727:       },
 7728:       {
 7729:         "a_type"   : math_inst.element_a,
 7730:         "b_type"   : math_inst.element_b,
 7731:         "c_type"   : DataType.void,
 7732:         "d_type"   : DataType.bf16,
 7733:         "acc_type" : math_inst.element_accumulator,
 7734:         "epi_type" : epi_type,
 7735:         "sf_type"  : math_inst.element_scale_factor,
 7736:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 7737:       },
 7738:       {
 7739:         "a_type"   : math_inst.element_a,
 7740:         "b_type"   : math_inst.element_b,
 7741:         "c_type"   : DataType.void,
 7742:         "d_type"   : DataType.e5m2,
 7743:         "acc_type" : math_inst.element_accumulator,
 7744:         "epi_type" : epi_type,
 7745:         "sf_type"  : math_inst.element_scale_factor,
 7746:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 7747:       },
 7748:       {
 7749:         "a_type"   : math_inst.element_a,
 7750:         "b_type"   : math_inst.element_b,
 7751:         "c_type"   : DataType.f16,
 7752:         "d_type"   : DataType.e5m2,
 7753:         "acc_type" : math_inst.element_accumulator,
 7754:         "epi_type" : epi_type,
 7755:         "sf_type"  : math_inst.element_scale_factor,
 7756:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 7757:       },
 7758:       {
 7759:         "a_type"   : math_inst.element_a,
 7760:         "b_type"   : math_inst.element_b,
 7761:         "c_type"   : DataType.f16,
 7762:         "d_type"   : DataType.e3m2,
 7763:         "acc_type" : math_inst.element_accumulator,
 7764:         "epi_type" : epi_type,
 7765:         "sf_type"  : math_inst.element_scale_factor,
 7766:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 7767:       }]
 7768: 
 7769:     # Set alignment d based on Destination format.
 7770:     for layout in layouts:
 7771:       layout[2][1] = 128 // DataTypeSize[data_types[0]["d_type"]]
 7772: 
 7773:     for data_type in data_types:
 7774:       CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_type,
 7775:         [[to_grouped_schedule(KernelScheduleType.Mxf8f6f4TmaWarpSpecialized1SmSm100, grouped), to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecialized1Sm, grouped)]]
 7776:         , tile_schedulers = tile_schedulers(data_type["sfd_type"]), gemm_kind=gemm_kind)
 7777: 
 7778:   for math_inst in math_instructions_2sm:
 7779:     assert math_inst.opcode_class == OpcodeClass.BlockScaledTensorOp
 7780:     tile_descriptions = []
 7781:     for cluster_shape in cluster_shapes_2sm:
 7782:       multiplier_2sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 7783:       tile_descriptions.append(
 7784:         TileDescription([
 7785:           math_inst.instruction_shape[0]     * multiplier_2sm[0],
 7786:           math_inst.instruction_shape[1]     * multiplier_2sm[1],
 7787:           math_inst.instruction_shape[2] * 4 * multiplier_2sm[2]],
 7788:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 7789: 
 7790:     data_types = [
 7791:       {
 7792:         "a_type"   : math_inst.element_a,
 7793:         "b_type"   : math_inst.element_b,
 7794:         "c_type"   : DataType.void,
 7795:         "d_type"   : DataType.f32,
 7796:         "acc_type" : math_inst.element_accumulator,
 7797:         "epi_type" : epi_type,
 7798:         "sf_type"  : math_inst.element_scale_factor,
 7799:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 7800:       },
 7801:       {
 7802:         "a_type"   : math_inst.element_a,
 7803:         "b_type"   : math_inst.element_b,
 7804:         "c_type"   : DataType.void,
 7805:         "d_type"   : DataType.bf16,
 7806:         "acc_type" : math_inst.element_accumulator,
 7807:         "epi_type" : epi_type,
 7808:         "sf_type"  : math_inst.element_scale_factor,
 7809:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 7810:       },
 7811:       {
 7812:         "a_type"   : math_inst.element_a,
 7813:         "b_type"   : math_inst.element_b,
 7814:         "c_type"   : DataType.void,
 7815:         "d_type"   : DataType.e5m2,
 7816:         "acc_type" : math_inst.element_accumulator,
 7817:         "epi_type" : epi_type,
 7818:         "sf_type"  : math_inst.element_scale_factor,
 7819:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 7820:       },
 7821:       {
 7822:         "a_type"   : math_inst.element_a,
 7823:         "b_type"   : math_inst.element_b,
 7824:         "c_type"   : DataType.f16,
 7825:         "d_type"   : DataType.e5m2,
 7826:         "acc_type" : math_inst.element_accumulator,
 7827:         "epi_type" : epi_type,
 7828:         "sf_type"  : math_inst.element_scale_factor,
 7829:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
 7830:       },
 7831:       {
 7832:         "a_type"   : math_inst.element_a,
 7833:         "b_type"   : math_inst.element_b,
 7834:         "c_type"   : DataType.f16,
 7835:         "d_type"   : DataType.e3m2,
 7836:         "acc_type" : math_inst.element_accumulator,
 7837:         "epi_type" : epi_type,
 7838:         "sf_type"  : math_inst.element_scale_factor,
 7839:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
 7840:       },
 7841:     ]
 7842: 
 7843:     # Set alignment d based on Destination format.
 7844:     for data_type in data_types:
 7845:       for layout in layouts:
 7846:         # alignment for a
 7847:         layout[0][1] = get_tma_alignment_elt(data_type["a_type"])
 7848:         # alignment for b
 7849:         layout[1][1] = get_tma_alignment_elt(data_type["b_type"])
 7850:         # alignment for d
 7851:         layout[2][1] = get_tma_alignment_elt(data_type["d_type"])
 7852:         for tile in tile_descriptions:
 7853:           math_inst = tile.math_instruction
 7854:           # Filter some kernels that does not meet the alignment requirements.
 7855:           if layout[0][0] == LayoutType.ColumnMajor:
 7856:             if math_inst.instruction_shape[0] // 2 % layout[0][1] != 0:
 7857:               continue
 7858:           else:
 7859:             if tile.threadblock_shape[2] // tile.cluster_shape[2] % layout[0][1] != 0:
 7860:               continue
 7861:   
 7862:           if layout[1][0] == LayoutType.RowMajor:
 7863:             if math_inst.instruction_shape[1] // 2 % layout[1][1] != 0:
 7864:               continue
 7865:           else:
 7866:             if tile.threadblock_shape[2] // tile.cluster_shape[2] % layout[1][1] != 0:
 7867:               continue
 7868:           
 7869:           if grouped:
 7870:             CreateGemmUniversal3xOperator(manifest, [layout], [tile], [data_type],
 7871:               [[to_grouped_schedule(KernelScheduleType.Mxf8f6f4TmaWarpSpecialized2SmSm100, grouped), to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecialized2Sm, grouped)]]
 7872:               , tile_schedulers = tile_schedulers(data_type["sfd_type"]), gemm_kind=gemm_kind)
 7873:           elif math_inst.instruction_shape[0] == 128:
 7874:             CreateGemmUniversal3xOperator(manifest, [layout], [tile], [data_type],
 7875:               [[KernelScheduleType.Mxf8f6f4TmaWarpSpecialized2SmSm100, EpilogueScheduleType.TmaWarpSpecialized2Sm]]
 7876:               , tile_schedulers = tile_schedulers(data_type["sfd_type"]), gemm_kind=gemm_kind)
 7877:           else:
 7878:             CreateGemmUniversal3xOperator(manifest, [layout], [tile], [data_type],
 7879:               [[KernelScheduleType.Mxf8f6f4TmaWarpSpecialized2SmSm100, EpilogueScheduleType.ScheduleAuto]]
 7880:               , tile_schedulers = tile_schedulers(data_type["sfd_type"]), gemm_kind=gemm_kind)
```
**EN:** Defines `GenerateSM100_TensorOp_mixed_8bits_UMMA_gemm_with_block_scaled()`, which generates sm100 tensor op mixed 8bits umma gemm with block scaled. Key helper calls include get_instantiation_level, is_grouped, generate_mxf8f6f4_math_instructions_sm100, generate_cluster_shapes_sm100, ThorSMRenumbering, CudaToolkitVersionSatisfies.

**CN:** 定义 `GenerateSM100_TensorOp_mixed_8bits_UMMA_gemm_with_block_scaled()`，用于生成 sm100 tensor op mixed 8bits umma gemm with block scaled。 其中会调用的重要辅助函数包括 get_instantiation_level, is_grouped, generate_mxf8f6f4_math_instructions_sm100, generate_cluster_shapes_sm100, ThorSMRenumbering, CudaToolkitVersionSatisfies。

### L7884-L8181 — Function `GenerateSM100_TensorOp_fp4_UMMA_gemm_with_block_scaled`

```python
 7884: def GenerateSM100_TensorOp_fp4_UMMA_gemm_with_block_scaled(manifest, cuda_version, gemm_kind=GemmKind.BlockScaledUniversal3x):
 7885:   # SM100 MMA with F4 + block scale
 7886:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 8):
 7887:     return
 7888: 
 7889:   instantiation_level = manifest.get_instantiation_level(pruned_level=591, default_level=591, exhaustive_level=9999)
 7890: 
 7891:   grouped = is_grouped(gemm_kind)
 7892: 
 7893:   # layouts for ABC and their alignments.
 7894:   layouts = [
 7895:     [[LayoutType.RowMajor,    32], [LayoutType.ColumnMajor, 32], [LayoutType.RowMajor,    0]],
 7896:     [[LayoutType.RowMajor,    32], [LayoutType.ColumnMajor, 32], [LayoutType.ColumnMajor, 0]],
 7897:   ]
 7898: 
 7899:   math_instructions_1sm, math_instructions_2sm = generate_mxf4nvf4_math_instructions_sm100(instantiation_level, enable_runtime_dtype=not grouped)
 7900: 
 7901:   def change_priority_func(shapes_1sm, shapes_2sm):
 7902:     shapes_1sm[(1,2,1)] = 6
 7903:     shapes_1sm[(1,4,1)] = 6
 7904:     shapes_2sm[(2,2,1)] = 6
 7905:     shapes_2sm[(2,4,1)] = 6
 7906:     shapes_2sm[(4,2,1)] = 6
 7907: 
 7908:   cluster_shapes_1sm, cluster_shapes_2sm = generate_cluster_shapes_sm100(instantiation_level, change_priority_func=change_priority_func)
 7909: 
 7910:   acc_types = [ DataType.f32 ] # Accumulator is always 32 bits for block scaled MMA instructions
 7911: 
 7912:   def tile_schedulers(sfdtype):
 7913:     # Only use the stream-K scheduler for non-void SFD to limit kernel count. When SFD is void,
 7914:     # the epilogue is the traditional linear combination, for which we already have tests with stream-K.
 7915:     if sfdtype["type"] == DataType.void or grouped:
 7916:       return [TileSchedulerType.Default]
 7917:     else:
 7918:       return [TileSchedulerType.Default, TileSchedulerType.StreamK]
 7919: 
 7920:   thor_sm = ThorSMRenumbering(cuda_version)
 7921: 
 7922:   min_cc = 100
 7923:   max_cc = thor_sm
 7924: 
 7925:   epi_type = DataType.f32
 7926: 
 7927:   is_runtime_datatype = lambda runtime_datatype: runtime_datatype in (DataType.f4, DataType.f6, DataType.f8)
 7928: 
 7929:   if thor_sm in manifest.compute_capabilities_baseline :
 7930:     if [4,4,1] in cluster_shapes_1sm :
 7931:       cluster_shapes_1sm.remove([4,4,1])
 7932:     if [4,4,1] in cluster_shapes_2sm :
 7933:       cluster_shapes_2sm.remove([4,4,1])
 7934: 
 7935:   # 1xSM MMA kernels
 7936:   for math_inst in math_instructions_1sm:
 7937:     assert math_inst.opcode_class == OpcodeClass.BlockScaledTensorOp
 7938:     tile_descriptions = []
 7939:     for cluster_shape in cluster_shapes_1sm:
 7940:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 7941:       tile_descriptions.append(
 7942:         TileDescription([
 7943:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 7944:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 7945:           math_inst.instruction_shape[2] * 4 * multiplier_1sm[2]],
 7946:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 7947:       assert math_inst.instruction_shape[2] * 4 == 256
 7948: 
 7949:     data_types = [
 7950:       {
 7951:         "a_type"   : math_inst.element_a,
 7952:         "b_type"   : math_inst.element_b,
 7953:         "c_type"   : DataType.void,
 7954:         "d_type"   : DataType.f32,
 7955:         "acc_type" : math_inst.element_accumulator,
 7956:         "epi_type" : epi_type,
 7957:         "sf_type"  : math_inst.element_scale_factor,
 7958:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 7959:       },
 7960:       {
 7961:         "a_type"   : math_inst.element_a,
 7962:         "b_type"   : math_inst.element_b,
 7963:         "c_type"   : DataType.bf16,
 7964:         "d_type"   : DataType.bf16,
 7965:         "acc_type" : math_inst.element_accumulator,
 7966:         "epi_type" : epi_type,
 7967:         "sf_type"  : math_inst.element_scale_factor,
 7968:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 7969:       },
 7970:       {
 7971:         "a_type"   : math_inst.element_a,
 7972:         "b_type"   : math_inst.element_b,
 7973:         "c_type"   : DataType.void,
 7974:         "d_type"   : DataType.e2m1,
 7975:         "acc_type" : math_inst.element_accumulator,
 7976:         "epi_type" : epi_type,
 7977:         "sf_type"  : math_inst.element_scale_factor,
 7978:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
 7979:       },
 7980:       {
 7981:         "a_type"   : math_inst.element_a,
 7982:         "b_type"   : math_inst.element_b,
 7983:         "c_type"   : DataType.void,
 7984:         "d_type"   : DataType.e5m2,
 7985:         "acc_type" : math_inst.element_accumulator,
 7986:         "epi_type" : epi_type,
 7987:         "sf_type"  : math_inst.element_scale_factor,
 7988:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 7989:       },
 7990:       {
 7991:         "a_type"   : math_inst.element_a,
 7992:         "b_type"   : math_inst.element_b,
 7993:         "c_type"   : DataType.f16,
 7994:         "d_type"   : DataType.e5m2,
 7995:         "acc_type" : math_inst.element_accumulator,
 7996:         "epi_type" : epi_type,
 7997:         "sf_type"  : math_inst.element_scale_factor,
 7998:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 7999:       },
 8000:       {
 8001:         "a_type"   : math_inst.element_a,
 8002:         "b_type"   : math_inst.element_b,
 8003:         "c_type"   : DataType.void,
 8004:         "d_type"   : DataType.e2m1,
 8005:         "acc_type" : math_inst.element_accumulator,
 8006:         "epi_type" : epi_type,
 8007:         "sf_type"  : math_inst.element_scale_factor,
 8008:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 16, "layout" : LayoutType.RowMajor}
 8009:       },
 8010:       {
 8011:         "a_type"   : math_inst.element_a,
 8012:         "b_type"   : math_inst.element_b,
 8013:         "c_type"   : DataType.f16,
 8014:         "d_type"   : DataType.e2m1,
 8015:         "acc_type" : math_inst.element_accumulator,
 8016:         "epi_type" : epi_type,
 8017:         "sf_type"  : math_inst.element_scale_factor,
 8018:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 16, "layout" : LayoutType.RowMajor}
 8019:       },
 8020:       {
 8021:         "a_type"   : math_inst.element_a,
 8022:         "b_type"   : math_inst.element_b,
 8023:         "c_type"   : DataType.f16,
 8024:         "d_type"   : DataType.e2m1,
 8025:         "acc_type" : math_inst.element_accumulator,
 8026:         "epi_type" : epi_type,
 8027:         "sf_type"  : math_inst.element_scale_factor,
 8028:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
 8029:       }
 8030:     ]
 8031: 
 8032:     # Set alignment d based on Destination format.
 8033:     for layout in layouts:
 8034:       layout[2][1] = 128 // DataTypeSize[data_types[0]["d_type"]]
 8035: 
 8036:     for layout in layouts:
 8037:       for data_type in data_types:
 8038:         if (data_type["sfd_type"]["type"] != DataType.void) and (data_type["d_type"] == DataType.e2m1) and (layout[2][0] == LayoutType.RowMajor):
 8039:           data_type["sfd_type"]["layout"] = layout[2][0] # For FP4 output , the scalefactor layout is same layout as D layout.
 8040:         if (data_type["sfd_type"]["type"] != DataType.void) and (data_type["d_type"] == DataType.e2m1) and (layout[2][0] == LayoutType.ColumnMajor):
 8041:             continue
 8042: 
 8043:         # E2M1 x E2M1, vector size 32, E8
 8044:         # E2M1 x E2M1, vector size 16, UE4M3
 8045:         isFp4 = math_inst.element_scale_factor == DataType.ue8m0 and  math_inst.element_a == DataType.e2m1 and math_inst.element_b == DataType.e2m1
 8046:         epi_schedule = to_grouped_schedule(EpilogueScheduleType.TmaWarpSpecialized1Sm, grouped)
 8047:         epi_nosmem_schedule = to_grouped_schedule(EpilogueScheduleType.NoSmemWarpSpecialized1Sm, grouped)
 8048:         nvfp4_kernel_schedule = to_grouped_schedule(KernelScheduleType.Nvf4TmaWarpSpecialized1SmSm100, grouped)
 8049:         fp4_kernel_schedule = to_grouped_schedule(KernelScheduleType.Mxf4TmaWarpSpecialized1SmSm100, grouped)
 8050: 
 8051:         nvfp4_schedules = [[nvfp4_kernel_schedule, epi_schedule], [nvfp4_kernel_schedule, epi_nosmem_schedule]]
 8052:         fp4_schedules   = [[fp4_kernel_schedule, epi_schedule], [fp4_kernel_schedule, epi_nosmem_schedule]]
 8053:         CreateGemmUniversal3xOperator(manifest, [layout], tile_descriptions, data_type, nvfp4_schedules
 8054:           , tile_schedulers=tile_schedulers(data_type["sfd_type"]), gemm_kind=gemm_kind
 8055:           )
 8056:         if isFp4:
 8057:           CreateGemmUniversal3xOperator(manifest, [layout], tile_descriptions, data_type, fp4_schedules
 8058:           , tile_schedulers=tile_schedulers(data_type["sfd_type"]), gemm_kind=gemm_kind
 8059:           )
 8060: 
 8061:   for math_inst in math_instructions_2sm:
 8062:     assert math_inst.opcode_class == OpcodeClass.BlockScaledTensorOp
 8063:     tile_descriptions = []
 8064:     for cluster_shape in cluster_shapes_2sm:
 8065:       multiplier_2sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 8066:       tile_descriptions.append(
 8067:         TileDescription([
 8068:           math_inst.instruction_shape[0]     * multiplier_2sm[0],
 8069:           math_inst.instruction_shape[1]     * multiplier_2sm[1],
 8070:           math_inst.instruction_shape[2] * 4 * multiplier_2sm[2]],
 8071:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 8072: 
 8073:     data_types = [
 8074:       {
 8075:         "a_type"   : math_inst.element_a,
 8076:         "b_type"   : math_inst.element_b,
 8077:         "c_type"   : DataType.void,
 8078:         "d_type"   : DataType.f32,
 8079:         "acc_type" : math_inst.element_accumulator,
 8080:         "epi_type" : epi_type,
 8081:         "sf_type"  : math_inst.element_scale_factor,
 8082:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8083:       },
 8084:       {
 8085:         "a_type"   : math_inst.element_a,
 8086:         "b_type"   : math_inst.element_b,
 8087:         "c_type"   : DataType.bf16,
 8088:         "d_type"   : DataType.bf16,
 8089:         "acc_type" : math_inst.element_accumulator,
 8090:         "epi_type" : epi_type,
 8091:         "sf_type"  : math_inst.element_scale_factor,
 8092:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8093:       },
 8094:       {
 8095:         "a_type"   : math_inst.element_a,
 8096:         "b_type"   : math_inst.element_b,
 8097:         "c_type"   : DataType.void,
 8098:         "d_type"   : DataType.e2m1,
 8099:         "acc_type" : math_inst.element_accumulator,
 8100:         "epi_type" : epi_type,
 8101:         "sf_type"  : math_inst.element_scale_factor,
 8102:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
 8103:       },
 8104:       {
 8105:         "a_type"   : math_inst.element_a,
 8106:         "b_type"   : math_inst.element_b,
 8107:         "c_type"   : DataType.void,
 8108:         "d_type"   : DataType.e5m2,
 8109:         "acc_type" : math_inst.element_accumulator,
 8110:         "epi_type" : epi_type,
 8111:         "sf_type"  : math_inst.element_scale_factor,
 8112:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8113:       },
 8114:       {
 8115:         "a_type"   : math_inst.element_a,
 8116:         "b_type"   : math_inst.element_b,
 8117:         "c_type"   : DataType.f16,
 8118:         "d_type"   : DataType.e5m2,
 8119:         "acc_type" : math_inst.element_accumulator,
 8120:         "epi_type" : epi_type,
 8121:         "sf_type"  : math_inst.element_scale_factor,
 8122:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8123:       },
 8124:       {
 8125:         "a_type"   : math_inst.element_a,
 8126:         "b_type"   : math_inst.element_b,
 8127:         "c_type"   : DataType.void,
 8128:         "d_type"   : DataType.e2m1,
 8129:         "acc_type" : math_inst.element_accumulator,
 8130:         "epi_type" : epi_type,
 8131:         "sf_type"  : math_inst.element_scale_factor,
 8132:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 16, "layout" : LayoutType.RowMajor}
 8133:       },
 8134:       {
 8135:         "a_type"   : math_inst.element_a,
 8136:         "b_type"   : math_inst.element_b,
 8137:         "c_type"   : DataType.f16,
 8138:         "d_type"   : DataType.e2m1,
 8139:         "acc_type" : math_inst.element_accumulator,
 8140:         "epi_type" : epi_type,
 8141:         "sf_type"  : math_inst.element_scale_factor,
 8142:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 16, "layout" : LayoutType.RowMajor}
 8143:       },
 8144:       {
 8145:         "a_type"   : math_inst.element_a,
 8146:         "b_type"   : math_inst.element_b,
 8147:         "c_type"   : DataType.f16,
 8148:         "d_type"   : DataType.e2m1,
 8149:         "acc_type" : math_inst.element_accumulator,
 8150:         "epi_type" : epi_type,
 8151:         "sf_type"  : math_inst.element_scale_factor,
 8152:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
 8153:       }
 8154:     ]
 8155: 
 8156:     # Set alignment d based on Destination format.
 8157:     for layout in layouts:
 8158:       layout[2][1] = 128 // DataTypeSize[data_types[0]["d_type"]]
 8159: 
 8160:     for layout in layouts:
 8161:       for data_type in data_types:
 8162:         if (data_type["sfd_type"]["type"] != DataType.void) and (data_type["d_type"] == DataType.e2m1) and (layout[2][0] == LayoutType.RowMajor):
 8163:           data_type["sfd_type"]["layout"] = layout[2][0] # For FP4 output , the scalefactor layout is same layout as D layout.
 8164:         if (data_type["sfd_type"]["type"] != DataType.void) and (data_type["d_type"] == DataType.e2m1) and (layout[2][0] == LayoutType.ColumnMajor):
 8165:             continue
 8166: 
 8167:         # E2M1 x E2M1, vector size 32, E8
 8168:         isFp4 = math_inst.element_scale_factor == DataType.ue8m0 and  math_inst.element_a == DataType.e2m1 and math_inst.element_b == DataType.e2m1
 8169: 
 8170:         epi_schedule = EpilogueScheduleType.ScheduleAuto if not grouped else EpilogueScheduleType.PtrArrayTmaWarpSpecialized2Sm
 8171:         epi_nosmem_schedule = to_grouped_schedule(EpilogueScheduleType.NoSmemWarpSpecialized2Sm, grouped)
 8172:         nvfp4_kernel_schedule = to_grouped_schedule(KernelScheduleType.Nvf4TmaWarpSpecialized2SmSm100, grouped)
 8173:         fp4_kernel_schedule = to_grouped_schedule(KernelScheduleType.Mxf4TmaWarpSpecialized2SmSm100, grouped)
 8174: 
 8175:         nvfp4_schedules = [[nvfp4_kernel_schedule, epi_schedule], [nvfp4_kernel_schedule, epi_nosmem_schedule]]
 8176:         fp4_schedules   = [[fp4_kernel_schedule, epi_schedule], [fp4_kernel_schedule, epi_nosmem_schedule]]
 8177:         CreateGemmUniversal3xOperator(manifest, [layout], tile_descriptions, data_type, nvfp4_schedules
 8178:           , tile_schedulers=tile_schedulers(data_type["sfd_type"]), gemm_kind=gemm_kind)
 8179:         if isFp4:
 8180:           CreateGemmUniversal3xOperator(manifest, [layout], tile_descriptions, data_type, fp4_schedules
 8181:           , tile_schedulers=tile_schedulers(data_type["sfd_type"]), gemm_kind=gemm_kind)
```
**EN:** Defines `GenerateSM100_TensorOp_fp4_UMMA_gemm_with_block_scaled()`, which generates sm100 tensor op fp4 umma gemm with block scaled. Key helper calls include get_instantiation_level, is_grouped, generate_mxf4nvf4_math_instructions_sm100, generate_cluster_shapes_sm100, ThorSMRenumbering, CudaToolkitVersionSatisfies.

**CN:** 定义 `GenerateSM100_TensorOp_fp4_UMMA_gemm_with_block_scaled()`，用于生成 sm100 tensor op fp4 umma gemm with block scaled。 其中会调用的重要辅助函数包括 get_instantiation_level, is_grouped, generate_mxf4nvf4_math_instructions_sm100, generate_cluster_shapes_sm100, ThorSMRenumbering, CudaToolkitVersionSatisfies。

### L8183-L8564 — Function `GenerateSM103_TensorOp_fp4_ultra_UMMA_gemm_with_block_scaled`

```python
 8183: def GenerateSM103_TensorOp_fp4_ultra_UMMA_gemm_with_block_scaled(manifest, cuda_version, gemm_kind=GemmKind.BlockScaledUniversal3x):
 8184:   # SM100 MMA with F4 + block scale
 8185:   if not CudaToolkitVersionSatisfies(cuda_version, 13, 0):
 8186:     return
 8187: 
 8188:   grouped = is_grouped(gemm_kind)
 8189: 
 8190:   # layouts for ABC and their alignments.
 8191:   layouts = [
 8192:     [[LayoutType.RowMajor,    32], [LayoutType.ColumnMajor, 32], [LayoutType.RowMajor,    0]],
 8193:     [[LayoutType.RowMajor,    32], [LayoutType.ColumnMajor, 32], [LayoutType.ColumnMajor, 0]],
 8194:   ]
 8195: 
 8196:   instruction_sizes_1sm = [
 8197:     [128, 128, 96], 
 8198:   ]
 8199: 
 8200:   instruction_sizes_2sm = [
 8201:     [256, 128, 96], 
 8202:     [256, 192, 96],
 8203:     [256, 256, 96]
 8204:   ]
 8205: 
 8206:   ab_types  = [
 8207:     DataType.f4,
 8208:     DataType.e2m1, 
 8209:   ]
 8210: 
 8211:   sf_types  = [
 8212:     DataType.ue4m3,
 8213:     DataType.ue8m0
 8214:   ]
 8215: 
 8216:   acc_types = [ DataType.f32 ] # Accumulator is always 32 bits for block scaled MMA instructions
 8217: 
 8218:   def tile_schedulers(sfdtype):
 8219:     # Only use the stream-K scheduler for non-void SFD to limit kernel count. When SFD is void,
 8220:     # the epilogue is the traditional linear combination, for which we already have tests with stream-K.
 8221:     if grouped:
 8222:       return [TileSchedulerType.Default]
 8223:     if sfdtype["type"] == DataType.void:
 8224:       return [TileSchedulerType.Default]
 8225:     else:
 8226:       return [TileSchedulerType.Default, TileSchedulerType.StreamK]
 8227: 
 8228:   min_cc = 103
 8229:   max_cc = 103
 8230:   epi_type = DataType.f32
 8231: 
 8232:   math_instructions_1sm = []
 8233: 
 8234:   is_runtime_datatype = lambda runtime_datatype: runtime_datatype in (DataType.f4, DataType.f6, DataType.f8)
 8235: 
 8236:   for instr_size, a_type, b_type, sf_type, acc_type in product(instruction_sizes_1sm, ab_types, ab_types, sf_types, acc_types):
 8237:     is_runtime_datatype_a = is_runtime_datatype(a_type)
 8238:     is_runtime_datatype_b = is_runtime_datatype(b_type)
 8239: 
 8240:     # A/B datatypes should be both static or dynamic
 8241:     if (is_runtime_datatype_a != is_runtime_datatype_b):
 8242:       continue
 8243: 
 8244:     math_instructions_1sm.append(
 8245:       MathInstruction(
 8246:         instr_size,
 8247:         a_type, b_type, acc_type,
 8248:         OpcodeClass.BlockScaledTensorOp,
 8249:         MathOperation.multiply_add,
 8250:         sf_type)
 8251:     )
 8252: 
 8253:   math_instructions_2sm = []
 8254: 
 8255:   for instr_size, a_type, b_type, sf_type, acc_type in product(instruction_sizes_2sm, ab_types, ab_types, sf_types, acc_types):
 8256:     is_runtime_datatype_a = is_runtime_datatype(a_type)
 8257:     is_runtime_datatype_b = is_runtime_datatype(b_type)
 8258: 
 8259:     # A/B datatypes should be both static or dynamic
 8260:     if (is_runtime_datatype_a != is_runtime_datatype_b):
 8261:       continue
 8262: 
 8263:     math_instructions_2sm.append(
 8264:       MathInstruction(
 8265:         instr_size,
 8266:         a_type, b_type, acc_type,
 8267:         OpcodeClass.BlockScaledTensorOp,
 8268:         MathOperation.multiply_add,
 8269:         sf_type)
 8270:     )
 8271: 
 8272:   cluster_shapes_1sm = [
 8273:     [1,1,1],
 8274:     # [1,2,1],
 8275:     [2,1,1],
 8276:     # [1,4,1],
 8277:     [4,4,1],
 8278:     DynamicClusterShape
 8279:   ]
 8280: 
 8281:   # 1xSM MMA kernels
 8282:   for math_inst in math_instructions_1sm:
 8283:     tile_descriptions = []
 8284:     for cluster_shape in cluster_shapes_1sm:
 8285:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 8286:       tile_descriptions.append(
 8287:         TileDescription([
 8288:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 8289:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 8290:           768],
 8291:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 8292: 
 8293:     data_types = [
 8294:       {
 8295:         "a_type"   : math_inst.element_a,
 8296:         "b_type"   : math_inst.element_b,
 8297:         "c_type"   : DataType.void,
 8298:         "d_type"   : DataType.f32,
 8299:         "acc_type" : math_inst.element_accumulator,
 8300:         "epi_type" : epi_type,
 8301:         "sf_type"  : math_inst.element_scale_factor,
 8302:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8303:       },
 8304:       {
 8305:         "a_type"   : math_inst.element_a,
 8306:         "b_type"   : math_inst.element_b,
 8307:         "c_type"   : DataType.bf16,
 8308:         "d_type"   : DataType.bf16,
 8309:         "acc_type" : math_inst.element_accumulator,
 8310:         "epi_type" : epi_type,
 8311:         "sf_type"  : math_inst.element_scale_factor,
 8312:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8313:       },
 8314:       {
 8315:         "a_type"   : math_inst.element_a,
 8316:         "b_type"   : math_inst.element_b,
 8317:         "c_type"   : DataType.void,
 8318:         "d_type"   : DataType.e2m1,
 8319:         "acc_type" : math_inst.element_accumulator,
 8320:         "epi_type" : epi_type,
 8321:         "sf_type"  : math_inst.element_scale_factor,
 8322:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
 8323:       },
 8324:       {
 8325:         "a_type"   : math_inst.element_a,
 8326:         "b_type"   : math_inst.element_b,
 8327:         "c_type"   : DataType.void,
 8328:         "d_type"   : DataType.e5m2,
 8329:         "acc_type" : math_inst.element_accumulator,
 8330:         "epi_type" : epi_type,
 8331:         "sf_type"  : math_inst.element_scale_factor,
 8332:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8333:       },
 8334:       {
 8335:         "a_type"   : math_inst.element_a,
 8336:         "b_type"   : math_inst.element_b,
 8337:         "c_type"   : DataType.void,
 8338:         "d_type"   : DataType.f16,
 8339:         "acc_type" : math_inst.element_accumulator,
 8340:         "epi_type" : epi_type,
 8341:         "sf_type"  : math_inst.element_scale_factor,
 8342:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8343:       },
 8344:       {
 8345:         "a_type"   : math_inst.element_a,
 8346:         "b_type"   : math_inst.element_b,
 8347:         "c_type"   : DataType.f16,
 8348:         "d_type"   : DataType.e5m2,
 8349:         "acc_type" : math_inst.element_accumulator,
 8350:         "epi_type" : epi_type,
 8351:         "sf_type"  : math_inst.element_scale_factor,
 8352:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8353:       },
 8354:       {
 8355:         "a_type"   : math_inst.element_a,
 8356:         "b_type"   : math_inst.element_b,
 8357:         "c_type"   : DataType.void,
 8358:         "d_type"   : DataType.e2m1,
 8359:         "acc_type" : math_inst.element_accumulator,
 8360:         "epi_type" : epi_type,
 8361:         "sf_type"  : math_inst.element_scale_factor,
 8362:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 16, "layout" : LayoutType.RowMajor}
 8363:       },
 8364:       {
 8365:         "a_type"   : math_inst.element_a,
 8366:         "b_type"   : math_inst.element_b,
 8367:         "c_type"   : DataType.f16,
 8368:         "d_type"   : DataType.e2m1,
 8369:         "acc_type" : math_inst.element_accumulator,
 8370:         "epi_type" : epi_type,
 8371:         "sf_type"  : math_inst.element_scale_factor,
 8372:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 16, "layout" : LayoutType.RowMajor}
 8373:       },
 8374:       {
 8375:         "a_type"   : math_inst.element_a,
 8376:         "b_type"   : math_inst.element_b,
 8377:         "c_type"   : DataType.f16,
 8378:         "d_type"   : DataType.e2m1,
 8379:         "acc_type" : math_inst.element_accumulator,
 8380:         "epi_type" : epi_type,
 8381:         "sf_type"  : math_inst.element_scale_factor,
 8382:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
 8383:       }
 8384:     ]
 8385: 
 8386:     # Set alignment d based on Destination format.
 8387:     for layout in layouts:
 8388:       for data_type in data_types:
 8389:         # Set alignment d based on Destination format.
 8390:         if DataTypeSize[data_type["c_type"]] == 0 :
 8391:           layout[2][1] = 256 // DataTypeSize[data_type["d_type"]]
 8392:         else:
 8393:           layout[2][1] = min(256 // DataTypeSize[data_type["d_type"]], 256 // DataTypeSize[data_type["c_type"]])
 8394:         
 8395:         if data_type["sfd_type"]["type"] != DataType.void and (data_type["d_type"] == DataType.e2m1) and (layout[2][0] == LayoutType.RowMajor):
 8396:           data_type["sfd_type"]["layout"] = layout[2][0] # For FP4 output , the scalefactor layout is same layout as D layout.
 8397:         if (data_type["sfd_type"]["type"] != DataType.void) and (data_type["d_type"] == DataType.e2m1) and (layout[2][0] == LayoutType.ColumnMajor):
 8398:             continue
 8399:         #   E2M1 x E2M1, vector size 32, E8
 8400:         isFp4 = math_inst.element_scale_factor == DataType.ue8m0 and  math_inst.element_a == DataType.e2m1 and math_inst.element_b == DataType.e2m1
 8401: 
 8402:         epilogue_1sm_schedule = to_grouped_schedule(EpilogueScheduleType.NoSmemWarpSpecialized1Sm, grouped)
 8403: 
 8404:         nvfp4_schedule                  = [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103, grouped), epilogue_1sm_schedule]              
 8405:         nvfp4_schedule_disable_prefetch = [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103DisablePrefetch, grouped), epilogue_1sm_schedule]                
 8406:         nvfp4_schedule_tma_prefetch     = [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103TmaPrefetch, grouped), epilogue_1sm_schedule]
 8407:         fp4_schedule                    = [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103, grouped), epilogue_1sm_schedule]
 8408:         fp4_schedule_disable_prefetch   = [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103DisablePrefetch, grouped), epilogue_1sm_schedule]
 8409:         fp4_schedule_tma_prefetch       = [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103TmaPrefetch, grouped), epilogue_1sm_schedule]
 8410:         nvfp4_schedules = [nvfp4_schedule, nvfp4_schedule_disable_prefetch, nvfp4_schedule_tma_prefetch]
 8411:         fp4_schedules   = [fp4_schedule, fp4_schedule_disable_prefetch, fp4_schedule_tma_prefetch]
 8412: 
 8413:         CreateGemmUniversal3xOperator(manifest, [layout], tile_descriptions, data_type, 
 8414:                                       nvfp4_schedules, tile_schedulers=tile_schedulers(data_type["sfd_type"]), gemm_kind=gemm_kind)
 8415:         if isFp4:
 8416:           CreateGemmUniversal3xOperator(manifest, [layout], tile_descriptions, data_type,
 8417:                                         fp4_schedules, tile_schedulers=tile_schedulers(data_type["sfd_type"]), gemm_kind=gemm_kind)
 8418: 
 8419:   cluster_shapes_2sm = [
 8420:     [2,1,1],
 8421:     # [2,2,1],
 8422:     # [2,4,1],
 8423:     [4,1,1],
 8424:     # [4,2,1],
 8425:     [4,4,1],
 8426:     DynamicClusterShape
 8427:   ]
 8428: 
 8429:   for math_inst in math_instructions_2sm:
 8430:     tile_descriptions = []
 8431:     for cluster_shape in cluster_shapes_2sm:
 8432:       multiplier_2sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 8433:       tile_descriptions.append(
 8434:         TileDescription([
 8435:           math_inst.instruction_shape[0]     * multiplier_2sm[0],
 8436:           math_inst.instruction_shape[1]     * multiplier_2sm[1],
 8437:           math_inst.instruction_shape[2] * 8 * multiplier_2sm[2]],
 8438:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 8439: 
 8440:     data_types = [
 8441:       {
 8442:         "a_type"   : math_inst.element_a,
 8443:         "b_type"   : math_inst.element_b,
 8444:         "c_type"   : DataType.void,
 8445:         "d_type"   : DataType.f32,
 8446:         "acc_type" : math_inst.element_accumulator,
 8447:         "epi_type" : epi_type,
 8448:         "sf_type"  : math_inst.element_scale_factor,
 8449:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8450:       },
 8451:       {
 8452:         "a_type"   : math_inst.element_a,
 8453:         "b_type"   : math_inst.element_b,
 8454:         "c_type"   : DataType.bf16,
 8455:         "d_type"   : DataType.bf16,
 8456:         "acc_type" : math_inst.element_accumulator,
 8457:         "epi_type" : epi_type,
 8458:         "sf_type"  : math_inst.element_scale_factor,
 8459:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8460:       },
 8461:       {
 8462:         "a_type"   : math_inst.element_a,
 8463:         "b_type"   : math_inst.element_b,
 8464:         "c_type"   : DataType.void,
 8465:         "d_type"   : DataType.e2m1,
 8466:         "acc_type" : math_inst.element_accumulator,
 8467:         "epi_type" : epi_type,
 8468:         "sf_type"  : math_inst.element_scale_factor,
 8469:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
 8470:       },
 8471:       {
 8472:         "a_type"   : math_inst.element_a,
 8473:         "b_type"   : math_inst.element_b,
 8474:         "c_type"   : DataType.void,
 8475:         "d_type"   : DataType.e5m2,
 8476:         "acc_type" : math_inst.element_accumulator,
 8477:         "epi_type" : epi_type,
 8478:         "sf_type"  : math_inst.element_scale_factor,
 8479:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8480:       },
 8481:       {
 8482:         "a_type"   : math_inst.element_a,
 8483:         "b_type"   : math_inst.element_b,
 8484:         "c_type"   : DataType.void,
 8485:         "d_type"   : DataType.f16,
 8486:         "acc_type" : math_inst.element_accumulator,
 8487:         "epi_type" : epi_type,
 8488:         "sf_type"  : math_inst.element_scale_factor,
 8489:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8490:       },
 8491:       {
 8492:         "a_type"   : math_inst.element_a,
 8493:         "b_type"   : math_inst.element_b,
 8494:         "c_type"   : DataType.f16,
 8495:         "d_type"   : DataType.e5m2,
 8496:         "acc_type" : math_inst.element_accumulator,
 8497:         "epi_type" : epi_type,
 8498:         "sf_type"  : math_inst.element_scale_factor,
 8499:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 8500:       },
 8501:       {
 8502:         "a_type"   : math_inst.element_a,
 8503:         "b_type"   : math_inst.element_b,
 8504:         "c_type"   : DataType.void,
 8505:         "d_type"   : DataType.e2m1,
 8506:         "acc_type" : math_inst.element_accumulator,
 8507:         "epi_type" : epi_type,
 8508:         "sf_type"  : math_inst.element_scale_factor,
 8509:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 16, "layout" : LayoutType.RowMajor}
 8510:       },
 8511:       {
 8512:         "a_type"   : math_inst.element_a,
 8513:         "b_type"   : math_inst.element_b,
 8514:         "c_type"   : DataType.f16,
 8515:         "d_type"   : DataType.e2m1,
 8516:         "acc_type" : math_inst.element_accumulator,
 8517:         "epi_type" : epi_type,
 8518:         "sf_type"  : math_inst.element_scale_factor,
 8519:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 16, "layout" : LayoutType.RowMajor}
 8520:       },
 8521:       {
 8522:         "a_type"   : math_inst.element_a,
 8523:         "b_type"   : math_inst.element_b,
 8524:         "c_type"   : DataType.f16,
 8525:         "d_type"   : DataType.e2m1,
 8526:         "acc_type" : math_inst.element_accumulator,
 8527:         "epi_type" : epi_type,
 8528:         "sf_type"  : math_inst.element_scale_factor,
 8529:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
 8530:       }
 8531:     ]
 8532: 
 8533:     # Set alignment d based on Destination format.
 8534:     for layout in layouts:
 8535:       for data_type in data_types:
 8536:         # Set alignment d based on Destination format.
 8537:         if DataTypeSize[data_type["c_type"]] == 0 :
 8538:           layout[2][1] = 256 // DataTypeSize[data_type["d_type"]]
 8539:         else:
 8540:           layout[2][1] = min(256 // DataTypeSize[data_type["d_type"]], 256 // DataTypeSize[data_type["c_type"]])
 8541:         
 8542:         if data_type["sfd_type"]["type"] != DataType.void and (data_type["d_type"] == DataType.e2m1) and (layout[2][0] == LayoutType.RowMajor):
 8543:           data_type["sfd_type"]["layout"] = layout[2][0] # For FP4 output , the scalefactor layout is same layout as D layout.
 8544:         if (data_type["sfd_type"]["type"] != DataType.void) and (data_type["d_type"] == DataType.e2m1) and (layout[2][0] == LayoutType.ColumnMajor):
 8545:             continue
 8546:         #   E2M1 x E2M1, vector size 32, E8
 8547:         isFp4 = math_inst.element_scale_factor == DataType.ue8m0 and  math_inst.element_a == DataType.e2m1 and math_inst.element_b == DataType.e2m1
 8548: 
 8549:         epilogue_2sm_schedule = to_grouped_schedule(EpilogueScheduleType.NoSmemWarpSpecialized2Sm, grouped)
 8550: 
 8551:         nvfp4_schedule                  = [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103, grouped), epilogue_2sm_schedule]              
 8552:         nvfp4_schedule_disable_prefetch = [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103DisablePrefetch, grouped), epilogue_2sm_schedule]                
 8553:         nvfp4_schedule_tma_prefetch     = [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103TmaPrefetch, grouped), epilogue_2sm_schedule]
 8554:         fp4_schedule                    = [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103, grouped), epilogue_2sm_schedule]
 8555:         fp4_schedule_disable_prefetch   = [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103DisablePrefetch, grouped), epilogue_2sm_schedule]
 8556:         fp4_schedule_tma_prefetch       = [to_grouped_schedule(KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103TmaPrefetch, grouped), epilogue_2sm_schedule]
 8557:         nvfp4_schedules = [nvfp4_schedule, nvfp4_schedule_disable_prefetch, nvfp4_schedule_tma_prefetch]
 8558:         fp4_schedules   = [fp4_schedule, fp4_schedule_disable_prefetch, fp4_schedule_tma_prefetch]
 8559: 
 8560:         CreateGemmUniversal3xOperator(manifest, [layout], tile_descriptions, data_type, 
 8561:                                       nvfp4_schedules, tile_schedulers=tile_schedulers(data_type["sfd_type"]), gemm_kind=gemm_kind)
 8562:         if isFp4:
 8563:           CreateGemmUniversal3xOperator(manifest, [layout], tile_descriptions, data_type,
 8564:                                         fp4_schedules, tile_schedulers=tile_schedulers(data_type["sfd_type"]), gemm_kind=gemm_kind)
```
**EN:** Defines `GenerateSM103_TensorOp_fp4_ultra_UMMA_gemm_with_block_scaled()`, which generates sm103 tensor op fp4 ultra umma gemm with block scaled. Key helper calls include is_grouped, product, CudaToolkitVersionSatisfies, is_runtime_datatype, append, MathInstruction.

**CN:** 定义 `GenerateSM103_TensorOp_fp4_ultra_UMMA_gemm_with_block_scaled()`，用于生成 sm103 tensor op fp4 ultra umma gemm with block scaled。 其中会调用的重要辅助函数包括 is_grouped, product, CudaToolkitVersionSatisfies, is_runtime_datatype, append, MathInstruction。

### L8567-L8787 — Function `GenerateSM100_TensorOp_int8_UMMA_gemm`

```python
 8567: def GenerateSM100_TensorOp_int8_UMMA_gemm(manifest, cuda_version):
 8568:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 8):
 8569:     return
 8570: 
 8571:   # layouts for ABC and their alignments.
 8572:   layouts = [
 8573:     [[LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 0]],
 8574:     [[LayoutType.ColumnMajor, 16], [LayoutType.RowMajor,    16], [LayoutType.ColumnMajor, 0]],
 8575:     [[LayoutType.RowMajor,    16], [LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 0]],
 8576:     [[LayoutType.RowMajor,    16], [LayoutType.RowMajor,    16], [LayoutType.ColumnMajor, 0]],
 8577:     [[LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.RowMajor,    0]],
 8578:     [[LayoutType.ColumnMajor, 16], [LayoutType.RowMajor,    16], [LayoutType.RowMajor,    0]],
 8579:     [[LayoutType.RowMajor,    16], [LayoutType.ColumnMajor, 16], [LayoutType.RowMajor,    0]],
 8580:     [[LayoutType.RowMajor,    16], [LayoutType.RowMajor,    16], [LayoutType.RowMajor,    0]],
 8581:   ]
 8582: 
 8583:   thor_sm = ThorSMRenumbering(cuda_version)
 8584: 
 8585:   min_cc = 100
 8586:   max_cc = thor_sm
 8587: 
 8588:   epi_type = DataType.f32
 8589: 
 8590:   math_instructions_1sm = [
 8591:     MathInstruction(
 8592:       [64, 128, 32],
 8593:       DataType.s8, DataType.s8, DataType.s32,
 8594:       OpcodeClass.TensorOp,
 8595:       MathOperation.multiply_add),
 8596:     MathInstruction(
 8597:       [128, 128, 32],
 8598:       DataType.s8, DataType.s8, DataType.s32,
 8599:       OpcodeClass.TensorOp,
 8600:       MathOperation.multiply_add),
 8601:     MathInstruction(
 8602:       [128, 256, 32],
 8603:       DataType.s8, DataType.s8, DataType.s32,
 8604:       OpcodeClass.TensorOp,
 8605:       MathOperation.multiply_add)]
 8606: 
 8607:   cluster_shapes_1sm = [[1,2,1], [2,1,1], [1,1,1], [1,4,1], [4,4,1]
 8608:                         , DynamicClusterShape
 8609:                        ]
 8610: 
 8611:   if thor_sm in manifest.compute_capabilities_baseline :
 8612:     cluster_shapes_1sm = [[1,2,1], [2,1,1], [1,1,1], [1,4,1]
 8613:                           , DynamicClusterShape
 8614:                          ]                    
 8615: 
 8616:   tile_schedulers = [
 8617:     TileSchedulerType.Default, TileSchedulerType.StreamK
 8618:   ]
 8619: 
 8620:   # 1xSM MMA kernels
 8621:   for math_inst in math_instructions_1sm:
 8622:     tile_descriptions = []
 8623:     for cluster_shape in cluster_shapes_1sm:
 8624:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 8625:       tile_descriptions.append(
 8626:         TileDescription([
 8627:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 8628:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 8629:           math_inst.instruction_shape[2] * 4 * multiplier_1sm[2]],
 8630:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 8631: 
 8632:     data_types = [
 8633:       {
 8634:         "a_type"   : math_inst.element_a,
 8635:         "b_type"   : math_inst.element_b,
 8636:         "c_type"   : math_inst.element_accumulator,
 8637:         "d_type"   : math_inst.element_accumulator,
 8638:         "acc_type" : math_inst.element_accumulator,
 8639:         "epi_type" : math_inst.element_accumulator,
 8640:       },
 8641:       {
 8642:         "a_type"   : math_inst.element_a,
 8643:         "b_type"   : math_inst.element_b,
 8644:         "c_type"   : DataType.void,
 8645:         "d_type"   : math_inst.element_accumulator,
 8646:         "acc_type" : math_inst.element_accumulator,
 8647:         "epi_type" : math_inst.element_accumulator,
 8648:       },
 8649:     ]
 8650:     # Set alignment d based on Destination format.
 8651:     for layout in layouts:
 8652:       layout[2][1] = 128 // DataTypeSize[data_types[0]["d_type"]]
 8653: 
 8654:     CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_types,
 8655:       [[KernelScheduleType.TmaWarpSpecialized1SmSm100, EpilogueScheduleType.TmaWarpSpecialized1Sm]],
 8656:       tile_schedulers=tile_schedulers)
 8657: 
 8658:     # for mixed precision kernels, also generate kernels that write output matrix in the A/B format
 8659:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 8660:     if math_inst.element_a != math_inst.element_accumulator:
 8661:       data_types_mixed = [
 8662:         {
 8663:           "a_type"   : math_inst.element_a,
 8664:           "b_type"   : math_inst.element_b,
 8665:           "c_type"   : math_inst.element_a,
 8666:           "d_type"   : math_inst.element_a,
 8667:           "acc_type" : math_inst.element_accumulator,
 8668:           "epi_type" : epi_type,
 8669:         },
 8670:         {
 8671:           "a_type"   : math_inst.element_a,
 8672:           "b_type"   : math_inst.element_b,
 8673:           "c_type"   : DataType.void,
 8674:           "d_type"   : math_inst.element_a,
 8675:           "acc_type" : math_inst.element_accumulator,
 8676:           "epi_type" : epi_type,
 8677:         },
 8678:       ]
 8679:       # Set alignment d based on Destination format.
 8680:       for layout in layouts:
 8681:         layout[2][1] = 128 // DataTypeSize[data_types_mixed[0]["d_type"]]
 8682: 
 8683:       CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_types_mixed,
 8684:         [[KernelScheduleType.TmaWarpSpecialized1SmSm100, EpilogueScheduleType.TmaWarpSpecialized1Sm]],
 8685:         tile_schedulers=tile_schedulers)
 8686: 
 8687:   # 2xSM MMA kernels
 8688:   math_instructions_2sm = [
 8689:     MathInstruction(
 8690:       [128, 128, 32],
 8691:       DataType.s8, DataType.s8, DataType.s32,
 8692:       OpcodeClass.TensorOp,
 8693:       MathOperation.multiply_add),
 8694:     MathInstruction(
 8695:       [128, 256, 32],
 8696:       DataType.s8, DataType.s8, DataType.s32,
 8697:       OpcodeClass.TensorOp,
 8698:       MathOperation.multiply_add),
 8699:     MathInstruction(
 8700:       [256, 128, 32],
 8701:       DataType.s8, DataType.s8, DataType.s32,
 8702:       OpcodeClass.TensorOp,
 8703:       MathOperation.multiply_add),
 8704:     MathInstruction(
 8705:       [256, 256, 32],
 8706:       DataType.s8, DataType.s8, DataType.s32,
 8707:       OpcodeClass.TensorOp,
 8708:       MathOperation.multiply_add),
 8709:   ]
 8710: 
 8711:   cluster_shapes_2sm = [[2,1,1], [2,2,1], [2,4,1], [4,1,1], [4,2,1], [4,4,1]
 8712:                         , DynamicClusterShape
 8713:                        ]
 8714: 
 8715:   if thor_sm in manifest.compute_capabilities_baseline :
 8716:     cluster_shapes_2sm = [[2,1,1], [2,2,1], [2,4,1], [4,1,1], [4,2,1]
 8717:                           , DynamicClusterShape
 8718:                          ]
 8719: 
 8720:   for math_inst in math_instructions_2sm:
 8721:     tile_descriptions = []
 8722:     for cluster_shape in cluster_shapes_2sm:
 8723:       multiplier_2sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 8724:       tile_descriptions.append(
 8725:         TileDescription([
 8726:           math_inst.instruction_shape[0]     * multiplier_2sm[0],
 8727:           math_inst.instruction_shape[1]     * multiplier_2sm[1],
 8728:           math_inst.instruction_shape[2] * 4 * multiplier_2sm[2]],
 8729:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 8730: 
 8731:     data_types = [
 8732:       {
 8733:         "a_type"   : math_inst.element_a,
 8734:         "b_type"   : math_inst.element_b,
 8735:         "c_type"   : math_inst.element_accumulator,
 8736:         "d_type"   : math_inst.element_accumulator,
 8737:         "acc_type" : math_inst.element_accumulator,
 8738:         "epi_type" : math_inst.element_accumulator,
 8739:       },
 8740:       {
 8741:         "a_type"   : math_inst.element_a,
 8742:         "b_type"   : math_inst.element_b,
 8743:         "c_type"   : DataType.void,
 8744:         "d_type"   : math_inst.element_accumulator,
 8745:         "acc_type" : math_inst.element_accumulator,
 8746:         "epi_type" : math_inst.element_accumulator,
 8747:       },
 8748:     ]
 8749:     # Set alignment d based on Destination format.
 8750:     for layout in layouts:
 8751:       layout[2][1] = 128 // DataTypeSize[data_types[0]["d_type"]]
 8752: 
 8753:     if math_inst.instruction_shape[0] == 128:
 8754:       epi_schedule = EpilogueScheduleType.TmaWarpSpecialized2Sm
 8755:     else:
 8756:       epi_schedule = EpilogueScheduleType.ScheduleAuto
 8757: 
 8758:     CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_types,
 8759:       [[KernelScheduleType.TmaWarpSpecialized2SmSm100, epi_schedule]], tile_schedulers=tile_schedulers)
 8760: 
 8761:     # for mixed precision kernels, also generate kernels that write output matrix in the A/B format
 8762:     # Avoid emitting two kernels if the accumulator type does not differ from the input type (e.g. F16 accumulation)
 8763:     if math_inst.element_a != math_inst.element_accumulator:
 8764:       data_types_mixed = [
 8765:         {
 8766:           "a_type"   : math_inst.element_a,
 8767:           "b_type"   : math_inst.element_b,
 8768:           "c_type"   : math_inst.element_a,
 8769:           "d_type"   : math_inst.element_a,
 8770:           "acc_type" : math_inst.element_accumulator,
 8771:           "epi_type" : epi_type,
 8772:         },
 8773:         {
 8774:           "a_type"   : math_inst.element_a,
 8775:           "b_type"   : math_inst.element_b,
 8776:           "c_type"   : DataType.void,
 8777:           "d_type"   : math_inst.element_a,
 8778:           "acc_type" : math_inst.element_accumulator,
 8779:           "epi_type" : epi_type,
 8780:         },
 8781:       ]
 8782:       # Set alignment d based on Destination format.
 8783:       for layout in layouts:
 8784:         layout[2][1] = 128 // DataTypeSize[data_types[0]["d_type"]]
 8785: 
 8786:       CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_types_mixed,
 8787:         [[KernelScheduleType.TmaWarpSpecialized2SmSm100, epi_schedule]], tile_schedulers=tile_schedulers)
```
**EN:** Defines `GenerateSM100_TensorOp_int8_UMMA_gemm()`, which generates sm100 tensor op int8 umma gemm. Key helper calls include ThorSMRenumbering, CudaToolkitVersionSatisfies, MathInstruction, CreateGemmUniversal3xOperator, append, TileDescription.

**CN:** 定义 `GenerateSM100_TensorOp_int8_UMMA_gemm()`，用于生成 sm100 tensor op int8 umma gemm。 其中会调用的重要辅助函数包括 ThorSMRenumbering, CudaToolkitVersionSatisfies, MathInstruction, CreateGemmUniversal3xOperator, append, TileDescription。

### L8790-L8916 — Function `GenerateSM100_SparseTensorOp_32b_UMMA_gemm`

```python
 8790: def GenerateSM100_SparseTensorOp_32b_UMMA_gemm(manifest, cuda_version):
 8791:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 8792:     return
 8793: 
 8794:   # layouts for ABC and their alignments.
 8795:   layouts = [
 8796:     # Alignment requirement will be over-write below
 8797:     [[LayoutType.RowMajor, -1], [LayoutType.ColumnMajor, -1], [LayoutType.RowMajor, -1]],
 8798:   ]
 8799: 
 8800:   thor_sm = ThorSMRenumbering(cuda_version)
 8801: 
 8802:   min_cc = 100
 8803:   max_cc = thor_sm
 8804: 
 8805:   tile_schedulers = [
 8806:     TileSchedulerType.Default, TileSchedulerType.StreamK
 8807:   ]
 8808: 
 8809:   kernel_data_types = [
 8810:     # void_c
 8811:     {
 8812:       "a_type"   : DataType.f32,
 8813:       "b_type"   : DataType.f32,
 8814:       "c_type"   : DataType.void,
 8815:       "d_type"   : DataType.f32,
 8816:       "acc_type" : DataType.f32,
 8817:       "epi_type" : DataType.f32,
 8818:     },
 8819:     # none void_c
 8820:     {
 8821:       "a_type"   : DataType.f32,
 8822:       "b_type"   : DataType.f32,
 8823:       "c_type"   : DataType.f32,
 8824:       "d_type"   : DataType.f32,
 8825:       "acc_type" : DataType.f32,
 8826:       "epi_type" : DataType.f32,
 8827:     },
 8828:   ]
 8829: 
 8830:   math_instructions_1sm = [
 8831:     MathInstruction(
 8832:       [128, 128, 16],
 8833:       DataType.tf32, DataType.tf32, DataType.f32,
 8834:       OpcodeClass.SparseTensorOp,
 8835:       MathOperation.multiply_add),
 8836:     MathInstruction(
 8837:       [128, 256, 16],
 8838:       DataType.tf32, DataType.tf32, DataType.f32,
 8839:       OpcodeClass.SparseTensorOp,
 8840:       MathOperation.multiply_add),
 8841:   ]
 8842: 
 8843:   math_instructions_2sm = [
 8844:     MathInstruction(
 8845:       [256, 128, 16],
 8846:       DataType.tf32, DataType.tf32, DataType.f32,
 8847:       OpcodeClass.SparseTensorOp,
 8848:       MathOperation.multiply_add),
 8849:     MathInstruction(
 8850:       [256, 256, 16],
 8851:       DataType.tf32, DataType.tf32, DataType.f32,
 8852:       OpcodeClass.SparseTensorOp,
 8853:       MathOperation.multiply_add),
 8854:   ]
 8855: 
 8856:   # 1xSM MMA kernels
 8857:   for math_inst in math_instructions_1sm:
 8858:     tile_descriptions = []
 8859:     for cluster_shape in sm100_cluster_shape_1sm:
 8860:       if thor_sm in manifest.compute_capabilities_baseline :
 8861:         if cluster_shape == [4,4,1] :
 8862:           continue
 8863:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 8864:       tile_descriptions.append(
 8865:         TileDescription([
 8866:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 8867:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 8868:           math_inst.instruction_shape[2] * 2 * multiplier_1sm[2]],
 8869:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 8870: 
 8871:     for kernel_data_type in kernel_data_types:
 8872:       # Update layout alignment
 8873:       # alignment for d might be different for each kernel_data_type
 8874:       layouts_copy = copy.deepcopy(layouts)
 8875:       for layout in layouts_copy:
 8876:         # alignment for a, 2 for sparsity
 8877:         layout[0][1] = get_tma_alignment_elt(kernel_data_type["a_type"]) * ( 2 if layout[0][0] == LayoutType.RowMajor else 1)
 8878:         # alignment for b
 8879:         layout[1][1] = get_tma_alignment_elt(kernel_data_type["b_type"])
 8880:         # alignment for d
 8881:         layout[2][1] = get_tma_alignment_elt(kernel_data_type["d_type"])
 8882: 
 8883:       CreateSparseGemmUniversal3xOperator(manifest, layouts_copy, tile_descriptions, [kernel_data_type],
 8884:         [[KernelScheduleType.SparseTmaWarpSpecialized1SmSm100, EpilogueScheduleType.TmaWarpSpecialized1Sm]],
 8885:         tile_schedulers=tile_schedulers)
 8886: 
 8887:   # 2xSM MMA kernels
 8888:   for math_inst in math_instructions_2sm:
 8889:     tile_descriptions = []
 8890:     for cluster_shape in sm100_cluster_shape_2sm:
 8891:       if thor_sm in manifest.compute_capabilities_baseline :
 8892:         if cluster_shape == [4,4,1] :
 8893:           continue
 8894:       multiplier_2sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 8895:       tile_descriptions.append(
 8896:         TileDescription([
 8897:           math_inst.instruction_shape[0]     * multiplier_2sm[0],
 8898:           math_inst.instruction_shape[1]     * multiplier_2sm[1],
 8899:           math_inst.instruction_shape[2] * 2 * multiplier_2sm[2]],
 8900:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 8901: 
 8902:     for kernel_data_type in kernel_data_types:
 8903:       # Update layout alignment
 8904:       # alignment for d might be different for each kernel_data_type
 8905:       layouts_copy = copy.deepcopy(layouts)
 8906:       for layout in layouts_copy:
 8907:         # alignment for a, 2 for sparsity
 8908:         layout[0][1] = get_tma_alignment_elt(kernel_data_type["a_type"]) * ( 2 if layout[0][0] == LayoutType.RowMajor else 1)
 8909:         # alignment for b
 8910:         layout[1][1] = get_tma_alignment_elt(kernel_data_type["b_type"])
 8911:         # alignment for d
 8912:         layout[2][1] = get_tma_alignment_elt(kernel_data_type["d_type"])
 8913: 
 8914:       CreateSparseGemmUniversal3xOperator(manifest, layouts_copy, tile_descriptions, [kernel_data_type],
 8915:         [[KernelScheduleType.SparseTmaWarpSpecialized2SmSm100, EpilogueScheduleType.TmaWarpSpecialized2Sm]],
 8916:         tile_schedulers=tile_schedulers)
```
**EN:** Defines `GenerateSM100_SparseTensorOp_32b_UMMA_gemm()`, which generates sm100 sparse tensor op 32b umma gemm. Key helper calls include ThorSMRenumbering, CudaToolkitVersionSatisfies, MathInstruction, append, deepcopy, CreateSparseGemmUniversal3xOperator.

**CN:** 定义 `GenerateSM100_SparseTensorOp_32b_UMMA_gemm()`，用于生成 sm100 sparse tensor op 32b umma gemm。 其中会调用的重要辅助函数包括 ThorSMRenumbering, CudaToolkitVersionSatisfies, MathInstruction, append, deepcopy, CreateSparseGemmUniversal3xOperator。

### L8918-L9044 — Function `GenerateSM100_SparseTensorOp_16b_UMMA_gemm`

```python
 8918: def GenerateSM100_SparseTensorOp_16b_UMMA_gemm(manifest, cuda_version):
 8919:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 8920:     return
 8921: 
 8922:   # layouts for ABC and their alignments.
 8923:   layouts = [
 8924:     # Alignment requirement will be over-write below
 8925:     [[LayoutType.RowMajor, -1], [LayoutType.ColumnMajor, -1], [LayoutType.RowMajor, -1]],
 8926:   ]
 8927: 
 8928:   thor_sm = ThorSMRenumbering(cuda_version)
 8929: 
 8930:   min_cc = 100
 8931:   max_cc = thor_sm
 8932: 
 8933:   tile_schedulers = [
 8934:     TileSchedulerType.Default, TileSchedulerType.StreamK
 8935:   ]
 8936: 
 8937:   kernel_data_types = [
 8938:     # void_c
 8939:     {
 8940:       "a_type"   : DataType.f16,
 8941:       "b_type"   : DataType.f16,
 8942:       "c_type"   : DataType.void,
 8943:       "d_type"   : DataType.f16,
 8944:       "acc_type" : DataType.f32,
 8945:       "epi_type" : DataType.f32,
 8946:     },
 8947:     # none void_c
 8948:     {
 8949:       "a_type"   : DataType.f16,
 8950:       "b_type"   : DataType.f16,
 8951:       "c_type"   : DataType.f16,
 8952:       "d_type"   : DataType.f16,
 8953:       "acc_type" : DataType.f32,
 8954:       "epi_type" : DataType.f32,
 8955:     },
 8956:   ]
 8957: 
 8958:   math_instructions_1sm = [
 8959:     MathInstruction(
 8960:       [128, 128, 32],
 8961:       DataType.f16, DataType.f16, DataType.f32,
 8962:       OpcodeClass.SparseTensorOp,
 8963:       MathOperation.multiply_add),
 8964:     MathInstruction(
 8965:       [128, 256, 32],
 8966:       DataType.f16, DataType.f16, DataType.f32,
 8967:       OpcodeClass.SparseTensorOp,
 8968:       MathOperation.multiply_add),
 8969:   ]
 8970: 
 8971:   math_instructions_2sm = [
 8972:     MathInstruction(
 8973:       [256, 128, 32],
 8974:       DataType.f16, DataType.f16, DataType.f32,
 8975:       OpcodeClass.SparseTensorOp,
 8976:       MathOperation.multiply_add),
 8977:     MathInstruction(
 8978:       [256, 256, 32],
 8979:       DataType.f16, DataType.f16, DataType.f32,
 8980:       OpcodeClass.SparseTensorOp,
 8981:       MathOperation.multiply_add),
 8982:   ]
 8983: 
 8984:   # 1xSM MMA kernels
 8985:   for math_inst in math_instructions_1sm:
 8986:     tile_descriptions = []
 8987:     for cluster_shape in sm100_cluster_shape_1sm:
 8988:       if thor_sm in manifest.compute_capabilities_baseline :
 8989:         if cluster_shape == [4,4,1] :
 8990:           continue
 8991:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 8992:       tile_descriptions.append(
 8993:         TileDescription([
 8994:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 8995:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 8996:           math_inst.instruction_shape[2] * 2 * multiplier_1sm[2]],
 8997:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 8998: 
 8999:     for kernel_data_type in kernel_data_types:
 9000:       # Update layout alignment
 9001:       # alignment for d might be different for each kernel_data_type
 9002:       layouts_copy = copy.deepcopy(layouts)
 9003:       for layout in layouts_copy:
 9004:         # alignment for a, 2 for sparsity
 9005:         layout[0][1] = get_tma_alignment_elt(kernel_data_type["a_type"]) * ( 2 if layout[0][0] == LayoutType.RowMajor else 1)
 9006:         # alignment for b
 9007:         layout[1][1] = get_tma_alignment_elt(kernel_data_type["b_type"])
 9008:         # alignment for d
 9009:         layout[2][1] = get_tma_alignment_elt(kernel_data_type["d_type"])
 9010: 
 9011:       CreateSparseGemmUniversal3xOperator(manifest, layouts_copy, tile_descriptions, [kernel_data_type],
 9012:         [[KernelScheduleType.SparseTmaWarpSpecialized1SmSm100, EpilogueScheduleType.TmaWarpSpecialized1Sm]],
 9013:         tile_schedulers=tile_schedulers)
 9014: 
 9015:   # 2xSM MMA kernels
 9016:   for math_inst in math_instructions_2sm:
 9017:     tile_descriptions = []
 9018:     for cluster_shape in sm100_cluster_shape_2sm:
 9019:       if thor_sm in manifest.compute_capabilities_baseline :
 9020:         if cluster_shape == [4,4,1] :
 9021:           continue
 9022:       multiplier_2sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 9023:       tile_descriptions.append(
 9024:         TileDescription([
 9025:           math_inst.instruction_shape[0]     * multiplier_2sm[0],
 9026:           math_inst.instruction_shape[1]     * multiplier_2sm[1],
 9027:           math_inst.instruction_shape[2] * 2 * multiplier_2sm[2]],
 9028:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 9029: 
 9030:     for kernel_data_type in kernel_data_types:
 9031:       # Update layout alignment
 9032:       # alignment for d might be different for each kernel_data_type
 9033:       layouts_copy = copy.deepcopy(layouts)
 9034:       for layout in layouts_copy:
 9035:         # alignment for a, 2 for sparsity
 9036:         layout[0][1] = get_tma_alignment_elt(kernel_data_type["a_type"]) * ( 2 if layout[0][0] == LayoutType.RowMajor else 1)
 9037:         # alignment for b
 9038:         layout[1][1] = get_tma_alignment_elt(kernel_data_type["b_type"])
 9039:         # alignment for d
 9040:         layout[2][1] = get_tma_alignment_elt(kernel_data_type["d_type"])
 9041: 
 9042:       CreateSparseGemmUniversal3xOperator(manifest, layouts_copy, tile_descriptions, [kernel_data_type],
 9043:         [[KernelScheduleType.SparseTmaWarpSpecialized2SmSm100, EpilogueScheduleType.TmaWarpSpecialized2Sm]],
 9044:         tile_schedulers=tile_schedulers)
```
**EN:** Defines `GenerateSM100_SparseTensorOp_16b_UMMA_gemm()`, which generates sm100 sparse tensor op 16b umma gemm. Key helper calls include ThorSMRenumbering, CudaToolkitVersionSatisfies, MathInstruction, append, deepcopy, CreateSparseGemmUniversal3xOperator.

**CN:** 定义 `GenerateSM100_SparseTensorOp_16b_UMMA_gemm()`，用于生成 sm100 sparse tensor op 16b umma gemm。 其中会调用的重要辅助函数包括 ThorSMRenumbering, CudaToolkitVersionSatisfies, MathInstruction, append, deepcopy, CreateSparseGemmUniversal3xOperator。

### L9046-L9171 — Function `GenerateSM100_SparseTensorOp_int8_UMMA_gemm`

```python
 9046: def GenerateSM100_SparseTensorOp_int8_UMMA_gemm(manifest, cuda_version):
 9047:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 9048:     return
 9049: 
 9050:   # layouts for ABC and their alignments.
 9051:   layouts = [
 9052:     # Alignment requirement will be over-write below
 9053:     [[LayoutType.RowMajor, -1], [LayoutType.ColumnMajor, -1], [LayoutType.RowMajor, -1]],
 9054:   ]
 9055: 
 9056:   thor_sm = ThorSMRenumbering(cuda_version)
 9057: 
 9058:   min_cc = 100
 9059:   max_cc = thor_sm
 9060: 
 9061:   tile_schedulers = [
 9062:     TileSchedulerType.Default, TileSchedulerType.StreamK
 9063:   ]
 9064: 
 9065:   kernel_data_types = [
 9066:     # void_c
 9067:     {
 9068:       "a_type"   : DataType.s8,
 9069:       "b_type"   : DataType.s8,
 9070:       "c_type"   : DataType.void,
 9071:       "d_type"   : DataType.s8,
 9072:       "acc_type" : DataType.f32,
 9073:       "epi_type" : DataType.f32,
 9074:     },
 9075:     # none void_c
 9076:     {
 9077:       "a_type"   : DataType.s8,
 9078:       "b_type"   : DataType.s8,
 9079:       "c_type"   : DataType.s8,
 9080:       "d_type"   : DataType.s8,
 9081:       "acc_type" : DataType.f32,
 9082:       "epi_type" : DataType.f32,
 9083:     },
 9084:   ]
 9085: 
 9086:   math_instructions_1sm = [
 9087:     MathInstruction(
 9088:       [128, 128, 64],
 9089:       DataType.s8, DataType.s8, DataType.s32,
 9090:       OpcodeClass.SparseTensorOp,
 9091:       MathOperation.multiply_add),
 9092:     MathInstruction(
 9093:       [128, 256, 64],
 9094:       DataType.s8, DataType.s8, DataType.s32,
 9095:       OpcodeClass.SparseTensorOp,
 9096:       MathOperation.multiply_add)]
 9097: 
 9098:   math_instructions_2sm = [
 9099:     MathInstruction(
 9100:       [256, 128, 64],
 9101:       DataType.s8, DataType.s8, DataType.s32,
 9102:       OpcodeClass.SparseTensorOp,
 9103:       MathOperation.multiply_add),
 9104:     MathInstruction(
 9105:       [256, 256, 64],
 9106:       DataType.s8, DataType.s8, DataType.s32,
 9107:       OpcodeClass.SparseTensorOp,
 9108:       MathOperation.multiply_add),
 9109:   ]
 9110: 
 9111:   # 1xSM MMA kernels
 9112:   for math_inst in math_instructions_1sm:
 9113:     tile_descriptions = []
 9114:     for cluster_shape in sm100_cluster_shape_1sm:
 9115:       if thor_sm in manifest.compute_capabilities_baseline :
 9116:         if cluster_shape == [4,4,1] :
 9117:           continue
 9118:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 9119:       tile_descriptions.append(
 9120:         TileDescription([
 9121:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 9122:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 9123:           math_inst.instruction_shape[2] * 2 * multiplier_1sm[2]],
 9124:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 9125: 
 9126:     for kernel_data_type in kernel_data_types:
 9127:       # Update layout alignment
 9128:       # alignment for d might be different for each kernel_data_type
 9129:       layouts_copy = copy.deepcopy(layouts)
 9130:       for layout in layouts_copy:
 9131:         # alignment for a, 2 for sparsity
 9132:         layout[0][1] = get_tma_alignment_elt(kernel_data_type["a_type"]) * ( 2 if layout[0][0] == LayoutType.RowMajor else 1)
 9133:         # alignment for b
 9134:         layout[1][1] = get_tma_alignment_elt(kernel_data_type["b_type"])
 9135:         # alignment for d
 9136:         layout[2][1] = get_tma_alignment_elt(kernel_data_type["d_type"])
 9137: 
 9138:       CreateSparseGemmUniversal3xOperator(manifest, layouts_copy, tile_descriptions, [kernel_data_type],
 9139:         [[KernelScheduleType.SparseTmaWarpSpecialized1SmSm100, EpilogueScheduleType.TmaWarpSpecialized1Sm]],
 9140:         tile_schedulers=tile_schedulers)
 9141: 
 9142:   # 2xSM MMA kernels
 9143:   for math_inst in math_instructions_2sm:
 9144:     tile_descriptions = []
 9145:     for cluster_shape in sm100_cluster_shape_2sm:
 9146:       if thor_sm in manifest.compute_capabilities_baseline :
 9147:         if cluster_shape == [4,4,1] :
 9148:           continue
 9149:       multiplier_2sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 9150:       tile_descriptions.append(
 9151:         TileDescription([
 9152:           math_inst.instruction_shape[0]     * multiplier_2sm[0],
 9153:           math_inst.instruction_shape[1]     * multiplier_2sm[1],
 9154:           math_inst.instruction_shape[2] * 2 * multiplier_2sm[2]],
 9155:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 9156: 
 9157:     for kernel_data_type in kernel_data_types:
 9158:       # Update layout alignment
 9159:       # alignment for d might be different for each kernel_data_type
 9160:       layouts_copy = copy.deepcopy(layouts)
 9161:       for layout in layouts_copy:
 9162:         # alignment for a, 2 for sparsity
 9163:         layout[0][1] = get_tma_alignment_elt(kernel_data_type["a_type"]) * ( 2 if layout[0][0] == LayoutType.RowMajor else 1)
 9164:         # alignment for b
 9165:         layout[1][1] = get_tma_alignment_elt(kernel_data_type["b_type"])
 9166:         # alignment for d
 9167:         layout[2][1] = get_tma_alignment_elt(kernel_data_type["d_type"])
 9168: 
 9169:       CreateSparseGemmUniversal3xOperator(manifest, layouts_copy, tile_descriptions, [kernel_data_type],
 9170:         [[KernelScheduleType.SparseTmaWarpSpecialized2SmSm100, EpilogueScheduleType.TmaWarpSpecialized2Sm]],
 9171:         tile_schedulers=tile_schedulers)
```
**EN:** Defines `GenerateSM100_SparseTensorOp_int8_UMMA_gemm()`, which generates sm100 sparse tensor op int8 umma gemm. Key helper calls include ThorSMRenumbering, CudaToolkitVersionSatisfies, MathInstruction, append, deepcopy, CreateSparseGemmUniversal3xOperator.

**CN:** 定义 `GenerateSM100_SparseTensorOp_int8_UMMA_gemm()`，用于生成 sm100 sparse tensor op int8 umma gemm。 其中会调用的重要辅助函数包括 ThorSMRenumbering, CudaToolkitVersionSatisfies, MathInstruction, append, deepcopy, CreateSparseGemmUniversal3xOperator。

### L9173-L9312 — Function `GenerateSM100_SparseTensorOp_fp8_UMMA_gemm`

```python
 9173: def GenerateSM100_SparseTensorOp_fp8_UMMA_gemm(manifest, cuda_version):
 9174:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 9175:     return
 9176: 
 9177:   # layouts for ABC and their alignments.
 9178:   layouts = [
 9179:     # Alignment requirement will be over-write below
 9180:     [[LayoutType.RowMajor, -1], [LayoutType.ColumnMajor, -1], [LayoutType.RowMajor, -1]],
 9181:   ]
 9182: 
 9183:   thor_sm = ThorSMRenumbering(cuda_version)
 9184: 
 9185:   min_cc = 100
 9186:   max_cc = thor_sm
 9187: 
 9188:   tile_schedulers = [
 9189:     TileSchedulerType.Default, TileSchedulerType.StreamK
 9190:   ]
 9191: 
 9192:   kernel_data_types = [
 9193:     # NOTE: a/b type in kernel will be overwrite below.
 9194:     #* void_c
 9195:     # f8_f8_f32_void_f16
 9196:     {
 9197:       "a_type"   : DataType.e4m3,
 9198:       "b_type"   : DataType.e4m3,
 9199:       "c_type"   : DataType.void,
 9200:       "d_type"   : DataType.f16,
 9201:       "acc_type" : DataType.f32,
 9202:       "epi_type" : DataType.f32,
 9203:     },
 9204:     #* non-void_c
 9205:     # f8_f8_f32_f16_f8
 9206:     {
 9207:       "a_type"   : DataType.e4m3,
 9208:       "b_type"   : DataType.e4m3,
 9209:       "c_type"   : DataType.f16,
 9210:       "d_type"   : DataType.e4m3,
 9211:       "acc_type" : DataType.f32,
 9212:       "epi_type" : DataType.f32,
 9213:     },
 9214:   ]
 9215: 
 9216:   math_instructions_1sm = [
 9217:     # Runtime DType
 9218:     MathInstruction(
 9219:       [128, 128, 64],
 9220:       DataType.f8, DataType.f8, DataType.f32,
 9221:       OpcodeClass.SparseTensorOp,
 9222:       MathOperation.multiply_add),
 9223:     MathInstruction(
 9224:       [128, 256, 64],
 9225:       DataType.f8, DataType.f8, DataType.f32,
 9226:       OpcodeClass.SparseTensorOp,
 9227:       MathOperation.multiply_add),
 9228:   ]
 9229: 
 9230:   math_instructions_2sm = [
 9231:     # Runtime DType
 9232:     MathInstruction(
 9233:       [256, 128, 64],
 9234:       DataType.f8, DataType.f8, DataType.f32,
 9235:       OpcodeClass.SparseTensorOp,
 9236:       MathOperation.multiply_add),
 9237:     MathInstruction(
 9238:       [256, 256, 64],
 9239:       DataType.f8, DataType.f8, DataType.f32,
 9240:       OpcodeClass.SparseTensorOp,
 9241:       MathOperation.multiply_add),
 9242:   ]
 9243: 
 9244:   # 1xSM MMA kernels
 9245:   for math_inst in math_instructions_1sm:
 9246:     tile_descriptions = []
 9247:     for cluster_shape in sm100_cluster_shape_1sm:
 9248:       if thor_sm in manifest.compute_capabilities_baseline :
 9249:         if cluster_shape == [4,4,1] :
 9250:           continue
 9251:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 9252:       tile_descriptions.append(
 9253:         TileDescription([
 9254:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 9255:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 9256:           math_inst.instruction_shape[2] * 2 * multiplier_1sm[2]],
 9257:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 9258: 
 9259:     for kernel_data_type in kernel_data_types:
 9260:       # Update input AB type
 9261:       kernel_data_type["a_type"] = math_inst.element_a
 9262:       kernel_data_type["b_type"] = math_inst.element_b
 9263: 
 9264:       # Update layout alignment
 9265:       # alignment for d might be different for each kernel_data_type
 9266:       layouts_copy = copy.deepcopy(layouts)
 9267:       for layout in layouts_copy:
 9268:         # alignment for a, 2 for sparsity
 9269:         layout[0][1] = get_tma_alignment_elt(kernel_data_type["a_type"]) * ( 2 if layout[0][0] == LayoutType.RowMajor else 1)
 9270:         # alignment for b
 9271:         layout[1][1] = get_tma_alignment_elt(kernel_data_type["b_type"])
 9272:         # alignment for d
 9273:         layout[2][1] = get_tma_alignment_elt(kernel_data_type["d_type"])
 9274: 
 9275:       CreateSparseGemmUniversal3xOperator(manifest, layouts_copy, tile_descriptions, [kernel_data_type],
 9276:         [[KernelScheduleType.SparseTmaWarpSpecialized1SmSm100, EpilogueScheduleType.TmaWarpSpecialized1Sm]],
 9277:         tile_schedulers=tile_schedulers)
 9278: 
 9279:   # 2xSM MMA kernels
 9280:   for math_inst in math_instructions_2sm:
 9281:     tile_descriptions = []
 9282:     for cluster_shape in sm100_cluster_shape_2sm:
 9283:       if thor_sm in manifest.compute_capabilities_baseline :
 9284:         if cluster_shape == [4,4,1] :
 9285:           continue
 9286:       multiplier_2sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 9287:       tile_descriptions.append(
 9288:         TileDescription([
 9289:           math_inst.instruction_shape[0]     * multiplier_2sm[0],
 9290:           math_inst.instruction_shape[1]     * multiplier_2sm[1],
 9291:           math_inst.instruction_shape[2] * 2 * multiplier_2sm[2]],
 9292:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 9293: 
 9294:     for kernel_data_type in kernel_data_types:
 9295:       # Update input AB type
 9296:       kernel_data_type["a_type"] = math_inst.element_a
 9297:       kernel_data_type["b_type"] = math_inst.element_b
 9298: 
 9299:       # Update layout alignment
 9300:       # alignment for d might be different for each kernel_data_type
 9301:       layouts_copy = copy.deepcopy(layouts)
 9302:       for layout in layouts_copy:
 9303:         # alignment for a, 2 for sparsity
 9304:         layout[0][1] = get_tma_alignment_elt(kernel_data_type["a_type"]) * ( 2 if layout[0][0] == LayoutType.RowMajor else 1)
 9305:         # alignment for b
 9306:         layout[1][1] = get_tma_alignment_elt(kernel_data_type["b_type"])
 9307:         # alignment for d
 9308:         layout[2][1] = get_tma_alignment_elt(kernel_data_type["d_type"])
 9309: 
 9310:       CreateSparseGemmUniversal3xOperator(manifest, layouts_copy, tile_descriptions, [kernel_data_type],
 9311:         [[KernelScheduleType.SparseTmaWarpSpecialized2SmSm100, EpilogueScheduleType.TmaWarpSpecialized2Sm]],
 9312:         tile_schedulers=tile_schedulers)
```
**EN:** Defines `GenerateSM100_SparseTensorOp_fp8_UMMA_gemm()`, which generates sm100 sparse tensor op fp8 umma gemm. Key helper calls include ThorSMRenumbering, CudaToolkitVersionSatisfies, MathInstruction, append, deepcopy, CreateSparseGemmUniversal3xOperator.

**CN:** 定义 `GenerateSM100_SparseTensorOp_fp8_UMMA_gemm()`，用于生成 sm100 sparse tensor op fp8 umma gemm。 其中会调用的重要辅助函数包括 ThorSMRenumbering, CudaToolkitVersionSatisfies, MathInstruction, append, deepcopy, CreateSparseGemmUniversal3xOperator。

### L9314-L9505 — Function `GenerateSM100_SparseTensorOp_mixed_8bits_UMMA_gemm`

```python
 9314: def GenerateSM100_SparseTensorOp_mixed_8bits_UMMA_gemm(manifest, cuda_version):
 9315:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 9316:     return
 9317: 
 9318:   # layouts for ABC and their alignments.
 9319:   layouts = [
 9320:     # Alignment requirement will be over-write below
 9321:     [[LayoutType.RowMajor, -1], [LayoutType.ColumnMajor, -1], [LayoutType.RowMajor, -1]],
 9322:   ]
 9323: 
 9324:   thor_sm = ThorSMRenumbering(cuda_version)
 9325: 
 9326:   min_cc = 100
 9327:   max_cc = thor_sm
 9328: 
 9329:   tile_schedulers = [
 9330:     TileSchedulerType.Default, TileSchedulerType.StreamK
 9331:   ]
 9332: 
 9333:   math_instructions_1sm = [
 9334:     # Runtime Dtype
 9335:     MathInstruction(
 9336:       [128, 128, 64],
 9337:       DataType.f4, DataType.f4, DataType.f32,
 9338:       OpcodeClass.SparseTensorOp,
 9339:       MathOperation.multiply_add),
 9340:     MathInstruction(
 9341:       [128, 256, 64],
 9342:       DataType.f4, DataType.f4, DataType.f32,
 9343:       OpcodeClass.SparseTensorOp,
 9344:       MathOperation.multiply_add),
 9345:   
 9346:     MathInstruction(
 9347:       [128, 128, 64],
 9348:       DataType.f6, DataType.f6, DataType.f32,
 9349:       OpcodeClass.SparseTensorOp,
 9350:       MathOperation.multiply_add),
 9351:     MathInstruction(
 9352:       [128, 256, 64],
 9353:       DataType.f6, DataType.f6, DataType.f32,
 9354:       OpcodeClass.SparseTensorOp,
 9355:       MathOperation.multiply_add),
 9356:   ]
 9357: 
 9358:   math_instructions_2sm = [
 9359:     # Runtime DType
 9360:     MathInstruction(
 9361:       [256, 128, 64],
 9362:       DataType.f4, DataType.f4, DataType.f32,
 9363:       OpcodeClass.SparseTensorOp,
 9364:       MathOperation.multiply_add),
 9365:     MathInstruction(
 9366:       [256, 256, 64],
 9367:       DataType.f4, DataType.f4, DataType.f32,
 9368:       OpcodeClass.SparseTensorOp,
 9369:       MathOperation.multiply_add),
 9370:   
 9371:     MathInstruction(
 9372:       [256, 128, 64],
 9373:       DataType.f6, DataType.f6, DataType.f32,
 9374:       OpcodeClass.SparseTensorOp,
 9375:       MathOperation.multiply_add),
 9376:     MathInstruction(
 9377:       [256, 256, 64],
 9378:       DataType.f6, DataType.f6, DataType.f32,
 9379:       OpcodeClass.SparseTensorOp,
 9380:       MathOperation.multiply_add),
 9381:   ]
 9382: 
 9383:   # 1xSM MMA kernels
 9384:   for math_inst in math_instructions_1sm:
 9385:     tile_descriptions = []
 9386:     for cluster_shape in sm100_cluster_shape_1sm:
 9387:       if thor_sm in manifest.compute_capabilities_baseline :
 9388:         if cluster_shape == [4,4,1] :
 9389:           continue
 9390:       multiplier_1sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else cluster_shape
 9391:       tile_descriptions.append(
 9392:         TileDescription([
 9393:           math_inst.instruction_shape[0]     * multiplier_1sm[0],
 9394:           math_inst.instruction_shape[1]     * multiplier_1sm[1],
 9395:           math_inst.instruction_shape[2] * 2 * multiplier_1sm[2]],
 9396:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 9397: 
 9398:     kernel_data_types = [
 9399:       # void_c
 9400:       {
 9401:         "a_type"   : math_inst.element_a,
 9402:         "b_type"   : math_inst.element_b,
 9403:         "c_type"   : DataType.f16,
 9404:         "d_type"   : DataType.f16,
 9405:         "acc_type" : math_inst.element_accumulator,
 9406:         "epi_type" : DataType.f32,
 9407:       },
 9408:       # none void_c
 9409:       {
 9410:         "a_type"   : math_inst.element_a,
 9411:         "b_type"   : math_inst.element_b,
 9412:         "c_type"   : DataType.void,
 9413:         "d_type"   : DataType.f16,
 9414:         "acc_type" : math_inst.element_accumulator,
 9415:         "epi_type" : DataType.f32,
 9416:       },
 9417:     ]
 9418: 
 9419:     for kernel_data_type in kernel_data_types:
 9420:       # Update layout alignment
 9421:       # alignment for d might be different for each kernel_data_type
 9422:       layouts_filtered = []
 9423:       for layout in layouts:
 9424:         layout_filter = copy.deepcopy(layout)
 9425:         # * A_K : Logical TileShape_K % 256 == 0
 9426:         # * A_M : TileShape_M % 128 == 0
 9427:         # * B_N : TileSize_N % 128 == 0
 9428:         # * B_K : TileSize_K % 128 == 0
 9429:         if ((layout_filter[0][0] == LayoutType.RowMajor and (math_inst.instruction_shape[2] * 2) % 256 == 0) or \
 9430:             (layout_filter[0][0] == LayoutType.ColumnMajor and math_inst.instruction_shape[0] % 128 == 0)) and \
 9431:            ((layout_filter[1][0] == LayoutType.RowMajor and math_inst.instruction_shape[1] % 128 == 0) or \
 9432:             (layout_filter[1][0] == LayoutType.ColumnMajor and (math_inst.instruction_shape[0] * 2) % 128 == 0)):
 9433:           # alignment for a, 2 for sparsity
 9434:           layout_filter[0][1] = get_tma_alignment_elt(kernel_data_type["a_type"]) * ( 2 if layout[0][0] == LayoutType.RowMajor else 1)
 9435:           # alignment for b
 9436:           layout_filter[1][1] = get_tma_alignment_elt(kernel_data_type["b_type"])
 9437:           # alignment for d
 9438:           layout_filter[2][1] = get_tma_alignment_elt(kernel_data_type["d_type"])
 9439:           layouts_filtered.append(layout_filter)
 9440: 
 9441:       CreateSparseGemmUniversal3xOperator(manifest, layouts_filtered, tile_descriptions, [kernel_data_type],
 9442:         [[KernelScheduleType.SparseTmaWarpSpecialized1SmSm100, EpilogueScheduleType.TmaWarpSpecialized1Sm]],
 9443:         tile_schedulers=tile_schedulers)
 9444: 
 9445:   # 2xSM MMA kernels
 9446:   for math_inst in math_instructions_2sm:
 9447:     tile_descriptions = []
 9448:     for cluster_shape in sm100_cluster_shape_2sm:
 9449:       if thor_sm in manifest.compute_capabilities_baseline :
 9450:         if cluster_shape == [4,4,1] :
 9451:           continue
 9452:       multiplier_2sm = (1, 1, 1) if cluster_shape == DynamicClusterShape else (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 9453:       tile_descriptions.append(
 9454:         TileDescription([
 9455:           math_inst.instruction_shape[0]     * multiplier_2sm[0],
 9456:           math_inst.instruction_shape[1]     * multiplier_2sm[1],
 9457:           math_inst.instruction_shape[2] * 2 * multiplier_2sm[2]],
 9458:           0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 9459: 
 9460:     kernel_data_types = [
 9461:       # void_c
 9462:       {
 9463:         "a_type"   : math_inst.element_a,
 9464:         "b_type"   : math_inst.element_b,
 9465:         "c_type"   : DataType.f16,
 9466:         "d_type"   : DataType.f16,
 9467:         "acc_type" : math_inst.element_accumulator,
 9468:         "epi_type" : DataType.f32,
 9469:       },
 9470:       # none void_c
 9471:       {
 9472:         "a_type"   : math_inst.element_a,
 9473:         "b_type"   : math_inst.element_b,
 9474:         "c_type"   : DataType.void,
 9475:         "d_type"   : DataType.f16,
 9476:         "acc_type" : math_inst.element_accumulator,
 9477:         "epi_type" : DataType.f32,
 9478:       },
 9479:     ]
 9480: 
 9481:     for kernel_data_type in kernel_data_types:
 9482:       # Update layout alignment
 9483:       # alignment for d might be different for each kernel_data_type
 9484:       layouts_filtered = []
 9485:       for layout in layouts:
 9486:         layout_filter = copy.deepcopy(layout)
 9487:         # * A_K : Logical TileShape_K % 256 == 0
 9488:         # * A_M : TileShape_M % 128 == 0
 9489:         # * B_N : TileSize_N % 256 == 0
 9490:         # * B_K : TileSize_K % 128 == 0
 9491:         if ((layout_filter[0][0] == LayoutType.RowMajor and (math_inst.instruction_shape[2] * 2) % 256 == 0) or \
 9492:             (layout_filter[0][0] == LayoutType.ColumnMajor and math_inst.instruction_shape[0] % 128 == 0)) and \
 9493:            ((layout_filter[1][0] == LayoutType.RowMajor and math_inst.instruction_shape[1] % 256 == 0) or \
 9494:             (layout_filter[1][0] == LayoutType.ColumnMajor and (math_inst.instruction_shape[0] * 2) % 128 == 0)):
 9495:           # alignment for a, 2 for sparsity
 9496:           layout_filter[0][1] = get_tma_alignment_elt(kernel_data_type["a_type"]) * ( 2 if layout[0][0] == LayoutType.RowMajor else 1)
 9497:           # alignment for b
 9498:           layout_filter[1][1] = get_tma_alignment_elt(kernel_data_type["b_type"])
 9499:           # alignment for d
 9500:           layout_filter[2][1] = get_tma_alignment_elt(kernel_data_type["d_type"])
 9501:           layouts_filtered.append(layout_filter)
 9502: 
 9503:       CreateSparseGemmUniversal3xOperator(manifest, layouts_filtered, tile_descriptions, [kernel_data_type],
 9504:         [[KernelScheduleType.SparseTmaWarpSpecialized2SmSm100, EpilogueScheduleType.TmaWarpSpecialized2Sm]],
 9505:         tile_schedulers=tile_schedulers)
```
**EN:** Defines `GenerateSM100_SparseTensorOp_mixed_8bits_UMMA_gemm()`, which generates sm100 sparse tensor op mixed 8bits umma gemm. Key helper calls include ThorSMRenumbering, CudaToolkitVersionSatisfies, MathInstruction, append, CreateSparseGemmUniversal3xOperator, TileDescription.

**CN:** 定义 `GenerateSM100_SparseTensorOp_mixed_8bits_UMMA_gemm()`，用于生成 sm100 sparse tensor op mixed 8bits umma gemm。 其中会调用的重要辅助函数包括 ThorSMRenumbering, CudaToolkitVersionSatisfies, MathInstruction, append, CreateSparseGemmUniversal3xOperator, TileDescription。

### L9506-L9507 — Comments

```python
 9506: 
 9507: # Conv Utility functions
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L9508-L9512 — Function `make_dims_and_alignments_triple`

```python
 9508: def make_dims_and_alignments_triple(dim: int, bit_per_element_A: int, bit_per_element_B: int, bit_per_element_C: int):
 9509:   bit_alignment_required_by_tma = 128
 9510:   return ((dim, bit_alignment_required_by_tma // bit_per_element_A), # A
 9511:           (dim, bit_alignment_required_by_tma // bit_per_element_B), # B
 9512:           (dim, bit_alignment_required_by_tma // bit_per_element_C)) # C
```
**EN:** Defines `make_dims_and_alignments_triple()`, which implements the make dims and alignments triple logic.

**CN:** 定义 `make_dims_and_alignments_triple()`，用于实现 make dims and alignments triple 相关逻辑。

### L9514-L9524 — Function `make_math_instruction_w_output`

```python
 9514: def make_math_instruction_w_output(data_types: Tuple[DataType, DataType, DataType, DataType],
 9515:                           instruction_shape: Tuple[int, int, int]) -> (MathInstruction, DataType):
 9516:   default_opcode = OpcodeClass.TensorOp
 9517:   default_math_op = MathOperation.multiply_add
 9518:   [A_data_type, B_data_type, Acc_data_type, Out_data_type] = data_types
 9519:   return (MathInstruction(
 9520:     instruction_shape,
 9521:     A_data_type, B_data_type, Acc_data_type,
 9522:     default_opcode,
 9523:     default_math_op
 9524:   ), Out_data_type)
```
**EN:** Defines `make_math_instruction_w_output()`, which implements the make math instruction w output logic. Key helper calls include MathInstruction.

**CN:** 定义 `make_math_instruction_w_output()`，用于实现 make math instruction w output 相关逻辑。 其中会调用的重要辅助函数包括 MathInstruction。

### L9526-L9530 — Documentation/setup

```python
 9526: """
 9527: Generate CUTLASS 3 convolution kernel(s) for SM100.
 9528: 
 9529: This is meant to be called from GenerateSM100.
 9530: """
```
**EN:** Carries standalone expressions or documentation that clarify how the surrounding code should behave.

**CN:** 包含独立表达式或文档说明，用于解释周边代码的行为。

### L9531-L9683 — Function `GenerateSM100_TensorOp_16b_UMMA_conv3x`

```python
 9531: def GenerateSM100_TensorOp_16b_UMMA_conv3x(manifest, cuda_version,
 9532:                                            log_indent_level: int = 0):
 9533:   log_debug_line('GenerateSM100_TensorOp_16b_UMMA_conv3x', log_indent_level)
 9534:   log_indent_level = log_indent_level + 1
 9535: 
 9536:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 9537:     return
 9538: 
 9539:   thor_sm = ThorSMRenumbering(cuda_version)
 9540: 
 9541:   minimum_compute_capability = 100
 9542:   maximum_compute_capability = thor_sm
 9543: 
 9544:   spatial_dims = [2, 3]
 9545: 
 9546:   conv_kinds = [
 9547:     ConvKind.Fprop,
 9548:     ConvKind.Dgrad,
 9549:     ConvKind.Wgrad
 9550:   ]
 9551: 
 9552:   stages = 0 # zero means "deduce the number of stages automatically"
 9553: 
 9554:   data_types_and_instruction_shapes_1sm = [
 9555:     # ((A,B,Acc,C/D), (InstM,InstN,InstK))
 9556:     ((DataType.f16, DataType.f16, DataType.f16, DataType.f16),    (64, 128, 16)),
 9557:     ((DataType.f16, DataType.f16, DataType.f16, DataType.f16),    (128, 128, 16)),
 9558:     ((DataType.f16, DataType.f16, DataType.f16, DataType.f16),    (128, 256, 16)),
 9559:     ((DataType.f16, DataType.f16, DataType.f32, DataType.f16),    (64, 128, 16)),
 9560:     ((DataType.f16, DataType.f16, DataType.f32, DataType.f16),    (128, 128, 16)),
 9561:     ((DataType.f16, DataType.f16, DataType.f32, DataType.f16),    (128, 256, 16)),
 9562:     ((DataType.bf16, DataType.bf16, DataType.f32, DataType.bf16), (64, 128, 16)),
 9563:     ((DataType.bf16, DataType.bf16, DataType.f32, DataType.bf16), (128, 128, 16)),
 9564:     ((DataType.bf16, DataType.bf16, DataType.f32, DataType.bf16), (128, 256, 16)),
 9565:   ]
 9566:   math_instructions_w_output_1sm = map(lambda x: make_math_instruction_w_output(*x),
 9567:                           data_types_and_instruction_shapes_1sm)
 9568: 
 9569:   cluster_shapes_1sm = [[1,1,1], [1,2,1], [1,4,1],[4,4,1]]
 9570: 
 9571:   if thor_sm in manifest.compute_capabilities_baseline :
 9572:     cluster_shapes_1sm = [[1,1,1], [1,2,1], [1,4,1]]
 9573: 
 9574:   # tile_descriptions is a 2-level list.
 9575:   # Each inner list is for each cluster shape.
 9576:   for math_inst, output_type in math_instructions_w_output_1sm:
 9577:     tile_descriptions = []
 9578:     for cluster_shape in cluster_shapes_1sm:
 9579:       cluster_multiplier = cluster_shape
 9580:       # Unlike SM90, SM100 tile shape calculation includes cluster shape.
 9581:       tile_shape = [
 9582:         math_inst.instruction_shape[0]     * cluster_multiplier[0],
 9583:         math_inst.instruction_shape[1]     * cluster_multiplier[1],
 9584:         math_inst.instruction_shape[2] * 4 * cluster_multiplier[2]
 9585:       ]
 9586:       warp_count = [4, 1, 1]
 9587:       tile_description = TileDescription(
 9588:         tile_shape, stages, warp_count, math_inst,
 9589:         minimum_compute_capability, maximum_compute_capability,
 9590:         cluster_shape)
 9591:       tile_descriptions.append(tile_description)
 9592: 
 9593:       # It's typical to get the data types from the math instruction.
 9594:       data_type = {
 9595:         "a_type"   : math_inst.element_a,
 9596:         "b_type"   : math_inst.element_b,
 9597:         "c_type"   : output_type,
 9598:         "d_type"   : output_type,
 9599:         "acc_type" : math_inst.element_accumulator,
 9600:         "epi_type" : math_inst.element_accumulator
 9601:       }
 9602: 
 9603:       dims_and_alignments = [make_dims_and_alignments_triple(dim, DataTypeSize[data_type["a_type"]], DataTypeSize[data_type["b_type"]], DataTypeSize[data_type["d_type"]]) for dim in spatial_dims]
 9604: 
 9605:       # Schedules
 9606:       mainloop_schedule = KernelScheduleType.ImplicitTmaWarpSpecialized1SmSm100
 9607:       epilogue_schedule = EpilogueScheduleType.ScheduleAuto
 9608:       schedule_pairs = [
 9609:         (mainloop_schedule, epilogue_schedule)
 9610:       ]
 9611: 
 9612:       for conv_kind in conv_kinds:
 9613:         CreateConvOperator3x(manifest,
 9614:                             dims_and_alignments = dims_and_alignments,
 9615:                             tile_descriptions = tile_descriptions,
 9616:                             data_types = data_type,
 9617:                             schedule_pairs = schedule_pairs,
 9618:                             conv_kind = conv_kind,
 9619:                             log_indent_level = log_indent_level)
 9620: 
 9621:   data_types_and_instruction_shapes_2sm = [
 9622:     # ((A,B,Acc,C/D), (InstM,InstN,InstK))
 9623:     ((DataType.f16, DataType.f16, DataType.f16, DataType.f16),    (128, 128, 16)),
 9624:     ((DataType.f16, DataType.f16, DataType.f16, DataType.f16),    (128, 256, 16)),
 9625:     ((DataType.f16, DataType.f16, DataType.f16, DataType.f16),    (256, 256, 16)),
 9626:     ((DataType.f16, DataType.f16, DataType.f32, DataType.f16),    (128, 128, 16)),
 9627:     ((DataType.f16, DataType.f16, DataType.f32, DataType.f16),    (128, 256, 16)),
 9628:     ((DataType.f16, DataType.f16, DataType.f32, DataType.f16),    (256, 256, 16)),
 9629:     ((DataType.bf16, DataType.bf16, DataType.f32, DataType.bf16), (128, 128, 16)),
 9630:     ((DataType.bf16, DataType.bf16, DataType.f32, DataType.bf16), (128, 256, 16)),
 9631:     ((DataType.bf16, DataType.bf16, DataType.f32, DataType.bf16), (256, 256, 16)),
 9632:   ]
 9633:   math_instructions_w_output_2sm = map(lambda x: make_math_instruction_w_output(*x),
 9634:                           data_types_and_instruction_shapes_2sm)
 9635: 
 9636:   cluster_shapes_2sm = [[2,1,1], [2,2,1], [2,4,1], [4,1,1], [4,2,1], [4,4,1]]
 9637:   if thor_sm in manifest.compute_capabilities_baseline :
 9638:     cluster_shapes_2sm = [[2,1,1], [2,2,1], [2,4,1], [4,1,1], [4,2,1]]
 9639: 
 9640:   for math_inst, output_type in math_instructions_w_output_2sm:
 9641:     tile_descriptions = []
 9642:     for cluster_shape in cluster_shapes_2sm:
 9643:       cluster_multiplier = (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 9644:       # Unlike SM90, SM100 tile shape calculation includes cluster shape.
 9645:       tile_shape = [
 9646:         math_inst.instruction_shape[0]     * cluster_multiplier[0],
 9647:         math_inst.instruction_shape[1]     * cluster_multiplier[1],
 9648:         math_inst.instruction_shape[2] * 4 * cluster_multiplier[2]
 9649:       ]
 9650:       warp_count = [4, 1, 1]
 9651:       tile_description = TileDescription(
 9652:         tile_shape, stages, warp_count, math_inst,
 9653:         minimum_compute_capability, maximum_compute_capability,
 9654:         cluster_shape)
 9655:       tile_descriptions.append(tile_description)
 9656: 
 9657:       # It's typical to get the data types from the math instruction.
 9658:       data_type = {
 9659:         "a_type"   : math_inst.element_a,
 9660:         "b_type"   : math_inst.element_b,
 9661:         "c_type"   : output_type,
 9662:         "d_type"   : output_type,
 9663:         "acc_type" : math_inst.element_accumulator,
 9664:         "epi_type" : math_inst.element_accumulator
 9665:       }
 9666: 
 9667:       dims_and_alignments = [make_dims_and_alignments_triple(dim, DataTypeSize[data_type["a_type"]], DataTypeSize[data_type["b_type"]], DataTypeSize[data_type["d_type"]]) for dim in spatial_dims]
 9668: 
 9669:       # Schedules
 9670:       mainloop_schedule = KernelScheduleType.ImplicitTmaWarpSpecialized2SmSm100
 9671:       epilogue_schedule = EpilogueScheduleType.ScheduleAuto
 9672:       schedule_pairs = [
 9673:         (mainloop_schedule, epilogue_schedule)
 9674:       ]
 9675: 
 9676:       for conv_kind in conv_kinds:
 9677:         CreateConvOperator3x(manifest,
 9678:                             dims_and_alignments = dims_and_alignments,
 9679:                             tile_descriptions = tile_descriptions,
 9680:                             data_types = data_type,
 9681:                             schedule_pairs = schedule_pairs,
 9682:                             conv_kind = conv_kind,
 9683:                             log_indent_level = log_indent_level)
```
**EN:** Defines `GenerateSM100_TensorOp_16b_UMMA_conv3x()`, which generates sm100 tensor op 16b umma conv3x. Key helper calls include log_debug_line, ThorSMRenumbering, map, CudaToolkitVersionSatisfies, make_math_instruction_w_output, TileDescription.

**CN:** 定义 `GenerateSM100_TensorOp_16b_UMMA_conv3x()`，用于生成 sm100 tensor op 16b umma conv3x。 其中会调用的重要辅助函数包括 log_debug_line, ThorSMRenumbering, map, CudaToolkitVersionSatisfies, make_math_instruction_w_output, TileDescription。

### L9685-L9830 — Function `GenerateSM100_TensorOp_fp8_UMMA_conv3x`

```python
 9685: def GenerateSM100_TensorOp_fp8_UMMA_conv3x(manifest, cuda_version,
 9686:                                            log_indent_level: int = 0):
 9687:   # Instantiate Fp8 Fprop kernels with e4m3 A/B, f32 Acc, e4m3/bf16/f16/f32 C/D
 9688:   log_debug_line('GenerateSM100_TensorOp_fp8_UMMA_conv3x', log_indent_level)
 9689:   log_indent_level = log_indent_level + 1
 9690: 
 9691:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
 9692:     return
 9693: 
 9694:   thor_sm = ThorSMRenumbering(cuda_version)
 9695: 
 9696:   minimum_compute_capability = 100
 9697:   maximum_compute_capability = thor_sm
 9698: 
 9699:   spatial_dims = [2, 3]
 9700:   stages = 0 # zero means "deduce the number of stages automatically"
 9701: 
 9702:   data_types_and_instruction_shapes_1sm = [
 9703:     # ((A,B,Acc,C/D), (InstM,InstN,InstK))
 9704:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.e4m3),   (64, 128, 32)),
 9705:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.e4m3),   (128, 128, 32)),
 9706:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.e4m3),   (128, 256, 32)),
 9707:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.f16),    (64, 128, 32)),
 9708:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.f16),    (128, 128, 32)),
 9709:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.f16),    (128, 256, 32)),
 9710:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.bf16),   (64, 128, 32)),
 9711:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.bf16),   (128, 128, 32)),
 9712:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.bf16),   (128, 256, 32)),
 9713:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.f32),    (64, 128, 32)),
 9714:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.f32),    (128, 128, 32)),
 9715:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.f32),    (128, 256, 32)),
 9716:   ]
 9717:   math_instructions_w_output_1sm = map(lambda x: make_math_instruction_w_output(*x),
 9718:                           data_types_and_instruction_shapes_1sm)
 9719: 
 9720:   cluster_shapes_1sm = [[1,1,1], [1,2,1], [1,4,1],[4,4,1]]
 9721:   if thor_sm in manifest.compute_capabilities_baseline :
 9722:     cluster_shapes_1sm = [[1,1,1], [1,2,1], [1,4,1]]
 9723: 
 9724:   for math_inst, output_type in math_instructions_w_output_1sm:
 9725:     tile_descriptions = []
 9726:     for cluster_shape in cluster_shapes_1sm:
 9727:       cluster_multiplier = cluster_shape
 9728:       # Unlike SM90, SM100 tile shape calculation includes cluster shape.
 9729:       tile_shape = [
 9730:         math_inst.instruction_shape[0]     * cluster_multiplier[0],
 9731:         math_inst.instruction_shape[1]     * cluster_multiplier[1],
 9732:         math_inst.instruction_shape[2] * 4 * cluster_multiplier[2]
 9733:       ]
 9734:       warp_count = [4, 1, 1]
 9735:       tile_description = TileDescription(
 9736:         tile_shape, stages, warp_count, math_inst,
 9737:         minimum_compute_capability, maximum_compute_capability,
 9738:         cluster_shape)
 9739:       tile_descriptions.append(tile_description)
 9740: 
 9741:       data_type = {
 9742:         "a_type"   : math_inst.element_a,
 9743:         "b_type"   : math_inst.element_b,
 9744:         "c_type"   : output_type,
 9745:         "d_type"   : output_type,
 9746:         "acc_type" : math_inst.element_accumulator,
 9747:         "epi_type" : math_inst.element_accumulator
 9748:       }
 9749: 
 9750:       dims_and_alignments = [make_dims_and_alignments_triple(dim, DataTypeSize[data_type["a_type"]], DataTypeSize[data_type["b_type"]], DataTypeSize[data_type["d_type"]]) for dim in spatial_dims]
 9751: 
 9752:       # Schedules
 9753:       mainloop_schedule = KernelScheduleType.ImplicitTmaWarpSpecialized1SmSm100
 9754:       epilogue_schedule = EpilogueScheduleType.ScheduleAuto
 9755:       schedule_pairs = [
 9756:         (mainloop_schedule, epilogue_schedule)
 9757:       ]
 9758: 
 9759:       CreateConvOperator3x(manifest,
 9760:                           dims_and_alignments = dims_and_alignments,
 9761:                           tile_descriptions = tile_descriptions,
 9762:                           data_types = data_type,
 9763:                           schedule_pairs = schedule_pairs,
 9764:                           conv_kind = ConvKind.Fprop,
 9765:                           log_indent_level = log_indent_level)
 9766: 
 9767:   data_types_and_instruction_shapes_2sm = [
 9768:     # ((A,B,Acc,C/D), (InstM,InstN,InstK))
 9769:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.e4m3),   (128, 128, 32)),
 9770:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.e4m3),   (128, 256, 32)),
 9771:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.e4m3),   (256, 256, 32)),
 9772:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.f16),    (128, 128, 32)),
 9773:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.f16),    (128, 256, 32)),
 9774:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.f16),    (256, 256, 32)),
 9775:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.bf16),   (128, 128, 32)),
 9776:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.bf16),   (128, 256, 32)),
 9777:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.bf16),   (256, 256, 32)),
 9778:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.f32),    (128, 128, 32)),
 9779:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.f32),    (128, 256, 32)),
 9780:     ((DataType.e4m3, DataType.e4m3, DataType.f32, DataType.f32),    (256, 256, 32)),
 9781:   ]
 9782:   math_instructions_w_output_2sm = map(lambda x: make_math_instruction_w_output(*x),
 9783:                           data_types_and_instruction_shapes_2sm)
 9784: 
 9785:   cluster_shapes_2sm = [[2,1,1], [2,2,1], [2,4,1], [4,1,1], [4,2,1], [4,4,1]]
 9786:   if thor_sm in manifest.compute_capabilities_baseline :
 9787:     cluster_shapes_2sm = [[2,1,1], [2,2,1], [2,4,1], [4,1,1], [4,2,1]]
 9788: 
 9789:   for math_inst, output_type in math_instructions_w_output_2sm:
 9790:     tile_descriptions = []
 9791:     for cluster_shape in cluster_shapes_2sm:
 9792:       cluster_multiplier = (cluster_shape[0] // 2, cluster_shape[1], cluster_shape[2])
 9793:       # Unlike SM90, SM100 tile shape calculation includes cluster shape.
 9794:       tile_shape = [
 9795:         math_inst.instruction_shape[0]     * cluster_multiplier[0],
 9796:         math_inst.instruction_shape[1]     * cluster_multiplier[1],
 9797:         math_inst.instruction_shape[2] * 4 * cluster_multiplier[2]
 9798:       ]
 9799:       warp_count = [4, 1, 1]
 9800:       tile_description = TileDescription(
 9801:         tile_shape, stages, warp_count, math_inst,
 9802:         minimum_compute_capability, maximum_compute_capability,
 9803:         cluster_shape)
 9804:       tile_descriptions.append(tile_description)
 9805: 
 9806:       data_type = {
 9807:         "a_type"   : math_inst.element_a,
 9808:         "b_type"   : math_inst.element_b,
 9809:         "c_type"   : output_type,
 9810:         "d_type"   : output_type,
 9811:         "acc_type" : math_inst.element_accumulator,
 9812:         "epi_type" : math_inst.element_accumulator
 9813:       }
 9814: 
 9815:       dims_and_alignments = [make_dims_and_alignments_triple(dim, DataTypeSize[data_type["a_type"]], DataTypeSize[data_type["b_type"]], DataTypeSize[data_type["d_type"]]) for dim in spatial_dims]
 9816: 
 9817:       # Schedules
 9818:       mainloop_schedule = KernelScheduleType.ImplicitTmaWarpSpecialized2SmSm100
 9819:       epilogue_schedule = EpilogueScheduleType.ScheduleAuto
 9820:       schedule_pairs = [
 9821:         (mainloop_schedule, epilogue_schedule)
 9822:       ]
 9823: 
 9824:       CreateConvOperator3x(manifest,
 9825:                           dims_and_alignments = dims_and_alignments,
 9826:                           tile_descriptions = tile_descriptions,
 9827:                           data_types = data_type,
 9828:                           schedule_pairs = schedule_pairs,
 9829:                           conv_kind = ConvKind.Fprop,
 9830:                           log_indent_level = log_indent_level)
```
**EN:** Defines `GenerateSM100_TensorOp_fp8_UMMA_conv3x()`, which generates sm100 tensor op fp8 umma conv3x. Key helper calls include log_debug_line, ThorSMRenumbering, map, CudaToolkitVersionSatisfies, make_math_instruction_w_output, TileDescription.

**CN:** 定义 `GenerateSM100_TensorOp_fp8_UMMA_conv3x()`，用于生成 sm100 tensor op fp8 umma conv3x。 其中会调用的重要辅助函数包括 log_debug_line, ThorSMRenumbering, map, CudaToolkitVersionSatisfies, make_math_instruction_w_output, TileDescription。

### L9832-L9958 — Function `GenerateSM120_TensorOp_mixed_8bits_UMMA_gemm_with_block_scaled`

```python
 9832: def GenerateSM120_TensorOp_mixed_8bits_UMMA_gemm_with_block_scaled(manifest, cuda_version):
 9833:   # SM120 MMA with mixed F4/F6/F8 inputs + block scale
 9834:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 8):
 9835:     return
 9836: 
 9837:   layouts = [
 9838:     [[LayoutType.RowMajor,    128], [LayoutType.ColumnMajor, 128], [LayoutType.RowMajor,    0]]
 9839:   ]
 9840: 
 9841:   instruction_sizes = [
 9842:     [16, 8, 32]
 9843:   ]
 9844: 
 9845:   tile_sizes = [
 9846:     [128, 128, 128]
 9847:   ]
 9848: 
 9849:   cluster_shape = [1,1,1]
 9850: 
 9851:   ab_types  = [
 9852:     DataType.e2m1, 
 9853:     DataType.e2m3, 
 9854:     DataType.e3m2,
 9855:     DataType.e5m2,
 9856:     DataType.e4m3,
 9857:   ]
 9858: 
 9859:   acc_types = [ DataType.f32 ]
 9860: 
 9861:   def is_pingpong(kernel_schedule):
 9862:     if kernel_schedule == KernelScheduleType.Mxf8f6f4TmaWarpSpecializedPingpongSm120:
 9863:       return True
 9864:     else:
 9865:       return False
 9866:     
 9867:   def tile_schedulers(sfdtype, kernel_schedule):
 9868:     # Pingpong kernel schedule doesn't support stream-K.
 9869:     # Only use the stream-K scheduler for non-void SFD to limit kernel count. When SFD is void,
 9870:     # the epilogue is the traditional linear combination, for which we already have tests with stream-K
 9871:     if is_pingpong(kernel_schedule):
 9872:       return [TileSchedulerType.Default]
 9873:     elif sfdtype["type"] == DataType.void:
 9874:       return [TileSchedulerType.Default]
 9875:     else:
 9876:       return [TileSchedulerType.Default, TileSchedulerType.StreamK]
 9877: 
 9878:   min_cc = 120
 9879:   max_cc = 121
 9880: 
 9881:   epi_type = DataType.f32
 9882:   
 9883:   math_instructions = []
 9884: 
 9885:   kernel_schedules = [
 9886:     KernelScheduleType.Mxf8f6f4TmaWarpSpecializedCooperativeSm120,
 9887:     KernelScheduleType.Mxf8f6f4TmaWarpSpecializedPingpongSm120
 9888:   ]
 9889: 
 9890:   for instr_size, a_type, b_type, acc_type in product(instruction_sizes, ab_types, ab_types, acc_types):
 9891:     math_instructions.append(
 9892:       MathInstruction(
 9893:         instr_size,
 9894:         a_type, b_type, acc_type,
 9895:         OpcodeClass.BlockScaledTensorOp,
 9896:         MathOperation.multiply_add,
 9897:         DataType.ue8m0)
 9898:     )
 9899: 
 9900:   for math_inst in math_instructions:
 9901:     tile_descriptions = []
 9902:     for tile_size in tile_sizes:
 9903:       tile_descriptions.append(
 9904:         TileDescription(tile_size, 0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
 9905: 
 9906:     data_types = [
 9907:       {
 9908:         "a_type"   : math_inst.element_a,
 9909:         "b_type"   : math_inst.element_b,
 9910:         "c_type"   : DataType.void,
 9911:         "d_type"   : DataType.f32,
 9912:         "acc_type" : math_inst.element_accumulator,
 9913:         "epi_type" : epi_type,
 9914:         "sf_type"  : math_inst.element_scale_factor,
 9915:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 9916:       },
 9917:       {
 9918:         "a_type"   : math_inst.element_a,
 9919:         "b_type"   : math_inst.element_b,
 9920:         "c_type"   : DataType.void,
 9921:         "d_type"   : DataType.e5m2,
 9922:         "acc_type" : math_inst.element_accumulator,
 9923:         "epi_type" : epi_type,
 9924:         "sf_type"  : math_inst.element_scale_factor,
 9925:         "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
 9926:       },
 9927:       {
 9928:         "a_type"   : math_inst.element_a,
 9929:         "b_type"   : math_inst.element_b,
 9930:         "c_type"   : DataType.f16,
 9931:         "d_type"   : DataType.e5m2,
 9932:         "acc_type" : math_inst.element_accumulator,
 9933:         "epi_type" : epi_type,
 9934:         "sf_type"  : math_inst.element_scale_factor,
 9935:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
 9936:       },
 9937:       {
 9938:         "a_type"   : math_inst.element_a,
 9939:         "b_type"   : math_inst.element_b,
 9940:         "c_type"   : DataType.f16,
 9941:         "d_type"   : DataType.e3m2,
 9942:         "acc_type" : math_inst.element_accumulator,
 9943:         "epi_type" : epi_type,
 9944:         "sf_type"  : math_inst.element_scale_factor,
 9945:         "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
 9946:       }
 9947:     ]
 9948: 
 9949:     # Set alignment d based on Destination format.
 9950:     for layout in layouts:
 9951:       layout[2][1] = 128 // DataTypeSize[data_types[0]["d_type"]]
 9952: 
 9953:     for data_type, kernel_schedule in product(data_types, kernel_schedules):
 9954:       CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_type,
 9955:         [[kernel_schedule, EpilogueScheduleType.ScheduleAuto]], 
 9956:         tile_schedulers = tile_schedulers(data_type["sfd_type"], kernel_schedule),
 9957:         gemm_kind = GemmKind.BlockScaledUniversal3x
 9958:         )
```
**EN:** Defines `GenerateSM120_TensorOp_mixed_8bits_UMMA_gemm_with_block_scaled()`, which generates sm120 tensor op mixed 8bits umma gemm with block scaled. Key helper calls include product, CudaToolkitVersionSatisfies, is_pingpong, append, MathInstruction, CreateGemmUniversal3xOperator.

**CN:** 定义 `GenerateSM120_TensorOp_mixed_8bits_UMMA_gemm_with_block_scaled()`，用于生成 sm120 tensor op mixed 8bits umma gemm with block scaled。 其中会调用的重要辅助函数包括 product, CudaToolkitVersionSatisfies, is_pingpong, append, MathInstruction, CreateGemmUniversal3xOperator。

### L9960-L10141 — Function `GenerateSM120_TensorOp_fp4_UMMA_gemm_with_block_scaled`

```python
 9960: def GenerateSM120_TensorOp_fp4_UMMA_gemm_with_block_scaled(manifest, cuda_version):
 9961:   # SM120 MMA with with F4 + block scale
 9962:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 8):
 9963:     return
 9964: 
 9965:   # layouts for ABC and their alignments.
 9966:   layouts = [
 9967:     [[LayoutType.RowMajor,    32], [LayoutType.ColumnMajor, 32], [LayoutType.RowMajor,    0]]
 9968:   ]
 9969: 
 9970:   instruction_sizes = [
 9971:     [16, 8, 64]
 9972:   ]
 9973: 
 9974:   tile_sizes_cooperative = [
 9975:     [128, 128, 128],
 9976:     [128, 128, 256],
 9977:     [256, 128, 128]
 9978:   ]
 9979: 
 9980:   tile_sizes_pingpong = [
 9981:     [128, 128, 128],
 9982:     [128, 128, 256]
 9983:   ]
 9984: 
 9985:   cluster_shape = [1,1,1]
 9986: 
 9987:   ab_types  = [
 9988:     DataType.e2m1
 9989:   ]
 9990: 
 9991:   sf_types  = [
 9992:     DataType.ue4m3,
 9993:     DataType.ue8m0
 9994:   ]
 9995: 
 9996:   acc_types = [ DataType.f32 ]
 9997: 
 9998:   def is_pingpong(kernel_schedule):
 9999:     if kernel_schedule == KernelScheduleType.Nvf4TmaWarpSpecializedPingpongSm120 or \
10000:        kernel_schedule == KernelScheduleType.Mxf4TmaWarpSpecializedPingpongSm120:
10001:       return True
10002:     else:
10003:       return False
10004:   
10005:   def is_nvf4(kernel_schedule):
10006:     if kernel_schedule == KernelScheduleType.Nvf4TmaWarpSpecializedCooperativeSm120 or \
10007:        kernel_schedule == KernelScheduleType.Nvf4TmaWarpSpecializedPingpongSm120:
10008:       return True
10009:     else:
10010:       return False
10011:     
10012:   def tile_schedulers(sfdtype, kernel_schedule):
10013:     # Pingpong kernel schedule doesn't support stream-K.
10014:     # Only use the stream-K scheduler for non-void SFD to limit kernel count. When SFD is void,
10015:     # the epilogue is the traditional linear combination, for which we already have tests with stream-K
10016:     if is_pingpong(kernel_schedule):
10017:       return [TileSchedulerType.Default]
10018:     elif sfdtype["type"] == DataType.void:
10019:       return [TileSchedulerType.Default]
10020:     else:
10021:       return [TileSchedulerType.Default, TileSchedulerType.StreamK]
10022: 
10023:   min_cc = 120
10024:   max_cc = 121
10025: 
10026:   epi_type = DataType.f32
10027:   
10028:   math_instructions = []
10029: 
10030:   kernel_schedules = [
10031:     KernelScheduleType.Nvf4TmaWarpSpecializedCooperativeSm120,
10032:     KernelScheduleType.Nvf4TmaWarpSpecializedPingpongSm120,
10033:     KernelScheduleType.Mxf4TmaWarpSpecializedCooperativeSm120,
10034:     KernelScheduleType.Mxf4TmaWarpSpecializedPingpongSm120
10035:   ]
10036: 
10037:   for instr_size, a_type, b_type, acc_type, sf_type in product(instruction_sizes, ab_types, ab_types, acc_types, sf_types):
10038:     math_instructions.append(
10039:       MathInstruction(
10040:         instr_size,
10041:         a_type, b_type, acc_type,
10042:         OpcodeClass.BlockScaledTensorOp,
10043:         MathOperation.multiply_add,
10044:         sf_type)
10045:     )
10046: 
10047:   for math_inst in math_instructions:
10048:     for kernel_schedule in kernel_schedules:
10049:       tile_descriptions = []
10050:       tile_sizes = tile_sizes_pingpong if is_pingpong(kernel_schedule) else tile_sizes_cooperative
10051:       for tile_size in tile_sizes:
10052:         # nvf4 kernel only supports ue4m3 SF
10053:         # mxf4 kernel only supports ue8m0 SF
10054:         if (math_inst.element_scale_factor == DataType.ue4m3 and is_nvf4(kernel_schedule)) or \
10055:            (math_inst.element_scale_factor == DataType.ue8m0 and not is_nvf4(kernel_schedule)):
10056:           tile_descriptions.append(
10057:             TileDescription(tile_size, 0, [4, 1, 1], math_inst, min_cc, max_cc, cluster_shape))
10058: 
10059:       data_types = [
10060:         {
10061:           "a_type"   : math_inst.element_a,
10062:           "b_type"   : math_inst.element_b,
10063:           "c_type"   : DataType.void,
10064:           "d_type"   : DataType.f32,
10065:           "acc_type" : math_inst.element_accumulator,
10066:           "epi_type" : epi_type,
10067:           "sf_type"  : math_inst.element_scale_factor,
10068:           "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
10069:         },
10070:         {
10071:           "a_type"   : math_inst.element_a,
10072:           "b_type"   : math_inst.element_b,
10073:           "c_type"   : DataType.void,
10074:           "d_type"   : DataType.e2m1,
10075:           "acc_type" : math_inst.element_accumulator,
10076:           "epi_type" : epi_type,
10077:           "sf_type"  : math_inst.element_scale_factor,
10078:           "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
10079:         },
10080:         {
10081:           "a_type"   : math_inst.element_a,
10082:           "b_type"   : math_inst.element_b,
10083:           "c_type"   : DataType.void,
10084:           "d_type"   : DataType.e5m2,
10085:           "acc_type" : math_inst.element_accumulator,
10086:           "epi_type" : epi_type,
10087:           "sf_type"  : math_inst.element_scale_factor,
10088:           "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
10089:         },
10090:         {
10091:           "a_type"   : math_inst.element_a,
10092:           "b_type"   : math_inst.element_b,
10093:           "c_type"   : DataType.f16,
10094:           "d_type"   : DataType.e5m2,
10095:           "acc_type" : math_inst.element_accumulator,
10096:           "epi_type" : epi_type,
10097:           "sf_type"  : math_inst.element_scale_factor,
10098:           "sfd_type" : {"type": DataType.void, "vector_size": None, "layout" : None}
10099:         },
10100:         {
10101:           "a_type"   : math_inst.element_a,
10102:           "b_type"   : math_inst.element_b,
10103:           "c_type"   : DataType.void,
10104:           "d_type"   : DataType.e2m1,
10105:           "acc_type" : math_inst.element_accumulator,
10106:           "epi_type" : epi_type,
10107:           "sf_type"  : math_inst.element_scale_factor,
10108:           "sfd_type" : {"type": DataType.ue8m0, "vector_size": 16, "layout" : LayoutType.RowMajor}
10109:         },
10110:         {
10111:           "a_type"   : math_inst.element_a,
10112:           "b_type"   : math_inst.element_b,
10113:           "c_type"   : DataType.f16,
10114:           "d_type"   : DataType.e2m1,
10115:           "acc_type" : math_inst.element_accumulator,
10116:           "epi_type" : epi_type,
10117:           "sf_type"  : math_inst.element_scale_factor,
10118:           "sfd_type" : {"type": DataType.ue8m0, "vector_size": 16, "layout" : LayoutType.RowMajor}
10119:         },
10120:         {
10121:           "a_type"   : math_inst.element_a,
10122:           "b_type"   : math_inst.element_b,
10123:           "c_type"   : DataType.f16,
10124:           "d_type"   : DataType.e2m1,
10125:           "acc_type" : math_inst.element_accumulator,
10126:           "epi_type" : epi_type,
10127:           "sf_type"  : math_inst.element_scale_factor,
10128:           "sfd_type" : {"type": DataType.ue8m0, "vector_size": 32, "layout" : LayoutType.RowMajor}
10129:         }
10130:       ]
10131: 
10132:       # Set alignment d based on Destination format.
10133:       for layout in layouts:
10134:         layout[2][1] = 128 // DataTypeSize[data_types[0]["d_type"]]
10135: 
10136:       for data_type in data_types:
10137:         CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_type,
10138:           [[kernel_schedule, EpilogueScheduleType.ScheduleAuto]], 
10139:           tile_schedulers = tile_schedulers(data_type["sfd_type"], kernel_schedule),
10140:           gemm_kind = GemmKind.BlockScaledUniversal3x
10141:           ) 
```
**EN:** Defines `GenerateSM120_TensorOp_fp4_UMMA_gemm_with_block_scaled()`, which generates sm120 tensor op fp4 umma gemm with block scaled. Key helper calls include product, CudaToolkitVersionSatisfies, is_pingpong, append, MathInstruction, CreateGemmUniversal3xOperator.

**CN:** 定义 `GenerateSM120_TensorOp_fp4_UMMA_gemm_with_block_scaled()`，用于生成 sm120 tensor op fp4 umma gemm with block scaled。 其中会调用的重要辅助函数包括 product, CudaToolkitVersionSatisfies, is_pingpong, append, MathInstruction, CreateGemmUniversal3xOperator。

### L10143-L10244 — Function `GenerateSM120_Sparse_TensorOp_gemm`

```python
10143: def GenerateSM120_Sparse_TensorOp_gemm(manifest, cuda_version):
10144:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 8):
10145:     return
10146: 
10147:   layouts = [
10148:     [[LayoutType.RowMajor, 256], [LayoutType.ColumnMajor, 128], [LayoutType.RowMajor, 0]]
10149:   ]
10150: 
10151:   tile_sizes = [
10152:     [128, 128, 256]
10153:   ]
10154: 
10155:   cluster_shape = [1,1,1]
10156:   
10157:   warp_count = [4, 2, 1]
10158: 
10159:   acc_types = [ DataType.f32 ]
10160: 
10161:   instruction_sizes_mxf8f6f4 = [
10162:     [16, 8, 64]
10163:   ]
10164: 
10165:   ab_types_mxf8f6f4  = [
10166:     DataType.e2m1, 
10167:     #DataType.e2m3, 
10168:     DataType.e3m2,
10169:     #DataType.e5m2,
10170:     DataType.e4m3,
10171:   ]
10172: 
10173:   def tile_schedulers(kernel_schedule):
10174:       return [TileSchedulerType.Default]
10175: 
10176:   min_cc = 120
10177:   max_cc = 121
10178: 
10179:   kernel_schedules = [
10180:     KernelScheduleType.F8f6f4SparseTmaWarpSpecializedCooperativeSm120,
10181:   ]
10182: 
10183:   math_instructions_mxf8f6f4 = []
10184: 
10185:   for instr_size, a_type, b_type, acc_type in product(instruction_sizes_mxf8f6f4, ab_types_mxf8f6f4, ab_types_mxf8f6f4, acc_types):
10186:     math_instructions_mxf8f6f4.append(
10187:       MathInstruction(
10188:         instr_size,
10189:         a_type, b_type, acc_type,
10190:         OpcodeClass.SparseTensorOp,
10191:         MathOperation.multiply_add)
10192:     )
10193: 
10194:   # Create gemm operator for mxf8f6f4
10195:   for math_inst in math_instructions_mxf8f6f4:
10196:     tile_descriptions_mxf8f6f4 = []
10197:     for tile_size in tile_sizes:
10198:       tile_descriptions_mxf8f6f4.append(
10199:         TileDescription(tile_size, 0, warp_count, math_inst, min_cc, max_cc, cluster_shape))
10200: 
10201:     data_types = [
10202:       {
10203:         "a_type"   : math_inst.element_a,
10204:         "b_type"   : math_inst.element_b,
10205:         "c_type"   : DataType.void,
10206:         "d_type"   : DataType.f32,
10207:         "acc_type" : math_inst.element_accumulator,
10208:         "epi_type" : DataType.f32
10209:       },
10210:       {
10211:         "a_type"   : math_inst.element_a,
10212:         "b_type"   : math_inst.element_b,
10213:         "c_type"   : DataType.void,
10214:         "d_type"   : DataType.e5m2,
10215:         "acc_type" : math_inst.element_accumulator,
10216:         "epi_type" : DataType.f32
10217:       },
10218:       {
10219:         "a_type"   : math_inst.element_a,
10220:         "b_type"   : math_inst.element_b,
10221:         "c_type"   : DataType.f16,
10222:         "d_type"   : DataType.e4m3,
10223:         "acc_type" : math_inst.element_accumulator,
10224:         "epi_type" : DataType.f32
10225:       },
10226:       {
10227:         "a_type"   : math_inst.element_a,
10228:         "b_type"   : math_inst.element_b,
10229:         "c_type"   : DataType.void,
10230:         "d_type"   : DataType.f16,
10231:         "acc_type" : math_inst.element_accumulator,
10232:         "epi_type" : DataType.f32
10233:       }
10234:     ]
10235: 
10236:     for data_type, kernel_schedule in product(data_types, kernel_schedules):
10237:       # Set alignment d based on Destination format
10238:       for layout in layouts:
10239:         layout[2][1] = int(128 // DataTypeSize[data_type["d_type"]])
10240:       # Create gemm operator
10241:       CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions_mxf8f6f4, data_type,
10242:         [[kernel_schedule, EpilogueScheduleType.ScheduleAuto]], 
10243:         tile_schedulers = tile_schedulers(kernel_schedule),
10244:         gemm_kind = GemmKind.SparseUniversal3x)
```
**EN:** Defines `GenerateSM120_Sparse_TensorOp_gemm()`, which generates sm120 sparse tensor op gemm. Key helper calls include product, CudaToolkitVersionSatisfies, append, MathInstruction, CreateGemmUniversal3xOperator, TileDescription.

**CN:** 定义 `GenerateSM120_Sparse_TensorOp_gemm()`，用于生成 sm120 sparse tensor op gemm。 其中会调用的重要辅助函数包括 product, CudaToolkitVersionSatisfies, append, MathInstruction, CreateGemmUniversal3xOperator, TileDescription。

### L10246-L10378 — Function `GenerateSM120_TensorOp_fp8_UMMA_gemm_with_blockwise`

```python
10246: def GenerateSM120_TensorOp_fp8_UMMA_gemm_with_blockwise(manifest, cuda_version, gemm_kind=GemmKind.BlockwiseUniversal3x):
10247:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 8):
10248:     return
10249: 
10250:   layouts = [
10251:     [[LayoutType.RowMajor, 128], [LayoutType.ColumnMajor, 128], [LayoutType.RowMajor, 16]],
10252:     [[LayoutType.RowMajor, 128], [LayoutType.ColumnMajor, 128], [LayoutType.ColumnMajor, 16]]
10253:   ]
10254: 
10255:   cooperative_tile_sizes = [
10256:     [128, 128, 128]
10257:   ]
10258:   pingpong_tile_sizes = [
10259:     [64, 128, 128]
10260:   ]
10261: 
10262:   def get_tile_sizes(kernel_scheduler):
10263:     if kernel_scheduler == KernelScheduleType.BlockwiseTmaWarpSpecializedPingpongSm120:
10264:       return pingpong_tile_sizes
10265:     return cooperative_tile_sizes
10266: 
10267:   def get_warp_count(kernel_scheduler):
10268:     if kernel_scheduler == KernelScheduleType.BlockwiseTmaWarpSpecializedPingpongSm120:
10269:       return [2, 2, 1]
10270:     return [4, 2, 1]
10271: 
10272:   def get_sf_sizes(tile_size):
10273:     sf_sizes = []
10274:     for vec_m in [1, 128]:
10275:       if tile_size[0] % vec_m > 0:
10276:         continue
10277:       for vec_n in [1, 128]:
10278:         if tile_size[1] % vec_m > 0:
10279:           continue
10280:         sf_sizes.append(
10281:           [vec_m, vec_n, 128]
10282:         )
10283:     return sf_sizes
10284: 
10285:   cluster_shape = [1,1,1]
10286: 
10287:   acc_types = [ DataType.f32 ]
10288: 
10289:   instruction_sizes = [
10290:     [16, 8, 32]
10291:   ]
10292: 
10293:   def tile_schedulers(kernel_schedule):
10294:       return [TileSchedulerType.Default]
10295: 
10296:   min_cc = 120
10297:   max_cc = 121
10298: 
10299:   kernel_schedulers = [
10300:     KernelScheduleType.BlockwiseTmaWarpSpecializedCooperativeSm120,
10301:     KernelScheduleType.BlockwiseTmaWarpSpecializedPingpongSm120
10302:   ]
10303: 
10304:   ab_types = [
10305:     [DataType.e4m3, DataType.e4m3],
10306:     [DataType.e4m3, DataType.e5m2]
10307:   ]
10308: 
10309:   math_instructions = []
10310: 
10311:   for instr_size, ab_type, acc_type in product(instruction_sizes, ab_types, acc_types):
10312:     a_type, b_type = ab_type
10313:     math_instructions.append(
10314:       MathInstruction(
10315:         instr_size,
10316:         a_type, b_type, acc_type,
10317:         OpcodeClass.TensorOp,
10318:         MathOperation.multiply_add)
10319:     )
10320: 
10321:   # Create gemm operator for mxf8f6f4
10322:   for kernel_schedule in kernel_schedulers:
10323:     tile_sizes = get_tile_sizes(kernel_schedule)
10324:     warp_count = get_warp_count(kernel_schedule)
10325:     for math_inst in math_instructions:
10326:       tile_descriptions = []
10327:       for tile_size in tile_sizes:
10328:         sf_sizes = get_sf_sizes(tile_size)
10329:         for sf_size in sf_sizes:
10330:           tile_descriptions.append(
10331:             TileDescription(tile_size, 0, warp_count, math_inst, min_cc, max_cc, cluster_shape,
10332:                             explicit_vector_sizes=sf_size)
10333:           )
10334: 
10335:       data_types = [
10336:         {
10337:           "a_type"   : math_inst.element_a,
10338:           "b_type"   : math_inst.element_b,
10339:           "c_type"   : DataType.f16,
10340:           "d_type"   : DataType.f16,
10341:           "acc_type" : math_inst.element_accumulator,
10342:           "epi_type" : DataType.f32
10343:         },
10344:         {
10345:           "a_type"   : math_inst.element_a,
10346:           "b_type"   : math_inst.element_b,
10347:           "c_type"   : DataType.bf16,
10348:           "d_type"   : DataType.bf16,
10349:           "acc_type" : math_inst.element_accumulator,
10350:           "epi_type" : DataType.f32
10351:         },
10352:         {
10353:           "a_type"   : math_inst.element_a,
10354:           "b_type"   : math_inst.element_b,
10355:           "c_type"   : DataType.void,
10356:           "d_type"   : DataType.f16,
10357:           "acc_type" : math_inst.element_accumulator,
10358:           "epi_type" : DataType.f32
10359:         },
10360:         {
10361:           "a_type"   : math_inst.element_a,
10362:           "b_type"   : math_inst.element_b,
10363:           "c_type"   : DataType.void,
10364:           "d_type"   : DataType.bf16,
10365:           "acc_type" : math_inst.element_accumulator,
10366:           "epi_type" : DataType.f32
10367:         }
10368:       ]
10369: 
10370:       for data_type in data_types:
10371:         # Set alignment d based on Destination format
10372:         for layout in layouts:
10373:           layout[2][1] = int(128 // DataTypeSize[data_type["d_type"]])
10374:         # Create gemm operator
10375:         CreateGemmUniversal3xOperator(manifest, layouts, tile_descriptions, data_type,
10376:           [[kernel_schedule, EpilogueScheduleType.ScheduleAuto]], 
10377:           tile_schedulers = tile_schedulers(kernel_schedule),
10378:           gemm_kind = gemm_kind)
```
**EN:** Defines `GenerateSM120_TensorOp_fp8_UMMA_gemm_with_blockwise()`, which generates sm120 tensor op fp8 umma gemm with blockwise. Key helper calls include product, CudaToolkitVersionSatisfies, append, get_tile_sizes, get_warp_count, MathInstruction.

**CN:** 定义 `GenerateSM120_TensorOp_fp8_UMMA_gemm_with_blockwise()`，用于生成 sm120 tensor op fp8 umma gemm with blockwise。 其中会调用的重要辅助函数包括 product, CudaToolkitVersionSatisfies, append, get_tile_sizes, get_warp_count, MathInstruction。

### L10380-L10432 — Function `GenerateSM100`

```python
10380: def GenerateSM100(manifest, cuda_version):
10381:   arch_family_cc = ['100f', '101f', '103a']
10382:   if CudaToolkitVersionSatisfies(cuda_version, 13, 0):
10383:     for old_cc, new_cc in [('101f', '110f')]:
10384:       arch_family_cc = [cc.replace(old_cc, new_cc) for cc in arch_family_cc]
10385: 
10386:   #
10387:   # Dense Gemm
10388:   #
10389:   GenerateSM100_TensorOp_16b_UMMA_gemm(manifest, cuda_version)
10390: 
10391:   GenerateSM100_TensorOp_32b_UMMA_gemm(manifest, cuda_version)
10392: 
10393:   if not bool(set(manifest.compute_capabilities_feature_set).intersection(arch_family_cc)):
10394:     GenerateSM100_TensorOp_int8_UMMA_gemm(manifest, cuda_version)
10395: 
10396:   GenerateSM100_TensorOp_fp8_UMMA_gemm(manifest, cuda_version)
10397:   # grouped GEMM
10398:   GenerateSM100_TensorOp_fp8_UMMA_gemm(manifest, cuda_version, gemm_kind=GemmKind.GroupedUniversal3x)
10399:   GenerateSM100_TensorOp_16b_UMMA_gemm(manifest, cuda_version, gemm_kind=GemmKind.GroupedUniversal3x)
10400: 
10401:   # StreamK is included in regular generation
10402:   GenerateSM100_TensorOp_mixed_8bits_UMMA_gemm(manifest, cuda_version)
10403: 
10404:   # Blockwise kernels
10405:   GenerateSM100_TensorOp_fp8_UMMA_gemm_with_blockwise(manifest, cuda_version)
10406:   GenerateSM100_TensorOp_fp8_UMMA_gemm_with_blockwise(manifest, cuda_version, gemm_kind=GemmKind.GroupedBlockwiseUniversal3x)
10407: 
10408:   #
10409:   # Sparse Gemm
10410:   #
10411:   GenerateSM100_SparseTensorOp_32b_UMMA_gemm(manifest, cuda_version)
10412:   GenerateSM100_SparseTensorOp_16b_UMMA_gemm(manifest, cuda_version)
10413:   if not bool(set(manifest.compute_capabilities_feature_set).intersection(arch_family_cc)):
10414:     GenerateSM100_SparseTensorOp_int8_UMMA_gemm(manifest, cuda_version)
10415:   GenerateSM100_SparseTensorOp_fp8_UMMA_gemm(manifest, cuda_version)
10416:   GenerateSM100_SparseTensorOp_mixed_8bits_UMMA_gemm(manifest, cuda_version)
10417: 
10418:   #
10419:   # Block Scaled Gemm
10420:   #
10421:   GenerateSM100_TensorOp_mixed_8bits_UMMA_gemm_with_block_scaled(manifest, cuda_version)
10422:   GenerateSM100_TensorOp_mixed_8bits_UMMA_gemm_with_block_scaled(manifest, cuda_version, gemm_kind=GemmKind.GroupedBlockScaledUniversal3x)
10423:   GenerateSM100_TensorOp_fp4_UMMA_gemm_with_block_scaled(manifest, cuda_version)
10424:   GenerateSM100_TensorOp_fp4_UMMA_gemm_with_block_scaled(manifest, cuda_version,  gemm_kind=GemmKind.GroupedBlockScaledUniversal3x)
10425:   
10426:   GenerateSM103_TensorOp_fp4_ultra_UMMA_gemm_with_block_scaled(manifest, cuda_version)
10427:   GenerateSM103_TensorOp_fp4_ultra_UMMA_gemm_with_block_scaled(manifest, cuda_version, gemm_kind=GemmKind.GroupedBlockScaledUniversal3x)
10428:   #
10429:   # Conv
10430:   #
10431:   GenerateSM100_TensorOp_16b_UMMA_conv3x(manifest, cuda_version)
10432:   GenerateSM100_TensorOp_fp8_UMMA_conv3x(manifest, cuda_version)
```
**EN:** Defines `GenerateSM100()`, which generates sm100. Key helper calls include CudaToolkitVersionSatisfies, GenerateSM100_TensorOp_16b_UMMA_gemm, GenerateSM100_TensorOp_32b_UMMA_gemm, GenerateSM100_TensorOp_fp8_UMMA_gemm, GenerateSM100_TensorOp_mixed_8bits_UMMA_gemm, GenerateSM100_TensorOp_fp8_UMMA_gemm_with_blockwise.

**CN:** 定义 `GenerateSM100()`，用于生成 sm100。 其中会调用的重要辅助函数包括 CudaToolkitVersionSatisfies, GenerateSM100_TensorOp_16b_UMMA_gemm, GenerateSM100_TensorOp_32b_UMMA_gemm, GenerateSM100_TensorOp_fp8_UMMA_gemm, GenerateSM100_TensorOp_mixed_8bits_UMMA_gemm, GenerateSM100_TensorOp_fp8_UMMA_gemm_with_blockwise。

### L10435-L10448 — Function `GenerateSM120`

```python
10435: def GenerateSM120(manifest, cuda_version):
10436:   # StreamK is included in regular generation #
10437:   #
10438:   # Dense Block Scaled Gemm
10439:   #
10440:   GenerateSM120_TensorOp_mixed_8bits_UMMA_gemm_with_block_scaled(manifest, cuda_version)
10441:   GenerateSM120_TensorOp_fp4_UMMA_gemm_with_block_scaled(manifest, cuda_version)
10442: 
10443:   #
10444:   # Sparse Gemm
10445:   #
10446:   GenerateSM120_Sparse_TensorOp_gemm(manifest, cuda_version)
10447:   GenerateSM120_TensorOp_fp8_UMMA_gemm_with_blockwise(manifest, cuda_version)
10448:   GenerateSM120_TensorOp_fp8_UMMA_gemm_with_blockwise(manifest, cuda_version, gemm_kind=GemmKind.GroupedBlockwiseUniversal3x)
```
**EN:** Defines `GenerateSM120()`, which generates sm120. Key helper calls include GenerateSM120_TensorOp_mixed_8bits_UMMA_gemm_with_block_scaled, GenerateSM120_TensorOp_fp4_UMMA_gemm_with_block_scaled, GenerateSM120_Sparse_TensorOp_gemm, GenerateSM120_TensorOp_fp8_UMMA_gemm_with_blockwise.

**CN:** 定义 `GenerateSM120()`，用于生成 sm120。 其中会调用的重要辅助函数包括 GenerateSM120_TensorOp_mixed_8bits_UMMA_gemm_with_block_scaled, GenerateSM120_TensorOp_fp4_UMMA_gemm_with_block_scaled, GenerateSM120_Sparse_TensorOp_gemm, GenerateSM120_TensorOp_fp8_UMMA_gemm_with_blockwise。

### L10449-L10451 — Comments

```python
10449: 
10450: ###################################################################################################
10451: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L10452-L10821 — Function `GenerateSM90_Conv3x`

```python
10452: def GenerateSM90_Conv3x(manifest, cuda_version,
10453:                         log_indent_level: int = 0):
10454:   """
10455:   Generate CUTLASS 3 convolution kernel(s) for SM90.
10456: 
10457:   This is meant to be called from GenerateSM90.
10458:   """
10459:   log_debug_line('GenerateSM90_Conv3x', log_indent_level)
10460:   log_indent_level = log_indent_level + 1
10461: 
10462:   if not CudaToolkitVersionSatisfies(cuda_version, 12, 0):
10463:     return
10464: 
10465:   minimum_compute_capability = 90
10466:   maximum_compute_capability = 90
10467: 
10468:   spatial_dims = (2, 3)
10469: 
10470:   # MMA shapes (MMA_M, MMA_N, MMA_K):
10471:   #
10472:   # Different hardware MMA instructions may have different MMA shapes.
10473:   # This function may generate kernels with different MMA shapes for
10474:   # different data types, either because the hardware only supports
10475:   # certain shapes for certain types, or for performance reasons
10476:   # (CUTLASS doesn't need to generate all valid kernels for the
10477:   # profiler library, just the best-performing ones).
10478:   #
10479:   # The kernel names refer to tile shapes (TILE_M, TILE_N, TILE_K)
10480:   # instead of MMA shapes.  For SM >= 90 kernels, TILE_K = 4 * MMA_K,
10481:   # where 4, the "number of MMA instructions per tile," is determined
10482:   # through some combination of modeling and experiment.
10483:   #
10484:   # For performance on sm90, generally CUTLASS generates 64x128
10485:   # instead of 128x64.
10486:   mma_64x64x16  = ( 64,  64,  16)
10487:   mma_64x64x8   = ( 64,  64,   8)
10488: 
10489:   num_mma_per_tile = 4
10490: 
10491:   # Cluster shapes (1, 1, 1) and (2, 2, 1) are valid,
10492:   # but not included, because they tend not to perform as well.
10493:   cluster_shapes = (
10494:     (2, 1, 1),
10495:     (1, 2, 1),
10496:    )
10497: 
10498:   fp16 = DataType.f16
10499:   bf16 = DataType.bf16
10500:   fp32 = DataType.f32
10501:   s8   = DataType.s8
10502:   s32  = DataType.s32
10503: 
10504:   # When generating kernels, the usual way is to specify 4 types,
10505:   # (A, B, Acc, C/D).  Tests instead have 5 types,
10506:   # (ElementAct, ElementFlt, ElementOut, ElementAcc, ElementCompute),
10507:   # where ElementCompute is also called 'epi_type',
10508:   # and corresponds to the type of epilogue activations.
10509:   # This script maps tests' 5 types to 4 types
10510:   # by making ElementCompute the same as ElementOut.
10511: 
10512:   fp16_fp32_fp16_fp32 = {
10513:     'a_type':   fp16, # ElementAct(ivation)
10514:     'b_type':   fp16, # ElementF(i)lt(er)
10515:     'c_type':   fp32, # ElementAcc
10516:     'd_type':   fp32, # ElementOut (used only by CollectiveEpilogue)
10517:     'acc_type': fp16, # ElementAcc
10518:     'epi_type': fp32, # ElementCompute (used only by CollectiveEpilogue)
10519:     'alignment_A': 8, # tma alignment elements of A
10520:     'alignment_B': 8, # tma alignment elements of B
10521:     'alignment_C': 4, # tma alignment elements of C
10522:   }
10523:   fp16_fp32_fp32_fp32 = {
10524:     'a_type':   fp16,
10525:     'b_type':   fp16,
10526:     'c_type':   fp32,
10527:     'd_type':   fp32,
10528:     'acc_type': fp32,
10529:     'epi_type': fp32,
10530:     'alignment_A': 8,
10531:     'alignment_B': 8,
10532:     'alignment_C': 4,
10533:   }
10534:   fp32_fp32_fp32_fp32 = {
10535:     'a_type':   fp32,
10536:     'b_type':   fp32,
10537:     'c_type':   fp32,
10538:     'd_type':   fp32,
10539:     'acc_type': fp32,
10540:     'epi_type': fp32,
10541:     'alignment_A': 4,
10542:     'alignment_B': 4,
10543:     'alignment_C': 4,
10544:   }
10545:   s8_s32_s32_s32 = {
10546:     'a_type':     s8,
10547:     'b_type':     s8,
10548:     'c_type':    s32,
10549:     'd_type':    s32,
10550:     'acc_type':  s32,
10551:     'epi_type':  s32,
10552:     'alignment_A': 16,
10553:     'alignment_B': 16,
10554:     'alignment_C': 4,
10555:   }
10556: 
10557:   # Other NVIDIA libraries may have the habit of specifying data types like this.
10558:   bf16bf16_bf16f32_f32 = {
10559:     'a_type':   bf16,
10560:     'b_type':   bf16,
10561:     'c_type':   fp32,
10562:     'd_type':   fp32,
10563:     'acc_type': fp32,
10564:     'epi_type': fp32,
10565:     'alignment_A': 8,
10566:     'alignment_B': 8,
10567:     'alignment_C': 4,
10568:   }
10569:   f16f16_f16f16_f16 = {
10570:     'a_type':   fp16,
10571:     'b_type':   fp16,
10572:     'c_type':   fp16,
10573:     'd_type':   fp16,
10574:     'acc_type': fp16,
10575:     'epi_type': fp16,
10576:     'alignment_A': 8,
10577:     'alignment_B': 8,
10578:     'alignment_C': 8,
10579:   }
10580:   f16f16_f16f32_f32 = {
10581:     'a_type':   fp16,
10582:     'b_type':   fp16,
10583:     'c_type':   fp16,
10584:     'd_type':   fp16,
10585:     'acc_type': fp32,
10586:     'epi_type': fp32,
10587:     'alignment_A': 8,
10588:     'alignment_B': 8,
10589:     'alignment_C': 8,
10590:   }
10591:   f32f32_tf32f32_f32 = fp32_fp32_fp32_fp32
10592: 
10593:   i8i8_i8i32_f32 = {
10594:     'a_type':     s8,
10595:     'b_type':     s8,
10596:     'c_type':    s32,
10597:     'd_type':    s32,
10598:     'acc_type':  s32,
10599:     'epi_type':  s32,
10600:     'alignment_A': 16,
10601:     'alignment_B': 16,
10602:     'alignment_C': 4,
10603:   }
10604: 
10605:   # Each element in the outermost iterable is one combination of
10606:   #
10607:   # (ConvKind, spatial_dimension, data_types, byte_alignments, mma_sizes, cluster_sizes)
10608:   #
10609:   # for which to generate a kernel.  spatial_dimension is the spatial
10610:   # dimension of the convolution: either 1, 2, or 3.  byte_alignments
10611:   # is a triple of required minimum byte alignments for A, B, and C.
10612:   #
10613:   # Note that itertools functions produce a single-pass generator.
10614:   # The code doesn't need a multipass iterable, but if one did, one
10615:   # could call `tuple` or `list` on the generator.
10616:   #
10617:   # While this happens to use the same cluster sizes for each element,
10618:   # the code doesn't require that.  Different convolution kinds, data
10619:   # types, or mma sizes might have different optimal cluster sizes.
10620:   combinations_of_parameters = chain(
10621:     # The following are all the kernels exercised in the unit tests.
10622:     # Please try to keep in sync with the unit tests.
10623:     product(
10624:       (
10625:         ConvKind.Fprop,
10626:       ),
10627:       spatial_dims,
10628:       (
10629:         fp16_fp32_fp16_fp32,
10630:         fp16_fp32_fp32_fp32,
10631:         s8_s32_s32_s32,
10632:       ),
10633:       (
10634:         mma_64x64x16,
10635:       ),
10636:       cluster_shapes
10637:     ),
10638:     product(
10639:       (
10640:         ConvKind.Fprop,
10641:       ),
10642:       spatial_dims,
10643:       (
10644:         fp32_fp32_fp32_fp32,
10645:       ),
10646:       (
10647:         mma_64x64x8,
10648:       ),
10649:       cluster_shapes
10650:     ),
10651:     product(
10652:       (
10653:         ConvKind.Dgrad,
10654:         ConvKind.Wgrad
10655:       ),
10656:       spatial_dims,
10657:       (
10658:         fp16_fp32_fp16_fp32,
10659:         fp16_fp32_fp32_fp32,
10660:       ),
10661:       (
10662:         mma_64x64x16,
10663:       ),
10664:       cluster_shapes
10665:     ),
10666:     # Kernels not necessarily in the unit tests, but used elsewhere
10667:     # and thus useful to have generated for profiling.  They may
10668:     # duplicate kernels above.  All of them are 2-D.  In general,
10669:     # CUTLASS prefers 64 x 128 to 128 x 64 on sm90, even if the
10670:     # hardware permits 128 x 64.
10671:     (
10672:       # Fprop
10673:       #
10674:       # bf16bf16_bf16f32_f32
10675:       #
10676:       # cluster shape (2, 1, 1)
10677:       #
10678:       (ConvKind.Fprop, 2, bf16bf16_bf16f32_f32, (128, 256,  8), (2, 1, 1)),
10679:       (ConvKind.Fprop, 2, bf16bf16_bf16f32_f32, (128, 256, 16), (2, 1, 1)),
10680:       (ConvKind.Fprop, 2, bf16bf16_bf16f32_f32, (256, 128,  8), (2, 1, 1)),
10681:       (ConvKind.Fprop, 2, bf16bf16_bf16f32_f32, (256, 128, 16), (2, 1, 1)),
10682:       #
10683:       # f16f16_f16f16_f16
10684:       #
10685:       # cluster shape (1, 1, 1)
10686:       #
10687:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, ( 64,  64,  8), (1, 1, 1)),
10688:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, ( 64,  64, 16), (1, 1, 1)),
10689:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, ( 64, 128,  8), (1, 1, 1)),
10690:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, ( 64, 128, 16), (1, 1, 1)),
10691:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, ( 64, 256,  8), (1, 1, 1)),
10692:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, ( 64, 256, 16), (1, 1, 1)),
10693:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, (128, 128,  8), (1, 1, 1)),
10694:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, (128, 128, 16), (1, 1, 1)),
10695:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, (128, 256,  8), (1, 1, 1)),
10696:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, (128, 256, 16), (1, 1, 1)),
10697:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, (256,  64,  8), (1, 1, 1)),
10698:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, (256,  64, 16), (1, 1, 1)),
10699:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, (256, 128,  8), (1, 1, 1)),
10700:       (ConvKind.Fprop, 2,    f16f16_f16f16_f16, (256, 128, 16), (1, 1, 1)),
10701:       #
10702:       # f16f16_f16f32_f32
10703:       #
10704:       # cluster shape (2, 1, 1)
10705:       #
10706:       (ConvKind.Fprop, 2,    f16f16_f16f32_f32, (128, 192,  8), (2, 1, 1)),
10707:       (ConvKind.Fprop, 2,    f16f16_f16f32_f32, (128, 192, 16), (2, 1, 1)),
10708:       (ConvKind.Fprop, 2,    f16f16_f16f32_f32, (128, 256,  8), (2, 1, 1)),
10709:       (ConvKind.Fprop, 2,    f16f16_f16f32_f32, (128, 256, 16), (2, 1, 1)),
10710:       (ConvKind.Fprop, 2,    f16f16_f16f32_f32, (256,  96,  8), (2, 1, 1)),
10711:       (ConvKind.Fprop, 2,    f16f16_f16f32_f32, (256,  96, 16), (2, 1, 1)),
10712:       (ConvKind.Fprop, 2,    f16f16_f16f32_f32, (256, 128,  8), (2, 1, 1)),
10713:       (ConvKind.Fprop, 2,    f16f16_f16f32_f32, (256, 128, 16), (2, 1, 1)),
10714:       #
10715:       # f32f32_tf32f32_f32
10716:       #
10717:       # cluster shape (2, 1, 1)
10718:       #
10719:       (ConvKind.Fprop, 2,   f32f32_tf32f32_f32, (128, 192,  8), (2, 1, 1)),
10720:       (ConvKind.Fprop, 2,   f32f32_tf32f32_f32, (128, 256,  8), (2, 1, 1)),
10721:       (ConvKind.Fprop, 2,   f32f32_tf32f32_f32, (256, 128,  8), (2, 1, 1)),
10722:       (ConvKind.Fprop, 2,   f32f32_tf32f32_f32, (256,  96,  8), (2, 1, 1)),
10723:       #
10724:       # i8i8_i8i32_f32
10725:       #
10726:       # cluster shape (2, 1, 1)
10727:       #
10728:       (ConvKind.Fprop, 2,       i8i8_i8i32_f32, (128, 256, 16), (2, 1, 1)),
10729:       (ConvKind.Fprop, 2,       i8i8_i8i32_f32, (128, 256, 32), (2, 1, 1)),
10730:       (ConvKind.Fprop, 2,       i8i8_i8i32_f32, (256, 128, 16), (2, 1, 1)),
10731:       (ConvKind.Fprop, 2,       i8i8_i8i32_f32, (256, 128, 32), (2, 1, 1)),
10732:       #
10733:       # Dgrad
10734:       #
10735:       # bf16bf16_bf16f32_f32
10736:       #
10737:       # cluster shape (2, 1, 1)
10738:       #
10739:       (ConvKind.Dgrad, 2, bf16bf16_bf16f32_f32, (128, 256,  8), (2, 1, 1)),
10740:       (ConvKind.Dgrad, 2, bf16bf16_bf16f32_f32, (128, 256, 16), (2, 1, 1)),
10741:       (ConvKind.Dgrad, 2, bf16bf16_bf16f32_f32, (256, 128,  8), (2, 1, 1)),
10742:       (ConvKind.Dgrad, 2, bf16bf16_bf16f32_f32, (256, 128, 16), (2, 1, 1)),
10743:       #
10744:       # f16f16_f16f16_f16
10745:       #
10746:       # cluster shape (1, 1, 1)
10747:       #
10748:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, ( 64,  64,  8), (1, 1, 1)),
10749:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, ( 64,  64, 16), (1, 1, 1)),
10750:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, ( 64, 128,  8), (1, 1, 1)),
10751:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, ( 64, 128, 16), (1, 1, 1)),
10752:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, ( 64, 256,  8), (1, 1, 1)),
10753:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, ( 64, 256, 16), (1, 1, 1)),
10754:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, (128, 128,  8), (1, 1, 1)),
10755:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, (128, 128, 16), (1, 1, 1)),
10756:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, (128, 256,  8), (1, 1, 1)),
10757:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, (128, 256, 16), (1, 1, 1)),
10758:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, (256,  64,  8), (1, 1, 1)),
10759:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, (256,  64, 16), (1, 1, 1)),
10760:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, (256, 128,  8), (1, 1, 1)),
10761:       (ConvKind.Dgrad, 2,    f16f16_f16f16_f16, (256, 128, 16), (1, 1, 1)),
10762:       #
10763:       # f16f16_f16f32_f32
10764:       #
10765:       # cluster shape (2, 1, 1)
10766:       #
10767:       (ConvKind.Dgrad, 2,    f16f16_f16f32_f32, (128, 256,  8), (2, 1, 1)),
10768:       (ConvKind.Dgrad, 2,    f16f16_f16f32_f32, (128, 256, 16), (2, 1, 1)),
10769:       (ConvKind.Dgrad, 2,    f16f16_f16f32_f32, (256, 128,  8), (2, 1, 1)),
10770:       (ConvKind.Dgrad, 2,    f16f16_f16f32_f32, (256, 128, 16), (2, 1, 1)),
10771:     ),
10772:   )
10773: 
10774:   # SM >= 90 kernels don't actually use warp_count, but the
10775:   # TileDescription class needs it.  The 4 in the default
10776:   # warp_count has nothing to do with num_mma_per_tile.
10777:   warp_count = [4, 1, 1]
10778: 
10779:   stages = 0 # zero means "deduce the number of stages automatically"
10780: 
10781:   mainloop_schedule = KernelScheduleType.ImplicitTmaWarpSpecializedSm90
10782:   epilogue_schedule = EpilogueScheduleType.TmaWarpSpecialized
10783:   schedule_pairs = (
10784:     (mainloop_schedule, epilogue_schedule),
10785:   )
10786:   tile_schedulers = (
10787:     TileSchedulerType.Default, # -> void
10788:   )
10789: 
10790:   def make_math_instruction(data_types: Dict[str, DataType],
10791:                             mma_shape: Tuple[int, int, int]) -> MathInstruction:
10792:     default_opcode = OpcodeClass.TensorOp
10793:     default_math_op = MathOperation.multiply_add
10794:     return MathInstruction(
10795:       mma_shape,
10796:       data_types['a_type'], data_types['b_type'], data_types['c_type'],
10797:       default_opcode,
10798:       default_math_op
10799:     )
10800: 
10801:   for (conv_kind, spatial_dim, data_types, mma_shape, cluster_shape) in combinations_of_parameters:
10802:     math_inst = make_math_instruction(data_types, mma_shape)
10803:     tile_shape = (mma_shape[0], mma_shape[1], num_mma_per_tile * mma_shape[2])
10804:     tile_description = TileDescription(tile_shape, stages, warp_count, math_inst,
10805:       minimum_compute_capability, maximum_compute_capability, cluster_shape)
10806:     assert(isinstance(spatial_dim, int))
10807:     dims_and_alignments = (
10808:       (
10809:         (spatial_dim, data_types['alignment_A']),
10810:         (spatial_dim, data_types['alignment_B']),
10811:         (spatial_dim, data_types['alignment_C']),
10812:       ),
10813:     )
10814:     CreateConvOperator3x(manifest,
10815:                          dims_and_alignments = dims_and_alignments,
10816:                          tile_descriptions = [tile_description],
10817:                          data_types = data_types,
10818:                          schedule_pairs = schedule_pairs,
10819:                          tile_schedulers = tile_schedulers,
10820:                          conv_kind = conv_kind,
10821:                          log_indent_level = log_indent_level)
```
**EN:** Defines `GenerateSM90_Conv3x()`. The docstring says: "Generate CUTLASS 3 convolution kernel(s) for SM90."; this block implements that behavior. Key helper calls include log_debug_line, chain, CudaToolkitVersionSatisfies, product, MathInstruction, make_math_instruction.

**CN:** 定义 `GenerateSM90_Conv3x()`。文档字符串说明其用途为：“Generate CUTLASS 3 convolution kernel(s) for SM90.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 log_debug_line, chain, CudaToolkitVersionSatisfies, product, MathInstruction, make_math_instruction。

### L10823-L10853 — Function `GenerateSM90`

```python
10823: def GenerateSM90(manifest, cuda_version):
10824:   GenerateSM90_TensorOp_16b_WGMMA_gemm(manifest, cuda_version)
10825:   GenerateSM90_TensorOp_16b_WGMMA_alignx_gemm(manifest, cuda_version)
10826:   GenerateSM90_TensorOp_tf32_WGMMA_gemm(manifest, cuda_version)
10827:   GenerateSM90_TensorOp_tf32_WGMMA_alignx_gemm(manifest, cuda_version)
10828:   GenerateSM90_TensorOp_int8_WGMMA_gemm(manifest, cuda_version)
10829:   GenerateSM90_TensorOp_int8_WGMMA_alignx_gemm(manifest, cuda_version)
10830:   GenerateSM90_TensorOp_fp8_WGMMA_gemm(manifest, cuda_version)
10831:   GenerateSM90_TensorOp_fp8_WGMMA_alignx_gemm(manifest, cuda_version)
10832:   GenerateSM90_TensorOp_mixed_dtype_WGMMA_gemm(manifest, cuda_version)
10833:   GenerateSM90_TensorOp_1684(manifest, cuda_version)
10834:   GenerateSM90_TensorOp_16b_WGMMA_gemm(manifest, cuda_version, gemm_kind=GemmKind.GroupedUniversal3x)
10835:   GenerateSM90_TensorOp_fp8_WGMMA_gemm(manifest, cuda_version, gemm_kind=GemmKind.GroupedUniversal3x)
10836:   GenerateSM90_TensorOp_1684_complex(manifest, cuda_version)
10837:   GenerateSM90_TensorOp_1684_complex_gaussian(manifest, cuda_version)
10838:   GenerateSM90_TensorOp_1684_rank_k(manifest, cuda_version)
10839:   GenerateSM90_TensorOp_1684_rank_k_complex(manifest, cuda_version)
10840:   GenerateSM90_TensorOp_1684_rank_k_complex_gaussian(manifest, cuda_version)
10841:   GenerateSM90_TensorOp_1684_trmm(manifest, cuda_version)
10842:   GenerateSM90_TensorOp_1684_trmm_complex(manifest, cuda_version)
10843:   GenerateSM90_TensorOp_1684_trmm_complex_gaussian(manifest, cuda_version)
10844:   GenerateSM90_TensorOp_1684_symm(manifest, cuda_version)
10845:   GenerateSM90_TensorOp_1684_symm_complex(manifest, cuda_version)
10846:   GenerateSM90_TensorOp_1684_symm_complex_gaussian(manifest, cuda_version)
10847:   GenerateSM90_Conv3x(manifest, cuda_version)
10848:   GenerateSM90_SparseTensorOp_16b_WGMMA_gemm(manifest, cuda_version)
10849:   GenerateSM90_SparseTensorOp_tf32_WGMMA_gemm(manifest, cuda_version)
10850:   GenerateSM90_SparseTensorOp_int8_WGMMA_gemm(manifest, cuda_version)
10851:   GenerateSM90_SparseTensorOp_fp8_WGMMA_gemm(manifest, cuda_version)
10852:   GenerateSM90_TensorOp_fp8_WGMMA_gemm_with_blockwise(manifest, cuda_version)
10853:   GenerateSM90_TensorOp_fp8_WGMMA_gemm_with_blockwise(manifest, cuda_version, gemm_kind=GemmKind.GroupedBlockwiseUniversal3x)
```
**EN:** Defines `GenerateSM90()`, which generates sm90. Key helper calls include GenerateSM90_TensorOp_16b_WGMMA_gemm, GenerateSM90_TensorOp_16b_WGMMA_alignx_gemm, GenerateSM90_TensorOp_tf32_WGMMA_gemm, GenerateSM90_TensorOp_tf32_WGMMA_alignx_gemm, GenerateSM90_TensorOp_int8_WGMMA_gemm, GenerateSM90_TensorOp_int8_WGMMA_alignx_gemm.

**CN:** 定义 `GenerateSM90()`，用于生成 sm90。 其中会调用的重要辅助函数包括 GenerateSM90_TensorOp_16b_WGMMA_gemm, GenerateSM90_TensorOp_16b_WGMMA_alignx_gemm, GenerateSM90_TensorOp_tf32_WGMMA_gemm, GenerateSM90_TensorOp_tf32_WGMMA_alignx_gemm, GenerateSM90_TensorOp_int8_WGMMA_gemm, GenerateSM90_TensorOp_int8_WGMMA_alignx_gemm。

### L10854-L10856 — Comments

```python
10854: 
10855: ###################################################################################################
10856: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L10857-L10865 — Function `GeneratePVC`

```python
10857: def GeneratePVC(manifest, cuda_version):
10858:     """
10859:     Generate CUTLASS kernels for PVC (Ponte Vecchio) architecture.
10860:     
10861:     PVC is Intel's Xe-HPC GPU architecture with compute capability 12.
10862:     
10863:     This is a legacy wrapper that calls GenerateIntelXe with arch=INTEL_XE12.
10864:     """
10865:     GenerateIntelXe(manifest, cuda_version, arch=INTEL_XE12)
```
**EN:** Defines `GeneratePVC()`. The docstring says: "Generate CUTLASS kernels for PVC (Ponte Vecchio) architecture."; this block implements that behavior. Key helper calls include GenerateIntelXe.

**CN:** 定义 `GeneratePVC()`。文档字符串说明其用途为：“Generate CUTLASS kernels for PVC (Ponte Vecchio) architecture.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 GenerateIntelXe。

### L10866-L10867 — Comments

```python
10866: 
10867: ###################################################################################################
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L10868-L10980 — Function `GenerateXe_TensorOp_16b_DPAS_gemm`

```python
10868: def GenerateXe_TensorOp_16b_DPAS_gemm(manifest, cuda_version, min_cc=20):
10869:     """Generate FP16/BF16 GEMM kernels for Intel Xe architecture using DPAS.
10870:     
10871:     :param min_cc: Architecture number (12 for PVC, 20 for BMG)
10872:     """
10873:     layout_list = [
10874:         [[LayoutType.RowMajor, 8], [LayoutType.RowMajor, 8], [LayoutType.RowMajor, 8]],
10875:         [[LayoutType.RowMajor, 8], [LayoutType.ColumnMajor, 8], [LayoutType.RowMajor, 8]],
10876:         [[LayoutType.ColumnMajor, 8], [LayoutType.RowMajor, 8], [LayoutType.RowMajor, 8]],
10877:         [[LayoutType.ColumnMajor, 8], [LayoutType.ColumnMajor, 8], [LayoutType.RowMajor, 8]],
10878:     ]
10879: 
10880:     math_instructions = [
10881:         MathInstruction(
10882:             [8, 16, 16],
10883:             DataType.f16, DataType.f16, DataType.f32,
10884:             OpcodeClass.TensorOp,
10885:             MathOperation.multiply_add),
10886:         MathInstruction(
10887:             [8, 16, 16],
10888:             DataType.f16, DataType.f16, DataType.f16,
10889:             OpcodeClass.TensorOp,
10890:             MathOperation.multiply_add),
10891:         MathInstruction(
10892:             [8, 16, 16],
10893:             DataType.bf16, DataType.bf16, DataType.f32,
10894:             OpcodeClass.TensorOp,
10895:             MathOperation.multiply_add),
10896:         MathInstruction(
10897:             [8, 16, 16],
10898:             DataType.bf16, DataType.bf16, DataType.bf16,
10899:             OpcodeClass.TensorOp,
10900:             MathOperation.multiply_add)
10901:     ]
10902: 
10903:     default_tiles_wg_sg = [
10904:         # === K=64 ===
10905:         ([256, 128, 64],[8,4,1]),
10906:         ([128, 256, 64],[4,8,1]),
10907:         ([128, 128, 64],[4,4,1]),
10908:         ([256, 64, 64],[8,2,1]),
10909:         ([64, 256, 64],[2,8,1]),
10910:         ([32, 32, 64],[4,2,1]),
10911:         ([16, 64, 64],[2,4,1]),
10912:         ([64, 16, 64],[8,1,1]),
10913: 
10914:         # === K=32 ===
10915:         ([512, 128, 32],[16,2,1]),
10916:         ([256, 256, 32],[8,4,1]),
10917:         ([256, 128, 32],[8,4,1]),
10918:         ([256, 64, 32],[8,2,1]),
10919:         ([128, 256, 32],[4,8,1]),
10920:         ([128, 64, 32],[4,2,1]),
10921:         ([32, 32, 32],[4,2,1]),
10922:         ([16, 64, 32],[2,4,1]),
10923:         ([64, 128, 32], [2,4,1]),
10924:         ([128, 128, 32], [4,4,1]),
10925: 
10926:         # === K=16 ===
10927:         ([256, 256, 16],[8,4,1]),
10928:     ]
10929: 
10930:     max_cc = min_cc
10931: 
10932:     # Expecting JSON of format i.e list of dictionaries [{"wg": [256, 256, 32], "sg": [8,4,1]}, ...]
10933:     custom_tile_shapes = []
10934:     if os.getenv("SYCL_TLA_ADDITIONAL_TILE_SHAPES"):
10935:       custom_json = os.getenv("SYCL_TLA_ADDITIONAL_TILE_SHAPES")
10936:       with open(custom_json, "r") as f:
10937:           try:
10938:             custom_tile_shapes = json.load(f)
10939:           except json.JSONDecodeError:
10940:             raise ValueError(f"Error decoding JSON : {custom_json}")
10941:     for tile in custom_tile_shapes:
10942:       default_tiles_wg_sg.append((tile["wg"],tile["sg"]))
10943: 
10944:     for math_inst in math_instructions:
10945:         tile_descriptions=[]
10946:         for wg_tile,sg_tile in default_tiles_wg_sg:
10947:           tile_descriptions.append(TileDescription(wg_tile,
10948:                   0, sg_tile, math_inst, min_cc, max_cc, [1, 1, 1]))
10949: 
10950:         # Generate kernels for different output (D) types
10951:         # Default: accumulator type (FP32 for mixed precision, same as input for native precision)
10952:         # For mixed precision (a_type != accumulator): also generate output in input precision
10953:         valid_d_types = [math_inst.element_accumulator]
10954:         if math_inst.element_a != math_inst.element_accumulator:
10955:             valid_d_types.append(math_inst.element_a)
10956:        
10957:         for d_type in valid_d_types:
10958:             # Generate operations both with and without bias (ElementC)
10959:             for c_type in [math_inst.element_accumulator, DataType.void]: 
10960:                 data_type = {
10961:                     "a_type": math_inst.element_a,
10962:                     "b_type": math_inst.element_b,
10963:                     "c_type": c_type,
10964:                     "d_type": d_type,
10965:                     "acc_type": math_inst.element_accumulator,
10966:                     "epi_type": math_inst.element_accumulator
10967:                 }
10968:                 
10969:                 schedules = [[KernelScheduleType.ScheduleAuto, EpilogueScheduleType.ScheduleAuto]]
10970: 
10971:                 CreateGemmUniversal3xOperator(manifest, layout_list, tile_descriptions, data_type, schedules, tile_schedulers=[TileSchedulerType.Persistent])
10972:                 # StreamK uses atomic reduction on the accumulator type in fixup()
10973:                 # via BlockStripedReduce::reduce() -> atomic_add<ElementAccumulator>.
10974:                 # SYCL atomic_ref only supports: int, unsigned int, long, unsigned long,
10975:                 # long long, unsigned long long, half, float, double and pointer types.
10976:                 # bfloat16_t is NOT supported, so skip StreamK when accumulator is bf16.
10977:                 sycl_streamk_safe_types = [DataType.f16, DataType.f32, DataType.f64]
10978:                 if data_type["acc_type"] in sycl_streamk_safe_types:
10979:                     schedules_cooperative = [[KernelScheduleType.XeCooperative, EpilogueScheduleType.ScheduleAuto]]
10980:                     CreateGemmUniversal3xOperator(manifest, layout_list, tile_descriptions, data_type, schedules_cooperative, tile_schedulers=[TileSchedulerType.StreamK])
```
**EN:** Defines `GenerateXe_TensorOp_16b_DPAS_gemm()`. The docstring says: "Generate FP16/BF16 GEMM kernels for Intel Xe architecture using DPAS."; this block implements that behavior. Key helper calls include getenv, MathInstruction, append, open, load, TileDescription.

**CN:** 定义 `GenerateXe_TensorOp_16b_DPAS_gemm()`。文档字符串说明其用途为：“Generate FP16/BF16 GEMM kernels for Intel Xe architecture using DPAS.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 getenv, MathInstruction, append, open, load, TileDescription。

### L10982-L11057 — Function `GenerateXe_TensorOp_fp8_DPAS_gemm`

```python
10982: def GenerateXe_TensorOp_fp8_DPAS_gemm(manifest, cuda_version, min_cc=20):
10983:     """Generate FP8 (E4M3/E5M2) GEMM kernels for Intel Xe architecture using DPAS.
10984:     
10985:     Supported combinations for regular GEMM:
10986:     - [e4m3, e4m3, fp32]: E4M3 x E4M3 -> FP32 (homogeneous)
10987:     - [e5m2, e5m2, fp32]: E5M2 x E5M2 -> FP32 (homogeneous)
10988:     
10989:     Note: Mixed precision (FP16/BF16 x FP8) requires grouped GEMM infrastructure
10990:     and is NOT supported for regular library generation.
10991:     
10992:     :param min_cc: Architecture number (12 for PVC, 20 for BMG)
10993:     """
10994:     layout_list = [
10995:         [[LayoutType.RowMajor, 16], [LayoutType.RowMajor, 16], [LayoutType.RowMajor, 8]],
10996:         [[LayoutType.RowMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.RowMajor, 8]],
10997:         [[LayoutType.ColumnMajor, 16], [LayoutType.RowMajor, 16], [LayoutType.RowMajor, 8]],
10998:         [[LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.RowMajor, 8]],
10999:     ]
11000: 
11001:     # FP8 math instructions for Intel Xe
11002:     # Only homogeneous types (same A and B type) for regular GEMM
11003:     math_instructions = [
11004:         # Homogeneous FP8 (same type for A and B) - SUPPORTED
11005:         MathInstruction(
11006:             [8, 16, 32],
11007:             DataType.e4m3, DataType.e4m3, DataType.f32,  # E4M3 x E4M3 -> FP32
11008:             OpcodeClass.TensorOp,
11009:             MathOperation.multiply_add),
11010:         MathInstruction(
11011:             [8, 16, 32],
11012:             DataType.e5m2, DataType.e5m2, DataType.f32,  # E5M2 x E5M2 -> FP32
11013:             OpcodeClass.TensorOp,
11014:             MathOperation.multiply_add),
11015:         
11016:         # DISABLED: Mixed precision FP16/BF16 x FP8 requires grouped GEMM
11017:         # These would need MainloopIntelXeXMX16GroupMixedPrecision which is only
11018:         # activated when IsGroup=true (KernelXePtrArrayCooperative schedule).
11019:         # Regular library GEMMs use MainloopIntelXeXMX16 which requires ElementA == ElementB.
11020:         #
11021:         # MathInstruction([8, 16, 32], DataType.f16, DataType.e5m2, DataType.f32, ...),
11022:         # MathInstruction([8, 16, 32], DataType.f16, DataType.e4m3, DataType.f32, ...),
11023:         # MathInstruction([8, 16, 32], DataType.bf16, DataType.e5m2, DataType.f32, ...),
11024:         # MathInstruction([8, 16, 32], DataType.bf16, DataType.e4m3, DataType.f32, ...),
11025:     ]
11026: 
11027:     max_cc = min_cc
11028: 
11029:     for math_inst in math_instructions:
11030:         tile_descriptions = [
11031:             TileDescription([256, 256, 64],
11032:                 0, [8, 4, 1], math_inst, min_cc, max_cc, [1, 1, 1]),
11033:             TileDescription([128, 256, 64],
11034:                 0, [4, 8, 1], math_inst, min_cc, max_cc, [1, 1, 1]),
11035:             TileDescription([256, 128, 64],
11036:                 0, [8, 4, 1], math_inst, min_cc, max_cc, [1, 1, 1]),
11037:             TileDescription([128, 128, 64],
11038:                 0, [4, 4, 1], math_inst, min_cc, max_cc, [1, 1, 1]),
11039:         ]
11040: 
11041:         # Generate kernels for different output (D) types
11042:         # Valid D types for FP8: fp32 (accumulator), bf16, fp16, e4m3, e5m2
11043:         valid_d_types = [DataType.f32, DataType.bf16, DataType.f16, DataType.e4m3, DataType.e5m2]
11044:         
11045:         for d_type in valid_d_types:
11046:             data_type = {
11047:                 "a_type": math_inst.element_a,
11048:                 "b_type": math_inst.element_b,
11049:                 "c_type": math_inst.element_accumulator,
11050:                 "d_type": d_type,
11051:                 "acc_type": math_inst.element_accumulator,
11052:                 "epi_type": math_inst.element_accumulator
11053:             }
11054: 
11055:             schedules = [[KernelScheduleType.ScheduleAuto, EpilogueScheduleType.ScheduleAuto]]
11056: 
11057:             CreateGemmUniversal3xOperator(manifest, layout_list, tile_descriptions, data_type, schedules, tile_schedulers=[TileSchedulerType.Persistent])
```
**EN:** Defines `GenerateXe_TensorOp_fp8_DPAS_gemm()`. The docstring says: "Generate FP8 (E4M3/E5M2) GEMM kernels for Intel Xe architecture using DPAS."; this block implements that behavior. Key helper calls include MathInstruction, TileDescription, CreateGemmUniversal3xOperator.

**CN:** 定义 `GenerateXe_TensorOp_fp8_DPAS_gemm()`。文档字符串说明其用途为：“Generate FP8 (E4M3/E5M2) GEMM kernels for Intel Xe architecture using DPAS.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 MathInstruction, TileDescription, CreateGemmUniversal3xOperator。

### L11059-L11113 — Function `GenerateXe_TensorOp_int8_DPAS_gemm`

```python
11059: def GenerateXe_TensorOp_int8_DPAS_gemm(manifest, cuda_version, min_cc=20):
11060:     """Generate INT8 GEMM kernels for Intel Xe architecture using DPAS.
11061:     
11062:     Supported: [int8, int8, int32] -> INT32 accumulator (hardware requirement)
11063:     
11064:     :param min_cc: Architecture number (12 for PVC, 20 for BMG)
11065:     """
11066:     layout_list = [
11067:         [[LayoutType.RowMajor, 16], [LayoutType.RowMajor, 16], [LayoutType.RowMajor, 4]],
11068:         [[LayoutType.RowMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.RowMajor, 4]],
11069:         [[LayoutType.ColumnMajor, 16], [LayoutType.RowMajor, 16], [LayoutType.RowMajor, 4]],
11070:         [[LayoutType.ColumnMajor, 16], [LayoutType.ColumnMajor, 16], [LayoutType.RowMajor, 4]],
11071:     ]
11072: 
11073:     # INT8 x INT8 -> INT32 (hardware requirement for Intel Xe)
11074:     math_instructions = [
11075:         MathInstruction(
11076:             [8, 16, 32],
11077:             DataType.s8, DataType.s8, DataType.s32,  # Changed from f32 to s32
11078:             OpcodeClass.TensorOp,
11079:             MathOperation.multiply_add),
11080:     ]
11081: 
11082:     max_cc = min_cc
11083: 
11084:     for math_inst in math_instructions:
11085:         tile_descriptions = [
11086:             TileDescription([256, 256, 64],
11087:                 0, [8, 4, 1], math_inst, min_cc, max_cc, [1, 1, 1]),
11088:             TileDescription([128, 256, 64],
11089:                 0, [4, 8, 1], math_inst, min_cc, max_cc, [1, 1, 1]),
11090:             TileDescription([256, 128, 64],
11091:                 0, [8, 4, 1], math_inst, min_cc, max_cc, [1, 1, 1]),
11092:             TileDescription([128, 128, 64],
11093:                 0, [4, 4, 1], math_inst, min_cc, max_cc, [1, 1, 1]),
11094:         ]
11095: 
11096:         # Generate kernels for different output (D) types
11097:         # Default: accumulator type (INT32)
11098:         # Also generate output in input precision (INT8) for quantized workflows
11099:         valid_d_types = [math_inst.element_accumulator, math_inst.element_a]
11100:         
11101:         for d_type in valid_d_types:
11102:             data_type = {
11103:                 "a_type": math_inst.element_a,
11104:                 "b_type": math_inst.element_b,
11105:                 "c_type": math_inst.element_accumulator,
11106:                 "d_type": d_type,
11107:                 "acc_type": math_inst.element_accumulator,
11108:                 "epi_type": math_inst.element_accumulator
11109:             }
11110: 
11111:             schedules = [[KernelScheduleType.ScheduleAuto, EpilogueScheduleType.ScheduleAuto]]
11112: 
11113:             CreateGemmUniversal3xOperator(manifest, layout_list, tile_descriptions, data_type, schedules, tile_schedulers=[TileSchedulerType.Persistent])
```
**EN:** Defines `GenerateXe_TensorOp_int8_DPAS_gemm()`. The docstring says: "Generate INT8 GEMM kernels for Intel Xe architecture using DPAS."; this block implements that behavior. Key helper calls include MathInstruction, TileDescription, CreateGemmUniversal3xOperator.

**CN:** 定义 `GenerateXe_TensorOp_int8_DPAS_gemm()`。文档字符串说明其用途为：“Generate INT8 GEMM kernels for Intel Xe architecture using DPAS.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 MathInstruction, TileDescription, CreateGemmUniversal3xOperator。

### L11116-L11162 — Function `GenerateXe_TensorOp_mixed_dtype_DPAS_gemm`

```python
11116: def GenerateXe_TensorOp_mixed_dtype_DPAS_gemm(manifest, cuda_version, min_cc=20):
11117:     """Generate mixed-precision GEMM kernels for Intel Xe architecture using DPAS.
11118:     
11119:     Supported: [fp16, int4, fp32] -> FP16 x INT4 with FP32 accumulator
11120:     
11121:     :param min_cc: Architecture number (12 for PVC, 20 for BMG)
11122:     """
11123:     layout_list = [
11124:         [[LayoutType.RowMajor, 8], [LayoutType.RowMajor, 32], [LayoutType.RowMajor, 8]],
11125:         [[LayoutType.RowMajor, 8], [LayoutType.ColumnMajor, 32], [LayoutType.RowMajor, 8]],
11126:         [[LayoutType.ColumnMajor, 8], [LayoutType.RowMajor, 32], [LayoutType.RowMajor, 8]],
11127:         [[LayoutType.ColumnMajor, 8], [LayoutType.ColumnMajor, 32], [LayoutType.RowMajor, 8]],
11128:     ]
11129: 
11130:     # Mixed precision: FP16 x INT4 -> FP32 (hardware requirement for Intel Xe)
11131:     math_instructions = [
11132:         MathInstruction(
11133:             [8, 16, 32],
11134:             DataType.f16, DataType.s4, DataType.f32,  # Changed from [s8, f16, f32] to [f16, s4, f32]
11135:             OpcodeClass.TensorOp,
11136:             MathOperation.multiply_add),
11137:     ]
11138: 
11139:     max_cc = min_cc
11140: 
11141:     for math_inst in math_instructions:
11142:         tile_descriptions = [
11143:             TileDescription([256, 256, 64],
11144:                 0, [8, 4, 1], math_inst, min_cc, max_cc, [1, 1, 1]),
11145:             TileDescription([128, 256, 64],
11146:                 0, [4, 8, 1], math_inst, min_cc, max_cc, [1, 1, 1]),
11147:             TileDescription([256, 128, 64],
11148:                 0, [8, 4, 1], math_inst, min_cc, max_cc, [1, 1, 1]),
11149:         ]
11150: 
11151:         data_type = {
11152:             "a_type": math_inst.element_a,
11153:             "b_type": math_inst.element_b,
11154:             "c_type": math_inst.element_accumulator,
11155:             "d_type": math_inst.element_accumulator,
11156:             "acc_type": math_inst.element_accumulator,
11157:             "epi_type": math_inst.element_accumulator
11158:         }
11159: 
11160:         schedules = [[KernelScheduleType.ScheduleAuto, EpilogueScheduleType.ScheduleAuto]]
11161: 
11162:         CreateGemmUniversal3xOperator(manifest, layout_list, tile_descriptions, data_type, schedules, tile_schedulers=[TileSchedulerType.Persistent])
```
**EN:** Defines `GenerateXe_TensorOp_mixed_dtype_DPAS_gemm()`. The docstring says: "Generate mixed-precision GEMM kernels for Intel Xe architecture using DPAS."; this block implements that behavior. Key helper calls include MathInstruction, CreateGemmUniversal3xOperator, TileDescription.

**CN:** 定义 `GenerateXe_TensorOp_mixed_dtype_DPAS_gemm()`。文档字符串说明其用途为：“Generate mixed-precision GEMM kernels for Intel Xe architecture using DPAS.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 MathInstruction, CreateGemmUniversal3xOperator, TileDescription。

### L11165-L11174 — Function `GenerateBMG`

```python
11165: def GenerateBMG(manifest, cuda_version):
11166:     """
11167:     Generate CUTLASS kernels for BMG (Battlemage/Xe2) architecture.
11168:     
11169:     BMG is Intel's Xe2 GPU architecture with compute capability 20.
11170:     Supports DPAS operations with FP16, BF16, FP8, and INT8 data types.
11171:     
11172:     This is a legacy wrapper that calls GenerateIntelXe with arch=INTEL_XE20.
11173:     """
11174:     GenerateIntelXe(manifest, cuda_version, arch=INTEL_XE20)
```
**EN:** Defines `GenerateBMG()`. The docstring says: "Generate CUTLASS kernels for BMG (Battlemage/Xe2) architecture."; this block implements that behavior. Key helper calls include GenerateIntelXe.

**CN:** 定义 `GenerateBMG()`。文档字符串说明其用途为：“Generate CUTLASS kernels for BMG (Battlemage/Xe2) architecture.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 GenerateIntelXe。

### L11176-L11198 — Function `GenerateIntelXe`

```python
11176: def GenerateIntelXe(manifest, cuda_version, arch):
11177:     """
11178:     Unified generator for Intel Xe GPU architectures.
11179:     
11180:     Supports both PVC (arch 12) and BMG (arch 20) with the same generation code.
11181:     The operations are identical, only the architecture number differs.
11182:     
11183:     Supported data types:
11184:     - FP16/BF16: [fp16/bf16, fp16/bf16, fp32]
11185:     - INT8: [int8, int8, int32]
11186:     - FP8: [fp8, fp8, fp32] (E4M3 or E5M2, same types only)
11187:     - Mixed: [fp16, int4, fp32]
11188:     
11189:     :param manifest: Manifest object to add operations to
11190:     :param cuda_version: CUDA version string (used for compatibility)
11191:     :param arch: Architecture number (12 for PVC, 20 for BMG)
11192:     """
11193:     if arch not in [INTEL_XE12, INTEL_XE20]:
11194:         raise ValueError(f"Unsupported Intel Xe architecture: {arch}. Supported: {INTEL_XE12} (PVC), {INTEL_XE20} (BMG)")
11195:     
11196:     # All Intel Xe architectures use the same generation functions
11197:     # Only the min_cc (architecture number) differs
11198:     GenerateXe_TensorOp_16b_DPAS_gemm(manifest, cuda_version, min_cc=arch)
```
**EN:** Defines `GenerateIntelXe()`. The docstring says: "Unified generator for Intel Xe GPU architectures."; this block implements that behavior. Key helper calls include GenerateXe_TensorOp_16b_DPAS_gemm, ValueError.

**CN:** 定义 `GenerateIntelXe()`。文档字符串说明其用途为：“Unified generator for Intel Xe GPU architectures.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 GenerateXe_TensorOp_16b_DPAS_gemm, ValueError。

### L11199-L11207 — Comments

```python
11199:     #DISABLED: FP8 GEMMs are not yet ready. Will be enabled once the tests are ready
11200:     #GenerateXe_TensorOp_fp8_DPAS_gemm(manifest, cuda_version, min_cc=arch)
11201:     #GenerateXe_TensorOp_int8_DPAS_gemm(manifest, cuda_version, min_cc=arch)
11202:     # DISABLED: Mixed precision (FP16 x INT4) requires grouped GEMM infrastructure
11203:     # Regular library generation uses MainloopIntelXeXMX16 which requires ElementA == ElementB
11204:     # GenerateXe_TensorOp_mixed_dtype_DPAS_gemm(manifest, cuda_version, min_cc=arch)
11205: 
11206: ###################################################################################################
11207: 
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L11208-L11222 — Function `numeric_log_level`

```python
11208: def numeric_log_level(log_level: str) -> int:
11209:   """
11210:   Converts the string identifier of the log level
11211:   into the numeric identifier used in setting the log level.
11212: 
11213:   :param x: string representation of log level (e.g., 'INFO', 'DEBUG')
11214:   :type x: str
11215: 
11216:   :return: numeric representation of log level
11217:   :rtype: int
11218:   """
11219:   numeric_level = getattr(logging, log_level.upper(), None)
11220:   if not isinstance(numeric_level, int):
11221:     raise ValueError(f'Invalid log level: {log_level}')
11222:   return numeric_level
```
**EN:** Defines `numeric_log_level()`. The docstring says: "Converts the string identifier of the log level into the numeric identifier used in setting the log level."; this block implements that behavior. Key helper calls include getattr, upper, isinstance, ValueError.

**CN:** 定义 `numeric_log_level()`。文档字符串说明其用途为：“Converts the string identifier of the log level into the numeric identifier used in setting the log level.”；这一代码块给出了该行为的具体实现。 其中会调用的重要辅助函数包括 getattr, upper, isinstance, ValueError。

### L11223-L11225 — Comments

```python
11223: 
11224: # This function for defining the ArgumentParser is used to make it easy for the CUTLASS Python interface
11225: # to leverage the functionality in this file without running this script via a shell prompt.
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L11226-L11261 — Function `define_parser`

```python
11226: def define_parser():
11227:   parser = argparse.ArgumentParser(description="Generates device kernel registration code for CUTLASS Kernels")
11228:   parser.add_argument("--operations", default="all", help="Specifies the operation to generate (gemm, all)")
11229:   parser.add_argument("--build-dir", default=".", required=False, help="CUTLASS top-level build directory")
11230:   parser.add_argument("--curr-build-dir", default=".", help="CUTLASS current build directory. cmake files will be emitted in this directory")
11231:   parser.add_argument("--generator-target", default='library', help="Target of CUTLASS Library Generator.")
11232:   parser.add_argument("--architectures", default='53;60;61;70;75;80;90;100', help="Target compute architectures")
11233:   parser.add_argument("--kernels", default='', help='Comma-delimited list to filter kernels by name.  ' +
11234:                       'Specifying this as \"all\" includes ALL the kernels, ' +
11235:                       'while not specifying this includes only the default set of kernels.')
11236:   parser.add_argument("--ignore-kernels", default='', help='Comma-delimited list of kernels ' +
11237:                       'to exclude from build.  For backwards compatibility reasons, ' +
11238:                       'this option only takes effect if --kernels is set to a nonempty value.')
11239:   parser.add_argument("--exclude-kernels", default='', help='Comma-delimited list of kernels ' +
11240:                       'to exclude from build.  In contrast to --ignore-kernels, ' +
11241:                       'this option always takes effect, ' +
11242:                       'whether or not --kernels is set to a nonempty value.  ' +
11243:                       'It also can exclude kernels from the filter file ' +
11244:                       '(see --kernel-filter-file option below).')
11245:   parser.add_argument("--filter-by-cc", default='True', type=str, help='If enabled, kernels whose compute capability range is not satisfied by the build target are excluded.')
11246:   parser.add_argument("--cuda-version", default="11.0.0", help="Semantic version string of CUDA Toolkit")
11247:   parser.add_argument('--kernel-filter-file',   type=str, default=None, required=False, help='Full path of filter file')
11248:   parser.add_argument('--heuristics-problems-file',   type=str, default=None, required=False, help='Full path of heuristics problem size description file, as a json list')
11249:   parser.add_argument('--heuristics-testlist-file',   type=str, default=None, required=False, help='Full path of heuristics testlist CSV file, to be passed to cutlass_profiler')
11250:   parser.add_argument('--heuristics-gpu',   type=str, default=None, required=False, help='GPU to use for evaluating heuristics offline. None or `auto` to autodetect using cuda', choices=['', 'auto', 'H100_SXM', 'H100_PCIE', 'H100_NVL', 'H200_SXM', 'H20_SXM', 'B200', 'GB200_NVL', 'RTX_5080', 'RTX_5090', 'RTX_PRO_6000'])
11251:   parser.add_argument('--heuristics-configs-per-problem',   type=int, default=10, required=False, help='Number of kernel configs to generate for each problem in the problem list')
11252:   parser.add_argument('--heuristics-restrict-kernels', action='store_true', help='Restrict heuristics mode to use only the default set of kernels emitted by generator.py')
11253:   parser.add_argument('--selected-kernel-list',   type=str, default=None, required=False,
11254:                         help='Specify the output log file containing all enabled kernels in this build')
11255:   parser.add_argument("--interface-dir", default=None, required=False, help="Interface header to kernels")
11256:   parser.add_argument("--disable-full-archs-compilation", action="store_true", required=False, help="Disable compilation for every archs in --architectures")
11257:   parser.add_argument("--log-level", default='info', type=numeric_log_level, required=False,
11258:                       help='Logging level to be used by the generator script')
11259:   parser.add_argument('--instantiation-level', type=str, default="", required=False, help="Instantiation level for SM90 kernels. Set to `max` and make sure `--kernels` is not empty to generate all possible configurations.")
11260:   _add_package_disablement_flag(parser)
11261:   return parser
```
**EN:** Defines `define_parser()`, which implements the define parser logic. Key helper calls include ArgumentParser, add_argument, _add_package_disablement_flag.

**CN:** 定义 `define_parser()`，用于实现 define parser 相关逻辑。 其中会调用的重要辅助函数包括 ArgumentParser, add_argument, _add_package_disablement_flag。

### L11264-L11328 — Conditional logic

```python
11264: if __name__ == "__main__":
11265:   parser = define_parser()
11266:   args = parser.parse_args()
11267: 
11268:   # Set the logging level based on the user-provided `--log-level` command-line option
11269:   logging.basicConfig(level=args.log_level)
11270: 
11271:   manifest = Manifest(args)
11272: 
11273:   archs = args.architectures.split(';')
11274: 
11275:   if args.heuristics_problems_file:
11276:     filter_manifest_and_write_heuristics_file(manifest, args)
11277: 
11278:   GenerateSM50(manifest, args.cuda_version)
11279:   GenerateSM60(manifest, args.cuda_version)
11280:   GenerateSM61(manifest, args.cuda_version)
11281:   GenerateSM70(manifest, args.cuda_version)
11282:   GenerateSM75(manifest, args.cuda_version)
11283:   GenerateSM80(manifest, args.cuda_version)
11284:   GenerateSM89(manifest, args.cuda_version)
11285:   GenerateSM90(manifest, args.cuda_version)
11286: 
11287:   blackwell_arch_list = [
11288:     "100a", "100f",
11289:     "101a", "101f",
11290:     "103a", "103f",
11291:     "110a", "110f",
11292:     "120a", "120f",
11293:     "121a", "121f",
11294:   ]
11295:   blackwell_enabled_arch = any(arch in blackwell_arch_list for arch in archs)
11296:   if blackwell_enabled_arch:
11297:     GenerateSM100(manifest, args.cuda_version)
11298:     GenerateSM120(manifest, args.cuda_version)
11299: 
11300:   # Intel Xe GPU architectures - unified handling for PVC and BMG
11301:   # Both architectures share the same generation code, just different arch numbers
11302:   
11303:   # Check for BMG (architecture INTEL_XE20)
11304:   bmg_arch_list = [str(INTEL_XE20), "bmg", "xe2", "intel_gpu_bmg_g21", "intel_gpu_bmg_g31"]
11305:   bmg_enabled_arch = any(arch.lower() in [x.lower() for x in bmg_arch_list] for arch in archs)
11306:   if bmg_enabled_arch:
11307:     GenerateIntelXe(manifest, args.cuda_version, arch=INTEL_XE20)
11308: 
11309:   # Check for PVC (architecture INTEL_XE12)
11310:   pvc_arch_list = [str(INTEL_XE12), "pvc", "intel_gpu_pvc"]
11311:   pvc_enabled_arch = any(arch.lower() in [x.lower() for x in pvc_arch_list] for arch in archs)
11312:   if pvc_enabled_arch:
11313:     GenerateIntelXe(manifest, args.cuda_version, arch=INTEL_XE12)
11314: 
11315:   if 'library' in args.generator_target.split(','):
11316:     manifest.emit(GeneratorTarget.Library)
11317: 
11318:   if 'kernel_testlist_l0' in args.generator_target.split(','):
11319:     emit_gemm_kernel_testlist(manifest, args.curr_build_dir, args.architectures, "functional_L0")
11320: 
11321:   if 'kernel_testlist_l1' in args.generator_target.split(','):
11322:     emit_gemm_kernel_testlist(manifest, args.curr_build_dir, args.architectures, "functional_L1")
11323:   
11324:   if args.selected_kernel_list is not None:
11325:     if len(manifest.selected_kernels) > 0:
11326:       with open(args.selected_kernel_list, 'w') as file_writer:
11327:         for line in manifest.selected_kernels:
11328:           file_writer.write("%s\n" % line)
```
**EN:** Applies conditional setup controlled by expressions such as __name__ == '__main__'.

**CN:** 根据诸如 __name__ == '__main__' 这样的条件表达式执行分支化初始化。

### L11329-L11330 — Comments

```python
11329: 
11330: ###################################################################################################
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

## Key Concepts / 关键概念

- **EN:** File role: Acts as the main kernel-library generator: it parses options, creates operation families, and populates manifests.
- **CN:** 文件角色：作为主内核库生成器：解析选项、创建各类操作族，并填充 manifest。
- **EN:** Main classes: `ConvOperation3x`
- **CN:** 主要类：`ConvOperation3x`
- **EN:** Main functions: `logging_prefix, log_debug_line, _add_package_disablement_flag, CudaToolkitVersionSatisfies, ThorSMRenumbering, EpilogueAlignment, DefaultSwizzlingFunctor, CreateGemmOperator, CreateGemmUniversal3xOperator, CreateSparseGemmUniversal3xOperator, CreateSparseGemmOperator, CreateGemmPlanarComplexOperator, ...`
- **CN:** 主要函数：`logging_prefix, log_debug_line, _add_package_disablement_flag, CudaToolkitVersionSatisfies, ThorSMRenumbering, EpilogueAlignment, DefaultSwizzlingFunctor, CreateGemmOperator, CreateGemmUniversal3xOperator, CreateSparseGemmUniversal3xOperator, CreateSparseGemmOperator, CreateGemmPlanarComplexOperator, ...`
- **EN:** Important constants/tables: `_LOGGER`
- **CN:** 重要常量/表：`_LOGGER`

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: `argparse, enum, itertools, logging, os.path, shutil, sys, copy, typing, json, builtins`
- **CN:** 标准库依赖：`argparse, enum, itertools, logging, os.path, shutil, sys, copy, typing, json, builtins`
- **EN:** Internal CUTLASS dependencies: `cutlass_library.library, cutlass_library.manifest, cutlass_library.heuristics, cutlass_library.emit_kernel_listing, cutlass_library.arch_constants, .sm90_utils, cutlass_library.sm100_utils, library, manifest, heuristics, emit_kernel_listing, sm90_utils, sm100_utils`
- **CN:** CUTLASS 内部依赖：`cutlass_library.library, cutlass_library.manifest, cutlass_library.heuristics, cutlass_library.emit_kernel_listing, cutlass_library.arch_constants, .sm90_utils, cutlass_library.sm100_utils, library, manifest, heuristics, emit_kernel_listing, sm90_utils, sm100_utils`
- **EN:** External/non-stdlib dependencies: `None explicitly imported`
- **CN:** 外部/非标准库依赖：`未显式导入`
