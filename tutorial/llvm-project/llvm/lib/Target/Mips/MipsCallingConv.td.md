# MipsCallingConv.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsCallingConv.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `MipsCallingConv` in LLVM TableGen DSL for the Mips backend, covering calling-convention rules and ABI lowering.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsCallingConv`，涵盖调用约定规则与 ABI 降级。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```tablegen
//===-- MipsCallingConv.td - Calling Conventions for Mips --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
// This describes the calling conventions for Mips architecture.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 11-16
```tablegen
/// CCIfSubtarget - Match if the current subtarget has a feature F.
class CCIfSubtarget<string F, CCAction A, string Invert = "">
    : CCIf<!strconcat(Invert,
                      "State.getMachineFunction().getSubtarget<MipsSubtarget>().",
                      F),
           A>;
```
- EN: Declares reusable TableGen class `CCIfSubtarget` for `MipsCallingConv`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCallingConv` 声明可复用的 TableGen 类 `CCIfSubtarget`，通常用于抽象共享字段、谓词或编码结构。

### Lines 18-19
```tablegen
// The inverse of CCIfSubtarget
class CCIfSubtargetNot<string F, CCAction A> : CCIfSubtarget<F, A, "!">;
```
- EN: Declares reusable TableGen class `CCIfSubtargetNot` for `MipsCallingConv`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCallingConv` 声明可复用的 TableGen 类 `CCIfSubtargetNot`，通常用于抽象共享字段、谓词或编码结构。

### Lines 21-23
```tablegen
/// Match if the original argument (before lowering) was a float.
/// For example, this is true for i32's that were lowered from soft-float.
class CCIfOrigArgWasFloat<CCAction A> : CCIf<"OrigTy->isFloatingPointTy()", A>;
```
- EN: Declares reusable TableGen class `CCIfOrigArgWasFloat` for `MipsCallingConv`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCallingConv` 声明可复用的 TableGen 类 `CCIfOrigArgWasFloat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 25-27
```tablegen
/// Match if the original argument (before lowering) was a 128-bit float (i.e.
/// long double).
class CCIfOrigArgWasF128<CCAction A> : CCIf<"OrigTy->isFP128Ty()", A>;
```
- EN: Declares reusable TableGen class `CCIfOrigArgWasF128` for `MipsCallingConv`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCallingConv` 声明可复用的 TableGen 类 `CCIfOrigArgWasF128`，通常用于抽象共享字段、谓词或编码结构。

### Lines 29-31
```tablegen
/// Match if the return was not a floating point vector.
class CCIfOrigArgWasNotVectorFloat<CCAction A>
    : CCIf<"!OrigTy->isVectorTy() || !OrigTy->isFPOrFPVectorTy()", A>;
```
- EN: Declares reusable TableGen class `CCIfOrigArgWasNotVectorFloat` for `MipsCallingConv`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCallingConv` 声明可复用的 TableGen 类 `CCIfOrigArgWasNotVectorFloat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 33-36
```tablegen
/// Match if the special calling conv is the specified value.
class CCIfSpecialCallingConv<string CC, CCAction A>
    : CCIf<"static_cast<MipsCCState *>(&State)->getSpecialCallingConv() == "
               "MipsCCState::" # CC, A>;
```
- EN: Declares reusable TableGen class `CCIfSpecialCallingConv` for `MipsCallingConv`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCallingConv` 声明可复用的 TableGen 类 `CCIfSpecialCallingConv`，通常用于抽象共享字段、谓词或编码结构。

### Lines 38-41
```tablegen
// For soft-float, f128 values are returned in A0_64 rather than V1_64.
def RetCC_F128SoftFloat : CallingConv<[
  CCAssignToReg<[V0_64, A0_64]>
]>;
```
- EN: Defines TableGen record `RetCC_F128SoftFloat` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RetCC_F128SoftFloat`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 43-46
```tablegen
// For hard-float, f128 values are returned as a pair of f64's rather than a
// pair of i64's.
def RetCC_F128HardFloat : CallingConv<[
  CCBitConvertToType<f64>,
```
- EN: Defines TableGen record `RetCC_F128HardFloat` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RetCC_F128HardFloat`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 48-51
```tablegen
  // Contrary to the ABI documentation, a struct containing a long double is
  // returned in $f0, and $f1 instead of the usual $f0, and $f2. This is to
  // match the de facto ABI as implemented by GCC.
  CCIfInReg<CCAssignToReg<[D0_64, D1_64]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 53-54
```tablegen
  CCAssignToReg<[D0_64, D2_64]>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 56-63
```tablegen
// Handle F128 specially since we can't identify the original type during the
// tablegen-erated code.
def RetCC_F128 : CallingConv<[
  CCIfSubtarget<"useSoftFloat()",
      CCIfType<[i64], CCDelegateTo<RetCC_F128SoftFloat>>>,
  CCIfSubtargetNot<"useSoftFloat()",
      CCIfType<[i64], CCDelegateTo<RetCC_F128HardFloat>>>
]>;
```
- EN: Defines TableGen record `RetCC_F128` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RetCC_F128`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 65-67
```tablegen
//===----------------------------------------------------------------------===//
// Mips O32 Calling Convention
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 69-71
```tablegen
def CC_MipsO32 : CallingConv<[
  // Promote i8/i16 arguments to i32.
  CCIfType<[i1, i8, i16], CCPromoteToType<i32>>,
```
- EN: Defines TableGen record `CC_MipsO32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_MipsO32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 73-75
```tablegen
  // Integer values get stored in stack slots that are 4 bytes in
  // size and 4-byte aligned.
  CCIfType<[i32, f32], CCAssignToStack<4, 4>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 77-80
```tablegen
  // Integer values get stored in stack slots that are 8 bytes in
  // size and 8-byte aligned.
  CCIfType<[f64], CCAssignToStack<8, 8>>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 82-86
```tablegen
// Only the return rules are defined here for O32. The rules for argument
// passing are defined in MipsISelLowering.cpp.
def RetCC_MipsO32 : CallingConv<[
  // Promote i1/i8/i16 return values to i32.
  CCIfType<[i1, i8, i16], CCPromoteToType<i32>>,
```
- EN: Defines TableGen record `RetCC_MipsO32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RetCC_MipsO32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 88-91
```tablegen
  // i32 are returned in registers V0, V1, A0, A1, unless the original return
  // type was a vector of floats.
  CCIfOrigArgWasNotVectorFloat<CCIfType<[i32],
                                        CCAssignToReg<[V0, V1, A0, A1]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 93-94
```tablegen
  // f32 are returned in registers F0, F2
  CCIfType<[f32], CCAssignToReg<[F0, F2]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 96-100
```tablegen
  // f64 arguments are returned in D0_64 and D2_64 in FP64bit mode or
  // in D0 and D1 in FP32bit mode.
  CCIfType<[f64], CCIfSubtarget<"isFP64bit()", CCAssignToReg<[D0_64, D2_64]>>>,
  CCIfType<[f64], CCIfSubtargetNot<"isFP64bit()", CCAssignToReg<[D0, D1]>>>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 102-103
```tablegen
def CC_MipsO32_FP32 : CustomCallingConv;
def CC_MipsO32_FP64 : CustomCallingConv;
```
- EN: Defines TableGen record `CC_MipsO32_FP32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_MipsO32_FP32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 105-108
```tablegen
def CC_MipsO32_FP : CallingConv<[
  CCIfSubtargetNot<"isFP64bit()", CCDelegateTo<CC_MipsO32_FP32>>,
  CCIfSubtarget<"isFP64bit()", CCDelegateTo<CC_MipsO32_FP64>>
]>;
```
- EN: Defines TableGen record `CC_MipsO32_FP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_MipsO32_FP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 110-112
```tablegen
//===----------------------------------------------------------------------===//
// Mips N32/64 Calling Convention
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 114-120
```tablegen
def CC_MipsN_SoftFloat : CallingConv<[
  CCAssignToRegWithShadow<[A0, A1, A2, A3,
                           T0, T1, T2, T3],
                          [D12_64, D13_64, D14_64, D15_64,
                           D16_64, D17_64, D18_64, D19_64]>,
  CCAssignToStack<4, 8>
]>;
```
- EN: Defines TableGen record `CC_MipsN_SoftFloat` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_MipsN_SoftFloat`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 122-125
```tablegen
def CC_MipsN : CallingConv<[
  CCIfType<[i8, i16, i32, i64],
      CCIfSubtargetNot<"isLittle()",
          CCIfInReg<CCPromoteToUpperBitsInType<i64>>>>,
```
- EN: Defines TableGen record `CC_MipsN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_MipsN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 127-130
```tablegen
  // First deal with f32s that got turned into i32s due to soft-float
  CCIfType<[i32],
      CCIfSubtarget<"useSoftFloat()",
          CCIfOrigArgWasFloat<CCDelegateTo<CC_MipsN_SoftFloat>>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 132-133
```tablegen
  // All remaining integers are promoted to 64-bit.
  CCIfType<[i8, i16, i32], CCPromoteToType<i64>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 135-139
```tablegen
  // Integer arguments are passed in integer registers.
  CCIfType<[i64], CCAssignToRegWithShadow<[A0_64, A1_64, A2_64, A3_64,
                                           T0_64, T1_64, T2_64, T3_64],
                                          [D12_64, D13_64, D14_64, D15_64,
                                           D16_64, D17_64, D18_64, D19_64]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 141-145
```tablegen
  // f32 arguments are passed in single precision FP registers.
  CCIfType<[f32], CCAssignToRegWithShadow<[F12, F13, F14, F15,
                                           F16, F17, F18, F19],
                                          [A0_64, A1_64, A2_64, A3_64,
                                           T0_64, T1_64, T2_64, T3_64]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 147-151
```tablegen
  // f64 arguments are passed in double precision FP registers.
  CCIfType<[f64], CCAssignToRegWithShadow<[D12_64, D13_64, D14_64, D15_64,
                                           D16_64, D17_64, D18_64, D19_64],
                                          [A0_64, A1_64, A2_64, A3_64,
                                           T0_64, T1_64, T2_64, T3_64]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 153-156
```tablegen
  // All stack parameter slots become 64-bit doublewords and are 8-byte aligned.
  CCIfType<[f32], CCAssignToStack<4, 8>>,
  CCIfType<[i64, f64], CCAssignToStack<8, 8>>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 158-163
```tablegen
// N32/64 variable arguments.
// All arguments are passed in integer registers.
def CC_MipsN_VarArg : CallingConv<[
  CCIfType<[i8, i16, i32, i64],
      CCIfSubtargetNot<"isLittle()",
          CCIfInReg<CCPromoteToUpperBitsInType<i64>>>>,
```
- EN: Defines TableGen record `CC_MipsN_VarArg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_MipsN_VarArg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 165-166
```tablegen
  // All integers are promoted to 64-bit.
  CCIfType<[i8, i16, i32], CCPromoteToType<i64>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 168-168
```tablegen
  CCIfType<[f32], CCAssignToReg<[A0, A1, A2, A3, T0, T1, T2, T3]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 170-171
```tablegen
  CCIfType<[i64, f64], CCAssignToReg<[A0_64, A1_64, A2_64, A3_64,
                                      T0_64, T1_64, T2_64, T3_64]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 173-176
```tablegen
  // All stack parameter slots become 64-bit doublewords and are 8-byte aligned.
  CCIfType<[f32], CCAssignToStack<4, 8>>,
  CCIfType<[i64, f64], CCAssignToStack<8, 8>>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 178-190
```tablegen
def RetCC_MipsN : CallingConv<[
  // f128 needs to be handled similarly to f32 and f64. However, f128 is not
  // legal and is lowered to i128 which is further lowered to a pair of i64's.
  // This presents us with a problem for the calling convention since hard-float
  // still needs to pass them in FPU registers, and soft-float needs to use $v0,
  // and $a0 instead of the usual $v0, and $v1. We therefore resort to a
  // pre-analyze (see PreAnalyzeReturnForF128()) step to pass information on
  // whether the result was originally an f128 into the tablegen-erated code.
  //
  // f128 should only occur for the N64 ABI where long double is 128-bit. On
  // N32, long double is equivalent to double.
  CCIfSubtargetNot<"isSingleFloat()",
      CCIfType<[i64], CCIfOrigArgWasF128<CCDelegateTo<RetCC_F128>>>>,
```
- EN: Defines TableGen record `RetCC_MipsN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RetCC_MipsN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 192-199
```tablegen
  // Aggregate returns are positioned at the lowest address in the slot for
  // both little and big-endian targets. When passing in registers, this
  // requires that big-endian targets shift the value into the upper bits.
  CCIfSubtarget<"isLittle()",
      CCIfType<[i8, i16, i32, i64], CCIfInReg<CCPromoteToType<i64>>>>,
  CCIfSubtargetNot<"isLittle()",
      CCIfType<[i8, i16, i32, i64],
          CCIfInReg<CCPromoteToUpperBitsInType<i64>>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 201-202
```tablegen
  // i64 are returned in registers V0_64, V1_64
  CCIfType<[i64], CCAssignToReg<[V0_64, V1_64]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 204-205
```tablegen
  // f32 are returned in registers F0, F2
  CCIfType<[f32], CCAssignToReg<[F0, F2]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 207-209
```tablegen
  // f64 are returned in registers D0, D2
  CCIfType<[f64], CCAssignToReg<[D0_64, D2_64]>>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 211-228
```tablegen
//===----------------------------------------------------------------------===//
// Mips FastCC Calling Convention
//===----------------------------------------------------------------------===//
def CC_MipsO32_FastCC : CallingConv<[
  // f64 arguments are passed in double-precision floating pointer registers.
  CCIfType<[f64], CCIfSubtargetNot<"isFP64bit()",
                                   CCAssignToReg<[D0, D1, D2, D3, D4, D5, D6,
                                                  D7, D8, D9]>>>,
  CCIfType<[f64], CCIfSubtarget<"isFP64bit()", CCIfSubtarget<"useOddSPReg()",
                                CCAssignToReg<[D0_64, D1_64, D2_64, D3_64,
                                               D4_64, D5_64, D6_64, D7_64,
                                               D8_64, D9_64, D10_64, D11_64,
                                               D12_64, D13_64, D14_64, D15_64,
                                               D16_64, D17_64, D18_64,
                                               D19_64]>>>>,
  CCIfType<[f64], CCIfSubtarget<"isFP64bit()", CCIfSubtarget<"noOddSPReg()",
                                CCAssignToReg<[D0_64, D2_64, D4_64, D6_64,
                                               D8_64, D10_64, D12_64, D14_64,
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 229-229
```tablegen
                                               D16_64, D18_64]>>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 231-233
```tablegen
  // Stack parameter slots for f64 are 64-bit doublewords and 8-byte aligned.
  CCIfType<[f64], CCAssignToStack<8, 8>>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 235-239
```tablegen
def CC_MipsN_FastCC : CallingConv<[
  // Integer arguments are passed in integer registers.
  CCIfType<[i64], CCAssignToReg<[A0_64, A1_64, A2_64, A3_64, T0_64, T1_64,
                                 T2_64, T3_64, T4_64, T5_64, T6_64, T7_64,
                                 T8_64, V1_64]>>,
```
- EN: Defines TableGen record `CC_MipsN_FastCC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_MipsN_FastCC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 241-245
```tablegen
  // f64 arguments are passed in double-precision floating pointer registers.
  CCIfType<[f64], CCAssignToReg<[D0_64, D1_64, D2_64, D3_64, D4_64, D5_64,
                                 D6_64, D7_64, D8_64, D9_64, D10_64, D11_64,
                                 D12_64, D13_64, D14_64, D15_64, D16_64, D17_64,
                                 D18_64, D19_64]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 247-250
```tablegen
  // Stack parameter slots for i64 and f64 are 64-bit doublewords and
  // 8-byte aligned.
  CCIfType<[i64, f64], CCAssignToStack<8, 8>>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 252-254
```tablegen
def CC_Mips_FastCC : CallingConv<[
  // Handles byval parameters.
  CCIfByVal<CCPassByVal<4, 4>>,
```
- EN: Defines TableGen record `CC_Mips_FastCC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_Mips_FastCC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 256-257
```tablegen
  // Promote i8/i16 arguments to i32.
  CCIfType<[i8, i16], CCPromoteToType<i32>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 259-262
```tablegen
  // Integer arguments are passed in integer registers. All scratch registers,
  // except for AT, V0 and T9, are available to be used as argument registers.
  CCIfType<[i32],
      CCAssignToReg<[A0, A1, A2, A3, T0, T1, T2, T3, T4, T5, T6, T7, T8, V1]>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 264-267
```tablegen
  // f32 arguments are passed in single-precision floating pointer registers.
  CCIfType<[f32], CCIfSubtarget<"useOddSPReg()",
      CCAssignToReg<[F0, F1, F2, F3, F4, F5, F6, F7, F8, F9, F10, F11, F12, F13,
                     F14, F15, F16, F17, F18, F19]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 269-271
```tablegen
  // Don't use odd numbered single-precision registers for -mno-odd-spreg.
  CCIfType<[f32], CCIfSubtarget<"noOddSPReg()",
      CCAssignToReg<[F0, F2, F4, F6, F8, F10, F12, F14, F16, F18]>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 273-274
```tablegen
  // Stack parameter slots for i32 and f32 are 32-bit words and 4-byte aligned.
  CCIfType<[i32, f32], CCAssignToStack<4, 4>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 276-278
```tablegen
  CCIfSubtarget<"isABI_O32()", CCDelegateTo<CC_MipsO32_FastCC>>,
  CCDelegateTo<CC_MipsN_FastCC>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 280-282
```tablegen
//===----------------------------------------------------------------------===//
// Mips Calling Convention Dispatch
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 284-288
```tablegen
def RetCC_Mips : CallingConv<[
  CCIfSubtarget<"isABI_N32()", CCDelegateTo<RetCC_MipsN>>,
  CCIfSubtarget<"isABI_N64()", CCDelegateTo<RetCC_MipsN>>,
  CCDelegateTo<RetCC_MipsO32>
]>;
```
- EN: Defines TableGen record `RetCC_Mips` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RetCC_Mips`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 290-293
```tablegen
def CC_Mips_ByVal : CallingConv<[
  CCIfSubtarget<"isABI_O32()", CCIfByVal<CCPassByVal<4, 4>>>,
  CCIfByVal<CCPassByVal<8, 8>>
]>;
```
- EN: Defines TableGen record `CC_Mips_ByVal` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_Mips_ByVal`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 295-296
```tablegen
def CC_Mips16RetHelper : CallingConv<[
  CCIfByVal<CCDelegateTo<CC_Mips_ByVal>>,
```
- EN: Defines TableGen record `CC_Mips16RetHelper` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_Mips16RetHelper`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 298-300
```tablegen
  // Integer arguments are passed in integer registers.
  CCIfType<[i32], CCAssignToReg<[V0, V1, A0, A1]>>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 302-306
```tablegen
def CC_Mips_FixedArg : CallingConv<[
  // Mips16 needs special handling on some functions.
  CCIf<"State.getCallingConv() != CallingConv::Fast",
      CCIfSpecialCallingConv<"Mips16RetHelperConv",
           CCDelegateTo<CC_Mips16RetHelper>>>,
```
- EN: Defines TableGen record `CC_Mips_FixedArg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_Mips_FixedArg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 308-308
```tablegen
  CCIfByVal<CCDelegateTo<CC_Mips_ByVal>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 310-323
```tablegen
  // f128 needs to be handled similarly to f32 and f64 on hard-float. However,
  // f128 is not legal and is lowered to i128 which is further lowered to a pair
  // of i64's.
  // This presents us with a problem for the calling convention since hard-float
  // still needs to pass them in FPU registers. We therefore resort to a
  // pre-analyze (see PreAnalyzeFormalArgsForF128()) step to pass information on
  // whether the argument was originally an f128 into the tablegen-erated code.
  //
  // f128 should only occur for the N64 ABI where long double is 128-bit. On
  // N32, long double is equivalent to double.
  CCIfType<[i64], 
      CCIfSubtargetNot<"isSingleFloat()",
          CCIfSubtargetNot<"useSoftFloat()", 
              CCIfOrigArgWasF128<CCBitConvertToType<f64>>>>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 325-325
```tablegen
  CCIfCC<"CallingConv::Fast", CCDelegateTo<CC_Mips_FastCC>>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 327-329
```tablegen
  CCIfSubtarget<"isABI_O32()", CCIfSubtargetNot<"isGP64bit()", CCDelegateTo<CC_MipsO32_FP>>>,
  CCDelegateTo<CC_MipsN>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 331-332
```tablegen
def CC_Mips_VarArg : CallingConv<[
  CCIfByVal<CCDelegateTo<CC_Mips_ByVal>>,
```
- EN: Defines TableGen record `CC_Mips_VarArg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_Mips_VarArg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 334-336
```tablegen
  CCIfSubtarget<"isABI_O32()", CCDelegateTo<CC_MipsO32_FP>>,
  CCDelegateTo<CC_MipsN_VarArg>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 338-341
```tablegen
def CC_Mips : CallingConv<[
  CCIfVarArg<CCIfArgVarArg<CCDelegateTo<CC_Mips_VarArg>>>,
  CCDelegateTo<CC_Mips_FixedArg>
]>;
```
- EN: Defines TableGen record `CC_Mips` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CC_Mips`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 343-345
```tablegen
//===----------------------------------------------------------------------===//
// Callee-saved register lists.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 347-348
```tablegen
def CSR_O32_SingleFloat : CalleeSavedRegs<(add(sequence "F%u", 31, 20), RA, FP,
                              (sequence "S%u", 7, 0))>;
```
- EN: Defines TableGen record `CSR_O32_SingleFloat` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_O32_SingleFloat`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 350-353
```tablegen
def CSR_O32_FPXX : CalleeSavedRegs<(add (sequence "D%u", 15, 10), RA, FP,
                                        (sequence "S%u", 7, 0))> {
  let OtherPreserved = (add (decimate (sequence "F%u", 30, 20), 2));
}
```
- EN: Defines TableGen record `CSR_O32_FPXX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_O32_FPXX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 355-356
```tablegen
def CSR_O32 : CalleeSavedRegs<(add (sequence "D%u", 15, 10), RA, FP,
                                   (sequence "S%u", 7, 0))>;
```
- EN: Defines TableGen record `CSR_O32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_O32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 358-360
```tablegen
def CSR_O32_FP64 :
  CalleeSavedRegs<(add (decimate (sequence "D%u_64", 30, 20), 2), RA, FP,
                       (sequence "S%u", 7, 0))>;
```
- EN: Defines TableGen record `CSR_O32_FP64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_O32_FP64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 362-363
```tablegen
def CSR_N32 : CalleeSavedRegs<(add(decimate(sequence "D%u_64", 30, 20), 2),
                  RA_64, FP_64, GP_64, (sequence "S%u_64", 7, 0))>;
```
- EN: Defines TableGen record `CSR_N32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_N32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 365-367
```tablegen
def CSR_N32_SingleFloat
    : CalleeSavedRegs<(add(decimate(sequence "F%u", 30, 20), 2), RA_64, FP_64,
          GP_64, (sequence "S%u_64", 7, 0))>;
```
- EN: Defines TableGen record `CSR_N32_SingleFloat` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_N32_SingleFloat`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 369-370
```tablegen
def CSR_N64 : CalleeSavedRegs<(add (sequence "D%u_64", 31, 24), RA_64, FP_64,
                                   GP_64, (sequence "S%u_64", 7, 0))>;
```
- EN: Defines TableGen record `CSR_N64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_N64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 372-373
```tablegen
def CSR_N64_SingleFloat : CalleeSavedRegs<(add(sequence "F%u", 31, 24), RA_64,
                              FP_64, GP_64, (sequence "S%u_64", 7, 0))>;
```
- EN: Defines TableGen record `CSR_N64_SingleFloat` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_N64_SingleFloat`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 375-378
```tablegen
def CSR_Mips16RetHelper :
  CalleeSavedRegs<(add V0, V1, FP,
                   (sequence "A%u", 3, 0), (sequence "S%u", 7, 0),
                   (sequence "D%u", 15, 10))>;
```
- EN: Defines TableGen record `CSR_Mips16RetHelper` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_Mips16RetHelper`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 380-384
```tablegen
def CSR_Interrupt_32R6 : CalleeSavedRegs<(add (sequence "A%u", 3, 0),
                                              (sequence "S%u", 7, 0),
                                              (sequence "V%u", 1, 0),
                                              (sequence "T%u", 9, 0),
                                              RA, FP, GP, AT)>;
```
- EN: Defines TableGen record `CSR_Interrupt_32R6` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_Interrupt_32R6`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 386-390
```tablegen
def CSR_Interrupt_32 : CalleeSavedRegs<(add (sequence "A%u", 3, 0),
                                            (sequence "S%u", 7, 0),
                                            (sequence "V%u", 1, 0),
                                            (sequence "T%u", 9, 0),
                                            RA, FP, GP, AT, LO0, HI0)>;
```
- EN: Defines TableGen record `CSR_Interrupt_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_Interrupt_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 392-396
```tablegen
def CSR_Interrupt_64R6 : CalleeSavedRegs<(add (sequence "A%u_64", 3, 0),
                                              (sequence "V%u_64", 1, 0),
                                              (sequence "S%u_64", 7, 0),
                                              (sequence "T%u_64", 9, 0),
                                              RA_64, FP_64, GP_64, AT_64)>;
```
- EN: Defines TableGen record `CSR_Interrupt_64R6` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_Interrupt_64R6`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 398-403
```tablegen
def CSR_Interrupt_64 : CalleeSavedRegs<(add (sequence "A%u_64", 3, 0),
                                            (sequence "S%u_64", 7, 0),
                                            (sequence "T%u_64", 9, 0),
                                            (sequence "V%u_64", 1, 0),
                                            RA_64, FP_64, GP_64, AT_64,
                                            LO0_64, HI0_64)>;
```
- EN: Defines TableGen record `CSR_Interrupt_64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CSR_Interrupt_64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

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
