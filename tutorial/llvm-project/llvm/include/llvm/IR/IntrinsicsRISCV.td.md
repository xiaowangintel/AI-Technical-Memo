# IntrinsicsRISCV.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/IntrinsicsRISCV.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file defines all of the RISCV-specific intrinsics.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/IR`，主要定义与 `IntrinsicsRISCV` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

````tablegen
//===- IntrinsicsRISCV.td - Defines RISCV intrinsics -------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines all of the RISCV-specific intrinsics.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Atomics

// Atomic Intrinsics have multiple versions for different access widths, which
// all follow one of the following signatures (depending on how many arguments
// they require).
//
// In fact, as these intrinsics take `llvm_anyptr_ty`, the given names are the
// canonical names, and the intrinsics used in the code will have a name
// suffixed with the pointer type they are specialised for (denoted `<p>` in the
// names below), in order to avoid type conflicts.

let TargetPrefix = "riscv" in {

  // T @llvm.<name>.<i>.<p>(any*, T, T, T imm);
  class RISCVMaskedAtomicRMWFourArg
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This file defines all of the RISCV-specific intrinsics.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This file defines all of the RISCV-specific intrinsics.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Banner comment marking a file or section boundary.
  **L13 CN**: 横幅注释，用于标记文件或章节边界。
- **L14 EN**: Comment explains nearby logic, invariants, or intent: `Atomics`.
  **L14 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Atomics`。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L16 EN**: Comment explains nearby logic, invariants, or intent: `Atomic Intrinsics have multiple versions for different access widths, which`.
  **L16 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Atomic Intrinsics have multiple versions for different access widths, which`。
- **L17 EN**: Comment explains nearby logic, invariants, or intent: `all follow one of the following signatures (depending on how many arguments`.
  **L17 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all follow one of the following signatures (depending on how many arguments`。
- **L18 EN**: Comment explains nearby logic, invariants, or intent: `they require).`.
  **L18 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`they require).`。
- **L19 EN**: Separator comment used for visual grouping.
  **L19 CN**: 用于视觉分组的分隔注释。
- **L20 EN**: Comment explains nearby logic, invariants, or intent: `In fact, as these intrinsics take `llvm_anyptr_ty`, the given names are the`.
  **L20 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`In fact, as these intrinsics take `llvm_anyptr_ty`, the given names are the`。
- **L21 EN**: Comment explains nearby logic, invariants, or intent: `canonical names, and the intrinsics used in the code will have a name`.
  **L21 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`canonical names, and the intrinsics used in the code will have a name`。
- **L22 EN**: Comment explains nearby logic, invariants, or intent: `suffixed with the pointer type they are specialised for (denoted `<p>` in the`.
  **L22 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`suffixed with the pointer type they are specialised for (denoted `<p>` in the`。
- **L23 EN**: Comment explains nearby logic, invariants, or intent: `names below), in order to avoid type conflicts.`.
  **L23 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`names below), in order to avoid type conflicts.`。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L25 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L25 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Comment explains nearby logic, invariants, or intent: `T @llvm.<name>.<i>.<p>(any*, T, T, T imm);`.
  **L27 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`T @llvm.<name>.<i>.<p>(any*, T, T, T imm);`。
- **L28 EN**: Declares class `RISCVMaskedAtomicRMWFourArg`.
  **L28 CN**: 声明 class `RISCVMaskedAtomicRMWFourArg`。

### Lines 29-56

````tablegen
      : Intrinsic<[llvm_anyint_ty], [llvm_anyptr_ty, LLVMMatchType<0>,
                                     LLVMMatchType<0>, LLVMMatchType<0>],
                  [IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<3>>]>;
  // T @llvm.<name>.<i>.<p>(any*, T, T, T, T imm);
  class RISCVMaskedAtomicRMWFiveArg
      : Intrinsic<[llvm_anyint_ty], [llvm_anyptr_ty, LLVMMatchType<0>,
                                     LLVMMatchType<0>, LLVMMatchType<0>,
                                     LLVMMatchType<0>],
                  [IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<4>>]>;

  // These intrinsics are intended only for internal compiler use (i.e. as
  // part of AtomicExpandpass via the emitMaskedAtomic*Intrinsic hooks). Their
  // names and semantics could change in the future.

  // @llvm.riscv.masked.atomicrmw.*.<i>.<p>(
  //   ptr addr, ixlen oparg, ixlen mask, ixlenimm ordering)
  def int_riscv_masked_atomicrmw_xchg : RISCVMaskedAtomicRMWFourArg;
  def int_riscv_masked_atomicrmw_add : RISCVMaskedAtomicRMWFourArg;
  def int_riscv_masked_atomicrmw_sub : RISCVMaskedAtomicRMWFourArg;
  def int_riscv_masked_atomicrmw_nand : RISCVMaskedAtomicRMWFourArg;
  def int_riscv_masked_atomicrmw_umax : RISCVMaskedAtomicRMWFourArg;
  def int_riscv_masked_atomicrmw_umin : RISCVMaskedAtomicRMWFourArg;
  // Signed min and max need an extra operand to do sign extension with.
  // @llvm.riscv.masked.atomicrmw.{max,min}.<i>.<p>(
  //   ptr addr, ixlen oparg, ixlen mask, ixlen shamt, ixlenimm ordering)
  def int_riscv_masked_atomicrmw_max : RISCVMaskedAtomicRMWFiveArg;
  def int_riscv_masked_atomicrmw_min : RISCVMaskedAtomicRMWFiveArg;

````
- **L29 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[llvm_anyint_ty], [llvm_anyptr_ty, LLVMMatchType<0>,`.
  **L29 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[llvm_anyint_ty], [llvm_anyptr_ty, LLVMMatchType<0>,`。
- **L30 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L30 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L31 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<3>>]>;`.
  **L31 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<3>>]>;`。
- **L32 EN**: Comment explains nearby logic, invariants, or intent: `T @llvm.<name>.<i>.<p>(any*, T, T, T, T imm);`.
  **L32 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`T @llvm.<name>.<i>.<p>(any*, T, T, T, T imm);`。
- **L33 EN**: Declares class `RISCVMaskedAtomicRMWFiveArg`.
  **L33 CN**: 声明 class `RISCVMaskedAtomicRMWFiveArg`。
- **L34 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[llvm_anyint_ty], [llvm_anyptr_ty, LLVMMatchType<0>,`.
  **L34 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[llvm_anyint_ty], [llvm_anyptr_ty, LLVMMatchType<0>,`。
- **L35 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L35 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L36 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L36 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L37 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<4>>]>;`.
  **L37 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<4>>]>;`。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L39 EN**: Comment explains nearby logic, invariants, or intent: `These intrinsics are intended only for internal compiler use (i.e. as`.
  **L39 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These intrinsics are intended only for internal compiler use (i.e. as`。
- **L40 EN**: Comment explains nearby logic, invariants, or intent: `part of AtomicExpandpass via the emitMaskedAtomic*Intrinsic hooks). Their`.
  **L40 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`part of AtomicExpandpass via the emitMaskedAtomic*Intrinsic hooks). Their`。
- **L41 EN**: Comment explains nearby logic, invariants, or intent: `names and semantics could change in the future.`.
  **L41 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`names and semantics could change in the future.`。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L43 EN**: Comment explains nearby logic, invariants, or intent: `@llvm.riscv.masked.atomicrmw.*.<i>.<p>(`.
  **L43 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`@llvm.riscv.masked.atomicrmw.*.<i>.<p>(`。
- **L44 EN**: Comment explains nearby logic, invariants, or intent: `ptr addr, ixlen oparg, ixlen mask, ixlenimm ordering)`.
  **L44 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ptr addr, ixlen oparg, ixlen mask, ixlenimm ordering)`。
- **L45 EN**: Declares TableGen def `int_riscv_masked_atomicrmw_xchg`.
  **L45 CN**: 声明 TableGen def `int_riscv_masked_atomicrmw_xchg`。
- **L46 EN**: Declares TableGen def `int_riscv_masked_atomicrmw_add`.
  **L46 CN**: 声明 TableGen def `int_riscv_masked_atomicrmw_add`。
- **L47 EN**: Declares TableGen def `int_riscv_masked_atomicrmw_sub`.
  **L47 CN**: 声明 TableGen def `int_riscv_masked_atomicrmw_sub`。
- **L48 EN**: Declares TableGen def `int_riscv_masked_atomicrmw_nand`.
  **L48 CN**: 声明 TableGen def `int_riscv_masked_atomicrmw_nand`。
- **L49 EN**: Declares TableGen def `int_riscv_masked_atomicrmw_umax`.
  **L49 CN**: 声明 TableGen def `int_riscv_masked_atomicrmw_umax`。
- **L50 EN**: Declares TableGen def `int_riscv_masked_atomicrmw_umin`.
  **L50 CN**: 声明 TableGen def `int_riscv_masked_atomicrmw_umin`。
- **L51 EN**: Comment explains nearby logic, invariants, or intent: `Signed min and max need an extra operand to do sign extension with.`.
  **L51 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Signed min and max need an extra operand to do sign extension with.`。
- **L52 EN**: Comment explains nearby logic, invariants, or intent: `@llvm.riscv.masked.atomicrmw.{max,min}.<i>.<p>(`.
  **L52 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`@llvm.riscv.masked.atomicrmw.{max,min}.<i>.<p>(`。
- **L53 EN**: Comment explains nearby logic, invariants, or intent: `ptr addr, ixlen oparg, ixlen mask, ixlen shamt, ixlenimm ordering)`.
  **L53 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ptr addr, ixlen oparg, ixlen mask, ixlen shamt, ixlenimm ordering)`。
- **L54 EN**: Declares TableGen def `int_riscv_masked_atomicrmw_max`.
  **L54 CN**: 声明 TableGen def `int_riscv_masked_atomicrmw_max`。
- **L55 EN**: Declares TableGen def `int_riscv_masked_atomicrmw_min`.
  **L55 CN**: 声明 TableGen def `int_riscv_masked_atomicrmw_min`。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 57-84

````tablegen
  // @llvm.riscv.masked.cmpxchg.<i>.<p>(
  //   ptr addr, ixlen cmpval, ixlen newval, ixlen mask, ixlenimm ordering)
  def int_riscv_masked_cmpxchg : RISCVMaskedAtomicRMWFiveArg;

} // TargetPrefix = "riscv"

//===----------------------------------------------------------------------===//
// Bitmanip (Bit Manipulation) Extension

let TargetPrefix = "riscv" in {

  class RISCVBitManipGPRIntrinsics
      : DefaultAttrsIntrinsic<[llvm_anyint_ty],
                              [LLVMMatchType<0>],
                              [IntrNoMem, IntrSpeculatable]>;
  class RISCVBitManipGPRGPRIntrinsics
      : DefaultAttrsIntrinsic<[llvm_anyint_ty],
                              [LLVMMatchType<0>, LLVMMatchType<0>],
                              [IntrNoMem, IntrSpeculatable]>;

  // Zbb
  def int_riscv_orc_b : RISCVBitManipGPRIntrinsics;

  // Zbc or Zbkc
  def int_riscv_clmulh : RISCVBitManipGPRGPRIntrinsics;

  // Zbc
  def int_riscv_clmulr : RISCVBitManipGPRGPRIntrinsics;
````
- **L57 EN**: Comment explains nearby logic, invariants, or intent: `@llvm.riscv.masked.cmpxchg.<i>.<p>(`.
  **L57 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`@llvm.riscv.masked.cmpxchg.<i>.<p>(`。
- **L58 EN**: Comment explains nearby logic, invariants, or intent: `ptr addr, ixlen cmpval, ixlen newval, ixlen mask, ixlenimm ordering)`.
  **L58 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ptr addr, ixlen cmpval, ixlen newval, ixlen mask, ixlenimm ordering)`。
- **L59 EN**: Declares TableGen def `int_riscv_masked_cmpxchg`.
  **L59 CN**: 声明 TableGen def `int_riscv_masked_cmpxchg`。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L61 EN**: Continues the surrounding expression or declaration: `} // TargetPrefix = "riscv"`.
  **L61 CN**: 继续构造周围的表达式或声明：`} // TargetPrefix = "riscv"`。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L63 EN**: Banner comment marking a file or section boundary.
  **L63 CN**: 横幅注释，用于标记文件或章节边界。
- **L64 EN**: Comment explains nearby logic, invariants, or intent: `Bitmanip (Bit Manipulation) Extension`.
  **L64 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Bitmanip (Bit Manipulation) Extension`。
- **L65 EN**: Blank line separating nearby declarations or logic blocks.
  **L65 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L66 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L66 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L67 EN**: Blank line separating nearby declarations or logic blocks.
  **L67 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L68 EN**: Declares class `RISCVBitManipGPRIntrinsics`.
  **L68 CN**: 声明 class `RISCVBitManipGPRIntrinsics`。
- **L69 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyint_ty],`.
  **L69 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyint_ty],`。
- **L70 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>],`.
  **L70 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>],`。
- **L71 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L71 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L72 EN**: Declares class `RISCVBitManipGPRGPRIntrinsics`.
  **L72 CN**: 声明 class `RISCVBitManipGPRGPRIntrinsics`。
- **L73 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyint_ty],`.
  **L73 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyint_ty],`。
- **L74 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L74 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L75 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L75 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L76 EN**: Blank line separating nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L77 EN**: Comment explains nearby logic, invariants, or intent: `Zbb`.
  **L77 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zbb`。
- **L78 EN**: Declares TableGen def `int_riscv_orc_b`.
  **L78 CN**: 声明 TableGen def `int_riscv_orc_b`。
- **L79 EN**: Blank line separating nearby declarations or logic blocks.
  **L79 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L80 EN**: Comment explains nearby logic, invariants, or intent: `Zbc or Zbkc`.
  **L80 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zbc or Zbkc`。
- **L81 EN**: Declares TableGen def `int_riscv_clmulh`.
  **L81 CN**: 声明 TableGen def `int_riscv_clmulh`。
- **L82 EN**: Blank line separating nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L83 EN**: Comment explains nearby logic, invariants, or intent: `Zbc`.
  **L83 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zbc`。
- **L84 EN**: Declares TableGen def `int_riscv_clmulr`.
  **L84 CN**: 声明 TableGen def `int_riscv_clmulr`。

### Lines 85-112

````tablegen

  // Zbkb
  def int_riscv_brev8 : RISCVBitManipGPRIntrinsics;
  def int_riscv_zip   : RISCVBitManipGPRIntrinsics;
  def int_riscv_unzip : RISCVBitManipGPRIntrinsics;

  // Zbkx
  def int_riscv_xperm4  : RISCVBitManipGPRGPRIntrinsics;
  def int_riscv_xperm8  : RISCVBitManipGPRGPRIntrinsics;
} // TargetPrefix = "riscv"

//===----------------------------------------------------------------------===//
// May-Be-Operations

let TargetPrefix = "riscv" in {

  // Zimop
  def int_riscv_mopr
      : DefaultAttrsIntrinsic<[llvm_anyint_ty],
                              [LLVMMatchType<0>, LLVMMatchType<0>],
                              [IntrNoMem, IntrSpeculatable, ImmArg<ArgIndex<1>>]>;
  def int_riscv_moprr
      : DefaultAttrsIntrinsic<[llvm_anyint_ty],
                              [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
                              [IntrNoMem, IntrSpeculatable, ImmArg<ArgIndex<2>>]>;
} // TargetPrefix = "riscv"

//===----------------------------------------------------------------------===//
````
- **L85 EN**: Blank line separating nearby declarations or logic blocks.
  **L85 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L86 EN**: Comment explains nearby logic, invariants, or intent: `Zbkb`.
  **L86 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zbkb`。
- **L87 EN**: Declares TableGen def `int_riscv_brev8`.
  **L87 CN**: 声明 TableGen def `int_riscv_brev8`。
- **L88 EN**: Declares TableGen def `int_riscv_zip`.
  **L88 CN**: 声明 TableGen def `int_riscv_zip`。
- **L89 EN**: Declares TableGen def `int_riscv_unzip`.
  **L89 CN**: 声明 TableGen def `int_riscv_unzip`。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L91 EN**: Comment explains nearby logic, invariants, or intent: `Zbkx`.
  **L91 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zbkx`。
- **L92 EN**: Declares TableGen def `int_riscv_xperm4`.
  **L92 CN**: 声明 TableGen def `int_riscv_xperm4`。
- **L93 EN**: Declares TableGen def `int_riscv_xperm8`.
  **L93 CN**: 声明 TableGen def `int_riscv_xperm8`。
- **L94 EN**: Continues the surrounding expression or declaration: `} // TargetPrefix = "riscv"`.
  **L94 CN**: 继续构造周围的表达式或声明：`} // TargetPrefix = "riscv"`。
- **L95 EN**: Blank line separating nearby declarations or logic blocks.
  **L95 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L96 EN**: Banner comment marking a file or section boundary.
  **L96 CN**: 横幅注释，用于标记文件或章节边界。
- **L97 EN**: Comment explains nearby logic, invariants, or intent: `May-Be-Operations`.
  **L97 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`May-Be-Operations`。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L99 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L99 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L100 EN**: Blank line separating nearby declarations or logic blocks.
  **L100 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L101 EN**: Comment explains nearby logic, invariants, or intent: `Zimop`.
  **L101 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zimop`。
- **L102 EN**: Declares TableGen def `int_riscv_mopr`.
  **L102 CN**: 声明 TableGen def `int_riscv_mopr`。
- **L103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyint_ty],`.
  **L103 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyint_ty],`。
- **L104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L104 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L105 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable, ImmArg<ArgIndex<1>>]>;`.
  **L105 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable, ImmArg<ArgIndex<1>>]>;`。
- **L106 EN**: Declares TableGen def `int_riscv_moprr`.
  **L106 CN**: 声明 TableGen def `int_riscv_moprr`。
- **L107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyint_ty],`.
  **L107 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyint_ty],`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L109 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable, ImmArg<ArgIndex<2>>]>;`.
  **L109 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable, ImmArg<ArgIndex<2>>]>;`。
- **L110 EN**: Continues the surrounding expression or declaration: `} // TargetPrefix = "riscv"`.
  **L110 CN**: 继续构造周围的表达式或声明：`} // TargetPrefix = "riscv"`。
- **L111 EN**: Blank line separating nearby declarations or logic blocks.
  **L111 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L112 EN**: Banner comment marking a file or section boundary.
  **L112 CN**: 横幅注释，用于标记文件或章节边界。

### Lines 113-140

````tablegen
// Vectors

// The intrinsic does not have any operand that must be extended.
defvar NoScalarOperand = 0xF;

// The intrinsic does not have a VL operand.
// (e.g., riscv_vmv_x_s and riscv_vfmv_f_s)
defvar NoVLOperand = 0x1F;

class RISCVVIntrinsic {
  // These intrinsics may accept illegal integer values in their llvm_anyint_ty
  // operand, so they have to be extended.
  Intrinsic IntrinsicID = !cast<Intrinsic>(NAME);
  bits<4> ScalarOperand = NoScalarOperand;
  bits<5> VLOperand = NoVLOperand;
  bit IsFPIntrinsic = 0;
}

let TargetPrefix = "riscv" in {
  // We use anyint here but we only support XLen.
  def int_riscv_vsetvli   : DefaultAttrsIntrinsic<[llvm_anyint_ty],
                           /* AVL */  [LLVMMatchType<0>,
                           /* VSEW */  LLVMMatchType<0>,
                           /* VLMUL */ LLVMMatchType<0>],
                                      [IntrNoMem,
                                       ImmArg<ArgIndex<1>>,
                                       ImmArg<ArgIndex<2>>]>;
  def int_riscv_vsetvlimax : DefaultAttrsIntrinsic<[llvm_anyint_ty],
````
- **L113 EN**: Comment explains nearby logic, invariants, or intent: `Vectors`.
  **L113 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vectors`。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L115 EN**: Comment explains nearby logic, invariants, or intent: `The intrinsic does not have any operand that must be extended.`.
  **L115 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The intrinsic does not have any operand that must be extended.`。
- **L116 EN**: Initializes variable `NoScalarOperand` from the right-hand expression.
  **L116 CN**: 使用右侧表达式初始化变量 `NoScalarOperand`。
- **L117 EN**: Blank line separating nearby declarations or logic blocks.
  **L117 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L118 EN**: Comment explains nearby logic, invariants, or intent: `The intrinsic does not have a VL operand.`.
  **L118 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The intrinsic does not have a VL operand.`。
- **L119 EN**: Comment explains nearby logic, invariants, or intent: `(e.g., riscv_vmv_x_s and riscv_vfmv_f_s)`.
  **L119 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(e.g., riscv_vmv_x_s and riscv_vfmv_f_s)`。
- **L120 EN**: Initializes variable `NoVLOperand` from the right-hand expression.
  **L120 CN**: 使用右侧表达式初始化变量 `NoVLOperand`。
- **L121 EN**: Blank line separating nearby declarations or logic blocks.
  **L121 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L122 EN**: Declares class `RISCVVIntrinsic`.
  **L122 CN**: 声明 class `RISCVVIntrinsic`。
- **L123 EN**: Comment explains nearby logic, invariants, or intent: `These intrinsics may accept illegal integer values in their llvm_anyint_ty`.
  **L123 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These intrinsics may accept illegal integer values in their llvm_anyint_ty`。
- **L124 EN**: Comment explains nearby logic, invariants, or intent: `operand, so they have to be extended.`.
  **L124 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`operand, so they have to be extended.`。
- **L125 EN**: Initializes variable `IntrinsicID` from the right-hand expression.
  **L125 CN**: 使用右侧表达式初始化变量 `IntrinsicID`。
- **L126 EN**: Initializes variable `ScalarOperand` from the right-hand expression.
  **L126 CN**: 使用右侧表达式初始化变量 `ScalarOperand`。
- **L127 EN**: Initializes variable `VLOperand` from the right-hand expression.
  **L127 CN**: 使用右侧表达式初始化变量 `VLOperand`。
- **L128 EN**: Initializes variable `IsFPIntrinsic` from the right-hand expression.
  **L128 CN**: 使用右侧表达式初始化变量 `IsFPIntrinsic`。
- **L129 EN**: Closes the current lexical scope or compound statement.
  **L129 CN**: 结束当前词法作用域或复合语句块。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L131 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L131 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L132 EN**: Comment explains nearby logic, invariants, or intent: `We use anyint here but we only support XLen.`.
  **L132 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`We use anyint here but we only support XLen.`。
- **L133 EN**: Declares TableGen def `int_riscv_vsetvli`.
  **L133 CN**: 声明 TableGen def `int_riscv_vsetvli`。
- **L134 EN**: Comment explains nearby logic, invariants, or intent: `AVL */  [LLVMMatchType<0>,`.
  **L134 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AVL */  [LLVMMatchType<0>,`。
- **L135 EN**: Comment explains nearby logic, invariants, or intent: `VSEW */  LLVMMatchType<0>,`.
  **L135 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VSEW */  LLVMMatchType<0>,`。
- **L136 EN**: Comment explains nearby logic, invariants, or intent: `VLMUL */ LLVMMatchType<0>],`.
  **L136 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VLMUL */ LLVMMatchType<0>],`。
- **L137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem,`.
  **L137 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem,`。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<1>>,`.
  **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<1>>,`。
- **L139 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<2>>]>;`.
  **L139 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<2>>]>;`。
- **L140 EN**: Declares TableGen def `int_riscv_vsetvlimax`.
  **L140 CN**: 声明 TableGen def `int_riscv_vsetvlimax`。

### Lines 141-168

````tablegen
                            /* VSEW */ [LLVMMatchType<0>,
                            /* VLMUL */ LLVMMatchType<0>],
                                      [IntrNoMem,
                                       ImmArg<ArgIndex<0>>,
                                       ImmArg<ArgIndex<1>>]>;

  // For unit stride mask load
  // Input: (pointer, vl)
  class RISCVUSMLoad
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [llvm_anyptr_ty, llvm_anyint_ty],
                    [NoCapture<ArgIndex<0>>, IntrReadMem, IntrArgMemOnly]>,
          RISCVVIntrinsic {
    let VLOperand = 1;
  }
  // For unit stride load
  // Input: (passthru, pointer, vl)
  class RISCVUSLoad
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyint_ty],
                    [NoCapture<ArgIndex<1>>, IntrReadMem, IntrArgMemOnly]>,
          RISCVVIntrinsic {
    let VLOperand = 2;
  }
  // For unit stride fault-only-first load
  // Input: (passthru, pointer, vl)
  // Output: (data, vl)
  // NOTE: We model this with default memory properties since we model writing
````
- **L141 EN**: Comment explains nearby logic, invariants, or intent: `VSEW */ [LLVMMatchType<0>,`.
  **L141 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VSEW */ [LLVMMatchType<0>,`。
- **L142 EN**: Comment explains nearby logic, invariants, or intent: `VLMUL */ LLVMMatchType<0>],`.
  **L142 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VLMUL */ LLVMMatchType<0>],`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem,`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem,`。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<0>>,`.
  **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<0>>,`。
- **L145 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<1>>]>;`.
  **L145 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<1>>]>;`。
- **L146 EN**: Blank line separating nearby declarations or logic blocks.
  **L146 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L147 EN**: Comment explains nearby logic, invariants, or intent: `For unit stride mask load`.
  **L147 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unit stride mask load`。
- **L148 EN**: Comment explains nearby logic, invariants, or intent: `Input: (pointer, vl)`.
  **L148 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (pointer, vl)`。
- **L149 EN**: Declares class `RISCVUSMLoad`.
  **L149 CN**: 声明 class `RISCVUSMLoad`。
- **L150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L150 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, llvm_anyint_ty],`.
  **L151 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, llvm_anyint_ty],`。
- **L152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[NoCapture<ArgIndex<0>>, IntrReadMem, IntrArgMemOnly]>,`.
  **L152 CN**: 继续一个多行参数列表、初始化器或聚合项：`[NoCapture<ArgIndex<0>>, IntrReadMem, IntrArgMemOnly]>,`。
- **L153 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L153 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L154 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L154 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L155 EN**: Closes the current lexical scope or compound statement.
  **L155 CN**: 结束当前词法作用域或复合语句块。
- **L156 EN**: Comment explains nearby logic, invariants, or intent: `For unit stride load`.
  **L156 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unit stride load`。
- **L157 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, pointer, vl)`.
  **L157 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, pointer, vl)`。
- **L158 EN**: Declares class `RISCVUSLoad`.
  **L158 CN**: 声明 class `RISCVUSLoad`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyint_ty],`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyint_ty],`。
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[NoCapture<ArgIndex<1>>, IntrReadMem, IntrArgMemOnly]>,`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`[NoCapture<ArgIndex<1>>, IntrReadMem, IntrArgMemOnly]>,`。
- **L162 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L162 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L163 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L163 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L164 EN**: Closes the current lexical scope or compound statement.
  **L164 CN**: 结束当前词法作用域或复合语句块。
- **L165 EN**: Comment explains nearby logic, invariants, or intent: `For unit stride fault-only-first load`.
  **L165 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unit stride fault-only-first load`。
- **L166 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, pointer, vl)`.
  **L166 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, pointer, vl)`。
- **L167 EN**: Comment explains nearby logic, invariants, or intent: `Output: (data, vl)`.
  **L167 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (data, vl)`。
- **L168 EN**: Comment highlights an implementation note: `NOTE: We model this with default memory properties since we model writing`.
  **L168 CN**: 注释强调了一条实现说明：`NOTE: We model this with default memory properties since we model writing`。

### Lines 169-196

````tablegen
  // VL as a side effect. IntrReadMem, IntrHasSideEffects does not work.
  class RISCVUSLoadFF
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty, llvm_anyint_ty],
                    [LLVMMatchType<0>, llvm_anyptr_ty, LLVMMatchType<1>],
                    [NoCapture<ArgIndex<1>>]>,
                    RISCVVIntrinsic {
    let VLOperand = 2;
  }
  // For unit stride load with mask
  // Input: (maskedoff, pointer, mask, vl, policy)
  class RISCVUSLoadMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty ],
                    [LLVMMatchType<0>, llvm_anyptr_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                     llvm_anyint_ty, LLVMMatchType<2>],
                    [NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<4>>, IntrReadMem,
                     IntrArgMemOnly]>,
                    RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For unit stride fault-only-first load with mask
  // Input: (maskedoff, pointer, mask, vl, policy)
  // Output: (data, vl)
  // NOTE: We model this with default memory properties since we model writing
  // VL as a side effect. IntrReadMem, IntrHasSideEffects does not work.
  class RISCVUSLoadFFMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty, llvm_anyint_ty],
                    [LLVMMatchType<0>, llvm_anyptr_ty,
````
- **L169 EN**: Comment explains nearby logic, invariants, or intent: `VL as a side effect. IntrReadMem, IntrHasSideEffects does not work.`.
  **L169 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VL as a side effect. IntrReadMem, IntrHasSideEffects does not work.`。
- **L170 EN**: Declares class `RISCVUSLoadFF`.
  **L170 CN**: 声明 class `RISCVUSLoadFF`。
- **L171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, llvm_anyint_ty],`.
  **L171 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, llvm_anyint_ty],`。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty, LLVMMatchType<1>],`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty, LLVMMatchType<1>],`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[NoCapture<ArgIndex<1>>]>,`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`[NoCapture<ArgIndex<1>>]>,`。
- **L174 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L174 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L175 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L175 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L176 EN**: Closes the current lexical scope or compound statement.
  **L176 CN**: 结束当前词法作用域或复合语句块。
- **L177 EN**: Comment explains nearby logic, invariants, or intent: `For unit stride load with mask`.
  **L177 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unit stride load with mask`。
- **L178 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, pointer, mask, vl, policy)`.
  **L178 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, pointer, mask, vl, policy)`。
- **L179 EN**: Declares class `RISCVUSLoadMasked`.
  **L179 CN**: 声明 class `RISCVUSLoadMasked`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty ],`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty ],`。
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty,`.
  **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty,`。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L183 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<4>>, IntrReadMem,`.
  **L184 CN**: 继续一个多行参数列表、初始化器或聚合项：`[NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<4>>, IntrReadMem,`。
- **L185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrArgMemOnly]>,`.
  **L185 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrArgMemOnly]>,`。
- **L186 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L186 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L187 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L187 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L188 EN**: Closes the current lexical scope or compound statement.
  **L188 CN**: 结束当前词法作用域或复合语句块。
- **L189 EN**: Comment explains nearby logic, invariants, or intent: `For unit stride fault-only-first load with mask`.
  **L189 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unit stride fault-only-first load with mask`。
- **L190 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, pointer, mask, vl, policy)`.
  **L190 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, pointer, mask, vl, policy)`。
- **L191 EN**: Comment explains nearby logic, invariants, or intent: `Output: (data, vl)`.
  **L191 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (data, vl)`。
- **L192 EN**: Comment highlights an implementation note: `NOTE: We model this with default memory properties since we model writing`.
  **L192 CN**: 注释强调了一条实现说明：`NOTE: We model this with default memory properties since we model writing`。
- **L193 EN**: Comment explains nearby logic, invariants, or intent: `VL as a side effect. IntrReadMem, IntrHasSideEffects does not work.`.
  **L193 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VL as a side effect. IntrReadMem, IntrHasSideEffects does not work.`。
- **L194 EN**: Declares class `RISCVUSLoadFFMasked`.
  **L194 CN**: 声明 class `RISCVUSLoadFFMasked`。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, llvm_anyint_ty],`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, llvm_anyint_ty],`。
- **L196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty,`.
  **L196 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty,`。

### Lines 197-224

````tablegen
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                     LLVMMatchType<1>, LLVMMatchType<1>],
                    [NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<4>>]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For strided load with passthru operand
  // Input: (passthru, pointer, stride, vl)
  class RISCVSLoad
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyptr_ty,
                     llvm_anyint_ty, LLVMMatchType<2>],
                    [NoCapture<ArgIndex<1>>, IntrReadMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For strided load with mask
  // Input: (maskedoff, pointer, stride, mask, vl, policy)
  class RISCVSLoadMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty ],
                    [LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyint_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, LLVMMatchType<2>,
                     LLVMMatchType<2>],
                    [NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<5>>, IntrReadMem]>,
                    RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // For indexed load with passthru operand
  // Input: (passthru, pointer, index, vl)
  class RISCVILoad
````
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>, LLVMMatchType<1>],`.
  **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>, LLVMMatchType<1>],`。
- **L199 EN**: Continues the surrounding expression or declaration: `[NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<4>>]>, RISCVVIntrinsic {`.
  **L199 CN**: 继续构造周围的表达式或声明：`[NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<4>>]>, RISCVVIntrinsic {`。
- **L200 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L200 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L201 EN**: Closes the current lexical scope or compound statement.
  **L201 CN**: 结束当前词法作用域或复合语句块。
- **L202 EN**: Comment explains nearby logic, invariants, or intent: `For strided load with passthru operand`.
  **L202 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For strided load with passthru operand`。
- **L203 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, pointer, stride, vl)`.
  **L203 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, pointer, stride, vl)`。
- **L204 EN**: Declares class `RISCVSLoad`.
  **L204 CN**: 声明 class `RISCVSLoad`。
- **L205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L205 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty,`.
  **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty,`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L208 EN**: Continues the surrounding expression or declaration: `[NoCapture<ArgIndex<1>>, IntrReadMem]>, RISCVVIntrinsic {`.
  **L208 CN**: 继续构造周围的表达式或声明：`[NoCapture<ArgIndex<1>>, IntrReadMem]>, RISCVVIntrinsic {`。
- **L209 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L209 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L210 EN**: Closes the current lexical scope or compound statement.
  **L210 CN**: 结束当前词法作用域或复合语句块。
- **L211 EN**: Comment explains nearby logic, invariants, or intent: `For strided load with mask`.
  **L211 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For strided load with mask`。
- **L212 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, pointer, stride, mask, vl, policy)`.
  **L212 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, pointer, stride, mask, vl, policy)`。
- **L213 EN**: Declares class `RISCVSLoadMasked`.
  **L213 CN**: 声明 class `RISCVSLoadMasked`。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty ],`.
  **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty ],`。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyint_ty,`.
  **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyint_ty,`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, LLVMMatchType<2>,`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, LLVMMatchType<2>,`。
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>],`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>],`。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<5>>, IntrReadMem]>,`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`[NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<5>>, IntrReadMem]>,`。
- **L219 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L219 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L220 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L220 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L221 EN**: Closes the current lexical scope or compound statement.
  **L221 CN**: 结束当前词法作用域或复合语句块。
- **L222 EN**: Comment explains nearby logic, invariants, or intent: `For indexed load with passthru operand`.
  **L222 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For indexed load with passthru operand`。
- **L223 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, pointer, index, vl)`.
  **L223 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, pointer, index, vl)`。
- **L224 EN**: Declares class `RISCVILoad`.
  **L224 CN**: 声明 class `RISCVILoad`。

### Lines 225-252

````tablegen
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyptr_ty,
                     llvm_anyvector_ty, llvm_anyint_ty],
                    [NoCapture<ArgIndex<1>>, IntrReadMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For indexed load with mask
  // Input: (maskedoff, pointer, index, mask, vl, policy)
  class RISCVILoadMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty ],
                    [LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyvector_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<3>],
                    [NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<5>>, IntrReadMem]>,
                    RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // For unit stride store
  // Input: (vector_in, pointer, vl)
  class RISCVUSStore
        : DefaultAttrsIntrinsic<[],
                    [llvm_anyvector_ty, llvm_anyptr_ty, llvm_anyint_ty],
                    [NoCapture<ArgIndex<1>>, IntrWriteMem, IntrArgMemOnly]>,
          RISCVVIntrinsic {
    let VLOperand = 2;
  }
  // For unit stride store with mask
  // Input: (vector_in, pointer, mask, vl)
````
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty,`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty,`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, llvm_anyint_ty],`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, llvm_anyint_ty],`。
- **L228 EN**: Continues the surrounding expression or declaration: `[NoCapture<ArgIndex<1>>, IntrReadMem]>, RISCVVIntrinsic {`.
  **L228 CN**: 继续构造周围的表达式或声明：`[NoCapture<ArgIndex<1>>, IntrReadMem]>, RISCVVIntrinsic {`。
- **L229 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L229 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L230 EN**: Closes the current lexical scope or compound statement.
  **L230 CN**: 结束当前词法作用域或复合语句块。
- **L231 EN**: Comment explains nearby logic, invariants, or intent: `For indexed load with mask`.
  **L231 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For indexed load with mask`。
- **L232 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, pointer, index, mask, vl, policy)`.
  **L232 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, pointer, index, mask, vl, policy)`。
- **L233 EN**: Declares class `RISCVILoadMasked`.
  **L233 CN**: 声明 class `RISCVILoadMasked`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty ],`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty ],`。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyvector_ty,`.
  **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyvector_ty,`。
- **L236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L236 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<3>],`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<3>],`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<5>>, IntrReadMem]>,`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`[NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<5>>, IntrReadMem]>,`。
- **L239 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L239 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L240 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L240 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L241 EN**: Closes the current lexical scope or compound statement.
  **L241 CN**: 结束当前词法作用域或复合语句块。
- **L242 EN**: Comment explains nearby logic, invariants, or intent: `For unit stride store`.
  **L242 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unit stride store`。
- **L243 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, pointer, vl)`.
  **L243 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, pointer, vl)`。
- **L244 EN**: Declares class `RISCVUSStore`.
  **L244 CN**: 声明 class `RISCVUSStore`。
- **L245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L245 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_anyptr_ty, llvm_anyint_ty],`.
  **L246 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_anyptr_ty, llvm_anyint_ty],`。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[NoCapture<ArgIndex<1>>, IntrWriteMem, IntrArgMemOnly]>,`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`[NoCapture<ArgIndex<1>>, IntrWriteMem, IntrArgMemOnly]>,`。
- **L248 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L248 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L249 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L249 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L250 EN**: Closes the current lexical scope or compound statement.
  **L250 CN**: 结束当前词法作用域或复合语句块。
- **L251 EN**: Comment explains nearby logic, invariants, or intent: `For unit stride store with mask`.
  **L251 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unit stride store with mask`。
- **L252 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, pointer, mask, vl)`.
  **L252 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, pointer, mask, vl)`。

### Lines 253-280

````tablegen
  class RISCVUSStoreMasked
        : DefaultAttrsIntrinsic<[],
                    [llvm_anyvector_ty, llvm_anyptr_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                     llvm_anyint_ty],
                    [NoCapture<ArgIndex<1>>, IntrWriteMem, IntrArgMemOnly]>,
          RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For strided store
  // Input: (vector_in, pointer, stride, vl)
  class RISCVSStore
        : DefaultAttrsIntrinsic<[],
                    [llvm_anyvector_ty, llvm_anyptr_ty,
                     llvm_anyint_ty, LLVMMatchType<2>],
                    [NoCapture<ArgIndex<1>>, IntrWriteMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For stride store with mask
  // Input: (vector_in, pointer, stirde, mask, vl)
  class RISCVSStoreMasked
        : DefaultAttrsIntrinsic<[],
                    [llvm_anyvector_ty, llvm_anyptr_ty, llvm_anyint_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, LLVMMatchType<2>],
                    [NoCapture<ArgIndex<1>>, IntrWriteMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // For indexed store
````
- **L253 EN**: Declares class `RISCVUSStoreMasked`.
  **L253 CN**: 声明 class `RISCVUSStoreMasked`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_anyptr_ty,`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_anyptr_ty,`。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L257 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[NoCapture<ArgIndex<1>>, IntrWriteMem, IntrArgMemOnly]>,`.
  **L258 CN**: 继续一个多行参数列表、初始化器或聚合项：`[NoCapture<ArgIndex<1>>, IntrWriteMem, IntrArgMemOnly]>,`。
- **L259 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L259 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L260 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L260 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L261 EN**: Closes the current lexical scope or compound statement.
  **L261 CN**: 结束当前词法作用域或复合语句块。
- **L262 EN**: Comment explains nearby logic, invariants, or intent: `For strided store`.
  **L262 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For strided store`。
- **L263 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, pointer, stride, vl)`.
  **L263 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, pointer, stride, vl)`。
- **L264 EN**: Declares class `RISCVSStore`.
  **L264 CN**: 声明 class `RISCVSStore`。
- **L265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L265 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_anyptr_ty,`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_anyptr_ty,`。
- **L267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L267 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L268 EN**: Continues the surrounding expression or declaration: `[NoCapture<ArgIndex<1>>, IntrWriteMem]>, RISCVVIntrinsic {`.
  **L268 CN**: 继续构造周围的表达式或声明：`[NoCapture<ArgIndex<1>>, IntrWriteMem]>, RISCVVIntrinsic {`。
- **L269 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L269 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L270 EN**: Closes the current lexical scope or compound statement.
  **L270 CN**: 结束当前词法作用域或复合语句块。
- **L271 EN**: Comment explains nearby logic, invariants, or intent: `For stride store with mask`.
  **L271 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For stride store with mask`。
- **L272 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, pointer, stirde, mask, vl)`.
  **L272 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, pointer, stirde, mask, vl)`。
- **L273 EN**: Declares class `RISCVSStoreMasked`.
  **L273 CN**: 声明 class `RISCVSStoreMasked`。
- **L274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L274 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_anyptr_ty, llvm_anyint_ty,`.
  **L275 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_anyptr_ty, llvm_anyint_ty,`。
- **L276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, LLVMMatchType<2>],`.
  **L276 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, LLVMMatchType<2>],`。
- **L277 EN**: Continues the surrounding expression or declaration: `[NoCapture<ArgIndex<1>>, IntrWriteMem]>, RISCVVIntrinsic {`.
  **L277 CN**: 继续构造周围的表达式或声明：`[NoCapture<ArgIndex<1>>, IntrWriteMem]>, RISCVVIntrinsic {`。
- **L278 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L278 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L279 EN**: Closes the current lexical scope or compound statement.
  **L279 CN**: 结束当前词法作用域或复合语句块。
- **L280 EN**: Comment explains nearby logic, invariants, or intent: `For indexed store`.
  **L280 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For indexed store`。

### Lines 281-308

````tablegen
  // Input: (vector_in, pointer, index, vl)
  class RISCVIStore
        : DefaultAttrsIntrinsic<[],
                    [llvm_anyvector_ty, llvm_anyptr_ty,
                     llvm_anyint_ty, llvm_anyint_ty],
                    [NoCapture<ArgIndex<1>>, IntrWriteMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For indexed store with mask
  // Input: (vector_in, pointer, index, mask, vl)
  class RISCVIStoreMasked
        : DefaultAttrsIntrinsic<[],
                    [llvm_anyvector_ty, llvm_anyptr_ty, llvm_anyvector_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty],
                    [NoCapture<ArgIndex<1>>, IntrWriteMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // For destination vector type is the same as source vector.
  // Input: (passthru, vector_in, vl)
  class RISCVUnaryAAUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 2;
  }
  // For destination vector type is the same as the source vector type
  // Input: (passthru, vector_in, vl, policy)
  class RISCVUnaryAAUnMaskedZvk<bit IsVS>
````
- **L281 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, pointer, index, vl)`.
  **L281 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, pointer, index, vl)`。
- **L282 EN**: Declares class `RISCVIStore`.
  **L282 CN**: 声明 class `RISCVIStore`。
- **L283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L283 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_anyptr_ty,`.
  **L284 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_anyptr_ty,`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, llvm_anyint_ty],`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, llvm_anyint_ty],`。
- **L286 EN**: Continues the surrounding expression or declaration: `[NoCapture<ArgIndex<1>>, IntrWriteMem]>, RISCVVIntrinsic {`.
  **L286 CN**: 继续构造周围的表达式或声明：`[NoCapture<ArgIndex<1>>, IntrWriteMem]>, RISCVVIntrinsic {`。
- **L287 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L287 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L288 EN**: Closes the current lexical scope or compound statement.
  **L288 CN**: 结束当前词法作用域或复合语句块。
- **L289 EN**: Comment explains nearby logic, invariants, or intent: `For indexed store with mask`.
  **L289 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For indexed store with mask`。
- **L290 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, pointer, index, mask, vl)`.
  **L290 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, pointer, index, mask, vl)`。
- **L291 EN**: Declares class `RISCVIStoreMasked`.
  **L291 CN**: 声明 class `RISCVIStoreMasked`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_anyptr_ty, llvm_anyvector_ty,`.
  **L293 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_anyptr_ty, llvm_anyvector_ty,`。
- **L294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty],`.
  **L294 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty],`。
- **L295 EN**: Continues the surrounding expression or declaration: `[NoCapture<ArgIndex<1>>, IntrWriteMem]>, RISCVVIntrinsic {`.
  **L295 CN**: 继续构造周围的表达式或声明：`[NoCapture<ArgIndex<1>>, IntrWriteMem]>, RISCVVIntrinsic {`。
- **L296 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L296 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L297 EN**: Closes the current lexical scope or compound statement.
  **L297 CN**: 结束当前词法作用域或复合语句块。
- **L298 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as source vector.`.
  **L298 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as source vector.`。
- **L299 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vl)`.
  **L299 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vl)`。
- **L300 EN**: Declares class `RISCVUnaryAAUnMasked`.
  **L300 CN**: 声明 class `RISCVUnaryAAUnMasked`。
- **L301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L301 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty],`.
  **L302 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty],`。
- **L303 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L303 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L304 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L304 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L305 EN**: Closes the current lexical scope or compound statement.
  **L305 CN**: 结束当前词法作用域或复合语句块。
- **L306 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as the source vector type`.
  **L306 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as the source vector type`。
- **L307 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vl, policy)`.
  **L307 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vl, policy)`。
- **L308 EN**: Declares class `RISCVUnaryAAUnMaskedZvk<bit`.
  **L308 CN**: 声明 class `RISCVUnaryAAUnMaskedZvk<bit`。

### Lines 309-336

````tablegen
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, !if(IsVS, llvm_anyvector_ty, LLVMMatchType<0>),
                     llvm_anyint_ty, !if(IsVS, LLVMMatchType<2>, LLVMMatchType<1>)],
                    [ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 2;
  }

  multiclass RISCVUnaryAAUnMaskedZvk<bit HasVV = 1, bit HasVS = 1> {
    if HasVV then
      def "int_riscv_" # NAME # "_vv" : RISCVUnaryAAUnMaskedZvk<IsVS=0>;

    if HasVS then
      def "int_riscv_" # NAME # "_vs" : RISCVUnaryAAUnMaskedZvk<IsVS=1>;
  }
  // For destination vector type is the same as first source vector (with mask).
  // Input: (vector_in, vector_in, mask, vl, policy)
  class RISCVUnaryAAMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<1>],
                    [ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For destination vector type is the same as source vector.
  // Input: (passthru, vector_in, frm, vl)
  class RISCVUnaryAAUnMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
````
- **L309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L309 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, !if(IsVS, llvm_anyvector_ty, LLVMMatchType<0>),`.
  **L310 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, !if(IsVS, llvm_anyvector_ty, LLVMMatchType<0>),`。
- **L311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, !if(IsVS, LLVMMatchType<2>, LLVMMatchType<1>)],`.
  **L311 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, !if(IsVS, LLVMMatchType<2>, LLVMMatchType<1>)],`。
- **L312 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L312 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L313 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L313 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L314 EN**: Closes the current lexical scope or compound statement.
  **L314 CN**: 结束当前词法作用域或复合语句块。
- **L315 EN**: Blank line separating nearby declarations or logic blocks.
  **L315 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L316 EN**: Declares TableGen multiclass `RISCVUnaryAAUnMaskedZvk`.
  **L316 CN**: 声明 TableGen multiclass `RISCVUnaryAAUnMaskedZvk`。
- **L317 EN**: Continues the surrounding expression or declaration: `if HasVV then`.
  **L317 CN**: 继续构造周围的表达式或声明：`if HasVV then`。
- **L318 EN**: Declares TableGen def `"int_riscv_"`.
  **L318 CN**: 声明 TableGen def `"int_riscv_"`。
- **L319 EN**: Blank line separating nearby declarations or logic blocks.
  **L319 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L320 EN**: Continues the surrounding expression or declaration: `if HasVS then`.
  **L320 CN**: 继续构造周围的表达式或声明：`if HasVS then`。
- **L321 EN**: Declares TableGen def `"int_riscv_"`.
  **L321 CN**: 声明 TableGen def `"int_riscv_"`。
- **L322 EN**: Closes the current lexical scope or compound statement.
  **L322 CN**: 结束当前词法作用域或复合语句块。
- **L323 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first source vector (with mask).`.
  **L323 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first source vector (with mask).`。
- **L324 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in, mask, vl, policy)`.
  **L324 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in, mask, vl, policy)`。
- **L325 EN**: Declares class `RISCVUnaryAAMasked`.
  **L325 CN**: 声明 class `RISCVUnaryAAMasked`。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L327 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L328 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>],`.
  **L329 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>],`。
- **L330 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L330 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L331 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L331 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L332 EN**: Closes the current lexical scope or compound statement.
  **L332 CN**: 结束当前词法作用域或复合语句块。
- **L333 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as source vector.`.
  **L333 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as source vector.`。
- **L334 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, frm, vl)`.
  **L334 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, frm, vl)`。
- **L335 EN**: Declares class `RISCVUnaryAAUnMaskedRoundingMode`.
  **L335 CN**: 声明 class `RISCVUnaryAAUnMaskedRoundingMode`。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。

### Lines 337-364

````tablegen
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty, LLVMMatchType<1>],
                    [ImmArg<ArgIndex<2>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For destination vector type is the same as first source vector (with mask).
  // Input: (vector_in, vector_in, mask, frm, vl, policy)
  class RISCVUnaryAAMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<1>, LLVMMatchType<1>],
                    [ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // Input: (passthru, vector_in, mask, vl)
  class RISCVCompress
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For mask logical operations without passthru operand.
  // Input: (vector_in, vector_in, vl)
  class RISCVMaskLogical
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
````
- **L337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty, LLVMMatchType<1>],`.
  **L337 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty, LLVMMatchType<1>],`。
- **L338 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<2>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L338 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<2>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L339 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L339 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L340 EN**: Closes the current lexical scope or compound statement.
  **L340 CN**: 结束当前词法作用域或复合语句块。
- **L341 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first source vector (with mask).`.
  **L341 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first source vector (with mask).`。
- **L342 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in, mask, frm, vl, policy)`.
  **L342 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in, mask, frm, vl, policy)`。
- **L343 EN**: Declares class `RISCVUnaryAAMaskedRoundingMode`.
  **L343 CN**: 声明 class `RISCVUnaryAAMaskedRoundingMode`。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L345 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L346 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>, LLVMMatchType<1>],`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>, LLVMMatchType<1>],`。
- **L348 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L348 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L349 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L349 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L350 EN**: Closes the current lexical scope or compound statement.
  **L350 CN**: 结束当前词法作用域或复合语句块。
- **L351 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, mask, vl)`.
  **L351 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, mask, vl)`。
- **L352 EN**: Declares class `RISCVCompress`.
  **L352 CN**: 声明 class `RISCVCompress`。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty],`.
  **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty],`。
- **L356 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L356 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L357 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L357 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L358 EN**: Closes the current lexical scope or compound statement.
  **L358 CN**: 结束当前词法作用域或复合语句块。
- **L359 EN**: Comment explains nearby logic, invariants, or intent: `For mask logical operations without passthru operand.`.
  **L359 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For mask logical operations without passthru operand.`。
- **L360 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in, vl)`.
  **L360 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in, vl)`。
- **L361 EN**: Declares class `RISCVMaskLogical`.
  **L361 CN**: 声明 class `RISCVMaskLogical`。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty],`.
  **L363 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty],`。
- **L364 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L364 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。

### Lines 365-392

````tablegen
    let VLOperand = 2;
  }
  // For destination vector type is the same as first and second source vector.
  // Input: (passthru, vector_in, vector_in, vl)
  class RISCVBinaryAAAUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>,
                     LLVMMatchType<0>, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For destination vector type is the same as first and second source vector (with mask).
  // Input: (passthru, vector_in, vector_in, mask, vl, policy)
  class RISCVBinaryAAAMasked
       : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                   [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,
                    LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                    LLVMMatchType<1>],
                   [ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // For destination vector type is the same as first and second source vector.
  // Input: (passthru, vector_in, int_vector_in, vl)
  class RISCVRGatherVVUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>,
                     LLVMVectorOfBitcastsToInt<0>, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
````
- **L365 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L365 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L366 EN**: Closes the current lexical scope or compound statement.
  **L366 CN**: 结束当前词法作用域或复合语句块。
- **L367 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first and second source vector.`.
  **L367 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first and second source vector.`。
- **L368 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in, vl)`.
  **L368 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in, vl)`。
- **L369 EN**: Declares class `RISCVBinaryAAAUnMasked`.
  **L369 CN**: 声明 class `RISCVBinaryAAAUnMasked`。
- **L370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L370 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L371 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, llvm_anyint_ty],`.
  **L372 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, llvm_anyint_ty],`。
- **L373 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L373 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L374 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L374 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L375 EN**: Closes the current lexical scope or compound statement.
  **L375 CN**: 结束当前词法作用域或复合语句块。
- **L376 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first and second source vector (with mask).`.
  **L376 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first and second source vector (with mask).`。
- **L377 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in, mask, vl, policy)`.
  **L377 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in, mask, vl, policy)`。
- **L378 EN**: Declares class `RISCVBinaryAAAMasked`.
  **L378 CN**: 声明 class `RISCVBinaryAAAMasked`。
- **L379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L379 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L380 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L381 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>],`.
  **L382 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>],`。
- **L383 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L383 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L384 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L384 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L385 EN**: Closes the current lexical scope or compound statement.
  **L385 CN**: 结束当前词法作用域或复合语句块。
- **L386 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first and second source vector.`.
  **L386 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first and second source vector.`。
- **L387 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, int_vector_in, vl)`.
  **L387 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, int_vector_in, vl)`。
- **L388 EN**: Declares class `RISCVRGatherVVUnMasked`.
  **L388 CN**: 声明 class `RISCVRGatherVVUnMasked`。
- **L389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L389 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L390 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorOfBitcastsToInt<0>, llvm_anyint_ty],`.
  **L391 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorOfBitcastsToInt<0>, llvm_anyint_ty],`。
- **L392 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L392 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。

### Lines 393-420

````tablegen
    let VLOperand = 3;
  }
  // For destination vector type is the same as first and second source vector.
  // Input: (vector_in, vector_in, int_vector_in, vl, policy)
  class RISCVRGatherVVMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, LLVMVectorOfBitcastsToInt<0>,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<1>],
                    [ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // Input: (passthru, vector_in, int16_vector_in, vl)
  class RISCVRGatherEI16VVUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>,
                     LLVMScalarOrSameVectorWidth<0, llvm_i16_ty>,
                     llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For destination vector type is the same as first and second source vector.
  // Input: (vector_in, vector_in, int16_vector_in, vl, policy)
  class RISCVRGatherEI16VVMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>,
                     LLVMScalarOrSameVectorWidth<0, llvm_i16_ty>,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
````
- **L393 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L393 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L394 EN**: Closes the current lexical scope or compound statement.
  **L394 CN**: 结束当前词法作用域或复合语句块。
- **L395 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first and second source vector.`.
  **L395 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first and second source vector.`。
- **L396 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in, int_vector_in, vl, policy)`.
  **L396 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in, int_vector_in, vl, policy)`。
- **L397 EN**: Declares class `RISCVRGatherVVMasked`.
  **L397 CN**: 声明 class `RISCVRGatherVVMasked`。
- **L398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L398 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMVectorOfBitcastsToInt<0>,`.
  **L399 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMVectorOfBitcastsToInt<0>,`。
- **L400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L400 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>],`.
  **L401 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>],`。
- **L402 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L402 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L403 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L403 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L404 EN**: Closes the current lexical scope or compound statement.
  **L404 CN**: 结束当前词法作用域或复合语句块。
- **L405 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, int16_vector_in, vl)`.
  **L405 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, int16_vector_in, vl)`。
- **L406 EN**: Declares class `RISCVRGatherEI16VVUnMasked`.
  **L406 CN**: 声明 class `RISCVRGatherEI16VVUnMasked`。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L408 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i16_ty>,`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i16_ty>,`。
- **L410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L410 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L411 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L411 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L412 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L412 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L413 EN**: Closes the current lexical scope or compound statement.
  **L413 CN**: 结束当前词法作用域或复合语句块。
- **L414 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first and second source vector.`.
  **L414 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first and second source vector.`。
- **L415 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in, int16_vector_in, vl, policy)`.
  **L415 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in, int16_vector_in, vl, policy)`。
- **L416 EN**: Declares class `RISCVRGatherEI16VVMasked`.
  **L416 CN**: 声明 class `RISCVRGatherEI16VVMasked`。
- **L417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L417 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L418 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L418 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i16_ty>,`.
  **L419 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i16_ty>,`。
- **L420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L420 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。

### Lines 421-448

````tablegen
                     LLVMMatchType<1>],
                    [ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // For destination vector type is the same as first source vector, and the
  // second operand is XLen.
  // Input: (passthru, vector_in, xlen_in, vl)
  class RISCVGatherVXUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty,
                     LLVMMatchType<1>],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For destination vector type is the same as first source vector (with mask).
  // Second operand is XLen.
  // Input: (maskedoff, vector_in, xlen_in, mask, vl, policy)
  class RISCVGatherVXMasked
       : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty,
                    LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, LLVMMatchType<1>,
                    LLVMMatchType<1>],
                   [ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // For destination vector type is the same as first source vector.
  // Input: (passthru, vector_in, vector_in/scalar_in, vl)
  class RISCVBinaryAAXUnMasked<bit IsVI = 0>
````
- **L421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>],`.
  **L421 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>],`。
- **L422 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L422 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L423 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L423 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L424 EN**: Closes the current lexical scope or compound statement.
  **L424 CN**: 结束当前词法作用域或复合语句块。
- **L425 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first source vector, and the`.
  **L425 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first source vector, and the`。
- **L426 EN**: Comment explains nearby logic, invariants, or intent: `second operand is XLen.`.
  **L426 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`second operand is XLen.`。
- **L427 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, xlen_in, vl)`.
  **L427 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, xlen_in, vl)`。
- **L428 EN**: Declares class `RISCVGatherVXUnMasked`.
  **L428 CN**: 声明 class `RISCVGatherVXUnMasked`。
- **L429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L429 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty,`.
  **L430 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty,`。
- **L431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>],`.
  **L431 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>],`。
- **L432 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L432 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L433 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L433 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L434 EN**: Closes the current lexical scope or compound statement.
  **L434 CN**: 结束当前词法作用域或复合语句块。
- **L435 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first source vector (with mask).`.
  **L435 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first source vector (with mask).`。
- **L436 EN**: Comment explains nearby logic, invariants, or intent: `Second operand is XLen.`.
  **L436 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Second operand is XLen.`。
- **L437 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, xlen_in, mask, vl, policy)`.
  **L437 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, xlen_in, mask, vl, policy)`。
- **L438 EN**: Declares class `RISCVGatherVXMasked`.
  **L438 CN**: 声明 class `RISCVGatherVXMasked`。
- **L439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L439 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty,`.
  **L440 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty,`。
- **L441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, LLVMMatchType<1>,`.
  **L441 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, LLVMMatchType<1>,`。
- **L442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>],`.
  **L442 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>],`。
- **L443 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L443 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L444 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L444 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L445 EN**: Closes the current lexical scope or compound statement.
  **L445 CN**: 结束当前词法作用域或复合语句块。
- **L446 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first source vector.`.
  **L446 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first source vector.`。
- **L447 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in/scalar_in, vl)`.
  **L447 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in/scalar_in, vl)`。
- **L448 EN**: Declares class `RISCVBinaryAAXUnMasked<bit`.
  **L448 CN**: 声明 class `RISCVBinaryAAXUnMasked<bit`。

### Lines 449-476

````tablegen
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,
                     llvm_anyint_ty],
                    !listconcat([IntrNoMem],
                                !if(IsVI, [ImmArg<ArgIndex<2>>], []))>,
                    RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 3;
  }
  // For destination vector type is the same as the source vector type.
  // Input: (passthru, vector_in, vector_in/scalar_in, vl, policy)
  class RISCVBinaryAAXUnMaskedZvk<bit IsVI = 0>
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                                [LLVMMatchType<0>, LLVMMatchType<0>,
                                 llvm_any_ty, llvm_anyint_ty, LLVMMatchType<2>],
                                !listconcat([ImmArg<ArgIndex<4>>, IntrNoMem],
                                            !if(IsVI, [ImmArg<ArgIndex<2>>], []))>,
                                RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 3;
  }
  // For destination vector type is the same as first source vector (with mask).
  // Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)
  class RISCVBinaryAAXMasked
       : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,
                    LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                    LLVMMatchType<2>],
````
- **L449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L449 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`.
  **L450 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`。
- **L451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L451 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([IntrNoMem],`.
  **L452 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([IntrNoMem],`。
- **L453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(IsVI, [ImmArg<ArgIndex<2>>], []))>,`.
  **L453 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(IsVI, [ImmArg<ArgIndex<2>>], []))>,`。
- **L454 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L454 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L455 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L455 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L456 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L456 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L457 EN**: Closes the current lexical scope or compound statement.
  **L457 CN**: 结束当前词法作用域或复合语句块。
- **L458 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as the source vector type.`.
  **L458 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as the source vector type.`。
- **L459 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in/scalar_in, vl, policy)`.
  **L459 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in/scalar_in, vl, policy)`。
- **L460 EN**: Declares class `RISCVBinaryAAXUnMaskedZvk<bit`.
  **L460 CN**: 声明 class `RISCVBinaryAAXUnMaskedZvk<bit`。
- **L461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L461 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L462 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L463 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_any_ty, llvm_anyint_ty, LLVMMatchType<2>],`.
  **L463 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_any_ty, llvm_anyint_ty, LLVMMatchType<2>],`。
- **L464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([ImmArg<ArgIndex<4>>, IntrNoMem],`.
  **L464 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([ImmArg<ArgIndex<4>>, IntrNoMem],`。
- **L465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(IsVI, [ImmArg<ArgIndex<2>>], []))>,`.
  **L465 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(IsVI, [ImmArg<ArgIndex<2>>], []))>,`。
- **L466 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L466 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L467 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L467 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L468 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L468 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L469 EN**: Closes the current lexical scope or compound statement.
  **L469 CN**: 结束当前词法作用域或复合语句块。
- **L470 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first source vector (with mask).`.
  **L470 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first source vector (with mask).`。
- **L471 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)`.
  **L471 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)`。
- **L472 EN**: Declares class `RISCVBinaryAAXMasked`.
  **L472 CN**: 声明 class `RISCVBinaryAAXMasked`。
- **L473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L473 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`.
  **L474 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`。
- **L475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L475 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>],`.
  **L476 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>],`。

### Lines 477-504

````tablegen
                   [ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 4;
  }
  // For destination vector type is the same as first source vector.
  // Input: (passthru, vector_in, vector_in/scalar_in, frm, vl)
  class RISCVBinaryAAXUnMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,
                     llvm_anyint_ty, LLVMMatchType<2>],
                    [ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 4;
  }
  // For destination vector type is the same as first source vector (with mask).
  // Input: (maskedoff, vector_in, vector_in/scalar_in, mask, frm, vl, policy)
  class RISCVBinaryAAXMaskedRoundingMode
       : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,
                    LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                    LLVMMatchType<2>, LLVMMatchType<2>],
                   [ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 5;
  }
  // For destination vector type is the same as first source vector. The
  // second source operand must match the destination type or be an XLen scalar.
  // Input: (passthru, vector_in, vector_in/scalar_in, vl)
````
- **L477 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L477 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L478 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L478 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L479 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L479 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L480 EN**: Closes the current lexical scope or compound statement.
  **L480 CN**: 结束当前词法作用域或复合语句块。
- **L481 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first source vector.`.
  **L481 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first source vector.`。
- **L482 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in/scalar_in, frm, vl)`.
  **L482 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in/scalar_in, frm, vl)`。
- **L483 EN**: Declares class `RISCVBinaryAAXUnMaskedRoundingMode`.
  **L483 CN**: 声明 class `RISCVBinaryAAXUnMaskedRoundingMode`。
- **L484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L484 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`.
  **L485 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`。
- **L486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L486 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L487 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L487 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L488 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L488 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L489 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L489 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L490 EN**: Closes the current lexical scope or compound statement.
  **L490 CN**: 结束当前词法作用域或复合语句块。
- **L491 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first source vector (with mask).`.
  **L491 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first source vector (with mask).`。
- **L492 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, vector_in/scalar_in, mask, frm, vl, policy)`.
  **L492 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, vector_in/scalar_in, mask, frm, vl, policy)`。
- **L493 EN**: Declares class `RISCVBinaryAAXMaskedRoundingMode`.
  **L493 CN**: 声明 class `RISCVBinaryAAXMaskedRoundingMode`。
- **L494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L494 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`.
  **L495 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`。
- **L496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L496 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>, LLVMMatchType<2>],`.
  **L497 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>, LLVMMatchType<2>],`。
- **L498 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L498 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L499 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L499 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L500 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L500 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L501 EN**: Closes the current lexical scope or compound statement.
  **L501 CN**: 结束当前词法作用域或复合语句块。
- **L502 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first source vector. The`.
  **L502 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first source vector. The`。
- **L503 EN**: Comment explains nearby logic, invariants, or intent: `second source operand must match the destination type or be an XLen scalar.`.
  **L503 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`second source operand must match the destination type or be an XLen scalar.`。
- **L504 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in/scalar_in, vl)`.
  **L504 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in/scalar_in, vl)`。

### Lines 505-532

````tablegen
  class RISCVBinaryAAShiftUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,
                     llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For destination vector type is the same as first source vector (with mask).
  // The second source operand must match the destination type or be an XLen scalar.
  // Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)
  class RISCVBinaryAAShiftMasked
       : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,
                    LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                    LLVMMatchType<2>],
                   [ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // For destination vector type is NOT the same as first source vector.
  // Input: (passthru, vector_in, vector_in/scalar_in, vl)
  class RISCVBinaryABXUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,
                     llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 3;
  }
````
- **L505 EN**: Declares class `RISCVBinaryAAShiftUnMasked`.
  **L505 CN**: 声明 class `RISCVBinaryAAShiftUnMasked`。
- **L506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L506 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`.
  **L507 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`。
- **L508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L508 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L509 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L509 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L510 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L510 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L511 EN**: Closes the current lexical scope or compound statement.
  **L511 CN**: 结束当前词法作用域或复合语句块。
- **L512 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first source vector (with mask).`.
  **L512 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first source vector (with mask).`。
- **L513 EN**: Comment explains nearby logic, invariants, or intent: `The second source operand must match the destination type or be an XLen scalar.`.
  **L513 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The second source operand must match the destination type or be an XLen scalar.`。
- **L514 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)`.
  **L514 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)`。
- **L515 EN**: Declares class `RISCVBinaryAAShiftMasked`.
  **L515 CN**: 声明 class `RISCVBinaryAAShiftMasked`。
- **L516 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L516 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`.
  **L517 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`。
- **L518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L518 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>],`.
  **L519 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>],`。
- **L520 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L520 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L521 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L521 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L522 EN**: Closes the current lexical scope or compound statement.
  **L522 CN**: 结束当前词法作用域或复合语句块。
- **L523 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is NOT the same as first source vector.`.
  **L523 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is NOT the same as first source vector.`。
- **L524 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in/scalar_in, vl)`.
  **L524 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in/scalar_in, vl)`。
- **L525 EN**: Declares class `RISCVBinaryABXUnMasked`.
  **L525 CN**: 声明 class `RISCVBinaryABXUnMasked`。
- **L526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L526 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`.
  **L527 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`。
- **L528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L528 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L529 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L529 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L530 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L530 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L531 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L531 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L532 EN**: Closes the current lexical scope or compound statement.
  **L532 CN**: 结束当前词法作用域或复合语句块。

### Lines 533-560

````tablegen
  // For destination vector type is NOT the same as first source vector (with mask).
  // Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)
  class RISCVBinaryABXMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<3>],
                    [ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 4;
  }
  // For destination vector type is NOT the same as first source vector.
  // Input: (passthru, vector_in, vector_in/scalar_in, frm, vl)
  class RISCVBinaryABXUnMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,
                     llvm_anyint_ty, LLVMMatchType<3>],
                    [ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 4;
  }
  // For destination vector type is NOT the same as first source vector (with mask).
  // Input: (maskedoff, vector_in, vector_in/scalar_in, mask, frm, vl, policy)
  class RISCVBinaryABXMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<3>, LLVMMatchType<3>],
````
- **L533 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is NOT the same as first source vector (with mask).`.
  **L533 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is NOT the same as first source vector (with mask).`。
- **L534 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)`.
  **L534 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)`。
- **L535 EN**: Declares class `RISCVBinaryABXMasked`.
  **L535 CN**: 声明 class `RISCVBinaryABXMasked`。
- **L536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L536 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`.
  **L537 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`。
- **L538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L538 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<3>],`.
  **L539 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<3>],`。
- **L540 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L540 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L541 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L541 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L542 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L542 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L543 EN**: Closes the current lexical scope or compound statement.
  **L543 CN**: 结束当前词法作用域或复合语句块。
- **L544 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is NOT the same as first source vector.`.
  **L544 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is NOT the same as first source vector.`。
- **L545 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in/scalar_in, frm, vl)`.
  **L545 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in/scalar_in, frm, vl)`。
- **L546 EN**: Declares class `RISCVBinaryABXUnMaskedRoundingMode`.
  **L546 CN**: 声明 class `RISCVBinaryABXUnMaskedRoundingMode`。
- **L547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L547 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`.
  **L548 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`。
- **L549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<3>],`.
  **L549 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<3>],`。
- **L550 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L550 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L551 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L551 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L552 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L552 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is NOT the same as first source vector (with mask).`.
  **L554 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is NOT the same as first source vector (with mask).`。
- **L555 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, vector_in/scalar_in, mask, frm, vl, policy)`.
  **L555 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, vector_in/scalar_in, mask, frm, vl, policy)`。
- **L556 EN**: Declares class `RISCVBinaryABXMaskedRoundingMode`.
  **L556 CN**: 声明 class `RISCVBinaryABXMaskedRoundingMode`。
- **L557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L557 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`.
  **L558 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`。
- **L559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L559 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<3>, LLVMMatchType<3>],`.
  **L560 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<3>, LLVMMatchType<3>],`。

### Lines 561-588

````tablegen
                    [ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 5;
  }
  // For destination vector type is NOT the same as first source vector. The
  // second source operand must match the destination type or be an XLen scalar.
  // Input: (passthru, vector_in, vector_in/scalar_in, vl)
  class RISCVBinaryABShiftUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,
                     llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For destination vector type is NOT the same as first source vector (with mask).
  // The second source operand must match the destination type or be an XLen scalar.
  // Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)
  class RISCVBinaryABShiftMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<3>],
                    [ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // For binary operations with V0 as input.
  // Input: (passthru, vector_in, vector_in/scalar_in, V0, vl)
  class RISCVBinaryWithV0
````
- **L561 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L561 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L562 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L562 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L563 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L563 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L564 EN**: Closes the current lexical scope or compound statement.
  **L564 CN**: 结束当前词法作用域或复合语句块。
- **L565 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is NOT the same as first source vector. The`.
  **L565 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is NOT the same as first source vector. The`。
- **L566 EN**: Comment explains nearby logic, invariants, or intent: `second source operand must match the destination type or be an XLen scalar.`.
  **L566 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`second source operand must match the destination type or be an XLen scalar.`。
- **L567 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in/scalar_in, vl)`.
  **L567 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in/scalar_in, vl)`。
- **L568 EN**: Declares class `RISCVBinaryABShiftUnMasked`.
  **L568 CN**: 声明 class `RISCVBinaryABShiftUnMasked`。
- **L569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L569 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`.
  **L570 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`。
- **L571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L571 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L572 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L572 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L573 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L573 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L574 EN**: Closes the current lexical scope or compound statement.
  **L574 CN**: 结束当前词法作用域或复合语句块。
- **L575 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is NOT the same as first source vector (with mask).`.
  **L575 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is NOT the same as first source vector (with mask).`。
- **L576 EN**: Comment explains nearby logic, invariants, or intent: `The second source operand must match the destination type or be an XLen scalar.`.
  **L576 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The second source operand must match the destination type or be an XLen scalar.`。
- **L577 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)`.
  **L577 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)`。
- **L578 EN**: Declares class `RISCVBinaryABShiftMasked`.
  **L578 CN**: 声明 class `RISCVBinaryABShiftMasked`。
- **L579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L579 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`.
  **L580 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`。
- **L581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L581 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<3>],`.
  **L582 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<3>],`。
- **L583 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L583 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L584 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L584 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L585 EN**: Closes the current lexical scope or compound statement.
  **L585 CN**: 结束当前词法作用域或复合语句块。
- **L586 EN**: Comment explains nearby logic, invariants, or intent: `For binary operations with V0 as input.`.
  **L586 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For binary operations with V0 as input.`。
- **L587 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in/scalar_in, V0, vl)`.
  **L587 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in/scalar_in, V0, vl)`。
- **L588 EN**: Declares class `RISCVBinaryWithV0`.
  **L588 CN**: 声明 class `RISCVBinaryWithV0`。

### Lines 589-616

````tablegen
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                     llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 4;
  }
  // For binary operations with mask type output and V0 as input.
  // Output: (mask type output)
  // Input: (vector_in, vector_in/scalar_in, V0, vl)
  class RISCVBinaryMOutWithV0
        :DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],
                   [llvm_anyvector_ty, llvm_any_ty,
                    LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                    llvm_anyint_ty],
                   [IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 1;
    let VLOperand = 3;
  }
  // For binary operations with mask type output.
  // Output: (mask type output)
  // Input: (vector_in, vector_in/scalar_in, vl)
  class RISCVBinaryMOut
        : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],
                    [llvm_anyvector_ty, llvm_any_ty, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 1;
````
- **L589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L589 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`.
  **L590 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`。
- **L591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L591 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L592 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L593 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L593 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L594 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L594 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L595 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L595 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L596 EN**: Closes the current lexical scope or compound statement.
  **L596 CN**: 结束当前词法作用域或复合语句块。
- **L597 EN**: Comment explains nearby logic, invariants, or intent: `For binary operations with mask type output and V0 as input.`.
  **L597 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For binary operations with mask type output and V0 as input.`。
- **L598 EN**: Comment explains nearby logic, invariants, or intent: `Output: (mask type output)`.
  **L598 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (mask type output)`。
- **L599 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in/scalar_in, V0, vl)`.
  **L599 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in/scalar_in, V0, vl)`。
- **L600 EN**: Declares class `RISCVBinaryMOutWithV0`.
  **L600 CN**: 声明 class `RISCVBinaryMOutWithV0`。
- **L601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `:DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`.
  **L601 CN**: 继续一个多行参数列表、初始化器或聚合项：`:DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`。
- **L602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_any_ty,`.
  **L602 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_any_ty,`。
- **L603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L603 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L604 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L605 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L605 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L606 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L606 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L607 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L607 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L608 EN**: Closes the current lexical scope or compound statement.
  **L608 CN**: 结束当前词法作用域或复合语句块。
- **L609 EN**: Comment explains nearby logic, invariants, or intent: `For binary operations with mask type output.`.
  **L609 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For binary operations with mask type output.`。
- **L610 EN**: Comment explains nearby logic, invariants, or intent: `Output: (mask type output)`.
  **L610 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (mask type output)`。
- **L611 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in/scalar_in, vl)`.
  **L611 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in/scalar_in, vl)`。
- **L612 EN**: Declares class `RISCVBinaryMOut`.
  **L612 CN**: 声明 class `RISCVBinaryMOut`。
- **L613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`.
  **L613 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_any_ty, llvm_anyint_ty],`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_any_ty, llvm_anyint_ty],`。
- **L615 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L615 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L616 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L616 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 617-644

````tablegen
    let VLOperand = 2;
  }
  // For binary operations with mask type output without mask.
  // Output: (mask type output)
  // Input: (vector_in, vector_in/scalar_in, vl)
  class RISCVCompareUnMasked
        : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],
                    [llvm_anyvector_ty, llvm_any_ty, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 1;
    let VLOperand = 2;
  }
  // For binary operations with mask type output with mask.
  // Output: (mask type output)
  // Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl)
  class RISCVCompareMasked
        : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],
                    [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                     llvm_anyvector_ty, llvm_any_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 4;
  }
  // For FP classify operations.
  // Output: (bit mask type output)
  // Input: (passthru, vector_in, vl)
  class RISCVClassifyUnMasked
````
- **L617 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L617 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L618 EN**: Closes the current lexical scope or compound statement.
  **L618 CN**: 结束当前词法作用域或复合语句块。
- **L619 EN**: Comment explains nearby logic, invariants, or intent: `For binary operations with mask type output without mask.`.
  **L619 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For binary operations with mask type output without mask.`。
- **L620 EN**: Comment explains nearby logic, invariants, or intent: `Output: (mask type output)`.
  **L620 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (mask type output)`。
- **L621 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in/scalar_in, vl)`.
  **L621 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in/scalar_in, vl)`。
- **L622 EN**: Declares class `RISCVCompareUnMasked`.
  **L622 CN**: 声明 class `RISCVCompareUnMasked`。
- **L623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`.
  **L623 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`。
- **L624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_any_ty, llvm_anyint_ty],`.
  **L624 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_any_ty, llvm_anyint_ty],`。
- **L625 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L625 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L626 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L626 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L627 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L627 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L628 EN**: Closes the current lexical scope or compound statement.
  **L628 CN**: 结束当前词法作用域或复合语句块。
- **L629 EN**: Comment explains nearby logic, invariants, or intent: `For binary operations with mask type output with mask.`.
  **L629 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For binary operations with mask type output with mask.`。
- **L630 EN**: Comment explains nearby logic, invariants, or intent: `Output: (mask type output)`.
  **L630 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (mask type output)`。
- **L631 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl)`.
  **L631 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl)`。
- **L632 EN**: Declares class `RISCVCompareMasked`.
  **L632 CN**: 声明 class `RISCVCompareMasked`。
- **L633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`.
  **L633 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`。
- **L634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L634 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, llvm_any_ty,`.
  **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, llvm_any_ty,`。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty],`.
  **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty],`。
- **L637 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L637 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L638 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L638 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L639 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L639 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L640 EN**: Closes the current lexical scope or compound statement.
  **L640 CN**: 结束当前词法作用域或复合语句块。
- **L641 EN**: Comment explains nearby logic, invariants, or intent: `For FP classify operations.`.
  **L641 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For FP classify operations.`。
- **L642 EN**: Comment explains nearby logic, invariants, or intent: `Output: (bit mask type output)`.
  **L642 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (bit mask type output)`。
- **L643 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vl)`.
  **L643 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vl)`。
- **L644 EN**: Declares class `RISCVClassifyUnMasked`.
  **L644 CN**: 声明 class `RISCVClassifyUnMasked`。

### Lines 645-672

````tablegen
        : DefaultAttrsIntrinsic<[LLVMVectorOfBitcastsToInt<0>],
                    [LLVMVectorOfBitcastsToInt<0>, llvm_anyvector_ty,
                      llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 1;
  }
  // For FP classify operations with mask.
  // Output: (bit mask type output)
  // Input: (maskedoff, vector_in, mask, vl, policy)
  class RISCVClassifyMasked
        : DefaultAttrsIntrinsic<[LLVMVectorOfBitcastsToInt<0>],
                    [LLVMVectorOfBitcastsToInt<0>, llvm_anyvector_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, 
                     llvm_anyint_ty, LLVMMatchType<1>],
                    [IntrNoMem, ImmArg<ArgIndex<4>>]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For Saturating binary operations.
  // The destination vector type is the same as first source vector.
  // Input: (passthru, vector_in, vector_in/scalar_in, vl)
  class RISCVSaturatingBinaryAAXUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,
                     llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 3;
  }
````
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMVectorOfBitcastsToInt<0>],`.
  **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMVectorOfBitcastsToInt<0>],`。
- **L646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMVectorOfBitcastsToInt<0>, llvm_anyvector_ty,`.
  **L646 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMVectorOfBitcastsToInt<0>, llvm_anyvector_ty,`。
- **L647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L647 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L648 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L648 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L649 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L649 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L650 EN**: Closes the current lexical scope or compound statement.
  **L650 CN**: 结束当前词法作用域或复合语句块。
- **L651 EN**: Comment explains nearby logic, invariants, or intent: `For FP classify operations with mask.`.
  **L651 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For FP classify operations with mask.`。
- **L652 EN**: Comment explains nearby logic, invariants, or intent: `Output: (bit mask type output)`.
  **L652 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (bit mask type output)`。
- **L653 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, mask, vl, policy)`.
  **L653 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, mask, vl, policy)`。
- **L654 EN**: Declares class `RISCVClassifyMasked`.
  **L654 CN**: 声明 class `RISCVClassifyMasked`。
- **L655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMVectorOfBitcastsToInt<0>],`.
  **L655 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMVectorOfBitcastsToInt<0>],`。
- **L656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMVectorOfBitcastsToInt<0>, llvm_anyvector_ty,`.
  **L656 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMVectorOfBitcastsToInt<0>, llvm_anyvector_ty,`。
- **L657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L657 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<1>],`.
  **L658 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<1>],`。
- **L659 EN**: Continues the surrounding expression or declaration: `[IntrNoMem, ImmArg<ArgIndex<4>>]>, RISCVVIntrinsic {`.
  **L659 CN**: 继续构造周围的表达式或声明：`[IntrNoMem, ImmArg<ArgIndex<4>>]>, RISCVVIntrinsic {`。
- **L660 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L660 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L661 EN**: Closes the current lexical scope or compound statement.
  **L661 CN**: 结束当前词法作用域或复合语句块。
- **L662 EN**: Comment explains nearby logic, invariants, or intent: `For Saturating binary operations.`.
  **L662 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Saturating binary operations.`。
- **L663 EN**: Comment explains nearby logic, invariants, or intent: `The destination vector type is the same as first source vector.`.
  **L663 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The destination vector type is the same as first source vector.`。
- **L664 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in/scalar_in, vl)`.
  **L664 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in/scalar_in, vl)`。
- **L665 EN**: Declares class `RISCVSaturatingBinaryAAXUnMasked`.
  **L665 CN**: 声明 class `RISCVSaturatingBinaryAAXUnMasked`。
- **L666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L666 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`.
  **L667 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`。
- **L668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L668 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L669 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L669 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L670 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L670 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L671 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L671 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L672 EN**: Closes the current lexical scope or compound statement.
  **L672 CN**: 结束当前词法作用域或复合语句块。

### Lines 673-700

````tablegen
  // For Saturating binary operations with rounding-mode operand
  // The destination vector type is the same as first source vector.
  // Input: (passthru, vector_in, vector_in/scalar_in, vxrm, vl)
  class RISCVSaturatingBinaryAAXUnMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,
                     llvm_anyint_ty, LLVMMatchType<2>],
                    [ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 4;
  }
  // For Saturating binary operations with mask.
  // The destination vector type is the same as first source vector.
  // Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)
  class RISCVSaturatingBinaryAAXMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<2>],
                    [ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 4;
  }
  // For Saturating binary operations with mask and rounding-mode operand
  // The destination vector type is the same as first source vector.
  // Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vxrm, vl, policy)
  class RISCVSaturatingBinaryAAXMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
````
- **L673 EN**: Comment explains nearby logic, invariants, or intent: `For Saturating binary operations with rounding-mode operand`.
  **L673 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Saturating binary operations with rounding-mode operand`。
- **L674 EN**: Comment explains nearby logic, invariants, or intent: `The destination vector type is the same as first source vector.`.
  **L674 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The destination vector type is the same as first source vector.`。
- **L675 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in/scalar_in, vxrm, vl)`.
  **L675 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in/scalar_in, vxrm, vl)`。
- **L676 EN**: Declares class `RISCVSaturatingBinaryAAXUnMaskedRoundingMode`.
  **L676 CN**: 声明 class `RISCVSaturatingBinaryAAXUnMaskedRoundingMode`。
- **L677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L677 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`.
  **L678 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`。
- **L679 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L679 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L680 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L680 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L681 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L681 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L682 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L682 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L683 EN**: Closes the current lexical scope or compound statement.
  **L683 CN**: 结束当前词法作用域或复合语句块。
- **L684 EN**: Comment explains nearby logic, invariants, or intent: `For Saturating binary operations with mask.`.
  **L684 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Saturating binary operations with mask.`。
- **L685 EN**: Comment explains nearby logic, invariants, or intent: `The destination vector type is the same as first source vector.`.
  **L685 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The destination vector type is the same as first source vector.`。
- **L686 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)`.
  **L686 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vl, policy)`。
- **L687 EN**: Declares class `RISCVSaturatingBinaryAAXMasked`.
  **L687 CN**: 声明 class `RISCVSaturatingBinaryAAXMasked`。
- **L688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L688 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`.
  **L689 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`。
- **L690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L690 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>],`.
  **L691 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>],`。
- **L692 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L692 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L693 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L693 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L694 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L694 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L695 EN**: Closes the current lexical scope or compound statement.
  **L695 CN**: 结束当前词法作用域或复合语句块。
- **L696 EN**: Comment explains nearby logic, invariants, or intent: `For Saturating binary operations with mask and rounding-mode operand`.
  **L696 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Saturating binary operations with mask and rounding-mode operand`。
- **L697 EN**: Comment explains nearby logic, invariants, or intent: `The destination vector type is the same as first source vector.`.
  **L697 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The destination vector type is the same as first source vector.`。
- **L698 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vxrm, vl, policy)`.
  **L698 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vxrm, vl, policy)`。
- **L699 EN**: Declares class `RISCVSaturatingBinaryAAXMaskedRoundingMode`.
  **L699 CN**: 声明 class `RISCVSaturatingBinaryAAXMaskedRoundingMode`。
- **L700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L700 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。

### Lines 701-728

````tablegen
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<2>, LLVMMatchType<2>],
                    [ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 5;
  }
  // For Saturating binary operations.
  // The destination vector type is the same as first source vector.
  // The second source operand matches the destination type or is an XLen scalar.
  // Input: (passthru, vector_in, vector_in/scalar_in, vxrm, vl)
  class RISCVSaturatingBinaryAAShiftUnMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,
                     llvm_anyint_ty, LLVMMatchType<2>],
                    [ImmArg<ArgIndex<3>>, IntrNoMem]>,
                    RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // For Saturating binary operations with mask.
  // The destination vector type is the same as first source vector.
  // The second source operand matches the destination type or is an XLen scalar.
  // Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vxrm, vl, policy)
  class RISCVSaturatingBinaryAAShiftMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<2>, LLVMMatchType<2>],
````
- **L701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`.
  **L701 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`。
- **L702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L702 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>, LLVMMatchType<2>],`.
  **L703 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>, LLVMMatchType<2>],`。
- **L704 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L704 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L705 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L705 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L706 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L706 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L707 EN**: Closes the current lexical scope or compound statement.
  **L707 CN**: 结束当前词法作用域或复合语句块。
- **L708 EN**: Comment explains nearby logic, invariants, or intent: `For Saturating binary operations.`.
  **L708 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Saturating binary operations.`。
- **L709 EN**: Comment explains nearby logic, invariants, or intent: `The destination vector type is the same as first source vector.`.
  **L709 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The destination vector type is the same as first source vector.`。
- **L710 EN**: Comment explains nearby logic, invariants, or intent: `The second source operand matches the destination type or is an XLen scalar.`.
  **L710 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The second source operand matches the destination type or is an XLen scalar.`。
- **L711 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in/scalar_in, vxrm, vl)`.
  **L711 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in/scalar_in, vxrm, vl)`。
- **L712 EN**: Declares class `RISCVSaturatingBinaryAAShiftUnMaskedRoundingMode`.
  **L712 CN**: 声明 class `RISCVSaturatingBinaryAAShiftUnMaskedRoundingMode`。
- **L713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L713 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L714 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`.
  **L714 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`。
- **L715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L715 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<3>>, IntrNoMem]>,`.
  **L716 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<3>>, IntrNoMem]>,`。
- **L717 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L717 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L718 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L718 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L719 EN**: Closes the current lexical scope or compound statement.
  **L719 CN**: 结束当前词法作用域或复合语句块。
- **L720 EN**: Comment explains nearby logic, invariants, or intent: `For Saturating binary operations with mask.`.
  **L720 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Saturating binary operations with mask.`。
- **L721 EN**: Comment explains nearby logic, invariants, or intent: `The destination vector type is the same as first source vector.`.
  **L721 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The destination vector type is the same as first source vector.`。
- **L722 EN**: Comment explains nearby logic, invariants, or intent: `The second source operand matches the destination type or is an XLen scalar.`.
  **L722 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The second source operand matches the destination type or is an XLen scalar.`。
- **L723 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vxrm, vl, policy)`.
  **L723 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vxrm, vl, policy)`。
- **L724 EN**: Declares class `RISCVSaturatingBinaryAAShiftMaskedRoundingMode`.
  **L724 CN**: 声明 class `RISCVSaturatingBinaryAAShiftMaskedRoundingMode`。
- **L725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L725 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`.
  **L726 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_any_ty,`。
- **L727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L727 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>, LLVMMatchType<2>],`.
  **L728 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>, LLVMMatchType<2>],`。

### Lines 729-756

````tablegen
                    [ImmArg<ArgIndex<4>>,ImmArg<ArgIndex<6>>, IntrNoMem]>,
                    RISCVVIntrinsic {
    let VLOperand = 5;
  }
  // For Saturating binary operations.
  // The destination vector type is NOT the same as first source vector.
  // The second source operand matches the destination type or is an XLen scalar.
  // Input: (passthru, vector_in, vector_in/scalar_in, vxrm, vl)
  class RISCVSaturatingBinaryABShiftUnMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,
                     llvm_anyint_ty, LLVMMatchType<3>],
                    [ImmArg<ArgIndex<3>>, IntrNoMem]>,
                    RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // For Saturating binary operations with mask.
  // The destination vector type is NOT the same as first source vector (with mask).
  // The second source operand matches the destination type or is an XLen scalar.
  // Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vxrm, vl, policy)
  class RISCVSaturatingBinaryABShiftMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<3>, LLVMMatchType<3>],
                    [ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 5;
  }
````
- **L729 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<4>>,ImmArg<ArgIndex<6>>, IntrNoMem]>,`.
  **L729 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<4>>,ImmArg<ArgIndex<6>>, IntrNoMem]>,`。
- **L730 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L730 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L731 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L731 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L732 EN**: Closes the current lexical scope or compound statement.
  **L732 CN**: 结束当前词法作用域或复合语句块。
- **L733 EN**: Comment explains nearby logic, invariants, or intent: `For Saturating binary operations.`.
  **L733 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Saturating binary operations.`。
- **L734 EN**: Comment explains nearby logic, invariants, or intent: `The destination vector type is NOT the same as first source vector.`.
  **L734 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The destination vector type is NOT the same as first source vector.`。
- **L735 EN**: Comment explains nearby logic, invariants, or intent: `The second source operand matches the destination type or is an XLen scalar.`.
  **L735 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The second source operand matches the destination type or is an XLen scalar.`。
- **L736 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in/scalar_in, vxrm, vl)`.
  **L736 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in/scalar_in, vxrm, vl)`。
- **L737 EN**: Declares class `RISCVSaturatingBinaryABShiftUnMaskedRoundingMode`.
  **L737 CN**: 声明 class `RISCVSaturatingBinaryABShiftUnMaskedRoundingMode`。
- **L738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L738 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`.
  **L739 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`。
- **L740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<3>],`.
  **L740 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<3>],`。
- **L741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<3>>, IntrNoMem]>,`.
  **L741 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<3>>, IntrNoMem]>,`。
- **L742 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L742 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L743 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L743 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L744 EN**: Closes the current lexical scope or compound statement.
  **L744 CN**: 结束当前词法作用域或复合语句块。
- **L745 EN**: Comment explains nearby logic, invariants, or intent: `For Saturating binary operations with mask.`.
  **L745 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Saturating binary operations with mask.`。
- **L746 EN**: Comment explains nearby logic, invariants, or intent: `The destination vector type is NOT the same as first source vector (with mask).`.
  **L746 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The destination vector type is NOT the same as first source vector (with mask).`。
- **L747 EN**: Comment explains nearby logic, invariants, or intent: `The second source operand matches the destination type or is an XLen scalar.`.
  **L747 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The second source operand matches the destination type or is an XLen scalar.`。
- **L748 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vxrm, vl, policy)`.
  **L748 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, vector_in/scalar_in, mask, vxrm, vl, policy)`。
- **L749 EN**: Declares class `RISCVSaturatingBinaryABShiftMaskedRoundingMode`.
  **L749 CN**: 声明 class `RISCVSaturatingBinaryABShiftMaskedRoundingMode`。
- **L750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L750 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`.
  **L751 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, llvm_any_ty,`。
- **L752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L752 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L753 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<3>, LLVMMatchType<3>],`.
  **L753 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<3>, LLVMMatchType<3>],`。
- **L754 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L754 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L755 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L755 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L756 EN**: Closes the current lexical scope or compound statement.
  **L756 CN**: 结束当前词法作用域或复合语句块。

### Lines 757-784

````tablegen
  // Input: (vector_in, vector_in, scalar_in, vl, policy)
  class RVVSlideUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty,
                     LLVMMatchType<1>, LLVMMatchType<1>],
                    [ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // Input: (vector_in, vector_in, vector_in/scalar_in, mask, vl, policy)
  class RVVSlideMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                     LLVMMatchType<1>, LLVMMatchType<1>],
                    [ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // UnMasked Vector Multiply-Add operations, its first operand can not be undef.
  // Input: (vector_in, vector_in/scalar, vector_in, vl, policy)
  class RISCVTernaryAAXAUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_any_ty, LLVMMatchType<0>,
                     llvm_anyint_ty, LLVMMatchType<2>],
                    [ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 1;
    let VLOperand = 3;
  }
  // Masked Vector Multiply-Add operations, its first operand can not be undef.
````
- **L757 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in, scalar_in, vl, policy)`.
  **L757 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in, scalar_in, vl, policy)`。
- **L758 EN**: Declares class `RVVSlideUnMasked`.
  **L758 CN**: 声明 class `RVVSlideUnMasked`。
- **L759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L759 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty,`.
  **L760 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty,`。
- **L761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>, LLVMMatchType<1>],`.
  **L761 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>, LLVMMatchType<1>],`。
- **L762 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L762 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L763 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L763 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L764 EN**: Closes the current lexical scope or compound statement.
  **L764 CN**: 结束当前词法作用域或复合语句块。
- **L765 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in, vector_in/scalar_in, mask, vl, policy)`.
  **L765 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in, vector_in/scalar_in, mask, vl, policy)`。
- **L766 EN**: Declares class `RVVSlideMasked`.
  **L766 CN**: 声明 class `RVVSlideMasked`。
- **L767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L767 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty,`.
  **L768 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyint_ty,`。
- **L769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L769 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>, LLVMMatchType<1>],`.
  **L770 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>, LLVMMatchType<1>],`。
- **L771 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L771 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L772 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L772 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L773 EN**: Closes the current lexical scope or compound statement.
  **L773 CN**: 结束当前词法作用域或复合语句块。
- **L774 EN**: Comment explains nearby logic, invariants, or intent: `UnMasked Vector Multiply-Add operations, its first operand can not be undef.`.
  **L774 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`UnMasked Vector Multiply-Add operations, its first operand can not be undef.`。
- **L775 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in/scalar, vector_in, vl, policy)`.
  **L775 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in/scalar, vector_in, vl, policy)`。
- **L776 EN**: Declares class `RISCVTernaryAAXAUnMasked`.
  **L776 CN**: 声明 class `RISCVTernaryAAXAUnMasked`。
- **L777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L777 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L778 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_any_ty, LLVMMatchType<0>,`.
  **L778 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_any_ty, LLVMMatchType<0>,`。
- **L779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L779 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L780 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L780 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L781 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L781 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L782 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L782 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L783 EN**: Closes the current lexical scope or compound statement.
  **L783 CN**: 结束当前词法作用域或复合语句块。
- **L784 EN**: Comment explains nearby logic, invariants, or intent: `Masked Vector Multiply-Add operations, its first operand can not be undef.`.
  **L784 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Masked Vector Multiply-Add operations, its first operand can not be undef.`。

### Lines 785-812

````tablegen
  // Input: (vector_in, vector_in/scalar, vector_in, mask, vl, policy
  class RISCVTernaryAAXAMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_any_ty, LLVMMatchType<0>,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                     llvm_anyint_ty, LLVMMatchType<2>],
                    [ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 1;
    let VLOperand = 4;
  }
  // UnMasked Vector Multiply-Add operations, its first operand can not be undef.
  // Input: (vector_in, vector_in/scalar, vector_in, frm, vl, policy)
  class RISCVTernaryAAXAUnMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_any_ty, LLVMMatchType<0>,
                     llvm_anyint_ty, LLVMMatchType<2>, LLVMMatchType<2>],
                    [ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<5>>, IntrNoMem]>,
                    RISCVVIntrinsic {
    let ScalarOperand = 1;
    let VLOperand = 4;
  }
  // Masked Vector Multiply-Add operations, its first operand can not be undef.
  // Input: (vector_in, vector_in/scalar, vector_in, mask, frm, vl, policy
  class RISCVTernaryAAXAMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_any_ty, LLVMMatchType<0>,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                     llvm_anyint_ty, LLVMMatchType<2>, LLVMMatchType<2>],
````
- **L785 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in/scalar, vector_in, mask, vl, policy`.
  **L785 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in/scalar, vector_in, mask, vl, policy`。
- **L786 EN**: Declares class `RISCVTernaryAAXAMasked`.
  **L786 CN**: 声明 class `RISCVTernaryAAXAMasked`。
- **L787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L787 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L788 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_any_ty, LLVMMatchType<0>,`.
  **L788 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_any_ty, LLVMMatchType<0>,`。
- **L789 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L789 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L790 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L790 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L791 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L791 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L792 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L792 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L793 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L793 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L794 EN**: Closes the current lexical scope or compound statement.
  **L794 CN**: 结束当前词法作用域或复合语句块。
- **L795 EN**: Comment explains nearby logic, invariants, or intent: `UnMasked Vector Multiply-Add operations, its first operand can not be undef.`.
  **L795 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`UnMasked Vector Multiply-Add operations, its first operand can not be undef.`。
- **L796 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in/scalar, vector_in, frm, vl, policy)`.
  **L796 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in/scalar, vector_in, frm, vl, policy)`。
- **L797 EN**: Declares class `RISCVTernaryAAXAUnMaskedRoundingMode`.
  **L797 CN**: 声明 class `RISCVTernaryAAXAUnMaskedRoundingMode`。
- **L798 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L798 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L799 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_any_ty, LLVMMatchType<0>,`.
  **L799 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_any_ty, LLVMMatchType<0>,`。
- **L800 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>, LLVMMatchType<2>],`.
  **L800 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>, LLVMMatchType<2>],`。
- **L801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<5>>, IntrNoMem]>,`.
  **L801 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<5>>, IntrNoMem]>,`。
- **L802 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L802 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L803 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L803 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L804 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L804 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L805 EN**: Closes the current lexical scope or compound statement.
  **L805 CN**: 结束当前词法作用域或复合语句块。
- **L806 EN**: Comment explains nearby logic, invariants, or intent: `Masked Vector Multiply-Add operations, its first operand can not be undef.`.
  **L806 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Masked Vector Multiply-Add operations, its first operand can not be undef.`。
- **L807 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in/scalar, vector_in, mask, frm, vl, policy`.
  **L807 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in/scalar, vector_in, mask, frm, vl, policy`。
- **L808 EN**: Declares class `RISCVTernaryAAXAMaskedRoundingMode`.
  **L808 CN**: 声明 class `RISCVTernaryAAXAMaskedRoundingMode`。
- **L809 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L809 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L810 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_any_ty, LLVMMatchType<0>,`.
  **L810 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_any_ty, LLVMMatchType<0>,`。
- **L811 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L811 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>, LLVMMatchType<2>],`.
  **L812 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>, LLVMMatchType<2>],`。

### Lines 813-840

````tablegen
                    [ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>,
                    RISCVVIntrinsic {
    let ScalarOperand = 1;
    let VLOperand = 5;
  }
  // UnMasked Widening Vector Multiply-Add operations, its first operand can not be undef.
  // Input: (vector_in, vector_in/scalar, vector_in, vl, policy)
  class RISCVTernaryWideUnMasked
        : DefaultAttrsIntrinsic< [llvm_anyvector_ty],
                     [LLVMMatchType<0>, llvm_any_ty, llvm_anyvector_ty,
                      llvm_anyint_ty, LLVMMatchType<3>],
                     [ImmArg<ArgIndex<4>>, IntrNoMem] >, RISCVVIntrinsic {
    let ScalarOperand = 1;
    let VLOperand = 3;
  }
  // Masked Widening Vector Multiply-Add operations, its first operand can not be undef.
  // Input: (vector_in, vector_in/scalar, vector_in, mask, vl, policy
  class RISCVTernaryWideMasked
        : DefaultAttrsIntrinsic< [llvm_anyvector_ty],
                     [LLVMMatchType<0>, llvm_any_ty, llvm_anyvector_ty,
                      LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                      llvm_anyint_ty, LLVMMatchType<3>],
                     [ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let ScalarOperand = 1;
    let VLOperand = 4;
  }
  // UnMasked Widening Vector Multiply-Add operations, its first operand can not be undef.
  // Input: (vector_in, vector_in/scalar, vector_in, frm, vl, policy)
````
- **L813 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>,`.
  **L813 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>,`。
- **L814 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L814 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L815 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L815 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L816 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L816 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L817 EN**: Closes the current lexical scope or compound statement.
  **L817 CN**: 结束当前词法作用域或复合语句块。
- **L818 EN**: Comment explains nearby logic, invariants, or intent: `UnMasked Widening Vector Multiply-Add operations, its first operand can not be undef.`.
  **L818 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`UnMasked Widening Vector Multiply-Add operations, its first operand can not be undef.`。
- **L819 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in/scalar, vector_in, vl, policy)`.
  **L819 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in/scalar, vector_in, vl, policy)`。
- **L820 EN**: Declares class `RISCVTernaryWideUnMasked`.
  **L820 CN**: 声明 class `RISCVTernaryWideUnMasked`。
- **L821 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic< [llvm_anyvector_ty],`.
  **L821 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic< [llvm_anyvector_ty],`。
- **L822 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_any_ty, llvm_anyvector_ty,`.
  **L822 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_any_ty, llvm_anyvector_ty,`。
- **L823 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<3>],`.
  **L823 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<3>],`。
- **L824 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, IntrNoMem] >, RISCVVIntrinsic {`.
  **L824 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, IntrNoMem] >, RISCVVIntrinsic {`。
- **L825 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L825 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L826 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L826 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L827 EN**: Closes the current lexical scope or compound statement.
  **L827 CN**: 结束当前词法作用域或复合语句块。
- **L828 EN**: Comment explains nearby logic, invariants, or intent: `Masked Widening Vector Multiply-Add operations, its first operand can not be undef.`.
  **L828 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Masked Widening Vector Multiply-Add operations, its first operand can not be undef.`。
- **L829 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in/scalar, vector_in, mask, vl, policy`.
  **L829 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in/scalar, vector_in, mask, vl, policy`。
- **L830 EN**: Declares class `RISCVTernaryWideMasked`.
  **L830 CN**: 声明 class `RISCVTernaryWideMasked`。
- **L831 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic< [llvm_anyvector_ty],`.
  **L831 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic< [llvm_anyvector_ty],`。
- **L832 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_any_ty, llvm_anyvector_ty,`.
  **L832 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_any_ty, llvm_anyvector_ty,`。
- **L833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L833 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L834 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<3>],`.
  **L834 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<3>],`。
- **L835 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L835 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L836 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L836 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L837 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L837 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L838 EN**: Closes the current lexical scope or compound statement.
  **L838 CN**: 结束当前词法作用域或复合语句块。
- **L839 EN**: Comment explains nearby logic, invariants, or intent: `UnMasked Widening Vector Multiply-Add operations, its first operand can not be undef.`.
  **L839 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`UnMasked Widening Vector Multiply-Add operations, its first operand can not be undef.`。
- **L840 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in/scalar, vector_in, frm, vl, policy)`.
  **L840 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in/scalar, vector_in, frm, vl, policy)`。

### Lines 841-868

````tablegen
  class RISCVTernaryWideUnMaskedRoundingMode
        : DefaultAttrsIntrinsic< [llvm_anyvector_ty],
                     [LLVMMatchType<0>, llvm_any_ty, llvm_anyvector_ty,
                      llvm_anyint_ty, LLVMMatchType<3>, LLVMMatchType<3>],
                     [ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<5>>, IntrNoMem] >,
                     RISCVVIntrinsic {
    let ScalarOperand = 1;
    let VLOperand = 4;
  }
  // Masked Widening Vector Multiply-Add operations, its first operand can not be undef.
  // Input: (vector_in, vector_in/scalar, vector_in, mask, frm, vl, policy
  class RISCVTernaryWideMaskedRoundingMode
        : DefaultAttrsIntrinsic< [llvm_anyvector_ty],
                     [LLVMMatchType<0>, llvm_any_ty, llvm_anyvector_ty,
                      LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                      llvm_anyint_ty, LLVMMatchType<3>, LLVMMatchType<3>],
                     [ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>,
                     RISCVVIntrinsic {
    let ScalarOperand = 1;
    let VLOperand = 5;
  }
  // For Reduction ternary operations.
  // For destination vector type is the same as first and third source vector.
  // Input: (vector_in, vector_in, vector_in, vl)
  class RISCVReductionUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>,
                     llvm_anyint_ty],
````
- **L841 EN**: Declares class `RISCVTernaryWideUnMaskedRoundingMode`.
  **L841 CN**: 声明 class `RISCVTernaryWideUnMaskedRoundingMode`。
- **L842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic< [llvm_anyvector_ty],`.
  **L842 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic< [llvm_anyvector_ty],`。
- **L843 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_any_ty, llvm_anyvector_ty,`.
  **L843 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_any_ty, llvm_anyvector_ty,`。
- **L844 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<3>, LLVMMatchType<3>],`.
  **L844 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<3>, LLVMMatchType<3>],`。
- **L845 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<5>>, IntrNoMem] >,`.
  **L845 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<5>>, IntrNoMem] >,`。
- **L846 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L846 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L847 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L847 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L848 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L848 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L849 EN**: Closes the current lexical scope or compound statement.
  **L849 CN**: 结束当前词法作用域或复合语句块。
- **L850 EN**: Comment explains nearby logic, invariants, or intent: `Masked Widening Vector Multiply-Add operations, its first operand can not be undef.`.
  **L850 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Masked Widening Vector Multiply-Add operations, its first operand can not be undef.`。
- **L851 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in/scalar, vector_in, mask, frm, vl, policy`.
  **L851 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in/scalar, vector_in, mask, frm, vl, policy`。
- **L852 EN**: Declares class `RISCVTernaryWideMaskedRoundingMode`.
  **L852 CN**: 声明 class `RISCVTernaryWideMaskedRoundingMode`。
- **L853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic< [llvm_anyvector_ty],`.
  **L853 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic< [llvm_anyvector_ty],`。
- **L854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_any_ty, llvm_anyvector_ty,`.
  **L854 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_any_ty, llvm_anyvector_ty,`。
- **L855 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L855 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<3>, LLVMMatchType<3>],`.
  **L856 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<3>, LLVMMatchType<3>],`。
- **L857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>,`.
  **L857 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>, IntrNoMem]>,`。
- **L858 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L858 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L859 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L859 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L860 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L860 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L861 EN**: Closes the current lexical scope or compound statement.
  **L861 CN**: 结束当前词法作用域或复合语句块。
- **L862 EN**: Comment explains nearby logic, invariants, or intent: `For Reduction ternary operations.`.
  **L862 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Reduction ternary operations.`。
- **L863 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first and third source vector.`.
  **L863 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first and third source vector.`。
- **L864 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in, vector_in, vl)`.
  **L864 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in, vector_in, vl)`。
- **L865 EN**: Declares class `RISCVReductionUnMasked`.
  **L865 CN**: 声明 class `RISCVReductionUnMasked`。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L867 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L868 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。

### Lines 869-896

````tablegen
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For Reduction ternary operations with mask.
  // For destination vector type is the same as first and third source vector.
  // The mask type come from second source vector.
  // Input: (maskedoff, vector_in, vector_in, vector_in, mask, vl)
  class RISCVReductionMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>,
                     LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // For Reduction ternary operations.
  // For destination vector type is the same as first and third source vector.
  // Input: (vector_in, vector_in, vector_in, frm, vl)
  class RISCVReductionUnMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>,
                     llvm_anyint_ty, LLVMMatchType<2>],
                    [ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }
  // For Reduction ternary operations with mask.
  // For destination vector type is the same as first and third source vector.
  // The mask type come from second source vector.
  // Input: (vector_in, vector_in, vector_in, mask, frm, vl)
````
- **L869 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L869 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L870 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L870 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L871 EN**: Closes the current lexical scope or compound statement.
  **L871 CN**: 结束当前词法作用域或复合语句块。
- **L872 EN**: Comment explains nearby logic, invariants, or intent: `For Reduction ternary operations with mask.`.
  **L872 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Reduction ternary operations with mask.`。
- **L873 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first and third source vector.`.
  **L873 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first and third source vector.`。
- **L874 EN**: Comment explains nearby logic, invariants, or intent: `The mask type come from second source vector.`.
  **L874 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The mask type come from second source vector.`。
- **L875 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, vector_in, vector_in, mask, vl)`.
  **L875 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, vector_in, vector_in, mask, vl)`。
- **L876 EN**: Declares class `RISCVReductionMasked`.
  **L876 CN**: 声明 class `RISCVReductionMasked`。
- **L877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L877 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L878 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L878 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L879 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>, llvm_anyint_ty],`.
  **L879 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>, llvm_anyint_ty],`。
- **L880 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L880 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L881 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L881 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L882 EN**: Closes the current lexical scope or compound statement.
  **L882 CN**: 结束当前词法作用域或复合语句块。
- **L883 EN**: Comment explains nearby logic, invariants, or intent: `For Reduction ternary operations.`.
  **L883 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Reduction ternary operations.`。
- **L884 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first and third source vector.`.
  **L884 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first and third source vector.`。
- **L885 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in, vector_in, frm, vl)`.
  **L885 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in, vector_in, frm, vl)`。
- **L886 EN**: Declares class `RISCVReductionUnMaskedRoundingMode`.
  **L886 CN**: 声明 class `RISCVReductionUnMaskedRoundingMode`。
- **L887 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L887 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L888 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L889 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L889 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L890 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L890 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L891 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L891 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L892 EN**: Closes the current lexical scope or compound statement.
  **L892 CN**: 结束当前词法作用域或复合语句块。
- **L893 EN**: Comment explains nearby logic, invariants, or intent: `For Reduction ternary operations with mask.`.
  **L893 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Reduction ternary operations with mask.`。
- **L894 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is the same as first and third source vector.`.
  **L894 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is the same as first and third source vector.`。
- **L895 EN**: Comment explains nearby logic, invariants, or intent: `The mask type come from second source vector.`.
  **L895 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The mask type come from second source vector.`。
- **L896 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in, vector_in, mask, frm, vl)`.
  **L896 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in, vector_in, mask, frm, vl)`。

### Lines 897-924

````tablegen
  class RISCVReductionMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>,
                     LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<2>],
                    [ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 5;
  }
  // For unary operations with scalar type output without mask
  // Output: (scalar type)
  // Input: (vector_in, vl)
  class RISCVMaskedUnarySOutUnMasked
        : DefaultAttrsIntrinsic<[LLVMMatchType<1>],
                    [llvm_anyvector_ty, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 1;
  }
  // For unary operations with scalar type output with mask
  // Output: (scalar type)
  // Input: (vector_in, mask, vl)
  class RISCVMaskedUnarySOutMasked
        : DefaultAttrsIntrinsic<[LLVMMatchType<1>],
                    [llvm_anyvector_ty, LLVMMatchType<0>, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 2;
  }
  // For destination vector type is NOT the same as source vector.
  // Input: (passthru, vector_in, vl)
````
- **L897 EN**: Declares class `RISCVReductionMaskedRoundingMode`.
  **L897 CN**: 声明 class `RISCVReductionMaskedRoundingMode`。
- **L898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L898 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L899 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L900 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>, llvm_anyint_ty,`.
  **L900 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>, llvm_anyint_ty,`。
- **L901 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>],`.
  **L901 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>],`。
- **L902 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L902 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L903 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L903 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L904 EN**: Closes the current lexical scope or compound statement.
  **L904 CN**: 结束当前词法作用域或复合语句块。
- **L905 EN**: Comment explains nearby logic, invariants, or intent: `For unary operations with scalar type output without mask`.
  **L905 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unary operations with scalar type output without mask`。
- **L906 EN**: Comment explains nearby logic, invariants, or intent: `Output: (scalar type)`.
  **L906 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (scalar type)`。
- **L907 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vl)`.
  **L907 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vl)`。
- **L908 EN**: Declares class `RISCVMaskedUnarySOutUnMasked`.
  **L908 CN**: 声明 class `RISCVMaskedUnarySOutUnMasked`。
- **L909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMMatchType<1>],`.
  **L909 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMMatchType<1>],`。
- **L910 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_anyint_ty],`.
  **L910 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_anyint_ty],`。
- **L911 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L911 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L912 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L912 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L913 EN**: Closes the current lexical scope or compound statement.
  **L913 CN**: 结束当前词法作用域或复合语句块。
- **L914 EN**: Comment explains nearby logic, invariants, or intent: `For unary operations with scalar type output with mask`.
  **L914 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unary operations with scalar type output with mask`。
- **L915 EN**: Comment explains nearby logic, invariants, or intent: `Output: (scalar type)`.
  **L915 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (scalar type)`。
- **L916 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, mask, vl)`.
  **L916 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, mask, vl)`。
- **L917 EN**: Declares class `RISCVMaskedUnarySOutMasked`.
  **L917 CN**: 声明 class `RISCVMaskedUnarySOutMasked`。
- **L918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMMatchType<1>],`.
  **L918 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMMatchType<1>],`。
- **L919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>, llvm_anyint_ty],`.
  **L919 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>, llvm_anyint_ty],`。
- **L920 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L920 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L921 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L921 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L922 EN**: Closes the current lexical scope or compound statement.
  **L922 CN**: 结束当前词法作用域或复合语句块。
- **L923 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is NOT the same as source vector.`.
  **L923 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is NOT the same as source vector.`。
- **L924 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vl)`.
  **L924 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vl)`。

### Lines 925-952

````tablegen
  class RISCVUnaryABUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 2;
  }
  // For destination vector type is NOT the same as source vector (with mask).
  // Input: (maskedoff, vector_in, mask, vl, policy)
  class RISCVUnaryABMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty,
                     LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>,
                     llvm_anyint_ty, LLVMMatchType<2>],
                    [ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For unary operations with the same vector type in/out without mask
  // Output: (vector)
  // Input: (vector_in, vl)
  class RISCVUnaryUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 1;
  }
  // For mask unary operations with mask type in/out with mask
  // Output: (mask type output)
  // Input: (mask type maskedoff, mask type vector_in, mask, vl)
````
- **L925 EN**: Declares class `RISCVUnaryABUnMasked`.
  **L925 CN**: 声明 class `RISCVUnaryABUnMasked`。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, llvm_anyint_ty],`.
  **L927 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, llvm_anyint_ty],`。
- **L928 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L928 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L929 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L929 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L930 EN**: Closes the current lexical scope or compound statement.
  **L930 CN**: 结束当前词法作用域或复合语句块。
- **L931 EN**: Comment explains nearby logic, invariants, or intent: `For destination vector type is NOT the same as source vector (with mask).`.
  **L931 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For destination vector type is NOT the same as source vector (with mask).`。
- **L932 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, mask, vl, policy)`.
  **L932 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, mask, vl, policy)`。
- **L933 EN**: Declares class `RISCVUnaryABMasked`.
  **L933 CN**: 声明 class `RISCVUnaryABMasked`。
- **L934 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L934 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L935 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L935 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>,`.
  **L936 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>,`。
- **L937 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L937 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L938 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L938 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L939 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L939 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L940 EN**: Closes the current lexical scope or compound statement.
  **L940 CN**: 结束当前词法作用域或复合语句块。
- **L941 EN**: Comment explains nearby logic, invariants, or intent: `For unary operations with the same vector type in/out without mask`.
  **L941 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unary operations with the same vector type in/out without mask`。
- **L942 EN**: Comment explains nearby logic, invariants, or intent: `Output: (vector)`.
  **L942 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (vector)`。
- **L943 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vl)`.
  **L943 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vl)`。
- **L944 EN**: Declares class `RISCVUnaryUnMasked`.
  **L944 CN**: 声明 class `RISCVUnaryUnMasked`。
- **L945 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L945 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L946 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyint_ty],`.
  **L946 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyint_ty],`。
- **L947 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L947 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L948 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L948 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L949 EN**: Closes the current lexical scope or compound statement.
  **L949 CN**: 结束当前词法作用域或复合语句块。
- **L950 EN**: Comment explains nearby logic, invariants, or intent: `For mask unary operations with mask type in/out with mask`.
  **L950 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For mask unary operations with mask type in/out with mask`。
- **L951 EN**: Comment explains nearby logic, invariants, or intent: `Output: (mask type output)`.
  **L951 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (mask type output)`。
- **L952 EN**: Comment explains nearby logic, invariants, or intent: `Input: (mask type maskedoff, mask type vector_in, mask, vl)`.
  **L952 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (mask type maskedoff, mask type vector_in, mask, vl)`。

### Lines 953-980

````tablegen
  class RISCVMaskedUnaryMOutMasked
        : DefaultAttrsIntrinsic<[llvm_anyint_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>,
                     LLVMMatchType<0>, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // Output: (vector)
  // Input: (vl)
  class RISCVNullaryIntrinsic
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [llvm_anyint_ty], [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 1;
  }
  // Output: (vector)
  // Input: (passthru, vl)
  class RISCVID
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 1;
  }
  // For Conversion unary operations.
  // Input: (passthru, vector_in, vl)
  class RISCVConversionUnMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, llvm_anyint_ty],
                    [IntrNoMem]>, RISCVVIntrinsic {
````
- **L953 EN**: Declares class `RISCVMaskedUnaryMOutMasked`.
  **L953 CN**: 声明 class `RISCVMaskedUnaryMOutMasked`。
- **L954 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyint_ty],`.
  **L954 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyint_ty],`。
- **L955 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L955 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L956 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, llvm_anyint_ty],`.
  **L956 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, llvm_anyint_ty],`。
- **L957 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L957 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L958 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L958 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L959 EN**: Closes the current lexical scope or compound statement.
  **L959 CN**: 结束当前词法作用域或复合语句块。
- **L960 EN**: Comment explains nearby logic, invariants, or intent: `Output: (vector)`.
  **L960 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (vector)`。
- **L961 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vl)`.
  **L961 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vl)`。
- **L962 EN**: Declares class `RISCVNullaryIntrinsic`.
  **L962 CN**: 声明 class `RISCVNullaryIntrinsic`。
- **L963 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L963 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L964 EN**: Continues the surrounding expression or declaration: `[llvm_anyint_ty], [IntrNoMem]>, RISCVVIntrinsic {`.
  **L964 CN**: 继续构造周围的表达式或声明：`[llvm_anyint_ty], [IntrNoMem]>, RISCVVIntrinsic {`。
- **L965 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L965 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L966 EN**: Closes the current lexical scope or compound statement.
  **L966 CN**: 结束当前词法作用域或复合语句块。
- **L967 EN**: Comment explains nearby logic, invariants, or intent: `Output: (vector)`.
  **L967 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (vector)`。
- **L968 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vl)`.
  **L968 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vl)`。
- **L969 EN**: Declares class `RISCVID`.
  **L969 CN**: 声明 class `RISCVID`。
- **L970 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L970 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L971 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyint_ty],`.
  **L971 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyint_ty],`。
- **L972 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L972 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L973 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L973 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L974 EN**: Closes the current lexical scope or compound statement.
  **L974 CN**: 结束当前词法作用域或复合语句块。
- **L975 EN**: Comment explains nearby logic, invariants, or intent: `For Conversion unary operations.`.
  **L975 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Conversion unary operations.`。
- **L976 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vl)`.
  **L976 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vl)`。
- **L977 EN**: Declares class `RISCVConversionUnMasked`.
  **L977 CN**: 声明 class `RISCVConversionUnMasked`。
- **L978 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L978 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L979 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, llvm_anyint_ty],`.
  **L979 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, llvm_anyint_ty],`。
- **L980 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L980 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。

### Lines 981-1008

````tablegen
    let VLOperand = 2;
  }
  // For Conversion unary operations with mask.
  // Input: (maskedoff, vector_in, mask, vl, policy)
  class RISCVConversionMasked
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<2>],
                    [ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For Conversion unary operations.
  // Input: (passthru, vector_in, frm, vl)
  class RISCVConversionUnMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty, llvm_anyint_ty,
                     LLVMMatchType<2>],
                    [ImmArg<ArgIndex<2>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For Conversion unary operations with mask.
  // Input: (maskedoff, vector_in, mask, frm, vl, policy)
  class RISCVConversionMaskedRoundingMode
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                    [LLVMMatchType<0>, llvm_anyvector_ty,
                     LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,
                     LLVMMatchType<2>, LLVMMatchType<2>],
````
- **L981 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L981 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L982 EN**: Closes the current lexical scope or compound statement.
  **L982 CN**: 结束当前词法作用域或复合语句块。
- **L983 EN**: Comment explains nearby logic, invariants, or intent: `For Conversion unary operations with mask.`.
  **L983 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Conversion unary operations with mask.`。
- **L984 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, mask, vl, policy)`.
  **L984 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, mask, vl, policy)`。
- **L985 EN**: Declares class `RISCVConversionMasked`.
  **L985 CN**: 声明 class `RISCVConversionMasked`。
- **L986 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L986 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L987 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L987 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L988 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L988 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L989 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>],`.
  **L989 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>],`。
- **L990 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L990 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L991 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L991 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L992 EN**: Closes the current lexical scope or compound statement.
  **L992 CN**: 结束当前词法作用域或复合语句块。
- **L993 EN**: Comment explains nearby logic, invariants, or intent: `For Conversion unary operations.`.
  **L993 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Conversion unary operations.`。
- **L994 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, frm, vl)`.
  **L994 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, frm, vl)`。
- **L995 EN**: Declares class `RISCVConversionUnMaskedRoundingMode`.
  **L995 CN**: 声明 class `RISCVConversionUnMaskedRoundingMode`。
- **L996 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L996 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L997 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, llvm_anyint_ty,`.
  **L997 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, llvm_anyint_ty,`。
- **L998 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>],`.
  **L998 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>],`。
- **L999 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<2>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L999 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<2>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L1000 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1000 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1001 EN**: Closes the current lexical scope or compound statement.
  **L1001 CN**: 结束当前词法作用域或复合语句块。
- **L1002 EN**: Comment explains nearby logic, invariants, or intent: `For Conversion unary operations with mask.`.
  **L1002 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For Conversion unary operations with mask.`。
- **L1003 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, mask, frm, vl, policy)`.
  **L1003 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, mask, frm, vl, policy)`。
- **L1004 EN**: Declares class `RISCVConversionMaskedRoundingMode`.
  **L1004 CN**: 声明 class `RISCVConversionMaskedRoundingMode`。
- **L1005 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1005 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1006 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L1006 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L1007 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`.
  **L1007 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyint_ty,`。
- **L1008 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>, LLVMMatchType<2>],`.
  **L1008 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>, LLVMMatchType<2>],`。

### Lines 1009-1036

````tablegen
                    [ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }

  // For unit stride segment load
  // Input: (passthru, pointer, vl, sew)
  class RISCVUSSegLoad
        : DefaultAttrsIntrinsic<[llvm_any_ty],
                                [LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyint_ty,
                                 LLVMMatchType<2>],
                    [NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<3>>, IntrReadMem,
                     IntrArgMemOnly]>,
          RISCVVIntrinsic {
    let VLOperand = 2;
  }
  // For unit stride segment load with mask
  // Input: (maskedoff, pointer, mask, vl, policy, sew)
  class RISCVUSSegLoadMasked
        : DefaultAttrsIntrinsic<[llvm_any_ty],
                                [LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyvector_ty,
                                 llvm_anyint_ty, LLVMMatchType<3>, LLVMMatchType<3>],
                    [ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>,
                     NoCapture<ArgIndex<1>>, IntrReadMem, IntrArgMemOnly]>,
          RISCVVIntrinsic {
    let VLOperand = 3;
  }

  // For unit stride fault-only-first segment load
````
- **L1009 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L1009 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L1010 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1010 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1011 EN**: Closes the current lexical scope or compound statement.
  **L1011 CN**: 结束当前词法作用域或复合语句块。
- **L1012 EN**: Blank line separating nearby declarations or logic blocks.
  **L1012 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1013 EN**: Comment explains nearby logic, invariants, or intent: `For unit stride segment load`.
  **L1013 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unit stride segment load`。
- **L1014 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, pointer, vl, sew)`.
  **L1014 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, pointer, vl, sew)`。
- **L1015 EN**: Declares class `RISCVUSSegLoad`.
  **L1015 CN**: 声明 class `RISCVUSSegLoad`。
- **L1016 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L1016 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L1017 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyint_ty,`.
  **L1017 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyint_ty,`。
- **L1018 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>],`.
  **L1018 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>],`。
- **L1019 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<3>>, IntrReadMem,`.
  **L1019 CN**: 继续一个多行参数列表、初始化器或聚合项：`[NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<3>>, IntrReadMem,`。
- **L1020 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrArgMemOnly]>,`.
  **L1020 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrArgMemOnly]>,`。
- **L1021 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L1021 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L1022 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1022 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1023 EN**: Closes the current lexical scope or compound statement.
  **L1023 CN**: 结束当前词法作用域或复合语句块。
- **L1024 EN**: Comment explains nearby logic, invariants, or intent: `For unit stride segment load with mask`.
  **L1024 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unit stride segment load with mask`。
- **L1025 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, pointer, mask, vl, policy, sew)`.
  **L1025 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, pointer, mask, vl, policy, sew)`。
- **L1026 EN**: Declares class `RISCVUSSegLoadMasked`.
  **L1026 CN**: 声明 class `RISCVUSSegLoadMasked`。
- **L1027 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L1027 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L1028 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyvector_ty,`.
  **L1028 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyvector_ty,`。
- **L1029 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<3>, LLVMMatchType<3>],`.
  **L1029 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<3>, LLVMMatchType<3>],`。
- **L1030 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>,`.
  **L1030 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>,`。
- **L1031 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NoCapture<ArgIndex<1>>, IntrReadMem, IntrArgMemOnly]>,`.
  **L1031 CN**: 继续一个多行参数列表、初始化器或聚合项：`NoCapture<ArgIndex<1>>, IntrReadMem, IntrArgMemOnly]>,`。
- **L1032 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L1032 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L1033 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1033 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1034 EN**: Closes the current lexical scope or compound statement.
  **L1034 CN**: 结束当前词法作用域或复合语句块。
- **L1035 EN**: Blank line separating nearby declarations or logic blocks.
  **L1035 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1036 EN**: Comment explains nearby logic, invariants, or intent: `For unit stride fault-only-first segment load`.
  **L1036 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unit stride fault-only-first segment load`。

### Lines 1037-1064

````tablegen
  // Input: (passthru, pointer, vl, sew)
  // Output: (data, vl)
  // NOTE: We model this with default memory properties since we model writing
  // VL as a side effect. IntrReadMem, IntrHasSideEffects does not work.
  class RISCVUSSegLoadFF
        : DefaultAttrsIntrinsic<[llvm_any_ty, llvm_anyint_ty],
                    [LLVMMatchType<0>, llvm_anyptr_ty, LLVMMatchType<1>, LLVMMatchType<1>],
                    [ImmArg<ArgIndex<3>>, NoCapture<ArgIndex<1>>]>, RISCVVIntrinsic {
    let VLOperand = 2;
  }
  // For unit stride fault-only-first segment load with mask
  // Input: (maskedoff, pointer, mask, vl, policy, sew)
  // Output: (data, vl)
  // NOTE: We model this with default memory properties since we model writing
  // VL as a side effect. IntrReadMem, IntrHasSideEffects does not work.
  class RISCVUSSegLoadFFMasked
        : DefaultAttrsIntrinsic<[llvm_any_ty, llvm_anyint_ty],
                     [LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyvector_ty,
                      LLVMMatchType<1>, LLVMMatchType<1>, LLVMMatchType<1>],
                    [ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, NoCapture<ArgIndex<1>>]>,
                    RISCVVIntrinsic {
    let VLOperand = 3;
  }

  // For stride segment load
  // Input: (passthru, pointer, offset, vl, sew)
  class RISCVSSegLoad
        : DefaultAttrsIntrinsic<[llvm_any_ty],
````
- **L1037 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, pointer, vl, sew)`.
  **L1037 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, pointer, vl, sew)`。
- **L1038 EN**: Comment explains nearby logic, invariants, or intent: `Output: (data, vl)`.
  **L1038 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (data, vl)`。
- **L1039 EN**: Comment highlights an implementation note: `NOTE: We model this with default memory properties since we model writing`.
  **L1039 CN**: 注释强调了一条实现说明：`NOTE: We model this with default memory properties since we model writing`。
- **L1040 EN**: Comment explains nearby logic, invariants, or intent: `VL as a side effect. IntrReadMem, IntrHasSideEffects does not work.`.
  **L1040 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VL as a side effect. IntrReadMem, IntrHasSideEffects does not work.`。
- **L1041 EN**: Declares class `RISCVUSSegLoadFF`.
  **L1041 CN**: 声明 class `RISCVUSSegLoadFF`。
- **L1042 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty, llvm_anyint_ty],`.
  **L1042 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty, llvm_anyint_ty],`。
- **L1043 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty, LLVMMatchType<1>, LLVMMatchType<1>],`.
  **L1043 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty, LLVMMatchType<1>, LLVMMatchType<1>],`。
- **L1044 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, NoCapture<ArgIndex<1>>]>, RISCVVIntrinsic {`.
  **L1044 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, NoCapture<ArgIndex<1>>]>, RISCVVIntrinsic {`。
- **L1045 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1045 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1046 EN**: Closes the current lexical scope or compound statement.
  **L1046 CN**: 结束当前词法作用域或复合语句块。
- **L1047 EN**: Comment explains nearby logic, invariants, or intent: `For unit stride fault-only-first segment load with mask`.
  **L1047 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unit stride fault-only-first segment load with mask`。
- **L1048 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, pointer, mask, vl, policy, sew)`.
  **L1048 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, pointer, mask, vl, policy, sew)`。
- **L1049 EN**: Comment explains nearby logic, invariants, or intent: `Output: (data, vl)`.
  **L1049 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (data, vl)`。
- **L1050 EN**: Comment highlights an implementation note: `NOTE: We model this with default memory properties since we model writing`.
  **L1050 CN**: 注释强调了一条实现说明：`NOTE: We model this with default memory properties since we model writing`。
- **L1051 EN**: Comment explains nearby logic, invariants, or intent: `VL as a side effect. IntrReadMem, IntrHasSideEffects does not work.`.
  **L1051 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VL as a side effect. IntrReadMem, IntrHasSideEffects does not work.`。
- **L1052 EN**: Declares class `RISCVUSSegLoadFFMasked`.
  **L1052 CN**: 声明 class `RISCVUSSegLoadFFMasked`。
- **L1053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty, llvm_anyint_ty],`.
  **L1053 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty, llvm_anyint_ty],`。
- **L1054 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyvector_ty,`.
  **L1054 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyvector_ty,`。
- **L1055 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>, LLVMMatchType<1>, LLVMMatchType<1>],`.
  **L1055 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>, LLVMMatchType<1>, LLVMMatchType<1>],`。
- **L1056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, NoCapture<ArgIndex<1>>]>,`.
  **L1056 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, NoCapture<ArgIndex<1>>]>,`。
- **L1057 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L1057 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L1058 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1058 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1059 EN**: Closes the current lexical scope or compound statement.
  **L1059 CN**: 结束当前词法作用域或复合语句块。
- **L1060 EN**: Blank line separating nearby declarations or logic blocks.
  **L1060 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1061 EN**: Comment explains nearby logic, invariants, or intent: `For stride segment load`.
  **L1061 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For stride segment load`。
- **L1062 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, pointer, offset, vl, sew)`.
  **L1062 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, pointer, offset, vl, sew)`。
- **L1063 EN**: Declares class `RISCVSSegLoad`.
  **L1063 CN**: 声明 class `RISCVSSegLoad`。
- **L1064 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L1064 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。

### Lines 1065-1092

````tablegen
                    [LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyint_ty,
                     LLVMMatchType<2>, LLVMMatchType<2>],
                    [ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrReadMem]>,
          RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For stride segment load with mask
  // Input: (maskedoff, pointer, offset, mask, vl, policy, sew)
  class RISCVSSegLoadMasked
        : DefaultAttrsIntrinsic<[llvm_any_ty],
                                [LLVMMatchType<0>, llvm_anyptr_ty,
                                 llvm_anyint_ty, llvm_anyvector_ty,
                                 LLVMMatchType<2>, LLVMMatchType<2>,
                                 LLVMMatchType<2>],
                    [ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>,
                     NoCapture<ArgIndex<1>>, IntrReadMem]>,
                    RISCVVIntrinsic {
    let VLOperand = 4;
  }

  // For indexed segment load
  // Input: (passthru, pointer, index, vl, sew)
  class RISCVISegLoad
        : DefaultAttrsIntrinsic<[llvm_any_ty],
                    [LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyvector_ty,
                     llvm_anyint_ty, LLVMMatchType<3>],
                    [ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrReadMem]>,
          RISCVVIntrinsic {
````
- **L1065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyint_ty,`.
  **L1065 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyint_ty,`。
- **L1066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>, LLVMMatchType<2>],`.
  **L1066 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>, LLVMMatchType<2>],`。
- **L1067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrReadMem]>,`.
  **L1067 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrReadMem]>,`。
- **L1068 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L1068 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L1069 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1069 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1070 EN**: Closes the current lexical scope or compound statement.
  **L1070 CN**: 结束当前词法作用域或复合语句块。
- **L1071 EN**: Comment explains nearby logic, invariants, or intent: `For stride segment load with mask`.
  **L1071 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For stride segment load with mask`。
- **L1072 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, pointer, offset, mask, vl, policy, sew)`.
  **L1072 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, pointer, offset, mask, vl, policy, sew)`。
- **L1073 EN**: Declares class `RISCVSSegLoadMasked`.
  **L1073 CN**: 声明 class `RISCVSSegLoadMasked`。
- **L1074 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L1074 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L1075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty,`.
  **L1075 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty,`。
- **L1076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, llvm_anyvector_ty,`.
  **L1076 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, llvm_anyvector_ty,`。
- **L1077 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>, LLVMMatchType<2>,`.
  **L1077 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>, LLVMMatchType<2>,`。
- **L1078 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>],`.
  **L1078 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>],`。
- **L1079 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>,`.
  **L1079 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>,`。
- **L1080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NoCapture<ArgIndex<1>>, IntrReadMem]>,`.
  **L1080 CN**: 继续一个多行参数列表、初始化器或聚合项：`NoCapture<ArgIndex<1>>, IntrReadMem]>,`。
- **L1081 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L1081 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L1082 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1082 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1083 EN**: Closes the current lexical scope or compound statement.
  **L1083 CN**: 结束当前词法作用域或复合语句块。
- **L1084 EN**: Blank line separating nearby declarations or logic blocks.
  **L1084 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1085 EN**: Comment explains nearby logic, invariants, or intent: `For indexed segment load`.
  **L1085 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For indexed segment load`。
- **L1086 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, pointer, index, vl, sew)`.
  **L1086 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, pointer, index, vl, sew)`。
- **L1087 EN**: Declares class `RISCVISegLoad`.
  **L1087 CN**: 声明 class `RISCVISegLoad`。
- **L1088 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L1088 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L1089 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyvector_ty,`.
  **L1089 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty, llvm_anyvector_ty,`。
- **L1090 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<3>],`.
  **L1090 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<3>],`。
- **L1091 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrReadMem]>,`.
  **L1091 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrReadMem]>,`。
- **L1092 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L1092 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。

### Lines 1093-1120

````tablegen
    let VLOperand = 3;
  }
  // For indexed segment load with mask
  // Input: (maskedoff, pointer, index, mask, vl, policy, sew)
  class RISCVISegLoadMasked
        : DefaultAttrsIntrinsic<[llvm_any_ty],
                                [LLVMMatchType<0>, llvm_anyptr_ty,
                                 llvm_anyvector_ty, llvm_anyvector_ty,
                                 llvm_anyint_ty, LLVMMatchType<4>, LLVMMatchType<4>],
                    [ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>,
                     NoCapture<ArgIndex<1>>, IntrReadMem]>, RISCVVIntrinsic {
    let VLOperand = 4;
  }

  // For unit stride segment store
  // Input: (value, pointer, vl, sew)
  class RISCVUSSegStore
        : DefaultAttrsIntrinsic<[],
                                [llvm_any_ty, llvm_anyptr_ty, llvm_anyint_ty,
                                 LLVMMatchType<2>],
                    [ImmArg<ArgIndex<3>>, NoCapture<ArgIndex<1>>, IntrWriteMem,
                     IntrArgMemOnly]>,
          RISCVVIntrinsic {
    let VLOperand = 2;
  }
  // For unit stride segment store with mask
  // Input: (value, pointer, mask, vl, sew)
  class RISCVUSSegStoreMasked
````
- **L1093 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1093 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1094 EN**: Closes the current lexical scope or compound statement.
  **L1094 CN**: 结束当前词法作用域或复合语句块。
- **L1095 EN**: Comment explains nearby logic, invariants, or intent: `For indexed segment load with mask`.
  **L1095 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For indexed segment load with mask`。
- **L1096 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, pointer, index, mask, vl, policy, sew)`.
  **L1096 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, pointer, index, mask, vl, policy, sew)`。
- **L1097 EN**: Declares class `RISCVISegLoadMasked`.
  **L1097 CN**: 声明 class `RISCVISegLoadMasked`。
- **L1098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L1098 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L1099 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyptr_ty,`.
  **L1099 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyptr_ty,`。
- **L1100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, llvm_anyvector_ty,`.
  **L1100 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, llvm_anyvector_ty,`。
- **L1101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<4>, LLVMMatchType<4>],`.
  **L1101 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<4>, LLVMMatchType<4>],`。
- **L1102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>,`.
  **L1102 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>,`。
- **L1103 EN**: Continues the surrounding expression or declaration: `NoCapture<ArgIndex<1>>, IntrReadMem]>, RISCVVIntrinsic {`.
  **L1103 CN**: 继续构造周围的表达式或声明：`NoCapture<ArgIndex<1>>, IntrReadMem]>, RISCVVIntrinsic {`。
- **L1104 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1104 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1105 EN**: Closes the current lexical scope or compound statement.
  **L1105 CN**: 结束当前词法作用域或复合语句块。
- **L1106 EN**: Blank line separating nearby declarations or logic blocks.
  **L1106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1107 EN**: Comment explains nearby logic, invariants, or intent: `For unit stride segment store`.
  **L1107 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unit stride segment store`。
- **L1108 EN**: Comment explains nearby logic, invariants, or intent: `Input: (value, pointer, vl, sew)`.
  **L1108 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (value, pointer, vl, sew)`。
- **L1109 EN**: Declares class `RISCVUSSegStore`.
  **L1109 CN**: 声明 class `RISCVUSSegStore`。
- **L1110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1110 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_anyptr_ty, llvm_anyint_ty,`.
  **L1111 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_anyptr_ty, llvm_anyint_ty,`。
- **L1112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>],`.
  **L1112 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>],`。
- **L1113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<3>>, NoCapture<ArgIndex<1>>, IntrWriteMem,`.
  **L1113 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<3>>, NoCapture<ArgIndex<1>>, IntrWriteMem,`。
- **L1114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrArgMemOnly]>,`.
  **L1114 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrArgMemOnly]>,`。
- **L1115 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L1115 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L1116 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1116 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1117 EN**: Closes the current lexical scope or compound statement.
  **L1117 CN**: 结束当前词法作用域或复合语句块。
- **L1118 EN**: Comment explains nearby logic, invariants, or intent: `For unit stride segment store with mask`.
  **L1118 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For unit stride segment store with mask`。
- **L1119 EN**: Comment explains nearby logic, invariants, or intent: `Input: (value, pointer, mask, vl, sew)`.
  **L1119 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (value, pointer, mask, vl, sew)`。
- **L1120 EN**: Declares class `RISCVUSSegStoreMasked`.
  **L1120 CN**: 声明 class `RISCVUSSegStoreMasked`。

### Lines 1121-1148

````tablegen
        : DefaultAttrsIntrinsic<[],
                                [llvm_any_ty, llvm_anyptr_ty,
                                 llvm_anyvector_ty, llvm_anyint_ty, LLVMMatchType<3>],
                    [ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrWriteMem,
                     IntrArgMemOnly]>,
          RISCVVIntrinsic {
    let VLOperand = 3;
  }

  // For stride segment store
  // Input: (value, pointer, offset, vl, sew)
  class RISCVSSegStore
        : DefaultAttrsIntrinsic<[],
                                [llvm_any_ty, llvm_anyptr_ty, llvm_anyint_ty,
                                 LLVMMatchType<2>, LLVMMatchType<2>],
                    [ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrWriteMem]>,
          RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For stride segment store with mask
  // Input: (value, pointer, offset, mask, vl, sew)
  class RISCVSSegStoreMasked
        : DefaultAttrsIntrinsic<[],
                                [llvm_any_ty, llvm_anyptr_ty, llvm_anyint_ty,
                                 llvm_anyvector_ty, LLVMMatchType<2>,
                                 LLVMMatchType<2>],
                    [ImmArg<ArgIndex<5>>, NoCapture<ArgIndex<1>>, IntrWriteMem]>,
          RISCVVIntrinsic {
````
- **L1121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1121 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_anyptr_ty,`.
  **L1122 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_anyptr_ty,`。
- **L1123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, llvm_anyint_ty, LLVMMatchType<3>],`.
  **L1123 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, llvm_anyint_ty, LLVMMatchType<3>],`。
- **L1124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrWriteMem,`.
  **L1124 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrWriteMem,`。
- **L1125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrArgMemOnly]>,`.
  **L1125 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrArgMemOnly]>,`。
- **L1126 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L1126 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L1127 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1127 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1128 EN**: Closes the current lexical scope or compound statement.
  **L1128 CN**: 结束当前词法作用域或复合语句块。
- **L1129 EN**: Blank line separating nearby declarations or logic blocks.
  **L1129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1130 EN**: Comment explains nearby logic, invariants, or intent: `For stride segment store`.
  **L1130 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For stride segment store`。
- **L1131 EN**: Comment explains nearby logic, invariants, or intent: `Input: (value, pointer, offset, vl, sew)`.
  **L1131 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (value, pointer, offset, vl, sew)`。
- **L1132 EN**: Declares class `RISCVSSegStore`.
  **L1132 CN**: 声明 class `RISCVSSegStore`。
- **L1133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1133 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_anyptr_ty, llvm_anyint_ty,`.
  **L1134 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_anyptr_ty, llvm_anyint_ty,`。
- **L1135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>, LLVMMatchType<2>],`.
  **L1135 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>, LLVMMatchType<2>],`。
- **L1136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrWriteMem]>,`.
  **L1136 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrWriteMem]>,`。
- **L1137 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L1137 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L1138 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1138 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1139 EN**: Closes the current lexical scope or compound statement.
  **L1139 CN**: 结束当前词法作用域或复合语句块。
- **L1140 EN**: Comment explains nearby logic, invariants, or intent: `For stride segment store with mask`.
  **L1140 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For stride segment store with mask`。
- **L1141 EN**: Comment explains nearby logic, invariants, or intent: `Input: (value, pointer, offset, mask, vl, sew)`.
  **L1141 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (value, pointer, offset, mask, vl, sew)`。
- **L1142 EN**: Declares class `RISCVSSegStoreMasked`.
  **L1142 CN**: 声明 class `RISCVSSegStoreMasked`。
- **L1143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1143 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_anyptr_ty, llvm_anyint_ty,`.
  **L1144 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_anyptr_ty, llvm_anyint_ty,`。
- **L1145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<2>,`.
  **L1145 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<2>,`。
- **L1146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<2>],`.
  **L1146 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<2>],`。
- **L1147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<5>>, NoCapture<ArgIndex<1>>, IntrWriteMem]>,`.
  **L1147 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<5>>, NoCapture<ArgIndex<1>>, IntrWriteMem]>,`。
- **L1148 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L1148 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。

### Lines 1149-1176

````tablegen
    let VLOperand = 4;
  }

  // For indexed segment store
  // Input: (value, pointer, offset, vl, sew)
  class RISCVISegStore
        : DefaultAttrsIntrinsic<[],
                                [llvm_any_ty, llvm_anyptr_ty, llvm_anyvector_ty,
                                 llvm_anyint_ty, LLVMMatchType<3>],
                    [ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrWriteMem]>,
          RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // For indexed segment store with mask
  // Input: (value, pointer, offset, mask, vl, sew)
  class RISCVISegStoreMasked
        : DefaultAttrsIntrinsic<[],
                                [llvm_any_ty, llvm_anyptr_ty, llvm_anyvector_ty,
                                 llvm_anyvector_ty, llvm_anyint_ty,
                                 LLVMMatchType<4>],
                    [ImmArg<ArgIndex<5>>, NoCapture<ArgIndex<1>>, IntrWriteMem]>,
          RISCVVIntrinsic {
    let VLOperand = 4;
  }

  multiclass RISCVUSLoad {
    def "int_riscv_" # NAME : RISCVUSLoad;
    def "int_riscv_" # NAME # "_mask" : RISCVUSLoadMasked;
````
- **L1149 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1149 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1150 EN**: Closes the current lexical scope or compound statement.
  **L1150 CN**: 结束当前词法作用域或复合语句块。
- **L1151 EN**: Blank line separating nearby declarations or logic blocks.
  **L1151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1152 EN**: Comment explains nearby logic, invariants, or intent: `For indexed segment store`.
  **L1152 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For indexed segment store`。
- **L1153 EN**: Comment explains nearby logic, invariants, or intent: `Input: (value, pointer, offset, vl, sew)`.
  **L1153 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (value, pointer, offset, vl, sew)`。
- **L1154 EN**: Declares class `RISCVISegStore`.
  **L1154 CN**: 声明 class `RISCVISegStore`。
- **L1155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1155 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_anyptr_ty, llvm_anyvector_ty,`.
  **L1156 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_anyptr_ty, llvm_anyvector_ty,`。
- **L1157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<3>],`.
  **L1157 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<3>],`。
- **L1158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrWriteMem]>,`.
  **L1158 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<4>>, NoCapture<ArgIndex<1>>, IntrWriteMem]>,`。
- **L1159 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L1159 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L1160 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1160 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1161 EN**: Closes the current lexical scope or compound statement.
  **L1161 CN**: 结束当前词法作用域或复合语句块。
- **L1162 EN**: Comment explains nearby logic, invariants, or intent: `For indexed segment store with mask`.
  **L1162 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For indexed segment store with mask`。
- **L1163 EN**: Comment explains nearby logic, invariants, or intent: `Input: (value, pointer, offset, mask, vl, sew)`.
  **L1163 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (value, pointer, offset, mask, vl, sew)`。
- **L1164 EN**: Declares class `RISCVISegStoreMasked`.
  **L1164 CN**: 声明 class `RISCVISegStoreMasked`。
- **L1165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1165 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_anyptr_ty, llvm_anyvector_ty,`.
  **L1166 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_anyptr_ty, llvm_anyvector_ty,`。
- **L1167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, llvm_anyint_ty,`.
  **L1167 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, llvm_anyint_ty,`。
- **L1168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<4>],`.
  **L1168 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<4>],`。
- **L1169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<5>>, NoCapture<ArgIndex<1>>, IntrWriteMem]>,`.
  **L1169 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<5>>, NoCapture<ArgIndex<1>>, IntrWriteMem]>,`。
- **L1170 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L1170 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L1171 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1171 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1172 EN**: Closes the current lexical scope or compound statement.
  **L1172 CN**: 结束当前词法作用域或复合语句块。
- **L1173 EN**: Blank line separating nearby declarations or logic blocks.
  **L1173 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1174 EN**: Declares TableGen multiclass `RISCVUSLoad`.
  **L1174 CN**: 声明 TableGen multiclass `RISCVUSLoad`。
- **L1175 EN**: Declares TableGen def `"int_riscv_"`.
  **L1175 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1176 EN**: Declares TableGen def `"int_riscv_"`.
  **L1176 CN**: 声明 TableGen def `"int_riscv_"`。

### Lines 1177-1204

````tablegen
  }
  multiclass RISCVUSLoadFF {
    def "int_riscv_" # NAME : RISCVUSLoadFF;
    def "int_riscv_" # NAME # "_mask" : RISCVUSLoadFFMasked;
  }
  multiclass RISCVSLoad {
    def "int_riscv_" # NAME : RISCVSLoad;
    def "int_riscv_" # NAME # "_mask" : RISCVSLoadMasked;
  }
  multiclass RISCVILoad {
    def "int_riscv_" # NAME : RISCVILoad;
    def "int_riscv_" # NAME # "_mask" : RISCVILoadMasked;
  }
  multiclass RISCVUSStore {
    def "int_riscv_" # NAME : RISCVUSStore;
    def "int_riscv_" # NAME # "_mask" : RISCVUSStoreMasked;
  }
  multiclass RISCVSStore {
    def "int_riscv_" # NAME : RISCVSStore;
    def "int_riscv_" # NAME # "_mask" : RISCVSStoreMasked;
  }

  multiclass RISCVIStore {
    def "int_riscv_" # NAME : RISCVIStore;
    def "int_riscv_" # NAME # "_mask" : RISCVIStoreMasked;
  }
  multiclass RISCVUnaryAA {
    def "int_riscv_" # NAME : RISCVUnaryAAUnMasked;
````
- **L1177 EN**: Closes the current lexical scope or compound statement.
  **L1177 CN**: 结束当前词法作用域或复合语句块。
- **L1178 EN**: Declares TableGen multiclass `RISCVUSLoadFF`.
  **L1178 CN**: 声明 TableGen multiclass `RISCVUSLoadFF`。
- **L1179 EN**: Declares TableGen def `"int_riscv_"`.
  **L1179 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1180 EN**: Declares TableGen def `"int_riscv_"`.
  **L1180 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1181 EN**: Closes the current lexical scope or compound statement.
  **L1181 CN**: 结束当前词法作用域或复合语句块。
- **L1182 EN**: Declares TableGen multiclass `RISCVSLoad`.
  **L1182 CN**: 声明 TableGen multiclass `RISCVSLoad`。
- **L1183 EN**: Declares TableGen def `"int_riscv_"`.
  **L1183 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1184 EN**: Declares TableGen def `"int_riscv_"`.
  **L1184 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1185 EN**: Closes the current lexical scope or compound statement.
  **L1185 CN**: 结束当前词法作用域或复合语句块。
- **L1186 EN**: Declares TableGen multiclass `RISCVILoad`.
  **L1186 CN**: 声明 TableGen multiclass `RISCVILoad`。
- **L1187 EN**: Declares TableGen def `"int_riscv_"`.
  **L1187 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1188 EN**: Declares TableGen def `"int_riscv_"`.
  **L1188 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1189 EN**: Closes the current lexical scope or compound statement.
  **L1189 CN**: 结束当前词法作用域或复合语句块。
- **L1190 EN**: Declares TableGen multiclass `RISCVUSStore`.
  **L1190 CN**: 声明 TableGen multiclass `RISCVUSStore`。
- **L1191 EN**: Declares TableGen def `"int_riscv_"`.
  **L1191 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1192 EN**: Declares TableGen def `"int_riscv_"`.
  **L1192 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1193 EN**: Closes the current lexical scope or compound statement.
  **L1193 CN**: 结束当前词法作用域或复合语句块。
- **L1194 EN**: Declares TableGen multiclass `RISCVSStore`.
  **L1194 CN**: 声明 TableGen multiclass `RISCVSStore`。
- **L1195 EN**: Declares TableGen def `"int_riscv_"`.
  **L1195 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1196 EN**: Declares TableGen def `"int_riscv_"`.
  **L1196 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1197 EN**: Closes the current lexical scope or compound statement.
  **L1197 CN**: 结束当前词法作用域或复合语句块。
- **L1198 EN**: Blank line separating nearby declarations or logic blocks.
  **L1198 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1199 EN**: Declares TableGen multiclass `RISCVIStore`.
  **L1199 CN**: 声明 TableGen multiclass `RISCVIStore`。
- **L1200 EN**: Declares TableGen def `"int_riscv_"`.
  **L1200 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1201 EN**: Declares TableGen def `"int_riscv_"`.
  **L1201 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1202 EN**: Closes the current lexical scope or compound statement.
  **L1202 CN**: 结束当前词法作用域或复合语句块。
- **L1203 EN**: Declares TableGen multiclass `RISCVUnaryAA`.
  **L1203 CN**: 声明 TableGen multiclass `RISCVUnaryAA`。
- **L1204 EN**: Declares TableGen def `"int_riscv_"`.
  **L1204 CN**: 声明 TableGen def `"int_riscv_"`。

### Lines 1205-1232

````tablegen
    def "int_riscv_" # NAME # "_mask" : RISCVUnaryAAMasked;
  }
  multiclass RISCVUnaryAARoundingMode {
    def "int_riscv_" # NAME : RISCVUnaryAAUnMaskedRoundingMode;
    def "int_riscv_" # NAME # "_mask" : RISCVUnaryAAMaskedRoundingMode;
  }
  multiclass RISCVUnaryAB {
    def "int_riscv_" # NAME : RISCVUnaryABUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVUnaryABMasked;
  }
  // AAX means the destination type(A) is the same as the first source
  // type(A). X means any type for the second source operand.
  multiclass RISCVBinaryAAX {
    def "int_riscv_" # NAME : RISCVBinaryAAXUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVBinaryAAXMasked;
  }
  multiclass RISCVBinaryAAXRoundingMode {
    def "int_riscv_" # NAME : RISCVBinaryAAXUnMaskedRoundingMode;
    def "int_riscv_" # NAME # "_mask" : RISCVBinaryAAXMaskedRoundingMode;
  }
  multiclass RISCVBinaryAAA {
    def "int_riscv_" # NAME : RISCVBinaryAAAUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVBinaryAAAMasked;
  }
  // Like RISCVBinaryAAX, but the second operand is used a shift amount so it
  // must be a vector or an XLen scalar.
  multiclass RISCVBinaryAAShift {
    def "int_riscv_" # NAME : RISCVBinaryAAShiftUnMasked;
````
- **L1205 EN**: Declares TableGen def `"int_riscv_"`.
  **L1205 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1206 EN**: Closes the current lexical scope or compound statement.
  **L1206 CN**: 结束当前词法作用域或复合语句块。
- **L1207 EN**: Declares TableGen multiclass `RISCVUnaryAARoundingMode`.
  **L1207 CN**: 声明 TableGen multiclass `RISCVUnaryAARoundingMode`。
- **L1208 EN**: Declares TableGen def `"int_riscv_"`.
  **L1208 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1209 EN**: Declares TableGen def `"int_riscv_"`.
  **L1209 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1210 EN**: Closes the current lexical scope or compound statement.
  **L1210 CN**: 结束当前词法作用域或复合语句块。
- **L1211 EN**: Declares TableGen multiclass `RISCVUnaryAB`.
  **L1211 CN**: 声明 TableGen multiclass `RISCVUnaryAB`。
- **L1212 EN**: Declares TableGen def `"int_riscv_"`.
  **L1212 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1213 EN**: Declares TableGen def `"int_riscv_"`.
  **L1213 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1214 EN**: Closes the current lexical scope or compound statement.
  **L1214 CN**: 结束当前词法作用域或复合语句块。
- **L1215 EN**: Comment explains nearby logic, invariants, or intent: `AAX means the destination type(A) is the same as the first source`.
  **L1215 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AAX means the destination type(A) is the same as the first source`。
- **L1216 EN**: Comment explains nearby logic, invariants, or intent: `type(A). X means any type for the second source operand.`.
  **L1216 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`type(A). X means any type for the second source operand.`。
- **L1217 EN**: Declares TableGen multiclass `RISCVBinaryAAX`.
  **L1217 CN**: 声明 TableGen multiclass `RISCVBinaryAAX`。
- **L1218 EN**: Declares TableGen def `"int_riscv_"`.
  **L1218 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1219 EN**: Declares TableGen def `"int_riscv_"`.
  **L1219 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1220 EN**: Closes the current lexical scope or compound statement.
  **L1220 CN**: 结束当前词法作用域或复合语句块。
- **L1221 EN**: Declares TableGen multiclass `RISCVBinaryAAXRoundingMode`.
  **L1221 CN**: 声明 TableGen multiclass `RISCVBinaryAAXRoundingMode`。
- **L1222 EN**: Declares TableGen def `"int_riscv_"`.
  **L1222 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1223 EN**: Declares TableGen def `"int_riscv_"`.
  **L1223 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1224 EN**: Closes the current lexical scope or compound statement.
  **L1224 CN**: 结束当前词法作用域或复合语句块。
- **L1225 EN**: Declares TableGen multiclass `RISCVBinaryAAA`.
  **L1225 CN**: 声明 TableGen multiclass `RISCVBinaryAAA`。
- **L1226 EN**: Declares TableGen def `"int_riscv_"`.
  **L1226 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1227 EN**: Declares TableGen def `"int_riscv_"`.
  **L1227 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1228 EN**: Closes the current lexical scope or compound statement.
  **L1228 CN**: 结束当前词法作用域或复合语句块。
- **L1229 EN**: Comment explains nearby logic, invariants, or intent: `Like RISCVBinaryAAX, but the second operand is used a shift amount so it`.
  **L1229 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Like RISCVBinaryAAX, but the second operand is used a shift amount so it`。
- **L1230 EN**: Comment explains nearby logic, invariants, or intent: `must be a vector or an XLen scalar.`.
  **L1230 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`must be a vector or an XLen scalar.`。
- **L1231 EN**: Declares TableGen multiclass `RISCVBinaryAAShift`.
  **L1231 CN**: 声明 TableGen multiclass `RISCVBinaryAAShift`。
- **L1232 EN**: Declares TableGen def `"int_riscv_"`.
  **L1232 CN**: 声明 TableGen def `"int_riscv_"`。

### Lines 1233-1260

````tablegen
    def "int_riscv_" # NAME # "_mask" : RISCVBinaryAAShiftMasked;
  }
  multiclass RISCVRGatherVV {
    def "int_riscv_" # NAME : RISCVRGatherVVUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVRGatherVVMasked;
  }
  multiclass RISCVRGatherVX {
    def "int_riscv_" # NAME : RISCVGatherVXUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVGatherVXMasked;
  }
  multiclass RISCVRGatherEI16VV {
    def "int_riscv_" # NAME : RISCVRGatherEI16VVUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVRGatherEI16VVMasked;
  }
  // ABX means the destination type(A) is different from the first source
  // type(B). X means any type for the second source operand.
  multiclass RISCVBinaryABX {
    def "int_riscv_" # NAME : RISCVBinaryABXUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVBinaryABXMasked;
  }
  multiclass RISCVBinaryABXRoundingMode {
    def "int_riscv_" # NAME : RISCVBinaryABXUnMaskedRoundingMode;
    def "int_riscv_" # NAME # "_mask" : RISCVBinaryABXMaskedRoundingMode;
  }
  // Like RISCVBinaryABX, but the second operand is used a shift amount so it
  // must be a vector or an XLen scalar.
  multiclass RISCVBinaryABShift {
    def "int_riscv_" # NAME : RISCVBinaryABShiftUnMasked;
````
- **L1233 EN**: Declares TableGen def `"int_riscv_"`.
  **L1233 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1234 EN**: Closes the current lexical scope or compound statement.
  **L1234 CN**: 结束当前词法作用域或复合语句块。
- **L1235 EN**: Declares TableGen multiclass `RISCVRGatherVV`.
  **L1235 CN**: 声明 TableGen multiclass `RISCVRGatherVV`。
- **L1236 EN**: Declares TableGen def `"int_riscv_"`.
  **L1236 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1237 EN**: Declares TableGen def `"int_riscv_"`.
  **L1237 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1238 EN**: Closes the current lexical scope or compound statement.
  **L1238 CN**: 结束当前词法作用域或复合语句块。
- **L1239 EN**: Declares TableGen multiclass `RISCVRGatherVX`.
  **L1239 CN**: 声明 TableGen multiclass `RISCVRGatherVX`。
- **L1240 EN**: Declares TableGen def `"int_riscv_"`.
  **L1240 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1241 EN**: Declares TableGen def `"int_riscv_"`.
  **L1241 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1242 EN**: Closes the current lexical scope or compound statement.
  **L1242 CN**: 结束当前词法作用域或复合语句块。
- **L1243 EN**: Declares TableGen multiclass `RISCVRGatherEI16VV`.
  **L1243 CN**: 声明 TableGen multiclass `RISCVRGatherEI16VV`。
- **L1244 EN**: Declares TableGen def `"int_riscv_"`.
  **L1244 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1245 EN**: Declares TableGen def `"int_riscv_"`.
  **L1245 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1246 EN**: Closes the current lexical scope or compound statement.
  **L1246 CN**: 结束当前词法作用域或复合语句块。
- **L1247 EN**: Comment explains nearby logic, invariants, or intent: `ABX means the destination type(A) is different from the first source`.
  **L1247 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ABX means the destination type(A) is different from the first source`。
- **L1248 EN**: Comment explains nearby logic, invariants, or intent: `type(B). X means any type for the second source operand.`.
  **L1248 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`type(B). X means any type for the second source operand.`。
- **L1249 EN**: Declares TableGen multiclass `RISCVBinaryABX`.
  **L1249 CN**: 声明 TableGen multiclass `RISCVBinaryABX`。
- **L1250 EN**: Declares TableGen def `"int_riscv_"`.
  **L1250 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1251 EN**: Declares TableGen def `"int_riscv_"`.
  **L1251 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1252 EN**: Closes the current lexical scope or compound statement.
  **L1252 CN**: 结束当前词法作用域或复合语句块。
- **L1253 EN**: Declares TableGen multiclass `RISCVBinaryABXRoundingMode`.
  **L1253 CN**: 声明 TableGen multiclass `RISCVBinaryABXRoundingMode`。
- **L1254 EN**: Declares TableGen def `"int_riscv_"`.
  **L1254 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1255 EN**: Declares TableGen def `"int_riscv_"`.
  **L1255 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1256 EN**: Closes the current lexical scope or compound statement.
  **L1256 CN**: 结束当前词法作用域或复合语句块。
- **L1257 EN**: Comment explains nearby logic, invariants, or intent: `Like RISCVBinaryABX, but the second operand is used a shift amount so it`.
  **L1257 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Like RISCVBinaryABX, but the second operand is used a shift amount so it`。
- **L1258 EN**: Comment explains nearby logic, invariants, or intent: `must be a vector or an XLen scalar.`.
  **L1258 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`must be a vector or an XLen scalar.`。
- **L1259 EN**: Declares TableGen multiclass `RISCVBinaryABShift`.
  **L1259 CN**: 声明 TableGen multiclass `RISCVBinaryABShift`。
- **L1260 EN**: Declares TableGen def `"int_riscv_"`.
  **L1260 CN**: 声明 TableGen def `"int_riscv_"`。

### Lines 1261-1288

````tablegen
    def "int_riscv_" # NAME # "_mask" : RISCVBinaryABShiftMasked;
  }
  multiclass RISCVBinaryWithV0 {
    def "int_riscv_" # NAME : RISCVBinaryWithV0;
  }
  multiclass RISCVBinaryMaskOutWithV0 {
    def "int_riscv_" # NAME : RISCVBinaryMOutWithV0;
  }
  multiclass RISCVBinaryMaskOut {
    def "int_riscv_" # NAME : RISCVBinaryMOut;
  }
  multiclass RISCVSaturatingBinaryAAX {
    def "int_riscv_" # NAME : RISCVSaturatingBinaryAAXUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVSaturatingBinaryAAXMasked;
  }
  multiclass RISCVSaturatingBinaryAAXRoundingMode {
    def "int_riscv_" # NAME : RISCVSaturatingBinaryAAXUnMaskedRoundingMode;
    def "int_riscv_" # NAME # "_mask" : RISCVSaturatingBinaryAAXMaskedRoundingMode;
  }
  multiclass RISCVSaturatingBinaryAAShiftRoundingMode {
    def "int_riscv_" # NAME : RISCVSaturatingBinaryAAShiftUnMaskedRoundingMode;
    def "int_riscv_" # NAME # "_mask" : RISCVSaturatingBinaryAAShiftMaskedRoundingMode;
  }
  multiclass RISCVSaturatingBinaryABShiftRoundingMode {
    def "int_riscv_" # NAME : RISCVSaturatingBinaryABShiftUnMaskedRoundingMode;
    def "int_riscv_" # NAME # "_mask" : RISCVSaturatingBinaryABShiftMaskedRoundingMode;
  }
  multiclass RVVSlide {
````
- **L1261 EN**: Declares TableGen def `"int_riscv_"`.
  **L1261 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1262 EN**: Closes the current lexical scope or compound statement.
  **L1262 CN**: 结束当前词法作用域或复合语句块。
- **L1263 EN**: Declares TableGen multiclass `RISCVBinaryWithV0`.
  **L1263 CN**: 声明 TableGen multiclass `RISCVBinaryWithV0`。
- **L1264 EN**: Declares TableGen def `"int_riscv_"`.
  **L1264 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1265 EN**: Closes the current lexical scope or compound statement.
  **L1265 CN**: 结束当前词法作用域或复合语句块。
- **L1266 EN**: Declares TableGen multiclass `RISCVBinaryMaskOutWithV0`.
  **L1266 CN**: 声明 TableGen multiclass `RISCVBinaryMaskOutWithV0`。
- **L1267 EN**: Declares TableGen def `"int_riscv_"`.
  **L1267 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1268 EN**: Closes the current lexical scope or compound statement.
  **L1268 CN**: 结束当前词法作用域或复合语句块。
- **L1269 EN**: Declares TableGen multiclass `RISCVBinaryMaskOut`.
  **L1269 CN**: 声明 TableGen multiclass `RISCVBinaryMaskOut`。
- **L1270 EN**: Declares TableGen def `"int_riscv_"`.
  **L1270 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1271 EN**: Closes the current lexical scope or compound statement.
  **L1271 CN**: 结束当前词法作用域或复合语句块。
- **L1272 EN**: Declares TableGen multiclass `RISCVSaturatingBinaryAAX`.
  **L1272 CN**: 声明 TableGen multiclass `RISCVSaturatingBinaryAAX`。
- **L1273 EN**: Declares TableGen def `"int_riscv_"`.
  **L1273 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1274 EN**: Declares TableGen def `"int_riscv_"`.
  **L1274 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1275 EN**: Closes the current lexical scope or compound statement.
  **L1275 CN**: 结束当前词法作用域或复合语句块。
- **L1276 EN**: Declares TableGen multiclass `RISCVSaturatingBinaryAAXRoundingMode`.
  **L1276 CN**: 声明 TableGen multiclass `RISCVSaturatingBinaryAAXRoundingMode`。
- **L1277 EN**: Declares TableGen def `"int_riscv_"`.
  **L1277 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1278 EN**: Declares TableGen def `"int_riscv_"`.
  **L1278 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1279 EN**: Closes the current lexical scope or compound statement.
  **L1279 CN**: 结束当前词法作用域或复合语句块。
- **L1280 EN**: Declares TableGen multiclass `RISCVSaturatingBinaryAAShiftRoundingMode`.
  **L1280 CN**: 声明 TableGen multiclass `RISCVSaturatingBinaryAAShiftRoundingMode`。
- **L1281 EN**: Declares TableGen def `"int_riscv_"`.
  **L1281 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1282 EN**: Declares TableGen def `"int_riscv_"`.
  **L1282 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1283 EN**: Closes the current lexical scope or compound statement.
  **L1283 CN**: 结束当前词法作用域或复合语句块。
- **L1284 EN**: Declares TableGen multiclass `RISCVSaturatingBinaryABShiftRoundingMode`.
  **L1284 CN**: 声明 TableGen multiclass `RISCVSaturatingBinaryABShiftRoundingMode`。
- **L1285 EN**: Declares TableGen def `"int_riscv_"`.
  **L1285 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1286 EN**: Declares TableGen def `"int_riscv_"`.
  **L1286 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1287 EN**: Closes the current lexical scope or compound statement.
  **L1287 CN**: 结束当前词法作用域或复合语句块。
- **L1288 EN**: Declares TableGen multiclass `RVVSlide`.
  **L1288 CN**: 声明 TableGen multiclass `RVVSlide`。

### Lines 1289-1316

````tablegen
    def "int_riscv_" # NAME : RVVSlideUnMasked;
    def "int_riscv_" # NAME # "_mask" : RVVSlideMasked;
  }
  multiclass RISCVTernaryAAXA {
    def "int_riscv_" # NAME : RISCVTernaryAAXAUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVTernaryAAXAMasked;
  }
  multiclass RISCVTernaryAAXARoundingMode {
    def "int_riscv_" # NAME : RISCVTernaryAAXAUnMaskedRoundingMode;
    def "int_riscv_" # NAME # "_mask" : RISCVTernaryAAXAMaskedRoundingMode;
  }
  multiclass RISCVCompare {
    def "int_riscv_" # NAME : RISCVCompareUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVCompareMasked;
  }
  multiclass RISCVClassify {
    def "int_riscv_" # NAME : RISCVClassifyUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVClassifyMasked;
  }
  multiclass RISCVTernaryWide {
    def "int_riscv_" # NAME : RISCVTernaryWideUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVTernaryWideMasked;
  }
  multiclass RISCVTernaryWideRoundingMode {
    def "int_riscv_" # NAME : RISCVTernaryWideUnMaskedRoundingMode;
    def "int_riscv_" # NAME # "_mask" : RISCVTernaryWideMaskedRoundingMode;
  }
  multiclass RISCVReduction {
````
- **L1289 EN**: Declares TableGen def `"int_riscv_"`.
  **L1289 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1290 EN**: Declares TableGen def `"int_riscv_"`.
  **L1290 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1291 EN**: Closes the current lexical scope or compound statement.
  **L1291 CN**: 结束当前词法作用域或复合语句块。
- **L1292 EN**: Declares TableGen multiclass `RISCVTernaryAAXA`.
  **L1292 CN**: 声明 TableGen multiclass `RISCVTernaryAAXA`。
- **L1293 EN**: Declares TableGen def `"int_riscv_"`.
  **L1293 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1294 EN**: Declares TableGen def `"int_riscv_"`.
  **L1294 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1295 EN**: Closes the current lexical scope or compound statement.
  **L1295 CN**: 结束当前词法作用域或复合语句块。
- **L1296 EN**: Declares TableGen multiclass `RISCVTernaryAAXARoundingMode`.
  **L1296 CN**: 声明 TableGen multiclass `RISCVTernaryAAXARoundingMode`。
- **L1297 EN**: Declares TableGen def `"int_riscv_"`.
  **L1297 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1298 EN**: Declares TableGen def `"int_riscv_"`.
  **L1298 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1299 EN**: Closes the current lexical scope or compound statement.
  **L1299 CN**: 结束当前词法作用域或复合语句块。
- **L1300 EN**: Declares TableGen multiclass `RISCVCompare`.
  **L1300 CN**: 声明 TableGen multiclass `RISCVCompare`。
- **L1301 EN**: Declares TableGen def `"int_riscv_"`.
  **L1301 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1302 EN**: Declares TableGen def `"int_riscv_"`.
  **L1302 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1303 EN**: Closes the current lexical scope or compound statement.
  **L1303 CN**: 结束当前词法作用域或复合语句块。
- **L1304 EN**: Declares TableGen multiclass `RISCVClassify`.
  **L1304 CN**: 声明 TableGen multiclass `RISCVClassify`。
- **L1305 EN**: Declares TableGen def `"int_riscv_"`.
  **L1305 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1306 EN**: Declares TableGen def `"int_riscv_"`.
  **L1306 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1307 EN**: Closes the current lexical scope or compound statement.
  **L1307 CN**: 结束当前词法作用域或复合语句块。
- **L1308 EN**: Declares TableGen multiclass `RISCVTernaryWide`.
  **L1308 CN**: 声明 TableGen multiclass `RISCVTernaryWide`。
- **L1309 EN**: Declares TableGen def `"int_riscv_"`.
  **L1309 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1310 EN**: Declares TableGen def `"int_riscv_"`.
  **L1310 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1311 EN**: Closes the current lexical scope or compound statement.
  **L1311 CN**: 结束当前词法作用域或复合语句块。
- **L1312 EN**: Declares TableGen multiclass `RISCVTernaryWideRoundingMode`.
  **L1312 CN**: 声明 TableGen multiclass `RISCVTernaryWideRoundingMode`。
- **L1313 EN**: Declares TableGen def `"int_riscv_"`.
  **L1313 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1314 EN**: Declares TableGen def `"int_riscv_"`.
  **L1314 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1315 EN**: Closes the current lexical scope or compound statement.
  **L1315 CN**: 结束当前词法作用域或复合语句块。
- **L1316 EN**: Declares TableGen multiclass `RISCVReduction`.
  **L1316 CN**: 声明 TableGen multiclass `RISCVReduction`。

### Lines 1317-1344

````tablegen
    def "int_riscv_" # NAME : RISCVReductionUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVReductionMasked;
  }
  multiclass RISCVReductionRoundingMode {
    def "int_riscv_" # NAME : RISCVReductionUnMaskedRoundingMode;
    def "int_riscv_" # NAME # "_mask" : RISCVReductionMaskedRoundingMode;
  }
  multiclass RISCVMaskedUnarySOut {
    def "int_riscv_" # NAME : RISCVMaskedUnarySOutUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVMaskedUnarySOutMasked;
  }
  multiclass RISCVMaskedUnaryMOut {
    def "int_riscv_" # NAME : RISCVUnaryUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVMaskedUnaryMOutMasked;
  }
  multiclass RISCVConversion {
    def "int_riscv_" #NAME :RISCVConversionUnMasked;
    def "int_riscv_" # NAME # "_mask" : RISCVConversionMasked;
  }
  multiclass RISCVConversionRoundingMode {
    def "int_riscv_" #NAME :RISCVConversionUnMaskedRoundingMode;
    def "int_riscv_" # NAME # "_mask" : RISCVConversionMaskedRoundingMode;
  }
  multiclass RISCVUSSegLoad {
    def "int_riscv_" # NAME : RISCVUSSegLoad;
    def "int_riscv_" # NAME # "_mask" : RISCVUSSegLoadMasked;
  }
  multiclass RISCVUSSegLoadFF {
````
- **L1317 EN**: Declares TableGen def `"int_riscv_"`.
  **L1317 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1318 EN**: Declares TableGen def `"int_riscv_"`.
  **L1318 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1319 EN**: Closes the current lexical scope or compound statement.
  **L1319 CN**: 结束当前词法作用域或复合语句块。
- **L1320 EN**: Declares TableGen multiclass `RISCVReductionRoundingMode`.
  **L1320 CN**: 声明 TableGen multiclass `RISCVReductionRoundingMode`。
- **L1321 EN**: Declares TableGen def `"int_riscv_"`.
  **L1321 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1322 EN**: Declares TableGen def `"int_riscv_"`.
  **L1322 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1323 EN**: Closes the current lexical scope or compound statement.
  **L1323 CN**: 结束当前词法作用域或复合语句块。
- **L1324 EN**: Declares TableGen multiclass `RISCVMaskedUnarySOut`.
  **L1324 CN**: 声明 TableGen multiclass `RISCVMaskedUnarySOut`。
- **L1325 EN**: Declares TableGen def `"int_riscv_"`.
  **L1325 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1326 EN**: Declares TableGen def `"int_riscv_"`.
  **L1326 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1327 EN**: Closes the current lexical scope or compound statement.
  **L1327 CN**: 结束当前词法作用域或复合语句块。
- **L1328 EN**: Declares TableGen multiclass `RISCVMaskedUnaryMOut`.
  **L1328 CN**: 声明 TableGen multiclass `RISCVMaskedUnaryMOut`。
- **L1329 EN**: Declares TableGen def `"int_riscv_"`.
  **L1329 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1330 EN**: Declares TableGen def `"int_riscv_"`.
  **L1330 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1331 EN**: Closes the current lexical scope or compound statement.
  **L1331 CN**: 结束当前词法作用域或复合语句块。
- **L1332 EN**: Declares TableGen multiclass `RISCVConversion`.
  **L1332 CN**: 声明 TableGen multiclass `RISCVConversion`。
- **L1333 EN**: Declares TableGen def `"int_riscv_"`.
  **L1333 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1334 EN**: Declares TableGen def `"int_riscv_"`.
  **L1334 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1335 EN**: Closes the current lexical scope or compound statement.
  **L1335 CN**: 结束当前词法作用域或复合语句块。
- **L1336 EN**: Declares TableGen multiclass `RISCVConversionRoundingMode`.
  **L1336 CN**: 声明 TableGen multiclass `RISCVConversionRoundingMode`。
- **L1337 EN**: Declares TableGen def `"int_riscv_"`.
  **L1337 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1338 EN**: Declares TableGen def `"int_riscv_"`.
  **L1338 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1339 EN**: Closes the current lexical scope or compound statement.
  **L1339 CN**: 结束当前词法作用域或复合语句块。
- **L1340 EN**: Declares TableGen multiclass `RISCVUSSegLoad`.
  **L1340 CN**: 声明 TableGen multiclass `RISCVUSSegLoad`。
- **L1341 EN**: Declares TableGen def `"int_riscv_"`.
  **L1341 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1342 EN**: Declares TableGen def `"int_riscv_"`.
  **L1342 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1343 EN**: Closes the current lexical scope or compound statement.
  **L1343 CN**: 结束当前词法作用域或复合语句块。
- **L1344 EN**: Declares TableGen multiclass `RISCVUSSegLoadFF`.
  **L1344 CN**: 声明 TableGen multiclass `RISCVUSSegLoadFF`。

### Lines 1345-1372

````tablegen
    def "int_riscv_" # NAME : RISCVUSSegLoadFF;
    def "int_riscv_" # NAME # "_mask" : RISCVUSSegLoadFFMasked;
  }
  multiclass RISCVSSegLoad {
    def "int_riscv_" # NAME : RISCVSSegLoad;
    def "int_riscv_" # NAME # "_mask" : RISCVSSegLoadMasked;
  }
  multiclass RISCVISegLoad {
    def "int_riscv_" # NAME : RISCVISegLoad;
    def "int_riscv_" # NAME # "_mask" : RISCVISegLoadMasked;
  }
  multiclass RISCVUSSegStore {
    def "int_riscv_" # NAME : RISCVUSSegStore;
    def "int_riscv_" # NAME # "_mask" : RISCVUSSegStoreMasked;
  }
  multiclass RISCVSSegStore {
    def "int_riscv_" # NAME : RISCVSSegStore;
    def "int_riscv_" # NAME # "_mask" : RISCVSSegStoreMasked;
  }
  multiclass RISCVISegStore {
    def "int_riscv_" # NAME : RISCVISegStore;
    def "int_riscv_" # NAME # "_mask" : RISCVISegStoreMasked;
  }

  //==-- Intrinsics to perform vector tuple subvector insertion/extraction --=//
  def int_riscv_tuple_insert
      : DefaultAttrsIntrinsic<[llvm_any_ty],
                              [LLVMMatchType<0>, llvm_anyvector_ty, llvm_i32_ty],
````
- **L1345 EN**: Declares TableGen def `"int_riscv_"`.
  **L1345 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1346 EN**: Declares TableGen def `"int_riscv_"`.
  **L1346 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1347 EN**: Closes the current lexical scope or compound statement.
  **L1347 CN**: 结束当前词法作用域或复合语句块。
- **L1348 EN**: Declares TableGen multiclass `RISCVSSegLoad`.
  **L1348 CN**: 声明 TableGen multiclass `RISCVSSegLoad`。
- **L1349 EN**: Declares TableGen def `"int_riscv_"`.
  **L1349 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1350 EN**: Declares TableGen def `"int_riscv_"`.
  **L1350 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1351 EN**: Closes the current lexical scope or compound statement.
  **L1351 CN**: 结束当前词法作用域或复合语句块。
- **L1352 EN**: Declares TableGen multiclass `RISCVISegLoad`.
  **L1352 CN**: 声明 TableGen multiclass `RISCVISegLoad`。
- **L1353 EN**: Declares TableGen def `"int_riscv_"`.
  **L1353 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1354 EN**: Declares TableGen def `"int_riscv_"`.
  **L1354 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1355 EN**: Closes the current lexical scope or compound statement.
  **L1355 CN**: 结束当前词法作用域或复合语句块。
- **L1356 EN**: Declares TableGen multiclass `RISCVUSSegStore`.
  **L1356 CN**: 声明 TableGen multiclass `RISCVUSSegStore`。
- **L1357 EN**: Declares TableGen def `"int_riscv_"`.
  **L1357 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1358 EN**: Declares TableGen def `"int_riscv_"`.
  **L1358 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1359 EN**: Closes the current lexical scope or compound statement.
  **L1359 CN**: 结束当前词法作用域或复合语句块。
- **L1360 EN**: Declares TableGen multiclass `RISCVSSegStore`.
  **L1360 CN**: 声明 TableGen multiclass `RISCVSSegStore`。
- **L1361 EN**: Declares TableGen def `"int_riscv_"`.
  **L1361 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1362 EN**: Declares TableGen def `"int_riscv_"`.
  **L1362 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1363 EN**: Closes the current lexical scope or compound statement.
  **L1363 CN**: 结束当前词法作用域或复合语句块。
- **L1364 EN**: Declares TableGen multiclass `RISCVISegStore`.
  **L1364 CN**: 声明 TableGen multiclass `RISCVISegStore`。
- **L1365 EN**: Declares TableGen def `"int_riscv_"`.
  **L1365 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1366 EN**: Declares TableGen def `"int_riscv_"`.
  **L1366 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1367 EN**: Closes the current lexical scope or compound statement.
  **L1367 CN**: 结束当前词法作用域或复合语句块。
- **L1368 EN**: Blank line separating nearby declarations or logic blocks.
  **L1368 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1369 EN**: Comment explains nearby logic, invariants, or intent: `==-- Intrinsics to perform vector tuple subvector insertion/extraction --=//`.
  **L1369 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`==-- Intrinsics to perform vector tuple subvector insertion/extraction --=//`。
- **L1370 EN**: Declares TableGen def `int_riscv_tuple_insert`.
  **L1370 CN**: 声明 TableGen def `int_riscv_tuple_insert`。
- **L1371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L1371 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L1372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, llvm_i32_ty],`.
  **L1372 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, llvm_i32_ty],`。

### Lines 1373-1400

````tablegen
                              [IntrNoMem, IntrSpeculatable, ImmArg<ArgIndex<2>>]>;

  def int_riscv_tuple_extract
      : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                              [llvm_any_ty, llvm_i32_ty],
                              [IntrNoMem, IntrSpeculatable, ImmArg<ArgIndex<1>>]>;

  defm vle : RISCVUSLoad;
  defm vleff : RISCVUSLoadFF;
  defm vse : RISCVUSStore;
  defm vlse: RISCVSLoad;
  defm vsse: RISCVSStore;
  defm vluxei : RISCVILoad;
  defm vloxei : RISCVILoad;
  defm vsoxei : RISCVIStore;
  defm vsuxei : RISCVIStore;

  def int_riscv_vlm : RISCVUSMLoad;
  def int_riscv_vsm : RISCVUSStore;

  defm vadd : RISCVBinaryAAX;
  defm vsub : RISCVBinaryAAX;
  defm vrsub : RISCVBinaryAAX;

  defm vwaddu : RISCVBinaryABX;
  defm vwadd : RISCVBinaryABX;
  defm vwaddu_w : RISCVBinaryAAX;
  defm vwadd_w : RISCVBinaryAAX;
````
- **L1373 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable, ImmArg<ArgIndex<2>>]>;`.
  **L1373 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable, ImmArg<ArgIndex<2>>]>;`。
- **L1374 EN**: Blank line separating nearby declarations or logic blocks.
  **L1374 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1375 EN**: Declares TableGen def `int_riscv_tuple_extract`.
  **L1375 CN**: 声明 TableGen def `int_riscv_tuple_extract`。
- **L1376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1376 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_i32_ty],`.
  **L1377 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_i32_ty],`。
- **L1378 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable, ImmArg<ArgIndex<1>>]>;`.
  **L1378 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable, ImmArg<ArgIndex<1>>]>;`。
- **L1379 EN**: Blank line separating nearby declarations or logic blocks.
  **L1379 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1380 EN**: Declares TableGen defm `vle`.
  **L1380 CN**: 声明 TableGen defm `vle`。
- **L1381 EN**: Declares TableGen defm `vleff`.
  **L1381 CN**: 声明 TableGen defm `vleff`。
- **L1382 EN**: Declares TableGen defm `vse`.
  **L1382 CN**: 声明 TableGen defm `vse`。
- **L1383 EN**: Declares TableGen defm `vlse`.
  **L1383 CN**: 声明 TableGen defm `vlse`。
- **L1384 EN**: Declares TableGen defm `vsse`.
  **L1384 CN**: 声明 TableGen defm `vsse`。
- **L1385 EN**: Declares TableGen defm `vluxei`.
  **L1385 CN**: 声明 TableGen defm `vluxei`。
- **L1386 EN**: Declares TableGen defm `vloxei`.
  **L1386 CN**: 声明 TableGen defm `vloxei`。
- **L1387 EN**: Declares TableGen defm `vsoxei`.
  **L1387 CN**: 声明 TableGen defm `vsoxei`。
- **L1388 EN**: Declares TableGen defm `vsuxei`.
  **L1388 CN**: 声明 TableGen defm `vsuxei`。
- **L1389 EN**: Blank line separating nearby declarations or logic blocks.
  **L1389 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1390 EN**: Declares TableGen def `int_riscv_vlm`.
  **L1390 CN**: 声明 TableGen def `int_riscv_vlm`。
- **L1391 EN**: Declares TableGen def `int_riscv_vsm`.
  **L1391 CN**: 声明 TableGen def `int_riscv_vsm`。
- **L1392 EN**: Blank line separating nearby declarations or logic blocks.
  **L1392 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1393 EN**: Declares TableGen defm `vadd`.
  **L1393 CN**: 声明 TableGen defm `vadd`。
- **L1394 EN**: Declares TableGen defm `vsub`.
  **L1394 CN**: 声明 TableGen defm `vsub`。
- **L1395 EN**: Declares TableGen defm `vrsub`.
  **L1395 CN**: 声明 TableGen defm `vrsub`。
- **L1396 EN**: Blank line separating nearby declarations or logic blocks.
  **L1396 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1397 EN**: Declares TableGen defm `vwaddu`.
  **L1397 CN**: 声明 TableGen defm `vwaddu`。
- **L1398 EN**: Declares TableGen defm `vwadd`.
  **L1398 CN**: 声明 TableGen defm `vwadd`。
- **L1399 EN**: Declares TableGen defm `vwaddu_w`.
  **L1399 CN**: 声明 TableGen defm `vwaddu_w`。
- **L1400 EN**: Declares TableGen defm `vwadd_w`.
  **L1400 CN**: 声明 TableGen defm `vwadd_w`。

### Lines 1401-1428

````tablegen
  defm vwsubu : RISCVBinaryABX;
  defm vwsub : RISCVBinaryABX;
  defm vwsubu_w : RISCVBinaryAAX;
  defm vwsub_w : RISCVBinaryAAX;

  defm vzext : RISCVUnaryAB;
  defm vsext : RISCVUnaryAB;

  defm vadc : RISCVBinaryWithV0;
  defm vmadc_carry_in : RISCVBinaryMaskOutWithV0;
  defm vmadc : RISCVBinaryMaskOut;

  defm vsbc : RISCVBinaryWithV0;
  defm vmsbc_borrow_in : RISCVBinaryMaskOutWithV0;
  defm vmsbc : RISCVBinaryMaskOut;

  defm vand : RISCVBinaryAAX;
  defm vor : RISCVBinaryAAX;
  defm vxor : RISCVBinaryAAX;

  defm vsll : RISCVBinaryAAShift;
  defm vsrl : RISCVBinaryAAShift;
  defm vsra : RISCVBinaryAAShift;

  defm vnsrl : RISCVBinaryABShift;
  defm vnsra : RISCVBinaryABShift;

  defm vmseq : RISCVCompare;
````
- **L1401 EN**: Declares TableGen defm `vwsubu`.
  **L1401 CN**: 声明 TableGen defm `vwsubu`。
- **L1402 EN**: Declares TableGen defm `vwsub`.
  **L1402 CN**: 声明 TableGen defm `vwsub`。
- **L1403 EN**: Declares TableGen defm `vwsubu_w`.
  **L1403 CN**: 声明 TableGen defm `vwsubu_w`。
- **L1404 EN**: Declares TableGen defm `vwsub_w`.
  **L1404 CN**: 声明 TableGen defm `vwsub_w`。
- **L1405 EN**: Blank line separating nearby declarations or logic blocks.
  **L1405 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1406 EN**: Declares TableGen defm `vzext`.
  **L1406 CN**: 声明 TableGen defm `vzext`。
- **L1407 EN**: Declares TableGen defm `vsext`.
  **L1407 CN**: 声明 TableGen defm `vsext`。
- **L1408 EN**: Blank line separating nearby declarations or logic blocks.
  **L1408 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1409 EN**: Declares TableGen defm `vadc`.
  **L1409 CN**: 声明 TableGen defm `vadc`。
- **L1410 EN**: Declares TableGen defm `vmadc_carry_in`.
  **L1410 CN**: 声明 TableGen defm `vmadc_carry_in`。
- **L1411 EN**: Declares TableGen defm `vmadc`.
  **L1411 CN**: 声明 TableGen defm `vmadc`。
- **L1412 EN**: Blank line separating nearby declarations or logic blocks.
  **L1412 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1413 EN**: Declares TableGen defm `vsbc`.
  **L1413 CN**: 声明 TableGen defm `vsbc`。
- **L1414 EN**: Declares TableGen defm `vmsbc_borrow_in`.
  **L1414 CN**: 声明 TableGen defm `vmsbc_borrow_in`。
- **L1415 EN**: Declares TableGen defm `vmsbc`.
  **L1415 CN**: 声明 TableGen defm `vmsbc`。
- **L1416 EN**: Blank line separating nearby declarations or logic blocks.
  **L1416 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1417 EN**: Declares TableGen defm `vand`.
  **L1417 CN**: 声明 TableGen defm `vand`。
- **L1418 EN**: Declares TableGen defm `vor`.
  **L1418 CN**: 声明 TableGen defm `vor`。
- **L1419 EN**: Declares TableGen defm `vxor`.
  **L1419 CN**: 声明 TableGen defm `vxor`。
- **L1420 EN**: Blank line separating nearby declarations or logic blocks.
  **L1420 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1421 EN**: Declares TableGen defm `vsll`.
  **L1421 CN**: 声明 TableGen defm `vsll`。
- **L1422 EN**: Declares TableGen defm `vsrl`.
  **L1422 CN**: 声明 TableGen defm `vsrl`。
- **L1423 EN**: Declares TableGen defm `vsra`.
  **L1423 CN**: 声明 TableGen defm `vsra`。
- **L1424 EN**: Blank line separating nearby declarations or logic blocks.
  **L1424 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1425 EN**: Declares TableGen defm `vnsrl`.
  **L1425 CN**: 声明 TableGen defm `vnsrl`。
- **L1426 EN**: Declares TableGen defm `vnsra`.
  **L1426 CN**: 声明 TableGen defm `vnsra`。
- **L1427 EN**: Blank line separating nearby declarations or logic blocks.
  **L1427 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1428 EN**: Declares TableGen defm `vmseq`.
  **L1428 CN**: 声明 TableGen defm `vmseq`。

### Lines 1429-1456

````tablegen
  defm vmsne : RISCVCompare;
  defm vmsltu : RISCVCompare;
  defm vmslt : RISCVCompare;
  defm vmsleu : RISCVCompare;
  defm vmsle : RISCVCompare;
  defm vmsgtu : RISCVCompare;
  defm vmsgt : RISCVCompare;
  defm vmsgeu : RISCVCompare;
  defm vmsge : RISCVCompare;

  defm vminu : RISCVBinaryAAX;
  defm vmin : RISCVBinaryAAX;
  defm vmaxu : RISCVBinaryAAX;
  defm vmax : RISCVBinaryAAX;

  defm vmul : RISCVBinaryAAX;
  defm vmulh : RISCVBinaryAAX;
  defm vmulhu : RISCVBinaryAAX;
  defm vmulhsu : RISCVBinaryAAX;

  defm vdivu : RISCVBinaryAAX;
  defm vdiv : RISCVBinaryAAX;
  defm vremu : RISCVBinaryAAX;
  defm vrem : RISCVBinaryAAX;

  defm vwmul : RISCVBinaryABX;
  defm vwmulu : RISCVBinaryABX;
  defm vwmulsu : RISCVBinaryABX;
````
- **L1429 EN**: Declares TableGen defm `vmsne`.
  **L1429 CN**: 声明 TableGen defm `vmsne`。
- **L1430 EN**: Declares TableGen defm `vmsltu`.
  **L1430 CN**: 声明 TableGen defm `vmsltu`。
- **L1431 EN**: Declares TableGen defm `vmslt`.
  **L1431 CN**: 声明 TableGen defm `vmslt`。
- **L1432 EN**: Declares TableGen defm `vmsleu`.
  **L1432 CN**: 声明 TableGen defm `vmsleu`。
- **L1433 EN**: Declares TableGen defm `vmsle`.
  **L1433 CN**: 声明 TableGen defm `vmsle`。
- **L1434 EN**: Declares TableGen defm `vmsgtu`.
  **L1434 CN**: 声明 TableGen defm `vmsgtu`。
- **L1435 EN**: Declares TableGen defm `vmsgt`.
  **L1435 CN**: 声明 TableGen defm `vmsgt`。
- **L1436 EN**: Declares TableGen defm `vmsgeu`.
  **L1436 CN**: 声明 TableGen defm `vmsgeu`。
- **L1437 EN**: Declares TableGen defm `vmsge`.
  **L1437 CN**: 声明 TableGen defm `vmsge`。
- **L1438 EN**: Blank line separating nearby declarations or logic blocks.
  **L1438 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1439 EN**: Declares TableGen defm `vminu`.
  **L1439 CN**: 声明 TableGen defm `vminu`。
- **L1440 EN**: Declares TableGen defm `vmin`.
  **L1440 CN**: 声明 TableGen defm `vmin`。
- **L1441 EN**: Declares TableGen defm `vmaxu`.
  **L1441 CN**: 声明 TableGen defm `vmaxu`。
- **L1442 EN**: Declares TableGen defm `vmax`.
  **L1442 CN**: 声明 TableGen defm `vmax`。
- **L1443 EN**: Blank line separating nearby declarations or logic blocks.
  **L1443 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1444 EN**: Declares TableGen defm `vmul`.
  **L1444 CN**: 声明 TableGen defm `vmul`。
- **L1445 EN**: Declares TableGen defm `vmulh`.
  **L1445 CN**: 声明 TableGen defm `vmulh`。
- **L1446 EN**: Declares TableGen defm `vmulhu`.
  **L1446 CN**: 声明 TableGen defm `vmulhu`。
- **L1447 EN**: Declares TableGen defm `vmulhsu`.
  **L1447 CN**: 声明 TableGen defm `vmulhsu`。
- **L1448 EN**: Blank line separating nearby declarations or logic blocks.
  **L1448 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1449 EN**: Declares TableGen defm `vdivu`.
  **L1449 CN**: 声明 TableGen defm `vdivu`。
- **L1450 EN**: Declares TableGen defm `vdiv`.
  **L1450 CN**: 声明 TableGen defm `vdiv`。
- **L1451 EN**: Declares TableGen defm `vremu`.
  **L1451 CN**: 声明 TableGen defm `vremu`。
- **L1452 EN**: Declares TableGen defm `vrem`.
  **L1452 CN**: 声明 TableGen defm `vrem`。
- **L1453 EN**: Blank line separating nearby declarations or logic blocks.
  **L1453 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1454 EN**: Declares TableGen defm `vwmul`.
  **L1454 CN**: 声明 TableGen defm `vwmul`。
- **L1455 EN**: Declares TableGen defm `vwmulu`.
  **L1455 CN**: 声明 TableGen defm `vwmulu`。
- **L1456 EN**: Declares TableGen defm `vwmulsu`.
  **L1456 CN**: 声明 TableGen defm `vwmulsu`。

### Lines 1457-1484

````tablegen

  defm vmacc : RISCVTernaryAAXA;
  defm vnmsac : RISCVTernaryAAXA;
  defm vmadd : RISCVTernaryAAXA;
  defm vnmsub : RISCVTernaryAAXA;

  defm vwmaccu  : RISCVTernaryWide;
  defm vwmacc   : RISCVTernaryWide;
  defm vwmaccus : RISCVTernaryWide;
  defm vwmaccsu : RISCVTernaryWide;

  let IsFPIntrinsic = 1 in {
    defm vfadd : RISCVBinaryAAXRoundingMode;
    defm vfsub : RISCVBinaryAAXRoundingMode;
    defm vfrsub : RISCVBinaryAAXRoundingMode;
    defm vfwadd : RISCVBinaryABXRoundingMode;
    defm vfwsub : RISCVBinaryABXRoundingMode;
    defm vfwadd_w : RISCVBinaryAAXRoundingMode;
    defm vfwsub_w : RISCVBinaryAAXRoundingMode;
  }

  defm vsaddu : RISCVSaturatingBinaryAAX;
  defm vsadd : RISCVSaturatingBinaryAAX;
  defm vssubu : RISCVSaturatingBinaryAAX;
  defm vssub : RISCVSaturatingBinaryAAX;

  defm vmerge : RISCVBinaryWithV0;

````
- **L1457 EN**: Blank line separating nearby declarations or logic blocks.
  **L1457 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1458 EN**: Declares TableGen defm `vmacc`.
  **L1458 CN**: 声明 TableGen defm `vmacc`。
- **L1459 EN**: Declares TableGen defm `vnmsac`.
  **L1459 CN**: 声明 TableGen defm `vnmsac`。
- **L1460 EN**: Declares TableGen defm `vmadd`.
  **L1460 CN**: 声明 TableGen defm `vmadd`。
- **L1461 EN**: Declares TableGen defm `vnmsub`.
  **L1461 CN**: 声明 TableGen defm `vnmsub`。
- **L1462 EN**: Blank line separating nearby declarations or logic blocks.
  **L1462 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1463 EN**: Declares TableGen defm `vwmaccu`.
  **L1463 CN**: 声明 TableGen defm `vwmaccu`。
- **L1464 EN**: Declares TableGen defm `vwmacc`.
  **L1464 CN**: 声明 TableGen defm `vwmacc`。
- **L1465 EN**: Declares TableGen defm `vwmaccus`.
  **L1465 CN**: 声明 TableGen defm `vwmaccus`。
- **L1466 EN**: Declares TableGen defm `vwmaccsu`.
  **L1466 CN**: 声明 TableGen defm `vwmaccsu`。
- **L1467 EN**: Blank line separating nearby declarations or logic blocks.
  **L1467 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1468 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1468 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1469 EN**: Declares TableGen defm `vfadd`.
  **L1469 CN**: 声明 TableGen defm `vfadd`。
- **L1470 EN**: Declares TableGen defm `vfsub`.
  **L1470 CN**: 声明 TableGen defm `vfsub`。
- **L1471 EN**: Declares TableGen defm `vfrsub`.
  **L1471 CN**: 声明 TableGen defm `vfrsub`。
- **L1472 EN**: Declares TableGen defm `vfwadd`.
  **L1472 CN**: 声明 TableGen defm `vfwadd`。
- **L1473 EN**: Declares TableGen defm `vfwsub`.
  **L1473 CN**: 声明 TableGen defm `vfwsub`。
- **L1474 EN**: Declares TableGen defm `vfwadd_w`.
  **L1474 CN**: 声明 TableGen defm `vfwadd_w`。
- **L1475 EN**: Declares TableGen defm `vfwsub_w`.
  **L1475 CN**: 声明 TableGen defm `vfwsub_w`。
- **L1476 EN**: Closes the current lexical scope or compound statement.
  **L1476 CN**: 结束当前词法作用域或复合语句块。
- **L1477 EN**: Blank line separating nearby declarations or logic blocks.
  **L1477 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1478 EN**: Declares TableGen defm `vsaddu`.
  **L1478 CN**: 声明 TableGen defm `vsaddu`。
- **L1479 EN**: Declares TableGen defm `vsadd`.
  **L1479 CN**: 声明 TableGen defm `vsadd`。
- **L1480 EN**: Declares TableGen defm `vssubu`.
  **L1480 CN**: 声明 TableGen defm `vssubu`。
- **L1481 EN**: Declares TableGen defm `vssub`.
  **L1481 CN**: 声明 TableGen defm `vssub`。
- **L1482 EN**: Blank line separating nearby declarations or logic blocks.
  **L1482 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1483 EN**: Declares TableGen defm `vmerge`.
  **L1483 CN**: 声明 TableGen defm `vmerge`。
- **L1484 EN**: Blank line separating nearby declarations or logic blocks.
  **L1484 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1485-1512

````tablegen
  // Output: (vector)
  // Input: (passthru, vector_in, vl)
  def int_riscv_vmv_v_v : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                                                [LLVMMatchType<0>,
                                                 LLVMMatchType<0>,
                                                 llvm_anyint_ty],
                                                [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 2;
  }
  // Output: (vector)
  // Input: (passthru, scalar, vl)
  def int_riscv_vmv_v_x : DefaultAttrsIntrinsic<[llvm_anyint_ty],
                                                 [LLVMMatchType<0>,
                                                  LLVMVectorElementType<0>,
                                                  llvm_anyint_ty],
                                                 [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 2;
  }
  // Output: (vector)
  // Input: (passthru, scalar, vl)
  def int_riscv_vfmv_v_f : DefaultAttrsIntrinsic<[llvm_anyfloat_ty],
                                                 [LLVMMatchType<0>,
                                                  LLVMVectorElementType<0>,
                                                  llvm_anyint_ty],
                                                 [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 2;
    let IsFPIntrinsic = 1;
  }
````
- **L1485 EN**: Comment explains nearby logic, invariants, or intent: `Output: (vector)`.
  **L1485 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (vector)`。
- **L1486 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vl)`.
  **L1486 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vl)`。
- **L1487 EN**: Declares TableGen def `int_riscv_vmv_v_v`.
  **L1487 CN**: 声明 TableGen def `int_riscv_vmv_v_v`。
- **L1488 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1488 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1489 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L1490 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L1491 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L1491 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L1492 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1492 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1493 EN**: Closes the current lexical scope or compound statement.
  **L1493 CN**: 结束当前词法作用域或复合语句块。
- **L1494 EN**: Comment explains nearby logic, invariants, or intent: `Output: (vector)`.
  **L1494 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (vector)`。
- **L1495 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, scalar, vl)`.
  **L1495 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, scalar, vl)`。
- **L1496 EN**: Declares TableGen def `int_riscv_vmv_v_x`.
  **L1496 CN**: 声明 TableGen def `int_riscv_vmv_v_x`。
- **L1497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1497 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1498 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorElementType<0>,`.
  **L1498 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorElementType<0>,`。
- **L1499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L1499 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L1500 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L1500 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L1501 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1501 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1502 EN**: Closes the current lexical scope or compound statement.
  **L1502 CN**: 结束当前词法作用域或复合语句块。
- **L1503 EN**: Comment explains nearby logic, invariants, or intent: `Output: (vector)`.
  **L1503 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (vector)`。
- **L1504 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, scalar, vl)`.
  **L1504 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, scalar, vl)`。
- **L1505 EN**: Declares TableGen def `int_riscv_vfmv_v_f`.
  **L1505 CN**: 声明 TableGen def `int_riscv_vfmv_v_f`。
- **L1506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1506 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorElementType<0>,`.
  **L1507 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorElementType<0>,`。
- **L1508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L1508 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L1509 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L1509 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L1510 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1510 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1511 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1511 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1512 EN**: Closes the current lexical scope or compound statement.
  **L1512 CN**: 结束当前词法作用域或复合语句块。

### Lines 1513-1540

````tablegen

  def int_riscv_vmv_x_s : DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],
                                                [llvm_anyint_ty],
                                                [IntrNoMem]>, RISCVVIntrinsic;
  def int_riscv_vmv_s_x : DefaultAttrsIntrinsic<[llvm_anyint_ty],
                                                [LLVMMatchType<0>,
                                                 LLVMVectorElementType<0>,
                                                 llvm_anyint_ty],
                                                [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 2;
  }

  def int_riscv_vfmv_f_s : DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],
                                                 [llvm_anyfloat_ty],
                                                 [IntrNoMem]>, RISCVVIntrinsic;
  def int_riscv_vfmv_s_f : DefaultAttrsIntrinsic<[llvm_anyfloat_ty],
                                                 [LLVMMatchType<0>,
                                                  LLVMVectorElementType<0>,
                                                  llvm_anyint_ty],
                                                 [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 2;
    let IsFPIntrinsic = 1;
  }

  let IsFPIntrinsic = 1 in {
    defm vfmul : RISCVBinaryAAXRoundingMode;
    defm vfdiv : RISCVBinaryAAXRoundingMode;
    defm vfrdiv : RISCVBinaryAAXRoundingMode;
````
- **L1513 EN**: Blank line separating nearby declarations or logic blocks.
  **L1513 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1514 EN**: Declares TableGen def `int_riscv_vmv_x_s`.
  **L1514 CN**: 声明 TableGen def `int_riscv_vmv_x_s`。
- **L1515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyint_ty],`.
  **L1515 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyint_ty],`。
- **L1516 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>, RISCVVIntrinsic;`.
  **L1516 CN**: 执行一条独立语句或声明：`[IntrNoMem]>, RISCVVIntrinsic;`。
- **L1517 EN**: Declares TableGen def `int_riscv_vmv_s_x`.
  **L1517 CN**: 声明 TableGen def `int_riscv_vmv_s_x`。
- **L1518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1518 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorElementType<0>,`.
  **L1519 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorElementType<0>,`。
- **L1520 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L1520 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L1521 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L1521 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L1522 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1522 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1523 EN**: Closes the current lexical scope or compound statement.
  **L1523 CN**: 结束当前词法作用域或复合语句块。
- **L1524 EN**: Blank line separating nearby declarations or logic blocks.
  **L1524 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1525 EN**: Declares TableGen def `int_riscv_vfmv_f_s`.
  **L1525 CN**: 声明 TableGen def `int_riscv_vfmv_f_s`。
- **L1526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyfloat_ty],`.
  **L1526 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyfloat_ty],`。
- **L1527 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>, RISCVVIntrinsic;`.
  **L1527 CN**: 执行一条独立语句或声明：`[IntrNoMem]>, RISCVVIntrinsic;`。
- **L1528 EN**: Declares TableGen def `int_riscv_vfmv_s_f`.
  **L1528 CN**: 声明 TableGen def `int_riscv_vfmv_s_f`。
- **L1529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1529 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorElementType<0>,`.
  **L1530 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorElementType<0>,`。
- **L1531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L1531 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L1532 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L1532 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L1533 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1533 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1534 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1534 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1535 EN**: Closes the current lexical scope or compound statement.
  **L1535 CN**: 结束当前词法作用域或复合语句块。
- **L1536 EN**: Blank line separating nearby declarations or logic blocks.
  **L1536 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1537 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1537 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1538 EN**: Declares TableGen defm `vfmul`.
  **L1538 CN**: 声明 TableGen defm `vfmul`。
- **L1539 EN**: Declares TableGen defm `vfdiv`.
  **L1539 CN**: 声明 TableGen defm `vfdiv`。
- **L1540 EN**: Declares TableGen defm `vfrdiv`.
  **L1540 CN**: 声明 TableGen defm `vfrdiv`。

### Lines 1541-1568

````tablegen

    defm vfwmul : RISCVBinaryABXRoundingMode;

    defm vfmacc : RISCVTernaryAAXARoundingMode;
    defm vfnmacc : RISCVTernaryAAXARoundingMode;
    defm vfmsac : RISCVTernaryAAXARoundingMode;
    defm vfnmsac : RISCVTernaryAAXARoundingMode;
    defm vfmadd : RISCVTernaryAAXARoundingMode;
    defm vfnmadd : RISCVTernaryAAXARoundingMode;
    defm vfmsub : RISCVTernaryAAXARoundingMode;
    defm vfnmsub : RISCVTernaryAAXARoundingMode;

    defm vfwmacc : RISCVTernaryWideRoundingMode;
    defm vfwmaccbf16 : RISCVTernaryWideRoundingMode;
    defm vfwnmacc : RISCVTernaryWideRoundingMode;
    defm vfwmsac : RISCVTernaryWideRoundingMode;
    defm vfwnmsac : RISCVTernaryWideRoundingMode;

    defm vfsqrt : RISCVUnaryAARoundingMode;
    defm vfrsqrt7 : RISCVUnaryAA;
    defm vfrec7 : RISCVUnaryAARoundingMode;

    defm vfmin : RISCVBinaryAAX;
    defm vfmax : RISCVBinaryAAX;

    defm vfsgnj : RISCVBinaryAAX;
    defm vfsgnjn : RISCVBinaryAAX;
    defm vfsgnjx : RISCVBinaryAAX;
````
- **L1541 EN**: Blank line separating nearby declarations or logic blocks.
  **L1541 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1542 EN**: Declares TableGen defm `vfwmul`.
  **L1542 CN**: 声明 TableGen defm `vfwmul`。
- **L1543 EN**: Blank line separating nearby declarations or logic blocks.
  **L1543 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1544 EN**: Declares TableGen defm `vfmacc`.
  **L1544 CN**: 声明 TableGen defm `vfmacc`。
- **L1545 EN**: Declares TableGen defm `vfnmacc`.
  **L1545 CN**: 声明 TableGen defm `vfnmacc`。
- **L1546 EN**: Declares TableGen defm `vfmsac`.
  **L1546 CN**: 声明 TableGen defm `vfmsac`。
- **L1547 EN**: Declares TableGen defm `vfnmsac`.
  **L1547 CN**: 声明 TableGen defm `vfnmsac`。
- **L1548 EN**: Declares TableGen defm `vfmadd`.
  **L1548 CN**: 声明 TableGen defm `vfmadd`。
- **L1549 EN**: Declares TableGen defm `vfnmadd`.
  **L1549 CN**: 声明 TableGen defm `vfnmadd`。
- **L1550 EN**: Declares TableGen defm `vfmsub`.
  **L1550 CN**: 声明 TableGen defm `vfmsub`。
- **L1551 EN**: Declares TableGen defm `vfnmsub`.
  **L1551 CN**: 声明 TableGen defm `vfnmsub`。
- **L1552 EN**: Blank line separating nearby declarations or logic blocks.
  **L1552 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1553 EN**: Declares TableGen defm `vfwmacc`.
  **L1553 CN**: 声明 TableGen defm `vfwmacc`。
- **L1554 EN**: Declares TableGen defm `vfwmaccbf16`.
  **L1554 CN**: 声明 TableGen defm `vfwmaccbf16`。
- **L1555 EN**: Declares TableGen defm `vfwnmacc`.
  **L1555 CN**: 声明 TableGen defm `vfwnmacc`。
- **L1556 EN**: Declares TableGen defm `vfwmsac`.
  **L1556 CN**: 声明 TableGen defm `vfwmsac`。
- **L1557 EN**: Declares TableGen defm `vfwnmsac`.
  **L1557 CN**: 声明 TableGen defm `vfwnmsac`。
- **L1558 EN**: Blank line separating nearby declarations or logic blocks.
  **L1558 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1559 EN**: Declares TableGen defm `vfsqrt`.
  **L1559 CN**: 声明 TableGen defm `vfsqrt`。
- **L1560 EN**: Declares TableGen defm `vfrsqrt7`.
  **L1560 CN**: 声明 TableGen defm `vfrsqrt7`。
- **L1561 EN**: Declares TableGen defm `vfrec7`.
  **L1561 CN**: 声明 TableGen defm `vfrec7`。
- **L1562 EN**: Blank line separating nearby declarations or logic blocks.
  **L1562 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1563 EN**: Declares TableGen defm `vfmin`.
  **L1563 CN**: 声明 TableGen defm `vfmin`。
- **L1564 EN**: Declares TableGen defm `vfmax`.
  **L1564 CN**: 声明 TableGen defm `vfmax`。
- **L1565 EN**: Blank line separating nearby declarations or logic blocks.
  **L1565 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1566 EN**: Declares TableGen defm `vfsgnj`.
  **L1566 CN**: 声明 TableGen defm `vfsgnj`。
- **L1567 EN**: Declares TableGen defm `vfsgnjn`.
  **L1567 CN**: 声明 TableGen defm `vfsgnjn`。
- **L1568 EN**: Declares TableGen defm `vfsgnjx`.
  **L1568 CN**: 声明 TableGen defm `vfsgnjx`。

### Lines 1569-1596

````tablegen

    defm vfclass : RISCVClassify;

    defm vfmerge : RISCVBinaryWithV0;
  }

  defm vslideup : RVVSlide;
  defm vslidedown : RVVSlide;

  defm vslide1up : RISCVBinaryAAX;
  defm vslide1down : RISCVBinaryAAX;

  let IsFPIntrinsic = 1 in {
    defm vfslide1up : RISCVBinaryAAX;
    defm vfslide1down : RISCVBinaryAAX;
  }

  defm vrgather_vv : RISCVRGatherVV;
  defm vrgather_vx : RISCVRGatherVX;
  defm vrgatherei16_vv : RISCVRGatherEI16VV;

  def int_riscv_vcompress : RISCVCompress;

  defm vaaddu : RISCVSaturatingBinaryAAXRoundingMode;
  defm vaadd : RISCVSaturatingBinaryAAXRoundingMode;
  defm vasubu : RISCVSaturatingBinaryAAXRoundingMode;
  defm vasub : RISCVSaturatingBinaryAAXRoundingMode;

````
- **L1569 EN**: Blank line separating nearby declarations or logic blocks.
  **L1569 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1570 EN**: Declares TableGen defm `vfclass`.
  **L1570 CN**: 声明 TableGen defm `vfclass`。
- **L1571 EN**: Blank line separating nearby declarations or logic blocks.
  **L1571 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1572 EN**: Declares TableGen defm `vfmerge`.
  **L1572 CN**: 声明 TableGen defm `vfmerge`。
- **L1573 EN**: Closes the current lexical scope or compound statement.
  **L1573 CN**: 结束当前词法作用域或复合语句块。
- **L1574 EN**: Blank line separating nearby declarations or logic blocks.
  **L1574 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1575 EN**: Declares TableGen defm `vslideup`.
  **L1575 CN**: 声明 TableGen defm `vslideup`。
- **L1576 EN**: Declares TableGen defm `vslidedown`.
  **L1576 CN**: 声明 TableGen defm `vslidedown`。
- **L1577 EN**: Blank line separating nearby declarations or logic blocks.
  **L1577 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1578 EN**: Declares TableGen defm `vslide1up`.
  **L1578 CN**: 声明 TableGen defm `vslide1up`。
- **L1579 EN**: Declares TableGen defm `vslide1down`.
  **L1579 CN**: 声明 TableGen defm `vslide1down`。
- **L1580 EN**: Blank line separating nearby declarations or logic blocks.
  **L1580 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1581 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1581 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1582 EN**: Declares TableGen defm `vfslide1up`.
  **L1582 CN**: 声明 TableGen defm `vfslide1up`。
- **L1583 EN**: Declares TableGen defm `vfslide1down`.
  **L1583 CN**: 声明 TableGen defm `vfslide1down`。
- **L1584 EN**: Closes the current lexical scope or compound statement.
  **L1584 CN**: 结束当前词法作用域或复合语句块。
- **L1585 EN**: Blank line separating nearby declarations or logic blocks.
  **L1585 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1586 EN**: Declares TableGen defm `vrgather_vv`.
  **L1586 CN**: 声明 TableGen defm `vrgather_vv`。
- **L1587 EN**: Declares TableGen defm `vrgather_vx`.
  **L1587 CN**: 声明 TableGen defm `vrgather_vx`。
- **L1588 EN**: Declares TableGen defm `vrgatherei16_vv`.
  **L1588 CN**: 声明 TableGen defm `vrgatherei16_vv`。
- **L1589 EN**: Blank line separating nearby declarations or logic blocks.
  **L1589 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1590 EN**: Declares TableGen def `int_riscv_vcompress`.
  **L1590 CN**: 声明 TableGen def `int_riscv_vcompress`。
- **L1591 EN**: Blank line separating nearby declarations or logic blocks.
  **L1591 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1592 EN**: Declares TableGen defm `vaaddu`.
  **L1592 CN**: 声明 TableGen defm `vaaddu`。
- **L1593 EN**: Declares TableGen defm `vaadd`.
  **L1593 CN**: 声明 TableGen defm `vaadd`。
- **L1594 EN**: Declares TableGen defm `vasubu`.
  **L1594 CN**: 声明 TableGen defm `vasubu`。
- **L1595 EN**: Declares TableGen defm `vasub`.
  **L1595 CN**: 声明 TableGen defm `vasub`。
- **L1596 EN**: Blank line separating nearby declarations or logic blocks.
  **L1596 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1597-1624

````tablegen
  defm vsmul : RISCVSaturatingBinaryAAXRoundingMode;

  defm vssrl : RISCVSaturatingBinaryAAShiftRoundingMode;
  defm vssra : RISCVSaturatingBinaryAAShiftRoundingMode;

  defm vnclipu : RISCVSaturatingBinaryABShiftRoundingMode;
  defm vnclip : RISCVSaturatingBinaryABShiftRoundingMode;

  let IsFPIntrinsic = 1 in {
    defm vmfeq : RISCVCompare;
    defm vmfne : RISCVCompare;
    defm vmflt : RISCVCompare;
    defm vmfle : RISCVCompare;
    defm vmfgt : RISCVCompare;
    defm vmfge : RISCVCompare;
  }

  defm vredsum : RISCVReduction;
  defm vredand : RISCVReduction;
  defm vredor : RISCVReduction;
  defm vredxor : RISCVReduction;
  defm vredminu : RISCVReduction;
  defm vredmin : RISCVReduction;
  defm vredmaxu : RISCVReduction;
  defm vredmax : RISCVReduction;

  defm vwredsumu : RISCVReduction;
  defm vwredsum : RISCVReduction;
````
- **L1597 EN**: Declares TableGen defm `vsmul`.
  **L1597 CN**: 声明 TableGen defm `vsmul`。
- **L1598 EN**: Blank line separating nearby declarations or logic blocks.
  **L1598 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1599 EN**: Declares TableGen defm `vssrl`.
  **L1599 CN**: 声明 TableGen defm `vssrl`。
- **L1600 EN**: Declares TableGen defm `vssra`.
  **L1600 CN**: 声明 TableGen defm `vssra`。
- **L1601 EN**: Blank line separating nearby declarations or logic blocks.
  **L1601 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1602 EN**: Declares TableGen defm `vnclipu`.
  **L1602 CN**: 声明 TableGen defm `vnclipu`。
- **L1603 EN**: Declares TableGen defm `vnclip`.
  **L1603 CN**: 声明 TableGen defm `vnclip`。
- **L1604 EN**: Blank line separating nearby declarations or logic blocks.
  **L1604 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1605 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1605 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1606 EN**: Declares TableGen defm `vmfeq`.
  **L1606 CN**: 声明 TableGen defm `vmfeq`。
- **L1607 EN**: Declares TableGen defm `vmfne`.
  **L1607 CN**: 声明 TableGen defm `vmfne`。
- **L1608 EN**: Declares TableGen defm `vmflt`.
  **L1608 CN**: 声明 TableGen defm `vmflt`。
- **L1609 EN**: Declares TableGen defm `vmfle`.
  **L1609 CN**: 声明 TableGen defm `vmfle`。
- **L1610 EN**: Declares TableGen defm `vmfgt`.
  **L1610 CN**: 声明 TableGen defm `vmfgt`。
- **L1611 EN**: Declares TableGen defm `vmfge`.
  **L1611 CN**: 声明 TableGen defm `vmfge`。
- **L1612 EN**: Closes the current lexical scope or compound statement.
  **L1612 CN**: 结束当前词法作用域或复合语句块。
- **L1613 EN**: Blank line separating nearby declarations or logic blocks.
  **L1613 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1614 EN**: Declares TableGen defm `vredsum`.
  **L1614 CN**: 声明 TableGen defm `vredsum`。
- **L1615 EN**: Declares TableGen defm `vredand`.
  **L1615 CN**: 声明 TableGen defm `vredand`。
- **L1616 EN**: Declares TableGen defm `vredor`.
  **L1616 CN**: 声明 TableGen defm `vredor`。
- **L1617 EN**: Declares TableGen defm `vredxor`.
  **L1617 CN**: 声明 TableGen defm `vredxor`。
- **L1618 EN**: Declares TableGen defm `vredminu`.
  **L1618 CN**: 声明 TableGen defm `vredminu`。
- **L1619 EN**: Declares TableGen defm `vredmin`.
  **L1619 CN**: 声明 TableGen defm `vredmin`。
- **L1620 EN**: Declares TableGen defm `vredmaxu`.
  **L1620 CN**: 声明 TableGen defm `vredmaxu`。
- **L1621 EN**: Declares TableGen defm `vredmax`.
  **L1621 CN**: 声明 TableGen defm `vredmax`。
- **L1622 EN**: Blank line separating nearby declarations or logic blocks.
  **L1622 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1623 EN**: Declares TableGen defm `vwredsumu`.
  **L1623 CN**: 声明 TableGen defm `vwredsumu`。
- **L1624 EN**: Declares TableGen defm `vwredsum`.
  **L1624 CN**: 声明 TableGen defm `vwredsum`。

### Lines 1625-1652

````tablegen

  let IsFPIntrinsic = 1 in {
    defm vfredosum : RISCVReductionRoundingMode;
    defm vfredusum : RISCVReductionRoundingMode;
    defm vfredmin : RISCVReduction;
    defm vfredmax : RISCVReduction;

    defm vfwredusum : RISCVReductionRoundingMode;
    defm vfwredosum : RISCVReductionRoundingMode;
  }

  def int_riscv_vmand: RISCVMaskLogical;
  def int_riscv_vmnand: RISCVMaskLogical;
  def int_riscv_vmandn: RISCVMaskLogical;
  def int_riscv_vmxor: RISCVMaskLogical;
  def int_riscv_vmor: RISCVMaskLogical;
  def int_riscv_vmnor: RISCVMaskLogical;
  def int_riscv_vmorn: RISCVMaskLogical;
  def int_riscv_vmxnor: RISCVMaskLogical;
  def int_riscv_vmclr : RISCVNullaryIntrinsic;
  def int_riscv_vmset : RISCVNullaryIntrinsic;

  defm vcpop : RISCVMaskedUnarySOut;
  defm vfirst : RISCVMaskedUnarySOut;
  defm vmsbf : RISCVMaskedUnaryMOut;
  defm vmsof : RISCVMaskedUnaryMOut;
  defm vmsif : RISCVMaskedUnaryMOut;

````
- **L1625 EN**: Blank line separating nearby declarations or logic blocks.
  **L1625 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1626 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1626 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1627 EN**: Declares TableGen defm `vfredosum`.
  **L1627 CN**: 声明 TableGen defm `vfredosum`。
- **L1628 EN**: Declares TableGen defm `vfredusum`.
  **L1628 CN**: 声明 TableGen defm `vfredusum`。
- **L1629 EN**: Declares TableGen defm `vfredmin`.
  **L1629 CN**: 声明 TableGen defm `vfredmin`。
- **L1630 EN**: Declares TableGen defm `vfredmax`.
  **L1630 CN**: 声明 TableGen defm `vfredmax`。
- **L1631 EN**: Blank line separating nearby declarations or logic blocks.
  **L1631 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1632 EN**: Declares TableGen defm `vfwredusum`.
  **L1632 CN**: 声明 TableGen defm `vfwredusum`。
- **L1633 EN**: Declares TableGen defm `vfwredosum`.
  **L1633 CN**: 声明 TableGen defm `vfwredosum`。
- **L1634 EN**: Closes the current lexical scope or compound statement.
  **L1634 CN**: 结束当前词法作用域或复合语句块。
- **L1635 EN**: Blank line separating nearby declarations or logic blocks.
  **L1635 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1636 EN**: Declares TableGen def `int_riscv_vmand`.
  **L1636 CN**: 声明 TableGen def `int_riscv_vmand`。
- **L1637 EN**: Declares TableGen def `int_riscv_vmnand`.
  **L1637 CN**: 声明 TableGen def `int_riscv_vmnand`。
- **L1638 EN**: Declares TableGen def `int_riscv_vmandn`.
  **L1638 CN**: 声明 TableGen def `int_riscv_vmandn`。
- **L1639 EN**: Declares TableGen def `int_riscv_vmxor`.
  **L1639 CN**: 声明 TableGen def `int_riscv_vmxor`。
- **L1640 EN**: Declares TableGen def `int_riscv_vmor`.
  **L1640 CN**: 声明 TableGen def `int_riscv_vmor`。
- **L1641 EN**: Declares TableGen def `int_riscv_vmnor`.
  **L1641 CN**: 声明 TableGen def `int_riscv_vmnor`。
- **L1642 EN**: Declares TableGen def `int_riscv_vmorn`.
  **L1642 CN**: 声明 TableGen def `int_riscv_vmorn`。
- **L1643 EN**: Declares TableGen def `int_riscv_vmxnor`.
  **L1643 CN**: 声明 TableGen def `int_riscv_vmxnor`。
- **L1644 EN**: Declares TableGen def `int_riscv_vmclr`.
  **L1644 CN**: 声明 TableGen def `int_riscv_vmclr`。
- **L1645 EN**: Declares TableGen def `int_riscv_vmset`.
  **L1645 CN**: 声明 TableGen def `int_riscv_vmset`。
- **L1646 EN**: Blank line separating nearby declarations or logic blocks.
  **L1646 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1647 EN**: Declares TableGen defm `vcpop`.
  **L1647 CN**: 声明 TableGen defm `vcpop`。
- **L1648 EN**: Declares TableGen defm `vfirst`.
  **L1648 CN**: 声明 TableGen defm `vfirst`。
- **L1649 EN**: Declares TableGen defm `vmsbf`.
  **L1649 CN**: 声明 TableGen defm `vmsbf`。
- **L1650 EN**: Declares TableGen defm `vmsof`.
  **L1650 CN**: 声明 TableGen defm `vmsof`。
- **L1651 EN**: Declares TableGen defm `vmsif`.
  **L1651 CN**: 声明 TableGen defm `vmsif`。
- **L1652 EN**: Blank line separating nearby declarations or logic blocks.
  **L1652 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1653-1680

````tablegen
  let IsFPIntrinsic = 1 in {
    defm vfcvt_xu_f_v : RISCVConversionRoundingMode;
    defm vfcvt_x_f_v : RISCVConversionRoundingMode;
    defm vfcvt_rtz_xu_f_v : RISCVConversion;
    defm vfcvt_rtz_x_f_v : RISCVConversion;
    defm vfcvt_f_xu_v : RISCVConversionRoundingMode;
    defm vfcvt_f_x_v : RISCVConversionRoundingMode;

    defm vfwcvt_f_xu_v : RISCVConversion;
    defm vfwcvt_f_x_v : RISCVConversion;
    defm vfwcvt_xu_f_v : RISCVConversionRoundingMode;
    defm vfwcvt_x_f_v : RISCVConversionRoundingMode;
    defm vfwcvt_rtz_xu_f_v : RISCVConversion;
    defm vfwcvt_rtz_x_f_v : RISCVConversion;
    defm vfwcvt_f_f_v : RISCVConversion;
    defm vfwcvtbf16_f_f_v : RISCVConversion;

    defm vfncvt_f_xu_w : RISCVConversionRoundingMode;
    defm vfncvt_f_x_w : RISCVConversionRoundingMode;
    defm vfncvt_xu_f_w : RISCVConversionRoundingMode;
    defm vfncvt_x_f_w : RISCVConversionRoundingMode;
    defm vfncvt_rtz_xu_f_w : RISCVConversion;
    defm vfncvt_rtz_x_f_w : RISCVConversion;
    defm vfncvt_f_f_w : RISCVConversionRoundingMode;
    defm vfncvtbf16_f_f_w : RISCVConversionRoundingMode;
    defm vfncvt_rod_f_f_w : RISCVConversion;
  }

````
- **L1653 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1653 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1654 EN**: Declares TableGen defm `vfcvt_xu_f_v`.
  **L1654 CN**: 声明 TableGen defm `vfcvt_xu_f_v`。
- **L1655 EN**: Declares TableGen defm `vfcvt_x_f_v`.
  **L1655 CN**: 声明 TableGen defm `vfcvt_x_f_v`。
- **L1656 EN**: Declares TableGen defm `vfcvt_rtz_xu_f_v`.
  **L1656 CN**: 声明 TableGen defm `vfcvt_rtz_xu_f_v`。
- **L1657 EN**: Declares TableGen defm `vfcvt_rtz_x_f_v`.
  **L1657 CN**: 声明 TableGen defm `vfcvt_rtz_x_f_v`。
- **L1658 EN**: Declares TableGen defm `vfcvt_f_xu_v`.
  **L1658 CN**: 声明 TableGen defm `vfcvt_f_xu_v`。
- **L1659 EN**: Declares TableGen defm `vfcvt_f_x_v`.
  **L1659 CN**: 声明 TableGen defm `vfcvt_f_x_v`。
- **L1660 EN**: Blank line separating nearby declarations or logic blocks.
  **L1660 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1661 EN**: Declares TableGen defm `vfwcvt_f_xu_v`.
  **L1661 CN**: 声明 TableGen defm `vfwcvt_f_xu_v`。
- **L1662 EN**: Declares TableGen defm `vfwcvt_f_x_v`.
  **L1662 CN**: 声明 TableGen defm `vfwcvt_f_x_v`。
- **L1663 EN**: Declares TableGen defm `vfwcvt_xu_f_v`.
  **L1663 CN**: 声明 TableGen defm `vfwcvt_xu_f_v`。
- **L1664 EN**: Declares TableGen defm `vfwcvt_x_f_v`.
  **L1664 CN**: 声明 TableGen defm `vfwcvt_x_f_v`。
- **L1665 EN**: Declares TableGen defm `vfwcvt_rtz_xu_f_v`.
  **L1665 CN**: 声明 TableGen defm `vfwcvt_rtz_xu_f_v`。
- **L1666 EN**: Declares TableGen defm `vfwcvt_rtz_x_f_v`.
  **L1666 CN**: 声明 TableGen defm `vfwcvt_rtz_x_f_v`。
- **L1667 EN**: Declares TableGen defm `vfwcvt_f_f_v`.
  **L1667 CN**: 声明 TableGen defm `vfwcvt_f_f_v`。
- **L1668 EN**: Declares TableGen defm `vfwcvtbf16_f_f_v`.
  **L1668 CN**: 声明 TableGen defm `vfwcvtbf16_f_f_v`。
- **L1669 EN**: Blank line separating nearby declarations or logic blocks.
  **L1669 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1670 EN**: Declares TableGen defm `vfncvt_f_xu_w`.
  **L1670 CN**: 声明 TableGen defm `vfncvt_f_xu_w`。
- **L1671 EN**: Declares TableGen defm `vfncvt_f_x_w`.
  **L1671 CN**: 声明 TableGen defm `vfncvt_f_x_w`。
- **L1672 EN**: Declares TableGen defm `vfncvt_xu_f_w`.
  **L1672 CN**: 声明 TableGen defm `vfncvt_xu_f_w`。
- **L1673 EN**: Declares TableGen defm `vfncvt_x_f_w`.
  **L1673 CN**: 声明 TableGen defm `vfncvt_x_f_w`。
- **L1674 EN**: Declares TableGen defm `vfncvt_rtz_xu_f_w`.
  **L1674 CN**: 声明 TableGen defm `vfncvt_rtz_xu_f_w`。
- **L1675 EN**: Declares TableGen defm `vfncvt_rtz_x_f_w`.
  **L1675 CN**: 声明 TableGen defm `vfncvt_rtz_x_f_w`。
- **L1676 EN**: Declares TableGen defm `vfncvt_f_f_w`.
  **L1676 CN**: 声明 TableGen defm `vfncvt_f_f_w`。
- **L1677 EN**: Declares TableGen defm `vfncvtbf16_f_f_w`.
  **L1677 CN**: 声明 TableGen defm `vfncvtbf16_f_f_w`。
- **L1678 EN**: Declares TableGen defm `vfncvt_rod_f_f_w`.
  **L1678 CN**: 声明 TableGen defm `vfncvt_rod_f_f_w`。
- **L1679 EN**: Closes the current lexical scope or compound statement.
  **L1679 CN**: 结束当前词法作用域或复合语句块。
- **L1680 EN**: Blank line separating nearby declarations or logic blocks.
  **L1680 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1681-1708

````tablegen
  // Output: (vector)
  // Input: (passthru, mask type input, vl)
  def int_riscv_viota
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                                [LLVMMatchType<0>,
                                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                                 llvm_anyint_ty],
                                [IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 2;
  }
  // Output: (vector)
  // Input: (maskedoff, mask type vector_in, mask, vl, policy)
  def int_riscv_viota_mask
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                                [LLVMMatchType<0>,
                                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                                 llvm_anyint_ty, LLVMMatchType<1>],
                                [ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 3;
  }
  // Output: (vector)
  // Input: (passthru, vl)
  def int_riscv_vid : RISCVID;

  // Output: (vector)
  // Input: (maskedoff, mask, vl, policy)
  def int_riscv_vid_mask
````
- **L1681 EN**: Comment explains nearby logic, invariants, or intent: `Output: (vector)`.
  **L1681 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (vector)`。
- **L1682 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, mask type input, vl)`.
  **L1682 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, mask type input, vl)`。
- **L1683 EN**: Declares TableGen def `int_riscv_viota`.
  **L1683 CN**: 声明 TableGen def `int_riscv_viota`。
- **L1684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1684 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1685 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1686 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1687 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L1687 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L1688 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L1688 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L1689 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1689 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1690 EN**: Closes the current lexical scope or compound statement.
  **L1690 CN**: 结束当前词法作用域或复合语句块。
- **L1691 EN**: Comment explains nearby logic, invariants, or intent: `Output: (vector)`.
  **L1691 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (vector)`。
- **L1692 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, mask type vector_in, mask, vl, policy)`.
  **L1692 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, mask type vector_in, mask, vl, policy)`。
- **L1693 EN**: Declares TableGen def `int_riscv_viota_mask`.
  **L1693 CN**: 声明 TableGen def `int_riscv_viota_mask`。
- **L1694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1694 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1695 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1696 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1697 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<1>],`.
  **L1698 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<1>],`。
- **L1699 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L1699 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L1700 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1700 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1701 EN**: Closes the current lexical scope or compound statement.
  **L1701 CN**: 结束当前词法作用域或复合语句块。
- **L1702 EN**: Comment explains nearby logic, invariants, or intent: `Output: (vector)`.
  **L1702 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (vector)`。
- **L1703 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vl)`.
  **L1703 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vl)`。
- **L1704 EN**: Declares TableGen def `int_riscv_vid`.
  **L1704 CN**: 声明 TableGen def `int_riscv_vid`。
- **L1705 EN**: Blank line separating nearby declarations or logic blocks.
  **L1705 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1706 EN**: Comment explains nearby logic, invariants, or intent: `Output: (vector)`.
  **L1706 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output: (vector)`。
- **L1707 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, mask, vl, policy)`.
  **L1707 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, mask, vl, policy)`。
- **L1708 EN**: Declares TableGen def `int_riscv_vid_mask`.
  **L1708 CN**: 声明 TableGen def `int_riscv_vid_mask`。

### Lines 1709-1736

````tablegen
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                                [LLVMMatchType<0>,
                                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                                 llvm_anyint_ty, LLVMMatchType<1>],
                                [ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {
    let VLOperand = 2;
  }

  foreach nf = [2, 3, 4, 5, 6, 7, 8] in {
    defm vlseg # nf : RISCVUSSegLoad;
    defm vlseg # nf # ff : RISCVUSSegLoadFF;
    defm vlsseg # nf : RISCVSSegLoad;
    defm vloxseg # nf : RISCVISegLoad;
    defm vluxseg # nf : RISCVISegLoad;
    defm vsseg # nf : RISCVUSSegStore;
    defm vssseg # nf : RISCVSSegStore;
    defm vsoxseg # nf : RISCVISegStore;
    defm vsuxseg # nf : RISCVISegStore;
  }

  // Segment loads/stores for fixed vectors.
  // Note: we only have the masked variants because RISCVVectorPeephole
  // would lower any instructions with all-ones mask into unmasked version
  // anyway.
  foreach nf = [2, 3, 4, 5, 6, 7, 8] in {
    // Input: (pointer, mask, vl)
    def int_riscv_seg # nf # _load_mask
          : DefaultAttrsIntrinsic<!listconcat([llvm_anyvector_ty],
````
- **L1709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1709 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1710 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1710 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1711 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<1>],`.
  **L1712 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<1>],`。
- **L1713 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L1713 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L1714 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1714 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1715 EN**: Closes the current lexical scope or compound statement.
  **L1715 CN**: 结束当前词法作用域或复合语句块。
- **L1716 EN**: Blank line separating nearby declarations or logic blocks.
  **L1716 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1717 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1717 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1718 EN**: Declares TableGen defm `vlseg`.
  **L1718 CN**: 声明 TableGen defm `vlseg`。
- **L1719 EN**: Declares TableGen defm `vlseg`.
  **L1719 CN**: 声明 TableGen defm `vlseg`。
- **L1720 EN**: Declares TableGen defm `vlsseg`.
  **L1720 CN**: 声明 TableGen defm `vlsseg`。
- **L1721 EN**: Declares TableGen defm `vloxseg`.
  **L1721 CN**: 声明 TableGen defm `vloxseg`。
- **L1722 EN**: Declares TableGen defm `vluxseg`.
  **L1722 CN**: 声明 TableGen defm `vluxseg`。
- **L1723 EN**: Declares TableGen defm `vsseg`.
  **L1723 CN**: 声明 TableGen defm `vsseg`。
- **L1724 EN**: Declares TableGen defm `vssseg`.
  **L1724 CN**: 声明 TableGen defm `vssseg`。
- **L1725 EN**: Declares TableGen defm `vsoxseg`.
  **L1725 CN**: 声明 TableGen defm `vsoxseg`。
- **L1726 EN**: Declares TableGen defm `vsuxseg`.
  **L1726 CN**: 声明 TableGen defm `vsuxseg`。
- **L1727 EN**: Closes the current lexical scope or compound statement.
  **L1727 CN**: 结束当前词法作用域或复合语句块。
- **L1728 EN**: Blank line separating nearby declarations or logic blocks.
  **L1728 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1729 EN**: Comment explains nearby logic, invariants, or intent: `Segment loads/stores for fixed vectors.`.
  **L1729 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Segment loads/stores for fixed vectors.`。
- **L1730 EN**: Comment explains nearby logic, invariants, or intent: `Note: we only have the masked variants because RISCVVectorPeephole`.
  **L1730 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note: we only have the masked variants because RISCVVectorPeephole`。
- **L1731 EN**: Comment explains nearby logic, invariants, or intent: `would lower any instructions with all-ones mask into unmasked version`.
  **L1731 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`would lower any instructions with all-ones mask into unmasked version`。
- **L1732 EN**: Comment explains nearby logic, invariants, or intent: `anyway.`.
  **L1732 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`anyway.`。
- **L1733 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1733 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1734 EN**: Comment explains nearby logic, invariants, or intent: `Input: (pointer, mask, vl)`.
  **L1734 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (pointer, mask, vl)`。
- **L1735 EN**: Declares TableGen def `int_riscv_seg`.
  **L1735 CN**: 声明 TableGen def `int_riscv_seg`。
- **L1736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<!listconcat([llvm_anyvector_ty],`.
  **L1736 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<!listconcat([llvm_anyvector_ty],`。

### Lines 1737-1764

````tablegen
                                              !listsplat(LLVMMatchType<0>,
                                              !add(nf, -1))),
                                  [llvm_anyptr_ty, LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                                   llvm_anyint_ty],
                                  [NoCapture<ArgIndex<0>>, IntrReadMem]>;

    // Input: (pointer, stride, mask, vl)
    def int_riscv_sseg # nf # _load_mask
          : DefaultAttrsIntrinsic<!listconcat([llvm_anyvector_ty],
                                              !listsplat(LLVMMatchType<0>,
                                              !add(nf, -1))),
                                  [llvm_anyptr_ty, llvm_anyint_ty,
                                   LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                                   llvm_anyint_ty],
                                  [NoCapture<ArgIndex<0>>, IntrReadMem]>;

    // Input: (<stored values>..., pointer, mask, vl)
    def int_riscv_seg # nf # _store_mask
          : DefaultAttrsIntrinsic<[],
                                  !listconcat([llvm_anyvector_ty],
                                              !listsplat(LLVMMatchType<0>,
                                                          !add(nf, -1)),
                                              [llvm_anyptr_ty, LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                                               llvm_anyint_ty]),
                                  [NoCapture<ArgIndex<nf>>, IntrWriteMem]>;

    // Input: (<stored values>..., pointer, stride, mask, vl)
    def int_riscv_sseg # nf # _store_mask
````
- **L1737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listsplat(LLVMMatchType<0>,`.
  **L1737 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listsplat(LLVMMatchType<0>,`。
- **L1738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!add(nf, -1))),`.
  **L1738 CN**: 继续一个多行参数列表、初始化器或聚合项：`!add(nf, -1))),`。
- **L1739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1739 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L1740 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L1741 EN**: Executes a standalone statement or declaration: `[NoCapture<ArgIndex<0>>, IntrReadMem]>;`.
  **L1741 CN**: 执行一条独立语句或声明：`[NoCapture<ArgIndex<0>>, IntrReadMem]>;`。
- **L1742 EN**: Blank line separating nearby declarations or logic blocks.
  **L1742 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1743 EN**: Comment explains nearby logic, invariants, or intent: `Input: (pointer, stride, mask, vl)`.
  **L1743 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (pointer, stride, mask, vl)`。
- **L1744 EN**: Declares TableGen def `int_riscv_sseg`.
  **L1744 CN**: 声明 TableGen def `int_riscv_sseg`。
- **L1745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<!listconcat([llvm_anyvector_ty],`.
  **L1745 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<!listconcat([llvm_anyvector_ty],`。
- **L1746 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listsplat(LLVMMatchType<0>,`.
  **L1746 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listsplat(LLVMMatchType<0>,`。
- **L1747 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!add(nf, -1))),`.
  **L1747 CN**: 继续一个多行参数列表、初始化器或聚合项：`!add(nf, -1))),`。
- **L1748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, llvm_anyint_ty,`.
  **L1748 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, llvm_anyint_ty,`。
- **L1749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1749 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L1750 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L1751 EN**: Executes a standalone statement or declaration: `[NoCapture<ArgIndex<0>>, IntrReadMem]>;`.
  **L1751 CN**: 执行一条独立语句或声明：`[NoCapture<ArgIndex<0>>, IntrReadMem]>;`。
- **L1752 EN**: Blank line separating nearby declarations or logic blocks.
  **L1752 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1753 EN**: Comment explains nearby logic, invariants, or intent: `Input: (<stored values>..., pointer, mask, vl)`.
  **L1753 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (<stored values>..., pointer, mask, vl)`。
- **L1754 EN**: Declares TableGen def `int_riscv_seg`.
  **L1754 CN**: 声明 TableGen def `int_riscv_seg`。
- **L1755 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1755 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1756 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_anyvector_ty],`.
  **L1756 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_anyvector_ty],`。
- **L1757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listsplat(LLVMMatchType<0>,`.
  **L1757 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listsplat(LLVMMatchType<0>,`。
- **L1758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!add(nf, -1)),`.
  **L1758 CN**: 继续一个多行参数列表、初始化器或聚合项：`!add(nf, -1)),`。
- **L1759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1759 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty]),`.
  **L1760 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty]),`。
- **L1761 EN**: Executes a standalone statement or declaration: `[NoCapture<ArgIndex<nf>>, IntrWriteMem]>;`.
  **L1761 CN**: 执行一条独立语句或声明：`[NoCapture<ArgIndex<nf>>, IntrWriteMem]>;`。
- **L1762 EN**: Blank line separating nearby declarations or logic blocks.
  **L1762 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1763 EN**: Comment explains nearby logic, invariants, or intent: `Input: (<stored values>..., pointer, stride, mask, vl)`.
  **L1763 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (<stored values>..., pointer, stride, mask, vl)`。
- **L1764 EN**: Declares TableGen def `int_riscv_sseg`.
  **L1764 CN**: 声明 TableGen def `int_riscv_sseg`。

### Lines 1765-1792

````tablegen
          : DefaultAttrsIntrinsic<[],
                                  !listconcat([llvm_anyvector_ty],
                                              !listsplat(LLVMMatchType<0>,
                                                          !add(nf, -1)),
                                              [llvm_anyptr_ty, llvm_anyint_ty,
                                               LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                                               llvm_anyint_ty]),
                                  [NoCapture<ArgIndex<nf>>, IntrWriteMem]>;
  }

} // TargetPrefix = "riscv"

//===----------------------------------------------------------------------===//
// Scalar Cryptography
//
// These intrinsics will lower directly into the corresponding instructions
// added by the scalar cyptography extension, if the extension is present.

let TargetPrefix = "riscv" in {

class ScalarCryptoByteSelect32
    : DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem, IntrSpeculatable,
                             ImmArg<ArgIndex<2>>]>;

class ScalarCryptoGprGprIntrinsic32
    : DefaultAttrsIntrinsic<[llvm_i32_ty],
````
- **L1765 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1765 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1766 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_anyvector_ty],`.
  **L1766 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_anyvector_ty],`。
- **L1767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listsplat(LLVMMatchType<0>,`.
  **L1767 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listsplat(LLVMMatchType<0>,`。
- **L1768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!add(nf, -1)),`.
  **L1768 CN**: 继续一个多行参数列表、初始化器或聚合项：`!add(nf, -1)),`。
- **L1769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, llvm_anyint_ty,`.
  **L1769 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, llvm_anyint_ty,`。
- **L1770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1770 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1771 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty]),`.
  **L1771 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty]),`。
- **L1772 EN**: Executes a standalone statement or declaration: `[NoCapture<ArgIndex<nf>>, IntrWriteMem]>;`.
  **L1772 CN**: 执行一条独立语句或声明：`[NoCapture<ArgIndex<nf>>, IntrWriteMem]>;`。
- **L1773 EN**: Closes the current lexical scope or compound statement.
  **L1773 CN**: 结束当前词法作用域或复合语句块。
- **L1774 EN**: Blank line separating nearby declarations or logic blocks.
  **L1774 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1775 EN**: Continues the surrounding expression or declaration: `} // TargetPrefix = "riscv"`.
  **L1775 CN**: 继续构造周围的表达式或声明：`} // TargetPrefix = "riscv"`。
- **L1776 EN**: Blank line separating nearby declarations or logic blocks.
  **L1776 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1777 EN**: Banner comment marking a file or section boundary.
  **L1777 CN**: 横幅注释，用于标记文件或章节边界。
- **L1778 EN**: Comment explains nearby logic, invariants, or intent: `Scalar Cryptography`.
  **L1778 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scalar Cryptography`。
- **L1779 EN**: Separator comment used for visual grouping.
  **L1779 CN**: 用于视觉分组的分隔注释。
- **L1780 EN**: Comment explains nearby logic, invariants, or intent: `These intrinsics will lower directly into the corresponding instructions`.
  **L1780 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These intrinsics will lower directly into the corresponding instructions`。
- **L1781 EN**: Comment explains nearby logic, invariants, or intent: `added by the scalar cyptography extension, if the extension is present.`.
  **L1781 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`added by the scalar cyptography extension, if the extension is present.`。
- **L1782 EN**: Blank line separating nearby declarations or logic blocks.
  **L1782 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1783 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1783 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1784 EN**: Blank line separating nearby declarations or logic blocks.
  **L1784 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1785 EN**: Declares class `ScalarCryptoByteSelect32`.
  **L1785 CN**: 声明 class `ScalarCryptoByteSelect32`。
- **L1786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L1786 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L1787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L1787 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L1788 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrSpeculatable,`.
  **L1788 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrSpeculatable,`。
- **L1789 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<2>>]>;`.
  **L1789 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<2>>]>;`。
- **L1790 EN**: Blank line separating nearby declarations or logic blocks.
  **L1790 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1791 EN**: Declares class `ScalarCryptoGprGprIntrinsic32`.
  **L1791 CN**: 声明 class `ScalarCryptoGprGprIntrinsic32`。
- **L1792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L1792 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i32_ty],`。

### Lines 1793-1820

````tablegen
                            [llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem, IntrSpeculatable]>;

class ScalarCryptoGprGprIntrinsic64
    : DefaultAttrsIntrinsic<[llvm_i64_ty],
                            [llvm_i64_ty, llvm_i64_ty],
                            [IntrNoMem, IntrSpeculatable]>;

class ScalarCryptoGprIntrinsic32
    : DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty],
                            [IntrNoMem, IntrSpeculatable]>;

class ScalarCryptoGprIntrinsic64
    : DefaultAttrsIntrinsic<[llvm_i64_ty],
                            [llvm_i64_ty],
                            [IntrNoMem, IntrSpeculatable]>;

// Zknd
def int_riscv_aes32dsi  : ScalarCryptoByteSelect32,
                          ClangBuiltin<"__builtin_riscv_aes32dsi">;
def int_riscv_aes32dsmi : ScalarCryptoByteSelect32,
                          ClangBuiltin<"__builtin_riscv_aes32dsmi">;

def int_riscv_aes64ds   : ScalarCryptoGprGprIntrinsic64,
                          ClangBuiltin<"__builtin_riscv_aes64ds">;
def int_riscv_aes64dsm  : ScalarCryptoGprGprIntrinsic64,
                          ClangBuiltin<"__builtin_riscv_aes64dsm">;
````
- **L1793 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L1793 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L1794 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L1794 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L1795 EN**: Blank line separating nearby declarations or logic blocks.
  **L1795 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1796 EN**: Declares class `ScalarCryptoGprGprIntrinsic64`.
  **L1796 CN**: 声明 class `ScalarCryptoGprGprIntrinsic64`。
- **L1797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i64_ty],`.
  **L1797 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i64_ty],`。
- **L1798 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i64_ty, llvm_i64_ty],`.
  **L1798 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i64_ty, llvm_i64_ty],`。
- **L1799 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L1799 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L1800 EN**: Blank line separating nearby declarations or logic blocks.
  **L1800 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1801 EN**: Declares class `ScalarCryptoGprIntrinsic32`.
  **L1801 CN**: 声明 class `ScalarCryptoGprIntrinsic32`。
- **L1802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L1802 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L1803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L1803 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L1804 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L1804 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L1805 EN**: Blank line separating nearby declarations or logic blocks.
  **L1805 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1806 EN**: Declares class `ScalarCryptoGprIntrinsic64`.
  **L1806 CN**: 声明 class `ScalarCryptoGprIntrinsic64`。
- **L1807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i64_ty],`.
  **L1807 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i64_ty],`。
- **L1808 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i64_ty],`.
  **L1808 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i64_ty],`。
- **L1809 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L1809 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L1810 EN**: Blank line separating nearby declarations or logic blocks.
  **L1810 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1811 EN**: Comment explains nearby logic, invariants, or intent: `Zknd`.
  **L1811 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zknd`。
- **L1812 EN**: Declares TableGen def `int_riscv_aes32dsi`.
  **L1812 CN**: 声明 TableGen def `int_riscv_aes32dsi`。
- **L1813 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_aes32dsi">;`.
  **L1813 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_aes32dsi">;`。
- **L1814 EN**: Declares TableGen def `int_riscv_aes32dsmi`.
  **L1814 CN**: 声明 TableGen def `int_riscv_aes32dsmi`。
- **L1815 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_aes32dsmi">;`.
  **L1815 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_aes32dsmi">;`。
- **L1816 EN**: Blank line separating nearby declarations or logic blocks.
  **L1816 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1817 EN**: Declares TableGen def `int_riscv_aes64ds`.
  **L1817 CN**: 声明 TableGen def `int_riscv_aes64ds`。
- **L1818 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_aes64ds">;`.
  **L1818 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_aes64ds">;`。
- **L1819 EN**: Declares TableGen def `int_riscv_aes64dsm`.
  **L1819 CN**: 声明 TableGen def `int_riscv_aes64dsm`。
- **L1820 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_aes64dsm">;`.
  **L1820 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_aes64dsm">;`。

### Lines 1821-1848

````tablegen

def int_riscv_aes64im   : ScalarCryptoGprIntrinsic64,
                          ClangBuiltin<"__builtin_riscv_aes64im">;

// Zkne
def int_riscv_aes32esi  : ScalarCryptoByteSelect32,
                          ClangBuiltin<"__builtin_riscv_aes32esi">;
def int_riscv_aes32esmi : ScalarCryptoByteSelect32,
                          ClangBuiltin<"__builtin_riscv_aes32esmi">;

def int_riscv_aes64es   : ScalarCryptoGprGprIntrinsic64,
                          ClangBuiltin<"__builtin_riscv_aes64es">;
def int_riscv_aes64esm  : ScalarCryptoGprGprIntrinsic64,
                          ClangBuiltin<"__builtin_riscv_aes64esm">;

// Zknd & Zkne
def int_riscv_aes64ks2  : ScalarCryptoGprGprIntrinsic64,
                          ClangBuiltin<"__builtin_riscv_aes64ks2">;
def int_riscv_aes64ks1i : DefaultAttrsIntrinsic<[llvm_i64_ty],
                                                [llvm_i64_ty, llvm_i32_ty],
                                                [IntrNoMem, IntrSpeculatable,
                                                 ImmArg<ArgIndex<1>>]>,
                          ClangBuiltin<"__builtin_riscv_aes64ks1i">;

// Zknh
def int_riscv_sha256sig0 : ScalarCryptoGprIntrinsic32;
def int_riscv_sha256sig1 : ScalarCryptoGprIntrinsic32;
def int_riscv_sha256sum0 : ScalarCryptoGprIntrinsic32;
````
- **L1821 EN**: Blank line separating nearby declarations or logic blocks.
  **L1821 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1822 EN**: Declares TableGen def `int_riscv_aes64im`.
  **L1822 CN**: 声明 TableGen def `int_riscv_aes64im`。
- **L1823 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_aes64im">;`.
  **L1823 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_aes64im">;`。
- **L1824 EN**: Blank line separating nearby declarations or logic blocks.
  **L1824 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1825 EN**: Comment explains nearby logic, invariants, or intent: `Zkne`.
  **L1825 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zkne`。
- **L1826 EN**: Declares TableGen def `int_riscv_aes32esi`.
  **L1826 CN**: 声明 TableGen def `int_riscv_aes32esi`。
- **L1827 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_aes32esi">;`.
  **L1827 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_aes32esi">;`。
- **L1828 EN**: Declares TableGen def `int_riscv_aes32esmi`.
  **L1828 CN**: 声明 TableGen def `int_riscv_aes32esmi`。
- **L1829 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_aes32esmi">;`.
  **L1829 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_aes32esmi">;`。
- **L1830 EN**: Blank line separating nearby declarations or logic blocks.
  **L1830 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1831 EN**: Declares TableGen def `int_riscv_aes64es`.
  **L1831 CN**: 声明 TableGen def `int_riscv_aes64es`。
- **L1832 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_aes64es">;`.
  **L1832 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_aes64es">;`。
- **L1833 EN**: Declares TableGen def `int_riscv_aes64esm`.
  **L1833 CN**: 声明 TableGen def `int_riscv_aes64esm`。
- **L1834 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_aes64esm">;`.
  **L1834 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_aes64esm">;`。
- **L1835 EN**: Blank line separating nearby declarations or logic blocks.
  **L1835 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1836 EN**: Comment explains nearby logic, invariants, or intent: `Zknd & Zkne`.
  **L1836 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zknd & Zkne`。
- **L1837 EN**: Declares TableGen def `int_riscv_aes64ks2`.
  **L1837 CN**: 声明 TableGen def `int_riscv_aes64ks2`。
- **L1838 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_aes64ks2">;`.
  **L1838 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_aes64ks2">;`。
- **L1839 EN**: Declares TableGen def `int_riscv_aes64ks1i`.
  **L1839 CN**: 声明 TableGen def `int_riscv_aes64ks1i`。
- **L1840 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i64_ty, llvm_i32_ty],`.
  **L1840 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i64_ty, llvm_i32_ty],`。
- **L1841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrSpeculatable,`.
  **L1841 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrSpeculatable,`。
- **L1842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<1>>]>,`.
  **L1842 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<1>>]>,`。
- **L1843 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_aes64ks1i">;`.
  **L1843 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_aes64ks1i">;`。
- **L1844 EN**: Blank line separating nearby declarations or logic blocks.
  **L1844 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1845 EN**: Comment explains nearby logic, invariants, or intent: `Zknh`.
  **L1845 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zknh`。
- **L1846 EN**: Declares TableGen def `int_riscv_sha256sig0`.
  **L1846 CN**: 声明 TableGen def `int_riscv_sha256sig0`。
- **L1847 EN**: Declares TableGen def `int_riscv_sha256sig1`.
  **L1847 CN**: 声明 TableGen def `int_riscv_sha256sig1`。
- **L1848 EN**: Declares TableGen def `int_riscv_sha256sum0`.
  **L1848 CN**: 声明 TableGen def `int_riscv_sha256sum0`。

### Lines 1849-1876

````tablegen
def int_riscv_sha256sum1 : ScalarCryptoGprIntrinsic32;

def int_riscv_sha512sig0l : ScalarCryptoGprGprIntrinsic32,
                            ClangBuiltin<"__builtin_riscv_sha512sig0l">;
def int_riscv_sha512sig0h : ScalarCryptoGprGprIntrinsic32,
                            ClangBuiltin<"__builtin_riscv_sha512sig0h">;
def int_riscv_sha512sig1l : ScalarCryptoGprGprIntrinsic32,
                            ClangBuiltin<"__builtin_riscv_sha512sig1l">;
def int_riscv_sha512sig1h : ScalarCryptoGprGprIntrinsic32,
                            ClangBuiltin<"__builtin_riscv_sha512sig1h">;
def int_riscv_sha512sum0r : ScalarCryptoGprGprIntrinsic32,
                            ClangBuiltin<"__builtin_riscv_sha512sum0r">;
def int_riscv_sha512sum1r : ScalarCryptoGprGprIntrinsic32,
                            ClangBuiltin<"__builtin_riscv_sha512sum1r">;

def int_riscv_sha512sig0 : ScalarCryptoGprIntrinsic64,
                           ClangBuiltin<"__builtin_riscv_sha512sig0">;
def int_riscv_sha512sig1 : ScalarCryptoGprIntrinsic64,
                           ClangBuiltin<"__builtin_riscv_sha512sig1">;
def int_riscv_sha512sum0 : ScalarCryptoGprIntrinsic64,
                           ClangBuiltin<"__builtin_riscv_sha512sum0">;
def int_riscv_sha512sum1 : ScalarCryptoGprIntrinsic64,
                           ClangBuiltin<"__builtin_riscv_sha512sum1">;

// Zksed
def int_riscv_sm4ks      : ScalarCryptoByteSelect32;
def int_riscv_sm4ed      : ScalarCryptoByteSelect32;

````
- **L1849 EN**: Declares TableGen def `int_riscv_sha256sum1`.
  **L1849 CN**: 声明 TableGen def `int_riscv_sha256sum1`。
- **L1850 EN**: Blank line separating nearby declarations or logic blocks.
  **L1850 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1851 EN**: Declares TableGen def `int_riscv_sha512sig0l`.
  **L1851 CN**: 声明 TableGen def `int_riscv_sha512sig0l`。
- **L1852 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_sha512sig0l">;`.
  **L1852 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_sha512sig0l">;`。
- **L1853 EN**: Declares TableGen def `int_riscv_sha512sig0h`.
  **L1853 CN**: 声明 TableGen def `int_riscv_sha512sig0h`。
- **L1854 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_sha512sig0h">;`.
  **L1854 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_sha512sig0h">;`。
- **L1855 EN**: Declares TableGen def `int_riscv_sha512sig1l`.
  **L1855 CN**: 声明 TableGen def `int_riscv_sha512sig1l`。
- **L1856 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_sha512sig1l">;`.
  **L1856 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_sha512sig1l">;`。
- **L1857 EN**: Declares TableGen def `int_riscv_sha512sig1h`.
  **L1857 CN**: 声明 TableGen def `int_riscv_sha512sig1h`。
- **L1858 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_sha512sig1h">;`.
  **L1858 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_sha512sig1h">;`。
- **L1859 EN**: Declares TableGen def `int_riscv_sha512sum0r`.
  **L1859 CN**: 声明 TableGen def `int_riscv_sha512sum0r`。
- **L1860 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_sha512sum0r">;`.
  **L1860 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_sha512sum0r">;`。
- **L1861 EN**: Declares TableGen def `int_riscv_sha512sum1r`.
  **L1861 CN**: 声明 TableGen def `int_riscv_sha512sum1r`。
- **L1862 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_sha512sum1r">;`.
  **L1862 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_sha512sum1r">;`。
- **L1863 EN**: Blank line separating nearby declarations or logic blocks.
  **L1863 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1864 EN**: Declares TableGen def `int_riscv_sha512sig0`.
  **L1864 CN**: 声明 TableGen def `int_riscv_sha512sig0`。
- **L1865 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_sha512sig0">;`.
  **L1865 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_sha512sig0">;`。
- **L1866 EN**: Declares TableGen def `int_riscv_sha512sig1`.
  **L1866 CN**: 声明 TableGen def `int_riscv_sha512sig1`。
- **L1867 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_sha512sig1">;`.
  **L1867 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_sha512sig1">;`。
- **L1868 EN**: Declares TableGen def `int_riscv_sha512sum0`.
  **L1868 CN**: 声明 TableGen def `int_riscv_sha512sum0`。
- **L1869 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_sha512sum0">;`.
  **L1869 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_sha512sum0">;`。
- **L1870 EN**: Declares TableGen def `int_riscv_sha512sum1`.
  **L1870 CN**: 声明 TableGen def `int_riscv_sha512sum1`。
- **L1871 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_riscv_sha512sum1">;`.
  **L1871 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_riscv_sha512sum1">;`。
- **L1872 EN**: Blank line separating nearby declarations or logic blocks.
  **L1872 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1873 EN**: Comment explains nearby logic, invariants, or intent: `Zksed`.
  **L1873 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zksed`。
- **L1874 EN**: Declares TableGen def `int_riscv_sm4ks`.
  **L1874 CN**: 声明 TableGen def `int_riscv_sm4ks`。
- **L1875 EN**: Declares TableGen def `int_riscv_sm4ed`.
  **L1875 CN**: 声明 TableGen def `int_riscv_sm4ed`。
- **L1876 EN**: Blank line separating nearby declarations or logic blocks.
  **L1876 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1877-1904

````tablegen
// Zksh
def int_riscv_sm3p0      : ScalarCryptoGprIntrinsic32;
def int_riscv_sm3p1      : ScalarCryptoGprIntrinsic32;
} // TargetPrefix = "riscv"

//===----------------------------------------------------------------------===//
// Vector Cryptography
//
// These intrinsics will lower directly into the corresponding instructions
// added by the vector cyptography extension, if the extension is present.
let TargetPrefix = "riscv" in {
  // Zvkb
  defm vandn             : RISCVBinaryAAX;
  defm vbrev8            : RISCVUnaryAA;
  defm vrev8             : RISCVUnaryAA;
  defm vrol              : RISCVBinaryAAX;
  defm vror              : RISCVBinaryAAX;

  // Zvbb
  defm vbrev             : RISCVUnaryAA;
  defm vclz              : RISCVUnaryAA;
  defm vctz              : RISCVUnaryAA;
  defm vcpopv            : RISCVUnaryAA;
  defm vwsll             : RISCVBinaryABX;

  // Zvbc
  defm vclmul            : RISCVBinaryAAX;
  defm vclmulh           : RISCVBinaryAAX;
````
- **L1877 EN**: Comment explains nearby logic, invariants, or intent: `Zksh`.
  **L1877 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zksh`。
- **L1878 EN**: Declares TableGen def `int_riscv_sm3p0`.
  **L1878 CN**: 声明 TableGen def `int_riscv_sm3p0`。
- **L1879 EN**: Declares TableGen def `int_riscv_sm3p1`.
  **L1879 CN**: 声明 TableGen def `int_riscv_sm3p1`。
- **L1880 EN**: Continues the surrounding expression or declaration: `} // TargetPrefix = "riscv"`.
  **L1880 CN**: 继续构造周围的表达式或声明：`} // TargetPrefix = "riscv"`。
- **L1881 EN**: Blank line separating nearby declarations or logic blocks.
  **L1881 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1882 EN**: Banner comment marking a file or section boundary.
  **L1882 CN**: 横幅注释，用于标记文件或章节边界。
- **L1883 EN**: Comment explains nearby logic, invariants, or intent: `Vector Cryptography`.
  **L1883 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Cryptography`。
- **L1884 EN**: Separator comment used for visual grouping.
  **L1884 CN**: 用于视觉分组的分隔注释。
- **L1885 EN**: Comment explains nearby logic, invariants, or intent: `These intrinsics will lower directly into the corresponding instructions`.
  **L1885 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These intrinsics will lower directly into the corresponding instructions`。
- **L1886 EN**: Comment explains nearby logic, invariants, or intent: `added by the vector cyptography extension, if the extension is present.`.
  **L1886 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`added by the vector cyptography extension, if the extension is present.`。
- **L1887 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1887 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1888 EN**: Comment explains nearby logic, invariants, or intent: `Zvkb`.
  **L1888 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zvkb`。
- **L1889 EN**: Declares TableGen defm `vandn`.
  **L1889 CN**: 声明 TableGen defm `vandn`。
- **L1890 EN**: Declares TableGen defm `vbrev8`.
  **L1890 CN**: 声明 TableGen defm `vbrev8`。
- **L1891 EN**: Declares TableGen defm `vrev8`.
  **L1891 CN**: 声明 TableGen defm `vrev8`。
- **L1892 EN**: Declares TableGen defm `vrol`.
  **L1892 CN**: 声明 TableGen defm `vrol`。
- **L1893 EN**: Declares TableGen defm `vror`.
  **L1893 CN**: 声明 TableGen defm `vror`。
- **L1894 EN**: Blank line separating nearby declarations or logic blocks.
  **L1894 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1895 EN**: Comment explains nearby logic, invariants, or intent: `Zvbb`.
  **L1895 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zvbb`。
- **L1896 EN**: Declares TableGen defm `vbrev`.
  **L1896 CN**: 声明 TableGen defm `vbrev`。
- **L1897 EN**: Declares TableGen defm `vclz`.
  **L1897 CN**: 声明 TableGen defm `vclz`。
- **L1898 EN**: Declares TableGen defm `vctz`.
  **L1898 CN**: 声明 TableGen defm `vctz`。
- **L1899 EN**: Declares TableGen defm `vcpopv`.
  **L1899 CN**: 声明 TableGen defm `vcpopv`。
- **L1900 EN**: Declares TableGen defm `vwsll`.
  **L1900 CN**: 声明 TableGen defm `vwsll`。
- **L1901 EN**: Blank line separating nearby declarations or logic blocks.
  **L1901 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1902 EN**: Comment explains nearby logic, invariants, or intent: `Zvbc`.
  **L1902 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zvbc`。
- **L1903 EN**: Declares TableGen defm `vclmul`.
  **L1903 CN**: 声明 TableGen defm `vclmul`。
- **L1904 EN**: Declares TableGen defm `vclmulh`.
  **L1904 CN**: 声明 TableGen defm `vclmulh`。

### Lines 1905-1932

````tablegen

  // Zvkg
  def int_riscv_vghsh    : RISCVBinaryAAXUnMaskedZvk;
  def int_riscv_vgmul_vv : RISCVUnaryAAUnMaskedZvk<IsVS=0>;

  // Zvkned
  defm vaesdf            : RISCVUnaryAAUnMaskedZvk;
  defm vaesdm            : RISCVUnaryAAUnMaskedZvk;
  defm vaesef            : RISCVUnaryAAUnMaskedZvk;
  defm vaesem            : RISCVUnaryAAUnMaskedZvk;
  def int_riscv_vaeskf1  : RISCVBinaryAAXUnMasked<IsVI=1>;
  def int_riscv_vaeskf2  : RISCVBinaryAAXUnMaskedZvk<IsVI=1>;
  defm vaesz             : RISCVUnaryAAUnMaskedZvk<HasVV=0>;

  // Zvknha or Zvknhb
  def int_riscv_vsha2ch  : RISCVBinaryAAXUnMaskedZvk;
  def int_riscv_vsha2cl  : RISCVBinaryAAXUnMaskedZvk;
  def int_riscv_vsha2ms  : RISCVBinaryAAXUnMaskedZvk;

  // Zvksed
  def int_riscv_vsm4k    : RISCVBinaryAAXUnMasked<IsVI=1>;
  defm vsm4r             : RISCVUnaryAAUnMaskedZvk;

  // Zvksh
  def int_riscv_vsm3c    : RISCVBinaryAAXUnMaskedZvk<IsVI=1>;
  def int_riscv_vsm3me   : RISCVBinaryAAXUnMasked;
} // TargetPrefix = "riscv"

````
- **L1905 EN**: Blank line separating nearby declarations or logic blocks.
  **L1905 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1906 EN**: Comment explains nearby logic, invariants, or intent: `Zvkg`.
  **L1906 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zvkg`。
- **L1907 EN**: Declares TableGen def `int_riscv_vghsh`.
  **L1907 CN**: 声明 TableGen def `int_riscv_vghsh`。
- **L1908 EN**: Declares TableGen def `int_riscv_vgmul_vv`.
  **L1908 CN**: 声明 TableGen def `int_riscv_vgmul_vv`。
- **L1909 EN**: Blank line separating nearby declarations or logic blocks.
  **L1909 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1910 EN**: Comment explains nearby logic, invariants, or intent: `Zvkned`.
  **L1910 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zvkned`。
- **L1911 EN**: Declares TableGen defm `vaesdf`.
  **L1911 CN**: 声明 TableGen defm `vaesdf`。
- **L1912 EN**: Declares TableGen defm `vaesdm`.
  **L1912 CN**: 声明 TableGen defm `vaesdm`。
- **L1913 EN**: Declares TableGen defm `vaesef`.
  **L1913 CN**: 声明 TableGen defm `vaesef`。
- **L1914 EN**: Declares TableGen defm `vaesem`.
  **L1914 CN**: 声明 TableGen defm `vaesem`。
- **L1915 EN**: Declares TableGen def `int_riscv_vaeskf1`.
  **L1915 CN**: 声明 TableGen def `int_riscv_vaeskf1`。
- **L1916 EN**: Declares TableGen def `int_riscv_vaeskf2`.
  **L1916 CN**: 声明 TableGen def `int_riscv_vaeskf2`。
- **L1917 EN**: Declares TableGen defm `vaesz`.
  **L1917 CN**: 声明 TableGen defm `vaesz`。
- **L1918 EN**: Blank line separating nearby declarations or logic blocks.
  **L1918 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1919 EN**: Comment explains nearby logic, invariants, or intent: `Zvknha or Zvknhb`.
  **L1919 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zvknha or Zvknhb`。
- **L1920 EN**: Declares TableGen def `int_riscv_vsha2ch`.
  **L1920 CN**: 声明 TableGen def `int_riscv_vsha2ch`。
- **L1921 EN**: Declares TableGen def `int_riscv_vsha2cl`.
  **L1921 CN**: 声明 TableGen def `int_riscv_vsha2cl`。
- **L1922 EN**: Declares TableGen def `int_riscv_vsha2ms`.
  **L1922 CN**: 声明 TableGen def `int_riscv_vsha2ms`。
- **L1923 EN**: Blank line separating nearby declarations or logic blocks.
  **L1923 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1924 EN**: Comment explains nearby logic, invariants, or intent: `Zvksed`.
  **L1924 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zvksed`。
- **L1925 EN**: Declares TableGen def `int_riscv_vsm4k`.
  **L1925 CN**: 声明 TableGen def `int_riscv_vsm4k`。
- **L1926 EN**: Declares TableGen defm `vsm4r`.
  **L1926 CN**: 声明 TableGen defm `vsm4r`。
- **L1927 EN**: Blank line separating nearby declarations or logic blocks.
  **L1927 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1928 EN**: Comment explains nearby logic, invariants, or intent: `Zvksh`.
  **L1928 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zvksh`。
- **L1929 EN**: Declares TableGen def `int_riscv_vsm3c`.
  **L1929 CN**: 声明 TableGen def `int_riscv_vsm3c`。
- **L1930 EN**: Declares TableGen def `int_riscv_vsm3me`.
  **L1930 CN**: 声明 TableGen def `int_riscv_vsm3me`。
- **L1931 EN**: Continues the surrounding expression or declaration: `} // TargetPrefix = "riscv"`.
  **L1931 CN**: 继续构造周围的表达式或声明：`} // TargetPrefix = "riscv"`。
- **L1932 EN**: Blank line separating nearby declarations or logic blocks.
  **L1932 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1933-1960

````tablegen
// Zvabd - Vector Absolute Difference
//===----------------------------------------------------------------------===//
let TargetPrefix = "riscv" in {
  defm vabs    : RISCVUnaryAA;
  defm vabd    : RISCVBinaryAAA;
  defm vabdu   : RISCVBinaryAAA;
  defm vwabda  : RISCVTernaryWide;
  defm vwabdau : RISCVTernaryWide;
} // TargetPrefix = "riscv"

// Zvzip - Reordering Structured Data in Vector Registers
//===----------------------------------------------------------------------===//
let TargetPrefix = "riscv" in {
  multiclass RISCVZip {
    // Input: (passthru, vector_in, vector_in, vl)
    def "int_riscv_" # NAME :
      DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [LLVMMatchType<0>, llvm_anyvector_ty,
                             LLVMMatchType<1>, llvm_anyint_ty],
                            [IntrNoMem]>, RISCVVIntrinsic {
      let VLOperand = 3;
    }

    // Input: (maskedoff, vector_in, vector_in, mask, vl, policy)
    def "int_riscv_" # NAME # "_mask" :
      DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [LLVMMatchType<0>, llvm_anyvector_ty,
                             LLVMMatchType<1>,
````
- **L1933 EN**: Comment explains nearby logic, invariants, or intent: `Zvabd - Vector Absolute Difference`.
  **L1933 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zvabd - Vector Absolute Difference`。
- **L1934 EN**: Banner comment marking a file or section boundary.
  **L1934 CN**: 横幅注释，用于标记文件或章节边界。
- **L1935 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1935 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1936 EN**: Declares TableGen defm `vabs`.
  **L1936 CN**: 声明 TableGen defm `vabs`。
- **L1937 EN**: Declares TableGen defm `vabd`.
  **L1937 CN**: 声明 TableGen defm `vabd`。
- **L1938 EN**: Declares TableGen defm `vabdu`.
  **L1938 CN**: 声明 TableGen defm `vabdu`。
- **L1939 EN**: Declares TableGen defm `vwabda`.
  **L1939 CN**: 声明 TableGen defm `vwabda`。
- **L1940 EN**: Declares TableGen defm `vwabdau`.
  **L1940 CN**: 声明 TableGen defm `vwabdau`。
- **L1941 EN**: Continues the surrounding expression or declaration: `} // TargetPrefix = "riscv"`.
  **L1941 CN**: 继续构造周围的表达式或声明：`} // TargetPrefix = "riscv"`。
- **L1942 EN**: Blank line separating nearby declarations or logic blocks.
  **L1942 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1943 EN**: Comment explains nearby logic, invariants, or intent: `Zvzip - Reordering Structured Data in Vector Registers`.
  **L1943 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zvzip - Reordering Structured Data in Vector Registers`。
- **L1944 EN**: Banner comment marking a file or section boundary.
  **L1944 CN**: 横幅注释，用于标记文件或章节边界。
- **L1945 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1945 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1946 EN**: Declares TableGen multiclass `RISCVZip`.
  **L1946 CN**: 声明 TableGen multiclass `RISCVZip`。
- **L1947 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vector_in, vl)`.
  **L1947 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vector_in, vl)`。
- **L1948 EN**: Declares TableGen def `"int_riscv_"`.
  **L1948 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1949 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L1950 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L1951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>, llvm_anyint_ty],`.
  **L1951 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>, llvm_anyint_ty],`。
- **L1952 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L1952 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L1953 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1953 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1954 EN**: Closes the current lexical scope or compound statement.
  **L1954 CN**: 结束当前词法作用域或复合语句块。
- **L1955 EN**: Blank line separating nearby declarations or logic blocks.
  **L1955 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1956 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, vector_in, mask, vl, policy)`.
  **L1956 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, vector_in, mask, vl, policy)`。
- **L1957 EN**: Declares TableGen def `"int_riscv_"`.
  **L1957 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1958 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1958 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1959 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L1959 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L1960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>,`.
  **L1960 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>,`。

### Lines 1961-1988

````tablegen
                             LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>,
                             llvm_anyint_ty, LLVMMatchType<2>],
                            [ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {
      let VLOperand = 4;
    }
  }

  multiclass RISCVUnzip {
    // Input: (passthru, vector_in, vl)
    def "int_riscv_" # NAME :
      DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [LLVMMatchType<0>, llvm_anyvector_ty,
                             llvm_anyint_ty],
                            [IntrNoMem]>, RISCVVIntrinsic {
      let VLOperand = 2;
    }

    // Input: (maskedoff, vector_in, mask, vl, policy)
    def "int_riscv_" # NAME # "_mask" :
      DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [LLVMMatchType<0>, llvm_anyvector_ty,
                             LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                             llvm_anyint_ty, LLVMMatchType<2>],
                            [ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {
      let VLOperand = 3;
    }
  }

````
- **L1961 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>,`.
  **L1961 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>,`。
- **L1962 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L1962 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L1963 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L1963 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L1964 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1964 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1965 EN**: Closes the current lexical scope or compound statement.
  **L1965 CN**: 结束当前词法作用域或复合语句块。
- **L1966 EN**: Closes the current lexical scope or compound statement.
  **L1966 CN**: 结束当前词法作用域或复合语句块。
- **L1967 EN**: Blank line separating nearby declarations or logic blocks.
  **L1967 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1968 EN**: Declares TableGen multiclass `RISCVUnzip`.
  **L1968 CN**: 声明 TableGen multiclass `RISCVUnzip`。
- **L1969 EN**: Comment explains nearby logic, invariants, or intent: `Input: (passthru, vector_in, vl)`.
  **L1969 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (passthru, vector_in, vl)`。
- **L1970 EN**: Declares TableGen def `"int_riscv_"`.
  **L1970 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1971 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1971 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L1972 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L1973 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty],`.
  **L1973 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty],`。
- **L1974 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]>, RISCVVIntrinsic {`.
  **L1974 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]>, RISCVVIntrinsic {`。
- **L1975 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1975 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1976 EN**: Closes the current lexical scope or compound statement.
  **L1976 CN**: 结束当前词法作用域或复合语句块。
- **L1977 EN**: Blank line separating nearby declarations or logic blocks.
  **L1977 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1978 EN**: Comment explains nearby logic, invariants, or intent: `Input: (maskedoff, vector_in, mask, vl, policy)`.
  **L1978 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (maskedoff, vector_in, mask, vl, policy)`。
- **L1979 EN**: Declares TableGen def `"int_riscv_"`.
  **L1979 CN**: 声明 TableGen def `"int_riscv_"`。
- **L1980 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1980 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1981 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L1981 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L1982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1982 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1983 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L1983 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L1984 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`.
  **L1984 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, IntrNoMem]>, RISCVVIntrinsic {`。
- **L1985 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1985 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1986 EN**: Closes the current lexical scope or compound statement.
  **L1986 CN**: 结束当前词法作用域或复合语句块。
- **L1987 EN**: Closes the current lexical scope or compound statement.
  **L1987 CN**: 结束当前词法作用域或复合语句块。
- **L1988 EN**: Blank line separating nearby declarations or logic blocks.
  **L1988 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1989-2016

````tablegen
  defm vzip    : RISCVZip;
  defm vunzipe : RISCVUnzip;
  defm vunzipo : RISCVUnzip;
  defm vpaire  : RISCVBinaryAAA;
  defm vpairo  : RISCVBinaryAAA;
}

//===----------------------------------------------------------------------===//
// Zvdot4a8i - Vector 4-element Dot Product of packed 8-bit Integers
//
// 8-bit Integer dot-product instructions performing the dot product between two
// 4-element vectors of 8-bit integer elements and accumulating it into a 32-bit
// integer accumulator.
let TargetPrefix = "riscv" in {
  // We use llvm_anyvector_ty and llvm_anyint_ty for future extensibility
  // purpose but only EEW=32 is defined for now
  // Input: (vector_in, vector_in, vector_in/scalar_in, vl, policy)
  class RISCVVDOT4AUnMasked<bit HasVV>
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                                [LLVMMatchType<0>, LLVMMatchType<0>,
                                 !if(HasVV, llvm_any_ty, llvm_anyint_ty),
                                 llvm_anyint_ty, LLVMMatchType<2>],
                                [ImmArg<ArgIndex<4>>, IntrNoMem]>,
                                RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 3;
  }
  // Input: (vector_in, vector_in, vector_in/scalar_in, mask, vl, policy)
````
- **L1989 EN**: Declares TableGen defm `vzip`.
  **L1989 CN**: 声明 TableGen defm `vzip`。
- **L1990 EN**: Declares TableGen defm `vunzipe`.
  **L1990 CN**: 声明 TableGen defm `vunzipe`。
- **L1991 EN**: Declares TableGen defm `vunzipo`.
  **L1991 CN**: 声明 TableGen defm `vunzipo`。
- **L1992 EN**: Declares TableGen defm `vpaire`.
  **L1992 CN**: 声明 TableGen defm `vpaire`。
- **L1993 EN**: Declares TableGen defm `vpairo`.
  **L1993 CN**: 声明 TableGen defm `vpairo`。
- **L1994 EN**: Closes the current lexical scope or compound statement.
  **L1994 CN**: 结束当前词法作用域或复合语句块。
- **L1995 EN**: Blank line separating nearby declarations or logic blocks.
  **L1995 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1996 EN**: Banner comment marking a file or section boundary.
  **L1996 CN**: 横幅注释，用于标记文件或章节边界。
- **L1997 EN**: Comment explains nearby logic, invariants, or intent: `Zvdot4a8i - Vector 4-element Dot Product of packed 8-bit Integers`.
  **L1997 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zvdot4a8i - Vector 4-element Dot Product of packed 8-bit Integers`。
- **L1998 EN**: Separator comment used for visual grouping.
  **L1998 CN**: 用于视觉分组的分隔注释。
- **L1999 EN**: Comment explains nearby logic, invariants, or intent: `8-bit Integer dot-product instructions performing the dot product between two`.
  **L1999 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`8-bit Integer dot-product instructions performing the dot product between two`。
- **L2000 EN**: Comment explains nearby logic, invariants, or intent: `4-element vectors of 8-bit integer elements and accumulating it into a 32-bit`.
  **L2000 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`4-element vectors of 8-bit integer elements and accumulating it into a 32-bit`。
- **L2001 EN**: Comment explains nearby logic, invariants, or intent: `integer accumulator.`.
  **L2001 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`integer accumulator.`。
- **L2002 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2002 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2003 EN**: Comment explains nearby logic, invariants, or intent: `We use llvm_anyvector_ty and llvm_anyint_ty for future extensibility`.
  **L2003 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`We use llvm_anyvector_ty and llvm_anyint_ty for future extensibility`。
- **L2004 EN**: Comment explains nearby logic, invariants, or intent: `purpose but only EEW=32 is defined for now`.
  **L2004 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`purpose but only EEW=32 is defined for now`。
- **L2005 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in, vector_in/scalar_in, vl, policy)`.
  **L2005 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in, vector_in/scalar_in, vl, policy)`。
- **L2006 EN**: Declares class `RISCVVDOT4AUnMasked<bit`.
  **L2006 CN**: 声明 class `RISCVVDOT4AUnMasked<bit`。
- **L2007 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L2007 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L2008 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L2008 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L2009 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(HasVV, llvm_any_ty, llvm_anyint_ty),`.
  **L2009 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(HasVV, llvm_any_ty, llvm_anyint_ty),`。
- **L2010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L2010 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L2011 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<4>>, IntrNoMem]>,`.
  **L2011 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<4>>, IntrNoMem]>,`。
- **L2012 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L2012 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L2013 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2013 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2014 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2014 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2015 EN**: Closes the current lexical scope or compound statement.
  **L2015 CN**: 结束当前词法作用域或复合语句块。
- **L2016 EN**: Comment explains nearby logic, invariants, or intent: `Input: (vector_in, vector_in, vector_in/scalar_in, mask, vl, policy)`.
  **L2016 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input: (vector_in, vector_in, vector_in/scalar_in, mask, vl, policy)`。

### Lines 2017-2044

````tablegen
  class RISCVVDOT4AMasked<bit HasVV>
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                                [LLVMMatchType<0>, LLVMMatchType<0>,
                                 !if(HasVV, llvm_any_ty, llvm_anyint_ty),
                                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                                 llvm_anyint_ty, LLVMMatchType<2>],
                                [ImmArg<ArgIndex<5>>, IntrNoMem]>,
                                RISCVVIntrinsic {
    let ScalarOperand = 2;
    let VLOperand = 4;
  }

  multiclass RISCVVDOT4A<bit HasVV = 1> {
    def "int_riscv_" # NAME : RISCVVDOT4AUnMasked<HasVV=HasVV>;
    def "int_riscv_" # NAME # "_mask" : RISCVVDOT4AMasked<HasVV=HasVV>;
  }

  defm vdot4a   : RISCVVDOT4A;
  defm vdot4au  : RISCVVDOT4A;
  defm vdot4asu : RISCVVDOT4A;
  defm vdot4aus : RISCVVDOT4A<HasVV=0>;
} // TargetPrefix = "riscv"


// Zihintpause extensions
//===----------------------------------------------------------------------===//
let TargetPrefix = "riscv" in
def int_riscv_pause : DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;
````
- **L2017 EN**: Declares class `RISCVVDOT4AMasked<bit`.
  **L2017 CN**: 声明 class `RISCVVDOT4AMasked<bit`。
- **L2018 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L2018 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L2019 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L2019 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L2020 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(HasVV, llvm_any_ty, llvm_anyint_ty),`.
  **L2020 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(HasVV, llvm_any_ty, llvm_anyint_ty),`。
- **L2021 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L2021 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L2022 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyint_ty, LLVMMatchType<2>],`.
  **L2022 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyint_ty, LLVMMatchType<2>],`。
- **L2023 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<5>>, IntrNoMem]>,`.
  **L2023 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<5>>, IntrNoMem]>,`。
- **L2024 EN**: Continues the surrounding expression or declaration: `RISCVVIntrinsic {`.
  **L2024 CN**: 继续构造周围的表达式或声明：`RISCVVIntrinsic {`。
- **L2025 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2025 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2026 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2026 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2027 EN**: Closes the current lexical scope or compound statement.
  **L2027 CN**: 结束当前词法作用域或复合语句块。
- **L2028 EN**: Blank line separating nearby declarations or logic blocks.
  **L2028 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2029 EN**: Declares TableGen multiclass `RISCVVDOT4A`.
  **L2029 CN**: 声明 TableGen multiclass `RISCVVDOT4A`。
- **L2030 EN**: Declares TableGen def `"int_riscv_"`.
  **L2030 CN**: 声明 TableGen def `"int_riscv_"`。
- **L2031 EN**: Declares TableGen def `"int_riscv_"`.
  **L2031 CN**: 声明 TableGen def `"int_riscv_"`。
- **L2032 EN**: Closes the current lexical scope or compound statement.
  **L2032 CN**: 结束当前词法作用域或复合语句块。
- **L2033 EN**: Blank line separating nearby declarations or logic blocks.
  **L2033 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2034 EN**: Declares TableGen defm `vdot4a`.
  **L2034 CN**: 声明 TableGen defm `vdot4a`。
- **L2035 EN**: Declares TableGen defm `vdot4au`.
  **L2035 CN**: 声明 TableGen defm `vdot4au`。
- **L2036 EN**: Declares TableGen defm `vdot4asu`.
  **L2036 CN**: 声明 TableGen defm `vdot4asu`。
- **L2037 EN**: Declares TableGen defm `vdot4aus`.
  **L2037 CN**: 声明 TableGen defm `vdot4aus`。
- **L2038 EN**: Continues the surrounding expression or declaration: `} // TargetPrefix = "riscv"`.
  **L2038 CN**: 继续构造周围的表达式或声明：`} // TargetPrefix = "riscv"`。
- **L2039 EN**: Blank line separating nearby declarations or logic blocks.
  **L2039 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2040 EN**: Blank line separating nearby declarations or logic blocks.
  **L2040 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2041 EN**: Comment explains nearby logic, invariants, or intent: `Zihintpause extensions`.
  **L2041 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zihintpause extensions`。
- **L2042 EN**: Banner comment marking a file or section boundary.
  **L2042 CN**: 横幅注释，用于标记文件或章节边界。
- **L2043 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2043 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2044 EN**: Declares TableGen def `int_riscv_pause`.
  **L2044 CN**: 声明 TableGen def `int_riscv_pause`。

### Lines 2045-2072

````tablegen


//===----------------------------------------------------------------------===//
// Zvfofp8min - OFP8 conversion extension
// The Zvfofp8min extension provides basic support for the two 8-bit
// floating-point formats defined in the Open Compute Project OFP8
// specification, OFP8 E4M3 and OFP8 E5M2.
let TargetPrefix = "riscv" in {
  // OFP8 to BF16 conversion instructions
  defm vfwcvt_f_f_v_alt : RISCVConversion;
  // BF16 to OFP8 conversion instructions
  defm vfncvt_sat_f_f_w : RISCVConversionRoundingMode;
  defm vfncvt_f_f_w_alt : RISCVConversionRoundingMode;
  defm vfncvt_sat_f_f_w_alt : RISCVConversionRoundingMode;
  // FP32 to OFP8 conversion instructions
  defm vfncvt_f_f_q : RISCVConversionRoundingMode;
  defm vfncvt_f_f_q_alt : RISCVConversionRoundingMode;
  defm vfncvt_sat_f_f_q : RISCVConversionRoundingMode;
  defm vfncvt_sat_f_f_q_alt : RISCVConversionRoundingMode;
} // TargetPrefix = "riscv"

// Vendor extensions
//===----------------------------------------------------------------------===//
include "llvm/IR/IntrinsicsRISCVXTHead.td"
include "llvm/IR/IntrinsicsRISCVXsf.td"
include "llvm/IR/IntrinsicsRISCVXCV.td"
include "llvm/IR/IntrinsicsRISCVXAndes.td"
include "llvm/IR/IntrinsicsRISCVXMIPS.td"
````
- **L2045 EN**: Blank line separating nearby declarations or logic blocks.
  **L2045 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2046 EN**: Blank line separating nearby declarations or logic blocks.
  **L2046 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2047 EN**: Banner comment marking a file or section boundary.
  **L2047 CN**: 横幅注释，用于标记文件或章节边界。
- **L2048 EN**: Comment explains nearby logic, invariants, or intent: `Zvfofp8min - OFP8 conversion extension`.
  **L2048 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zvfofp8min - OFP8 conversion extension`。
- **L2049 EN**: Comment explains nearby logic, invariants, or intent: `The Zvfofp8min extension provides basic support for the two 8-bit`.
  **L2049 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The Zvfofp8min extension provides basic support for the two 8-bit`。
- **L2050 EN**: Comment explains nearby logic, invariants, or intent: `floating-point formats defined in the Open Compute Project OFP8`.
  **L2050 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`floating-point formats defined in the Open Compute Project OFP8`。
- **L2051 EN**: Comment explains nearby logic, invariants, or intent: `specification, OFP8 E4M3 and OFP8 E5M2.`.
  **L2051 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`specification, OFP8 E4M3 and OFP8 E5M2.`。
- **L2052 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2052 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2053 EN**: Comment explains nearby logic, invariants, or intent: `OFP8 to BF16 conversion instructions`.
  **L2053 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`OFP8 to BF16 conversion instructions`。
- **L2054 EN**: Declares TableGen defm `vfwcvt_f_f_v_alt`.
  **L2054 CN**: 声明 TableGen defm `vfwcvt_f_f_v_alt`。
- **L2055 EN**: Comment explains nearby logic, invariants, or intent: `BF16 to OFP8 conversion instructions`.
  **L2055 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BF16 to OFP8 conversion instructions`。
- **L2056 EN**: Declares TableGen defm `vfncvt_sat_f_f_w`.
  **L2056 CN**: 声明 TableGen defm `vfncvt_sat_f_f_w`。
- **L2057 EN**: Declares TableGen defm `vfncvt_f_f_w_alt`.
  **L2057 CN**: 声明 TableGen defm `vfncvt_f_f_w_alt`。
- **L2058 EN**: Declares TableGen defm `vfncvt_sat_f_f_w_alt`.
  **L2058 CN**: 声明 TableGen defm `vfncvt_sat_f_f_w_alt`。
- **L2059 EN**: Comment explains nearby logic, invariants, or intent: `FP32 to OFP8 conversion instructions`.
  **L2059 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FP32 to OFP8 conversion instructions`。
- **L2060 EN**: Declares TableGen defm `vfncvt_f_f_q`.
  **L2060 CN**: 声明 TableGen defm `vfncvt_f_f_q`。
- **L2061 EN**: Declares TableGen defm `vfncvt_f_f_q_alt`.
  **L2061 CN**: 声明 TableGen defm `vfncvt_f_f_q_alt`。
- **L2062 EN**: Declares TableGen defm `vfncvt_sat_f_f_q`.
  **L2062 CN**: 声明 TableGen defm `vfncvt_sat_f_f_q`。
- **L2063 EN**: Declares TableGen defm `vfncvt_sat_f_f_q_alt`.
  **L2063 CN**: 声明 TableGen defm `vfncvt_sat_f_f_q_alt`。
- **L2064 EN**: Continues the surrounding expression or declaration: `} // TargetPrefix = "riscv"`.
  **L2064 CN**: 继续构造周围的表达式或声明：`} // TargetPrefix = "riscv"`。
- **L2065 EN**: Blank line separating nearby declarations or logic blocks.
  **L2065 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2066 EN**: Comment explains nearby logic, invariants, or intent: `Vendor extensions`.
  **L2066 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vendor extensions`。
- **L2067 EN**: Banner comment marking a file or section boundary.
  **L2067 CN**: 横幅注释，用于标记文件或章节边界。
- **L2068 EN**: Imports TableGen file "llvm/IR/IntrinsicsRISCVXTHead.td" to reuse LLVM IR core types, instructions, and ownership utilities.
  **L2068 CN**: 导入 TableGen 文件 "llvm/IR/IntrinsicsRISCVXTHead.td" 以复用LLVM IR 核心类型、指令与所有权辅助组件。
- **L2069 EN**: Imports TableGen file "llvm/IR/IntrinsicsRISCVXsf.td" to reuse LLVM IR core types, instructions, and ownership utilities.
  **L2069 CN**: 导入 TableGen 文件 "llvm/IR/IntrinsicsRISCVXsf.td" 以复用LLVM IR 核心类型、指令与所有权辅助组件。
- **L2070 EN**: Imports TableGen file "llvm/IR/IntrinsicsRISCVXCV.td" to reuse LLVM IR core types, instructions, and ownership utilities.
  **L2070 CN**: 导入 TableGen 文件 "llvm/IR/IntrinsicsRISCVXCV.td" 以复用LLVM IR 核心类型、指令与所有权辅助组件。
- **L2071 EN**: Imports TableGen file "llvm/IR/IntrinsicsRISCVXAndes.td" to reuse LLVM IR core types, instructions, and ownership utilities.
  **L2071 CN**: 导入 TableGen 文件 "llvm/IR/IntrinsicsRISCVXAndes.td" 以复用LLVM IR 核心类型、指令与所有权辅助组件。
- **L2072 EN**: Imports TableGen file "llvm/IR/IntrinsicsRISCVXMIPS.td" to reuse LLVM IR core types, instructions, and ownership utilities.
  **L2072 CN**: 导入 TableGen 文件 "llvm/IR/IntrinsicsRISCVXMIPS.td" 以复用LLVM IR 核心类型、指令与所有权辅助组件。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **ORC JIT runtime and remote execution / ORC JIT 运行时与远程执行**
- **Instruction semantics / 指令语义**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Target intrinsic catalogs / 目标 Intrinsic 目录**

## Dependencies / 依赖关系

- `llvm/IR/IntrinsicsRISCVXTHead.td`: Provides LLVM IR core types, instructions, and ownership utilities. / 提供LLVM IR 核心类型、指令与所有权辅助组件。
- `llvm/IR/IntrinsicsRISCVXsf.td`: Provides LLVM IR core types, instructions, and ownership utilities. / 提供LLVM IR 核心类型、指令与所有权辅助组件。
- `llvm/IR/IntrinsicsRISCVXCV.td`: Provides LLVM IR core types, instructions, and ownership utilities. / 提供LLVM IR 核心类型、指令与所有权辅助组件。
- `llvm/IR/IntrinsicsRISCVXAndes.td`: Provides LLVM IR core types, instructions, and ownership utilities. / 提供LLVM IR 核心类型、指令与所有权辅助组件。
- `llvm/IR/IntrinsicsRISCVXMIPS.td`: Provides LLVM IR core types, instructions, and ownership utilities. / 提供LLVM IR 核心类型、指令与所有权辅助组件。
