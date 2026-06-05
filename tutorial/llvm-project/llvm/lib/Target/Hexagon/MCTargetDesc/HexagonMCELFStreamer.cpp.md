# HexagonMCELFStreamer.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonMCELFStreamer.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file is a stub that parses a MCInst bundle and passes the
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //=== HexagonMCELFStreamer.cpp - Hexagon subclass of MCELFStreamer -------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file is a stub that parses a MCInst bundle and passes the
    10: // instructions on to the real streamer.
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #include "MCTargetDesc/HexagonMCELFStreamer.h"
    15: #include "HexagonTargetStreamer.h"
    16: #include "MCTargetDesc/HexagonMCChecker.h"
    17: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    18: #include "MCTargetDesc/HexagonMCShuffler.h"
    19: #include "llvm/ADT/StringRef.h"
    20: #include "llvm/BinaryFormat/ELF.h"
    21: #include "llvm/MC/MCAsmBackend.h"
    22: #include "llvm/MC/MCAssembler.h"
    23: #include "llvm/MC/MCCodeEmitter.h"
    24: #include "llvm/MC/MCContext.h"
    25: #include "llvm/MC/MCExpr.h"
    26: #include "llvm/MC/MCInst.h"
    27: #include "llvm/MC/MCObjectStreamer.h"
    28: #include "llvm/MC/MCObjectWriter.h"
    29: #include "llvm/MC/MCSection.h"
    30: #include "llvm/MC/MCSectionELF.h"
    31: #include "llvm/MC/MCStreamer.h"
    32: #include "llvm/MC/MCSubtargetInfo.h"
    33: #include "llvm/MC/MCSymbol.h"
    34: #include "llvm/MC/MCSymbolELF.h"
    35: #include "llvm/Support/CommandLine.h"
    36: #include "llvm/Support/ErrorHandling.h"
    37: #include "llvm/Support/HexagonAttributes.h"
    38: #include "llvm/Support/MathExtras.h"
    39: #include <cassert>
    40: #include <cstdint>
    41: 
    42: #define DEBUG_TYPE "hexagonmcelfstreamer"
    43: 
    44: using namespace llvm;
    45: 
    46: static cl::opt<unsigned> GPSize
    47:   ("gpsize", cl::NotHidden,
    48:    cl::desc("Global Pointer Addressing Size.  The default size is 8."),
    49:    cl::Prefix,
    50:    cl::init(8));
```
- EN: It imports headers such as MCTargetDesc/HexagonMCELFStreamer.h, HexagonTargetStreamer.h, MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonMCInstrInfo.h, ... (27 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as GPSize, translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 MCTargetDesc/HexagonMCELFStreamer.h, HexagonTargetStreamer.h, MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonMCInstrInfo.h, ... (27 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 GPSize 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-100 / 第 51-100 行

```cpp
    51: 
    52: HexagonMCELFStreamer::HexagonMCELFStreamer(
    53:     MCContext &Context, std::unique_ptr<MCAsmBackend> TAB,
    54:     std::unique_ptr<MCObjectWriter> OW, std::unique_ptr<MCCodeEmitter> Emitter)
    55:     : MCELFStreamer(Context, std::move(TAB), std::move(OW), std::move(Emitter)),
    56:       MCII(createHexagonMCInstrInfo()) {}
    57: 
    58: HexagonMCELFStreamer::HexagonMCELFStreamer(
    59:     MCContext &Context, std::unique_ptr<MCAsmBackend> TAB,
    60:     std::unique_ptr<MCObjectWriter> OW, std::unique_ptr<MCCodeEmitter> Emitter,
    61:     MCAssembler *Assembler)
    62:     : MCELFStreamer(Context, std::move(TAB), std::move(OW), std::move(Emitter)),
    63:       MCII(createHexagonMCInstrInfo()) {}
    64: 
    65: void HexagonMCELFStreamer::emitInstruction(const MCInst &MCB,
    66:                                            const MCSubtargetInfo &STI) {
    67:   assert(MCB.getOpcode() == Hexagon::BUNDLE);
    68:   assert(HexagonMCInstrInfo::bundleSize(MCB) <= HEXAGON_PACKET_SIZE);
    69:   assert(HexagonMCInstrInfo::bundleSize(MCB) > 0);
    70: 
    71:   // At this point, MCB is a bundle
    72:   // Iterate through the bundle and assign addends for the instructions
    73:   for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCB)) {
    74:     MCInst *MCI = const_cast<MCInst *>(I.getInst());
    75:     EmitSymbol(*MCI);
    76:   }
    77: 
    78:   MCObjectStreamer::emitInstruction(MCB, STI);
    79: }
    80: 
    81: void HexagonMCELFStreamer::EmitSymbol(const MCInst &Inst) {
    82:   // Scan for values.
    83:   for (unsigned i = Inst.getNumOperands(); i--;)
    84:     if (Inst.getOperand(i).isExpr())
    85:       visitUsedExpr(*Inst.getOperand(i).getExpr());
    86: }
    87: 
    88: // EmitCommonSymbol and EmitLocalCommonSymbol are extended versions of the
    89: // functions found in MCELFStreamer.cpp taking AccessSize as an additional
    90: // parameter.
    91: void HexagonMCELFStreamer::HexagonMCEmitCommonSymbol(MCSymbol *Symbol,
    92:                                                      uint64_t Size,
    93:                                                      Align ByteAlignment,
    94:                                                      unsigned AccessSize) {
    95:   getAssembler().registerSymbol(*Symbol);
    96:   StringRef sbss[4] = {".sbss.1", ".sbss.2", ".sbss.4", ".sbss.8"};
    97: 
    98:   auto ELFSymbol = static_cast<MCSymbolELF *>(Symbol);
    99:   if (!ELFSymbol->isBindingSet())
   100:     ELFSymbol->setBinding(ELF::STB_GLOBAL);
```
- EN: It declares or implements routines such as HexagonMCELFStreamer::HexagonMCELFStreamer, HexagonMCELFStreamer::emitInstruction, assert, getInst, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCELFStreamer, HexagonMCInstrInfo, HexagonMCEmitCommonSymbol, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCELFStreamer::HexagonMCELFStreamer, HexagonMCELFStreamer::emitInstruction, assert, getInst, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCELFStreamer, HexagonMCInstrInfo, HexagonMCEmitCommonSymbol，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101: 
   102:   ELFSymbol->setType(ELF::STT_OBJECT);
   103: 
   104:   if (ELFSymbol->getBinding() == ELF::STB_LOCAL) {
   105:     StringRef SectionName =
   106:         ((AccessSize == 0) || (Size == 0) || (Size > GPSize))
   107:             ? ".bss"
   108:             : sbss[(Log2_64(AccessSize))];
   109:     MCSection &Section = *getAssembler().getContext().getELFSection(
   110:         SectionName, ELF::SHT_NOBITS, ELF::SHF_WRITE | ELF::SHF_ALLOC);
   111:     MCSectionSubPair P = getCurrentSection();
   112:     switchSection(&Section);
   113: 
   114:     if (ELFSymbol->isUndefined()) {
   115:       emitValueToAlignment(ByteAlignment, 0, 1, 0);
   116:       emitLabel(Symbol);
   117:       emitZeros(Size);
   118:     }
   119: 
   120:     // Update the maximum alignment of the section if necessary.
   121:     Section.ensureMinAlignment(ByteAlignment);
   122: 
   123:     switchSection(P.first, P.second);
   124:   } else {
   125:     if (ELFSymbol->declareCommon(Size, ByteAlignment))
   126:       report_fatal_error("Symbol: " + Symbol->getName() +
   127:                          " redeclared as different type");
   128:     if ((AccessSize) && (Size <= GPSize)) {
   129:       uint64_t SectionIndex =
   130:           (AccessSize <= GPSize)
   131:               ? ELF::SHN_HEXAGON_SCOMMON + llvm::bit_width(AccessSize)
   132:               : (unsigned)ELF::SHN_HEXAGON_SCOMMON;
   133:       ELFSymbol->setIndex(SectionIndex);
   134:     }
   135:   }
   136: 
   137:   ELFSymbol->setSize(MCConstantExpr::create(Size, getContext()));
   138: }
   139: 
   140: void HexagonMCELFStreamer::HexagonMCEmitLocalCommonSymbol(MCSymbol *Symbol,
   141:                                                           uint64_t Size,
   142:                                                           Align ByteAlignment,
   143:                                                           unsigned AccessSize) {
   144:   getAssembler().registerSymbol(*Symbol);
   145:   auto ELFSymbol = static_cast<const MCSymbolELF *>(Symbol);
   146:   ELFSymbol->setBinding(ELF::STB_LOCAL);
   147:   HexagonMCEmitCommonSymbol(Symbol, Size, ByteAlignment, AccessSize);
   148: }
   149: 
   150: static unsigned featureToArchVersion(unsigned Feature) {
```
- EN: It declares or implements routines such as setType, getAssembler, getCurrentSection, switchSection, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCELFStreamer, HexagonMCEmitLocalCommonSymbol, HexagonMCEmitCommonSymbol, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 setType, getAssembler, getCurrentSection, switchSection, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCELFStreamer, HexagonMCEmitLocalCommonSymbol, HexagonMCEmitCommonSymbol，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   switch (Feature) {
   152:   case Hexagon::ArchV5:
   153:     return 5;
   154:   case Hexagon::ArchV55:
   155:     return 55;
   156:   case Hexagon::ArchV60:
   157:   case Hexagon::ExtensionHVXV60:
   158:     return 60;
   159:   case Hexagon::ArchV62:
   160:   case Hexagon::ExtensionHVXV62:
   161:     return 62;
   162:   case Hexagon::ArchV65:
   163:   case Hexagon::ExtensionHVXV65:
   164:     return 65;
   165:   case Hexagon::ArchV66:
   166:   case Hexagon::ExtensionHVXV66:
   167:     return 66;
   168:   case Hexagon::ArchV67:
   169:   case Hexagon::ExtensionHVXV67:
   170:     return 67;
   171:   case Hexagon::ArchV68:
   172:   case Hexagon::ExtensionHVXV68:
   173:     return 68;
   174:   case Hexagon::ArchV69:
   175:   case Hexagon::ExtensionHVXV69:
   176:     return 69;
   177:   case Hexagon::ArchV71:
   178:   case Hexagon::ExtensionHVXV71:
   179:     return 71;
   180:   case Hexagon::ArchV73:
   181:   case Hexagon::ExtensionHVXV73:
   182:     return 73;
   183:   case Hexagon::ArchV75:
   184:   case Hexagon::ExtensionHVXV75:
   185:     return 75;
   186:   case Hexagon::ArchV79:
   187:   case Hexagon::ExtensionHVXV79:
   188:     return 79;
   189:   case Hexagon::ArchV81:
   190:   case Hexagon::ExtensionHVXV81:
   191:     return 81;
   192:   }
   193:   llvm_unreachable("Expected valid arch feature");
   194:   return 0;
   195: }
   196: 
   197: void HexagonTargetStreamer::emitTargetAttributes(const MCSubtargetInfo &STI) {
   198:   auto Features = STI.getFeatureBits();
   199:   unsigned Arch = featureToArchVersion(Hexagon_MC::getArchVersion(Features));
   200:   std::optional<unsigned> HVXArch = Hexagon_MC::getHVXVersion(Features);
```
- EN: It declares or implements routines such as llvm_unreachable, HexagonTargetStreamer::emitTargetAttributes, getFeatureBits, featureToArchVersion, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTargetStreamer, Hexagon_MC, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm_unreachable, HexagonTargetStreamer::emitTargetAttributes, getFeatureBits, featureToArchVersion, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTargetStreamer, Hexagon_MC，说明了它与同级后端组件的连接关系。

### Lines 201-225 / 第 201-225 行

```cpp
   201:   emitAttribute(HexagonAttrs::ARCH, Arch);
   202:   if (HVXArch)
   203:     emitAttribute(HexagonAttrs::HVXARCH, featureToArchVersion(*HVXArch));
   204:   if (Features.test(Hexagon::ExtensionHVXIEEEFP))
   205:     emitAttribute(HexagonAttrs::HVXIEEEFP, 1);
   206:   if (Features.test(Hexagon::ExtensionHVXQFloat))
   207:     emitAttribute(HexagonAttrs::HVXQFLOAT, 1);
   208:   if (Features.test(Hexagon::ExtensionZReg))
   209:     emitAttribute(HexagonAttrs::ZREG, 1);
   210:   if (Features.test(Hexagon::ExtensionAudio))
   211:     emitAttribute(HexagonAttrs::AUDIO, 1);
   212:   if (Features.test(Hexagon::FeatureCabac))
   213:     emitAttribute(HexagonAttrs::CABAC, 1);
   214: }
   215: 
   216: namespace llvm {
   217: MCStreamer *createHexagonELFStreamer(Triple const &TT, MCContext &Context,
   218:                                      std::unique_ptr<MCAsmBackend> MAB,
   219:                                      std::unique_ptr<MCObjectWriter> OW,
   220:                                      std::unique_ptr<MCCodeEmitter> CE) {
   221:   return new HexagonMCELFStreamer(Context, std::move(MAB), std::move(OW),
   222:                                   std::move(CE));
   223:   }
   224: 
   225: } // end namespace llvm
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as emitAttribute, createHexagonELFStreamer, HexagonMCELFStreamer, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonAttrs, HexagonMCELFStreamer, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 emitAttribute, createHexagonELFStreamer, HexagonMCELFStreamer 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonAttrs, HexagonMCELFStreamer，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonMCELFStreamer.h, HexagonTargetStreamer.h, MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCShuffler.h, llvm/ADT/StringRef.h, llvm/BinaryFormat/ELF.h, llvm/MC/MCAsmBackend.h, llvm/MC/MCAssembler.h, llvm/MC/MCCodeEmitter.h, ... (27 total)`
- Hexagon symbols / Hexagon 符号: `HexagonMCELFStreamer, HexagonTargetStreamer, HexagonMCChecker, HexagonMCInstrInfo, HexagonMCShuffler, HexagonAttributes, HexagonMCEmitCommonSymbol, HexagonMCEmitLocalCommonSymbol, Hexagon_MC, HexagonAttrs`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
