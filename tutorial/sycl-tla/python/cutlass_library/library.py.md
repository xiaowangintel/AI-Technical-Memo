# library.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/cutlass_library/library.py`
- **EN:** Defines the shared enums, type/layout metadata, tags, and helper utilities consumed by the whole generator stack.
- **CN:** 定义整个生成器体系共享的枚举、类型/布局元数据、标签与辅助工具。

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
  34: Data types and tags used for emitting CUTLASS C++ kernels
  35: """
  36: 
  37: import enum
  38: import re
  39: 
  40: # The following block implements enum.auto() for Python 3.5 variants that don't include it such
  41: # as the default 3.5.2 on Ubuntu 16.04.
  42: #
  43: # https://codereview.stackexchange.com/questions/177309/reimplementing-pythons-enum-auto-for-compatibility
  44: 
  45: try:
  46:   from enum import auto as enum_auto
  47: except ImportError:
  48:   __cutlass_library_auto_enum = 0
  49:   def enum_auto() -> int:
  50:     global __cutlass_library_auto_enum
  51:     i = __cutlass_library_auto_enum
  52:     __cutlass_library_auto_enum += 1
  53:     return i
```
**EN:** Imports and/or re-exports modules such as enum, re so later code can reuse shared definitions.

**CN:** 导入或重新导出诸如 enum, re 等模块，使后续代码可以复用共享定义。

### L54-L57 — Comments

```python
  54: 
  55: ###################################################################################################
  56: 
  57: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L58-L58 — Class `GeneratorTarget`

```python
  58: class GeneratorTarget(enum.Enum):
```
**EN:** Introduces class `GeneratorTarget`, which packages the generator target logic into a reusable type.

**CN:** 引入类 `GeneratorTarget`，将 generator target 相关逻辑封装为可复用类型。

### L59-L59 — Data definitions

```python
  59:   Library = enum_auto()
```
**EN:** Defines or updates module/class-level data such as Library; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Library；这些值会被后续生成器与 emitter 引用。

### L60-L60 — Comments

```python
  60: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L61-L72 — Import fallback

```python
  61: GeneratorTargetNames = {
  62:   GeneratorTarget.Library: 'library'
  63: }
  64: #
  65: 
  66: ###################################################################################################
  67: 
  68: # Architecture constants import with fallback for relative imports
  69: try:
  70:   from cutlass_library.arch_constants import INTEL_XE_ARCH_MIN, INTEL_XE_ARCH_MAX, CUDA_ARCH_MIN, INTEL_XE12, INTEL_XE20
  71: except ImportError:
  72:   from arch_constants import INTEL_XE_ARCH_MIN, INTEL_XE_ARCH_MAX, CUDA_ARCH_MIN, INTEL_XE12, INTEL_XE20
```
**EN:** Uses `try`/`except` to prefer package imports and fall back to local imports, keeping both installed-package and script execution modes working.

**CN:** 通过 `try`/`except` 优先使用包导入，并在失败时回退到本地导入，从而同时兼容已安装包与脚本运行模式。

### L75-L75 — Class `DataType`

```python
  75: class DataType(enum.Enum):
```
**EN:** Introduces class `DataType`, which packages the data type logic into a reusable type.

**CN:** 引入类 `DataType`，将 data type 相关逻辑封装为可复用类型。

### L76-L122 — Data definitions

```python
  76:   void = enum_auto()  # primarily used to disable C tensor for epilogues
  77:   b1 = enum_auto()
  78:   u2 = enum_auto()
  79:   u4 = enum_auto()
  80:   u8 = enum_auto()
  81:   u16 = enum_auto()
  82:   u32 = enum_auto()
  83:   u64 = enum_auto()
  84:   s2 = enum_auto()
  85:   s4 = enum_auto()
  86:   s8 = enum_auto()
  87:   s16 = enum_auto()
  88:   s32 = enum_auto()
  89:   s64 = enum_auto()
  90:   e4m3 = enum_auto()
  91:   e5m2 = enum_auto()
  92:   f8 = enum_auto()    
  93:   f6 = enum_auto()    
  94:   f4 = enum_auto()    
  95:   e3m2 = enum_auto()     
  96:   e2m3 = enum_auto()     
  97:   e2m1 = enum_auto()     
  98:   ue8m0 = enum_auto()    
  99:   ue4m3 = enum_auto()    
 100:   f16 = enum_auto()
 101:   bf16 = enum_auto()
 102:   f32 = enum_auto()
 103:   tf32 = enum_auto()
 104:   f64 = enum_auto()
 105:   cf16 = enum_auto()
 106:   cbf16 = enum_auto()
 107:   cf32 = enum_auto()
 108:   ctf32 = enum_auto()
 109:   cf64 = enum_auto()
 110:   cs2 = enum_auto()
 111:   cs4 = enum_auto()
 112:   cs8 = enum_auto()
 113:   cs16 = enum_auto()
 114:   cs32 = enum_auto()
 115:   cs64 = enum_auto()
 116:   cu2 = enum_auto()
 117:   cu4 = enum_auto()
 118:   cu8 = enum_auto()
 119:   cu16 = enum_auto()
 120:   cu32 = enum_auto()
 121:   cu64 = enum_auto()
 122:   invalid = enum_auto()
```
**EN:** Defines or updates module/class-level data such as void, b1, u2, u4, u8, u16, u32, u64, ...; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 void, b1, u2, u4, u8, u16, u32, u64, ...；这些值会被后续生成器与 emitter 引用。

### L123-L124 — Comments

```python
 123: 
 124: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L125-L286 — Data definitions

```python
 125: ShortDataTypeNames = {
 126:   DataType.s32: 'i',
 127:   DataType.e4m3: 'e4m3',
 128:   DataType.e5m2: 'e5m2',
 129:   DataType.f16: 'h',
 130:   DataType.f32: 's',
 131:   DataType.f64: 'd',
 132:   DataType.bf16: 'bf16',
 133:   DataType.cf32: 'c',
 134:   DataType.cf64: 'z',
 135:   DataType.f8: 'f8',      
 136:   DataType.f6: 'f6',      
 137:   DataType.f4: 'f4',      
 138: }
 139: 
 140: #
 141: DataTypeNames = {
 142:   DataType.void: "void",
 143:   DataType.b1: "b1",
 144:   DataType.u2: "u2",
 145:   DataType.u4: "u4",
 146:   DataType.u8: "u8",
 147:   DataType.u16: "u16",
 148:   DataType.u32: "u32",
 149:   DataType.u64: "u64",
 150:   DataType.s2: "s2",
 151:   DataType.s4: "s4",
 152:   DataType.s8: "s8",
 153:   DataType.s16: "s16",
 154:   DataType.s32: "s32",
 155:   DataType.s64: "s64",
 156:   DataType.e4m3: 'e4m3',
 157:   DataType.e5m2: 'e5m2',
 158:   DataType.f8: 'f8',     
 159:   DataType.f6: 'f6',     
 160:   DataType.f4: 'f4',     
 161:   DataType.e2m3: 'e2m3',       
 162:   DataType.e3m2: 'e3m2',       
 163:   DataType.e2m1: 'e2m1',       
 164:   DataType.ue8m0: 'ue8m0',     
 165:   DataType.ue4m3: 'ue4m3',     
 166:   DataType.f16: "f16",
 167:   DataType.bf16: "bf16",
 168:   DataType.f32: "f32",
 169:   DataType.tf32: "tf32",
 170:   DataType.f64: "f64",
 171:   DataType.cf16: "cf16",
 172:   DataType.cbf16: "cbf16",
 173:   DataType.cf32: "cf32",
 174:   DataType.ctf32: "ctf32",
 175:   DataType.cf64: "cf64",
 176:   DataType.cu2: "cu2",
 177:   DataType.cu4: "cu4",
 178:   DataType.cu8: "cu8",
 179:   DataType.cu16: "cu16",
 180:   DataType.cu32: "cu32",
 181:   DataType.cu64: "cu64",
 182:   DataType.cs2: "cs2",
 183:   DataType.cs4: "cs4",
 184:   DataType.cs8: "cs8",
 185:   DataType.cs16: "cs16",
 186:   DataType.cs32: "cs32",
 187:   DataType.cs64: "cs64",
 188: }
 189: 
 190: DataTypeTag = {
 191:   DataType.void: "void",
 192:   DataType.b1: "cutlass::uint1b_t",
 193:   DataType.u2: "cutlass::uint2b_t",
 194:   DataType.u4: "cutlass::uint4b_t",
 195:   DataType.u8: "uint8_t",
 196:   DataType.u16: "uint16_t",
 197:   DataType.u32: "uint32_t",
 198:   DataType.u64: "uint64_t",
 199:   DataType.s2: "cutlass::int2b_t",
 200:   DataType.s4: "cutlass::int4b_t",
 201:   DataType.s8: "int8_t",
 202:   DataType.s16: "int16_t",
 203:   DataType.s32: "int32_t",
 204:   DataType.s64: "int64_t",
 205:   DataType.e4m3: 'cutlass::float_e4m3_t',
 206:   DataType.e5m2: 'cutlass::float_e5m2_t',
 207:   DataType.f8: 'cutlass::type_erased_dynamic_float8_t',      
 208:   DataType.f6: 'cutlass::type_erased_dynamic_float6_t',      
 209:   DataType.f4: 'cutlass::type_erased_dynamic_float4_t',      
 210:   DataType.e2m3: 'cutlass::float_e2m3_t',                       
 211:   DataType.e3m2: 'cutlass::float_e3m2_t',                       
 212:   DataType.e2m1: 'cutlass::float_e2m1_t',                       
 213:   DataType.ue8m0: 'cutlass::float_ue8m0_t',                     
 214:   DataType.ue4m3: 'cutlass::float_ue4m3_t',                     
 215:   DataType.f16: "cutlass::half_t",
 216:   DataType.bf16: "cutlass::bfloat16_t",
 217:   DataType.f32: "float",
 218:   DataType.tf32: "cutlass::tfloat32_t",
 219:   DataType.f64: "double",
 220:   DataType.cf16: "cutlass::complex<cutlass::half_t>",
 221:   DataType.cbf16: "cutlass::complex<cutlass::bfloat16_t>",
 222:   DataType.cf32: "cutlass::complex<float>",
 223:   DataType.ctf32: "cutlass::complex<cutlass::tfloat32_t>",
 224:   DataType.cf64: "cutlass::complex<double>",
 225:   DataType.cu2: "cutlass::complex<cutlass::uint2b_t>",
 226:   DataType.cu4: "cutlass::complex<cutlass::uint4b_t>",
 227:   DataType.cu8: "cutlass::complex<cutlass::uint8_t>",
 228:   DataType.cu16: "cutlass::complex<cutlass::uint16_t>",
 229:   DataType.cu32: "cutlass::complex<cutlass::uint32_t>",
 230:   DataType.cu64: "cutlass::complex<cutlass::uint64_t>",
 231:   DataType.cs2: "cutlass::complex<cutlass::int2b_t>",
 232:   DataType.cs4: "cutlass::complex<cutlass::int4b_t>",
 233:   DataType.cs8: "cutlass::complex<cutlass::int8_t>",
 234:   DataType.cs16: "cutlass::complex<cutlass::int16_t>",
 235:   DataType.cs32: "cutlass::complex<cutlass::int32_t>",
 236:   DataType.cs64: "cutlass::complex<cutlass::int64_t>",
 237: }
 238: 
 239: DataTypeSize = {
 240:   DataType.void: 0,
 241:   DataType.b1: 1,
 242:   DataType.u2: 2,
 243:   DataType.u4: 4,
 244:   DataType.u8: 8,
 245:   DataType.u16: 16,
 246:   DataType.u32: 32,
 247:   DataType.u64: 64,
 248:   DataType.s2: 2,
 249:   DataType.s4: 4,
 250:   DataType.s8: 8,
 251:   DataType.s16: 16,
 252:   DataType.s32: 32,
 253:   DataType.s64: 64,
 254:   DataType.e4m3: 8,
 255:   DataType.e5m2: 8,
 256:   DataType.f8: 8,
 257:   DataType.f6: 6,
 258:   DataType.f4: 4,
 259:   DataType.e2m3: 6,
 260:   DataType.e3m2: 6,
 261:   DataType.e2m1: 4,
 262:   DataType.ue8m0: 8,
 263:   DataType.ue4m3: 8,
 264:   DataType.f16: 16,
 265:   DataType.bf16: 16,
 266:   DataType.f32: 32,
 267:   DataType.tf32: 32,
 268:   DataType.f64: 64,
 269:   DataType.cf16: 32,
 270:   DataType.cbf16: 32,
 271:   DataType.cf32: 64,
 272:   DataType.ctf32: 32,
 273:   DataType.cf64: 128,
 274:   DataType.cu2: 4,
 275:   DataType.cu4: 8,
 276:   DataType.cu8: 16,
 277:   DataType.cu16: 32,
 278:   DataType.cu32: 64,
 279:   DataType.cu64: 128,
 280:   DataType.cs2: 4,
 281:   DataType.cs4: 8,
 282:   DataType.cs8: 16,
 283:   DataType.cs16: 32,
 284:   DataType.cs32: 64,
 285:   DataType.cs64: 128,
 286: }
```
**EN:** Defines or updates module/class-level data such as ShortDataTypeNames, DataTypeNames, DataTypeTag, DataTypeSize; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 ShortDataTypeNames, DataTypeNames, DataTypeTag, DataTypeSize；这些值会被后续生成器与 emitter 引用。

### L287-L289 — Comments

```python
 287: 
 288: ###################################################################################################
 289: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L290-L290 — Class `BlasMode`

```python
 290: class BlasMode(enum.Enum):
```
**EN:** Introduces class `BlasMode`, which packages the blas mode logic into a reusable type.

**CN:** 引入类 `BlasMode`，将 blas mode 相关逻辑封装为可复用类型。

### L291-L292 — Data definitions

```python
 291:   symmetric = enum_auto()
 292:   hermitian = enum_auto()
```
**EN:** Defines or updates module/class-level data such as symmetric, hermitian; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 symmetric, hermitian；这些值会被后续生成器与 emitter 引用。

### L293-L294 — Comments

```python
 293: 
 294: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L295-L298 — Data definitions

```python
 295: BlasModeTag = {
 296:   BlasMode.symmetric: 'cutlass::BlasMode::kSymmetric',
 297:   BlasMode.hermitian: 'cutlass::BlasMode::kHermitian',
 298: }
```
**EN:** Defines or updates module/class-level data such as BlasModeTag; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 BlasModeTag；这些值会被后续生成器与 emitter 引用。

### L299-L300 — Comments

```python
 299: 
 300: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L301-L301 — Class `ComplexTransform`

```python
 301: class ComplexTransform(enum.Enum):
```
**EN:** Introduces class `ComplexTransform`, which packages the complex transform logic into a reusable type.

**CN:** 引入类 `ComplexTransform`，将 complex transform 相关逻辑封装为可复用类型。

### L302-L303 — Data definitions

```python
 302:   none = enum_auto()
 303:   conj = enum_auto()
```
**EN:** Defines or updates module/class-level data such as none, conj; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 none, conj；这些值会被后续生成器与 emitter 引用。

### L304-L305 — Comments

```python
 304: 
 305: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L306-L322 — Data definitions

```python
 306: ComplexTransformTag = {
 307:   ComplexTransform.none: 'cutlass::ComplexTransform::kNone',
 308:   ComplexTransform.conj: 'cutlass::ComplexTransform::kConjugate',
 309: }
 310: 
 311: # Used for cutlass3x complex kernel collective mainloop builder instantiation
 312: ComplexTransformTag3x = {
 313:   ComplexTransform.none: 'cute::identity',
 314:   ComplexTransform.conj: 'cute::conjugate',
 315: }
 316: 
 317: #
 318: RealComplexBijection = [
 319:   (DataType.f16, DataType.cf16),
 320:   (DataType.f32, DataType.cf32),
 321:   (DataType.f64, DataType.cf64),
 322: ]
```
**EN:** Defines or updates module/class-level data such as ComplexTransformTag, ComplexTransformTag3x, RealComplexBijection; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 ComplexTransformTag, ComplexTransformTag3x, RealComplexBijection；这些值会被后续生成器与 emitter 引用。

### L323-L324 — Comments

```python
 323: 
 324: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L325-L329 — Function `is_complex`

```python
 325: def is_complex(data_type):
 326:   for r, c in RealComplexBijection:
 327:     if data_type == c:
 328:       return True
 329:   return False
```
**EN:** Defines `is_complex()`, a predicate that checks whether complex.

**CN:** 定义 `is_complex()`，用于判断是否满足 complex 这一条件。

### L331-L332 — Function `is_block_scaled`

```python
 331: def is_block_scaled(gemm_kind):
 332:   return gemm_kind in (GemmKind.BlockScaledUniversal3x, GemmKind.GroupedBlockScaledUniversal3x)
```
**EN:** Defines `is_block_scaled()`, a predicate that checks whether block scaled.

**CN:** 定义 `is_block_scaled()`，用于判断是否满足 block scaled 这一条件。

### L334-L335 — Function `is_blockwise`

```python
 334: def is_blockwise(gemm_kind):
 335:   return gemm_kind in (GemmKind.BlockwiseUniversal3x, GemmKind.GroupedBlockwiseUniversal3x)
```
**EN:** Defines `is_blockwise()`, a predicate that checks whether blockwise.

**CN:** 定义 `is_blockwise()`，用于判断是否满足 blockwise 这一条件。

### L337-L339 — Function `is_grouped`

```python
 337: def is_grouped(gemm_kind):
 338:   return gemm_kind in (GemmKind.GroupedUniversal3x, 
 339:     GemmKind.GroupedBlockScaledUniversal3x, GemmKind.GroupedBlockwiseUniversal3x)
```
**EN:** Defines `is_grouped()`, a predicate that checks whether grouped.

**CN:** 定义 `is_grouped()`，用于判断是否满足 grouped 这一条件。

### L340-L341 — Comments

```python
 340: 
 341: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L342-L346 — Function `get_complex_from_real`

```python
 342: def get_complex_from_real(real_type):
 343:   for r, c in RealComplexBijection:
 344:     if real_type == r:
 345:       return c
 346:   return DataType.invalid
```
**EN:** Defines `get_complex_from_real()`, which returns or derives complex from real.

**CN:** 定义 `get_complex_from_real()`，用于返回或推导 complex from real。

### L347-L348 — Comments

```python
 347: 
 348: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L349-L353 — Function `get_real_from_complex`

```python
 349: def get_real_from_complex(complex_type):
 350:   for r, c in RealComplexBijection:
 351:     if complex_type == c:
 352:       return r
 353:   return DataType.invalid
```
**EN:** Defines `get_real_from_complex()`, which returns or derives real from complex.

**CN:** 定义 `get_real_from_complex()`，用于返回或推导 real from complex。

### L354-L355 — Comments

```python
 354: 
 355: # TMA requires an alignment of 128 bits for all data types
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L356-L362 — Function `get_tma_alignment`

```python
 356: def get_tma_alignment(data_type):
 357:   if data_type == DataType.void:
 358:     return 0
 359:   elif DataTypeSize[data_type] == 6:
 360:     return 128 # 96B alignment for 16U6 format 
 361:   else:
 362:     return 128 // DataTypeSize[data_type]
```
**EN:** Defines `get_tma_alignment()`, which returns or derives tma alignment.

**CN:** 定义 `get_tma_alignment()`，用于返回或推导 tma alignment。

### L363-L364 — Comments

```python
 363: 
 364: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L365-L365 — Class `ComplexMultiplyOp`

```python
 365: class ComplexMultiplyOp(enum.Enum):
```
**EN:** Introduces class `ComplexMultiplyOp`, which packages the complex multiply op logic into a reusable type.

**CN:** 引入类 `ComplexMultiplyOp`，将 complex multiply op 相关逻辑封装为可复用类型。

### L366-L367 — Data definitions

```python
 366:   multiply_add = enum_auto()
 367:   gaussian = enum_auto()
```
**EN:** Defines or updates module/class-level data such as multiply_add, gaussian; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 multiply_add, gaussian；这些值会被后续生成器与 emitter 引用。

### L368-L371 — Comments

```python
 368: 
 369: ###################################################################################################
 370: 
 371: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L372-L372 — Class `MathOperation`

```python
 372: class MathOperation(enum.Enum):
```
**EN:** Introduces class `MathOperation`, a descriptor for math operations.

**CN:** 引入类 `MathOperation`，它是 math 操作的描述对象。

### L373-L384 — Data definitions

```python
 373:   multiply_add = enum_auto()
 374:   multiply_add_saturate = enum_auto()
 375:   multiply_add_mixed_input_upcast = enum_auto()
 376:   xor_popc = enum_auto()
 377:   and_popc = enum_auto()
 378:   multiply_add_fast_bf16 = enum_auto()
 379:   multiply_add_fast_f16 = enum_auto()
 380:   multiply_add_fast_f32 = enum_auto()
 381:   multiply_add_complex_fast_f32 = enum_auto()
 382:   multiply_add_complex = enum_auto()
 383:   multiply_add_complex_gaussian = enum_auto()
 384:   multiply_add_fast_accum = enum_auto()
```
**EN:** Defines or updates module/class-level data such as multiply_add, multiply_add_saturate, multiply_add_mixed_input_upcast, xor_popc, and_popc, multiply_add_fast_bf16, multiply_add_fast_f16, multiply_add_fast_f32, ...; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 multiply_add, multiply_add_saturate, multiply_add_mixed_input_upcast, xor_popc, and_popc, multiply_add_fast_bf16, multiply_add_fast_f16, multiply_add_fast_f32, ...；这些值会被后续生成器与 emitter 引用。

### L385-L386 — Comments

```python
 385: 
 386: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L387-L400 — Data definitions

```python
 387: MathOperationTag = {
 388:   MathOperation.multiply_add: 'cutlass::arch::OpMultiplyAdd',
 389:   MathOperation.multiply_add_saturate: 'cutlass::arch::OpMultiplyAddSaturate',
 390:   MathOperation.multiply_add_mixed_input_upcast: 'cutlass::arch::OpMultiplyAddMixedInputUpcast',
 391:   MathOperation.xor_popc: 'cutlass::arch::OpXorPopc',
 392:   MathOperation.and_popc: 'cutlass::arch::OpAndPopc',
 393:   MathOperation.multiply_add_fast_bf16: 'cutlass::arch::OpMultiplyAddFastBF16',
 394:   MathOperation.multiply_add_fast_f16: 'cutlass::arch::OpMultiplyAddFastF16',
 395:   MathOperation.multiply_add_fast_f32: 'cutlass::arch::OpMultiplyAddFastF32',
 396:   MathOperation.multiply_add_complex_fast_f32: 'cutlass::arch::OpMultiplyAddComplexFastF32',
 397:   MathOperation.multiply_add_complex: 'cutlass::arch::OpMultiplyAddComplex',
 398:   MathOperation.multiply_add_complex_gaussian: 'cutlass::arch::OpMultiplyAddGaussianComplex',
 399:   MathOperation.multiply_add_fast_accum: 'cutlass::arch::OpMultiplyAddFastAccum',
 400: }
```
**EN:** Defines or updates module/class-level data such as MathOperationTag; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 MathOperationTag；这些值会被后续生成器与 emitter 引用。

### L401-L404 — Comments

```python
 401: 
 402: ###################################################################################################
 403: 
 404: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L405-L405 — Class `LayoutType`

```python
 405: class LayoutType(enum.Enum):
```
**EN:** Introduces class `LayoutType`, which packages the layout type logic into a reusable type.

**CN:** 引入类 `LayoutType`，将 layout type 相关逻辑封装为可复用类型。

### L406-L425 — Data definitions

```python
 406:   ColumnMajor = enum_auto()
 407:   RowMajor = enum_auto()
 408:   ColumnMajorInterleaved2 = enum_auto()
 409:   RowMajorInterleaved2 = enum_auto()
 410:   ColumnMajorInterleaved32 = enum_auto()
 411:   RowMajorInterleaved32 = enum_auto()
 412:   ColumnMajorInterleaved64 = enum_auto()
 413:   RowMajorInterleaved64 = enum_auto()
 414:   TensorNWC = enum_auto()
 415:   TensorNHWC = enum_auto()
 416:   TensorNDHWC = enum_auto()
 417:   TensorNCHW = enum_auto()
 418:   TensorNGHWC = enum_auto()
 419:   TensorNC32HW32 = enum_auto()
 420:   TensorNC64HW64 = enum_auto()
 421:   TensorC32RSK32 = enum_auto()
 422:   TensorC64RSK64 = enum_auto()
 423:   TensorKCS = enum_auto()
 424:   TensorKCSR = enum_auto()
 425:   TensorKCSRT = enum_auto()
```
**EN:** Defines or updates module/class-level data such as ColumnMajor, RowMajor, ColumnMajorInterleaved2, RowMajorInterleaved2, ColumnMajorInterleaved32, RowMajorInterleaved32, ColumnMajorInterleaved64, RowMajorInterleaved64, ...; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 ColumnMajor, RowMajor, ColumnMajorInterleaved2, RowMajorInterleaved2, ColumnMajorInterleaved32, RowMajorInterleaved32, ColumnMajorInterleaved64, RowMajorInterleaved64, ...；这些值会被后续生成器与 emitter 引用。

### L426-L427 — Comments

```python
 426: 
 427: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L428-L494 — Data definitions

```python
 428: LayoutTag = {
 429:   LayoutType.ColumnMajor: 'cutlass::layout::ColumnMajor',
 430:   LayoutType.RowMajor: 'cutlass::layout::RowMajor',
 431:   LayoutType.ColumnMajorInterleaved2: 'cutlass::layout::ColumnMajorInterleaved<2>',
 432:   LayoutType.RowMajorInterleaved2: 'cutlass::layout::RowMajorInterleaved<2>',
 433:   LayoutType.ColumnMajorInterleaved32: 'cutlass::layout::ColumnMajorInterleaved<32>',
 434:   LayoutType.RowMajorInterleaved32: 'cutlass::layout::RowMajorInterleaved<32>',
 435:   LayoutType.ColumnMajorInterleaved64: 'cutlass::layout::ColumnMajorInterleaved<64>',
 436:   LayoutType.RowMajorInterleaved64: 'cutlass::layout::RowMajorInterleaved<64>',
 437:   LayoutType.TensorNWC: 'cutlass::layout::TensorNWC',
 438:   LayoutType.TensorNHWC: 'cutlass::layout::TensorNHWC',
 439:   LayoutType.TensorNDHWC: 'cutlass::layout::TensorNDHWC',
 440:   LayoutType.TensorNCHW: 'cutlass::layout::TensorNCHW',
 441:   LayoutType.TensorNGHWC: 'cutlass::layout::TensorNGHWC',
 442:   LayoutType.TensorNC32HW32: 'cutlass::layout::TensorNCxHWx<32>',
 443:   LayoutType.TensorC32RSK32: 'cutlass::layout::TensorCxRSKx<32>',
 444:   LayoutType.TensorNC64HW64: 'cutlass::layout::TensorNCxHWx<64>',
 445:   LayoutType.TensorC64RSK64: 'cutlass::layout::TensorCxRSKx<64>',
 446:   LayoutType.TensorKCS: 'cutlass::layout::TensorKCS',
 447:   LayoutType.TensorKCSR: 'cutlass::layout::TensorKCSR',
 448:   LayoutType.TensorKCSRT: 'cutlass::layout::TensorKCSRT'
 449: }
 450: 
 451: #
 452: TransposedLayout = {
 453:   LayoutType.ColumnMajor: LayoutType.RowMajor,
 454:   LayoutType.RowMajor: LayoutType.ColumnMajor,
 455:   LayoutType.ColumnMajorInterleaved2: LayoutType.RowMajorInterleaved2,
 456:   LayoutType.RowMajorInterleaved2: LayoutType.ColumnMajorInterleaved2,
 457:   LayoutType.ColumnMajorInterleaved32: LayoutType.RowMajorInterleaved32,
 458:   LayoutType.RowMajorInterleaved32: LayoutType.ColumnMajorInterleaved32,
 459:   LayoutType.ColumnMajorInterleaved64: LayoutType.RowMajorInterleaved64,
 460:   LayoutType.RowMajorInterleaved64: LayoutType.ColumnMajorInterleaved64,
 461:   LayoutType.TensorNHWC: LayoutType.TensorNHWC
 462: }
 463: 
 464: #
 465: ShortLayoutTypeNames = {
 466:   LayoutType.ColumnMajor: 'n',
 467:   LayoutType.ColumnMajorInterleaved2: 'n2',
 468:   LayoutType.ColumnMajorInterleaved32: 'n32',
 469:   LayoutType.ColumnMajorInterleaved64: 'n64',
 470:   LayoutType.RowMajor: 't',
 471:   LayoutType.RowMajorInterleaved2: 't2',
 472:   LayoutType.RowMajorInterleaved32: 't32',
 473:   LayoutType.RowMajorInterleaved64: 't64',
 474:   LayoutType.TensorNWC: 'nwc',
 475:   LayoutType.TensorNHWC: 'nhwc',
 476:   LayoutType.TensorNDHWC: 'ndhwc',
 477:   LayoutType.TensorNCHW: 'nchw',
 478:   LayoutType.TensorNGHWC: 'nghwc',
 479:   LayoutType.TensorNC32HW32: 'nc32hw32',
 480:   LayoutType.TensorNC64HW64: 'nc64hw64',
 481:   LayoutType.TensorC32RSK32: 'c32rsk32',
 482:   LayoutType.TensorC64RSK64: 'c64rsk64',
 483:   LayoutType.TensorKCS: 'kcs',
 484:   LayoutType.TensorKCSR: 'kcsr',
 485:   LayoutType.TensorKCSRT: 'kcsrt'
 486: }
 487: 
 488: #
 489: ShortComplexLayoutNames = {
 490:   (LayoutType.ColumnMajor, ComplexTransform.none): 'n',
 491:   (LayoutType.ColumnMajor, ComplexTransform.conj): 'c',
 492:   (LayoutType.RowMajor, ComplexTransform.none): 't',
 493:   (LayoutType.RowMajor, ComplexTransform.conj): 'h'
 494: }
```
**EN:** Defines or updates module/class-level data such as LayoutTag, TransposedLayout, ShortLayoutTypeNames, ShortComplexLayoutNames; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 LayoutTag, TransposedLayout, ShortLayoutTypeNames, ShortComplexLayoutNames；这些值会被后续生成器与 emitter 引用。

### L495-L496 — Comments

```python
 495: 
 496: ###################################################################################################
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L497-L497 — Class `KernelScheduleType`

```python
 497: class KernelScheduleType(enum.Enum):
```
**EN:** Introduces class `KernelScheduleType`, which packages the kernel schedule type logic into a reusable type.

**CN:** 引入类 `KernelScheduleType`，将 kernel schedule type 相关逻辑封装为可复用类型。

### L498-L601 — Data definitions

```python
 498:   ScheduleAuto = enum_auto()
 499:   Multistage = enum_auto()
 500:   CpAsyncWarpSpecialized = enum_auto()
 501:   CpAsyncWarpSpecializedPingpong = enum_auto()
 502:   CpAsyncWarpSpecializedCooperative = enum_auto()
 503:   Tma = enum_auto()
 504:   TmaWarpSpecialized = enum_auto()
 505:   TmaWarpSpecializedPingpong = enum_auto()
 506:   TmaWarpSpecializedCooperative = enum_auto()
 507:   TmaWarpSpecializedFP8FastAccum = enum_auto()
 508:   TmaWarpSpecializedCooperativeFP8FastAccum = enum_auto()
 509:   TmaWarpSpecializedPingpongFP8FastAccum = enum_auto()
 510:   ImplicitTmaWarpSpecializedSm90 = enum_auto()
 511:   PtrArrayTmaWarpSpecializedCooperative = enum_auto()
 512:   PtrArrayTmaWarpSpecializedCooperativeFP8FastAccum = enum_auto()
 513:   PtrArrayTmaWarpSpecializedPingpong = enum_auto()
 514:   PtrArrayTmaWarpSpecializedPingpongFP8FastAccum = enum_auto()
 515: 
 516:   BlockwiseTmaWarpSpecializedCooperative = enum_auto()
 517:   PtrArrayBlockwiseTmaWarpSpecializedCooperative = enum_auto()
 518:   BlockwiseTmaWarpSpecializedPingpong = enum_auto()
 519:   PtrArrayBlockwiseTmaWarpSpecializedPingpong = enum_auto()
 520: 
 521:   TmaWarpSpecialized1SmSm100 = enum_auto()
 522:   TmaWarpSpecialized2SmSm100 = enum_auto()
 523:   ImplicitTmaWarpSpecialized1SmSm100 = enum_auto()
 524:   ImplicitTmaWarpSpecialized2SmSm100 = enum_auto()
 525: 
 526:   PtrArrayTmaWarpSpecialized1SmSm100 = enum_auto()
 527:   PtrArrayTmaWarpSpecialized2SmSm100 = enum_auto()
 528: 
 529:   PtrArrayTmaWarpSpecialized1SmBlockScaledSm100 = enum_auto()
 530:   PtrArrayTmaWarpSpecialized2SmBlockScaledSm100 = enum_auto()
 531:   PtrArrayNvf4TmaWarpSpecialized1SmSm100 = enum_auto()
 532:   PtrArrayNvf4TmaWarpSpecialized2SmSm100 = enum_auto()
 533:   PtrArrayMxf4TmaWarpSpecialized1SmSm100 = enum_auto()
 534:   PtrArrayMxf4TmaWarpSpecialized2SmSm100 = enum_auto()
 535:   PtrArrayMxf8f6f4TmaWarpSpecialized1SmSm100 = enum_auto()
 536:   PtrArrayMxf8f6f4TmaWarpSpecialized2SmSm100 = enum_auto()
 537: 
 538:   SparseTmaWarpSpecialized1SmSm100 = enum_auto()
 539:   SparseTmaWarpSpecialized2SmSm100 = enum_auto()
 540: 
 541:   BlockScaledTmaWarpSpecialized1SmSm100 = enum_auto()
 542:   BlockScaledTmaWarpSpecialized2SmSm100 = enum_auto()
 543:   Mxf8f6f4TmaWarpSpecialized1SmSm100 = enum_auto()
 544:   Mxf8f6f4TmaWarpSpecialized2SmSm100 = enum_auto()
 545: 
 546:   BlockwiseTmaWarpSpecialized1SmSm100 = enum_auto()
 547:   BlockwiseTmaWarpSpecialized2SmSm100 = enum_auto()
 548: 
 549:   PtrArrayBlockwiseTmaWarpSpecialized1SmSm100 = enum_auto()
 550:   PtrArrayBlockwiseTmaWarpSpecialized2SmSm100 = enum_auto()
 551: 
 552: 
 553:   Mxf4TmaWarpSpecialized1SmSm100 = enum_auto()
 554:   Mxf4TmaWarpSpecialized2SmSm100 = enum_auto()
 555:   Nvf4TmaWarpSpecialized1SmSm100 = enum_auto()
 556:   Nvf4TmaWarpSpecialized2SmSm100 = enum_auto()
 557: 
 558:   # FP4 Ultra
 559:   MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103 = enum_auto()
 560:   MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103 = enum_auto()
 561:   MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103 = enum_auto()
 562:   MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103 = enum_auto()
 563: 
 564:   MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103DisablePrefetch = enum_auto()
 565:   MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103DisablePrefetch = enum_auto()
 566:   MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103DisablePrefetch = enum_auto()
 567:   MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103DisablePrefetch = enum_auto()
 568: 
 569:   MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103TmaPrefetch = enum_auto()
 570:   MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103TmaPrefetch = enum_auto()
 571:   MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103TmaPrefetch = enum_auto()
 572:   MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103TmaPrefetch = enum_auto()
 573: 
 574:   PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs16Sm103 = enum_auto()
 575:   PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs16Sm103 = enum_auto()
 576:   PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs32Sm103 = enum_auto()
 577:   PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs32Sm103 = enum_auto()
 578: 
 579:   PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs16Sm103DisablePrefetch = enum_auto()
 580:   PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs16Sm103DisablePrefetch = enum_auto()
 581:   PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs32Sm103DisablePrefetch = enum_auto()
 582:   PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs32Sm103DisablePrefetch = enum_auto()
 583: 
 584:   PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs16Sm103TmaPrefetch = enum_auto()
 585:   PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs16Sm103TmaPrefetch = enum_auto()
 586:   PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs32Sm103TmaPrefetch = enum_auto()
 587:   PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs32Sm103TmaPrefetch = enum_auto()
 588: 
 589:   Mxf8f6f4TmaWarpSpecializedCooperativeSm120 = enum_auto()
 590:   Mxf8f6f4TmaWarpSpecializedPingpongSm120 = enum_auto()
 591:   Nvf4TmaWarpSpecializedCooperativeSm120 = enum_auto()
 592:   Nvf4TmaWarpSpecializedPingpongSm120 = enum_auto()
 593:   Mxf4TmaWarpSpecializedCooperativeSm120 = enum_auto()
 594:   Mxf4TmaWarpSpecializedPingpongSm120 = enum_auto()
 595: 
 596:   F8f6f4SparseTmaWarpSpecializedCooperativeSm120 = enum_auto()
 597: 
 598:   BlockwiseTmaWarpSpecializedCooperativeSm120 = enum_auto()
 599:   BlockwiseTmaWarpSpecializedPingpongSm120 = enum_auto()
 600: 
 601:   XeCooperative = enum_auto()
```
**EN:** Defines or updates module/class-level data such as ScheduleAuto, Multistage, CpAsyncWarpSpecialized, CpAsyncWarpSpecializedPingpong, CpAsyncWarpSpecializedCooperative, Tma, TmaWarpSpecialized, TmaWarpSpecializedPingpong, ...; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 ScheduleAuto, Multistage, CpAsyncWarpSpecialized, CpAsyncWarpSpecializedPingpong, CpAsyncWarpSpecializedCooperative, Tma, TmaWarpSpecialized, TmaWarpSpecializedPingpong, ...；这些值会被后续生成器与 emitter 引用。

### L603-L816 — Data definitions

```python
 603: KernelScheduleTag = {
 604:   KernelScheduleType.ScheduleAuto: 'cutlass::gemm::collective::KernelScheduleAuto',
 605:   KernelScheduleType.Multistage: 'cutlass::gemm::KernelMultistage',
 606:   KernelScheduleType.CpAsyncWarpSpecialized: 'cutlass::gemm::KernelCpAsyncWarpSpecialized',
 607:   KernelScheduleType.CpAsyncWarpSpecializedPingpong: 'cutlass::gemm::KernelCpAsyncWarpSpecializedPingpong',
 608:   KernelScheduleType.CpAsyncWarpSpecializedCooperative: 'cutlass::gemm::KernelCpAsyncWarpSpecializedCooperative',
 609:   KernelScheduleType.Tma: 'cutlass::gemm::KernelTma',
 610:   KernelScheduleType.TmaWarpSpecialized: 'cutlass::gemm::KernelTmaWarpSpecialized',
 611:   KernelScheduleType.TmaWarpSpecializedPingpong: 'cutlass::gemm::KernelTmaWarpSpecializedPingpong',
 612:   KernelScheduleType.TmaWarpSpecializedCooperative: 'cutlass::gemm::KernelTmaWarpSpecializedCooperative',
 613:   KernelScheduleType.TmaWarpSpecializedFP8FastAccum: 'cutlass::gemm::KernelTmaWarpSpecializedFP8FastAccum',
 614:   KernelScheduleType.TmaWarpSpecializedCooperativeFP8FastAccum: 'cutlass::gemm::KernelTmaWarpSpecializedCooperativeFP8FastAccum',
 615:   KernelScheduleType.TmaWarpSpecializedPingpongFP8FastAccum: 'cutlass::gemm::KernelTmaWarpSpecializedPingpongFP8FastAccum',
 616:   KernelScheduleType.ImplicitTmaWarpSpecializedSm90: 'cutlass::conv::KernelImplicitTmaWarpSpecializedSm90',
 617: 
 618:   KernelScheduleType.BlockwiseTmaWarpSpecializedCooperative: 'cutlass::gemm::KernelTmaWarpSpecializedCooperativeFP8Blockwise',
 619:   KernelScheduleType.BlockwiseTmaWarpSpecializedPingpong: 'cutlass::gemm::KernelTmaWarpSpecializedPingpongFP8Blockwise',
 620: 
 621:   KernelScheduleType.TmaWarpSpecialized1SmSm100: 'cutlass::gemm::KernelTmaWarpSpecialized1SmSm100',
 622:   KernelScheduleType.TmaWarpSpecialized2SmSm100: 'cutlass::gemm::KernelTmaWarpSpecialized2SmSm100',
 623: 
 624:   KernelScheduleType.ImplicitTmaWarpSpecialized1SmSm100: 'cutlass::conv::KernelImplicitTmaWarpSpecialized1SmSm100',
 625:   KernelScheduleType.ImplicitTmaWarpSpecialized2SmSm100: 'cutlass::conv::KernelImplicitTmaWarpSpecialized2SmSm100',
 626: 
 627:   KernelScheduleType.PtrArrayTmaWarpSpecialized1SmSm100: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmSm100',
 628:   KernelScheduleType.PtrArrayTmaWarpSpecialized2SmSm100: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmSm100',
 629: 
 630:   KernelScheduleType.SparseTmaWarpSpecialized1SmSm100: 'cutlass::gemm::KernelSparseTmaWarpSpecialized1SmSm100',
 631:   KernelScheduleType.SparseTmaWarpSpecialized2SmSm100: 'cutlass::gemm::KernelSparseTmaWarpSpecialized2SmSm100',
 632: 
 633:   KernelScheduleType.BlockScaledTmaWarpSpecialized1SmSm100: 'cutlass::gemm::KernelTmaWarpSpecialized1SmBlockScaledSm100',
 634:   KernelScheduleType.BlockScaledTmaWarpSpecialized2SmSm100: 'cutlass::gemm::KernelTmaWarpSpecialized2SmBlockScaledSm100',
 635:   KernelScheduleType.Mxf8f6f4TmaWarpSpecialized1SmSm100: 'cutlass::gemm::KernelTmaWarpSpecialized1SmMxf8f6f4Sm100',
 636:   KernelScheduleType.Mxf8f6f4TmaWarpSpecialized2SmSm100: 'cutlass::gemm::KernelTmaWarpSpecialized2SmMxf8f6f4Sm100',
 637: 
 638:   KernelScheduleType.BlockwiseTmaWarpSpecialized1SmSm100: 'cutlass::gemm::KernelTmaWarpSpecializedBlockwise1SmSm100',
 639:   KernelScheduleType.BlockwiseTmaWarpSpecialized2SmSm100: 'cutlass::gemm::KernelTmaWarpSpecializedBlockwise2SmSm100',
 640: 
 641:   KernelScheduleType.PtrArrayBlockwiseTmaWarpSpecialized1SmSm100: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecializedBlockwise1SmSm100',
 642:   KernelScheduleType.PtrArrayBlockwiseTmaWarpSpecialized2SmSm100: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecializedBlockwise2SmSm100',
 643: 
 644:   KernelScheduleType.Mxf4TmaWarpSpecialized1SmSm100: 'cutlass::gemm::KernelTmaWarpSpecialized1SmMxf4Sm100',
 645:   KernelScheduleType.Mxf4TmaWarpSpecialized2SmSm100: 'cutlass::gemm::KernelTmaWarpSpecialized2SmMxf4Sm100',
 646:   KernelScheduleType.Nvf4TmaWarpSpecialized1SmSm100: 'cutlass::gemm::KernelTmaWarpSpecialized1SmNvf4Sm100',
 647:   KernelScheduleType.Nvf4TmaWarpSpecialized2SmSm100: 'cutlass::gemm::KernelTmaWarpSpecialized2SmNvf4Sm100',
 648: 
 649:   # FP4 Ultra
 650:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103: 'cutlass::gemm::KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103',
 651:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103: 'cutlass::gemm::KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103',
 652:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103: 'cutlass::gemm::KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103',
 653:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103: 'cutlass::gemm::KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103',
 654:   
 655:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103TmaPrefetch: 'cutlass::gemm::KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch',
 656:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103TmaPrefetch: 'cutlass::gemm::KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch',
 657:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103TmaPrefetch: 'cutlass::gemm::KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103TmaPrefetch',
 658:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103TmaPrefetch: 'cutlass::gemm::KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103TmaPrefetch',
 659: 
 660:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103DisablePrefetch: 'cutlass::gemm::KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103DisablePrefetch',
 661:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103DisablePrefetch: 'cutlass::gemm::KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103DisablePrefetch',
 662:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103DisablePrefetch: 'cutlass::gemm::KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103DisablePrefetch',
 663:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103DisablePrefetch: 'cutlass::gemm::KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103DisablePrefetch',
 664:   
 665:   KernelScheduleType.PtrArrayTmaWarpSpecializedCooperative: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperative',
 666:   KernelScheduleType.PtrArrayTmaWarpSpecializedCooperativeFP8FastAccum: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperativeFP8FastAccum',
 667:   KernelScheduleType.PtrArrayTmaWarpSpecializedPingpong: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpong',
 668:   KernelScheduleType.PtrArrayTmaWarpSpecializedPingpongFP8FastAccum: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpongFP8FastAccum',
 669: 
 670:   KernelScheduleType.PtrArrayBlockwiseTmaWarpSpecializedCooperative: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperativeFP8Blockwise',
 671:   KernelScheduleType.PtrArrayBlockwiseTmaWarpSpecializedPingpong: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpongFP8Blockwise',
 672: 
 673:   KernelScheduleType.PtrArrayTmaWarpSpecialized1SmBlockScaledSm100: "cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmBlockScaledSm100",
 674:   KernelScheduleType.PtrArrayTmaWarpSpecialized2SmBlockScaledSm100: "cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmBlockScaledSm100",
 675:   KernelScheduleType.PtrArrayNvf4TmaWarpSpecialized1SmSm100: "cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmNvf4Sm100",
 676:   KernelScheduleType.PtrArrayNvf4TmaWarpSpecialized2SmSm100: "cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmNvf4Sm100",
 677:   KernelScheduleType.PtrArrayMxf4TmaWarpSpecialized1SmSm100: "cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmMxf4Sm100",
 678:   KernelScheduleType.PtrArrayMxf4TmaWarpSpecialized2SmSm100: "cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmMxf4Sm100",
 679:   KernelScheduleType.PtrArrayMxf8f6f4TmaWarpSpecialized1SmSm100: "cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmMxf8f6f4Sm100",
 680:   KernelScheduleType.PtrArrayMxf8f6f4TmaWarpSpecialized2SmSm100: "cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmMxf8f6f4Sm100",
 681: 
 682:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs16Sm103: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103',
 683:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs16Sm103: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103',
 684:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs32Sm103: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103',
 685:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs32Sm103: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103',
 686:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs16Sm103TmaPrefetch: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch',
 687:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs16Sm103TmaPrefetch: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch',
 688:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs32Sm103TmaPrefetch: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103TmaPrefetch',
 689:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs32Sm103TmaPrefetch: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103TmaPrefetch',
 690:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs16Sm103DisablePrefetch: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103DisablePrefetch',
 691:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs16Sm103DisablePrefetch: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103DisablePrefetch',
 692:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs32Sm103DisablePrefetch: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103DisablePrefetch',
 693:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs32Sm103DisablePrefetch: 'cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103DisablePrefetch',
 694: 
 695:   KernelScheduleType.Mxf8f6f4TmaWarpSpecializedCooperativeSm120: 'cutlass::gemm::KernelTmaWarpSpecializedMxf8f6f4Sm120',
 696:   KernelScheduleType.Mxf8f6f4TmaWarpSpecializedPingpongSm120: 'cutlass::gemm::KernelTmaWarpSpecializedPingpongMxf8f6f4Sm120',
 697:   KernelScheduleType.Nvf4TmaWarpSpecializedCooperativeSm120: 'cutlass::gemm::KernelTmaWarpSpecializedNvf4Sm120',
 698:   KernelScheduleType.Nvf4TmaWarpSpecializedPingpongSm120: 'cutlass::gemm::KernelTmaWarpSpecializedPingpongNvf4Sm120',
 699:   KernelScheduleType.Mxf4TmaWarpSpecializedCooperativeSm120: 'cutlass::gemm::KernelTmaWarpSpecializedMxf4Sm120',
 700:   KernelScheduleType.Mxf4TmaWarpSpecializedPingpongSm120: 'cutlass::gemm::KernelTmaWarpSpecializedPingpongMxf4Sm120',
 701: 
 702:   KernelScheduleType.F8f6f4SparseTmaWarpSpecializedCooperativeSm120: 'cutlass::gemm::KernelScheduleSparseF8f6f4Sm120',
 703: 
 704:   KernelScheduleType.BlockwiseTmaWarpSpecializedCooperativeSm120: 'cutlass::gemm::KernelTmaWarpSpecializedBlockwiseCooperativeSm120',
 705:   KernelScheduleType.BlockwiseTmaWarpSpecializedPingpongSm120: 'cutlass::gemm::KernelTmaWarpSpecializedBlockwisePingpongSm120',
 706: 
 707:   KernelScheduleType.XeCooperative: 'cutlass::gemm::KernelXeCooperative',
 708: }
 709: 
 710: #
 711: KernelScheduleSuffixes = {
 712:   KernelScheduleType.ScheduleAuto: '',
 713:   KernelScheduleType.Multistage: '_cpasync',
 714:   KernelScheduleType.CpAsyncWarpSpecialized: '_cpasync_warpspecialized',
 715:   KernelScheduleType.CpAsyncWarpSpecializedPingpong: '_cpasync_warpspecialized_pingpong',
 716:   KernelScheduleType.CpAsyncWarpSpecializedCooperative: '_cpasync_warpspecialized_cooperative',
 717:   KernelScheduleType.Tma: '_unspecialized',
 718:   KernelScheduleType.TmaWarpSpecialized: '_warpspecialized',
 719:   KernelScheduleType.TmaWarpSpecializedPingpong: '_warpspecialized_pingpong',
 720:   KernelScheduleType.TmaWarpSpecializedCooperative: '_warpspecialized_cooperative',
 721:   KernelScheduleType.TmaWarpSpecializedFP8FastAccum: '_warpspecialized_fp8_fastaccum',
 722:   KernelScheduleType.TmaWarpSpecializedCooperativeFP8FastAccum: '_warpspecialized_cooperative_fp8_fastaccum',
 723:   KernelScheduleType.TmaWarpSpecializedPingpongFP8FastAccum: '_warpspecialized_pingpong_fp8_fastaccum',
 724:   KernelScheduleType.ImplicitTmaWarpSpecializedSm90: '_warpspecialized',
 725: 
 726:   KernelScheduleType.BlockwiseTmaWarpSpecializedCooperative: '_warpspecialized_cooperative',
 727:   KernelScheduleType.BlockwiseTmaWarpSpecializedPingpong: '_warpspecialized_pingpong',
 728: 
 729:   KernelScheduleType.TmaWarpSpecialized1SmSm100: '_1sm',
 730:   KernelScheduleType.TmaWarpSpecialized2SmSm100: '_2sm',
 731: 
 732:   KernelScheduleType.ImplicitTmaWarpSpecialized1SmSm100: '_1sm',
 733:   KernelScheduleType.ImplicitTmaWarpSpecialized2SmSm100: '_2sm',
 734: 
 735:   KernelScheduleType.PtrArrayTmaWarpSpecialized1SmSm100: '_1sm',
 736:   KernelScheduleType.PtrArrayTmaWarpSpecialized2SmSm100: '_2sm',
 737: 
 738:   KernelScheduleType.SparseTmaWarpSpecialized1SmSm100: '_1sm',
 739:   KernelScheduleType.SparseTmaWarpSpecialized2SmSm100: '_2sm',
 740: 
 741:   KernelScheduleType.BlockScaledTmaWarpSpecialized1SmSm100: '_1sm',
 742:   KernelScheduleType.BlockScaledTmaWarpSpecialized2SmSm100: '_2sm',
 743:   KernelScheduleType.Mxf8f6f4TmaWarpSpecialized1SmSm100: '_q_1sm',
 744:   KernelScheduleType.Mxf8f6f4TmaWarpSpecialized2SmSm100: '_q_2sm',
 745: 
 746:   KernelScheduleType.BlockwiseTmaWarpSpecialized1SmSm100: '_1sm',
 747:   KernelScheduleType.BlockwiseTmaWarpSpecialized2SmSm100: '_2sm',
 748:   KernelScheduleType.PtrArrayBlockwiseTmaWarpSpecialized1SmSm100: '_1sm',
 749:   KernelScheduleType.PtrArrayBlockwiseTmaWarpSpecialized2SmSm100: '_2sm',
 750: 
 751:   KernelScheduleType.Mxf4TmaWarpSpecialized1SmSm100: '_o_vs32_1sm',
 752:   KernelScheduleType.Mxf4TmaWarpSpecialized2SmSm100: '_o_vs32_2sm',
 753:   KernelScheduleType.Nvf4TmaWarpSpecialized1SmSm100: '_o_vs16_1sm',
 754:   KernelScheduleType.Nvf4TmaWarpSpecialized2SmSm100: '_o_vs16_2sm',
 755: 
 756:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103: '_o_vs16_ultra_1sm',
 757:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103: '_o_vs16_ultra_2sm',
 758:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103: '_o_vs32_ultra_1sm',
 759:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103: '_o_vs32_ultra_2sm',
 760: 
 761:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103DisablePrefetch: '_o_vs16_ultra_1sm_nopf',
 762:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103DisablePrefetch: '_o_vs16_ultra_2sm_nopf',
 763:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103DisablePrefetch: '_o_vs32_ultra_1sm_nopf',
 764:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103DisablePrefetch: '_o_vs32_ultra_2sm_nopf',
 765: 
 766:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103TmaPrefetch: '_o_vs16_ultra_1sm_tmapf',
 767:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103TmaPrefetch: '_o_vs16_ultra_2sm_tmapf',
 768:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103TmaPrefetch: '_o_vs32_ultra_1sm_tmapf',
 769:   KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103TmaPrefetch: '_o_vs32_ultra_2sm_tmapf',
 770: 
 771:   KernelScheduleType.PtrArrayTmaWarpSpecializedCooperative: '_warpspecialized_cooperative',
 772:   KernelScheduleType.PtrArrayTmaWarpSpecializedCooperativeFP8FastAccum: '_warpspecialized_cooperative_fp8_fastaccum',
 773:   KernelScheduleType.PtrArrayTmaWarpSpecializedPingpong: '_warpspecialized_pingpong',
 774:   KernelScheduleType.PtrArrayTmaWarpSpecializedPingpongFP8FastAccum: '_warpspecialized_pingpong_fp8_fastaccum',
 775: 
 776:   KernelScheduleType.PtrArrayBlockwiseTmaWarpSpecializedCooperative: '_warpspecialized_cooperative',
 777:   KernelScheduleType.PtrArrayBlockwiseTmaWarpSpecializedPingpong: '_warpspecialized_pingpong',
 778: 
 779:   KernelScheduleType.PtrArrayTmaWarpSpecialized1SmBlockScaledSm100: '_1sm',
 780:   KernelScheduleType.PtrArrayTmaWarpSpecialized2SmBlockScaledSm100: '_2sm',
 781:   KernelScheduleType.PtrArrayNvf4TmaWarpSpecialized1SmSm100: '_o_vs16_1sm',
 782:   KernelScheduleType.PtrArrayNvf4TmaWarpSpecialized2SmSm100: '_o_vs16_2sm',
 783:   KernelScheduleType.PtrArrayMxf4TmaWarpSpecialized1SmSm100: '_o_vs32_1sm',
 784:   KernelScheduleType.PtrArrayMxf4TmaWarpSpecialized2SmSm100: '_o_vs32_2sm',
 785:   KernelScheduleType.PtrArrayMxf8f6f4TmaWarpSpecialized1SmSm100: '_o_vs32_1sm',
 786:   KernelScheduleType.PtrArrayMxf8f6f4TmaWarpSpecialized2SmSm100: '_o_vs32_2sm',
 787: 
 788:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs16Sm103: '_o_vs16_ultra_1sm',
 789:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs16Sm103: '_o_vs16_ultra_2sm',
 790:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs32Sm103: '_o_vs32_ultra_1sm',
 791:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs32Sm103: '_o_vs32_ultra_2sm',
 792: 
 793:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs16Sm103DisablePrefetch: '_o_vs16_ultra_1sm_nopf',
 794:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs16Sm103DisablePrefetch: '_o_vs16_ultra_2sm_nopf',
 795:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs32Sm103DisablePrefetch: '_o_vs32_ultra_1sm_nopf',
 796:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs32Sm103DisablePrefetch: '_o_vs32_ultra_2sm_nopf',
 797: 
 798:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs16Sm103TmaPrefetch: '_o_vs16_ultra_1sm_tmapf',
 799:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs16Sm103TmaPrefetch: '_o_vs16_ultra_2sm_tmapf',
 800:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs32Sm103TmaPrefetch: '_o_vs32_ultra_1sm_tmapf',
 801:   KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs32Sm103TmaPrefetch: '_o_vs32_ultra_2sm_tmapf',
 802: 
 803:   KernelScheduleType.Mxf8f6f4TmaWarpSpecializedCooperativeSm120: '_cooperative_q',
 804:   KernelScheduleType.Mxf8f6f4TmaWarpSpecializedPingpongSm120: '_pingpong_q',
 805:   KernelScheduleType.Nvf4TmaWarpSpecializedCooperativeSm120: '_cooperative_o_vs16',
 806:   KernelScheduleType.Nvf4TmaWarpSpecializedPingpongSm120: '_pingpong_o_vs16',
 807:   KernelScheduleType.Mxf4TmaWarpSpecializedCooperativeSm120: '_cooperative_o_vs32',
 808:   KernelScheduleType.Mxf4TmaWarpSpecializedPingpongSm120: '_pingpong_o_vs32',
 809: 
 810:   KernelScheduleType.F8f6f4SparseTmaWarpSpecializedCooperativeSm120: '_q',
 811: 
 812:   KernelScheduleType.BlockwiseTmaWarpSpecializedCooperativeSm120: '_cooperative_q',
 813:   KernelScheduleType.BlockwiseTmaWarpSpecializedPingpongSm120: '_pingpong_q',
 814: 
 815:   KernelScheduleType.XeCooperative: '_xe_cooperative',
 816: }
```
**EN:** Defines or updates module/class-level data such as KernelScheduleTag, KernelScheduleSuffixes; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 KernelScheduleTag, KernelScheduleSuffixes；这些值会被后续生成器与 emitter 引用。

### L818-L818 — Class `EpilogueScheduleType`

```python
 818: class EpilogueScheduleType(enum.Enum):
```
**EN:** Introduces class `EpilogueScheduleType`, which packages the epilogue schedule type logic into a reusable type.

**CN:** 引入类 `EpilogueScheduleType`，将 epilogue schedule type 相关逻辑封装为可复用类型。

### L819-L842 — Data definitions

```python
 819:   ScheduleAuto = enum_auto()
 820:   EpilogueTransposed = enum_auto()
 821:   NoSmemWarpSpecialized = enum_auto()
 822:   PtrArrayNoSmemWarpSpecialized = enum_auto()
 823:   NoSmemWarpSpecialized1Sm = enum_auto()
 824:   NoSmemWarpSpecialized2Sm = enum_auto()
 825:   FastF32NoSmemWarpSpecialized1Sm = enum_auto()
 826:   FastF32NoSmemWarpSpecialized2Sm = enum_auto()
 827:   BlockwiseNoSmemWarpSpecialized1Sm = enum_auto()
 828:   BlockwiseNoSmemWarpSpecialized2Sm = enum_auto()
 829:   PtrArrayNoSmemWarpSpecialized1Sm = enum_auto()
 830:   PtrArrayNoSmemWarpSpecialized2Sm = enum_auto()
 831:   PtrArrayFastF32NoSmemWarpSpecialized1Sm = enum_auto()
 832:   PtrArrayFastF32NoSmemWarpSpecialized2Sm = enum_auto()
 833:   PtrArrayBlockwiseNoSmemWarpSpecialized1Sm = enum_auto()
 834:   PtrArrayBlockwiseNoSmemWarpSpecialized2Sm = enum_auto()
 835:   TmaWarpSpecialized = enum_auto()
 836:   TmaWarpSpecializedCooperative = enum_auto()
 837:   TmaWarpSpecialized1Sm = enum_auto() 
 838:   TmaWarpSpecialized2Sm = enum_auto() 
 839:   PtrArrayTmaWarpSpecialized1Sm = enum_auto()
 840:   PtrArrayTmaWarpSpecialized2Sm = enum_auto()
 841:   PtrArrayTmaWarpSpecializedPingpong = enum_auto()
 842:   PtrArrayTmaWarpSpecializedCooperative = enum_auto()
```
**EN:** Defines or updates module/class-level data such as ScheduleAuto, EpilogueTransposed, NoSmemWarpSpecialized, PtrArrayNoSmemWarpSpecialized, NoSmemWarpSpecialized1Sm, NoSmemWarpSpecialized2Sm, FastF32NoSmemWarpSpecialized1Sm, FastF32NoSmemWarpSpecialized2Sm, ...; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 ScheduleAuto, EpilogueTransposed, NoSmemWarpSpecialized, PtrArrayNoSmemWarpSpecialized, NoSmemWarpSpecialized1Sm, NoSmemWarpSpecialized2Sm, FastF32NoSmemWarpSpecialized1Sm, FastF32NoSmemWarpSpecialized2Sm, ...；这些值会被后续生成器与 emitter 引用。

### L843-L844 — Comments

```python
 843: 
 844: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L845-L898 — Data definitions

```python
 845: EpilogueScheduleTag = {
 846:   EpilogueScheduleType.ScheduleAuto: 'cutlass::epilogue::collective::EpilogueScheduleAuto',
 847:   EpilogueScheduleType.EpilogueTransposed: 'cutlass::gemm::EpilogueTransposed',
 848:   EpilogueScheduleType.NoSmemWarpSpecialized: 'cutlass::epilogue::NoSmemWarpSpecialized',
 849:   EpilogueScheduleType.PtrArrayNoSmemWarpSpecialized: 'cutlass::epilogue::PtrArrayNoSmemWarpSpecialized',
 850:   EpilogueScheduleType.NoSmemWarpSpecialized1Sm: 'cutlass::epilogue::NoSmemWarpSpecialized1Sm',
 851:   EpilogueScheduleType.NoSmemWarpSpecialized2Sm: 'cutlass::epilogue::NoSmemWarpSpecialized2Sm',
 852:   EpilogueScheduleType.FastF32NoSmemWarpSpecialized1Sm: 'cutlass::epilogue::FastF32NoSmemWarpSpecialized1Sm',
 853:   EpilogueScheduleType.FastF32NoSmemWarpSpecialized2Sm: 'cutlass::epilogue::FastF32NoSmemWarpSpecialized2Sm',
 854:   EpilogueScheduleType.BlockwiseNoSmemWarpSpecialized1Sm: 'cutlass::epilogue::BlockwiseNoSmemWarpSpecialized1Sm',
 855:   EpilogueScheduleType.BlockwiseNoSmemWarpSpecialized2Sm: 'cutlass::epilogue::BlockwiseNoSmemWarpSpecialized2Sm',
 856:   EpilogueScheduleType.PtrArrayNoSmemWarpSpecialized1Sm: 'cutlass::epilogue::PtrArrayNoSmemWarpSpecialized1Sm',
 857:   EpilogueScheduleType.PtrArrayNoSmemWarpSpecialized2Sm: 'cutlass::epilogue::PtrArrayNoSmemWarpSpecialized2Sm',
 858:   EpilogueScheduleType.PtrArrayFastF32NoSmemWarpSpecialized1Sm: 'cutlass::epilogue::PtrArrayFastF32NoSmemWarpSpecialized1Sm',
 859:   EpilogueScheduleType.PtrArrayFastF32NoSmemWarpSpecialized2Sm: 'cutlass::epilogue::PtrArrayFastF32NoSmemWarpSpecialized2Sm',
 860:   EpilogueScheduleType.PtrArrayBlockwiseNoSmemWarpSpecialized1Sm: 'cutlass::epilogue::PtrArrayBlockwiseNoSmemWarpSpecialized1Sm',
 861:   EpilogueScheduleType.PtrArrayBlockwiseNoSmemWarpSpecialized2Sm: 'cutlass::epilogue::PtrArrayBlockwiseNoSmemWarpSpecialized2Sm',
 862:   EpilogueScheduleType.TmaWarpSpecialized: 'cutlass::epilogue::TmaWarpSpecialized',
 863:   EpilogueScheduleType.TmaWarpSpecializedCooperative: 'cutlass::epilogue::TmaWarpSpecializedCooperative',
 864:   EpilogueScheduleType.TmaWarpSpecialized1Sm: 'cutlass::epilogue::TmaWarpSpecialized1Sm', 
 865:   EpilogueScheduleType.TmaWarpSpecialized2Sm: 'cutlass::epilogue::TmaWarpSpecialized2Sm', 
 866:   EpilogueScheduleType.PtrArrayTmaWarpSpecialized1Sm: 'cutlass::epilogue::PtrArrayTmaWarpSpecialized1Sm',
 867:   EpilogueScheduleType.PtrArrayTmaWarpSpecialized2Sm: 'cutlass::epilogue::PtrArrayTmaWarpSpecialized2Sm',
 868:   EpilogueScheduleType.PtrArrayTmaWarpSpecializedCooperative: 'cutlass::epilogue::PtrArrayTmaWarpSpecializedCooperative',
 869:   EpilogueScheduleType.PtrArrayTmaWarpSpecializedPingpong: 'cutlass::epilogue::PtrArrayTmaWarpSpecializedPingpong',
 870: }
 871: 
 872: #
 873: EpilogueScheduleSuffixes = {
 874:   EpilogueScheduleType.ScheduleAuto: '',
 875:   EpilogueScheduleType.EpilogueTransposed: '',
 876:   EpilogueScheduleType.NoSmemWarpSpecialized: '_epi_nosmem',
 877:   EpilogueScheduleType.PtrArrayNoSmemWarpSpecialized: '_epi_nosmem',
 878:   EpilogueScheduleType.NoSmemWarpSpecialized1Sm: '_epi_nosmem',
 879:   EpilogueScheduleType.NoSmemWarpSpecialized2Sm: '_epi_nosmem',
 880:   EpilogueScheduleType.FastF32NoSmemWarpSpecialized1Sm: '_epi_nosmem_fastf32',
 881:   EpilogueScheduleType.FastF32NoSmemWarpSpecialized2Sm: '_epi_nosmem_fastf32',
 882:   EpilogueScheduleType.BlockwiseNoSmemWarpSpecialized1Sm: '_epi_nosmem',
 883:   EpilogueScheduleType.BlockwiseNoSmemWarpSpecialized2Sm: '_epi_nosmem',
 884:   EpilogueScheduleType.PtrArrayNoSmemWarpSpecialized1Sm: '_epi_nosmem',
 885:   EpilogueScheduleType.PtrArrayNoSmemWarpSpecialized2Sm: '_epi_nosmem',
 886:   EpilogueScheduleType.PtrArrayFastF32NoSmemWarpSpecialized1Sm: '_epi_nosmem_fastf32',
 887:   EpilogueScheduleType.PtrArrayFastF32NoSmemWarpSpecialized2Sm: '_epi_nosmem_fastf32',
 888:   EpilogueScheduleType.PtrArrayBlockwiseNoSmemWarpSpecialized1Sm: '_epi_nosmem',
 889:   EpilogueScheduleType.PtrArrayBlockwiseNoSmemWarpSpecialized2Sm: '_epi_nosmem',
 890:   EpilogueScheduleType.TmaWarpSpecialized: '_epi_tma',
 891:   EpilogueScheduleType.TmaWarpSpecializedCooperative: '_epi_tma',
 892:   EpilogueScheduleType.TmaWarpSpecialized1Sm: '', 
 893:   EpilogueScheduleType.TmaWarpSpecialized2Sm: '_epi_tma', 
 894:   EpilogueScheduleType.PtrArrayTmaWarpSpecialized1Sm: '',
 895:   EpilogueScheduleType.PtrArrayTmaWarpSpecialized2Sm: '_epi_tma',
 896:   EpilogueScheduleType.PtrArrayTmaWarpSpecializedCooperative: '_epi_tma',
 897:   EpilogueScheduleType.PtrArrayTmaWarpSpecializedPingpong: '_epi_tma',
 898: }
```
**EN:** Defines or updates module/class-level data such as EpilogueScheduleTag, EpilogueScheduleSuffixes; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 EpilogueScheduleTag, EpilogueScheduleSuffixes；这些值会被后续生成器与 emitter 引用。

### L900-L900 — Class `EpilogueFunctor3x`

```python
 900: class EpilogueFunctor3x(enum.Enum):
```
**EN:** Introduces class `EpilogueFunctor3x`, which packages the epilogue functor3x logic into a reusable type.

**CN:** 引入类 `EpilogueFunctor3x`，将 epilogue functor3x 相关逻辑封装为可复用类型。

### L901-L902 — Data definitions

```python
 901:   LinearCombination = enum_auto()
 902:   LinearCombinationBlockScaleFactor = enum_auto() 
```
**EN:** Defines or updates module/class-level data such as LinearCombination, LinearCombinationBlockScaleFactor; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 LinearCombination, LinearCombinationBlockScaleFactor；这些值会被后续生成器与 emitter 引用。

### L903-L904 — Comments

```python
 903: 
 904: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L905-L908 — Data definitions

```python
 905: EpilogueFunctor3xTag = {
 906:   EpilogueFunctor3x.LinearCombination: 'cutlass::epilogue::fusion::LinearCombination',
 907:   EpilogueFunctor3x.LinearCombinationBlockScaleFactor: 'cutlass::epilogue::fusion::LinCombBlockScaleFactor',  
 908: }
```
**EN:** Defines or updates module/class-level data such as EpilogueFunctor3xTag; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 EpilogueFunctor3xTag；这些值会被后续生成器与 emitter 引用。

### L909-L910 — Comments

```python
 909: 
 910: # TMA epilogues have certain alignment requirements as calculated in get_tma_alignment(data_type)
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L911-L922 — Function `is_tma_epilogue`

```python
 911: def is_tma_epilogue(epilogue_schedule_type):
 912:   return epilogue_schedule_type in [
 913:     EpilogueScheduleType.ScheduleAuto,
 914:     EpilogueScheduleType.TmaWarpSpecialized,
 915:     EpilogueScheduleType.TmaWarpSpecializedCooperative,
 916:     EpilogueScheduleType.TmaWarpSpecialized1Sm,
 917:     EpilogueScheduleType.TmaWarpSpecialized2Sm,
 918:     EpilogueScheduleType.PtrArrayTmaWarpSpecialized1Sm,
 919:     EpilogueScheduleType.PtrArrayTmaWarpSpecialized2Sm,
 920:     EpilogueScheduleType.PtrArrayTmaWarpSpecializedCooperative,
 921:     EpilogueScheduleType.PtrArrayTmaWarpSpecializedPingpong,
 922:   ]
```
**EN:** Defines `is_tma_epilogue()`, a predicate that checks whether tma epilogue.

**CN:** 定义 `is_tma_epilogue()`，用于判断是否满足 tma epilogue 这一条件。

### L924-L971 — Function `to_grouped_schedule`

```python
 924: def to_grouped_schedule(schedule, grouped):
 925:   if not grouped:
 926:     return schedule
 927: 
 928:   group_schedule_map = {
 929:     # SM90
 930:     KernelScheduleType.TmaWarpSpecializedCooperative : KernelScheduleType.PtrArrayTmaWarpSpecializedCooperative,
 931:     KernelScheduleType.BlockwiseTmaWarpSpecializedCooperative : KernelScheduleType.PtrArrayBlockwiseTmaWarpSpecializedCooperative,
 932:     KernelScheduleType.BlockwiseTmaWarpSpecializedPingpong : KernelScheduleType.PtrArrayBlockwiseTmaWarpSpecializedPingpong,
 933:     KernelScheduleType.TmaWarpSpecializedPingpong    : KernelScheduleType.PtrArrayTmaWarpSpecializedPingpong,
 934:     KernelScheduleType.TmaWarpSpecializedCooperativeFP8FastAccum : KernelScheduleType.PtrArrayTmaWarpSpecializedCooperativeFP8FastAccum,
 935:     KernelScheduleType.TmaWarpSpecializedPingpongFP8FastAccum    : KernelScheduleType.PtrArrayTmaWarpSpecializedPingpongFP8FastAccum,
 936:     EpilogueScheduleType.TmaWarpSpecialized            : EpilogueScheduleType.PtrArrayTmaWarpSpecializedPingpong,
 937:     EpilogueScheduleType.TmaWarpSpecializedCooperative : EpilogueScheduleType.PtrArrayTmaWarpSpecializedCooperative,
 938:     EpilogueScheduleType.NoSmemWarpSpecialized         : EpilogueScheduleType.PtrArrayNoSmemWarpSpecialized,
 939:     # SM100
 940:     KernelScheduleType.TmaWarpSpecialized1SmSm100: KernelScheduleType.PtrArrayTmaWarpSpecialized1SmSm100,
 941:     KernelScheduleType.TmaWarpSpecialized2SmSm100: KernelScheduleType.PtrArrayTmaWarpSpecialized2SmSm100,
 942:     KernelScheduleType.Nvf4TmaWarpSpecialized1SmSm100 : KernelScheduleType.PtrArrayNvf4TmaWarpSpecialized1SmSm100,
 943:     KernelScheduleType.Nvf4TmaWarpSpecialized2SmSm100 : KernelScheduleType.PtrArrayNvf4TmaWarpSpecialized2SmSm100,
 944:     KernelScheduleType.Mxf4TmaWarpSpecialized1SmSm100 : KernelScheduleType.PtrArrayMxf4TmaWarpSpecialized1SmSm100,
 945:     KernelScheduleType.Mxf4TmaWarpSpecialized2SmSm100 : KernelScheduleType.PtrArrayMxf4TmaWarpSpecialized2SmSm100,
 946:     KernelScheduleType.Mxf8f6f4TmaWarpSpecialized1SmSm100 : KernelScheduleType.PtrArrayMxf8f6f4TmaWarpSpecialized1SmSm100,
 947:     KernelScheduleType.Mxf8f6f4TmaWarpSpecialized2SmSm100 : KernelScheduleType.PtrArrayMxf8f6f4TmaWarpSpecialized2SmSm100,
 948:     KernelScheduleType.BlockwiseTmaWarpSpecialized1SmSm100 : KernelScheduleType.PtrArrayBlockwiseTmaWarpSpecialized1SmSm100,
 949:     KernelScheduleType.BlockwiseTmaWarpSpecialized2SmSm100 : KernelScheduleType.PtrArrayBlockwiseTmaWarpSpecialized2SmSm100,
 950:     EpilogueScheduleType.TmaWarpSpecialized1Sm: EpilogueScheduleType.PtrArrayTmaWarpSpecialized1Sm,
 951:     EpilogueScheduleType.TmaWarpSpecialized2Sm: EpilogueScheduleType.PtrArrayTmaWarpSpecialized2Sm,
 952:     EpilogueScheduleType.NoSmemWarpSpecialized1Sm: EpilogueScheduleType.PtrArrayNoSmemWarpSpecialized1Sm,
 953:     EpilogueScheduleType.NoSmemWarpSpecialized2Sm: EpilogueScheduleType.PtrArrayNoSmemWarpSpecialized2Sm,
 954:     EpilogueScheduleType.BlockwiseNoSmemWarpSpecialized1Sm: EpilogueScheduleType.PtrArrayBlockwiseNoSmemWarpSpecialized1Sm,
 955:     EpilogueScheduleType.BlockwiseNoSmemWarpSpecialized2Sm: EpilogueScheduleType.PtrArrayBlockwiseNoSmemWarpSpecialized2Sm,
 956:     # SM103
 957:     KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103: KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs16Sm103,
 958:     KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103: KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs16Sm103,
 959:     KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103: KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs32Sm103,
 960:     KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103: KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs32Sm103,
 961:     KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103DisablePrefetch: KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs16Sm103DisablePrefetch,
 962:     KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103DisablePrefetch: KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs16Sm103DisablePrefetch,
 963:     KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103DisablePrefetch: KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs32Sm103DisablePrefetch,
 964:     KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103DisablePrefetch: KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs32Sm103DisablePrefetch,
 965:     KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs16Sm103TmaPrefetch: KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs16Sm103TmaPrefetch,
 966:     KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs16Sm103TmaPrefetch: KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs16Sm103TmaPrefetch,
 967:     KernelScheduleType.MxNvf4UltraTmaWarpSpecialized1SmVs32Sm103TmaPrefetch: KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized1SmVs32Sm103TmaPrefetch,
 968:     KernelScheduleType.MxNvf4UltraTmaWarpSpecialized2SmVs32Sm103TmaPrefetch: KernelScheduleType.PtrArrayMxNvf4UltraTmaWarpSpecialized2SmVs32Sm103TmaPrefetch,
 969:   }
 970: 
 971:   return group_schedule_map[schedule]
```
**EN:** Defines `to_grouped_schedule()`, which implements the to grouped schedule logic.

**CN:** 定义 `to_grouped_schedule()`，用于实现 to grouped schedule 相关逻辑。

### L973-L973 — Class `TileSchedulerType`

```python
 973: class TileSchedulerType(enum.Enum):
```
**EN:** Introduces class `TileSchedulerType`, which packages the tile scheduler type logic into a reusable type.

**CN:** 引入类 `TileSchedulerType`，将 tile scheduler type 相关逻辑封装为可复用类型。

### L974-L976 — Data definitions

```python
 974:   Default = enum_auto()
 975:   Persistent = enum_auto()
 976:   StreamK = enum_auto()
```
**EN:** Defines or updates module/class-level data such as Default, Persistent, StreamK; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Default, Persistent, StreamK；这些值会被后续生成器与 emitter 引用。

### L977-L977 — Comments

```python
 977: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L978-L989 — Data definitions

```python
 978: TileSchedulerTag = {
 979:   TileSchedulerType.Default: 'void',
 980:   TileSchedulerType.Persistent: 'cutlass::gemm::PersistentScheduler',
 981:   TileSchedulerType.StreamK: 'cutlass::gemm::StreamKScheduler',
 982: }
 983: 
 984: #
 985: TileSchedulerSuffixes = {
 986:   TileSchedulerType.Default: '',
 987:   TileSchedulerType.Persistent: '',
 988:   TileSchedulerType.StreamK: '_stream_k',
 989: }
```
**EN:** Defines or updates module/class-level data such as TileSchedulerTag, TileSchedulerSuffixes; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 TileSchedulerTag, TileSchedulerSuffixes；这些值会被后续生成器与 emitter 引用。

### L990-L993 — Comments

```python
 990: 
 991: ###################################################################################################
 992: 
 993: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L994-L994 — Class `SideMode`

```python
 994: class SideMode(enum.Enum):
```
**EN:** Introduces class `SideMode`, which packages the side mode logic into a reusable type.

**CN:** 引入类 `SideMode`，将 side mode 相关逻辑封装为可复用类型。

### L995-L996 — Data definitions

```python
 995:   Left = enum_auto()
 996:   Right = enum_auto()
```
**EN:** Defines or updates module/class-level data such as Left, Right; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Left, Right；这些值会被后续生成器与 emitter 引用。

### L997-L998 — Comments

```python
 997: 
 998: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L999-L1008 — Data definitions

```python
 999: SideModeTag = {
1000:   SideMode.Left: 'cutlass::SideMode::kLeft',
1001:   SideMode.Right: 'cutlass::SideMode::kRight'
1002: }
1003: 
1004: #
1005: ShortSideModeNames = {
1006:   SideMode.Left: 'ls',
1007:   SideMode.Right: 'rs'
1008: }
```
**EN:** Defines or updates module/class-level data such as SideModeTag, ShortSideModeNames; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 SideModeTag, ShortSideModeNames；这些值会被后续生成器与 emitter 引用。

### L1009-L1012 — Comments

```python
1009: 
1010: ###################################################################################################
1011: 
1012: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1013-L1013 — Class `FillMode`

```python
1013: class FillMode(enum.Enum):
```
**EN:** Introduces class `FillMode`, which packages the fill mode logic into a reusable type.

**CN:** 引入类 `FillMode`，将 fill mode 相关逻辑封装为可复用类型。

### L1014-L1015 — Data definitions

```python
1014:   Lower = enum_auto()
1015:   Upper = enum_auto()
```
**EN:** Defines or updates module/class-level data such as Lower, Upper; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Lower, Upper；这些值会被后续生成器与 emitter 引用。

### L1016-L1017 — Comments

```python
1016: 
1017: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1018-L1027 — Data definitions

```python
1018: FillModeTag = {
1019:   FillMode.Lower: 'cutlass::FillMode::kLower',
1020:   FillMode.Upper: 'cutlass::FillMode::kUpper'
1021: }
1022: 
1023: #
1024: ShortFillModeNames = {
1025:   FillMode.Lower: 'l',
1026:   FillMode.Upper: 'u'
1027: }
```
**EN:** Defines or updates module/class-level data such as FillModeTag, ShortFillModeNames; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 FillModeTag, ShortFillModeNames；这些值会被后续生成器与 emitter 引用。

### L1028-L1031 — Comments

```python
1028: 
1029: ###################################################################################################
1030: 
1031: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1032-L1032 — Class `DiagType`

```python
1032: class DiagType(enum.Enum):
```
**EN:** Introduces class `DiagType`, which packages the diag type logic into a reusable type.

**CN:** 引入类 `DiagType`，将 diag type 相关逻辑封装为可复用类型。

### L1033-L1034 — Data definitions

```python
1033:   NonUnit = enum_auto()
1034:   Unit = enum_auto()
```
**EN:** Defines or updates module/class-level data such as NonUnit, Unit; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 NonUnit, Unit；这些值会被后续生成器与 emitter 引用。

### L1035-L1036 — Comments

```python
1035: 
1036: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1037-L1046 — Data definitions

```python
1037: DiagTypeTag = {
1038:   DiagType.NonUnit: 'cutlass::DiagType::kNonUnit',
1039:   DiagType.Unit: 'cutlass::DiagType::kUnit'
1040: }
1041: 
1042: #
1043: ShortDiagTypeNames = {
1044:   DiagType.NonUnit: 'nu',
1045:   DiagType.Unit: 'un'
1046: }
```
**EN:** Defines or updates module/class-level data such as DiagTypeTag, ShortDiagTypeNames; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 DiagTypeTag, ShortDiagTypeNames；这些值会被后续生成器与 emitter 引用。

### L1047-L1050 — Comments

```python
1047: 
1048: ###################################################################################################
1049: 
1050: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1051-L1051 — Class `OpcodeClass`

```python
1051: class OpcodeClass(enum.Enum):
```
**EN:** Introduces class `OpcodeClass`, which packages the opcode class logic into a reusable type.

**CN:** 引入类 `OpcodeClass`，将 opcode class 相关逻辑封装为可复用类型。

### L1052-L1056 — Data definitions

```python
1052:   Simt = enum_auto()
1053:   TensorOp = enum_auto()
1054:   WmmaTensorOp = enum_auto()
1055:   SparseTensorOp = enum_auto()
1056:   BlockScaledTensorOp = enum_auto()                                     
```
**EN:** Defines or updates module/class-level data such as Simt, TensorOp, WmmaTensorOp, SparseTensorOp, BlockScaledTensorOp; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Simt, TensorOp, WmmaTensorOp, SparseTensorOp, BlockScaledTensorOp；这些值会被后续生成器与 emitter 引用。

### L1059-L1073 — Data definitions

```python
1059: OpcodeClassNames = {
1060:   OpcodeClass.Simt: 'simt',
1061:   OpcodeClass.TensorOp: 'tensorop',
1062:   OpcodeClass.WmmaTensorOp: 'wmma_tensorop',
1063:   OpcodeClass.SparseTensorOp: 'sptensorop',
1064:   OpcodeClass.BlockScaledTensorOp: 'bstensorop'                         
1065: }
1066: 
1067: OpcodeClassTag = {
1068:   OpcodeClass.Simt: 'cutlass::arch::OpClassSimt',
1069:   OpcodeClass.TensorOp: 'cutlass::arch::OpClassTensorOp',
1070:   OpcodeClass.WmmaTensorOp: 'cutlass::arch::OpClassWmmaTensorOp',
1071:   OpcodeClass.SparseTensorOp: 'cutlass::arch::OpClassSparseTensorOp',
1072:   OpcodeClass.BlockScaledTensorOp: 'cutlass::arch::OpClassBlockScaledTensorOp'    
1073: }
```
**EN:** Defines or updates module/class-level data such as OpcodeClassNames, OpcodeClassTag; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 OpcodeClassNames, OpcodeClassTag；这些值会被后续生成器与 emitter 引用。

### L1074-L1077 — Comments

```python
1074: 
1075: ###################################################################################################
1076: 
1077: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1078-L1078 — Class `OperationKind`

```python
1078: class OperationKind(enum.Enum):
```
**EN:** Introduces class `OperationKind`, which packages the operation kind logic into a reusable type.

**CN:** 引入类 `OperationKind`，将 operation kind 相关逻辑封装为可复用类型。

### L1079-L1085 — Data definitions

```python
1079:   Gemm = enum_auto()
1080:   RankK = enum_auto()
1081:   Rank2K = enum_auto()
1082:   Trmm = enum_auto()
1083:   Symm = enum_auto()
1084:   Conv2d = enum_auto()
1085:   Conv3d = enum_auto()
```
**EN:** Defines or updates module/class-level data such as Gemm, RankK, Rank2K, Trmm, Symm, Conv2d, Conv3d; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Gemm, RankK, Rank2K, Trmm, Symm, Conv2d, Conv3d；这些值会被后续生成器与 emitter 引用。

### L1086-L1087 — Comments

```python
1086: 
1087: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1088-L1096 — Data definitions

```python
1088: OperationKindNames = {
1089:   OperationKind.Gemm: 'gemm'
1090:   , OperationKind.RankK: 'rank_k'
1091:   , OperationKind.Rank2K: 'rank_2k'
1092:   , OperationKind.Trmm: 'trmm'
1093:   , OperationKind.Symm: 'symm'
1094:   , OperationKind.Conv2d: 'conv2d'
1095:   , OperationKind.Conv3d: 'conv3d'
1096: }
```
**EN:** Defines or updates module/class-level data such as OperationKindNames; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 OperationKindNames；这些值会被后续生成器与 emitter 引用。

### L1097-L1098 — Comments

```python
1097: 
1098: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1099-L1099 — Class `Target`

```python
1099: class Target(enum.Enum):
```
**EN:** Introduces class `Target`, which packages the target logic into a reusable type.

**CN:** 引入类 `Target`，将 target 相关逻辑封装为可复用类型。

### L1100-L1100 — Data definitions

```python
1100:   library = enum_auto()
```
**EN:** Defines or updates module/class-level data such as library; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 library；这些值会被后续生成器与 emitter 引用。

### L1101-L1101 — Comments

```python
1101: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1102-L1128 — Data definitions

```python
1102: ArchitectureNames = {
1103:   INTEL_XE12: 'pvc',
1104:   INTEL_XE20: 'bmg',
1105:   50: 'maxwell',
1106:   60: 'pascal',
1107:   61: 'pascal',
1108:   70: 'volta',
1109:   75: 'turing',
1110:   80: 'ampere',
1111:   89: 'ada',
1112:   90: 'hopper'
1113: }
1114: 
1115: #
1116: SharedMemPerCC = {
1117:   INTEL_XE12: 128, # 128 KiB of SMEM on Intel PVC
1118:   INTEL_XE20: 128, # 128 KiB of SMEM on Intel BMG
1119:   70:  96, #  96KB of SMEM
1120:   72:  96, #  96KB of SMEM
1121:   75:  64, #  64KB of SMEM
1122:   80: 163, # 163KB of SMEM - 1KB reserved for the driver
1123:   86:  99, #  99KB of SMEM - 1KB reserved for the driver
1124:   87: 163, # 163KB of SMEM - 1KB reserved for the driver
1125:   89:  99, #  99KB of SMEM - 1KB reserved for the driver
1126:   90: 227, # 227KB of SMEM - 1KB reserved for the driver
1127:   100: 227, # 227KB of SMEM - 1KB reserved for the driver
1128: }
```
**EN:** Defines or updates module/class-level data such as ArchitectureNames, SharedMemPerCC; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 ArchitectureNames, SharedMemPerCC；这些值会被后续生成器与 emitter 引用。

### L1129-L1132 — Comments

```python
1129: 
1130: ###################################################################################################
1131: 
1132: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1133-L1144 — Function `SubstituteTemplate`

```python
1133: def SubstituteTemplate(template, values):
1134:   text = template
1135:   changed = True
1136:   while changed:
1137:     changed = False
1138:     for key, value in values.items():
1139:       regex = "\\$\\{%s\\}" % key
1140:       newtext = re.sub(regex, value, text)
1141:       if newtext != text:
1142:         changed = True
1143:       text = newtext
1144:   return text
```
**EN:** Defines `SubstituteTemplate()`, which implements the substitute template logic. Key helper calls include items, sub.

**CN:** 定义 `SubstituteTemplate()`，用于实现 substitute template 相关逻辑。 其中会调用的重要辅助函数包括 items, sub。

### L1145-L1148 — Comments

```python
1145: 
1146: ###################################################################################################
1147: 
1148: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1149-L1149 — Class `GemmKind`

```python
1149: class GemmKind(enum.Enum):
```
**EN:** Introduces class `GemmKind`, which packages the gemm kind logic into a reusable type.

**CN:** 引入类 `GemmKind`，将 gemm kind 相关逻辑封装为可复用类型。

### L1150-L1162 — Data definitions

```python
1150:   Gemm = enum_auto()
1151:   Sparse = enum_auto()
1152:   Universal = enum_auto()
1153:   Universal3x = enum_auto()
1154:   SparseUniversal3x = enum_auto()
1155:   PlanarComplex = enum_auto()
1156:   PlanarComplexArray = enum_auto()
1157:   Grouped = enum_auto()
1158:   BlockScaledUniversal3x = enum_auto()                                   
1159:   GroupedUniversal3x = enum_auto()
1160:   GroupedBlockScaledUniversal3x = enum_auto()
1161:   BlockwiseUniversal3x = enum_auto()
1162:   GroupedBlockwiseUniversal3x = enum_auto()
```
**EN:** Defines or updates module/class-level data such as Gemm, Sparse, Universal, Universal3x, SparseUniversal3x, PlanarComplex, PlanarComplexArray, Grouped, ...; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Gemm, Sparse, Universal, Universal3x, SparseUniversal3x, PlanarComplex, PlanarComplexArray, Grouped, ...；这些值会被后续生成器与 emitter 引用。

### L1163-L1164 — Comments

```python
1163: 
1164: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1165-L1179 — Data definitions

```python
1165: GemmKindNames = {
1166:   GemmKind.Gemm: "gemm",
1167:   GemmKind.Sparse: "spgemm",
1168:   GemmKind.Universal: "gemm",
1169:   GemmKind.Universal3x: "gemm",
1170:   GemmKind.SparseUniversal3x: "spgemm",
1171:   GemmKind.PlanarComplex: "gemm_planar_complex",
1172:   GemmKind.PlanarComplexArray: "gemm_planar_complex_array",
1173:   GemmKind.Grouped: "gemm_grouped",
1174:   GemmKind.BlockScaledUniversal3x: "gemm",
1175:   GemmKind.GroupedUniversal3x: "gemm_grouped",
1176:   GemmKind.GroupedBlockScaledUniversal3x: "gemm_grouped",
1177:   GemmKind.BlockwiseUniversal3x: "gemm",
1178:   GemmKind.GroupedBlockwiseUniversal3x: "gemm_grouped"
1179: }
```
**EN:** Defines or updates module/class-level data such as GemmKindNames; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 GemmKindNames；这些值会被后续生成器与 emitter 引用。

### L1180-L1181 — Comments

```python
1180: 
1181: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1182-L1182 — Class `RankKKind`

```python
1182: class RankKKind(enum.Enum):
```
**EN:** Introduces class `RankKKind`, which packages the rank kkind logic into a reusable type.

**CN:** 引入类 `RankKKind`，将 rank kkind 相关逻辑封装为可复用类型。

### L1183-L1183 — Data definitions

```python
1183:   Universal = enum_auto()
```
**EN:** Defines or updates module/class-level data such as Universal; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Universal；这些值会被后续生成器与 emitter 引用。

### L1184-L1185 — Comments

```python
1184: 
1185: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1186-L1188 — Data definitions

```python
1186: RankKKindNames = {
1187:   RankKKind.Universal: "rank_k"
1188: }
```
**EN:** Defines or updates module/class-level data such as RankKKindNames; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 RankKKindNames；这些值会被后续生成器与 emitter 引用。

### L1189-L1190 — Comments

```python
1189: 
1190: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1191-L1191 — Class `TrmmKind`

```python
1191: class TrmmKind(enum.Enum):
```
**EN:** Introduces class `TrmmKind`, which packages the trmm kind logic into a reusable type.

**CN:** 引入类 `TrmmKind`，将 trmm kind 相关逻辑封装为可复用类型。

### L1192-L1192 — Data definitions

```python
1192:   Universal = enum_auto()
```
**EN:** Defines or updates module/class-level data such as Universal; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Universal；这些值会被后续生成器与 emitter 引用。

### L1193-L1194 — Comments

```python
1193: 
1194: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1195-L1197 — Data definitions

```python
1195: TrmmKindNames = {
1196:   TrmmKind.Universal: "trmm"
1197: }
```
**EN:** Defines or updates module/class-level data such as TrmmKindNames; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 TrmmKindNames；这些值会被后续生成器与 emitter 引用。

### L1198-L1199 — Comments

```python
1198: 
1199: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1200-L1200 — Class `SymmKind`

```python
1200: class SymmKind(enum.Enum):
```
**EN:** Introduces class `SymmKind`, which packages the symm kind logic into a reusable type.

**CN:** 引入类 `SymmKind`，将 symm kind 相关逻辑封装为可复用类型。

### L1201-L1201 — Data definitions

```python
1201:   Universal = enum_auto()
```
**EN:** Defines or updates module/class-level data such as Universal; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Universal；这些值会被后续生成器与 emitter 引用。

### L1202-L1203 — Comments

```python
1202: 
1203: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1204-L1206 — Data definitions

```python
1204: SymmKindNames = {
1205:   SymmKind.Universal: "symm"
1206: }
```
**EN:** Defines or updates module/class-level data such as SymmKindNames; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 SymmKindNames；这些值会被后续生成器与 emitter 引用。

### L1207-L1208 — Comments

```python
1207: 
1208: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1209-L1209 — Class `EpilogueFunctor`

```python
1209: class EpilogueFunctor(enum.Enum):
```
**EN:** Introduces class `EpilogueFunctor`, which packages the epilogue functor logic into a reusable type.

**CN:** 引入类 `EpilogueFunctor`，将 epilogue functor 相关逻辑封装为可复用类型。

### L1210-L1211 — Data definitions

```python
1210:   LinearCombination = enum_auto()
1211:   LinearCombinationClamp = enum_auto()
```
**EN:** Defines or updates module/class-level data such as LinearCombination, LinearCombinationClamp; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 LinearCombination, LinearCombinationClamp；这些值会被后续生成器与 emitter 引用。

### L1212-L1213 — Comments

```python
1212: 
1213: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1214-L1217 — Data definitions

```python
1214: EpilogueFunctorTag = {
1215:   EpilogueFunctor.LinearCombination: 'cutlass::epilogue::thread::LinearCombination',
1216:   EpilogueFunctor.LinearCombinationClamp: 'cutlass::epilogue::thread::LinearCombinationClamp',
1217: }
```
**EN:** Defines or updates module/class-level data such as EpilogueFunctorTag; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 EpilogueFunctorTag；这些值会被后续生成器与 emitter 引用。

### L1218-L1219 — Comments

```python
1218: 
1219: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1220-L1220 — Class `MixedInputMode`

```python
1220: class MixedInputMode(enum.Enum):
```
**EN:** Introduces class `MixedInputMode`, which packages the mixed input mode logic into a reusable type.

**CN:** 引入类 `MixedInputMode`，将 mixed input mode 相关逻辑封装为可复用类型。

### L1221-L1223 — Data definitions

```python
1221:   ConvertOnly = enum_auto()
1222:   ScaleOnly = enum_auto()
1223:   ScaleWithZeroPoint = enum_auto()
```
**EN:** Defines or updates module/class-level data such as ConvertOnly, ScaleOnly, ScaleWithZeroPoint; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 ConvertOnly, ScaleOnly, ScaleWithZeroPoint；这些值会被后续生成器与 emitter 引用。

### L1224-L1225 — Comments

```python
1224: 
1225: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1226-L1226 — Class `SwizzlingFunctor`

```python
1226: class SwizzlingFunctor(enum.Enum):
```
**EN:** Introduces class `SwizzlingFunctor`, which packages the swizzling functor logic into a reusable type.

**CN:** 引入类 `SwizzlingFunctor`，将 swizzling functor 相关逻辑封装为可复用类型。

### L1227-L1235 — Data definitions

```python
1227:   Identity1 = enum_auto()
1228:   Identity2 = enum_auto()
1229:   Identity4 = enum_auto()
1230:   Identity8 = enum_auto()
1231:   Horizontal = enum_auto()
1232:   StridedDgradIdentity1 = enum_auto()
1233:   StridedDgradIdentity4 = enum_auto()
1234:   StridedDgradHorizontal = enum_auto()
1235:   StreamK = enum_auto()
```
**EN:** Defines or updates module/class-level data such as Identity1, Identity2, Identity4, Identity8, Horizontal, StridedDgradIdentity1, StridedDgradIdentity4, StridedDgradHorizontal, ...; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Identity1, Identity2, Identity4, Identity8, Horizontal, StridedDgradIdentity1, StridedDgradIdentity4, StridedDgradHorizontal, ...；这些值会被后续生成器与 emitter 引用。

### L1236-L1237 — Comments

```python
1236: 
1237: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1238-L1248 — Data definitions

```python
1238: SwizzlingFunctorTag = {
1239:   SwizzlingFunctor.Identity1: 'cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<1>',
1240:   SwizzlingFunctor.Identity2: 'cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<2>',
1241:   SwizzlingFunctor.Identity4: 'cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<4>',
1242:   SwizzlingFunctor.Identity8: 'cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<8>',
1243:   SwizzlingFunctor.Horizontal: 'cutlass::gemm::threadblock::GemmHorizontalThreadblockSwizzle',
1244:   SwizzlingFunctor.StridedDgradIdentity1: 'cutlass::conv::threadblock::StridedDgradIdentityThreadblockSwizzle<1>',
1245:   SwizzlingFunctor.StridedDgradIdentity4: 'cutlass::conv::threadblock::StridedDgradIdentityThreadblockSwizzle<4>',
1246:   SwizzlingFunctor.StridedDgradHorizontal: 'cutlass::conv::threadblock::StridedDgradHorizontalThreadblockSwizzle',
1247:   SwizzlingFunctor.StreamK: 'cutlass::gemm::threadblock::ThreadblockSwizzleStreamK',
1248: }
```
**EN:** Defines or updates module/class-level data such as SwizzlingFunctorTag; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 SwizzlingFunctorTag；这些值会被后续生成器与 emitter 引用。

### L1249-L1250 — Comments

```python
1249: 
1250: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1251-L1251 — Class `GroupScheduleMode`

```python
1251: class GroupScheduleMode(enum.Enum):
```
**EN:** Introduces class `GroupScheduleMode`, which packages the group schedule mode logic into a reusable type.

**CN:** 引入类 `GroupScheduleMode`，将 group schedule mode 相关逻辑封装为可复用类型。

### L1252-L1253 — Data definitions

```python
1252:   Device = enum_auto(),
1253:   Host = enum_auto()
```
**EN:** Defines or updates module/class-level data such as Device, Host; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Device, Host；这些值会被后续生成器与 emitter 引用。

### L1254-L1255 — Comments

```python
1254: 
1255: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1256-L1265 — Data definitions

```python
1256: GroupScheduleModeTag = {
1257:   GroupScheduleMode.Device: 'cutlass::gemm::kernel::GroupScheduleMode::kDeviceOnly',
1258:   GroupScheduleMode.Host: 'cutlass::gemm::kernel::GroupScheduleMode::kHostPrecompute'
1259: }
1260: 
1261: #
1262: ShortGroupScheduleModeNames = {
1263:   GroupScheduleMode.Device: 'Device',
1264:   GroupScheduleMode.Host: 'Host'
1265: }
```
**EN:** Defines or updates module/class-level data such as GroupScheduleModeTag, ShortGroupScheduleModeNames; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 GroupScheduleModeTag, ShortGroupScheduleModeNames；这些值会被后续生成器与 emitter 引用。

### L1266-L1269 — Comments

```python
1266: 
1267: ###################################################################################################
1268: 
1269: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1270-L1270 — Class `ConvKind`

```python
1270: class ConvKind(enum.IntEnum):
```
**EN:** Introduces class `ConvKind`, which packages the conv kind logic into a reusable type.

**CN:** 引入类 `ConvKind`，将 conv kind 相关逻辑封装为可复用类型。

### L1271-L1273 — Data definitions

```python
1271:   Fprop = 0
1272:   Dgrad = 1
1273:   Wgrad = 2
```
**EN:** Defines or updates module/class-level data such as Fprop, Dgrad, Wgrad; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Fprop, Dgrad, Wgrad；这些值会被后续生成器与 emitter 引用。

### L1274-L1275 — Comments

```python
1274: 
1275: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1276-L1286 — Data definitions

```python
1276: ConvKindTag = {
1277:   ConvKind.Fprop: 'cutlass::conv::Operator::kFprop',
1278:   ConvKind.Dgrad: 'cutlass::conv::Operator::kDgrad',
1279:   ConvKind.Wgrad: 'cutlass::conv::Operator::kWgrad'
1280: }
1281: 
1282: ConvKindNames = {
1283:   ConvKind.Fprop: 'fprop',
1284:   ConvKind.Dgrad: 'dgrad',
1285:   ConvKind.Wgrad: 'wgrad',
1286: }
```
**EN:** Defines or updates module/class-level data such as ConvKindTag, ConvKindNames; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 ConvKindTag, ConvKindNames；这些值会被后续生成器与 emitter 引用。

### L1288-L1288 — Class `ConvMode`

```python
1288: class ConvMode(enum.IntEnum):
```
**EN:** Introduces class `ConvMode`, which packages the conv mode logic into a reusable type.

**CN:** 引入类 `ConvMode`，将 conv mode 相关逻辑封装为可复用类型。

### L1289-L1290 — Data definitions

```python
1289:   CrossCorrelation = 0
1290:   Convolution = 1
```
**EN:** Defines or updates module/class-level data such as CrossCorrelation, Convolution; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 CrossCorrelation, Convolution；这些值会被后续生成器与 emitter 引用。

### L1291-L1292 — Comments

```python
1291: 
1292: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1293-L1293 — Class `IteratorAlgorithm`

```python
1293: class IteratorAlgorithm(enum.Enum):
```
**EN:** Introduces class `IteratorAlgorithm`, which packages the iterator algorithm logic into a reusable type.

**CN:** 引入类 `IteratorAlgorithm`，将 iterator algorithm 相关逻辑封装为可复用类型。

### L1294-L1298 — Data definitions

```python
1294:   Analytic = 0
1295:   Optimized = 1
1296:   FixedChannels = 2
1297:   FewChannels = 3
1298:   FixedStrideDilation = 4
```
**EN:** Defines or updates module/class-level data such as Analytic, Optimized, FixedChannels, FewChannels, FixedStrideDilation; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Analytic, Optimized, FixedChannels, FewChannels, FixedStrideDilation；这些值会被后续生成器与 emitter 引用。

### L1299-L1300 — Comments

```python
1299: 
1300: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1301-L1315 — Data definitions

```python
1301: IteratorAlgorithmTag = {
1302:   IteratorAlgorithm.Analytic: 'cutlass::conv::IteratorAlgorithm::kAnalytic',
1303:   IteratorAlgorithm.Optimized: 'cutlass::conv::IteratorAlgorithm::kOptimized',
1304:   IteratorAlgorithm.FixedChannels: 'cutlass::conv::IteratorAlgorithm::kFixedChannels',
1305:   IteratorAlgorithm.FewChannels: 'cutlass::conv::IteratorAlgorithm::kFewChannels',
1306:   IteratorAlgorithm.FixedStrideDilation: 'cutlass::conv::IteratorAlgorithm::kFixedStrideDilation'
1307: }
1308: 
1309: IteratorAlgorithmNames = {
1310:   IteratorAlgorithm.Analytic: 'analytic',
1311:   IteratorAlgorithm.Optimized: 'optimized',
1312:   IteratorAlgorithm.FixedChannels: 'fixed_channels',
1313:   IteratorAlgorithm.FewChannels: 'few_channels',
1314:   IteratorAlgorithm.FixedStrideDilation: 'fixed_stride_dilation'
1315: }
```
**EN:** Defines or updates module/class-level data such as IteratorAlgorithmTag, IteratorAlgorithmNames; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 IteratorAlgorithmTag, IteratorAlgorithmNames；这些值会被后续生成器与 emitter 引用。

### L1316-L1317 — Comments

```python
1316: 
1317: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1318-L1318 — Class `StrideSupport`

```python
1318: class StrideSupport(enum.Enum):
```
**EN:** Introduces class `StrideSupport`, which packages the stride support logic into a reusable type.

**CN:** 引入类 `StrideSupport`，将 stride support 相关逻辑封装为可复用类型。

### L1319-L1321 — Data definitions

```python
1319:   Strided = 0
1320:   Unity = 1
1321:   Fixed = 2
```
**EN:** Defines or updates module/class-level data such as Strided, Unity, Fixed; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Strided, Unity, Fixed；这些值会被后续生成器与 emitter 引用。

### L1322-L1323 — Comments

```python
1322: 
1323: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1324-L1334 — Data definitions

```python
1324: StrideSupportTag = {
1325:   StrideSupport.Strided: 'cutlass::conv::StrideSupport::kStrided',
1326:   StrideSupport.Unity: 'cutlass::conv::StrideSupport::kUnity',
1327:   StrideSupport.Fixed: 'cutlass::conv::StrideSupport::kFixed'
1328: }
1329: 
1330: StrideSupportNames = {
1331:   StrideSupport.Strided: '',
1332:   StrideSupport.Unity: 'unity_stride',
1333:   StrideSupport.Fixed: 'fixed_stride'
1334: }
```
**EN:** Defines or updates module/class-level data such as StrideSupportTag, StrideSupportNames; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 StrideSupportTag, StrideSupportNames；这些值会被后续生成器与 emitter 引用。

### L1335-L1336 — Comments

```python
1335: 
1336: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1337-L1337 — Class `GroupMode`

```python
1337: class GroupMode(enum.Enum):
```
**EN:** Introduces class `GroupMode`, which packages the group mode logic into a reusable type.

**CN:** 引入类 `GroupMode`，将 group mode 相关逻辑封装为可复用类型。

### L1338-L1341 — Data definitions

```python
1338:   NoneGroup = enum_auto()         # dense conv (G=1)
1339:   SingleGroup = enum_auto()       # grouped convolution (single group per CTA)
1340:   MultipleGroup = enum_auto()     # grouped convolution ( multiple groups per CTA)
1341:   Depthwise = enum_auto()         # Depthwise convolution ( C=K=G )
```
**EN:** Defines or updates module/class-level data such as NoneGroup, SingleGroup, MultipleGroup, Depthwise; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 NoneGroup, SingleGroup, MultipleGroup, Depthwise；这些值会被后续生成器与 emitter 引用。

### L1342-L1343 — Comments

```python
1342: 
1343: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1344-L1358 — Data definitions

```python
1344: GroupModeTag = {
1345:   GroupMode.NoneGroup: 'cutlass::conv::GroupMode::kNone',
1346:   GroupMode.SingleGroup: 'cutlass::conv::GroupMode::kSingleGroup',
1347:   GroupMode.MultipleGroup: 'cutlass::conv::GroupMode::kMultipleGroup',
1348:   GroupMode.Depthwise: 'cutlass::conv::GroupMode::kDepthwise',
1349: }
1350: 
1351: GroupModeNames = {
1352:   GroupMode.NoneGroup: '',
1353:   GroupMode.SingleGroup: 'single_group',
1354:   GroupMode.MultipleGroup: 'multiple_group',
1355:   GroupMode.Depthwise: 'depthwise',
1356: }
1357: 
1358: DynamicClusterShape = [0, 0, 1] 
```
**EN:** Defines or updates module/class-level data such as GroupModeTag, GroupModeNames, DynamicClusterShape; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 GroupModeTag, GroupModeNames, DynamicClusterShape；这些值会被后续生成器与 emitter 引用。

### L1359-L1362 — Comments

```python
1359: 
1360: ###################################################################################################
1361: 
1362: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1363-L1363 — Class `MathInstruction`

```python
1363: class MathInstruction:
```
**EN:** Introduces class `MathInstruction`, which packages the math instruction logic into a reusable type.

**CN:** 引入类 `MathInstruction`，将 math instruction 相关逻辑封装为可复用类型。

### L1364-L1377 — Function `__init__`

```python
1364:   def __init__(self,
1365:       instruction_shape,                                            \
1366:       element_a, element_b, element_accumulator,                    \
1367:       opcode_class, math_operation = MathOperation.multiply_add     \
1368:       , element_scale_factor = None 
1369:     ):
1370: 
1371:     self.instruction_shape = instruction_shape
1372:     self.element_a = element_a
1373:     self.element_b = element_b
1374:     self.element_accumulator = element_accumulator
1375:     self.opcode_class = opcode_class
1376:     self.math_operation = math_operation
1377:     self.element_scale_factor = element_scale_factor 
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L1378-L1379 — Comments

```python
1378: 
1379: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1380-L1381 — Class `TileDescription`

```python
1380: class TileDescription:
1381: 
```
**EN:** Introduces class `TileDescription`, which packages the tile description logic into a reusable type.

**CN:** 引入类 `TileDescription`，将 tile description 相关逻辑封装为可复用类型。

### L1382-L1391 — Function `__init__`

```python
1382:   def __init__(self, threadblock_shape, stages, warp_count, math_instruction, min_compute, max_compute, cluster_shape = [1,1,1], explicit_vector_sizes = None):
1383:     self.threadblock_shape = threadblock_shape
1384:     self.tile_shape = threadblock_shape
1385:     self.stages = stages
1386:     self.warp_count = warp_count
1387:     self.math_instruction = math_instruction
1388:     self.minimum_compute_capability = min_compute
1389:     self.maximum_compute_capability = max_compute
1390:     self.cluster_shape = cluster_shape
1391:     self.explicit_vector_sizes = explicit_vector_sizes
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L1393-L1404 — Function `procedural_name`

```python
1393:   def procedural_name(self):
1394:     if self.minimum_compute_capability >= 90:
1395:       return "{tbm}x{tbn}x{tbk}_{cm}x{cn}x{ck}_{s}".format(
1396:         tbm = self.threadblock_shape[0],
1397:         tbn = self.threadblock_shape[1],
1398:         tbk = self.threadblock_shape[2],
1399:         cm = self.cluster_shape[0],
1400:         cn = self.cluster_shape[1],
1401:         ck = self.cluster_shape[2],
1402:         s = self.stages)
1403:     else:
1404:       return "%dx%d_%dx%d" % (self.threadblock_shape[0], self.threadblock_shape[1], self.threadblock_shape[2], self.stages)
```
**EN:** Defines `procedural_name()`, which implements the procedural name logic. Key helper calls include format.

**CN:** 定义 `procedural_name()`，用于实现 procedural name 相关逻辑。 其中会调用的重要辅助函数包括 format。

### L1405-L1406 — Comments

```python
1405: 
1406: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1407-L1407 — Class `Direct2dConvFixedStrideDilationTileDescription`

```python
1407: class Direct2dConvFixedStrideDilationTileDescription:
```
**EN:** Introduces class `Direct2dConvFixedStrideDilationTileDescription`, which packages the direct2d conv fixed stride dilation tile description logic into a reusable type.

**CN:** 引入类 `Direct2dConvFixedStrideDilationTileDescription`，将 direct2d conv fixed stride dilation tile description 相关逻辑封装为可复用类型。

### L1408-L1418 — Function `__init__`

```python
1408:   def __init__(self, threadblock_output_shape, filter_shape, stages, stride, dilation, warp_count, math_instruction, min_compute, max_compute):
1409:     self.threadblock_shape = [threadblock_output_shape[0]*threadblock_output_shape[1]*threadblock_output_shape[2], threadblock_output_shape[3], filter_shape[0]*filter_shape[1]]
1410:     self.threadblock_output_shape = threadblock_output_shape
1411:     self.filter_shape = filter_shape
1412:     self.stages = stages
1413:     self.warp_count = warp_count
1414:     self.stride = stride
1415:     self.dilation =  dilation
1416:     self.math_instruction = math_instruction
1417:     self.minimum_compute_capability = min_compute
1418:     self.maximum_compute_capability = max_compute
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L1420-L1437 — Function `procedural_name`

```python
1420:   def procedural_name(self):
1421:     str_name = "%dx%dx%d_%dx%dx%dx%d_%d_filter%dx%d" % (self.threadblock_shape[0],
1422:                                       self.threadblock_shape[1],
1423:                                       self.threadblock_shape[2],
1424:                                       self.threadblock_output_shape[0],
1425:                                       self.threadblock_output_shape[1],
1426:                                       self.threadblock_output_shape[2],
1427:                                       self.threadblock_output_shape[3],
1428:                                       self.stages,
1429:                                       self.filter_shape[0],
1430:                                       self.filter_shape[1])
1431:     # Fixed Strided and dilation
1432:     if self.stride != [-1, -1] and self.dilation != [-1, -1]:
1433:       str_name += "_stride%dx%d_dilation%dx%d" % (self.stride[0],
1434:                                                   self.stride[1],
1435:                                                   self.dilation[0],
1436:                                                   self.dilation[1])
1437:     return str_name
```
**EN:** Defines `procedural_name()`, which implements the procedural name logic.

**CN:** 定义 `procedural_name()`，用于实现 procedural name 相关逻辑。

### L1438-L1439 — Comments

```python
1438: 
1439: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1440-L1440 — Class `Direct2dConvFixedStrideDilationTileDescription`

```python
1440: class Direct2dConvFixedStrideDilationTileDescription:
```
**EN:** Introduces class `Direct2dConvFixedStrideDilationTileDescription`, which packages the direct2d conv fixed stride dilation tile description logic into a reusable type.

**CN:** 引入类 `Direct2dConvFixedStrideDilationTileDescription`，将 direct2d conv fixed stride dilation tile description 相关逻辑封装为可复用类型。

### L1441-L1451 — Function `__init__`

```python
1441:   def __init__(self, threadblock_output_shape, filter_shape, stages, stride, dilation, warp_count, math_instruction, min_compute, max_compute):
1442:     self.threadblock_shape = [threadblock_output_shape[0]*threadblock_output_shape[1]*threadblock_output_shape[2], threadblock_output_shape[3], filter_shape[0]*filter_shape[1]]
1443:     self.threadblock_output_shape = threadblock_output_shape
1444:     self.filter_shape = filter_shape
1445:     self.stages = stages
1446:     self.warp_count = warp_count
1447:     self.stride = stride
1448:     self.dilation =  dilation
1449:     self.math_instruction = math_instruction
1450:     self.minimum_compute_capability = min_compute
1451:     self.maximum_compute_capability = max_compute
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L1453-L1470 — Function `procedural_name`

```python
1453:   def procedural_name(self):
1454:     str_name = "%dx%dx%d_%dx%dx%dx%d_%d_filter%dx%d" % (self.threadblock_shape[0],
1455:                                       self.threadblock_shape[1],
1456:                                       self.threadblock_shape[2],
1457:                                       self.threadblock_output_shape[0],
1458:                                       self.threadblock_output_shape[1],
1459:                                       self.threadblock_output_shape[2],
1460:                                       self.threadblock_output_shape[3],
1461:                                       self.stages,
1462:                                       self.filter_shape[0],
1463:                                       self.filter_shape[1])
1464:     # Fixed Strided and dilation
1465:     if self.stride != [-1, -1] and self.dilation != [-1, -1]:
1466:       str_name += "_stride%dx%d_dilation%dx%d" % (self.stride[0],
1467:                                                   self.stride[1],
1468:                                                   self.dilation[0],
1469:                                                   self.dilation[1])
1470:     return str_name
```
**EN:** Defines `procedural_name()`, which implements the procedural name logic.

**CN:** 定义 `procedural_name()`，用于实现 procedural name 相关逻辑。

### L1471-L1472 — Comments

```python
1471: 
1472: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1473-L1473 — Class `TensorDescription`

```python
1473: class TensorDescription:
```
**EN:** Introduces class `TensorDescription`, which packages the tensor description logic into a reusable type.

**CN:** 引入类 `TensorDescription`，将 tensor description 相关逻辑封装为可复用类型。

### L1474-L1478 — Function `__init__`

```python
1474:   def __init__(self, element, layout, alignment = 1, complex_transform = ComplexTransform.none):
1475:     self.element = element
1476:     self.layout = layout
1477:     self.alignment = alignment
1478:     self.complex_transform = complex_transform
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L1479-L1480 — Comments

```python
1479: 
1480: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1481-L1481 — Class `SymmetricTensorDescription`

```python
1481: class SymmetricTensorDescription:
```
**EN:** Introduces class `SymmetricTensorDescription`, which packages the symmetric tensor description logic into a reusable type.

**CN:** 引入类 `SymmetricTensorDescription`，将 symmetric tensor description 相关逻辑封装为可复用类型。

### L1482-L1488 — Function `__init__`

```python
1482:   def __init__(self, element, layout, fill_mode, alignment = 1, complex_transform = ComplexTransform.none, side_mode = SideMode.Left):
1483:     self.element = element
1484:     self.layout = layout
1485:     self.fill_mode = fill_mode
1486:     self.alignment = alignment
1487:     self.complex_transform = complex_transform
1488:     self.side_mode = side_mode
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L1489-L1490 — Comments

```python
1489: 
1490: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1491-L1491 — Class `TriangularTensorDescription`

```python
1491: class TriangularTensorDescription:
```
**EN:** Introduces class `TriangularTensorDescription`, which packages the triangular tensor description logic into a reusable type.

**CN:** 引入类 `TriangularTensorDescription`，将 triangular tensor description 相关逻辑封装为可复用类型。

### L1492-L1499 — Function `__init__`

```python
1492:   def __init__(self, element, layout, side_mode, fill_mode, diag_type, alignment = 1, complex_transform = ComplexTransform.none):
1493:     self.element = element
1494:     self.layout = layout
1495:     self.side_mode = side_mode
1496:     self.fill_mode = fill_mode
1497:     self.diag_type = diag_type
1498:     self.alignment = alignment
1499:     self.complex_transform = complex_transform
```
**EN:** Defines `__init__()`, which implements the init logic.

**CN:** 定义 `__init__()`，用于实现 init 相关逻辑。

### L1500-L1501 — Comments

```python
1500: 
1501: #
```
**EN:** Contains standalone comments or separators that document the next code region.

**CN:** 包含用于说明后续代码区域的独立注释或分隔线。

### L1502-L1529 — Function `CalculateSmemUsage`

```python
1502: def CalculateSmemUsage(operation):
1503:   cta_shape = operation.tile_description.threadblock_shape
1504:   stages = operation.tile_description.stages
1505: 
1506:   if operation.operation_kind == OperationKind.Gemm and operation.gemm_kind == GemmKind.Sparse:
1507:     # Elements represented by 8 bits of metadata (based on 4:8, 2:4 or 1:2 sparsity)
1508:     if DataTypeSize[operation.A.element] == 32:
1509:       elements_per_8b_md = 2
1510:     elif DataTypeSize[operation.A.element] == 4:
1511:       elements_per_8b_md = 8
1512:     else:
1513:       elements_per_8b_md = 4
1514: 
1515:     smem_per_stage = DataTypeSize[operation.A.element] * cta_shape[0] * (cta_shape[2] // 2) // 8 + \
1516:                      DataTypeSize[operation.B.element] * cta_shape[1] * cta_shape[2] // 8 + \
1517:                      cta_shape[0] * (cta_shape[2] // 2) // elements_per_8b_md
1518:   else:
1519:     # Few BLAS3 operations only have A tensor
1520:     data_type_size_a = DataTypeSize[operation.A.element]
1521:     data_type_size_b = DataTypeSize[operation.A.element]
1522:     if operation.is_mixed_input():
1523:       data_type_size_b = DataTypeSize[operation.B.element]
1524: 
1525:     smem_per_stage = data_type_size_a * cta_shape[0] * cta_shape[2] // 8 + \
1526:                      data_type_size_b * cta_shape[1] * cta_shape[2] // 8
1527: 
1528:   smem_usage = smem_per_stage * stages
1529:   return (smem_usage >> 10)
```
**EN:** Defines `CalculateSmemUsage()`, which implements the calculate smem usage logic. Key helper calls include is_mixed_input.

**CN:** 定义 `CalculateSmemUsage()`，用于实现 calculate smem usage 相关逻辑。 其中会调用的重要辅助函数包括 is_mixed_input。

### L1532-L1532 — Class `GemmUniversalMode`

```python
1532: class GemmUniversalMode(enum.IntEnum):
```
**EN:** Introduces class `GemmUniversalMode`. The docstring says: "Types corresponding to GemmUniversalMode"; the class body implements that role.

**CN:** 引入类 `GemmUniversalMode`。文档字符串说明其用途为：“Types corresponding to GemmUniversalMode”；类体负责实现这一职责。

### L1533-L1539 — Data definitions

```python
1533:   """
1534:   Types corresponding to GemmUniversalMode
1535:   """
1536:   Gemm = 0
1537:   GemmSplitKParallel = 1
1538:   Batched = 2
1539:   Array = 3
```
**EN:** Defines or updates module/class-level data such as Gemm, GemmSplitKParallel, Batched, Array; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 Gemm, GemmSplitKParallel, Batched, Array；这些值会被后续生成器与 emitter 引用。

### L1542-L1542 — Class `SplitKMode`

```python
1542: class SplitKMode(enum.IntEnum):
```
**EN:** Introduces class `SplitKMode`. The docstring says: "Types corresponding to SplitKMode"; the class body implements that role.

**CN:** 引入类 `SplitKMode`。文档字符串说明其用途为：“Types corresponding to SplitKMode”；类体负责实现这一职责。

### L1543-L1548 — Data definitions

```python
1543:   """
1544:   Types corresponding to SplitKMode
1545:   """
1546:   NoneSplitK = 0
1547:   Serial = 1
1548:   Parallel = 2
```
**EN:** Defines or updates module/class-level data such as NoneSplitK, Serial, Parallel; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 NoneSplitK, Serial, Parallel；这些值会被后续生成器与 emitter 引用。

## Key Concepts / 关键概念

- **EN:** File role: Defines the shared enums, type/layout metadata, tags, and helper utilities consumed by the whole generator stack.
- **CN:** 文件角色：定义整个生成器体系共享的枚举、类型/布局元数据、标签与辅助工具。
- **EN:** Main classes: `GeneratorTarget, DataType, BlasMode, ComplexTransform, ComplexMultiplyOp, MathOperation, LayoutType, KernelScheduleType, EpilogueScheduleType, EpilogueFunctor3x, ...`
- **CN:** 主要类：`GeneratorTarget, DataType, BlasMode, ComplexTransform, ComplexMultiplyOp, MathOperation, LayoutType, KernelScheduleType, EpilogueScheduleType, EpilogueFunctor3x, ...`
- **EN:** Main functions: `is_complex, is_block_scaled, is_blockwise, is_grouped, get_complex_from_real, get_real_from_complex, get_tma_alignment, is_tma_epilogue, to_grouped_schedule, SubstituteTemplate, CalculateSmemUsage`
- **CN:** 主要函数：`is_complex, is_block_scaled, is_blockwise, is_grouped, get_complex_from_real, get_real_from_complex, get_tma_alignment, is_tma_epilogue, to_grouped_schedule, SubstituteTemplate, CalculateSmemUsage`
- **EN:** Important constants/tables: `GeneratorTargetNames, ShortDataTypeNames, DataTypeNames, DataTypeTag, BlasModeTag, ComplexTransformTag, MathOperationTag, LayoutTag, ShortLayoutTypeNames, ShortComplexLayoutNames, KernelScheduleTag, EpilogueScheduleTag, ...`
- **CN:** 重要常量/表：`GeneratorTargetNames, ShortDataTypeNames, DataTypeNames, DataTypeTag, BlasModeTag, ComplexTransformTag, MathOperationTag, LayoutTag, ShortLayoutTypeNames, ShortComplexLayoutNames, KernelScheduleTag, EpilogueScheduleTag, ...`

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: `enum, re`
- **CN:** 标准库依赖：`enum, re`
- **EN:** Internal CUTLASS dependencies: `cutlass_library.arch_constants, arch_constants`
- **CN:** CUTLASS 内部依赖：`cutlass_library.arch_constants, arch_constants`
- **EN:** External/non-stdlib dependencies: `None explicitly imported`
- **CN:** 外部/非标准库依赖：`未显式导入`
