# ARCAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCAsmPrinter.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains a printer that converts from our internal representation of machine-dependent LLVM code to GNU format ARC assembly language.
- 目的（中文）: 实现汇编打印器，把 LLVM 机器指令降级为文本汇编或 MC 层输出。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCAsmPrinter.cpp - ARC LLVM assembly writer -------------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains a printer that converts from our internal representation
  10: // of machine-dependent LLVM code to GNU format ARC assembly language.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "ARC.h"
  15: #include "ARCMCInstLower.h"
  16: #include "ARCSubtarget.h"
  17: #include "ARCTargetMachine.h"
  18: #include "MCTargetDesc/ARCInstPrinter.h"
  19: #include "TargetInfo/ARCTargetInfo.h"
  20: #include "llvm/CodeGen/AsmPrinter.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/MachineInstr.h"
  22: #include "llvm/MC/MCAsmInfo.h"
  23: #include "llvm/MC/MCInst.h"
  24: #include "llvm/MC/MCStreamer.h"
  25: #include "llvm/MC/TargetRegistry.h"
  26: #include "llvm/Support/raw_ostream.h"
  27: 
  28: using namespace llvm;
  29: 
  30: #define DEBUG_TYPE "asm-printer"
  31: 
  32: namespace {
  33: 
  34: class ARCAsmPrinter : public AsmPrinter {
  35:   ARCMCInstLower MCInstLowering;
  36: 
  37: public:
  38:   static char ID;
  39: 
  40:   explicit ARCAsmPrinter(TargetMachine &TM,
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as ARCAsmPrinter, which organize the target-specific behavior exposed by the file. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 ARCAsmPrinter 等接口或数据结构，用于组织该文件暴露的目标专用行为。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 41-60

```cpp
  41:                          std::unique_ptr<MCStreamer> Streamer)
  42:       : AsmPrinter(TM, std::move(Streamer), ID),
  43:         MCInstLowering(&OutContext, *this) {}
  44: 
  45:   StringRef getPassName() const override { return "ARC Assembly Printer"; }
  46:   void emitInstruction(const MachineInstr *MI) override;
  47: 
  48:   bool runOnMachineFunction(MachineFunction &MF) override;
  49: };
  50: 
  51: } // end anonymous namespace
  52: 
  53: void ARCAsmPrinter::emitInstruction(const MachineInstr *MI) {
  54:   ARC_MC::verifyInstructionPredicates(MI->getOpcode(),
  55:                                       getSubtargetInfo().getFeatureBits());
  56: 
  57:   SmallString<128> Str;
  58:   raw_svector_ostream O(Str);
  59: 
  60:   switch (MI->getOpcode()) {
```

- EN: Function bodies or method definitions such as AsmPrinter, emitInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: AsmPrinter, emitInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 61-80

```cpp
  61:   case ARC::DBG_VALUE:
  62:     llvm_unreachable("Should be handled target independently");
  63:     break;
  64:   }
  65: 
  66:   MCInst TmpInst;
  67:   MCInstLowering.Lower(MI, TmpInst);
  68:   EmitToStreamer(*OutStreamer, TmpInst);
  69: }
  70: 
  71: bool ARCAsmPrinter::runOnMachineFunction(MachineFunction &MF) {
  72:   // Functions are 4-byte aligned.
  73:   MF.ensureAlignment(Align(4));
  74:   return AsmPrinter::runOnMachineFunction(MF);
  75: }
  76: 
  77: char ARCAsmPrinter::ID = 0;
  78: 
  79: INITIALIZE_PASS(ARCAsmPrinter, "arc-asm-printer", "ARC Assmebly Printer", false,
  80:                 false)
```

- EN: Function bodies or method definitions such as runOnMachineFunction contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: runOnMachineFunction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 81-85

```cpp
  81: 
  82: // Force static initialization.
  83: extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeARCAsmPrinter() {
  84:   RegisterAsmPrinter<ARCAsmPrinter> X(getTheARCTarget());
  85: }
```

- EN: In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 从文件角色看，它参与将机器指令打印或降级为汇编输出。

## Key Concepts / 关键概念

- MachineInstr to MC lowering / MachineInstr 到 MC 降级
- Assembly syntax emission / 汇编语法输出
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- MC instruction representation / MC 指令表示
- MC streaming and emission / MC 流式输出
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARC.h`, `ARCMCInstLower.h`, `ARCSubtarget.h`, `ARCTargetMachine.h`, `MCTargetDesc/ARCInstPrinter.h`, `TargetInfo/ARCTargetInfo.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineInstr.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, LLVM MC
