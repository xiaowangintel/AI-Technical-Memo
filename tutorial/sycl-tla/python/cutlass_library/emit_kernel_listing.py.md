# emit_kernel_listing.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/cutlass_library/emit_kernel_listing.py`
- **EN:** Builds structured kernel listings and feature summaries for validation, profiling, and test generation.
- **CN:** 为验证、性能分析与测试生成构建结构化的内核清单及特征摘要。

## Line-by-Line Analysis / 逐行分析

### L1-L45 — Header comments

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
 33: #
 34: #
 35: # \brief Generates the CUTLASS kernel listing with kernel filtering
 36: #
 37: 
 38: #
 39: 
 40: ###############################################################################
 41: # Example usage:
 42: # generator.py --operations all --generator-target kernel_listing \
 43: # --architectures "70;75;80" --kernels "*" --disable-cutlass-package-imports
 44: ###############################################################################
 45: 
```
**EN:** Contains the license banner and file-level header comments.

**CN:** 包含许可证横幅以及文件级头部注释。

### L46-L75 — Imports and setup

```python
 46: import collections
 47: import csv
 48: import json
 49: import math
 50: import os
 51: 
 52: try:
 53:   import builtins
 54:   if hasattr(builtins, "CUTLASS_IGNORE_PACKAGE") and CUTLASS_IGNORE_PACKAGE == True:
 55:     raise ImportError("Disabling attempt to import cutlass_library")
 56:   from cutlass_library.library import *
 57: except ImportError:
 58:   from library import *
 59: 
 60: audit_csv_fields = [
 61:   "KernelType", "KernelName", "Type_A", "Type_B", "Type_C", "Type_Acc", "Type_EpilogueScale", "Type_D", "Type_SFA", "Type_SFD",
 62:   "Layout_A", "Layout_B", "Layout_C", "Layout_D", 
 63:   "Alignment_A", "Alignment_B", "Alignment_C", "Alignment_D",  
 64:   "1SM/2SM", 
 65:   "StreamK Enabled", "Support Runtime_Cluster_Shape", "Support Runtime_Input_Types",
 66:   "Test Counts"
 67: ]
 68: 
 69: audit_csv_runtime_fields = [
 70:   "KerneIndex", "KernelName", 
 71:   "Inst_M", "Inst_N", "Inst_K", "Tile_M", "Tile_N", "Tile_K",
 72:   "Cluster_M", "Cluster_N", "Cluster_K", "Preferred_Cluster_M", "Preferred_Cluster_N", "Preferred_Cluster_K", "Fallback_Cluster_M", "Fallback_Cluster_N", "Fallback_Cluster_K",
 73:   "M", "N", "K", "L", "Alpha_val", "Beta_val",
 74:   "Runtime_Input_Types Enabled", "Runtime_Cluster_Shape Enabled"
 75: ]
```
**EN:** Imports and/or re-exports modules such as collections, csv, json, math, os so later code can reuse shared definitions.

**CN:** 导入或重新导出诸如 collections, csv, json, math, os 等模块，使后续代码可以复用共享定义。

### L77-L83 — Function `hash_cutlass_string`

```python
 77: def hash_cutlass_string(input_string):
 78:   mma_cluster_shape_pattern = r"_\d+x\d+x\d+"         # Matches MMA and Cluster shapes (e.g., '_128x128x256', '_0x0x1')
 79: 
 80:   # Remove MMA and Cluster shapes (e.g., '_128x128x256', '_0x0x1')
 81:   output = re.sub(mma_cluster_shape_pattern, "", input_string)
 82: 
 83:   return output
```
**EN:** Defines `hash_cutlass_string()`, which implements the hash cutlass string logic. Key helper calls include sub.

**CN:** 定义 `hash_cutlass_string()`，用于实现 hash cutlass string 相关逻辑。 其中会调用的重要辅助函数包括 sub。

### L85-L114 — Function `transform_hashed_string`

```python
 85: def transform_hashed_string(hashed_kernel_name, runtime_datatype_a, runtime_datatype_b):
 86:   # Define a dictionary mapping the detected types to runtime values
 87:   datatype_map = {
 88:     'f4_f4': runtime_datatype_a + '_' + runtime_datatype_b,
 89:     'f4_f6': runtime_datatype_a + '_' + runtime_datatype_b,
 90:     'f4_f8': runtime_datatype_a + '_' + runtime_datatype_b,
 91:     'f6_f4': runtime_datatype_a + '_' + runtime_datatype_b,
 92:     'f6_f6': runtime_datatype_a + '_' + runtime_datatype_b,
 93:     'f6_f8': runtime_datatype_a + '_' + runtime_datatype_b,
 94:     'f8_f4': runtime_datatype_a + '_' + runtime_datatype_b,
 95:     'f8_f6': runtime_datatype_a + '_' + runtime_datatype_b,
 96:     'f8_f8': runtime_datatype_a + '_' + runtime_datatype_b,
 97:     'ue8m0xf4_ue8m0xf4': 'ue8m0x' + runtime_datatype_a + '_ue8m0x' + runtime_datatype_b,
 98:     'ue4m3xf4_ue4m3xf4': 'ue4m3x' + runtime_datatype_a + '_ue4m3x' + runtime_datatype_b,
 99:     'ue8m0xf4_ue8m0xf6': 'ue8m0x' + runtime_datatype_a + '_ue8m0x' + runtime_datatype_b,
100:     'ue8m0xf4_ue8m0xf8': 'ue8m0x' + runtime_datatype_a + '_ue8m0x' + runtime_datatype_b,
101:     'ue8m0xf6_ue8m0xf4': 'ue8m0x' + runtime_datatype_a + '_ue8m0x' + runtime_datatype_b,
102:     'ue8m0xf6_ue8m0xf6': 'ue8m0x' + runtime_datatype_a + '_ue8m0x' + runtime_datatype_b,
103:     'ue8m0xf8_ue8m0xf4': 'ue8m0x' + runtime_datatype_a + '_ue8m0x' + runtime_datatype_b,
104:     'ue8m0xf8_ue8m0xf6': 'ue8m0x' + runtime_datatype_a + '_ue8m0x' + runtime_datatype_b,
105:     'ue8m0xf8_ue8m0xf8': 'ue8m0x' + runtime_datatype_a + '_ue8m0x' + runtime_datatype_b,
106:   }
107: 
108:   # Regular expression to detect all the keys in datatype_map
109:   pattern = re.compile(r'(' + '|'.join(map(re.escape, datatype_map.keys())) + r')')
110: 
111:   # Replace detected patterns using the dictionary
112:   updated_kernel_name = pattern.sub(lambda match: datatype_map[match.group(0)], hashed_kernel_name)
113: 
114:   return updated_kernel_name
```
**EN:** Defines `transform_hashed_string()`, which implements the transform hashed string logic. Key helper calls include compile, sub, join, group, map, keys.

**CN:** 定义 `transform_hashed_string()`，用于实现 transform hashed string 相关逻辑。 其中会调用的重要辅助函数包括 compile, sub, join, group, map, keys。

### L115-L116 — Comments

```python
115: 
116: # This helper function reports foundational kernel features: datatypes, layouts, alignment and stream-k.
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L117-L158 — Function `get_kernel_features`

```python
117: def get_kernel_features(operation, kernel_name,
118:               dynamic_datatype, runtime_input_datatype):
119:   numcta_inst = "2sm" if "2sm" in kernel_name else "1sm"
120:   math_inst = operation.tile_description.math_instruction
121: 
122:   if dynamic_datatype:
123:       dtype_name_A = runtime_input_datatype[0]
124:       dtype_name_B = runtime_input_datatype[1]
125:   else:
126:       dtype_name_A = DataTypeNames[operation.A.element]
127:       dtype_name_B = DataTypeNames[operation.B.element]
128: 
129:   layout_name_A = ShortLayoutTypeNames[operation.A.layout]
130:   layout_name_B = ShortLayoutTypeNames[operation.B.layout]
131:   layout_name_C = ShortLayoutTypeNames[operation.C.layout]
132:   layout_name_D = ShortLayoutTypeNames[operation.D.layout]
133: 
134:   scale_factor_D_type = operation.ScaleFactorD.element if hasattr(operation, "ScaleFactorD") else DataType.void
135:   scale_factor_A_type = getattr(operation, "ScaleFactorA", DataType.void)
136:   audit_vals = [
137:           "BlockScaledGEMM" if math_inst.opcode_class == OpcodeClass.BlockScaledTensorOp else "GEMM",
138:           kernel_name,
139:           dtype_name_A,
140:           dtype_name_B,
141:           DataTypeNames[operation.C.element],
142:           DataTypeNames[operation.tile_description.math_instruction.element_accumulator],
143:           DataTypeNames[operation.element_epilogue],
144:           DataTypeNames[operation.D.element],
145:           DataTypeNames[scale_factor_D_type],
146:           DataTypeNames[scale_factor_A_type],
147:           layout_name_A,
148:           layout_name_B,
149:           layout_name_C,
150:           layout_name_D,
151:           str(operation.A.alignment),
152:           str(operation.B.alignment),
153:           str(operation.C.alignment),
154:           str(operation.D.alignment),
155:           numcta_inst,
156:           "Y" if 'stream_k' in kernel_name else "N",
157:   ]
158:   return audit_vals
```
**EN:** Defines `get_kernel_features()`, which returns or derives kernel features. Key helper calls include getattr, hasattr, str.

**CN:** 定义 `get_kernel_features()`，用于返回或推导 kernel features。 其中会调用的重要辅助函数包括 getattr, hasattr, str。

### L159-L160 — Comments

```python
159: 
160: # This helper function reports other performance-related kernel parameters and those can be specified at runtime: cluster_shape, instruction shap, m/n/k and alpha/beta.
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L161-L188 — Function `get_kernel_params`

```python
161: def get_kernel_params(operation, kernel_name, cluster_shape, fallback_cluster_shape, problem_shape, alpha, beta, dynamic_datatype, dynamic_cluster):
162:   math_inst = operation.tile_description.math_instruction
163:   audit_vals = [
164:           str(math_inst.instruction_shape[0]),
165:           str(math_inst.instruction_shape[1]),
166:           str(math_inst.instruction_shape[2]),
167:           str(operation.tile_description.threadblock_shape[0]),
168:           str(operation.tile_description.threadblock_shape[1]),
169:           str(operation.tile_description.threadblock_shape[2]),
170:           str(operation.tile_description.cluster_shape[0]),
171:           str(operation.tile_description.cluster_shape[1]),
172:           str(operation.tile_description.cluster_shape[2]),
173:           str(cluster_shape[0]),
174:           str(cluster_shape[1]),
175:           str(cluster_shape[2]),
176:           str(fallback_cluster_shape[0]),
177:           str(fallback_cluster_shape[1]),
178:           str(fallback_cluster_shape[2]),
179:           str(problem_shape[0]),
180:           str(problem_shape[1]),
181:           str(problem_shape[2]),
182:           str(problem_shape[3]),
183:           str(alpha),
184:           str(beta),
185:           "Y" if dynamic_datatype else "N",
186:           "Y" if dynamic_cluster else "N",
187:   ]
188:   return audit_vals
```
**EN:** Defines `get_kernel_params()`, which returns or derives kernel params. Key helper calls include str.

**CN:** 定义 `get_kernel_params()`，用于返回或推导 kernel params。 其中会调用的重要辅助函数包括 str。

### L191-L204 — Function `_getSubOperationType`

```python
191: def _getSubOperationType(kernel):
192: 
193:   if kernel.operation_kind == OperationKind.Gemm:
194:       return GemmKindNames[kernel.gemm_kind]
195:   elif kernel.operation_kind == OperationKind.Conv2d:
196:     return "conv_" + ConvKindNames[kernel.conv_kind]
197:   elif kernel.operation_kind == OperationKind.Syrk:
198:     return "syrk_" + SyrkKindNames[kernel.syrk_kind]
199:   elif kernel.operation_kind == OperationKind.Trmm:
200:     return "trmm_" + TrmmKindNames[kernel.trmm_kind]
201:   elif kernel.operation_kind == OperationKind.Symm:
202:     return "symm_" + SymmKindNames[kernel.symm_kind]
203:   else:
204:     raise Exception("Unsupported kernel type")
```
**EN:** Defines `_getSubOperationType()`, which implements the get sub operation type logic. Key helper calls include Exception.

**CN:** 定义 `_getSubOperationType()`，用于实现 get sub operation type 相关逻辑。 其中会调用的重要辅助函数包括 Exception。

### L206-L207 — Function `_get_inst_shape`

```python
206: def _get_inst_shape(math_instruction):
207:   return "".join(str(x) for x in math_instruction.instruction_shape)
```
**EN:** Defines `_get_inst_shape()`, which implements the get inst shape logic. Key helper calls include join, str.

**CN:** 定义 `_get_inst_shape()`，用于实现 get inst shape 相关逻辑。 其中会调用的重要辅助函数包括 join, str。

### L209-L210 — Function `_is_simt_inst`

```python
209: def _is_simt_inst(math_instruction):
210:   return _get_inst_shape(math_instruction) in ["111","114"]
```
**EN:** Defines `_is_simt_inst()`, which implements the is simt inst logic. Key helper calls include _get_inst_shape.

**CN:** 定义 `_is_simt_inst()`，用于实现 is simt inst 相关逻辑。 其中会调用的重要辅助函数包括 _get_inst_shape。

### L212-L254 — Function `_getInstType`

```python
212: def _getInstType(input_precision, accumulate_precision, math_instruction):
213: 
214:   # inst_shape
215:   inst_shape = _get_inst_shape(math_instruction)
216: 
217:   # input precision
218:   if input_precision == "fp32" and inst_shape != "111":
219:     inp = "tf32"
220:   else:
221:     inp = input_precision
222: 
223:   # Handle SIMT op types first
224:   if _is_simt_inst(math_instruction):
225: 
226:     simt_input_precision_to_inst = {
227:       "fp32": "FFMA",
228:       "fp64": "DFMA",
229:       "fp16": "HFMA",
230:       "int8": "IDP4A",
231:     }
232:     inst = simt_input_precision_to_inst[input_precision]
233: 
234:   else: # Tensor op instructions
235: 
236:     if accumulate_precision == "cf64":
237:       fp64_acc_map = {
238:         MathOperation.multiply_add_complex_gaussian : "gz",
239:         MathOperation.multiply_add_complex          : "z",
240:       }
241:       acc = fp64_acc_map[math_instruction.math_operation]
242:     else:
243:       tensor_op_acc_map = {
244:         "fp32" : "s",
245:         "cf32" : "s",
246:         "fp16" : "h",
247:         "int32": "i",
248:         "fp64" : "d",
249:       }
250:       acc = tensor_op_acc_map[accumulate_precision]
251: 
252:     inst = "{}{}{}".format(acc, inst_shape, inp)
253: 
254:   return inst
```
**EN:** Defines `_getInstType()`, which implements the get inst type logic. Key helper calls include _get_inst_shape, _is_simt_inst, format.

**CN:** 定义 `_getInstType()`，用于实现 get inst type 相关逻辑。 其中会调用的重要辅助函数包括 _get_inst_shape, _is_simt_inst, format。

### L255-L255 — Comments

```python
255: # TODO: Computes FLOps/Bytes for GEMM - revisit for conv
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L256-L277 — Function `_computeFlopsPerByte`

```python
256: def _computeFlopsPerByte(operation, m, n, k, batch_count=1, beta=0.0, num_groups=1):
257:   assert not (batch_count > 1 and num_groups > 1)
258: 
259:   # TODO: adjust for sparsity
260:   gmem_bytes = (
261:     (DataTypeSize[operation.A.element] * m // 8) * k +
262:     (DataTypeSize[operation.B.element] * n // 8) * k +
263:     (DataTypeSize[operation.C.element] * m // 8) * n
264:   )
265: 
266:   # TODO: complex-valued support
267:   flops = 2 * (m * n * k)
268: 
269:   if bool(beta):
270:     gmem_bytes += (DataTypeSize[operation.C.element] * m // 8) * n
271:     flops += 2 * m * n
272: 
273:   multiplier = max(batch_count, num_groups)
274:   gmem_bytes *= multiplier
275:   flops *= multiplier
276: 
277:   return flops / gmem_bytes
```
**EN:** Defines `_computeFlopsPerByte()`, which implements the compute flops per byte logic. Key helper calls include bool, max.

**CN:** 定义 `_computeFlopsPerByte()`，用于实现 compute flops per byte 相关逻辑。 其中会调用的重要辅助函数包括 bool, max。

### L279-L867 — Function `emit_gemm_kernel_testlist`

```python
279: def emit_gemm_kernel_testlist(manifest, curr_build_dir, arch, mode
280:                               ):
281:   # For functional testing, we prefer to run reference computing on device if any
282:   reference_device_archs = ["100a", "103a"]
283:   run_reference_on_device = True if arch in reference_device_archs and mode in ["functional_L0", "functional_L1"] else False
284:   profiler_flags_for_verification = "device" if run_reference_on_device else "host"
285: 
286:   # beta values for L0 and L1
287:   # TODO: randomize beta values for wider coverage
288:   beta_values = [0.5]
289: 
290:   is_supported_arch = (arch in ["100a", "100f", "101a", "101f", "103a", "110a", "110f", "120a", "120f", "121a", "121f"])
291: 
292:   is_runtime_datatype_enabled = mode == "functional_L0" and is_supported_arch
293: 
294:   if (mode == "functional_L0") and is_supported_arch:
295:     problem_waves = [0.5, 1.25, 2.5]
296: 
297:     #
298:     # Dense Gemm
299:     #
300: 
301:     sm100_mma_data_type_general = [
302:       'gemm_f16_f16_f16_f16_f16',
303:       'gemm_f16_f16_f16_void_f16',
304:       #'gemm_f16_f16_f32_f16_f16',
305:       'tf32gemm_f32_f32_f32_f32_f32',
306:       'bf16gemm_f32_f32_f32_f32_f32',
307:     ]
308: 
309:     exclude_archs = arch not in ("103a")
310:     if exclude_archs:
311:       sm100_mma_data_type_general.append('gemm_s8_s8_s32_s8_s8')
312: 
313:     sm100_mma_data_type_runtime_dtype = [
314:       'gemm.*f4_f4_f32_f32_f32',
315:       'gemm.*f6_f6_f32_f32_f32',
316:       'gemm.*f8_f8_f32_f32_f32',
317:     ]
318: 
319:     sm100_mma_cluster_size = [
320:       '8x1x1',
321:       '4x4x1', '2x1x1',
322:       '0x0x1' # dynamic cluster
323:     ]
324: 
325:     # Restrict to two layouts to reduce L0 build and test time.
326:     sm100_mma_layouts = [ 
327:       'tnt', 
328:       'ntn' 
329:     ]
330: 
331:     # regex list must be in kernel procedural name order
332:     sm100_mma_filter_regex_1sm = "cutlass3x_sm100_tensorop.*(" + ").*(".join([ "|".join(x) for x in [sm100_mma_data_type_general, sm100_mma_cluster_size, sm100_mma_layouts]]) + ").*1sm.*"
333:     sm100_mma_filter_regex_2sm = "cutlass3x_sm100_tensorop.*(" + ").*(".join([ "|".join(x) for x in [sm100_mma_data_type_general, sm100_mma_cluster_size, sm100_mma_layouts]]) + ").*2sm.*"
334: 
335:     sm100_mma_filter_regex_1sm_runtime = "cutlass3x_sm100_tensorop.*(" + ").*(".join([ "|".join(x) for x in [sm100_mma_data_type_runtime_dtype, sm100_mma_cluster_size, sm100_mma_layouts]]) + ").*1sm.*"
336:     sm100_mma_filter_regex_2sm_runtime = "cutlass3x_sm100_tensorop.*(" + ").*(".join([ "|".join(x) for x in [sm100_mma_data_type_runtime_dtype, sm100_mma_cluster_size, sm100_mma_layouts]]) + ").*2sm.*"
337: 
338:     #
339:     # Block Scale Gemm
340:     #
341: 
342:     block_scaled_data_type = [
343:       # runtime datatypes
344:       'gemm.*ue8m0xf4_ue8m0xf4_f32_f16_e5m2',
345:       'gemm.*ue4m3xf4_ue4m3xf4_f32_f16_e5m2',
346:       'gemm.*ue8m0xf4_ue8m0xf6_f32_f16_e5m2',
347:       #'gemm.*ue8m0xf4_ue8m0xf4_f32_f16_ue8m0xe2m1',
348:       'gemm.*ue8m0xf6_ue8m0xf6_f32_f16_ue8m0xe3m2',
349:     ]
350: 
351:     block_scaled_tile_k = ['x128_', 'x256_']
352: 
353:     sm103_block_scaled_data_type = [
354:       'gemm.*ue8m0xf4_ue8m0xf4_f32_f16_e5m2',
355:       'gemm.*ue8m0xf4_ue8m0xf4_f32_f16_ue8m0xe2m1',
356:     ]
357: 
358:     sm103_block_scaled_tile_k = ['x768_']
359: 
360:     block_scaled_cluster_size = [
361:       '4x4x1', '2x1x1',
362:       '0x0x1' # dynamic cluster
363:     ]
364: 
365:     block_scaled_layouts = ['tnt']
366:     # regex list must be in kernel procedural name order
367:     block_scaled_filter_regex_1sm = "cutlass3x_sm100_bstensorop.*(" + ").*(".join([ "|".join(x) for x in [block_scaled_data_type, block_scaled_tile_k, block_scaled_cluster_size, block_scaled_layouts]]) + ").*1sm.*"
368:     block_scaled_filter_regex_2sm = "cutlass3x_sm100_bstensorop.*(" + ").*(".join([ "|".join(x) for x in [block_scaled_data_type, block_scaled_tile_k, block_scaled_cluster_size, block_scaled_layouts]]) + ").*2sm.*"
369:     
370:     sm103_block_scaled_prefetch_policy = ['tmapf']
371:     sm103_block_scaled_filter_regex_1sm = "cutlass3x_sm103_bstensorop.*(" + ").*(".join([ "|".join(x) for x in [sm103_block_scaled_data_type, sm103_block_scaled_tile_k, block_scaled_cluster_size, block_scaled_layouts]]) + ").*1sm.*(" + "|".join(sm103_block_scaled_prefetch_policy) + ").*"
372:     sm103_block_scaled_filter_regex_2sm = "cutlass3x_sm103_bstensorop.*(" + ").*(".join([ "|".join(x) for x in [sm103_block_scaled_data_type, sm103_block_scaled_tile_k, block_scaled_cluster_size, block_scaled_layouts]]) + ").*2sm.*(" + "|".join(sm103_block_scaled_prefetch_policy) + ").*"
373: 
374:     if arch in ["100a", "100f"]:
375:       kernel_filter = f"({sm100_mma_filter_regex_1sm})|" \
376:                       f"({sm100_mma_filter_regex_2sm})|" \
377:                       f"({sm100_mma_filter_regex_1sm_runtime})|" \
378:                       f"({sm100_mma_filter_regex_2sm_runtime})|" \
379:                       f"({block_scaled_filter_regex_1sm})|" \
380:                       f"({block_scaled_filter_regex_2sm})"
381:     elif arch in ["101a", "101f", "110a", "110f"]:
382:       kernel_filter = f"({sm100_mma_filter_regex_1sm})|" \
383:                       f"({sm100_mma_filter_regex_2sm})|" \
384:                       f"({sm100_mma_filter_regex_1sm_runtime})|" \
385:                       f"({sm100_mma_filter_regex_2sm_runtime})|" \
386:                       f"({block_scaled_filter_regex_1sm})|" \
387:                       f"({block_scaled_filter_regex_2sm})"
388:     elif arch in ["103a"]:
389:       kernel_filter = f"({sm100_mma_filter_regex_1sm})|" \
390:                       f"({sm100_mma_filter_regex_2sm})|" \
391:                       f"({sm100_mma_filter_regex_1sm_runtime})|" \
392:                       f"({sm100_mma_filter_regex_2sm_runtime})|" \
393:                       f"({block_scaled_filter_regex_1sm})|" \
394:                       f"({block_scaled_filter_regex_2sm})|" \
395:                       f"({sm103_block_scaled_filter_regex_1sm})|" \
396:                       f"({sm103_block_scaled_filter_regex_2sm})"
397:     elif arch in ["120a", "120f", "121a", "121f"]:
398: 
399:       # blockscaled sm120_mma kernels
400:       blockscaled_sm120_mma_kernel_cta_tiles = [
401:         [ '128x128' ]
402:       ]
403: 
404:       # Restrict to two layouts to reduce L0 build and test time.
405:       blockscaled_sm120_mma_layouts = [ 'tn' ]
406:       filter_regex_blockscaled_sm120_mma = "cutlass3x_sm120_bstensorop.*(" + ").*(".join([ "|".join(x) for x in [blockscaled_sm120_mma_kernel_cta_tiles[0], blockscaled_sm120_mma_layouts]]) + ").*"
407:       
408:       problem_waves = [0.5, 1.25, 2.5]
409: 
410:       kernel_filter = f"({filter_regex_blockscaled_sm120_mma})"
411:     else:
412:       error_message = "unsupported arch, only support sm100a, sm100f, sm101a, sm101f, sm110a, sm110f, sm103a, sm120a, sm120f, sm121a, sm121f"
413:       raise Exception(error_message)
414: 
415:   elif mode == "functional_L1":
416:     sm100_mma_cluster_size = [
417:                     '0x0x1' # dynamic cluster
418:                      ]
419:     # Restrict to two layouts to reduce L1 build and test time.
420:     sm100_mma_layouts = ['tnt', 'ntn']
421:     sm100_mma_filter_regex_1sm = "cutlass3x_sm100_tensorop.*(" + ").*(".join([ "|".join(x) for x in [sm100_mma_cluster_size, sm100_mma_layouts]]) + ").*1sm.*"
422:     sm100_mma_filter_regex_2sm = "cutlass3x_sm100_tensorop.*(" + ").*(".join([ "|".join(x) for x in [sm100_mma_cluster_size, sm100_mma_layouts]]) + ").*2sm.*"
423:     block_scaled_data_type = [
424:       'ue8m0xe2m1_ue8m0xe2m1_f32_f16_e5m2',
425:       'ue8m0xe2m1_ue8m0xe2m3_f32_f16_e5m2',
426:       'ue8m0xmx8s26_ue8m0xmx8s26_f32_f16_e5m2',
427:       'ue8m0xe2m1_ue8m0xe2m1_f32_f16_ue8m0xe2m1',
428:       'ue8m0xe2m3_ue8m0xe2m3_f32_f16_ue8m0xe3m2',
429:     ]
430: 
431:     sm103_block_scaled_data_type = [
432:       'ue8m0xe2m1_ue8m0xe2m1_f32_f16_e5m2',
433:       'ue8m0xe2m1_ue8m0xe2m1_f32_f16_ue8m0xe2m1',
434:     ]
435: 
436:     block_scaled_cluster_size = ['0x0x1']
437:     block_scaled_layouts = ['tnt']
438: 
439:     # regex list must be in kernel procedural name order
440:     block_scaled_filter_regex_1sm = "cutlass3x_sm100_bstensorop.*(" + ").*(".join([ "|".join(x) for x in [block_scaled_data_type, block_scaled_cluster_size, block_scaled_layouts]]) + ").*1sm.*"
441:     block_scaled_filter_regex_2sm = "cutlass3x_sm100_bstensorop.*(" + ").*(".join([ "|".join(x) for x in [block_scaled_data_type, block_scaled_cluster_size, block_scaled_layouts]]) + ").*2sm.*"
442: 
443:     sm103_block_scaled_filter_regex_1sm = "cutlass3x_sm103_bstensorop.*(" + ").*(".join([ "|".join(x) for x in [sm103_block_scaled_data_type, block_scaled_cluster_size, block_scaled_layouts]]) + ").*1sm.*"
444:     sm103_block_scaled_filter_regex_2sm = "cutlass3x_sm103_bstensorop.*(" + ").*(".join([ "|".join(x) for x in [sm103_block_scaled_data_type, block_scaled_cluster_size, block_scaled_layouts]]) + ").*2sm.*"
445: 
446:     filter_regex_sm100_mma = f"({sm100_mma_filter_regex_1sm})|" \
447:                           f"({sm100_mma_filter_regex_2sm})|" \
448:                           f"({block_scaled_filter_regex_1sm})|" \
449:                           f"({block_scaled_filter_regex_2sm})" \
450:                           f"({sm103_block_scaled_filter_regex_1sm})|" \
451:                           f"({sm103_block_scaled_filter_regex_2sm})"
452:     # CTA tiles for sm120 MMA - only run one tile size to reduce build/test times
453:     sm120_mma_kernel_cta_tiles = [
454:       # h1688, s1688, i16832, i8816
455:       [ '256x128' ],
456:       # d884, c1688,
457:       [ '128x128' ],
458:       # c1688, z884
459:       [ '128x64' ],
460:       # gz884
461:       [ '64x64' ]
462:     ]
463: 
464:     # sm120 MMA instruction shapes, planar complex type excluded as they are not required
465:     sm120_mma_instruction_shapes = [
466:       [ 'h1688gemm_(?!planar_complex)',
467:         's1688gemm_f16',
468:         's1688gemm_bf16',
469:         's1688gemm_tf32',
470:         'i16832gemm',
471:         'i8816gemm' ],
472:       [ 'd884gemm', 'c1688tf32gemm' ] ,
473:       [ 'c1688gemm',
474:         'z884gemm'  ],
475:       [ 'gz884gemm']
476:     ]
477: 
478:     # It's not pretty, but not sure why different instructions support different tile sizes.
479:     filter_regex_sm120_mma_0 = "cutlass_tensorop.*(" + ").*(".join([ "|".join(x) for x in [sm120_mma_instruction_shapes[0], sm120_mma_kernel_cta_tiles[0]]]) + ").*"
480:     filter_regex_sm120_mma_1 = "cutlass_tensorop.*(" + ").*(".join([ "|".join(x) for x in [sm120_mma_instruction_shapes[1], sm120_mma_kernel_cta_tiles[1]]]) + ").*"
481:     filter_regex_sm120_mma_2 = "cutlass_tensorop.*(" + ").*(".join([ "|".join(x) for x in [sm120_mma_instruction_shapes[2], sm120_mma_kernel_cta_tiles[2]]]) + ").*"
482:     filter_regex_sm120_mma_3 = "cutlass_tensorop.*(" + ").*(".join([ "|".join(x) for x in [sm120_mma_instruction_shapes[3], sm120_mma_kernel_cta_tiles[3]]]) + ").*"
483: 
484:     filter_regex_sm120_mma = f"({filter_regex_sm120_mma_0})|({filter_regex_sm120_mma_1})|({filter_regex_sm120_mma_2})|({filter_regex_sm120_mma_3})"
485: 
486:     problem_waves = [0.5, 1.25, 2.5]
487: 
488:     if arch in ["120a", "120f", "121a", "121f"]:
489:       kernel_filter = f"({filter_regex_sm120_mma})"
490:     else:
491:       kernel_filter = f"({filter_regex_sm100_mma})"
492:   else:
493:     raise ValueError()
494: 
495:   outfile_name    = os.path.join(curr_build_dir, f"FK_{mode}_testlist_SM{arch}_cutlass3x_gemm.csv")
496: 
497:   audit_file_name = os.path.join(curr_build_dir, f"FK_{mode}_audit_SM{arch}_cutlass3x_gemm.csv")
498: 
499:   audit_file_params_name = os.path.join(curr_build_dir, f"FK_{mode}_audit_params_SM{arch}_cutlass3x_gemm.csv")
500: 
501:   kernel_filter_re = re.compile(kernel_filter)
502:   testcase_counter = 0
503:   kernels_emitted = 0
504:   kernels_total = 0
505: 
506:   perf_json_list = []
507:   kernel_name_set = set()
508: 
509:   testlist_csv_fields = ["testcase", "metadata"]
510:   testlist_csv_rows = []
511:   auditlist_csv_map = {}
512:   auditlist_csv_params_map = {}
513: 
514:   kernel_features = {}
515: 
516:   for cc in manifest.operations[OperationKind.Gemm].keys():
517:     for kernel_name, operation_l in manifest.operations[OperationKind.Gemm][cc].items():
518:       assert(len(operation_l) == 1)
519:       kernels_total += 1
520:       if len(kernel_filter_re.findall(kernel_name)) == 0:
521:           continue
522:       # Only test f16 I/O void C kernels in void C kernel set
523:       # Exception: Use void C kernels for more accurate perf testing
524:       if '_void_' in kernel_name and  'perf_' not in mode:
525:         if 'f16_f16_f16_void_f16' not in kernel_name :
526:           continue
527: 
528:       kernels_emitted += 1
529:       kernel_name_set.add(kernel_name)
530:       hashed_kernel_name = hash_cutlass_string(kernel_name)
531:       operation = operation_l[0]
532: 
533:       dynamic_cluster = (operation.tile_description.cluster_shape[0] == 0
534:                           or operation.tile_description.cluster_shape[1] == 0)
535: 
536:       dynamic_datatype = "f8" in kernel_name or "f6" in kernel_name or "f4" in kernel_name
537: 
538:       runtime_input_datatypes = [None]
539: 
540:       if dynamic_datatype:
541:         if "f4_f4" in kernel_name:
542:           runtime_input_datatypes = [['e2m1','e2m1']]
543:         elif "f4_f6" in kernel_name:
544:           runtime_input_datatypes = [['e2m1','e3m2']]
545:         elif "f4_f8" in kernel_name:
546:           runtime_input_datatypes = [['e2m1','e4m3']]
547: 
548:         elif "f6_f4" in kernel_name:
549:           runtime_input_datatypes = [['e3m2','e2m1']]
550:         elif "f6_f6" in kernel_name:
551:           runtime_input_datatypes = [['e3m2','e3m2']]
552:         elif "f6_f8" in kernel_name:
553:           runtime_input_datatypes = [['e3m2','e4m3']]
554: 
555:         elif "f8_f4" in kernel_name:
556:           runtime_input_datatypes = [['e4m3','e2m1']]
557:         elif "f8_f6" in kernel_name:
558:           runtime_input_datatypes = [['e4m3','e3m2']]
559:         elif "f8_f8" in kernel_name:
560:           runtime_input_datatypes = [
561:                                     # mask out those not covered in statically encoded test cases
562:                                     #  ['e5m2','e4m3'],
563:                                     #  ['e4m3','e5m2'],
564:                                       ['e4m3','e4m3']
565:                                     ]
566: 
567:         # block scaled kernels
568:         elif "ue8m0xf4_ue8m0xf4" in kernel_name:
569:           runtime_input_datatypes = [['e2m1','e2m1']]
570:         elif "ue4m3xf4_ue4m3xf4" in kernel_name:
571:           runtime_input_datatypes = [['e2m1','e2m1']]
572:         elif "ue8m0xf4_ue8m0xf6" in kernel_name:
573:           runtime_input_datatypes = [['e2m1','e2m3']]
574:         elif "ue8m0xf4_ue8m0xf8" in kernel_name:
575:           runtime_input_datatypes = [['e2m1','e4m3']]
576: 
577:         elif "ue8m0xf6_ue8m0xf4" in kernel_name:
578:           runtime_input_datatypes = [['e2m3','e2m1']]
579:         elif "ue8m0xf6_ue8m0xf6" in kernel_name:
580:           runtime_input_datatypes = [['e2m3','e2m3']]
581:         elif "ue8m0xf8_ue8m0xf4" in kernel_name:
582:           runtime_input_datatypes = [['e4m3','e2m1']]
583: 
584:         elif "ue8m0xf8_ue8m0xf4" in kernel_name:
585:           runtime_input_datatypes = [['e4m3','e2m1']]
586:         elif "ue8m0xf8_ue8m0xf6" in kernel_name:
587:           runtime_input_datatypes = [['e4m3','e2m3']]
588:         elif "ue8m0xf8_ue8m0xf8" in kernel_name:
589:           runtime_input_datatypes = [['e4m3','e4m3']]
590: 
591:       if "bstensorop" in kernel_name or is_blockwise(manifest.operations_by_name[kernel_name].gemm_kind):
592:         profiler_flags_for_verification = "host"
593: 
594:       # reduce L1 test runtime if reference kernel is not running on device.
595:       if mode == "functional_L1" and profiler_flags_for_verification == "host" :
596:         problem_waves = [0.5, 2.5]
597:       
598: 
599:       if dynamic_cluster:
600:         if mode == "functional_L0":
601:           runtime_cluster_shapes = [[1,1,1],                   [2,2,1]]
602:         else:
603:           runtime_cluster_shapes = [[1,1,1], [1,2,1], [2,1,1], [2,2,1], [1,4,1], [4,1,1], [2,4,1], [4,2,1], [4,4,1]]
604:           # reduce L1 test runtime if reference kernel is not running on device.
605:           if profiler_flags_for_verification == "host":
606:             runtime_cluster_shapes = [[1,1,1], [1,2,1], [2,1,1], [2,2,1], [1,4,1], [4,1,1]]
607:         cta_tile_shape_m, cta_tile_shape_n, cta_tile_shape_k = operation.tile_description.threadblock_shape
608:       else:
609:         runtime_cluster_shapes = [operation.tile_description.cluster_shape]
610:         cta_tile_shape_m = int(operation.tile_description.threadblock_shape[0] / operation.tile_description.cluster_shape[0])
611:         cta_tile_shape_n = int(operation.tile_description.threadblock_shape[1] / operation.tile_description.cluster_shape[1])
612:         cta_tile_shape_k = int(operation.tile_description.threadblock_shape[2] / operation.tile_description.cluster_shape[2])
613: 
614:       alignment_a = operation.A.alignment
615:       alignment_b = operation.B.alignment
616:       alignment_c = operation.C.alignment
617:       alignment_ab_max = max(alignment_a, alignment_b)
618: 
619:       layout3x = operation.layout_name_3x()
620:       data_types = operation.datatype_name_3x()
621: 
622:       ctas_per_mma_instruction = 1
623:       if '_2sm' in kernel_name:
624:         ctas_per_mma_instruction = 2
625:         valid_cluster_shapes = []
626: 
627:         # Remove any cluster shapes that have cluster_m that is not divisible by 2
628:         for cs in runtime_cluster_shapes:
629:           if cs[0] % 2 == 0:
630:             valid_cluster_shapes.append(cs)
631:         runtime_cluster_shapes = valid_cluster_shapes
632: 
633:       kernel_problem_waves = problem_waves
634:       if mode == "functional_L0" or mode == "functional_L1":
635:         # for functional testing, we want to perturb just a little from even shapes
636:         # large K = 8 is chosen such that some kernels will warp around their smem buffers, and some will not
637:         # -16 ensures that we are TMA aligned even for FP8/Int8
638:         min_k = alignment_ab_max if cta_tile_shape_k == alignment_ab_max else cta_tile_shape_k - alignment_ab_max
639:         max_k = (cta_tile_shape_k*8) - alignment_ab_max
640:         problem_shapes_k = [min_k, max_k]
641:         sm_count = 16
642:         swizzle_sizes = [0]
643:         # Larger k and less than half wave trigger streamk +separate reduction case to be generated
644:         if 'stream_k' in kernel_name:
645:           problem_shapes_k = [max_k, cta_tile_shape_k*32]
646:           kernel_problem_waves = [0.125, 1.25, 2.5]
647:       else:
648:         raise ValueError
649: 
650:       if "void" in kernel_name:
651:         beta_values = [0]
652: 
653:       alignment_shift_m = max(alignment_c, alignment_a)
654:       alignment_shift_n = max(alignment_c, alignment_b)
655: 
656:       is_first_line = True
657:       for index_waves, waves in enumerate(kernel_problem_waves):
658:         for index_k, k in enumerate(problem_shapes_k):
659:           for beta in beta_values:
660:             for cluster_shape in runtime_cluster_shapes:
661:               for runtime_input_datatype in runtime_input_datatypes:
662:                 for swizzle_size in swizzle_sizes:
663:                   grid_size = waves * sm_count
664:                   cluster_shape_m, cluster_shape_n, cluster_shape_k = tuple(cluster_shape)
665:                   if cluster_shape_m >= cluster_shape_n:
666:                     grid_m = cluster_shape_m
667:                     grid_n = grid_size / grid_m
668:                     grid_n = max( int((grid_n + cluster_shape_n - 1) / cluster_shape_n) * cluster_shape_n, 1)
669:                   else:
670:                     grid_n = cluster_shape_n
671:                     grid_m = grid_size / grid_n
672:                     grid_m = max( int((grid_m + cluster_shape_m - 1) / cluster_shape_m) * cluster_shape_m, 1)
673: 
674:                   verification_required = False
675:                   if mode == "functional_L0" or mode == "functional_L1":
676:                     if '_void_' not in kernel_name:
677:                       verification_required = True
678: 
679:                     m = max(int(grid_m * cta_tile_shape_m), alignment_ab_max)
680:                     n = max(int(grid_n * cta_tile_shape_n), alignment_ab_max)
681:                     k = int(k)
682: 
683:                     # For functional testing, we want to perturb just a little from even shapes.
684:                     # Only do this if the perturbation does not cause one of the dimensions of the
685:                     # problem size to go to zero. This can occur for blockscaling kernels for which
686:                     # the alignment requirements for A and B can be quite large (e.g., 256).
687:                     if m > alignment_shift_m:
688:                       m -= alignment_shift_m
689:                     if n > alignment_shift_n:
690:                       n -= alignment_shift_n
691: 
692:                     if '_n32t32_' in kernel_name:
693:                       continue
694:                   batch_count = 1
695:                   if mode == "functional_L0" or mode == "functional_L1" :
696:                     if index_waves == 0 and index_k == 0 :
697:                       batch_count = 3 if mode == "functional_L0" else 5
698:                   gemm_op = "gemm"
699: 
700:                   grouped = is_grouped(manifest.operations_by_name[kernel_name].gemm_kind)
701:                   num_groups = 1
702:                   if grouped:
703:                     gemm_op = "grouped_gemm"
704:                     num_groups = 3 # small to limit test time in host block-scaled reference kernels
705:                     batch_count = 1
706:                   elif "bstensorop" in kernel_name:
707:                     gemm_op = "block_scaled_gemm"
708:                   elif is_blockwise(manifest.operations_by_name[kernel_name].gemm_kind):
709:                     gemm_op = "blockwise_gemm"
710: 
711:                   problem_size_category = ['smallK','largeK'][index_k] + '_' + ['beta==0','beta!=0'][bool(beta)]
712: 
713:                   assert m > 0 and n > 0 and k > 0
714: 
715:                   # Emit per-testcase metadata for perf testing usage, eventually in perf database
716:                   metadata_dict = {
717:                     "input_params": {
718:                       'problem_size_category' : problem_size_category,
719:                       'operation' : _getSubOperationType(operation),
720:                       'datatype' : data_types,
721:                       'layout' : layout3x,
722:                       'm' : m,
723:                       'n' : n,
724:                       'k' : k,
725:                       'beta' : beta,
726:                       'flops_per_byte' : _computeFlopsPerByte(operation, m, n, k, batch_count, beta, num_groups)
727:                     },
728:                     "runtime_params": {
729:                       'ctas_per_mma_instruction' : ctas_per_mma_instruction,
730:                       'tilesize_m' : cta_tile_shape_m,
731:                       'tilesize_n' : cta_tile_shape_n,
732:                       'tilesize_k' : cta_tile_shape_k,
733:                       'cluster_shape_m' : cluster_shape_m,
734:                       'cluster_shape_n' : cluster_shape_n,
735:                     }
736:                   }
737: 
738:                   cluster_m_fallback = ctas_per_mma_instruction if dynamic_cluster else cluster_shape_m
739:                   cluster_n_fallback = 1 if dynamic_cluster else cluster_shape_n
740:                   cluster_k_fallback = 1 if dynamic_cluster else cluster_shape_k
741: 
742: 
743:                   if dynamic_datatype:
744:                     runtime_datatype_a, runtime_datatype_b = tuple(runtime_input_datatype)
745:                     metadata_dict["runtime_params"]["runtime_datatype_a"] = runtime_datatype_a
746:                     metadata_dict["runtime_params"]["runtime_datatype_b"] = runtime_datatype_b
747: 
748:                   testcase_metadata = [
749:                     f"cutlass_profiler --operation={gemm_op}" +
750:                     (f" --verification-providers=device --providers=cutlass" if profiler_flags_for_verification == "device" else " --mode=trace") +
751:                     f" --error-on-no-match --error-if-nothing-is-profiled" +
752:                     f" --kernels={kernel_name}" +
753:                     f" --m={str(m)}" +
754:                     f" --n={str(n)}" +
755:                     f" --k={str(k)}" +
756:                     (f" --num_groups={str(num_groups)}" if grouped else "") +
757:                     f" --cluster_m={str(cluster_shape_m)}" +
758:                     f" --cluster_n={str(cluster_shape_n)}" +
759:                     f" --cluster_k={str(cluster_shape_k)}" +
760:                     f" --cluster_m_fallback={str(cluster_m_fallback)}" +
761:                     f" --cluster_n_fallback={str(cluster_n_fallback)}" +
762:                     f" --cluster_k_fallback={str(cluster_k_fallback)}" +
763:                     f" --beta={str(beta)}" +
764:                     ("" if grouped else f" --batch_count={str(batch_count)}") +
765:                     f" --swizzle_size={str(swizzle_size)}" +
766:                     f" --verification-required={str(verification_required).lower()}"
767:                   ] \
768: 
769:                   output_dynamic_datatype = dynamic_datatype
770:                   if output_dynamic_datatype:
771:                     testcase_metadata[0] += (f" --runtime_input_datatype_a={runtime_datatype_a}" +
772:                                               f" --runtime_input_datatype_b={runtime_datatype_b}")
773: 
774:                   testcase_metadata.append(json.dumps(metadata_dict))
775:                   testlist_csv_rows.append(testcase_metadata)
776:                   testcase_counter += 1
777: 
778:                   alpha = 1.0
779: 
780:                   if dynamic_datatype:
781:                     hashed_kernel_name = transform_hashed_string(hashed_kernel_name, runtime_datatype_a, runtime_datatype_b)
782: 
783:                   # If kernel_name is new, initialize its feature set with defaults
784:                   if hashed_kernel_name not in kernel_features:
785:                     kernel_features[hashed_kernel_name] = {
786:                       "is_support_dynamic_cluster": False,
787:                       "is_support_dynamic_datatype": False,
788:                     }
789: 
790:                   # Update features for the hashed kernel name
791:                   kernel_features[hashed_kernel_name]["is_support_dynamic_cluster"] |= dynamic_cluster
792:                   kernel_features[hashed_kernel_name]["is_support_dynamic_datatype"] |= dynamic_datatype
793: 
794:                   if hashed_kernel_name not in auditlist_csv_params_map:
795:                     auditlist_csv_params_map[hashed_kernel_name] = []
796: 
797:                   audit_row_params = get_kernel_params(
798:                     operation,
799:                     hashed_kernel_name,
800:                     (cluster_shape_m, cluster_shape_n, cluster_shape_k),
801:                     (cluster_m_fallback, cluster_n_fallback, cluster_k_fallback),
802:                     (m, n, k, batch_count),
803:                     alpha, beta,
804:                     dynamic_datatype, dynamic_cluster
805:                   )
806: 
807:                   auditlist_csv_params_map[hashed_kernel_name].append(audit_row_params)
808: 
809:                   if hashed_kernel_name not in auditlist_csv_map:
810:                     audit_row = get_kernel_features(operation, hashed_kernel_name, dynamic_datatype, runtime_input_datatype)
811:                     auditlist_csv_map[hashed_kernel_name] = audit_row
812: 
813:   with open(outfile_name, 'w') as testlist_csv:
814:     csv_writer = csv.writer(testlist_csv, delimiter=',')
815:     csv_writer.writerow(testlist_csv_fields)
816:     csv_writer.writerows(testlist_csv_rows)
817: 
818:   with open(audit_file_name, 'w') as auditlist_csv:
819:     csv_writer = csv.writer(auditlist_csv, delimiter=',')
820:     csv_writer.writerow(audit_csv_fields)
821:     for hashed_kernel_name, row in auditlist_csv_map.items():
822:       # Append the dynamic features as "Y" or "N"
823:       dynamic_cluster_flag = "Y" if kernel_features[hashed_kernel_name]["is_support_dynamic_cluster"] else "N"
824:       dynamic_datatype_flag = "Y" if kernel_features[hashed_kernel_name]["is_support_dynamic_datatype"] else "N"
825:       test_count = len(auditlist_csv_params_map[hashed_kernel_name])
826:       csv_writer.writerow(row + [dynamic_cluster_flag, dynamic_datatype_flag, test_count])
827: 
828:   with open(audit_file_params_name, 'w') as auditlist_csv:
829:     csv_writer = csv.writer(auditlist_csv, delimiter=',')
830:     csv_writer.writerow(audit_csv_runtime_fields)
831:     for kernel_index, (hashed_kernel_name, rows) in enumerate(auditlist_csv_params_map.items(), start=1):
832:       for i, row in enumerate(rows):
833:         if i == 0:
834:           csv_writer.writerow([kernel_index, hashed_kernel_name] + row)
835:         else:
836:           csv_writer.writerow(["", ""] + row)
837: 
838:   print(f"Generated a total of {testcase_counter} test cases for {kernels_emitted} kernels out of {kernels_total} total.")
839: 
840:   # Generate a newline separated list of kernel filters
841:   assert(len(kernel_name_set) == kernels_emitted)
842:   output_filter_enabled = True
843:   if output_filter_enabled:
844:     kernel_filter_outfile_name = os.path.join(curr_build_dir, f"FK_{mode}_testlist_SM{arch}_cutlass3x_gemm_kernel_filter.list")
845:   with open(kernel_filter_outfile_name, "w") as file:
846:       kernel_name_set = set(map(lambda x: x.replace("_epi_tma", ""), kernel_name_set))
847:       for kernel_name in kernel_name_set:
848:           file.write(kernel_name + "\n")
849: 
850:   # Sort L0 and L1 kernel list and csv file to avoid mixing cutlass3.x kernels and sm120_mma kernels in cutlass2.x generated together.
851:   if mode == "functional_L0" or mode == "functional_L1":
852:     # Sort the .csv file
853:     outfile_name = os.path.join(curr_build_dir, f"FK_{mode}_testlist_SM{arch}_cutlass3x_gemm.csv")
854:     with open(outfile_name) as file:
855:       data = file.readlines()
856:       data.sort()
857:     with open(outfile_name, 'w') as file:
858:       for i in range(len(data)):
859:         file.write(data[i])
860:     # Sort the kernel list
861:     kernel_filter_outfile_name = os.path.join(curr_build_dir, f"FK_{mode}_testlist_SM{arch}_cutlass3x_gemm_kernel_filter.list")
862:     with open(kernel_filter_outfile_name) as file:
863:       data = file.readlines()
864:       data.sort()
865:     with open(kernel_filter_outfile_name, 'w') as file:
866:       for i in range(len(data)):
867:         file.write(data[i])
```
**EN:** Defines `emit_gemm_kernel_testlist()`, which emits gemm kernel testlist. Key helper calls include join, compile, set, keys, print, items.

**CN:** 定义 `emit_gemm_kernel_testlist()`，用于输出/生成 gemm kernel testlist。 其中会调用的重要辅助函数包括 join, compile, set, keys, print, items。

## Key Concepts / 关键概念

- **EN:** File role: Builds structured kernel listings and feature summaries for validation, profiling, and test generation.
- **CN:** 文件角色：为验证、性能分析与测试生成构建结构化的内核清单及特征摘要。
- **EN:** Main functions: `hash_cutlass_string, transform_hashed_string, get_kernel_features, get_kernel_params, _getSubOperationType, _get_inst_shape, _is_simt_inst, _getInstType, _computeFlopsPerByte, emit_gemm_kernel_testlist`
- **CN:** 主要函数：`hash_cutlass_string, transform_hashed_string, get_kernel_features, get_kernel_params, _getSubOperationType, _get_inst_shape, _is_simt_inst, _getInstType, _computeFlopsPerByte, emit_gemm_kernel_testlist`

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: `collections, csv, json, math, os, builtins`
- **CN:** 标准库依赖：`collections, csv, json, math, os, builtins`
- **EN:** Internal CUTLASS dependencies: `cutlass_library.library, library`
- **CN:** CUTLASS 内部依赖：`cutlass_library.library, library`
- **EN:** External/non-stdlib dependencies: `None explicitly imported`
- **CN:** 外部/非标准库依赖：`未显式导入`
