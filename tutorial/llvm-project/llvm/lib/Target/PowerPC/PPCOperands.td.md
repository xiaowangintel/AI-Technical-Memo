# PPCOperands.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCOperands.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCOperands.td - PowerPC instruction operands. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCOperands.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===-- PPCOperands.td - PowerPC instruction operands -------*- tablegen -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```tablegen
//===----------------------------------------------------------------------===//
//
// This file defines PowerPC instruction operands, including immediate
// operands and addressing modes.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file defines PowerPC instruction operands, including immediate".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file defines PowerPC instruction operands, including immediate”。

### Lines 12-13

```tablegen
//===----------------------------------------------------------------------===//
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 14-15

```tablegen
//===----------------------------------------------------------------------===//
// Immediate operand base classes
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Immediate operand base classes".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Immediate operand base classes”。

### Lines 16-22

```tablegen
//===----------------------------------------------------------------------===//

// Base class for immediate AsmOperandClass definitions.
class ImmediateAsmOperand<string predicate, string render="addImmOperands">
    : AsmOperandClass {
  let Name = NAME;
  let PredicateMethod = predicate;
```
- **EN**: Declares a backend-facing type `ImmediateAsmOperand` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `ImmediateAsmOperand`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 23-29

```tablegen
  let RenderMethod = render;
}

// Base class for signed immediate operands.
class ImmediateOp<ValueType vt, string asmop, int width> : Operand<vt> {
  let PrintMethod = "printSImmOperand<"#width#">";
  let DecoderMethod = "decodeSImmOperand<"#width#">";
```
- **EN**: Declares a backend-facing type `ImmediateOp` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `ImmediateOp`，并勾勒出周边代码会依赖的接口或状态。

### Lines 30-36

```tablegen
  let ParserMatchClass = !cast<AsmOperandClass>(asmop);
  let OperandType = "OPERAND_IMMEDIATE";
}

// Base class for unsigned immediate operands.
class UImmediateOp<ValueType vt, string asmop, int width>
    : ImmediateOp<vt, asmop, width> {
```
- **EN**: Declares a backend-facing type `UImmediateOp` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `UImmediateOp`，并勾勒出周边代码会依赖的接口或状态。

### Lines 37-43

```tablegen
  let PrintMethod = "printUImmOperand<"#width#">";
  let DecoderMethod = "decodeUImmOperand<"#width#">";
}

// Base class for signed immediate operands with relocation.
class ImmediateRelocOp<ValueType vt, string asmop, int width, string fixup,
                       string decoder = ""> : Operand<vt> {
```
- **EN**: Declares a backend-facing type `ImmediateRelocOp` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `ImmediateRelocOp`，并勾勒出周边代码会依赖的接口或状态。

### Lines 44-53

```tablegen
  let PrintMethod = "print"#asmop#"Operand";
  let DecoderMethod = !if(!eq(decoder, ""),
                          "decodeSImmOperand<"#width#">",
                          decoder);
  let ParserMatchClass = !cast<AsmOperandClass>(asmop);
  let EncoderMethod = "getImmEncoding<" # fixup # ">";
  let OperandType = "OPERAND_IMMEDIATE";
}

// Base class for unsigned immediate operands with relocation.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 54-60

```tablegen
class UImmediateRelocOp<ValueType vt, string asmop, int width, string fixup,
                        string decoder = "">
    : ImmediateRelocOp<vt, asmop, width, fixup> {
  let DecoderMethod = !if(!eq(decoder, ""),
                          "decodeUImmOperand<"#width#">",
                          decoder);
}
```
- **EN**: Declares a backend-facing type `UImmediateRelocOp` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `UImmediateRelocOp`，并勾勒出周边代码会依赖的接口或状态。

### Lines 61-63

```tablegen
//===----------------------------------------------------------------------===//
// Multiclasses for complete immediate definitions
// (AsmOperand + Operand + ImmLeaf).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Multiclasses for complete immediate definitions".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Multiclasses for complete immediate definitions”。

### Lines 64-70

```tablegen
//===----------------------------------------------------------------------===//

multiclass ImmPatterns<ValueType vt, code pred, SDNodeXForm xform> {
  def _pat : ImmLeaf<vt, pred, xform>;
  def _timm : TImmLeaf<vt, pred, xform>;
}
```
- **EN**: Adds declarative TableGen records such as `ImmPatterns` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ImmPatterns`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 71-79

```tablegen
multiclass SignedImmediate<ValueType vt, code pred, SDNodeXForm xform,
                           string asmop, int width >
    : ImmPatterns<vt, pred, xform> {
  def "" : ImmediateOp<vt, asmop, width>;
}

multiclass UnsignedImmediate<ValueType vt, code pred, SDNodeXForm xform,
                             string asmop, int width >
    : ImmPatterns<vt, pred, xform> {
```
- **EN**: Adds declarative TableGen records such as `SignedImmediate`, `UnsignedImmediate` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SignedImmediate`, `UnsignedImmediate`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 80-89

```tablegen
  def "" : UImmediateOp<vt, asmop, width>;
}

multiclass SignedImmediateReloc<ValueType vt, code pred, SDNodeXForm xform,
                                string asmop, int width, string fixup>
    : ImmPatterns<vt, pred, xform> {
  def "" : ImmediateRelocOp<vt, asmop, width, fixup>;
}

// Helper multiclass for unsigned immediates with relocation fixup string.
```
- **EN**: Adds declarative TableGen records such as `SignedImmediateReloc` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SignedImmediateReloc`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 90-97

```tablegen
multiclass UnsignedImmediateReloc<ValueType vt, code pred, SDNodeXForm xform,
                                  string asmop, int width, string fixup>
    : ImmPatterns<vt, pred, xform> {
  def "" : UImmediateRelocOp<vt, asmop, width, fixup>;
}

// Multiclass for signed immediate operands with both regular and
// PC-relative versions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Multiclass for signed immediate operands with both regular and". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Multiclass for signed immediate operands with both regular and”。 这些声明会进入生成式模式匹配逻辑。

### Lines 98-105

```tablegen
multiclass SignedImmediateWithPCRel<ValueType vt, code pred, SDNodeXForm xform,
                                    string asmop, int width, string fixup_imm,
                                    string fixup_pcrel>
      : SignedImmediateReloc<vt, pred, xform, asmop, width, fixup_imm> {
  // PC-relative immediate: instantiate with PC-relative fixup
  defm _pcrel : SignedImmediateReloc<vt, pred, xform, asmop, width, fixup_pcrel>;
}
```
- **EN**: Adds declarative TableGen records such as `SignedImmediateWithPCRel` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SignedImmediateWithPCRel`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 106-107

```tablegen
//===----------------------------------------------------------------------===//
// Immediate transformation functions
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Immediate transformation functions".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Immediate transformation functions”。

### Lines 108-115

```tablegen
//===----------------------------------------------------------------------===//

// NEG_S32 - Negate a signed 32-bit immediate value
def NEG_S32 : SDNodeXForm<imm, [{
  // Transformation function: -imm
  return getI32Imm(-N->getSExtValue(), SDLoc(N));
}]>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "NEG_S32 - Negate a signed 32-bit immediate value". Notable symbols in this range include `NEG_S32`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“NEG_S32 - Negate a signed 32-bit immediate value”。 该区间中较显眼的符号包括 `NEG_S32`。

### Lines 116-117

```tablegen
//===----------------------------------------------------------------------===//
// Immediate AsmOperand definitions
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Immediate AsmOperand definitions".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Immediate AsmOperand definitions”。

### Lines 118-124

```tablegen
//===----------------------------------------------------------------------===//
def U1Imm : ImmediateAsmOperand<"isUImm<1>">;
def U2Imm : ImmediateAsmOperand<"isUImm<2>">;
def U3Imm : ImmediateAsmOperand<"isUImm<3>">;
def U4Imm : ImmediateAsmOperand<"isUImm<4>">;
def U5Imm : ImmediateAsmOperand<"isUImm<5>">;
def U6Imm : ImmediateAsmOperand<"isUImm<6>">;
```
- **EN**: Adds declarative TableGen records such as `U1Imm`, `U2Imm`, `U3Imm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `U1Imm`, `U2Imm`, `U3Imm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 125-132

```tablegen
def U7Imm : ImmediateAsmOperand<"isUImm<7>">;
def U8Imm : ImmediateAsmOperand<"isUImm<8>">;
def U10Imm : ImmediateAsmOperand<"isUImm<10>">;
def U12Imm : ImmediateAsmOperand<"isUImm<12>">;
def U32Imm : ImmediateAsmOperand<"isUImm<32>">;
def S5Imm : ImmediateAsmOperand<"isSImm<5>">;

// Special cases that have custom predicat and/or render method.
```
- **EN**: Adds declarative TableGen records such as `U7Imm`, `U8Imm`, `U10Imm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `U7Imm`, `U8Imm`, `U10Imm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 133-139

```tablegen
def ATBitsAsHint : ImmediateAsmOperand<"isATBitsAsHint">; //Predicate always fails.
def ImmZero : ImmediateAsmOperand<"isImmZero">;
def U16Imm : ImmediateAsmOperand<"isU16Imm","addU16ImmOperands">;
def S16Imm : ImmediateAsmOperand<"isS16Imm","addS16ImmOperands">;
def S17Imm : ImmediateAsmOperand<"isS17Imm","addS16ImmOperands">;
def S32Imm : ImmediateAsmOperand<"isS32Imm">;
def S34Imm : ImmediateAsmOperand<"isS34Imm">;
```
- **EN**: Adds declarative TableGen records such as `ATBitsAsHint`, `ImmZero`, `U16Imm` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATBitsAsHint`, `ImmZero`, `U16Imm`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 140-141

```tablegen
def NegS32Imm : ImmediateAsmOperand<"isS32Imm","addNegImmOperands">;
```
- **EN**: Adds declarative TableGen records such as `NegS32Imm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NegS32Imm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 142-143

```tablegen
//===----------------------------------------------------------------------===//
// i32 immediate operands
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "i32 immediate operands".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“i32 immediate operands”。

### Lines 144-151

```tablegen
//===----------------------------------------------------------------------===//

defm u1imm : UnsignedImmediate<i32,
  [{ return isUInt<1>(Imm); }], NOOP_SDNodeXForm,
  "U1Imm", 1>;
defm u2imm : UnsignedImmediate<i32,
  [{ return isUInt<2>(Imm); }], NOOP_SDNodeXForm,
  "U2Imm", 2>;
```
- **EN**: Adds declarative TableGen records such as `u1imm`, `u2imm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `u1imm`, `u2imm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 152-160

```tablegen
defm u3imm : UnsignedImmediate<i32,
  [{ return isUInt<3>(Imm); }], NOOP_SDNodeXForm,
  "U3Imm", 3>;
defm u4imm : UnsignedImmediate<i32,
  [{ return isUInt<4>(Imm); }], NOOP_SDNodeXForm,
  "U4Imm", 4>;
defm u5imm : UnsignedImmediate<i32,
  [{ return isUInt<5>(Imm); }], NOOP_SDNodeXForm,
  "U5Imm", 5>;
```
- **EN**: Adds declarative TableGen records such as `u3imm`, `u4imm`, `u5imm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `u3imm`, `u4imm`, `u5imm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 161-172

```tablegen
defm u6imm : UnsignedImmediate<i32,
  [{ return isUInt<6>(Imm); }], NOOP_SDNodeXForm,
  "U6Imm", 6>;
defm u7imm : UnsignedImmediate<i32,
  [{ return isUInt<7>(Imm); }], NOOP_SDNodeXForm,
  "U7Imm", 7>;
defm u8imm : UnsignedImmediate<i32,
  [{ return isUInt<8>(Imm); }], NOOP_SDNodeXForm,
  "U8Imm", 8>;

// Truncating version for BUILD_VECTOR operands that may be sign-extended.
// Uses the same parser class as u8imm but with a truncating print method.
```
- **EN**: Adds declarative TableGen records such as `u6imm`, `u7imm`, `u8imm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `u6imm`, `u7imm`, `u8imm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 173-181

```tablegen
def u8imm_trunc : UImmediateOp<i32, "U8Imm", 8> {
  let PrintMethod = "printU8ImmOperandTrunc";
}
defm u10imm : UnsignedImmediate<i32,
  [{ return isUInt<10>(Imm); }], NOOP_SDNodeXForm,
  "U10Imm", 10>;
defm u12imm : UnsignedImmediate<i32,
  [{ return isUInt<12>(Imm); }], NOOP_SDNodeXForm,
  "U12Imm", 12>;
```
- **EN**: Adds declarative TableGen records such as `u8imm_trunc`, `u10imm`, `u12imm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `u8imm_trunc`, `u10imm`, `u12imm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 182-189

```tablegen
defm u32imm : UnsignedImmediate<i32,
  [{ return isUInt<32>(Imm); }], NOOP_SDNodeXForm,
  "U32Imm", 32>;

defm s5imm : SignedImmediate<i32,
  [{ return isInt<5>(Imm); }], NOOP_SDNodeXForm,
  "S5Imm", 5>;
```
- **EN**: Adds declarative TableGen records such as `u32imm`, `s5imm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `u32imm`, `s5imm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 190-196

```tablegen
defm s32imm : SignedImmediateWithPCRel<i32,
  [{ return isInt<32>(Imm); }], NOOP_SDNodeXForm,
  "S32Imm", 32, "PPC::fixup_ppc_imm32", "PPC::fixup_ppc_pcrel32">;

defm s34imm : SignedImmediateReloc<i32,
  [{ return isInt<34>(Imm); }], NOOP_SDNodeXForm,
  "S34Imm", 34, "PPC::fixup_ppc_imm34">;
```
- **EN**: Adds declarative TableGen records such as `s32imm`, `s34imm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `s32imm`, `s34imm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 197-200

```tablegen
defm neg_s32imm : SignedImmediateReloc<i32,
  [{ return isInt<32>(Imm); }], NEG_S32,
  "NegS32Imm", 32, "PPC::fixup_ppc_imm32">;
```
- **EN**: Adds declarative TableGen records such as `neg_s32imm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `neg_s32imm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 201-202

```tablegen
//===----------------------------------------------------------------------===//
// i64 immediate operands
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "i64 immediate operands".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“i64 immediate operands”。

### Lines 203-210

```tablegen
//===----------------------------------------------------------------------===//

defm s32imm64 : SignedImmediateWithPCRel<i64,
  [{ return isInt<32>(Imm); }], NOOP_SDNodeXForm,
  "S32Imm", 32, "PPC::fixup_ppc_imm32", "PPC::fixup_ppc_pcrel32">;
defm s34imm64 : SignedImmediateWithPCRel<i64,
  [{ return isInt<34>(Imm); }], NOOP_SDNodeXForm,
  "S34Imm", 34, "PPC::fixup_ppc_imm34", "PPC::fixup_ppc_pcrel34">;
```
- **EN**: Adds declarative TableGen records such as `s32imm64`, `s34imm64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `s32imm64`, `s34imm64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 211-215

```tablegen
defm neg_s32imm64 : SignedImmediateReloc<i64,
  [{ return isInt<32>(Imm); }], NEG_S32,
  "NegS32Imm", 32, "PPC::fixup_ppc_imm32">;
```
- **EN**: Adds declarative TableGen records such as `neg_s32imm64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `neg_s32imm64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 216-217

```tablegen
//===----------------------------------------------------------------------===//
// Special case immediate operands
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Special case immediate operands".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Special case immediate operands”。

### Lines 218-224

```tablegen
//===----------------------------------------------------------------------===//

// immZero represents a hardcoded zero register encoding, it is NOT used in DAG
// patterns. It is only used as an instruction operand for assembly/disassembly,
// specifically to represent a hardcoded zero register value in PC-relative
// addressing modes.
def immZero : Operand<i32> {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "immZero represents a hardcoded zero register encoding, it is NOT used in DAG". Notable symbols in this range include `immZero`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“immZero represents a hardcoded zero register encoding, it is NOT used in DAG”。 该区间中较显眼的符号包括 `immZero`。

### Lines 225-234

```tablegen
  let PrintMethod = "printImmZeroOperand";
  let ParserMatchClass = ImmZero;
  let DecoderMethod = "decodeImmZeroOperand";
  let OperandType = "OPERAND_IMMEDIATE";
}

// atimm is used to represent branch prediction hints, not a general immediate
// value. It's a 2-bit AT (Address Translation) field in PPC branch instructions
// and is an assembly-only operand that prints as `+` or `-` symbols, not
// numeric values.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "atimm is used to represent branch prediction hints, not a general immediate".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“atimm is used to represent branch prediction hints, not a general immediate”。

### Lines 235-241

```tablegen
def atimm : Operand<i32> {
  let PrintMethod = "printATBitsAsHint";
  let ParserMatchClass = ATBitsAsHint;
  let OperandType = "OPERAND_IMMEDIATE";
}

// Special cases: s16imm and u16imm have custom encoder methods.
```
- **EN**: Adds declarative TableGen records such as `atimm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `atimm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 242-249

```tablegen
defm s16imm : SignedImmediateReloc<i32,
  [{ return isInt<16>(Imm); }], NOOP_SDNodeXForm,
  "S16Imm", 16, "PPC::fixup_ppc_half16">;
defm u16imm : UnsignedImmediateReloc<i32,
  [{ return isUInt<16>(Imm); }], NOOP_SDNodeXForm,
  "U16Imm", 16, "PPC::fixup_ppc_half16">;

// s16imm64 uses imm64SExt16 pattern to match the operand type.
```
- **EN**: Adds declarative TableGen records such as `s16imm`, `u16imm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `s16imm`, `u16imm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 250-260

```tablegen
def s16imm64 : ImmediateRelocOp<i64, "S16Imm", 16, "PPC::fixup_ppc_half16">;

// u16imm64 uses two different patterns depending on the instruction context:
// * immZExt16 - For low 16-bit immediates
// * imm16ShiftedZExt - For high 16-bit immediates (shifted)
def u16imm64 : UImmediateRelocOp<i64, "U16Imm", 16, "PPC::fixup_ppc_half16">;

// Special case: s17imm uses S16Imm print method but accepts wider range.
// This operand type is used for addis/lis to allow the assembler parser
// to accept immediates in the range -65536..65535 for compatibility
// with the GNU assembler. The operand is treated as 16-bit otherwise.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "u16imm64 uses two different patterns depending on the instruction context:". Notable symbols in this range include `s16imm64`, `u16imm64`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“u16imm64 uses two different patterns depending on the instruction context:”。 该区间中较显眼的符号包括 `s16imm64`, `u16imm64`。

### Lines 261-267

```tablegen
def s17imm : ImmediateRelocOp<i32, "S17Imm", 16, "PPC::fixup_ppc_half16"> {
  let PrintMethod = "printS16ImmOperand";
}
def s17imm64 : ImmediateRelocOp<i64, "S17Imm", 16, "PPC::fixup_ppc_half16"> {
  let PrintMethod = "printS16ImmOperand";
}
```
- **EN**: Adds declarative TableGen records such as `s17imm`, `s17imm64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `s17imm`, `s17imm64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 268-269

```tablegen
//===----------------------------------------------------------------------===//
// Branch target operands
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Branch target operands".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Branch target operands”。

### Lines 270-276

```tablegen
//===----------------------------------------------------------------------===//

def PPCDirectBrAsmOperand : AsmOperandClass {
  let Name = "DirectBr"; let PredicateMethod = "isDirectBr";
  let RenderMethod = "addBranchTargetOperands";
}
def directbrtarget : Operand<OtherVT> {
```
- **EN**: Adds declarative TableGen records such as `PPCDirectBrAsmOperand`, `directbrtarget` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCDirectBrAsmOperand`, `directbrtarget`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 277-283

```tablegen
  let PrintMethod = "printBranchOperand";
  let EncoderMethod = "getDirectBrEncoding";
  let DecoderMethod = "decodeDirectBrTarget";
  let ParserMatchClass = PPCDirectBrAsmOperand;
  let OperandType = "OPERAND_PCREL";
}
def absdirectbrtarget : Operand<OtherVT> {
```
- **EN**: Adds declarative TableGen records such as `absdirectbrtarget` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `absdirectbrtarget`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 284-290

```tablegen
  let PrintMethod = "printAbsBranchOperand";
  let EncoderMethod = "getAbsDirectBrEncoding";
  let DecoderMethod = "decodeDirectBrTarget";
  let ParserMatchClass = PPCDirectBrAsmOperand;
}
def PPCCondBrAsmOperand : AsmOperandClass {
  let Name = "CondBr"; let PredicateMethod = "isCondBr";
```
- **EN**: Adds declarative TableGen records such as `PPCCondBrAsmOperand` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCCondBrAsmOperand`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 291-297

```tablegen
  let RenderMethod = "addBranchTargetOperands";
}
def condbrtarget : Operand<OtherVT> {
  let PrintMethod = "printBranchOperand";
  let EncoderMethod = "getCondBrEncoding";
  let DecoderMethod = "decodeCondBrTarget";
  let ParserMatchClass = PPCCondBrAsmOperand;
```
- **EN**: Adds declarative TableGen records such as `condbrtarget` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `condbrtarget`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 298-305

```tablegen
  let OperandType = "OPERAND_PCREL";
}
def abscondbrtarget : Operand<OtherVT> {
  let PrintMethod = "printAbsBranchOperand";
  let EncoderMethod = "getAbsCondBrEncoding";
  let DecoderMethod = "decodeCondBrTarget";
  let ParserMatchClass = PPCCondBrAsmOperand;
}
```
- **EN**: Adds declarative TableGen records such as `abscondbrtarget` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `abscondbrtarget`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 306-312

```tablegen
def calltarget : Operand<iPTR> {
  let PrintMethod = "printBranchOperand";
  let EncoderMethod = "getDirectBrEncoding";
  let DecoderMethod = "decodeDirectBrTarget";
  let ParserMatchClass = PPCDirectBrAsmOperand;
  let OperandType = "OPERAND_PCREL";
}
```
- **EN**: Adds declarative TableGen records such as `calltarget` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `calltarget`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 313-319

```tablegen
def abscalltarget : Operand<iPTR> {
  let PrintMethod = "printAbsBranchOperand";
  let EncoderMethod = "getAbsDirectBrEncoding";
  let DecoderMethod = "decodeDirectBrTarget";
  let ParserMatchClass = PPCDirectBrAsmOperand;
}
```
- **EN**: Adds declarative TableGen records such as `abscalltarget` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `abscalltarget`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 320-321

```tablegen
//===----------------------------------------------------------------------===//
// CR bit mask operand
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "CR bit mask operand".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“CR bit mask operand”。

### Lines 322-328

```tablegen
//===----------------------------------------------------------------------===//

def PPCCRBitMaskOperand : AsmOperandClass {
 let Name = "CRBitMask"; let PredicateMethod = "isCRBitMask";
}
def crbitm: Operand<i8> {
  let PrintMethod = "printcrbitm";
```
- **EN**: Adds declarative TableGen records such as `PPCCRBitMaskOperand`, `crbitm` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCCRBitMaskOperand`, `crbitm`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 329-333

```tablegen
  let EncoderMethod = "get_crbitm_encoding";
  let DecoderMethod = "decodeCRBitMOperand";
  let ParserMatchClass = PPCCRBitMaskOperand;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 334-335

```tablegen
//===----------------------------------------------------------------------===//
// Displacement operands
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Displacement operands".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Displacement operands”。

### Lines 336-342

```tablegen
//===----------------------------------------------------------------------===//

def PPCDispRI34Operand : AsmOperandClass {
  let Name = "DispRI34"; let PredicateMethod = "isS34Imm";
  let RenderMethod = "addImmOperands";
}
def dispRI34 : Operand<iPTR> {
```
- **EN**: Adds declarative TableGen records such as `PPCDispRI34Operand`, `dispRI34` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCDispRI34Operand`, `dispRI34`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 343-349

```tablegen
  let ParserMatchClass = PPCDispRI34Operand;
  let EncoderMethod = "getDispRI34Encoding";
  let DecoderMethod = "decodeSImmOperand<34>";
}
def dispRI34_pcrel : Operand<iPTR> {
  let ParserMatchClass = PPCDispRI34Operand;
  let EncoderMethod = "getDispRI34PCRelEncoding";
```
- **EN**: Adds declarative TableGen records such as `dispRI34_pcrel` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `dispRI34_pcrel`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 350-356

```tablegen
  let DecoderMethod = "decodeSImmOperand<34>";
}
def PPCDispRIOperand : AsmOperandClass {
 let Name = "DispRI"; let PredicateMethod = "isS16Imm";
 let RenderMethod = "addS16ImmOperands";
}
def dispRI : Operand<iPTR> {
```
- **EN**: Adds declarative TableGen records such as `PPCDispRIOperand`, `dispRI` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCDispRIOperand`, `dispRI`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 357-363

```tablegen
  let ParserMatchClass = PPCDispRIOperand;
  let EncoderMethod = "getDispRIEncoding";
}
def PPCDispRIXOperand : AsmOperandClass {
 let Name = "DispRIX"; let PredicateMethod = "isS16ImmX4";
 let RenderMethod = "addS16ImmOperands";
}
```
- **EN**: Adds declarative TableGen records such as `PPCDispRIXOperand` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCDispRIXOperand`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 364-370

```tablegen
def dispRIX : Operand<iPTR> {
  let ParserMatchClass = PPCDispRIXOperand;
  let EncoderMethod = "getDispRIXEncoding";
  let DecoderMethod = "decodeDispRIXOperand";
}
def PPCDispRIHashOperand : AsmOperandClass {
  let Name = "DispRIHash"; let PredicateMethod = "isHashImmX8";
```
- **EN**: Adds declarative TableGen records such as `dispRIX`, `PPCDispRIHashOperand` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `dispRIX`, `PPCDispRIHashOperand`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 371-377

```tablegen
  let RenderMethod = "addImmOperands";
}
def dispRIHash : Operand<iPTR> {
  let ParserMatchClass = PPCDispRIHashOperand;
  let EncoderMethod = "getDispRIHashEncoding";
  let DecoderMethod = "decodeDispRIHashOperand";
}
```
- **EN**: Adds declarative TableGen records such as `dispRIHash` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `dispRIHash`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 378-384

```tablegen
def PPCDispRIX16Operand : AsmOperandClass {
 let Name = "DispRIX16"; let PredicateMethod = "isS16ImmX16";
 let RenderMethod = "addS16ImmOperands";
}
def dispRIX16 : Operand<iPTR> {
  let ParserMatchClass = PPCDispRIX16Operand;
  let EncoderMethod = "getDispRIX16Encoding";
```
- **EN**: Adds declarative TableGen records such as `PPCDispRIX16Operand`, `dispRIX16` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCDispRIX16Operand`, `dispRIX16`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 385-391

```tablegen
  let DecoderMethod = "decodeDispRIX16Operand";
}
def PPCDispSPE8Operand : AsmOperandClass {
 let Name = "DispSPE8"; let PredicateMethod = "isU8ImmX8";
 let RenderMethod = "addImmOperands";
}
def dispSPE8 : Operand<iPTR> {
```
- **EN**: Adds declarative TableGen records such as `PPCDispSPE8Operand`, `dispSPE8` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCDispSPE8Operand`, `dispSPE8`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 392-399

```tablegen
  let ParserMatchClass = PPCDispSPE8Operand;
  let DecoderMethod = "decodeDispSPE8Operand";
  let EncoderMethod = "getDispSPE8Encoding";
}
def PPCDispSPE4Operand : AsmOperandClass {
 let Name = "DispSPE4"; let PredicateMethod = "isU7ImmX4";
 let RenderMethod = "addImmOperands";
}
```
- **EN**: Adds declarative TableGen records such as `PPCDispSPE4Operand` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCDispSPE4Operand`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 400-406

```tablegen
def dispSPE4 : Operand<iPTR> {
  let ParserMatchClass = PPCDispSPE4Operand;
  let DecoderMethod = "decodeDispSPE4Operand";
  let EncoderMethod = "getDispSPE4Encoding";
}
def PPCDispSPE2Operand : AsmOperandClass {
 let Name = "DispSPE2"; let PredicateMethod = "isU6ImmX2";
```
- **EN**: Adds declarative TableGen records such as `dispSPE4`, `PPCDispSPE2Operand` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `dispSPE4`, `PPCDispSPE2Operand`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 407-414

```tablegen
 let RenderMethod = "addImmOperands";
}
def dispSPE2 : Operand<iPTR> {
  let ParserMatchClass = PPCDispSPE2Operand;
  let DecoderMethod = "decodeDispSPE2Operand";
  let EncoderMethod = "getDispSPE2Encoding";
}
```
- **EN**: Adds declarative TableGen records such as `dispSPE2` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `dispSPE2`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 415-416

```tablegen
//===----------------------------------------------------------------------===//
// TLS operands
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "TLS operands".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“TLS operands”。

### Lines 417-423

```tablegen
//===----------------------------------------------------------------------===//

def PPCTLSRegOperand : AsmOperandClass {
  let Name = "TLSReg"; let PredicateMethod = "isTLSReg";
  let RenderMethod = "addTLSRegOperands";
}
def tlsreg32 : Operand<i32> {
```
- **EN**: Adds declarative TableGen records such as `PPCTLSRegOperand`, `tlsreg32` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCTLSRegOperand`, `tlsreg32`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 424-430

```tablegen
  let EncoderMethod = "getTLSRegEncoding";
  let ParserMatchClass = PPCTLSRegOperand;
}
def tlsgd32 : Operand<i32> {}
def tlscall32 : Operand<i32> {
  let PrintMethod = "printTLSCall";
  let MIOperandInfo = (ops calltarget:$func, tlsgd32:$sym);
```
- **EN**: Adds declarative TableGen records such as `tlsgd32`, `tlscall32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `tlsgd32`, `tlscall32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 431-432

```tablegen
  let EncoderMethod = "getTLSCallEncoding";
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
