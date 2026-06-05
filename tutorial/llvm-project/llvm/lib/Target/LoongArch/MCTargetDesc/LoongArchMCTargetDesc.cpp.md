# LoongArchMCTargetDesc.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/MCTargetDesc/LoongArchMCTargetDesc.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- LoongArchMCTargetDesc.cpp - LoongArch Target Descriptions ---------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file provides LoongArch specific target descriptions.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LoongArchMCTargetDesc.h"
  14: #include "LoongArchELFStreamer.h"
  15: #include "LoongArchInstPrinter.h"
  16: #include "LoongArchMCAsmInfo.h"
  17: #include "TargetInfo/LoongArchTargetInfo.h"
  18: #include "llvm/MC/MCAsmBackend.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchMCTargetDesc.h`, `LoongArchELFStreamer.h`, `LoongArchInstPrinter.h`, `LoongArchMCAsmInfo.h`, `LoongArchTargetInfo.h`, `MCAsmBackend.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchMCTargetDesc.h`, `LoongArchELFStreamer.h`, `LoongArchInstPrinter.h`, `LoongArchMCAsmInfo.h`, `LoongArchTargetInfo.h`, `MCAsmBackend.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/MC/MCAsmInfo.h"
  20: #include "llvm/MC/MCCodeEmitter.h"
  21: #include "llvm/MC/MCDwarf.h"
  22: #include "llvm/MC/MCInstrAnalysis.h"
  23: #include "llvm/MC/MCInstrInfo.h"
  24: #include "llvm/MC/MCObjectWriter.h"
  25: #include "llvm/MC/MCRegisterInfo.h"
  26: #include "llvm/MC/MCSubtargetInfo.h"
  27: #include "llvm/MC/TargetRegistry.h"
  28: #include "llvm/Support/Compiler.h"
  29: #include <bitset>
  30: 
  31: #define GET_INSTRINFO_MC_DESC
  32: #define ENABLE_INSTR_PREDICATE_VERIFIER
  33: #include "LoongArchGenInstrInfo.inc"
  34: 
  35: #define GET_REGINFO_MC_DESC
  36: #include "LoongArchGenRegisterInfo.inc"
```
- **EN**: It imports dependencies such as `MCAsmInfo.h`, `MCCodeEmitter.h`, `MCDwarf.h`, `MCInstrAnalysis.h`, `MCInstrInfo.h`, `MCObjectWriter.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file. Generated `.inc` fragments are pulled in here, a common LLVM technique for TableGen-produced code.
- **CN**: 它引入了 `MCAsmInfo.h`, `MCCodeEmitter.h`, `MCDwarf.h`, `MCInstrAnalysis.h`, `MCInstrInfo.h`, `MCObjectWriter.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。 这里引入了生成的 `.inc` 片段，这是 LLVM 使用 TableGen 产物的常见方式。

### Lines 37-54 / 第 37-54 行
```cpp
  37: 
  38: #define GET_SUBTARGETINFO_MC_DESC
  39: #include "LoongArchGenSubtargetInfo.inc"
  40: 
  41: using namespace llvm;
  42: 
  43: static MCRegisterInfo *createLoongArchMCRegisterInfo(const Triple &TT) {
  44:   MCRegisterInfo *X = new MCRegisterInfo();
  45:   InitLoongArchMCRegisterInfo(X, LoongArch::R1);
  46:   return X;
  47: }
  48: 
  49: static MCInstrInfo *createLoongArchMCInstrInfo() {
  50:   MCInstrInfo *X = new MCInstrInfo();
  51:   InitLoongArchMCInstrInfo(X);
  52:   return X;
  53: }
  54: 
```
- **EN**: It imports dependencies such as `LoongArchGenSubtargetInfo.inc` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file. Generated `.inc` fragments are pulled in here, a common LLVM technique for TableGen-produced code.
- **CN**: 它引入了 `LoongArchGenSubtargetInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。 这里引入了生成的 `.inc` 片段，这是 LLVM 使用 TableGen 产物的常见方式。

### Lines 55-72 / 第 55-72 行
```cpp
  55: static MCSubtargetInfo *
  56: createLoongArchMCSubtargetInfo(const Triple &TT, StringRef CPU, StringRef FS) {
  57:   if (CPU.empty() || CPU == "generic")
  58:     CPU = TT.isArch64Bit() ? "generic-la64" : "generic-la32";
  59:   return createLoongArchMCSubtargetInfoImpl(TT, CPU, /*TuneCPU*/ CPU, FS);
  60: }
  61: 
  62: static MCAsmInfo *createLoongArchMCAsmInfo(const MCRegisterInfo &MRI,
  63:                                            const Triple &TT,
  64:                                            const MCTargetOptions &Options) {
  65:   MCAsmInfo *MAI = new LoongArchMCAsmInfo(TT, Options);
  66: 
  67:   // Initial state of the frame pointer is sp(r3).
  68:   unsigned SP = MRI.getDwarfRegNum(LoongArch::R3, true);
  69:   MCCFIInstruction Inst = MCCFIInstruction::cfiDefCfa(nullptr, SP, 0);
  70:   MAI->addInitialFrameState(Inst);
  71: 
  72:   return MAI;
```
- **EN**: The range implements or declares functions including `createLoongArchMCSubtargetInfo`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `createLoongArchMCSubtargetInfo` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 73-90 / 第 73-90 行
```cpp
  73: }
  74: 
  75: static MCInstPrinter *createLoongArchMCInstPrinter(const Triple &T,
  76:                                                    unsigned SyntaxVariant,
  77:                                                    const MCAsmInfo &MAI,
  78:                                                    const MCInstrInfo &MII,
  79:                                                    const MCRegisterInfo &MRI) {
  80:   return new LoongArchInstPrinter(MAI, MII, MRI);
  81: }
  82: 
  83: static MCTargetStreamer *
  84: createLoongArchObjectTargetStreamer(MCStreamer &S, const MCSubtargetInfo &STI) {
  85:   return STI.getTargetTriple().isOSBinFormatELF()
  86:              ? new LoongArchTargetELFStreamer(S, STI)
  87:              : nullptr;
  88: }
  89: 
  90: static MCTargetStreamer *
```
- **EN**: The range implements or declares functions including `createLoongArchObjectTargetStreamer`.
- **CN**: 这一段实现或声明了 `createLoongArchObjectTargetStreamer` 等函数。

### Lines 91-108 / 第 91-108 行
```cpp
  91: createLoongArchAsmTargetStreamer(MCStreamer &S, formatted_raw_ostream &OS,
  92:                                  MCInstPrinter *InstPrint) {
  93:   return new LoongArchTargetAsmStreamer(S, OS);
  94: }
  95: 
  96: namespace {
  97: 
  98: class LoongArchMCInstrAnalysis : public MCInstrAnalysis {
  99:   int64_t GPRState[31] = {};
 100:   std::bitset<31> GPRValidMask;
 101: 
 102:   static bool isGPR(MCRegister Reg) {
 103:     return Reg >= LoongArch::R0 && Reg <= LoongArch::R31;
 104:   }
 105: 
 106:   static unsigned getRegIndex(MCRegister Reg) {
 107:     assert(isGPR(Reg) && Reg != LoongArch::R0 && "Invalid GPR reg");
 108:     return Reg - LoongArch::R1;
```
- **EN**: This block declares or refines TableGen records such as `LoongArchMCInstrAnalysis`. The range implements or declares functions including `isGPR`, `getRegIndex`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 该代码块声明或细化了 `LoongArchMCInstrAnalysis` 等 TableGen 记录。 这一段实现或声明了 `isGPR`, `getRegIndex` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   }
 110: 
 111:   void setGPRState(MCRegister Reg, std::optional<int64_t> Value) {
 112:     if (Reg == LoongArch::R0)
 113:       return;
 114: 
 115:     auto Index = getRegIndex(Reg);
 116: 
 117:     if (Value) {
 118:       GPRState[Index] = *Value;
 119:       GPRValidMask.set(Index);
 120:     } else {
 121:       GPRValidMask.reset(Index);
 122:     }
 123:   }
 124: 
 125:   std::optional<int64_t> getGPRState(MCRegister Reg) const {
 126:     if (Reg == LoongArch::R0)
```
- **EN**: The range implements or declares functions including `setGPRState`, `getGPRState`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `setGPRState`, `getGPRState` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:       return 0;
 128: 
 129:     auto Index = getRegIndex(Reg);
 130: 
 131:     if (GPRValidMask.test(Index))
 132:       return GPRState[Index];
 133:     return std::nullopt;
 134:   }
 135: 
 136: public:
 137:   explicit LoongArchMCInstrAnalysis(const MCInstrInfo *Info)
 138:       : MCInstrAnalysis(Info) {}
 139: 
 140:   void resetState() override { GPRValidMask.reset(); }
 141: 
 142:   void updateState(const MCInst &Inst, const MCSubtargetInfo *STI,
 143:                    uint64_t Addr) override {
 144:     // Terminators mark the end of a basic block which means the sequentially
```
- **EN**: The range implements or declares functions including `LoongArchMCInstrAnalysis`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchMCInstrAnalysis` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145:     // next instruction will be the first of another basic block and the current
 146:     // state will typically not be valid anymore. For calls, we assume all
 147:     // registers may be clobbered by the callee (TODO: should we take the
 148:     // calling convention into account?).
 149:     if (isTerminator(Inst) || isCall(Inst)) {
 150:       resetState();
 151:       return;
 152:     }
 153: 
 154:     switch (Inst.getOpcode()) {
 155:     default: {
 156:       // Clear the state of all defined registers for instructions that we don't
 157:       // explicitly support.
 158:       auto NumDefs = Info->get(Inst.getOpcode()).getNumDefs();
 159:       for (unsigned I = 0; I < NumDefs; ++I) {
 160:         auto DefReg = Inst.getOperand(I).getReg();
 161:         if (isGPR(DefReg))
 162:           setGPRState(DefReg, std::nullopt);
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 163-180 / 第 163-180 行
```cpp
 163:       }
 164:       break;
 165:     }
 166:     case LoongArch::PCADDU18I:
 167:       setGPRState(
 168:           Inst.getOperand(0).getReg(),
 169:           Addr + SignExtend64<38>(
 170:                      static_cast<uint64_t>(Inst.getOperand(1).getImm()) << 18));
 171:       break;
 172:     }
 173:   }
 174: 
 175:   bool evaluateBranch(const MCInst &Inst, uint64_t Addr, uint64_t Size,
 176:                       uint64_t &Target) const override {
 177:     unsigned NumOps = Inst.getNumOperands();
 178:     if ((isBranch(Inst) && !isIndirectBranch(Inst)) ||
 179:         Inst.getOpcode() == LoongArch::BL) {
 180:       Target = Addr + Inst.getOperand(NumOps - 1).getImm();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181:       return true;
 182:     }
 183: 
 184:     if (Inst.getOpcode() == LoongArch::JIRL) {
 185:       if (auto TargetRegState = getGPRState(Inst.getOperand(1).getReg())) {
 186:         Target = *TargetRegState + Inst.getOperand(2).getImm();
 187:         return true;
 188:       }
 189:       return false;
 190:     }
 191: 
 192:     return false;
 193:   }
 194: 
 195:   bool isTerminator(const MCInst &Inst) const override {
 196:     if (MCInstrAnalysis::isTerminator(Inst))
 197:       return true;
 198: 
```
- **EN**: The range implements or declares functions including `isTerminator`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isTerminator` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-216 / 第 199-216 行
```cpp
 199:     switch (Inst.getOpcode()) {
 200:     default:
 201:       return false;
 202:     case LoongArch::JIRL:
 203:       return Inst.getOperand(0).getReg() == LoongArch::R0;
 204:     }
 205:   }
 206: 
 207:   bool isCall(const MCInst &Inst) const override {
 208:     if (MCInstrAnalysis::isCall(Inst))
 209:       return true;
 210: 
 211:     switch (Inst.getOpcode()) {
 212:     default:
 213:       return false;
 214:     case LoongArch::JIRL:
 215:       return Inst.getOperand(0).getReg() != LoongArch::R0;
 216:     }
```
- **EN**: The range implements or declares functions including `isCall`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isCall` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 217-234 / 第 217-234 行
```cpp
 217:   }
 218: 
 219:   bool isReturn(const MCInst &Inst) const override {
 220:     if (MCInstrAnalysis::isReturn(Inst))
 221:       return true;
 222: 
 223:     switch (Inst.getOpcode()) {
 224:     default:
 225:       return false;
 226:     case LoongArch::JIRL:
 227:       return Inst.getOperand(0).getReg() == LoongArch::R0 &&
 228:              Inst.getOperand(1).getReg() == LoongArch::R1;
 229:     }
 230:   }
 231: 
 232:   bool isBranch(const MCInst &Inst) const override {
 233:     if (MCInstrAnalysis::isBranch(Inst))
 234:       return true;
```
- **EN**: The range implements or declares functions including `isReturn`, `isBranch`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isReturn`, `isBranch` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235: 
 236:     switch (Inst.getOpcode()) {
 237:     default:
 238:       return false;
 239:     case LoongArch::JIRL:
 240:       return Inst.getOperand(0).getReg() == LoongArch::R0 &&
 241:              Inst.getOperand(1).getReg() != LoongArch::R1;
 242:     }
 243:   }
 244: 
 245:   bool isUnconditionalBranch(const MCInst &Inst) const override {
 246:     if (MCInstrAnalysis::isUnconditionalBranch(Inst))
 247:       return true;
 248: 
 249:     switch (Inst.getOpcode()) {
 250:     default:
 251:       return false;
 252:     case LoongArch::JIRL:
```
- **EN**: The range implements or declares functions including `isUnconditionalBranch`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isUnconditionalBranch` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-270 / 第 253-270 行
```cpp
 253:       return Inst.getOperand(0).getReg() == LoongArch::R0 &&
 254:              Inst.getOperand(1).getReg() != LoongArch::R1;
 255:     }
 256:   }
 257: 
 258:   bool isIndirectBranch(const MCInst &Inst) const override {
 259:     if (MCInstrAnalysis::isIndirectBranch(Inst))
 260:       return true;
 261: 
 262:     switch (Inst.getOpcode()) {
 263:     default:
 264:       return false;
 265:     case LoongArch::JIRL:
 266:       return Inst.getOperand(0).getReg() == LoongArch::R0 &&
 267:              Inst.getOperand(1).getReg() != LoongArch::R1;
 268:     }
 269:   }
 270: };
```
- **EN**: The range implements or declares functions including `isIndirectBranch`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isIndirectBranch` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 271-288 / 第 271-288 行
```cpp
 271: 
 272: } // end namespace
 273: 
 274: static MCInstrAnalysis *createLoongArchInstrAnalysis(const MCInstrInfo *Info) {
 275:   return new LoongArchMCInstrAnalysis(Info);
 276: }
 277: 
 278: namespace {
 279: MCStreamer *createLoongArchELFStreamer(const Triple &T, MCContext &Context,
 280:                                        std::unique_ptr<MCAsmBackend> &&MAB,
 281:                                        std::unique_ptr<MCObjectWriter> &&MOW,
 282:                                        std::unique_ptr<MCCodeEmitter> &&MCE) {
 283:   return createLoongArchELFStreamer(Context, std::move(MAB), std::move(MOW),
 284:                                     std::move(MCE));
 285: }
 286: } // end namespace
 287: 
 288: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 289-305 / 第 289-305 行
```cpp
 289: LLVMInitializeLoongArchTargetMC() {
 290:   for (Target *T : {&getTheLoongArch32Target(), &getTheLoongArch64Target()}) {
 291:     TargetRegistry::RegisterMCRegInfo(*T, createLoongArchMCRegisterInfo);
 292:     TargetRegistry::RegisterMCInstrInfo(*T, createLoongArchMCInstrInfo);
 293:     TargetRegistry::RegisterMCSubtargetInfo(*T, createLoongArchMCSubtargetInfo);
 294:     TargetRegistry::RegisterMCAsmInfo(*T, createLoongArchMCAsmInfo);
 295:     TargetRegistry::RegisterMCCodeEmitter(*T, createLoongArchMCCodeEmitter);
 296:     TargetRegistry::RegisterMCAsmBackend(*T, createLoongArchAsmBackend);
 297:     TargetRegistry::RegisterMCInstPrinter(*T, createLoongArchMCInstPrinter);
 298:     TargetRegistry::RegisterMCInstrAnalysis(*T, createLoongArchInstrAnalysis);
 299:     TargetRegistry::RegisterELFStreamer(*T, createLoongArchELFStreamer);
 300:     TargetRegistry::RegisterObjectTargetStreamer(
 301:         *T, createLoongArchObjectTargetStreamer);
 302:     TargetRegistry::RegisterAsmTargetStreamer(*T,
 303:                                               createLoongArchAsmTargetStreamer);
 304:   }
 305: }
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **MC streaming**: Emits sections, symbols, expressions, and encoded bytes to object or assembly output. / 向目标文件或汇编输出节、符号、表达式和编码字节。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `LoongArchMCTargetDesc.h`
- `LoongArchELFStreamer.h`
- `LoongArchInstPrinter.h`
- `LoongArchMCAsmInfo.h`
- `TargetInfo/LoongArchTargetInfo.h`
- `llvm/MC/MCAsmBackend.h`
- `llvm/MC/MCAsmInfo.h`
- `llvm/MC/MCCodeEmitter.h`
- `llvm/MC/MCDwarf.h`
- `llvm/MC/MCInstrAnalysis.h`
- `llvm/MC/MCInstrInfo.h`
- `llvm/MC/MCObjectWriter.h`
- `llvm/MC/MCRegisterInfo.h`
- `llvm/MC/MCSubtargetInfo.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/Compiler.h`
- `...` (4 more include dependencies omitted for brevity / 其余 4 个 include 依赖已省略)
