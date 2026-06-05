# WebAssemblyAsmPrinter.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyAsmPrinter.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: WebAssembly implementation of AsmPrinter-*- C++.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyAsmPrinter.h`，主要负责 WebAssembly 后端的汇编与 MC 发射逻辑。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
// WebAssemblyAsmPrinter.h - WebAssembly implementation of AsmPrinter-*- C++ -*-
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-13

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYASMPRINTER_H
#define LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYASMPRINTER_H

#include "WebAssemblyMachineFunctionInfo.h"
#include "WebAssemblySubtarget.h"
```
- **EN**: Pulls in direct dependencies required by this assembly / MC emission logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该汇编与 MC 发射逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 14-20

```cpp
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/Target/TargetMachine.h"

namespace llvm {
class WebAssemblyTargetStreamer;
```
- **EN**: Pulls in direct dependencies required by this assembly / MC emission logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该汇编与 MC 发射逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 21-42

```cpp
class LLVM_LIBRARY_VISIBILITY WebAssemblyAsmPrinter final : public AsmPrinter {
public:
  static char ID;

private:
  const WebAssemblySubtarget *Subtarget;
  const MachineRegisterInfo *MRI;
  WebAssemblyFunctionInfo *MFI;
  bool signaturesEmitted = false;

public:
  explicit WebAssemblyAsmPrinter(TargetMachine &TM,
                                 std::unique_ptr<MCStreamer> Streamer)
      : AsmPrinter(TM, std::move(Streamer), ID), Subtarget(nullptr),
        MRI(nullptr), MFI(nullptr) {}

  StringRef getPassName() const override {
    return "WebAssembly Assembly Printer";
  }

  const WebAssemblySubtarget &getSubtarget() const { return *Subtarget; }
```
- **EN**: Declares a backend-facing type `LLVM_LIBRARY_VISIBILITY`, `WebAssemblyAsmPrinter`, `AsmPrinter` and outlines the API or state that nearby code will rely on. Register assignment and register-class constraints matter here.
- **CN**: 这里声明面向后端的类型 `LLVM_LIBRARY_VISIBILITY`, `WebAssemblyAsmPrinter`, `AsmPrinter`，并勾勒出周边代码会依赖的接口或状态。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 43-44

```cpp
  //===------------------------------------------------------------------===//
  // MachineFunctionPass Implementation.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "MachineFunctionPass Implementation.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“MachineFunctionPass Implementation.”。

### Lines 45-53

```cpp
  //===------------------------------------------------------------------===//

  bool runOnMachineFunction(MachineFunction &MF) override {
    Subtarget = &MF.getSubtarget<WebAssemblySubtarget>();
    MRI = &MF.getRegInfo();
    MFI = MF.getInfo<WebAssemblyFunctionInfo>();
    return AsmPrinter::runOnMachineFunction(MF);
  }
```
- **EN**: Implements helper routine(s) `runOnMachineFunction`, `getRegInfo` for this portion of the WebAssembly backend assembly / MC emission logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `runOnMachineFunction`, `getRegInfo`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 54-55

```cpp
  //===------------------------------------------------------------------===//
  // AsmPrinter Implementation.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "AsmPrinter Implementation.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“AsmPrinter Implementation.”。

### Lines 56-81

```cpp
  //===------------------------------------------------------------------===//

  void emitEndOfAsmFile(Module &M) override;
  void EmitProducerInfo(Module &M);
  void EmitTargetFeatures(Module &M);
  void EmitFunctionAttributes(Module &M);
  void emitSymbolType(const MCSymbolWasm *Sym);
  void emitGlobalVariable(const GlobalVariable *GV) override;
  void emitJumpTableInfo() override;
  void emitConstantPool() override;
  void emitFunctionBodyStart() override;
  void emitInstruction(const MachineInstr *MI) override;
  bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
                       const char *ExtraCode, raw_ostream &OS) override;
  bool PrintAsmMemoryOperand(const MachineInstr *MI, unsigned OpNo,
                             const char *ExtraCode, raw_ostream &OS) override;

  MVT getRegType(unsigned RegNo) const;
  std::string regToString(const MachineOperand &MO);
  WebAssemblyTargetStreamer *getTargetStreamer();
  MCSymbolWasm *getMCSymbolForFunction(const Function *F,
                                       wasm::WasmSignature *Sig,
                                       bool &InvokeDetected);
  MCSymbol *getOrCreateWasmSymbol(StringRef Name);
  void emitDecls(const Module &M);
};
```
- **EN**: Declares function entry points including `emitEndOfAsmFile`, `EmitProducerInfo`, `EmitTargetFeatures` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `emitEndOfAsmFile`, `EmitProducerInfo`, `EmitTargetFeatures`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 82-85

```cpp

} // end namespace llvm

#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Assembly / MC emission logic / 汇编与 MC 发射逻辑
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Assembly or MC emission / 汇编或 MC 发射
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置
- Object format integration / 目标文件格式集成

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblySubtarget.h`
- `llvm/CodeGen/AsmPrinter.h`
- `llvm/MC/MCStreamer.h`
- `llvm/Target/TargetMachine.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
