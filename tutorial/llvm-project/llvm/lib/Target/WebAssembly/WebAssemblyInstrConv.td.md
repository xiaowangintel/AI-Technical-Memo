# WebAssemblyInstrConv.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyInstrConv.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: WebAssembly datatype conversions, truncations, reinterpretations, promotions, and demotions operand code-gen constructs. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyInstrConv.td`，主要负责 WebAssembly 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===-- WebAssemblyInstrConv.td-WebAssembly Conversion support -*- tablegen -*-=
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-12

```tablegen
//===----------------------------------------------------------------------===//
///
/// \file
/// WebAssembly datatype conversions, truncations, reinterpretations,
/// promotions, and demotions operand code-gen constructs.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 13-22

```tablegen
//===----------------------------------------------------------------------===//

defm I32_WRAP_I64 : I<(outs I32:$dst), (ins I64:$src), (outs), (ins),
                      [(set I32:$dst, (trunc I64:$src))],
                      "i32.wrap_i64\t$dst, $src", "i32.wrap_i64", 0xa7>;

defm I64_EXTEND_S_I32 : I<(outs I64:$dst), (ins I32:$src), (outs), (ins),
                          [(set I64:$dst, (sext I32:$src))],
                          "i64.extend_i32_s\t$dst, $src", "i64.extend_i32_s",
                          0xac>;
```
- **EN**: Adds declarative TableGen records such as `I32_WRAP_I64`, `I64_EXTEND_S_I32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I32_WRAP_I64`, `I64_EXTEND_S_I32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 23-32

```tablegen
defm I64_EXTEND_U_I32 : I<(outs I64:$dst), (ins I32:$src), (outs), (ins),
                          [(set I64:$dst, (zext I32:$src))],
                          "i64.extend_i32_u\t$dst, $src", "i64.extend_i32_u",
                          0xad>;

let Predicates = [HasSignExt] in {
defm I32_EXTEND8_S_I32 : I<(outs I32:$dst), (ins I32:$src), (outs), (ins),
                           [(set I32:$dst, (sext_inreg I32:$src, i8))],
                           "i32.extend8_s\t$dst, $src", "i32.extend8_s",
                           0xc0>;
```
- **EN**: Adds declarative TableGen records such as `I64_EXTEND_U_I32`, `I32_EXTEND8_S_I32` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I64_EXTEND_U_I32`, `I32_EXTEND8_S_I32`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 33-40

```tablegen
defm I32_EXTEND16_S_I32 : I<(outs I32:$dst), (ins I32:$src), (outs), (ins),
                            [(set I32:$dst, (sext_inreg I32:$src, i16))],
                            "i32.extend16_s\t$dst, $src", "i32.extend16_s",
                            0xc1>;
defm I64_EXTEND8_S_I64 : I<(outs I64:$dst), (ins I64:$src), (outs), (ins),
                            [(set I64:$dst, (sext_inreg I64:$src, i8))],
                            "i64.extend8_s\t$dst, $src", "i64.extend8_s",
                            0xc2>;
```
- **EN**: Adds declarative TableGen records such as `I32_EXTEND16_S_I32`, `I64_EXTEND8_S_I64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I32_EXTEND16_S_I32`, `I64_EXTEND8_S_I64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 41-53

```tablegen
defm I64_EXTEND16_S_I64 : I<(outs I64:$dst), (ins I64:$src), (outs), (ins),
                            [(set I64:$dst, (sext_inreg I64:$src, i16))],
                            "i64.extend16_s\t$dst, $src", "i64.extend16_s",
                            0xc3>;
defm I64_EXTEND32_S_I64 : I<(outs I64:$dst), (ins I64:$src), (outs), (ins),
                            [(set I64:$dst, (sext_inreg I64:$src, i32))],
                            "i64.extend32_s\t$dst, $src", "i64.extend32_s",
                            0xc4>;
} // Predicates = [HasSignExt]

// Expand a "don't care" extend into zero-extend (chosen over sign-extend
// somewhat arbitrarily, although it favors popular hardware architectures
// and is conceptually a simpler operation).
```
- **EN**: Adds declarative TableGen records such as `I64_EXTEND16_S_I64`, `I64_EXTEND32_S_I64` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I64_EXTEND16_S_I64`, `I64_EXTEND32_S_I64`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 54-62

```tablegen
def : Pat<(i64 (anyext I32:$src)), (I64_EXTEND_U_I32 I32:$src)>;

// Conversion from floating point to integer instructions which don't trap on
// overflow or invalid.
defm I32_TRUNC_S_SAT_F32 : I<(outs I32:$dst), (ins F32:$src), (outs), (ins),
                             [(set I32:$dst, (fp_to_sint F32:$src))],
                             "i32.trunc_sat_f32_s\t$dst, $src",
                             "i32.trunc_sat_f32_s", 0xfc00>,
                             Requires<[HasNontrappingFPToInt]>;
```
- **EN**: Adds declarative TableGen records such as `I32_TRUNC_S_SAT_F32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I32_TRUNC_S_SAT_F32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 63-72

```tablegen
defm I32_TRUNC_U_SAT_F32 : I<(outs I32:$dst), (ins F32:$src), (outs), (ins),
                             [(set I32:$dst, (fp_to_uint F32:$src))],
                             "i32.trunc_sat_f32_u\t$dst, $src",
                             "i32.trunc_sat_f32_u", 0xfc01>,
                             Requires<[HasNontrappingFPToInt]>;
defm I64_TRUNC_S_SAT_F32 : I<(outs I64:$dst), (ins F32:$src), (outs), (ins),
                             [(set I64:$dst, (fp_to_sint F32:$src))],
                             "i64.trunc_sat_f32_s\t$dst, $src",
                             "i64.trunc_sat_f32_s", 0xfc04>,
                             Requires<[HasNontrappingFPToInt]>;
```
- **EN**: Adds declarative TableGen records such as `I32_TRUNC_U_SAT_F32`, `I64_TRUNC_S_SAT_F32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I32_TRUNC_U_SAT_F32`, `I64_TRUNC_S_SAT_F32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 73-82

```tablegen
defm I64_TRUNC_U_SAT_F32 : I<(outs I64:$dst), (ins F32:$src), (outs), (ins),
                             [(set I64:$dst, (fp_to_uint F32:$src))],
                             "i64.trunc_sat_f32_u\t$dst, $src",
                             "i64.trunc_sat_f32_u", 0xfc05>,
                             Requires<[HasNontrappingFPToInt]>;
defm I32_TRUNC_S_SAT_F64 : I<(outs I32:$dst), (ins F64:$src), (outs), (ins),
                             [(set I32:$dst, (fp_to_sint F64:$src))],
                             "i32.trunc_sat_f64_s\t$dst, $src",
                             "i32.trunc_sat_f64_s", 0xfc02>,
                             Requires<[HasNontrappingFPToInt]>;
```
- **EN**: Adds declarative TableGen records such as `I64_TRUNC_U_SAT_F32`, `I32_TRUNC_S_SAT_F64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I64_TRUNC_U_SAT_F32`, `I32_TRUNC_S_SAT_F64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 83-92

```tablegen
defm I32_TRUNC_U_SAT_F64 : I<(outs I32:$dst), (ins F64:$src), (outs), (ins),
                             [(set I32:$dst, (fp_to_uint F64:$src))],
                             "i32.trunc_sat_f64_u\t$dst, $src",
                             "i32.trunc_sat_f64_u", 0xfc03>,
                             Requires<[HasNontrappingFPToInt]>;
defm I64_TRUNC_S_SAT_F64 : I<(outs I64:$dst), (ins F64:$src), (outs), (ins),
                             [(set I64:$dst, (fp_to_sint F64:$src))],
                             "i64.trunc_sat_f64_s\t$dst, $src",
                             "i64.trunc_sat_f64_s", 0xfc06>,
                             Requires<[HasNontrappingFPToInt]>;
```
- **EN**: Adds declarative TableGen records such as `I32_TRUNC_U_SAT_F64`, `I64_TRUNC_S_SAT_F64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I32_TRUNC_U_SAT_F64`, `I64_TRUNC_S_SAT_F64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 93-99

```tablegen
defm I64_TRUNC_U_SAT_F64 : I<(outs I64:$dst), (ins F64:$src), (outs), (ins),
                             [(set I64:$dst, (fp_to_uint F64:$src))],
                             "i64.trunc_sat_f64_u\t$dst, $src",
                             "i64.trunc_sat_f64_u", 0xfc07>,
                             Requires<[HasNontrappingFPToInt]>;

// Support the explicitly saturating operations as well.
```
- **EN**: Adds declarative TableGen records such as `I64_TRUNC_U_SAT_F64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I64_TRUNC_U_SAT_F64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 100-106

```tablegen
def : Pat<(fp_to_sint_sat F32:$src, i32), (I32_TRUNC_S_SAT_F32 F32:$src)>;
def : Pat<(fp_to_uint_sat F32:$src, i32), (I32_TRUNC_U_SAT_F32 F32:$src)>;
def : Pat<(fp_to_sint_sat F64:$src, i32), (I32_TRUNC_S_SAT_F64 F64:$src)>;
def : Pat<(fp_to_uint_sat F64:$src, i32), (I32_TRUNC_U_SAT_F64 F64:$src)>;
def : Pat<(fp_to_sint_sat F32:$src, i64), (I64_TRUNC_S_SAT_F32 F32:$src)>;
def : Pat<(fp_to_uint_sat F32:$src, i64), (I64_TRUNC_U_SAT_F32 F32:$src)>;
def : Pat<(fp_to_sint_sat F64:$src, i64), (I64_TRUNC_S_SAT_F64 F64:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 107-114

```tablegen
def : Pat<(fp_to_uint_sat F64:$src, i64), (I64_TRUNC_U_SAT_F64 F64:$src)>;

// Conversion from floating point to integer pseudo-instructions which don't
// trap on overflow or invalid.
let usesCustomInserter = 1, isCodeGenOnly = 1 in {
defm FP_TO_SINT_I32_F32 : I<(outs I32:$dst), (ins F32:$src), (outs), (ins),
                            [(set I32:$dst, (fp_to_sint F32:$src))], "", "", 0>,
                            Requires<[NotHasNontrappingFPToInt]>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Conversion from floating point to integer pseudo-instructions which don't". Notable symbols in this range include `FP_TO_SINT_I32_F32`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Conversion from floating point to integer pseudo-instructions which don't”。 该区间中较显眼的符号包括 `FP_TO_SINT_I32_F32`。

### Lines 115-123

```tablegen
defm FP_TO_UINT_I32_F32 : I<(outs I32:$dst), (ins F32:$src), (outs), (ins),
                            [(set I32:$dst, (fp_to_uint F32:$src))], "", "", 0>,
                            Requires<[NotHasNontrappingFPToInt]>;
defm FP_TO_SINT_I64_F32 : I<(outs I64:$dst), (ins F32:$src), (outs), (ins),
                            [(set I64:$dst, (fp_to_sint F32:$src))], "", "", 0>,
                            Requires<[NotHasNontrappingFPToInt]>;
defm FP_TO_UINT_I64_F32 : I<(outs I64:$dst), (ins F32:$src), (outs), (ins),
                            [(set I64:$dst, (fp_to_uint F32:$src))], "", "", 0>,
                            Requires<[NotHasNontrappingFPToInt]>;
```
- **EN**: Adds declarative TableGen records such as `FP_TO_UINT_I32_F32`, `FP_TO_SINT_I64_F32`, `FP_TO_UINT_I64_F32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FP_TO_UINT_I32_F32`, `FP_TO_SINT_I64_F32`, `FP_TO_UINT_I64_F32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 124-132

```tablegen
defm FP_TO_SINT_I32_F64 : I<(outs I32:$dst), (ins F64:$src), (outs), (ins),
                            [(set I32:$dst, (fp_to_sint F64:$src))], "", "", 0>,
                            Requires<[NotHasNontrappingFPToInt]>;
defm FP_TO_UINT_I32_F64 : I<(outs I32:$dst), (ins F64:$src), (outs), (ins),
                            [(set I32:$dst, (fp_to_uint F64:$src))], "", "", 0>,
                            Requires<[NotHasNontrappingFPToInt]>;
defm FP_TO_SINT_I64_F64 : I<(outs I64:$dst), (ins F64:$src), (outs), (ins),
                            [(set I64:$dst, (fp_to_sint F64:$src))], "", "", 0>,
                            Requires<[NotHasNontrappingFPToInt]>;
```
- **EN**: Adds declarative TableGen records such as `FP_TO_SINT_I32_F64`, `FP_TO_UINT_I32_F64`, `FP_TO_SINT_I64_F64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FP_TO_SINT_I32_F64`, `FP_TO_UINT_I32_F64`, `FP_TO_SINT_I64_F64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 133-139

```tablegen
defm FP_TO_UINT_I64_F64 : I<(outs I64:$dst), (ins F64:$src), (outs), (ins),
                            [(set I64:$dst, (fp_to_uint F64:$src))], "", "", 0>,
                            Requires<[NotHasNontrappingFPToInt]>;
} // usesCustomInserter, isCodeGenOnly = 1

// Conversion from floating point to integer traps on overflow and invalid.
let hasSideEffects = 1 in {
```
- **EN**: Adds declarative TableGen records such as `FP_TO_UINT_I64_F64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FP_TO_UINT_I64_F64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 140-148

```tablegen
defm I32_TRUNC_S_F32 : I<(outs I32:$dst), (ins F32:$src), (outs), (ins),
                         [], "i32.trunc_f32_s\t$dst, $src", "i32.trunc_f32_s",
                         0xa8>;
defm I32_TRUNC_U_F32 : I<(outs I32:$dst), (ins F32:$src), (outs), (ins),
                         [], "i32.trunc_f32_u\t$dst, $src", "i32.trunc_f32_u",
                         0xa9>;
defm I64_TRUNC_S_F32 : I<(outs I64:$dst), (ins F32:$src), (outs), (ins),
                         [], "i64.trunc_f32_s\t$dst, $src", "i64.trunc_f32_s",
                         0xae>;
```
- **EN**: Adds declarative TableGen records such as `I32_TRUNC_S_F32`, `I32_TRUNC_U_F32`, `I64_TRUNC_S_F32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I32_TRUNC_S_F32`, `I32_TRUNC_U_F32`, `I64_TRUNC_S_F32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 149-157

```tablegen
defm I64_TRUNC_U_F32 : I<(outs I64:$dst), (ins F32:$src), (outs), (ins),
                         [], "i64.trunc_f32_u\t$dst, $src", "i64.trunc_f32_u",
                         0xaf>;
defm I32_TRUNC_S_F64 : I<(outs I32:$dst), (ins F64:$src), (outs), (ins),
                         [], "i32.trunc_f64_s\t$dst, $src", "i32.trunc_f64_s",
                         0xaa>;
defm I32_TRUNC_U_F64 : I<(outs I32:$dst), (ins F64:$src), (outs), (ins),
                         [], "i32.trunc_f64_u\t$dst, $src", "i32.trunc_f64_u",
                         0xab>;
```
- **EN**: Adds declarative TableGen records such as `I64_TRUNC_U_F32`, `I32_TRUNC_S_F64`, `I32_TRUNC_U_F64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I64_TRUNC_U_F32`, `I32_TRUNC_S_F64`, `I32_TRUNC_U_F64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 158-165

```tablegen
defm I64_TRUNC_S_F64 : I<(outs I64:$dst), (ins F64:$src), (outs), (ins),
                         [], "i64.trunc_f64_s\t$dst, $src", "i64.trunc_f64_s",
                         0xb0>;
defm I64_TRUNC_U_F64 : I<(outs I64:$dst), (ins F64:$src), (outs), (ins),
                         [], "i64.trunc_f64_u\t$dst, $src", "i64.trunc_f64_u",
                         0xb1>;
} // hasSideEffects = 1
```
- **EN**: Adds declarative TableGen records such as `I64_TRUNC_S_F64`, `I64_TRUNC_U_F64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I64_TRUNC_S_F64`, `I64_TRUNC_U_F64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 166-173

```tablegen
def : Pat<(int_wasm_trunc_signed F32:$src),
          (I32_TRUNC_S_F32 F32:$src)>;
def : Pat<(int_wasm_trunc_unsigned F32:$src),
          (I32_TRUNC_U_F32 F32:$src)>;
def : Pat<(int_wasm_trunc_signed F64:$src),
          (I32_TRUNC_S_F64 F64:$src)>;
def : Pat<(int_wasm_trunc_unsigned F64:$src),
          (I32_TRUNC_U_F64 F64:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 174-182

```tablegen
def : Pat<(int_wasm_trunc_signed F32:$src),
          (I64_TRUNC_S_F32 F32:$src)>;
def : Pat<(int_wasm_trunc_unsigned F32:$src),
          (I64_TRUNC_U_F32 F32:$src)>;
def : Pat<(int_wasm_trunc_signed F64:$src),
          (I64_TRUNC_S_F64 F64:$src)>;
def : Pat<(int_wasm_trunc_unsigned F64:$src),
          (I64_TRUNC_U_F64 F64:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 183-190

```tablegen
defm F32_CONVERT_S_I32 : I<(outs F32:$dst), (ins I32:$src), (outs), (ins),
                           [(set F32:$dst, (sint_to_fp I32:$src))],
                           "f32.convert_i32_s\t$dst, $src", "f32.convert_i32_s",
                           0xb2>;
defm F32_CONVERT_U_I32 : I<(outs F32:$dst), (ins I32:$src), (outs), (ins),
                           [(set F32:$dst, (uint_to_fp I32:$src))],
                           "f32.convert_i32_u\t$dst, $src", "f32.convert_i32_u",
                           0xb3>;
```
- **EN**: Adds declarative TableGen records such as `F32_CONVERT_S_I32`, `F32_CONVERT_U_I32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `F32_CONVERT_S_I32`, `F32_CONVERT_U_I32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 191-198

```tablegen
defm F64_CONVERT_S_I32 : I<(outs F64:$dst), (ins I32:$src), (outs), (ins),
                           [(set F64:$dst, (sint_to_fp I32:$src))],
                           "f64.convert_i32_s\t$dst, $src", "f64.convert_i32_s",
                           0xb7>;
defm F64_CONVERT_U_I32 : I<(outs F64:$dst), (ins I32:$src), (outs), (ins),
                           [(set F64:$dst, (uint_to_fp I32:$src))],
                           "f64.convert_i32_u\t$dst, $src", "f64.convert_i32_u",
                           0xb8>;
```
- **EN**: Adds declarative TableGen records such as `F64_CONVERT_S_I32`, `F64_CONVERT_U_I32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `F64_CONVERT_S_I32`, `F64_CONVERT_U_I32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 199-206

```tablegen
defm F32_CONVERT_S_I64 : I<(outs F32:$dst), (ins I64:$src), (outs), (ins),
                           [(set F32:$dst, (sint_to_fp I64:$src))],
                           "f32.convert_i64_s\t$dst, $src", "f32.convert_i64_s",
                           0xb4>;
defm F32_CONVERT_U_I64 : I<(outs F32:$dst), (ins I64:$src), (outs), (ins),
                           [(set F32:$dst, (uint_to_fp I64:$src))],
                           "f32.convert_i64_u\t$dst, $src", "f32.convert_i64_u",
                           0xb5>;
```
- **EN**: Adds declarative TableGen records such as `F32_CONVERT_S_I64`, `F32_CONVERT_U_I64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `F32_CONVERT_S_I64`, `F32_CONVERT_U_I64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 207-215

```tablegen
defm F64_CONVERT_S_I64 : I<(outs F64:$dst), (ins I64:$src), (outs), (ins),
                           [(set F64:$dst, (sint_to_fp I64:$src))],
                           "f64.convert_i64_s\t$dst, $src", "f64.convert_i64_s",
                           0xb9>;
defm F64_CONVERT_U_I64 : I<(outs F64:$dst), (ins I64:$src), (outs), (ins),
                           [(set F64:$dst, (uint_to_fp I64:$src))],
                           "f64.convert_i64_u\t$dst, $src", "f64.convert_i64_u",
                           0xba>;
```
- **EN**: Adds declarative TableGen records such as `F64_CONVERT_S_I64`, `F64_CONVERT_U_I64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `F64_CONVERT_S_I64`, `F64_CONVERT_U_I64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 216-224

```tablegen
defm F64_PROMOTE_F32 : I<(outs F64:$dst), (ins F32:$src), (outs), (ins),
                         [(set F64:$dst, (fpextend F32:$src))],
                         "f64.promote_f32\t$dst, $src", "f64.promote_f32",
                         0xbb>;
defm F32_DEMOTE_F64 : I<(outs F32:$dst), (ins F64:$src), (outs), (ins),
                        [(set F32:$dst, (fpround F64:$src))],
                        "f32.demote_f64\t$dst, $src", "f32.demote_f64",
                        0xb6>;
```
- **EN**: Adds declarative TableGen records such as `F64_PROMOTE_F32`, `F32_DEMOTE_F64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `F64_PROMOTE_F32`, `F32_DEMOTE_F64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 225-232

```tablegen
defm I32_REINTERPRET_F32 : I<(outs I32:$dst), (ins F32:$src), (outs), (ins),
                             [(set I32:$dst, (bitconvert F32:$src))],
                             "i32.reinterpret_f32\t$dst, $src",
                             "i32.reinterpret_f32", 0xbc>;
defm F32_REINTERPRET_I32 : I<(outs F32:$dst), (ins I32:$src), (outs), (ins),
                             [(set F32:$dst, (bitconvert I32:$src))],
                             "f32.reinterpret_i32\t$dst, $src",
                             "f32.reinterpret_i32", 0xbe>;
```
- **EN**: Adds declarative TableGen records such as `I32_REINTERPRET_F32`, `F32_REINTERPRET_I32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I32_REINTERPRET_F32`, `F32_REINTERPRET_I32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 233-240

```tablegen
defm I64_REINTERPRET_F64 : I<(outs I64:$dst), (ins F64:$src), (outs), (ins),
                             [(set I64:$dst, (bitconvert F64:$src))],
                             "i64.reinterpret_f64\t$dst, $src",
                             "i64.reinterpret_f64", 0xbd>;
defm F64_REINTERPRET_I64 : I<(outs F64:$dst), (ins I64:$src), (outs), (ins),
                             [(set F64:$dst, (bitconvert I64:$src))],
                             "f64.reinterpret_i64\t$dst, $src",
                             "f64.reinterpret_i64", 0xbf>;
```
- **EN**: Adds declarative TableGen records such as `I64_REINTERPRET_F64`, `F64_REINTERPRET_I64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I64_REINTERPRET_F64`, `F64_REINTERPRET_I64`，LLVM 随后会把它们展开成生成表或辅助代码。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- Generated backend tables / 生成式后端表
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
