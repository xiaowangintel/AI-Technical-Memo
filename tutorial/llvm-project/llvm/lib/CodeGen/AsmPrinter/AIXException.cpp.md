# AIXException.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/AsmPrinter/AIXException.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `AIX Exception Impl` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“AIX Exception Impl”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===-- CodeGen/AsmPrinter/AIXException.cpp - AIX Exception Impl ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains support for writing AIX exception info into asm files.
//
//===----------------------------------------------------------------------===//

#include "DwarfException.h"
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
#include "llvm/IR/Module.h"
#include "llvm/MC/MCSectionXCOFF.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/Target/TargetLoweringObjectFile.h"
````
- **L1 EN**: Comment documents: `===-- CodeGen/AsmPrinter/AIXException.cpp - AIX Exception Impl ---------…`.
  **L1 CN**: 注释说明：`===-- CodeGen/AsmPrinter/AIXException.cpp - AIX Exception Impl ---------…`。
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
- **L9 EN**: Comment documents: `This file contains support for writing AIX exception info into asm files…`.
  **L9 CN**: 注释说明：`This file contains support for writing AIX exception info into asm files…`。
- **L10 EN**: Continues the surrounding comment block.
  **L10 CN**: 延续周围的注释块。
- **L11 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L11 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L12 EN**: Separates nearby statements for readability.
  **L12 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L13 EN**: Includes system header `DwarfException.h`.
  **L13 CN**: 引入系统头文件 `DwarfException.h`。
- **L14 EN**: Includes LLVM header `llvm/CodeGen/AsmPrinter.h` for AsmPrinter support.
  **L14 CN**: 引入 LLVM 头文件 `llvm/CodeGen/AsmPrinter.h`，用于 AsmPrinter 相关支持。
- **L15 EN**: Includes LLVM header `llvm/CodeGen/MachineModuleInfo.h` for MachineModuleInfo support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineModuleInfo.h`，用于 MachineModuleInfo 相关支持。
- **L16 EN**: Includes LLVM header `llvm/CodeGen/TargetLoweringObjectFileImpl.h` for TargetLoweringObjectFileImpl support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetLoweringObjectFileImpl.h`，用于 TargetLoweringObjectFileImpl 相关支持。
- **L17 EN**: Includes LLVM header `llvm/IR/Module.h` for Module support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/IR/Module.h`，用于 Module 相关支持。
- **L18 EN**: Includes LLVM header `llvm/MC/MCSectionXCOFF.h` for MCSectionXCOFF support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/MC/MCSectionXCOFF.h`，用于 MCSectionXCOFF 相关支持。
- **L19 EN**: Includes LLVM header `llvm/MC/MCStreamer.h` for MCStreamer support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/MC/MCStreamer.h`，用于 MCStreamer 相关支持。
- **L20 EN**: Includes LLVM header `llvm/Target/TargetLoweringObjectFile.h` for TargetLoweringObjectFile support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/Target/TargetLoweringObjectFile.h`，用于 TargetLoweringObjectFile 相关支持。

### Lines 21-40

````cpp
#include "llvm/Target/TargetMachine.h"

using namespace llvm;

AIXException::AIXException(AsmPrinter *A) : EHStreamer(A) {}

void AIXException::emitExceptionInfoTable(const MCSymbol *LSDA,
                                          const MCSymbol *PerSym) {
  // Generate EH Info Table.
  // The EH Info Table, aka, 'compat unwind section' on AIX, have the following
  // format: struct eh_info_t {
  //   unsigned version;           /* EH info verion 0 */
  // #if defined(__64BIT__)
  //   char _pad[4];               /* padding */
  // #endif
  //   unsigned long lsda;         /* Pointer to LSDA */
  //   unsigned long personality;  /* Pointer to the personality routine */
  //   }

  auto *EHInfo = static_cast<MCSectionXCOFF *>(
````
- **L21 EN**: Includes LLVM header `llvm/Target/TargetMachine.h` for TargetMachine support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/Target/TargetMachine.h`，用于 TargetMachine 相关支持。
- **L22 EN**: Separates nearby statements for readability.
  **L22 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L23 EN**: Imports namespace `llvm` into this translation unit.
  **L23 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L24 EN**: Separates nearby statements for readability.
  **L24 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L25 EN**: Provides part of the signature for `AIXException`.
  **L25 CN**: 给出 `AIXException` 的一部分签名。
- **L26 EN**: Separates nearby statements for readability.
  **L26 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L27 EN**: Provides part of the signature for `emitExceptionInfoTable`.
  **L27 CN**: 给出 `emitExceptionInfoTable` 的一部分签名。
- **L28 EN**: Starts block `const MCSymbol *PerSym)`.
  **L28 CN**: 开始代码块 `const MCSymbol *PerSym)`。
- **L29 EN**: Comment documents: `Generate EH Info Table.`.
  **L29 CN**: 注释说明：`Generate EH Info Table.`。
- **L30 EN**: Comment documents: `The EH Info Table, aka, 'compat unwind section' on AIX, have the followi…`.
  **L30 CN**: 注释说明：`The EH Info Table, aka, 'compat unwind section' on AIX, have the followi…`。
- **L31 EN**: Comment documents: `format: struct eh_info_t {`.
  **L31 CN**: 注释说明：`format: struct eh_info_t {`。
- **L32 EN**: Comment documents: `unsigned version; /* EH info verion 0`.
  **L32 CN**: 注释说明：`unsigned version; /* EH info verion 0`。
- **L33 EN**: Comment documents: `#if defined(__64BIT__)`.
  **L33 CN**: 注释说明：`#if defined(__64BIT__)`。
- **L34 EN**: Comment documents: `char _pad[4]; /* padding`.
  **L34 CN**: 注释说明：`char _pad[4]; /* padding`。
- **L35 EN**: Comment documents: `#endif`.
  **L35 CN**: 注释说明：`#endif`。
- **L36 EN**: Comment documents: `unsigned long lsda; /* Pointer to LSDA`.
  **L36 CN**: 注释说明：`unsigned long lsda; /* Pointer to LSDA`。
- **L37 EN**: Comment documents: `unsigned long personality; /* Pointer to the personality routine`.
  **L37 CN**: 注释说明：`unsigned long personality; /* Pointer to the personality routine`。
- **L38 EN**: Comment documents: `}`.
  **L38 CN**: 注释说明：`}`。
- **L39 EN**: Separates nearby statements for readability.
  **L39 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L40 EN**: Continues logic with `auto *EHInfo = static_cast<MCSectionXCOFF *>(`.
  **L40 CN**: 继续处理逻辑：`auto *EHInfo = static_cast<MCSectionXCOFF *>(`。

### Lines 41-60

````cpp
      Asm->getObjFileLowering().getCompactUnwindSection());
  if (Asm->TM.getFunctionSections()) {
    // If option -ffunction-sections is on, append the function name to the
    // name of EH Info Table csect so that each function has its own EH Info
    // Table csect. This helps the linker to garbage-collect EH info of unused
    // functions.
    SmallString<128> NameStr = EHInfo->getName();
    raw_svector_ostream(NameStr) << '.' << Asm->MF->getFunction().getName();
    EHInfo = Asm->OutContext.getXCOFFSection(NameStr, EHInfo->getKind(),
                                             EHInfo->getCsectProp());
  }
  Asm->OutStreamer->switchSection(EHInfo);
  MCSymbol *EHInfoLabel =
      TargetLoweringObjectFileXCOFF::getEHInfoTableSymbol(Asm->MF);
  Asm->OutStreamer->emitLabel(EHInfoLabel);

  // Version number.
  Asm->emitInt32(0);

  const DataLayout &DL = MMI->getModule()->getDataLayout();
````
- **L41 EN**: Executes statement `Asm->getObjFileLowering().getCompactUnwindSection());`.
  **L41 CN**: 执行语句 `Asm->getObjFileLowering().getCompactUnwindSection());`。
- **L42 EN**: Begins a conditional branch.
  **L42 CN**: 开始一个条件分支。
- **L43 EN**: Comment documents: `If option -ffunction-sections is on, append the function name to the`.
  **L43 CN**: 注释说明：`If option -ffunction-sections is on, append the function name to the`。
- **L44 EN**: Comment documents: `name of EH Info Table csect so that each function has its own EH Info`.
  **L44 CN**: 注释说明：`name of EH Info Table csect so that each function has its own EH Info`。
- **L45 EN**: Comment documents: `Table csect. This helps the linker to garbage-collect EH info of unused`.
  **L45 CN**: 注释说明：`Table csect. This helps the linker to garbage-collect EH info of unused`。
- **L46 EN**: Comment documents: `functions.`.
  **L46 CN**: 注释说明：`functions.`。
- **L47 EN**: Assigns or initializes `SmallString<128> NameStr`.
  **L47 CN**: 对 `SmallString<128> NameStr` 进行赋值或初始化。
- **L48 EN**: Executes statement `raw_svector_ostream(NameStr) << '.' << Asm->MF->getFunction().getName();`.
  **L48 CN**: 执行语句 `raw_svector_ostream(NameStr) << '.' << Asm->MF->getFunction().getName();`。
- **L49 EN**: Continues logic with `EHInfo = Asm->OutContext.getXCOFFSection(NameStr, EHInfo->getKind(),`.
  **L49 CN**: 继续处理逻辑：`EHInfo = Asm->OutContext.getXCOFFSection(NameStr, EHInfo->getKind(),`。
- **L50 EN**: Executes statement `EHInfo->getCsectProp());`.
  **L50 CN**: 执行语句 `EHInfo->getCsectProp());`。
- **L51 EN**: Closes the current scope.
  **L51 CN**: 关闭当前作用域。
- **L52 EN**: Executes statement `Asm->OutStreamer->switchSection(EHInfo);`.
  **L52 CN**: 执行语句 `Asm->OutStreamer->switchSection(EHInfo);`。
- **L53 EN**: Continues logic with `MCSymbol *EHInfoLabel =`.
  **L53 CN**: 继续处理逻辑：`MCSymbol *EHInfoLabel =`。
- **L54 EN**: Declares function or method `getEHInfoTableSymbol`.
  **L54 CN**: 声明函数或方法 `getEHInfoTableSymbol`。
- **L55 EN**: Executes statement `Asm->OutStreamer->emitLabel(EHInfoLabel);`.
  **L55 CN**: 执行语句 `Asm->OutStreamer->emitLabel(EHInfoLabel);`。
- **L56 EN**: Separates nearby statements for readability.
  **L56 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L57 EN**: Comment documents: `Version number.`.
  **L57 CN**: 注释说明：`Version number.`。
- **L58 EN**: Executes statement `Asm->emitInt32(0);`.
  **L58 CN**: 执行语句 `Asm->emitInt32(0);`。
- **L59 EN**: Separates nearby statements for readability.
  **L59 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L60 EN**: Assigns or initializes `const DataLayout &DL`.
  **L60 CN**: 对 `const DataLayout &DL` 进行赋值或初始化。

### Lines 61-80

````cpp
  const unsigned PointerSize = DL.getPointerSize();

  // Add necessary paddings in 64 bit mode.
  Asm->OutStreamer->emitValueToAlignment(Align(PointerSize));

  // LSDA location.
  Asm->OutStreamer->emitValue(MCSymbolRefExpr::create(LSDA, Asm->OutContext),
                              PointerSize);

  // Personality routine.
  Asm->OutStreamer->emitValue(MCSymbolRefExpr::create(PerSym, Asm->OutContext),
                              PointerSize);
}

void AIXException::endFunction(const MachineFunction *MF) {
  // There is no easy way to access register information in `AIXException`
  // class. when ShouldEmitEHBlock is false and VRs are saved, A dumy eh info
  // table are emitted in PPCAIXAsmPrinter::emitFunctionBodyEnd.
  if (!TargetLoweringObjectFileXCOFF::ShouldEmitEHBlock(MF))
    return;
````
- **L61 EN**: Assigns or initializes `const unsigned PointerSize`.
  **L61 CN**: 对 `const unsigned PointerSize` 进行赋值或初始化。
- **L62 EN**: Separates nearby statements for readability.
  **L62 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L63 EN**: Comment documents: `Add necessary paddings in 64 bit mode.`.
  **L63 CN**: 注释说明：`Add necessary paddings in 64 bit mode.`。
- **L64 EN**: Executes statement `Asm->OutStreamer->emitValueToAlignment(Align(PointerSize));`.
  **L64 CN**: 执行语句 `Asm->OutStreamer->emitValueToAlignment(Align(PointerSize));`。
- **L65 EN**: Separates nearby statements for readability.
  **L65 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L66 EN**: Comment documents: `LSDA location.`.
  **L66 CN**: 注释说明：`LSDA location.`。
- **L67 EN**: Provides part of the signature for `emitValue`.
  **L67 CN**: 给出 `emitValue` 的一部分签名。
- **L68 EN**: Executes statement `PointerSize);`.
  **L68 CN**: 执行语句 `PointerSize);`。
- **L69 EN**: Separates nearby statements for readability.
  **L69 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L70 EN**: Comment documents: `Personality routine.`.
  **L70 CN**: 注释说明：`Personality routine.`。
- **L71 EN**: Provides part of the signature for `emitValue`.
  **L71 CN**: 给出 `emitValue` 的一部分签名。
- **L72 EN**: Executes statement `PointerSize);`.
  **L72 CN**: 执行语句 `PointerSize);`。
- **L73 EN**: Closes the current scope.
  **L73 CN**: 关闭当前作用域。
- **L74 EN**: Separates nearby statements for readability.
  **L74 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L75 EN**: Begins the definition of `endFunction`.
  **L75 CN**: 开始定义 `endFunction`。
- **L76 EN**: Comment documents: `There is no easy way to access register information in 'AIXException'`.
  **L76 CN**: 注释说明：`There is no easy way to access register information in 'AIXException'`。
- **L77 EN**: Comment documents: `class. when ShouldEmitEHBlock is false and VRs are saved, A dumy eh info`.
  **L77 CN**: 注释说明：`class. when ShouldEmitEHBlock is false and VRs are saved, A dumy eh info`。
- **L78 EN**: Comment documents: `table are emitted in PPCAIXAsmPrinter::emitFunctionBodyEnd.`.
  **L78 CN**: 注释说明：`table are emitted in PPCAIXAsmPrinter::emitFunctionBodyEnd.`。
- **L79 EN**: Begins a conditional branch.
  **L79 CN**: 开始一个条件分支。
- **L80 EN**: Returns control to the caller.
  **L80 CN**: 将控制流返回给调用者。

### Lines 81-92

````cpp

  const MCSymbol *LSDALabel = emitExceptionTable();

  const Function &F = MF->getFunction();
  assert(F.hasPersonalityFn() &&
         "Landingpads are presented, but no personality routine is found.");
  const auto *Per =
      cast<GlobalValue>(F.getPersonalityFn()->stripPointerCasts());
  const MCSymbol *PerSym = Asm->TM.getSymbol(Per);

  emitExceptionInfoTable(LSDALabel, PerSym);
}
````
- **L81 EN**: Separates nearby statements for readability.
  **L81 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L82 EN**: Assigns or initializes `const MCSymbol *LSDALabel`.
  **L82 CN**: 对 `const MCSymbol *LSDALabel` 进行赋值或初始化。
- **L83 EN**: Separates nearby statements for readability.
  **L83 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L84 EN**: Assigns or initializes `const Function &F`.
  **L84 CN**: 对 `const Function &F` 进行赋值或初始化。
- **L85 EN**: Checks an invariant in debug builds.
  **L85 CN**: 在调试构建中检查一个不变量。
- **L86 EN**: Executes statement `"Landingpads are presented, but no personality routine is found.");`.
  **L86 CN**: 执行语句 `"Landingpads are presented, but no personality routine is found.");`。
- **L87 EN**: Continues logic with `const auto *Per =`.
  **L87 CN**: 继续处理逻辑：`const auto *Per =`。
- **L88 EN**: Executes statement `cast<GlobalValue>(F.getPersonalityFn()->stripPointerCasts());`.
  **L88 CN**: 执行语句 `cast<GlobalValue>(F.getPersonalityFn()->stripPointerCasts());`。
- **L89 EN**: Assigns or initializes `const MCSymbol *PerSym`.
  **L89 CN**: 对 `const MCSymbol *PerSym` 进行赋值或初始化。
- **L90 EN**: Separates nearby statements for readability.
  **L90 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L91 EN**: Executes statement `emitExceptionInfoTable(LSDALabel, PerSym);`.
  **L91 CN**: 执行语句 `emitExceptionInfoTable(LSDALabel, PerSym);`。
- **L92 EN**: Closes the current scope.
  **L92 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Register management** / **寄存器管理**
- **Machine function processing** / **MachineFunction 处理**
- **Target-specific hooks** / **目标相关钩子**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineModuleInfo.h`, `llvm/CodeGen/TargetLoweringObjectFileImpl.h`, `llvm/IR/Module.h`, `llvm/MC/MCSectionXCOFF.h`, `llvm/MC/MCStreamer.h`, `llvm/Target/TargetLoweringObjectFile.h`, `llvm/Target/TargetMachine.h`
- **System headers / 系统头文件**: `DwarfException.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
