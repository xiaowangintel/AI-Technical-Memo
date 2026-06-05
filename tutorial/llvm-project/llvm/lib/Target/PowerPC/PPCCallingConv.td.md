# PPCCallingConv.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCCallingConv.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCCallingConv.td - Calling Conventions for PowerPC. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCCallingConv.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===- PPCCallingConv.td - Calling Conventions for PowerPC -*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. ABI and calling-convention details are important in this part of the code.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 7-11

```tablegen
//===----------------------------------------------------------------------===//
//
// This describes the calling conventions for the PowerPC 32- and 64-bit
// architectures.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This describes the calling conventions for the PowerPC 32- and 64-bit".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This describes the calling conventions for the PowerPC 32- and 64-bit”。

### Lines 12-18

```tablegen
//===----------------------------------------------------------------------===//

/// CCIfSubtarget - Match if the current subtarget has a feature F.
class CCIfSubtarget<string F, CCAction A>
    : CCIf<!strconcat("State.getMachineFunction().getSubtarget<PPCSubtarget>().",
                     F),
          A>;
```
- **EN**: Declares a backend-facing type `CCIfSubtarget` and outlines the API or state that nearby code will rely on. Subtarget feature gating influences the behavior here.
- **CN**: 这里声明面向后端的类型 `CCIfSubtarget`，并勾勒出周边代码会依赖的接口或状态。 子目标特性裁剪会影响这里的行为。

### Lines 19-27

```tablegen
class CCIfNotSubtarget<string F, CCAction A>
    : CCIf<!strconcat("!State.getMachineFunction().getSubtarget<PPCSubtarget>().",
                     F),
          A>;
class CCIfOrigArgWasNotPPCF128<CCAction A>
    : CCIf<"!OrigTy->isPPC_FP128Ty()", A>;
class CCIfOrigArgWasPPCF128<CCAction A>
    : CCIf<"OrigTy->isPPC_FP128Ty()", A>;
```
- **EN**: Declares a backend-facing type `CCIfNotSubtarget`, `CCIfOrigArgWasNotPPCF128`, `CCIfOrigArgWasPPCF128` and outlines the API or state that nearby code will rely on. Subtarget feature gating influences the behavior here.
- **CN**: 这里声明面向后端的类型 `CCIfNotSubtarget`, `CCIfOrigArgWasNotPPCF128`, `CCIfOrigArgWasPPCF128`，并勾勒出周边代码会依赖的接口或状态。 子目标特性裁剪会影响这里的行为。

### Lines 28-29

```tablegen
//===----------------------------------------------------------------------===//
// Return Value Calling Convention
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Return Value Calling Convention".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Return Value Calling Convention”。

### Lines 30-38

```tablegen
//===----------------------------------------------------------------------===//

// PPC64 AnyReg return-value convention. No explicit register is specified for
// the return-value. The register allocator is allowed and expected to choose
// any free register.
//
// This calling convention is currently only supported by the stackmap and
// patchpoint intrinsics. All other uses will result in an assert on Debug
// builds. On Release builds we fallback to the PPC C calling convention.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PPC64 AnyReg return-value convention. No explicit register is specified for".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PPC64 AnyReg return-value convention. No explicit register is specified for”。

### Lines 39-65

```tablegen
def RetCC_PPC64_AnyReg : CallingConv<[
  CCCustom<"CC_PPC_AnyReg_Error">
]>;

// Return-value convention for PowerPC coldcc.
let Entry = 1 in
def RetCC_PPC_Cold : CallingConv<[
  // Use the same return registers as RetCC_PPC, but limited to only
  // one return value. The remaining return values will be saved to
  // the stack.
  CCIfType<[i32, i1], CCIfSubtarget<"isPPC64()", CCPromoteToType<i64>>>,
  CCIfType<[i1], CCIfNotSubtarget<"isPPC64()", CCPromoteToType<i32>>>,

  CCIfType<[i32], CCAssignToReg<[R3]>>,
  CCIfType<[i64], CCAssignToReg<[X3]>>,
  CCIfType<[i128], CCAssignToReg<[X3]>>,

  CCIfType<[f32], CCAssignToReg<[F1]>>,
  CCIfType<[f64], CCAssignToReg<[F1]>>,
  CCIfType<[f128], CCIfSubtarget<"hasAltivec()", CCAssignToReg<[V2]>>>,

  CCIfType<[v16i8, v8i16, v4i32, v2i64, v1i128, v4f32, v2f64],
           CCIfSubtarget<"hasAltivec()",
           CCAssignToReg<[V2]>>>
]>;

// Return-value convention for PowerPC
```
- **EN**: Adds declarative TableGen records such as `RetCC_PPC64_AnyReg`, `RetCC_PPC_Cold` that LLVM later expands into generated tables or helper code. ABI and calling-convention details are important in this part of the code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RetCC_PPC64_AnyReg`, `RetCC_PPC_Cold`，LLVM 随后会把它们展开成生成表或辅助代码。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 66-103

```tablegen
let Entry = 1 in
def RetCC_PPC : CallingConv<[
  CCIfCC<"CallingConv::AnyReg", CCDelegateTo<RetCC_PPC64_AnyReg>>,

  // On PPC64, integer return values are always promoted to i64
  CCIfType<[i32, i1], CCIfSubtarget<"isPPC64()", CCPromoteToType<i64>>>,
  CCIfType<[i1], CCIfNotSubtarget<"isPPC64()", CCPromoteToType<i32>>>,

  CCIfType<[i32], CCAssignToReg<[R3, R4, R5, R6, R7, R8, R9, R10]>>,
  CCIfType<[i64], CCAssignToReg<[X3, X4, X5, X6]>>,
  CCIfType<[i128], CCAssignToReg<[X3, X4, X5, X6]>>,

  // Floating point types returned as "direct" go into F1 .. F8; note that
  // only the ELFv2 ABI fully utilizes all these registers.
  CCIfNotSubtarget<"hasSPE()",
       CCIfType<[f32], CCAssignToReg<[F1, F2, F3, F4, F5, F6, F7, F8]>>>,
  CCIfNotSubtarget<"hasSPE()",
       CCIfType<[f64], CCAssignToReg<[F1, F2, F3, F4, F5, F6, F7, F8]>>>,
  CCIfSubtarget<"hasSPE()",
       CCIfType<[f32], CCAssignToReg<[R3, R4, R5, R6, R7, R8, R9, R10]>>>,
  CCIfSubtarget<"hasSPE()",
       CCIfType<[f64], CCCustom<"CC_PPC32_SPE_RetF64">>>,

  // For P9, f128 are passed in vector registers.
  CCIfType<[f128],
           CCIfSubtarget<"hasAltivec()",
           CCAssignToReg<[V2, V3, V4, V5, V6, V7, V8, V9]>>>,

  // Vector types returned as "direct" go into V2 .. V9; note that only the
  // ELFv2 ABI fully utilizes all these registers.
  CCIfType<[v16i8, v8i16, v4i32, v2i64, v1i128, v4f32, v2f64],
           CCIfSubtarget<"hasAltivec()",
           CCAssignToReg<[V2, V3, V4, V5, V6, V7, V8, V9]>>>
]>;

// No explicit register is specified for the AnyReg calling convention. The
// register allocator may assign the arguments to any free register.
//
```
- **EN**: Adds declarative TableGen records such as `RetCC_PPC` that LLVM later expands into generated tables or helper code. ABI and calling-convention details are important in this part of the code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RetCC_PPC`，LLVM 随后会把它们展开成生成表或辅助代码。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 104-116

```tablegen
// This calling convention is currently only supported by the stackmap and
// patchpoint intrinsics. All other uses will result in an assert on Debug
// builds. On Release builds we fallback to the PPC C calling convention.
def CC_PPC64_AnyReg : CallingConv<[
  CCCustom<"CC_PPC_AnyReg_Error">
]>;

// Calling Convention corresponding to the 64-bit PowerPC ELFv2 ABI.
// This calling convention currently only handles integers, floats and
// vectors within registers, as well as it handles the shadowing of GPRs
// when floating point and vector arguments are used.
// FIXME: This calling convention needs to be extended to handle all types and
// complexities of the ABI.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This calling convention is currently only supported by the stackmap and". ABI and calling-convention details are important in this part of the code.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This calling convention is currently only supported by the stackmap and”。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 117-154

```tablegen
let Entry = 1 in
def CC_PPC64_ELF : CallingConv<[
  CCIfCC<"CallingConv::AnyReg", CCDelegateTo<CC_PPC64_AnyReg>>,

  CCIfType<[i1],  CCPromoteToType<i64>>,
  CCIfType<[i8],  CCPromoteToType<i64>>,
  CCIfType<[i16], CCPromoteToType<i64>>,
  CCIfType<[i32], CCPromoteToType<i64>>,
  CCIfType<[i64], CCAssignToReg<[X3, X4, X5, X6, X7, X8, X9, X10]>>,

  // Handle fp types and shadow the corresponding registers as necessary.
  CCIfType<[f32, f64], CCIfNotVarArg<CCCustom<"CC_PPC64_ELF_Shadow_GPR_Regs">>>,
  CCIfType<[f32, f64],
           CCIfNotVarArg<CCAssignToReg<[F1, F2, F3, F4, F5, F6, F7, F8, F9, F10,
                                        F11, F12, F13]>>>,

  // f128 is handled through vector registers instead of fp registers.
  CCIfType<[f128],
           CCIfSubtarget<"hasAltivec()",
           CCIfNotVarArg<CCCustom<"CC_PPC64_ELF_Shadow_GPR_Regs">>>>,
  CCIfType<[f128],
           CCIfSubtarget<"hasAltivec()",
           CCIfNotVarArg<CCAssignToReg<[V2, V3, V4, V5, V6, V7, V8, V9, V10,
                                        V11, V12, V13]>>>>,

  // Handle support for vector types, and shadow GPRs as necessary.
  CCIfType<[v16i8, v8i16, v4i32, v2i64, v4f32, v2f64, v1i128],
           CCIfSubtarget<"hasAltivec()",
           CCIfNotVarArg<CCCustom<"CC_PPC64_ELF_Shadow_GPR_Regs">>>>,
  CCIfType<[v16i8, v8i16, v4i32, v2i64, v4f32, v2f64, v1i128],
           CCIfSubtarget<"hasAltivec()",
           CCIfNotVarArg<CCAssignToReg<[V2, V3, V4, V5, V6, V7, V8, V9, V10,
                                        V11, V12, V13]>>>>,
]>;

// Simple calling convention for 64-bit ELF PowerPC fast isel.
// Only handle ints and floats.  All ints are promoted to i64.
// Vector types and quadword ints are not handled.
```
- **EN**: Adds declarative TableGen records such as `CC_PPC64_ELF` that LLVM later expands into generated tables or helper code. ABI and calling-convention details are important in this part of the code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CC_PPC64_ELF`，LLVM 随后会把它们展开成生成表或辅助代码。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 155-170

```tablegen
let Entry = 1 in
def CC_PPC64_ELF_FIS : CallingConv<[
  CCIfCC<"CallingConv::AnyReg", CCDelegateTo<CC_PPC64_AnyReg>>,

  CCIfType<[i1],  CCPromoteToType<i64>>,
  CCIfType<[i8],  CCPromoteToType<i64>>,
  CCIfType<[i16], CCPromoteToType<i64>>,
  CCIfType<[i32], CCPromoteToType<i64>>,
  CCIfType<[i64], CCAssignToReg<[X3, X4, X5, X6, X7, X8, X9, X10]>>,
  CCIfType<[f32, f64], CCAssignToReg<[F1, F2, F3, F4, F5, F6, F7, F8]>>
]>;

// Simple return-value convention for 64-bit ELF PowerPC fast isel.
// All small ints are promoted to i64.  Vector types, quadword ints,
// and multiple register returns are "supported" to avoid compile
// errors, but none are handled by the fast selector.
```
- **EN**: Adds declarative TableGen records such as `CC_PPC64_ELF_FIS` that LLVM later expands into generated tables or helper code. ABI and calling-convention details are important in this part of the code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CC_PPC64_ELF_FIS`，LLVM 随后会把它们展开成生成表或辅助代码。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 171-190

```tablegen
let Entry = 1 in
def RetCC_PPC64_ELF_FIS : CallingConv<[
  CCIfCC<"CallingConv::AnyReg", CCDelegateTo<RetCC_PPC64_AnyReg>>,

  CCIfType<[i1],   CCPromoteToType<i64>>,
  CCIfType<[i8],   CCPromoteToType<i64>>,
  CCIfType<[i16],  CCPromoteToType<i64>>,
  CCIfType<[i32],  CCPromoteToType<i64>>,
  CCIfType<[i64],  CCAssignToReg<[X3, X4, X5, X6]>>,
  CCIfType<[i128], CCAssignToReg<[X3, X4, X5, X6]>>,
  CCIfType<[f32],  CCAssignToReg<[F1, F2, F3, F4, F5, F6, F7, F8]>>,
  CCIfType<[f64],  CCAssignToReg<[F1, F2, F3, F4, F5, F6, F7, F8]>>,
  CCIfType<[f128],
           CCIfSubtarget<"hasAltivec()",
           CCAssignToReg<[V2, V3, V4, V5, V6, V7, V8, V9]>>>,
  CCIfType<[v16i8, v8i16, v4i32, v2i64, v1i128, v4f32, v2f64],
           CCIfSubtarget<"hasAltivec()",
           CCAssignToReg<[V2, V3, V4, V5, V6, V7, V8, V9]>>>
]>;
```
- **EN**: Adds declarative TableGen records such as `RetCC_PPC64_ELF_FIS` that LLVM later expands into generated tables or helper code. ABI and calling-convention details are important in this part of the code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RetCC_PPC64_ELF_FIS`，LLVM 随后会把它们展开成生成表或辅助代码。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 191-192

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC System V Release 4 32-bit ABI
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PowerPC System V Release 4 32-bit ABI".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PowerPC System V Release 4 32-bit ABI”。

### Lines 193-230

```tablegen
//===----------------------------------------------------------------------===//

def CC_PPC32_SVR4_Common : CallingConv<[
  CCIfType<[i1], CCPromoteToType<i32>>,

  // The ABI requires i64 to be passed in two adjacent registers with the first
  // register having an odd register number.
  CCIfType<[i32],
  CCIfSplit<CCIfSubtarget<"useSoftFloat()", 
            CCIfOrigArgWasNotPPCF128<
            CCCustom<"CC_PPC32_SVR4_Custom_AlignArgRegs">>>>>,
  
  CCIfType<[i32],
  CCIfSplit<CCIfNotSubtarget<"useSoftFloat()", 
                            CCCustom<"CC_PPC32_SVR4_Custom_AlignArgRegs">>>>,
  CCIfType<[f64],
  CCIfSubtarget<"hasSPE()",
                CCCustom<"CC_PPC32_SVR4_Custom_AlignArgRegs">>>,
  CCIfSplit<CCIfSubtarget<"useSoftFloat()",
                          CCIfOrigArgWasPPCF128<CCCustom<
                          "CC_PPC32_SVR4_Custom_SkipLastArgRegsPPCF128">>>>,

  // The 'nest' parameter, if any, is passed in R11.
  CCIfNest<CCAssignToReg<[R11]>>,

  // The first 8 integer arguments are passed in integer registers.
  CCIfType<[i32], CCAssignToReg<[R3, R4, R5, R6, R7, R8, R9, R10]>>,

  // Make sure the i64 words from a long double are either both passed in
  // registers or both passed on the stack.
  CCIfType<[f64], CCIfSplit<CCCustom<"CC_PPC32_SVR4_Custom_AlignFPArgRegs">>>,
  
  // FP values are passed in F1 - F8.
  CCIfType<[f32, f64],
           CCIfNotSubtarget<"hasSPE()",
                            CCAssignToReg<[F1, F2, F3, F4, F5, F6, F7, F8]>>>,
  CCIfType<[f64],
           CCIfSubtarget<"hasSPE()",
```
- **EN**: Adds declarative TableGen records such as `CC_PPC32_SVR4_Common` that LLVM later expands into generated tables or helper code. ABI and calling-convention details are important in this part of the code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CC_PPC32_SVR4_Common`，LLVM 随后会把它们展开成生成表或辅助代码。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 231-251

```tablegen
                         CCCustom<"CC_PPC32_SPE_CustomSplitFP64">>>,
  CCIfType<[f32],
           CCIfSubtarget<"hasSPE()",
                         CCAssignToReg<[R3, R4, R5, R6, R7, R8, R9, R10]>>>,

  // Split arguments have an alignment of 8 bytes on the stack.
  CCIfType<[i32], CCIfSplit<CCAssignToStack<4, 8>>>,
  
  CCIfType<[i32], CCAssignToStack<4, 4>>,
  
  CCIfType<[f32], CCAssignToStack<4, 4>>,
  CCIfType<[f64], CCAssignToStack<8, 8>>,

  // Vectors and float128 get 16-byte stack slots that are 16-byte aligned.
  CCIfType<[v16i8, v8i16, v4i32, v4f32, v2f64, v2i64], CCAssignToStack<16, 16>>,
  CCIfType<[f128], CCIfSubtarget<"hasAltivec()", CCAssignToStack<16, 16>>>
]>;

// This calling convention puts vector arguments always on the stack. It is used
// to assign vector arguments which belong to the variable portion of the
// parameter list of a variable argument function.
```
- **EN**: Declares function entry points that other backend components call later. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 该区间与栈帧布局或栈访问相关。

### Lines 252-258

```tablegen
let Entry = 1 in
def CC_PPC32_SVR4_VarArg : CallingConv<[
  CCDelegateTo<CC_PPC32_SVR4_Common>
]>;

// In contrast to CC_PPC32_SVR4_VarArg, this calling convention first tries to
// put vector arguments in vector registers before putting them on the stack.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "In contrast to CC_PPC32_SVR4_VarArg, this calling convention first tries to". ABI and calling-convention details are important in this part of the code.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“In contrast to CC_PPC32_SVR4_VarArg, this calling convention first tries to”。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 259-286

```tablegen
let Entry = 1 in
def CC_PPC32_SVR4 : CallingConv<[
  // The first 12 Vector arguments are passed in AltiVec registers.
  CCIfType<[v16i8, v8i16, v4i32, v2i64, v1i128, v4f32, v2f64],
           CCIfSubtarget<"hasAltivec()", CCAssignToReg<[V2, V3, V4, V5, V6, V7,
                          V8, V9, V10, V11, V12, V13]>>>,

  // Float128 types treated as vector arguments.
  CCIfType<[f128],
           CCIfSubtarget<"hasAltivec()", CCAssignToReg<[V2, V3, V4, V5, V6, V7,
                          V8, V9, V10, V11, V12, V13]>>>,
           
  CCDelegateTo<CC_PPC32_SVR4_Common>
]>;  

// Helper "calling convention" to handle aggregate by value arguments.
// Aggregate by value arguments are always placed in the local variable space
// of the caller. This calling convention is only used to assign those stack
// offsets in the callers stack frame.
//
// Still, the address of the aggregate copy in the callers stack frame is passed
// in a GPR (or in the parameter list area if all GPRs are allocated) from the
// caller to the callee. The location for the address argument is assigned by
// the CC_PPC32_SVR4 calling convention.
//
// The only purpose of CC_PPC32_SVR4_Custom_Dummy is to skip arguments which are
// not passed by value.
 
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The first 12 Vector arguments are passed in AltiVec registers.". ABI and calling-convention details are important in this part of the code.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The first 12 Vector arguments are passed in AltiVec registers.”。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 287-293

```tablegen
let Entry = 1 in
def CC_PPC32_SVR4_ByVal : CallingConv<[
  CCIfByVal<CCPassByVal<4, 4>>,
  
  CCCustom<"CC_PPC32_SVR4_Custom_Dummy">
]>;
```
- **EN**: Adds declarative TableGen records such as `CC_PPC32_SVR4_ByVal` that LLVM later expands into generated tables or helper code. ABI and calling-convention details are important in this part of the code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CC_PPC32_SVR4_ByVal`，LLVM 随后会把它们展开成生成表或辅助代码。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 294-301

```tablegen
def CSR_Altivec : CalleeSavedRegs<(add V20, V21, V22, V23, V24, V25, V26, V27,
                                       V28, V29, V30, V31)>;

// SPE does not use FPRs, so break out the common register set as base.
def CSR_SVR432_COMM : CalleeSavedRegs<(add R14, R15, R16, R17, R18, R19, R20,
                                          R21, R22, R23, R24, R25, R26, R27,
                                          R28, R29, R30, R31, CR2, CR3, CR4
                                      )>;
```
- **EN**: Adds declarative TableGen records such as `CSR_Altivec`, `CSR_SVR432_COMM` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_Altivec`, `CSR_SVR432_COMM`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 302-309

```tablegen
def CSR_SVR432 :  CalleeSavedRegs<(add CSR_SVR432_COMM, F14, F15, F16, F17, F18,
                                        F19, F20, F21, F22, F23, F24, F25, F26,
                                        F27, F28, F29, F30, F31
                                   )>;
def CSR_SPE : CalleeSavedRegs<(add S14, S15, S16, S17, S18, S19, S20, S21, S22,
                                   S23, S24, S25, S26, S27, S28, S29, S30 
                              )>;
```
- **EN**: Adds declarative TableGen records such as `CSR_SVR432`, `CSR_SPE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_SVR432`, `CSR_SPE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 310-317

```tablegen
def CSR_SPE_NO_S30_31 : CalleeSavedRegs<(add S14, S15, S16, S17, S18, S19, S20, S21,
				          S22, S23, S24, S25, S26, S27, S28, S29
			      )>;

def CSR_SVR432_Altivec : CalleeSavedRegs<(add CSR_SVR432, CSR_Altivec)>;

def CSR_SVR432_SPE : CalleeSavedRegs<(add CSR_SVR432_COMM, CSR_SPE)>;
```
- **EN**: Adds declarative TableGen records such as `CSR_SPE_NO_S30_31`, `CSR_SVR432_Altivec`, `CSR_SVR432_SPE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_SPE_NO_S30_31`, `CSR_SVR432_Altivec`, `CSR_SVR432_SPE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 318-326

```tablegen
def CSR_SVR432_SPE_NO_S30_31 : CalleeSavedRegs<(add CSR_SVR432_COMM, CSR_SPE_NO_S30_31)>;

def CSR_AIX32 : CalleeSavedRegs<(add R13, R14, R15, R16, R17, R18, R19, R20,
                                     R21, R22, R23, R24, R25, R26, R27, R28,
                                     R29, R30, R31, F14, F15, F16, F17, F18,
                                     F19, F20, F21, F22, F23, F24, F25, F26,
                                     F27, F28, F29, F30, F31, CR2, CR3, CR4
                                )>;
```
- **EN**: Adds declarative TableGen records such as `CSR_SVR432_SPE_NO_S30_31`, `CSR_AIX32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_SVR432_SPE_NO_S30_31`, `CSR_AIX32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 327-337

```tablegen
def CSR_AIX32_Altivec : CalleeSavedRegs<(add CSR_AIX32, CSR_Altivec)>;

// Common CalleeSavedRegs for SVR4 and AIX.
def CSR_PPC64   : CalleeSavedRegs<(add X14, X15, X16, X17, X18, X19, X20,
                                        X21, X22, X23, X24, X25, X26, X27, X28,
                                        X29, X30, X31, F14, F15, F16, F17, F18,
                                        F19, F20, F21, F22, F23, F24, F25, F26,
                                        F27, F28, F29, F30, F31, CR2, CR3, CR4
                                   )>;
```
- **EN**: Adds declarative TableGen records such as `CSR_AIX32_Altivec`, `CSR_PPC64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_AIX32_Altivec`, `CSR_PPC64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 338-353

```tablegen
def CSR_PPC64_Altivec : CalleeSavedRegs<(add CSR_PPC64, CSR_Altivec)>;

def CSR_PPC64_R2 : CalleeSavedRegs<(add CSR_PPC64, X2)>;

def CSR_PPC64_R2_Altivec : CalleeSavedRegs<(add CSR_PPC64_Altivec, X2)>;

def CSR_NoRegs : CalleeSavedRegs<(add)>;

// coldcc calling convection marks most registers as non-volatile.
// Do not include r1 since the stack pointer is never considered a CSR.
// Do not include r2, since it is the TOC register and is added depending
// on whether or not the function uses the TOC and is a non-leaf.
// Do not include r0,r11,r13 as they are optional in functional linkage
// and value may be altered by inter-library calls.
// Do not include r12 as it is used as a scratch register.
// Do not include return registers r3, f1, v2.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "coldcc calling convection marks most registers as non-volatile.". Notable symbols in this range include `CSR_PPC64_Altivec`, `CSR_PPC64_R2`, `CSR_PPC64_R2_Altivec`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“coldcc calling convection marks most registers as non-volatile.”。 该区间中较显眼的符号包括 `CSR_PPC64_Altivec`, `CSR_PPC64_R2`, `CSR_PPC64_R2_Altivec`。

### Lines 354-361

```tablegen
def CSR_SVR32_ColdCC_Common : CalleeSavedRegs<(add (sequence "R%u", 4, 10),
                                                (sequence "R%u", 14, 31),
                                                (sequence "CR%u", 0, 7))>;

def CSR_SVR32_ColdCC : CalleeSavedRegs<(add CSR_SVR32_ColdCC_Common,
                                          F0, (sequence "F%u", 2, 31))>;
```
- **EN**: Adds declarative TableGen records such as `CSR_SVR32_ColdCC_Common`, `CSR_SVR32_ColdCC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_SVR32_ColdCC_Common`, `CSR_SVR32_ColdCC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 362-369

```tablegen
def CSR_SVR32_ColdCC_Altivec : CalleeSavedRegs<(add CSR_SVR32_ColdCC,
                                            (sequence "V%u", 0, 1),
                                            (sequence "V%u", 3, 31))>;

def CSR_SVR32_ColdCC_SPE : CalleeSavedRegs<(add CSR_SVR32_ColdCC_Common,
                                            (sequence "S%u", 4, 10),
                                            (sequence "S%u", 14, 31))>;
```
- **EN**: Adds declarative TableGen records such as `CSR_SVR32_ColdCC_Altivec`, `CSR_SVR32_ColdCC_SPE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_SVR32_ColdCC_Altivec`, `CSR_SVR32_ColdCC_SPE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 370-376

```tablegen
def CSR_SVR64_ColdCC : CalleeSavedRegs<(add  (sequence "X%u", 4, 10),
                                             (sequence "X%u", 14, 31),
                                             F0, (sequence "F%u", 2, 31),
                                             (sequence "CR%u", 0, 7))>;

def CSR_SVR64_ColdCC_R2: CalleeSavedRegs<(add CSR_SVR64_ColdCC, X2)>;
```
- **EN**: Adds declarative TableGen records such as `CSR_SVR64_ColdCC`, `CSR_SVR64_ColdCC_R2` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_SVR64_ColdCC`, `CSR_SVR64_ColdCC_R2`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 377-387

```tablegen
def CSR_SVR64_ColdCC_Altivec : CalleeSavedRegs<(add CSR_SVR64_ColdCC,
                                             (sequence "V%u", 0, 1),
                                             (sequence "V%u", 3, 31))>;

def CSR_SVR64_ColdCC_R2_Altivec : CalleeSavedRegs<(add CSR_SVR64_ColdCC_Altivec, X2)>;

def CSR_64_AllRegs: CalleeSavedRegs<(add X0, (sequence "X%u", 3, 10),
                                             (sequence "X%u", 14, 31),
                                             (sequence "F%u", 0, 31),
                                             (sequence "CR%u", 0, 7))>;
```
- **EN**: Adds declarative TableGen records such as `CSR_SVR64_ColdCC_Altivec`, `CSR_SVR64_ColdCC_R2_Altivec`, `CSR_64_AllRegs` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_SVR64_ColdCC_Altivec`, `CSR_SVR64_ColdCC_R2_Altivec`, `CSR_64_AllRegs`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 388-396

```tablegen
def CSR_64_AllRegs_Altivec : CalleeSavedRegs<(add CSR_64_AllRegs,
                                             (sequence "V%u", 0, 31))>;

def CSR_64_AllRegs_AIX_Dflt_Altivec : CalleeSavedRegs<(add CSR_64_AllRegs,
                                             (sequence "V%u", 0, 19))>;

def CSR_64_AllRegs_VSX : CalleeSavedRegs<(add CSR_64_AllRegs_Altivec,
                                         (sequence "VSL%u", 0, 31))>;
```
- **EN**: Adds declarative TableGen records such as `CSR_64_AllRegs_Altivec`, `CSR_64_AllRegs_AIX_Dflt_Altivec`, `CSR_64_AllRegs_VSX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_64_AllRegs_Altivec`, `CSR_64_AllRegs_AIX_Dflt_Altivec`, `CSR_64_AllRegs_VSX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 397-404

```tablegen
def CSR_64_AllRegs_AIX_Dflt_VSX : CalleeSavedRegs<(add CSR_64_AllRegs_Altivec,
                                         (sequence "VSL%u", 0, 19))>;

def CSR_ALL_VSRP : CalleeSavedRegs<(sequence "VSRp%u", 0, 31)>;

def CSR_VSRP :
  CalleeSavedRegs<(add VSRp26, VSRp27, VSRp28, VSRp29, VSRp30, VSRp31)>;
```
- **EN**: Adds declarative TableGen records such as `CSR_64_AllRegs_AIX_Dflt_VSX`, `CSR_ALL_VSRP`, `CSR_VSRP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_64_AllRegs_AIX_Dflt_VSX`, `CSR_ALL_VSRP`, `CSR_VSRP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 405-413

```tablegen
def CSR_SVR432_VSRP : CalleeSavedRegs<(add CSR_SVR432_Altivec, CSR_VSRP)>;

def CSR_SVR464_VSRP : CalleeSavedRegs<(add CSR_PPC64_Altivec, CSR_VSRP)>;

def CSR_SVR464_R2_VSRP : CalleeSavedRegs<(add CSR_SVR464_VSRP, X2)>;

def CSR_SVR32_ColdCC_VSRP : CalleeSavedRegs<(add CSR_SVR32_ColdCC_Altivec,
                                            (sub CSR_ALL_VSRP, VSRp17))>;
```
- **EN**: Adds declarative TableGen records such as `CSR_SVR432_VSRP`, `CSR_SVR464_VSRP`, `CSR_SVR464_R2_VSRP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_SVR432_VSRP`, `CSR_SVR464_VSRP`, `CSR_SVR464_R2_VSRP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 414-421

```tablegen
def CSR_SVR64_ColdCC_VSRP : CalleeSavedRegs<(add CSR_SVR64_ColdCC,
                                            (sub CSR_ALL_VSRP, VSRp17))>;

def CSR_SVR64_ColdCC_R2_VSRP : CalleeSavedRegs<(add CSR_SVR64_ColdCC_VSRP, X2)>;

def CSR_64_AllRegs_VSRP :
  CalleeSavedRegs<(add CSR_64_AllRegs_VSX, CSR_ALL_VSRP)>;
```
- **EN**: Adds declarative TableGen records such as `CSR_SVR64_ColdCC_VSRP`, `CSR_SVR64_ColdCC_R2_VSRP`, `CSR_64_AllRegs_VSRP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_SVR64_ColdCC_VSRP`, `CSR_SVR64_ColdCC_R2_VSRP`, `CSR_64_AllRegs_VSRP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 422-426

```tablegen
def CSR_AIX64_VSRP : CalleeSavedRegs<(add CSR_PPC64_Altivec, CSR_VSRP)>;

def CSR_AIX64_R2_VSRP : CalleeSavedRegs<(add CSR_AIX64_VSRP, X2)>;

def CSR_AIX32_VSRP : CalleeSavedRegs<(add CSR_AIX32_Altivec, CSR_VSRP)>;
```
- **EN**: Adds declarative TableGen records such as `CSR_AIX64_VSRP`, `CSR_AIX64_R2_VSRP`, `CSR_AIX32_VSRP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CSR_AIX64_VSRP`, `CSR_AIX64_R2_VSRP`, `CSR_AIX32_VSRP`，LLVM 随后会把它们展开成生成表或辅助代码。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Calling convention handling / 调用约定处理
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Object format integration / 目标文件格式集成
- Declarative TableGen records / 声明式 TableGen 记录

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
