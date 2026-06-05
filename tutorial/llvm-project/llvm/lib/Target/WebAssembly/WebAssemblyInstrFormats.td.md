# WebAssemblyInstrFormats.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyInstrFormats.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: WebAssembly instruction format definitions. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyInstrFormats.td`，主要负责 WebAssembly 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//=- WebAssemblyInstrFormats.td - WebAssembly Instr. Formats -*- tablegen -*-=//
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
/// WebAssembly instruction format definitions.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-23

```tablegen
//===----------------------------------------------------------------------===//

// WebAssembly Instruction Format.
// We instantiate 2 of these for every actual instruction (register based
// and stack based), see below.
class WebAssemblyInst<bits<32> inst, string asmstr, bit stack, bit is64>
  : StackRel, RegisterRel, Wasm64Rel, Instruction {
  bits<32> Inst = inst; // Instruction encoding.
  bit StackBased = stack;
  string BaseName = NAME;
  bit IsWasm64 = is64;
  string Wasm32Name = !subst("_A64", "_A32", NAME);
```
- **EN**: Declares a backend-facing type `WebAssemblyInst` and outlines the API or state that nearby code will rely on. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里声明面向后端的类型 `WebAssemblyInst`，并勾勒出周边代码会依赖的接口或状态。 该区间与栈帧布局或栈访问相关。

### Lines 24-32

```tablegen
  let Namespace   = "WebAssembly";
  let Pattern     = [];
  let AsmString   = asmstr;
  // When there are multiple instructions that map to the same encoding (in
  // e.g. the disassembler use case) prefer the one where IsCanonical == 1.
  bit IsCanonical = 0;
}

// Normal instructions. Default instantiation of a WebAssemblyInst.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "When there are multiple instructions that map to the same encoding (in". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“When there are multiple instructions that map to the same encoding (in”。 这些声明会进入生成式模式匹配逻辑。

### Lines 33-54

```tablegen
class NI<dag oops, dag iops, list<dag> pattern, bit stack,
         string asmstr = "", bits<32> inst = -1, bit is64 = false>
    : WebAssemblyInst<inst, asmstr, stack, is64> {
  dag OutOperandList = oops;
  dag InOperandList  = iops;
  let Pattern        = pattern;
  let Defs           = [ARGUMENTS];
}

// Generates both register and stack based versions of one actual instruction.
// We have 2 sets of operands (oops & iops) for the register and stack
// based version of this instruction, as well as the corresponding asmstr.
// The register versions have virtual-register operands which correspond to wasm
// locals or stack locations. Each use and def of the register corresponds to an
// implicit local.get / local.set or access of stack operands in wasm. These
// instructions are used for ISel and all MI passes. The stack versions of the
// instructions do not have register operands (they implicitly operate on the
// stack), and local.gets and local.sets are explicit. The register instructions
// are converted to their corresponding stack instructions before lowering to
// MC.
// Every instruction should want to be based on this multi-class to guarantee
// there is always an equivalent pair of instructions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Generates both register and stack based versions of one actual instruction.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Generates both register and stack based versions of one actual instruction.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 55-64

```tablegen
multiclass I<dag oops_r, dag iops_r, dag oops_s, dag iops_s,
             list<dag> pattern_r, string asmstr_r = "", string asmstr_s = "",
             bits<32> inst = -1, bit is64 = false> {
  let isCodeGenOnly = 1 in
  def "" : NI<oops_r, iops_r, pattern_r, false, asmstr_r, inst, is64>;
  let BaseName = NAME in
  def _S : NI<oops_s, iops_s, [], true, asmstr_s, inst, is64>;
}

// For instructions that have no register ops, so both sets are the same.
```
- **EN**: Adds declarative TableGen records such as `I` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 65-68

```tablegen
multiclass NRI<dag oops, dag iops, list<dag> pattern, string asmstr = "",
               bits<32> inst = -1> {
  defm "": I<oops, iops, oops, iops, pattern, asmstr, asmstr, inst>;
}
```
- **EN**: Adds declarative TableGen records such as `NRI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NRI`，LLVM 随后会把它们展开成生成表或辅助代码。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Stack frame management / 栈帧管理
- Object format integration / 目标文件格式集成
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- Generated backend tables / 生成式后端表
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
