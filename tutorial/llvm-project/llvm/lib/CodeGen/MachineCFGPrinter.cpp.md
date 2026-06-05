# MachineCFGPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/MachineCFGPrinter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `DOT Printer for Machine Functions` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“DOT Printer for Machine Functions”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- MachineCFGPrinter.cpp - DOT Printer for Machine Functions ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//===----------------------------------------------------------------------===//
//
// This file defines the `-dot-machine-cfg` analysis pass, which emits
// Machine Function in DOT format in file titled `<prefix>.<function-name>.dot.
//===----------------------------------------------------------------------===//

#include "llvm/CodeGen/MachineCFGPrinter.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachinePassManager.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
````
- **L1 EN**: Comment documents: `===- MachineCFGPrinter.cpp - DOT Printer for Machine Functions ---------…`.
  **L1 CN**: 注释说明：`===- MachineCFGPrinter.cpp - DOT Printer for Machine Functions ---------…`。
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
- **L8 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L8 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L9 EN**: Continues the surrounding comment block.
  **L9 CN**: 延续周围的注释块。
- **L10 EN**: Comment documents: `This file defines the '-dot-machine-cfg' analysis pass, which emits`.
  **L10 CN**: 注释说明：`This file defines the '-dot-machine-cfg' analysis pass, which emits`。
- **L11 EN**: Comment documents: `Machine Function in DOT format in file titled '<prefix>.<function-name>.…`.
  **L11 CN**: 注释说明：`Machine Function in DOT format in file titled '<prefix>.<function-name>.…`。
- **L12 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L12 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L13 EN**: Separates nearby statements for readability.
  **L13 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L14 EN**: Includes LLVM header `llvm/CodeGen/MachineCFGPrinter.h` for MachineCFGPrinter support.
  **L14 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineCFGPrinter.h`，用于 MachineCFGPrinter 相关支持。
- **L15 EN**: Includes LLVM header `llvm/CodeGen/MachineBasicBlock.h` for MachineBasicBlock support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineBasicBlock.h`，用于 MachineBasicBlock 相关支持。
- **L16 EN**: Includes LLVM header `llvm/CodeGen/MachineFunctionPass.h` for MachineFunctionPass support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFunctionPass.h`，用于 MachineFunctionPass 相关支持。
- **L17 EN**: Includes LLVM header `llvm/CodeGen/MachinePassManager.h` for MachinePassManager support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachinePassManager.h`，用于 MachinePassManager 相关支持。
- **L18 EN**: Includes LLVM header `llvm/CodeGen/TargetSubtargetInfo.h` for TargetSubtargetInfo support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetSubtargetInfo.h`，用于 TargetSubtargetInfo 相关支持。
- **L19 EN**: Includes LLVM header `llvm/InitializePasses.h` for InitializePasses support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/InitializePasses.h`，用于 InitializePasses 相关支持。
- **L20 EN**: Includes LLVM header `llvm/Pass.h` for Pass support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/Pass.h`，用于 Pass 相关支持。

### Lines 21-40

````cpp
#include "llvm/PassRegistry.h"
#include "llvm/Support/GraphWriter.h"

using namespace llvm;

#define DEBUG_TYPE "dot-machine-cfg"

static cl::opt<std::string>
    MCFGFuncName("mcfg-func-name", cl::Hidden,
                 cl::desc("The name of a function (or its substring)"
                          " whose CFG is viewed/printed."));

static cl::opt<std::string> MCFGDotFilenamePrefix(
    "mcfg-dot-filename-prefix", cl::Hidden,
    cl::desc("The prefix used for the Machine CFG dot file names."));

static cl::opt<bool>
    CFGOnly("dot-mcfg-only", cl::init(false), cl::Hidden,
            cl::desc("Print only the CFG without blocks body"));

````
- **L21 EN**: Includes LLVM header `llvm/PassRegistry.h` for PassRegistry support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/PassRegistry.h`，用于 PassRegistry 相关支持。
- **L22 EN**: Includes LLVM header `llvm/Support/GraphWriter.h` for GraphWriter support.
  **L22 CN**: 引入 LLVM 头文件 `llvm/Support/GraphWriter.h`，用于 GraphWriter 相关支持。
- **L23 EN**: Separates nearby statements for readability.
  **L23 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L24 EN**: Imports namespace `llvm` into this translation unit.
  **L24 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L25 EN**: Separates nearby statements for readability.
  **L25 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L26 EN**: Defines the LLVM debug channel used by this file.
  **L26 CN**: 定义该文件使用的 LLVM 调试通道。
- **L27 EN**: Separates nearby statements for readability.
  **L27 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L28 EN**: Declares LLVM command-line option `command-line option`.
  **L28 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L29 EN**: Continues logic with `MCFGFuncName("mcfg-func-name", cl::Hidden,`.
  **L29 CN**: 继续处理逻辑：`MCFGFuncName("mcfg-func-name", cl::Hidden,`。
- **L30 EN**: Provides part of the signature for `desc`.
  **L30 CN**: 给出 `desc` 的一部分签名。
- **L31 EN**: Executes statement `" whose CFG is viewed/printed."));`.
  **L31 CN**: 执行语句 `" whose CFG is viewed/printed."));`。
- **L32 EN**: Separates nearby statements for readability.
  **L32 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L33 EN**: Declares LLVM command-line option `command-line option`.
  **L33 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L34 EN**: Continues logic with `"mcfg-dot-filename-prefix", cl::Hidden,`.
  **L34 CN**: 继续处理逻辑：`"mcfg-dot-filename-prefix", cl::Hidden,`。
- **L35 EN**: Declares function or method `desc`.
  **L35 CN**: 声明函数或方法 `desc`。
- **L36 EN**: Separates nearby statements for readability.
  **L36 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L37 EN**: Declares LLVM command-line option `command-line option`.
  **L37 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L38 EN**: Provides part of the signature for `CFGOnly`.
  **L38 CN**: 给出 `CFGOnly` 的一部分签名。
- **L39 EN**: Declares function or method `desc`.
  **L39 CN**: 声明函数或方法 `desc`。
- **L40 EN**: Separates nearby statements for readability.
  **L40 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 41-60

````cpp
static void writeMCFGToDotFile(MachineFunction &MF) {
  std::string Filename =
      (MCFGDotFilenamePrefix + "." + MF.getName() + ".dot").str();
  errs() << "Writing '" << Filename << "'...";

  std::error_code EC;
  raw_fd_ostream File(Filename, EC, sys::fs::OF_Text);

  DOTMachineFuncInfo MCFGInfo(&MF);

  if (!EC)
    WriteGraph(File, &MCFGInfo, CFGOnly);
  else
    errs() << "  error opening file for writing!";
  errs() << '\n';
}

namespace {

class MachineCFGPrinterLegacy : public MachineFunctionPass {
````
- **L41 EN**: Begins the definition of `writeMCFGToDotFile`.
  **L41 CN**: 开始定义 `writeMCFGToDotFile`。
- **L42 EN**: Continues logic with `std::string Filename =`.
  **L42 CN**: 继续处理逻辑：`std::string Filename =`。
- **L43 EN**: Executes statement `(MCFGDotFilenamePrefix + "." + MF.getName() + ".dot").str();`.
  **L43 CN**: 执行语句 `(MCFGDotFilenamePrefix + "." + MF.getName() + ".dot").str();`。
- **L44 EN**: Executes statement `errs() << "Writing '" << Filename << "'...";`.
  **L44 CN**: 执行语句 `errs() << "Writing '" << Filename << "'...";`。
- **L45 EN**: Separates nearby statements for readability.
  **L45 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L46 EN**: Executes statement `std::error_code EC;`.
  **L46 CN**: 执行语句 `std::error_code EC;`。
- **L47 EN**: Declares function or method `File`.
  **L47 CN**: 声明函数或方法 `File`。
- **L48 EN**: Separates nearby statements for readability.
  **L48 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L49 EN**: Declares function or method `MCFGInfo`.
  **L49 CN**: 声明函数或方法 `MCFGInfo`。
- **L50 EN**: Separates nearby statements for readability.
  **L50 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L51 EN**: Begins a conditional branch.
  **L51 CN**: 开始一个条件分支。
- **L52 EN**: Executes statement `WriteGraph(File, &MCFGInfo, CFGOnly);`.
  **L52 CN**: 执行语句 `WriteGraph(File, &MCFGInfo, CFGOnly);`。
- **L53 EN**: Handles the fallback branch.
  **L53 CN**: 处理兜底分支。
- **L54 EN**: Executes statement `errs() << " error opening file for writing!";`.
  **L54 CN**: 执行语句 `errs() << " error opening file for writing!";`。
- **L55 EN**: Executes statement `errs() << '\n';`.
  **L55 CN**: 执行语句 `errs() << '\n';`。
- **L56 EN**: Closes the current scope.
  **L56 CN**: 关闭当前作用域。
- **L57 EN**: Separates nearby statements for readability.
  **L57 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L58 EN**: Opens namespace ``.
  **L58 CN**: 打开命名空间 ``。
- **L59 EN**: Separates nearby statements for readability.
  **L59 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L60 EN**: Starts the declaration of class `MachineCFGPrinterLegacy`.
  **L60 CN**: 开始声明 class `MachineCFGPrinterLegacy`。

### Lines 61-80

````cpp
public:
  static char ID;

  MachineCFGPrinterLegacy();

  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

} // namespace

char MachineCFGPrinterLegacy::ID = 0;

char &llvm::MachineCFGPrinterID = MachineCFGPrinterLegacy::ID;

INITIALIZE_PASS(MachineCFGPrinterLegacy, DEBUG_TYPE, "Machine CFG Printer Pass",
````
- **L61 EN**: Continues logic with `public:`.
  **L61 CN**: 继续处理逻辑：`public:`。
- **L62 EN**: Executes statement `static char ID;`.
  **L62 CN**: 执行语句 `static char ID;`。
- **L63 EN**: Separates nearby statements for readability.
  **L63 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L64 EN**: Executes statement `MachineCFGPrinterLegacy();`.
  **L64 CN**: 执行语句 `MachineCFGPrinterLegacy();`。
- **L65 EN**: Separates nearby statements for readability.
  **L65 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L66 EN**: Declares function or method `runOnMachineFunction`.
  **L66 CN**: 声明函数或方法 `runOnMachineFunction`。
- **L67 EN**: Separates nearby statements for readability.
  **L67 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L68 EN**: Begins the definition of `getAnalysisUsage`.
  **L68 CN**: 开始定义 `getAnalysisUsage`。
- **L69 EN**: Executes statement `AU.setPreservesCFG();`.
  **L69 CN**: 执行语句 `AU.setPreservesCFG();`。
- **L70 EN**: Declares function or method `getAnalysisUsage`.
  **L70 CN**: 声明函数或方法 `getAnalysisUsage`。
- **L71 EN**: Closes the current scope.
  **L71 CN**: 关闭当前作用域。
- **L72 EN**: Closes the current scope.
  **L72 CN**: 关闭当前作用域。
- **L73 EN**: Separates nearby statements for readability.
  **L73 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L74 EN**: Continues logic with `} // namespace`.
  **L74 CN**: 继续处理逻辑：`} // namespace`。
- **L75 EN**: Separates nearby statements for readability.
  **L75 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L76 EN**: Assigns or initializes `char MachineCFGPrinterLegacy::ID`.
  **L76 CN**: 对 `char MachineCFGPrinterLegacy::ID` 进行赋值或初始化。
- **L77 EN**: Separates nearby statements for readability.
  **L77 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L78 EN**: Assigns or initializes `char &llvm::MachineCFGPrinterID`.
  **L78 CN**: 对 `char &llvm::MachineCFGPrinterID` 进行赋值或初始化。
- **L79 EN**: Separates nearby statements for readability.
  **L79 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L80 EN**: Continues logic with `INITIALIZE_PASS(MachineCFGPrinterLegacy, DEBUG_TYPE, "Machine CFG Printe…`.
  **L80 CN**: 继续处理逻辑：`INITIALIZE_PASS(MachineCFGPrinterLegacy, DEBUG_TYPE, "Machine CFG Printe…`。

### Lines 81-100

````cpp
                false, true)

/// Default construct and initialize the pass.
MachineCFGPrinterLegacy::MachineCFGPrinterLegacy() : MachineFunctionPass(ID) {}

bool MachineCFGPrinterLegacy::runOnMachineFunction(MachineFunction &MF) {
  if (!MCFGFuncName.empty() && !MF.getName().contains(MCFGFuncName))
    return false;
  errs() << "Writing Machine CFG for function ";
  errs().write_escaped(MF.getName()) << '\n';

  writeMCFGToDotFile(MF);
  return false;
}

PreservedAnalyses
MachineCFGPrinterPass::run(MachineFunction &MF,
                           MachineFunctionAnalysisManager &MFAM) {
  if (!MCFGFuncName.empty() && !MF.getName().contains(MCFGFuncName))
    return PreservedAnalyses::all();
````
- **L81 EN**: Continues logic with `false, true)`.
  **L81 CN**: 继续处理逻辑：`false, true)`。
- **L82 EN**: Separates nearby statements for readability.
  **L82 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L83 EN**: Comment documents: `Default construct and initialize the pass.`.
  **L83 CN**: 注释说明：`Default construct and initialize the pass.`。
- **L84 EN**: Provides part of the signature for `MachineCFGPrinterLegacy`.
  **L84 CN**: 给出 `MachineCFGPrinterLegacy` 的一部分签名。
- **L85 EN**: Separates nearby statements for readability.
  **L85 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L86 EN**: Begins the definition of `runOnMachineFunction`.
  **L86 CN**: 开始定义 `runOnMachineFunction`。
- **L87 EN**: Begins a conditional branch.
  **L87 CN**: 开始一个条件分支。
- **L88 EN**: Returns `false` to the caller.
  **L88 CN**: 向调用者返回 `false`。
- **L89 EN**: Executes statement `errs() << "Writing Machine CFG for function ";`.
  **L89 CN**: 执行语句 `errs() << "Writing Machine CFG for function ";`。
- **L90 EN**: Executes statement `errs().write_escaped(MF.getName()) << '\n';`.
  **L90 CN**: 执行语句 `errs().write_escaped(MF.getName()) << '\n';`。
- **L91 EN**: Separates nearby statements for readability.
  **L91 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L92 EN**: Executes statement `writeMCFGToDotFile(MF);`.
  **L92 CN**: 执行语句 `writeMCFGToDotFile(MF);`。
- **L93 EN**: Returns `false` to the caller.
  **L93 CN**: 向调用者返回 `false`。
- **L94 EN**: Closes the current scope.
  **L94 CN**: 关闭当前作用域。
- **L95 EN**: Separates nearby statements for readability.
  **L95 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L96 EN**: Continues logic with `PreservedAnalyses`.
  **L96 CN**: 继续处理逻辑：`PreservedAnalyses`。
- **L97 EN**: Provides part of the signature for `run`.
  **L97 CN**: 给出 `run` 的一部分签名。
- **L98 EN**: Starts block `MachineFunctionAnalysisManager &MFAM)`.
  **L98 CN**: 开始代码块 `MachineFunctionAnalysisManager &MFAM)`。
- **L99 EN**: Begins a conditional branch.
  **L99 CN**: 开始一个条件分支。
- **L100 EN**: Returns `PreservedAnalyses::all()` to the caller.
  **L100 CN**: 向调用者返回 `PreservedAnalyses::all()`。

### Lines 101-106

````cpp
  errs() << "Writing Machine CFG for function ";
  errs().write_escaped(MF.getName()) << '\n';

  writeMCFGToDotFile(MF);
  return PreservedAnalyses::all();
}
````
- **L101 EN**: Executes statement `errs() << "Writing Machine CFG for function ";`.
  **L101 CN**: 执行语句 `errs() << "Writing Machine CFG for function ";`。
- **L102 EN**: Executes statement `errs().write_escaped(MF.getName()) << '\n';`.
  **L102 CN**: 执行语句 `errs().write_escaped(MF.getName()) << '\n';`。
- **L103 EN**: Separates nearby statements for readability.
  **L103 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L104 EN**: Executes statement `writeMCFGToDotFile(MF);`.
  **L104 CN**: 执行语句 `writeMCFGToDotFile(MF);`。
- **L105 EN**: Returns `PreservedAnalyses::all()` to the caller.
  **L105 CN**: 向调用者返回 `PreservedAnalyses::all()`。
- **L106 EN**: Closes the current scope.
  **L106 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Machine function processing** / **MachineFunction 处理**
- **Machine basic block handling** / **MachineBasicBlock 处理**
- **Debug information handling** / **调试信息处理**
- **Target-specific hooks** / **目标相关钩子**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/MachineCFGPrinter.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachinePassManager.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/InitializePasses.h`, `llvm/Pass.h`, `llvm/PassRegistry.h`, `llvm/Support/GraphWriter.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
