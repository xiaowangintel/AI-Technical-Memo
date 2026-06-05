# WebAssemblyInstrMemory.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyInstrMemory.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: WebAssembly Memory operand code-gen constructs. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyInstrMemory.td`，主要负责 WebAssembly 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
// WebAssemblyInstrMemory.td-WebAssembly Memory codegen support -*- tablegen -*-
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
/// WebAssembly Memory operand code-gen constructs.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-32

```tablegen
//===----------------------------------------------------------------------===//

// TODO:
//  - WebAssemblyTargetLowering having to do with atomics
//  - Each has optional alignment.

// WebAssembly has i8/i16/i32/i64/f32/f64 memory types, but doesn't have i8/i16
// local types. These memory-only types instead zero- or sign-extend into local
// types when loading, and truncate when storing.

// Address Operands

// These patterns match the static (offset) and dynamic (address stack operand)
// operands for loads and stores, based on a combination of target global
// addresses and constants.
// For example,
// (load (add tga, x))   -> load offset=tga, addr=x
// (store v, tga)        -> store v, offset=tga, addr=0
// (load (add const, x)) -> load offset=const, addr=x
// (store v, const)      -> store v, offset=const, addr=0
// (load x)              -> load offset=0, addr=x
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "TODO:".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“TODO:”。

### Lines 33-39

```tablegen
def AddrOps32 : ComplexPattern<i32, 2, "SelectAddrOperands32">;
def AddrOps64 : ComplexPattern<i64, 2, "SelectAddrOperands64">;

// Defines atomic and non-atomic loads, regular and extending.
multiclass WebAssemblyLoad<WebAssemblyRegClass rc, string Name, int Opcode,
                           list<Predicate> reqs = []> {
  let mayLoad = 1, UseNamedOperandTable = 1 in {
```
- **EN**: Adds declarative TableGen records such as `AddrOps32`, `AddrOps64`, `WebAssemblyLoad` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `AddrOps32`, `AddrOps64`, `WebAssemblyLoad`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 40-57

```tablegen
  defm "_A32": I<(outs rc:$dst),
                 (ins P2Align:$p2align, offset32_op:$off, I32:$addr),
                 (outs), (ins P2Align:$p2align, offset32_op:$off),
                 [], !strconcat(Name, "\t$dst, ${off}(${addr})${p2align}"),
                 !strconcat(Name, "\t${off}${p2align}"), Opcode, false>,
               Requires<reqs>;
  defm "_A64": I<(outs rc:$dst),
                 (ins P2Align:$p2align, offset64_op:$off, I64:$addr),
                 (outs), (ins P2Align:$p2align, offset64_op:$off),
                 [], !strconcat(Name, "\t$dst, ${off}(${addr})${p2align}"),
                 !strconcat(Name, "\t${off}${p2align}"), Opcode, true>,
               Requires<reqs>;
  }
}

// Basic load.
// FIXME: When we can break syntax compatibility, reorder the fields in the
// asmstrings to match the binary encoding.
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend TableGen declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 58-64

```tablegen
defm LOAD_I32 : WebAssemblyLoad<I32, "i32.load", 0x28, []>;
defm LOAD_I64 : WebAssemblyLoad<I64, "i64.load", 0x29, []>;
defm LOAD_F32 : WebAssemblyLoad<F32, "f32.load", 0x2a, []>;
defm LOAD_F64 : WebAssemblyLoad<F64, "f64.load", 0x2b, []>;

// Extending load.
defm LOAD8_S_I32 : WebAssemblyLoad<I32, "i32.load8_s", 0x2c, []>;
```
- **EN**: Adds declarative TableGen records such as `LOAD_I32`, `LOAD_I64`, `LOAD_F32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LOAD_I32`, `LOAD_I64`, `LOAD_F32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 65-71

```tablegen
defm LOAD8_U_I32 : WebAssemblyLoad<I32, "i32.load8_u", 0x2d, []>;
defm LOAD16_S_I32 : WebAssemblyLoad<I32, "i32.load16_s", 0x2e, []>;
defm LOAD16_U_I32 : WebAssemblyLoad<I32, "i32.load16_u", 0x2f, []>;
defm LOAD8_S_I64 : WebAssemblyLoad<I64, "i64.load8_s", 0x30, []>;
defm LOAD8_U_I64 : WebAssemblyLoad<I64, "i64.load8_u", 0x31, []>;
defm LOAD16_S_I64 : WebAssemblyLoad<I64, "i64.load16_s", 0x32, []>;
defm LOAD16_U_I64 : WebAssemblyLoad<I64, "i64.load16_u", 0x33, []>;
```
- **EN**: Adds declarative TableGen records such as `LOAD8_U_I32`, `LOAD16_S_I32`, `LOAD16_U_I32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LOAD8_U_I32`, `LOAD16_S_I32`, `LOAD16_U_I32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 72-80

```tablegen
defm LOAD32_S_I64 : WebAssemblyLoad<I64, "i64.load32_s", 0x34, []>;
defm LOAD32_U_I64 : WebAssemblyLoad<I64, "i64.load32_u", 0x35, []>;

// Half-precision load.
defm LOAD_F16_F32 :
  WebAssemblyLoad<F32, "f32.load_f16", 0xfc30, [HasFP16]>;

// Pattern matching
```
- **EN**: Adds declarative TableGen records such as `LOAD32_S_I64`, `LOAD32_U_I64`, `LOAD_F16_F32` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LOAD32_S_I64`, `LOAD32_U_I64`, `LOAD_F16_F32`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 81-87

```tablegen
multiclass LoadPat<ValueType ty, SDPatternOperator kind, string Name> {
      def : Pat<(ty (kind (AddrOps32 offset32_op:$offset, I32:$addr))),
           (!cast<NI>(Name # "_A32") 0,
                                     offset32_op:$offset,
                                     I32:$addr)>,
                                     Requires<[HasAddr32]>;
```
- **EN**: Adds declarative TableGen records such as `LoadPat` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LoadPat`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 88-94

```tablegen
      def : Pat<(ty (kind (AddrOps64 offset64_op:$offset, I64:$addr))),
           (!cast<NI>(Name # "_A64") 0,
                                     offset64_op:$offset,
                                     I64:$addr)>,
                                     Requires<[HasAddr64]>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 95-101

```tablegen
defm : LoadPat<i32, load, "LOAD_I32">;
defm : LoadPat<i64, load, "LOAD_I64">;
defm : LoadPat<f32, load, "LOAD_F32">;
defm : LoadPat<f64, load, "LOAD_F64">;

defm : LoadPat<i32, sextloadi8, "LOAD8_S_I32">;
defm : LoadPat<i32, sextloadi16, "LOAD16_S_I32">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 102-108

```tablegen
defm : LoadPat<i64, sextloadi8, "LOAD8_S_I64">;
defm : LoadPat<i64, sextloadi16, "LOAD16_S_I64">;
defm : LoadPat<i64, sextloadi32, "LOAD32_S_I64">;

defm : LoadPat<i32, zextloadi8, "LOAD8_U_I32">;
defm : LoadPat<i32, zextloadi16, "LOAD16_U_I32">;
defm : LoadPat<i64, zextloadi8, "LOAD8_U_I64">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 109-115

```tablegen
defm : LoadPat<i64, zextloadi16, "LOAD16_U_I64">;
defm : LoadPat<i64, zextloadi32, "LOAD32_U_I64">;

defm : LoadPat<i32, extloadi8, "LOAD8_U_I32">;
defm : LoadPat<i32, extloadi16, "LOAD16_U_I32">;
defm : LoadPat<i64, extloadi8, "LOAD8_U_I64">;
defm : LoadPat<i64, extloadi16, "LOAD16_U_I64">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 116-123

```tablegen
defm : LoadPat<i64, extloadi32, "LOAD32_U_I64">;

defm : LoadPat<f32, extloadf16, "LOAD_F16_F32">;
defm : LoadPat<f32, int_wasm_loadf16_f32, "LOAD_F16_F32">;

// Defines atomic and non-atomic stores, regular and truncating
multiclass WebAssemblyStore<WebAssemblyRegClass rc, string Name, int Opcode,
                            list<Predicate> reqs = []> {
```
- **EN**: Adds declarative TableGen records such as `WebAssemblyStore` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WebAssemblyStore`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 124-131

```tablegen
  let mayStore = 1, UseNamedOperandTable = 1 in
  defm "_A32" : I<(outs),
                  (ins P2Align:$p2align, offset32_op:$off, I32:$addr, rc:$val),
                  (outs),
                  (ins P2Align:$p2align, offset32_op:$off), [],
                  !strconcat(Name, "\t${off}(${addr})${p2align}, $val"),
                  !strconcat(Name, "\t${off}${p2align}"), Opcode, false>,
                Requires<reqs>;
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend TableGen declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 132-143

```tablegen
  let mayStore = 1, UseNamedOperandTable = 1 in
  defm "_A64" : I<(outs),
                  (ins P2Align:$p2align, offset64_op:$off, I64:$addr, rc:$val),
                  (outs),
                  (ins P2Align:$p2align, offset64_op:$off), [],
                  !strconcat(Name, "\t${off}(${addr})${p2align}, $val"),
                  !strconcat(Name, "\t${off}${p2align}"), Opcode, true>,
                Requires<reqs>;
}

// Basic store.
// Note: WebAssembly inverts SelectionDAG's usual operand order.
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend TableGen declarations for the backend. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 TableGen 声明所需的辅助例程。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 144-155

```tablegen
defm STORE_I32  : WebAssemblyStore<I32, "i32.store", 0x36>;
defm STORE_I64  : WebAssemblyStore<I64, "i64.store", 0x37>;
defm STORE_F32  : WebAssemblyStore<F32, "f32.store", 0x38>;
defm STORE_F64  : WebAssemblyStore<F64, "f64.store", 0x39>;

multiclass StorePat<ValueType ty, SDPatternOperator kind, string Name> {
  def : Pat<(kind ty:$val, (AddrOps32 offset32_op:$offset, I32:$addr)),
            (!cast<NI>(Name # "_A32") 0,
                                      offset32_op:$offset,
                                      I32:$addr,
                                      ty:$val)>,
            Requires<[HasAddr32]>;
```
- **EN**: Adds declarative TableGen records such as `STORE_I32`, `STORE_I64`, `STORE_F32` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STORE_I32`, `STORE_I64`, `STORE_F32`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 156-163

```tablegen
  def : Pat<(kind ty:$val, (AddrOps64 offset64_op:$offset, I64:$addr)),
            (!cast<NI>(Name # "_A64") 0,
                                      offset64_op:$offset,
                                      I64:$addr,
                                      ty:$val)>,
            Requires<[HasAddr64]>;
}
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 164-170

```tablegen
defm : StorePat<i32, store, "STORE_I32">;
defm : StorePat<i64, store, "STORE_I64">;
defm : StorePat<f32, store, "STORE_F32">;
defm : StorePat<f64, store, "STORE_F64">;

// Truncating store.
defm STORE8_I32 : WebAssemblyStore<I32, "i32.store8", 0x3a>;
```
- **EN**: Adds declarative TableGen records such as `STORE8_I32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STORE8_I32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 171-179

```tablegen
defm STORE16_I32 : WebAssemblyStore<I32, "i32.store16", 0x3b>;
defm STORE8_I64 : WebAssemblyStore<I64, "i64.store8", 0x3c>;
defm STORE16_I64 : WebAssemblyStore<I64, "i64.store16", 0x3d>;
defm STORE32_I64 : WebAssemblyStore<I64, "i64.store32", 0x3e>;

// Half-precision store.
defm STORE_F16_F32 :
  WebAssemblyStore<F32, "f32.store_f16", 0xfc31, [HasFP16]>;
```
- **EN**: Adds declarative TableGen records such as `STORE16_I32`, `STORE8_I64`, `STORE16_I64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STORE16_I32`, `STORE8_I64`, `STORE16_I64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 180-186

```tablegen
defm : StorePat<i32, truncstorei8, "STORE8_I32">;
defm : StorePat<i32, truncstorei16, "STORE16_I32">;
defm : StorePat<i64, truncstorei8, "STORE8_I64">;
defm : StorePat<i64, truncstorei16, "STORE16_I64">;
defm : StorePat<i64, truncstorei32, "STORE32_I64">;

defm : StorePat<f32, truncstoref16, "STORE_F16_F32">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 187-198

```tablegen
defm : StorePat<f32, int_wasm_storef16_f32, "STORE_F16_F32">;

multiclass MemoryOps<WebAssemblyRegClass rc, string B> {
// Current memory size.
defm MEMORY_SIZE_A#B : I<(outs rc:$dst), (ins i32imm:$flags),
                         (outs), (ins i32imm:$flags),
                         [(set rc:$dst,
                           (int_wasm_memory_size (i32 imm:$flags)))],
                         "memory.size\t$dst, $flags", "memory.size\t$flags",
                         0x3f>;

// Grow memory.
```
- **EN**: Adds declarative TableGen records such as `MemoryOps`, `MEMORY_SIZE_A` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MemoryOps`, `MEMORY_SIZE_A`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 199-207

```tablegen
defm MEMORY_GROW_A#B : I<(outs rc:$dst), (ins i32imm:$flags, rc:$delta),
                         (outs), (ins i32imm:$flags),
                         [(set rc:$dst,
                           (int_wasm_memory_grow (i32 imm:$flags),
                             rc:$delta))],
                         "memory.grow\t$dst, $flags, $delta",
                         "memory.grow\t$flags", 0x40>;
}
```
- **EN**: Adds declarative TableGen records such as `MEMORY_GROW_A` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MEMORY_GROW_A`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 208-209

```tablegen
defm : MemoryOps<I32, "32">;
defm : MemoryOps<I64, "64">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- SelectionDAG lowering / SelectionDAG lowering
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- Generated backend tables / 生成式后端表
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
