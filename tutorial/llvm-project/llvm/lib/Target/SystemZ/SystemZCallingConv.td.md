# SystemZCallingConv.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZCallingConv.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines SystemZ backend metadata using LLVM TableGen DSL; specifically it defines calling convention rules and register/stack assignments.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义 SystemZ 后端元数据；具体而言，它定义调用约定规则以及寄存器/栈分配。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: //=- SystemZCallingConv.td - Calling conventions for SystemZ -*- tablegen -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: // This describes the calling conventions for the SystemZ ABI.
   9: //===----------------------------------------------------------------------===//
  10: 
  11: class CCIfExtend<CCAction A>
  12:   : CCIf<"ArgFlags.isSExt() || ArgFlags.isZExt()", A>;
  13: 
  14: class CCIfSubtarget<string F, CCAction A>
  15:   : CCIf<!strconcat("State.getMachineFunction()."
  16:                     "getSubtarget<SystemZSubtarget>().", F),
  17:          A>;
  18: 
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `CCIfExtend`, `CCIfSubtarget`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `CCIfExtend`, `CCIfSubtarget` 等 TableGen 记录。

### Lines 19-36 / 第 19-36 行
```tablegen
  19: // Match if this specific argument was widened from a short vector type.
  20: class CCIfShortVector<CCAction A>
  21:     : CCIf<"OrigTy->isVectorTy() && OrigTy->getPrimitiveSizeInBits() <= 64", A>;
  22: 
  23: 
  24: //===----------------------------------------------------------------------===//
  25: // z/Linux return value calling convention
  26: //===----------------------------------------------------------------------===//
  27: def RetCC_SystemZ_ELF : CallingConv<[
  28:   // Promote i32 to i64 if it has an explicit extension type.
  29:   CCIfType<[i32], CCIfExtend<CCPromoteToType<i64>>>,
  30: 
  31:   // A SwiftError is returned in R9.
  32:   CCIfSwiftError<CCIfType<[i64], CCAssignToReg<[R9D]>>>,
  33: 
  34:   // ABI-compliant code returns 64-bit integers in R2.  Make the other
  35:   // call-clobbered argument registers available for code that doesn't
  36:   // care about the ABI.  (R6 is an argument register too, but is
```
- **EN**: This block declares or refines TableGen records such as `CCIfShortVector`, `RetCC_SystemZ_ELF`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `CCIfShortVector`, `RetCC_SystemZ_ELF` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 37-54 / 第 37-54 行
```tablegen
  37:   // call-saved and therefore not suitable for return values.)
  38:   CCIfType<[i32], CCAssignToReg<[R2L, R3L, R4L, R5L]>>,
  39:   CCIfType<[i64], CCAssignToReg<[R2D, R3D, R4D, R5D]>>,
  40: 
  41:   // ABI-complaint code returns float and double in F0.  Make the
  42:   // other floating-point argument registers available for code that
  43:   // doesn't care about the ABI.  All floating-point argument registers
  44:   // are call-clobbered, so we can use all of them here.
  45:   CCIfType<[f16], CCAssignToReg<[F0H, F2H, F4H, F6H]>>,
  46:   CCIfType<[f32], CCAssignToReg<[F0S, F2S, F4S, F6S]>>,
  47:   CCIfType<[f64], CCAssignToReg<[F0D, F2D, F4D, F6D]>>,
  48: 
  49:   // Similarly for vectors, with V24 being the ABI-compliant choice.
  50:   // Sub-128 vectors are returned in the same way, but they're widened
  51:   // to one of these types during type legalization.
  52:   CCIfSubtarget<"hasVector()",
  53:     CCIfType<[v16i8, v8i16, v4i32, v2i64, v8f16, v4f32, v2f64],
  54:              CCAssignToReg<[V24, V26, V28, V30, V25, V27, V29, V31]>>>
```
- **EN**: This span continues the file's main responsibility: defines SystemZ backend metadata using LLVM TableGen DSL; specifically it defines calling convention rules and register/stack assignments.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```tablegen
  55: ]>;
  56: 
  57: //===----------------------------------------------------------------------===//
  58: // z/Linux argument calling conventions for GHC
  59: //===----------------------------------------------------------------------===//
  60: def CC_SystemZ_GHC : CallingConv<[
  61:   // Pass in STG registers: Base, Sp, Hp, R1, R2, R3, R4, R5, R6, R7, R8, SpLim
  62:   CCIfType<[i64], CCAssignToReg<[R7D, R8D, R10D, R11D, R12D, R13D,
  63:                                  R6D, R2D, R3D, R4D, R5D, R9D]>>,
  64: 
  65:   // Pass in STG registers: F1, ..., F6
  66:   CCIfType<[f32], CCAssignToReg<[F8S, F9S, F10S, F11S, F0S, F1S]>>,
  67: 
  68:   // Pass in STG registers: D1, ..., D6
  69:   CCIfType<[f64], CCAssignToReg<[F12D, F13D, F14D, F15D, F2D, F3D]>>,
  70: 
  71:   // Pass in STG registers: XMM1, ..., XMM6
  72:   CCIfSubtarget<"hasVector()",
```
- **EN**: This block declares or refines TableGen records such as `CC_SystemZ_GHC`.
- **CN**: 该代码块声明或细化了 `CC_SystemZ_GHC` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73:     CCIfType<[v16i8, v8i16, v4i32, v2i64, v4f32, v2f64],
  74:              CCIfArgFixed<CCAssignToReg<[V16, V17, V18, V19, V20, V21]>>>>,
  75: 
  76:   // Fail otherwise
  77:   CCCustom<"CC_SystemZ_GHC_Error">
  78: ]>;
  79: 
  80: //===----------------------------------------------------------------------===//
  81: // z/Linux argument calling conventions
  82: //===----------------------------------------------------------------------===//
  83: def CC_SystemZ_ELF : CallingConv<[
  84:   CCIfCC<"CallingConv::GHC", CCDelegateTo<CC_SystemZ_GHC>>,
  85: 
  86:   // Promote i32 to i64 if it has an explicit extension type.
  87:   // The convention is that true integer arguments that are smaller
  88:   // than 64 bits should be marked as extended, but structures that
  89:   // are smaller than 64 bits shouldn't.
  90:   CCIfType<[i32], CCIfExtend<CCPromoteToType<i64>>>,
```
- **EN**: This block declares or refines TableGen records such as `CC_SystemZ_ELF`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `CC_SystemZ_ELF` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```tablegen
  91: 
  92:   // A SwiftSelf is passed in callee-saved R10.
  93:   CCIfSwiftSelf<CCIfType<[i64], CCAssignToReg<[R10D]>>>,
  94: 
  95:   // A SwiftError is passed in callee-saved R9.
  96:   CCIfSwiftError<CCIfType<[i64], CCAssignToReg<[R9D]>>>,
  97: 
  98:   // Force i128 (if the type is legal) and long double values to the stack
  99:   // and pass i64 pointers to them.
 100:   CCIfType<[i128, f128], CCPassIndirect<i64>>,
 101:   // If i128 is not legal, such values are already split into two i64 here,
 102:   // so we have to use a custom handler.
 103:   CCIfType<[i64], CCCustom<"CC_SystemZ_I128Indirect">>,
 104: 
 105:   // The first 5 integer arguments are passed in R2-R6.  Note that R6
 106:   // is call-saved.
 107:   CCIfType<[i32], CCAssignToReg<[R2L, R3L, R4L, R5L, R6L]>>,
 108:   CCIfType<[i64], CCAssignToReg<[R2D, R3D, R4D, R5D, R6D]>>,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```tablegen
 109: 
 110:   // The first 4 float and double arguments are passed in even registers F0-F6.
 111:   CCIfType<[f16], CCAssignToReg<[F0H, F2H, F4H, F6H]>>,
 112:   CCIfType<[f32], CCAssignToReg<[F0S, F2S, F4S, F6S]>>,
 113:   CCIfType<[f64], CCAssignToReg<[F0D, F2D, F4D, F6D]>>,
 114: 
 115:   // The first 8 named vector arguments are passed in V24-V31.  Sub-128 vectors
 116:   // are passed in the same way, but they're widened to one of these types
 117:   // during type legalization.
 118:   CCIfSubtarget<"hasVector()",
 119:     CCIfType<[v16i8, v8i16, v4i32, v2i64, v8f16, v4f32, v2f64],
 120:              CCIfArgFixed<CCAssignToReg<[V24, V26, V28, V30,
 121:                                          V25, V27, V29, V31]>>>>,
 122: 
 123:   // However, sub-128 vectors which need to go on the stack occupy just a
 124:   // single 8-byte-aligned 8-byte stack slot.  Pass as i64.
 125:   CCIfSubtarget<"hasVector()",
 126:     CCIfType<[v16i8, v8i16, v4i32, v2i64, v8f16, v4f32, v2f64],
```
- **EN**: This span continues the file's main responsibility: defines SystemZ backend metadata using LLVM TableGen DSL; specifically it defines calling convention rules and register/stack assignments.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 127-144 / 第 127-144 行
```tablegen
 127:              CCIfShortVector<CCBitConvertToType<i64>>>>,
 128: 
 129:   // Other vector arguments are passed in 8-byte-aligned 16-byte stack slots.
 130:   CCIfSubtarget<"hasVector()",
 131:     CCIfType<[v16i8, v8i16, v4i32, v2i64, v8f16, v4f32, v2f64],
 132:              CCAssignToStack<16, 8>>>,
 133: 
 134:   // Other arguments are passed in 8-byte-aligned 8-byte stack slots.
 135:   CCIfType<[i32, i64, f16, f32, f64], CCAssignToStack<8, 8>>
 136: ]>;
 137: 
 138: //===----------------------------------------------------------------------===//
 139: // z/Linux callee-saved registers
 140: //===----------------------------------------------------------------------===//
 141: def CSR_SystemZ_ELF : CalleeSavedRegs<(add (sequence "R%dD", 6, 15),
 142:                                        (sequence "F%dD", 8, 15))>;
 143: 
 144: // R9 is used to return SwiftError; remove it from CSR.
```
- **EN**: This block declares or refines TableGen records such as `CSR_SystemZ_ELF`.
- **CN**: 该代码块声明或细化了 `CSR_SystemZ_ELF` 等 TableGen 记录。

### Lines 145-162 / 第 145-162 行
```tablegen
 145: def CSR_SystemZ_SwiftError : CalleeSavedRegs<(sub CSR_SystemZ_ELF, R9D)>;
 146: 
 147: // "All registers" as used by the AnyReg calling convention.
 148: // Note that registers 0 and 1 are still defined as intra-call scratch
 149: // registers that may be clobbered e.g. by PLT stubs.
 150: def CSR_SystemZ_AllRegs : CalleeSavedRegs<(add (sequence "R%dD", 2, 15),
 151:                                                (sequence "F%dD", 0, 15))>;
 152: def CSR_SystemZ_AllRegs_Vector : CalleeSavedRegs<(add (sequence "R%dD", 2, 15),
 153:                                                       (sequence "V%d", 0, 31))>;
 154: 
 155: def CSR_SystemZ_NoRegs : CalleeSavedRegs<(add)>;
 156: 
 157: //===----------------------------------------------------------------------===//
 158: // z/OS XPLINK64 callee-saved registers
 159: //===----------------------------------------------------------------------===//
 160: def CSR_SystemZ_XPLINK64 : CalleeSavedRegs<(add (sequence "R%dD", 8, 15),
 161:                                                 (sequence "F%dD", 15, 8))>;
 162: 
```
- **EN**: This block declares or refines TableGen records such as `CSR_SystemZ_SwiftError`, `CSR_SystemZ_AllRegs`, `CSR_SystemZ_AllRegs_Vector`, `CSR_SystemZ_NoRegs`, `CSR_SystemZ_XPLINK64`.
- **CN**: 该代码块声明或细化了 `CSR_SystemZ_SwiftError`, `CSR_SystemZ_AllRegs`, `CSR_SystemZ_AllRegs_Vector`, `CSR_SystemZ_NoRegs`, `CSR_SystemZ_XPLINK64` 等 TableGen 记录。

### Lines 163-180 / 第 163-180 行
```tablegen
 163: def CSR_SystemZ_XPLINK64_Vector : CalleeSavedRegs<(add CSR_SystemZ_XPLINK64,
 164:                                                    (sequence "V%d", 23, 16))>;
 165: 
 166: //===----------------------------------------------------------------------===//
 167: // z/OS XPLINK64 return value calling convention
 168: //===----------------------------------------------------------------------===//
 169: def RetCC_SystemZ_XPLINK64 : CallingConv<[
 170:   // XPLINK64 ABI compliant code widens integral types smaller than i64
 171:   // to i64.
 172:   CCIfType<[i32], CCPromoteToType<i64>>,
 173: 
 174:   // Structs of size 1-24 bytes are returned in R1D, R2D, and R3D.
 175:   CCIfType<[i64], CCIfInReg<CCAssignToReg<[R1D, R2D, R3D]>>>,
 176:   // An i64 is returned in R3D. R2D and R1D provided for ABI non-compliant
 177:   // code.
 178:   CCIfType<[i64], CCAssignToReg<[R3D, R2D, R1D]>>,
 179: 
 180:   // ABI compliant code returns floating point values in FPR0, FPR2, FPR4
```
- **EN**: This block declares or refines TableGen records such as `CSR_SystemZ_XPLINK64_Vector`, `RetCC_SystemZ_XPLINK64`.
- **CN**: 该代码块声明或细化了 `CSR_SystemZ_XPLINK64_Vector`, `RetCC_SystemZ_XPLINK64` 等 TableGen 记录。

### Lines 181-198 / 第 181-198 行
```tablegen
 181:   // and FPR6, using as many registers as required.
 182:   // All floating point return-value registers are call-clobbered.
 183:   CCIfType<[f32], CCAssignToReg<[F0S, F2S, F4S, F6S]>>,
 184:   CCIfType<[f64], CCAssignToReg<[F0D, F2D, F4D, F6D]>>,
 185: 
 186:   // ABI compliant code returns f128 in F0D and F2D, hence F0Q.
 187:   // F4D and F6D, hence F4Q are used for complex long double types.
 188:   CCIfType<[f128], CCAssignToReg<[F0Q,F4Q]>>,
 189: 
 190:   // ABI compliant code returns vectors in VR24 but other registers
 191:   // are provided for code that does not care about the ABI.
 192:   CCIfSubtarget<"hasVector()",
 193:     CCIfType<[v16i8, v8i16, v4i32, v2i64, v4f32, v2f64],
 194:              CCAssignToReg<[V24, V25, V26, V27, V28, V29, V30, V31]>>>
 195: ]>;
 196: 
 197: //===----------------------------------------------------------------------===//
 198: // z/OS XPLINK64 argument calling conventions
```
- **EN**: This span continues the file's main responsibility: defines SystemZ backend metadata using LLVM TableGen DSL; specifically it defines calling convention rules and register/stack assignments.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 199-216 / 第 199-216 行
```tablegen
 199: //===----------------------------------------------------------------------===//
 200: // XPLink uses a logical argument list consisting of contiguous register-size
 201: // words (8 bytes in 64-Bit mode) where some arguments are passed in registers
 202: // and some in storage.
 203: // Even though 3 GPRs, 4 FPRs, and 8 VRs may be used,
 204: // space must be reserved for all the args on stack.
 205: // The first three register-sized words of the parameter area are passed in
 206: // GPRs 1-3. FP values and vector-type arguments are instead passed in FPRs
 207: // and VRs respectively, but if a FP value or vector argument occupies one of
 208: // the first three register-sized words of the parameter area, the corresponding
 209: // GPR's value is not used to pass arguments.
 210: //
 211: // The XPLINK64 Calling Convention is fully specified in Chapter 22 of the z/OS
 212: // Language Environment Vendor Interfaces. Appendix B of the same document contains
 213: // examples.
 214: 
 215: def CC_SystemZ_XPLINK64 : CallingConv<[
 216:   // XPLINK64 ABI compliant code widens integral types smaller than i64
```
- **EN**: This block declares or refines TableGen records such as `CC_SystemZ_XPLINK64`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `CC_SystemZ_XPLINK64` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 217-234 / 第 217-234 行
```tablegen
 217:   // to i64 before placing the parameters either on the stack or in registers.
 218:   CCIfType<[i32], CCIfExtend<CCPromoteToType<i64>>>,
 219:   // Promote f32 to f64 and bitcast to i64, if it needs to be passed in GPRs.
 220:   // Although we assign the f32 vararg to be bitcast, it will first be promoted
 221:   // to an f64 within convertValVTToLocVT().
 222:   CCIfType<[f32, f64], CCIfArgVarArg<CCBitConvertToType<i64>>>,
 223:   // Pointers are always passed in full 64-bit registers.
 224:   CCIfPtr<CCCustom<"CC_XPLINK64_Pointer">>,
 225:   // long double, can only be passed in GPR2 and GPR3, if available,
 226:   // hence R2Q
 227:   CCIfType<[f128], CCIfArgVarArg<CCCustom<"CC_XPLINK64_Allocate128BitVararg">>>,
 228:   // Non fixed vector arguments are treated in the same way as long
 229:   // doubles.
 230:   CCIfSubtarget<"hasVector()",
 231:     CCIfType<[v16i8, v8i16, v4i32, v2i64, v4f32, v2f64],
 232:       CCIfArgVarArg<CCCustom<"CC_XPLINK64_Allocate128BitVararg">>>>,
 233: 
 234:   // A SwiftSelf is passed in callee-saved R10.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```tablegen
 235:   CCIfSwiftSelf<CCIfType<[i64], CCAssignToReg<[R10D]>>>,
 236: 
 237:   // A SwiftError is passed in R0.
 238:   CCIfSwiftError<CCIfType<[i64], CCAssignToReg<[R0D]>>>,
 239: 
 240:   // Force i128 values to the stack and pass i64 pointers to them.
 241:   CCIfType<[i128], CCPassIndirect<i64>>,
 242:   // If i128 is not legal, such values are already split into two i64 here,
 243:   // so we have to use a custom handler.
 244:   CCIfType<[i64], CCCustom<"CC_SystemZ_I128Indirect">>,
 245:   // The first 3 integer arguments are passed in registers R1D-R3D.
 246:   // The rest will be passed in the user area. The address offset of the user
 247:   // area can be found in register R4D.
 248:   CCIfType<[i64], CCAssignToRegAndStack<[R1D, R2D, R3D], 8, 8>>,
 249: 
 250:   // The first 8 named vector arguments are passed in V24-V31. Sub-128 vectors
 251:   // are passed in the same way, but they're widened to one of these types
 252:   // during type legalization.
```
- **EN**: This span continues the file's main responsibility: defines SystemZ backend metadata using LLVM TableGen DSL; specifically it defines calling convention rules and register/stack assignments.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 253-270 / 第 253-270 行
```tablegen
 253:   CCIfSubtarget<"hasVector()",
 254:     CCIfType<[v16i8, v8i16, v4i32, v2i64, v4f32, v2f64],
 255:       CCIfArgFixed<CCCustom<"CC_XPLINK64_Shadow_Reg">>>>,
 256:   CCIfSubtarget<"hasVector()",
 257:     CCIfType<[v16i8, v8i16, v4i32, v2i64, v4f32, v2f64],
 258:               CCIfArgFixed<CCAssignToRegAndStack<[V24, V25, V26, V27,
 259:                                                   V28, V29, V30, V31], 16, 8>>>>,
 260: 
 261:   // The first 4 named float and double arguments are passed in registers
 262:   // FPR0-FPR6. The rest will be passed in the user area.
 263:   CCIfType<[f32, f64], CCIfArgFixed<CCCustom<"CC_XPLINK64_Shadow_Reg">>>,
 264:   CCIfType<[f32],
 265:            CCIfArgFixed<CCAssignToRegAndStack<[F0S, F2S, F4S, F6S], 4, 8>>>,
 266:   CCIfType<[f64],
 267:            CCIfArgFixed<CCAssignToRegAndStack<[F0D, F2D, F4D, F6D], 8, 8>>>,
 268: 
 269:   // The first 2 long double arguments are passed in register FPR0/FPR2
 270:   // and FPR4/FPR6. The rest will be passed in the user area.
```
- **EN**: This span continues the file's main responsibility: defines SystemZ backend metadata using LLVM TableGen DSL; specifically it defines calling convention rules and register/stack assignments.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 271-288 / 第 271-288 行
```tablegen
 271:   CCIfType<[f128], CCIfArgFixed<CCCustom<"CC_XPLINK64_Shadow_Reg">>>,
 272:   CCIfType<[f128], CCIfArgFixed<CCAssignToRegAndStack<[F0Q, F4Q], 16, 8>>>,
 273: 
 274:   // Other arguments are passed in 8-byte-aligned 8-byte stack slots.
 275:   CCIfType<[i32, i64, f32, f64], CCAssignToStack<8, 8>>,
 276:   // Other f128 arguments are passed in 8-byte-aligned 16-byte stack slots.
 277:   CCIfType<[f128], CCAssignToStack<16, 8>>,
 278:   // Vector arguments are passed in 8-byte-alinged 16-byte stack slots too.
 279:   CCIfSubtarget<"hasVector()",
 280:     CCIfType<[v16i8, v8i16, v4i32, v2i64, v4f32, v2f64],
 281:              CCAssignToStack<16, 8>>>
 282: ]>;
 283: 
 284: //===----------------------------------------------------------------------===//
 285: // s390x return value calling convention
 286: //===----------------------------------------------------------------------===//
 287: 
 288: def RetCC_SystemZ : CallingConv<[
```
- **EN**: This block declares or refines TableGen records such as `RetCC_SystemZ`.
- **CN**: 该代码块声明或细化了 `RetCC_SystemZ` 等 TableGen 记录。

### Lines 289-306 / 第 289-306 行
```tablegen
 289:   // zOS XPLINK64
 290:   CCIfSubtarget<"isTargetXPLINK64()", CCDelegateTo<RetCC_SystemZ_XPLINK64>>,
 291: 
 292:   // ELF Linux SystemZ
 293:   CCIfSubtarget<"isTargetELF()", CCDelegateTo<RetCC_SystemZ_ELF>>
 294: ]>;
 295: 
 296: 
 297: //===----------------------------------------------------------------------===//
 298: // s390x argument calling conventions
 299: //===----------------------------------------------------------------------===//
 300: def CC_SystemZ : CallingConv<[
 301:   // zOS XPLINK64
 302:   CCIfSubtarget<"isTargetXPLINK64()", CCDelegateTo<CC_SystemZ_XPLINK64>>,
 303: 
 304:   // ELF Linux SystemZ
 305:   CCIfSubtarget<"isTargetELF()", CCDelegateTo<CC_SystemZ_ELF>>
 306: ]>;
```
- **EN**: This block declares or refines TableGen records such as `CC_SystemZ`.
- **CN**: 该代码块声明或细化了 `CC_SystemZ` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
