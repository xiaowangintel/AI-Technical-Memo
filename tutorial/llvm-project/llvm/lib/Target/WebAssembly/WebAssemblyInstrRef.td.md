# WebAssemblyInstrRef.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyInstrRef.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: WebAssembly reference type operand codegen constructs. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyInstrRef.td`，主要负责 WebAssembly 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
// WebAssemblyInstrRef.td - WebAssembly reference type codegen --*- tablegen -*-
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
/// WebAssembly reference type operand codegen constructs.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-23

```tablegen
//===----------------------------------------------------------------------===//

multiclass REF_I<WebAssemblyRegClass rc, ValueType vt, string ht> {
  defm REF_NULL_#rc : I<(outs rc:$dst), (ins),
                        (outs), (ins),
                        [(set rc:$dst, (!cast<Intrinsic>("int_wasm_ref_null_" # ht)))],
                        "ref.null_" # ht # "$dst",
                        "ref.null_" # ht,
                        !cond(!eq(ht, "func")   : 0xd070,
                              !eq(ht, "extern") : 0xd06f,
                              !eq(ht, "exn")    : 0xd069)>,
                      Requires<[HasReferenceTypes]>;
```
- **EN**: Adds declarative TableGen records such as `REF_I` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `REF_I`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 24-30

```tablegen
  defm SELECT_#rc: I<(outs rc:$dst), (ins rc:$lhs, rc:$rhs, I32:$cond),
                     (outs), (ins),
                     [(set rc:$dst,
                       (select I32:$cond, rc:$lhs, rc:$rhs))],
                     vt#".select\t$dst, $lhs, $rhs, $cond",
                     vt#".select", 0x1b>,
                   Requires<[HasReferenceTypes]>;
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 31-38

```tablegen
  defm REF_IS_NULL_#rc
      : I<(outs I32:$dst), (ins rc:$ref), (outs), (ins),
          [(set I32:$dst, (!cast<Intrinsic>("int_wasm_ref_is_null_" # ht) rc:$ref))],
          "ref.is_null\t$ref",
          "ref.is_null", 0xd1>,
        Requires<[HasReferenceTypes]>;
}
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 39-48

```tablegen
defm REF_TEST_FUNCREF : I<(outs I32:$res), (ins TypeIndex:$type, FUNCREF:$ref),
                          (outs), (ins TypeIndex:$type), [],
                          "ref.test\t$type, $ref", "ref.test $type", 0xfb14>,
                        Requires<[HasGC]>;

defm REF_CAST_FUNCREF : I<(outs FUNCREF:$res), (ins TypeIndex:$type, FUNCREF:$ref),
                          (outs), (ins TypeIndex:$type), [],
                          "ref.cast\t$type, $ref", "ref.cast $type", 0xfb16>,
                        Requires<[HasGC]>;
```
- **EN**: Adds declarative TableGen records such as `REF_TEST_FUNCREF`, `REF_CAST_FUNCREF` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `REF_TEST_FUNCREF`, `REF_CAST_FUNCREF`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 49-55

```tablegen
defm REF_FUNC : I<(outs FUNCREF:$res), (ins function32_op:$func),
                    (outs), (ins function32_op:$func), [],
                    "ref.func\t$func", "ref.func $func", 0xd2>,
                Requires<[HasReferenceTypes]>;

// A vector of value types used by the typed select instruction (select t*).
def TypeListAsmOperand : AsmOperandClass { let Name = "TypeList"; }
```
- **EN**: Adds declarative TableGen records such as `REF_FUNC`, `TypeListAsmOperand` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `REF_FUNC`, `TypeListAsmOperand`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 56-66

```tablegen
let OperandNamespace = "WebAssembly", OperandType = "OPERAND_VALTYPE_LIST" in
def typelist : Operand<i32> {
  let ParserMatchClass = TypeListAsmOperand;
  let PrintMethod = "printTypeList";
}

// Typed select: pops an i32 condition and two values of each declared type,
// then pushes the declared types back. ISel continues to use the numeric
// SELECT_<type> instructions at 0x1b; SELECT_T exists purely at the MC layer
// so the assembler and disassembler understand the typed form. It has no
// register operands distinct from its stack form, hence NRI.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Typed select: pops an i32 condition and two values of each declared type,".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Typed select: pops an i32 condition and two values of each declared type,”。

### Lines 67-74

```tablegen
defm SELECT_T : NRI<(outs), (ins typelist:$list), [],
                    "select\t$list", 0x1c>,
                Requires<[HasReferenceTypes]>;

defm "" : REF_I<FUNCREF, funcref, "func">;
defm "" : REF_I<EXTERNREF, externref, "extern">;
defm "" : REF_I<EXNREF, exnref, "exn">;
```
- **EN**: Adds declarative TableGen records such as `SELECT_T` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SELECT_T`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 75-80

```tablegen
foreach rc = [FUNCREF, EXTERNREF, EXNREF] in {
def : Pat<(select (i32 (setne I32:$cond, 0)), rc:$lhs, rc:$rhs),
          (!cast<Instruction>("SELECT_"#rc) rc:$lhs, rc:$rhs, I32:$cond)>;
def : Pat<(select (i32 (seteq I32:$cond, 0)), rc:$lhs, rc:$rhs),
          (!cast<Instruction>("SELECT_"#rc) rc:$rhs, rc:$lhs, I32:$cond)>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

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

- Generated backend tables / 生成式后端表
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
