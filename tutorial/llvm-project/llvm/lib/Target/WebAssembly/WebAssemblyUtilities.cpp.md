# WebAssemblyUtilities.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyUtilities.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements several utility functions for WebAssembly.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyUtilities.cpp`，主要负责 WebAssembly 后端的后端共享工具函数。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyUtilities.cpp - WebAssembly Utility Functions ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements several utility functions for WebAssembly.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-18

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyUtilities.h"
#include "WebAssemblyMachineFunctionInfo.h"
#include "WebAssemblyTargetMachine.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/IR/Function.h"
```
- **EN**: Pulls in direct dependencies required by this shared backend utility helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端共享工具函数所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 19-56

```cpp
#include "llvm/MC/MCContext.h"
using namespace llvm;

// Function names in libc++abi and libunwind
const char *const WebAssembly::CxaBeginCatchFn = "__cxa_begin_catch";
const char *const WebAssembly::CxaRethrowFn = "__cxa_rethrow";
const char *const WebAssembly::StdTerminateFn = "_ZSt9terminatev";
const char *const WebAssembly::PersonalityWrapperFn =
    "_Unwind_Wasm_CallPersonality";

/// Test whether MI is a child of some other node in an expression tree.
bool WebAssembly::isChild(const MachineInstr &MI,
                          const WebAssemblyFunctionInfo &MFI) {
  if (MI.getNumOperands() == 0)
    return false;
  const MachineOperand &MO = MI.getOperand(0);
  if (!MO.isReg() || MO.isImplicit() || !MO.isDef())
    return false;
  Register Reg = MO.getReg();
  return Reg.isVirtual() && MFI.isVRegStackified(Reg);
}

bool WebAssembly::mayThrow(const MachineInstr &MI) {
  switch (MI.getOpcode()) {
  case WebAssembly::THROW:
  case WebAssembly::THROW_S:
  case WebAssembly::THROW_REF:
  case WebAssembly::THROW_REF_S:
  case WebAssembly::RETHROW:
  case WebAssembly::RETHROW_S:
    return true;
  }
  if (isCallIndirect(MI.getOpcode()))
    return true;
  if (!MI.isCall())
    return false;

  const MachineOperand &MO = getCalleeOp(MI);
```
- **EN**: Pulls in direct dependencies required by this shared backend utility helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端共享工具函数所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 57-94

```cpp
  assert(MO.isGlobal() || MO.isSymbol());

  if (MO.isSymbol()) {
    // Some intrinsics are lowered to calls to external symbols, which are then
    // lowered to calls to library functions. Most of libcalls don't throw, but
    // we only list some of them here now.
    // TODO Consider adding 'nounwind' info in TargetLowering::CallLoweringInfo
    // instead for more accurate info.
    const char *Name = MO.getSymbolName();
    if (strcmp(Name, "memcpy") == 0 || strcmp(Name, "memmove") == 0 ||
        strcmp(Name, "memset") == 0)
      return false;
    return true;
  }

  const auto *F = dyn_cast<Function>(MO.getGlobal());
  if (!F)
    return true;
  if (F->doesNotThrow())
    return false;
  // These functions never throw
  if (F->getName() == CxaBeginCatchFn || F->getName() == PersonalityWrapperFn ||
      F->getName() == StdTerminateFn)
    return false;

  // TODO Can we exclude call instructions that are marked as 'nounwind' in the
  // original LLVm IR? (Even when the callee may throw)
  return true;
}

const MachineOperand &WebAssembly::getCalleeOp(const MachineInstr &MI) {
  switch (MI.getOpcode()) {
  case WebAssembly::CALL:
  case WebAssembly::CALL_S:
  case WebAssembly::RET_CALL:
  case WebAssembly::RET_CALL_S:
    return MI.getOperand(MI.getNumExplicitDefs());
  case WebAssembly::CALL_INDIRECT:
```
- **EN**: Implements helper routine(s) `isGlobal`, `isSymbol`, `getSymbolName` for this portion of the WebAssembly backend shared backend utility helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端共享工具函数所需的辅助例程 `isGlobal`, `isSymbol`, `getSymbolName`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 95-132

```cpp
  case WebAssembly::CALL_INDIRECT_S:
  case WebAssembly::RET_CALL_INDIRECT:
  case WebAssembly::RET_CALL_INDIRECT_S:
    return MI.getOperand(MI.getNumExplicitOperands() - 1);
  default:
    llvm_unreachable("Not a call instruction");
  }
}

MCSymbolWasm *WebAssembly::getOrCreateFunctionTableSymbol(
    MCContext &Ctx, const WebAssemblySubtarget *Subtarget) {
  StringRef Name = "__indirect_function_table";
  auto *Sym = static_cast<MCSymbolWasm *>(Ctx.lookupSymbol(Name));
  if (Sym) {
    if (!Sym->isFunctionTable())
      Ctx.reportError(SMLoc(), "symbol is not a wasm funcref table");
  } else {
    bool is64 = Subtarget && Subtarget->getTargetTriple().isArch64Bit();
    Sym = static_cast<MCSymbolWasm *>(Ctx.getOrCreateSymbol(Name));
    Sym->setFunctionTable(is64);
    // The default function table is synthesized by the linker.
  }
  // MVP object files can't have symtab entries for tables.
  if (!(Subtarget && Subtarget->hasCallIndirectOverlong()))
    Sym->setOmitFromLinkingSection();
  return Sym;
}

MCSymbolWasm *WebAssembly::getOrCreateFuncrefCallTableSymbol(
    MCContext &Ctx, const WebAssemblySubtarget *Subtarget) {
  StringRef Name = "__funcref_call_table";
  auto *Sym = static_cast<MCSymbolWasm *>(Ctx.lookupSymbol(Name));
  if (Sym) {
    if (!Sym->isFunctionTable())
      Ctx.reportError(SMLoc(), "symbol is not a wasm funcref table");
  } else {
    Sym = static_cast<MCSymbolWasm *>(Ctx.getOrCreateSymbol(Name));
```
- **EN**: Implements helper routine(s) `getOperand`, `getNumExplicitOperands`, `llvm_unreachable` for this portion of the WebAssembly backend shared backend utility helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端共享工具函数所需的辅助例程 `getOperand`, `getNumExplicitOperands`, `llvm_unreachable`。 子目标特性裁剪会影响这里的行为。

### Lines 133-170

```cpp
    // Setting Weak ensure only one table is left after linking when multiple
    // modules define the table.
    Sym->setWeak(true);

    wasm::WasmLimits Limits = {0, 1, 1, 0};
    wasm::WasmTableType TableType = {wasm::ValType::FUNCREF, Limits};
    Sym->setType(wasm::WASM_SYMBOL_TYPE_TABLE);
    Sym->setTableType(TableType);
  }
  // MVP object files can't have symtab entries for tables.
  if (!(Subtarget && Subtarget->hasCallIndirectOverlong()))
    Sym->setOmitFromLinkingSection();
  return Sym;
}

// Find a catch instruction from an EH pad.
MachineInstr *WebAssembly::findCatch(MachineBasicBlock *EHPad) {
  assert(EHPad->isEHPad());
  auto Pos = EHPad->begin();
  // Skip any label or debug instructions. Also skip 'end' marker instructions
  // that may exist after marker placement in CFGStackify.
  while (Pos != EHPad->end() &&
         (Pos->isLabel() || Pos->isDebugInstr() || isMarker(Pos->getOpcode())))
    Pos++;
  if (Pos != EHPad->end() && WebAssembly::isCatch(Pos->getOpcode()))
    return &*Pos;
  return nullptr;
}

unsigned WebAssembly::getCopyOpcodeForRegClass(const TargetRegisterClass *RC) {
  assert(RC != nullptr);
  switch (RC->getID()) {
  case WebAssembly::I32RegClassID:
    return WebAssembly::COPY_I32;
  case WebAssembly::I64RegClassID:
    return WebAssembly::COPY_I64;
  case WebAssembly::F32RegClassID:
    return WebAssembly::COPY_F32;
```
- **EN**: Implements helper routine(s) `setWeak`, `setType`, `setTableType` for this portion of the WebAssembly backend shared backend utility helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端共享工具函数所需的辅助例程 `setWeak`, `setType`, `setTableType`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 171-196

```cpp
  case WebAssembly::F64RegClassID:
    return WebAssembly::COPY_F64;
  case WebAssembly::V128RegClassID:
    return WebAssembly::COPY_V128;
  case WebAssembly::FUNCREFRegClassID:
    return WebAssembly::COPY_FUNCREF;
  case WebAssembly::EXTERNREFRegClassID:
    return WebAssembly::COPY_EXTERNREF;
  case WebAssembly::EXNREFRegClassID:
    return WebAssembly::COPY_EXNREF;
  default:
    llvm_unreachable("Unexpected register class");
  }
}

bool WebAssembly::canLowerMultivalueReturn(
    const WebAssemblySubtarget *Subtarget) {
  const auto &TM = static_cast<const WebAssemblyTargetMachine &>(
      Subtarget->getTargetLowering()->getTargetMachine());
  return Subtarget->hasMultivalue() && TM.usesMultivalueABI();
}

bool WebAssembly::canLowerReturn(size_t ResultSize,
                                 const WebAssemblySubtarget *Subtarget) {
  return ResultSize <= 1 || canLowerMultivalueReturn(Subtarget);
}
```
- **EN**: Implements helper routine(s) `llvm_unreachable`, `canLowerMultivalueReturn`, `getTargetLowering` for this portion of the WebAssembly backend shared backend utility helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端共享工具函数所需的辅助例程 `llvm_unreachable`, `canLowerMultivalueReturn`, `getTargetLowering`。 子目标特性裁剪会影响这里的行为。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Shared backend utility helpers / 后端共享工具函数
- Calling convention handling / 调用约定处理
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyUtilities.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblyTargetMachine.h`
- `llvm/CodeGen/MachineInstr.h`
- `llvm/IR/Function.h`
- `llvm/MC/MCContext.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
