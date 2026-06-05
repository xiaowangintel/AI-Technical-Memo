# SystemZMCTargetDesc.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/MCTargetDesc/SystemZMCTargetDesc.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZMCTargetDesc.cpp - SystemZ target descriptions -------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "SystemZMCTargetDesc.h"
  10: #include "SystemZGNUInstPrinter.h"
  11: #include "SystemZHLASMAsmStreamer.h"
  12: #include "SystemZHLASMInstPrinter.h"
  13: #include "SystemZMCAsmInfo.h"
  14: #include "SystemZTargetStreamer.h"
  15: #include "TargetInfo/SystemZTargetInfo.h"
  16: #include "llvm/MC/MCContext.h"
  17: #include "llvm/MC/MCDwarf.h"
  18: #include "llvm/MC/MCInst.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZMCTargetDesc.h`, `SystemZGNUInstPrinter.h`, `SystemZHLASMAsmStreamer.h`, `SystemZHLASMInstPrinter.h`, `SystemZMCAsmInfo.h`, `SystemZTargetStreamer.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZMCTargetDesc.h`, `SystemZGNUInstPrinter.h`, `SystemZHLASMAsmStreamer.h`, `SystemZHLASMInstPrinter.h`, `SystemZMCAsmInfo.h`, `SystemZTargetStreamer.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/MC/MCInstrAnalysis.h"
  20: #include "llvm/MC/MCInstrInfo.h"
  21: #include "llvm/MC/MCRegisterInfo.h"
  22: #include "llvm/MC/MCStreamer.h"
  23: #include "llvm/MC/MCSubtargetInfo.h"
  24: #include "llvm/MC/TargetRegistry.h"
  25: #include "llvm/Support/CommandLine.h"
  26: #include "llvm/Support/Compiler.h"
  27: 
  28: using namespace llvm;
  29: 
  30: #define GET_INSTRINFO_MC_DESC
  31: #define ENABLE_INSTR_PREDICATE_VERIFIER
  32: #include "SystemZGenInstrInfo.inc"
  33: 
  34: #define GET_SUBTARGETINFO_MC_DESC
  35: #include "SystemZGenSubtargetInfo.inc"
  36: 
```
- **EN**: It imports dependencies such as `MCInstrAnalysis.h`, `MCInstrInfo.h`, `MCRegisterInfo.h`, `MCStreamer.h`, `MCSubtargetInfo.h`, `TargetRegistry.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file. Generated `.inc` fragments are pulled in here, a common LLVM technique for TableGen-produced code.
- **CN**: 它引入了 `MCInstrAnalysis.h`, `MCInstrInfo.h`, `MCRegisterInfo.h`, `MCStreamer.h`, `MCSubtargetInfo.h`, `TargetRegistry.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。 这里引入了生成的 `.inc` 片段，这是 LLVM 使用 TableGen 产物的常见方式。

### Lines 37-54 / 第 37-54 行
```cpp
  37: #define GET_REGINFO_MC_DESC
  38: #include "SystemZGenRegisterInfo.inc"
  39: 
  40: // Temporary option to assist with the migration to a new HLASMAsmStreamer on
  41: // z/OS
  42: static cl::opt<bool> GNUAsOnzOSCL("emit-gnuas-syntax-on-zos",
  43:                                   cl::desc("Emit GNU Assembly Syntax on z/OS."),
  44:                                   cl::init(false));
  45: 
  46: const unsigned SystemZMC::GR32Regs[16] = {
  47:     SystemZ::R0L,  SystemZ::R1L,  SystemZ::R2L,  SystemZ::R3L,
  48:     SystemZ::R4L,  SystemZ::R5L,  SystemZ::R6L,  SystemZ::R7L,
  49:     SystemZ::R8L,  SystemZ::R9L,  SystemZ::R10L, SystemZ::R11L,
  50:     SystemZ::R12L, SystemZ::R13L, SystemZ::R14L, SystemZ::R15L};
  51: 
  52: const unsigned SystemZMC::GRH32Regs[16] = {
  53:     SystemZ::R0H,  SystemZ::R1H,  SystemZ::R2H,  SystemZ::R3H,
  54:     SystemZ::R4H,  SystemZ::R5H,  SystemZ::R6H,  SystemZ::R7H,
```
- **EN**: It imports dependencies such as `SystemZGenRegisterInfo.inc` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file. Generated `.inc` fragments are pulled in here, a common LLVM technique for TableGen-produced code.
- **CN**: 它引入了 `SystemZGenRegisterInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。 这里引入了生成的 `.inc` 片段，这是 LLVM 使用 TableGen 产物的常见方式。

### Lines 55-72 / 第 55-72 行
```cpp
  55:     SystemZ::R8H,  SystemZ::R9H,  SystemZ::R10H, SystemZ::R11H,
  56:     SystemZ::R12H, SystemZ::R13H, SystemZ::R14H, SystemZ::R15H};
  57: 
  58: const unsigned SystemZMC::GR64Regs[16] = {
  59:     SystemZ::R0D,  SystemZ::R1D,  SystemZ::R2D,  SystemZ::R3D,
  60:     SystemZ::R4D,  SystemZ::R5D,  SystemZ::R6D,  SystemZ::R7D,
  61:     SystemZ::R8D,  SystemZ::R9D,  SystemZ::R10D, SystemZ::R11D,
  62:     SystemZ::R12D, SystemZ::R13D, SystemZ::R14D, SystemZ::R15D};
  63: 
  64: const unsigned SystemZMC::GR128Regs[16] = {
  65:     SystemZ::R0Q, 0, SystemZ::R2Q,  0, SystemZ::R4Q,  0, SystemZ::R6Q,  0,
  66:     SystemZ::R8Q, 0, SystemZ::R10Q, 0, SystemZ::R12Q, 0, SystemZ::R14Q, 0};
  67: 
  68: const unsigned SystemZMC::FP16Regs[16] = {
  69:     SystemZ::F0H,  SystemZ::F1H,  SystemZ::F2H,  SystemZ::F3H,
  70:     SystemZ::F4H,  SystemZ::F5H,  SystemZ::F6H,  SystemZ::F7H,
  71:     SystemZ::F8H,  SystemZ::F9H,  SystemZ::F10H, SystemZ::F11H,
  72:     SystemZ::F12H, SystemZ::F13H, SystemZ::F14H, SystemZ::F15H};
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73: 
  74: const unsigned SystemZMC::FP32Regs[16] = {
  75:     SystemZ::F0S,  SystemZ::F1S,  SystemZ::F2S,  SystemZ::F3S,
  76:     SystemZ::F4S,  SystemZ::F5S,  SystemZ::F6S,  SystemZ::F7S,
  77:     SystemZ::F8S,  SystemZ::F9S,  SystemZ::F10S, SystemZ::F11S,
  78:     SystemZ::F12S, SystemZ::F13S, SystemZ::F14S, SystemZ::F15S};
  79: 
  80: const unsigned SystemZMC::FP64Regs[16] = {
  81:     SystemZ::F0D,  SystemZ::F1D,  SystemZ::F2D,  SystemZ::F3D,
  82:     SystemZ::F4D,  SystemZ::F5D,  SystemZ::F6D,  SystemZ::F7D,
  83:     SystemZ::F8D,  SystemZ::F9D,  SystemZ::F10D, SystemZ::F11D,
  84:     SystemZ::F12D, SystemZ::F13D, SystemZ::F14D, SystemZ::F15D};
  85: 
  86: const unsigned SystemZMC::FP128Regs[16] = {
  87:     SystemZ::F0Q, SystemZ::F1Q, 0, 0, SystemZ::F4Q,  SystemZ::F5Q,  0, 0,
  88:     SystemZ::F8Q, SystemZ::F9Q, 0, 0, SystemZ::F12Q, SystemZ::F13Q, 0, 0};
  89: 
  90: const unsigned SystemZMC::VR16Regs[32] = {
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 91-108 / 第 91-108 行
```cpp
  91:     SystemZ::F0H,  SystemZ::F1H,  SystemZ::F2H,  SystemZ::F3H,  SystemZ::F4H,
  92:     SystemZ::F5H,  SystemZ::F6H,  SystemZ::F7H,  SystemZ::F8H,  SystemZ::F9H,
  93:     SystemZ::F10H, SystemZ::F11H, SystemZ::F12H, SystemZ::F13H, SystemZ::F14H,
  94:     SystemZ::F15H, SystemZ::F16H, SystemZ::F17H, SystemZ::F18H, SystemZ::F19H,
  95:     SystemZ::F20H, SystemZ::F21H, SystemZ::F22H, SystemZ::F23H, SystemZ::F24H,
  96:     SystemZ::F25H, SystemZ::F26H, SystemZ::F27H, SystemZ::F28H, SystemZ::F29H,
  97:     SystemZ::F30H, SystemZ::F31H};
  98: 
  99: const unsigned SystemZMC::VR32Regs[32] = {
 100:     SystemZ::F0S,  SystemZ::F1S,  SystemZ::F2S,  SystemZ::F3S,  SystemZ::F4S,
 101:     SystemZ::F5S,  SystemZ::F6S,  SystemZ::F7S,  SystemZ::F8S,  SystemZ::F9S,
 102:     SystemZ::F10S, SystemZ::F11S, SystemZ::F12S, SystemZ::F13S, SystemZ::F14S,
 103:     SystemZ::F15S, SystemZ::F16S, SystemZ::F17S, SystemZ::F18S, SystemZ::F19S,
 104:     SystemZ::F20S, SystemZ::F21S, SystemZ::F22S, SystemZ::F23S, SystemZ::F24S,
 105:     SystemZ::F25S, SystemZ::F26S, SystemZ::F27S, SystemZ::F28S, SystemZ::F29S,
 106:     SystemZ::F30S, SystemZ::F31S};
 107: 
 108: const unsigned SystemZMC::VR64Regs[32] = {
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 109-126 / 第 109-126 行
```cpp
 109:     SystemZ::F0D,  SystemZ::F1D,  SystemZ::F2D,  SystemZ::F3D,  SystemZ::F4D,
 110:     SystemZ::F5D,  SystemZ::F6D,  SystemZ::F7D,  SystemZ::F8D,  SystemZ::F9D,
 111:     SystemZ::F10D, SystemZ::F11D, SystemZ::F12D, SystemZ::F13D, SystemZ::F14D,
 112:     SystemZ::F15D, SystemZ::F16D, SystemZ::F17D, SystemZ::F18D, SystemZ::F19D,
 113:     SystemZ::F20D, SystemZ::F21D, SystemZ::F22D, SystemZ::F23D, SystemZ::F24D,
 114:     SystemZ::F25D, SystemZ::F26D, SystemZ::F27D, SystemZ::F28D, SystemZ::F29D,
 115:     SystemZ::F30D, SystemZ::F31D};
 116: 
 117: const unsigned SystemZMC::VR128Regs[32] = {
 118:     SystemZ::V0,  SystemZ::V1,  SystemZ::V2,  SystemZ::V3,  SystemZ::V4,
 119:     SystemZ::V5,  SystemZ::V6,  SystemZ::V7,  SystemZ::V8,  SystemZ::V9,
 120:     SystemZ::V10, SystemZ::V11, SystemZ::V12, SystemZ::V13, SystemZ::V14,
 121:     SystemZ::V15, SystemZ::V16, SystemZ::V17, SystemZ::V18, SystemZ::V19,
 122:     SystemZ::V20, SystemZ::V21, SystemZ::V22, SystemZ::V23, SystemZ::V24,
 123:     SystemZ::V25, SystemZ::V26, SystemZ::V27, SystemZ::V28, SystemZ::V29,
 124:     SystemZ::V30, SystemZ::V31};
 125: 
 126: const unsigned SystemZMC::AR32Regs[16] = {
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 127-144 / 第 127-144 行
```cpp
 127:     SystemZ::A0,  SystemZ::A1,  SystemZ::A2,  SystemZ::A3,
 128:     SystemZ::A4,  SystemZ::A5,  SystemZ::A6,  SystemZ::A7,
 129:     SystemZ::A8,  SystemZ::A9,  SystemZ::A10, SystemZ::A11,
 130:     SystemZ::A12, SystemZ::A13, SystemZ::A14, SystemZ::A15};
 131: 
 132: const unsigned SystemZMC::CR64Regs[16] = {
 133:     SystemZ::C0,  SystemZ::C1,  SystemZ::C2,  SystemZ::C3,
 134:     SystemZ::C4,  SystemZ::C5,  SystemZ::C6,  SystemZ::C7,
 135:     SystemZ::C8,  SystemZ::C9,  SystemZ::C10, SystemZ::C11,
 136:     SystemZ::C12, SystemZ::C13, SystemZ::C14, SystemZ::C15};
 137: 
 138: unsigned SystemZMC::getFirstReg(unsigned Reg) {
 139:   static unsigned Map[SystemZ::NUM_TARGET_REGS];
 140:   static bool Initialized = false;
 141:   if (!Initialized) {
 142:     for (unsigned I = 0; I < 16; ++I) {
 143:       Map[GR32Regs[I]] = I;
 144:       Map[GRH32Regs[I]] = I;
```
- **EN**: The range implements or declares functions including `SystemZMC::getFirstReg`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZMC::getFirstReg` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 145-162 / 第 145-162 行
```cpp
 145:       Map[GR64Regs[I]] = I;
 146:       Map[GR128Regs[I]] = I;
 147:       Map[FP128Regs[I]] = I;
 148:       Map[AR32Regs[I]] = I;
 149:     }
 150:     for (unsigned I = 0; I < 32; ++I) {
 151:       Map[VR16Regs[I]] = I;
 152:       Map[VR32Regs[I]] = I;
 153:       Map[VR64Regs[I]] = I;
 154:       Map[VR128Regs[I]] = I;
 155:     }
 156:   }
 157:   assert(Reg < SystemZ::NUM_TARGET_REGS);
 158:   return Map[Reg];
 159: }
 160: 
 161: static MCAsmInfo *createSystemZMCAsmInfo(const MCRegisterInfo &MRI,
 162:                                          const Triple &TT,
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 163-180 / 第 163-180 行
```cpp
 163:                                          const MCTargetOptions &Options) {
 164:   if (TT.isOSzOS())
 165:     return new SystemZMCAsmInfoGOFF(TT, Options);
 166: 
 167:   MCAsmInfo *MAI = new SystemZMCAsmInfoELF(TT, Options);
 168:   MCCFIInstruction Inst = MCCFIInstruction::cfiDefCfa(
 169:       nullptr, MRI.getDwarfRegNum(SystemZ::R15D, true),
 170:       SystemZMC::ELFCFAOffsetFromInitialSP);
 171:   MAI->addInitialFrameState(Inst);
 172:   return MAI;
 173: }
 174: 
 175: static MCInstrInfo *createSystemZMCInstrInfo() {
 176:   MCInstrInfo *X = new MCInstrInfo();
 177:   InitSystemZMCInstrInfo(X);
 178:   return X;
 179: }
 180: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181: static MCRegisterInfo *createSystemZMCRegisterInfo(const Triple &TT) {
 182:   MCRegisterInfo *X = new MCRegisterInfo();
 183:   InitSystemZMCRegisterInfo(X, SystemZ::R14D);
 184:   return X;
 185: }
 186: 
 187: static MCSubtargetInfo *
 188: createSystemZMCSubtargetInfo(const Triple &TT, StringRef CPU, StringRef FS) {
 189:   return createSystemZMCSubtargetInfoImpl(TT, CPU, /*TuneCPU*/ CPU, FS);
 190: }
 191: 
 192: static MCInstPrinter *createSystemZMCInstPrinter(const Triple &T,
 193:                                                  unsigned SyntaxVariant,
 194:                                                  const MCAsmInfo &MAI,
 195:                                                  const MCInstrInfo &MII,
 196:                                                  const MCRegisterInfo &MRI) {
 197:   if (SyntaxVariant == AD_HLASM)
 198:     return new SystemZHLASMInstPrinter(MAI, MII, MRI);
```
- **EN**: The range implements or declares functions including `createSystemZMCSubtargetInfo`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `createSystemZMCSubtargetInfo` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-216 / 第 199-216 行
```cpp
 199: 
 200:   return new SystemZGNUInstPrinter(MAI, MII, MRI);
 201: }
 202: 
 203: static MCTargetStreamer *createAsmTargetStreamer(MCStreamer &S,
 204:                                                  formatted_raw_ostream &OS,
 205:                                                  MCInstPrinter *InstPrint) {
 206:   if (S.getContext().getTargetTriple().isOSzOS() && !GNUAsOnzOSCL)
 207:     return new SystemZTargetHLASMStreamer(S, OS);
 208:   else
 209:     return new SystemZTargetGNUStreamer(S, OS);
 210: }
 211: 
 212: static MCStreamer *createSystemZAsmStreamer(
 213:     MCContext &Ctx, std::unique_ptr<formatted_raw_ostream> OS,
 214:     std::unique_ptr<MCInstPrinter> IP, std::unique_ptr<MCCodeEmitter> CE,
 215:     std::unique_ptr<MCAsmBackend> TAB) {
 216: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 217-234 / 第 217-234 行
```cpp
 217:   auto TT = Ctx.getTargetTriple();
 218:   if (TT.isOSzOS() && !GNUAsOnzOSCL)
 219:     return new SystemZHLASMAsmStreamer(Ctx, std::move(OS), std::move(IP),
 220:                                        std::move(CE), std::move(TAB));
 221: 
 222:   return llvm::createAsmStreamer(Ctx, std::move(OS), std::move(IP),
 223:                                  std::move(CE), std::move(TAB));
 224: }
 225: 
 226: static MCTargetStreamer *
 227: createObjectTargetStreamer(MCStreamer &S, const MCSubtargetInfo &STI) {
 228:   if (S.getContext().getTargetTriple().isOSzOS())
 229:     return new SystemZTargetGOFFStreamer(S);
 230:   else
 231:     return new SystemZTargetELFStreamer(S);
 232: }
 233: 
 234: static MCTargetStreamer *
```
- **EN**: The range implements or declares functions including `createObjectTargetStreamer`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `createObjectTargetStreamer` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235: createNullTargetStreamer(MCStreamer &S) {
 236:   return new SystemZTargetStreamer(S);
 237: }
 238: 
 239: static MCInstrAnalysis *createSystemZMCInstrAnalysis(const MCInstrInfo *Info) {
 240:   return new MCInstrAnalysis(Info);
 241: }
 242: 
 243: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
 244: LLVMInitializeSystemZTargetMC() {
 245:   // Register the MCAsmInfo.
 246:   TargetRegistry::RegisterMCAsmInfo(getTheSystemZTarget(),
 247:                                     createSystemZMCAsmInfo);
 248: 
 249:   // Register the MCCodeEmitter.
 250:   TargetRegistry::RegisterMCCodeEmitter(getTheSystemZTarget(),
 251:                                         createSystemZMCCodeEmitter);
 252: 
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 253-270 / 第 253-270 行
```cpp
 253:   // Register the MCInstrInfo.
 254:   TargetRegistry::RegisterMCInstrInfo(getTheSystemZTarget(),
 255:                                       createSystemZMCInstrInfo);
 256: 
 257:   // Register the MCRegisterInfo.
 258:   TargetRegistry::RegisterMCRegInfo(getTheSystemZTarget(),
 259:                                     createSystemZMCRegisterInfo);
 260: 
 261:   // Register the MCSubtargetInfo.
 262:   TargetRegistry::RegisterMCSubtargetInfo(getTheSystemZTarget(),
 263:                                           createSystemZMCSubtargetInfo);
 264: 
 265:   // Register the MCAsmBackend.
 266:   TargetRegistry::RegisterMCAsmBackend(getTheSystemZTarget(),
 267:                                        createSystemZMCAsmBackend);
 268: 
 269:   // Register the MCInstPrinter.
 270:   TargetRegistry::RegisterMCInstPrinter(getTheSystemZTarget(),
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 271-288 / 第 271-288 行
```cpp
 271:                                         createSystemZMCInstPrinter);
 272: 
 273:   // Register the asm streamer.
 274:   TargetRegistry::RegisterAsmStreamer(getTheSystemZTarget(),
 275:                                       createSystemZAsmStreamer);
 276: 
 277:   // Register the asm target streamer.
 278:   TargetRegistry::RegisterAsmTargetStreamer(getTheSystemZTarget(),
 279:                                             createAsmTargetStreamer);
 280: 
 281:   // Register the obj streamer
 282:   TargetRegistry::RegisterObjectTargetStreamer(getTheSystemZTarget(),
 283:                                                createObjectTargetStreamer);
 284: 
 285:   // Register the null streamer
 286:   TargetRegistry::RegisterNullTargetStreamer(getTheSystemZTarget(),
 287:                                              createNullTargetStreamer);
 288: 
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 289-292 / 第 289-292 行
```cpp
 289:   // Register the MCInstrAnalysis.
 290:   TargetRegistry::RegisterMCInstrAnalysis(getTheSystemZTarget(),
 291:                                           createSystemZMCInstrAnalysis);
 292: }
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **MC streaming**: Emits sections, symbols, expressions, and encoded bytes to object or assembly output. / 向目标文件或汇编输出节、符号、表达式和编码字节。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `SystemZMCTargetDesc.h`
- `SystemZGNUInstPrinter.h`
- `SystemZHLASMAsmStreamer.h`
- `SystemZHLASMInstPrinter.h`
- `SystemZMCAsmInfo.h`
- `SystemZTargetStreamer.h`
- `TargetInfo/SystemZTargetInfo.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCDwarf.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCInstrAnalysis.h`
- `llvm/MC/MCInstrInfo.h`
- `llvm/MC/MCRegisterInfo.h`
- `llvm/MC/MCStreamer.h`
- `llvm/MC/MCSubtargetInfo.h`
- `llvm/MC/TargetRegistry.h`
- `...` (5 more include dependencies omitted for brevity / 其余 5 个 include 依赖已省略)
