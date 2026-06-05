# WebAssemblyInstrCall.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyInstrCall.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: WebAssembly Call operand code-gen constructs. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyInstrCall.td`，主要负责 WebAssembly 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===- WebAssemblyInstrCall.td-WebAssembly Call codegen support -*- tablegen -*-
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
/// WebAssembly Call operand code-gen constructs.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-18

```tablegen
//===----------------------------------------------------------------------===//

// TODO: addr64: These currently assume the callee address is 32-bit.
// FIXME: add $type to first call_indirect asmstr (and maybe $flags)

// Call sequence markers. These have an immediate which represents the amount of
// stack space to allocate or free, which is used for varargs lowering.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "TODO: addr64: These currently assume the callee address is 32-bit.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“TODO: addr64: These currently assume the callee address is 32-bit.”。

### Lines 19-26

```tablegen
let Uses = [SP32, SP64], Defs = [SP32, SP64], isCodeGenOnly = 1 in {
defm ADJCALLSTACKDOWN : NRI<(outs), (ins i32imm:$amt, i32imm:$amt2),
                            [(WebAssemblycallseq_start timm:$amt, timm:$amt2)]>;
defm ADJCALLSTACKUP : NRI<(outs), (ins i32imm:$amt, i32imm:$amt2),
                          [(WebAssemblycallseq_end timm:$amt, timm:$amt2)]>;
} // Uses = [SP32, SP64], Defs = [SP32, SP64], isCodeGenOnly = 1
```
- **EN**: Adds declarative TableGen records such as `ADJCALLSTACKDOWN`, `ADJCALLSTACKUP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADJCALLSTACKDOWN`, `ADJCALLSTACKUP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 27-34

```tablegen
let Uses = [SP32, SP64], isCall = 1 in {

// CALL should take both variadic arguments and produce variadic results, but
// this is not possible to model directly. Instead, we select calls to a
// CALL_PARAMS taking variadic arguments linked with a CALL_RESULTS that handles
// producing the call's variadic results. We recombine the two in a custom
// inserter hook after DAG ISel, so passes over MachineInstrs will only ever
// observe CALL nodes with all of the expected variadic uses and defs.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "CALL should take both variadic arguments and produce variadic results, but". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“CALL should take both variadic arguments and produce variadic results, but”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 35-41

```tablegen
let isPseudo = 1 in
defm CALL_PARAMS :
  I<(outs), (ins function32_op:$callee, variable_ops),
    (outs), (ins function32_op:$callee), [],
    "call_params\t$callee", "call_params\t$callee", -1>;

let variadicOpsAreDefs = 1, usesCustomInserter = 1, isPseudo = 1 in
```
- **EN**: Adds declarative TableGen records such as `CALL_PARAMS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CALL_PARAMS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 42-53

```tablegen
defm CALL_RESULTS :
  I<(outs), (ins variable_ops), (outs), (ins), [],
     "call_results", "call_results", -1>;

let variadicOpsAreDefs = 1, usesCustomInserter = 1, isPseudo = 1 in
defm RET_CALL_RESULTS :
  I<(outs), (ins variable_ops), (outs), (ins), [],
     "return_call_results", "return_call_results", -1>;

// Note that instructions with variable_ops have custom printers in
// WebAssemblyInstPrinter.cpp.
```
- **EN**: Adds declarative TableGen records such as `CALL_RESULTS`, `RET_CALL_RESULTS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CALL_RESULTS`, `RET_CALL_RESULTS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 54-60

```tablegen
let variadicOpsAreDefs = 1 in
defm CALL :
  I<(outs), (ins function32_op:$callee, variable_ops),
    (outs), (ins function32_op:$callee), [],
    "call", "call\t$callee", 0x10>;

let variadicOpsAreDefs = 1 in
```
- **EN**: Adds declarative TableGen records such as `CALL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CALL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 61-68

```tablegen
defm CALL_INDIRECT :
  I<(outs),
    (ins TypeIndex:$type, table32_op:$table, variable_ops),
    (outs),
    (ins TypeIndex:$type, table32_op:$table),
    [],
    "call_indirect\t$type, $table", "call_indirect\t$type, $table", 0x11>;
```
- **EN**: Adds declarative TableGen records such as `CALL_INDIRECT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CALL_INDIRECT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 69-75

```tablegen
let isReturn = 1, isTerminator = 1, hasCtrlDep = 1, isBarrier = 1 in
defm RET_CALL :
  I<(outs), (ins function32_op:$callee, variable_ops),
    (outs), (ins function32_op:$callee), [],
    "return_call    \t$callee", "return_call\t$callee", 0x12>,
  Requires<[HasTailCall]>;
```
- **EN**: Adds declarative TableGen records such as `RET_CALL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RET_CALL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 76-87

```tablegen
let isReturn = 1, isTerminator = 1, hasCtrlDep = 1, isBarrier = 1 in
defm RET_CALL_INDIRECT :
  I<(outs), (ins TypeIndex:$type, table32_op:$table, variable_ops),
    (outs), (ins TypeIndex:$type, table32_op:$table), [],
    "return_call_indirect\t$type, $table", "return_call_indirect\t$type, $table",
    0x13>,
  Requires<[HasTailCall]>;

// Typed function references: call_ref / return_call_ref pop a funcref from the
// stack and branch to it, using the given type index to describe the signature.
// These were introduced with the function-references proposal, which was folded
// into the GC proposal, so they are gated on HasGC.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Typed function references: call_ref / return_call_ref pop a funcref from the". Notable symbols in this range include `RET_CALL_INDIRECT`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Typed function references: call_ref / return_call_ref pop a funcref from the”。 该区间中较显眼的符号包括 `RET_CALL_INDIRECT`。

### Lines 88-94

```tablegen
let variadicOpsAreDefs = 1 in
defm CALL_REF :
  I<(outs), (ins TypeIndex:$type, FUNCREF:$ref, variable_ops),
    (outs), (ins TypeIndex:$type), [],
    "call_ref\t$type, $ref", "call_ref\t$type", 0x14>,
  Requires<[HasGC]>;
```
- **EN**: Adds declarative TableGen records such as `CALL_REF` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CALL_REF`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 95-102

```tablegen
let isReturn = 1, isTerminator = 1, hasCtrlDep = 1, isBarrier = 1 in
defm RET_CALL_REF :
  I<(outs), (ins TypeIndex:$type, FUNCREF:$ref, variable_ops),
    (outs), (ins TypeIndex:$type), [],
    "return_call_ref\t$type, $ref", "return_call_ref\t$type", 0x15>,
  Requires<[HasTailCall, HasGC]>;

} // Uses = [SP32,SP64], isCall = 1
```
- **EN**: Adds declarative TableGen records such as `RET_CALL_REF` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RET_CALL_REF`，LLVM 随后会把它们展开成生成表或辅助代码。

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

- LLVM CodeGen layer / LLVM CodeGen 层
- Generated backend tables / 生成式后端表
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
