# CSKYMCTargetDesc.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/MCTargetDesc/CSKYMCTargetDesc.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file provides CSKY specific target descriptions.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYMCTargetDesc.cpp - CSKY Target Descriptions -------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// This file provides CSKY specific target descriptions.
  10: ///
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "CSKYMCTargetDesc.h"
  14: #include "CSKYAsmBackend.h"
  15: #include "CSKYELFStreamer.h"
  16: #include "CSKYInstPrinter.h"
  17: #include "CSKYMCAsmInfo.h"
  18: #include "CSKYTargetStreamer.h"
  19: #include "TargetInfo/CSKYTargetInfo.h"
  20: #include "llvm/MC/MCAssembler.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCInstrAnalysis.h"
  22: #include "llvm/MC/MCInstrInfo.h"
  23: #include "llvm/MC/MCRegisterInfo.h"
  24: #include "llvm/MC/MCSubtargetInfo.h"
  25: #include "llvm/MC/TargetRegistry.h"
  26: 
  27: #define GET_INSTRINFO_MC_DESC
  28: #define ENABLE_INSTR_PREDICATE_VERIFIER
  29: #include "CSKYGenInstrInfo.inc"
  30: 
  31: #define GET_REGINFO_MC_DESC
  32: #include "CSKYGenRegisterInfo.inc"
  33: 
  34: #define GET_SUBTARGETINFO_MC_DESC
  35: #include "CSKYGenSubtargetInfo.inc"
  36: 
  37: using namespace llvm;
  38: 
  39: static MCAsmInfo *createCSKYMCAsmInfo(const MCRegisterInfo &MRI,
  40:                                       const Triple &TT,
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 41-60

```cpp
  41:                                       const MCTargetOptions &Options) {
  42:   MCAsmInfo *MAI = new CSKYMCAsmInfo(TT, Options);
  43: 
  44:   // Initial state of the frame pointer is SP.
  45:   unsigned Reg = MRI.getDwarfRegNum(CSKY::R14, true);
  46:   MCCFIInstruction Inst = MCCFIInstruction::cfiDefCfa(nullptr, Reg, 0);
  47:   MAI->addInitialFrameState(Inst);
  48:   return MAI;
  49: }
  50: 
  51: static MCInstrInfo *createCSKYMCInstrInfo() {
  52:   MCInstrInfo *Info = new MCInstrInfo();
  53:   InitCSKYMCInstrInfo(Info);
  54:   return Info;
  55: }
  56: 
  57: static MCInstPrinter *createCSKYMCInstPrinter(const Triple &T,
  58:                                               unsigned SyntaxVariant,
  59:                                               const MCAsmInfo &MAI,
  60:                                               const MCInstrInfo &MII,
```

- EN: This range continues the implementation of the backend component described by CSKYMCTargetDesc.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 61-80

```cpp
  61:                                               const MCRegisterInfo &MRI) {
  62:   return new CSKYInstPrinter(MAI, MII, MRI);
  63: }
  64: 
  65: static MCRegisterInfo *createCSKYMCRegisterInfo(const Triple &TT) {
  66:   MCRegisterInfo *Info = new MCRegisterInfo();
  67:   InitCSKYMCRegisterInfo(Info, CSKY::R15);
  68:   return Info;
  69: }
  70: 
  71: static MCSubtargetInfo *createCSKYMCSubtargetInfo(const Triple &TT,
  72:                                                   StringRef CPU, StringRef FS) {
  73:   std::string CPUName = std::string(CPU);
  74:   if (CPUName.empty())
  75:     CPUName = "generic";
  76:   return createCSKYMCSubtargetInfoImpl(TT, CPUName, /*TuneCPU=*/CPUName, FS);
  77: }
  78: 
  79: static MCTargetStreamer *
  80: createCSKYObjectTargetStreamer(MCStreamer &S, const MCSubtargetInfo &STI) {
```

- EN: Function bodies or method definitions such as createCSKYObjectTargetStreamer contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: createCSKYObjectTargetStreamer 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 81-100

```cpp
  81:   const Triple &TT = STI.getTargetTriple();
  82:   if (TT.isOSBinFormatELF())
  83:     return new CSKYTargetELFStreamer(S, STI);
  84:   return nullptr;
  85: }
  86: 
  87: static MCStreamer *createELFStreamer(const Triple &T, MCContext &Ctx,
  88:                                      std::unique_ptr<MCAsmBackend> &&MAB,
  89:                                      std::unique_ptr<MCObjectWriter> &&OW,
  90:                                      std::unique_ptr<MCCodeEmitter> &&Emitter) {
  91:   CSKYELFStreamer *S = new CSKYELFStreamer(Ctx, std::move(MAB), std::move(OW),
  92:                                            std::move(Emitter));
  93: 
  94:   return S;
  95: }
  96: 
  97: static MCTargetStreamer *
  98: createCSKYAsmTargetStreamer(MCStreamer &S, formatted_raw_ostream &OS,
  99:                             MCInstPrinter *InstPrinter) {
 100:   return new CSKYTargetAsmStreamer(S, OS);
```

- EN: Function bodies or method definitions such as createCSKYAsmTargetStreamer contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: createCSKYAsmTargetStreamer 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101: }
 102: 
 103: static MCTargetStreamer *createCSKYNullTargetStreamer(MCStreamer &S) {
 104:   return new CSKYTargetStreamer(S);
 105: }
 106: 
 107: namespace {
 108: 
 109: class CSKYMCInstrAnalysis : public MCInstrAnalysis {
 110: public:
 111:   explicit CSKYMCInstrAnalysis(const MCInstrInfo *Info)
 112:       : MCInstrAnalysis(Info) {}
 113: 
 114:   bool evaluateBranch(const MCInst &Inst, uint64_t Addr, uint64_t Size,
 115:                       uint64_t &Target) const override {
 116:     if (isConditionalBranch(Inst) || isUnconditionalBranch(Inst)) {
 117:       int64_t Imm;
 118:       Imm = Inst.getOperand(Inst.getNumOperands() - 1).getImm();
 119:       Target = Addr + Imm;
 120:       return true;
```

- EN: This chunk introduces interfaces or data structures such as CSKYMCInstrAnalysis, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as CSKYMCInstrAnalysis contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这一段引入了 CSKYMCInstrAnalysis 等接口或数据结构，用于组织该文件暴露的目标专用行为。 CSKYMCInstrAnalysis 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121:     }
 122: 
 123:     if (Inst.getOpcode() == CSKY::BSR32) {
 124:       Target = Addr + Inst.getOperand(0).getImm();
 125:       return true;
 126:     }
 127: 
 128:     switch (Inst.getOpcode()) {
 129:     default:
 130:       return false;
 131:     case CSKY::LRW16:
 132:     case CSKY::LRW32:
 133:     case CSKY::JSRI32:
 134:     case CSKY::JMPI32:
 135:       int64_t Imm = Inst.getOperand(Inst.getNumOperands() - 1).getImm();
 136:       Target = ((Addr + Imm) & 0xFFFFFFFC);
 137:       return true;
 138:     }
 139: 
 140:     return false;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 141-160

```cpp
 141:   }
 142: };
 143: 
 144: } // end anonymous namespace
 145: 
 146: static MCInstrAnalysis *createCSKYInstrAnalysis(const MCInstrInfo *Info) {
 147:   return new CSKYMCInstrAnalysis(Info);
 148: }
 149: 
 150: extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeCSKYTargetMC() {
 151:   auto &CSKYTarget = getTheCSKYTarget();
 152:   TargetRegistry::RegisterMCAsmBackend(CSKYTarget, createCSKYAsmBackend);
 153:   TargetRegistry::RegisterMCAsmInfo(CSKYTarget, createCSKYMCAsmInfo);
 154:   TargetRegistry::RegisterMCInstrInfo(CSKYTarget, createCSKYMCInstrInfo);
 155:   TargetRegistry::RegisterMCRegInfo(CSKYTarget, createCSKYMCRegisterInfo);
 156:   TargetRegistry::RegisterMCCodeEmitter(CSKYTarget, createCSKYMCCodeEmitter);
 157:   TargetRegistry::RegisterMCInstPrinter(CSKYTarget, createCSKYMCInstPrinter);
 158:   TargetRegistry::RegisterMCSubtargetInfo(CSKYTarget,
 159:                                           createCSKYMCSubtargetInfo);
 160:   TargetRegistry::RegisterELFStreamer(CSKYTarget, createELFStreamer);
```

- EN: This range continues the implementation of the backend component described by CSKYMCTargetDesc.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 161-169

```cpp
 161:   TargetRegistry::RegisterObjectTargetStreamer(CSKYTarget,
 162:                                                createCSKYObjectTargetStreamer);
 163:   TargetRegistry::RegisterAsmTargetStreamer(CSKYTarget,
 164:                                             createCSKYAsmTargetStreamer);
 165:   // Register the null target streamer.
 166:   TargetRegistry::RegisterNullTargetStreamer(CSKYTarget,
 167:                                              createCSKYNullTargetStreamer);
 168:   TargetRegistry::RegisterMCInstrAnalysis(CSKYTarget, createCSKYInstrAnalysis);
 169: }
```

- EN: This range continues the implementation of the backend component described by CSKYMCTargetDesc.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- MC instruction representation / MC 指令表示
- MC streaming and emission / MC 流式输出
- Assembler backend policies / 汇编后端策略
- Binary encoding / 二进制编码
- CPU feature modelling / CPU 特性建模
- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYMCTargetDesc.h`, `CSKYAsmBackend.h`, `CSKYELFStreamer.h`, `CSKYInstPrinter.h`, `CSKYMCAsmInfo.h`, `CSKYTargetStreamer.h`, `TargetInfo/CSKYTargetInfo.h`, `llvm/MC/MCAssembler.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Generated or companion files / 生成或配套文件: `CSKYGenInstrInfo.inc`, `CSKYGenRegisterInfo.inc`, `CSKYGenSubtargetInfo.inc`
- Local companions / 本地配套文件: `CSKYMCTargetDesc.h`
