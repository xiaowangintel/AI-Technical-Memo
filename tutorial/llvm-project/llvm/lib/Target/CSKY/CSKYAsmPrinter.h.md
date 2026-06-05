# CSKYAsmPrinter.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYAsmPrinter.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Implements the assembly printer that lowers LLVM machine instructions to textual assembly or MC layer output.
- 目的（中文）: 实现汇编打印器，把 LLVM 机器指令降级为文本汇编或 MC 层输出。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYAsmPrinter.h - CSKY implementation of AsmPrinter ----*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #ifndef LLVM_LIB_TARGET_CSKY_CSKYASMPRINTER_H
  10: #define LLVM_LIB_TARGET_CSKY_CSKYASMPRINTER_H
  11: 
  12: #include "CSKYMCInstLower.h"
  13: #include "CSKYSubtarget.h"
  14: #include "llvm/CodeGen/AsmPrinter.h"
  15: #include "llvm/MC/MCDirectives.h"
  16: 
  17: namespace llvm {
  18: class LLVM_LIBRARY_VISIBILITY CSKYAsmPrinter : public AsmPrinter {
  19:   CSKYMCInstLower MCInstLowering;
  20: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as LLVM_LIBRARY_VISIBILITY, which organize the target-specific behavior exposed by the file. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 LLVM_LIBRARY_VISIBILITY 等接口或数据结构，用于组织该文件暴露的目标专用行为。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 21-40

```cpp
  21:   const MCSubtargetInfo *Subtarget;
  22:   const TargetInstrInfo *TII;
  23: 
  24:   bool InConstantPool = false;
  25: 
  26:   /// Keep a pointer to constantpool entries of the current
  27:   /// MachineFunction.
  28:   MachineConstantPool *MCP;
  29: 
  30:   void expandTLSLA(const MachineInstr *MI);
  31:   void emitCustomConstantPool(const MachineInstr *MI);
  32:   void emitAttributes();
  33: 
  34: public:
  35:   explicit CSKYAsmPrinter(TargetMachine &TM,
  36:                           std::unique_ptr<MCStreamer> Streamer);
  37: 
  38:   StringRef getPassName() const override { return "CSKY Assembly Printer"; }
  39: 
  40:   void EmitToStreamer(MCStreamer &S, const MCInst &Inst);
```

- EN: In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 41-60

```cpp
  41: 
  42:   /// tblgen'erated driver function for lowering simple MI->MC
  43:   /// pseudo instructions.
  44:   bool lowerPseudoInstExpansion(const MachineInstr *MI, MCInst &Inst);
  45: 
  46:   void emitMachineConstantPoolValue(MachineConstantPoolValue *MCPV) override;
  47: 
  48:   void emitFunctionBodyEnd() override;
  49: 
  50:   void emitStartOfAsmFile(Module &M) override;
  51: 
  52:   void emitEndOfAsmFile(Module &M) override;
  53: 
  54:   void emitInstruction(const MachineInstr *MI) override;
  55: 
  56:   bool runOnMachineFunction(MachineFunction &MF) override;
  57: 
  58:   // we emit constant pools customly!
  59:   void emitConstantPool() override {}
  60: 
```

- EN: In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 61-69

```cpp
  61:   bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
  62:                        const char *ExtraCode, raw_ostream &OS) override;
  63: 
  64:   bool PrintAsmMemoryOperand(const MachineInstr *MI, unsigned OpNo,
  65:                              const char *ExtraCode, raw_ostream &OS) override;
  66: };
  67: } // end namespace llvm
  68: 
  69: #endif // LLVM_LIB_TARGET_CSKY_CSKYASMPRINTER_H
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

## Key Concepts / 关键概念

- MachineInstr to MC lowering / MachineInstr 到 MC 降级
- Assembly syntax emission / 汇编语法输出
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- MC instruction representation / MC 指令表示
- MC streaming and emission / MC 流式输出
- Instruction semantics helpers / 指令语义辅助逻辑
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYMCInstLower.h`, `CSKYSubtarget.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/MC/MCDirectives.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, LLVM MC
- Local companions / 本地配套文件: `CSKYAsmPrinter.cpp`
