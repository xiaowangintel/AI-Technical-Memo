# WebAssemblyInstrAtomics.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyInstrAtomics.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: WebAssembly Atomic operand code-gen constructs. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyInstrAtomics.td`，主要负责 WebAssembly 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
// WebAssemblyInstrAtomics.td-WebAssembly Atomic codegen support-*- tablegen -*-
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
/// WebAssembly Atomic operand code-gen constructs.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-20

```tablegen
//===----------------------------------------------------------------------===//

def AtomicAddrOps32 : ComplexPattern<i32, 3, "SelectAtomicAddrOperands32"> {
  let WantsRoot = 1;
}
def AtomicAddrOps64 : ComplexPattern<i64, 3, "SelectAtomicAddrOperands64"> {
  let WantsRoot = 1;
}
```
- **EN**: Adds declarative TableGen records such as `AtomicAddrOps32`, `AtomicAddrOps64` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `AtomicAddrOps32`, `AtomicAddrOps64`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 21-30

```tablegen
let UseNamedOperandTable = 1 in
multiclass ATOMIC_I<dag oops_r, dag iops_r, dag oops_s, dag iops_s,
                    list<dag> pattern_r, string asmstr_r,
                    string asmstr_s, bits<32> atomic_op,
                    bit is64 = false> {
  defm "" : I<oops_r, iops_r, oops_s, iops_s, pattern_r, asmstr_r, asmstr_s,
              !or(0xfe00, !and(0xff, atomic_op)), is64>,
            Requires<[HasAtomics]>;
}
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_I` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_I`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 31-37

```tablegen
multiclass ATOMIC_NRI<dag oops, dag iops, list<dag> pattern, string asmstr = "",
                      bits<32> atomic_op = -1> {
  defm "" : NRI<oops, iops, pattern, asmstr,
                !or(0xfe00, !and(0xff, atomic_op))>,
            Requires<[HasAtomics]>;
}
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_NRI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_NRI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 38-39

```tablegen
//===----------------------------------------------------------------------===//
// Atomic wait / notify
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Atomic wait / notify".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Atomic wait / notify”。

### Lines 40-50

```tablegen
//===----------------------------------------------------------------------===//

let hasSideEffects = 1 in {
defm MEMORY_ATOMIC_NOTIFY_A32 :
  ATOMIC_I<(outs I32:$dst),
           (ins MemOrder:$order, P2Align:$p2align, offset32_op:$off, I32:$addr,
                I32:$count),
           (outs), (ins MemOrder:$order, P2Align:$p2align, offset32_op:$off),
           [], "memory.atomic.notify \t$dst, "
               "${order} ${off}(${addr})${p2align}, $count",
           "memory.atomic.notify \t${order} ${off}${p2align}", 0x00, false>;
```
- **EN**: Adds declarative TableGen records such as `MEMORY_ATOMIC_NOTIFY_A32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MEMORY_ATOMIC_NOTIFY_A32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 51-58

```tablegen
defm MEMORY_ATOMIC_NOTIFY_A64 :
  ATOMIC_I<(outs I32:$dst),
           (ins MemOrder:$order, P2Align:$p2align, offset64_op:$off, I64:$addr,
                I32:$count),
           (outs), (ins MemOrder:$order, P2Align:$p2align, offset64_op:$off),
           [], "memory.atomic.notify \t$dst, "
               "${order} ${off}(${addr})${p2align}, $count",
           "memory.atomic.notify \t${order} ${off}${p2align}", 0x00, true>;
```
- **EN**: Adds declarative TableGen records such as `MEMORY_ATOMIC_NOTIFY_A64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MEMORY_ATOMIC_NOTIFY_A64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 59-67

```tablegen
let mayLoad = 1 in {
defm MEMORY_ATOMIC_WAIT32_A32 :
  ATOMIC_I<(outs I32:$dst),
           (ins MemOrder:$order, P2Align:$p2align, offset32_op:$off, I32:$addr,
                I32:$exp, I64:$timeout),
           (outs), (ins MemOrder:$order, P2Align:$p2align, offset32_op:$off),
           [], "memory.atomic.wait32 \t$dst, "
               "${order} ${off}(${addr})${p2align}, $exp, $timeout",
           "memory.atomic.wait32 \t${order} ${off}${p2align}", 0x01, false>;
```
- **EN**: Adds declarative TableGen records such as `MEMORY_ATOMIC_WAIT32_A32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MEMORY_ATOMIC_WAIT32_A32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 68-75

```tablegen
defm MEMORY_ATOMIC_WAIT32_A64 :
  ATOMIC_I<(outs I32:$dst),
           (ins MemOrder:$order, P2Align:$p2align, offset64_op:$off, I64:$addr,
                I32:$exp, I64:$timeout),
           (outs), (ins MemOrder:$order, P2Align:$p2align, offset64_op:$off),
           [], "memory.atomic.wait32 \t$dst, "
               "${order} ${off}(${addr})${p2align}, $exp, $timeout",
           "memory.atomic.wait32 \t${order} ${off}${p2align}", 0x01, true>;
```
- **EN**: Adds declarative TableGen records such as `MEMORY_ATOMIC_WAIT32_A64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MEMORY_ATOMIC_WAIT32_A64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 76-83

```tablegen
defm MEMORY_ATOMIC_WAIT64_A32 :
  ATOMIC_I<(outs I32:$dst),
           (ins MemOrder:$order, P2Align:$p2align, offset32_op:$off, I32:$addr,
                I64:$exp, I64:$timeout),
           (outs), (ins MemOrder:$order, P2Align:$p2align, offset32_op:$off),
           [], "memory.atomic.wait64 \t$dst, "
               "${order} ${off}(${addr})${p2align}, $exp, $timeout",
           "memory.atomic.wait64 \t${order} ${off}${p2align}", 0x02, false>;
```
- **EN**: Adds declarative TableGen records such as `MEMORY_ATOMIC_WAIT64_A32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MEMORY_ATOMIC_WAIT64_A32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 84-94

```tablegen
defm MEMORY_ATOMIC_WAIT64_A64 :
  ATOMIC_I<(outs I32:$dst),
           (ins MemOrder:$order, P2Align:$p2align, offset64_op:$off, I64:$addr,
                I64:$exp, I64:$timeout),
           (outs), (ins MemOrder:$order, P2Align:$p2align, offset64_op:$off),
           [], "memory.atomic.wait64 \t$dst, "
               "${order} ${off}(${addr})${p2align}, $exp, $timeout",
           "memory.atomic.wait64 \t${order} ${off}${p2align}", 0x02, true>;
} // mayLoad = 1
} // hasSideEffects = 1
```
- **EN**: Adds declarative TableGen records such as `MEMORY_ATOMIC_WAIT64_A64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MEMORY_ATOMIC_WAIT64_A64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 95-104

```tablegen
def NotifyPat_A32 :
  Pat<(i32 (int_wasm_memory_atomic_notify (AddrOps32 offset32_op:$offset, I32:$addr), I32:$count)),
      (MEMORY_ATOMIC_NOTIFY_A32 0, 0, $offset, $addr, $count)>,
  Requires<[HasAddr32, HasAtomics]>;
def NotifyPat_A64 :
  Pat<(i32 (int_wasm_memory_atomic_notify (AddrOps64 offset64_op:$offset, I64:$addr), I32:$count)),
      (MEMORY_ATOMIC_NOTIFY_A64 0, 0, $offset, $addr, $count)>,
  Requires<[HasAddr64, HasAtomics]>;
```
- **EN**: Adds declarative TableGen records such as `NotifyPat_A32`, `NotifyPat_A64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NotifyPat_A32`, `NotifyPat_A64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 105-115

```tablegen
multiclass WaitPat<ValueType ty, Intrinsic kind, string inst> {
  def WaitPat_A32 :
    Pat<(i32 (kind (AddrOps32 offset32_op:$offset, I32:$addr), ty:$exp, I64:$timeout)),
        (!cast<NI>(inst#_A32) 0, 0, $offset, $addr, $exp, $timeout)>,
    Requires<[HasAddr32, HasAtomics]>;
  def WaitPat_A64 :
    Pat<(i32 (kind (AddrOps64 offset64_op:$offset, I64:$addr), ty:$exp, I64:$timeout)),
        (!cast<NI>(inst#_A64) 0, 0, $offset, $addr, $exp, $timeout)>,
    Requires<[HasAddr64, HasAtomics]>;
}
```
- **EN**: Adds declarative TableGen records such as `WaitPat` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WaitPat`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 116-118

```tablegen
defm : WaitPat<i32, int_wasm_memory_atomic_wait32, "MEMORY_ATOMIC_WAIT32">;
defm : WaitPat<i64, int_wasm_memory_atomic_wait64, "MEMORY_ATOMIC_WAIT64">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 119-120

```tablegen
//===----------------------------------------------------------------------===//
// Atomic fences
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Atomic fences".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Atomic fences”。

### Lines 121-127

```tablegen
//===----------------------------------------------------------------------===//

// A compiler fence instruction that prevents reordering of instructions.
let Defs = [ARGUMENTS] in {
let isPseudo = 1, hasSideEffects = 1 in
defm COMPILER_FENCE : ATOMIC_NRI<(outs), (ins), [], "compiler_fence">;
let hasSideEffects = 1 in
```
- **EN**: Adds declarative TableGen records such as `COMPILER_FENCE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `COMPILER_FENCE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 128-131

```tablegen
defm ATOMIC_FENCE : ATOMIC_NRI<(outs), (ins MemOrder:$order), [],
                               "atomic.fence\t${order}", 0x03>;
} // Defs = [ARGUMENTS]
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_FENCE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_FENCE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 132-133

```tablegen
//===----------------------------------------------------------------------===//
// Atomic loads
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Atomic loads".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Atomic loads”。

### Lines 134-146

```tablegen
//===----------------------------------------------------------------------===//

multiclass AtomicLoad<WebAssemblyRegClass rc, string name, int atomic_op> {
  let mayLoad = 1, UseNamedOperandTable = 1 in {
  defm "_A32": ATOMIC_I<(outs rc:$dst),
                        (ins MemOrder:$order, P2Align:$p2align,
                             offset32_op:$off, I32:$addr),
                        (outs), (ins MemOrder:$order, P2Align:$p2align,
                                     offset32_op:$off),
                        [], !strconcat(name, "\t$dst, ${order} "
                                             "${off}(${addr})${p2align}"),
                        !strconcat(name, "\t${order} ${off}${p2align}"),
                        atomic_op, false>;
```
- **EN**: Adds declarative TableGen records such as `AtomicLoad` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `AtomicLoad`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 147-158

```tablegen
  defm "_A64": ATOMIC_I<(outs rc:$dst),
                        (ins MemOrder:$order, P2Align:$p2align,
                             offset64_op:$off, I64:$addr),
                        (outs), (ins MemOrder:$order, P2Align:$p2align,
                                     offset64_op:$off),
                        [], !strconcat(name, "\t$dst, ${order} "
                                             "${off}(${addr})${p2align}"),
                        !strconcat(name, "\t${order} ${off}${p2align}"),
                        atomic_op, true>;
  }
}
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend TableGen declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 159-167

```tablegen
multiclass AtomicLoadPat<ValueType ty, SDPatternOperator kind, string Name> {
  def : Pat<(ty (kind (AtomicAddrOps32 offset32_op:$offset, I32:$addr, i32imm:$order))),
            (!cast<NI>(Name # "_A32") $order, 0, offset32_op:$offset, I32:$addr)>,
        Requires<[HasAddr32, HasAtomics]>;
  def : Pat<(ty (kind (AtomicAddrOps64 offset64_op:$offset, I64:$addr, i32imm:$order))),
            (!cast<NI>(Name # "_A64") $order, 0, offset64_op:$offset, I64:$addr)>,
        Requires<[HasAddr64, HasAtomics]>;
}
```
- **EN**: Adds declarative TableGen records such as `AtomicLoadPat` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `AtomicLoadPat`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 168-176

```tablegen
defm ATOMIC_LOAD_I32 : AtomicLoad<I32, "i32.atomic.load", 0x10>;
defm ATOMIC_LOAD_I64 : AtomicLoad<I64, "i64.atomic.load", 0x11>;

// Select loads
defm : AtomicLoadPat<i32, atomic_load_nonext_32, "ATOMIC_LOAD_I32">;
defm : AtomicLoadPat<i64, atomic_load_nonext_64, "ATOMIC_LOAD_I64">;

// Extending loads. Note that there are only zero-extending atomic loads, no
// sign-extending loads.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Select loads". Notable symbols in this range include `ATOMIC_LOAD_I32`, `ATOMIC_LOAD_I64`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Select loads”。 该区间中较显眼的符号包括 `ATOMIC_LOAD_I32`, `ATOMIC_LOAD_I64`。

### Lines 177-192

```tablegen
defm ATOMIC_LOAD8_U_I32 : AtomicLoad<I32, "i32.atomic.load8_u", 0x12>;
defm ATOMIC_LOAD16_U_I32 : AtomicLoad<I32, "i32.atomic.load16_u", 0x13>;
defm ATOMIC_LOAD8_U_I64 : AtomicLoad<I64, "i64.atomic.load8_u", 0x14>;
defm ATOMIC_LOAD16_U_I64 : AtomicLoad<I64, "i64.atomic.load16_u", 0x15>;
defm ATOMIC_LOAD32_U_I64 : AtomicLoad<I64, "i64.atomic.load32_u", 0x16>;

// Fragments for extending loads. These are different from regular loads because
// the SDNodes are derived from AtomicSDNode rather than LoadSDNode and
// therefore don't have the extension type field. So instead of matching that,
// we match the patterns that the type legalizer expands them to.

// Unlike regular loads, extension to i64 is handled differently than i32.
// i64 (zext (i8 (atomic_load_nonext_8))) gets legalized to
// i64 (and (i64 (anyext (i32 (atomic_load_zext_8)))), 255)
// Extension to i32 is elided by SelectionDAG as our atomic loads are
// zero-extending.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Fragments for extending loads. These are different from regular loads because". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Fragments for extending loads. These are different from regular loads because”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 193-206

```tablegen
def zext_aload_8_64 :
  PatFrag<(ops node:$addr),
          (i64 (zext (i32 (atomic_load_azext_8 node:$addr))))>;
def zext_aload_16_64 :
  PatFrag<(ops node:$addr),
          (i64 (zext (i32 (atomic_load_azext_16 node:$addr))))>;
def zext_aload_32_64 :
  PatFrag<(ops node:$addr),
          (i64 (zext (i32 (atomic_load_nonext_32 node:$addr))))>;

// We don't have single sext atomic load instructions. So for sext loads, we
// match bare subword loads (for 32-bit results) and anyext loads (for 64-bit
// results) and select a zext load; the next instruction will be sext_inreg
// which is selected by itself.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "We don't have single sext atomic load instructions. So for sext loads, we". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“We don't have single sext atomic load instructions. So for sext loads, we”。 这些声明会进入生成式模式匹配逻辑。

### Lines 207-213

```tablegen
def sext_aload_8_64 :
  PatFrag<(ops node:$addr), (anyext (i32 (atomic_load_azext_8 node:$addr)))>;
def sext_aload_16_64 :
  PatFrag<(ops node:$addr), (anyext (i32 (atomic_load_azext_16 node:$addr)))>;

// Select zero-extending loads
defm : AtomicLoadPat<i64, zext_aload_8_64, "ATOMIC_LOAD8_U_I64">;
```
- **EN**: Adds declarative TableGen records such as `sext_aload_8_64`, `sext_aload_16_64` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `sext_aload_8_64`, `sext_aload_16_64`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 214-220

```tablegen
defm : AtomicLoadPat<i64, zext_aload_16_64, "ATOMIC_LOAD16_U_I64">;
defm : AtomicLoadPat<i64, zext_aload_32_64, "ATOMIC_LOAD32_U_I64">;

// Select sign-extending loads
defm : AtomicLoadPat<i32, atomic_load_zext_8, "ATOMIC_LOAD8_U_I32">;
defm : AtomicLoadPat<i32, atomic_load_zext_16, "ATOMIC_LOAD16_U_I32">;
defm : AtomicLoadPat<i64, sext_aload_8_64, "ATOMIC_LOAD8_U_I64">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 221-224

```tablegen
defm : AtomicLoadPat<i64, sext_aload_16_64, "ATOMIC_LOAD16_U_I64">;
// 32->64 sext load gets selected as i32.atomic.load, i64.extend_i32_s
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "32->64 sext load gets selected as i32.atomic.load, i64.extend_i32_s".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“32->64 sext load gets selected as i32.atomic.load, i64.extend_i32_s”。

### Lines 225-226

```tablegen
//===----------------------------------------------------------------------===//
// Atomic stores
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Atomic stores".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Atomic stores”。

### Lines 227-239

```tablegen
//===----------------------------------------------------------------------===//

multiclass AtomicStore<WebAssemblyRegClass rc, string name, int atomic_op> {
  let mayStore = 1, UseNamedOperandTable = 1 in {
  defm "_A32" : ATOMIC_I<(outs),
                         (ins MemOrder:$order, P2Align:$p2align,
                              offset32_op:$off, I32:$addr, rc:$val),
                         (outs), (ins MemOrder:$order, P2Align:$p2align,
                                      offset32_op:$off), [],
                         !strconcat(name, "\t${order} ${off}"
                                          "(${addr})${p2align}, $val"),
                         !strconcat(name, "\t${order} ${off}${p2align}"),
                         atomic_op, false>;
```
- **EN**: Adds declarative TableGen records such as `AtomicStore` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `AtomicStore`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 240-251

```tablegen
  defm "_A64" : ATOMIC_I<(outs),
                         (ins MemOrder:$order, P2Align:$p2align,
                              offset64_op:$off, I64:$addr, rc:$val),
                         (outs), (ins MemOrder:$order, P2Align:$p2align,
                                      offset64_op:$off), [],
                         !strconcat(name, "\t${order} ${off}"
                                          "(${addr})${p2align}, $val"),
                         !strconcat(name, "\t${order} ${off}${p2align}"),
                         atomic_op, true>;
  }
}
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend TableGen declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 252-258

```tablegen
defm ATOMIC_STORE_I32 : AtomicStore<I32, "i32.atomic.store", 0x17>;
defm ATOMIC_STORE_I64 : AtomicStore<I64, "i64.atomic.store", 0x18>;

multiclass AStorePat<ValueType ty, PatFrag kind, string inst> {
  def : Pat<(kind ty:$val, (AtomicAddrOps32 offset32_op:$offset, I32:$addr, i32imm:$order)),
            (!cast<NI>(inst#_A32) $order, 0, $offset, $addr, $val)>,
        Requires<[HasAddr32, HasAtomics]>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_STORE_I32`, `ATOMIC_STORE_I64`, `AStorePat` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_STORE_I32`, `ATOMIC_STORE_I64`, `AStorePat`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 259-266

```tablegen
  def : Pat<(kind ty:$val, (AtomicAddrOps64 offset64_op:$offset, I64:$addr, i32imm:$order)),
            (!cast<NI>(inst#_A64) $order, 0, $offset, $addr, $val)>,
        Requires<[HasAddr64, HasAtomics]>;
}
defm : AStorePat<i32, atomic_store_32, "ATOMIC_STORE_I32">;
defm : AStorePat<i64, atomic_store_64, "ATOMIC_STORE_I64">;

// Truncating stores.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 267-277

```tablegen
defm ATOMIC_STORE8_I32 : AtomicStore<I32, "i32.atomic.store8", 0x19>;
defm ATOMIC_STORE16_I32 : AtomicStore<I32, "i32.atomic.store16", 0x1a>;
defm ATOMIC_STORE8_I64 : AtomicStore<I64, "i64.atomic.store8", 0x1b>;
defm ATOMIC_STORE16_I64 : AtomicStore<I64, "i64.atomic.store16", 0x1c>;
defm ATOMIC_STORE32_I64 : AtomicStore<I64, "i64.atomic.store32", 0x1d>;

// Fragments for truncating stores.

// We don't have single truncating atomic store instructions. For 32-bit
// instructions, we just need to match bare atomic stores. On the other hand,
// truncating stores from i64 values are once truncated to i32 first.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Fragments for truncating stores.". Notable symbols in this range include `ATOMIC_STORE8_I32`, `ATOMIC_STORE16_I32`, `ATOMIC_STORE8_I64`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Fragments for truncating stores.”。 该区间中较显眼的符号包括 `ATOMIC_STORE8_I32`, `ATOMIC_STORE16_I32`, `ATOMIC_STORE8_I64`。

### Lines 278-285

```tablegen
class trunc_astore_64<PatFrag kind> :
  PatFrag<(ops node:$val, node:$addr),
          (kind (i32 (trunc (i64 node:$val))), node:$addr)>;
def trunc_astore_8_64 : trunc_astore_64<atomic_store_8>;
def trunc_astore_16_64 : trunc_astore_64<atomic_store_16>;
def trunc_astore_32_64 : trunc_astore_64<atomic_store_32>;

// Truncating stores with no constant offset
```
- **EN**: Declares a backend-facing type `trunc_astore_64`, `trunc_astore_8_64`, `trunc_astore_16_64` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `trunc_astore_64`, `trunc_astore_8_64`, `trunc_astore_16_64`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 286-291

```tablegen
defm : AStorePat<i32, atomic_store_8, "ATOMIC_STORE8_I32">;
defm : AStorePat<i32, atomic_store_16, "ATOMIC_STORE16_I32">;
defm : AStorePat<i64, trunc_astore_8_64, "ATOMIC_STORE8_I64">;
defm : AStorePat<i64, trunc_astore_16_64, "ATOMIC_STORE16_I64">;
defm : AStorePat<i64, trunc_astore_32_64, "ATOMIC_STORE32_I64">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 292-293

```tablegen
//===----------------------------------------------------------------------===//
// Atomic binary read-modify-writes
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Atomic binary read-modify-writes".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Atomic binary read-modify-writes”。

### Lines 294-305

```tablegen
//===----------------------------------------------------------------------===//

multiclass WebAssemblyBinRMW<WebAssemblyRegClass rc, string name,
                             int atomic_op> {
  defm "_A32" :
    ATOMIC_I<(outs rc:$dst),
             (ins MemOrder:$order, P2Align:$p2align, offset32_op:$off,
                  I32:$addr, rc:$val),
             (outs), (ins MemOrder:$order, P2Align:$p2align, offset32_op:$off),
             [], !strconcat(name, "\t$dst, ${order} "
                                  "${off}(${addr})${p2align}, $val"),
             !strconcat(name, "\t${order} ${off}${p2align}"), atomic_op, false>;
```
- **EN**: Adds declarative TableGen records such as `WebAssemblyBinRMW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WebAssemblyBinRMW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 306-315

```tablegen
  defm "_A64" :
    ATOMIC_I<(outs rc:$dst),
             (ins MemOrder:$order, P2Align:$p2align, offset64_op:$off,
                  I64:$addr, rc:$val),
             (outs), (ins MemOrder:$order, P2Align:$p2align, offset64_op:$off),
             [], !strconcat(name, "\t$dst, ${order} "
                                  "${off}(${addr})${p2align}, $val"),
             !strconcat(name, "\t${order} ${off}${p2align}"), atomic_op, true>;
}
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend TableGen declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 316-323

```tablegen
defm ATOMIC_RMW_ADD_I32 : WebAssemblyBinRMW<I32, "i32.atomic.rmw.add", 0x1e>;
defm ATOMIC_RMW_ADD_I64 : WebAssemblyBinRMW<I64, "i64.atomic.rmw.add", 0x1f>;
defm ATOMIC_RMW8_U_ADD_I32 :
  WebAssemblyBinRMW<I32, "i32.atomic.rmw8.add_u", 0x20>;
defm ATOMIC_RMW16_U_ADD_I32 :
  WebAssemblyBinRMW<I32, "i32.atomic.rmw16.add_u", 0x21>;
defm ATOMIC_RMW8_U_ADD_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw8.add_u", 0x22>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_RMW_ADD_I32`, `ATOMIC_RMW_ADD_I64`, `ATOMIC_RMW8_U_ADD_I32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_RMW_ADD_I32`, `ATOMIC_RMW_ADD_I64`, `ATOMIC_RMW8_U_ADD_I32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 324-330

```tablegen
defm ATOMIC_RMW16_U_ADD_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw16.add_u", 0x23>;
defm ATOMIC_RMW32_U_ADD_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw32.add_u", 0x24>;

defm ATOMIC_RMW_SUB_I32 : WebAssemblyBinRMW<I32, "i32.atomic.rmw.sub", 0x25>;
defm ATOMIC_RMW_SUB_I64 : WebAssemblyBinRMW<I64, "i64.atomic.rmw.sub", 0x26>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_RMW16_U_ADD_I64`, `ATOMIC_RMW32_U_ADD_I64`, `ATOMIC_RMW_SUB_I32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_RMW16_U_ADD_I64`, `ATOMIC_RMW32_U_ADD_I64`, `ATOMIC_RMW_SUB_I32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 331-338

```tablegen
defm ATOMIC_RMW8_U_SUB_I32 :
  WebAssemblyBinRMW<I32, "i32.atomic.rmw8.sub_u", 0x27>;
defm ATOMIC_RMW16_U_SUB_I32 :
  WebAssemblyBinRMW<I32, "i32.atomic.rmw16.sub_u", 0x28>;
defm ATOMIC_RMW8_U_SUB_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw8.sub_u", 0x29>;
defm ATOMIC_RMW16_U_SUB_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw16.sub_u", 0x2a>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_RMW8_U_SUB_I32`, `ATOMIC_RMW16_U_SUB_I32`, `ATOMIC_RMW8_U_SUB_I64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_RMW8_U_SUB_I32`, `ATOMIC_RMW16_U_SUB_I32`, `ATOMIC_RMW8_U_SUB_I64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 339-345

```tablegen
defm ATOMIC_RMW32_U_SUB_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw32.sub_u", 0x2b>;

defm ATOMIC_RMW_AND_I32 : WebAssemblyBinRMW<I32, "i32.atomic.rmw.and", 0x2c>;
defm ATOMIC_RMW_AND_I64 : WebAssemblyBinRMW<I64, "i64.atomic.rmw.and", 0x2d>;
defm ATOMIC_RMW8_U_AND_I32 :
  WebAssemblyBinRMW<I32, "i32.atomic.rmw8.and_u", 0x2e>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_RMW32_U_SUB_I64`, `ATOMIC_RMW_AND_I32`, `ATOMIC_RMW_AND_I64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_RMW32_U_SUB_I64`, `ATOMIC_RMW_AND_I32`, `ATOMIC_RMW_AND_I64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 346-354

```tablegen
defm ATOMIC_RMW16_U_AND_I32 :
  WebAssemblyBinRMW<I32, "i32.atomic.rmw16.and_u", 0x2f>;
defm ATOMIC_RMW8_U_AND_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw8.and_u", 0x30>;
defm ATOMIC_RMW16_U_AND_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw16.and_u", 0x31>;
defm ATOMIC_RMW32_U_AND_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw32.and_u", 0x32>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_RMW16_U_AND_I32`, `ATOMIC_RMW8_U_AND_I64`, `ATOMIC_RMW16_U_AND_I64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_RMW16_U_AND_I32`, `ATOMIC_RMW8_U_AND_I64`, `ATOMIC_RMW16_U_AND_I64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 355-362

```tablegen
defm ATOMIC_RMW_OR_I32 : WebAssemblyBinRMW<I32, "i32.atomic.rmw.or", 0x33>;
defm ATOMIC_RMW_OR_I64 : WebAssemblyBinRMW<I64, "i64.atomic.rmw.or", 0x34>;
defm ATOMIC_RMW8_U_OR_I32 :
  WebAssemblyBinRMW<I32, "i32.atomic.rmw8.or_u", 0x35>;
defm ATOMIC_RMW16_U_OR_I32 :
  WebAssemblyBinRMW<I32, "i32.atomic.rmw16.or_u", 0x36>;
defm ATOMIC_RMW8_U_OR_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw8.or_u", 0x37>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_RMW_OR_I32`, `ATOMIC_RMW_OR_I64`, `ATOMIC_RMW8_U_OR_I32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_RMW_OR_I32`, `ATOMIC_RMW_OR_I64`, `ATOMIC_RMW8_U_OR_I32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 363-369

```tablegen
defm ATOMIC_RMW16_U_OR_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw16.or_u", 0x38>;
defm ATOMIC_RMW32_U_OR_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw32.or_u", 0x39>;

defm ATOMIC_RMW_XOR_I32 : WebAssemblyBinRMW<I32, "i32.atomic.rmw.xor", 0x3a>;
defm ATOMIC_RMW_XOR_I64 : WebAssemblyBinRMW<I64, "i64.atomic.rmw.xor", 0x3b>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_RMW16_U_OR_I64`, `ATOMIC_RMW32_U_OR_I64`, `ATOMIC_RMW_XOR_I32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_RMW16_U_OR_I64`, `ATOMIC_RMW32_U_OR_I64`, `ATOMIC_RMW_XOR_I32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 370-377

```tablegen
defm ATOMIC_RMW8_U_XOR_I32 :
  WebAssemblyBinRMW<I32, "i32.atomic.rmw8.xor_u", 0x3c>;
defm ATOMIC_RMW16_U_XOR_I32 :
  WebAssemblyBinRMW<I32, "i32.atomic.rmw16.xor_u", 0x3d>;
defm ATOMIC_RMW8_U_XOR_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw8.xor_u", 0x3e>;
defm ATOMIC_RMW16_U_XOR_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw16.xor_u", 0x3f>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_RMW8_U_XOR_I32`, `ATOMIC_RMW16_U_XOR_I32`, `ATOMIC_RMW8_U_XOR_I64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_RMW8_U_XOR_I32`, `ATOMIC_RMW16_U_XOR_I32`, `ATOMIC_RMW8_U_XOR_I64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 378-384

```tablegen
defm ATOMIC_RMW32_U_XOR_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw32.xor_u", 0x40>;

defm ATOMIC_RMW_XCHG_I32 :
  WebAssemblyBinRMW<I32, "i32.atomic.rmw.xchg", 0x41>;
defm ATOMIC_RMW_XCHG_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw.xchg", 0x42>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_RMW32_U_XOR_I64`, `ATOMIC_RMW_XCHG_I32`, `ATOMIC_RMW_XCHG_I64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_RMW32_U_XOR_I64`, `ATOMIC_RMW_XCHG_I32`, `ATOMIC_RMW_XCHG_I64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 385-392

```tablegen
defm ATOMIC_RMW8_U_XCHG_I32 :
  WebAssemblyBinRMW<I32, "i32.atomic.rmw8.xchg_u", 0x43>;
defm ATOMIC_RMW16_U_XCHG_I32 :
  WebAssemblyBinRMW<I32, "i32.atomic.rmw16.xchg_u", 0x44>;
defm ATOMIC_RMW8_U_XCHG_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw8.xchg_u", 0x45>;
defm ATOMIC_RMW16_U_XCHG_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw16.xchg_u", 0x46>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_RMW8_U_XCHG_I32`, `ATOMIC_RMW16_U_XCHG_I32`, `ATOMIC_RMW8_U_XCHG_I64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_RMW8_U_XCHG_I32`, `ATOMIC_RMW16_U_XCHG_I32`, `ATOMIC_RMW8_U_XCHG_I64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 393-400

```tablegen
defm ATOMIC_RMW32_U_XCHG_I64 :
  WebAssemblyBinRMW<I64, "i64.atomic.rmw32.xchg_u", 0x47>;

multiclass BinRMWPat<ValueType ty, PatFrag kind, string inst> {
  def : Pat<(ty (kind (AtomicAddrOps32 offset32_op:$offset, I32:$addr,
                       i32imm:$order), ty:$val)),
            (!cast<NI>(inst#_A32) $order, 0, $offset, $addr, $val)>,
        Requires<[HasAddr32, HasAtomics]>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_RMW32_U_XCHG_I64`, `BinRMWPat` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_RMW32_U_XCHG_I64`, `BinRMWPat`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 401-407

```tablegen
  def : Pat<(ty (kind (AtomicAddrOps64 offset64_op:$offset, I64:$addr,
                       i32imm:$order), ty:$val)),
            (!cast<NI>(inst#_A64) $order, 0, $offset, $addr, $val)>,
        Requires<[HasAddr64, HasAtomics]>;
}

// Patterns for various addressing modes.
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 408-415

```tablegen
multiclass BinRMWPattern<PatFrag rmw_32, PatFrag rmw_64, string inst_32,
                         string inst_64> {
  defm : BinRMWPat<i32, rmw_32, inst_32>;
  defm : BinRMWPat<i64, rmw_64, inst_64>;
}

defm : BinRMWPattern<atomic_load_add_i32, atomic_load_add_i64,
                     "ATOMIC_RMW_ADD_I32", "ATOMIC_RMW_ADD_I64">;
```
- **EN**: Adds declarative TableGen records such as `BinRMWPattern` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BinRMWPattern`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 416-423

```tablegen
defm : BinRMWPattern<atomic_load_sub_i32, atomic_load_sub_i64,
                     "ATOMIC_RMW_SUB_I32", "ATOMIC_RMW_SUB_I64">;
defm : BinRMWPattern<atomic_load_and_i32, atomic_load_and_i64,
                     "ATOMIC_RMW_AND_I32", "ATOMIC_RMW_AND_I64">;
defm : BinRMWPattern<atomic_load_or_i32, atomic_load_or_i64,
                     "ATOMIC_RMW_OR_I32", "ATOMIC_RMW_OR_I64">;
defm : BinRMWPattern<atomic_load_xor_i32, atomic_load_xor_i64,
                     "ATOMIC_RMW_XOR_I32", "ATOMIC_RMW_XOR_I64">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 424-431

```tablegen
defm : BinRMWPattern<atomic_swap_i32, atomic_swap_i64,
                     "ATOMIC_RMW_XCHG_I32", "ATOMIC_RMW_XCHG_I64">;

// Truncating & zero-extending binary RMW patterns.
// These are combined patterns of truncating store patterns and zero-extending
// load patterns above.
class zext_bin_rmw_8_32<PatFrag kind> :
  PatFrag<(ops node:$addr, node:$val), (i32 (kind node:$addr, node:$val))>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Truncating & zero-extending binary RMW patterns.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Truncating & zero-extending binary RMW patterns.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 432-443

```tablegen
class zext_bin_rmw_16_32<PatFrag kind> : zext_bin_rmw_8_32<kind>;
class zext_bin_rmw_8_64<PatFrag kind> :
  PatFrag<(ops node:$addr, node:$val),
          (zext (i32 (kind node:$addr, (i32 (trunc (i64 node:$val))))))>;
class zext_bin_rmw_16_64<PatFrag kind> : zext_bin_rmw_8_64<kind>;
class zext_bin_rmw_32_64<PatFrag kind> : zext_bin_rmw_8_64<kind>;

// Truncating & sign-extending binary RMW patterns.
// These are combined patterns of truncating store patterns and sign-extending
// load patterns above. We match subword RMWs (for 32-bit) and anyext RMWs (for
// 64-bit) and select a zext RMW; the next instruction will be sext_inreg which
// is selected by itself.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Truncating & sign-extending binary RMW patterns.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Truncating & sign-extending binary RMW patterns.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 444-453

```tablegen
class sext_bin_rmw_8_32<PatFrag kind> :
  PatFrag<(ops node:$addr, node:$val), (kind node:$addr, node:$val)>;
class sext_bin_rmw_16_32<PatFrag kind> : sext_bin_rmw_8_32<kind>;
class sext_bin_rmw_8_64<PatFrag kind> :
  PatFrag<(ops node:$addr, node:$val),
          (anyext (i32 (kind node:$addr, (i32 (trunc (i64 node:$val))))))>;
class sext_bin_rmw_16_64<PatFrag kind> : sext_bin_rmw_8_64<kind>;
// 32->64 sext RMW gets selected as i32.atomic.rmw.***, i64.extend_i32_s

// Patterns for various addressing modes for truncating-extending binary RMWs.
```
- **EN**: Declares a backend-facing type `sext_bin_rmw_8_32`, `sext_bin_rmw_16_32`, `sext_bin_rmw_8_64` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `sext_bin_rmw_8_32`, `sext_bin_rmw_16_32`, `sext_bin_rmw_8_64`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 454-460

```tablegen
multiclass BinRMWTruncExtPattern<
  PatFrag rmw_8, PatFrag rmw_16, PatFrag rmw_32,
  string inst8_32, string inst16_32, string inst8_64, string inst16_64, string inst32_64> {
  // Truncating-extending binary RMWs
  defm : BinRMWPat<i32, zext_bin_rmw_8_32<rmw_8>, inst8_32>;
  defm : BinRMWPat<i32, zext_bin_rmw_16_32<rmw_16>, inst16_32>;
  defm : BinRMWPat<i64, zext_bin_rmw_8_64<rmw_8>, inst8_64>;
```
- **EN**: Adds declarative TableGen records such as `BinRMWTruncExtPattern` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BinRMWTruncExtPattern`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 461-469

```tablegen
  defm : BinRMWPat<i64, zext_bin_rmw_16_64<rmw_16>, inst16_64>;
  defm : BinRMWPat<i64, zext_bin_rmw_32_64<rmw_32>, inst32_64>;

  defm : BinRMWPat<i32, sext_bin_rmw_8_32<rmw_8>, inst8_32>;
  defm : BinRMWPat<i32, sext_bin_rmw_16_32<rmw_16>, inst16_32>;
  defm : BinRMWPat<i64, sext_bin_rmw_8_64<rmw_8>, inst8_64>;
  defm : BinRMWPat<i64, sext_bin_rmw_16_64<rmw_16>, inst16_64>;
}
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 470-477

```tablegen
defm : BinRMWTruncExtPattern<
  atomic_load_add_i8, atomic_load_add_i16, atomic_load_add_i32,
  "ATOMIC_RMW8_U_ADD_I32", "ATOMIC_RMW16_U_ADD_I32",
  "ATOMIC_RMW8_U_ADD_I64", "ATOMIC_RMW16_U_ADD_I64", "ATOMIC_RMW32_U_ADD_I64">;
defm : BinRMWTruncExtPattern<
  atomic_load_sub_i8, atomic_load_sub_i16, atomic_load_sub_i32,
  "ATOMIC_RMW8_U_SUB_I32", "ATOMIC_RMW16_U_SUB_I32",
  "ATOMIC_RMW8_U_SUB_I64", "ATOMIC_RMW16_U_SUB_I64", "ATOMIC_RMW32_U_SUB_I64">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 478-485

```tablegen
defm : BinRMWTruncExtPattern<
  atomic_load_and_i8, atomic_load_and_i16, atomic_load_and_i32,
  "ATOMIC_RMW8_U_AND_I32", "ATOMIC_RMW16_U_AND_I32",
  "ATOMIC_RMW8_U_AND_I64", "ATOMIC_RMW16_U_AND_I64", "ATOMIC_RMW32_U_AND_I64">;
defm : BinRMWTruncExtPattern<
  atomic_load_or_i8, atomic_load_or_i16, atomic_load_or_i32,
  "ATOMIC_RMW8_U_OR_I32", "ATOMIC_RMW16_U_OR_I32",
  "ATOMIC_RMW8_U_OR_I64", "ATOMIC_RMW16_U_OR_I64", "ATOMIC_RMW32_U_OR_I64">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 486-495

```tablegen
defm : BinRMWTruncExtPattern<
  atomic_load_xor_i8, atomic_load_xor_i16, atomic_load_xor_i32,
  "ATOMIC_RMW8_U_XOR_I32", "ATOMIC_RMW16_U_XOR_I32",
  "ATOMIC_RMW8_U_XOR_I64", "ATOMIC_RMW16_U_XOR_I64", "ATOMIC_RMW32_U_XOR_I64">;
defm : BinRMWTruncExtPattern<
  atomic_swap_i8, atomic_swap_i16, atomic_swap_i32,
  "ATOMIC_RMW8_U_XCHG_I32", "ATOMIC_RMW16_U_XCHG_I32",
  "ATOMIC_RMW8_U_XCHG_I64", "ATOMIC_RMW16_U_XCHG_I64",
  "ATOMIC_RMW32_U_XCHG_I64">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 496-497

```tablegen
//===----------------------------------------------------------------------===//
// Atomic ternary read-modify-writes
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Atomic ternary read-modify-writes".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Atomic ternary read-modify-writes”。

### Lines 498-505

```tablegen
//===----------------------------------------------------------------------===//

// TODO LLVM IR's cmpxchg instruction returns a pair of {loaded value, success
// flag}. When we use the success flag or both values, we can't make use of i64
// truncate/extend versions of instructions for now, which is suboptimal.
// Consider adding a pass after instruction selection that optimizes this case
// if it is frequent.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "TODO LLVM IR's cmpxchg instruction returns a pair of {loaded value, success".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“TODO LLVM IR's cmpxchg instruction returns a pair of {loaded value, success”。

### Lines 506-515

```tablegen
multiclass WebAssemblyTerRMW<WebAssemblyRegClass rc, string name,
                             int atomic_op> {
  defm "_A32" :
    ATOMIC_I<(outs rc:$dst),
             (ins MemOrder:$order, P2Align:$p2align, offset32_op:$off,
                  I32:$addr, rc:$exp, rc:$new_),
             (outs), (ins MemOrder:$order, P2Align:$p2align, offset32_op:$off),
             [], !strconcat(name, "\t$dst, ${order} "
                                  "${off}(${addr})${p2align}, $exp, $new_"),
             !strconcat(name, "\t${order} ${off}${p2align}"), atomic_op, false>;
```
- **EN**: Adds declarative TableGen records such as `WebAssemblyTerRMW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WebAssemblyTerRMW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 516-525

```tablegen
  defm "_A64" :
    ATOMIC_I<(outs rc:$dst),
             (ins MemOrder:$order, P2Align:$p2align, offset64_op:$off,
                  I64:$addr, rc:$exp, rc:$new_),
             (outs), (ins MemOrder:$order, P2Align:$p2align, offset64_op:$off),
             [], !strconcat(name, "\t$dst, ${order} "
                                  "${off}(${addr})${p2align}, $exp, $new_"),
             !strconcat(name, "\t${order} ${off}${p2align}"), atomic_op, true>;
}
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend TableGen declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 526-533

```tablegen
defm ATOMIC_RMW_CMPXCHG_I32 :
  WebAssemblyTerRMW<I32, "i32.atomic.rmw.cmpxchg", 0x48>;
defm ATOMIC_RMW_CMPXCHG_I64 :
  WebAssemblyTerRMW<I64, "i64.atomic.rmw.cmpxchg", 0x49>;
defm ATOMIC_RMW8_U_CMPXCHG_I32 :
  WebAssemblyTerRMW<I32, "i32.atomic.rmw8.cmpxchg_u", 0x4a>;
defm ATOMIC_RMW16_U_CMPXCHG_I32 :
  WebAssemblyTerRMW<I32, "i32.atomic.rmw16.cmpxchg_u", 0x4b>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_RMW_CMPXCHG_I32`, `ATOMIC_RMW_CMPXCHG_I64`, `ATOMIC_RMW8_U_CMPXCHG_I32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_RMW_CMPXCHG_I32`, `ATOMIC_RMW_CMPXCHG_I64`, `ATOMIC_RMW8_U_CMPXCHG_I32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 534-540

```tablegen
defm ATOMIC_RMW8_U_CMPXCHG_I64 :
  WebAssemblyTerRMW<I64, "i64.atomic.rmw8.cmpxchg_u", 0x4c>;
defm ATOMIC_RMW16_U_CMPXCHG_I64 :
  WebAssemblyTerRMW<I64, "i64.atomic.rmw16.cmpxchg_u", 0x4d>;
defm ATOMIC_RMW32_U_CMPXCHG_I64 :
  WebAssemblyTerRMW<I64, "i64.atomic.rmw32.cmpxchg_u", 0x4e>;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_RMW8_U_CMPXCHG_I64`, `ATOMIC_RMW16_U_CMPXCHG_I64`, `ATOMIC_RMW32_U_CMPXCHG_I64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_RMW8_U_CMPXCHG_I64`, `ATOMIC_RMW16_U_CMPXCHG_I64`, `ATOMIC_RMW32_U_CMPXCHG_I64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 541-549

```tablegen
multiclass TerRMWPat<ValueType ty, PatFrag kind, string inst> {
  def : Pat<(ty (kind (AtomicAddrOps32 offset32_op:$offset, I32:$addr, i32imm:$order), ty:$exp, ty:$new)),
            (!cast<NI>(inst#_A32) $order, 0, $offset, $addr, $exp, $new)>,
        Requires<[HasAddr32, HasAtomics]>;
  def : Pat<(ty (kind (AtomicAddrOps64 offset64_op:$offset, I64:$addr, i32imm:$order), ty:$exp, ty:$new)),
            (!cast<NI>(inst#_A64) $order, 0, $offset, $addr, $exp, $new)>,
        Requires<[HasAddr64, HasAtomics]>;
}
```
- **EN**: Adds declarative TableGen records such as `TerRMWPat` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TerRMWPat`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 550-558

```tablegen
defm : TerRMWPat<i32, atomic_cmp_swap_i32, "ATOMIC_RMW_CMPXCHG_I32">;
defm : TerRMWPat<i64, atomic_cmp_swap_i64, "ATOMIC_RMW_CMPXCHG_I64">;

// Truncating & zero-extending ternary RMW patterns.
// DAG legalization & optimization before instruction selection may introduce
// additional nodes such as anyext or assertzext depending on operand types.
class zext_ter_rmw_8_32<PatFrag kind> :
  PatFrag<(ops node:$addr, node:$exp, node:$new),
          (i32 (kind node:$addr, node:$exp, node:$new))>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Truncating & zero-extending ternary RMW patterns.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Truncating & zero-extending ternary RMW patterns.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 559-565

```tablegen
class zext_ter_rmw_16_32<PatFrag kind> : zext_ter_rmw_8_32<kind>;
class zext_ter_rmw_8_64<PatFrag kind> :
  PatFrag<(ops node:$addr, node:$exp, node:$new),
          (zext (i32 (assertzext (i32 (kind node:$addr,
                                            (i32 (trunc (i64 node:$exp))),
                                            (i32 (trunc (i64 node:$new))))))))>;
class zext_ter_rmw_16_64<PatFrag kind> : zext_ter_rmw_8_64<kind>;
```
- **EN**: Declares a backend-facing type `zext_ter_rmw_16_32`, `zext_ter_rmw_8_64`, `zext_ter_rmw_16_64` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `zext_ter_rmw_16_32`, `zext_ter_rmw_8_64`, `zext_ter_rmw_16_64`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 566-575

```tablegen
class zext_ter_rmw_32_64<PatFrag kind> :
  PatFrag<(ops node:$addr, node:$exp, node:$new),
          (zext (i32 (kind node:$addr,
                           (i32 (trunc (i64 node:$exp))),
                           (i32 (trunc (i64 node:$new))))))>;

// Truncating & sign-extending ternary RMW patterns.
// We match subword RMWs (for 32-bit) and anyext RMWs (for 64-bit) and select a
// zext RMW; the next instruction will be sext_inreg which is selected by
// itself.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Truncating & sign-extending ternary RMW patterns.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Truncating & sign-extending ternary RMW patterns.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 576-585

```tablegen
class sext_ter_rmw_8_32<PatFrag kind> :
  PatFrag<(ops node:$addr, node:$exp, node:$new),
          (kind node:$addr, node:$exp, node:$new)>;
class sext_ter_rmw_16_32<PatFrag kind> : sext_ter_rmw_8_32<kind>;
class sext_ter_rmw_8_64<PatFrag kind> :
  PatFrag<(ops node:$addr, node:$exp, node:$new),
          (anyext (i32 (assertzext (i32
            (kind node:$addr,
                  (i32 (trunc (i64 node:$exp))),
                  (i32 (trunc (i64 node:$new))))))))>;
```
- **EN**: Declares a backend-facing type `sext_ter_rmw_8_32`, `sext_ter_rmw_16_32`, `sext_ter_rmw_8_64` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `sext_ter_rmw_8_32`, `sext_ter_rmw_16_32`, `sext_ter_rmw_8_64`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 586-592

```tablegen
class sext_ter_rmw_16_64<PatFrag kind> : sext_ter_rmw_8_64<kind>;
// 32->64 sext RMW gets selected as i32.atomic.rmw.***, i64.extend_i32_s

defm : TerRMWPat<i32, zext_ter_rmw_8_32<atomic_cmp_swap_i8>, "ATOMIC_RMW8_U_CMPXCHG_I32">;
defm : TerRMWPat<i32, zext_ter_rmw_16_32<atomic_cmp_swap_i16>, "ATOMIC_RMW16_U_CMPXCHG_I32">;
defm : TerRMWPat<i64, zext_ter_rmw_8_64<atomic_cmp_swap_i8>, "ATOMIC_RMW8_U_CMPXCHG_I64">;
defm : TerRMWPat<i64, zext_ter_rmw_16_64<atomic_cmp_swap_i16>, "ATOMIC_RMW16_U_CMPXCHG_I64">;
```
- **EN**: Declares a backend-facing type `sext_ter_rmw_16_64` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `sext_ter_rmw_16_64`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 593-598

```tablegen
defm : TerRMWPat<i64, zext_ter_rmw_32_64<atomic_cmp_swap_i32>, "ATOMIC_RMW32_U_CMPXCHG_I64">;

defm : TerRMWPat<i32, sext_ter_rmw_8_32<atomic_cmp_swap_i8>, "ATOMIC_RMW8_U_CMPXCHG_I32">;
defm : TerRMWPat<i32, sext_ter_rmw_16_32<atomic_cmp_swap_i16>, "ATOMIC_RMW16_U_CMPXCHG_I32">;
defm : TerRMWPat<i64, sext_ter_rmw_8_64<atomic_cmp_swap_i8>, "ATOMIC_RMW8_U_CMPXCHG_I64">;
defm : TerRMWPat<i64, sext_ter_rmw_16_64<atomic_cmp_swap_i16>, "ATOMIC_RMW16_U_CMPXCHG_I64">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- SelectionDAG lowering / SelectionDAG lowering
- Declarative TableGen records / 声明式 TableGen 记录
- Atomic operation support / 原子操作支持
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- Generated backend tables / 生成式后端表
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
