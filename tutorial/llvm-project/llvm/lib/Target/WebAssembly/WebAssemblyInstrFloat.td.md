# WebAssemblyInstrFloat.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyInstrFloat.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: WebAssembly Floating-point operand code-gen constructs. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyInstrFloat.td`，主要负责 WebAssembly 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
// WebAssemblyInstrFloat.td-WebAssembly Float codegen support ---*- tablegen -*-
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
///
/// \file
/// WebAssembly Floating-point operand code-gen constructs.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-19

```tablegen
//===----------------------------------------------------------------------===//

multiclass UnaryFP<SDNode node, string name, bits<32> f32Inst,
                   bits<32> f64Inst> {
  defm _F32 : I<(outs F32:$dst), (ins F32:$src), (outs), (ins),
                [(set F32:$dst, (node F32:$src))],
                !strconcat("f32.", !strconcat(name, "\t$dst, $src")),
                !strconcat("f32.", name), f32Inst>;
```
- **EN**: Adds declarative TableGen records such as `UnaryFP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `UnaryFP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 20-26

```tablegen
  defm _F64 : I<(outs F64:$dst), (ins F64:$src), (outs), (ins),
                [(set F64:$dst, (node F64:$src))],
                !strconcat("f64.", !strconcat(name, "\t$dst, $src")),
                !strconcat("f64.", name), f64Inst>;
}
multiclass BinaryFP<SDNode node, string name, bits<32> f32Inst,
                    bits<32> f64Inst> {
```
- **EN**: Adds declarative TableGen records such as `BinaryFP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BinaryFP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 27-35

```tablegen
  defm _F32 : I<(outs F32:$dst), (ins F32:$lhs, F32:$rhs), (outs), (ins),
                [(set F32:$dst, (node F32:$lhs, F32:$rhs))],
                !strconcat("f32.", !strconcat(name, "\t$dst, $lhs, $rhs")),
                !strconcat("f32.", name), f32Inst>;
  defm _F64 : I<(outs F64:$dst), (ins F64:$lhs, F64:$rhs), (outs), (ins),
                [(set F64:$dst, (node F64:$lhs, F64:$rhs))],
                !strconcat("f64.", !strconcat(name, "\t$dst, $lhs, $rhs")),
                !strconcat("f64.", name), f64Inst>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 36-46

```tablegen
multiclass ComparisonFP<CondCode cond, string name, bits<32> f32Inst, bits<32> f64Inst> {
  defm _F32 : I<(outs I32:$dst), (ins F32:$lhs, F32:$rhs), (outs), (ins),
                [(set I32:$dst, (setcc F32:$lhs, F32:$rhs, cond))],
                !strconcat("f32.", !strconcat(name, "\t$dst, $lhs, $rhs")),
                !strconcat("f32.", name), f32Inst>;
  defm  _F64 : I<(outs I32:$dst), (ins F64:$lhs, F64:$rhs), (outs), (ins),
                [(set I32:$dst, (setcc F64:$lhs, F64:$rhs, cond))],
                !strconcat("f64.", !strconcat(name, "\t$dst, $lhs, $rhs")),
                !strconcat("f64.", name), f64Inst>;
}
```
- **EN**: Adds declarative TableGen records such as `ComparisonFP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ComparisonFP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 47-54

```tablegen
let isCommutable = 1 in
defm ADD : BinaryFP<fadd, "add ", 0x92, 0xa0>;
defm SUB : BinaryFP<fsub, "sub ", 0x93, 0xa1>;
let isCommutable = 1 in
defm MUL : BinaryFP<fmul, "mul ", 0x94, 0xa2>;
defm DIV : BinaryFP<fdiv, "div ", 0x95, 0xa3>;
defm SQRT : UnaryFP<fsqrt, "sqrt", 0x91, 0x9f>;
```
- **EN**: Adds declarative TableGen records such as `ADD`, `SUB`, `MUL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADD`, `SUB`, `MUL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 55-63

```tablegen
defm ABS : UnaryFP<fabs, "abs ", 0x8b, 0x99>;
defm NEG : UnaryFP<fneg, "neg ", 0x8c, 0x9a>;
defm COPYSIGN : BinaryFP<fcopysign, "copysign", 0x98, 0xa6>;

let isCommutable = 1 in {
defm MIN : BinaryFP<fminimum, "min ", 0x96, 0xa4>;
defm MAX : BinaryFP<fmaximum, "max ", 0x97, 0xa5>;
} // isCommutable = 1
```
- **EN**: Adds declarative TableGen records such as `ABS`, `NEG`, `COPYSIGN` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ABS`, `NEG`, `COPYSIGN`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 64-71

```tablegen
defm CEIL : UnaryFP<fceil, "ceil", 0x8d, 0x9b>;
defm FLOOR : UnaryFP<ffloor, "floor", 0x8e, 0x9c>;
defm TRUNC : UnaryFP<ftrunc, "trunc", 0x8f, 0x9d>;
defm NEAREST : UnaryFP<fnearbyint, "nearest", 0x90, 0x9e>;

// DAGCombine oddly folds casts into the rhs of copysign. Unfold them.
def : Pat<(fcopysign F64:$lhs, F32:$rhs),
          (COPYSIGN_F64 F64:$lhs, (F64_PROMOTE_F32 F32:$rhs))>;
```
- **EN**: Adds declarative TableGen records such as `CEIL`, `FLOOR`, `TRUNC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CEIL`, `FLOOR`, `TRUNC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 72-79

```tablegen
def : Pat<(fcopysign F32:$lhs, F64:$rhs),
          (COPYSIGN_F32 F32:$lhs, (F32_DEMOTE_F64 F64:$rhs))>;

// WebAssembly doesn't expose inexact exceptions, so map frint to fnearbyint.
def : Pat<(frint f32:$src), (NEAREST_F32 f32:$src)>;
def : Pat<(frint f64:$src), (NEAREST_F64 f64:$src)>;

// WebAssembly always rounds ties-to-even, so map froundeven to fnearbyint.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssembly doesn't expose inexact exceptions, so map frint to fnearbyint.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssembly doesn't expose inexact exceptions, so map frint to fnearbyint.”。

### Lines 80-86

```tablegen
def : Pat<(froundeven f32:$src), (NEAREST_F32 f32:$src)>;
def : Pat<(froundeven f64:$src), (NEAREST_F64 f64:$src)>;

let isCommutable = 1 in {
defm EQ : ComparisonFP<SETOEQ, "eq  ", 0x5b, 0x61>;
defm NE : ComparisonFP<SETUNE, "ne  ", 0x5c, 0x62>;
} // isCommutable = 1
```
- **EN**: Adds declarative TableGen records such as `EQ`, `NE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EQ`, `NE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 87-93

```tablegen
defm LT : ComparisonFP<SETOLT, "lt  ", 0x5d, 0x63>;
defm LE : ComparisonFP<SETOLE, "le  ", 0x5f, 0x65>;
defm GT : ComparisonFP<SETOGT, "gt  ", 0x5e, 0x64>;
defm GE : ComparisonFP<SETOGE, "ge  ", 0x60, 0x66>;

// Don't care floating-point comparisons, supported via other comparisons.
def : Pat<(seteq f32:$lhs, f32:$rhs), (EQ_F32 f32:$lhs, f32:$rhs)>;
```
- **EN**: Adds declarative TableGen records such as `LT`, `LE`, `GT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LT`, `LE`, `GT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 94-100

```tablegen
def : Pat<(setne f32:$lhs, f32:$rhs), (NE_F32 f32:$lhs, f32:$rhs)>;
def : Pat<(setlt f32:$lhs, f32:$rhs), (LT_F32 f32:$lhs, f32:$rhs)>;
def : Pat<(setle f32:$lhs, f32:$rhs), (LE_F32 f32:$lhs, f32:$rhs)>;
def : Pat<(setgt f32:$lhs, f32:$rhs), (GT_F32 f32:$lhs, f32:$rhs)>;
def : Pat<(setge f32:$lhs, f32:$rhs), (GE_F32 f32:$lhs, f32:$rhs)>;
def : Pat<(seteq f64:$lhs, f64:$rhs), (EQ_F64 f64:$lhs, f64:$rhs)>;
def : Pat<(setne f64:$lhs, f64:$rhs), (NE_F64 f64:$lhs, f64:$rhs)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 101-109

```tablegen
def : Pat<(setlt f64:$lhs, f64:$rhs), (LT_F64 f64:$lhs, f64:$rhs)>;
def : Pat<(setle f64:$lhs, f64:$rhs), (LE_F64 f64:$lhs, f64:$rhs)>;
def : Pat<(setgt f64:$lhs, f64:$rhs), (GT_F64 f64:$lhs, f64:$rhs)>;
def : Pat<(setge f64:$lhs, f64:$rhs), (GE_F64 f64:$lhs, f64:$rhs)>;

defm SELECT_F32 : I<(outs F32:$dst), (ins F32:$lhs, F32:$rhs, I32:$cond),
                    (outs), (ins),
                    [(set F32:$dst, (select I32:$cond, F32:$lhs, F32:$rhs))],
                    "f32.select\t$dst, $lhs, $rhs, $cond", "f32.select", 0x1b>;
```
- **EN**: Adds declarative TableGen records such as `SELECT_F32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SELECT_F32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 110-117

```tablegen
defm SELECT_F64 : I<(outs F64:$dst), (ins F64:$lhs, F64:$rhs, I32:$cond),
                    (outs), (ins),
                    [(set F64:$dst, (select I32:$cond, F64:$lhs, F64:$rhs))],
                    "f64.select\t$dst, $lhs, $rhs, $cond", "f64.select", 0x1b>;

// ISD::SELECT requires its operand to conform to getBooleanContents, but
// WebAssembly's select interprets any non-zero value as true, so we can fold
// a setne with 0 into a select.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "ISD::SELECT requires its operand to conform to getBooleanContents, but". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“ISD::SELECT requires its operand to conform to getBooleanContents, but”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 118-125

```tablegen
def : Pat<(select (i32 (setne I32:$cond, 0)), F32:$lhs, F32:$rhs),
          (SELECT_F32 F32:$lhs, F32:$rhs, I32:$cond)>;
def : Pat<(select (i32 (setne I32:$cond, 0)), F64:$lhs, F64:$rhs),
          (SELECT_F64 F64:$lhs, F64:$rhs, I32:$cond)>;

// And again, this time with seteq instead of setne and the arms reversed.
def : Pat<(select (i32 (seteq I32:$cond, 0)), F32:$lhs, F32:$rhs),
          (SELECT_F32 F32:$rhs, F32:$lhs, I32:$cond)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 126-127

```tablegen
def : Pat<(select (i32 (seteq I32:$cond, 0)), F64:$lhs, F64:$rhs),
          (SELECT_F64 F64:$rhs, F64:$lhs, I32:$cond)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- Generated backend tables / 生成式后端表
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
