# XRayInstrumentation.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/XRayInstrumentation.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `Adds XRay instrumentation to functions.` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“Adds XRay instrumentation to functions.”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- XRayInstrumentation.cpp - Adds XRay instrumentation to functions. --===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements a MachineFunctionPass that inserts the appropriate
// XRay instrumentation instructions. We look for XRay-specific attributes
// on the function to determine whether we should insert the replacement
// operations.
//
//===---------------------------------------------------------------------===//

#include "llvm/CodeGen/XRayInstrumentation.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineDominators.h"
````
- **L1 EN**: Comment documents: `===- XRayInstrumentation.cpp - Adds XRay instrumentation to functions. -…`.
  **L1 CN**: 注释说明：`===- XRayInstrumentation.cpp - Adds XRay instrumentation to functions. -…`。
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
- **L9 EN**: Comment documents: `This file implements a MachineFunctionPass that inserts the appropriate`.
  **L9 CN**: 注释说明：`This file implements a MachineFunctionPass that inserts the appropriate`。
- **L10 EN**: Comment documents: `XRay instrumentation instructions. We look for XRay-specific attributes`.
  **L10 CN**: 注释说明：`XRay instrumentation instructions. We look for XRay-specific attributes`。
- **L11 EN**: Comment documents: `on the function to determine whether we should insert the replacement`.
  **L11 CN**: 注释说明：`on the function to determine whether we should insert the replacement`。
- **L12 EN**: Comment documents: `operations.`.
  **L12 CN**: 注释说明：`operations.`。
- **L13 EN**: Continues the surrounding comment block.
  **L13 CN**: 延续周围的注释块。
- **L14 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L14 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L15 EN**: Separates nearby statements for readability.
  **L15 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L16 EN**: Includes LLVM header `llvm/CodeGen/XRayInstrumentation.h` for XRayInstrumentation support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/CodeGen/XRayInstrumentation.h`，用于 XRayInstrumentation 相关支持。
- **L17 EN**: Includes LLVM header `llvm/ADT/STLExtras.h` for STLExtras support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/ADT/STLExtras.h`，用于 STLExtras 相关支持。
- **L18 EN**: Includes LLVM header `llvm/ADT/SmallVector.h` for SmallVector support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/ADT/SmallVector.h`，用于 SmallVector 相关支持。
- **L19 EN**: Includes LLVM header `llvm/CodeGen/MachineBasicBlock.h` for MachineBasicBlock support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineBasicBlock.h`，用于 MachineBasicBlock 相关支持。
- **L20 EN**: Includes LLVM header `llvm/CodeGen/MachineDominators.h` for MachineDominators support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineDominators.h`，用于 MachineDominators 相关支持。

### Lines 21-40

````cpp
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionAnalysis.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineLoopInfo.h"
#include "llvm/CodeGen/MachinePassManager.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/Attributes.h"
#include "llvm/IR/DiagnosticInfo.h"
#include "llvm/IR/Function.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/TargetParser/Triple.h"

using namespace llvm;

namespace {

````
- **L21 EN**: Includes LLVM header `llvm/CodeGen/MachineFunction.h` for MachineFunction support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFunction.h`，用于 MachineFunction 相关支持。
- **L22 EN**: Includes LLVM header `llvm/CodeGen/MachineFunctionAnalysis.h` for MachineFunctionAnalysis support.
  **L22 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFunctionAnalysis.h`，用于 MachineFunctionAnalysis 相关支持。
- **L23 EN**: Includes LLVM header `llvm/CodeGen/MachineFunctionPass.h` for MachineFunctionPass support.
  **L23 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFunctionPass.h`，用于 MachineFunctionPass 相关支持。
- **L24 EN**: Includes LLVM header `llvm/CodeGen/MachineInstrBuilder.h` for MachineInstrBuilder support.
  **L24 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineInstrBuilder.h`，用于 MachineInstrBuilder 相关支持。
- **L25 EN**: Includes LLVM header `llvm/CodeGen/MachineLoopInfo.h` for MachineLoopInfo support.
  **L25 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineLoopInfo.h`，用于 MachineLoopInfo 相关支持。
- **L26 EN**: Includes LLVM header `llvm/CodeGen/MachinePassManager.h` for MachinePassManager support.
  **L26 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachinePassManager.h`，用于 MachinePassManager 相关支持。
- **L27 EN**: Includes LLVM header `llvm/CodeGen/TargetInstrInfo.h` for TargetInstrInfo support.
  **L27 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetInstrInfo.h`，用于 TargetInstrInfo 相关支持。
- **L28 EN**: Includes LLVM header `llvm/CodeGen/TargetSubtargetInfo.h` for TargetSubtargetInfo support.
  **L28 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetSubtargetInfo.h`，用于 TargetSubtargetInfo 相关支持。
- **L29 EN**: Includes LLVM header `llvm/IR/Attributes.h` for Attributes support.
  **L29 CN**: 引入 LLVM 头文件 `llvm/IR/Attributes.h`，用于 Attributes 相关支持。
- **L30 EN**: Includes LLVM header `llvm/IR/DiagnosticInfo.h` for DiagnosticInfo support.
  **L30 CN**: 引入 LLVM 头文件 `llvm/IR/DiagnosticInfo.h`，用于 DiagnosticInfo 相关支持。
- **L31 EN**: Includes LLVM header `llvm/IR/Function.h` for Function support.
  **L31 CN**: 引入 LLVM 头文件 `llvm/IR/Function.h`，用于 Function 相关支持。
- **L32 EN**: Includes LLVM header `llvm/InitializePasses.h` for InitializePasses support.
  **L32 CN**: 引入 LLVM 头文件 `llvm/InitializePasses.h`，用于 InitializePasses 相关支持。
- **L33 EN**: Includes LLVM header `llvm/Pass.h` for Pass support.
  **L33 CN**: 引入 LLVM 头文件 `llvm/Pass.h`，用于 Pass 相关支持。
- **L34 EN**: Includes LLVM header `llvm/Target/TargetMachine.h` for TargetMachine support.
  **L34 CN**: 引入 LLVM 头文件 `llvm/Target/TargetMachine.h`，用于 TargetMachine 相关支持。
- **L35 EN**: Includes LLVM header `llvm/TargetParser/Triple.h` for Triple support.
  **L35 CN**: 引入 LLVM 头文件 `llvm/TargetParser/Triple.h`，用于 Triple 相关支持。
- **L36 EN**: Separates nearby statements for readability.
  **L36 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L37 EN**: Imports namespace `llvm` into this translation unit.
  **L37 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L38 EN**: Separates nearby statements for readability.
  **L38 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L39 EN**: Opens namespace ``.
  **L39 CN**: 打开命名空间 ``。
- **L40 EN**: Separates nearby statements for readability.
  **L40 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 41-60

````cpp
struct InstrumentationOptions {
  // Whether to emit PATCHABLE_TAIL_CALL.
  bool HandleTailcall;

  // Whether to emit PATCHABLE_RET/PATCHABLE_FUNCTION_EXIT for all forms of
  // return, e.g. conditional return.
  bool HandleAllReturns;
};

struct XRayInstrumentationLegacy : public MachineFunctionPass {
  static char ID;

  XRayInstrumentationLegacy() : MachineFunctionPass(ID) {}

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addPreserved<MachineLoopInfoWrapperPass>();
    AU.addPreserved<MachineDominatorTreeWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
````
- **L41 EN**: Starts the declaration of struct `InstrumentationOptions`.
  **L41 CN**: 开始声明 struct `InstrumentationOptions`。
- **L42 EN**: Comment documents: `Whether to emit PATCHABLE_TAIL_CALL.`.
  **L42 CN**: 注释说明：`Whether to emit PATCHABLE_TAIL_CALL.`。
- **L43 EN**: Executes statement `bool HandleTailcall;`.
  **L43 CN**: 执行语句 `bool HandleTailcall;`。
- **L44 EN**: Separates nearby statements for readability.
  **L44 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L45 EN**: Comment documents: `Whether to emit PATCHABLE_RET/PATCHABLE_FUNCTION_EXIT for all forms of`.
  **L45 CN**: 注释说明：`Whether to emit PATCHABLE_RET/PATCHABLE_FUNCTION_EXIT for all forms of`。
- **L46 EN**: Comment documents: `return, e.g. conditional return.`.
  **L46 CN**: 注释说明：`return, e.g. conditional return.`。
- **L47 EN**: Executes statement `bool HandleAllReturns;`.
  **L47 CN**: 执行语句 `bool HandleAllReturns;`。
- **L48 EN**: Closes the current scope.
  **L48 CN**: 关闭当前作用域。
- **L49 EN**: Separates nearby statements for readability.
  **L49 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L50 EN**: Starts the declaration of struct `XRayInstrumentationLegacy`.
  **L50 CN**: 开始声明 struct `XRayInstrumentationLegacy`。
- **L51 EN**: Executes statement `static char ID;`.
  **L51 CN**: 执行语句 `static char ID;`。
- **L52 EN**: Separates nearby statements for readability.
  **L52 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L53 EN**: Continues logic with `XRayInstrumentationLegacy() : MachineFunctionPass(ID) {}`.
  **L53 CN**: 继续处理逻辑：`XRayInstrumentationLegacy() : MachineFunctionPass(ID) {}`。
- **L54 EN**: Separates nearby statements for readability.
  **L54 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L55 EN**: Begins the definition of `getAnalysisUsage`.
  **L55 CN**: 开始定义 `getAnalysisUsage`。
- **L56 EN**: Executes statement `AU.setPreservesCFG();`.
  **L56 CN**: 执行语句 `AU.setPreservesCFG();`。
- **L57 EN**: Executes statement `AU.addPreserved<MachineLoopInfoWrapperPass>();`.
  **L57 CN**: 执行语句 `AU.addPreserved<MachineLoopInfoWrapperPass>();`。
- **L58 EN**: Executes statement `AU.addPreserved<MachineDominatorTreeWrapperPass>();`.
  **L58 CN**: 执行语句 `AU.addPreserved<MachineDominatorTreeWrapperPass>();`。
- **L59 EN**: Declares function or method `getAnalysisUsage`.
  **L59 CN**: 声明函数或方法 `getAnalysisUsage`。
- **L60 EN**: Closes the current scope.
  **L60 CN**: 关闭当前作用域。

### Lines 61-80

````cpp

  bool runOnMachineFunction(MachineFunction &MF) override;
};

struct XRayInstrumentation {
  XRayInstrumentation(MachineDominatorTree *MDT, MachineLoopInfo *MLI)
      : MDT(MDT), MLI(MLI) {}

  bool run(MachineFunction &MF);

  // Methods for use in the NPM and legacy passes, can be removed once migration
  // is complete.
  static bool alwaysInstrument(Function &F) {
    auto InstrAttr = F.getFnAttribute("function-instrument");
    return InstrAttr.isStringAttribute() &&
           InstrAttr.getValueAsString() == "xray-always";
  }

  static bool needMDTAndMLIAnalyses(Function &F) {
    auto IgnoreLoopsAttr = F.getFnAttribute("xray-ignore-loops");
````
- **L61 EN**: Separates nearby statements for readability.
  **L61 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L62 EN**: Declares function or method `runOnMachineFunction`.
  **L62 CN**: 声明函数或方法 `runOnMachineFunction`。
- **L63 EN**: Closes the current scope.
  **L63 CN**: 关闭当前作用域。
- **L64 EN**: Separates nearby statements for readability.
  **L64 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L65 EN**: Starts the declaration of struct `XRayInstrumentation`.
  **L65 CN**: 开始声明 struct `XRayInstrumentation`。
- **L66 EN**: Continues logic with `XRayInstrumentation(MachineDominatorTree *MDT, MachineLoopInfo *MLI)`.
  **L66 CN**: 继续处理逻辑：`XRayInstrumentation(MachineDominatorTree *MDT, MachineLoopInfo *MLI)`。
- **L67 EN**: Provides part of the signature for `MDT`.
  **L67 CN**: 给出 `MDT` 的一部分签名。
- **L68 EN**: Separates nearby statements for readability.
  **L68 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L69 EN**: Declares function or method `run`.
  **L69 CN**: 声明函数或方法 `run`。
- **L70 EN**: Separates nearby statements for readability.
  **L70 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L71 EN**: Comment documents: `Methods for use in the NPM and legacy passes, can be removed once migrat…`.
  **L71 CN**: 注释说明：`Methods for use in the NPM and legacy passes, can be removed once migrat…`。
- **L72 EN**: Comment documents: `is complete.`.
  **L72 CN**: 注释说明：`is complete.`。
- **L73 EN**: Begins the definition of `alwaysInstrument`.
  **L73 CN**: 开始定义 `alwaysInstrument`。
- **L74 EN**: Assigns or initializes `auto InstrAttr`.
  **L74 CN**: 对 `auto InstrAttr` 进行赋值或初始化。
- **L75 EN**: Returns `InstrAttr.isStringAttribute() &&` to the caller.
  **L75 CN**: 向调用者返回 `InstrAttr.isStringAttribute() &&`。
- **L76 EN**: Assigns or initializes `InstrAttr.getValueAsString()`.
  **L76 CN**: 对 `InstrAttr.getValueAsString()` 进行赋值或初始化。
- **L77 EN**: Closes the current scope.
  **L77 CN**: 关闭当前作用域。
- **L78 EN**: Separates nearby statements for readability.
  **L78 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L79 EN**: Begins the definition of `needMDTAndMLIAnalyses`.
  **L79 CN**: 开始定义 `needMDTAndMLIAnalyses`。
- **L80 EN**: Assigns or initializes `auto IgnoreLoopsAttr`.
  **L80 CN**: 对 `auto IgnoreLoopsAttr` 进行赋值或初始化。

### Lines 81-100

````cpp
    auto AlwaysInstrument = XRayInstrumentation::alwaysInstrument(F);
    return !AlwaysInstrument && !IgnoreLoopsAttr.isValid();
  }

private:
  // Replace the original RET instruction with the exit sled code ("patchable
  //   ret" pseudo-instruction), so that at runtime XRay can replace the sled
  //   with a code jumping to XRay trampoline, which calls the tracing handler
  //   and, in the end, issues the RET instruction.
  // This is the approach to go on CPUs which have a single RET instruction,
  //   like x86/x86_64.
  void replaceRetWithPatchableRet(MachineFunction &MF,
                                  const TargetInstrInfo *TII,
                                  InstrumentationOptions);

  // Prepend the original return instruction with the exit sled code ("patchable
  //   function exit" pseudo-instruction), preserving the original return
  //   instruction just after the exit sled code.
  // This is the approach to go on CPUs which have multiple options for the
  //   return instruction, like ARM. For such CPUs we can't just jump into the
````
- **L81 EN**: Declares function or method `alwaysInstrument`.
  **L81 CN**: 声明函数或方法 `alwaysInstrument`。
- **L82 EN**: Returns `!AlwaysInstrument && !IgnoreLoopsAttr.isValid()` to the caller.
  **L82 CN**: 向调用者返回 `!AlwaysInstrument && !IgnoreLoopsAttr.isValid()`。
- **L83 EN**: Closes the current scope.
  **L83 CN**: 关闭当前作用域。
- **L84 EN**: Separates nearby statements for readability.
  **L84 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L85 EN**: Continues logic with `private:`.
  **L85 CN**: 继续处理逻辑：`private:`。
- **L86 EN**: Comment documents: `Replace the original RET instruction with the exit sled code ("patchable`.
  **L86 CN**: 注释说明：`Replace the original RET instruction with the exit sled code ("patchable`。
- **L87 EN**: Comment documents: `ret" pseudo-instruction), so that at runtime XRay can replace the sled`.
  **L87 CN**: 注释说明：`ret" pseudo-instruction), so that at runtime XRay can replace the sled`。
- **L88 EN**: Comment documents: `with a code jumping to XRay trampoline, which calls the tracing handler`.
  **L88 CN**: 注释说明：`with a code jumping to XRay trampoline, which calls the tracing handler`。
- **L89 EN**: Comment documents: `and, in the end, issues the RET instruction.`.
  **L89 CN**: 注释说明：`and, in the end, issues the RET instruction.`。
- **L90 EN**: Comment documents: `This is the approach to go on CPUs which have a single RET instruction,`.
  **L90 CN**: 注释说明：`This is the approach to go on CPUs which have a single RET instruction,`。
- **L91 EN**: Comment documents: `like x86/x86_64.`.
  **L91 CN**: 注释说明：`like x86/x86_64.`。
- **L92 EN**: Provides part of the signature for `replaceRetWithPatchableRet`.
  **L92 CN**: 给出 `replaceRetWithPatchableRet` 的一部分签名。
- **L93 EN**: Continues logic with `const TargetInstrInfo *TII,`.
  **L93 CN**: 继续处理逻辑：`const TargetInstrInfo *TII,`。
- **L94 EN**: Executes statement `InstrumentationOptions);`.
  **L94 CN**: 执行语句 `InstrumentationOptions);`。
- **L95 EN**: Separates nearby statements for readability.
  **L95 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L96 EN**: Comment documents: `Prepend the original return instruction with the exit sled code ("patcha…`.
  **L96 CN**: 注释说明：`Prepend the original return instruction with the exit sled code ("patcha…`。
- **L97 EN**: Comment documents: `function exit" pseudo-instruction), preserving the original return`.
  **L97 CN**: 注释说明：`function exit" pseudo-instruction), preserving the original return`。
- **L98 EN**: Comment documents: `instruction just after the exit sled code.`.
  **L98 CN**: 注释说明：`instruction just after the exit sled code.`。
- **L99 EN**: Comment documents: `This is the approach to go on CPUs which have multiple options for the`.
  **L99 CN**: 注释说明：`This is the approach to go on CPUs which have multiple options for the`。
- **L100 EN**: Comment documents: `return instruction, like ARM. For such CPUs we can't just jump into the`.
  **L100 CN**: 注释说明：`return instruction, like ARM. For such CPUs we can't just jump into the`。

### Lines 101-120

````cpp
  //   XRay trampoline and issue a single return instruction there. We rather
  //   have to call the trampoline and return from it to the original return
  //   instruction of the function being instrumented.
  void prependRetWithPatchableExit(MachineFunction &MF,
                                   const TargetInstrInfo *TII,
                                   InstrumentationOptions);

  MachineDominatorTree *MDT;
  MachineLoopInfo *MLI;
};

} // end anonymous namespace

void XRayInstrumentation::replaceRetWithPatchableRet(
    MachineFunction &MF, const TargetInstrInfo *TII,
    InstrumentationOptions op) {
  // We look for *all* terminators and returns, then replace those with
  // PATCHABLE_RET instructions.
  SmallVector<MachineInstr *, 4> Terminators;
  for (auto &MBB : MF) {
````
- **L101 EN**: Comment documents: `XRay trampoline and issue a single return instruction there. We rather`.
  **L101 CN**: 注释说明：`XRay trampoline and issue a single return instruction there. We rather`。
- **L102 EN**: Comment documents: `have to call the trampoline and return from it to the original return`.
  **L102 CN**: 注释说明：`have to call the trampoline and return from it to the original return`。
- **L103 EN**: Comment documents: `instruction of the function being instrumented.`.
  **L103 CN**: 注释说明：`instruction of the function being instrumented.`。
- **L104 EN**: Provides part of the signature for `prependRetWithPatchableExit`.
  **L104 CN**: 给出 `prependRetWithPatchableExit` 的一部分签名。
- **L105 EN**: Continues logic with `const TargetInstrInfo *TII,`.
  **L105 CN**: 继续处理逻辑：`const TargetInstrInfo *TII,`。
- **L106 EN**: Executes statement `InstrumentationOptions);`.
  **L106 CN**: 执行语句 `InstrumentationOptions);`。
- **L107 EN**: Separates nearby statements for readability.
  **L107 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L108 EN**: Executes statement `MachineDominatorTree *MDT;`.
  **L108 CN**: 执行语句 `MachineDominatorTree *MDT;`。
- **L109 EN**: Executes statement `MachineLoopInfo *MLI;`.
  **L109 CN**: 执行语句 `MachineLoopInfo *MLI;`。
- **L110 EN**: Closes the current scope.
  **L110 CN**: 关闭当前作用域。
- **L111 EN**: Separates nearby statements for readability.
  **L111 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L112 EN**: Continues logic with `} // end anonymous namespace`.
  **L112 CN**: 继续处理逻辑：`} // end anonymous namespace`。
- **L113 EN**: Separates nearby statements for readability.
  **L113 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L114 EN**: Provides part of the signature for `replaceRetWithPatchableRet`.
  **L114 CN**: 给出 `replaceRetWithPatchableRet` 的一部分签名。
- **L115 EN**: Continues logic with `MachineFunction &MF, const TargetInstrInfo *TII,`.
  **L115 CN**: 继续处理逻辑：`MachineFunction &MF, const TargetInstrInfo *TII,`。
- **L116 EN**: Starts block `InstrumentationOptions op)`.
  **L116 CN**: 开始代码块 `InstrumentationOptions op)`。
- **L117 EN**: Comment documents: `We look for *all* terminators and returns, then replace those with`.
  **L117 CN**: 注释说明：`We look for *all* terminators and returns, then replace those with`。
- **L118 EN**: Comment documents: `PATCHABLE_RET instructions.`.
  **L118 CN**: 注释说明：`PATCHABLE_RET instructions.`。
- **L119 EN**: Executes statement `SmallVector<MachineInstr *, 4> Terminators;`.
  **L119 CN**: 执行语句 `SmallVector<MachineInstr *, 4> Terminators;`。
- **L120 EN**: Starts a loop over a sequence or range.
  **L120 CN**: 开始遍历序列或范围的循环。

### Lines 121-140

````cpp
    for (auto &T : MBB.terminators()) {
      unsigned Opc = 0;
      if (T.isReturn() &&
          (op.HandleAllReturns || T.getOpcode() == TII->getReturnOpcode())) {
        // Replace return instructions with:
        //   PATCHABLE_RET <Opcode>, <Operand>...
        Opc = TargetOpcode::PATCHABLE_RET;
      }
      if (TII->isTailCall(T) && op.HandleTailcall) {
        // Treat the tail call as a return instruction, which has a
        // different-looking sled than the normal return case.
        Opc = TargetOpcode::PATCHABLE_TAIL_CALL;
      }
      if (Opc != 0) {
        auto MIB = BuildMI(MBB, T, T.getDebugLoc(), TII->get(Opc))
                       .addImm(T.getOpcode());
        for (auto &MO : T.operands())
          MIB.add(MO);
        Terminators.push_back(&T);
        if (T.shouldUpdateAdditionalCallInfo())
````
- **L121 EN**: Starts a loop over a sequence or range.
  **L121 CN**: 开始遍历序列或范围的循环。
- **L122 EN**: Assigns or initializes `unsigned Opc`.
  **L122 CN**: 对 `unsigned Opc` 进行赋值或初始化。
- **L123 EN**: Begins a conditional branch.
  **L123 CN**: 开始一个条件分支。
- **L124 EN**: Starts block `(op.HandleAllReturns || T.getOpcode() == TII->getReturnOpcode()))`.
  **L124 CN**: 开始代码块 `(op.HandleAllReturns || T.getOpcode() == TII->getReturnOpcode()))`。
- **L125 EN**: Comment documents: `Replace return instructions with:`.
  **L125 CN**: 注释说明：`Replace return instructions with:`。
- **L126 EN**: Comment documents: `PATCHABLE_RET <Opcode>, <Operand>...`.
  **L126 CN**: 注释说明：`PATCHABLE_RET <Opcode>, <Operand>...`。
- **L127 EN**: Assigns or initializes `Opc`.
  **L127 CN**: 对 `Opc` 进行赋值或初始化。
- **L128 EN**: Closes the current scope.
  **L128 CN**: 关闭当前作用域。
- **L129 EN**: Begins a conditional branch.
  **L129 CN**: 开始一个条件分支。
- **L130 EN**: Comment documents: `Treat the tail call as a return instruction, which has a`.
  **L130 CN**: 注释说明：`Treat the tail call as a return instruction, which has a`。
- **L131 EN**: Comment documents: `different-looking sled than the normal return case.`.
  **L131 CN**: 注释说明：`different-looking sled than the normal return case.`。
- **L132 EN**: Assigns or initializes `Opc`.
  **L132 CN**: 对 `Opc` 进行赋值或初始化。
- **L133 EN**: Closes the current scope.
  **L133 CN**: 关闭当前作用域。
- **L134 EN**: Begins a conditional branch.
  **L134 CN**: 开始一个条件分支。
- **L135 EN**: Continues logic with `auto MIB = BuildMI(MBB, T, T.getDebugLoc(), TII->get(Opc))`.
  **L135 CN**: 继续处理逻辑：`auto MIB = BuildMI(MBB, T, T.getDebugLoc(), TII->get(Opc))`。
- **L136 EN**: Executes statement `.addImm(T.getOpcode());`.
  **L136 CN**: 执行语句 `.addImm(T.getOpcode());`。
- **L137 EN**: Starts a loop over a sequence or range.
  **L137 CN**: 开始遍历序列或范围的循环。
- **L138 EN**: Executes statement `MIB.add(MO);`.
  **L138 CN**: 执行语句 `MIB.add(MO);`。
- **L139 EN**: Executes statement `Terminators.push_back(&T);`.
  **L139 CN**: 执行语句 `Terminators.push_back(&T);`。
- **L140 EN**: Begins a conditional branch.
  **L140 CN**: 开始一个条件分支。

### Lines 141-160

````cpp
          MF.eraseAdditionalCallInfo(&T);
      }
    }
  }

  for (auto &I : Terminators)
    I->eraseFromParent();
}

void XRayInstrumentation::prependRetWithPatchableExit(
    MachineFunction &MF, const TargetInstrInfo *TII,
    InstrumentationOptions op) {
  for (auto &MBB : MF)
    for (auto &T : MBB.terminators()) {
      unsigned Opc = 0;
      if (T.isReturn() &&
          (op.HandleAllReturns || T.getOpcode() == TII->getReturnOpcode())) {
        Opc = TargetOpcode::PATCHABLE_FUNCTION_EXIT;
      }
      if (TII->isTailCall(T) && op.HandleTailcall) {
````
- **L141 EN**: Executes statement `MF.eraseAdditionalCallInfo(&T);`.
  **L141 CN**: 执行语句 `MF.eraseAdditionalCallInfo(&T);`。
- **L142 EN**: Closes the current scope.
  **L142 CN**: 关闭当前作用域。
- **L143 EN**: Closes the current scope.
  **L143 CN**: 关闭当前作用域。
- **L144 EN**: Closes the current scope.
  **L144 CN**: 关闭当前作用域。
- **L145 EN**: Separates nearby statements for readability.
  **L145 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L146 EN**: Starts a loop over a sequence or range.
  **L146 CN**: 开始遍历序列或范围的循环。
- **L147 EN**: Executes statement `I->eraseFromParent();`.
  **L147 CN**: 执行语句 `I->eraseFromParent();`。
- **L148 EN**: Closes the current scope.
  **L148 CN**: 关闭当前作用域。
- **L149 EN**: Separates nearby statements for readability.
  **L149 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L150 EN**: Provides part of the signature for `prependRetWithPatchableExit`.
  **L150 CN**: 给出 `prependRetWithPatchableExit` 的一部分签名。
- **L151 EN**: Continues logic with `MachineFunction &MF, const TargetInstrInfo *TII,`.
  **L151 CN**: 继续处理逻辑：`MachineFunction &MF, const TargetInstrInfo *TII,`。
- **L152 EN**: Starts block `InstrumentationOptions op)`.
  **L152 CN**: 开始代码块 `InstrumentationOptions op)`。
- **L153 EN**: Starts a loop over a sequence or range.
  **L153 CN**: 开始遍历序列或范围的循环。
- **L154 EN**: Starts a loop over a sequence or range.
  **L154 CN**: 开始遍历序列或范围的循环。
- **L155 EN**: Assigns or initializes `unsigned Opc`.
  **L155 CN**: 对 `unsigned Opc` 进行赋值或初始化。
- **L156 EN**: Begins a conditional branch.
  **L156 CN**: 开始一个条件分支。
- **L157 EN**: Starts block `(op.HandleAllReturns || T.getOpcode() == TII->getReturnOpcode()))`.
  **L157 CN**: 开始代码块 `(op.HandleAllReturns || T.getOpcode() == TII->getReturnOpcode()))`。
- **L158 EN**: Assigns or initializes `Opc`.
  **L158 CN**: 对 `Opc` 进行赋值或初始化。
- **L159 EN**: Closes the current scope.
  **L159 CN**: 关闭当前作用域。
- **L160 EN**: Begins a conditional branch.
  **L160 CN**: 开始一个条件分支。

### Lines 161-180

````cpp
        Opc = TargetOpcode::PATCHABLE_TAIL_CALL;
      }
      if (Opc != 0) {
        // Prepend the return instruction with PATCHABLE_FUNCTION_EXIT or
        //   PATCHABLE_TAIL_CALL .
        BuildMI(MBB, T, T.getDebugLoc(), TII->get(Opc));
      }
    }
}

PreservedAnalyses
XRayInstrumentationPass::run(MachineFunction &MF,
                             MachineFunctionAnalysisManager &MFAM) {
  MachineDominatorTree *MDT = nullptr;
  MachineLoopInfo *MLI = nullptr;

  if (XRayInstrumentation::needMDTAndMLIAnalyses(MF.getFunction())) {
    MDT = MFAM.getCachedResult<MachineDominatorTreeAnalysis>(MF);
    MLI = MFAM.getCachedResult<MachineLoopAnalysis>(MF);
  }
````
- **L161 EN**: Assigns or initializes `Opc`.
  **L161 CN**: 对 `Opc` 进行赋值或初始化。
- **L162 EN**: Closes the current scope.
  **L162 CN**: 关闭当前作用域。
- **L163 EN**: Begins a conditional branch.
  **L163 CN**: 开始一个条件分支。
- **L164 EN**: Comment documents: `Prepend the return instruction with PATCHABLE_FUNCTION_EXIT or`.
  **L164 CN**: 注释说明：`Prepend the return instruction with PATCHABLE_FUNCTION_EXIT or`。
- **L165 EN**: Comment documents: `PATCHABLE_TAIL_CALL .`.
  **L165 CN**: 注释说明：`PATCHABLE_TAIL_CALL .`。
- **L166 EN**: Executes statement `BuildMI(MBB, T, T.getDebugLoc(), TII->get(Opc));`.
  **L166 CN**: 执行语句 `BuildMI(MBB, T, T.getDebugLoc(), TII->get(Opc));`。
- **L167 EN**: Closes the current scope.
  **L167 CN**: 关闭当前作用域。
- **L168 EN**: Closes the current scope.
  **L168 CN**: 关闭当前作用域。
- **L169 EN**: Closes the current scope.
  **L169 CN**: 关闭当前作用域。
- **L170 EN**: Separates nearby statements for readability.
  **L170 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L171 EN**: Continues logic with `PreservedAnalyses`.
  **L171 CN**: 继续处理逻辑：`PreservedAnalyses`。
- **L172 EN**: Provides part of the signature for `run`.
  **L172 CN**: 给出 `run` 的一部分签名。
- **L173 EN**: Starts block `MachineFunctionAnalysisManager &MFAM)`.
  **L173 CN**: 开始代码块 `MachineFunctionAnalysisManager &MFAM)`。
- **L174 EN**: Assigns or initializes `MachineDominatorTree *MDT`.
  **L174 CN**: 对 `MachineDominatorTree *MDT` 进行赋值或初始化。
- **L175 EN**: Assigns or initializes `MachineLoopInfo *MLI`.
  **L175 CN**: 对 `MachineLoopInfo *MLI` 进行赋值或初始化。
- **L176 EN**: Separates nearby statements for readability.
  **L176 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L177 EN**: Begins a conditional branch.
  **L177 CN**: 开始一个条件分支。
- **L178 EN**: Assigns or initializes `MDT`.
  **L178 CN**: 对 `MDT` 进行赋值或初始化。
- **L179 EN**: Assigns or initializes `MLI`.
  **L179 CN**: 对 `MLI` 进行赋值或初始化。
- **L180 EN**: Closes the current scope.
  **L180 CN**: 关闭当前作用域。

### Lines 181-200

````cpp

  if (!XRayInstrumentation(MDT, MLI).run(MF))
    return PreservedAnalyses::all();

  auto PA = getMachineFunctionPassPreservedAnalyses();
  PA.preserveSet<CFGAnalyses>();
  return PA;
}

bool XRayInstrumentationLegacy::runOnMachineFunction(MachineFunction &MF) {
  MachineDominatorTree *MDT = nullptr;
  MachineLoopInfo *MLI = nullptr;
  if (XRayInstrumentation::needMDTAndMLIAnalyses(MF.getFunction())) {
    auto *MDTWrapper =
        getAnalysisIfAvailable<MachineDominatorTreeWrapperPass>();
    MDT = MDTWrapper ? &MDTWrapper->getDomTree() : nullptr;
    auto *MLIWrapper = getAnalysisIfAvailable<MachineLoopInfoWrapperPass>();
    MLI = MLIWrapper ? &MLIWrapper->getLI() : nullptr;
  }
  return XRayInstrumentation(MDT, MLI).run(MF);
````
- **L181 EN**: Separates nearby statements for readability.
  **L181 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L182 EN**: Begins a conditional branch.
  **L182 CN**: 开始一个条件分支。
- **L183 EN**: Returns `PreservedAnalyses::all()` to the caller.
  **L183 CN**: 向调用者返回 `PreservedAnalyses::all()`。
- **L184 EN**: Separates nearby statements for readability.
  **L184 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L185 EN**: Assigns or initializes `auto PA`.
  **L185 CN**: 对 `auto PA` 进行赋值或初始化。
- **L186 EN**: Executes statement `PA.preserveSet<CFGAnalyses>();`.
  **L186 CN**: 执行语句 `PA.preserveSet<CFGAnalyses>();`。
- **L187 EN**: Returns `PA` to the caller.
  **L187 CN**: 向调用者返回 `PA`。
- **L188 EN**: Closes the current scope.
  **L188 CN**: 关闭当前作用域。
- **L189 EN**: Separates nearby statements for readability.
  **L189 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L190 EN**: Begins the definition of `runOnMachineFunction`.
  **L190 CN**: 开始定义 `runOnMachineFunction`。
- **L191 EN**: Assigns or initializes `MachineDominatorTree *MDT`.
  **L191 CN**: 对 `MachineDominatorTree *MDT` 进行赋值或初始化。
- **L192 EN**: Assigns or initializes `MachineLoopInfo *MLI`.
  **L192 CN**: 对 `MachineLoopInfo *MLI` 进行赋值或初始化。
- **L193 EN**: Begins a conditional branch.
  **L193 CN**: 开始一个条件分支。
- **L194 EN**: Continues logic with `auto *MDTWrapper =`.
  **L194 CN**: 继续处理逻辑：`auto *MDTWrapper =`。
- **L195 EN**: Executes statement `getAnalysisIfAvailable<MachineDominatorTreeWrapperPass>();`.
  **L195 CN**: 执行语句 `getAnalysisIfAvailable<MachineDominatorTreeWrapperPass>();`。
- **L196 EN**: Assigns or initializes `MDT`.
  **L196 CN**: 对 `MDT` 进行赋值或初始化。
- **L197 EN**: Assigns or initializes `auto *MLIWrapper`.
  **L197 CN**: 对 `auto *MLIWrapper` 进行赋值或初始化。
- **L198 EN**: Assigns or initializes `MLI`.
  **L198 CN**: 对 `MLI` 进行赋值或初始化。
- **L199 EN**: Closes the current scope.
  **L199 CN**: 关闭当前作用域。
- **L200 EN**: Returns `XRayInstrumentation(MDT, MLI).run(MF)` to the caller.
  **L200 CN**: 向调用者返回 `XRayInstrumentation(MDT, MLI).run(MF)`。

### Lines 201-220

````cpp
}

bool XRayInstrumentation::run(MachineFunction &MF) {
  auto &F = MF.getFunction();
  auto InstrAttr = F.getFnAttribute("function-instrument");
  bool AlwaysInstrument = alwaysInstrument(F);
  bool NeverInstrument = InstrAttr.isStringAttribute() &&
                         InstrAttr.getValueAsString() == "xray-never";
  if (NeverInstrument && !AlwaysInstrument)
    return false;
  auto IgnoreLoopsAttr = F.getFnAttribute("xray-ignore-loops");

  uint64_t XRayThreshold = 0;
  if (!AlwaysInstrument) {
    bool IgnoreLoops = IgnoreLoopsAttr.isValid();
    XRayThreshold = F.getFnAttributeAsParsedInteger(
        "xray-instruction-threshold", std::numeric_limits<uint64_t>::max());
    if (XRayThreshold == std::numeric_limits<uint64_t>::max())
      return false;

````
- **L201 EN**: Closes the current scope.
  **L201 CN**: 关闭当前作用域。
- **L202 EN**: Separates nearby statements for readability.
  **L202 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L203 EN**: Begins the definition of `run`.
  **L203 CN**: 开始定义 `run`。
- **L204 EN**: Assigns or initializes `auto &F`.
  **L204 CN**: 对 `auto &F` 进行赋值或初始化。
- **L205 EN**: Assigns or initializes `auto InstrAttr`.
  **L205 CN**: 对 `auto InstrAttr` 进行赋值或初始化。
- **L206 EN**: Assigns or initializes `bool AlwaysInstrument`.
  **L206 CN**: 对 `bool AlwaysInstrument` 进行赋值或初始化。
- **L207 EN**: Continues logic with `bool NeverInstrument = InstrAttr.isStringAttribute() &&`.
  **L207 CN**: 继续处理逻辑：`bool NeverInstrument = InstrAttr.isStringAttribute() &&`。
- **L208 EN**: Assigns or initializes `InstrAttr.getValueAsString()`.
  **L208 CN**: 对 `InstrAttr.getValueAsString()` 进行赋值或初始化。
- **L209 EN**: Begins a conditional branch.
  **L209 CN**: 开始一个条件分支。
- **L210 EN**: Returns `false` to the caller.
  **L210 CN**: 向调用者返回 `false`。
- **L211 EN**: Assigns or initializes `auto IgnoreLoopsAttr`.
  **L211 CN**: 对 `auto IgnoreLoopsAttr` 进行赋值或初始化。
- **L212 EN**: Separates nearby statements for readability.
  **L212 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L213 EN**: Assigns or initializes `uint64_t XRayThreshold`.
  **L213 CN**: 对 `uint64_t XRayThreshold` 进行赋值或初始化。
- **L214 EN**: Begins a conditional branch.
  **L214 CN**: 开始一个条件分支。
- **L215 EN**: Assigns or initializes `bool IgnoreLoops`.
  **L215 CN**: 对 `bool IgnoreLoops` 进行赋值或初始化。
- **L216 EN**: Continues logic with `XRayThreshold = F.getFnAttributeAsParsedInteger(`.
  **L216 CN**: 继续处理逻辑：`XRayThreshold = F.getFnAttributeAsParsedInteger(`。
- **L217 EN**: Declares function or method `max`.
  **L217 CN**: 声明函数或方法 `max`。
- **L218 EN**: Begins a conditional branch.
  **L218 CN**: 开始一个条件分支。
- **L219 EN**: Returns `false` to the caller.
  **L219 CN**: 向调用者返回 `false`。
- **L220 EN**: Separates nearby statements for readability.
  **L220 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 221-240

````cpp
    // Count the number of MachineInstr`s in MachineFunction
    uint64_t MICount = 0;
    for (const auto &MBB : MF)
      MICount += MBB.size();

    bool TooFewInstrs = MICount < XRayThreshold;

    if (!IgnoreLoops) {
      // Get MachineDominatorTree or compute it on the fly if it's unavailable
      MachineDominatorTree ComputedMDT;
      if (!MDT) {
        ComputedMDT.recalculate(MF);
        MDT = &ComputedMDT;
      }

      // Get MachineLoopInfo or compute it on the fly if it's unavailable
      MachineLoopInfo ComputedMLI;
      if (!MLI) {
        ComputedMLI.analyze(*MDT);
        MLI = &ComputedMLI;
````
- **L221 EN**: Comment documents: `Count the number of MachineInstr's in MachineFunction`.
  **L221 CN**: 注释说明：`Count the number of MachineInstr's in MachineFunction`。
- **L222 EN**: Assigns or initializes `uint64_t MICount`.
  **L222 CN**: 对 `uint64_t MICount` 进行赋值或初始化。
- **L223 EN**: Starts a loop over a sequence or range.
  **L223 CN**: 开始遍历序列或范围的循环。
- **L224 EN**: Assigns or initializes `MICount +`.
  **L224 CN**: 对 `MICount +` 进行赋值或初始化。
- **L225 EN**: Separates nearby statements for readability.
  **L225 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L226 EN**: Assigns or initializes `bool TooFewInstrs`.
  **L226 CN**: 对 `bool TooFewInstrs` 进行赋值或初始化。
- **L227 EN**: Separates nearby statements for readability.
  **L227 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L228 EN**: Begins a conditional branch.
  **L228 CN**: 开始一个条件分支。
- **L229 EN**: Comment documents: `Get MachineDominatorTree or compute it on the fly if it's unavailable`.
  **L229 CN**: 注释说明：`Get MachineDominatorTree or compute it on the fly if it's unavailable`。
- **L230 EN**: Executes statement `MachineDominatorTree ComputedMDT;`.
  **L230 CN**: 执行语句 `MachineDominatorTree ComputedMDT;`。
- **L231 EN**: Begins a conditional branch.
  **L231 CN**: 开始一个条件分支。
- **L232 EN**: Executes statement `ComputedMDT.recalculate(MF);`.
  **L232 CN**: 执行语句 `ComputedMDT.recalculate(MF);`。
- **L233 EN**: Assigns or initializes `MDT`.
  **L233 CN**: 对 `MDT` 进行赋值或初始化。
- **L234 EN**: Closes the current scope.
  **L234 CN**: 关闭当前作用域。
- **L235 EN**: Separates nearby statements for readability.
  **L235 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L236 EN**: Comment documents: `Get MachineLoopInfo or compute it on the fly if it's unavailable`.
  **L236 CN**: 注释说明：`Get MachineLoopInfo or compute it on the fly if it's unavailable`。
- **L237 EN**: Executes statement `MachineLoopInfo ComputedMLI;`.
  **L237 CN**: 执行语句 `MachineLoopInfo ComputedMLI;`。
- **L238 EN**: Begins a conditional branch.
  **L238 CN**: 开始一个条件分支。
- **L239 EN**: Executes statement `ComputedMLI.analyze(*MDT);`.
  **L239 CN**: 执行语句 `ComputedMLI.analyze(*MDT);`。
- **L240 EN**: Assigns or initializes `MLI`.
  **L240 CN**: 对 `MLI` 进行赋值或初始化。

### Lines 241-260

````cpp
      }

      // Check if we have a loop.
      // FIXME: Maybe make this smarter, and see whether the loops are dependent
      // on inputs or side-effects?
      if (MLI->empty() && TooFewInstrs)
        return false; // Function is too small and has no loops.
    } else if (TooFewInstrs) {
      // Function is too small
      return false;
    }
  }

  // We look for the first non-empty MachineBasicBlock, so that we can insert
  // the function instrumentation in the appropriate place.
  auto MBI = llvm::find_if(
      MF, [&](const MachineBasicBlock &MBB) { return !MBB.empty(); });
  if (MBI == MF.end())
    return false; // The function is empty.

````
- **L241 EN**: Closes the current scope.
  **L241 CN**: 关闭当前作用域。
- **L242 EN**: Separates nearby statements for readability.
  **L242 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L243 EN**: Comment documents: `Check if we have a loop.`.
  **L243 CN**: 注释说明：`Check if we have a loop.`。
- **L244 EN**: Comment documents: `FIXME: Maybe make this smarter, and see whether the loops are dependent`.
  **L244 CN**: 注释说明：`FIXME: Maybe make this smarter, and see whether the loops are dependent`。
- **L245 EN**: Comment documents: `on inputs or side-effects?`.
  **L245 CN**: 注释说明：`on inputs or side-effects?`。
- **L246 EN**: Begins a conditional branch.
  **L246 CN**: 开始一个条件分支。
- **L247 EN**: Returns `false; // Function is too small and has no loops.` to the caller.
  **L247 CN**: 向调用者返回 `false; // Function is too small and has no loops.`。
- **L248 EN**: Starts block `} else if (TooFewInstrs)`.
  **L248 CN**: 开始代码块 `} else if (TooFewInstrs)`。
- **L249 EN**: Comment documents: `Function is too small`.
  **L249 CN**: 注释说明：`Function is too small`。
- **L250 EN**: Returns `false` to the caller.
  **L250 CN**: 向调用者返回 `false`。
- **L251 EN**: Closes the current scope.
  **L251 CN**: 关闭当前作用域。
- **L252 EN**: Closes the current scope.
  **L252 CN**: 关闭当前作用域。
- **L253 EN**: Separates nearby statements for readability.
  **L253 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L254 EN**: Comment documents: `We look for the first non-empty MachineBasicBlock, so that we can insert`.
  **L254 CN**: 注释说明：`We look for the first non-empty MachineBasicBlock, so that we can insert`。
- **L255 EN**: Comment documents: `the function instrumentation in the appropriate place.`.
  **L255 CN**: 注释说明：`the function instrumentation in the appropriate place.`。
- **L256 EN**: Provides part of the signature for `find_if`.
  **L256 CN**: 给出 `find_if` 的一部分签名。
- **L257 EN**: Executes statement `MF, [&](const MachineBasicBlock &MBB) { return !MBB.empty(); });`.
  **L257 CN**: 执行语句 `MF, [&](const MachineBasicBlock &MBB) { return !MBB.empty(); });`。
- **L258 EN**: Begins a conditional branch.
  **L258 CN**: 开始一个条件分支。
- **L259 EN**: Returns `false; // The function is empty.` to the caller.
  **L259 CN**: 向调用者返回 `false; // The function is empty.`。
- **L260 EN**: Separates nearby statements for readability.
  **L260 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 261-280

````cpp
  auto *TII = MF.getSubtarget().getInstrInfo();
  auto &FirstMBB = *MBI;
  auto &FirstMI = *FirstMBB.begin();

  if (!MF.getSubtarget().isXRaySupported()) {

    const Function &Fn = FirstMBB.getParent()->getFunction();
    Fn.getContext().diagnose(DiagnosticInfoUnsupported(
        Fn, "An attempt to perform XRay instrumentation for an"
            " unsupported target."));

    return false;
  }

  if (!F.hasFnAttribute("xray-skip-entry")) {
    // First, insert an PATCHABLE_FUNCTION_ENTER as the first instruction of the
    // MachineFunction.
    BuildMI(FirstMBB, FirstMI, FirstMI.getDebugLoc(),
            TII->get(TargetOpcode::PATCHABLE_FUNCTION_ENTER));
  }
````
- **L261 EN**: Assigns or initializes `auto *TII`.
  **L261 CN**: 对 `auto *TII` 进行赋值或初始化。
- **L262 EN**: Assigns or initializes `auto &FirstMBB`.
  **L262 CN**: 对 `auto &FirstMBB` 进行赋值或初始化。
- **L263 EN**: Assigns or initializes `auto &FirstMI`.
  **L263 CN**: 对 `auto &FirstMI` 进行赋值或初始化。
- **L264 EN**: Separates nearby statements for readability.
  **L264 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L265 EN**: Begins a conditional branch.
  **L265 CN**: 开始一个条件分支。
- **L266 EN**: Separates nearby statements for readability.
  **L266 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L267 EN**: Assigns or initializes `const Function &Fn`.
  **L267 CN**: 对 `const Function &Fn` 进行赋值或初始化。
- **L268 EN**: Continues logic with `Fn.getContext().diagnose(DiagnosticInfoUnsupported(`.
  **L268 CN**: 继续处理逻辑：`Fn.getContext().diagnose(DiagnosticInfoUnsupported(`。
- **L269 EN**: Continues logic with `Fn, "An attempt to perform XRay instrumentation for an"`.
  **L269 CN**: 继续处理逻辑：`Fn, "An attempt to perform XRay instrumentation for an"`。
- **L270 EN**: Executes statement `" unsupported target."));`.
  **L270 CN**: 执行语句 `" unsupported target."));`。
- **L271 EN**: Separates nearby statements for readability.
  **L271 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L272 EN**: Returns `false` to the caller.
  **L272 CN**: 向调用者返回 `false`。
- **L273 EN**: Closes the current scope.
  **L273 CN**: 关闭当前作用域。
- **L274 EN**: Separates nearby statements for readability.
  **L274 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L275 EN**: Begins a conditional branch.
  **L275 CN**: 开始一个条件分支。
- **L276 EN**: Comment documents: `First, insert an PATCHABLE_FUNCTION_ENTER as the first instruction of th…`.
  **L276 CN**: 注释说明：`First, insert an PATCHABLE_FUNCTION_ENTER as the first instruction of th…`。
- **L277 EN**: Comment documents: `MachineFunction.`.
  **L277 CN**: 注释说明：`MachineFunction.`。
- **L278 EN**: Continues logic with `BuildMI(FirstMBB, FirstMI, FirstMI.getDebugLoc(),`.
  **L278 CN**: 继续处理逻辑：`BuildMI(FirstMBB, FirstMI, FirstMI.getDebugLoc(),`。
- **L279 EN**: Executes statement `TII->get(TargetOpcode::PATCHABLE_FUNCTION_ENTER));`.
  **L279 CN**: 执行语句 `TII->get(TargetOpcode::PATCHABLE_FUNCTION_ENTER));`。
- **L280 EN**: Closes the current scope.
  **L280 CN**: 关闭当前作用域。

### Lines 281-300

````cpp

  if (!F.hasFnAttribute("xray-skip-exit")) {
    switch (MF.getTarget().getTargetTriple().getArch()) {
    case Triple::ArchType::arm:
    case Triple::ArchType::thumb:
    case Triple::ArchType::aarch64:
    case Triple::ArchType::hexagon:
    case Triple::ArchType::loongarch64:
    case Triple::ArchType::mips:
    case Triple::ArchType::mipsel:
    case Triple::ArchType::mips64:
    case Triple::ArchType::mips64el:
    case Triple::ArchType::riscv32:
    case Triple::ArchType::riscv64: {
      // For the architectures which don't have a single return instruction
      InstrumentationOptions op;
      // AArch64 and RISC-V support patching tail calls.
      op.HandleTailcall = MF.getTarget().getTargetTriple().isAArch64() ||
                          MF.getTarget().getTargetTriple().isRISCV();
      op.HandleAllReturns = true;
````
- **L281 EN**: Separates nearby statements for readability.
  **L281 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L282 EN**: Begins a conditional branch.
  **L282 CN**: 开始一个条件分支。
- **L283 EN**: Starts a multi-way branch.
  **L283 CN**: 开始一个多路分支。
- **L284 EN**: Handles one switch case.
  **L284 CN**: 处理一个 switch 分支。
- **L285 EN**: Handles one switch case.
  **L285 CN**: 处理一个 switch 分支。
- **L286 EN**: Handles one switch case.
  **L286 CN**: 处理一个 switch 分支。
- **L287 EN**: Handles one switch case.
  **L287 CN**: 处理一个 switch 分支。
- **L288 EN**: Handles one switch case.
  **L288 CN**: 处理一个 switch 分支。
- **L289 EN**: Handles one switch case.
  **L289 CN**: 处理一个 switch 分支。
- **L290 EN**: Handles one switch case.
  **L290 CN**: 处理一个 switch 分支。
- **L291 EN**: Handles one switch case.
  **L291 CN**: 处理一个 switch 分支。
- **L292 EN**: Handles one switch case.
  **L292 CN**: 处理一个 switch 分支。
- **L293 EN**: Handles one switch case.
  **L293 CN**: 处理一个 switch 分支。
- **L294 EN**: Handles one switch case.
  **L294 CN**: 处理一个 switch 分支。
- **L295 EN**: Comment documents: `For the architectures which don't have a single return instruction`.
  **L295 CN**: 注释说明：`For the architectures which don't have a single return instruction`。
- **L296 EN**: Executes statement `InstrumentationOptions op;`.
  **L296 CN**: 执行语句 `InstrumentationOptions op;`。
- **L297 EN**: Comment documents: `AArch64 and RISC-V support patching tail calls.`.
  **L297 CN**: 注释说明：`AArch64 and RISC-V support patching tail calls.`。
- **L298 EN**: Continues logic with `op.HandleTailcall = MF.getTarget().getTargetTriple().isAArch64() ||`.
  **L298 CN**: 继续处理逻辑：`op.HandleTailcall = MF.getTarget().getTargetTriple().isAArch64() ||`。
- **L299 EN**: Executes statement `MF.getTarget().getTargetTriple().isRISCV();`.
  **L299 CN**: 执行语句 `MF.getTarget().getTargetTriple().isRISCV();`。
- **L300 EN**: Assigns or initializes `op.HandleAllReturns`.
  **L300 CN**: 对 `op.HandleAllReturns` 进行赋值或初始化。

### Lines 301-320

````cpp
      prependRetWithPatchableExit(MF, TII, op);
      break;
    }
    case Triple::ArchType::ppc64le:
    case Triple::ArchType::systemz: {
      // PPC has conditional returns. Turn them into branch and plain returns.
      InstrumentationOptions op;
      op.HandleTailcall = false;
      op.HandleAllReturns = true;
      replaceRetWithPatchableRet(MF, TII, op);
      break;
    }
    default: {
      // For the architectures that have a single return instruction (such as
      //   RETQ on x86_64).
      InstrumentationOptions op;
      op.HandleTailcall = true;
      op.HandleAllReturns = false;
      replaceRetWithPatchableRet(MF, TII, op);
      break;
````
- **L301 EN**: Executes statement `prependRetWithPatchableExit(MF, TII, op);`.
  **L301 CN**: 执行语句 `prependRetWithPatchableExit(MF, TII, op);`。
- **L302 EN**: Breaks out of the current control-flow construct.
  **L302 CN**: 跳出当前控制流结构。
- **L303 EN**: Closes the current scope.
  **L303 CN**: 关闭当前作用域。
- **L304 EN**: Handles one switch case.
  **L304 CN**: 处理一个 switch 分支。
- **L305 EN**: Handles one switch case.
  **L305 CN**: 处理一个 switch 分支。
- **L306 EN**: Comment documents: `PPC has conditional returns. Turn them into branch and plain returns.`.
  **L306 CN**: 注释说明：`PPC has conditional returns. Turn them into branch and plain returns.`。
- **L307 EN**: Executes statement `InstrumentationOptions op;`.
  **L307 CN**: 执行语句 `InstrumentationOptions op;`。
- **L308 EN**: Assigns or initializes `op.HandleTailcall`.
  **L308 CN**: 对 `op.HandleTailcall` 进行赋值或初始化。
- **L309 EN**: Assigns or initializes `op.HandleAllReturns`.
  **L309 CN**: 对 `op.HandleAllReturns` 进行赋值或初始化。
- **L310 EN**: Executes statement `replaceRetWithPatchableRet(MF, TII, op);`.
  **L310 CN**: 执行语句 `replaceRetWithPatchableRet(MF, TII, op);`。
- **L311 EN**: Breaks out of the current control-flow construct.
  **L311 CN**: 跳出当前控制流结构。
- **L312 EN**: Closes the current scope.
  **L312 CN**: 关闭当前作用域。
- **L313 EN**: Handles the default switch case.
  **L313 CN**: 处理 switch 的默认分支。
- **L314 EN**: Comment documents: `For the architectures that have a single return instruction (such as`.
  **L314 CN**: 注释说明：`For the architectures that have a single return instruction (such as`。
- **L315 EN**: Comment documents: `RETQ on x86_64).`.
  **L315 CN**: 注释说明：`RETQ on x86_64).`。
- **L316 EN**: Executes statement `InstrumentationOptions op;`.
  **L316 CN**: 执行语句 `InstrumentationOptions op;`。
- **L317 EN**: Assigns or initializes `op.HandleTailcall`.
  **L317 CN**: 对 `op.HandleTailcall` 进行赋值或初始化。
- **L318 EN**: Assigns or initializes `op.HandleAllReturns`.
  **L318 CN**: 对 `op.HandleAllReturns` 进行赋值或初始化。
- **L319 EN**: Executes statement `replaceRetWithPatchableRet(MF, TII, op);`.
  **L319 CN**: 执行语句 `replaceRetWithPatchableRet(MF, TII, op);`。
- **L320 EN**: Breaks out of the current control-flow construct.
  **L320 CN**: 跳出当前控制流结构。

### Lines 321-333

````cpp
    }
    }
  }
  return true;
}

char XRayInstrumentationLegacy::ID = 0;
char &llvm::XRayInstrumentationID = XRayInstrumentationLegacy::ID;
INITIALIZE_PASS_BEGIN(XRayInstrumentationLegacy, "xray-instrumentation",
                      "Insert XRay ops", false, false)
INITIALIZE_PASS_DEPENDENCY(MachineLoopInfoWrapperPass)
INITIALIZE_PASS_END(XRayInstrumentationLegacy, "xray-instrumentation",
                    "Insert XRay ops", false, false)
````
- **L321 EN**: Closes the current scope.
  **L321 CN**: 关闭当前作用域。
- **L322 EN**: Closes the current scope.
  **L322 CN**: 关闭当前作用域。
- **L323 EN**: Closes the current scope.
  **L323 CN**: 关闭当前作用域。
- **L324 EN**: Returns `true` to the caller.
  **L324 CN**: 向调用者返回 `true`。
- **L325 EN**: Closes the current scope.
  **L325 CN**: 关闭当前作用域。
- **L326 EN**: Separates nearby statements for readability.
  **L326 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L327 EN**: Assigns or initializes `char XRayInstrumentationLegacy::ID`.
  **L327 CN**: 对 `char XRayInstrumentationLegacy::ID` 进行赋值或初始化。
- **L328 EN**: Assigns or initializes `char &llvm::XRayInstrumentationID`.
  **L328 CN**: 对 `char &llvm::XRayInstrumentationID` 进行赋值或初始化。
- **L329 EN**: Continues logic with `INITIALIZE_PASS_BEGIN(XRayInstrumentationLegacy, "xray-instrumentation",`.
  **L329 CN**: 继续处理逻辑：`INITIALIZE_PASS_BEGIN(XRayInstrumentationLegacy, "xray-instrumentation",`。
- **L330 EN**: Continues logic with `"Insert XRay ops", false, false)`.
  **L330 CN**: 继续处理逻辑：`"Insert XRay ops", false, false)`。
- **L331 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(MachineLoopInfoWrapperPass)`.
  **L331 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(MachineLoopInfoWrapperPass)`。
- **L332 EN**: Continues logic with `INITIALIZE_PASS_END(XRayInstrumentationLegacy, "xray-instrumentation",`.
  **L332 CN**: 继续处理逻辑：`INITIALIZE_PASS_END(XRayInstrumentationLegacy, "xray-instrumentation",`。
- **L333 EN**: Continues logic with `"Insert XRay ops", false, false)`.
  **L333 CN**: 继续处理逻辑：`"Insert XRay ops", false, false)`。

## Key Concepts / 关键概念
- **Machine function processing** / **MachineFunction 处理**
- **Machine basic block handling** / **MachineBasicBlock 处理**
- **Machine instruction manipulation** / **机器指令操作**
- **Control-flow updates** / **控制流更新**
- **Debug information handling** / **调试信息处理**
- **Target-specific hooks** / **目标相关钩子**
- **Pseudo-instruction expansion** / **伪指令展开**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/XRayInstrumentation.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineDominators.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionAnalysis.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineLoopInfo.h`, `llvm/CodeGen/MachinePassManager.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/IR/Attributes.h`, `llvm/IR/DiagnosticInfo.h`, `llvm/IR/Function.h`, `llvm/InitializePasses.h`, `llvm/Pass.h`, `llvm/Target/TargetMachine.h`, `llvm/TargetParser/Triple.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
