# WasmException.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/AsmPrinter/WasmException.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `Wasm Exception Impl` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“Wasm Exception Impl”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===-- CodeGen/AsmPrinter/WasmException.cpp - Wasm Exception Impl --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains support for writing WebAssembly exception info into asm
// files.
//
//===----------------------------------------------------------------------===//

#include "WasmException.h"
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/IR/Mangler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCStreamer.h"
using namespace llvm;
````
- **L1 EN**: Comment documents: `===-- CodeGen/AsmPrinter/WasmException.cpp - Wasm Exception Impl -------…`.
  **L1 CN**: 注释说明：`===-- CodeGen/AsmPrinter/WasmException.cpp - Wasm Exception Impl -------…`。
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
- **L9 EN**: Comment documents: `This file contains support for writing WebAssembly exception info into a…`.
  **L9 CN**: 注释说明：`This file contains support for writing WebAssembly exception info into a…`。
- **L10 EN**: Comment documents: `files.`.
  **L10 CN**: 注释说明：`files.`。
- **L11 EN**: Continues the surrounding comment block.
  **L11 CN**: 延续周围的注释块。
- **L12 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L12 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L13 EN**: Separates nearby statements for readability.
  **L13 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L14 EN**: Includes system header `WasmException.h`.
  **L14 CN**: 引入系统头文件 `WasmException.h`。
- **L15 EN**: Includes LLVM header `llvm/CodeGen/AsmPrinter.h` for AsmPrinter support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/CodeGen/AsmPrinter.h`，用于 AsmPrinter 相关支持。
- **L16 EN**: Includes LLVM header `llvm/CodeGen/MachineFunction.h` for MachineFunction support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFunction.h`，用于 MachineFunction 相关支持。
- **L17 EN**: Includes LLVM header `llvm/IR/Mangler.h` for Mangler support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/IR/Mangler.h`，用于 Mangler 相关支持。
- **L18 EN**: Includes LLVM header `llvm/MC/MCContext.h` for MCContext support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/MC/MCContext.h`，用于 MCContext 相关支持。
- **L19 EN**: Includes LLVM header `llvm/MC/MCStreamer.h` for MCStreamer support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/MC/MCStreamer.h`，用于 MCStreamer 相关支持。
- **L20 EN**: Imports namespace `llvm` into this translation unit.
  **L20 CN**: 将命名空间 `llvm` 引入当前编译单元。

### Lines 21-40

````cpp

void WasmException::endFunction(const MachineFunction *MF) {
  bool ShouldEmitExceptionTable = false;
  for (const LandingPadInfo &Info : MF->getLandingPads()) {
    if (MF->hasWasmLandingPadIndex(Info.LandingPadBlock)) {
      ShouldEmitExceptionTable = true;
      break;
    }
  }
  if (!ShouldEmitExceptionTable)
    return;
  MCSymbol *LSDALabel = emitExceptionTable();
  assert(LSDALabel && ".GCC_exception_table has not been emitted!");

  // Wasm requires every data section symbol to have a .size set. So we emit an
  // end marker and set the size as the difference between the start end the end
  // marker.
  MCSymbol *LSDAEndLabel = Asm->createTempSymbol("GCC_except_table_end");
  Asm->OutStreamer->emitLabel(LSDAEndLabel);
  MCContext &OutContext = Asm->OutStreamer->getContext();
````
- **L21 EN**: Separates nearby statements for readability.
  **L21 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L22 EN**: Begins the definition of `endFunction`.
  **L22 CN**: 开始定义 `endFunction`。
- **L23 EN**: Assigns or initializes `bool ShouldEmitExceptionTable`.
  **L23 CN**: 对 `bool ShouldEmitExceptionTable` 进行赋值或初始化。
- **L24 EN**: Starts a loop over a sequence or range.
  **L24 CN**: 开始遍历序列或范围的循环。
- **L25 EN**: Begins a conditional branch.
  **L25 CN**: 开始一个条件分支。
- **L26 EN**: Assigns or initializes `ShouldEmitExceptionTable`.
  **L26 CN**: 对 `ShouldEmitExceptionTable` 进行赋值或初始化。
- **L27 EN**: Breaks out of the current control-flow construct.
  **L27 CN**: 跳出当前控制流结构。
- **L28 EN**: Closes the current scope.
  **L28 CN**: 关闭当前作用域。
- **L29 EN**: Closes the current scope.
  **L29 CN**: 关闭当前作用域。
- **L30 EN**: Begins a conditional branch.
  **L30 CN**: 开始一个条件分支。
- **L31 EN**: Returns control to the caller.
  **L31 CN**: 将控制流返回给调用者。
- **L32 EN**: Assigns or initializes `MCSymbol *LSDALabel`.
  **L32 CN**: 对 `MCSymbol *LSDALabel` 进行赋值或初始化。
- **L33 EN**: Checks an invariant in debug builds.
  **L33 CN**: 在调试构建中检查一个不变量。
- **L34 EN**: Separates nearby statements for readability.
  **L34 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L35 EN**: Comment documents: `Wasm requires every data section symbol to have a .size set. So we emit …`.
  **L35 CN**: 注释说明：`Wasm requires every data section symbol to have a .size set. So we emit …`。
- **L36 EN**: Comment documents: `end marker and set the size as the difference between the start end the …`.
  **L36 CN**: 注释说明：`end marker and set the size as the difference between the start end the …`。
- **L37 EN**: Comment documents: `marker.`.
  **L37 CN**: 注释说明：`marker.`。
- **L38 EN**: Assigns or initializes `MCSymbol *LSDAEndLabel`.
  **L38 CN**: 对 `MCSymbol *LSDAEndLabel` 进行赋值或初始化。
- **L39 EN**: Executes statement `Asm->OutStreamer->emitLabel(LSDAEndLabel);`.
  **L39 CN**: 执行语句 `Asm->OutStreamer->emitLabel(LSDAEndLabel);`。
- **L40 EN**: Assigns or initializes `MCContext &OutContext`.
  **L40 CN**: 对 `MCContext &OutContext` 进行赋值或初始化。

### Lines 41-60

````cpp
  const MCExpr *SizeExp = MCBinaryExpr::createSub(
      MCSymbolRefExpr::create(LSDAEndLabel, OutContext),
      MCSymbolRefExpr::create(LSDALabel, OutContext), OutContext);
  Asm->OutStreamer->emitELFSize(LSDALabel, SizeExp);
}

// Compute the call-site table for wasm EH. Even though we use the same function
// name to share the common routines, a call site entry in the table corresponds
// to not a call site for possibly-throwing functions but a landing pad. In wasm
// EH the VM is responsible for stack unwinding. After an exception occurs and
// the stack is unwound, the control flow is transferred to wasm 'catch'
// instruction by the VM, after which the personality function is called from
// the compiler-generated code. Refer to WasmEHPrepare pass for more
// information.
void WasmException::computeCallSiteTable(
    SmallVectorImpl<CallSiteEntry> &CallSites,
    SmallVectorImpl<CallSiteRange> &CallSiteRanges,
    const SmallVectorImpl<const LandingPadInfo *> &LandingPads,
    const SmallVectorImpl<unsigned> &FirstActions) {
  MachineFunction &MF = *Asm->MF;
````
- **L41 EN**: Provides part of the signature for `createSub`.
  **L41 CN**: 给出 `createSub` 的一部分签名。
- **L42 EN**: Provides part of the signature for `create`.
  **L42 CN**: 给出 `create` 的一部分签名。
- **L43 EN**: Declares function or method `create`.
  **L43 CN**: 声明函数或方法 `create`。
- **L44 EN**: Executes statement `Asm->OutStreamer->emitELFSize(LSDALabel, SizeExp);`.
  **L44 CN**: 执行语句 `Asm->OutStreamer->emitELFSize(LSDALabel, SizeExp);`。
- **L45 EN**: Closes the current scope.
  **L45 CN**: 关闭当前作用域。
- **L46 EN**: Separates nearby statements for readability.
  **L46 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L47 EN**: Comment documents: `Compute the call-site table for wasm EH. Even though we use the same fun…`.
  **L47 CN**: 注释说明：`Compute the call-site table for wasm EH. Even though we use the same fun…`。
- **L48 EN**: Comment documents: `name to share the common routines, a call site entry in the table corres…`.
  **L48 CN**: 注释说明：`name to share the common routines, a call site entry in the table corres…`。
- **L49 EN**: Comment documents: `to not a call site for possibly-throwing functions but a landing pad. In…`.
  **L49 CN**: 注释说明：`to not a call site for possibly-throwing functions but a landing pad. In…`。
- **L50 EN**: Comment documents: `EH the VM is responsible for stack unwinding. After an exception occurs …`.
  **L50 CN**: 注释说明：`EH the VM is responsible for stack unwinding. After an exception occurs …`。
- **L51 EN**: Comment documents: `the stack is unwound, the control flow is transferred to wasm 'catch'`.
  **L51 CN**: 注释说明：`the stack is unwound, the control flow is transferred to wasm 'catch'`。
- **L52 EN**: Comment documents: `instruction by the VM, after which the personality function is called fr…`.
  **L52 CN**: 注释说明：`instruction by the VM, after which the personality function is called fr…`。
- **L53 EN**: Comment documents: `the compiler-generated code. Refer to WasmEHPrepare pass for more`.
  **L53 CN**: 注释说明：`the compiler-generated code. Refer to WasmEHPrepare pass for more`。
- **L54 EN**: Comment documents: `information.`.
  **L54 CN**: 注释说明：`information.`。
- **L55 EN**: Provides part of the signature for `computeCallSiteTable`.
  **L55 CN**: 给出 `computeCallSiteTable` 的一部分签名。
- **L56 EN**: Continues logic with `SmallVectorImpl<CallSiteEntry> &CallSites,`.
  **L56 CN**: 继续处理逻辑：`SmallVectorImpl<CallSiteEntry> &CallSites,`。
- **L57 EN**: Continues logic with `SmallVectorImpl<CallSiteRange> &CallSiteRanges,`.
  **L57 CN**: 继续处理逻辑：`SmallVectorImpl<CallSiteRange> &CallSiteRanges,`。
- **L58 EN**: Continues logic with `const SmallVectorImpl<const LandingPadInfo *> &LandingPads,`.
  **L58 CN**: 继续处理逻辑：`const SmallVectorImpl<const LandingPadInfo *> &LandingPads,`。
- **L59 EN**: Starts block `const SmallVectorImpl<unsigned> &FirstActions)`.
  **L59 CN**: 开始代码块 `const SmallVectorImpl<unsigned> &FirstActions)`。
- **L60 EN**: Assigns or initializes `MachineFunction &MF`.
  **L60 CN**: 对 `MachineFunction &MF` 进行赋值或初始化。

### Lines 61-75

````cpp
  for (unsigned I = 0, N = LandingPads.size(); I < N; ++I) {
    const LandingPadInfo *Info = LandingPads[I];
    MachineBasicBlock *LPad = Info->LandingPadBlock;
    // We don't emit LSDA for single catch (...).
    if (!MF.hasWasmLandingPadIndex(LPad))
      continue;
    // Wasm EH must maintain the EH pads in the order assigned to them by the
    // WasmEHPrepare pass.
    unsigned LPadIndex = MF.getWasmLandingPadIndex(LPad);
    CallSiteEntry Site = {nullptr, nullptr, Info, FirstActions[I]};
    if (CallSites.size() < LPadIndex + 1)
      CallSites.resize(LPadIndex + 1);
    CallSites[LPadIndex] = Site;
  }
}
````
- **L61 EN**: Starts a loop over a sequence or range.
  **L61 CN**: 开始遍历序列或范围的循环。
- **L62 EN**: Assigns or initializes `const LandingPadInfo *Info`.
  **L62 CN**: 对 `const LandingPadInfo *Info` 进行赋值或初始化。
- **L63 EN**: Assigns or initializes `MachineBasicBlock *LPad`.
  **L63 CN**: 对 `MachineBasicBlock *LPad` 进行赋值或初始化。
- **L64 EN**: Comment documents: `We don't emit LSDA for single catch (...).`.
  **L64 CN**: 注释说明：`We don't emit LSDA for single catch (...).`。
- **L65 EN**: Begins a conditional branch.
  **L65 CN**: 开始一个条件分支。
- **L66 EN**: Skips to the next loop iteration.
  **L66 CN**: 跳到下一次循环迭代。
- **L67 EN**: Comment documents: `Wasm EH must maintain the EH pads in the order assigned to them by the`.
  **L67 CN**: 注释说明：`Wasm EH must maintain the EH pads in the order assigned to them by the`。
- **L68 EN**: Comment documents: `WasmEHPrepare pass.`.
  **L68 CN**: 注释说明：`WasmEHPrepare pass.`。
- **L69 EN**: Assigns or initializes `unsigned LPadIndex`.
  **L69 CN**: 对 `unsigned LPadIndex` 进行赋值或初始化。
- **L70 EN**: Assigns or initializes `CallSiteEntry Site`.
  **L70 CN**: 对 `CallSiteEntry Site` 进行赋值或初始化。
- **L71 EN**: Begins a conditional branch.
  **L71 CN**: 开始一个条件分支。
- **L72 EN**: Executes statement `CallSites.resize(LPadIndex + 1);`.
  **L72 CN**: 执行语句 `CallSites.resize(LPadIndex + 1);`。
- **L73 EN**: Assigns or initializes `CallSites[LPadIndex]`.
  **L73 CN**: 对 `CallSites[LPadIndex]` 进行赋值或初始化。
- **L74 EN**: Closes the current scope.
  **L74 CN**: 关闭当前作用域。
- **L75 EN**: Closes the current scope.
  **L75 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Machine function processing** / **MachineFunction 处理**
- **Machine basic block handling** / **MachineBasicBlock 处理**
- **Stack frame management** / **栈帧管理**
- **Garbage-collection support** / **垃圾回收支持**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/IR/Mangler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCStreamer.h`
- **System headers / 系统头文件**: `WasmException.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
