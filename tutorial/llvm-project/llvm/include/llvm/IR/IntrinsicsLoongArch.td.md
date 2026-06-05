# IntrinsicsLoongArch.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/IntrinsicsLoongArch.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file defines all of the LoongArch-specific intrinsics.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/IR`，主要定义与 `IntrinsicsLoongArch` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````tablegen
//===- IntrinsicsLoongArch.td - Defines LoongArch intrinsics *- tablegen -*===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines all of the LoongArch-specific intrinsics.
//
//===----------------------------------------------------------------------===//

let TargetPrefix = "loongarch" in {

//===----------------------------------------------------------------------===//
// Atomics

// T @llvm.<name>.T.<p>(any*, T, T, T imm);
class MaskedAtomicRMW<LLVMType itype>
    : Intrinsic<[itype], [llvm_anyptr_ty, itype, itype, itype],
                [IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<3>>]>;
// T @llvm.<name>.T.<p>(any*, T, T, T, T imm);
class LoongArchMaskedAtomicRMWFiveArg<LLVMType itype>
    : Intrinsic<[itype], [llvm_anyptr_ty, itype, itype, itype, itype],
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
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This file defines all of the LoongArch-specific intrinsics.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This file defines all of the LoongArch-specific intrinsics.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L13 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L15 EN**: Banner comment marking a file or section boundary.
  **L15 CN**: 横幅注释，用于标记文件或章节边界。
- **L16 EN**: Comment explains nearby logic, invariants, or intent: `Atomics`.
  **L16 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Atomics`。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L18 EN**: Comment explains nearby logic, invariants, or intent: `T @llvm.<name>.T.<p>(any*, T, T, T imm);`.
  **L18 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`T @llvm.<name>.T.<p>(any*, T, T, T imm);`。
- **L19 EN**: Declares class `MaskedAtomicRMW<LLVMType`.
  **L19 CN**: 声明 class `MaskedAtomicRMW<LLVMType`。
- **L20 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[itype], [llvm_anyptr_ty, itype, itype, itype],`.
  **L20 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[itype], [llvm_anyptr_ty, itype, itype, itype],`。
- **L21 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<3>>]>;`.
  **L21 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<3>>]>;`。
- **L22 EN**: Comment explains nearby logic, invariants, or intent: `T @llvm.<name>.T.<p>(any*, T, T, T, T imm);`.
  **L22 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`T @llvm.<name>.T.<p>(any*, T, T, T, T imm);`。
- **L23 EN**: Declares class `LoongArchMaskedAtomicRMWFiveArg<LLVMType`.
  **L23 CN**: 声明 class `LoongArchMaskedAtomicRMWFiveArg<LLVMType`。
- **L24 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[itype], [llvm_anyptr_ty, itype, itype, itype, itype],`.
  **L24 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[itype], [llvm_anyptr_ty, itype, itype, itype, itype],`。

### Lines 25-48

````tablegen
                [IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<4>>]>;

// We define 32-bit and 64-bit variants of the above, where T stands for i32
// or i64 respectively:
multiclass MaskedAtomicRMWIntrinsics {
  // i32 @llvm.<name>.i32.<p>(any*, i32, i32, i32 imm);
  def _i32 : MaskedAtomicRMW<llvm_i32_ty>;
  // i64 @llvm.<name>.i64.<p>(any*, i64, i64, i64 imm);
  def _i64 : MaskedAtomicRMW<llvm_i64_ty>;
}

multiclass MaskedAtomicRMWFiveOpIntrinsics {
  // i32 @llvm.<name>.i32.<p>(any*, i32, i32, i32, i32 imm);
  def _i32 : LoongArchMaskedAtomicRMWFiveArg<llvm_i32_ty>;
  // i64 @llvm.<name>.i64.<p>(any*, i64, i64, i64, i64 imm);
  def _i64 : LoongArchMaskedAtomicRMWFiveArg<llvm_i64_ty>;
}

defm int_loongarch_masked_atomicrmw_xchg : MaskedAtomicRMWIntrinsics;
defm int_loongarch_masked_atomicrmw_add : MaskedAtomicRMWIntrinsics;
defm int_loongarch_masked_atomicrmw_sub : MaskedAtomicRMWIntrinsics;
defm int_loongarch_masked_atomicrmw_nand : MaskedAtomicRMWIntrinsics;
defm int_loongarch_masked_atomicrmw_umax : MaskedAtomicRMWIntrinsics;
defm int_loongarch_masked_atomicrmw_umin : MaskedAtomicRMWIntrinsics;
````
- **L25 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<4>>]>;`.
  **L25 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<4>>]>;`。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Comment explains nearby logic, invariants, or intent: `We define 32-bit and 64-bit variants of the above, where T stands for i32`.
  **L27 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`We define 32-bit and 64-bit variants of the above, where T stands for i32`。
- **L28 EN**: Comment explains nearby logic, invariants, or intent: `or i64 respectively:`.
  **L28 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`or i64 respectively:`。
- **L29 EN**: Declares TableGen multiclass `MaskedAtomicRMWIntrinsics`.
  **L29 CN**: 声明 TableGen multiclass `MaskedAtomicRMWIntrinsics`。
- **L30 EN**: Comment explains nearby logic, invariants, or intent: `i32 @llvm.<name>.i32.<p>(any*, i32, i32, i32 imm);`.
  **L30 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i32 @llvm.<name>.i32.<p>(any*, i32, i32, i32 imm);`。
- **L31 EN**: Declares TableGen def `_i32`.
  **L31 CN**: 声明 TableGen def `_i32`。
- **L32 EN**: Comment explains nearby logic, invariants, or intent: `i64 @llvm.<name>.i64.<p>(any*, i64, i64, i64 imm);`.
  **L32 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i64 @llvm.<name>.i64.<p>(any*, i64, i64, i64 imm);`。
- **L33 EN**: Declares TableGen def `_i64`.
  **L33 CN**: 声明 TableGen def `_i64`。
- **L34 EN**: Closes the current lexical scope or compound statement.
  **L34 CN**: 结束当前词法作用域或复合语句块。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L36 EN**: Declares TableGen multiclass `MaskedAtomicRMWFiveOpIntrinsics`.
  **L36 CN**: 声明 TableGen multiclass `MaskedAtomicRMWFiveOpIntrinsics`。
- **L37 EN**: Comment explains nearby logic, invariants, or intent: `i32 @llvm.<name>.i32.<p>(any*, i32, i32, i32, i32 imm);`.
  **L37 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i32 @llvm.<name>.i32.<p>(any*, i32, i32, i32, i32 imm);`。
- **L38 EN**: Declares TableGen def `_i32`.
  **L38 CN**: 声明 TableGen def `_i32`。
- **L39 EN**: Comment explains nearby logic, invariants, or intent: `i64 @llvm.<name>.i64.<p>(any*, i64, i64, i64, i64 imm);`.
  **L39 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i64 @llvm.<name>.i64.<p>(any*, i64, i64, i64, i64 imm);`。
- **L40 EN**: Declares TableGen def `_i64`.
  **L40 CN**: 声明 TableGen def `_i64`。
- **L41 EN**: Closes the current lexical scope or compound statement.
  **L41 CN**: 结束当前词法作用域或复合语句块。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L43 EN**: Declares TableGen defm `int_loongarch_masked_atomicrmw_xchg`.
  **L43 CN**: 声明 TableGen defm `int_loongarch_masked_atomicrmw_xchg`。
- **L44 EN**: Declares TableGen defm `int_loongarch_masked_atomicrmw_add`.
  **L44 CN**: 声明 TableGen defm `int_loongarch_masked_atomicrmw_add`。
- **L45 EN**: Declares TableGen defm `int_loongarch_masked_atomicrmw_sub`.
  **L45 CN**: 声明 TableGen defm `int_loongarch_masked_atomicrmw_sub`。
- **L46 EN**: Declares TableGen defm `int_loongarch_masked_atomicrmw_nand`.
  **L46 CN**: 声明 TableGen defm `int_loongarch_masked_atomicrmw_nand`。
- **L47 EN**: Declares TableGen defm `int_loongarch_masked_atomicrmw_umax`.
  **L47 CN**: 声明 TableGen defm `int_loongarch_masked_atomicrmw_umax`。
- **L48 EN**: Declares TableGen defm `int_loongarch_masked_atomicrmw_umin`.
  **L48 CN**: 声明 TableGen defm `int_loongarch_masked_atomicrmw_umin`。

### Lines 49-72

````tablegen
defm int_loongarch_masked_atomicrmw_max : MaskedAtomicRMWFiveOpIntrinsics;
defm int_loongarch_masked_atomicrmw_min : MaskedAtomicRMWFiveOpIntrinsics;

// @llvm.loongarch.masked.cmpxchg.<i32,i64>.<p>(
//   ptr addr, grlen cmpval, grlen newval, grlen mask, grlenimm ordering)
defm int_loongarch_masked_cmpxchg : MaskedAtomicRMWFiveOpIntrinsics;

//===----------------------------------------------------------------------===//
// LoongArch BASE

class BaseInt<list<LLVMType> ret_types, list<LLVMType> param_types,
              list<IntrinsicProperty> intr_properties = []>
    : Intrinsic<ret_types, param_types, intr_properties>,
      ClangBuiltin<!subst("int_loongarch", "__builtin_loongarch", NAME)>;

def int_loongarch_break : BaseInt<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;
def int_loongarch_cacop_d : BaseInt<[], [llvm_i64_ty, llvm_i64_ty, llvm_i64_ty],
                                    [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>]>;
def int_loongarch_cacop_w : BaseInt<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
                                    [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>]>;
def int_loongarch_dbar : BaseInt<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;

def int_loongarch_ibar : BaseInt<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;
def int_loongarch_movfcsr2gr : BaseInt<[llvm_i32_ty], [llvm_i32_ty],
````
- **L49 EN**: Declares TableGen defm `int_loongarch_masked_atomicrmw_max`.
  **L49 CN**: 声明 TableGen defm `int_loongarch_masked_atomicrmw_max`。
- **L50 EN**: Declares TableGen defm `int_loongarch_masked_atomicrmw_min`.
  **L50 CN**: 声明 TableGen defm `int_loongarch_masked_atomicrmw_min`。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L52 EN**: Comment explains nearby logic, invariants, or intent: `@llvm.loongarch.masked.cmpxchg.<i32,i64>.<p>(`.
  **L52 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`@llvm.loongarch.masked.cmpxchg.<i32,i64>.<p>(`。
- **L53 EN**: Comment explains nearby logic, invariants, or intent: `ptr addr, grlen cmpval, grlen newval, grlen mask, grlenimm ordering)`.
  **L53 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ptr addr, grlen cmpval, grlen newval, grlen mask, grlenimm ordering)`。
- **L54 EN**: Declares TableGen defm `int_loongarch_masked_cmpxchg`.
  **L54 CN**: 声明 TableGen defm `int_loongarch_masked_cmpxchg`。
- **L55 EN**: Blank line separating nearby declarations or logic blocks.
  **L55 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L56 EN**: Banner comment marking a file or section boundary.
  **L56 CN**: 横幅注释，用于标记文件或章节边界。
- **L57 EN**: Comment explains nearby logic, invariants, or intent: `LoongArch BASE`.
  **L57 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`LoongArch BASE`。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L59 EN**: Declares class `BaseInt<list<LLVMType>`.
  **L59 CN**: 声明 class `BaseInt<list<LLVMType>`。
- **L60 EN**: Continues the surrounding expression or declaration: `list<IntrinsicProperty> intr_properties = []>`.
  **L60 CN**: 继续构造周围的表达式或声明：`list<IntrinsicProperty> intr_properties = []>`。
- **L61 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<ret_types, param_types, intr_properties>,`.
  **L61 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<ret_types, param_types, intr_properties>,`。
- **L62 EN**: Executes a call or declaration centered on `ClangBuiltin<!subst`.
  **L62 CN**: 执行以 `ClangBuiltin<!subst` 为核心的调用或声明。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L64 EN**: Declares TableGen def `int_loongarch_break`.
  **L64 CN**: 声明 TableGen def `int_loongarch_break`。
- **L65 EN**: Declares TableGen def `int_loongarch_cacop_d`.
  **L65 CN**: 声明 TableGen def `int_loongarch_cacop_d`。
- **L66 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>]>;`.
  **L66 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>]>;`。
- **L67 EN**: Declares TableGen def `int_loongarch_cacop_w`.
  **L67 CN**: 声明 TableGen def `int_loongarch_cacop_w`。
- **L68 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>]>;`.
  **L68 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>]>;`。
- **L69 EN**: Declares TableGen def `int_loongarch_dbar`.
  **L69 CN**: 声明 TableGen def `int_loongarch_dbar`。
- **L70 EN**: Blank line separating nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L71 EN**: Declares TableGen def `int_loongarch_ibar`.
  **L71 CN**: 声明 TableGen def `int_loongarch_ibar`。
- **L72 EN**: Declares TableGen def `int_loongarch_movfcsr2gr`.
  **L72 CN**: 声明 TableGen def `int_loongarch_movfcsr2gr`。

### Lines 73-96

````tablegen
                                       [ImmArg<ArgIndex<0>>]>;
def int_loongarch_movgr2fcsr : BaseInt<[], [llvm_i32_ty, llvm_i32_ty],
                                       [ImmArg<ArgIndex<0>>]>;
def int_loongarch_syscall : BaseInt<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;

def int_loongarch_crc_w_b_w : BaseInt<[llvm_i32_ty],
                                      [llvm_i32_ty, llvm_i32_ty]>;
def int_loongarch_crc_w_h_w : BaseInt<[llvm_i32_ty],
                                      [llvm_i32_ty, llvm_i32_ty]>;
def int_loongarch_crc_w_w_w : BaseInt<[llvm_i32_ty],
                                      [llvm_i32_ty, llvm_i32_ty]>;
def int_loongarch_crc_w_d_w : BaseInt<[llvm_i32_ty],
                                      [llvm_i64_ty, llvm_i32_ty]>;

def int_loongarch_crcc_w_b_w : BaseInt<[llvm_i32_ty],
                                       [llvm_i32_ty, llvm_i32_ty]>;
def int_loongarch_crcc_w_h_w : BaseInt<[llvm_i32_ty],
                                       [llvm_i32_ty, llvm_i32_ty]>;
def int_loongarch_crcc_w_w_w : BaseInt<[llvm_i32_ty],
                                       [llvm_i32_ty, llvm_i32_ty]>;
def int_loongarch_crcc_w_d_w : BaseInt<[llvm_i32_ty],
                                       [llvm_i64_ty, llvm_i32_ty]>;

def int_loongarch_csrrd_w : BaseInt<[llvm_i32_ty], [llvm_i32_ty],
````
- **L73 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>]>;`.
  **L73 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>]>;`。
- **L74 EN**: Declares TableGen def `int_loongarch_movgr2fcsr`.
  **L74 CN**: 声明 TableGen def `int_loongarch_movgr2fcsr`。
- **L75 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>]>;`.
  **L75 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>]>;`。
- **L76 EN**: Declares TableGen def `int_loongarch_syscall`.
  **L76 CN**: 声明 TableGen def `int_loongarch_syscall`。
- **L77 EN**: Blank line separating nearby declarations or logic blocks.
  **L77 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L78 EN**: Declares TableGen def `int_loongarch_crc_w_b_w`.
  **L78 CN**: 声明 TableGen def `int_loongarch_crc_w_b_w`。
- **L79 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty, llvm_i32_ty]>;`.
  **L79 CN**: 执行一条独立语句或声明：`[llvm_i32_ty, llvm_i32_ty]>;`。
- **L80 EN**: Declares TableGen def `int_loongarch_crc_w_h_w`.
  **L80 CN**: 声明 TableGen def `int_loongarch_crc_w_h_w`。
- **L81 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty, llvm_i32_ty]>;`.
  **L81 CN**: 执行一条独立语句或声明：`[llvm_i32_ty, llvm_i32_ty]>;`。
- **L82 EN**: Declares TableGen def `int_loongarch_crc_w_w_w`.
  **L82 CN**: 声明 TableGen def `int_loongarch_crc_w_w_w`。
- **L83 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty, llvm_i32_ty]>;`.
  **L83 CN**: 执行一条独立语句或声明：`[llvm_i32_ty, llvm_i32_ty]>;`。
- **L84 EN**: Declares TableGen def `int_loongarch_crc_w_d_w`.
  **L84 CN**: 声明 TableGen def `int_loongarch_crc_w_d_w`。
- **L85 EN**: Executes a standalone statement or declaration: `[llvm_i64_ty, llvm_i32_ty]>;`.
  **L85 CN**: 执行一条独立语句或声明：`[llvm_i64_ty, llvm_i32_ty]>;`。
- **L86 EN**: Blank line separating nearby declarations or logic blocks.
  **L86 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L87 EN**: Declares TableGen def `int_loongarch_crcc_w_b_w`.
  **L87 CN**: 声明 TableGen def `int_loongarch_crcc_w_b_w`。
- **L88 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty, llvm_i32_ty]>;`.
  **L88 CN**: 执行一条独立语句或声明：`[llvm_i32_ty, llvm_i32_ty]>;`。
- **L89 EN**: Declares TableGen def `int_loongarch_crcc_w_h_w`.
  **L89 CN**: 声明 TableGen def `int_loongarch_crcc_w_h_w`。
- **L90 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty, llvm_i32_ty]>;`.
  **L90 CN**: 执行一条独立语句或声明：`[llvm_i32_ty, llvm_i32_ty]>;`。
- **L91 EN**: Declares TableGen def `int_loongarch_crcc_w_w_w`.
  **L91 CN**: 声明 TableGen def `int_loongarch_crcc_w_w_w`。
- **L92 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty, llvm_i32_ty]>;`.
  **L92 CN**: 执行一条独立语句或声明：`[llvm_i32_ty, llvm_i32_ty]>;`。
- **L93 EN**: Declares TableGen def `int_loongarch_crcc_w_d_w`.
  **L93 CN**: 声明 TableGen def `int_loongarch_crcc_w_d_w`。
- **L94 EN**: Executes a standalone statement or declaration: `[llvm_i64_ty, llvm_i32_ty]>;`.
  **L94 CN**: 执行一条独立语句或声明：`[llvm_i64_ty, llvm_i32_ty]>;`。
- **L95 EN**: Blank line separating nearby declarations or logic blocks.
  **L95 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L96 EN**: Declares TableGen def `int_loongarch_csrrd_w`.
  **L96 CN**: 声明 TableGen def `int_loongarch_csrrd_w`。

### Lines 97-120

````tablegen
                                    [ImmArg<ArgIndex<0>>]>;
def int_loongarch_csrrd_d : BaseInt<[llvm_i64_ty], [llvm_i32_ty],
                                    [ImmArg<ArgIndex<0>>]>;
def int_loongarch_csrwr_w : BaseInt<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                                    [ImmArg<ArgIndex<1>>]>;
def int_loongarch_csrwr_d : BaseInt<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty],
                                    [ImmArg<ArgIndex<1>>]>;
def int_loongarch_csrxchg_w : BaseInt<[llvm_i32_ty],
                                      [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
                                      [ImmArg<ArgIndex<2>>]>;
def int_loongarch_csrxchg_d : BaseInt<[llvm_i64_ty],
                                      [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty],
                                      [ImmArg<ArgIndex<2>>]>;

def int_loongarch_iocsrrd_b : BaseInt<[llvm_i32_ty], [llvm_i32_ty]>;
def int_loongarch_iocsrrd_h : BaseInt<[llvm_i32_ty], [llvm_i32_ty]>;
def int_loongarch_iocsrrd_w : BaseInt<[llvm_i32_ty], [llvm_i32_ty]>;
def int_loongarch_iocsrrd_d : BaseInt<[llvm_i64_ty], [llvm_i32_ty]>;

def int_loongarch_iocsrwr_b : BaseInt<[], [llvm_i32_ty, llvm_i32_ty]>;
def int_loongarch_iocsrwr_h : BaseInt<[], [llvm_i32_ty, llvm_i32_ty]>;
def int_loongarch_iocsrwr_w : BaseInt<[], [llvm_i32_ty, llvm_i32_ty]>;
def int_loongarch_iocsrwr_d : BaseInt<[], [llvm_i64_ty, llvm_i32_ty]>;

````
- **L97 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>]>;`.
  **L97 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>]>;`。
- **L98 EN**: Declares TableGen def `int_loongarch_csrrd_d`.
  **L98 CN**: 声明 TableGen def `int_loongarch_csrrd_d`。
- **L99 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>]>;`.
  **L99 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>]>;`。
- **L100 EN**: Declares TableGen def `int_loongarch_csrwr_w`.
  **L100 CN**: 声明 TableGen def `int_loongarch_csrwr_w`。
- **L101 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<1>>]>;`.
  **L101 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<1>>]>;`。
- **L102 EN**: Declares TableGen def `int_loongarch_csrwr_d`.
  **L102 CN**: 声明 TableGen def `int_loongarch_csrwr_d`。
- **L103 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<1>>]>;`.
  **L103 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<1>>]>;`。
- **L104 EN**: Declares TableGen def `int_loongarch_csrxchg_w`.
  **L104 CN**: 声明 TableGen def `int_loongarch_csrxchg_w`。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L106 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<2>>]>;`.
  **L106 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<2>>]>;`。
- **L107 EN**: Declares TableGen def `int_loongarch_csrxchg_d`.
  **L107 CN**: 声明 TableGen def `int_loongarch_csrxchg_d`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i64_ty, llvm_i64_ty, llvm_i32_ty],`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i64_ty, llvm_i64_ty, llvm_i32_ty],`。
- **L109 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<2>>]>;`.
  **L109 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<2>>]>;`。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L111 EN**: Declares TableGen def `int_loongarch_iocsrrd_b`.
  **L111 CN**: 声明 TableGen def `int_loongarch_iocsrrd_b`。
- **L112 EN**: Declares TableGen def `int_loongarch_iocsrrd_h`.
  **L112 CN**: 声明 TableGen def `int_loongarch_iocsrrd_h`。
- **L113 EN**: Declares TableGen def `int_loongarch_iocsrrd_w`.
  **L113 CN**: 声明 TableGen def `int_loongarch_iocsrrd_w`。
- **L114 EN**: Declares TableGen def `int_loongarch_iocsrrd_d`.
  **L114 CN**: 声明 TableGen def `int_loongarch_iocsrrd_d`。
- **L115 EN**: Blank line separating nearby declarations or logic blocks.
  **L115 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L116 EN**: Declares TableGen def `int_loongarch_iocsrwr_b`.
  **L116 CN**: 声明 TableGen def `int_loongarch_iocsrwr_b`。
- **L117 EN**: Declares TableGen def `int_loongarch_iocsrwr_h`.
  **L117 CN**: 声明 TableGen def `int_loongarch_iocsrwr_h`。
- **L118 EN**: Declares TableGen def `int_loongarch_iocsrwr_w`.
  **L118 CN**: 声明 TableGen def `int_loongarch_iocsrwr_w`。
- **L119 EN**: Declares TableGen def `int_loongarch_iocsrwr_d`.
  **L119 CN**: 声明 TableGen def `int_loongarch_iocsrwr_d`。
- **L120 EN**: Blank line separating nearby declarations or logic blocks.
  **L120 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 121-144

````tablegen
def int_loongarch_cpucfg : BaseInt<[llvm_i32_ty], [llvm_i32_ty]>;

def int_loongarch_asrtle_d : BaseInt<[], [llvm_i64_ty, llvm_i64_ty]>;
def int_loongarch_asrtgt_d : BaseInt<[], [llvm_i64_ty, llvm_i64_ty]>;

def int_loongarch_lddir_d : BaseInt<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
                                    [ImmArg<ArgIndex<1>>]>;
def int_loongarch_ldpte_d : BaseInt<[], [llvm_i64_ty, llvm_i64_ty],
                                    [ImmArg<ArgIndex<1>>]>;

def int_loongarch_frecipe_s : BaseInt<[llvm_float_ty], [llvm_float_ty],
                                      [IntrNoMem]>;
def int_loongarch_frecipe_d : BaseInt<[llvm_double_ty], [llvm_double_ty],
                                      [IntrNoMem]>;
def int_loongarch_frsqrte_s : BaseInt<[llvm_float_ty], [llvm_float_ty],
                                      [IntrNoMem]>;
def int_loongarch_frsqrte_d : BaseInt<[llvm_double_ty], [llvm_double_ty],
                                      [IntrNoMem]>;
} // TargetPrefix = "loongarch"

/// Vector intrinsic

class VecInt<list<LLVMType> ret_types, list<LLVMType> param_types,
             list<IntrinsicProperty> intr_properties = []>
````
- **L121 EN**: Declares TableGen def `int_loongarch_cpucfg`.
  **L121 CN**: 声明 TableGen def `int_loongarch_cpucfg`。
- **L122 EN**: Blank line separating nearby declarations or logic blocks.
  **L122 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L123 EN**: Declares TableGen def `int_loongarch_asrtle_d`.
  **L123 CN**: 声明 TableGen def `int_loongarch_asrtle_d`。
- **L124 EN**: Declares TableGen def `int_loongarch_asrtgt_d`.
  **L124 CN**: 声明 TableGen def `int_loongarch_asrtgt_d`。
- **L125 EN**: Blank line separating nearby declarations or logic blocks.
  **L125 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L126 EN**: Declares TableGen def `int_loongarch_lddir_d`.
  **L126 CN**: 声明 TableGen def `int_loongarch_lddir_d`。
- **L127 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<1>>]>;`.
  **L127 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<1>>]>;`。
- **L128 EN**: Declares TableGen def `int_loongarch_ldpte_d`.
  **L128 CN**: 声明 TableGen def `int_loongarch_ldpte_d`。
- **L129 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<1>>]>;`.
  **L129 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<1>>]>;`。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L131 EN**: Declares TableGen def `int_loongarch_frecipe_s`.
  **L131 CN**: 声明 TableGen def `int_loongarch_frecipe_s`。
- **L132 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L132 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L133 EN**: Declares TableGen def `int_loongarch_frecipe_d`.
  **L133 CN**: 声明 TableGen def `int_loongarch_frecipe_d`。
- **L134 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L134 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L135 EN**: Declares TableGen def `int_loongarch_frsqrte_s`.
  **L135 CN**: 声明 TableGen def `int_loongarch_frsqrte_s`。
- **L136 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L136 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L137 EN**: Declares TableGen def `int_loongarch_frsqrte_d`.
  **L137 CN**: 声明 TableGen def `int_loongarch_frsqrte_d`。
- **L138 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L138 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L139 EN**: Continues the surrounding expression or declaration: `} // TargetPrefix = "loongarch"`.
  **L139 CN**: 继续构造周围的表达式或声明：`} // TargetPrefix = "loongarch"`。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L141 EN**: Comment explains nearby logic, invariants, or intent: `Vector intrinsic`.
  **L141 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector intrinsic`。
- **L142 EN**: Blank line separating nearby declarations or logic blocks.
  **L142 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L143 EN**: Declares class `VecInt<list<LLVMType>`.
  **L143 CN**: 声明 class `VecInt<list<LLVMType>`。
- **L144 EN**: Continues the surrounding expression or declaration: `list<IntrinsicProperty> intr_properties = []>`.
  **L144 CN**: 继续构造周围的表达式或声明：`list<IntrinsicProperty> intr_properties = []>`。

### Lines 145-168

````tablegen
    : Intrinsic<ret_types, param_types, intr_properties>,
      ClangBuiltin<!subst("int_loongarch", "__builtin", NAME)>;

class DefaultAttrsVecInt<list<LLVMType> ret_types, list<LLVMType> param_types,
             list<IntrinsicProperty> intr_properties = []>
    : DefaultAttrsIntrinsic<ret_types, param_types, intr_properties>,
      ClangBuiltin<!subst("int_loongarch", "__builtin", NAME)>;

//===----------------------------------------------------------------------===//
// LSX

let TargetPrefix = "loongarch" in {

foreach inst = ["vadd_b", "vsub_b",
                "vsadd_b", "vsadd_bu", "vssub_b", "vssub_bu",
                "vavg_b", "vavg_bu", "vavgr_b", "vavgr_bu",
                "vabsd_b", "vabsd_bu", "vadda_b",
                "vmax_b", "vmax_bu", "vmin_b", "vmin_bu",
                "vmul_b", "vmuh_b", "vmuh_bu",
                "vdiv_b", "vdiv_bu", "vmod_b", "vmod_bu", "vsigncov_b",
                "vand_v", "vor_v", "vxor_v", "vnor_v", "vandn_v", "vorn_v",
                "vsll_b", "vsrl_b", "vsra_b", "vrotr_b", "vsrlr_b", "vsrar_b",
                "vbitclr_b", "vbitset_b", "vbitrev_b",
                "vseq_b", "vsle_b", "vsle_bu", "vslt_b", "vslt_bu",
````
- **L145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<ret_types, param_types, intr_properties>,`.
  **L145 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<ret_types, param_types, intr_properties>,`。
- **L146 EN**: Executes a call or declaration centered on `ClangBuiltin<!subst`.
  **L146 CN**: 执行以 `ClangBuiltin<!subst` 为核心的调用或声明。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L148 EN**: Declares class `DefaultAttrsVecInt<list<LLVMType>`.
  **L148 CN**: 声明 class `DefaultAttrsVecInt<list<LLVMType>`。
- **L149 EN**: Continues the surrounding expression or declaration: `list<IntrinsicProperty> intr_properties = []>`.
  **L149 CN**: 继续构造周围的表达式或声明：`list<IntrinsicProperty> intr_properties = []>`。
- **L150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<ret_types, param_types, intr_properties>,`.
  **L150 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<ret_types, param_types, intr_properties>,`。
- **L151 EN**: Executes a call or declaration centered on `ClangBuiltin<!subst`.
  **L151 CN**: 执行以 `ClangBuiltin<!subst` 为核心的调用或声明。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L153 EN**: Banner comment marking a file or section boundary.
  **L153 CN**: 横幅注释，用于标记文件或章节边界。
- **L154 EN**: Comment explains nearby logic, invariants, or intent: `LSX`.
  **L154 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`LSX`。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L156 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L156 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L157 EN**: Blank line separating nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L158 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L158 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsadd_b", "vsadd_bu", "vssub_b", "vssub_bu",`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsadd_b", "vsadd_bu", "vssub_b", "vssub_bu",`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vavg_b", "vavg_bu", "vavgr_b", "vavgr_bu",`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vavg_b", "vavg_bu", "vavgr_b", "vavgr_bu",`。
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vabsd_b", "vabsd_bu", "vadda_b",`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vabsd_b", "vabsd_bu", "vadda_b",`。
- **L162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmax_b", "vmax_bu", "vmin_b", "vmin_bu",`.
  **L162 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmax_b", "vmax_bu", "vmin_b", "vmin_bu",`。
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmul_b", "vmuh_b", "vmuh_bu",`.
  **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmul_b", "vmuh_b", "vmuh_bu",`。
- **L164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vdiv_b", "vdiv_bu", "vmod_b", "vmod_bu", "vsigncov_b",`.
  **L164 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vdiv_b", "vdiv_bu", "vmod_b", "vmod_bu", "vsigncov_b",`。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vand_v", "vor_v", "vxor_v", "vnor_v", "vandn_v", "vorn_v",`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vand_v", "vor_v", "vxor_v", "vnor_v", "vandn_v", "vorn_v",`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsll_b", "vsrl_b", "vsra_b", "vrotr_b", "vsrlr_b", "vsrar_b",`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsll_b", "vsrl_b", "vsra_b", "vrotr_b", "vsrlr_b", "vsrar_b",`。
- **L167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vbitclr_b", "vbitset_b", "vbitrev_b",`.
  **L167 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vbitclr_b", "vbitset_b", "vbitrev_b",`。
- **L168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vseq_b", "vsle_b", "vsle_bu", "vslt_b", "vslt_bu",`.
  **L168 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vseq_b", "vsle_b", "vsle_bu", "vslt_b", "vslt_bu",`。

### Lines 169-192

````tablegen
                "vpackev_b", "vpackod_b", "vpickev_b", "vpickod_b",
                "vilvl_b", "vilvh_b"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v16i8_ty],
                                       [llvm_v16i8_ty, llvm_v16i8_ty],
                                       [IntrNoMem]>;

foreach inst = ["vadd_h", "vsub_h",
                "vsadd_h", "vsadd_hu", "vssub_h", "vssub_hu",
                "vavg_h", "vavg_hu", "vavgr_h", "vavgr_hu",
                "vabsd_h", "vabsd_hu", "vadda_h",
                "vmax_h", "vmax_hu", "vmin_h", "vmin_hu",
                "vmul_h", "vmuh_h", "vmuh_hu",
                "vdiv_h", "vdiv_hu", "vmod_h", "vmod_hu", "vsigncov_h",
                "vsll_h", "vsrl_h", "vsra_h", "vrotr_h", "vsrlr_h", "vsrar_h",
                "vbitclr_h", "vbitset_h", "vbitrev_h",
                "vseq_h", "vsle_h", "vsle_hu", "vslt_h", "vslt_hu",
                "vpackev_h", "vpackod_h", "vpickev_h", "vpickod_h",
                "vilvl_h", "vilvh_h"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v8i16_ty],
                                       [llvm_v8i16_ty, llvm_v8i16_ty],
                                       [IntrNoMem]>;

foreach inst = ["vadd_w", "vsub_w",
                "vsadd_w", "vsadd_wu", "vssub_w", "vssub_wu",
````
- **L169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vpackev_b", "vpackod_b", "vpickev_b", "vpickod_b",`.
  **L169 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vpackev_b", "vpackod_b", "vpickev_b", "vpickod_b",`。
- **L170 EN**: Continues the surrounding expression or declaration: `"vilvl_b", "vilvh_b"] in`.
  **L170 CN**: 继续构造周围的表达式或声明：`"vilvl_b", "vilvh_b"] in`。
- **L171 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L171 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L173 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L173 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L175 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L175 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsadd_h", "vsadd_hu", "vssub_h", "vssub_hu",`.
  **L176 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsadd_h", "vsadd_hu", "vssub_h", "vssub_hu",`。
- **L177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vavg_h", "vavg_hu", "vavgr_h", "vavgr_hu",`.
  **L177 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vavg_h", "vavg_hu", "vavgr_h", "vavgr_hu",`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vabsd_h", "vabsd_hu", "vadda_h",`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vabsd_h", "vabsd_hu", "vadda_h",`。
- **L179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmax_h", "vmax_hu", "vmin_h", "vmin_hu",`.
  **L179 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmax_h", "vmax_hu", "vmin_h", "vmin_hu",`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmul_h", "vmuh_h", "vmuh_hu",`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmul_h", "vmuh_h", "vmuh_hu",`。
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vdiv_h", "vdiv_hu", "vmod_h", "vmod_hu", "vsigncov_h",`.
  **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vdiv_h", "vdiv_hu", "vmod_h", "vmod_hu", "vsigncov_h",`。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsll_h", "vsrl_h", "vsra_h", "vrotr_h", "vsrlr_h", "vsrar_h",`.
  **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsll_h", "vsrl_h", "vsra_h", "vrotr_h", "vsrlr_h", "vsrar_h",`。
- **L183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vbitclr_h", "vbitset_h", "vbitrev_h",`.
  **L183 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vbitclr_h", "vbitset_h", "vbitrev_h",`。
- **L184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vseq_h", "vsle_h", "vsle_hu", "vslt_h", "vslt_hu",`.
  **L184 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vseq_h", "vsle_h", "vsle_hu", "vslt_h", "vslt_hu",`。
- **L185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vpackev_h", "vpackod_h", "vpickev_h", "vpickod_h",`.
  **L185 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vpackev_h", "vpackod_h", "vpickev_h", "vpickod_h",`。
- **L186 EN**: Continues the surrounding expression or declaration: `"vilvl_h", "vilvh_h"] in`.
  **L186 CN**: 继续构造周围的表达式或声明：`"vilvl_h", "vilvh_h"] in`。
- **L187 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L187 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L188 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L189 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L189 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L190 EN**: Blank line separating nearby declarations or logic blocks.
  **L190 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L191 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L191 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsadd_w", "vsadd_wu", "vssub_w", "vssub_wu",`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsadd_w", "vsadd_wu", "vssub_w", "vssub_wu",`。

### Lines 193-216

````tablegen
                "vavg_w", "vavg_wu", "vavgr_w", "vavgr_wu",
                "vabsd_w", "vabsd_wu", "vadda_w",
                "vmax_w", "vmax_wu", "vmin_w", "vmin_wu",
                "vmul_w", "vmuh_w", "vmuh_wu",
                "vdiv_w", "vdiv_wu", "vmod_w", "vmod_wu", "vsigncov_w",
                "vsll_w", "vsrl_w", "vsra_w", "vrotr_w", "vsrlr_w", "vsrar_w",
                "vbitclr_w", "vbitset_w", "vbitrev_w",
                "vseq_w", "vsle_w", "vsle_wu", "vslt_w", "vslt_wu",
                "vpackev_w", "vpackod_w", "vpickev_w", "vpickod_w",
                "vilvl_w", "vilvh_w"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4i32_ty],
                                       [llvm_v4i32_ty, llvm_v4i32_ty],
                                       [IntrNoMem]>;

foreach inst = ["vadd_d", "vadd_q", "vsub_d", "vsub_q",
                "vsadd_d", "vsadd_du", "vssub_d", "vssub_du",
                "vhaddw_q_d", "vhaddw_qu_du", "vhsubw_q_d", "vhsubw_qu_du",
                "vaddwev_q_d", "vaddwod_q_d", "vsubwev_q_d", "vsubwod_q_d",
                "vaddwev_q_du", "vaddwod_q_du", "vsubwev_q_du", "vsubwod_q_du",
                "vaddwev_q_du_d", "vaddwod_q_du_d",
                "vavg_d", "vavg_du", "vavgr_d", "vavgr_du",
                "vabsd_d", "vabsd_du", "vadda_d",
                "vmax_d", "vmax_du", "vmin_d", "vmin_du",
                "vmul_d", "vmuh_d", "vmuh_du",
````
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vavg_w", "vavg_wu", "vavgr_w", "vavgr_wu",`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vavg_w", "vavg_wu", "vavgr_w", "vavgr_wu",`。
- **L194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vabsd_w", "vabsd_wu", "vadda_w",`.
  **L194 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vabsd_w", "vabsd_wu", "vadda_w",`。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmax_w", "vmax_wu", "vmin_w", "vmin_wu",`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmax_w", "vmax_wu", "vmin_w", "vmin_wu",`。
- **L196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmul_w", "vmuh_w", "vmuh_wu",`.
  **L196 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmul_w", "vmuh_w", "vmuh_wu",`。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vdiv_w", "vdiv_wu", "vmod_w", "vmod_wu", "vsigncov_w",`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vdiv_w", "vdiv_wu", "vmod_w", "vmod_wu", "vsigncov_w",`。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsll_w", "vsrl_w", "vsra_w", "vrotr_w", "vsrlr_w", "vsrar_w",`.
  **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsll_w", "vsrl_w", "vsra_w", "vrotr_w", "vsrlr_w", "vsrar_w",`。
- **L199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vbitclr_w", "vbitset_w", "vbitrev_w",`.
  **L199 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vbitclr_w", "vbitset_w", "vbitrev_w",`。
- **L200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vseq_w", "vsle_w", "vsle_wu", "vslt_w", "vslt_wu",`.
  **L200 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vseq_w", "vsle_w", "vsle_wu", "vslt_w", "vslt_wu",`。
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vpackev_w", "vpackod_w", "vpickev_w", "vpickod_w",`.
  **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vpackev_w", "vpackod_w", "vpickev_w", "vpickod_w",`。
- **L202 EN**: Continues the surrounding expression or declaration: `"vilvl_w", "vilvh_w"] in`.
  **L202 CN**: 继续构造周围的表达式或声明：`"vilvl_w", "vilvh_w"] in`。
- **L203 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L203 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L204 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L205 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L205 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L207 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L207 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsadd_d", "vsadd_du", "vssub_d", "vssub_du",`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsadd_d", "vsadd_du", "vssub_d", "vssub_du",`。
- **L209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vhaddw_q_d", "vhaddw_qu_du", "vhsubw_q_d", "vhsubw_qu_du",`.
  **L209 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vhaddw_q_d", "vhaddw_qu_du", "vhsubw_q_d", "vhsubw_qu_du",`。
- **L210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vaddwev_q_d", "vaddwod_q_d", "vsubwev_q_d", "vsubwod_q_d",`.
  **L210 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vaddwev_q_d", "vaddwod_q_d", "vsubwev_q_d", "vsubwod_q_d",`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vaddwev_q_du", "vaddwod_q_du", "vsubwev_q_du", "vsubwod_q_du",`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vaddwev_q_du", "vaddwod_q_du", "vsubwev_q_du", "vsubwod_q_du",`。
- **L212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vaddwev_q_du_d", "vaddwod_q_du_d",`.
  **L212 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vaddwev_q_du_d", "vaddwod_q_du_d",`。
- **L213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vavg_d", "vavg_du", "vavgr_d", "vavgr_du",`.
  **L213 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vavg_d", "vavg_du", "vavgr_d", "vavgr_du",`。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vabsd_d", "vabsd_du", "vadda_d",`.
  **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vabsd_d", "vabsd_du", "vadda_d",`。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmax_d", "vmax_du", "vmin_d", "vmin_du",`.
  **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmax_d", "vmax_du", "vmin_d", "vmin_du",`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmul_d", "vmuh_d", "vmuh_du",`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmul_d", "vmuh_d", "vmuh_du",`。

### Lines 217-240

````tablegen
                "vmulwev_q_d", "vmulwod_q_d", "vmulwev_q_du", "vmulwod_q_du",
                "vmulwev_q_du_d", "vmulwod_q_du_d",
                "vdiv_d", "vdiv_du", "vmod_d", "vmod_du", "vsigncov_d",
                "vsll_d", "vsrl_d", "vsra_d", "vrotr_d", "vsrlr_d", "vsrar_d",
                "vbitclr_d", "vbitset_d", "vbitrev_d",
                "vseq_d", "vsle_d", "vsle_du", "vslt_d", "vslt_du",
                "vpackev_d", "vpackod_d", "vpickev_d", "vpickod_d",
                "vilvl_d", "vilvh_d"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2i64_ty],
                                       [llvm_v2i64_ty, llvm_v2i64_ty],
                                       [IntrNoMem]>;

foreach inst = ["vaddi_bu", "vsubi_bu",
                "vmaxi_b", "vmaxi_bu", "vmini_b", "vmini_bu",
                "vsat_b", "vsat_bu",
                "vandi_b", "vori_b", "vxori_b", "vnori_b",
                "vslli_b", "vsrli_b", "vsrai_b", "vrotri_b",
                "vsrlri_b", "vsrari_b",
                "vbitclri_b", "vbitseti_b", "vbitrevi_b",
                "vseqi_b", "vslei_b", "vslei_bu", "vslti_b", "vslti_bu",
                "vreplvei_b", "vbsll_v", "vbsrl_v", "vshuf4i_b"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v16i8_ty],
                                       [llvm_v16i8_ty, llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<1>>]>;
````
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmulwev_q_d", "vmulwod_q_d", "vmulwev_q_du", "vmulwod_q_du",`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmulwev_q_d", "vmulwod_q_d", "vmulwev_q_du", "vmulwod_q_du",`。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmulwev_q_du_d", "vmulwod_q_du_d",`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmulwev_q_du_d", "vmulwod_q_du_d",`。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vdiv_d", "vdiv_du", "vmod_d", "vmod_du", "vsigncov_d",`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vdiv_d", "vdiv_du", "vmod_d", "vmod_du", "vsigncov_d",`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsll_d", "vsrl_d", "vsra_d", "vrotr_d", "vsrlr_d", "vsrar_d",`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsll_d", "vsrl_d", "vsra_d", "vrotr_d", "vsrlr_d", "vsrar_d",`。
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vbitclr_d", "vbitset_d", "vbitrev_d",`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vbitclr_d", "vbitset_d", "vbitrev_d",`。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vseq_d", "vsle_d", "vsle_du", "vslt_d", "vslt_du",`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vseq_d", "vsle_d", "vsle_du", "vslt_d", "vslt_du",`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vpackev_d", "vpackod_d", "vpickev_d", "vpickod_d",`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vpackev_d", "vpackod_d", "vpickev_d", "vpickod_d",`。
- **L224 EN**: Continues the surrounding expression or declaration: `"vilvl_d", "vilvh_d"] in`.
  **L224 CN**: 继续构造周围的表达式或声明：`"vilvl_d", "vilvh_d"] in`。
- **L225 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L225 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L227 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L227 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L228 EN**: Blank line separating nearby declarations or logic blocks.
  **L228 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L229 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L229 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmaxi_b", "vmaxi_bu", "vmini_b", "vmini_bu",`.
  **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmaxi_b", "vmaxi_bu", "vmini_b", "vmini_bu",`。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsat_b", "vsat_bu",`.
  **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsat_b", "vsat_bu",`。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vandi_b", "vori_b", "vxori_b", "vnori_b",`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vandi_b", "vori_b", "vxori_b", "vnori_b",`。
- **L233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vslli_b", "vsrli_b", "vsrai_b", "vrotri_b",`.
  **L233 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vslli_b", "vsrli_b", "vsrai_b", "vrotri_b",`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsrlri_b", "vsrari_b",`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsrlri_b", "vsrari_b",`。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vbitclri_b", "vbitseti_b", "vbitrevi_b",`.
  **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vbitclri_b", "vbitseti_b", "vbitrevi_b",`。
- **L236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vseqi_b", "vslei_b", "vslei_bu", "vslti_b", "vslti_bu",`.
  **L236 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vseqi_b", "vslei_b", "vslei_bu", "vslti_b", "vslti_bu",`。
- **L237 EN**: Continues the surrounding expression or declaration: `"vreplvei_b", "vbsll_v", "vbsrl_v", "vshuf4i_b"] in`.
  **L237 CN**: 继续构造周围的表达式或声明：`"vreplvei_b", "vbsll_v", "vbsrl_v", "vshuf4i_b"] in`。
- **L238 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L238 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_i32_ty],`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_i32_ty],`。
- **L240 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L240 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。

### Lines 241-264

````tablegen
foreach inst = ["vaddi_hu", "vsubi_hu",
                "vmaxi_h", "vmaxi_hu", "vmini_h", "vmini_hu",
                "vsat_h", "vsat_hu",
                "vslli_h", "vsrli_h", "vsrai_h", "vrotri_h",
                "vsrlri_h", "vsrari_h",
                "vbitclri_h", "vbitseti_h", "vbitrevi_h",
                "vseqi_h", "vslei_h", "vslei_hu", "vslti_h", "vslti_hu",
                "vreplvei_h", "vshuf4i_h"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v8i16_ty],
                                       [llvm_v8i16_ty, llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<1>>]>;
foreach inst = ["vaddi_wu", "vsubi_wu",
                "vmaxi_w", "vmaxi_wu", "vmini_w", "vmini_wu",
                "vsat_w", "vsat_wu",
                "vslli_w", "vsrli_w", "vsrai_w", "vrotri_w",
                "vsrlri_w", "vsrari_w",
                "vbitclri_w", "vbitseti_w", "vbitrevi_w",
                "vseqi_w", "vslei_w", "vslei_wu", "vslti_w", "vslti_wu",
                "vreplvei_w", "vshuf4i_w"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4i32_ty],
                                       [llvm_v4i32_ty, llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<1>>]>;
foreach inst = ["vaddi_du", "vsubi_du",
                "vmaxi_d", "vmaxi_du", "vmini_d", "vmini_du",
````
- **L241 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L241 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmaxi_h", "vmaxi_hu", "vmini_h", "vmini_hu",`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmaxi_h", "vmaxi_hu", "vmini_h", "vmini_hu",`。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsat_h", "vsat_hu",`.
  **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsat_h", "vsat_hu",`。
- **L244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vslli_h", "vsrli_h", "vsrai_h", "vrotri_h",`.
  **L244 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vslli_h", "vsrli_h", "vsrai_h", "vrotri_h",`。
- **L245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsrlri_h", "vsrari_h",`.
  **L245 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsrlri_h", "vsrari_h",`。
- **L246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vbitclri_h", "vbitseti_h", "vbitrevi_h",`.
  **L246 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vbitclri_h", "vbitseti_h", "vbitrevi_h",`。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vseqi_h", "vslei_h", "vslei_hu", "vslti_h", "vslti_hu",`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vseqi_h", "vslei_h", "vslei_hu", "vslti_h", "vslti_hu",`。
- **L248 EN**: Continues the surrounding expression or declaration: `"vreplvei_h", "vshuf4i_h"] in`.
  **L248 CN**: 继续构造周围的表达式或声明：`"vreplvei_h", "vshuf4i_h"] in`。
- **L249 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L249 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_i32_ty],`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_i32_ty],`。
- **L251 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L251 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L252 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L252 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmaxi_w", "vmaxi_wu", "vmini_w", "vmini_wu",`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmaxi_w", "vmaxi_wu", "vmini_w", "vmini_wu",`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsat_w", "vsat_wu",`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsat_w", "vsat_wu",`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vslli_w", "vsrli_w", "vsrai_w", "vrotri_w",`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vslli_w", "vsrli_w", "vsrai_w", "vrotri_w",`。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsrlri_w", "vsrari_w",`.
  **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsrlri_w", "vsrari_w",`。
- **L257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vbitclri_w", "vbitseti_w", "vbitrevi_w",`.
  **L257 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vbitclri_w", "vbitseti_w", "vbitrevi_w",`。
- **L258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vseqi_w", "vslei_w", "vslei_wu", "vslti_w", "vslti_wu",`.
  **L258 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vseqi_w", "vslei_w", "vslei_wu", "vslti_w", "vslti_wu",`。
- **L259 EN**: Continues the surrounding expression or declaration: `"vreplvei_w", "vshuf4i_w"] in`.
  **L259 CN**: 继续构造周围的表达式或声明：`"vreplvei_w", "vshuf4i_w"] in`。
- **L260 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L260 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_i32_ty],`.
  **L261 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_i32_ty],`。
- **L262 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L262 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L263 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L263 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmaxi_d", "vmaxi_du", "vmini_d", "vmini_du",`.
  **L264 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmaxi_d", "vmaxi_du", "vmini_d", "vmini_du",`。

### Lines 265-288

````tablegen
                "vsat_d", "vsat_du",
                "vslli_d", "vsrli_d", "vsrai_d", "vrotri_d",
                "vsrlri_d", "vsrari_d",
                "vbitclri_d", "vbitseti_d", "vbitrevi_d",
                "vseqi_d", "vslei_d", "vslei_du", "vslti_d", "vslti_du",
                "vreplvei_d"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2i64_ty],
                                       [llvm_v2i64_ty, llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<1>>]>;

foreach inst = ["vhaddw_h_b", "vhaddw_hu_bu", "vhsubw_h_b", "vhsubw_hu_bu",
                "vaddwev_h_b", "vaddwod_h_b", "vsubwev_h_b", "vsubwod_h_b",
                "vaddwev_h_bu", "vaddwod_h_bu", "vsubwev_h_bu", "vsubwod_h_bu",
                "vaddwev_h_bu_b", "vaddwod_h_bu_b",
                "vmulwev_h_b", "vmulwod_h_b", "vmulwev_h_bu", "vmulwod_h_bu",
                "vmulwev_h_bu_b", "vmulwod_h_bu_b"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v8i16_ty],
                                       [llvm_v16i8_ty, llvm_v16i8_ty],
                                       [IntrNoMem]>;

foreach inst = ["vhaddw_w_h", "vhaddw_wu_hu", "vhsubw_w_h", "vhsubw_wu_hu",
                "vaddwev_w_h", "vaddwod_w_h", "vsubwev_w_h", "vsubwod_w_h",
                "vaddwev_w_hu", "vaddwod_w_hu", "vsubwev_w_hu", "vsubwod_w_hu",
                "vaddwev_w_hu_h", "vaddwod_w_hu_h",
````
- **L265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsat_d", "vsat_du",`.
  **L265 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsat_d", "vsat_du",`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vslli_d", "vsrli_d", "vsrai_d", "vrotri_d",`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vslli_d", "vsrli_d", "vsrai_d", "vrotri_d",`。
- **L267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vsrlri_d", "vsrari_d",`.
  **L267 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vsrlri_d", "vsrari_d",`。
- **L268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vbitclri_d", "vbitseti_d", "vbitrevi_d",`.
  **L268 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vbitclri_d", "vbitseti_d", "vbitrevi_d",`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vseqi_d", "vslei_d", "vslei_du", "vslti_d", "vslti_du",`.
  **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vseqi_d", "vslei_d", "vslei_du", "vslti_d", "vslti_du",`。
- **L270 EN**: Continues the surrounding expression or declaration: `"vreplvei_d"] in`.
  **L270 CN**: 继续构造周围的表达式或声明：`"vreplvei_d"] in`。
- **L271 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L271 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_i32_ty],`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_i32_ty],`。
- **L273 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L273 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L274 EN**: Blank line separating nearby declarations or logic blocks.
  **L274 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L275 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L275 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vaddwev_h_b", "vaddwod_h_b", "vsubwev_h_b", "vsubwod_h_b",`.
  **L276 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vaddwev_h_b", "vaddwod_h_b", "vsubwev_h_b", "vsubwod_h_b",`。
- **L277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vaddwev_h_bu", "vaddwod_h_bu", "vsubwev_h_bu", "vsubwod_h_bu",`.
  **L277 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vaddwev_h_bu", "vaddwod_h_bu", "vsubwev_h_bu", "vsubwod_h_bu",`。
- **L278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vaddwev_h_bu_b", "vaddwod_h_bu_b",`.
  **L278 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vaddwev_h_bu_b", "vaddwod_h_bu_b",`。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmulwev_h_b", "vmulwod_h_b", "vmulwev_h_bu", "vmulwod_h_bu",`.
  **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmulwev_h_b", "vmulwod_h_b", "vmulwev_h_bu", "vmulwod_h_bu",`。
- **L280 EN**: Continues the surrounding expression or declaration: `"vmulwev_h_bu_b", "vmulwod_h_bu_b"] in`.
  **L280 CN**: 继续构造周围的表达式或声明：`"vmulwev_h_bu_b", "vmulwod_h_bu_b"] in`。
- **L281 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L281 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L282 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L283 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L283 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L284 EN**: Blank line separating nearby declarations or logic blocks.
  **L284 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L285 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L285 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vaddwev_w_h", "vaddwod_w_h", "vsubwev_w_h", "vsubwod_w_h",`.
  **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vaddwev_w_h", "vaddwod_w_h", "vsubwev_w_h", "vsubwod_w_h",`。
- **L287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vaddwev_w_hu", "vaddwod_w_hu", "vsubwev_w_hu", "vsubwod_w_hu",`.
  **L287 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vaddwev_w_hu", "vaddwod_w_hu", "vsubwev_w_hu", "vsubwod_w_hu",`。
- **L288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vaddwev_w_hu_h", "vaddwod_w_hu_h",`.
  **L288 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vaddwev_w_hu_h", "vaddwod_w_hu_h",`。

### Lines 289-312

````tablegen
                "vmulwev_w_h", "vmulwod_w_h", "vmulwev_w_hu", "vmulwod_w_hu",
                "vmulwev_w_hu_h", "vmulwod_w_hu_h"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4i32_ty],
                                       [llvm_v8i16_ty, llvm_v8i16_ty],
                                       [IntrNoMem]>;

foreach inst = ["vhaddw_d_w", "vhaddw_du_wu", "vhsubw_d_w", "vhsubw_du_wu",
                "vaddwev_d_w", "vaddwod_d_w", "vsubwev_d_w", "vsubwod_d_w",
                "vaddwev_d_wu", "vaddwod_d_wu", "vsubwev_d_wu", "vsubwod_d_wu",
                "vaddwev_d_wu_w", "vaddwod_d_wu_w",
                "vmulwev_d_w", "vmulwod_d_w", "vmulwev_d_wu", "vmulwod_d_wu",
                "vmulwev_d_wu_w", "vmulwod_d_wu_w"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2i64_ty],
                                       [llvm_v4i32_ty, llvm_v4i32_ty],
                                       [IntrNoMem]>;

foreach inst = ["vsrln_b_h", "vsran_b_h", "vsrlrn_b_h", "vsrarn_b_h",
                "vssrln_b_h", "vssran_b_h", "vssrln_bu_h", "vssran_bu_h",
                "vssrlrn_b_h", "vssrarn_b_h", "vssrlrn_bu_h", "vssrarn_bu_h"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v16i8_ty],
                                       [llvm_v8i16_ty, llvm_v8i16_ty],
                                       [IntrNoMem]>;

foreach inst = ["vsrln_h_w", "vsran_h_w", "vsrlrn_h_w", "vsrarn_h_w",
````
- **L289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmulwev_w_h", "vmulwod_w_h", "vmulwev_w_hu", "vmulwod_w_hu",`.
  **L289 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmulwev_w_h", "vmulwod_w_h", "vmulwev_w_hu", "vmulwod_w_hu",`。
- **L290 EN**: Continues the surrounding expression or declaration: `"vmulwev_w_hu_h", "vmulwod_w_hu_h"] in`.
  **L290 CN**: 继续构造周围的表达式或声明：`"vmulwev_w_hu_h", "vmulwod_w_hu_h"] in`。
- **L291 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L291 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L293 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L293 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L295 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L295 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vaddwev_d_w", "vaddwod_d_w", "vsubwev_d_w", "vsubwod_d_w",`.
  **L296 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vaddwev_d_w", "vaddwod_d_w", "vsubwev_d_w", "vsubwod_d_w",`。
- **L297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vaddwev_d_wu", "vaddwod_d_wu", "vsubwev_d_wu", "vsubwod_d_wu",`.
  **L297 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vaddwev_d_wu", "vaddwod_d_wu", "vsubwev_d_wu", "vsubwod_d_wu",`。
- **L298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vaddwev_d_wu_w", "vaddwod_d_wu_w",`.
  **L298 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vaddwev_d_wu_w", "vaddwod_d_wu_w",`。
- **L299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vmulwev_d_w", "vmulwod_d_w", "vmulwev_d_wu", "vmulwod_d_wu",`.
  **L299 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vmulwev_d_w", "vmulwod_d_w", "vmulwev_d_wu", "vmulwod_d_wu",`。
- **L300 EN**: Continues the surrounding expression or declaration: `"vmulwev_d_wu_w", "vmulwod_d_wu_w"] in`.
  **L300 CN**: 继续构造周围的表达式或声明：`"vmulwev_d_wu_w", "vmulwod_d_wu_w"] in`。
- **L301 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L301 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L302 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L303 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L303 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L304 EN**: Blank line separating nearby declarations or logic blocks.
  **L304 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L305 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L305 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vssrln_b_h", "vssran_b_h", "vssrln_bu_h", "vssran_bu_h",`.
  **L306 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vssrln_b_h", "vssran_b_h", "vssrln_bu_h", "vssran_bu_h",`。
- **L307 EN**: Continues the surrounding expression or declaration: `"vssrlrn_b_h", "vssrarn_b_h", "vssrlrn_bu_h", "vssrarn_bu_h"] in`.
  **L307 CN**: 继续构造周围的表达式或声明：`"vssrlrn_b_h", "vssrarn_b_h", "vssrlrn_bu_h", "vssrarn_bu_h"] in`。
- **L308 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L308 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L309 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L310 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L310 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L311 EN**: Blank line separating nearby declarations or logic blocks.
  **L311 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L312 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L312 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 313-336

````tablegen
                "vssrln_h_w", "vssran_h_w", "vssrln_hu_w", "vssran_hu_w",
                "vssrlrn_h_w", "vssrarn_h_w", "vssrlrn_hu_w", "vssrarn_hu_w"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v8i16_ty],
                                       [llvm_v4i32_ty, llvm_v4i32_ty],
                                       [IntrNoMem]>;

foreach inst = ["vsrln_w_d", "vsran_w_d", "vsrlrn_w_d", "vsrarn_w_d",
                "vssrln_w_d", "vssran_w_d", "vssrln_wu_d", "vssran_wu_d",
                "vssrlrn_w_d", "vssrarn_w_d", "vssrlrn_wu_d", "vssrarn_wu_d"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4i32_ty],
                                       [llvm_v2i64_ty, llvm_v2i64_ty],
                                       [IntrNoMem]>;

foreach inst = ["vmadd_b", "vmsub_b", "vfrstp_b", "vbitsel_v", "vshuf_b"] in
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v16i8_ty],
             [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],
             [IntrNoMem]>;
foreach inst = ["vmadd_h", "vmsub_h", "vfrstp_h", "vshuf_h"] in
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v8i16_ty],
             [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],
             [IntrNoMem]>;
foreach inst = ["vmadd_w", "vmsub_w", "vshuf_w"] in
````
- **L313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vssrln_h_w", "vssran_h_w", "vssrln_hu_w", "vssran_hu_w",`.
  **L313 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vssrln_h_w", "vssran_h_w", "vssrln_hu_w", "vssran_hu_w",`。
- **L314 EN**: Continues the surrounding expression or declaration: `"vssrlrn_h_w", "vssrarn_h_w", "vssrlrn_hu_w", "vssrarn_hu_w"] in`.
  **L314 CN**: 继续构造周围的表达式或声明：`"vssrlrn_h_w", "vssrarn_h_w", "vssrlrn_hu_w", "vssrarn_hu_w"] in`。
- **L315 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L315 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L316 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L317 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L317 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L318 EN**: Blank line separating nearby declarations or logic blocks.
  **L318 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L319 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L319 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vssrln_w_d", "vssran_w_d", "vssrln_wu_d", "vssran_wu_d",`.
  **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vssrln_w_d", "vssran_w_d", "vssrln_wu_d", "vssran_wu_d",`。
- **L321 EN**: Continues the surrounding expression or declaration: `"vssrlrn_w_d", "vssrarn_w_d", "vssrlrn_wu_d", "vssrarn_wu_d"] in`.
  **L321 CN**: 继续构造周围的表达式或声明：`"vssrlrn_w_d", "vssrarn_w_d", "vssrlrn_wu_d", "vssrarn_wu_d"] in`。
- **L322 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L322 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L323 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L324 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L324 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L325 EN**: Blank line separating nearby declarations or logic blocks.
  **L325 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L326 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L326 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L327 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L327 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v16i8_ty],`.
  **L328 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v16i8_ty],`。
- **L329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L329 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L330 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L330 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L331 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L331 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L332 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L332 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v8i16_ty],`.
  **L333 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v8i16_ty],`。
- **L334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L334 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L335 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L335 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L336 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L336 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 337-360

````tablegen
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v4i32_ty],
             [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
             [IntrNoMem]>;
foreach inst = ["vmadd_d", "vmsub_d", "vshuf_d"] in
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v2i64_ty],
             [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],
             [IntrNoMem]>;

foreach inst = ["vsrlni_b_h", "vsrani_b_h", "vsrlrni_b_h", "vsrarni_b_h",
                "vssrlni_b_h", "vssrani_b_h", "vssrlni_bu_h", "vssrani_bu_h",
                "vssrlrni_b_h", "vssrarni_b_h", "vssrlrni_bu_h", "vssrarni_bu_h",
                "vfrstpi_b", "vbitseli_b", "vextrins_b"] in
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v16i8_ty],
             [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
             [IntrNoMem, ImmArg<ArgIndex<2>>]>;
foreach inst = ["vsrlni_h_w", "vsrani_h_w", "vsrlrni_h_w", "vsrarni_h_w",
                "vssrlni_h_w", "vssrani_h_w", "vssrlni_hu_w", "vssrani_hu_w",
                "vssrlrni_h_w", "vssrarni_h_w", "vssrlrni_hu_w", "vssrarni_hu_w",
                "vfrstpi_h", "vextrins_h"] in
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v8i16_ty],
````
- **L337 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L337 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v4i32_ty],`.
  **L338 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v4i32_ty],`。
- **L339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L339 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L340 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L340 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L341 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L341 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L342 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L342 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v2i64_ty],`.
  **L343 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v2i64_ty],`。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L345 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L345 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L346 EN**: Blank line separating nearby declarations or logic blocks.
  **L346 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L347 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L347 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vssrlni_b_h", "vssrani_b_h", "vssrlni_bu_h", "vssrani_bu_h",`.
  **L348 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vssrlni_b_h", "vssrani_b_h", "vssrlni_bu_h", "vssrani_bu_h",`。
- **L349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vssrlrni_b_h", "vssrarni_b_h", "vssrlrni_bu_h", "vssrarni_bu_h",`.
  **L349 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vssrlrni_b_h", "vssrarni_b_h", "vssrlrni_bu_h", "vssrarni_bu_h",`。
- **L350 EN**: Continues the surrounding expression or declaration: `"vfrstpi_b", "vbitseli_b", "vextrins_b"] in`.
  **L350 CN**: 继续构造周围的表达式或声明：`"vfrstpi_b", "vbitseli_b", "vextrins_b"] in`。
- **L351 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L351 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v16i8_ty],`.
  **L352 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v16i8_ty],`。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L354 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L354 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L355 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L355 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vssrlni_h_w", "vssrani_h_w", "vssrlni_hu_w", "vssrani_hu_w",`.
  **L356 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vssrlni_h_w", "vssrani_h_w", "vssrlni_hu_w", "vssrani_hu_w",`。
- **L357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vssrlrni_h_w", "vssrarni_h_w", "vssrlrni_hu_w", "vssrarni_hu_w",`.
  **L357 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vssrlrni_h_w", "vssrarni_h_w", "vssrlrni_hu_w", "vssrarni_hu_w",`。
- **L358 EN**: Continues the surrounding expression or declaration: `"vfrstpi_h", "vextrins_h"] in`.
  **L358 CN**: 继续构造周围的表达式或声明：`"vfrstpi_h", "vextrins_h"] in`。
- **L359 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L359 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v8i16_ty],`.
  **L360 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v8i16_ty],`。

### Lines 361-384

````tablegen
             [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],
             [IntrNoMem, ImmArg<ArgIndex<2>>]>;
foreach inst = ["vsrlni_w_d", "vsrani_w_d", "vsrlrni_w_d", "vsrarni_w_d",
                "vssrlni_w_d", "vssrani_w_d", "vssrlni_wu_d", "vssrani_wu_d",
                "vssrlrni_w_d", "vssrarni_w_d", "vssrlrni_wu_d", "vssrarni_wu_d",
                "vpermi_w", "vextrins_w"] in
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v4i32_ty],
             [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],
             [IntrNoMem, ImmArg<ArgIndex<2>>]>;
foreach inst = ["vsrlni_d_q", "vsrani_d_q", "vsrlrni_d_q", "vsrarni_d_q",
                "vssrlni_d_q", "vssrani_d_q", "vssrlni_du_q", "vssrani_du_q",
                "vssrlrni_d_q", "vssrarni_d_q", "vssrlrni_du_q", "vssrarni_du_q",
                "vshuf4i_d", "vextrins_d"] in
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v2i64_ty],
             [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],
             [IntrNoMem, ImmArg<ArgIndex<2>>]>;

foreach inst = ["vmaddwev_h_b", "vmaddwod_h_b", "vmaddwev_h_bu",
                "vmaddwod_h_bu", "vmaddwev_h_bu_b", "vmaddwod_h_bu_b"] in
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v8i16_ty],
             [llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],
````
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],`.
  **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],`。
- **L362 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L362 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L363 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L363 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vssrlni_w_d", "vssrani_w_d", "vssrlni_wu_d", "vssrani_wu_d",`.
  **L364 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vssrlni_w_d", "vssrani_w_d", "vssrlni_wu_d", "vssrani_wu_d",`。
- **L365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vssrlrni_w_d", "vssrarni_w_d", "vssrlrni_wu_d", "vssrarni_wu_d",`.
  **L365 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vssrlrni_w_d", "vssrarni_w_d", "vssrlrni_wu_d", "vssrarni_wu_d",`。
- **L366 EN**: Continues the surrounding expression or declaration: `"vpermi_w", "vextrins_w"] in`.
  **L366 CN**: 继续构造周围的表达式或声明：`"vpermi_w", "vextrins_w"] in`。
- **L367 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L367 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v4i32_ty],`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v4i32_ty],`。
- **L369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],`.
  **L369 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],`。
- **L370 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L370 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L371 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L371 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vssrlni_d_q", "vssrani_d_q", "vssrlni_du_q", "vssrani_du_q",`.
  **L372 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vssrlni_d_q", "vssrani_d_q", "vssrlni_du_q", "vssrani_du_q",`。
- **L373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vssrlrni_d_q", "vssrarni_d_q", "vssrlrni_du_q", "vssrarni_du_q",`.
  **L373 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vssrlrni_d_q", "vssrarni_d_q", "vssrlrni_du_q", "vssrarni_du_q",`。
- **L374 EN**: Continues the surrounding expression or declaration: `"vshuf4i_d", "vextrins_d"] in`.
  **L374 CN**: 继续构造周围的表达式或声明：`"vshuf4i_d", "vextrins_d"] in`。
- **L375 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L375 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v2i64_ty],`.
  **L376 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v2i64_ty],`。
- **L377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`.
  **L377 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`。
- **L378 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L378 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L379 EN**: Blank line separating nearby declarations or logic blocks.
  **L379 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L380 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L380 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L381 EN**: Continues the surrounding expression or declaration: `"vmaddwod_h_bu", "vmaddwev_h_bu_b", "vmaddwod_h_bu_b"] in`.
  **L381 CN**: 继续构造周围的表达式或声明：`"vmaddwod_h_bu", "vmaddwev_h_bu_b", "vmaddwod_h_bu_b"] in`。
- **L382 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L382 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v8i16_ty],`.
  **L383 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v8i16_ty],`。
- **L384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L384 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。

### Lines 385-408

````tablegen
             [IntrNoMem]>;
foreach inst = ["vmaddwev_w_h", "vmaddwod_w_h", "vmaddwev_w_hu",
                "vmaddwod_w_hu", "vmaddwev_w_hu_h", "vmaddwod_w_hu_h"] in
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v4i32_ty],
             [llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],
             [IntrNoMem]>;
foreach inst = ["vmaddwev_d_w", "vmaddwod_d_w", "vmaddwev_d_wu",
                "vmaddwod_d_wu", "vmaddwev_d_wu_w", "vmaddwod_d_wu_w"] in
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v2i64_ty],
             [llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],
             [IntrNoMem]>;
foreach inst = ["vmaddwev_q_d", "vmaddwod_q_d", "vmaddwev_q_du",
                "vmaddwod_q_du", "vmaddwev_q_du_d", "vmaddwod_q_du_d"] in
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v2i64_ty],
             [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],
             [IntrNoMem]>;

foreach inst = ["vsllwil_h_b", "vsllwil_hu_bu"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v8i16_ty],
                                       [llvm_v16i8_ty, llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<1>>]>;
````
- **L385 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L385 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L386 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L386 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L387 EN**: Continues the surrounding expression or declaration: `"vmaddwod_w_hu", "vmaddwev_w_hu_h", "vmaddwod_w_hu_h"] in`.
  **L387 CN**: 继续构造周围的表达式或声明：`"vmaddwod_w_hu", "vmaddwev_w_hu_h", "vmaddwod_w_hu_h"] in`。
- **L388 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L388 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v4i32_ty],`.
  **L389 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v4i32_ty],`。
- **L390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L390 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L391 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L391 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L392 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L392 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L393 EN**: Continues the surrounding expression or declaration: `"vmaddwod_d_wu", "vmaddwev_d_wu_w", "vmaddwod_d_wu_w"] in`.
  **L393 CN**: 继续构造周围的表达式或声明：`"vmaddwod_d_wu", "vmaddwev_d_wu_w", "vmaddwod_d_wu_w"] in`。
- **L394 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L394 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v2i64_ty],`.
  **L395 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v2i64_ty],`。
- **L396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L396 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L397 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L397 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L398 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L398 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L399 EN**: Continues the surrounding expression or declaration: `"vmaddwod_q_du", "vmaddwev_q_du_d", "vmaddwod_q_du_d"] in`.
  **L399 CN**: 继续构造周围的表达式或声明：`"vmaddwod_q_du", "vmaddwev_q_du_d", "vmaddwod_q_du_d"] in`。
- **L400 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L400 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v2i64_ty],`.
  **L401 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v2i64_ty],`。
- **L402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L402 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L403 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L403 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L404 EN**: Blank line separating nearby declarations or logic blocks.
  **L404 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L405 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L405 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L406 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L406 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_i32_ty],`.
  **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_i32_ty],`。
- **L408 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L408 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。

### Lines 409-432

````tablegen
foreach inst = ["vsllwil_w_h", "vsllwil_wu_hu"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4i32_ty],
                                       [llvm_v8i16_ty, llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<1>>]>;
foreach inst = ["vsllwil_d_w", "vsllwil_du_wu"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2i64_ty],
                                       [llvm_v4i32_ty, llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<1>>]>;

foreach inst = ["vneg_b", "vmskltz_b", "vmskgez_b", "vmsknz_b",
                "vclo_b", "vclz_b", "vpcnt_b"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v16i8_ty], [llvm_v16i8_ty],
                                       [IntrNoMem]>;
foreach inst = ["vneg_h", "vmskltz_h", "vclo_h", "vclz_h", "vpcnt_h"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v8i16_ty], [llvm_v8i16_ty],
                                       [IntrNoMem]>;
foreach inst = ["vneg_w", "vmskltz_w", "vclo_w", "vclz_w", "vpcnt_w"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4i32_ty], [llvm_v4i32_ty],
                                       [IntrNoMem]>;
foreach inst = ["vneg_d", "vexth_q_d", "vexth_qu_du", "vmskltz_d",
                "vextl_q_d", "vextl_qu_du", "vclo_d", "vclz_d", "vpcnt_d"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2i64_ty], [llvm_v2i64_ty],
                                       [IntrNoMem]>;

````
- **L409 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L409 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L410 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L410 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_i32_ty],`.
  **L411 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_i32_ty],`。
- **L412 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L412 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L413 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L413 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L414 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L414 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_i32_ty],`.
  **L415 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_i32_ty],`。
- **L416 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L416 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L417 EN**: Blank line separating nearby declarations or logic blocks.
  **L417 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L418 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L418 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L419 EN**: Continues the surrounding expression or declaration: `"vclo_b", "vclz_b", "vpcnt_b"] in`.
  **L419 CN**: 继续构造周围的表达式或声明：`"vclo_b", "vclz_b", "vpcnt_b"] in`。
- **L420 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L420 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L421 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L421 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L422 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L422 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L423 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L423 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L424 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L424 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L425 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L425 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L426 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L426 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L427 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L427 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L428 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L428 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L429 EN**: Continues the surrounding expression or declaration: `"vextl_q_d", "vextl_qu_du", "vclo_d", "vclz_d", "vpcnt_d"] in`.
  **L429 CN**: 继续构造周围的表达式或声明：`"vextl_q_d", "vextl_qu_du", "vclo_d", "vclz_d", "vpcnt_d"] in`。
- **L430 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L430 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L431 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L431 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L432 EN**: Blank line separating nearby declarations or logic blocks.
  **L432 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 433-456

````tablegen
foreach inst = ["vexth_h_b", "vexth_hu_bu"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v8i16_ty], [llvm_v16i8_ty],
                                       [IntrNoMem]>;
foreach inst = ["vexth_w_h", "vexth_wu_hu"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4i32_ty], [llvm_v8i16_ty],
                                       [IntrNoMem]>;
foreach inst = ["vexth_d_w", "vexth_du_wu"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2i64_ty], [llvm_v4i32_ty],
                                       [IntrNoMem]>;

def int_loongarch_lsx_vldi : VecInt<[llvm_v2i64_ty], [llvm_i32_ty],
                                    [IntrNoMem, ImmArg<ArgIndex<0>>]>;
def int_loongarch_lsx_vrepli_b : VecInt<[llvm_v16i8_ty], [llvm_i32_ty],
                                        [IntrNoMem, ImmArg<ArgIndex<0>>]>;
def int_loongarch_lsx_vrepli_h : VecInt<[llvm_v8i16_ty], [llvm_i32_ty],
                                        [IntrNoMem, ImmArg<ArgIndex<0>>]>;
def int_loongarch_lsx_vrepli_w : VecInt<[llvm_v4i32_ty], [llvm_i32_ty],
                                        [IntrNoMem, ImmArg<ArgIndex<0>>]>;
def int_loongarch_lsx_vrepli_d : VecInt<[llvm_v2i64_ty], [llvm_i32_ty],
                                        [IntrNoMem, ImmArg<ArgIndex<0>>]>;

def int_loongarch_lsx_vreplgr2vr_b : VecInt<[llvm_v16i8_ty], [llvm_i32_ty],
                                            [IntrNoMem]>;
def int_loongarch_lsx_vreplgr2vr_h : VecInt<[llvm_v8i16_ty], [llvm_i32_ty],
````
- **L433 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L433 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L434 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L434 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L435 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L435 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L436 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L436 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L437 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L437 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L438 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L438 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L439 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L439 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L440 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L440 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L441 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L441 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L442 EN**: Blank line separating nearby declarations or logic blocks.
  **L442 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L443 EN**: Declares TableGen def `int_loongarch_lsx_vldi`.
  **L443 CN**: 声明 TableGen def `int_loongarch_lsx_vldi`。
- **L444 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L444 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L445 EN**: Declares TableGen def `int_loongarch_lsx_vrepli_b`.
  **L445 CN**: 声明 TableGen def `int_loongarch_lsx_vrepli_b`。
- **L446 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L446 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L447 EN**: Declares TableGen def `int_loongarch_lsx_vrepli_h`.
  **L447 CN**: 声明 TableGen def `int_loongarch_lsx_vrepli_h`。
- **L448 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L448 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L449 EN**: Declares TableGen def `int_loongarch_lsx_vrepli_w`.
  **L449 CN**: 声明 TableGen def `int_loongarch_lsx_vrepli_w`。
- **L450 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L450 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L451 EN**: Declares TableGen def `int_loongarch_lsx_vrepli_d`.
  **L451 CN**: 声明 TableGen def `int_loongarch_lsx_vrepli_d`。
- **L452 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L452 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L453 EN**: Blank line separating nearby declarations or logic blocks.
  **L453 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L454 EN**: Declares TableGen def `int_loongarch_lsx_vreplgr2vr_b`.
  **L454 CN**: 声明 TableGen def `int_loongarch_lsx_vreplgr2vr_b`。
- **L455 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L455 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L456 EN**: Declares TableGen def `int_loongarch_lsx_vreplgr2vr_h`.
  **L456 CN**: 声明 TableGen def `int_loongarch_lsx_vreplgr2vr_h`。

### Lines 457-480

````tablegen
                                            [IntrNoMem]>;
def int_loongarch_lsx_vreplgr2vr_w : VecInt<[llvm_v4i32_ty], [llvm_i32_ty],
                                            [IntrNoMem]>;
def int_loongarch_lsx_vreplgr2vr_d : VecInt<[llvm_v2i64_ty], [llvm_i64_ty],
                                            [IntrNoMem]>;

def int_loongarch_lsx_vinsgr2vr_b
  : VecInt<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],
           [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_loongarch_lsx_vinsgr2vr_h
  : VecInt<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty, llvm_i32_ty],
           [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_loongarch_lsx_vinsgr2vr_w
  : VecInt<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],
           [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_loongarch_lsx_vinsgr2vr_d
  : VecInt<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i64_ty, llvm_i32_ty],
           [IntrNoMem, ImmArg<ArgIndex<2>>]>;

def int_loongarch_lsx_vreplve_b
  : VecInt<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;
def int_loongarch_lsx_vreplve_h
  : VecInt<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;
def int_loongarch_lsx_vreplve_w
````
- **L457 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L457 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L458 EN**: Declares TableGen def `int_loongarch_lsx_vreplgr2vr_w`.
  **L458 CN**: 声明 TableGen def `int_loongarch_lsx_vreplgr2vr_w`。
- **L459 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L459 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L460 EN**: Declares TableGen def `int_loongarch_lsx_vreplgr2vr_d`.
  **L460 CN**: 声明 TableGen def `int_loongarch_lsx_vreplgr2vr_d`。
- **L461 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L461 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L462 EN**: Blank line separating nearby declarations or logic blocks.
  **L462 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L463 EN**: Declares TableGen def `int_loongarch_lsx_vinsgr2vr_b`.
  **L463 CN**: 声明 TableGen def `int_loongarch_lsx_vinsgr2vr_b`。
- **L464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L464 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L465 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L465 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L466 EN**: Declares TableGen def `int_loongarch_lsx_vinsgr2vr_h`.
  **L466 CN**: 声明 TableGen def `int_loongarch_lsx_vinsgr2vr_h`。
- **L467 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L467 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L468 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L468 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L469 EN**: Declares TableGen def `int_loongarch_lsx_vinsgr2vr_w`.
  **L469 CN**: 声明 TableGen def `int_loongarch_lsx_vinsgr2vr_w`。
- **L470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L470 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L471 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L471 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L472 EN**: Declares TableGen def `int_loongarch_lsx_vinsgr2vr_d`.
  **L472 CN**: 声明 TableGen def `int_loongarch_lsx_vinsgr2vr_d`。
- **L473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i64_ty, llvm_i32_ty],`.
  **L473 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i64_ty, llvm_i32_ty],`。
- **L474 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L474 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L475 EN**: Blank line separating nearby declarations or logic blocks.
  **L475 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L476 EN**: Declares TableGen def `int_loongarch_lsx_vreplve_b`.
  **L476 CN**: 声明 TableGen def `int_loongarch_lsx_vreplve_b`。
- **L477 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L477 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L478 EN**: Declares TableGen def `int_loongarch_lsx_vreplve_h`.
  **L478 CN**: 声明 TableGen def `int_loongarch_lsx_vreplve_h`。
- **L479 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L479 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L480 EN**: Declares TableGen def `int_loongarch_lsx_vreplve_w`.
  **L480 CN**: 声明 TableGen def `int_loongarch_lsx_vreplve_w`。

### Lines 481-504

````tablegen
  : VecInt<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;
def int_loongarch_lsx_vreplve_d
  : VecInt<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;

foreach inst = ["vpickve2gr_b", "vpickve2gr_bu" ] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_i32_ty],
                                       [llvm_v16i8_ty, llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<1>>]>;
foreach inst = ["vpickve2gr_h", "vpickve2gr_hu" ] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_i32_ty],
                                       [llvm_v8i16_ty, llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<1>>]>;
foreach inst = ["vpickve2gr_w", "vpickve2gr_wu" ] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_i32_ty],
                                       [llvm_v4i32_ty, llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<1>>]>;
foreach inst = ["vpickve2gr_d", "vpickve2gr_du" ] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_i64_ty],
                                       [llvm_v2i64_ty, llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_loongarch_lsx_bz_b : VecInt<[llvm_i32_ty], [llvm_v16i8_ty],
                                    [IntrNoMem]>;
def int_loongarch_lsx_bz_h : VecInt<[llvm_i32_ty], [llvm_v8i16_ty],
````
- **L481 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L481 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L482 EN**: Declares TableGen def `int_loongarch_lsx_vreplve_d`.
  **L482 CN**: 声明 TableGen def `int_loongarch_lsx_vreplve_d`。
- **L483 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L483 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L484 EN**: Blank line separating nearby declarations or logic blocks.
  **L484 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L485 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L485 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L486 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L486 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_i32_ty],`.
  **L487 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_i32_ty],`。
- **L488 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L488 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L489 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L489 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L490 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L490 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_i32_ty],`.
  **L491 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_i32_ty],`。
- **L492 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L492 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L493 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L493 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L494 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L494 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_i32_ty],`.
  **L495 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_i32_ty],`。
- **L496 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L496 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L497 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L497 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L498 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L498 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_i32_ty],`.
  **L499 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_i32_ty],`。
- **L500 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L500 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L501 EN**: Blank line separating nearby declarations or logic blocks.
  **L501 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L502 EN**: Declares TableGen def `int_loongarch_lsx_bz_b`.
  **L502 CN**: 声明 TableGen def `int_loongarch_lsx_bz_b`。
- **L503 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L503 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L504 EN**: Declares TableGen def `int_loongarch_lsx_bz_h`.
  **L504 CN**: 声明 TableGen def `int_loongarch_lsx_bz_h`。

### Lines 505-528

````tablegen
                                    [IntrNoMem]>;
def int_loongarch_lsx_bz_w : VecInt<[llvm_i32_ty], [llvm_v4i32_ty],
                                    [IntrNoMem]>;
def int_loongarch_lsx_bz_d : VecInt<[llvm_i32_ty], [llvm_v2i64_ty],
                                    [IntrNoMem]>;
def int_loongarch_lsx_bz_v : VecInt<[llvm_i32_ty], [llvm_v16i8_ty],
                                    [IntrNoMem]>;

def int_loongarch_lsx_bnz_v : VecInt<[llvm_i32_ty], [llvm_v16i8_ty],
                                     [IntrNoMem]>;
def int_loongarch_lsx_bnz_b : VecInt<[llvm_i32_ty], [llvm_v16i8_ty],
                                     [IntrNoMem]>;
def int_loongarch_lsx_bnz_h : VecInt<[llvm_i32_ty], [llvm_v8i16_ty],
                                     [IntrNoMem]>;
def int_loongarch_lsx_bnz_w : VecInt<[llvm_i32_ty], [llvm_v4i32_ty],
                                     [IntrNoMem]>;
def int_loongarch_lsx_bnz_d : VecInt<[llvm_i32_ty], [llvm_v2i64_ty],
                                     [IntrNoMem]>;

// LSX Float

foreach inst = ["vfadd_s", "vfsub_s", "vfmul_s", "vfdiv_s",
                "vfmax_s", "vfmin_s", "vfmaxa_s", "vfmina_s"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4f32_ty],
````
- **L505 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L505 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L506 EN**: Declares TableGen def `int_loongarch_lsx_bz_w`.
  **L506 CN**: 声明 TableGen def `int_loongarch_lsx_bz_w`。
- **L507 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L507 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L508 EN**: Declares TableGen def `int_loongarch_lsx_bz_d`.
  **L508 CN**: 声明 TableGen def `int_loongarch_lsx_bz_d`。
- **L509 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L509 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L510 EN**: Declares TableGen def `int_loongarch_lsx_bz_v`.
  **L510 CN**: 声明 TableGen def `int_loongarch_lsx_bz_v`。
- **L511 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L511 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L513 EN**: Declares TableGen def `int_loongarch_lsx_bnz_v`.
  **L513 CN**: 声明 TableGen def `int_loongarch_lsx_bnz_v`。
- **L514 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L514 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L515 EN**: Declares TableGen def `int_loongarch_lsx_bnz_b`.
  **L515 CN**: 声明 TableGen def `int_loongarch_lsx_bnz_b`。
- **L516 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L516 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L517 EN**: Declares TableGen def `int_loongarch_lsx_bnz_h`.
  **L517 CN**: 声明 TableGen def `int_loongarch_lsx_bnz_h`。
- **L518 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L518 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L519 EN**: Declares TableGen def `int_loongarch_lsx_bnz_w`.
  **L519 CN**: 声明 TableGen def `int_loongarch_lsx_bnz_w`。
- **L520 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L520 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L521 EN**: Declares TableGen def `int_loongarch_lsx_bnz_d`.
  **L521 CN**: 声明 TableGen def `int_loongarch_lsx_bnz_d`。
- **L522 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L522 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L523 EN**: Blank line separating nearby declarations or logic blocks.
  **L523 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L524 EN**: Comment explains nearby logic, invariants, or intent: `LSX Float`.
  **L524 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`LSX Float`。
- **L525 EN**: Blank line separating nearby declarations or logic blocks.
  **L525 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L526 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L526 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L527 EN**: Continues the surrounding expression or declaration: `"vfmax_s", "vfmin_s", "vfmaxa_s", "vfmina_s"] in`.
  **L527 CN**: 继续构造周围的表达式或声明：`"vfmax_s", "vfmin_s", "vfmaxa_s", "vfmina_s"] in`。
- **L528 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L528 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。

### Lines 529-552

````tablegen
                                       [llvm_v4f32_ty, llvm_v4f32_ty],
                                       [IntrNoMem]>;
foreach inst = ["vfadd_d", "vfsub_d", "vfmul_d", "vfdiv_d",
                "vfmax_d", "vfmin_d", "vfmaxa_d", "vfmina_d"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2f64_ty],
                                       [llvm_v2f64_ty, llvm_v2f64_ty],
                                       [IntrNoMem]>;

foreach inst = ["vfmadd_s", "vfmsub_s", "vfnmadd_s", "vfnmsub_s"] in
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v4f32_ty],
             [llvm_v4f32_ty, llvm_v4f32_ty, llvm_v4f32_ty],
             [IntrNoMem]>;
foreach inst = ["vfmadd_d", "vfmsub_d", "vfnmadd_d", "vfnmsub_d"] in
  def int_loongarch_lsx_#inst
    : VecInt<[llvm_v2f64_ty],
             [llvm_v2f64_ty, llvm_v2f64_ty, llvm_v2f64_ty],
             [IntrNoMem]>;

foreach inst = ["vflogb_s", "vfsqrt_s", "vfrecip_s", "vfrsqrt_s", "vfrint_s",
                "vfrecipe_s", "vfrsqrte_s",
                "vfrintrne_s", "vfrintrz_s", "vfrintrp_s", "vfrintrm_s"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4f32_ty], [llvm_v4f32_ty],
                                       [IntrNoMem]>;
````
- **L529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty, llvm_v4f32_ty],`.
  **L529 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty, llvm_v4f32_ty],`。
- **L530 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L530 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L531 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L531 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L532 EN**: Continues the surrounding expression or declaration: `"vfmax_d", "vfmin_d", "vfmaxa_d", "vfmina_d"] in`.
  **L532 CN**: 继续构造周围的表达式或声明：`"vfmax_d", "vfmin_d", "vfmaxa_d", "vfmina_d"] in`。
- **L533 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L533 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2f64_ty, llvm_v2f64_ty],`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2f64_ty, llvm_v2f64_ty],`。
- **L535 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L535 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L536 EN**: Blank line separating nearby declarations or logic blocks.
  **L536 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L537 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L537 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L538 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L538 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v4f32_ty],`.
  **L539 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v4f32_ty],`。
- **L540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty, llvm_v4f32_ty, llvm_v4f32_ty],`.
  **L540 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty, llvm_v4f32_ty, llvm_v4f32_ty],`。
- **L541 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L541 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L542 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L542 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L543 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L543 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v2f64_ty],`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v2f64_ty],`。
- **L545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2f64_ty, llvm_v2f64_ty, llvm_v2f64_ty],`.
  **L545 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2f64_ty, llvm_v2f64_ty, llvm_v2f64_ty],`。
- **L546 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L546 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L547 EN**: Blank line separating nearby declarations or logic blocks.
  **L547 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L548 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L548 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vfrecipe_s", "vfrsqrte_s",`.
  **L549 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vfrecipe_s", "vfrsqrte_s",`。
- **L550 EN**: Continues the surrounding expression or declaration: `"vfrintrne_s", "vfrintrz_s", "vfrintrp_s", "vfrintrm_s"] in`.
  **L550 CN**: 继续构造周围的表达式或声明：`"vfrintrne_s", "vfrintrz_s", "vfrintrp_s", "vfrintrm_s"] in`。
- **L551 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L551 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L552 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L552 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 553-576

````tablegen
foreach inst = ["vflogb_d", "vfsqrt_d", "vfrecip_d", "vfrsqrt_d", "vfrint_d",
                "vfrecipe_d", "vfrsqrte_d",
                "vfrintrne_d", "vfrintrz_d", "vfrintrp_d", "vfrintrm_d"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2f64_ty], [llvm_v2f64_ty],
                                       [IntrNoMem]>;

foreach inst = ["vfcvtl_s_h", "vfcvth_s_h"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4f32_ty], [llvm_v8i16_ty],
                                       [IntrNoMem]>;
foreach inst = ["vfcvtl_d_s", "vfcvth_d_s"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2f64_ty], [llvm_v4f32_ty],
                                       [IntrNoMem]>;

foreach inst = ["vftintrne_w_s", "vftintrz_w_s", "vftintrp_w_s", "vftintrm_w_s",
                "vftint_w_s", "vftintrz_wu_s", "vftint_wu_s", "vfclass_s"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4i32_ty], [llvm_v4f32_ty],
                                       [IntrNoMem]>;
foreach inst = ["vftintrne_l_d", "vftintrz_l_d", "vftintrp_l_d", "vftintrm_l_d",
                "vftint_l_d", "vftintrz_lu_d", "vftint_lu_d", "vfclass_d"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2i64_ty], [llvm_v2f64_ty],
                                       [IntrNoMem]>;

foreach inst = ["vftintrnel_l_s", "vftintrneh_l_s", "vftintrzl_l_s",
                "vftintrzh_l_s", "vftintrpl_l_s", "vftintrph_l_s",
````
- **L553 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L553 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vfrecipe_d", "vfrsqrte_d",`.
  **L554 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vfrecipe_d", "vfrsqrte_d",`。
- **L555 EN**: Continues the surrounding expression or declaration: `"vfrintrne_d", "vfrintrz_d", "vfrintrp_d", "vfrintrm_d"] in`.
  **L555 CN**: 继续构造周围的表达式或声明：`"vfrintrne_d", "vfrintrz_d", "vfrintrp_d", "vfrintrm_d"] in`。
- **L556 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L556 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L557 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L557 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L558 EN**: Blank line separating nearby declarations or logic blocks.
  **L558 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L559 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L559 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L560 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L560 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L561 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L561 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L562 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L562 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L563 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L563 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L564 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L564 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L565 EN**: Blank line separating nearby declarations or logic blocks.
  **L565 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L566 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L566 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L567 EN**: Continues the surrounding expression or declaration: `"vftint_w_s", "vftintrz_wu_s", "vftint_wu_s", "vfclass_s"] in`.
  **L567 CN**: 继续构造周围的表达式或声明：`"vftint_w_s", "vftintrz_wu_s", "vftint_wu_s", "vfclass_s"] in`。
- **L568 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L568 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L569 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L569 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L570 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L570 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L571 EN**: Continues the surrounding expression or declaration: `"vftint_l_d", "vftintrz_lu_d", "vftint_lu_d", "vfclass_d"] in`.
  **L571 CN**: 继续构造周围的表达式或声明：`"vftint_l_d", "vftintrz_lu_d", "vftint_lu_d", "vfclass_d"] in`。
- **L572 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L572 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L573 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L573 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L574 EN**: Blank line separating nearby declarations or logic blocks.
  **L574 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L575 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L575 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vftintrzh_l_s", "vftintrpl_l_s", "vftintrph_l_s",`.
  **L576 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vftintrzh_l_s", "vftintrpl_l_s", "vftintrph_l_s",`。

### Lines 577-600

````tablegen
                "vftintrml_l_s", "vftintrmh_l_s", "vftintl_l_s",
                "vftinth_l_s"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2i64_ty], [llvm_v4f32_ty],
                                       [IntrNoMem]>;

foreach inst = ["vffint_s_w", "vffint_s_wu"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4f32_ty], [llvm_v4i32_ty],
                                       [IntrNoMem]>;
foreach inst = ["vffint_d_l", "vffint_d_lu"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2f64_ty], [llvm_v2i64_ty],
                                       [IntrNoMem]>;

foreach inst = ["vffintl_d_w", "vffinth_d_w"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2f64_ty], [llvm_v4i32_ty],
                                       [IntrNoMem]>;

foreach inst = ["vffint_s_l"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4f32_ty],
                                       [llvm_v2i64_ty, llvm_v2i64_ty],
                                       [IntrNoMem]>;
foreach inst = ["vftintrne_w_d", "vftintrz_w_d", "vftintrp_w_d", "vftintrm_w_d",
                "vftint_w_d"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4i32_ty],
                                       [llvm_v2f64_ty, llvm_v2f64_ty],
````
- **L577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vftintrml_l_s", "vftintrmh_l_s", "vftintl_l_s",`.
  **L577 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vftintrml_l_s", "vftintrmh_l_s", "vftintl_l_s",`。
- **L578 EN**: Continues the surrounding expression or declaration: `"vftinth_l_s"] in`.
  **L578 CN**: 继续构造周围的表达式或声明：`"vftinth_l_s"] in`。
- **L579 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L579 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L580 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L580 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L581 EN**: Blank line separating nearby declarations or logic blocks.
  **L581 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L582 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L582 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L583 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L583 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L584 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L584 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L585 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L585 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L586 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L586 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L587 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L587 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L588 EN**: Blank line separating nearby declarations or logic blocks.
  **L588 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L589 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L589 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L590 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L590 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L591 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L591 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L592 EN**: Blank line separating nearby declarations or logic blocks.
  **L592 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L593 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L593 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L594 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L594 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L595 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L596 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L596 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L597 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L597 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L598 EN**: Continues the surrounding expression or declaration: `"vftint_w_d"] in`.
  **L598 CN**: 继续构造周围的表达式或声明：`"vftint_w_d"] in`。
- **L599 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L599 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2f64_ty, llvm_v2f64_ty],`.
  **L600 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2f64_ty, llvm_v2f64_ty],`。

### Lines 601-624

````tablegen
                                       [IntrNoMem]>;

foreach inst = ["vfcvt_h_s"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v8i16_ty],
                                       [llvm_v4f32_ty, llvm_v4f32_ty],
                                       [IntrNoMem]>;
foreach inst = ["vfcvt_s_d"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4f32_ty],
                                       [llvm_v2f64_ty, llvm_v2f64_ty],
                                       [IntrNoMem]>;

foreach inst = ["vfcmp_caf_s", "vfcmp_cun_s", "vfcmp_ceq_s", "vfcmp_cueq_s",
                "vfcmp_clt_s", "vfcmp_cult_s", "vfcmp_cle_s", "vfcmp_cule_s",
                "vfcmp_cne_s", "vfcmp_cor_s", "vfcmp_cune_s",
                "vfcmp_saf_s", "vfcmp_sun_s", "vfcmp_seq_s", "vfcmp_sueq_s",
                "vfcmp_slt_s", "vfcmp_sult_s", "vfcmp_sle_s", "vfcmp_sule_s",
                "vfcmp_sne_s", "vfcmp_sor_s", "vfcmp_sune_s"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v4i32_ty],
                                       [llvm_v4f32_ty, llvm_v4f32_ty],
                                       [IntrNoMem]>;
foreach inst = ["vfcmp_caf_d", "vfcmp_cun_d", "vfcmp_ceq_d", "vfcmp_cueq_d",
                "vfcmp_clt_d", "vfcmp_cult_d", "vfcmp_cle_d", "vfcmp_cule_d",
                "vfcmp_cne_d", "vfcmp_cor_d", "vfcmp_cune_d",
                "vfcmp_saf_d", "vfcmp_sun_d", "vfcmp_seq_d", "vfcmp_sueq_d",
````
- **L601 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L601 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L602 EN**: Blank line separating nearby declarations or logic blocks.
  **L602 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L603 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L603 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L604 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L604 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty, llvm_v4f32_ty],`.
  **L605 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty, llvm_v4f32_ty],`。
- **L606 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L606 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L607 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L607 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L608 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L608 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2f64_ty, llvm_v2f64_ty],`.
  **L609 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2f64_ty, llvm_v2f64_ty],`。
- **L610 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L610 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L611 EN**: Blank line separating nearby declarations or logic blocks.
  **L611 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L612 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L612 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vfcmp_clt_s", "vfcmp_cult_s", "vfcmp_cle_s", "vfcmp_cule_s",`.
  **L613 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vfcmp_clt_s", "vfcmp_cult_s", "vfcmp_cle_s", "vfcmp_cule_s",`。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vfcmp_cne_s", "vfcmp_cor_s", "vfcmp_cune_s",`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vfcmp_cne_s", "vfcmp_cor_s", "vfcmp_cune_s",`。
- **L615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vfcmp_saf_s", "vfcmp_sun_s", "vfcmp_seq_s", "vfcmp_sueq_s",`.
  **L615 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vfcmp_saf_s", "vfcmp_sun_s", "vfcmp_seq_s", "vfcmp_sueq_s",`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vfcmp_slt_s", "vfcmp_sult_s", "vfcmp_sle_s", "vfcmp_sule_s",`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vfcmp_slt_s", "vfcmp_sult_s", "vfcmp_sle_s", "vfcmp_sule_s",`。
- **L617 EN**: Continues the surrounding expression or declaration: `"vfcmp_sne_s", "vfcmp_sor_s", "vfcmp_sune_s"] in`.
  **L617 CN**: 继续构造周围的表达式或声明：`"vfcmp_sne_s", "vfcmp_sor_s", "vfcmp_sune_s"] in`。
- **L618 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L618 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L619 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty, llvm_v4f32_ty],`.
  **L619 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty, llvm_v4f32_ty],`。
- **L620 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L620 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L621 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L621 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vfcmp_clt_d", "vfcmp_cult_d", "vfcmp_cle_d", "vfcmp_cule_d",`.
  **L622 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vfcmp_clt_d", "vfcmp_cult_d", "vfcmp_cle_d", "vfcmp_cule_d",`。
- **L623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vfcmp_cne_d", "vfcmp_cor_d", "vfcmp_cune_d",`.
  **L623 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vfcmp_cne_d", "vfcmp_cor_d", "vfcmp_cune_d",`。
- **L624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vfcmp_saf_d", "vfcmp_sun_d", "vfcmp_seq_d", "vfcmp_sueq_d",`.
  **L624 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vfcmp_saf_d", "vfcmp_sun_d", "vfcmp_seq_d", "vfcmp_sueq_d",`。

### Lines 625-648

````tablegen
                "vfcmp_slt_d", "vfcmp_sult_d", "vfcmp_sle_d", "vfcmp_sule_d",
                "vfcmp_sne_d", "vfcmp_sor_d", "vfcmp_sune_d"] in
  def int_loongarch_lsx_#inst : VecInt<[llvm_v2i64_ty],
                                       [llvm_v2f64_ty, llvm_v2f64_ty],
                                       [IntrNoMem]>;

// LSX load/store
def int_loongarch_lsx_vld
  : DefaultAttrsVecInt<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i32_ty],
           [IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;
def int_loongarch_lsx_vldx
  : DefaultAttrsVecInt<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i64_ty],
           [IntrReadMem, IntrArgMemOnly]>;
def int_loongarch_lsx_vldrepl_b
  : DefaultAttrsVecInt<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i32_ty],
           [IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;
def int_loongarch_lsx_vldrepl_h
  : DefaultAttrsVecInt<[llvm_v8i16_ty], [llvm_ptr_ty, llvm_i32_ty],
           [IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;
def int_loongarch_lsx_vldrepl_w
  : DefaultAttrsVecInt<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i32_ty],
           [IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;
def int_loongarch_lsx_vldrepl_d
  : DefaultAttrsVecInt<[llvm_v2i64_ty], [llvm_ptr_ty, llvm_i32_ty],
````
- **L625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vfcmp_slt_d", "vfcmp_sult_d", "vfcmp_sle_d", "vfcmp_sule_d",`.
  **L625 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vfcmp_slt_d", "vfcmp_sult_d", "vfcmp_sle_d", "vfcmp_sule_d",`。
- **L626 EN**: Continues the surrounding expression or declaration: `"vfcmp_sne_d", "vfcmp_sor_d", "vfcmp_sune_d"] in`.
  **L626 CN**: 继续构造周围的表达式或声明：`"vfcmp_sne_d", "vfcmp_sor_d", "vfcmp_sune_d"] in`。
- **L627 EN**: Declares TableGen def `int_loongarch_lsx_#inst`.
  **L627 CN**: 声明 TableGen def `int_loongarch_lsx_#inst`。
- **L628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2f64_ty, llvm_v2f64_ty],`.
  **L628 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2f64_ty, llvm_v2f64_ty],`。
- **L629 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L629 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L630 EN**: Blank line separating nearby declarations or logic blocks.
  **L630 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L631 EN**: Comment explains nearby logic, invariants, or intent: `LSX load/store`.
  **L631 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`LSX load/store`。
- **L632 EN**: Declares TableGen def `int_loongarch_lsx_vld`.
  **L632 CN**: 声明 TableGen def `int_loongarch_lsx_vld`。
- **L633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsVecInt<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L633 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsVecInt<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L634 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`.
  **L634 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`。
- **L635 EN**: Declares TableGen def `int_loongarch_lsx_vldx`.
  **L635 CN**: 声明 TableGen def `int_loongarch_lsx_vldx`。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsVecInt<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i64_ty],`.
  **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsVecInt<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i64_ty],`。
- **L637 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L637 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L638 EN**: Declares TableGen def `int_loongarch_lsx_vldrepl_b`.
  **L638 CN**: 声明 TableGen def `int_loongarch_lsx_vldrepl_b`。
- **L639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsVecInt<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L639 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsVecInt<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L640 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`.
  **L640 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`。
- **L641 EN**: Declares TableGen def `int_loongarch_lsx_vldrepl_h`.
  **L641 CN**: 声明 TableGen def `int_loongarch_lsx_vldrepl_h`。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsVecInt<[llvm_v8i16_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsVecInt<[llvm_v8i16_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L643 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`.
  **L643 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`。
- **L644 EN**: Declares TableGen def `int_loongarch_lsx_vldrepl_w`.
  **L644 CN**: 声明 TableGen def `int_loongarch_lsx_vldrepl_w`。
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsVecInt<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsVecInt<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L646 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`.
  **L646 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`。
- **L647 EN**: Declares TableGen def `int_loongarch_lsx_vldrepl_d`.
  **L647 CN**: 声明 TableGen def `int_loongarch_lsx_vldrepl_d`。
- **L648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsVecInt<[llvm_v2i64_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L648 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsVecInt<[llvm_v2i64_ty], [llvm_ptr_ty, llvm_i32_ty],`。

### Lines 649-672

````tablegen
           [IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;

def int_loongarch_lsx_vst
  : VecInt<[], [llvm_v16i8_ty, llvm_ptr_ty, llvm_i32_ty],
           [IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;
def int_loongarch_lsx_vstx
  : VecInt<[], [llvm_v16i8_ty, llvm_ptr_ty, llvm_i64_ty],
           [IntrWriteMem, IntrArgMemOnly]>;
def int_loongarch_lsx_vstelm_b
  : VecInt<[], [llvm_v16i8_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],
           [IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;
def int_loongarch_lsx_vstelm_h
  : VecInt<[], [llvm_v8i16_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],
           [IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;
def int_loongarch_lsx_vstelm_w
  : VecInt<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],
           [IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;
def int_loongarch_lsx_vstelm_d
  : VecInt<[], [llvm_v2i64_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],
           [IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;

} // TargetPrefix = "loongarch"

//===----------------------------------------------------------------------===//
````
- **L649 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`.
  **L649 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`。
- **L650 EN**: Blank line separating nearby declarations or logic blocks.
  **L650 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L651 EN**: Declares TableGen def `int_loongarch_lsx_vst`.
  **L651 CN**: 声明 TableGen def `int_loongarch_lsx_vst`。
- **L652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[], [llvm_v16i8_ty, llvm_ptr_ty, llvm_i32_ty],`.
  **L652 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[], [llvm_v16i8_ty, llvm_ptr_ty, llvm_i32_ty],`。
- **L653 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`.
  **L653 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`。
- **L654 EN**: Declares TableGen def `int_loongarch_lsx_vstx`.
  **L654 CN**: 声明 TableGen def `int_loongarch_lsx_vstx`。
- **L655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[], [llvm_v16i8_ty, llvm_ptr_ty, llvm_i64_ty],`.
  **L655 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[], [llvm_v16i8_ty, llvm_ptr_ty, llvm_i64_ty],`。
- **L656 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L656 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L657 EN**: Declares TableGen def `int_loongarch_lsx_vstelm_b`.
  **L657 CN**: 声明 TableGen def `int_loongarch_lsx_vstelm_b`。
- **L658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[], [llvm_v16i8_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L658 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[], [llvm_v16i8_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L659 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`.
  **L659 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`。
- **L660 EN**: Declares TableGen def `int_loongarch_lsx_vstelm_h`.
  **L660 CN**: 声明 TableGen def `int_loongarch_lsx_vstelm_h`。
- **L661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[], [llvm_v8i16_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L661 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[], [llvm_v8i16_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L662 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`.
  **L662 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`。
- **L663 EN**: Declares TableGen def `int_loongarch_lsx_vstelm_w`.
  **L663 CN**: 声明 TableGen def `int_loongarch_lsx_vstelm_w`。
- **L664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L664 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L665 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`.
  **L665 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`。
- **L666 EN**: Declares TableGen def `int_loongarch_lsx_vstelm_d`.
  **L666 CN**: 声明 TableGen def `int_loongarch_lsx_vstelm_d`。
- **L667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[], [llvm_v2i64_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L667 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[], [llvm_v2i64_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L668 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`.
  **L668 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`。
- **L669 EN**: Blank line separating nearby declarations or logic blocks.
  **L669 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L670 EN**: Continues the surrounding expression or declaration: `} // TargetPrefix = "loongarch"`.
  **L670 CN**: 继续构造周围的表达式或声明：`} // TargetPrefix = "loongarch"`。
- **L671 EN**: Blank line separating nearby declarations or logic blocks.
  **L671 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L672 EN**: Banner comment marking a file or section boundary.
  **L672 CN**: 横幅注释，用于标记文件或章节边界。

### Lines 673-696

````tablegen
// LASX

let TargetPrefix = "loongarch" in {
foreach inst = ["xvadd_b", "xvsub_b",
                "xvsadd_b", "xvsadd_bu", "xvssub_b", "xvssub_bu",
                "xvavg_b", "xvavg_bu", "xvavgr_b", "xvavgr_bu",
                "xvabsd_b", "xvabsd_bu", "xvadda_b",
                "xvmax_b", "xvmax_bu", "xvmin_b", "xvmin_bu",
                "xvmul_b", "xvmuh_b", "xvmuh_bu",
                "xvdiv_b", "xvdiv_bu", "xvmod_b", "xvmod_bu", "xvsigncov_b",
                "xvand_v", "xvor_v", "xvxor_v", "xvnor_v", "xvandn_v", "xvorn_v",
                "xvsll_b", "xvsrl_b", "xvsra_b", "xvrotr_b", "xvsrlr_b", "xvsrar_b",
                "xvbitclr_b", "xvbitset_b", "xvbitrev_b",
                "xvseq_b", "xvsle_b", "xvsle_bu", "xvslt_b", "xvslt_bu",
                "xvpackev_b", "xvpackod_b", "xvpickev_b", "xvpickod_b",
                "xvilvl_b", "xvilvh_b"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v32i8_ty],
                                        [llvm_v32i8_ty, llvm_v32i8_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvadd_h", "xvsub_h",
                "xvsadd_h", "xvsadd_hu", "xvssub_h", "xvssub_hu",
                "xvavg_h", "xvavg_hu", "xvavgr_h", "xvavgr_hu",
                "xvabsd_h", "xvabsd_hu", "xvadda_h",
````
- **L673 EN**: Comment explains nearby logic, invariants, or intent: `LASX`.
  **L673 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`LASX`。
- **L674 EN**: Blank line separating nearby declarations or logic blocks.
  **L674 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L675 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L675 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L676 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L676 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsadd_b", "xvsadd_bu", "xvssub_b", "xvssub_bu",`.
  **L677 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsadd_b", "xvsadd_bu", "xvssub_b", "xvssub_bu",`。
- **L678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvavg_b", "xvavg_bu", "xvavgr_b", "xvavgr_bu",`.
  **L678 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvavg_b", "xvavg_bu", "xvavgr_b", "xvavgr_bu",`。
- **L679 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvabsd_b", "xvabsd_bu", "xvadda_b",`.
  **L679 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvabsd_b", "xvabsd_bu", "xvadda_b",`。
- **L680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmax_b", "xvmax_bu", "xvmin_b", "xvmin_bu",`.
  **L680 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmax_b", "xvmax_bu", "xvmin_b", "xvmin_bu",`。
- **L681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmul_b", "xvmuh_b", "xvmuh_bu",`.
  **L681 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmul_b", "xvmuh_b", "xvmuh_bu",`。
- **L682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvdiv_b", "xvdiv_bu", "xvmod_b", "xvmod_bu", "xvsigncov_b",`.
  **L682 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvdiv_b", "xvdiv_bu", "xvmod_b", "xvmod_bu", "xvsigncov_b",`。
- **L683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvand_v", "xvor_v", "xvxor_v", "xvnor_v", "xvandn_v", "xvorn_v",`.
  **L683 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvand_v", "xvor_v", "xvxor_v", "xvnor_v", "xvandn_v", "xvorn_v",`。
- **L684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsll_b", "xvsrl_b", "xvsra_b", "xvrotr_b", "xvsrlr_b", "xvsrar_b",`.
  **L684 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsll_b", "xvsrl_b", "xvsra_b", "xvrotr_b", "xvsrlr_b", "xvsrar_b",`。
- **L685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvbitclr_b", "xvbitset_b", "xvbitrev_b",`.
  **L685 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvbitclr_b", "xvbitset_b", "xvbitrev_b",`。
- **L686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvseq_b", "xvsle_b", "xvsle_bu", "xvslt_b", "xvslt_bu",`.
  **L686 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvseq_b", "xvsle_b", "xvsle_bu", "xvslt_b", "xvslt_bu",`。
- **L687 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvpackev_b", "xvpackod_b", "xvpickev_b", "xvpickod_b",`.
  **L687 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvpackev_b", "xvpackod_b", "xvpickev_b", "xvpickod_b",`。
- **L688 EN**: Continues the surrounding expression or declaration: `"xvilvl_b", "xvilvh_b"] in`.
  **L688 CN**: 继续构造周围的表达式或声明：`"xvilvl_b", "xvilvh_b"] in`。
- **L689 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L689 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v32i8_ty, llvm_v32i8_ty],`.
  **L690 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v32i8_ty, llvm_v32i8_ty],`。
- **L691 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L691 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L692 EN**: Blank line separating nearby declarations or logic blocks.
  **L692 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L693 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L693 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsadd_h", "xvsadd_hu", "xvssub_h", "xvssub_hu",`.
  **L694 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsadd_h", "xvsadd_hu", "xvssub_h", "xvssub_hu",`。
- **L695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvavg_h", "xvavg_hu", "xvavgr_h", "xvavgr_hu",`.
  **L695 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvavg_h", "xvavg_hu", "xvavgr_h", "xvavgr_hu",`。
- **L696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvabsd_h", "xvabsd_hu", "xvadda_h",`.
  **L696 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvabsd_h", "xvabsd_hu", "xvadda_h",`。

### Lines 697-720

````tablegen
                "xvmax_h", "xvmax_hu", "xvmin_h", "xvmin_hu",
                "xvmul_h", "xvmuh_h", "xvmuh_hu",
                "xvdiv_h", "xvdiv_hu", "xvmod_h", "xvmod_hu", "xvsigncov_h",
                "xvsll_h", "xvsrl_h", "xvsra_h", "xvrotr_h", "xvsrlr_h", "xvsrar_h",
                "xvbitclr_h", "xvbitset_h", "xvbitrev_h",
                "xvseq_h", "xvsle_h", "xvsle_hu", "xvslt_h", "xvslt_hu",
                "xvpackev_h", "xvpackod_h", "xvpickev_h", "xvpickod_h",
                "xvilvl_h", "xvilvh_h"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v16i16_ty],
                                        [llvm_v16i16_ty, llvm_v16i16_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvadd_w", "xvsub_w",
                "xvsadd_w", "xvsadd_wu", "xvssub_w", "xvssub_wu",
                "xvavg_w", "xvavg_wu", "xvavgr_w", "xvavgr_wu",
                "xvabsd_w", "xvabsd_wu", "xvadda_w",
                "xvmax_w", "xvmax_wu", "xvmin_w", "xvmin_wu",
                "xvmul_w", "xvmuh_w", "xvmuh_wu",
                "xvdiv_w", "xvdiv_wu", "xvmod_w", "xvmod_wu", "xvsigncov_w",
                "xvsll_w", "xvsrl_w", "xvsra_w", "xvrotr_w", "xvsrlr_w", "xvsrar_w",
                "xvbitclr_w", "xvbitset_w", "xvbitrev_w",
                "xvseq_w", "xvsle_w", "xvsle_wu", "xvslt_w", "xvslt_wu",
                "xvpackev_w", "xvpackod_w", "xvpickev_w", "xvpickod_w",
                "xvilvl_w", "xvilvh_w", "xvperm_w"] in
````
- **L697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmax_h", "xvmax_hu", "xvmin_h", "xvmin_hu",`.
  **L697 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmax_h", "xvmax_hu", "xvmin_h", "xvmin_hu",`。
- **L698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmul_h", "xvmuh_h", "xvmuh_hu",`.
  **L698 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmul_h", "xvmuh_h", "xvmuh_hu",`。
- **L699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvdiv_h", "xvdiv_hu", "xvmod_h", "xvmod_hu", "xvsigncov_h",`.
  **L699 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvdiv_h", "xvdiv_hu", "xvmod_h", "xvmod_hu", "xvsigncov_h",`。
- **L700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsll_h", "xvsrl_h", "xvsra_h", "xvrotr_h", "xvsrlr_h", "xvsrar_h",`.
  **L700 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsll_h", "xvsrl_h", "xvsra_h", "xvrotr_h", "xvsrlr_h", "xvsrar_h",`。
- **L701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvbitclr_h", "xvbitset_h", "xvbitrev_h",`.
  **L701 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvbitclr_h", "xvbitset_h", "xvbitrev_h",`。
- **L702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvseq_h", "xvsle_h", "xvsle_hu", "xvslt_h", "xvslt_hu",`.
  **L702 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvseq_h", "xvsle_h", "xvsle_hu", "xvslt_h", "xvslt_hu",`。
- **L703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvpackev_h", "xvpackod_h", "xvpickev_h", "xvpickod_h",`.
  **L703 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvpackev_h", "xvpackod_h", "xvpickev_h", "xvpickod_h",`。
- **L704 EN**: Continues the surrounding expression or declaration: `"xvilvl_h", "xvilvh_h"] in`.
  **L704 CN**: 继续构造周围的表达式或声明：`"xvilvl_h", "xvilvh_h"] in`。
- **L705 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L705 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i16_ty, llvm_v16i16_ty],`.
  **L706 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i16_ty, llvm_v16i16_ty],`。
- **L707 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L707 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L708 EN**: Blank line separating nearby declarations or logic blocks.
  **L708 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L709 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L709 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L710 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsadd_w", "xvsadd_wu", "xvssub_w", "xvssub_wu",`.
  **L710 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsadd_w", "xvsadd_wu", "xvssub_w", "xvssub_wu",`。
- **L711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvavg_w", "xvavg_wu", "xvavgr_w", "xvavgr_wu",`.
  **L711 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvavg_w", "xvavg_wu", "xvavgr_w", "xvavgr_wu",`。
- **L712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvabsd_w", "xvabsd_wu", "xvadda_w",`.
  **L712 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvabsd_w", "xvabsd_wu", "xvadda_w",`。
- **L713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmax_w", "xvmax_wu", "xvmin_w", "xvmin_wu",`.
  **L713 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmax_w", "xvmax_wu", "xvmin_w", "xvmin_wu",`。
- **L714 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmul_w", "xvmuh_w", "xvmuh_wu",`.
  **L714 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmul_w", "xvmuh_w", "xvmuh_wu",`。
- **L715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvdiv_w", "xvdiv_wu", "xvmod_w", "xvmod_wu", "xvsigncov_w",`.
  **L715 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvdiv_w", "xvdiv_wu", "xvmod_w", "xvmod_wu", "xvsigncov_w",`。
- **L716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsll_w", "xvsrl_w", "xvsra_w", "xvrotr_w", "xvsrlr_w", "xvsrar_w",`.
  **L716 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsll_w", "xvsrl_w", "xvsra_w", "xvrotr_w", "xvsrlr_w", "xvsrar_w",`。
- **L717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvbitclr_w", "xvbitset_w", "xvbitrev_w",`.
  **L717 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvbitclr_w", "xvbitset_w", "xvbitrev_w",`。
- **L718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvseq_w", "xvsle_w", "xvsle_wu", "xvslt_w", "xvslt_wu",`.
  **L718 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvseq_w", "xvsle_w", "xvsle_wu", "xvslt_w", "xvslt_wu",`。
- **L719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvpackev_w", "xvpackod_w", "xvpickev_w", "xvpickod_w",`.
  **L719 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvpackev_w", "xvpackod_w", "xvpickev_w", "xvpickod_w",`。
- **L720 EN**: Continues the surrounding expression or declaration: `"xvilvl_w", "xvilvh_w", "xvperm_w"] in`.
  **L720 CN**: 继续构造周围的表达式或声明：`"xvilvl_w", "xvilvh_w", "xvperm_w"] in`。

### Lines 721-744

````tablegen
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8i32_ty],
                                        [llvm_v8i32_ty, llvm_v8i32_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvadd_d", "xvadd_q", "xvsub_d", "xvsub_q",
                "xvsadd_d", "xvsadd_du", "xvssub_d", "xvssub_du",
                "xvhaddw_q_d", "xvhaddw_qu_du", "xvhsubw_q_d", "xvhsubw_qu_du",
                "xvaddwev_q_d", "xvaddwod_q_d", "xvsubwev_q_d", "xvsubwod_q_d",
                "xvaddwev_q_du", "xvaddwod_q_du", "xvsubwev_q_du", "xvsubwod_q_du",
                "xvaddwev_q_du_d", "xvaddwod_q_du_d",
                "xvavg_d", "xvavg_du", "xvavgr_d", "xvavgr_du",
                "xvabsd_d", "xvabsd_du", "xvadda_d",
                "xvmax_d", "xvmax_du", "xvmin_d", "xvmin_du",
                "xvmul_d", "xvmuh_d", "xvmuh_du",
                "xvmulwev_q_d", "xvmulwod_q_d", "xvmulwev_q_du", "xvmulwod_q_du",
                "xvmulwev_q_du_d", "xvmulwod_q_du_d",
                "xvdiv_d", "xvdiv_du", "xvmod_d", "xvmod_du", "xvsigncov_d",
                "xvsll_d", "xvsrl_d", "xvsra_d", "xvrotr_d", "xvsrlr_d", "xvsrar_d",
                "xvbitclr_d", "xvbitset_d", "xvbitrev_d",
                "xvseq_d", "xvsle_d", "xvsle_du", "xvslt_d", "xvslt_du",
                "xvpackev_d", "xvpackod_d", "xvpickev_d", "xvpickod_d",
                "xvilvl_d", "xvilvh_d"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4i64_ty],
                                        [llvm_v4i64_ty, llvm_v4i64_ty],
````
- **L721 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L721 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L722 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i32_ty, llvm_v8i32_ty],`.
  **L722 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i32_ty, llvm_v8i32_ty],`。
- **L723 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L723 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L724 EN**: Blank line separating nearby declarations or logic blocks.
  **L724 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L725 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L725 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsadd_d", "xvsadd_du", "xvssub_d", "xvssub_du",`.
  **L726 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsadd_d", "xvsadd_du", "xvssub_d", "xvssub_du",`。
- **L727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvhaddw_q_d", "xvhaddw_qu_du", "xvhsubw_q_d", "xvhsubw_qu_du",`.
  **L727 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvhaddw_q_d", "xvhaddw_qu_du", "xvhsubw_q_d", "xvhsubw_qu_du",`。
- **L728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvaddwev_q_d", "xvaddwod_q_d", "xvsubwev_q_d", "xvsubwod_q_d",`.
  **L728 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvaddwev_q_d", "xvaddwod_q_d", "xvsubwev_q_d", "xvsubwod_q_d",`。
- **L729 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvaddwev_q_du", "xvaddwod_q_du", "xvsubwev_q_du", "xvsubwod_q_du",`.
  **L729 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvaddwev_q_du", "xvaddwod_q_du", "xvsubwev_q_du", "xvsubwod_q_du",`。
- **L730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvaddwev_q_du_d", "xvaddwod_q_du_d",`.
  **L730 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvaddwev_q_du_d", "xvaddwod_q_du_d",`。
- **L731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvavg_d", "xvavg_du", "xvavgr_d", "xvavgr_du",`.
  **L731 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvavg_d", "xvavg_du", "xvavgr_d", "xvavgr_du",`。
- **L732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvabsd_d", "xvabsd_du", "xvadda_d",`.
  **L732 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvabsd_d", "xvabsd_du", "xvadda_d",`。
- **L733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmax_d", "xvmax_du", "xvmin_d", "xvmin_du",`.
  **L733 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmax_d", "xvmax_du", "xvmin_d", "xvmin_du",`。
- **L734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmul_d", "xvmuh_d", "xvmuh_du",`.
  **L734 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmul_d", "xvmuh_d", "xvmuh_du",`。
- **L735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmulwev_q_d", "xvmulwod_q_d", "xvmulwev_q_du", "xvmulwod_q_du",`.
  **L735 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmulwev_q_d", "xvmulwod_q_d", "xvmulwev_q_du", "xvmulwod_q_du",`。
- **L736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmulwev_q_du_d", "xvmulwod_q_du_d",`.
  **L736 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmulwev_q_du_d", "xvmulwod_q_du_d",`。
- **L737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvdiv_d", "xvdiv_du", "xvmod_d", "xvmod_du", "xvsigncov_d",`.
  **L737 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvdiv_d", "xvdiv_du", "xvmod_d", "xvmod_du", "xvsigncov_d",`。
- **L738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsll_d", "xvsrl_d", "xvsra_d", "xvrotr_d", "xvsrlr_d", "xvsrar_d",`.
  **L738 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsll_d", "xvsrl_d", "xvsra_d", "xvrotr_d", "xvsrlr_d", "xvsrar_d",`。
- **L739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvbitclr_d", "xvbitset_d", "xvbitrev_d",`.
  **L739 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvbitclr_d", "xvbitset_d", "xvbitrev_d",`。
- **L740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvseq_d", "xvsle_d", "xvsle_du", "xvslt_d", "xvslt_du",`.
  **L740 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvseq_d", "xvsle_d", "xvsle_du", "xvslt_d", "xvslt_du",`。
- **L741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvpackev_d", "xvpackod_d", "xvpickev_d", "xvpickod_d",`.
  **L741 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvpackev_d", "xvpackod_d", "xvpickev_d", "xvpickod_d",`。
- **L742 EN**: Continues the surrounding expression or declaration: `"xvilvl_d", "xvilvh_d"] in`.
  **L742 CN**: 继续构造周围的表达式或声明：`"xvilvl_d", "xvilvh_d"] in`。
- **L743 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L743 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i64_ty, llvm_v4i64_ty],`.
  **L744 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i64_ty, llvm_v4i64_ty],`。

### Lines 745-768

````tablegen
                                        [IntrNoMem]>;

foreach inst = ["xvaddi_bu", "xvsubi_bu",
                "xvmaxi_b", "xvmaxi_bu", "xvmini_b", "xvmini_bu",
                "xvsat_b", "xvsat_bu",
                "xvandi_b", "xvori_b", "xvxori_b", "xvnori_b",
                "xvslli_b", "xvsrli_b", "xvsrai_b", "xvrotri_b",
                "xvsrlri_b", "xvsrari_b",
                "xvbitclri_b", "xvbitseti_b", "xvbitrevi_b",
                "xvseqi_b", "xvslei_b", "xvslei_bu", "xvslti_b", "xvslti_bu",
                "xvrepl128vei_b", "xvbsll_v", "xvbsrl_v", "xvshuf4i_b"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v32i8_ty],
                                        [llvm_v32i8_ty, llvm_i32_ty],
                                        [IntrNoMem, ImmArg<ArgIndex<1>>]>;
foreach inst = ["xvaddi_hu", "xvsubi_hu",
                "xvmaxi_h", "xvmaxi_hu", "xvmini_h", "xvmini_hu",
                "xvsat_h", "xvsat_hu",
                "xvslli_h", "xvsrli_h", "xvsrai_h", "xvrotri_h",
                "xvsrlri_h", "xvsrari_h",
                "xvbitclri_h", "xvbitseti_h", "xvbitrevi_h",
                "xvseqi_h", "xvslei_h", "xvslei_hu", "xvslti_h", "xvslti_hu",
                "xvrepl128vei_h", "xvshuf4i_h"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v16i16_ty],
                                        [llvm_v16i16_ty, llvm_i32_ty],
````
- **L745 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L745 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L746 EN**: Blank line separating nearby declarations or logic blocks.
  **L746 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L747 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L747 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmaxi_b", "xvmaxi_bu", "xvmini_b", "xvmini_bu",`.
  **L748 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmaxi_b", "xvmaxi_bu", "xvmini_b", "xvmini_bu",`。
- **L749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsat_b", "xvsat_bu",`.
  **L749 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsat_b", "xvsat_bu",`。
- **L750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvandi_b", "xvori_b", "xvxori_b", "xvnori_b",`.
  **L750 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvandi_b", "xvori_b", "xvxori_b", "xvnori_b",`。
- **L751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvslli_b", "xvsrli_b", "xvsrai_b", "xvrotri_b",`.
  **L751 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvslli_b", "xvsrli_b", "xvsrai_b", "xvrotri_b",`。
- **L752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsrlri_b", "xvsrari_b",`.
  **L752 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsrlri_b", "xvsrari_b",`。
- **L753 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvbitclri_b", "xvbitseti_b", "xvbitrevi_b",`.
  **L753 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvbitclri_b", "xvbitseti_b", "xvbitrevi_b",`。
- **L754 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvseqi_b", "xvslei_b", "xvslei_bu", "xvslti_b", "xvslti_bu",`.
  **L754 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvseqi_b", "xvslei_b", "xvslei_bu", "xvslti_b", "xvslti_bu",`。
- **L755 EN**: Continues the surrounding expression or declaration: `"xvrepl128vei_b", "xvbsll_v", "xvbsrl_v", "xvshuf4i_b"] in`.
  **L755 CN**: 继续构造周围的表达式或声明：`"xvrepl128vei_b", "xvbsll_v", "xvbsrl_v", "xvshuf4i_b"] in`。
- **L756 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L756 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v32i8_ty, llvm_i32_ty],`.
  **L757 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v32i8_ty, llvm_i32_ty],`。
- **L758 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L758 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L759 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L759 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmaxi_h", "xvmaxi_hu", "xvmini_h", "xvmini_hu",`.
  **L760 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmaxi_h", "xvmaxi_hu", "xvmini_h", "xvmini_hu",`。
- **L761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsat_h", "xvsat_hu",`.
  **L761 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsat_h", "xvsat_hu",`。
- **L762 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvslli_h", "xvsrli_h", "xvsrai_h", "xvrotri_h",`.
  **L762 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvslli_h", "xvsrli_h", "xvsrai_h", "xvrotri_h",`。
- **L763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsrlri_h", "xvsrari_h",`.
  **L763 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsrlri_h", "xvsrari_h",`。
- **L764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvbitclri_h", "xvbitseti_h", "xvbitrevi_h",`.
  **L764 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvbitclri_h", "xvbitseti_h", "xvbitrevi_h",`。
- **L765 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvseqi_h", "xvslei_h", "xvslei_hu", "xvslti_h", "xvslti_hu",`.
  **L765 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvseqi_h", "xvslei_h", "xvslei_hu", "xvslti_h", "xvslti_hu",`。
- **L766 EN**: Continues the surrounding expression or declaration: `"xvrepl128vei_h", "xvshuf4i_h"] in`.
  **L766 CN**: 继续构造周围的表达式或声明：`"xvrepl128vei_h", "xvshuf4i_h"] in`。
- **L767 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L767 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i16_ty, llvm_i32_ty],`.
  **L768 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i16_ty, llvm_i32_ty],`。

### Lines 769-792

````tablegen
                                        [IntrNoMem, ImmArg<ArgIndex<1>>]>;
foreach inst = ["xvaddi_wu", "xvsubi_wu",
                "xvmaxi_w", "xvmaxi_wu", "xvmini_w", "xvmini_wu",
                "xvsat_w", "xvsat_wu",
                "xvslli_w", "xvsrli_w", "xvsrai_w", "xvrotri_w",
                "xvsrlri_w", "xvsrari_w",
                "xvbitclri_w", "xvbitseti_w", "xvbitrevi_w",
                "xvseqi_w", "xvslei_w", "xvslei_wu", "xvslti_w", "xvslti_wu",
                "xvrepl128vei_w", "xvshuf4i_w", "xvpickve_w"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8i32_ty],
                                        [llvm_v8i32_ty, llvm_i32_ty],
                                        [IntrNoMem, ImmArg<ArgIndex<1>>]>;
foreach inst = ["xvaddi_du", "xvsubi_du",
                "xvmaxi_d", "xvmaxi_du", "xvmini_d", "xvmini_du",
                "xvsat_d", "xvsat_du",
                "xvslli_d", "xvsrli_d", "xvsrai_d", "xvrotri_d",
                "xvsrlri_d", "xvsrari_d",
                "xvbitclri_d", "xvbitseti_d", "xvbitrevi_d",
                "xvseqi_d", "xvslei_d", "xvslei_du", "xvslti_d", "xvslti_du",
                "xvrepl128vei_d", "xvpermi_d", "xvpickve_d"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4i64_ty],
                                        [llvm_v4i64_ty, llvm_i32_ty],
                                        [IntrNoMem, ImmArg<ArgIndex<1>>]>;

````
- **L769 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L769 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L770 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L770 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L771 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmaxi_w", "xvmaxi_wu", "xvmini_w", "xvmini_wu",`.
  **L771 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmaxi_w", "xvmaxi_wu", "xvmini_w", "xvmini_wu",`。
- **L772 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsat_w", "xvsat_wu",`.
  **L772 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsat_w", "xvsat_wu",`。
- **L773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvslli_w", "xvsrli_w", "xvsrai_w", "xvrotri_w",`.
  **L773 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvslli_w", "xvsrli_w", "xvsrai_w", "xvrotri_w",`。
- **L774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsrlri_w", "xvsrari_w",`.
  **L774 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsrlri_w", "xvsrari_w",`。
- **L775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvbitclri_w", "xvbitseti_w", "xvbitrevi_w",`.
  **L775 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvbitclri_w", "xvbitseti_w", "xvbitrevi_w",`。
- **L776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvseqi_w", "xvslei_w", "xvslei_wu", "xvslti_w", "xvslti_wu",`.
  **L776 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvseqi_w", "xvslei_w", "xvslei_wu", "xvslti_w", "xvslti_wu",`。
- **L777 EN**: Continues the surrounding expression or declaration: `"xvrepl128vei_w", "xvshuf4i_w", "xvpickve_w"] in`.
  **L777 CN**: 继续构造周围的表达式或声明：`"xvrepl128vei_w", "xvshuf4i_w", "xvpickve_w"] in`。
- **L778 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L778 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i32_ty, llvm_i32_ty],`.
  **L779 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i32_ty, llvm_i32_ty],`。
- **L780 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L780 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L781 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L781 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmaxi_d", "xvmaxi_du", "xvmini_d", "xvmini_du",`.
  **L782 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmaxi_d", "xvmaxi_du", "xvmini_d", "xvmini_du",`。
- **L783 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsat_d", "xvsat_du",`.
  **L783 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsat_d", "xvsat_du",`。
- **L784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvslli_d", "xvsrli_d", "xvsrai_d", "xvrotri_d",`.
  **L784 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvslli_d", "xvsrli_d", "xvsrai_d", "xvrotri_d",`。
- **L785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvsrlri_d", "xvsrari_d",`.
  **L785 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvsrlri_d", "xvsrari_d",`。
- **L786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvbitclri_d", "xvbitseti_d", "xvbitrevi_d",`.
  **L786 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvbitclri_d", "xvbitseti_d", "xvbitrevi_d",`。
- **L787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvseqi_d", "xvslei_d", "xvslei_du", "xvslti_d", "xvslti_du",`.
  **L787 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvseqi_d", "xvslei_d", "xvslei_du", "xvslti_d", "xvslti_du",`。
- **L788 EN**: Continues the surrounding expression or declaration: `"xvrepl128vei_d", "xvpermi_d", "xvpickve_d"] in`.
  **L788 CN**: 继续构造周围的表达式或声明：`"xvrepl128vei_d", "xvpermi_d", "xvpickve_d"] in`。
- **L789 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L789 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L790 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i64_ty, llvm_i32_ty],`.
  **L790 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i64_ty, llvm_i32_ty],`。
- **L791 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L791 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L792 EN**: Blank line separating nearby declarations or logic blocks.
  **L792 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 793-816

````tablegen
foreach inst = ["xvhaddw_h_b", "xvhaddw_hu_bu", "xvhsubw_h_b", "xvhsubw_hu_bu",
                "xvaddwev_h_b", "xvaddwod_h_b", "xvsubwev_h_b", "xvsubwod_h_b",
                "xvaddwev_h_bu", "xvaddwod_h_bu", "xvsubwev_h_bu", "xvsubwod_h_bu",
                "xvaddwev_h_bu_b", "xvaddwod_h_bu_b",
                "xvmulwev_h_b", "xvmulwod_h_b", "xvmulwev_h_bu", "xvmulwod_h_bu",
                "xvmulwev_h_bu_b", "xvmulwod_h_bu_b"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v16i16_ty],
                                        [llvm_v32i8_ty, llvm_v32i8_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvhaddw_w_h", "xvhaddw_wu_hu", "xvhsubw_w_h", "xvhsubw_wu_hu",
                "xvaddwev_w_h", "xvaddwod_w_h", "xvsubwev_w_h", "xvsubwod_w_h",
                "xvaddwev_w_hu", "xvaddwod_w_hu", "xvsubwev_w_hu", "xvsubwod_w_hu",
                "xvaddwev_w_hu_h", "xvaddwod_w_hu_h",
                "xvmulwev_w_h", "xvmulwod_w_h", "xvmulwev_w_hu", "xvmulwod_w_hu",
                "xvmulwev_w_hu_h", "xvmulwod_w_hu_h"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8i32_ty],
                                        [llvm_v16i16_ty, llvm_v16i16_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvhaddw_d_w", "xvhaddw_du_wu", "xvhsubw_d_w", "xvhsubw_du_wu",
                "xvaddwev_d_w", "xvaddwod_d_w", "xvsubwev_d_w", "xvsubwod_d_w",
                "xvaddwev_d_wu", "xvaddwod_d_wu", "xvsubwev_d_wu", "xvsubwod_d_wu",
                "xvaddwev_d_wu_w", "xvaddwod_d_wu_w",
````
- **L793 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L793 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L794 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvaddwev_h_b", "xvaddwod_h_b", "xvsubwev_h_b", "xvsubwod_h_b",`.
  **L794 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvaddwev_h_b", "xvaddwod_h_b", "xvsubwev_h_b", "xvsubwod_h_b",`。
- **L795 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvaddwev_h_bu", "xvaddwod_h_bu", "xvsubwev_h_bu", "xvsubwod_h_bu",`.
  **L795 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvaddwev_h_bu", "xvaddwod_h_bu", "xvsubwev_h_bu", "xvsubwod_h_bu",`。
- **L796 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvaddwev_h_bu_b", "xvaddwod_h_bu_b",`.
  **L796 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvaddwev_h_bu_b", "xvaddwod_h_bu_b",`。
- **L797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmulwev_h_b", "xvmulwod_h_b", "xvmulwev_h_bu", "xvmulwod_h_bu",`.
  **L797 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmulwev_h_b", "xvmulwod_h_b", "xvmulwev_h_bu", "xvmulwod_h_bu",`。
- **L798 EN**: Continues the surrounding expression or declaration: `"xvmulwev_h_bu_b", "xvmulwod_h_bu_b"] in`.
  **L798 CN**: 继续构造周围的表达式或声明：`"xvmulwev_h_bu_b", "xvmulwod_h_bu_b"] in`。
- **L799 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L799 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L800 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v32i8_ty, llvm_v32i8_ty],`.
  **L800 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v32i8_ty, llvm_v32i8_ty],`。
- **L801 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L801 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L802 EN**: Blank line separating nearby declarations or logic blocks.
  **L802 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L803 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L803 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvaddwev_w_h", "xvaddwod_w_h", "xvsubwev_w_h", "xvsubwod_w_h",`.
  **L804 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvaddwev_w_h", "xvaddwod_w_h", "xvsubwev_w_h", "xvsubwod_w_h",`。
- **L805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvaddwev_w_hu", "xvaddwod_w_hu", "xvsubwev_w_hu", "xvsubwod_w_hu",`.
  **L805 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvaddwev_w_hu", "xvaddwod_w_hu", "xvsubwev_w_hu", "xvsubwod_w_hu",`。
- **L806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvaddwev_w_hu_h", "xvaddwod_w_hu_h",`.
  **L806 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvaddwev_w_hu_h", "xvaddwod_w_hu_h",`。
- **L807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmulwev_w_h", "xvmulwod_w_h", "xvmulwev_w_hu", "xvmulwod_w_hu",`.
  **L807 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmulwev_w_h", "xvmulwod_w_h", "xvmulwev_w_hu", "xvmulwod_w_hu",`。
- **L808 EN**: Continues the surrounding expression or declaration: `"xvmulwev_w_hu_h", "xvmulwod_w_hu_h"] in`.
  **L808 CN**: 继续构造周围的表达式或声明：`"xvmulwev_w_hu_h", "xvmulwod_w_hu_h"] in`。
- **L809 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L809 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L810 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i16_ty, llvm_v16i16_ty],`.
  **L810 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i16_ty, llvm_v16i16_ty],`。
- **L811 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L811 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L812 EN**: Blank line separating nearby declarations or logic blocks.
  **L812 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L813 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L813 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L814 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvaddwev_d_w", "xvaddwod_d_w", "xvsubwev_d_w", "xvsubwod_d_w",`.
  **L814 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvaddwev_d_w", "xvaddwod_d_w", "xvsubwev_d_w", "xvsubwod_d_w",`。
- **L815 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvaddwev_d_wu", "xvaddwod_d_wu", "xvsubwev_d_wu", "xvsubwod_d_wu",`.
  **L815 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvaddwev_d_wu", "xvaddwod_d_wu", "xvsubwev_d_wu", "xvsubwod_d_wu",`。
- **L816 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvaddwev_d_wu_w", "xvaddwod_d_wu_w",`.
  **L816 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvaddwev_d_wu_w", "xvaddwod_d_wu_w",`。

### Lines 817-840

````tablegen
                "xvmulwev_d_w", "xvmulwod_d_w", "xvmulwev_d_wu", "xvmulwod_d_wu",
                "xvmulwev_d_wu_w", "xvmulwod_d_wu_w"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4i64_ty],
                                        [llvm_v8i32_ty, llvm_v8i32_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvsrln_b_h", "xvsran_b_h", "xvsrlrn_b_h", "xvsrarn_b_h",
                "xvssrln_b_h", "xvssran_b_h", "xvssrln_bu_h", "xvssran_bu_h",
                "xvssrlrn_b_h", "xvssrarn_b_h", "xvssrlrn_bu_h", "xvssrarn_bu_h"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v32i8_ty],
                                        [llvm_v16i16_ty, llvm_v16i16_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvsrln_h_w", "xvsran_h_w", "xvsrlrn_h_w", "xvsrarn_h_w",
                "xvssrln_h_w", "xvssran_h_w", "xvssrln_hu_w", "xvssran_hu_w",
                "xvssrlrn_h_w", "xvssrarn_h_w", "xvssrlrn_hu_w", "xvssrarn_hu_w"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v16i16_ty],
                                        [llvm_v8i32_ty, llvm_v8i32_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvsrln_w_d", "xvsran_w_d", "xvsrlrn_w_d", "xvsrarn_w_d",
                "xvssrln_w_d", "xvssran_w_d", "xvssrln_wu_d", "xvssran_wu_d",
                "xvssrlrn_w_d", "xvssrarn_w_d", "xvssrlrn_wu_d", "xvssrarn_wu_d"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8i32_ty],
````
- **L817 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvmulwev_d_w", "xvmulwod_d_w", "xvmulwev_d_wu", "xvmulwod_d_wu",`.
  **L817 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvmulwev_d_w", "xvmulwod_d_w", "xvmulwev_d_wu", "xvmulwod_d_wu",`。
- **L818 EN**: Continues the surrounding expression or declaration: `"xvmulwev_d_wu_w", "xvmulwod_d_wu_w"] in`.
  **L818 CN**: 继续构造周围的表达式或声明：`"xvmulwev_d_wu_w", "xvmulwod_d_wu_w"] in`。
- **L819 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L819 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L820 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i32_ty, llvm_v8i32_ty],`.
  **L820 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i32_ty, llvm_v8i32_ty],`。
- **L821 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L821 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L823 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L823 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvssrln_b_h", "xvssran_b_h", "xvssrln_bu_h", "xvssran_bu_h",`.
  **L824 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvssrln_b_h", "xvssran_b_h", "xvssrln_bu_h", "xvssran_bu_h",`。
- **L825 EN**: Continues the surrounding expression or declaration: `"xvssrlrn_b_h", "xvssrarn_b_h", "xvssrlrn_bu_h", "xvssrarn_bu_h"] in`.
  **L825 CN**: 继续构造周围的表达式或声明：`"xvssrlrn_b_h", "xvssrarn_b_h", "xvssrlrn_bu_h", "xvssrarn_bu_h"] in`。
- **L826 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L826 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L827 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i16_ty, llvm_v16i16_ty],`.
  **L827 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i16_ty, llvm_v16i16_ty],`。
- **L828 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L828 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L829 EN**: Blank line separating nearby declarations or logic blocks.
  **L829 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L830 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L830 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L831 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvssrln_h_w", "xvssran_h_w", "xvssrln_hu_w", "xvssran_hu_w",`.
  **L831 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvssrln_h_w", "xvssran_h_w", "xvssrln_hu_w", "xvssran_hu_w",`。
- **L832 EN**: Continues the surrounding expression or declaration: `"xvssrlrn_h_w", "xvssrarn_h_w", "xvssrlrn_hu_w", "xvssrarn_hu_w"] in`.
  **L832 CN**: 继续构造周围的表达式或声明：`"xvssrlrn_h_w", "xvssrarn_h_w", "xvssrlrn_hu_w", "xvssrarn_hu_w"] in`。
- **L833 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L833 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L834 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i32_ty, llvm_v8i32_ty],`.
  **L834 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i32_ty, llvm_v8i32_ty],`。
- **L835 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L835 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L836 EN**: Blank line separating nearby declarations or logic blocks.
  **L836 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L837 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L837 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L838 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvssrln_w_d", "xvssran_w_d", "xvssrln_wu_d", "xvssran_wu_d",`.
  **L838 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvssrln_w_d", "xvssran_w_d", "xvssrln_wu_d", "xvssran_wu_d",`。
- **L839 EN**: Continues the surrounding expression or declaration: `"xvssrlrn_w_d", "xvssrarn_w_d", "xvssrlrn_wu_d", "xvssrarn_wu_d"] in`.
  **L839 CN**: 继续构造周围的表达式或声明：`"xvssrlrn_w_d", "xvssrarn_w_d", "xvssrlrn_wu_d", "xvssrarn_wu_d"] in`。
- **L840 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L840 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。

### Lines 841-864

````tablegen
                                        [llvm_v4i64_ty, llvm_v4i64_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvmadd_b", "xvmsub_b", "xvfrstp_b", "xvbitsel_v", "xvshuf_b"] in
  def int_loongarch_lasx_#inst
    : VecInt<[llvm_v32i8_ty],
             [llvm_v32i8_ty, llvm_v32i8_ty, llvm_v32i8_ty],
             [IntrNoMem]>;
foreach inst = ["xvmadd_h", "xvmsub_h", "xvfrstp_h", "xvshuf_h"] in
  def int_loongarch_lasx_#inst
    : VecInt<[llvm_v16i16_ty],
             [llvm_v16i16_ty, llvm_v16i16_ty, llvm_v16i16_ty],
             [IntrNoMem]>;
foreach inst = ["xvmadd_w", "xvmsub_w", "xvshuf_w"] in
  def int_loongarch_lasx_#inst
    : VecInt<[llvm_v8i32_ty],
             [llvm_v8i32_ty, llvm_v8i32_ty, llvm_v8i32_ty],
             [IntrNoMem]>;
foreach inst = ["xvmadd_d", "xvmsub_d", "xvshuf_d"] in
  def int_loongarch_lasx_#inst
    : VecInt<[llvm_v4i64_ty],
             [llvm_v4i64_ty, llvm_v4i64_ty, llvm_v4i64_ty],
             [IntrNoMem]>;

````
- **L841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i64_ty, llvm_v4i64_ty],`.
  **L841 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i64_ty, llvm_v4i64_ty],`。
- **L842 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L842 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L843 EN**: Blank line separating nearby declarations or logic blocks.
  **L843 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L844 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L844 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L845 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L845 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L846 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v32i8_ty],`.
  **L846 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v32i8_ty],`。
- **L847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v32i8_ty, llvm_v32i8_ty, llvm_v32i8_ty],`.
  **L847 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v32i8_ty, llvm_v32i8_ty, llvm_v32i8_ty],`。
- **L848 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L848 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L849 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L849 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L850 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L850 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L851 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v16i16_ty],`.
  **L851 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v16i16_ty],`。
- **L852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i16_ty, llvm_v16i16_ty, llvm_v16i16_ty],`.
  **L852 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i16_ty, llvm_v16i16_ty, llvm_v16i16_ty],`。
- **L853 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L853 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L854 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L854 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L855 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L855 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v8i32_ty],`.
  **L856 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v8i32_ty],`。
- **L857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i32_ty, llvm_v8i32_ty, llvm_v8i32_ty],`.
  **L857 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i32_ty, llvm_v8i32_ty, llvm_v8i32_ty],`。
- **L858 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L858 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L859 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L859 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L860 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L860 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L861 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v4i64_ty],`.
  **L861 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v4i64_ty],`。
- **L862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i64_ty, llvm_v4i64_ty, llvm_v4i64_ty],`.
  **L862 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i64_ty, llvm_v4i64_ty, llvm_v4i64_ty],`。
- **L863 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L863 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L864 EN**: Blank line separating nearby declarations or logic blocks.
  **L864 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 865-888

````tablegen
foreach inst = ["xvsrlni_b_h", "xvsrani_b_h", "xvsrlrni_b_h", "xvsrarni_b_h",
                "xvssrlni_b_h", "xvssrani_b_h", "xvssrlni_bu_h", "xvssrani_bu_h",
                "xvssrlrni_b_h", "xvssrarni_b_h", "xvssrlrni_bu_h", "xvssrarni_bu_h",
                "xvfrstpi_b", "xvbitseli_b", "xvextrins_b", "xvpermi_q"] in
  def int_loongarch_lasx_#inst
    : VecInt<[llvm_v32i8_ty],
             [llvm_v32i8_ty, llvm_v32i8_ty, llvm_i32_ty],
             [IntrNoMem, ImmArg<ArgIndex<2>>]>;
foreach inst = ["xvsrlni_h_w", "xvsrani_h_w", "xvsrlrni_h_w", "xvsrarni_h_w",
                "xvssrlni_h_w", "xvssrani_h_w", "xvssrlni_hu_w", "xvssrani_hu_w",
                "xvssrlrni_h_w", "xvssrarni_h_w", "xvssrlrni_hu_w", "xvssrarni_hu_w",
                "xvfrstpi_h", "xvextrins_h"] in
  def int_loongarch_lasx_#inst
    : VecInt<[llvm_v16i16_ty],
             [llvm_v16i16_ty, llvm_v16i16_ty, llvm_i32_ty],
             [IntrNoMem, ImmArg<ArgIndex<2>>]>;
foreach inst = ["xvsrlni_w_d", "xvsrani_w_d", "xvsrlrni_w_d", "xvsrarni_w_d",
                "xvssrlni_w_d", "xvssrani_w_d", "xvssrlni_wu_d", "xvssrani_wu_d",
                "xvssrlrni_w_d", "xvssrarni_w_d", "xvssrlrni_wu_d", "xvssrarni_wu_d",
                "xvpermi_w", "xvextrins_w", "xvinsve0_w"] in
  def int_loongarch_lasx_#inst
    : VecInt<[llvm_v8i32_ty],
             [llvm_v8i32_ty, llvm_v8i32_ty, llvm_i32_ty],
             [IntrNoMem, ImmArg<ArgIndex<2>>]>;
````
- **L865 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L865 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvssrlni_b_h", "xvssrani_b_h", "xvssrlni_bu_h", "xvssrani_bu_h",`.
  **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvssrlni_b_h", "xvssrani_b_h", "xvssrlni_bu_h", "xvssrani_bu_h",`。
- **L867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvssrlrni_b_h", "xvssrarni_b_h", "xvssrlrni_bu_h", "xvssrarni_bu_h",`.
  **L867 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvssrlrni_b_h", "xvssrarni_b_h", "xvssrlrni_bu_h", "xvssrarni_bu_h",`。
- **L868 EN**: Continues the surrounding expression or declaration: `"xvfrstpi_b", "xvbitseli_b", "xvextrins_b", "xvpermi_q"] in`.
  **L868 CN**: 继续构造周围的表达式或声明：`"xvfrstpi_b", "xvbitseli_b", "xvextrins_b", "xvpermi_q"] in`。
- **L869 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L869 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v32i8_ty],`.
  **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v32i8_ty],`。
- **L871 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v32i8_ty, llvm_v32i8_ty, llvm_i32_ty],`.
  **L871 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v32i8_ty, llvm_v32i8_ty, llvm_i32_ty],`。
- **L872 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L872 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L873 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L873 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvssrlni_h_w", "xvssrani_h_w", "xvssrlni_hu_w", "xvssrani_hu_w",`.
  **L874 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvssrlni_h_w", "xvssrani_h_w", "xvssrlni_hu_w", "xvssrani_hu_w",`。
- **L875 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvssrlrni_h_w", "xvssrarni_h_w", "xvssrlrni_hu_w", "xvssrarni_hu_w",`.
  **L875 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvssrlrni_h_w", "xvssrarni_h_w", "xvssrlrni_hu_w", "xvssrarni_hu_w",`。
- **L876 EN**: Continues the surrounding expression or declaration: `"xvfrstpi_h", "xvextrins_h"] in`.
  **L876 CN**: 继续构造周围的表达式或声明：`"xvfrstpi_h", "xvextrins_h"] in`。
- **L877 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L877 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L878 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v16i16_ty],`.
  **L878 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v16i16_ty],`。
- **L879 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i16_ty, llvm_v16i16_ty, llvm_i32_ty],`.
  **L879 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i16_ty, llvm_v16i16_ty, llvm_i32_ty],`。
- **L880 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L880 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L881 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L881 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L882 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvssrlni_w_d", "xvssrani_w_d", "xvssrlni_wu_d", "xvssrani_wu_d",`.
  **L882 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvssrlni_w_d", "xvssrani_w_d", "xvssrlni_wu_d", "xvssrani_wu_d",`。
- **L883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvssrlrni_w_d", "xvssrarni_w_d", "xvssrlrni_wu_d", "xvssrarni_wu_d",`.
  **L883 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvssrlrni_w_d", "xvssrarni_w_d", "xvssrlrni_wu_d", "xvssrarni_wu_d",`。
- **L884 EN**: Continues the surrounding expression or declaration: `"xvpermi_w", "xvextrins_w", "xvinsve0_w"] in`.
  **L884 CN**: 继续构造周围的表达式或声明：`"xvpermi_w", "xvextrins_w", "xvinsve0_w"] in`。
- **L885 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L885 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L886 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v8i32_ty],`.
  **L886 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v8i32_ty],`。
- **L887 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i32_ty, llvm_v8i32_ty, llvm_i32_ty],`.
  **L887 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i32_ty, llvm_v8i32_ty, llvm_i32_ty],`。
- **L888 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L888 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。

### Lines 889-912

````tablegen
foreach inst = ["xvsrlni_d_q", "xvsrani_d_q", "xvsrlrni_d_q", "xvsrarni_d_q",
                "xvssrlni_d_q", "xvssrani_d_q", "xvssrlni_du_q", "xvssrani_du_q",
                "xvssrlrni_d_q", "xvssrarni_d_q", "xvssrlrni_du_q", "xvssrarni_du_q",
                "xvshuf4i_d", "xvextrins_d", "xvinsve0_d"] in
  def int_loongarch_lasx_#inst
    : VecInt<[llvm_v4i64_ty],
             [llvm_v4i64_ty, llvm_v4i64_ty, llvm_i32_ty],
             [IntrNoMem, ImmArg<ArgIndex<2>>]>;

foreach inst = ["xvmaddwev_h_b", "xvmaddwod_h_b", "xvmaddwev_h_bu",
                "xvmaddwod_h_bu", "xvmaddwev_h_bu_b", "xvmaddwod_h_bu_b"] in
  def int_loongarch_lasx_#inst
    : VecInt<[llvm_v16i16_ty],
             [llvm_v16i16_ty, llvm_v32i8_ty, llvm_v32i8_ty],
             [IntrNoMem]>;
foreach inst = ["xvmaddwev_w_h", "xvmaddwod_w_h", "xvmaddwev_w_hu",
                "xvmaddwod_w_hu", "xvmaddwev_w_hu_h", "xvmaddwod_w_hu_h"] in
  def int_loongarch_lasx_#inst
    : VecInt<[llvm_v8i32_ty],
             [llvm_v8i32_ty, llvm_v16i16_ty, llvm_v16i16_ty],
             [IntrNoMem]>;
foreach inst = ["xvmaddwev_d_w", "xvmaddwod_d_w", "xvmaddwev_d_wu",
                "xvmaddwod_d_wu", "xvmaddwev_d_wu_w", "xvmaddwod_d_wu_w"] in
  def int_loongarch_lasx_#inst
````
- **L889 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L889 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L890 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvssrlni_d_q", "xvssrani_d_q", "xvssrlni_du_q", "xvssrani_du_q",`.
  **L890 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvssrlni_d_q", "xvssrani_d_q", "xvssrlni_du_q", "xvssrani_du_q",`。
- **L891 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvssrlrni_d_q", "xvssrarni_d_q", "xvssrlrni_du_q", "xvssrarni_du_q",`.
  **L891 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvssrlrni_d_q", "xvssrarni_d_q", "xvssrlrni_du_q", "xvssrarni_du_q",`。
- **L892 EN**: Continues the surrounding expression or declaration: `"xvshuf4i_d", "xvextrins_d", "xvinsve0_d"] in`.
  **L892 CN**: 继续构造周围的表达式或声明：`"xvshuf4i_d", "xvextrins_d", "xvinsve0_d"] in`。
- **L893 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L893 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L894 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v4i64_ty],`.
  **L894 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v4i64_ty],`。
- **L895 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i64_ty, llvm_v4i64_ty, llvm_i32_ty],`.
  **L895 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i64_ty, llvm_v4i64_ty, llvm_i32_ty],`。
- **L896 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L896 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L897 EN**: Blank line separating nearby declarations or logic blocks.
  **L897 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L898 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L898 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L899 EN**: Continues the surrounding expression or declaration: `"xvmaddwod_h_bu", "xvmaddwev_h_bu_b", "xvmaddwod_h_bu_b"] in`.
  **L899 CN**: 继续构造周围的表达式或声明：`"xvmaddwod_h_bu", "xvmaddwev_h_bu_b", "xvmaddwod_h_bu_b"] in`。
- **L900 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L900 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L901 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v16i16_ty],`.
  **L901 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v16i16_ty],`。
- **L902 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i16_ty, llvm_v32i8_ty, llvm_v32i8_ty],`.
  **L902 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i16_ty, llvm_v32i8_ty, llvm_v32i8_ty],`。
- **L903 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L903 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L904 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L904 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L905 EN**: Continues the surrounding expression or declaration: `"xvmaddwod_w_hu", "xvmaddwev_w_hu_h", "xvmaddwod_w_hu_h"] in`.
  **L905 CN**: 继续构造周围的表达式或声明：`"xvmaddwod_w_hu", "xvmaddwev_w_hu_h", "xvmaddwod_w_hu_h"] in`。
- **L906 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L906 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L907 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v8i32_ty],`.
  **L907 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v8i32_ty],`。
- **L908 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i32_ty, llvm_v16i16_ty, llvm_v16i16_ty],`.
  **L908 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i32_ty, llvm_v16i16_ty, llvm_v16i16_ty],`。
- **L909 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L909 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L910 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L910 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L911 EN**: Continues the surrounding expression or declaration: `"xvmaddwod_d_wu", "xvmaddwev_d_wu_w", "xvmaddwod_d_wu_w"] in`.
  **L911 CN**: 继续构造周围的表达式或声明：`"xvmaddwod_d_wu", "xvmaddwev_d_wu_w", "xvmaddwod_d_wu_w"] in`。
- **L912 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L912 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。

### Lines 913-936

````tablegen
    : VecInt<[llvm_v4i64_ty],
             [llvm_v4i64_ty, llvm_v8i32_ty, llvm_v8i32_ty],
             [IntrNoMem]>;
foreach inst = ["xvmaddwev_q_d", "xvmaddwod_q_d", "xvmaddwev_q_du",
                "xvmaddwod_q_du", "xvmaddwev_q_du_d", "xvmaddwod_q_du_d"] in
  def int_loongarch_lasx_#inst
    : VecInt<[llvm_v4i64_ty],
             [llvm_v4i64_ty, llvm_v4i64_ty, llvm_v4i64_ty],
             [IntrNoMem]>;

foreach inst = ["xvsllwil_h_b", "xvsllwil_hu_bu"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v16i16_ty],
                                        [llvm_v32i8_ty, llvm_i32_ty],
                                        [IntrNoMem, ImmArg<ArgIndex<1>>]>;
foreach inst = ["xvsllwil_w_h", "xvsllwil_wu_hu"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8i32_ty],
                                        [llvm_v16i16_ty, llvm_i32_ty],
                                        [IntrNoMem, ImmArg<ArgIndex<1>>]>;
foreach inst = ["xvsllwil_d_w", "xvsllwil_du_wu"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4i64_ty],
                                        [llvm_v8i32_ty, llvm_i32_ty],
                                        [IntrNoMem, ImmArg<ArgIndex<1>>]>;

foreach inst = ["xvneg_b", "xvmskltz_b", "xvmskgez_b", "xvmsknz_b",
````
- **L913 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v4i64_ty],`.
  **L913 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v4i64_ty],`。
- **L914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i64_ty, llvm_v8i32_ty, llvm_v8i32_ty],`.
  **L914 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i64_ty, llvm_v8i32_ty, llvm_v8i32_ty],`。
- **L915 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L915 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L916 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L916 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L917 EN**: Continues the surrounding expression or declaration: `"xvmaddwod_q_du", "xvmaddwev_q_du_d", "xvmaddwod_q_du_d"] in`.
  **L917 CN**: 继续构造周围的表达式或声明：`"xvmaddwod_q_du", "xvmaddwev_q_du_d", "xvmaddwod_q_du_d"] in`。
- **L918 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L918 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v4i64_ty],`.
  **L919 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v4i64_ty],`。
- **L920 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i64_ty, llvm_v4i64_ty, llvm_v4i64_ty],`.
  **L920 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i64_ty, llvm_v4i64_ty, llvm_v4i64_ty],`。
- **L921 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L921 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L922 EN**: Blank line separating nearby declarations or logic blocks.
  **L922 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L923 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L923 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L924 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L924 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L925 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v32i8_ty, llvm_i32_ty],`.
  **L925 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v32i8_ty, llvm_i32_ty],`。
- **L926 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L926 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L927 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L927 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L928 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L928 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L929 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i16_ty, llvm_i32_ty],`.
  **L929 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i16_ty, llvm_i32_ty],`。
- **L930 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L930 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L931 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L931 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L932 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L932 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L933 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i32_ty, llvm_i32_ty],`.
  **L933 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i32_ty, llvm_i32_ty],`。
- **L934 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L934 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L935 EN**: Blank line separating nearby declarations or logic blocks.
  **L935 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L936 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L936 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 937-960

````tablegen
                "xvclo_b", "xvclz_b", "xvpcnt_b",
                "xvreplve0_b", "xvreplve0_q"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v32i8_ty], [llvm_v32i8_ty],
                                        [IntrNoMem]>;
foreach inst = ["xvneg_h", "xvmskltz_h", "xvclo_h", "xvclz_h", "xvpcnt_h",
                "xvreplve0_h"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v16i16_ty], [llvm_v16i16_ty],
                                        [IntrNoMem]>;
foreach inst = ["xvneg_w", "xvmskltz_w", "xvclo_w", "xvclz_w", "xvpcnt_w",
                "xvreplve0_w"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8i32_ty], [llvm_v8i32_ty],
                                        [IntrNoMem]>;
foreach inst = ["xvneg_d", "xvexth_q_d", "xvexth_qu_du", "xvmskltz_d",
                "xvextl_q_d", "xvextl_qu_du", "xvclo_d", "xvclz_d", "xvpcnt_d",
                "xvreplve0_d"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4i64_ty], [llvm_v4i64_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvexth_h_b", "xvexth_hu_bu", "vext2xv_h_b", "vext2xv_hu_bu"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v16i16_ty], [llvm_v32i8_ty],
                                        [IntrNoMem]>;
foreach inst = ["xvexth_w_h", "xvexth_wu_hu", "vext2xv_w_h", "vext2xv_wu_hu"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8i32_ty], [llvm_v16i16_ty],
                                        [IntrNoMem]>;
````
- **L937 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvclo_b", "xvclz_b", "xvpcnt_b",`.
  **L937 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvclo_b", "xvclz_b", "xvpcnt_b",`。
- **L938 EN**: Continues the surrounding expression or declaration: `"xvreplve0_b", "xvreplve0_q"] in`.
  **L938 CN**: 继续构造周围的表达式或声明：`"xvreplve0_b", "xvreplve0_q"] in`。
- **L939 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L939 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L940 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L940 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L941 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L941 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L942 EN**: Continues the surrounding expression or declaration: `"xvreplve0_h"] in`.
  **L942 CN**: 继续构造周围的表达式或声明：`"xvreplve0_h"] in`。
- **L943 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L943 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L944 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L944 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L945 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L945 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L946 EN**: Continues the surrounding expression or declaration: `"xvreplve0_w"] in`.
  **L946 CN**: 继续构造周围的表达式或声明：`"xvreplve0_w"] in`。
- **L947 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L947 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L948 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L948 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L949 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L949 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvextl_q_d", "xvextl_qu_du", "xvclo_d", "xvclz_d", "xvpcnt_d",`.
  **L950 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvextl_q_d", "xvextl_qu_du", "xvclo_d", "xvclz_d", "xvpcnt_d",`。
- **L951 EN**: Continues the surrounding expression or declaration: `"xvreplve0_d"] in`.
  **L951 CN**: 继续构造周围的表达式或声明：`"xvreplve0_d"] in`。
- **L952 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L952 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L953 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L953 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L954 EN**: Blank line separating nearby declarations or logic blocks.
  **L954 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L955 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L955 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L956 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L956 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L957 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L957 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L958 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L958 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L959 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L959 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L960 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L960 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 961-984

````tablegen
foreach inst = ["xvexth_d_w", "xvexth_du_wu", "vext2xv_d_w", "vext2xv_du_wu"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4i64_ty], [llvm_v8i32_ty],
                                        [IntrNoMem]>;

foreach inst = ["vext2xv_w_b", "vext2xv_wu_bu"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8i32_ty], [llvm_v32i8_ty],
                                        [IntrNoMem]>;
foreach inst = ["vext2xv_d_h", "vext2xv_du_hu"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4i64_ty], [llvm_v16i16_ty],
                                        [IntrNoMem]>;

foreach inst = ["vext2xv_d_b", "vext2xv_du_bu"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4i64_ty], [llvm_v32i8_ty],
                                        [IntrNoMem]>;

def int_loongarch_lasx_xvldi : VecInt<[llvm_v4i64_ty], [llvm_i32_ty],
                                      [IntrNoMem, ImmArg<ArgIndex<0>>]>;
def int_loongarch_lasx_xvrepli_b : VecInt<[llvm_v32i8_ty], [llvm_i32_ty],
                                          [IntrNoMem, ImmArg<ArgIndex<0>>]>;
def int_loongarch_lasx_xvrepli_h : VecInt<[llvm_v16i16_ty], [llvm_i32_ty],
                                          [IntrNoMem, ImmArg<ArgIndex<0>>]>;
def int_loongarch_lasx_xvrepli_w : VecInt<[llvm_v8i32_ty], [llvm_i32_ty],
                                          [IntrNoMem, ImmArg<ArgIndex<0>>]>;
def int_loongarch_lasx_xvrepli_d : VecInt<[llvm_v4i64_ty], [llvm_i32_ty],
````
- **L961 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L961 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L962 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L962 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L963 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L963 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L964 EN**: Blank line separating nearby declarations or logic blocks.
  **L964 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L965 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L965 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L966 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L966 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L967 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L967 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L968 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L968 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L969 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L969 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L970 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L970 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L971 EN**: Blank line separating nearby declarations or logic blocks.
  **L971 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L972 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L972 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L973 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L973 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L974 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L974 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L975 EN**: Blank line separating nearby declarations or logic blocks.
  **L975 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L976 EN**: Declares TableGen def `int_loongarch_lasx_xvldi`.
  **L976 CN**: 声明 TableGen def `int_loongarch_lasx_xvldi`。
- **L977 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L977 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L978 EN**: Declares TableGen def `int_loongarch_lasx_xvrepli_b`.
  **L978 CN**: 声明 TableGen def `int_loongarch_lasx_xvrepli_b`。
- **L979 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L979 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L980 EN**: Declares TableGen def `int_loongarch_lasx_xvrepli_h`.
  **L980 CN**: 声明 TableGen def `int_loongarch_lasx_xvrepli_h`。
- **L981 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L981 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L982 EN**: Declares TableGen def `int_loongarch_lasx_xvrepli_w`.
  **L982 CN**: 声明 TableGen def `int_loongarch_lasx_xvrepli_w`。
- **L983 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L983 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L984 EN**: Declares TableGen def `int_loongarch_lasx_xvrepli_d`.
  **L984 CN**: 声明 TableGen def `int_loongarch_lasx_xvrepli_d`。

### Lines 985-1008

````tablegen
                                          [IntrNoMem, ImmArg<ArgIndex<0>>]>;

def int_loongarch_lasx_xvreplgr2vr_b : VecInt<[llvm_v32i8_ty], [llvm_i32_ty],
                                             [IntrNoMem]>;
def int_loongarch_lasx_xvreplgr2vr_h : VecInt<[llvm_v16i16_ty], [llvm_i32_ty],
                                             [IntrNoMem]>;
def int_loongarch_lasx_xvreplgr2vr_w : VecInt<[llvm_v8i32_ty], [llvm_i32_ty],
                                             [IntrNoMem]>;
def int_loongarch_lasx_xvreplgr2vr_d : VecInt<[llvm_v4i64_ty], [llvm_i64_ty],
                                             [IntrNoMem]>;

def int_loongarch_lasx_xvinsgr2vr_w
  : VecInt<[llvm_v8i32_ty], [llvm_v8i32_ty, llvm_i32_ty, llvm_i32_ty],
           [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_loongarch_lasx_xvinsgr2vr_d
  : VecInt<[llvm_v4i64_ty], [llvm_v4i64_ty, llvm_i64_ty, llvm_i32_ty],
           [IntrNoMem, ImmArg<ArgIndex<2>>]>;

def int_loongarch_lasx_xvreplve_b
  : VecInt<[llvm_v32i8_ty], [llvm_v32i8_ty, llvm_i32_ty], [IntrNoMem]>;
def int_loongarch_lasx_xvreplve_h
  : VecInt<[llvm_v16i16_ty], [llvm_v16i16_ty, llvm_i32_ty], [IntrNoMem]>;
def int_loongarch_lasx_xvreplve_w
  : VecInt<[llvm_v8i32_ty], [llvm_v8i32_ty, llvm_i32_ty], [IntrNoMem]>;
````
- **L985 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L985 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L986 EN**: Blank line separating nearby declarations or logic blocks.
  **L986 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L987 EN**: Declares TableGen def `int_loongarch_lasx_xvreplgr2vr_b`.
  **L987 CN**: 声明 TableGen def `int_loongarch_lasx_xvreplgr2vr_b`。
- **L988 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L988 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L989 EN**: Declares TableGen def `int_loongarch_lasx_xvreplgr2vr_h`.
  **L989 CN**: 声明 TableGen def `int_loongarch_lasx_xvreplgr2vr_h`。
- **L990 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L990 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L991 EN**: Declares TableGen def `int_loongarch_lasx_xvreplgr2vr_w`.
  **L991 CN**: 声明 TableGen def `int_loongarch_lasx_xvreplgr2vr_w`。
- **L992 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L992 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L993 EN**: Declares TableGen def `int_loongarch_lasx_xvreplgr2vr_d`.
  **L993 CN**: 声明 TableGen def `int_loongarch_lasx_xvreplgr2vr_d`。
- **L994 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L994 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L995 EN**: Blank line separating nearby declarations or logic blocks.
  **L995 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L996 EN**: Declares TableGen def `int_loongarch_lasx_xvinsgr2vr_w`.
  **L996 CN**: 声明 TableGen def `int_loongarch_lasx_xvinsgr2vr_w`。
- **L997 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v8i32_ty], [llvm_v8i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L997 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v8i32_ty], [llvm_v8i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L998 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L998 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L999 EN**: Declares TableGen def `int_loongarch_lasx_xvinsgr2vr_d`.
  **L999 CN**: 声明 TableGen def `int_loongarch_lasx_xvinsgr2vr_d`。
- **L1000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v4i64_ty], [llvm_v4i64_ty, llvm_i64_ty, llvm_i32_ty],`.
  **L1000 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v4i64_ty], [llvm_v4i64_ty, llvm_i64_ty, llvm_i32_ty],`。
- **L1001 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L1001 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L1002 EN**: Blank line separating nearby declarations or logic blocks.
  **L1002 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1003 EN**: Declares TableGen def `int_loongarch_lasx_xvreplve_b`.
  **L1003 CN**: 声明 TableGen def `int_loongarch_lasx_xvreplve_b`。
- **L1004 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v32i8_ty], [llvm_v32i8_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1004 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v32i8_ty], [llvm_v32i8_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1005 EN**: Declares TableGen def `int_loongarch_lasx_xvreplve_h`.
  **L1005 CN**: 声明 TableGen def `int_loongarch_lasx_xvreplve_h`。
- **L1006 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v16i16_ty], [llvm_v16i16_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1006 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v16i16_ty], [llvm_v16i16_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1007 EN**: Declares TableGen def `int_loongarch_lasx_xvreplve_w`.
  **L1007 CN**: 声明 TableGen def `int_loongarch_lasx_xvreplve_w`。
- **L1008 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v8i32_ty], [llvm_v8i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1008 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v8i32_ty], [llvm_v8i32_ty, llvm_i32_ty], [IntrNoMem]>;`。

### Lines 1009-1032

````tablegen
def int_loongarch_lasx_xvreplve_d
  : VecInt<[llvm_v4i64_ty], [llvm_v4i64_ty, llvm_i32_ty], [IntrNoMem]>;

foreach inst = ["xvpickve2gr_w", "xvpickve2gr_wu" ] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_i32_ty],
                                        [llvm_v8i32_ty, llvm_i32_ty],
                                        [IntrNoMem, ImmArg<ArgIndex<1>>]>;
foreach inst = ["xvpickve2gr_d", "xvpickve2gr_du" ] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_i64_ty],
                                        [llvm_v4i64_ty, llvm_i32_ty],
                                        [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_loongarch_lasx_xbz_b : VecInt<[llvm_i32_ty], [llvm_v32i8_ty],
                                      [IntrNoMem]>;
def int_loongarch_lasx_xbz_h : VecInt<[llvm_i32_ty], [llvm_v16i16_ty],
                                      [IntrNoMem]>;
def int_loongarch_lasx_xbz_w : VecInt<[llvm_i32_ty], [llvm_v8i32_ty],
                                      [IntrNoMem]>;
def int_loongarch_lasx_xbz_d : VecInt<[llvm_i32_ty], [llvm_v4i64_ty],
                                      [IntrNoMem]>;
def int_loongarch_lasx_xbz_v : VecInt<[llvm_i32_ty], [llvm_v32i8_ty],
                                      [IntrNoMem]>;

def int_loongarch_lasx_xbnz_v : VecInt<[llvm_i32_ty], [llvm_v32i8_ty],
````
- **L1009 EN**: Declares TableGen def `int_loongarch_lasx_xvreplve_d`.
  **L1009 CN**: 声明 TableGen def `int_loongarch_lasx_xvreplve_d`。
- **L1010 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v4i64_ty], [llvm_v4i64_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1010 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v4i64_ty], [llvm_v4i64_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1011 EN**: Blank line separating nearby declarations or logic blocks.
  **L1011 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1012 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1012 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1013 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1013 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1014 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i32_ty, llvm_i32_ty],`.
  **L1014 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i32_ty, llvm_i32_ty],`。
- **L1015 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1015 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1016 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1016 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1017 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1017 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1018 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i64_ty, llvm_i32_ty],`.
  **L1018 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i64_ty, llvm_i32_ty],`。
- **L1019 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1019 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1020 EN**: Blank line separating nearby declarations or logic blocks.
  **L1020 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1021 EN**: Declares TableGen def `int_loongarch_lasx_xbz_b`.
  **L1021 CN**: 声明 TableGen def `int_loongarch_lasx_xbz_b`。
- **L1022 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1022 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1023 EN**: Declares TableGen def `int_loongarch_lasx_xbz_h`.
  **L1023 CN**: 声明 TableGen def `int_loongarch_lasx_xbz_h`。
- **L1024 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1024 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1025 EN**: Declares TableGen def `int_loongarch_lasx_xbz_w`.
  **L1025 CN**: 声明 TableGen def `int_loongarch_lasx_xbz_w`。
- **L1026 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1026 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1027 EN**: Declares TableGen def `int_loongarch_lasx_xbz_d`.
  **L1027 CN**: 声明 TableGen def `int_loongarch_lasx_xbz_d`。
- **L1028 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1028 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1029 EN**: Declares TableGen def `int_loongarch_lasx_xbz_v`.
  **L1029 CN**: 声明 TableGen def `int_loongarch_lasx_xbz_v`。
- **L1030 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1030 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1031 EN**: Blank line separating nearby declarations or logic blocks.
  **L1031 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1032 EN**: Declares TableGen def `int_loongarch_lasx_xbnz_v`.
  **L1032 CN**: 声明 TableGen def `int_loongarch_lasx_xbnz_v`。

### Lines 1033-1056

````tablegen
                                       [IntrNoMem]>;
def int_loongarch_lasx_xbnz_b : VecInt<[llvm_i32_ty], [llvm_v32i8_ty],
                                       [IntrNoMem]>;
def int_loongarch_lasx_xbnz_h : VecInt<[llvm_i32_ty], [llvm_v16i16_ty],
                                       [IntrNoMem]>;
def int_loongarch_lasx_xbnz_w : VecInt<[llvm_i32_ty], [llvm_v8i32_ty],
                                       [IntrNoMem]>;
def int_loongarch_lasx_xbnz_d : VecInt<[llvm_i32_ty], [llvm_v4i64_ty],
                                       [IntrNoMem]>;

// LASX Float

foreach inst = ["xvfadd_s", "xvfsub_s", "xvfmul_s", "xvfdiv_s",
                "xvfmax_s", "xvfmin_s", "xvfmaxa_s", "xvfmina_s"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8f32_ty],
                                        [llvm_v8f32_ty, llvm_v8f32_ty],
                                        [IntrNoMem]>;
foreach inst = ["xvfadd_d", "xvfsub_d", "xvfmul_d", "xvfdiv_d",
                "xvfmax_d", "xvfmin_d", "xvfmaxa_d", "xvfmina_d"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4f64_ty],
                                        [llvm_v4f64_ty, llvm_v4f64_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvfmadd_s", "xvfmsub_s", "xvfnmadd_s", "xvfnmsub_s"] in
````
- **L1033 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1033 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1034 EN**: Declares TableGen def `int_loongarch_lasx_xbnz_b`.
  **L1034 CN**: 声明 TableGen def `int_loongarch_lasx_xbnz_b`。
- **L1035 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1035 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1036 EN**: Declares TableGen def `int_loongarch_lasx_xbnz_h`.
  **L1036 CN**: 声明 TableGen def `int_loongarch_lasx_xbnz_h`。
- **L1037 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1037 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1038 EN**: Declares TableGen def `int_loongarch_lasx_xbnz_w`.
  **L1038 CN**: 声明 TableGen def `int_loongarch_lasx_xbnz_w`。
- **L1039 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1039 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1040 EN**: Declares TableGen def `int_loongarch_lasx_xbnz_d`.
  **L1040 CN**: 声明 TableGen def `int_loongarch_lasx_xbnz_d`。
- **L1041 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1041 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1042 EN**: Blank line separating nearby declarations or logic blocks.
  **L1042 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1043 EN**: Comment explains nearby logic, invariants, or intent: `LASX Float`.
  **L1043 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`LASX Float`。
- **L1044 EN**: Blank line separating nearby declarations or logic blocks.
  **L1044 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1045 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1045 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1046 EN**: Continues the surrounding expression or declaration: `"xvfmax_s", "xvfmin_s", "xvfmaxa_s", "xvfmina_s"] in`.
  **L1046 CN**: 继续构造周围的表达式或声明：`"xvfmax_s", "xvfmin_s", "xvfmaxa_s", "xvfmina_s"] in`。
- **L1047 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1047 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8f32_ty, llvm_v8f32_ty],`.
  **L1048 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8f32_ty, llvm_v8f32_ty],`。
- **L1049 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1049 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1050 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1050 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1051 EN**: Continues the surrounding expression or declaration: `"xvfmax_d", "xvfmin_d", "xvfmaxa_d", "xvfmina_d"] in`.
  **L1051 CN**: 继续构造周围的表达式或声明：`"xvfmax_d", "xvfmin_d", "xvfmaxa_d", "xvfmina_d"] in`。
- **L1052 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1052 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f64_ty, llvm_v4f64_ty],`.
  **L1053 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f64_ty, llvm_v4f64_ty],`。
- **L1054 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1054 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1055 EN**: Blank line separating nearby declarations or logic blocks.
  **L1055 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1056 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1056 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1057-1080

````tablegen
  def int_loongarch_lasx_#inst
    : VecInt<[llvm_v8f32_ty],
             [llvm_v8f32_ty, llvm_v8f32_ty, llvm_v8f32_ty],
             [IntrNoMem]>;
foreach inst = ["xvfmadd_d", "xvfmsub_d", "xvfnmadd_d", "xvfnmsub_d"] in
  def int_loongarch_lasx_#inst
    : VecInt<[llvm_v4f64_ty],
             [llvm_v4f64_ty, llvm_v4f64_ty, llvm_v4f64_ty],
             [IntrNoMem]>;

foreach inst = ["xvflogb_s", "xvfsqrt_s", "xvfrecip_s", "xvfrsqrt_s", "xvfrint_s",
                "xvfrecipe_s", "xvfrsqrte_s",
                "xvfrintrne_s", "xvfrintrz_s", "xvfrintrp_s", "xvfrintrm_s"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8f32_ty], [llvm_v8f32_ty],
                                        [IntrNoMem]>;
foreach inst = ["xvflogb_d", "xvfsqrt_d", "xvfrecip_d", "xvfrsqrt_d", "xvfrint_d",
                "xvfrecipe_d", "xvfrsqrte_d",
                "xvfrintrne_d", "xvfrintrz_d", "xvfrintrp_d", "xvfrintrm_d"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4f64_ty], [llvm_v4f64_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvfcvtl_s_h", "xvfcvth_s_h"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8f32_ty], [llvm_v16i16_ty],
                                        [IntrNoMem]>;
````
- **L1057 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1057 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1058 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v8f32_ty],`.
  **L1058 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v8f32_ty],`。
- **L1059 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8f32_ty, llvm_v8f32_ty, llvm_v8f32_ty],`.
  **L1059 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8f32_ty, llvm_v8f32_ty, llvm_v8f32_ty],`。
- **L1060 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1060 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1061 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1061 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1062 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1062 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1063 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v4f64_ty],`.
  **L1063 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v4f64_ty],`。
- **L1064 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f64_ty, llvm_v4f64_ty, llvm_v4f64_ty],`.
  **L1064 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f64_ty, llvm_v4f64_ty, llvm_v4f64_ty],`。
- **L1065 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1065 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1066 EN**: Blank line separating nearby declarations or logic blocks.
  **L1066 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1067 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1067 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1068 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvfrecipe_s", "xvfrsqrte_s",`.
  **L1068 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvfrecipe_s", "xvfrsqrte_s",`。
- **L1069 EN**: Continues the surrounding expression or declaration: `"xvfrintrne_s", "xvfrintrz_s", "xvfrintrp_s", "xvfrintrm_s"] in`.
  **L1069 CN**: 继续构造周围的表达式或声明：`"xvfrintrne_s", "xvfrintrz_s", "xvfrintrp_s", "xvfrintrm_s"] in`。
- **L1070 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1070 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1071 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1071 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1072 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1072 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvfrecipe_d", "xvfrsqrte_d",`.
  **L1073 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvfrecipe_d", "xvfrsqrte_d",`。
- **L1074 EN**: Continues the surrounding expression or declaration: `"xvfrintrne_d", "xvfrintrz_d", "xvfrintrp_d", "xvfrintrm_d"] in`.
  **L1074 CN**: 继续构造周围的表达式或声明：`"xvfrintrne_d", "xvfrintrz_d", "xvfrintrp_d", "xvfrintrm_d"] in`。
- **L1075 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1075 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1076 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1076 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1077 EN**: Blank line separating nearby declarations or logic blocks.
  **L1077 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1078 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1078 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1079 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1079 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1080 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1080 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 1081-1104

````tablegen
foreach inst = ["xvfcvtl_d_s", "xvfcvth_d_s"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4f64_ty], [llvm_v8f32_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvftintrne_w_s", "xvftintrz_w_s", "xvftintrp_w_s", "xvftintrm_w_s",
                "xvftint_w_s", "xvftintrz_wu_s", "xvftint_wu_s", "xvfclass_s"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8i32_ty], [llvm_v8f32_ty],
                                        [IntrNoMem]>;
foreach inst = ["xvftintrne_l_d", "xvftintrz_l_d", "xvftintrp_l_d", "xvftintrm_l_d",
                "xvftint_l_d", "xvftintrz_lu_d", "xvftint_lu_d", "xvfclass_d"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4i64_ty], [llvm_v4f64_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvftintrnel_l_s", "xvftintrneh_l_s", "xvftintrzl_l_s",
                "xvftintrzh_l_s", "xvftintrpl_l_s", "xvftintrph_l_s",
                "xvftintrml_l_s", "xvftintrmh_l_s", "xvftintl_l_s",
                "xvftinth_l_s"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4i64_ty], [llvm_v8f32_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvffint_s_w", "xvffint_s_wu"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8f32_ty], [llvm_v8i32_ty],
                                        [IntrNoMem]>;
foreach inst = ["xvffint_d_l", "xvffint_d_lu"] in
````
- **L1081 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1081 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1082 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1082 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1083 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1083 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1084 EN**: Blank line separating nearby declarations or logic blocks.
  **L1084 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1085 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1085 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1086 EN**: Continues the surrounding expression or declaration: `"xvftint_w_s", "xvftintrz_wu_s", "xvftint_wu_s", "xvfclass_s"] in`.
  **L1086 CN**: 继续构造周围的表达式或声明：`"xvftint_w_s", "xvftintrz_wu_s", "xvftint_wu_s", "xvfclass_s"] in`。
- **L1087 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1087 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1088 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1088 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1089 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1089 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1090 EN**: Continues the surrounding expression or declaration: `"xvftint_l_d", "xvftintrz_lu_d", "xvftint_lu_d", "xvfclass_d"] in`.
  **L1090 CN**: 继续构造周围的表达式或声明：`"xvftint_l_d", "xvftintrz_lu_d", "xvftint_lu_d", "xvfclass_d"] in`。
- **L1091 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1091 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1092 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1092 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1093 EN**: Blank line separating nearby declarations or logic blocks.
  **L1093 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1094 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1094 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1095 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvftintrzh_l_s", "xvftintrpl_l_s", "xvftintrph_l_s",`.
  **L1095 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvftintrzh_l_s", "xvftintrpl_l_s", "xvftintrph_l_s",`。
- **L1096 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvftintrml_l_s", "xvftintrmh_l_s", "xvftintl_l_s",`.
  **L1096 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvftintrml_l_s", "xvftintrmh_l_s", "xvftintl_l_s",`。
- **L1097 EN**: Continues the surrounding expression or declaration: `"xvftinth_l_s"] in`.
  **L1097 CN**: 继续构造周围的表达式或声明：`"xvftinth_l_s"] in`。
- **L1098 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1098 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1099 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1099 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1100 EN**: Blank line separating nearby declarations or logic blocks.
  **L1100 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1101 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1101 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1102 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1102 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1103 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1103 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1104 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1104 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1105-1128

````tablegen
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4f64_ty], [llvm_v4i64_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvffintl_d_w", "xvffinth_d_w"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4f64_ty], [llvm_v8i32_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvffint_s_l"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8f32_ty],
                                        [llvm_v4i64_ty, llvm_v4i64_ty],
                                        [IntrNoMem]>;
foreach inst = ["xvftintrne_w_d", "xvftintrz_w_d", "xvftintrp_w_d", "xvftintrm_w_d",
                "xvftint_w_d"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8i32_ty],
                                        [llvm_v4f64_ty, llvm_v4f64_ty],
                                        [IntrNoMem]>;

foreach inst = ["xvfcvt_h_s"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v16i16_ty],
                                        [llvm_v8f32_ty, llvm_v8f32_ty],
                                        [IntrNoMem]>;
foreach inst = ["xvfcvt_s_d"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8f32_ty],
                                        [llvm_v4f64_ty, llvm_v4f64_ty],
````
- **L1105 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1105 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1106 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1106 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1107 EN**: Blank line separating nearby declarations or logic blocks.
  **L1107 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1108 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1108 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1109 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1109 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1110 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1110 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1111 EN**: Blank line separating nearby declarations or logic blocks.
  **L1111 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1112 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1112 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1113 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1113 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i64_ty, llvm_v4i64_ty],`.
  **L1114 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i64_ty, llvm_v4i64_ty],`。
- **L1115 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1115 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1116 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1116 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1117 EN**: Continues the surrounding expression or declaration: `"xvftint_w_d"] in`.
  **L1117 CN**: 继续构造周围的表达式或声明：`"xvftint_w_d"] in`。
- **L1118 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1118 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f64_ty, llvm_v4f64_ty],`.
  **L1119 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f64_ty, llvm_v4f64_ty],`。
- **L1120 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1120 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1121 EN**: Blank line separating nearby declarations or logic blocks.
  **L1121 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1122 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1122 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1123 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1123 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8f32_ty, llvm_v8f32_ty],`.
  **L1124 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8f32_ty, llvm_v8f32_ty],`。
- **L1125 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1125 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1126 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1126 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1127 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1127 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f64_ty, llvm_v4f64_ty],`.
  **L1128 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f64_ty, llvm_v4f64_ty],`。

### Lines 1129-1152

````tablegen
                                        [IntrNoMem]>;

foreach inst = ["xvfcmp_caf_s", "xvfcmp_cun_s", "xvfcmp_ceq_s", "xvfcmp_cueq_s",
                "xvfcmp_clt_s", "xvfcmp_cult_s", "xvfcmp_cle_s", "xvfcmp_cule_s",
                "xvfcmp_cne_s", "xvfcmp_cor_s", "xvfcmp_cune_s",
                "xvfcmp_saf_s", "xvfcmp_sun_s", "xvfcmp_seq_s", "xvfcmp_sueq_s",
                "xvfcmp_slt_s", "xvfcmp_sult_s", "xvfcmp_sle_s", "xvfcmp_sule_s",
                "xvfcmp_sne_s", "xvfcmp_sor_s", "xvfcmp_sune_s"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v8i32_ty],
                                        [llvm_v8f32_ty, llvm_v8f32_ty],
                                        [IntrNoMem]>;
foreach inst = ["xvfcmp_caf_d", "xvfcmp_cun_d", "xvfcmp_ceq_d", "xvfcmp_cueq_d",
                "xvfcmp_clt_d", "xvfcmp_cult_d", "xvfcmp_cle_d", "xvfcmp_cule_d",
                "xvfcmp_cne_d", "xvfcmp_cor_d", "xvfcmp_cune_d",
                "xvfcmp_saf_d", "xvfcmp_sun_d", "xvfcmp_seq_d", "xvfcmp_sueq_d",
                "xvfcmp_slt_d", "xvfcmp_sult_d", "xvfcmp_sle_d", "xvfcmp_sule_d",
                "xvfcmp_sne_d", "xvfcmp_sor_d", "xvfcmp_sune_d"] in
  def int_loongarch_lasx_#inst : VecInt<[llvm_v4i64_ty],
                                        [llvm_v4f64_ty, llvm_v4f64_ty],
                                        [IntrNoMem]>;

def int_loongarch_lasx_xvpickve_w_f
  : VecInt<[llvm_v8f32_ty], [llvm_v8f32_ty, llvm_i32_ty],
           [IntrNoMem, ImmArg<ArgIndex<1>>]>;
````
- **L1129 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1129 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1130 EN**: Blank line separating nearby declarations or logic blocks.
  **L1130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1131 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1131 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvfcmp_clt_s", "xvfcmp_cult_s", "xvfcmp_cle_s", "xvfcmp_cule_s",`.
  **L1132 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvfcmp_clt_s", "xvfcmp_cult_s", "xvfcmp_cle_s", "xvfcmp_cule_s",`。
- **L1133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvfcmp_cne_s", "xvfcmp_cor_s", "xvfcmp_cune_s",`.
  **L1133 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvfcmp_cne_s", "xvfcmp_cor_s", "xvfcmp_cune_s",`。
- **L1134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvfcmp_saf_s", "xvfcmp_sun_s", "xvfcmp_seq_s", "xvfcmp_sueq_s",`.
  **L1134 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvfcmp_saf_s", "xvfcmp_sun_s", "xvfcmp_seq_s", "xvfcmp_sueq_s",`。
- **L1135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvfcmp_slt_s", "xvfcmp_sult_s", "xvfcmp_sle_s", "xvfcmp_sule_s",`.
  **L1135 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvfcmp_slt_s", "xvfcmp_sult_s", "xvfcmp_sle_s", "xvfcmp_sule_s",`。
- **L1136 EN**: Continues the surrounding expression or declaration: `"xvfcmp_sne_s", "xvfcmp_sor_s", "xvfcmp_sune_s"] in`.
  **L1136 CN**: 继续构造周围的表达式或声明：`"xvfcmp_sne_s", "xvfcmp_sor_s", "xvfcmp_sune_s"] in`。
- **L1137 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1137 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8f32_ty, llvm_v8f32_ty],`.
  **L1138 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8f32_ty, llvm_v8f32_ty],`。
- **L1139 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1139 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1140 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1140 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvfcmp_clt_d", "xvfcmp_cult_d", "xvfcmp_cle_d", "xvfcmp_cule_d",`.
  **L1141 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvfcmp_clt_d", "xvfcmp_cult_d", "xvfcmp_cle_d", "xvfcmp_cule_d",`。
- **L1142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvfcmp_cne_d", "xvfcmp_cor_d", "xvfcmp_cune_d",`.
  **L1142 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvfcmp_cne_d", "xvfcmp_cor_d", "xvfcmp_cune_d",`。
- **L1143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvfcmp_saf_d", "xvfcmp_sun_d", "xvfcmp_seq_d", "xvfcmp_sueq_d",`.
  **L1143 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvfcmp_saf_d", "xvfcmp_sun_d", "xvfcmp_seq_d", "xvfcmp_sueq_d",`。
- **L1144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"xvfcmp_slt_d", "xvfcmp_sult_d", "xvfcmp_sle_d", "xvfcmp_sule_d",`.
  **L1144 CN**: 继续一个多行参数列表、初始化器或聚合项：`"xvfcmp_slt_d", "xvfcmp_sult_d", "xvfcmp_sle_d", "xvfcmp_sule_d",`。
- **L1145 EN**: Continues the surrounding expression or declaration: `"xvfcmp_sne_d", "xvfcmp_sor_d", "xvfcmp_sune_d"] in`.
  **L1145 CN**: 继续构造周围的表达式或声明：`"xvfcmp_sne_d", "xvfcmp_sor_d", "xvfcmp_sune_d"] in`。
- **L1146 EN**: Declares TableGen def `int_loongarch_lasx_#inst`.
  **L1146 CN**: 声明 TableGen def `int_loongarch_lasx_#inst`。
- **L1147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f64_ty, llvm_v4f64_ty],`.
  **L1147 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f64_ty, llvm_v4f64_ty],`。
- **L1148 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1148 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1149 EN**: Blank line separating nearby declarations or logic blocks.
  **L1149 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1150 EN**: Declares TableGen def `int_loongarch_lasx_xvpickve_w_f`.
  **L1150 CN**: 声明 TableGen def `int_loongarch_lasx_xvpickve_w_f`。
- **L1151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v8f32_ty], [llvm_v8f32_ty, llvm_i32_ty],`.
  **L1151 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v8f32_ty], [llvm_v8f32_ty, llvm_i32_ty],`。
- **L1152 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1152 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。

### Lines 1153-1176

````tablegen
def int_loongarch_lasx_xvpickve_d_f
  : VecInt<[llvm_v4f64_ty], [llvm_v4f64_ty, llvm_i32_ty],
           [IntrNoMem, ImmArg<ArgIndex<1>>]>;

// LASX load/store
def int_loongarch_lasx_xvld
  : DefaultAttrsVecInt<[llvm_v32i8_ty], [llvm_ptr_ty, llvm_i32_ty],
           [IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;
def int_loongarch_lasx_xvldx
  : DefaultAttrsVecInt<[llvm_v32i8_ty], [llvm_ptr_ty, llvm_i64_ty],
           [IntrReadMem, IntrArgMemOnly]>;
def int_loongarch_lasx_xvldrepl_b
  : DefaultAttrsVecInt<[llvm_v32i8_ty], [llvm_ptr_ty, llvm_i32_ty],
           [IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;
def int_loongarch_lasx_xvldrepl_h
  : DefaultAttrsVecInt<[llvm_v16i16_ty], [llvm_ptr_ty, llvm_i32_ty],
           [IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;
def int_loongarch_lasx_xvldrepl_w
  : DefaultAttrsVecInt<[llvm_v8i32_ty], [llvm_ptr_ty, llvm_i32_ty],
           [IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;
def int_loongarch_lasx_xvldrepl_d
  : DefaultAttrsVecInt<[llvm_v4i64_ty], [llvm_ptr_ty, llvm_i32_ty],
           [IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;

````
- **L1153 EN**: Declares TableGen def `int_loongarch_lasx_xvpickve_d_f`.
  **L1153 CN**: 声明 TableGen def `int_loongarch_lasx_xvpickve_d_f`。
- **L1154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[llvm_v4f64_ty], [llvm_v4f64_ty, llvm_i32_ty],`.
  **L1154 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[llvm_v4f64_ty], [llvm_v4f64_ty, llvm_i32_ty],`。
- **L1155 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1155 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1156 EN**: Blank line separating nearby declarations or logic blocks.
  **L1156 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1157 EN**: Comment explains nearby logic, invariants, or intent: `LASX load/store`.
  **L1157 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`LASX load/store`。
- **L1158 EN**: Declares TableGen def `int_loongarch_lasx_xvld`.
  **L1158 CN**: 声明 TableGen def `int_loongarch_lasx_xvld`。
- **L1159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsVecInt<[llvm_v32i8_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L1159 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsVecInt<[llvm_v32i8_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L1160 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`.
  **L1160 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`。
- **L1161 EN**: Declares TableGen def `int_loongarch_lasx_xvldx`.
  **L1161 CN**: 声明 TableGen def `int_loongarch_lasx_xvldx`。
- **L1162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsVecInt<[llvm_v32i8_ty], [llvm_ptr_ty, llvm_i64_ty],`.
  **L1162 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsVecInt<[llvm_v32i8_ty], [llvm_ptr_ty, llvm_i64_ty],`。
- **L1163 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1163 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1164 EN**: Declares TableGen def `int_loongarch_lasx_xvldrepl_b`.
  **L1164 CN**: 声明 TableGen def `int_loongarch_lasx_xvldrepl_b`。
- **L1165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsVecInt<[llvm_v32i8_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L1165 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsVecInt<[llvm_v32i8_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L1166 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`.
  **L1166 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`。
- **L1167 EN**: Declares TableGen def `int_loongarch_lasx_xvldrepl_h`.
  **L1167 CN**: 声明 TableGen def `int_loongarch_lasx_xvldrepl_h`。
- **L1168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsVecInt<[llvm_v16i16_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L1168 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsVecInt<[llvm_v16i16_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L1169 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`.
  **L1169 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`。
- **L1170 EN**: Declares TableGen def `int_loongarch_lasx_xvldrepl_w`.
  **L1170 CN**: 声明 TableGen def `int_loongarch_lasx_xvldrepl_w`。
- **L1171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsVecInt<[llvm_v8i32_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L1171 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsVecInt<[llvm_v8i32_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L1172 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`.
  **L1172 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`。
- **L1173 EN**: Declares TableGen def `int_loongarch_lasx_xvldrepl_d`.
  **L1173 CN**: 声明 TableGen def `int_loongarch_lasx_xvldrepl_d`。
- **L1174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsVecInt<[llvm_v4i64_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L1174 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsVecInt<[llvm_v4i64_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L1175 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`.
  **L1175 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`。
- **L1176 EN**: Blank line separating nearby declarations or logic blocks.
  **L1176 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1177-1200

````tablegen
def int_loongarch_lasx_xvst
  : VecInt<[], [llvm_v32i8_ty, llvm_ptr_ty, llvm_i32_ty],
           [IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;
def int_loongarch_lasx_xvstx
  : VecInt<[], [llvm_v32i8_ty, llvm_ptr_ty, llvm_i64_ty],
           [IntrWriteMem, IntrArgMemOnly]>;
def int_loongarch_lasx_xvstelm_b
  : VecInt<[], [llvm_v32i8_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],
           [IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;
def int_loongarch_lasx_xvstelm_h
  : VecInt<[], [llvm_v16i16_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],
           [IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;
def int_loongarch_lasx_xvstelm_w
  : VecInt<[], [llvm_v8i32_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],
           [IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;
def int_loongarch_lasx_xvstelm_d
  : VecInt<[], [llvm_v4i64_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],
           [IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;

// LASX and LSX conversion
def int_loongarch_lasx_cast_128_s
  : VecInt<[llvm_v8f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_loongarch_lasx_cast_128_d
  : VecInt<[llvm_v4f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;
````
- **L1177 EN**: Declares TableGen def `int_loongarch_lasx_xvst`.
  **L1177 CN**: 声明 TableGen def `int_loongarch_lasx_xvst`。
- **L1178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[], [llvm_v32i8_ty, llvm_ptr_ty, llvm_i32_ty],`.
  **L1178 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[], [llvm_v32i8_ty, llvm_ptr_ty, llvm_i32_ty],`。
- **L1179 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`.
  **L1179 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`。
- **L1180 EN**: Declares TableGen def `int_loongarch_lasx_xvstx`.
  **L1180 CN**: 声明 TableGen def `int_loongarch_lasx_xvstx`。
- **L1181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[], [llvm_v32i8_ty, llvm_ptr_ty, llvm_i64_ty],`.
  **L1181 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[], [llvm_v32i8_ty, llvm_ptr_ty, llvm_i64_ty],`。
- **L1182 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L1182 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L1183 EN**: Declares TableGen def `int_loongarch_lasx_xvstelm_b`.
  **L1183 CN**: 声明 TableGen def `int_loongarch_lasx_xvstelm_b`。
- **L1184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[], [llvm_v32i8_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L1184 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[], [llvm_v32i8_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L1185 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`.
  **L1185 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`。
- **L1186 EN**: Declares TableGen def `int_loongarch_lasx_xvstelm_h`.
  **L1186 CN**: 声明 TableGen def `int_loongarch_lasx_xvstelm_h`。
- **L1187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[], [llvm_v16i16_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L1187 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[], [llvm_v16i16_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L1188 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`.
  **L1188 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`。
- **L1189 EN**: Declares TableGen def `int_loongarch_lasx_xvstelm_w`.
  **L1189 CN**: 声明 TableGen def `int_loongarch_lasx_xvstelm_w`。
- **L1190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[], [llvm_v8i32_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L1190 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[], [llvm_v8i32_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L1191 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`.
  **L1191 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`。
- **L1192 EN**: Declares TableGen def `int_loongarch_lasx_xvstelm_d`.
  **L1192 CN**: 声明 TableGen def `int_loongarch_lasx_xvstelm_d`。
- **L1193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: VecInt<[], [llvm_v4i64_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L1193 CN**: 继续一个多行参数列表、初始化器或聚合项：`: VecInt<[], [llvm_v4i64_ty, llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L1194 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`.
  **L1194 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`。
- **L1195 EN**: Blank line separating nearby declarations or logic blocks.
  **L1195 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1196 EN**: Comment explains nearby logic, invariants, or intent: `LASX and LSX conversion`.
  **L1196 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`LASX and LSX conversion`。
- **L1197 EN**: Declares TableGen def `int_loongarch_lasx_cast_128_s`.
  **L1197 CN**: 声明 TableGen def `int_loongarch_lasx_cast_128_s`。
- **L1198 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v8f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1198 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v8f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1199 EN**: Declares TableGen def `int_loongarch_lasx_cast_128_d`.
  **L1199 CN**: 声明 TableGen def `int_loongarch_lasx_cast_128_d`。
- **L1200 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v4f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1200 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v4f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。

### Lines 1201-1224

````tablegen
def int_loongarch_lasx_cast_128
  : VecInt<[llvm_v4i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;
def int_loongarch_lasx_concat_128_s
  : VecInt<[llvm_v8f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_loongarch_lasx_concat_128_d
  : VecInt<[llvm_v4f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;
def int_loongarch_lasx_concat_128
  : VecInt<[llvm_v4i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;
def int_loongarch_lasx_extract_128_lo_s
  : VecInt<[llvm_v4f32_ty], [llvm_v8f32_ty], [IntrNoMem]>;
def int_loongarch_lasx_extract_128_lo_d
  : VecInt<[llvm_v2f64_ty], [llvm_v4f64_ty], [IntrNoMem]>;
def int_loongarch_lasx_extract_128_lo
  : VecInt<[llvm_v2i64_ty], [llvm_v4i64_ty], [IntrNoMem]>;
def int_loongarch_lasx_extract_128_hi_s
  : VecInt<[llvm_v4f32_ty], [llvm_v8f32_ty], [IntrNoMem]>;
def int_loongarch_lasx_extract_128_hi_d
  : VecInt<[llvm_v2f64_ty], [llvm_v4f64_ty], [IntrNoMem]>;
def int_loongarch_lasx_extract_128_hi
  : VecInt<[llvm_v2i64_ty], [llvm_v4i64_ty], [IntrNoMem]>;
def int_loongarch_lasx_insert_128_lo_s
  : VecInt<[llvm_v8f32_ty], [llvm_v8f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_loongarch_lasx_insert_128_lo_d
  : VecInt<[llvm_v4f64_ty], [llvm_v4f64_ty, llvm_v2f64_ty], [IntrNoMem]>;
````
- **L1201 EN**: Declares TableGen def `int_loongarch_lasx_cast_128`.
  **L1201 CN**: 声明 TableGen def `int_loongarch_lasx_cast_128`。
- **L1202 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v4i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1202 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v4i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1203 EN**: Declares TableGen def `int_loongarch_lasx_concat_128_s`.
  **L1203 CN**: 声明 TableGen def `int_loongarch_lasx_concat_128_s`。
- **L1204 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v8f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1204 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v8f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1205 EN**: Declares TableGen def `int_loongarch_lasx_concat_128_d`.
  **L1205 CN**: 声明 TableGen def `int_loongarch_lasx_concat_128_d`。
- **L1206 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v4f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1206 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v4f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1207 EN**: Declares TableGen def `int_loongarch_lasx_concat_128`.
  **L1207 CN**: 声明 TableGen def `int_loongarch_lasx_concat_128`。
- **L1208 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v4i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1208 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v4i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1209 EN**: Declares TableGen def `int_loongarch_lasx_extract_128_lo_s`.
  **L1209 CN**: 声明 TableGen def `int_loongarch_lasx_extract_128_lo_s`。
- **L1210 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v4f32_ty], [llvm_v8f32_ty], [IntrNoMem]>;`.
  **L1210 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v4f32_ty], [llvm_v8f32_ty], [IntrNoMem]>;`。
- **L1211 EN**: Declares TableGen def `int_loongarch_lasx_extract_128_lo_d`.
  **L1211 CN**: 声明 TableGen def `int_loongarch_lasx_extract_128_lo_d`。
- **L1212 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v2f64_ty], [llvm_v4f64_ty], [IntrNoMem]>;`.
  **L1212 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v2f64_ty], [llvm_v4f64_ty], [IntrNoMem]>;`。
- **L1213 EN**: Declares TableGen def `int_loongarch_lasx_extract_128_lo`.
  **L1213 CN**: 声明 TableGen def `int_loongarch_lasx_extract_128_lo`。
- **L1214 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v2i64_ty], [llvm_v4i64_ty], [IntrNoMem]>;`.
  **L1214 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v2i64_ty], [llvm_v4i64_ty], [IntrNoMem]>;`。
- **L1215 EN**: Declares TableGen def `int_loongarch_lasx_extract_128_hi_s`.
  **L1215 CN**: 声明 TableGen def `int_loongarch_lasx_extract_128_hi_s`。
- **L1216 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v4f32_ty], [llvm_v8f32_ty], [IntrNoMem]>;`.
  **L1216 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v4f32_ty], [llvm_v8f32_ty], [IntrNoMem]>;`。
- **L1217 EN**: Declares TableGen def `int_loongarch_lasx_extract_128_hi_d`.
  **L1217 CN**: 声明 TableGen def `int_loongarch_lasx_extract_128_hi_d`。
- **L1218 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v2f64_ty], [llvm_v4f64_ty], [IntrNoMem]>;`.
  **L1218 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v2f64_ty], [llvm_v4f64_ty], [IntrNoMem]>;`。
- **L1219 EN**: Declares TableGen def `int_loongarch_lasx_extract_128_hi`.
  **L1219 CN**: 声明 TableGen def `int_loongarch_lasx_extract_128_hi`。
- **L1220 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v2i64_ty], [llvm_v4i64_ty], [IntrNoMem]>;`.
  **L1220 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v2i64_ty], [llvm_v4i64_ty], [IntrNoMem]>;`。
- **L1221 EN**: Declares TableGen def `int_loongarch_lasx_insert_128_lo_s`.
  **L1221 CN**: 声明 TableGen def `int_loongarch_lasx_insert_128_lo_s`。
- **L1222 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v8f32_ty], [llvm_v8f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1222 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v8f32_ty], [llvm_v8f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1223 EN**: Declares TableGen def `int_loongarch_lasx_insert_128_lo_d`.
  **L1223 CN**: 声明 TableGen def `int_loongarch_lasx_insert_128_lo_d`。
- **L1224 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v4f64_ty], [llvm_v4f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1224 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v4f64_ty], [llvm_v4f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。

### Lines 1225-1233

````tablegen
def int_loongarch_lasx_insert_128_lo
  : VecInt<[llvm_v4i64_ty], [llvm_v4i64_ty, llvm_v2i64_ty], [IntrNoMem]>;
def int_loongarch_lasx_insert_128_hi_s
  : VecInt<[llvm_v8f32_ty], [llvm_v8f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_loongarch_lasx_insert_128_hi_d
  : VecInt<[llvm_v4f64_ty], [llvm_v4f64_ty, llvm_v2f64_ty], [IntrNoMem]>;
def int_loongarch_lasx_insert_128_hi
  : VecInt<[llvm_v4i64_ty], [llvm_v4i64_ty, llvm_v2i64_ty], [IntrNoMem]>;
} // TargetPrefix = "loongarch"
````
- **L1225 EN**: Declares TableGen def `int_loongarch_lasx_insert_128_lo`.
  **L1225 CN**: 声明 TableGen def `int_loongarch_lasx_insert_128_lo`。
- **L1226 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v4i64_ty], [llvm_v4i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1226 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v4i64_ty], [llvm_v4i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1227 EN**: Declares TableGen def `int_loongarch_lasx_insert_128_hi_s`.
  **L1227 CN**: 声明 TableGen def `int_loongarch_lasx_insert_128_hi_s`。
- **L1228 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v8f32_ty], [llvm_v8f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1228 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v8f32_ty], [llvm_v8f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1229 EN**: Declares TableGen def `int_loongarch_lasx_insert_128_hi_d`.
  **L1229 CN**: 声明 TableGen def `int_loongarch_lasx_insert_128_hi_d`。
- **L1230 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v4f64_ty], [llvm_v4f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1230 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v4f64_ty], [llvm_v4f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1231 EN**: Declares TableGen def `int_loongarch_lasx_insert_128_hi`.
  **L1231 CN**: 声明 TableGen def `int_loongarch_lasx_insert_128_hi`。
- **L1232 EN**: Executes a standalone statement or declaration: `: VecInt<[llvm_v4i64_ty], [llvm_v4i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1232 CN**: 执行一条独立语句或声明：`: VecInt<[llvm_v4i64_ty], [llvm_v4i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1233 EN**: Continues the surrounding expression or declaration: `} // TargetPrefix = "loongarch"`.
  **L1233 CN**: 继续构造周围的表达式或声明：`} // TargetPrefix = "loongarch"`。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Target intrinsic catalogs / 目标 Intrinsic 目录**

## Dependencies / 依赖关系

- No direct include-style dependencies appear in this file. / 该文件中没有直接出现 include 风格依赖。
