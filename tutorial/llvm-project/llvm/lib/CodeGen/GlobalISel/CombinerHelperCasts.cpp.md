# CombinerHelperCasts.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/GlobalISel/CombinerHelperCasts.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `See https://llvm.org/LICENSE.txt for license information` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“See https://llvm.org/LICENSE.txt for license information”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- CombinerHelperCasts.cpp---------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements CombinerHelper for G_ANYEXT, G_SEXT, G_TRUNC, and
// G_ZEXT
//
//===----------------------------------------------------------------------===//
#include "llvm/CodeGen/GlobalISel/CombinerHelper.h"
#include "llvm/CodeGen/GlobalISel/LegalizerHelper.h"
#include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/GlobalISel/Utils.h"
#include "llvm/CodeGen/LowLevelTypeUtils.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
````
- **L1 EN**: Comment documents: `===- CombinerHelperCasts.cpp--------------------------------------------…`.
  **L1 CN**: 注释说明：`===- CombinerHelperCasts.cpp--------------------------------------------…`。
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
- **L9 EN**: Comment documents: `This file implements CombinerHelper for G_ANYEXT, G_SEXT, G_TRUNC, and`.
  **L9 CN**: 注释说明：`This file implements CombinerHelper for G_ANYEXT, G_SEXT, G_TRUNC, and`。
- **L10 EN**: Comment documents: `G_ZEXT`.
  **L10 CN**: 注释说明：`G_ZEXT`。
- **L11 EN**: Continues the surrounding comment block.
  **L11 CN**: 延续周围的注释块。
- **L12 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L12 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L13 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/CombinerHelper.h` for CombinerHelper support.
  **L13 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/CombinerHelper.h`，用于 CombinerHelper 相关支持。
- **L14 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/LegalizerHelper.h` for LegalizerHelper support.
  **L14 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/LegalizerHelper.h`，用于 LegalizerHelper 相关支持。
- **L15 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/LegalizerInfo.h` for LegalizerInfo support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/LegalizerInfo.h`，用于 LegalizerInfo 相关支持。
- **L16 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/MachineIRBuilder.h` for MachineIRBuilder support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`，用于 MachineIRBuilder 相关支持。
- **L17 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/Utils.h` for Utils support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/Utils.h`，用于 Utils 相关支持。
- **L18 EN**: Includes LLVM header `llvm/CodeGen/LowLevelTypeUtils.h` for LowLevelTypeUtils support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/CodeGen/LowLevelTypeUtils.h`，用于 LowLevelTypeUtils 相关支持。
- **L19 EN**: Includes LLVM header `llvm/CodeGen/MachineOperand.h` for MachineOperand support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineOperand.h`，用于 MachineOperand 相关支持。
- **L20 EN**: Includes LLVM header `llvm/CodeGen/MachineRegisterInfo.h` for MachineRegisterInfo support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineRegisterInfo.h`，用于 MachineRegisterInfo 相关支持。

### Lines 21-40

````cpp
#include "llvm/CodeGen/TargetOpcodes.h"
#include "llvm/Support/Casting.h"

#define DEBUG_TYPE "gi-combiner"

using namespace llvm;

bool CombinerHelper::matchSextOfTrunc(const MachineOperand &MO,
                                      BuildFnTy &MatchInfo) const {
  GSext *Sext = cast<GSext>(getDefIgnoringCopies(MO.getReg(), MRI));
  GTrunc *Trunc = cast<GTrunc>(getDefIgnoringCopies(Sext->getSrcReg(), MRI));

  Register Dst = Sext->getReg(0);
  Register Src = Trunc->getSrcReg();

  LLT DstTy = MRI.getType(Dst);
  LLT SrcTy = MRI.getType(Src);

  // Combines without nsw trunc.
  if (!Trunc->getFlag(MachineInstr::NoSWrap)) {
````
- **L21 EN**: Includes LLVM header `llvm/CodeGen/TargetOpcodes.h` for TargetOpcodes support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetOpcodes.h`，用于 TargetOpcodes 相关支持。
- **L22 EN**: Includes LLVM header `llvm/Support/Casting.h` for Casting support.
  **L22 CN**: 引入 LLVM 头文件 `llvm/Support/Casting.h`，用于 Casting 相关支持。
- **L23 EN**: Separates nearby statements for readability.
  **L23 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L24 EN**: Defines the LLVM debug channel used by this file.
  **L24 CN**: 定义该文件使用的 LLVM 调试通道。
- **L25 EN**: Separates nearby statements for readability.
  **L25 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L26 EN**: Imports namespace `llvm` into this translation unit.
  **L26 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L27 EN**: Separates nearby statements for readability.
  **L27 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L28 EN**: Provides part of the signature for `matchSextOfTrunc`.
  **L28 CN**: 给出 `matchSextOfTrunc` 的一部分签名。
- **L29 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L29 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L30 EN**: Assigns or initializes `GSext *Sext`.
  **L30 CN**: 对 `GSext *Sext` 进行赋值或初始化。
- **L31 EN**: Assigns or initializes `GTrunc *Trunc`.
  **L31 CN**: 对 `GTrunc *Trunc` 进行赋值或初始化。
- **L32 EN**: Separates nearby statements for readability.
  **L32 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L33 EN**: Assigns or initializes `Register Dst`.
  **L33 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L34 EN**: Assigns or initializes `Register Src`.
  **L34 CN**: 对 `Register Src` 进行赋值或初始化。
- **L35 EN**: Separates nearby statements for readability.
  **L35 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L36 EN**: Assigns or initializes `LLT DstTy`.
  **L36 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L37 EN**: Assigns or initializes `LLT SrcTy`.
  **L37 CN**: 对 `LLT SrcTy` 进行赋值或初始化。
- **L38 EN**: Separates nearby statements for readability.
  **L38 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L39 EN**: Comment documents: `Combines without nsw trunc.`.
  **L39 CN**: 注释说明：`Combines without nsw trunc.`。
- **L40 EN**: Begins a conditional branch.
  **L40 CN**: 开始一个条件分支。

### Lines 41-60

````cpp
    if (DstTy != SrcTy ||
        !isLegalOrBeforeLegalizer({TargetOpcode::G_SEXT_INREG, {DstTy, SrcTy}}))
      return false;

    // Do this for 8 bit values and up. We don't want to do it for e.g. G_TRUNC
    // to i1.
    unsigned TruncWidth = MRI.getType(Trunc->getReg(0)).getScalarSizeInBits();
    if (TruncWidth < 8)
      return false;

    MatchInfo = [=](MachineIRBuilder &B) {
      B.buildSExtInReg(Dst, Src, TruncWidth);
    };
    return true;
  }

  // Combines for nsw trunc.

  if (DstTy == SrcTy) {
    MatchInfo = [=](MachineIRBuilder &B) { B.buildCopy(Dst, Src); };
````
- **L41 EN**: Begins a conditional branch.
  **L41 CN**: 开始一个条件分支。
- **L42 EN**: Continues logic with `!isLegalOrBeforeLegalizer({TargetOpcode::G_SEXT_INREG, {DstTy, SrcTy}}))`.
  **L42 CN**: 继续处理逻辑：`!isLegalOrBeforeLegalizer({TargetOpcode::G_SEXT_INREG, {DstTy, SrcTy}}))`。
- **L43 EN**: Returns `false` to the caller.
  **L43 CN**: 向调用者返回 `false`。
- **L44 EN**: Separates nearby statements for readability.
  **L44 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L45 EN**: Comment documents: `Do this for 8 bit values and up. We don't want to do it for e.g. G_TRUNC`.
  **L45 CN**: 注释说明：`Do this for 8 bit values and up. We don't want to do it for e.g. G_TRUNC`。
- **L46 EN**: Comment documents: `to i1.`.
  **L46 CN**: 注释说明：`to i1.`。
- **L47 EN**: Assigns or initializes `unsigned TruncWidth`.
  **L47 CN**: 对 `unsigned TruncWidth` 进行赋值或初始化。
- **L48 EN**: Begins a conditional branch.
  **L48 CN**: 开始一个条件分支。
- **L49 EN**: Returns `false` to the caller.
  **L49 CN**: 向调用者返回 `false`。
- **L50 EN**: Separates nearby statements for readability.
  **L50 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L51 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L51 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L52 EN**: Executes statement `B.buildSExtInReg(Dst, Src, TruncWidth);`.
  **L52 CN**: 执行语句 `B.buildSExtInReg(Dst, Src, TruncWidth);`。
- **L53 EN**: Closes the current scope.
  **L53 CN**: 关闭当前作用域。
- **L54 EN**: Returns `true` to the caller.
  **L54 CN**: 向调用者返回 `true`。
- **L55 EN**: Closes the current scope.
  **L55 CN**: 关闭当前作用域。
- **L56 EN**: Separates nearby statements for readability.
  **L56 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L57 EN**: Comment documents: `Combines for nsw trunc.`.
  **L57 CN**: 注释说明：`Combines for nsw trunc.`。
- **L58 EN**: Separates nearby statements for readability.
  **L58 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L59 EN**: Begins a conditional branch.
  **L59 CN**: 开始一个条件分支。
- **L60 EN**: Assigns or initializes `MatchInfo`.
  **L60 CN**: 对 `MatchInfo` 进行赋值或初始化。

### Lines 61-80

````cpp
    return true;
  }

  if (DstTy.getScalarSizeInBits() < SrcTy.getScalarSizeInBits() &&
      isLegalOrBeforeLegalizer({TargetOpcode::G_TRUNC, {DstTy, SrcTy}})) {
    MatchInfo = [=](MachineIRBuilder &B) {
      B.buildTrunc(Dst, Src, MachineInstr::MIFlag::NoSWrap);
    };
    return true;
  }

  if (DstTy.getScalarSizeInBits() > SrcTy.getScalarSizeInBits() &&
      isLegalOrBeforeLegalizer({TargetOpcode::G_SEXT, {DstTy, SrcTy}})) {
    MatchInfo = [=](MachineIRBuilder &B) { B.buildSExt(Dst, Src); };
    return true;
  }

  return false;
}

````
- **L61 EN**: Returns `true` to the caller.
  **L61 CN**: 向调用者返回 `true`。
- **L62 EN**: Closes the current scope.
  **L62 CN**: 关闭当前作用域。
- **L63 EN**: Separates nearby statements for readability.
  **L63 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L64 EN**: Begins a conditional branch.
  **L64 CN**: 开始一个条件分支。
- **L65 EN**: Starts block `isLegalOrBeforeLegalizer({TargetOpcode::G_TRUNC, {DstTy, SrcTy}}))`.
  **L65 CN**: 开始代码块 `isLegalOrBeforeLegalizer({TargetOpcode::G_TRUNC, {DstTy, SrcTy}}))`。
- **L66 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L66 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L67 EN**: Executes statement `B.buildTrunc(Dst, Src, MachineInstr::MIFlag::NoSWrap);`.
  **L67 CN**: 执行语句 `B.buildTrunc(Dst, Src, MachineInstr::MIFlag::NoSWrap);`。
- **L68 EN**: Closes the current scope.
  **L68 CN**: 关闭当前作用域。
- **L69 EN**: Returns `true` to the caller.
  **L69 CN**: 向调用者返回 `true`。
- **L70 EN**: Closes the current scope.
  **L70 CN**: 关闭当前作用域。
- **L71 EN**: Separates nearby statements for readability.
  **L71 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L72 EN**: Begins a conditional branch.
  **L72 CN**: 开始一个条件分支。
- **L73 EN**: Starts block `isLegalOrBeforeLegalizer({TargetOpcode::G_SEXT, {DstTy, SrcTy}}))`.
  **L73 CN**: 开始代码块 `isLegalOrBeforeLegalizer({TargetOpcode::G_SEXT, {DstTy, SrcTy}}))`。
- **L74 EN**: Assigns or initializes `MatchInfo`.
  **L74 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L75 EN**: Returns `true` to the caller.
  **L75 CN**: 向调用者返回 `true`。
- **L76 EN**: Closes the current scope.
  **L76 CN**: 关闭当前作用域。
- **L77 EN**: Separates nearby statements for readability.
  **L77 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L78 EN**: Returns `false` to the caller.
  **L78 CN**: 向调用者返回 `false`。
- **L79 EN**: Closes the current scope.
  **L79 CN**: 关闭当前作用域。
- **L80 EN**: Separates nearby statements for readability.
  **L80 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 81-100

````cpp
bool CombinerHelper::matchZextOfTrunc(const MachineOperand &MO,
                                      BuildFnTy &MatchInfo) const {
  GZext *Zext = cast<GZext>(getDefIgnoringCopies(MO.getReg(), MRI));
  GTrunc *Trunc = cast<GTrunc>(getDefIgnoringCopies(Zext->getSrcReg(), MRI));

  Register Dst = Zext->getReg(0);
  Register Src = Trunc->getSrcReg();

  LLT DstTy = MRI.getType(Dst);
  LLT SrcTy = MRI.getType(Src);

  if (DstTy == SrcTy) {
    MatchInfo = [=](MachineIRBuilder &B) { B.buildCopy(Dst, Src); };
    return true;
  }

  if (DstTy.getScalarSizeInBits() < SrcTy.getScalarSizeInBits() &&
      isLegalOrBeforeLegalizer({TargetOpcode::G_TRUNC, {DstTy, SrcTy}})) {
    MatchInfo = [=](MachineIRBuilder &B) {
      B.buildTrunc(Dst, Src, MachineInstr::MIFlag::NoUWrap);
````
- **L81 EN**: Provides part of the signature for `matchZextOfTrunc`.
  **L81 CN**: 给出 `matchZextOfTrunc` 的一部分签名。
- **L82 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L82 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L83 EN**: Assigns or initializes `GZext *Zext`.
  **L83 CN**: 对 `GZext *Zext` 进行赋值或初始化。
- **L84 EN**: Assigns or initializes `GTrunc *Trunc`.
  **L84 CN**: 对 `GTrunc *Trunc` 进行赋值或初始化。
- **L85 EN**: Separates nearby statements for readability.
  **L85 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L86 EN**: Assigns or initializes `Register Dst`.
  **L86 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L87 EN**: Assigns or initializes `Register Src`.
  **L87 CN**: 对 `Register Src` 进行赋值或初始化。
- **L88 EN**: Separates nearby statements for readability.
  **L88 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L89 EN**: Assigns or initializes `LLT DstTy`.
  **L89 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L90 EN**: Assigns or initializes `LLT SrcTy`.
  **L90 CN**: 对 `LLT SrcTy` 进行赋值或初始化。
- **L91 EN**: Separates nearby statements for readability.
  **L91 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L92 EN**: Begins a conditional branch.
  **L92 CN**: 开始一个条件分支。
- **L93 EN**: Assigns or initializes `MatchInfo`.
  **L93 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L94 EN**: Returns `true` to the caller.
  **L94 CN**: 向调用者返回 `true`。
- **L95 EN**: Closes the current scope.
  **L95 CN**: 关闭当前作用域。
- **L96 EN**: Separates nearby statements for readability.
  **L96 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L97 EN**: Begins a conditional branch.
  **L97 CN**: 开始一个条件分支。
- **L98 EN**: Starts block `isLegalOrBeforeLegalizer({TargetOpcode::G_TRUNC, {DstTy, SrcTy}}))`.
  **L98 CN**: 开始代码块 `isLegalOrBeforeLegalizer({TargetOpcode::G_TRUNC, {DstTy, SrcTy}}))`。
- **L99 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L99 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L100 EN**: Executes statement `B.buildTrunc(Dst, Src, MachineInstr::MIFlag::NoUWrap);`.
  **L100 CN**: 执行语句 `B.buildTrunc(Dst, Src, MachineInstr::MIFlag::NoUWrap);`。

### Lines 101-120

````cpp
    };
    return true;
  }

  if (DstTy.getScalarSizeInBits() > SrcTy.getScalarSizeInBits() &&
      isLegalOrBeforeLegalizer({TargetOpcode::G_ZEXT, {DstTy, SrcTy}})) {
    MatchInfo = [=](MachineIRBuilder &B) {
      B.buildZExt(Dst, Src, MachineInstr::MIFlag::NonNeg);
    };
    return true;
  }

  return false;
}

bool CombinerHelper::matchNonNegZext(const MachineOperand &MO,
                                     BuildFnTy &MatchInfo) const {
  GZext *Zext = cast<GZext>(MRI.getVRegDef(MO.getReg()));

  Register Dst = Zext->getReg(0);
````
- **L101 EN**: Closes the current scope.
  **L101 CN**: 关闭当前作用域。
- **L102 EN**: Returns `true` to the caller.
  **L102 CN**: 向调用者返回 `true`。
- **L103 EN**: Closes the current scope.
  **L103 CN**: 关闭当前作用域。
- **L104 EN**: Separates nearby statements for readability.
  **L104 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L105 EN**: Begins a conditional branch.
  **L105 CN**: 开始一个条件分支。
- **L106 EN**: Starts block `isLegalOrBeforeLegalizer({TargetOpcode::G_ZEXT, {DstTy, SrcTy}}))`.
  **L106 CN**: 开始代码块 `isLegalOrBeforeLegalizer({TargetOpcode::G_ZEXT, {DstTy, SrcTy}}))`。
- **L107 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L107 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L108 EN**: Executes statement `B.buildZExt(Dst, Src, MachineInstr::MIFlag::NonNeg);`.
  **L108 CN**: 执行语句 `B.buildZExt(Dst, Src, MachineInstr::MIFlag::NonNeg);`。
- **L109 EN**: Closes the current scope.
  **L109 CN**: 关闭当前作用域。
- **L110 EN**: Returns `true` to the caller.
  **L110 CN**: 向调用者返回 `true`。
- **L111 EN**: Closes the current scope.
  **L111 CN**: 关闭当前作用域。
- **L112 EN**: Separates nearby statements for readability.
  **L112 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L113 EN**: Returns `false` to the caller.
  **L113 CN**: 向调用者返回 `false`。
- **L114 EN**: Closes the current scope.
  **L114 CN**: 关闭当前作用域。
- **L115 EN**: Separates nearby statements for readability.
  **L115 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L116 EN**: Provides part of the signature for `matchNonNegZext`.
  **L116 CN**: 给出 `matchNonNegZext` 的一部分签名。
- **L117 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L117 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L118 EN**: Assigns or initializes `GZext *Zext`.
  **L118 CN**: 对 `GZext *Zext` 进行赋值或初始化。
- **L119 EN**: Separates nearby statements for readability.
  **L119 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L120 EN**: Assigns or initializes `Register Dst`.
  **L120 CN**: 对 `Register Dst` 进行赋值或初始化。

### Lines 121-140

````cpp
  Register Src = Zext->getSrcReg();

  LLT DstTy = MRI.getType(Dst);
  LLT SrcTy = MRI.getType(Src);
  const auto &TLI = getTargetLowering();

  // Convert zext nneg to sext if sext is the preferred form for the target.
  if (isLegalOrBeforeLegalizer({TargetOpcode::G_SEXT, {DstTy, SrcTy}}) &&
      TLI.isSExtCheaperThanZExt(getMVTForLLT(SrcTy), getMVTForLLT(DstTy))) {
    MatchInfo = [=](MachineIRBuilder &B) { B.buildSExt(Dst, Src); };
    return true;
  }

  return false;
}

bool CombinerHelper::matchTruncateOfExt(const MachineInstr &Root,
                                        const MachineInstr &ExtMI,
                                        BuildFnTy &MatchInfo) const {
  const GTrunc *Trunc = cast<GTrunc>(&Root);
````
- **L121 EN**: Assigns or initializes `Register Src`.
  **L121 CN**: 对 `Register Src` 进行赋值或初始化。
- **L122 EN**: Separates nearby statements for readability.
  **L122 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L123 EN**: Assigns or initializes `LLT DstTy`.
  **L123 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L124 EN**: Assigns or initializes `LLT SrcTy`.
  **L124 CN**: 对 `LLT SrcTy` 进行赋值或初始化。
- **L125 EN**: Assigns or initializes `const auto &TLI`.
  **L125 CN**: 对 `const auto &TLI` 进行赋值或初始化。
- **L126 EN**: Separates nearby statements for readability.
  **L126 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L127 EN**: Comment documents: `Convert zext nneg to sext if sext is the preferred form for the target.`.
  **L127 CN**: 注释说明：`Convert zext nneg to sext if sext is the preferred form for the target.`。
- **L128 EN**: Begins a conditional branch.
  **L128 CN**: 开始一个条件分支。
- **L129 EN**: Starts block `TLI.isSExtCheaperThanZExt(getMVTForLLT(SrcTy), getMVTForLLT(DstTy)))`.
  **L129 CN**: 开始代码块 `TLI.isSExtCheaperThanZExt(getMVTForLLT(SrcTy), getMVTForLLT(DstTy)))`。
- **L130 EN**: Assigns or initializes `MatchInfo`.
  **L130 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L131 EN**: Returns `true` to the caller.
  **L131 CN**: 向调用者返回 `true`。
- **L132 EN**: Closes the current scope.
  **L132 CN**: 关闭当前作用域。
- **L133 EN**: Separates nearby statements for readability.
  **L133 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L134 EN**: Returns `false` to the caller.
  **L134 CN**: 向调用者返回 `false`。
- **L135 EN**: Closes the current scope.
  **L135 CN**: 关闭当前作用域。
- **L136 EN**: Separates nearby statements for readability.
  **L136 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L137 EN**: Provides part of the signature for `matchTruncateOfExt`.
  **L137 CN**: 给出 `matchTruncateOfExt` 的一部分签名。
- **L138 EN**: Continues logic with `const MachineInstr &ExtMI,`.
  **L138 CN**: 继续处理逻辑：`const MachineInstr &ExtMI,`。
- **L139 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L139 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L140 EN**: Assigns or initializes `const GTrunc *Trunc`.
  **L140 CN**: 对 `const GTrunc *Trunc` 进行赋值或初始化。

### Lines 141-160

````cpp
  const GExtOp *Ext = cast<GExtOp>(&ExtMI);

  if (!MRI.hasOneNonDBGUse(Ext->getReg(0)))
    return false;

  Register Dst = Trunc->getReg(0);
  Register Src = Ext->getSrcReg();
  LLT DstTy = MRI.getType(Dst);
  LLT SrcTy = MRI.getType(Src);

  if (SrcTy == DstTy) {
    // The source and the destination are equally sized. We need to copy.
    MatchInfo = [=](MachineIRBuilder &B) { B.buildCopy(Dst, Src); };

    return true;
  }

  if (SrcTy.getScalarSizeInBits() < DstTy.getScalarSizeInBits()) {
    // If the source is smaller than the destination, we need to extend.

````
- **L141 EN**: Assigns or initializes `const GExtOp *Ext`.
  **L141 CN**: 对 `const GExtOp *Ext` 进行赋值或初始化。
- **L142 EN**: Separates nearby statements for readability.
  **L142 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L143 EN**: Begins a conditional branch.
  **L143 CN**: 开始一个条件分支。
- **L144 EN**: Returns `false` to the caller.
  **L144 CN**: 向调用者返回 `false`。
- **L145 EN**: Separates nearby statements for readability.
  **L145 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L146 EN**: Assigns or initializes `Register Dst`.
  **L146 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L147 EN**: Assigns or initializes `Register Src`.
  **L147 CN**: 对 `Register Src` 进行赋值或初始化。
- **L148 EN**: Assigns or initializes `LLT DstTy`.
  **L148 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L149 EN**: Assigns or initializes `LLT SrcTy`.
  **L149 CN**: 对 `LLT SrcTy` 进行赋值或初始化。
- **L150 EN**: Separates nearby statements for readability.
  **L150 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L151 EN**: Begins a conditional branch.
  **L151 CN**: 开始一个条件分支。
- **L152 EN**: Comment documents: `The source and the destination are equally sized. We need to copy.`.
  **L152 CN**: 注释说明：`The source and the destination are equally sized. We need to copy.`。
- **L153 EN**: Assigns or initializes `MatchInfo`.
  **L153 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L154 EN**: Separates nearby statements for readability.
  **L154 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L155 EN**: Returns `true` to the caller.
  **L155 CN**: 向调用者返回 `true`。
- **L156 EN**: Closes the current scope.
  **L156 CN**: 关闭当前作用域。
- **L157 EN**: Separates nearby statements for readability.
  **L157 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L158 EN**: Begins a conditional branch.
  **L158 CN**: 开始一个条件分支。
- **L159 EN**: Comment documents: `If the source is smaller than the destination, we need to extend.`.
  **L159 CN**: 注释说明：`If the source is smaller than the destination, we need to extend.`。
- **L160 EN**: Separates nearby statements for readability.
  **L160 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 161-180

````cpp
    if (!isLegalOrBeforeLegalizer({Ext->getOpcode(), {DstTy, SrcTy}}))
      return false;

    MatchInfo = [=](MachineIRBuilder &B) {
      B.buildInstr(Ext->getOpcode(), {Dst}, {Src});
    };

    return true;
  }

  if (SrcTy.getScalarSizeInBits() > DstTy.getScalarSizeInBits()) {
    // If the source is larger than the destination, then we need to truncate.

    if (!isLegalOrBeforeLegalizer({TargetOpcode::G_TRUNC, {DstTy, SrcTy}}))
      return false;

    MatchInfo = [=](MachineIRBuilder &B) { B.buildTrunc(Dst, Src); };

    return true;
  }
````
- **L161 EN**: Begins a conditional branch.
  **L161 CN**: 开始一个条件分支。
- **L162 EN**: Returns `false` to the caller.
  **L162 CN**: 向调用者返回 `false`。
- **L163 EN**: Separates nearby statements for readability.
  **L163 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L164 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L164 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L165 EN**: Executes statement `B.buildInstr(Ext->getOpcode(), {Dst}, {Src});`.
  **L165 CN**: 执行语句 `B.buildInstr(Ext->getOpcode(), {Dst}, {Src});`。
- **L166 EN**: Closes the current scope.
  **L166 CN**: 关闭当前作用域。
- **L167 EN**: Separates nearby statements for readability.
  **L167 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L168 EN**: Returns `true` to the caller.
  **L168 CN**: 向调用者返回 `true`。
- **L169 EN**: Closes the current scope.
  **L169 CN**: 关闭当前作用域。
- **L170 EN**: Separates nearby statements for readability.
  **L170 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L171 EN**: Begins a conditional branch.
  **L171 CN**: 开始一个条件分支。
- **L172 EN**: Comment documents: `If the source is larger than the destination, then we need to truncate.`.
  **L172 CN**: 注释说明：`If the source is larger than the destination, then we need to truncate.`。
- **L173 EN**: Separates nearby statements for readability.
  **L173 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L174 EN**: Begins a conditional branch.
  **L174 CN**: 开始一个条件分支。
- **L175 EN**: Returns `false` to the caller.
  **L175 CN**: 向调用者返回 `false`。
- **L176 EN**: Separates nearby statements for readability.
  **L176 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L177 EN**: Assigns or initializes `MatchInfo`.
  **L177 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L178 EN**: Separates nearby statements for readability.
  **L178 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L179 EN**: Returns `true` to the caller.
  **L179 CN**: 向调用者返回 `true`。
- **L180 EN**: Closes the current scope.
  **L180 CN**: 关闭当前作用域。

### Lines 181-200

````cpp

  return false;
}

bool CombinerHelper::isCastFree(unsigned Opcode, LLT ToTy, LLT FromTy) const {
  const TargetLowering &TLI = getTargetLowering();
  LLVMContext &Ctx = getContext();

  switch (Opcode) {
  case TargetOpcode::G_ANYEXT:
  case TargetOpcode::G_ZEXT:
    return TLI.isZExtFree(FromTy, ToTy, Ctx);
  case TargetOpcode::G_TRUNC:
    return TLI.isTruncateFree(FromTy, ToTy, Ctx);
  default:
    return false;
  }
}

bool CombinerHelper::matchCastOfSelect(const MachineInstr &CastMI,
````
- **L181 EN**: Separates nearby statements for readability.
  **L181 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L182 EN**: Returns `false` to the caller.
  **L182 CN**: 向调用者返回 `false`。
- **L183 EN**: Closes the current scope.
  **L183 CN**: 关闭当前作用域。
- **L184 EN**: Separates nearby statements for readability.
  **L184 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L185 EN**: Begins the definition of `isCastFree`.
  **L185 CN**: 开始定义 `isCastFree`。
- **L186 EN**: Assigns or initializes `const TargetLowering &TLI`.
  **L186 CN**: 对 `const TargetLowering &TLI` 进行赋值或初始化。
- **L187 EN**: Assigns or initializes `LLVMContext &Ctx`.
  **L187 CN**: 对 `LLVMContext &Ctx` 进行赋值或初始化。
- **L188 EN**: Separates nearby statements for readability.
  **L188 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L189 EN**: Starts a multi-way branch.
  **L189 CN**: 开始一个多路分支。
- **L190 EN**: Handles one switch case.
  **L190 CN**: 处理一个 switch 分支。
- **L191 EN**: Handles one switch case.
  **L191 CN**: 处理一个 switch 分支。
- **L192 EN**: Returns `TLI.isZExtFree(FromTy, ToTy, Ctx)` to the caller.
  **L192 CN**: 向调用者返回 `TLI.isZExtFree(FromTy, ToTy, Ctx)`。
- **L193 EN**: Handles one switch case.
  **L193 CN**: 处理一个 switch 分支。
- **L194 EN**: Returns `TLI.isTruncateFree(FromTy, ToTy, Ctx)` to the caller.
  **L194 CN**: 向调用者返回 `TLI.isTruncateFree(FromTy, ToTy, Ctx)`。
- **L195 EN**: Handles the default switch case.
  **L195 CN**: 处理 switch 的默认分支。
- **L196 EN**: Returns `false` to the caller.
  **L196 CN**: 向调用者返回 `false`。
- **L197 EN**: Closes the current scope.
  **L197 CN**: 关闭当前作用域。
- **L198 EN**: Closes the current scope.
  **L198 CN**: 关闭当前作用域。
- **L199 EN**: Separates nearby statements for readability.
  **L199 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L200 EN**: Provides part of the signature for `matchCastOfSelect`.
  **L200 CN**: 给出 `matchCastOfSelect` 的一部分签名。

### Lines 201-220

````cpp
                                       const MachineInstr &SelectMI,
                                       BuildFnTy &MatchInfo) const {
  const GExtOrTruncOp *Cast = cast<GExtOrTruncOp>(&CastMI);
  const GSelect *Select = cast<GSelect>(&SelectMI);

  if (!MRI.hasOneNonDBGUse(Select->getReg(0)))
    return false;

  Register Dst = Cast->getReg(0);
  LLT DstTy = MRI.getType(Dst);
  LLT CondTy = MRI.getType(Select->getCondReg());
  Register TrueReg = Select->getTrueReg();
  Register FalseReg = Select->getFalseReg();
  LLT SrcTy = MRI.getType(TrueReg);
  Register Cond = Select->getCondReg();

  if (!isLegalOrBeforeLegalizer({TargetOpcode::G_SELECT, {DstTy, CondTy}}))
    return false;

  if (!isCastFree(Cast->getOpcode(), DstTy, SrcTy))
````
- **L201 EN**: Continues logic with `const MachineInstr &SelectMI,`.
  **L201 CN**: 继续处理逻辑：`const MachineInstr &SelectMI,`。
- **L202 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L202 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L203 EN**: Assigns or initializes `const GExtOrTruncOp *Cast`.
  **L203 CN**: 对 `const GExtOrTruncOp *Cast` 进行赋值或初始化。
- **L204 EN**: Assigns or initializes `const GSelect *Select`.
  **L204 CN**: 对 `const GSelect *Select` 进行赋值或初始化。
- **L205 EN**: Separates nearby statements for readability.
  **L205 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L206 EN**: Begins a conditional branch.
  **L206 CN**: 开始一个条件分支。
- **L207 EN**: Returns `false` to the caller.
  **L207 CN**: 向调用者返回 `false`。
- **L208 EN**: Separates nearby statements for readability.
  **L208 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L209 EN**: Assigns or initializes `Register Dst`.
  **L209 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L210 EN**: Assigns or initializes `LLT DstTy`.
  **L210 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L211 EN**: Assigns or initializes `LLT CondTy`.
  **L211 CN**: 对 `LLT CondTy` 进行赋值或初始化。
- **L212 EN**: Assigns or initializes `Register TrueReg`.
  **L212 CN**: 对 `Register TrueReg` 进行赋值或初始化。
- **L213 EN**: Assigns or initializes `Register FalseReg`.
  **L213 CN**: 对 `Register FalseReg` 进行赋值或初始化。
- **L214 EN**: Assigns or initializes `LLT SrcTy`.
  **L214 CN**: 对 `LLT SrcTy` 进行赋值或初始化。
- **L215 EN**: Assigns or initializes `Register Cond`.
  **L215 CN**: 对 `Register Cond` 进行赋值或初始化。
- **L216 EN**: Separates nearby statements for readability.
  **L216 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L217 EN**: Begins a conditional branch.
  **L217 CN**: 开始一个条件分支。
- **L218 EN**: Returns `false` to the caller.
  **L218 CN**: 向调用者返回 `false`。
- **L219 EN**: Separates nearby statements for readability.
  **L219 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L220 EN**: Begins a conditional branch.
  **L220 CN**: 开始一个条件分支。

### Lines 221-240

````cpp
    return false;

  MatchInfo = [=](MachineIRBuilder &B) {
    auto True = B.buildInstr(Cast->getOpcode(), {DstTy}, {TrueReg});
    auto False = B.buildInstr(Cast->getOpcode(), {DstTy}, {FalseReg});
    B.buildSelect(Dst, Cond, True, False);
  };

  return true;
}

bool CombinerHelper::matchExtOfExt(const MachineInstr &FirstMI,
                                   const MachineInstr &SecondMI,
                                   BuildFnTy &MatchInfo) const {
  const GExtOp *First = cast<GExtOp>(&FirstMI);
  const GExtOp *Second = cast<GExtOp>(&SecondMI);

  Register Dst = First->getReg(0);
  Register Src = Second->getSrcReg();
  LLT DstTy = MRI.getType(Dst);
````
- **L221 EN**: Returns `false` to the caller.
  **L221 CN**: 向调用者返回 `false`。
- **L222 EN**: Separates nearby statements for readability.
  **L222 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L223 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L223 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L224 EN**: Assigns or initializes `auto True`.
  **L224 CN**: 对 `auto True` 进行赋值或初始化。
- **L225 EN**: Assigns or initializes `auto False`.
  **L225 CN**: 对 `auto False` 进行赋值或初始化。
- **L226 EN**: Executes statement `B.buildSelect(Dst, Cond, True, False);`.
  **L226 CN**: 执行语句 `B.buildSelect(Dst, Cond, True, False);`。
- **L227 EN**: Closes the current scope.
  **L227 CN**: 关闭当前作用域。
- **L228 EN**: Separates nearby statements for readability.
  **L228 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L229 EN**: Returns `true` to the caller.
  **L229 CN**: 向调用者返回 `true`。
- **L230 EN**: Closes the current scope.
  **L230 CN**: 关闭当前作用域。
- **L231 EN**: Separates nearby statements for readability.
  **L231 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L232 EN**: Provides part of the signature for `matchExtOfExt`.
  **L232 CN**: 给出 `matchExtOfExt` 的一部分签名。
- **L233 EN**: Continues logic with `const MachineInstr &SecondMI,`.
  **L233 CN**: 继续处理逻辑：`const MachineInstr &SecondMI,`。
- **L234 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L234 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L235 EN**: Assigns or initializes `const GExtOp *First`.
  **L235 CN**: 对 `const GExtOp *First` 进行赋值或初始化。
- **L236 EN**: Assigns or initializes `const GExtOp *Second`.
  **L236 CN**: 对 `const GExtOp *Second` 进行赋值或初始化。
- **L237 EN**: Separates nearby statements for readability.
  **L237 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L238 EN**: Assigns or initializes `Register Dst`.
  **L238 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L239 EN**: Assigns or initializes `Register Src`.
  **L239 CN**: 对 `Register Src` 进行赋值或初始化。
- **L240 EN**: Assigns or initializes `LLT DstTy`.
  **L240 CN**: 对 `LLT DstTy` 进行赋值或初始化。

### Lines 241-260

````cpp
  LLT SrcTy = MRI.getType(Src);

  if (!MRI.hasOneNonDBGUse(Second->getReg(0)))
    return false;

  // ext of ext -> later ext
  if (First->getOpcode() == Second->getOpcode() &&
      isLegalOrBeforeLegalizer({Second->getOpcode(), {DstTy, SrcTy}})) {
    if (Second->getOpcode() == TargetOpcode::G_ZEXT) {
      MachineInstr::MIFlag Flag = MachineInstr::MIFlag::NoFlags;
      if (Second->getFlag(MachineInstr::MIFlag::NonNeg))
        Flag = MachineInstr::MIFlag::NonNeg;
      MatchInfo = [=](MachineIRBuilder &B) { B.buildZExt(Dst, Src, Flag); };
      return true;
    }
    // not zext -> no flags
    MatchInfo = [=](MachineIRBuilder &B) {
      B.buildInstr(Second->getOpcode(), {Dst}, {Src});
    };
    return true;
````
- **L241 EN**: Assigns or initializes `LLT SrcTy`.
  **L241 CN**: 对 `LLT SrcTy` 进行赋值或初始化。
- **L242 EN**: Separates nearby statements for readability.
  **L242 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L243 EN**: Begins a conditional branch.
  **L243 CN**: 开始一个条件分支。
- **L244 EN**: Returns `false` to the caller.
  **L244 CN**: 向调用者返回 `false`。
- **L245 EN**: Separates nearby statements for readability.
  **L245 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L246 EN**: Comment documents: `ext of ext -> later ext`.
  **L246 CN**: 注释说明：`ext of ext -> later ext`。
- **L247 EN**: Begins a conditional branch.
  **L247 CN**: 开始一个条件分支。
- **L248 EN**: Starts block `isLegalOrBeforeLegalizer({Second->getOpcode(), {DstTy, SrcTy}}))`.
  **L248 CN**: 开始代码块 `isLegalOrBeforeLegalizer({Second->getOpcode(), {DstTy, SrcTy}}))`。
- **L249 EN**: Begins a conditional branch.
  **L249 CN**: 开始一个条件分支。
- **L250 EN**: Assigns or initializes `MachineInstr::MIFlag Flag`.
  **L250 CN**: 对 `MachineInstr::MIFlag Flag` 进行赋值或初始化。
- **L251 EN**: Begins a conditional branch.
  **L251 CN**: 开始一个条件分支。
- **L252 EN**: Assigns or initializes `Flag`.
  **L252 CN**: 对 `Flag` 进行赋值或初始化。
- **L253 EN**: Assigns or initializes `MatchInfo`.
  **L253 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L254 EN**: Returns `true` to the caller.
  **L254 CN**: 向调用者返回 `true`。
- **L255 EN**: Closes the current scope.
  **L255 CN**: 关闭当前作用域。
- **L256 EN**: Comment documents: `not zext -> no flags`.
  **L256 CN**: 注释说明：`not zext -> no flags`。
- **L257 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L257 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L258 EN**: Executes statement `B.buildInstr(Second->getOpcode(), {Dst}, {Src});`.
  **L258 CN**: 执行语句 `B.buildInstr(Second->getOpcode(), {Dst}, {Src});`。
- **L259 EN**: Closes the current scope.
  **L259 CN**: 关闭当前作用域。
- **L260 EN**: Returns `true` to the caller.
  **L260 CN**: 向调用者返回 `true`。

### Lines 261-280

````cpp
  }

  // anyext of sext/zext  -> sext/zext
  // -> pick anyext as second ext, then ext of ext
  if (First->getOpcode() == TargetOpcode::G_ANYEXT &&
      isLegalOrBeforeLegalizer({Second->getOpcode(), {DstTy, SrcTy}})) {
    if (Second->getOpcode() == TargetOpcode::G_ZEXT) {
      MachineInstr::MIFlag Flag = MachineInstr::MIFlag::NoFlags;
      if (Second->getFlag(MachineInstr::MIFlag::NonNeg))
        Flag = MachineInstr::MIFlag::NonNeg;
      MatchInfo = [=](MachineIRBuilder &B) { B.buildZExt(Dst, Src, Flag); };
      return true;
    }
    MatchInfo = [=](MachineIRBuilder &B) { B.buildSExt(Dst, Src); };
    return true;
  }

  // sext/zext of anyext -> sext/zext
  // -> pick anyext as first ext, then ext of ext
  if (Second->getOpcode() == TargetOpcode::G_ANYEXT &&
````
- **L261 EN**: Closes the current scope.
  **L261 CN**: 关闭当前作用域。
- **L262 EN**: Separates nearby statements for readability.
  **L262 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L263 EN**: Comment documents: `anyext of sext/zext -> sext/zext`.
  **L263 CN**: 注释说明：`anyext of sext/zext -> sext/zext`。
- **L264 EN**: Comment documents: `-> pick anyext as second ext, then ext of ext`.
  **L264 CN**: 注释说明：`-> pick anyext as second ext, then ext of ext`。
- **L265 EN**: Begins a conditional branch.
  **L265 CN**: 开始一个条件分支。
- **L266 EN**: Starts block `isLegalOrBeforeLegalizer({Second->getOpcode(), {DstTy, SrcTy}}))`.
  **L266 CN**: 开始代码块 `isLegalOrBeforeLegalizer({Second->getOpcode(), {DstTy, SrcTy}}))`。
- **L267 EN**: Begins a conditional branch.
  **L267 CN**: 开始一个条件分支。
- **L268 EN**: Assigns or initializes `MachineInstr::MIFlag Flag`.
  **L268 CN**: 对 `MachineInstr::MIFlag Flag` 进行赋值或初始化。
- **L269 EN**: Begins a conditional branch.
  **L269 CN**: 开始一个条件分支。
- **L270 EN**: Assigns or initializes `Flag`.
  **L270 CN**: 对 `Flag` 进行赋值或初始化。
- **L271 EN**: Assigns or initializes `MatchInfo`.
  **L271 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L272 EN**: Returns `true` to the caller.
  **L272 CN**: 向调用者返回 `true`。
- **L273 EN**: Closes the current scope.
  **L273 CN**: 关闭当前作用域。
- **L274 EN**: Assigns or initializes `MatchInfo`.
  **L274 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L275 EN**: Returns `true` to the caller.
  **L275 CN**: 向调用者返回 `true`。
- **L276 EN**: Closes the current scope.
  **L276 CN**: 关闭当前作用域。
- **L277 EN**: Separates nearby statements for readability.
  **L277 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L278 EN**: Comment documents: `sext/zext of anyext -> sext/zext`.
  **L278 CN**: 注释说明：`sext/zext of anyext -> sext/zext`。
- **L279 EN**: Comment documents: `-> pick anyext as first ext, then ext of ext`.
  **L279 CN**: 注释说明：`-> pick anyext as first ext, then ext of ext`。
- **L280 EN**: Begins a conditional branch.
  **L280 CN**: 开始一个条件分支。

### Lines 281-300

````cpp
      isLegalOrBeforeLegalizer({First->getOpcode(), {DstTy, SrcTy}})) {
    if (First->getOpcode() == TargetOpcode::G_ZEXT) {
      MachineInstr::MIFlag Flag = MachineInstr::MIFlag::NoFlags;
      if (First->getFlag(MachineInstr::MIFlag::NonNeg))
        Flag = MachineInstr::MIFlag::NonNeg;
      MatchInfo = [=](MachineIRBuilder &B) { B.buildZExt(Dst, Src, Flag); };
      return true;
    }
    MatchInfo = [=](MachineIRBuilder &B) { B.buildSExt(Dst, Src); };
    return true;
  }

  return false;
}

bool CombinerHelper::matchCastOfBuildVector(const MachineInstr &CastMI,
                                            const MachineInstr &BVMI,
                                            BuildFnTy &MatchInfo) const {
  const GExtOrTruncOp *Cast = cast<GExtOrTruncOp>(&CastMI);
  const GBuildVector *BV = cast<GBuildVector>(&BVMI);
````
- **L281 EN**: Starts block `isLegalOrBeforeLegalizer({First->getOpcode(), {DstTy, SrcTy}}))`.
  **L281 CN**: 开始代码块 `isLegalOrBeforeLegalizer({First->getOpcode(), {DstTy, SrcTy}}))`。
- **L282 EN**: Begins a conditional branch.
  **L282 CN**: 开始一个条件分支。
- **L283 EN**: Assigns or initializes `MachineInstr::MIFlag Flag`.
  **L283 CN**: 对 `MachineInstr::MIFlag Flag` 进行赋值或初始化。
- **L284 EN**: Begins a conditional branch.
  **L284 CN**: 开始一个条件分支。
- **L285 EN**: Assigns or initializes `Flag`.
  **L285 CN**: 对 `Flag` 进行赋值或初始化。
- **L286 EN**: Assigns or initializes `MatchInfo`.
  **L286 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L287 EN**: Returns `true` to the caller.
  **L287 CN**: 向调用者返回 `true`。
- **L288 EN**: Closes the current scope.
  **L288 CN**: 关闭当前作用域。
- **L289 EN**: Assigns or initializes `MatchInfo`.
  **L289 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L290 EN**: Returns `true` to the caller.
  **L290 CN**: 向调用者返回 `true`。
- **L291 EN**: Closes the current scope.
  **L291 CN**: 关闭当前作用域。
- **L292 EN**: Separates nearby statements for readability.
  **L292 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L293 EN**: Returns `false` to the caller.
  **L293 CN**: 向调用者返回 `false`。
- **L294 EN**: Closes the current scope.
  **L294 CN**: 关闭当前作用域。
- **L295 EN**: Separates nearby statements for readability.
  **L295 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L296 EN**: Provides part of the signature for `matchCastOfBuildVector`.
  **L296 CN**: 给出 `matchCastOfBuildVector` 的一部分签名。
- **L297 EN**: Continues logic with `const MachineInstr &BVMI,`.
  **L297 CN**: 继续处理逻辑：`const MachineInstr &BVMI,`。
- **L298 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L298 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L299 EN**: Assigns or initializes `const GExtOrTruncOp *Cast`.
  **L299 CN**: 对 `const GExtOrTruncOp *Cast` 进行赋值或初始化。
- **L300 EN**: Assigns or initializes `const GBuildVector *BV`.
  **L300 CN**: 对 `const GBuildVector *BV` 进行赋值或初始化。

### Lines 301-320

````cpp

  if (!MRI.hasOneNonDBGUse(BV->getReg(0)))
    return false;

  Register Dst = Cast->getReg(0);
  // The type of the new build vector.
  LLT DstTy = MRI.getType(Dst);
  // The scalar or element type of the new build vector.
  LLT ElemTy = DstTy.getScalarType();
  // The scalar or element type of the old build vector.
  LLT InputElemTy = MRI.getType(BV->getReg(0)).getElementType();

  // Check legality of new build vector, the scalar casts, and profitability of
  // the many casts.
  if (!isLegalOrBeforeLegalizer(
          {TargetOpcode::G_BUILD_VECTOR, {DstTy, ElemTy}}) ||
      !isLegalOrBeforeLegalizer({Cast->getOpcode(), {ElemTy, InputElemTy}}) ||
      !isCastFree(Cast->getOpcode(), ElemTy, InputElemTy))
    return false;

````
- **L301 EN**: Separates nearby statements for readability.
  **L301 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L302 EN**: Begins a conditional branch.
  **L302 CN**: 开始一个条件分支。
- **L303 EN**: Returns `false` to the caller.
  **L303 CN**: 向调用者返回 `false`。
- **L304 EN**: Separates nearby statements for readability.
  **L304 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L305 EN**: Assigns or initializes `Register Dst`.
  **L305 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L306 EN**: Comment documents: `The type of the new build vector.`.
  **L306 CN**: 注释说明：`The type of the new build vector.`。
- **L307 EN**: Assigns or initializes `LLT DstTy`.
  **L307 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L308 EN**: Comment documents: `The scalar or element type of the new build vector.`.
  **L308 CN**: 注释说明：`The scalar or element type of the new build vector.`。
- **L309 EN**: Assigns or initializes `LLT ElemTy`.
  **L309 CN**: 对 `LLT ElemTy` 进行赋值或初始化。
- **L310 EN**: Comment documents: `The scalar or element type of the old build vector.`.
  **L310 CN**: 注释说明：`The scalar or element type of the old build vector.`。
- **L311 EN**: Assigns or initializes `LLT InputElemTy`.
  **L311 CN**: 对 `LLT InputElemTy` 进行赋值或初始化。
- **L312 EN**: Separates nearby statements for readability.
  **L312 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L313 EN**: Comment documents: `Check legality of new build vector, the scalar casts, and profitability …`.
  **L313 CN**: 注释说明：`Check legality of new build vector, the scalar casts, and profitability …`。
- **L314 EN**: Comment documents: `the many casts.`.
  **L314 CN**: 注释说明：`the many casts.`。
- **L315 EN**: Begins a conditional branch.
  **L315 CN**: 开始一个条件分支。
- **L316 EN**: Continues logic with `{TargetOpcode::G_BUILD_VECTOR, {DstTy, ElemTy}}) ||`.
  **L316 CN**: 继续处理逻辑：`{TargetOpcode::G_BUILD_VECTOR, {DstTy, ElemTy}}) ||`。
- **L317 EN**: Continues logic with `!isLegalOrBeforeLegalizer({Cast->getOpcode(), {ElemTy, InputElemTy}}) ||`.
  **L317 CN**: 继续处理逻辑：`!isLegalOrBeforeLegalizer({Cast->getOpcode(), {ElemTy, InputElemTy}}) ||`。
- **L318 EN**: Continues logic with `!isCastFree(Cast->getOpcode(), ElemTy, InputElemTy))`.
  **L318 CN**: 继续处理逻辑：`!isCastFree(Cast->getOpcode(), ElemTy, InputElemTy))`。
- **L319 EN**: Returns `false` to the caller.
  **L319 CN**: 向调用者返回 `false`。
- **L320 EN**: Separates nearby statements for readability.
  **L320 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 321-340

````cpp
  MatchInfo = [=](MachineIRBuilder &B) {
    SmallVector<Register> Casts;
    unsigned Elements = BV->getNumSources();
    for (unsigned I = 0; I < Elements; ++I) {
      auto CastI =
          B.buildInstr(Cast->getOpcode(), {ElemTy}, {BV->getSourceReg(I)});
      Casts.push_back(CastI.getReg(0));
    }

    B.buildBuildVector(Dst, Casts);
  };

  return true;
}

bool CombinerHelper::matchNarrowBinop(const MachineInstr &TruncMI,
                                      const MachineInstr &BinopMI,
                                      BuildFnTy &MatchInfo) const {
  const GTrunc *Trunc = cast<GTrunc>(&TruncMI);
  const GBinOp *BinOp = cast<GBinOp>(&BinopMI);
````
- **L321 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L321 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L322 EN**: Executes statement `SmallVector<Register> Casts;`.
  **L322 CN**: 执行语句 `SmallVector<Register> Casts;`。
- **L323 EN**: Assigns or initializes `unsigned Elements`.
  **L323 CN**: 对 `unsigned Elements` 进行赋值或初始化。
- **L324 EN**: Starts a loop over a sequence or range.
  **L324 CN**: 开始遍历序列或范围的循环。
- **L325 EN**: Continues logic with `auto CastI =`.
  **L325 CN**: 继续处理逻辑：`auto CastI =`。
- **L326 EN**: Executes statement `B.buildInstr(Cast->getOpcode(), {ElemTy}, {BV->getSourceReg(I)});`.
  **L326 CN**: 执行语句 `B.buildInstr(Cast->getOpcode(), {ElemTy}, {BV->getSourceReg(I)});`。
- **L327 EN**: Executes statement `Casts.push_back(CastI.getReg(0));`.
  **L327 CN**: 执行语句 `Casts.push_back(CastI.getReg(0));`。
- **L328 EN**: Closes the current scope.
  **L328 CN**: 关闭当前作用域。
- **L329 EN**: Separates nearby statements for readability.
  **L329 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L330 EN**: Executes statement `B.buildBuildVector(Dst, Casts);`.
  **L330 CN**: 执行语句 `B.buildBuildVector(Dst, Casts);`。
- **L331 EN**: Closes the current scope.
  **L331 CN**: 关闭当前作用域。
- **L332 EN**: Separates nearby statements for readability.
  **L332 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L333 EN**: Returns `true` to the caller.
  **L333 CN**: 向调用者返回 `true`。
- **L334 EN**: Closes the current scope.
  **L334 CN**: 关闭当前作用域。
- **L335 EN**: Separates nearby statements for readability.
  **L335 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L336 EN**: Provides part of the signature for `matchNarrowBinop`.
  **L336 CN**: 给出 `matchNarrowBinop` 的一部分签名。
- **L337 EN**: Continues logic with `const MachineInstr &BinopMI,`.
  **L337 CN**: 继续处理逻辑：`const MachineInstr &BinopMI,`。
- **L338 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L338 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L339 EN**: Assigns or initializes `const GTrunc *Trunc`.
  **L339 CN**: 对 `const GTrunc *Trunc` 进行赋值或初始化。
- **L340 EN**: Assigns or initializes `const GBinOp *BinOp`.
  **L340 CN**: 对 `const GBinOp *BinOp` 进行赋值或初始化。

### Lines 341-360

````cpp

  if (!MRI.hasOneNonDBGUse(BinOp->getReg(0)))
    return false;

  Register Dst = Trunc->getReg(0);
  LLT DstTy = MRI.getType(Dst);

  // Is narrow binop legal?
  if (!isLegalOrBeforeLegalizer({BinOp->getOpcode(), {DstTy}}))
    return false;

  MatchInfo = [=](MachineIRBuilder &B) {
    auto LHS = B.buildTrunc(DstTy, BinOp->getLHSReg());
    auto RHS = B.buildTrunc(DstTy, BinOp->getRHSReg());
    B.buildInstr(BinOp->getOpcode(), {Dst}, {LHS, RHS});
  };

  return true;
}

````
- **L341 EN**: Separates nearby statements for readability.
  **L341 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L342 EN**: Begins a conditional branch.
  **L342 CN**: 开始一个条件分支。
- **L343 EN**: Returns `false` to the caller.
  **L343 CN**: 向调用者返回 `false`。
- **L344 EN**: Separates nearby statements for readability.
  **L344 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L345 EN**: Assigns or initializes `Register Dst`.
  **L345 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L346 EN**: Assigns or initializes `LLT DstTy`.
  **L346 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L347 EN**: Separates nearby statements for readability.
  **L347 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L348 EN**: Comment documents: `Is narrow binop legal?`.
  **L348 CN**: 注释说明：`Is narrow binop legal?`。
- **L349 EN**: Begins a conditional branch.
  **L349 CN**: 开始一个条件分支。
- **L350 EN**: Returns `false` to the caller.
  **L350 CN**: 向调用者返回 `false`。
- **L351 EN**: Separates nearby statements for readability.
  **L351 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L352 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L352 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L353 EN**: Assigns or initializes `auto LHS`.
  **L353 CN**: 对 `auto LHS` 进行赋值或初始化。
- **L354 EN**: Assigns or initializes `auto RHS`.
  **L354 CN**: 对 `auto RHS` 进行赋值或初始化。
- **L355 EN**: Executes statement `B.buildInstr(BinOp->getOpcode(), {Dst}, {LHS, RHS});`.
  **L355 CN**: 执行语句 `B.buildInstr(BinOp->getOpcode(), {Dst}, {LHS, RHS});`。
- **L356 EN**: Closes the current scope.
  **L356 CN**: 关闭当前作用域。
- **L357 EN**: Separates nearby statements for readability.
  **L357 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L358 EN**: Returns `true` to the caller.
  **L358 CN**: 向调用者返回 `true`。
- **L359 EN**: Closes the current scope.
  **L359 CN**: 关闭当前作用域。
- **L360 EN**: Separates nearby statements for readability.
  **L360 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 361-380

````cpp
bool CombinerHelper::matchCastOfInteger(const MachineInstr &CastMI,
                                        APInt &MatchInfo) const {
  const GExtOrTruncOp *Cast = cast<GExtOrTruncOp>(&CastMI);

  APInt Input = getIConstantFromReg(Cast->getSrcReg(), MRI);

  LLT DstTy = MRI.getType(Cast->getReg(0));

  if (!isConstantLegalOrBeforeLegalizer(DstTy))
    return false;

  switch (Cast->getOpcode()) {
  case TargetOpcode::G_TRUNC: {
    MatchInfo = Input.trunc(DstTy.getScalarSizeInBits());
    return true;
  }
  default:
    return false;
  }
}
````
- **L361 EN**: Provides part of the signature for `matchCastOfInteger`.
  **L361 CN**: 给出 `matchCastOfInteger` 的一部分签名。
- **L362 EN**: Starts block `APInt &MatchInfo) const`.
  **L362 CN**: 开始代码块 `APInt &MatchInfo) const`。
- **L363 EN**: Assigns or initializes `const GExtOrTruncOp *Cast`.
  **L363 CN**: 对 `const GExtOrTruncOp *Cast` 进行赋值或初始化。
- **L364 EN**: Separates nearby statements for readability.
  **L364 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L365 EN**: Assigns or initializes `APInt Input`.
  **L365 CN**: 对 `APInt Input` 进行赋值或初始化。
- **L366 EN**: Separates nearby statements for readability.
  **L366 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L367 EN**: Assigns or initializes `LLT DstTy`.
  **L367 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L368 EN**: Separates nearby statements for readability.
  **L368 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L369 EN**: Begins a conditional branch.
  **L369 CN**: 开始一个条件分支。
- **L370 EN**: Returns `false` to the caller.
  **L370 CN**: 向调用者返回 `false`。
- **L371 EN**: Separates nearby statements for readability.
  **L371 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L372 EN**: Starts a multi-way branch.
  **L372 CN**: 开始一个多路分支。
- **L373 EN**: Handles one switch case.
  **L373 CN**: 处理一个 switch 分支。
- **L374 EN**: Assigns or initializes `MatchInfo`.
  **L374 CN**: 对 `MatchInfo` 进行赋值或初始化。
- **L375 EN**: Returns `true` to the caller.
  **L375 CN**: 向调用者返回 `true`。
- **L376 EN**: Closes the current scope.
  **L376 CN**: 关闭当前作用域。
- **L377 EN**: Handles the default switch case.
  **L377 CN**: 处理 switch 的默认分支。
- **L378 EN**: Returns `false` to the caller.
  **L378 CN**: 向调用者返回 `false`。
- **L379 EN**: Closes the current scope.
  **L379 CN**: 关闭当前作用域。
- **L380 EN**: Closes the current scope.
  **L380 CN**: 关闭当前作用域。

### Lines 381-400

````cpp

bool CombinerHelper::matchRedundantSextInReg(MachineInstr &Root,
                                             MachineInstr &Other,
                                             BuildFnTy &MatchInfo) const {
  assert(Root.getOpcode() == TargetOpcode::G_SEXT_INREG &&
         Other.getOpcode() == TargetOpcode::G_SEXT_INREG);

  unsigned RootWidth = Root.getOperand(2).getImm();
  unsigned OtherWidth = Other.getOperand(2).getImm();

  Register Dst = Root.getOperand(0).getReg();
  Register OtherDst = Other.getOperand(0).getReg();
  Register Src = Other.getOperand(1).getReg();

  if (RootWidth >= OtherWidth) {
    // The root sext_inreg is entirely redundant because the other one
    // is narrower.
    if (!canReplaceReg(Dst, OtherDst, MRI))
      return false;

````
- **L381 EN**: Separates nearby statements for readability.
  **L381 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L382 EN**: Provides part of the signature for `matchRedundantSextInReg`.
  **L382 CN**: 给出 `matchRedundantSextInReg` 的一部分签名。
- **L383 EN**: Continues logic with `MachineInstr &Other,`.
  **L383 CN**: 继续处理逻辑：`MachineInstr &Other,`。
- **L384 EN**: Starts block `BuildFnTy &MatchInfo) const`.
  **L384 CN**: 开始代码块 `BuildFnTy &MatchInfo) const`。
- **L385 EN**: Checks an invariant in debug builds.
  **L385 CN**: 在调试构建中检查一个不变量。
- **L386 EN**: Assigns or initializes `Other.getOpcode()`.
  **L386 CN**: 对 `Other.getOpcode()` 进行赋值或初始化。
- **L387 EN**: Separates nearby statements for readability.
  **L387 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L388 EN**: Assigns or initializes `unsigned RootWidth`.
  **L388 CN**: 对 `unsigned RootWidth` 进行赋值或初始化。
- **L389 EN**: Assigns or initializes `unsigned OtherWidth`.
  **L389 CN**: 对 `unsigned OtherWidth` 进行赋值或初始化。
- **L390 EN**: Separates nearby statements for readability.
  **L390 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L391 EN**: Assigns or initializes `Register Dst`.
  **L391 CN**: 对 `Register Dst` 进行赋值或初始化。
- **L392 EN**: Assigns or initializes `Register OtherDst`.
  **L392 CN**: 对 `Register OtherDst` 进行赋值或初始化。
- **L393 EN**: Assigns or initializes `Register Src`.
  **L393 CN**: 对 `Register Src` 进行赋值或初始化。
- **L394 EN**: Separates nearby statements for readability.
  **L394 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L395 EN**: Begins a conditional branch.
  **L395 CN**: 开始一个条件分支。
- **L396 EN**: Comment documents: `The root sext_inreg is entirely redundant because the other one`.
  **L396 CN**: 注释说明：`The root sext_inreg is entirely redundant because the other one`。
- **L397 EN**: Comment documents: `is narrower.`.
  **L397 CN**: 注释说明：`is narrower.`。
- **L398 EN**: Begins a conditional branch.
  **L398 CN**: 开始一个条件分支。
- **L399 EN**: Returns `false` to the caller.
  **L399 CN**: 向调用者返回 `false`。
- **L400 EN**: Separates nearby statements for readability.
  **L400 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 401-415

````cpp
    MatchInfo = [=](MachineIRBuilder &B) {
      Observer.changingAllUsesOfReg(MRI, Dst);
      MRI.replaceRegWith(Dst, OtherDst);
      Observer.finishedChangingAllUsesOfReg();
    };
  } else {
    // RootWidth < OtherWidth, rewrite this G_SEXT_INREG with the source of the
    // other G_SEXT_INREG.
    MatchInfo = [=](MachineIRBuilder &B) {
      B.buildSExtInReg(Dst, Src, RootWidth);
    };
  }

  return true;
}
````
- **L401 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L401 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L402 EN**: Executes statement `Observer.changingAllUsesOfReg(MRI, Dst);`.
  **L402 CN**: 执行语句 `Observer.changingAllUsesOfReg(MRI, Dst);`。
- **L403 EN**: Executes statement `MRI.replaceRegWith(Dst, OtherDst);`.
  **L403 CN**: 执行语句 `MRI.replaceRegWith(Dst, OtherDst);`。
- **L404 EN**: Executes statement `Observer.finishedChangingAllUsesOfReg();`.
  **L404 CN**: 执行语句 `Observer.finishedChangingAllUsesOfReg();`。
- **L405 EN**: Closes the current scope.
  **L405 CN**: 关闭当前作用域。
- **L406 EN**: Starts block `} else`.
  **L406 CN**: 开始代码块 `} else`。
- **L407 EN**: Comment documents: `RootWidth < OtherWidth, rewrite this G_SEXT_INREG with the source of the`.
  **L407 CN**: 注释说明：`RootWidth < OtherWidth, rewrite this G_SEXT_INREG with the source of the`。
- **L408 EN**: Comment documents: `other G_SEXT_INREG.`.
  **L408 CN**: 注释说明：`other G_SEXT_INREG.`。
- **L409 EN**: Starts block `MatchInfo = [=](MachineIRBuilder &B)`.
  **L409 CN**: 开始代码块 `MatchInfo = [=](MachineIRBuilder &B)`。
- **L410 EN**: Executes statement `B.buildSExtInReg(Dst, Src, RootWidth);`.
  **L410 CN**: 执行语句 `B.buildSExtInReg(Dst, Src, RootWidth);`。
- **L411 EN**: Closes the current scope.
  **L411 CN**: 关闭当前作用域。
- **L412 EN**: Closes the current scope.
  **L412 CN**: 关闭当前作用域。
- **L413 EN**: Separates nearby statements for readability.
  **L413 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L414 EN**: Returns `true` to the caller.
  **L414 CN**: 向调用者返回 `true`。
- **L415 EN**: Closes the current scope.
  **L415 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Register management** / **寄存器管理**
- **Machine instruction manipulation** / **机器指令操作**
- **GlobalISel pipeline** / **GlobalISel 流水线**
- **Debug information handling** / **调试信息处理**
- **Target-specific hooks** / **目标相关钩子**
- **Garbage-collection support** / **垃圾回收支持**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/GlobalISel/CombinerHelper.h`, `llvm/CodeGen/GlobalISel/LegalizerHelper.h`, `llvm/CodeGen/GlobalISel/LegalizerInfo.h`, `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`, `llvm/CodeGen/GlobalISel/Utils.h`, `llvm/CodeGen/LowLevelTypeUtils.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/TargetOpcodes.h`, `llvm/Support/Casting.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
