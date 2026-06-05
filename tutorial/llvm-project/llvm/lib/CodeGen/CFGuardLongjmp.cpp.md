# CFGuardLongjmp.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/CFGuardLongjmp.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `Longjmp symbols for CFGuard --------*- C++ -*` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“Longjmp symbols for CFGuard --------*- C++ -*”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===-- CFGuardLongjmp.cpp - Longjmp symbols for CFGuard --------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file contains a machine function pass to insert a symbol after each
/// call to _setjmp and store this in the MachineFunction's LongjmpTargets
/// vector. This will be used to emit the table of valid longjmp targets used
/// by Control Flow Guard.
///
//===----------------------------------------------------------------------===//

#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
````
- **L1 EN**: Comment documents: `===-- CFGuardLongjmp.cpp - Longjmp symbols for CFGuard --------*- C++ -*…`.
  **L1 CN**: 注释说明：`===-- CFGuardLongjmp.cpp - Longjmp symbols for CFGuard --------*- C++ -*…`。
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
- **L9 EN**: Comment documents: `\file`.
  **L9 CN**: 注释说明：`\file`。
- **L10 EN**: Comment documents: `This file contains a machine function pass to insert a symbol after each`.
  **L10 CN**: 注释说明：`This file contains a machine function pass to insert a symbol after each`。
- **L11 EN**: Comment documents: `call to _setjmp and store this in the MachineFunction's LongjmpTargets`.
  **L11 CN**: 注释说明：`call to _setjmp and store this in the MachineFunction's LongjmpTargets`。
- **L12 EN**: Comment documents: `vector. This will be used to emit the table of valid longjmp targets use…`.
  **L12 CN**: 注释说明：`vector. This will be used to emit the table of valid longjmp targets use…`。
- **L13 EN**: Comment documents: `by Control Flow Guard.`.
  **L13 CN**: 注释说明：`by Control Flow Guard.`。
- **L14 EN**: Continues the surrounding comment block.
  **L14 CN**: 延续周围的注释块。
- **L15 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L15 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L16 EN**: Separates nearby statements for readability.
  **L16 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L17 EN**: Includes LLVM header `llvm/ADT/Statistic.h` for Statistic support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/ADT/Statistic.h`，用于 Statistic 相关支持。
- **L18 EN**: Includes LLVM header `llvm/CodeGen/MachineBasicBlock.h` for MachineBasicBlock support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineBasicBlock.h`，用于 MachineBasicBlock 相关支持。
- **L19 EN**: Includes LLVM header `llvm/CodeGen/MachineFunctionPass.h` for MachineFunctionPass support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFunctionPass.h`，用于 MachineFunctionPass 相关支持。
- **L20 EN**: Includes LLVM header `llvm/CodeGen/MachineInstr.h` for MachineInstr support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineInstr.h`，用于 MachineInstr 相关支持。

### Lines 21-40

````cpp
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/IR/Module.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "cfguard-longjmp"

STATISTIC(CFGuardLongjmpTargets,
          "Number of Control Flow Guard longjmp targets");

namespace {

/// MachineFunction pass to insert a symbol after each call to _setjmp and store
/// this in the MachineFunction's LongjmpTargets vector.
class CFGuardLongjmp : public MachineFunctionPass {
public:
  static char ID;
````
- **L21 EN**: Includes LLVM header `llvm/CodeGen/MachineModuleInfo.h` for MachineModuleInfo support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineModuleInfo.h`，用于 MachineModuleInfo 相关支持。
- **L22 EN**: Includes LLVM header `llvm/CodeGen/MachineOperand.h` for MachineOperand support.
  **L22 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineOperand.h`，用于 MachineOperand 相关支持。
- **L23 EN**: Includes LLVM header `llvm/CodeGen/Passes.h` for Passes support.
  **L23 CN**: 引入 LLVM 头文件 `llvm/CodeGen/Passes.h`，用于 Passes 相关支持。
- **L24 EN**: Includes LLVM header `llvm/IR/Module.h` for Module support.
  **L24 CN**: 引入 LLVM 头文件 `llvm/IR/Module.h`，用于 Module 相关支持。
- **L25 EN**: Includes LLVM header `llvm/InitializePasses.h` for InitializePasses support.
  **L25 CN**: 引入 LLVM 头文件 `llvm/InitializePasses.h`，用于 InitializePasses 相关支持。
- **L26 EN**: Separates nearby statements for readability.
  **L26 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L27 EN**: Imports namespace `llvm` into this translation unit.
  **L27 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L28 EN**: Separates nearby statements for readability.
  **L28 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L29 EN**: Defines the LLVM debug channel used by this file.
  **L29 CN**: 定义该文件使用的 LLVM 调试通道。
- **L30 EN**: Separates nearby statements for readability.
  **L30 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L31 EN**: Registers a pass statistic counter.
  **L31 CN**: 注册一个 pass 统计计数器。
- **L32 EN**: Executes statement `"Number of Control Flow Guard longjmp targets");`.
  **L32 CN**: 执行语句 `"Number of Control Flow Guard longjmp targets");`。
- **L33 EN**: Separates nearby statements for readability.
  **L33 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L34 EN**: Opens namespace ``.
  **L34 CN**: 打开命名空间 ``。
- **L35 EN**: Separates nearby statements for readability.
  **L35 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L36 EN**: Comment documents: `MachineFunction pass to insert a symbol after each call to _setjmp and s…`.
  **L36 CN**: 注释说明：`MachineFunction pass to insert a symbol after each call to _setjmp and s…`。
- **L37 EN**: Comment documents: `this in the MachineFunction's LongjmpTargets vector.`.
  **L37 CN**: 注释说明：`this in the MachineFunction's LongjmpTargets vector.`。
- **L38 EN**: Starts the declaration of class `CFGuardLongjmp`.
  **L38 CN**: 开始声明 class `CFGuardLongjmp`。
- **L39 EN**: Continues logic with `public:`.
  **L39 CN**: 继续处理逻辑：`public:`。
- **L40 EN**: Executes statement `static char ID;`.
  **L40 CN**: 执行语句 `static char ID;`。

### Lines 41-60

````cpp

  CFGuardLongjmp() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override {
    return "Control Flow Guard longjmp targets";
  }

  bool runOnMachineFunction(MachineFunction &MF) override;
};

} // end anonymous namespace

char CFGuardLongjmp::ID = 0;

INITIALIZE_PASS(CFGuardLongjmp, "CFGuardLongjmp",
                "Insert symbols at valid longjmp targets for /guard:cf", false,
                false)
FunctionPass *llvm::createCFGuardLongjmpPass() { return new CFGuardLongjmp(); }

bool CFGuardLongjmp::runOnMachineFunction(MachineFunction &MF) {
````
- **L41 EN**: Separates nearby statements for readability.
  **L41 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L42 EN**: Continues logic with `CFGuardLongjmp() : MachineFunctionPass(ID) {}`.
  **L42 CN**: 继续处理逻辑：`CFGuardLongjmp() : MachineFunctionPass(ID) {}`。
- **L43 EN**: Separates nearby statements for readability.
  **L43 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L44 EN**: Begins the definition of `getPassName`.
  **L44 CN**: 开始定义 `getPassName`。
- **L45 EN**: Returns `"Control Flow Guard longjmp targets"` to the caller.
  **L45 CN**: 向调用者返回 `"Control Flow Guard longjmp targets"`。
- **L46 EN**: Closes the current scope.
  **L46 CN**: 关闭当前作用域。
- **L47 EN**: Separates nearby statements for readability.
  **L47 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L48 EN**: Declares function or method `runOnMachineFunction`.
  **L48 CN**: 声明函数或方法 `runOnMachineFunction`。
- **L49 EN**: Closes the current scope.
  **L49 CN**: 关闭当前作用域。
- **L50 EN**: Separates nearby statements for readability.
  **L50 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L51 EN**: Continues logic with `} // end anonymous namespace`.
  **L51 CN**: 继续处理逻辑：`} // end anonymous namespace`。
- **L52 EN**: Separates nearby statements for readability.
  **L52 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L53 EN**: Assigns or initializes `char CFGuardLongjmp::ID`.
  **L53 CN**: 对 `char CFGuardLongjmp::ID` 进行赋值或初始化。
- **L54 EN**: Separates nearby statements for readability.
  **L54 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L55 EN**: Continues logic with `INITIALIZE_PASS(CFGuardLongjmp, "CFGuardLongjmp",`.
  **L55 CN**: 继续处理逻辑：`INITIALIZE_PASS(CFGuardLongjmp, "CFGuardLongjmp",`。
- **L56 EN**: Continues logic with `"Insert symbols at valid longjmp targets for /guard:cf", false,`.
  **L56 CN**: 继续处理逻辑：`"Insert symbols at valid longjmp targets for /guard:cf", false,`。
- **L57 EN**: Continues logic with `false)`.
  **L57 CN**: 继续处理逻辑：`false)`。
- **L58 EN**: Provides part of the signature for `createCFGuardLongjmpPass`.
  **L58 CN**: 给出 `createCFGuardLongjmpPass` 的一部分签名。
- **L59 EN**: Separates nearby statements for readability.
  **L59 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L60 EN**: Begins the definition of `runOnMachineFunction`.
  **L60 CN**: 开始定义 `runOnMachineFunction`。

### Lines 61-80

````cpp

  // Skip modules for which the cfguard flag is not set.
  if (MF.getFunction().getParent()->getControlFlowGuardMode() ==
      ControlFlowGuardMode::Disabled)
    return false;

  // Skip functions that do not have calls to _setjmp.
  if (!MF.getFunction().callsFunctionThatReturnsTwice())
    return false;

  SmallVector<MachineInstr *, 8> SetjmpCalls;

  // Iterate over all instructions in the function and add calls to functions
  // that return twice to the list of targets.
  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : MBB) {

      // Skip instructions that are not calls.
      if (!MI.isCall() || MI.getNumOperands() < 1)
        continue;
````
- **L61 EN**: Separates nearby statements for readability.
  **L61 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L62 EN**: Comment documents: `Skip modules for which the cfguard flag is not set.`.
  **L62 CN**: 注释说明：`Skip modules for which the cfguard flag is not set.`。
- **L63 EN**: Begins a conditional branch.
  **L63 CN**: 开始一个条件分支。
- **L64 EN**: Continues logic with `ControlFlowGuardMode::Disabled)`.
  **L64 CN**: 继续处理逻辑：`ControlFlowGuardMode::Disabled)`。
- **L65 EN**: Returns `false` to the caller.
  **L65 CN**: 向调用者返回 `false`。
- **L66 EN**: Separates nearby statements for readability.
  **L66 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L67 EN**: Comment documents: `Skip functions that do not have calls to _setjmp.`.
  **L67 CN**: 注释说明：`Skip functions that do not have calls to _setjmp.`。
- **L68 EN**: Begins a conditional branch.
  **L68 CN**: 开始一个条件分支。
- **L69 EN**: Returns `false` to the caller.
  **L69 CN**: 向调用者返回 `false`。
- **L70 EN**: Separates nearby statements for readability.
  **L70 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L71 EN**: Executes statement `SmallVector<MachineInstr *, 8> SetjmpCalls;`.
  **L71 CN**: 执行语句 `SmallVector<MachineInstr *, 8> SetjmpCalls;`。
- **L72 EN**: Separates nearby statements for readability.
  **L72 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L73 EN**: Comment documents: `Iterate over all instructions in the function and add calls to functions`.
  **L73 CN**: 注释说明：`Iterate over all instructions in the function and add calls to functions`。
- **L74 EN**: Comment documents: `that return twice to the list of targets.`.
  **L74 CN**: 注释说明：`that return twice to the list of targets.`。
- **L75 EN**: Starts a loop over a sequence or range.
  **L75 CN**: 开始遍历序列或范围的循环。
- **L76 EN**: Starts a loop over a sequence or range.
  **L76 CN**: 开始遍历序列或范围的循环。
- **L77 EN**: Separates nearby statements for readability.
  **L77 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L78 EN**: Comment documents: `Skip instructions that are not calls.`.
  **L78 CN**: 注释说明：`Skip instructions that are not calls.`。
- **L79 EN**: Begins a conditional branch.
  **L79 CN**: 开始一个条件分支。
- **L80 EN**: Skips to the next loop iteration.
  **L80 CN**: 跳到下一次循环迭代。

### Lines 81-100

````cpp

      // Iterate over operands to find calls to global functions.
      for (MachineOperand &MO : MI.operands()) {
        if (!MO.isGlobal())
          continue;

        auto *F = dyn_cast<Function>(MO.getGlobal());
        if (!F)
          continue;

        // If the instruction calls a function that returns twice, add
        // it to the list of targets.
        if (F->hasFnAttribute(Attribute::ReturnsTwice)) {
          SetjmpCalls.push_back(&MI);
          break;
        }
      }
    }
  }

````
- **L81 EN**: Separates nearby statements for readability.
  **L81 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L82 EN**: Comment documents: `Iterate over operands to find calls to global functions.`.
  **L82 CN**: 注释说明：`Iterate over operands to find calls to global functions.`。
- **L83 EN**: Starts a loop over a sequence or range.
  **L83 CN**: 开始遍历序列或范围的循环。
- **L84 EN**: Begins a conditional branch.
  **L84 CN**: 开始一个条件分支。
- **L85 EN**: Skips to the next loop iteration.
  **L85 CN**: 跳到下一次循环迭代。
- **L86 EN**: Separates nearby statements for readability.
  **L86 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L87 EN**: Assigns or initializes `auto *F`.
  **L87 CN**: 对 `auto *F` 进行赋值或初始化。
- **L88 EN**: Begins a conditional branch.
  **L88 CN**: 开始一个条件分支。
- **L89 EN**: Skips to the next loop iteration.
  **L89 CN**: 跳到下一次循环迭代。
- **L90 EN**: Separates nearby statements for readability.
  **L90 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L91 EN**: Comment documents: `If the instruction calls a function that returns twice, add`.
  **L91 CN**: 注释说明：`If the instruction calls a function that returns twice, add`。
- **L92 EN**: Comment documents: `it to the list of targets.`.
  **L92 CN**: 注释说明：`it to the list of targets.`。
- **L93 EN**: Begins a conditional branch.
  **L93 CN**: 开始一个条件分支。
- **L94 EN**: Executes statement `SetjmpCalls.push_back(&MI);`.
  **L94 CN**: 执行语句 `SetjmpCalls.push_back(&MI);`。
- **L95 EN**: Breaks out of the current control-flow construct.
  **L95 CN**: 跳出当前控制流结构。
- **L96 EN**: Closes the current scope.
  **L96 CN**: 关闭当前作用域。
- **L97 EN**: Closes the current scope.
  **L97 CN**: 关闭当前作用域。
- **L98 EN**: Closes the current scope.
  **L98 CN**: 关闭当前作用域。
- **L99 EN**: Closes the current scope.
  **L99 CN**: 关闭当前作用域。
- **L100 EN**: Separates nearby statements for readability.
  **L100 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 101-120

````cpp
  if (SetjmpCalls.empty())
    return false;

  unsigned SetjmpNum = 0;

  // For each possible target, create a new symbol and insert it immediately
  // after the call to setjmp. Add this symbol to the MachineFunction's list
  // of longjmp targets.
  for (MachineInstr *Setjmp : SetjmpCalls) {
    SmallString<128> SymbolName;
    raw_svector_ostream(SymbolName) << "$cfgsj_" << MF.getName() << SetjmpNum++;
    MCSymbol *SjSymbol = MF.getContext().getOrCreateSymbol(SymbolName);

    Setjmp->setPostInstrSymbol(MF, SjSymbol);
    MF.addLongjmpTarget(SjSymbol);
    CFGuardLongjmpTargets++;
  }

  return true;
}
````
- **L101 EN**: Begins a conditional branch.
  **L101 CN**: 开始一个条件分支。
- **L102 EN**: Returns `false` to the caller.
  **L102 CN**: 向调用者返回 `false`。
- **L103 EN**: Separates nearby statements for readability.
  **L103 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L104 EN**: Assigns or initializes `unsigned SetjmpNum`.
  **L104 CN**: 对 `unsigned SetjmpNum` 进行赋值或初始化。
- **L105 EN**: Separates nearby statements for readability.
  **L105 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L106 EN**: Comment documents: `For each possible target, create a new symbol and insert it immediately`.
  **L106 CN**: 注释说明：`For each possible target, create a new symbol and insert it immediately`。
- **L107 EN**: Comment documents: `after the call to setjmp. Add this symbol to the MachineFunction's list`.
  **L107 CN**: 注释说明：`after the call to setjmp. Add this symbol to the MachineFunction's list`。
- **L108 EN**: Comment documents: `of longjmp targets.`.
  **L108 CN**: 注释说明：`of longjmp targets.`。
- **L109 EN**: Starts a loop over a sequence or range.
  **L109 CN**: 开始遍历序列或范围的循环。
- **L110 EN**: Executes statement `SmallString<128> SymbolName;`.
  **L110 CN**: 执行语句 `SmallString<128> SymbolName;`。
- **L111 EN**: Executes statement `raw_svector_ostream(SymbolName) << "$cfgsj_" << MF.getName() << SetjmpNu…`.
  **L111 CN**: 执行语句 `raw_svector_ostream(SymbolName) << "$cfgsj_" << MF.getName() << SetjmpNu…`。
- **L112 EN**: Assigns or initializes `MCSymbol *SjSymbol`.
  **L112 CN**: 对 `MCSymbol *SjSymbol` 进行赋值或初始化。
- **L113 EN**: Separates nearby statements for readability.
  **L113 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L114 EN**: Executes statement `Setjmp->setPostInstrSymbol(MF, SjSymbol);`.
  **L114 CN**: 执行语句 `Setjmp->setPostInstrSymbol(MF, SjSymbol);`。
- **L115 EN**: Executes statement `MF.addLongjmpTarget(SjSymbol);`.
  **L115 CN**: 执行语句 `MF.addLongjmpTarget(SjSymbol);`。
- **L116 EN**: Executes statement `CFGuardLongjmpTargets++;`.
  **L116 CN**: 执行语句 `CFGuardLongjmpTargets++;`。
- **L117 EN**: Closes the current scope.
  **L117 CN**: 关闭当前作用域。
- **L118 EN**: Separates nearby statements for readability.
  **L118 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L119 EN**: Returns `true` to the caller.
  **L119 CN**: 向调用者返回 `true`。
- **L120 EN**: Closes the current scope.
  **L120 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Machine function processing** / **MachineFunction 处理**
- **Machine basic block handling** / **MachineBasicBlock 处理**
- **Machine instruction manipulation** / **机器指令操作**
- **Debug information handling** / **调试信息处理**
- **Target-specific hooks** / **目标相关钩子**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/ADT/Statistic.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineModuleInfo.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/CodeGen/Passes.h`, `llvm/IR/Module.h`, `llvm/InitializePasses.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
