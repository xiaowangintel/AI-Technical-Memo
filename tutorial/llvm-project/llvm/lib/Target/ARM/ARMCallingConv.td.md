# ARMCallingConv.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMCallingConv.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `ARMCallingConv` in LLVM TableGen DSL for the ARM backend, covering calling-convention rules and ABI lowering.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMCallingConv`，涵盖调用约定规则与 ABI 降级。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```tablegen
//===-- ARMCallingConv.td - Calling Conventions for ARM ----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
// This describes the calling conventions for ARM architecture.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 11-13
```tablegen
/// CCIfAlign - Match of the original alignment of the arg
class CCIfAlign<string Align, CCAction A>:
  CCIf<!strconcat("ArgFlags.getNonZeroOrigAlign() == ", Align), A>;
```
- EN: Declares reusable TableGen class `CCIfAlign` for `ARMCallingConv`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMCallingConv` 声明可复用的 TableGen 类 `CCIfAlign`，通常用于抽象共享字段、谓词或编码结构。

### Lines 15-19
```tablegen
//===----------------------------------------------------------------------===//
// ARM APCS Calling Convention
//===----------------------------------------------------------------------===//
let Entry = 1 in
def CC_ARM_APCS : CallingConv<[
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 21-22
```tablegen
  // Handles byval parameters.
  CCIfByVal<CCPassByVal<4, 4>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 24-24
```tablegen
  CCIfType<[i1, i8, i16], CCPromoteToType<i32>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 26-27
```tablegen
  // Pass SwiftSelf in a callee saved register.
  CCIfSwiftSelf<CCIfType<[i32], CCAssignToReg<[R10]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 29-30
```tablegen
  // A SwiftError is passed in R8.
  CCIfSwiftError<CCIfType<[i32], CCAssignToReg<[R8]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 32-34
```tablegen
  // Handle all vector types as either f64 or v2f64.
  CCIfType<[v1i64, v2i32, v4i16, v4f16, v4bf16, v8i8, v2f32], CCBitConvertToType<f64>>,
  CCIfType<[v2i64, v4i32, v8i16, v8f16, v8bf16, v16i8, v4f32], CCBitConvertToType<v2f64>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 36-37
```tablegen
  // f64 and v2f64 are passed in adjacent GPRs, possibly split onto the stack
  CCIfType<[f64, v2f64], CCCustom<"CC_ARM_APCS_Custom_f64">>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 39-40
```tablegen
  CCIfType<[f32], CCBitConvertToType<i32>>,
  CCIfType<[i32], CCAssignToReg<[R0, R1, R2, R3]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 42-45
```tablegen
  CCIfType<[i32], CCAssignToStack<4, 4>>,
  CCIfType<[f64], CCAssignToStack<8, 4>>,
  CCIfType<[v2f64], CCAssignToStack<16, 4>>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 47-50
```tablegen
let Entry = 1 in
def RetCC_ARM_APCS : CallingConv<[
  CCIfType<[i1, i8, i16], CCPromoteToType<i32>>,
  CCIfType<[f32], CCBitConvertToType<i32>>,
```
- EN: Defines TableGen record `RetCC_ARM_APCS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RetCC_ARM_APCS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 52-53
```tablegen
  // Pass SwiftSelf in a callee saved register.
  CCIfSwiftSelf<CCIfType<[i32], CCAssignToReg<[R10]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 55-56
```tablegen
  // A SwiftError is returned in R8.
  CCIfSwiftError<CCIfType<[i32], CCAssignToReg<[R8]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 58-60
```tablegen
  // Handle all vector types as either f64 or v2f64.
  CCIfType<[v1i64, v2i32, v4i16, v4f16, v4bf16, v8i8, v2f32], CCBitConvertToType<f64>>,
  CCIfType<[v2i64, v4i32, v8i16, v8f16, v8bf16, v16i8, v4f32], CCBitConvertToType<v2f64>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 62-62
```tablegen
  CCIfType<[f64, v2f64], CCCustom<"RetCC_ARM_APCS_Custom_f64">>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 64-66
```tablegen
  CCIfType<[i32], CCAssignToReg<[R0, R1, R2, R3]>>,
  CCIfType<[i64], CCAssignToRegWithShadow<[R0, R2], [R1, R3]>>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 68-75
```tablegen
//===----------------------------------------------------------------------===//
// ARM APCS Calling Convention for FastCC (when VFP2 or later is available)
//===----------------------------------------------------------------------===//
let Entry = 1 in
def FastCC_ARM_APCS : CallingConv<[
  // Handle all vector types as either f64 or v2f64.
  CCIfType<[v1i64, v2i32, v4i16, v4f16, v4bf16, v8i8, v2f32], CCBitConvertToType<f64>>,
  CCIfType<[v2i64, v4i32, v8i16, v8f16, v8bf16, v16i8, v4f32], CCBitConvertToType<v2f64>>,
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 77-80
```tablegen
  CCIfType<[v2f64], CCAssignToReg<[Q0, Q1, Q2, Q3]>>,
  CCIfType<[f64], CCAssignToReg<[D0, D1, D2, D3, D4, D5, D6, D7]>>,
  CCIfType<[f32], CCAssignToReg<[S0, S1, S2, S3, S4, S5, S6, S7, S8,
                                 S9, S10, S11, S12, S13, S14, S15]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 82-86
```tablegen
  // CPRCs may be allocated to co-processor registers or the stack - they
  // may never be allocated to core registers.
  CCIfType<[f32], CCAssignToStackWithShadow<4, 4, [Q0, Q1, Q2, Q3]>>,
  CCIfType<[f64], CCAssignToStackWithShadow<8, 4, [Q0, Q1, Q2, Q3]>>,
  CCIfType<[v2f64], CCAssignToStackWithShadow<16, 4, [Q0, Q1, Q2, Q3]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 88-89
```tablegen
  CCDelegateTo<CC_ARM_APCS>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 91-95
```tablegen
let Entry = 1 in
def RetFastCC_ARM_APCS : CallingConv<[
  // Handle all vector types as either f64 or v2f64.
  CCIfType<[v1i64, v2i32, v4i16, v4f16, v4bf16, v8i8, v2f32], CCBitConvertToType<f64>>,
  CCIfType<[v2i64, v4i32, v8i16, v8f16, v8bf16, v16i8, v4f32], CCBitConvertToType<v2f64>>,
```
- EN: Defines TableGen record `RetFastCC_ARM_APCS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RetFastCC_ARM_APCS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 97-102
```tablegen
  CCIfType<[v2f64], CCAssignToReg<[Q0, Q1, Q2, Q3]>>,
  CCIfType<[f64], CCAssignToReg<[D0, D1, D2, D3, D4, D5, D6, D7]>>,
  CCIfType<[f32], CCAssignToReg<[S0, S1, S2, S3, S4, S5, S6, S7, S8,
                                 S9, S10, S11, S12, S13, S14, S15]>>,
  CCDelegateTo<RetCC_ARM_APCS>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 104-106
```tablegen
//===----------------------------------------------------------------------===//
// ARM APCS Calling Convention for GHC
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 108-112
```tablegen
let Entry = 1 in
def CC_ARM_APCS_GHC : CallingConv<[
  // Handle all vector types as either f64 or v2f64.
  CCIfType<[v1i64, v2i32, v4i16, v4f16, v4bf16, v8i8, v2f32], CCBitConvertToType<f64>>,
  CCIfType<[v2i64, v4i32, v8i16, v8f16, v8bf16, v16i8, v4f32], CCBitConvertToType<v2f64>>,
```
- EN: Defines TableGen record `CC_ARM_APCS_GHC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_ARM_APCS_GHC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 114-116
```tablegen
  CCIfType<[v2f64], CCAssignToReg<[Q4, Q5]>>,
  CCIfType<[f64], CCAssignToReg<[D8, D9, D10, D11]>>,
  CCIfType<[f32], CCAssignToReg<[S16, S17, S18, S19, S20, S21, S22, S23]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 118-119
```tablegen
  // Promote i8/i16 arguments to i32.
  CCIfType<[i8, i16], CCPromoteToType<i32>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 121-123
```tablegen
  // Pass in STG registers: Base, Sp, Hp, R1, R2, R3, R4, SpLim
  CCIfType<[i32], CCAssignToReg<[R4, R5, R6, R7, R8, R9, R10, R11]>>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 125-127
```tablegen
//===----------------------------------------------------------------------===//
// ARM AAPCS (EABI) Calling Convention, common parts
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 129-129
```tablegen
def CC_ARM_AAPCS_Common : CallingConv<[
```
- EN: Defines TableGen record `CC_ARM_AAPCS_Common` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_ARM_AAPCS_Common`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 131-131
```tablegen
  CCIfType<[i1, i8, i16], CCPromoteToType<i32>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 133-138
```tablegen
  // i64/f64 is passed in even pairs of GPRs
  // i64 is 8-aligned i32 here, so we may need to eat R1 as a pad register
  // (and the same is true for f64 if VFP is not enabled)
  CCIfType<[i32], CCIfAlign<"8", CCAssignToRegWithShadow<[R0, R2], [R0, R1]>>>,
  CCIfType<[i32], CCIf<"ArgFlags.getNonZeroOrigAlign() != Align(8)",
                       CCAssignToReg<[R0, R1, R2, R3]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 140-148
```tablegen
  CCIfType<[i32], CCIfAlign<"8", CCAssignToStackWithShadow<4, 8, [R0, R1, R2, R3]>>>,
  CCIfType<[i32], CCAssignToStackWithShadow<4, 4, [R0, R1, R2, R3]>>,
  CCIfType<[f32], CCAssignToStackWithShadow<4, 4, [Q0, Q1, Q2, Q3]>>,
  CCIfType<[f16, bf16], CCCustom<"CC_ARM_AAPCS_Common_Custom_f16_Stack">>,
  CCIfType<[f64], CCAssignToStackWithShadow<8, 8, [Q0, Q1, Q2, Q3]>>,
  CCIfType<[v2f64], CCIfAlign<"16",
           CCAssignToStackWithShadow<16, 16, [Q0, Q1, Q2, Q3]>>>,
  CCIfType<[v2f64], CCAssignToStackWithShadow<16, 8, [Q0, Q1, Q2, Q3]>>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 150-154
```tablegen
def RetCC_ARM_AAPCS_Common : CallingConv<[
  CCIfType<[i1, i8, i16], CCPromoteToType<i32>>,
  CCIfType<[i32], CCAssignToReg<[R0, R1, R2, R3]>>,
  CCIfType<[i64], CCAssignToRegWithShadow<[R0, R2], [R1, R3]>>
]>;
```
- EN: Defines TableGen record `RetCC_ARM_AAPCS_Common` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RetCC_ARM_AAPCS_Common`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 156-158
```tablegen
//===----------------------------------------------------------------------===//
// ARM AAPCS (EABI) Calling Convention
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 160-163
```tablegen
let Entry = 1 in
def CC_ARM_AAPCS : CallingConv<[
  // Handles byval parameters.
  CCIfByVal<CCPassByVal<4, 4>>,
```
- EN: Defines TableGen record `CC_ARM_AAPCS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_ARM_AAPCS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 165-166
```tablegen
  // The 'nest' parameter, if any, is passed in R12.
  CCIfNest<CCAssignToReg<[R12]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 168-170
```tablegen
  // Handle all vector types as either f64 or v2f64.
  CCIfType<[v1i64, v2i32, v4i16, v4f16, v4bf16, v8i8, v2f32], CCBitConvertToType<f64>>,
  CCIfType<[v2i64, v4i32, v8i16, v8f16, v8bf16, v16i8, v4f32], CCBitConvertToType<v2f64>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 172-173
```tablegen
  // Pass SwiftSelf in a callee saved register.
  CCIfSwiftSelf<CCIfType<[i32], CCAssignToReg<[R10]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 175-176
```tablegen
  // A SwiftError is passed in R8.
  CCIfSwiftError<CCIfType<[i32], CCAssignToReg<[R8]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 178-182
```tablegen
  CCIfType<[f64, v2f64], CCCustom<"CC_ARM_AAPCS_Custom_f64">>,
  CCIfType<[f32], CCBitConvertToType<i32>>,
  CCIfType<[f16, bf16], CCCustom<"CC_ARM_AAPCS_Custom_f16">>,
  CCDelegateTo<CC_ARM_AAPCS_Common>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 184-188
```tablegen
let Entry = 1 in
def RetCC_ARM_AAPCS : CallingConv<[
  // Handle all vector types as either f64 or v2f64.
  CCIfType<[v1i64, v2i32, v4i16, v4f16, v4bf16, v8i8, v2f32], CCBitConvertToType<f64>>,
  CCIfType<[v2i64, v4i32, v8i16, v8f16, v8bf16, v16i8, v4f32], CCBitConvertToType<v2f64>>,
```
- EN: Defines TableGen record `RetCC_ARM_AAPCS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RetCC_ARM_AAPCS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 190-191
```tablegen
  // Pass SwiftSelf in a callee saved register.
  CCIfSwiftSelf<CCIfType<[i32], CCAssignToReg<[R10]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 193-194
```tablegen
  // A SwiftError is returned in R8.
  CCIfSwiftError<CCIfType<[i32], CCAssignToReg<[R8]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 196-198
```tablegen
  CCIfType<[f64, v2f64], CCCustom<"RetCC_ARM_AAPCS_Custom_f64">>,
  CCIfType<[f32], CCBitConvertToType<i32>>,
  CCIfType<[f16, bf16], CCCustom<"CC_ARM_AAPCS_Custom_f16">>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 200-201
```tablegen
  CCDelegateTo<RetCC_ARM_AAPCS_Common>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 203-206
```tablegen
//===----------------------------------------------------------------------===//
// ARM AAPCS-VFP (EABI) Calling Convention
// Also used for FastCC (when VFP2 or later is available)
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 208-211
```tablegen
let Entry = 1 in
def CC_ARM_AAPCS_VFP : CallingConv<[
  // Handles byval parameters.
  CCIfByVal<CCPassByVal<4, 4>>,
```
- EN: Defines TableGen record `CC_ARM_AAPCS_VFP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_ARM_AAPCS_VFP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 213-215
```tablegen
  // Handle all vector types as either f64 or v2f64.
  CCIfType<[v1i64, v2i32, v4i16, v4f16, v4bf16, v8i8, v2f32], CCBitConvertToType<f64>>,
  CCIfType<[v2i64, v4i32, v8i16, v8f16, v8bf16, v16i8, v4f32], CCBitConvertToType<v2f64>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 217-218
```tablegen
  // Pass SwiftSelf in a callee saved register.
  CCIfSwiftSelf<CCIfType<[i32], CCAssignToReg<[R10]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 220-221
```tablegen
  // A SwiftError is passed in R8.
  CCIfSwiftError<CCIfType<[i32], CCAssignToReg<[R8]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 223-224
```tablegen
  // HFAs are passed in a contiguous block of registers, or on the stack
  CCIfConsecutiveRegs<CCCustom<"CC_ARM_AAPCS_Custom_Aggregate">>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 226-232
```tablegen
  CCIfType<[v2f64], CCAssignToReg<[Q0, Q1, Q2, Q3]>>,
  CCIfType<[f64], CCAssignToReg<[D0, D1, D2, D3, D4, D5, D6, D7]>>,
  CCIfType<[f32], CCAssignToReg<[S0, S1, S2, S3, S4, S5, S6, S7, S8,
                                 S9, S10, S11, S12, S13, S14, S15]>>,
  CCIfType<[f16, bf16], CCCustom<"CC_ARM_AAPCS_VFP_Custom_f16">>,
  CCDelegateTo<CC_ARM_AAPCS_Common>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 234-238
```tablegen
let Entry = 1 in
def RetCC_ARM_AAPCS_VFP : CallingConv<[
  // Handle all vector types as either f64 or v2f64.
  CCIfType<[v1i64, v2i32, v4i16, v4f16, v4bf16, v8i8, v2f32], CCBitConvertToType<f64>>,
  CCIfType<[v2i64, v4i32, v8i16, v8f16, v8bf16, v16i8, v4f32], CCBitConvertToType<v2f64>>,
```
- EN: Defines TableGen record `RetCC_ARM_AAPCS_VFP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RetCC_ARM_AAPCS_VFP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 240-241
```tablegen
  // Pass SwiftSelf in a callee saved register.
  CCIfSwiftSelf<CCIfType<[i32], CCAssignToReg<[R10]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 243-244
```tablegen
  // A SwiftError is returned in R8.
  CCIfSwiftError<CCIfType<[i32], CCAssignToReg<[R8]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 246-252
```tablegen
  CCIfType<[v2f64], CCAssignToReg<[Q0, Q1, Q2, Q3]>>,
  CCIfType<[f64], CCAssignToReg<[D0, D1, D2, D3, D4, D5, D6, D7]>>,
  CCIfType<[f32], CCAssignToReg<[S0, S1, S2, S3, S4, S5, S6, S7, S8,
                                 S9, S10, S11, S12, S13, S14, S15]>>,
  CCIfType<[f16, bf16], CCCustom<"CC_ARM_AAPCS_VFP_Custom_f16">>,
  CCDelegateTo<RetCC_ARM_AAPCS_Common>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 255-260
```tablegen
// Windows Control Flow Guard checks take a single argument (the target function
// address) and have no return value.
let Entry = 1 in
def CC_ARM_Win32_CFGuard_Check : CallingConv<[
  CCIfType<[i32], CCAssignToReg<[R0]>>
]>;
```
- EN: Defines TableGen record `CC_ARM_Win32_CFGuard_Check` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_ARM_Win32_CFGuard_Check`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 264-266
```tablegen
//===----------------------------------------------------------------------===//
// Callee-saved register lists.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 268-269
```tablegen
def CSR_NoRegs : CalleeSavedRegs<(add)>;
def CSR_FPRegs : CalleeSavedRegs<(add (sequence "D%u", 0, 31))>;
```
- EN: Defines TableGen record `CSR_NoRegs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_NoRegs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 271-273
```tablegen
def CSR_FP_Interrupt_Regs : CalleeSavedRegs<(add FPSCR, FPEXC, (sequence "D%u", 15, 0))>;
def CSR_FP_NEON_Interrupt_Regs : CalleeSavedRegs<(add CSR_FP_Interrupt_Regs, 
                                                  (sequence "D%u", 31, 16))>;
```
- EN: Defines TableGen record `CSR_FP_Interrupt_Regs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_FP_Interrupt_Regs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 275-276
```tablegen
def CSR_AAPCS : CalleeSavedRegs<(add LR, R11, R10, R9, R8, R7, R6, R5, R4,
                                     (sequence "D%u", 15, 8))>;
```
- EN: Defines TableGen record `CSR_AAPCS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_AAPCS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 278-278
```tablegen
def CSR_AAPCS_FP : CalleeSavedRegs<(add CSR_AAPCS, CSR_FP_Interrupt_Regs)>;
```
- EN: Defines TableGen record `CSR_AAPCS_FP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_AAPCS_FP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 280-286
```tablegen
// The order of callee-saved registers needs to match the order we actually push
// them in FrameLowering, because this order is what's used by
// PrologEpilogInserter to allocate frame index slots. So when R7 is the frame
// pointer, we use this ATPCS alternative.
def CSR_ATPCS_SplitPush : CalleeSavedRegs<(add LR, R7, R6, R5, R4,
                                               R11, R10, R9, R8,
                                               (sequence "D%u", 15, 8))>;
```
- EN: Defines TableGen record `CSR_ATPCS_SplitPush` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_ATPCS_SplitPush`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 288-290
```tablegen
def CSR_Win_SplitFP : CalleeSavedRegs<(add R10, R9, R8, R7, R6, R5, R4,
                                               (sequence "D%u", 15, 8),
                                               LR, R11)>;
```
- EN: Defines TableGen record `CSR_Win_SplitFP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_Win_SplitFP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 292-293
```tablegen
def CSR_ATPCS_SplitPush_FP : CalleeSavedRegs<(add CSR_ATPCS_SplitPush,
                                              CSR_FP_Interrupt_Regs)>;
```
- EN: Defines TableGen record `CSR_ATPCS_SplitPush_FP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_ATPCS_SplitPush_FP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 295-298
```tablegen
// The Windows Control Flow Guard Check function preserves the same registers as
// AAPCS, and also preserves all floating point registers.
def CSR_Win_AAPCS_CFGuard_Check : CalleeSavedRegs<(add LR, R11, R10, R9, R8, R7,
                                     R6, R5, R4, (sequence "D%u", 15, 0))>;
```
- EN: Defines TableGen record `CSR_Win_AAPCS_CFGuard_Check` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_Win_AAPCS_CFGuard_Check`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 300-301
```tablegen
// R8 is used to pass swifterror, remove it from CSR.
def CSR_AAPCS_SwiftError : CalleeSavedRegs<(sub CSR_AAPCS, R8)>;
```
- EN: Defines TableGen record `CSR_AAPCS_SwiftError` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_AAPCS_SwiftError`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 303-304
```tablegen
// R10 is used to pass swiftself, remove it from CSR.
def CSR_AAPCS_SwiftTail : CalleeSavedRegs<(sub CSR_AAPCS, R10)>;
```
- EN: Defines TableGen record `CSR_AAPCS_SwiftTail` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_AAPCS_SwiftTail`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 307-309
```tablegen
// R8 is used to pass swifterror, remove it from CSR.
def CSR_ATPCS_SplitPush_SwiftError : CalleeSavedRegs<(sub CSR_ATPCS_SplitPush,
                                                      R8)>;
```
- EN: Defines TableGen record `CSR_ATPCS_SplitPush_SwiftError` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_ATPCS_SplitPush_SwiftError`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 311-313
```tablegen
// R10 is used to pass swifterror, remove it from CSR.
def CSR_ATPCS_SplitPush_SwiftTail : CalleeSavedRegs<(sub CSR_ATPCS_SplitPush,
                                                     R10)>;
```
- EN: Defines TableGen record `CSR_ATPCS_SplitPush_SwiftTail` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_ATPCS_SplitPush_SwiftTail`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 315-325
```tablegen
// Sometimes we need to split the push of the callee-saved GPRs into two
// regions, to ensure that the frame chain record is set up correctly. These
// list the callee-saved registers in the order they end up on the stack, which
// depends on whether the frame pointer is r7 or r11.
def CSR_AAPCS_SplitPush_R11 : CalleeSavedRegs<(add R10, R9, R8, R7, R6, R5, R4,
                                                   LR, R11,
                                                   (sequence "D%u", 15, 8))>;
def CSR_AAPCS_SplitPush_R7 : CalleeSavedRegs<(add LR, R11,
                                                  R7, R6, R5, R4,
                                                  R10, R9, R8,
                                                  (sequence "D%u", 15, 8))>;
```
- EN: Defines TableGen record `CSR_AAPCS_SplitPush_R11` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_AAPCS_SplitPush_R11`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 327-333
```tablegen
// Constructors and destructors return 'this' in the ARM C++ ABI; since 'this'
// and the pointer return value are both passed in R0 in these cases, this can
// be partially modelled by treating R0 as a callee-saved register
// Only the resulting RegMask is used; the SaveList is ignored
def CSR_AAPCS_ThisReturn : CalleeSavedRegs<(add LR, R11, R10, R9, R8, R7, R6,
                                            R5, R4, (sequence "D%u", 15, 8),
                                            R0)>;
```
- EN: Defines TableGen record `CSR_AAPCS_ThisReturn` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_AAPCS_ThisReturn`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 335-337
```tablegen
// iOS ABI deviates from ARM standard ABI. R9 is not a callee-saved register.
// Also save R7-R4 first to match the stack frame fixed spill areas.
def CSR_iOS : CalleeSavedRegs<(add LR, R7, R6, R5, R4, (sub CSR_AAPCS, R9))>;
```
- EN: Defines TableGen record `CSR_iOS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_iOS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 339-340
```tablegen
// R8 is used to pass swifterror, remove it from CSR.
def CSR_iOS_SwiftError : CalleeSavedRegs<(sub CSR_iOS, R8)>;
```
- EN: Defines TableGen record `CSR_iOS_SwiftError` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_iOS_SwiftError`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 342-343
```tablegen
// R10 is used to pass swiftself, remove it from CSR.
def CSR_iOS_SwiftTail : CalleeSavedRegs<(sub CSR_iOS, R10)>;
```
- EN: Defines TableGen record `CSR_iOS_SwiftTail` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_iOS_SwiftTail`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 345-346
```tablegen
def CSR_iOS_ThisReturn : CalleeSavedRegs<(add LR, R7, R6, R5, R4,
                                         (sub CSR_AAPCS_ThisReturn, R9))>;
```
- EN: Defines TableGen record `CSR_iOS_ThisReturn` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_iOS_ThisReturn`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 348-350
```tablegen
def CSR_iOS_TLSCall
    : CalleeSavedRegs<(add LR, SP, (sub(sequence "R%u", 12, 1), R9, R12),
                      (sequence "D%u", 31, 0))>;
```
- EN: Defines TableGen record `CSR_iOS_TLSCall` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_iOS_TLSCall`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 352-355
```tablegen
// C++ TLS access function saves all registers except SP. Try to match
// the order of CSRs in CSR_iOS.
def CSR_iOS_CXX_TLS : CalleeSavedRegs<(add CSR_iOS, (sequence "R%u", 12, 1),
                                           (sequence "D%u", 31, 0))>;
```
- EN: Defines TableGen record `CSR_iOS_CXX_TLS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_iOS_CXX_TLS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 357-358
```tablegen
// CSRs that are handled by prologue, epilogue.
def CSR_iOS_CXX_TLS_PE : CalleeSavedRegs<(add LR, R12, R11, R7, R5, R4)>;
```
- EN: Defines TableGen record `CSR_iOS_CXX_TLS_PE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_iOS_CXX_TLS_PE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 360-362
```tablegen
// CSRs that are handled explicitly via copies.
def CSR_iOS_CXX_TLS_ViaCopy : CalleeSavedRegs<(sub CSR_iOS_CXX_TLS,
                                                   CSR_iOS_CXX_TLS_PE)>;
```
- EN: Defines TableGen record `CSR_iOS_CXX_TLS_ViaCopy` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_iOS_CXX_TLS_ViaCopy`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 364-367
```tablegen
// The "interrupt" attribute is used to generate code that is acceptable in
// exception-handlers of various kinds. It makes us use a different return
// instruction (handled elsewhere) and affects which registers we must return to
// our "caller" in the same state as we receive them.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 369-372
```tablegen
// For most interrupts, all registers except SP and LR are shared with
// user-space. We mark LR to be saved anyway, since this is what the ARM backend
// generally does rather than tracking its liveness as a normal register.
def CSR_GenericInt : CalleeSavedRegs<(add LR, (sequence "R%u", 12, 0))>;
```
- EN: Defines TableGen record `CSR_GenericInt` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_GenericInt`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 374-375
```tablegen
def CSR_GenericInt_FP : CalleeSavedRegs<(add CSR_GenericInt, 
                                         CSR_FP_Interrupt_Regs)>;
```
- EN: Defines TableGen record `CSR_GenericInt_FP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_GenericInt_FP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 377-378
```tablegen
def CSR_GenericInt_FP_NEON : CalleeSavedRegs<(add CSR_GenericInt_FP, 
                                              CSR_FP_NEON_Interrupt_Regs)>;
```
- EN: Defines TableGen record `CSR_GenericInt_FP_NEON` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_GenericInt_FP_NEON`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 381-382
```tablegen
// The fast interrupt handlers have more private state and get their own copies
// of R8-R12, in addition to SP and LR. As before, mark LR for saving too.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 384-387
```tablegen
// FIXME: we mark R11 as callee-saved since it's often the frame-pointer, and
// current frame lowering expects to encounter it while processing callee-saved
// registers.
def CSR_FIQ : CalleeSavedRegs<(add LR, R11, (sequence "R%u", 7, 0))>;
```
- EN: Defines TableGen record `CSR_FIQ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_FIQ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 389-389
```tablegen
def CSR_FIQ_FP : CalleeSavedRegs<(add CSR_FIQ, CSR_FP_Interrupt_Regs)>;
```
- EN: Defines TableGen record `CSR_FIQ_FP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_FIQ_FP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 391-392
```tablegen
def CSR_FIQ_FP_NEON : CalleeSavedRegs<(add CSR_FIQ_FP, 
                                       CSR_FP_NEON_Interrupt_Regs)>;
```
- EN: Defines TableGen record `CSR_FIQ_FP_NEON` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_FIQ_FP_NEON`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

## Key Concepts / 关键概念

- EN: Primary role: calling-convention rules and ABI lowering.
  - CN: 核心职责：调用约定规则与 ABI 降级。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
