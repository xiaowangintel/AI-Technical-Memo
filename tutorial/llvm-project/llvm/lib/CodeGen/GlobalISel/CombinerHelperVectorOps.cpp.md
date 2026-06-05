# CombinerHelperVectorOps.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/GlobalISel/CombinerHelperVectorOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `See https://llvm.org/LICENSE.txt for license information` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“See https://llvm.org/LICENSE.txt for license information”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- CombinerHelperVectorOps.cpp-----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements CombinerHelper for G_EXTRACT_VECTOR_ELT,
// G_INSERT_VECTOR_ELT, and G_VSCALE
//
//===----------------------------------------------------------------------===//
#include "llvm/CodeGen/GlobalISel/CombinerHelper.h"
#include "llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"
#include "llvm/CodeGen/GlobalISel/LegalizerHelper.h"
#include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
#include "llvm/CodeGen/GlobalISel/MIPatternMatch.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/GlobalISel/Utils.h"
#include "llvm/CodeGen/LowLevelTypeUtils.h"
````
- **L1 EN**: Comment documents: `===- CombinerHelperVectorOps.cpp----------------------------------------…`.
  **L1 CN**: 注释说明：`===- CombinerHelperVectorOps.cpp----------------------------------------…`。
- **L2 EN**: Continues the surrounding comment block.
  **L2 CN**: 延续周围的注释块。
- **L3 EN**: Comment documents: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`.
  **L3 CN**: 注释说明：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`。
- **L4 EN**: Comment documents: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment documents: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Continues the surrounding comment block.
  **L6 CN**: 延续周围的注释块。
- **L7 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L7 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L8 EN**: Continues the surrounding comment block.
  **L8 CN**: 延续周围的注释块。
- **L9 EN**: Comment documents: `This file implements CombinerHelper for G_EXTRACT_VECTOR_ELT,`.
  **L9 CN**: 注释说明：`This file implements CombinerHelper for G_EXTRACT_VECTOR_ELT,`。
- **L10 EN**: Comment documents: `G_INSERT_VECTOR_ELT, and G_VSCALE`.
  **L10 CN**: 注释说明：`G_INSERT_VECTOR_ELT, and G_VSCALE`。
- **L11 EN**: Continues the surrounding comment block.
  **L11 CN**: 延续周围的注释块。
- **L12 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L12 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L13 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/CombinerHelper.h` for CombinerHelper support.
  **L13 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/CombinerHelper.h`，用于 CombinerHelper 相关支持。
- **L14 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/GenericMachineInstrs.h` for GenericMachineInstrs support.
  **L14 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/GenericMachineInstrs.h`，用于 GenericMachineInstrs 相关支持。
- **L15 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/LegalizerHelper.h` for LegalizerHelper support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/LegalizerHelper.h`，用于 LegalizerHelper 相关支持。
- **L16 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/LegalizerInfo.h` for LegalizerInfo support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/LegalizerInfo.h`，用于 LegalizerInfo 相关支持。
- **L17 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/MIPatternMatch.h` for MIPatternMatch support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/MIPatternMatch.h`，用于 MIPatternMatch 相关支持。
- **L18 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/MachineIRBuilder.h` for MachineIRBuilder support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`，用于 MachineIRBuilder 相关支持。
- **L19 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/Utils.h` for Utils support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/Utils.h`，用于 Utils 相关支持。
- **L20 EN**: Includes LLVM header `llvm/CodeGen/LowLevelTypeUtils.h` for LowLevelTypeUtils support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/CodeGen/LowLevelTypeUtils.h`，用于 LowLevelTypeUtils 相关支持。

### Lines 21-40

````cpp
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetLowering.h"
#include "llvm/CodeGen/TargetOpcodes.h"
#include "llvm/Support/Casting.h"
#include <optional>

#define DEBUG_TYPE "gi-combiner"

using namespace llvm;
using namespace MIPatternMatch;

bool CombinerHelper::matchExtractVectorElement(MachineInstr &MI,
                                               BuildFnTy &MatchInfo) const {
  GExtractVectorElement *Extract = cast<GExtractVectorElement>(&MI);

  Register Dst = Extract->getReg(0);
  Register Vector = Extract->getVectorReg();
  Register Index = Extract->getIndexReg();
  LLT DstTy = MRI.getType(Dst);
````
- **L21 EN**: Includes LLVM header `llvm/CodeGen/MachineOperand.h` for MachineOperand support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineOperand.h`，用于 MachineOperand 相关支持。
- **L22 EN**: Includes LLVM header `llvm/CodeGen/MachineRegisterInfo.h` for MachineRegisterInfo support.
  **L22 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineRegisterInfo.h`，用于 MachineRegisterInfo 相关支持。
- **L23 EN**: Includes LLVM header `llvm/CodeGen/TargetLowering.h` for TargetLowering support.
  **L23 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetLowering.h`，用于 TargetLowering 相关支持。
- **L24 EN**: Includes LLVM header `llvm/CodeGen/TargetOpcodes.h` for TargetOpcodes support.
  **L24 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetOpcodes.h`，用于 TargetOpcodes 相关支持。
- **L25 EN**: Includes LLVM header `llvm/Support/Casting.h` for Casting support.
  **L25 CN**: 引入 LLVM 头文件 `llvm/Support/Casting.h`，用于 Casting 相关支持。
- **L26 EN**: Includes system header `optional`.
  **L26 CN**: 引入系统头文件 `optional`。
- **L27 EN**: Separates nearby statements for readability.
  **L27 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L28 EN**: Defines the LLVM debug channel used by this file.
  **L28 CN**: 定义该文件使用的 LLVM 调试通道。
- **L29 EN**: Separates nearby statements for readability.
  **L29 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L30 EN**: Imports namespace `llvm` into this translation unit.
  **L30 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L31 EN**: Imports namespace `MIPatternMatch` into this translation unit.
  **L31 CN**: 将命名空间 `MIPatternMatch` 引入当前编译单元。
- **L32 EN**: Separates nearby statements for readability.
  **L32 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L33 EN**: Provides part of the signature for `matchExtractVectorElement`.
  **L33 CN**: 给出 `matchExtractVectorElement` 的一部分签名。
- **L34 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L34 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L35 EN**: Assigns or initializes `GExtractVectorElement *Extract`.
  **L35 CN**: 对 `GExtractVectorElement *Extract` 进行赋值或初始化。
- **L36 EN**: Separates nearby statements for readability.
  **L36 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L37 EN**: Assigns or initializes `Register Dst`.
  **L37 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L38 EN**: Assigns or initializes `Register Vector`.
  **L38 CN**: 对 `Register Vector` 进行赋值或初始化。
- **L39 EN**: Assigns or initializes `Register Index`.
  **L39 CN**: 对 `Register Index` 进行赋值或初始化。
- **L40 EN**: Assigns or initializes `LLT DstTy`.
  **L40 CN**: 对 `LLT DstTy` 进行赋值或初始化。

### Lines 41-60

````cpp
  LLT VectorTy = MRI.getType(Vector);

  // The vector register can be def'd by various ops that have vector as its
  // type. They can all be used for constant folding, scalarizing,
  // canonicalization, or combining based on symmetry.
  //
  // vector like ops
  // * build vector
  // * build vector trunc
  // * shuffle vector
  // * splat vector
  // * concat vectors
  // * insert/extract vector element
  // * insert/extract subvector
  // * vector loads
  // * scalable vector loads
  //
  // compute like ops
  // * binary ops
  // * unary ops
````
- **L41 EN**: Assigns or initializes `LLT VectorTy`.
  **L41 CN**: 对 `LLT VectorTy` 进行赋值或初始化。
- **L42 EN**: Separates nearby statements for readability.
  **L42 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L43 EN**: Comment documents: `The vector register can be def'd by various ops that have vector as its`.
  **L43 CN**: 注释说明：`The vector register can be def'd by various ops that have vector as its`。
- **L44 EN**: Comment documents: `type. They can all be used for constant folding, scalarizing,`.
  **L44 CN**: 注释说明：`type. They can all be used for constant folding, scalarizing,`。
- **L45 EN**: Comment documents: `canonicalization, or combining based on symmetry.`.
  **L45 CN**: 注释说明：`canonicalization, or combining based on symmetry.`。
- **L46 EN**: Continues the surrounding comment block.
  **L46 CN**: 延续周围的注释块。
- **L47 EN**: Comment documents: `vector like ops`.
  **L47 CN**: 注释说明：`vector like ops`。
- **L48 EN**: Comment documents: `build vector`.
  **L48 CN**: 注释说明：`build vector`。
- **L49 EN**: Comment documents: `build vector trunc`.
  **L49 CN**: 注释说明：`build vector trunc`。
- **L50 EN**: Comment documents: `shuffle vector`.
  **L50 CN**: 注释说明：`shuffle vector`。
- **L51 EN**: Comment documents: `splat vector`.
  **L51 CN**: 注释说明：`splat vector`。
- **L52 EN**: Comment documents: `concat vectors`.
  **L52 CN**: 注释说明：`concat vectors`。
- **L53 EN**: Comment documents: `insert/extract vector element`.
  **L53 CN**: 注释说明：`insert/extract vector element`。
- **L54 EN**: Comment documents: `insert/extract subvector`.
  **L54 CN**: 注释说明：`insert/extract subvector`。
- **L55 EN**: Comment documents: `vector loads`.
  **L55 CN**: 注释说明：`vector loads`。
- **L56 EN**: Comment documents: `scalable vector loads`.
  **L56 CN**: 注释说明：`scalable vector loads`。
- **L57 EN**: Continues the surrounding comment block.
  **L57 CN**: 延续周围的注释块。
- **L58 EN**: Comment documents: `compute like ops`.
  **L58 CN**: 注释说明：`compute like ops`。
- **L59 EN**: Comment documents: `binary ops`.
  **L59 CN**: 注释说明：`binary ops`。
- **L60 EN**: Comment documents: `unary ops`.
  **L60 CN**: 注释说明：`unary ops`。

### Lines 61-80

````cpp
  //  * exts and truncs
  //  * casts
  //  * fneg
  // * select
  // * phis
  // * cmps
  // * freeze
  // * bitcast
  // * undef

  // We try to get the value of the Index register.
  std::optional<ValueAndVReg> MaybeIndex =
      getIConstantVRegValWithLookThrough(Index, MRI);
  std::optional<APInt> IndexC = std::nullopt;

  if (MaybeIndex)
    IndexC = MaybeIndex->Value;

  // Fold extractVectorElement(Vector, TOOLARGE) -> undef
  if (IndexC && VectorTy.isFixedVector() &&
````
- **L61 EN**: Comment documents: `exts and truncs`.
  **L61 CN**: 注释说明：`exts and truncs`。
- **L62 EN**: Comment documents: `casts`.
  **L62 CN**: 注释说明：`casts`。
- **L63 EN**: Comment documents: `fneg`.
  **L63 CN**: 注释说明：`fneg`。
- **L64 EN**: Comment documents: `select`.
  **L64 CN**: 注释说明：`select`。
- **L65 EN**: Comment documents: `phis`.
  **L65 CN**: 注释说明：`phis`。
- **L66 EN**: Comment documents: `cmps`.
  **L66 CN**: 注释说明：`cmps`。
- **L67 EN**: Comment documents: `freeze`.
  **L67 CN**: 注释说明：`freeze`。
- **L68 EN**: Comment documents: `bitcast`.
  **L68 CN**: 注释说明：`bitcast`。
- **L69 EN**: Comment documents: `undef`.
  **L69 CN**: 注释说明：`undef`。
- **L70 EN**: Separates nearby statements for readability.
  **L70 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L71 EN**: Comment documents: `We try to get the value of the Index register.`.
  **L71 CN**: 注释说明：`We try to get the value of the Index register.`。
- **L72 EN**: Continues logic with `std::optional<ValueAndVReg> MaybeIndex =`.
  **L72 CN**: 继续处理逻辑：`std::optional<ValueAndVReg> MaybeIndex =`。
- **L73 EN**: Executes statement `getIConstantVRegValWithLookThrough(Index, MRI);`.
  **L73 CN**: 执行语句 `getIConstantVRegValWithLookThrough(Index, MRI);`。
- **L74 EN**: Assigns or initializes `std::optional<APInt> IndexC`.
  **L74 CN**: 对 `std::optional<APInt> IndexC` 进行赋值或初始化。
- **L75 EN**: Separates nearby statements for readability.
  **L75 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L76 EN**: Begins a conditional branch.
  **L76 CN**: 开始一个条件分支。
- **L77 EN**: Assigns or initializes `IndexC`.
  **L77 CN**: 对 `IndexC` 进行赋值或初始化。
- **L78 EN**: Separates nearby statements for readability.
  **L78 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L79 EN**: Comment documents: `Fold extractVectorElement(Vector, TOOLARGE) -> undef`.
  **L79 CN**: 注释说明：`Fold extractVectorElement(Vector, TOOLARGE) -> undef`。
- **L80 EN**: Begins a conditional branch.
  **L80 CN**: 开始一个条件分支。

### Lines 81-100

````cpp
      IndexC->uge(VectorTy.getNumElements()) &&
      isLegalOrBeforeLegalizer({TargetOpcode::G_IMPLICIT_DEF, {DstTy}})) {
    // For fixed-length vectors, it's invalid to extract out-of-range elements.
    MatchInfo = [=](MachineIRBuilder &B) { B.buildUndef(Dst); };
    return true;
  }

  return false;
}

bool CombinerHelper::matchExtractVectorElementWithDifferentIndices(
    const MachineOperand &MO, BuildFnTy &MatchInfo) const {
  MachineInstr *Root = getDefIgnoringCopies(MO.getReg(), MRI);
  GExtractVectorElement *Extract = cast<GExtractVectorElement>(Root);

  //
  //  %idx1:_(s64) = G_CONSTANT i64 1
  //  %idx2:_(s64) = G_CONSTANT i64 2
  //  %insert:_(<2 x s32>) = G_INSERT_VECTOR_ELT_ELT %bv(<2 x s32>),
  //  %value(s32), %idx2(s64) %extract:_(s32) = G_EXTRACT_VECTOR_ELT %insert(<2
````
- **L81 EN**: Continues logic with `IndexC->uge(VectorTy.getNumElements()) &&`.
  **L81 CN**: 继续处理逻辑：`IndexC->uge(VectorTy.getNumElements()) &&`。
- **L82 EN**: Starts block `isLegalOrBeforeLegalizer({TargetOpcode::G_IMPLICIT_DEF, {DstTy}}))`.
  **L82 CN**: 开始代码块 `isLegalOrBeforeLegalizer({TargetOpcode::G_IMPLICIT_DEF, {DstTy}}))`。
- **L83 EN**: Comment documents: `For fixed-length vectors, it's invalid to extract out-of-range elements.`.
  **L83 CN**: 注释说明：`For fixed-length vectors, it's invalid to extract out-of-range elements.`。
- **L84 EN**: Assigns or initializes `MatchInfo`.
  **L84 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L85 EN**: Returns `true` to the caller.
  **L85 CN**: 向调用者返回 `true`。
- **L86 EN**: Closes the current scope.
  **L86 CN**: 关闭当前作用域。
- **L87 EN**: Separates nearby statements for readability.
  **L87 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L88 EN**: Returns `false` to the caller.
  **L88 CN**: 向调用者返回 `false`。
- **L89 EN**: Closes the current scope.
  **L89 CN**: 关闭当前作用域。
- **L90 EN**: Separates nearby statements for readability.
  **L90 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L91 EN**: Provides part of the signature for `matchExtractVectorElementWithDifferentIndices`.
  **L91 CN**: 给出 `matchExtractVectorElementWithDifferentIndices` 的一部分签名。
- **L92 EN**: Starts block `const MachineOperand &MO, BuildFnTy &MatchInfo) const`.
  **L92 CN**: 开始代码块 `const MachineOperand &MO, BuildFnTy &MatchInfo) const`。
- **L93 EN**: Assigns or initializes `MachineInstr *Root`.
  **L93 CN**: 对 `MachineInstr *Root` 进行赋值或初始化。
- **L94 EN**: Assigns or initializes `GExtractVectorElement *Extract`.
  **L94 CN**: 对 `GExtractVectorElement *Extract` 进行赋值或初始化。
- **L95 EN**: Separates nearby statements for readability.
  **L95 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L96 EN**: Continues the surrounding comment block.
  **L96 CN**: 延续周围的注释块。
- **L97 EN**: Comment documents: `%idx1:_(s64) = G_CONSTANT i64 1`.
  **L97 CN**: 注释说明：`%idx1:_(s64) = G_CONSTANT i64 1`。
- **L98 EN**: Comment documents: `%idx2:_(s64) = G_CONSTANT i64 2`.
  **L98 CN**: 注释说明：`%idx2:_(s64) = G_CONSTANT i64 2`。
- **L99 EN**: Comment documents: `%insert:_(<2 x s32>) = G_INSERT_VECTOR_ELT_ELT %bv(<2 x s32>),`.
  **L99 CN**: 注释说明：`%insert:_(<2 x s32>) = G_INSERT_VECTOR_ELT_ELT %bv(<2 x s32>),`。
- **L100 EN**: Comment documents: `%value(s32), %idx2(s64) %extract:_(s32) = G_EXTRACT_VECTOR_ELT %insert(<…`.
  **L100 CN**: 注释说明：`%value(s32), %idx2(s64) %extract:_(s32) = G_EXTRACT_VECTOR_ELT %insert(<…`。

### Lines 101-120

````cpp
  //  x s32>), %idx1(s64)
  //
  //  -->
  //
  //  %insert:_(<2 x s32>) = G_INSERT_VECTOR_ELT_ELT %bv(<2 x s32>),
  //  %value(s32), %idx2(s64) %extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x
  //  s32>), %idx1(s64)
  //
  //

  Register Index = Extract->getIndexReg();

  // We try to get the value of the Index register.
  std::optional<ValueAndVReg> MaybeIndex =
      getIConstantVRegValWithLookThrough(Index, MRI);
  std::optional<APInt> IndexC = std::nullopt;

  if (!MaybeIndex)
    return false;
  else
````
- **L101 EN**: Comment documents: `x s32>), %idx1(s64)`.
  **L101 CN**: 注释说明：`x s32>), %idx1(s64)`。
- **L102 EN**: Continues the surrounding comment block.
  **L102 CN**: 延续周围的注释块。
- **L103 EN**: Comment documents: `-->`.
  **L103 CN**: 注释说明：`-->`。
- **L104 EN**: Continues the surrounding comment block.
  **L104 CN**: 延续周围的注释块。
- **L105 EN**: Comment documents: `%insert:_(<2 x s32>) = G_INSERT_VECTOR_ELT_ELT %bv(<2 x s32>),`.
  **L105 CN**: 注释说明：`%insert:_(<2 x s32>) = G_INSERT_VECTOR_ELT_ELT %bv(<2 x s32>),`。
- **L106 EN**: Comment documents: `%value(s32), %idx2(s64) %extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x`.
  **L106 CN**: 注释说明：`%value(s32), %idx2(s64) %extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x`。
- **L107 EN**: Comment documents: `s32>), %idx1(s64)`.
  **L107 CN**: 注释说明：`s32>), %idx1(s64)`。
- **L108 EN**: Continues the surrounding comment block.
  **L108 CN**: 延续周围的注释块。
- **L109 EN**: Continues the surrounding comment block.
  **L109 CN**: 延续周围的注释块。
- **L110 EN**: Separates nearby statements for readability.
  **L110 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L111 EN**: Assigns or initializes `Register Index`.
  **L111 CN**: 对 `Register Index` 进行赋值或初始化。
- **L112 EN**: Separates nearby statements for readability.
  **L112 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L113 EN**: Comment documents: `We try to get the value of the Index register.`.
  **L113 CN**: 注释说明：`We try to get the value of the Index register.`。
- **L114 EN**: Continues logic with `std::optional<ValueAndVReg> MaybeIndex =`.
  **L114 CN**: 继续处理逻辑：`std::optional<ValueAndVReg> MaybeIndex =`。
- **L115 EN**: Executes statement `getIConstantVRegValWithLookThrough(Index, MRI);`.
  **L115 CN**: 执行语句 `getIConstantVRegValWithLookThrough(Index, MRI);`。
- **L116 EN**: Assigns or initializes `std::optional<APInt> IndexC`.
  **L116 CN**: 对 `std::optional<APInt> IndexC` 进行赋值或初始化。
- **L117 EN**: Separates nearby statements for readability.
  **L117 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L118 EN**: Begins a conditional branch.
  **L118 CN**: 开始一个条件分支。
- **L119 EN**: Returns `false` to the caller.
  **L119 CN**: 向调用者返回 `false`。
- **L120 EN**: Handles the fallback branch.
  **L120 CN**: 处理兜底分支。

### Lines 121-140

````cpp
    IndexC = MaybeIndex->Value;

  Register Vector = Extract->getVectorReg();

  GInsertVectorElement *Insert =
      getOpcodeDef<GInsertVectorElement>(Vector, MRI);
  if (!Insert)
    return false;

  Register Dst = Extract->getReg(0);

  std::optional<ValueAndVReg> MaybeInsertIndex =
      getIConstantVRegValWithLookThrough(Insert->getIndexReg(), MRI);

  if (MaybeInsertIndex && MaybeInsertIndex->Value != *IndexC) {
    // There is no one-use check. We have to keep the insert. When both Index
    // registers are constants and not equal, we can look into the Vector
    // register of the insert.
    MatchInfo = [=](MachineIRBuilder &B) {
      B.buildExtractVectorElement(Dst, Insert->getVectorReg(), Index);
````
- **L121 EN**: Assigns or initializes `IndexC`.
  **L121 CN**: 对 `IndexC` 进行赋值或初始化。
- **L122 EN**: Separates nearby statements for readability.
  **L122 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L123 EN**: Assigns or initializes `Register Vector`.
  **L123 CN**: 对 `Register Vector` 进行赋值或初始化。
- **L124 EN**: Separates nearby statements for readability.
  **L124 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L125 EN**: Continues logic with `GInsertVectorElement *Insert =`.
  **L125 CN**: 继续处理逻辑：`GInsertVectorElement *Insert =`。
- **L126 EN**: Executes statement `getOpcodeDef<GInsertVectorElement>(Vector, MRI);`.
  **L126 CN**: 执行语句 `getOpcodeDef<GInsertVectorElement>(Vector, MRI);`。
- **L127 EN**: Begins a conditional branch.
  **L127 CN**: 开始一个条件分支。
- **L128 EN**: Returns `false` to the caller.
  **L128 CN**: 向调用者返回 `false`。
- **L129 EN**: Separates nearby statements for readability.
  **L129 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L130 EN**: Assigns or initializes `Register Dst`.
  **L130 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L131 EN**: Separates nearby statements for readability.
  **L131 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L132 EN**: Continues logic with `std::optional<ValueAndVReg> MaybeInsertIndex =`.
  **L132 CN**: 继续处理逻辑：`std::optional<ValueAndVReg> MaybeInsertIndex =`。
- **L133 EN**: Executes statement `getIConstantVRegValWithLookThrough(Insert->getIndexReg(), MRI);`.
  **L133 CN**: 执行语句 `getIConstantVRegValWithLookThrough(Insert->getIndexReg(), MRI);`。
- **L134 EN**: Separates nearby statements for readability.
  **L134 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L135 EN**: Begins a conditional branch.
  **L135 CN**: 开始一个条件分支。
- **L136 EN**: Comment documents: `There is no one-use check. We have to keep the insert. When both Index`.
  **L136 CN**: 注释说明：`There is no one-use check. We have to keep the insert. When both Index`。
- **L137 EN**: Comment documents: `registers are constants and not equal, we can look into the Vector`.
  **L137 CN**: 注释说明：`registers are constants and not equal, we can look into the Vector`。
- **L138 EN**: Comment documents: `register of the insert.`.
  **L138 CN**: 注释说明：`register of the insert.`。
- **L139 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L139 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L140 EN**: Executes statement `B.buildExtractVectorElement(Dst, Insert->getVectorReg(), Index);`.
  **L140 CN**: 执行语句 `B.buildExtractVectorElement(Dst, Insert->getVectorReg(), Index);`。

### Lines 141-160

````cpp
    };
    return true;
  }

  return false;
}

bool CombinerHelper::matchExtractVectorElementWithBuildVector(
    const MachineInstr &MI, const MachineInstr &MI2,
    BuildFnTy &MatchInfo) const {
  const GExtractVectorElement *Extract = cast<GExtractVectorElement>(&MI);
  const GBuildVector *Build = cast<GBuildVector>(&MI2);

  //
  //  %zero:_(s64) = G_CONSTANT i64 0
  //  %bv:_(<2 x s32>) = G_BUILD_VECTOR %arg1(s32), %arg2(s32)
  //  %extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x s32>), %zero(s64)
  //
  //  -->
  //
````
- **L141 EN**: Closes the current scope.
  **L141 CN**: 关闭当前作用域。
- **L142 EN**: Returns `true` to the caller.
  **L142 CN**: 向调用者返回 `true`。
- **L143 EN**: Closes the current scope.
  **L143 CN**: 关闭当前作用域。
- **L144 EN**: Separates nearby statements for readability.
  **L144 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L145 EN**: Returns `false` to the caller.
  **L145 CN**: 向调用者返回 `false`。
- **L146 EN**: Closes the current scope.
  **L146 CN**: 关闭当前作用域。
- **L147 EN**: Separates nearby statements for readability.
  **L147 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L148 EN**: Provides part of the signature for `matchExtractVectorElementWithBuildVector`.
  **L148 CN**: 给出 `matchExtractVectorElementWithBuildVector` 的一部分签名。
- **L149 EN**: Continues logic with `const MachineInstr &MI, const MachineInstr &MI2,`.
  **L149 CN**: 继续处理逻辑：`const MachineInstr &MI, const MachineInstr &MI2,`。
- **L150 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L150 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L151 EN**: Assigns or initializes `const GExtractVectorElement *Extract`.
  **L151 CN**: 对 `const GExtractVectorElement *Extract` 进行赋值或初始化。
- **L152 EN**: Assigns or initializes `const GBuildVector *Build`.
  **L152 CN**: 对 `const GBuildVector *Build` 进行赋值或初始化。
- **L153 EN**: Separates nearby statements for readability.
  **L153 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L154 EN**: Continues the surrounding comment block.
  **L154 CN**: 延续周围的注释块。
- **L155 EN**: Comment documents: `%zero:_(s64) = G_CONSTANT i64 0`.
  **L155 CN**: 注释说明：`%zero:_(s64) = G_CONSTANT i64 0`。
- **L156 EN**: Comment documents: `%bv:_(<2 x s32>) = G_BUILD_VECTOR %arg1(s32), %arg2(s32)`.
  **L156 CN**: 注释说明：`%bv:_(<2 x s32>) = G_BUILD_VECTOR %arg1(s32), %arg2(s32)`。
- **L157 EN**: Comment documents: `%extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x s32>), %zero(s64)`.
  **L157 CN**: 注释说明：`%extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x s32>), %zero(s64)`。
- **L158 EN**: Continues the surrounding comment block.
  **L158 CN**: 延续周围的注释块。
- **L159 EN**: Comment documents: `-->`.
  **L159 CN**: 注释说明：`-->`。
- **L160 EN**: Continues the surrounding comment block.
  **L160 CN**: 延续周围的注释块。

### Lines 161-180

````cpp
  //  %extract:_(32) = COPY %arg1(s32)
  //
  //

  Register Vector = Extract->getVectorReg();
  LLT VectorTy = MRI.getType(Vector);

  // There is a one-use check. There are more combines on build vectors.
  EVT Ty(getMVTForLLT(VectorTy));
  if (!MRI.hasOneNonDBGUse(Build->getReg(0)) ||
      !getTargetLowering().aggressivelyPreferBuildVectorSources(Ty))
    return false;

  APInt Index = getIConstantFromReg(Extract->getIndexReg(), MRI);

  // We now know that there is a buildVector def'd on the Vector register and
  // the index is const. The combine will succeed.

  Register Dst = Extract->getReg(0);

````
- **L161 EN**: Comment documents: `%extract:_(32) = COPY %arg1(s32)`.
  **L161 CN**: 注释说明：`%extract:_(32) = COPY %arg1(s32)`。
- **L162 EN**: Continues the surrounding comment block.
  **L162 CN**: 延续周围的注释块。
- **L163 EN**: Continues the surrounding comment block.
  **L163 CN**: 延续周围的注释块。
- **L164 EN**: Separates nearby statements for readability.
  **L164 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L165 EN**: Assigns or initializes `Register Vector`.
  **L165 CN**: 对 `Register Vector` 进行赋值或初始化。
- **L166 EN**: Assigns or initializes `LLT VectorTy`.
  **L166 CN**: 对 `LLT VectorTy` 进行赋值或初始化。
- **L167 EN**: Separates nearby statements for readability.
  **L167 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L168 EN**: Comment documents: `There is a one-use check. There are more combines on build vectors.`.
  **L168 CN**: 注释说明：`There is a one-use check. There are more combines on build vectors.`。
- **L169 EN**: Declares function or method `Ty`.
  **L169 CN**: 声明函数或方法 `Ty`。
- **L170 EN**: Begins a conditional branch.
  **L170 CN**: 开始一个条件分支。
- **L171 EN**: Continues logic with `!getTargetLowering().aggressivelyPreferBuildVectorSources(Ty))`.
  **L171 CN**: 继续处理逻辑：`!getTargetLowering().aggressivelyPreferBuildVectorSources(Ty))`。
- **L172 EN**: Returns `false` to the caller.
  **L172 CN**: 向调用者返回 `false`。
- **L173 EN**: Separates nearby statements for readability.
  **L173 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L174 EN**: Assigns or initializes `APInt Index`.
  **L174 CN**: 对 `APInt Index` 进行赋值或初始化。
- **L175 EN**: Separates nearby statements for readability.
  **L175 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L176 EN**: Comment documents: `We now know that there is a buildVector def'd on the Vector register and`.
  **L176 CN**: 注释说明：`We now know that there is a buildVector def'd on the Vector register and`。
- **L177 EN**: Comment documents: `the index is const. The combine will succeed.`.
  **L177 CN**: 注释说明：`the index is const. The combine will succeed.`。
- **L178 EN**: Separates nearby statements for readability.
  **L178 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L179 EN**: Assigns or initializes `Register Dst`.
  **L179 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L180 EN**: Separates nearby statements for readability.
  **L180 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 181-200

````cpp
  MatchInfo = [=](MachineIRBuilder &B) {
    B.buildCopy(Dst, Build->getSourceReg(Index.getZExtValue()));
  };

  return true;
}

bool CombinerHelper::matchExtractVectorElementWithBuildVectorTrunc(
    const MachineOperand &MO, BuildFnTy &MatchInfo) const {
  MachineInstr *Root = getDefIgnoringCopies(MO.getReg(), MRI);
  GExtractVectorElement *Extract = cast<GExtractVectorElement>(Root);

  //
  //  %zero:_(s64) = G_CONSTANT i64 0
  //  %bv:_(<2 x s32>) = G_BUILD_VECTOR_TRUNC %arg1(s64), %arg2(s64)
  //  %extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x s32>), %zero(s64)
  //
  //  -->
  //
  //  %extract:_(32) = G_TRUNC %arg1(s64)
````
- **L181 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L181 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L182 EN**: Executes statement `B.buildCopy(Dst, Build->getSourceReg(Index.getZExtValue()));`.
  **L182 CN**: 执行语句 `B.buildCopy(Dst, Build->getSourceReg(Index.getZExtValue()));`。
- **L183 EN**: Closes the current scope.
  **L183 CN**: 关闭当前作用域。
- **L184 EN**: Separates nearby statements for readability.
  **L184 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L185 EN**: Returns `true` to the caller.
  **L185 CN**: 向调用者返回 `true`。
- **L186 EN**: Closes the current scope.
  **L186 CN**: 关闭当前作用域。
- **L187 EN**: Separates nearby statements for readability.
  **L187 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L188 EN**: Provides part of the signature for `matchExtractVectorElementWithBuildVectorTrunc`.
  **L188 CN**: 给出 `matchExtractVectorElementWithBuildVectorTrunc` 的一部分签名。
- **L189 EN**: Starts block `const MachineOperand &MO, BuildFnTy &MatchInfo) const`.
  **L189 CN**: 开始代码块 `const MachineOperand &MO, BuildFnTy &MatchInfo) const`。
- **L190 EN**: Assigns or initializes `MachineInstr *Root`.
  **L190 CN**: 对 `MachineInstr *Root` 进行赋值或初始化。
- **L191 EN**: Assigns or initializes `GExtractVectorElement *Extract`.
  **L191 CN**: 对 `GExtractVectorElement *Extract` 进行赋值或初始化。
- **L192 EN**: Separates nearby statements for readability.
  **L192 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L193 EN**: Continues the surrounding comment block.
  **L193 CN**: 延续周围的注释块。
- **L194 EN**: Comment documents: `%zero:_(s64) = G_CONSTANT i64 0`.
  **L194 CN**: 注释说明：`%zero:_(s64) = G_CONSTANT i64 0`。
- **L195 EN**: Comment documents: `%bv:_(<2 x s32>) = G_BUILD_VECTOR_TRUNC %arg1(s64), %arg2(s64)`.
  **L195 CN**: 注释说明：`%bv:_(<2 x s32>) = G_BUILD_VECTOR_TRUNC %arg1(s64), %arg2(s64)`。
- **L196 EN**: Comment documents: `%extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x s32>), %zero(s64)`.
  **L196 CN**: 注释说明：`%extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x s32>), %zero(s64)`。
- **L197 EN**: Continues the surrounding comment block.
  **L197 CN**: 延续周围的注释块。
- **L198 EN**: Comment documents: `-->`.
  **L198 CN**: 注释说明：`-->`。
- **L199 EN**: Continues the surrounding comment block.
  **L199 CN**: 延续周围的注释块。
- **L200 EN**: Comment documents: `%extract:_(32) = G_TRUNC %arg1(s64)`.
  **L200 CN**: 注释说明：`%extract:_(32) = G_TRUNC %arg1(s64)`。

### Lines 201-220

````cpp
  //
  //
  //
  //  %bv:_(<2 x s32>) = G_BUILD_VECTOR_TRUNC %arg1(s64), %arg2(s64)
  //  %extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x s32>), %opaque(s64)
  //
  //  -->
  //
  //  %bv:_(<2 x s32>) = G_BUILD_VECTOR_TRUNC %arg1(s64), %arg2(s64)
  //  %extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x s32>), %opaque(s64)
  //

  Register Vector = Extract->getVectorReg();

  // We expect a buildVectorTrunc on the Vector register.
  GBuildVectorTrunc *Build = getOpcodeDef<GBuildVectorTrunc>(Vector, MRI);
  if (!Build)
    return false;

  LLT VectorTy = MRI.getType(Vector);
````
- **L201 EN**: Continues the surrounding comment block.
  **L201 CN**: 延续周围的注释块。
- **L202 EN**: Continues the surrounding comment block.
  **L202 CN**: 延续周围的注释块。
- **L203 EN**: Continues the surrounding comment block.
  **L203 CN**: 延续周围的注释块。
- **L204 EN**: Comment documents: `%bv:_(<2 x s32>) = G_BUILD_VECTOR_TRUNC %arg1(s64), %arg2(s64)`.
  **L204 CN**: 注释说明：`%bv:_(<2 x s32>) = G_BUILD_VECTOR_TRUNC %arg1(s64), %arg2(s64)`。
- **L205 EN**: Comment documents: `%extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x s32>), %opaque(s64)`.
  **L205 CN**: 注释说明：`%extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x s32>), %opaque(s64)`。
- **L206 EN**: Continues the surrounding comment block.
  **L206 CN**: 延续周围的注释块。
- **L207 EN**: Comment documents: `-->`.
  **L207 CN**: 注释说明：`-->`。
- **L208 EN**: Continues the surrounding comment block.
  **L208 CN**: 延续周围的注释块。
- **L209 EN**: Comment documents: `%bv:_(<2 x s32>) = G_BUILD_VECTOR_TRUNC %arg1(s64), %arg2(s64)`.
  **L209 CN**: 注释说明：`%bv:_(<2 x s32>) = G_BUILD_VECTOR_TRUNC %arg1(s64), %arg2(s64)`。
- **L210 EN**: Comment documents: `%extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x s32>), %opaque(s64)`.
  **L210 CN**: 注释说明：`%extract:_(s32) = G_EXTRACT_VECTOR_ELT %bv(<2 x s32>), %opaque(s64)`。
- **L211 EN**: Continues the surrounding comment block.
  **L211 CN**: 延续周围的注释块。
- **L212 EN**: Separates nearby statements for readability.
  **L212 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L213 EN**: Assigns or initializes `Register Vector`.
  **L213 CN**: 对 `Register Vector` 进行赋值或初始化。
- **L214 EN**: Separates nearby statements for readability.
  **L214 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L215 EN**: Comment documents: `We expect a buildVectorTrunc on the Vector register.`.
  **L215 CN**: 注释说明：`We expect a buildVectorTrunc on the Vector register.`。
- **L216 EN**: Assigns or initializes `GBuildVectorTrunc *Build`.
  **L216 CN**: 对 `GBuildVectorTrunc *Build` 进行赋值或初始化。
- **L217 EN**: Begins a conditional branch.
  **L217 CN**: 开始一个条件分支。
- **L218 EN**: Returns `false` to the caller.
  **L218 CN**: 向调用者返回 `false`。
- **L219 EN**: Separates nearby statements for readability.
  **L219 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L220 EN**: Assigns or initializes `LLT VectorTy`.
  **L220 CN**: 对 `LLT VectorTy` 进行赋值或初始化。

### Lines 221-240

````cpp

  // There is a one-use check. There are more combines on build vectors.
  EVT Ty(getMVTForLLT(VectorTy));
  if (!MRI.hasOneNonDBGUse(Build->getReg(0)) ||
      !getTargetLowering().aggressivelyPreferBuildVectorSources(Ty))
    return false;

  Register Index = Extract->getIndexReg();

  // If the Index is constant, then we can extract the element from the given
  // offset.
  std::optional<ValueAndVReg> MaybeIndex =
      getIConstantVRegValWithLookThrough(Index, MRI);
  if (!MaybeIndex)
    return false;

  // We now know that there is a buildVectorTrunc def'd on the Vector register
  // and the index is const. The combine will succeed.

  Register Dst = Extract->getReg(0);
````
- **L221 EN**: Separates nearby statements for readability.
  **L221 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L222 EN**: Comment documents: `There is a one-use check. There are more combines on build vectors.`.
  **L222 CN**: 注释说明：`There is a one-use check. There are more combines on build vectors.`。
- **L223 EN**: Declares function or method `Ty`.
  **L223 CN**: 声明函数或方法 `Ty`。
- **L224 EN**: Begins a conditional branch.
  **L224 CN**: 开始一个条件分支。
- **L225 EN**: Continues logic with `!getTargetLowering().aggressivelyPreferBuildVectorSources(Ty))`.
  **L225 CN**: 继续处理逻辑：`!getTargetLowering().aggressivelyPreferBuildVectorSources(Ty))`。
- **L226 EN**: Returns `false` to the caller.
  **L226 CN**: 向调用者返回 `false`。
- **L227 EN**: Separates nearby statements for readability.
  **L227 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L228 EN**: Assigns or initializes `Register Index`.
  **L228 CN**: 对 `Register Index` 进行赋值或初始化。
- **L229 EN**: Separates nearby statements for readability.
  **L229 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L230 EN**: Comment documents: `If the Index is constant, then we can extract the element from the given`.
  **L230 CN**: 注释说明：`If the Index is constant, then we can extract the element from the given`。
- **L231 EN**: Comment documents: `offset.`.
  **L231 CN**: 注释说明：`offset.`。
- **L232 EN**: Continues logic with `std::optional<ValueAndVReg> MaybeIndex =`.
  **L232 CN**: 继续处理逻辑：`std::optional<ValueAndVReg> MaybeIndex =`。
- **L233 EN**: Executes statement `getIConstantVRegValWithLookThrough(Index, MRI);`.
  **L233 CN**: 执行语句 `getIConstantVRegValWithLookThrough(Index, MRI);`。
- **L234 EN**: Begins a conditional branch.
  **L234 CN**: 开始一个条件分支。
- **L235 EN**: Returns `false` to the caller.
  **L235 CN**: 向调用者返回 `false`。
- **L236 EN**: Separates nearby statements for readability.
  **L236 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L237 EN**: Comment documents: `We now know that there is a buildVectorTrunc def'd on the Vector registe…`.
  **L237 CN**: 注释说明：`We now know that there is a buildVectorTrunc def'd on the Vector registe…`。
- **L238 EN**: Comment documents: `and the index is const. The combine will succeed.`.
  **L238 CN**: 注释说明：`and the index is const. The combine will succeed.`。
- **L239 EN**: Separates nearby statements for readability.
  **L239 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L240 EN**: Assigns or initializes `Register Dst`.
  **L240 CN**: 对 `Register Dst` 进行赋值或初始化。

### Lines 241-260

````cpp
  LLT DstTy = MRI.getType(Dst);
  LLT SrcTy = MRI.getType(Build->getSourceReg(0));

  // For buildVectorTrunc, the inputs are truncated.
  if (!isLegalOrBeforeLegalizer({TargetOpcode::G_TRUNC, {DstTy, SrcTy}}))
    return false;

  MatchInfo = [=](MachineIRBuilder &B) {
    B.buildTrunc(Dst, Build->getSourceReg(MaybeIndex->Value.getZExtValue()));
  };

  return true;
}

bool CombinerHelper::matchExtractVectorElementWithShuffleVector(
    const MachineInstr &MI, const MachineInstr &MI2,
    BuildFnTy &MatchInfo) const {
  const GExtractVectorElement *Extract = cast<GExtractVectorElement>(&MI);
  const GShuffleVector *Shuffle = cast<GShuffleVector>(&MI2);

````
- **L241 EN**: Assigns or initializes `LLT DstTy`.
  **L241 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L242 EN**: Assigns or initializes `LLT SrcTy`.
  **L242 CN**: 对 `LLT SrcTy` 进行赋值或初始化。
- **L243 EN**: Separates nearby statements for readability.
  **L243 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L244 EN**: Comment documents: `For buildVectorTrunc, the inputs are truncated.`.
  **L244 CN**: 注释说明：`For buildVectorTrunc, the inputs are truncated.`。
- **L245 EN**: Begins a conditional branch.
  **L245 CN**: 开始一个条件分支。
- **L246 EN**: Returns `false` to the caller.
  **L246 CN**: 向调用者返回 `false`。
- **L247 EN**: Separates nearby statements for readability.
  **L247 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L248 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L248 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L249 EN**: Executes statement `B.buildTrunc(Dst, Build->getSourceReg(MaybeIndex->Value.getZExtValue()))…`.
  **L249 CN**: 执行语句 `B.buildTrunc(Dst, Build->getSourceReg(MaybeIndex->Value.getZExtValue()))…`。
- **L250 EN**: Closes the current scope.
  **L250 CN**: 关闭当前作用域。
- **L251 EN**: Separates nearby statements for readability.
  **L251 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L252 EN**: Returns `true` to the caller.
  **L252 CN**: 向调用者返回 `true`。
- **L253 EN**: Closes the current scope.
  **L253 CN**: 关闭当前作用域。
- **L254 EN**: Separates nearby statements for readability.
  **L254 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L255 EN**: Provides part of the signature for `matchExtractVectorElementWithShuffleVector`.
  **L255 CN**: 给出 `matchExtractVectorElementWithShuffleVector` 的一部分签名。
- **L256 EN**: Continues logic with `const MachineInstr &MI, const MachineInstr &MI2,`.
  **L256 CN**: 继续处理逻辑：`const MachineInstr &MI, const MachineInstr &MI2,`。
- **L257 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L257 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L258 EN**: Assigns or initializes `const GExtractVectorElement *Extract`.
  **L258 CN**: 对 `const GExtractVectorElement *Extract` 进行赋值或初始化。
- **L259 EN**: Assigns or initializes `const GShuffleVector *Shuffle`.
  **L259 CN**: 对 `const GShuffleVector *Shuffle` 进行赋值或初始化。
- **L260 EN**: Separates nearby statements for readability.
  **L260 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 261-280

````cpp
  //
  //  %zero:_(s64) = G_CONSTANT i64 0
  //  %sv:_(<4 x s32>) = G_SHUFFLE_SHUFFLE %arg1(<4 x s32>), %arg2(<4 x s32>),
  //                     shufflemask(0, 0, 0, 0)
  //  %extract:_(s32) = G_EXTRACT_VECTOR_ELT %sv(<4 x s32>), %zero(s64)
  //
  //  -->
  //
  //  %zero1:_(s64) = G_CONSTANT i64 0
  //  %extract:_(s32) = G_EXTRACT_VECTOR_ELT %arg1(<4 x s32>), %zero1(s64)
  //
  //
  //
  //
  //  %three:_(s64) = G_CONSTANT i64 3
  //  %sv:_(<4 x s32>) = G_SHUFFLE_SHUFFLE %arg1(<4 x s32>), %arg2(<4 x s32>),
  //                     shufflemask(0, 0, 0, -1)
  //  %extract:_(s32) = G_EXTRACT_VECTOR_ELT %sv(<4 x s32>), %three(s64)
  //
  //  -->
````
- **L261 EN**: Continues the surrounding comment block.
  **L261 CN**: 延续周围的注释块。
- **L262 EN**: Comment documents: `%zero:_(s64) = G_CONSTANT i64 0`.
  **L262 CN**: 注释说明：`%zero:_(s64) = G_CONSTANT i64 0`。
- **L263 EN**: Comment documents: `%sv:_(<4 x s32>) = G_SHUFFLE_SHUFFLE %arg1(<4 x s32>), %arg2(<4 x s32>),`.
  **L263 CN**: 注释说明：`%sv:_(<4 x s32>) = G_SHUFFLE_SHUFFLE %arg1(<4 x s32>), %arg2(<4 x s32>),`。
- **L264 EN**: Comment documents: `shufflemask(0, 0, 0, 0)`.
  **L264 CN**: 注释说明：`shufflemask(0, 0, 0, 0)`。
- **L265 EN**: Comment documents: `%extract:_(s32) = G_EXTRACT_VECTOR_ELT %sv(<4 x s32>), %zero(s64)`.
  **L265 CN**: 注释说明：`%extract:_(s32) = G_EXTRACT_VECTOR_ELT %sv(<4 x s32>), %zero(s64)`。
- **L266 EN**: Continues the surrounding comment block.
  **L266 CN**: 延续周围的注释块。
- **L267 EN**: Comment documents: `-->`.
  **L267 CN**: 注释说明：`-->`。
- **L268 EN**: Continues the surrounding comment block.
  **L268 CN**: 延续周围的注释块。
- **L269 EN**: Comment documents: `%zero1:_(s64) = G_CONSTANT i64 0`.
  **L269 CN**: 注释说明：`%zero1:_(s64) = G_CONSTANT i64 0`。
- **L270 EN**: Comment documents: `%extract:_(s32) = G_EXTRACT_VECTOR_ELT %arg1(<4 x s32>), %zero1(s64)`.
  **L270 CN**: 注释说明：`%extract:_(s32) = G_EXTRACT_VECTOR_ELT %arg1(<4 x s32>), %zero1(s64)`。
- **L271 EN**: Continues the surrounding comment block.
  **L271 CN**: 延续周围的注释块。
- **L272 EN**: Continues the surrounding comment block.
  **L272 CN**: 延续周围的注释块。
- **L273 EN**: Continues the surrounding comment block.
  **L273 CN**: 延续周围的注释块。
- **L274 EN**: Continues the surrounding comment block.
  **L274 CN**: 延续周围的注释块。
- **L275 EN**: Comment documents: `%three:_(s64) = G_CONSTANT i64 3`.
  **L275 CN**: 注释说明：`%three:_(s64) = G_CONSTANT i64 3`。
- **L276 EN**: Comment documents: `%sv:_(<4 x s32>) = G_SHUFFLE_SHUFFLE %arg1(<4 x s32>), %arg2(<4 x s32>),`.
  **L276 CN**: 注释说明：`%sv:_(<4 x s32>) = G_SHUFFLE_SHUFFLE %arg1(<4 x s32>), %arg2(<4 x s32>),`。
- **L277 EN**: Comment documents: `shufflemask(0, 0, 0, -1)`.
  **L277 CN**: 注释说明：`shufflemask(0, 0, 0, -1)`。
- **L278 EN**: Comment documents: `%extract:_(s32) = G_EXTRACT_VECTOR_ELT %sv(<4 x s32>), %three(s64)`.
  **L278 CN**: 注释说明：`%extract:_(s32) = G_EXTRACT_VECTOR_ELT %sv(<4 x s32>), %three(s64)`。
- **L279 EN**: Continues the surrounding comment block.
  **L279 CN**: 延续周围的注释块。
- **L280 EN**: Comment documents: `-->`.
  **L280 CN**: 注释说明：`-->`。

### Lines 281-300

````cpp
  //
  //  %extract:_(s32) = G_IMPLICIT_DEF
  //
  //

  APInt Index = getIConstantFromReg(Extract->getIndexReg(), MRI);

  ArrayRef<int> Mask = Shuffle->getMask();

  unsigned Offset = Index.getZExtValue();
  int SrcIdx = Mask[Offset];

  LLT Src1Type = MRI.getType(Shuffle->getSrc1Reg());
  // At the IR level a <1 x ty> shuffle  vector is valid, but we want to extract
  // from a vector.
  assert(Src1Type.isVector() && "expected to extract from a vector");
  unsigned LHSWidth = Src1Type.isVector() ? Src1Type.getNumElements() : 1;

  // Note that there is no one use check.
  Register Dst = Extract->getReg(0);
````
- **L281 EN**: Continues the surrounding comment block.
  **L281 CN**: 延续周围的注释块。
- **L282 EN**: Comment documents: `%extract:_(s32) = G_IMPLICIT_DEF`.
  **L282 CN**: 注释说明：`%extract:_(s32) = G_IMPLICIT_DEF`。
- **L283 EN**: Continues the surrounding comment block.
  **L283 CN**: 延续周围的注释块。
- **L284 EN**: Continues the surrounding comment block.
  **L284 CN**: 延续周围的注释块。
- **L285 EN**: Separates nearby statements for readability.
  **L285 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L286 EN**: Assigns or initializes `APInt Index`.
  **L286 CN**: 对 `APInt Index` 进行赋值或初始化。
- **L287 EN**: Separates nearby statements for readability.
  **L287 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L288 EN**: Assigns or initializes `ArrayRef<int> Mask`.
  **L288 CN**: 对 `ArrayRef<int> Mask` 进行赋值或初始化。
- **L289 EN**: Separates nearby statements for readability.
  **L289 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L290 EN**: Assigns or initializes `unsigned Offset`.
  **L290 CN**: 对 `unsigned Offset` 进行赋值或初始化。
- **L291 EN**: Assigns or initializes `int SrcIdx`.
  **L291 CN**: 对 `int SrcIdx` 进行赋值或初始化。
- **L292 EN**: Separates nearby statements for readability.
  **L292 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L293 EN**: Assigns or initializes `LLT Src1Type`.
  **L293 CN**: 对 `LLT Src1Type` 进行赋值或初始化。
- **L294 EN**: Comment documents: `At the IR level a <1 x ty> shuffle vector is valid, but we want to extra…`.
  **L294 CN**: 注释说明：`At the IR level a <1 x ty> shuffle vector is valid, but we want to extra…`。
- **L295 EN**: Comment documents: `from a vector.`.
  **L295 CN**: 注释说明：`from a vector.`。
- **L296 EN**: Checks an invariant in debug builds.
  **L296 CN**: 在调试构建中检查一个不变量。
- **L297 EN**: Assigns or initializes `unsigned LHSWidth`.
  **L297 CN**: 对 `unsigned LHSWidth` 进行赋值或初始化。
- **L298 EN**: Separates nearby statements for readability.
  **L298 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L299 EN**: Comment documents: `Note that there is no one use check.`.
  **L299 CN**: 注释说明：`Note that there is no one use check.`。
- **L300 EN**: Assigns or initializes `Register Dst`.
  **L300 CN**: 对 `Register Dst` 进行赋值或初始化。

### Lines 301-320

````cpp
  LLT DstTy = MRI.getType(Dst);

  if (SrcIdx < 0 &&
      isLegalOrBeforeLegalizer({TargetOpcode::G_IMPLICIT_DEF, {DstTy}})) {
    MatchInfo = [=](MachineIRBuilder &B) { B.buildUndef(Dst); };
    return true;
  }

  // If the legality check failed, then we still have to abort.
  if (SrcIdx < 0)
    return false;

  Register NewVector;

  // We check in which vector and at what offset to look through.
  if (SrcIdx < (int)LHSWidth) {
    NewVector = Shuffle->getSrc1Reg();
    // SrcIdx unchanged
  } else { // SrcIdx >= LHSWidth
    NewVector = Shuffle->getSrc2Reg();
````
- **L301 EN**: Assigns or initializes `LLT DstTy`.
  **L301 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L302 EN**: Separates nearby statements for readability.
  **L302 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L303 EN**: Begins a conditional branch.
  **L303 CN**: 开始一个条件分支。
- **L304 EN**: Starts block `isLegalOrBeforeLegalizer({TargetOpcode::G_IMPLICIT_DEF, {DstTy}}))`.
  **L304 CN**: 开始代码块 `isLegalOrBeforeLegalizer({TargetOpcode::G_IMPLICIT_DEF, {DstTy}}))`。
- **L305 EN**: Assigns or initializes `MatchInfo`.
  **L305 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L306 EN**: Returns `true` to the caller.
  **L306 CN**: 向调用者返回 `true`。
- **L307 EN**: Closes the current scope.
  **L307 CN**: 关闭当前作用域。
- **L308 EN**: Separates nearby statements for readability.
  **L308 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L309 EN**: Comment documents: `If the legality check failed, then we still have to abort.`.
  **L309 CN**: 注释说明：`If the legality check failed, then we still have to abort.`。
- **L310 EN**: Begins a conditional branch.
  **L310 CN**: 开始一个条件分支。
- **L311 EN**: Returns `false` to the caller.
  **L311 CN**: 向调用者返回 `false`。
- **L312 EN**: Separates nearby statements for readability.
  **L312 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L313 EN**: Executes statement `Register NewVector;`.
  **L313 CN**: 执行语句 `Register NewVector;`。
- **L314 EN**: Separates nearby statements for readability.
  **L314 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L315 EN**: Comment documents: `We check in which vector and at what offset to look through.`.
  **L315 CN**: 注释说明：`We check in which vector and at what offset to look through.`。
- **L316 EN**: Begins a conditional branch.
  **L316 CN**: 开始一个条件分支。
- **L317 EN**: Assigns or initializes `NewVector`.
  **L317 CN**: 对 `NewVector` 进行赋值或初始化。
- **L318 EN**: Comment documents: `SrcIdx unchanged`.
  **L318 CN**: 注释说明：`SrcIdx unchanged`。
- **L319 EN**: Continues logic with `} else { // SrcIdx >= LHSWidth`.
  **L319 CN**: 继续处理逻辑：`} else { // SrcIdx >= LHSWidth`。
- **L320 EN**: Assigns or initializes `NewVector`.
  **L320 CN**: 对 `NewVector` 进行赋值或初始化。

### Lines 321-340

````cpp
    SrcIdx -= LHSWidth;
  }

  LLT IdxTy = MRI.getType(Extract->getIndexReg());
  LLT NewVectorTy = MRI.getType(NewVector);

  // We check the legality of the look through.
  if (!isLegalOrBeforeLegalizer(
          {TargetOpcode::G_EXTRACT_VECTOR_ELT, {DstTy, NewVectorTy, IdxTy}}) ||
      !isConstantLegalOrBeforeLegalizer({IdxTy}))
    return false;

  // We look through the shuffle vector.
  MatchInfo = [=](MachineIRBuilder &B) {
    auto Idx = B.buildConstant(IdxTy, SrcIdx);
    B.buildExtractVectorElement(Dst, NewVector, Idx);
  };

  return true;
}
````
- **L321 EN**: Assigns or initializes `SrcIdx -`.
  **L321 CN**: 对 `SrcIdx -` 进行赋值或初始化。
- **L322 EN**: Closes the current scope.
  **L322 CN**: 关闭当前作用域。
- **L323 EN**: Separates nearby statements for readability.
  **L323 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L324 EN**: Assigns or initializes `LLT IdxTy`.
  **L324 CN**: 对 `LLT IdxTy` 进行赋值或初始化。
- **L325 EN**: Assigns or initializes `LLT NewVectorTy`.
  **L325 CN**: 对 `LLT NewVectorTy` 进行赋值或初始化。
- **L326 EN**: Separates nearby statements for readability.
  **L326 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L327 EN**: Comment documents: `We check the legality of the look through.`.
  **L327 CN**: 注释说明：`We check the legality of the look through.`。
- **L328 EN**: Begins a conditional branch.
  **L328 CN**: 开始一个条件分支。
- **L329 EN**: Continues logic with `{TargetOpcode::G_EXTRACT_VECTOR_ELT, {DstTy, NewVectorTy, IdxTy}}) ||`.
  **L329 CN**: 继续处理逻辑：`{TargetOpcode::G_EXTRACT_VECTOR_ELT, {DstTy, NewVectorTy, IdxTy}}) ||`。
- **L330 EN**: Continues logic with `!isConstantLegalOrBeforeLegalizer({IdxTy}))`.
  **L330 CN**: 继续处理逻辑：`!isConstantLegalOrBeforeLegalizer({IdxTy}))`。
- **L331 EN**: Returns `false` to the caller.
  **L331 CN**: 向调用者返回 `false`。
- **L332 EN**: Separates nearby statements for readability.
  **L332 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L333 EN**: Comment documents: `We look through the shuffle vector.`.
  **L333 CN**: 注释说明：`We look through the shuffle vector.`。
- **L334 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L334 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L335 EN**: Assigns or initializes `auto Idx`.
  **L335 CN**: 对 `auto Idx` 进行赋值或初始化。
- **L336 EN**: Executes statement `B.buildExtractVectorElement(Dst, NewVector, Idx);`.
  **L336 CN**: 执行语句 `B.buildExtractVectorElement(Dst, NewVector, Idx);`。
- **L337 EN**: Closes the current scope.
  **L337 CN**: 关闭当前作用域。
- **L338 EN**: Separates nearby statements for readability.
  **L338 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L339 EN**: Returns `true` to the caller.
  **L339 CN**: 向调用者返回 `true`。
- **L340 EN**: Closes the current scope.
  **L340 CN**: 关闭当前作用域。

### Lines 341-360

````cpp

bool CombinerHelper::matchInsertVectorElementOOB(MachineInstr &MI,
                                                 BuildFnTy &MatchInfo) const {
  GInsertVectorElement *Insert = cast<GInsertVectorElement>(&MI);

  Register Dst = Insert->getReg(0);
  LLT DstTy = MRI.getType(Dst);
  Register Index = Insert->getIndexReg();

  if (!DstTy.isFixedVector())
    return false;

  std::optional<ValueAndVReg> MaybeIndex =
      getIConstantVRegValWithLookThrough(Index, MRI);

  if (MaybeIndex && MaybeIndex->Value.uge(DstTy.getNumElements()) &&
      isLegalOrBeforeLegalizer({TargetOpcode::G_IMPLICIT_DEF, {DstTy}})) {
    MatchInfo = [=](MachineIRBuilder &B) { B.buildUndef(Dst); };
    return true;
  }
````
- **L341 EN**: Separates nearby statements for readability.
  **L341 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L342 EN**: Provides part of the signature for `matchInsertVectorElementOOB`.
  **L342 CN**: 给出 `matchInsertVectorElementOOB` 的一部分签名。
- **L343 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L343 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L344 EN**: Assigns or initializes `GInsertVectorElement *Insert`.
  **L344 CN**: 对 `GInsertVectorElement *Insert` 进行赋值或初始化。
- **L345 EN**: Separates nearby statements for readability.
  **L345 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L346 EN**: Assigns or initializes `Register Dst`.
  **L346 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L347 EN**: Assigns or initializes `LLT DstTy`.
  **L347 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L348 EN**: Assigns or initializes `Register Index`.
  **L348 CN**: 对 `Register Index` 进行赋值或初始化。
- **L349 EN**: Separates nearby statements for readability.
  **L349 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L350 EN**: Begins a conditional branch.
  **L350 CN**: 开始一个条件分支。
- **L351 EN**: Returns `false` to the caller.
  **L351 CN**: 向调用者返回 `false`。
- **L352 EN**: Separates nearby statements for readability.
  **L352 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L353 EN**: Continues logic with `std::optional<ValueAndVReg> MaybeIndex =`.
  **L353 CN**: 继续处理逻辑：`std::optional<ValueAndVReg> MaybeIndex =`。
- **L354 EN**: Executes statement `getIConstantVRegValWithLookThrough(Index, MRI);`.
  **L354 CN**: 执行语句 `getIConstantVRegValWithLookThrough(Index, MRI);`。
- **L355 EN**: Separates nearby statements for readability.
  **L355 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L356 EN**: Begins a conditional branch.
  **L356 CN**: 开始一个条件分支。
- **L357 EN**: Starts block `isLegalOrBeforeLegalizer({TargetOpcode::G_IMPLICIT_DEF, {DstTy}}))`.
  **L357 CN**: 开始代码块 `isLegalOrBeforeLegalizer({TargetOpcode::G_IMPLICIT_DEF, {DstTy}}))`。
- **L358 EN**: Assigns or initializes `MatchInfo`.
  **L358 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L359 EN**: Returns `true` to the caller.
  **L359 CN**: 向调用者返回 `true`。
- **L360 EN**: Closes the current scope.
  **L360 CN**: 关闭当前作用域。

### Lines 361-380

````cpp

  return false;
}

bool CombinerHelper::matchAddOfVScale(const MachineOperand &MO,
                                      BuildFnTy &MatchInfo) const {
  GAdd *Add = cast<GAdd>(MRI.getVRegDef(MO.getReg()));
  GVScale *LHSVScale = cast<GVScale>(MRI.getVRegDef(Add->getLHSReg()));
  GVScale *RHSVScale = cast<GVScale>(MRI.getVRegDef(Add->getRHSReg()));

  Register Dst = Add->getReg(0);

  if (!MRI.hasOneNonDBGUse(LHSVScale->getReg(0)) ||
      !MRI.hasOneNonDBGUse(RHSVScale->getReg(0)))
    return false;

  MatchInfo = [=](MachineIRBuilder &B) {
    B.buildVScale(Dst, LHSVScale->getSrc() + RHSVScale->getSrc());
  };

````
- **L361 EN**: Separates nearby statements for readability.
  **L361 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L362 EN**: Returns `false` to the caller.
  **L362 CN**: 向调用者返回 `false`。
- **L363 EN**: Closes the current scope.
  **L363 CN**: 关闭当前作用域。
- **L364 EN**: Separates nearby statements for readability.
  **L364 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L365 EN**: Provides part of the signature for `matchAddOfVScale`.
  **L365 CN**: 给出 `matchAddOfVScale` 的一部分签名。
- **L366 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L366 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L367 EN**: Assigns or initializes `GAdd *Add`.
  **L367 CN**: 对 `GAdd *Add` 进行赋值或初始化。
- **L368 EN**: Assigns or initializes `GVScale *LHSVScale`.
  **L368 CN**: 对 `GVScale *LHSVScale` 进行赋值或初始化。
- **L369 EN**: Assigns or initializes `GVScale *RHSVScale`.
  **L369 CN**: 对 `GVScale *RHSVScale` 进行赋值或初始化。
- **L370 EN**: Separates nearby statements for readability.
  **L370 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L371 EN**: Assigns or initializes `Register Dst`.
  **L371 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L372 EN**: Separates nearby statements for readability.
  **L372 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L373 EN**: Begins a conditional branch.
  **L373 CN**: 开始一个条件分支。
- **L374 EN**: Continues logic with `!MRI.hasOneNonDBGUse(RHSVScale->getReg(0)))`.
  **L374 CN**: 继续处理逻辑：`!MRI.hasOneNonDBGUse(RHSVScale->getReg(0)))`。
- **L375 EN**: Returns `false` to the caller.
  **L375 CN**: 向调用者返回 `false`。
- **L376 EN**: Separates nearby statements for readability.
  **L376 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L377 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L377 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L378 EN**: Executes statement `B.buildVScale(Dst, LHSVScale->getSrc() + RHSVScale->getSrc());`.
  **L378 CN**: 执行语句 `B.buildVScale(Dst, LHSVScale->getSrc() + RHSVScale->getSrc());`。
- **L379 EN**: Closes the current scope.
  **L379 CN**: 关闭当前作用域。
- **L380 EN**: Separates nearby statements for readability.
  **L380 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 381-400

````cpp
  return true;
}

bool CombinerHelper::matchMulOfVScale(const MachineOperand &MO,
                                      BuildFnTy &MatchInfo) const {
  GMul *Mul = cast<GMul>(MRI.getVRegDef(MO.getReg()));
  GVScale *LHSVScale = cast<GVScale>(MRI.getVRegDef(Mul->getLHSReg()));

  std::optional<APInt> MaybeRHS = getIConstantVRegVal(Mul->getRHSReg(), MRI);
  if (!MaybeRHS)
    return false;

  Register Dst = MO.getReg();

  if (!MRI.hasOneNonDBGUse(LHSVScale->getReg(0)))
    return false;

  MatchInfo = [=](MachineIRBuilder &B) {
    B.buildVScale(Dst, LHSVScale->getSrc() * *MaybeRHS);
  };
````
- **L381 EN**: Returns `true` to the caller.
  **L381 CN**: 向调用者返回 `true`。
- **L382 EN**: Closes the current scope.
  **L382 CN**: 关闭当前作用域。
- **L383 EN**: Separates nearby statements for readability.
  **L383 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L384 EN**: Provides part of the signature for `matchMulOfVScale`.
  **L384 CN**: 给出 `matchMulOfVScale` 的一部分签名。
- **L385 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L385 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L386 EN**: Assigns or initializes `GMul *Mul`.
  **L386 CN**: 对 `GMul *Mul` 进行赋值或初始化。
- **L387 EN**: Assigns or initializes `GVScale *LHSVScale`.
  **L387 CN**: 对 `GVScale *LHSVScale` 进行赋值或初始化。
- **L388 EN**: Separates nearby statements for readability.
  **L388 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L389 EN**: Assigns or initializes `std::optional<APInt> MaybeRHS`.
  **L389 CN**: 对 `std::optional<APInt> MaybeRHS` 进行赋值或初始化。
- **L390 EN**: Begins a conditional branch.
  **L390 CN**: 开始一个条件分支。
- **L391 EN**: Returns `false` to the caller.
  **L391 CN**: 向调用者返回 `false`。
- **L392 EN**: Separates nearby statements for readability.
  **L392 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L393 EN**: Assigns or initializes `Register Dst`.
  **L393 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L394 EN**: Separates nearby statements for readability.
  **L394 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L395 EN**: Begins a conditional branch.
  **L395 CN**: 开始一个条件分支。
- **L396 EN**: Returns `false` to the caller.
  **L396 CN**: 向调用者返回 `false`。
- **L397 EN**: Separates nearby statements for readability.
  **L397 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L398 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L398 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L399 EN**: Executes statement `B.buildVScale(Dst, LHSVScale->getSrc() * *MaybeRHS);`.
  **L399 CN**: 执行语句 `B.buildVScale(Dst, LHSVScale->getSrc() * *MaybeRHS);`。
- **L400 EN**: Closes the current scope.
  **L400 CN**: 关闭当前作用域。

### Lines 401-420

````cpp

  return true;
}

bool CombinerHelper::matchSubOfVScale(const MachineOperand &MO,
                                      BuildFnTy &MatchInfo) const {
  GSub *Sub = cast<GSub>(MRI.getVRegDef(MO.getReg()));
  GVScale *RHSVScale = cast<GVScale>(MRI.getVRegDef(Sub->getRHSReg()));

  Register Dst = MO.getReg();
  LLT DstTy = MRI.getType(Dst);

  if (!MRI.hasOneNonDBGUse(RHSVScale->getReg(0)) ||
      !isLegalOrBeforeLegalizer({TargetOpcode::G_ADD, DstTy}))
    return false;

  MatchInfo = [=](MachineIRBuilder &B) {
    auto VScale = B.buildVScale(DstTy, -RHSVScale->getSrc());
    B.buildAdd(Dst, Sub->getLHSReg(), VScale, Sub->getFlags());
  };
````
- **L401 EN**: Separates nearby statements for readability.
  **L401 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L402 EN**: Returns `true` to the caller.
  **L402 CN**: 向调用者返回 `true`。
- **L403 EN**: Closes the current scope.
  **L403 CN**: 关闭当前作用域。
- **L404 EN**: Separates nearby statements for readability.
  **L404 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L405 EN**: Provides part of the signature for `matchSubOfVScale`.
  **L405 CN**: 给出 `matchSubOfVScale` 的一部分签名。
- **L406 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L406 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L407 EN**: Assigns or initializes `GSub *Sub`.
  **L407 CN**: 对 `GSub *Sub` 进行赋值或初始化。
- **L408 EN**: Assigns or initializes `GVScale *RHSVScale`.
  **L408 CN**: 对 `GVScale *RHSVScale` 进行赋值或初始化。
- **L409 EN**: Separates nearby statements for readability.
  **L409 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L410 EN**: Assigns or initializes `Register Dst`.
  **L410 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L411 EN**: Assigns or initializes `LLT DstTy`.
  **L411 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L412 EN**: Separates nearby statements for readability.
  **L412 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L413 EN**: Begins a conditional branch.
  **L413 CN**: 开始一个条件分支。
- **L414 EN**: Continues logic with `!isLegalOrBeforeLegalizer({TargetOpcode::G_ADD, DstTy}))`.
  **L414 CN**: 继续处理逻辑：`!isLegalOrBeforeLegalizer({TargetOpcode::G_ADD, DstTy}))`。
- **L415 EN**: Returns `false` to the caller.
  **L415 CN**: 向调用者返回 `false`。
- **L416 EN**: Separates nearby statements for readability.
  **L416 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L417 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L417 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L418 EN**: Assigns or initializes `auto VScale`.
  **L418 CN**: 对 `auto VScale` 进行赋值或初始化。
- **L419 EN**: Executes statement `B.buildAdd(Dst, Sub->getLHSReg(), VScale, Sub->getFlags());`.
  **L419 CN**: 执行语句 `B.buildAdd(Dst, Sub->getLHSReg(), VScale, Sub->getFlags());`。
- **L420 EN**: Closes the current scope.
  **L420 CN**: 关闭当前作用域。

### Lines 421-440

````cpp

  return true;
}

bool CombinerHelper::matchShlOfVScale(const MachineOperand &MO,
                                      BuildFnTy &MatchInfo) const {
  GShl *Shl = cast<GShl>(MRI.getVRegDef(MO.getReg()));
  GVScale *LHSVScale = cast<GVScale>(MRI.getVRegDef(Shl->getSrcReg()));

  std::optional<APInt> MaybeRHS = getIConstantVRegVal(Shl->getShiftReg(), MRI);
  if (!MaybeRHS)
    return false;

  Register Dst = MO.getReg();
  LLT DstTy = MRI.getType(Dst);

  if (!MRI.hasOneNonDBGUse(LHSVScale->getReg(0)) ||
      !isLegalOrBeforeLegalizer({TargetOpcode::G_VSCALE, DstTy}))
    return false;

````
- **L421 EN**: Separates nearby statements for readability.
  **L421 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L422 EN**: Returns `true` to the caller.
  **L422 CN**: 向调用者返回 `true`。
- **L423 EN**: Closes the current scope.
  **L423 CN**: 关闭当前作用域。
- **L424 EN**: Separates nearby statements for readability.
  **L424 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L425 EN**: Provides part of the signature for `matchShlOfVScale`.
  **L425 CN**: 给出 `matchShlOfVScale` 的一部分签名。
- **L426 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L426 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L427 EN**: Assigns or initializes `GShl *Shl`.
  **L427 CN**: 对 `GShl *Shl` 进行赋值或初始化。
- **L428 EN**: Assigns or initializes `GVScale *LHSVScale`.
  **L428 CN**: 对 `GVScale *LHSVScale` 进行赋值或初始化。
- **L429 EN**: Separates nearby statements for readability.
  **L429 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L430 EN**: Assigns or initializes `std::optional<APInt> MaybeRHS`.
  **L430 CN**: 对 `std::optional<APInt> MaybeRHS` 进行赋值或初始化。
- **L431 EN**: Begins a conditional branch.
  **L431 CN**: 开始一个条件分支。
- **L432 EN**: Returns `false` to the caller.
  **L432 CN**: 向调用者返回 `false`。
- **L433 EN**: Separates nearby statements for readability.
  **L433 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L434 EN**: Assigns or initializes `Register Dst`.
  **L434 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L435 EN**: Assigns or initializes `LLT DstTy`.
  **L435 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L436 EN**: Separates nearby statements for readability.
  **L436 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L437 EN**: Begins a conditional branch.
  **L437 CN**: 开始一个条件分支。
- **L438 EN**: Continues logic with `!isLegalOrBeforeLegalizer({TargetOpcode::G_VSCALE, DstTy}))`.
  **L438 CN**: 继续处理逻辑：`!isLegalOrBeforeLegalizer({TargetOpcode::G_VSCALE, DstTy}))`。
- **L439 EN**: Returns `false` to the caller.
  **L439 CN**: 向调用者返回 `false`。
- **L440 EN**: Separates nearby statements for readability.
  **L440 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 441-446

````cpp
  MatchInfo = [=](MachineIRBuilder &B) {
    B.buildVScale(Dst, LHSVScale->getSrc().shl(*MaybeRHS));
  };

  return true;
}
````
- **L441 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L441 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L442 EN**: Executes statement `B.buildVScale(Dst, LHSVScale->getSrc().shl(*MaybeRHS));`.
  **L442 CN**: 执行语句 `B.buildVScale(Dst, LHSVScale->getSrc().shl(*MaybeRHS));`。
- **L443 EN**: Closes the current scope.
  **L443 CN**: 关闭当前作用域。
- **L444 EN**: Separates nearby statements for readability.
  **L444 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L445 EN**: Returns `true` to the caller.
  **L445 CN**: 向调用者返回 `true`。
- **L446 EN**: Closes the current scope.
  **L446 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Register management** / **寄存器管理**
- **Machine instruction manipulation** / **机器指令操作**
- **GlobalISel pipeline** / **GlobalISel 流水线**
- **Debug information handling** / **调试信息处理**
- **SSA/PHI lowering** / **SSA/PHI 降低**
- **Target-specific hooks** / **目标相关钩子**
- **Garbage-collection support** / **垃圾回收支持**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/GlobalISel/CombinerHelper.h`, `llvm/CodeGen/GlobalISel/GenericMachineInstrs.h`, `llvm/CodeGen/GlobalISel/LegalizerHelper.h`, `llvm/CodeGen/GlobalISel/LegalizerInfo.h`, `llvm/CodeGen/GlobalISel/MIPatternMatch.h`, `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`, `llvm/CodeGen/GlobalISel/Utils.h`, `llvm/CodeGen/LowLevelTypeUtils.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/CodeGen/TargetOpcodes.h`, `llvm/Support/Casting.h`
- **System headers / 系统头文件**: `optional`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
