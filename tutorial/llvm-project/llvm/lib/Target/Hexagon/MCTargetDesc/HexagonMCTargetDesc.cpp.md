# HexagonMCTargetDesc.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonMCTargetDesc.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file provides Hexagon specific target descriptions.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===-- HexagonMCTargetDesc.cpp - Hexagon Target Descriptions -------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file provides Hexagon specific target descriptions.
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    14: #include "HexagonDepArch.h"
    15: #include "HexagonTargetStreamer.h"
    16: #include "MCTargetDesc/HexagonInstPrinter.h"
    17: #include "MCTargetDesc/HexagonMCAsmInfo.h"
    18: #include "MCTargetDesc/HexagonMCELFStreamer.h"
    19: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    20: #include "TargetInfo/HexagonTargetInfo.h"
    21: #include "llvm/ADT/StringExtras.h"
    22: #include "llvm/ADT/StringRef.h"
    23: #include "llvm/BinaryFormat/ELF.h"
    24: #include "llvm/MC/MCAsmBackend.h"
    25: #include "llvm/MC/MCAssembler.h"
    26: #include "llvm/MC/MCCodeEmitter.h"
    27: #include "llvm/MC/MCContext.h"
    28: #include "llvm/MC/MCDwarf.h"
    29: #include "llvm/MC/MCELFObjectWriter.h"
    30: #include "llvm/MC/MCELFStreamer.h"
    31: #include "llvm/MC/MCInstrAnalysis.h"
    32: #include "llvm/MC/MCInstrInfo.h"
    33: #include "llvm/MC/MCRegisterInfo.h"
    34: #include "llvm/MC/MCStreamer.h"
    35: #include "llvm/MC/MCSubtargetInfo.h"
    36: #include "llvm/MC/TargetRegistry.h"
    37: #include "llvm/Support/Compiler.h"
    38: #include "llvm/Support/ErrorHandling.h"
    39: #include "llvm/Support/HexagonAttributes.h"
    40: #include "llvm/Support/raw_ostream.h"
    41: #include <cassert>
    42: #include <cstdint>
    43: #include <mutex>
    44: #include <new>
    45: #include <string>
    46: #include <unordered_map>
    47: 
    48: using namespace llvm;
    49: 
    50: #define GET_INSTRINFO_MC_DESC
    51: #define ENABLE_INSTR_PREDICATE_VERIFIER
    52: #include "HexagonGenInstrInfo.inc"
    53: 
    54: #define GET_SUBTARGETINFO_MC_DESC
    55: #include "HexagonGenSubtargetInfo.inc"
    56: 
    57: #define GET_REGINFO_MC_DESC
    58: #include "HexagonGenRegisterInfo.inc"
    59: 
    60: cl::opt<bool> llvm::HexagonDisableCompound
    61:   ("mno-compound",
    62:    cl::desc("Disable looking for compound instructions for Hexagon"));
    63: 
    64: cl::opt<bool> llvm::HexagonDisableDuplex
    65:   ("mno-pairing",
    66:    cl::desc("Disable looking for duplex instructions for Hexagon"));
    67: 
    68: namespace { // These flags are to be deprecated
    69: cl::opt<bool> MV5("mv5", cl::Hidden, cl::desc("Build for Hexagon V5"),
    70:                   cl::init(false));
    71: cl::opt<bool> MV55("mv55", cl::Hidden, cl::desc("Build for Hexagon V55"),
    72:                    cl::init(false));
    73: cl::opt<bool> MV60("mv60", cl::Hidden, cl::desc("Build for Hexagon V60"),
    74:                    cl::init(false));
    75: cl::opt<bool> MV62("mv62", cl::Hidden, cl::desc("Build for Hexagon V62"),
    76:                    cl::init(false));
    77: cl::opt<bool> MV65("mv65", cl::Hidden, cl::desc("Build for Hexagon V65"),
    78:                    cl::init(false));
    79: cl::opt<bool> MV66("mv66", cl::Hidden, cl::desc("Build for Hexagon V66"),
    80:                    cl::init(false));
    81: cl::opt<bool> MV67("mv67", cl::Hidden, cl::desc("Build for Hexagon V67"),
    82:                    cl::init(false));
    83: cl::opt<bool> MV67T("mv67t", cl::Hidden, cl::desc("Build for Hexagon V67T"),
    84:                     cl::init(false));
    85: cl::opt<bool> MV68("mv68", cl::Hidden, cl::desc("Build for Hexagon V68"),
    86:                    cl::init(false));
    87: cl::opt<bool> MV69("mv69", cl::Hidden, cl::desc("Build for Hexagon V69"),
    88:                    cl::init(false));
    89: cl::opt<bool> MV71("mv71", cl::Hidden, cl::desc("Build for Hexagon V71"),
    90:                    cl::init(false));
    91: cl::opt<bool> MV71T("mv71t", cl::Hidden, cl::desc("Build for Hexagon V71T"),
    92:                     cl::init(false));
    93: cl::opt<bool> MV73("mv73", cl::Hidden, cl::desc("Build for Hexagon V73"),
    94:                    cl::init(false));
    95: cl::opt<bool> MV75("mv75", cl::Hidden, cl::desc("Build for Hexagon V75"),
    96:                    cl::init(false));
    97: cl::opt<bool> MV79("mv79", cl::Hidden, cl::desc("Build for Hexagon V79"),
    98:                    cl::init(false));
    99: cl::opt<bool> MV81("mv81", cl::Hidden, cl::desc("Build for Hexagon V81"),
   100:                    cl::init(false));
```
- EN: It imports headers such as MCTargetDesc/HexagonMCTargetDesc.h, HexagonDepArch.h, HexagonTargetStreamer.h, MCTargetDesc/HexagonInstPrinter.h, ... (37 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as llvm::HexagonDisableCompound, llvm::HexagonDisableDuplex, MV5, MV55, ... (18 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 MCTargetDesc/HexagonMCTargetDesc.h, HexagonDepArch.h, HexagonTargetStreamer.h, MCTargetDesc/HexagonInstPrinter.h, ... (37 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 llvm::HexagonDisableCompound, llvm::HexagonDisableDuplex, MV5, MV55, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-200 / 第 101-200 行

```cpp
   101: } // namespace
   102: 
   103: static cl::opt<Hexagon::ArchEnum> EnableHVX(
   104:     "mhvx", cl::desc("Enable Hexagon Vector eXtensions"),
   105:     cl::values(clEnumValN(Hexagon::ArchEnum::V60, "v60", "Build for HVX v60"),
   106:                clEnumValN(Hexagon::ArchEnum::V62, "v62", "Build for HVX v62"),
   107:                clEnumValN(Hexagon::ArchEnum::V65, "v65", "Build for HVX v65"),
   108:                clEnumValN(Hexagon::ArchEnum::V66, "v66", "Build for HVX v66"),
   109:                clEnumValN(Hexagon::ArchEnum::V67, "v67", "Build for HVX v67"),
   110:                clEnumValN(Hexagon::ArchEnum::V68, "v68", "Build for HVX v68"),
   111:                clEnumValN(Hexagon::ArchEnum::V69, "v69", "Build for HVX v69"),
   112:                clEnumValN(Hexagon::ArchEnum::V71, "v71", "Build for HVX v71"),
   113:                clEnumValN(Hexagon::ArchEnum::V73, "v73", "Build for HVX v73"),
   114:                clEnumValN(Hexagon::ArchEnum::V75, "v75", "Build for HVX v75"),
   115:                clEnumValN(Hexagon::ArchEnum::V79, "v79", "Build for HVX v79"),
   116:                clEnumValN(Hexagon::ArchEnum::V81, "v81", "Build for HVX v81"),
   117:                // Sentinel for no value specified.
   118:                clEnumValN(Hexagon::ArchEnum::Generic, "", "")),
   119:     // Sentinel for flag not present.
   120:     cl::init(Hexagon::ArchEnum::NoArch), cl::ValueOptional);
   121: 
   122: static cl::opt<bool>
   123:   DisableHVX("mno-hvx", cl::Hidden,
   124:              cl::desc("Disable Hexagon Vector eXtensions"));
   125: 
   126: static cl::opt<bool>
   127:     EnableHvxIeeeFp("mhvx-ieee-fp", cl::Hidden,
   128:                     cl::desc("Enable HVX IEEE floating point extensions"));
   129: static cl::opt<bool> EnableHexagonCabac
   130:   ("mcabac", cl::desc("tbd"), cl::init(false));
   131: 
   132: static constexpr StringRef DefaultArch = "hexagonv68";
   133: 
   134: static StringRef HexagonGetArchVariant() {
   135:   if (MV5)
   136:     return "hexagonv5";
   137:   if (MV55)
   138:     return "hexagonv55";
   139:   if (MV60)
   140:     return "hexagonv60";
   141:   if (MV62)
   142:     return "hexagonv62";
   143:   if (MV65)
   144:     return "hexagonv65";
   145:   if (MV66)
   146:     return "hexagonv66";
   147:   if (MV67)
   148:     return "hexagonv67";
   149:   if (MV67T)
   150:     return "hexagonv67t";
   151:   if (MV68)
   152:     return "hexagonv68";
   153:   if (MV69)
   154:     return "hexagonv69";
   155:   if (MV71)
   156:     return "hexagonv71";
   157:   if (MV71T)
   158:     return "hexagonv71t";
   159:   if (MV73)
   160:     return "hexagonv73";
   161:   if (MV75)
   162:     return "hexagonv75";
   163:   if (MV79)
   164:     return "hexagonv79";
   165:   if (MV81)
   166:     return "hexagonv81";
   167: 
   168:   return "";
   169: }
   170: 
   171: StringRef Hexagon_MC::selectHexagonCPU(StringRef CPU) {
   172:   StringRef ArchV = HexagonGetArchVariant();
   173:   if (!ArchV.empty() && !CPU.empty()) {
   174:     // Tiny cores have a "t" suffix that is discarded when creating a secondary
   175:     // non-tiny subtarget.  See: addArchSubtarget
   176:     std::pair<StringRef, StringRef> ArchP = ArchV.split('t');
   177:     std::pair<StringRef, StringRef> CPUP = CPU.split('t');
   178:     if (ArchP.first != CPUP.first)
   179:       report_fatal_error("conflicting architectures specified.");
   180:     return CPU;
   181:   }
   182:   if (ArchV.empty()) {
   183:     if (CPU.empty())
   184:       CPU = DefaultArch;
   185:     return CPU;
   186:   }
   187:   return ArchV;
   188: }
   189: 
   190: unsigned llvm::HexagonGetLastSlot() { return HexagonItinerariesV5FU::SLOT3; }
   191: 
   192: unsigned llvm::HexagonConvertUnits(unsigned ItinUnits, unsigned *Lanes) {
   193:   enum {
   194:     CVI_NONE = 0,
   195:     CVI_XLANE = 1 << 0,
   196:     CVI_SHIFT = 1 << 1,
   197:     CVI_MPY0 = 1 << 2,
   198:     CVI_MPY1 = 1 << 3,
   199:     CVI_ZW = 1 << 4
   200:   };
```
- EN: It opens namespaces (static) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as EnableHVX, DisableHVX, EnableHvxIeeeFp, EnableHexagonCabac, ... (9 total), translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（static），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 EnableHVX, DisableHVX, EnableHvxIeeeFp, EnableHexagonCabac, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-300 / 第 201-300 行

```cpp
   201: 
   202:   if (ItinUnits == HexagonItinerariesV62FU::CVI_ALL ||
   203:       ItinUnits == HexagonItinerariesV62FU::CVI_ALL_NOMEM)
   204:     return (*Lanes = 4, CVI_XLANE);
   205:   else if (ItinUnits & HexagonItinerariesV62FU::CVI_MPY01 &&
   206:            ItinUnits & HexagonItinerariesV62FU::CVI_XLSHF)
   207:     return (*Lanes = 2, CVI_XLANE | CVI_MPY0);
   208:   else if (ItinUnits & HexagonItinerariesV62FU::CVI_MPY01)
   209:     return (*Lanes = 2, CVI_MPY0);
   210:   else if (ItinUnits & HexagonItinerariesV62FU::CVI_XLSHF)
   211:     return (*Lanes = 2, CVI_XLANE);
   212:   else if (ItinUnits & HexagonItinerariesV62FU::CVI_XLANE &&
   213:            ItinUnits & HexagonItinerariesV62FU::CVI_SHIFT &&
   214:            ItinUnits & HexagonItinerariesV62FU::CVI_MPY0 &&
   215:            ItinUnits & HexagonItinerariesV62FU::CVI_MPY1)
   216:     return (*Lanes = 1, CVI_XLANE | CVI_SHIFT | CVI_MPY0 | CVI_MPY1);
   217:   else if (ItinUnits & HexagonItinerariesV62FU::CVI_XLANE &&
   218:            ItinUnits & HexagonItinerariesV62FU::CVI_SHIFT)
   219:     return (*Lanes = 1, CVI_XLANE | CVI_SHIFT);
   220:   else if (ItinUnits & HexagonItinerariesV62FU::CVI_MPY0 &&
   221:            ItinUnits & HexagonItinerariesV62FU::CVI_MPY1)
   222:     return (*Lanes = 1, CVI_MPY0 | CVI_MPY1);
   223:   else if (ItinUnits == HexagonItinerariesV62FU::CVI_ZW)
   224:     return (*Lanes = 1, CVI_ZW);
   225:   else if (ItinUnits == HexagonItinerariesV62FU::CVI_XLANE)
   226:     return (*Lanes = 1, CVI_XLANE);
   227:   else if (ItinUnits == HexagonItinerariesV62FU::CVI_SHIFT)
   228:     return (*Lanes = 1, CVI_SHIFT);
   229: 
   230:   return (*Lanes = 0, CVI_NONE);
   231: }
   232: 
   233: 
   234: namespace llvm {
   235: namespace HexagonFUnits {
   236: bool isSlot0Only(unsigned units) {
   237:   return HexagonItinerariesV62FU::SLOT0 == units;
   238: }
   239: } // namespace HexagonFUnits
   240: } // namespace llvm
   241: 
   242: namespace {
   243: 
   244: class HexagonTargetAsmStreamer : public HexagonTargetStreamer {
   245:   formatted_raw_ostream &OS;
   246: 
   247: public:
   248:   HexagonTargetAsmStreamer(MCStreamer &S, formatted_raw_ostream &OS,
   249:                            MCInstPrinter &IP)
   250:       : HexagonTargetStreamer(S), OS(OS) {}
   251: 
   252:   void prettyPrintAsm(MCInstPrinter &InstPrinter, uint64_t Address,
   253:                       const MCInst &Inst, const MCSubtargetInfo &STI,
   254:                       raw_ostream &OS) override {
   255:     assert(HexagonMCInstrInfo::isBundle(Inst));
   256:     assert(HexagonMCInstrInfo::bundleSize(Inst) <= HEXAGON_PACKET_SIZE);
   257:     std::string Buffer;
   258:     {
   259:       raw_string_ostream TempStream(Buffer);
   260:       for (auto &I : HexagonMCInstrInfo::bundleInstructions(Inst)) {
   261:         InstPrinter.printInst(I.getInst(), Address, "", STI, TempStream);
   262:         TempStream << "\n";
   263:       }
   264:     }
   265: 
   266:     std::string LoopString = "";
   267:     bool IsLoop0 = HexagonMCInstrInfo::isInnerLoop(Inst);
   268:     bool IsLoop1 = HexagonMCInstrInfo::isOuterLoop(Inst);
   269:     if (IsLoop0) {
   270:       LoopString += (IsLoop1 ? " :endloop01" : " :endloop0");
   271:     } else if (IsLoop1) {
   272:       LoopString += " :endloop1";
   273:     }
   274: 
   275:     StringRef Contents(Buffer);
   276:     auto PacketBundle = Contents.rsplit('\n');
   277:     auto HeadTail = PacketBundle.first.split('\n');
   278:     StringRef Separator = "\n";
   279:     StringRef Indent = "\t";
   280:     OS << "\t{\n";
   281:     while (!HeadTail.first.empty()) {
   282:       StringRef InstTxt;
   283:       auto Duplex = HeadTail.first.split('\v');
   284:       if (!Duplex.second.empty()) {
   285:         OS << Indent << Duplex.first << Separator;
   286:         InstTxt = Duplex.second;
   287:       } else if (!HeadTail.first.trim().starts_with("immext")) {
   288:         InstTxt = Duplex.first;
   289:       }
   290:       if (!InstTxt.empty())
   291:         OS << Indent << InstTxt << Separator;
   292:       HeadTail = HeadTail.second.split('\n');
   293:     }
   294: 
   295:     if (HexagonMCInstrInfo::isMemReorderDisabled(Inst))
   296:       OS << "\n\t} :mem_noshuf" << LoopString;
   297:     else
   298:       OS << "\t}" << LoopString;
   299:   }
   300: 
```
- EN: It opens namespaces (llvm, HexagonFUnits) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonTargetAsmStreamer, which carry the state or API of this component. It defines declarative TableGen records like HexagonTargetAsmStreamer; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as isSlot0Only, HexagonTargetAsmStreamer, prettyPrintAsm, assert, ... (11 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（llvm, HexagonFUnits），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonTargetAsmStreamer 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonTargetAsmStreamer 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 isSlot0Only, HexagonTargetAsmStreamer, prettyPrintAsm, assert, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 301-400 / 第 301-400 行

```cpp
   301:   void finish() override { finishAttributeSection(); }
   302: 
   303:   void finishAttributeSection() override {}
   304: 
   305:   void emitAttribute(unsigned Attribute, unsigned Value) override {
   306:     OS << "\t.attribute\t" << Attribute << ", " << Twine(Value);
   307:     if (getStreamer().isVerboseAsm()) {
   308:       StringRef Name = ELFAttrs::attrTypeAsString(
   309:           Attribute, HexagonAttrs::getHexagonAttributeTags());
   310:       if (!Name.empty())
   311:         OS << "\t// " << Name;
   312:     }
   313:     OS << "\n";
   314:   }
   315: };
   316: 
   317: class HexagonTargetELFStreamer : public HexagonTargetStreamer {
   318: public:
   319:   MCELFStreamer &getStreamer() {
   320:     return static_cast<MCELFStreamer &>(Streamer);
   321:   }
   322:   HexagonTargetELFStreamer(MCStreamer &S, MCSubtargetInfo const &STI)
   323:       : HexagonTargetStreamer(S) {
   324:     getStreamer().getWriter().setELFHeaderEFlags(Hexagon_MC::GetELFFlags(STI));
   325:   }
   326: 
   327:   void emitCommonSymbolSorted(MCSymbol *Symbol, uint64_t Size,
   328:                               unsigned ByteAlignment,
   329:                               unsigned AccessSize) override {
   330:     HexagonMCELFStreamer &HexagonELFStreamer =
   331:         static_cast<HexagonMCELFStreamer &>(getStreamer());
   332:     HexagonELFStreamer.HexagonMCEmitCommonSymbol(
   333:         Symbol, Size, Align(ByteAlignment), AccessSize);
   334:   }
   335: 
   336:   void emitLocalCommonSymbolSorted(MCSymbol *Symbol, uint64_t Size,
   337:                                    unsigned ByteAlignment,
   338:                                    unsigned AccessSize) override {
   339:     HexagonMCELFStreamer &HexagonELFStreamer =
   340:         static_cast<HexagonMCELFStreamer &>(getStreamer());
   341:     HexagonELFStreamer.HexagonMCEmitLocalCommonSymbol(
   342:         Symbol, Size, Align(ByteAlignment), AccessSize);
   343:   }
   344: 
   345:   void finish() override { finishAttributeSection(); }
   346: 
   347:   void reset() override { AttributeSection = nullptr; }
   348: 
   349: private:
   350:   MCSection *AttributeSection = nullptr;
   351: 
   352:   void finishAttributeSection() override {
   353:     MCELFStreamer &S = getStreamer();
   354:     if (S.Contents.empty())
   355:       return;
   356: 
   357:     S.emitAttributesSection("hexagon", ".hexagon.attributes",
   358:                             ELF::SHT_HEXAGON_ATTRIBUTES, AttributeSection);
   359:   }
   360: 
   361:   void emitAttribute(uint32_t Attribute, uint32_t Value) override {
   362:     getStreamer().setAttributeItem(Attribute, Value,
   363:                                    /*OverwriteExisting=*/true);
   364:   }
   365: };
   366: 
   367: } // end anonymous namespace
   368: 
   369: llvm::MCInstrInfo *llvm::createHexagonMCInstrInfo() {
   370:   MCInstrInfo *X = new MCInstrInfo();
   371:   InitHexagonMCInstrInfo(X);
   372:   return X;
   373: }
   374: 
   375: static MCRegisterInfo *createHexagonMCRegisterInfo(const Triple &TT) {
   376:   MCRegisterInfo *X = new MCRegisterInfo();
   377:   InitHexagonMCRegisterInfo(X, Hexagon::R31, /*DwarfFlavour=*/0,
   378:                             /*EHFlavour=*/0, /*PC=*/Hexagon::PC);
   379:   return X;
   380: }
   381: 
   382: static MCAsmInfo *createHexagonMCAsmInfo(const MCRegisterInfo &MRI,
   383:                                          const Triple &TT,
   384:                                          const MCTargetOptions &Options) {
   385:   MCAsmInfo *MAI = new HexagonMCAsmInfo(TT, Options);
   386: 
   387:   // VirtualFP = (R30 + #0).
   388:   MCCFIInstruction Inst = MCCFIInstruction::cfiDefCfa(
   389:       nullptr, MRI.getDwarfRegNum(Hexagon::R30, true), 0);
   390:   MAI->addInitialFrameState(Inst);
   391: 
   392:   return MAI;
   393: }
   394: 
   395: static MCInstPrinter *createHexagonMCInstPrinter(const Triple &T,
   396:                                                  unsigned SyntaxVariant,
   397:                                                  const MCAsmInfo &MAI,
   398:                                                  const MCInstrInfo &MII,
   399:                                                  const MCRegisterInfo &MRI)
   400: {
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonTargetELFStreamer, which carry the state or API of this component. It defines declarative TableGen records like HexagonTargetELFStreamer; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as finish, finishAttributeSection, emitAttribute, Twine, ... (24 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonTargetELFStreamer 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonTargetELFStreamer 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 finish, finishAttributeSection, emitAttribute, Twine, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 401-500 / 第 401-500 行

```cpp
   401:   if (SyntaxVariant == 0)
   402:     return new HexagonInstPrinter(MAI, MII, MRI);
   403:   else
   404:     return nullptr;
   405: }
   406: 
   407: static MCTargetStreamer *createMCAsmTargetStreamer(MCStreamer &S,
   408:                                                    formatted_raw_ostream &OS,
   409:                                                    MCInstPrinter *IP) {
   410:   return new HexagonTargetAsmStreamer(S, OS, *IP);
   411: }
   412: 
   413: static MCStreamer *createMCStreamer(Triple const &T, MCContext &Context,
   414:                                     std::unique_ptr<MCAsmBackend> &&MAB,
   415:                                     std::unique_ptr<MCObjectWriter> &&OW,
   416:                                     std::unique_ptr<MCCodeEmitter> &&Emitter) {
   417:   return createHexagonELFStreamer(T, Context, std::move(MAB), std::move(OW),
   418:                                   std::move(Emitter));
   419: }
   420: 
   421: static MCTargetStreamer *
   422: createHexagonObjectTargetStreamer(MCStreamer &S, const MCSubtargetInfo &STI) {
   423:   return new HexagonTargetELFStreamer(S, STI);
   424: }
   425: 
   426: static MCTargetStreamer *createHexagonNullTargetStreamer(MCStreamer &S) {
   427:   return new HexagonTargetStreamer(S);
   428: }
   429: 
   430: [[maybe_unused]] static void clearFeature(MCSubtargetInfo *STI, uint64_t F) {
   431:   if (STI->hasFeature(F))
   432:     STI->ToggleFeature(F);
   433: }
   434: 
   435: [[maybe_unused]] static bool checkFeature(MCSubtargetInfo *STI, uint64_t F) {
   436:   return STI->hasFeature(F);
   437: }
   438: 
   439: namespace {
   440: std::string selectHexagonFS(StringRef CPU, StringRef FS) {
   441:   SmallVector<StringRef, 3> Result;
   442:   if (!FS.empty())
   443:     Result.push_back(FS);
   444: 
   445:   switch (EnableHVX) {
   446:   case Hexagon::ArchEnum::V5:
   447:   case Hexagon::ArchEnum::V55:
   448:     break;
   449:   case Hexagon::ArchEnum::V60:
   450:     Result.push_back("+hvxv60");
   451:     break;
   452:   case Hexagon::ArchEnum::V62:
   453:     Result.push_back("+hvxv62");
   454:     break;
   455:   case Hexagon::ArchEnum::V65:
   456:     Result.push_back("+hvxv65");
   457:     break;
   458:   case Hexagon::ArchEnum::V66:
   459:     Result.push_back("+hvxv66");
   460:     break;
   461:   case Hexagon::ArchEnum::V67:
   462:     Result.push_back("+hvxv67");
   463:     break;
   464:   case Hexagon::ArchEnum::V68:
   465:     Result.push_back("+hvxv68");
   466:     break;
   467:   case Hexagon::ArchEnum::V69:
   468:     Result.push_back("+hvxv69");
   469:     break;
   470:   case Hexagon::ArchEnum::V71:
   471:     Result.push_back("+hvxv71");
   472:     break;
   473:   case Hexagon::ArchEnum::V73:
   474:     Result.push_back("+hvxv73");
   475:     break;
   476:   case Hexagon::ArchEnum::V75:
   477:     Result.push_back("+hvxv75");
   478:     break;
   479:   case Hexagon::ArchEnum::V79:
   480:     Result.push_back("+hvxv79");
   481:     break;
   482:   case Hexagon::ArchEnum::V81:
   483:     Result.push_back("+hvxv81");
   484:     break;
   485: 
   486:   case Hexagon::ArchEnum::Generic: {
   487:     Result.push_back(StringSwitch<StringRef>(CPU)
   488:                          .Case("hexagonv60", "+hvxv60")
   489:                          .Case("hexagonv62", "+hvxv62")
   490:                          .Case("hexagonv65", "+hvxv65")
   491:                          .Case("hexagonv66", "+hvxv66")
   492:                          .Case("hexagonv67", "+hvxv67")
   493:                          .Case("hexagonv67t", "+hvxv67")
   494:                          .Case("hexagonv68", "+hvxv68")
   495:                          .Case("hexagonv69", "+hvxv69")
   496:                          .Case("hexagonv71", "+hvxv71")
   497:                          .Case("hexagonv71t", "+hvxv71")
   498:                          .Case("hexagonv73", "+hvxv73")
   499:                          .Case("hexagonv75", "+hvxv75")
   500:                          .Case("hexagonv79", "+hvxv79")
```
- EN: It declares or implements routines such as createMCAsmTargetStreamer, HexagonTargetAsmStreamer, createMCStreamer, createHexagonELFStreamer, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstPrinter, HexagonTargetAsmStreamer, HexagonTargetELFStreamer, HexagonTargetStreamer, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 createMCAsmTargetStreamer, HexagonTargetAsmStreamer, createMCStreamer, createHexagonELFStreamer, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstPrinter, HexagonTargetAsmStreamer, HexagonTargetELFStreamer, HexagonTargetStreamer，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501:                          .Case("hexagonv81", "+hvxv81"));
   502:     break;
   503:   }
   504:   case Hexagon::ArchEnum::NoArch:
   505:     // Sentinel if -mhvx isn't specified
   506:     break;
   507:   }
   508:   if (EnableHvxIeeeFp)
   509:     Result.push_back("+hvx-ieee-fp");
   510:   if (EnableHexagonCabac)
   511:     Result.push_back("+cabac");
   512: 
   513:   return join(Result.begin(), Result.end(), ",");
   514: }
   515: }
   516: 
   517: static bool isCPUValid(StringRef CPU) {
   518:   return Hexagon::getCpu(CPU).has_value();
   519: }
   520: 
   521: namespace {
   522: std::pair<std::string, std::string> selectCPUAndFS(StringRef CPU,
   523:                                                    StringRef FS) {
   524:   std::pair<std::string, std::string> Result;
   525:   Result.first = std::string(Hexagon_MC::selectHexagonCPU(CPU));
   526:   Result.second = selectHexagonFS(Result.first, FS);
   527:   return Result;
   528: }
   529: std::mutex ArchSubtargetMutex;
   530: std::unordered_map<std::string, std::unique_ptr<MCSubtargetInfo const>>
   531:     ArchSubtarget;
   532: } // namespace
   533: 
   534: MCSubtargetInfo const *
   535: Hexagon_MC::getArchSubtarget(MCSubtargetInfo const *STI) {
   536:   std::lock_guard<std::mutex> Lock(ArchSubtargetMutex);
   537:   auto Existing = ArchSubtarget.find(std::string(STI->getCPU()));
   538:   if (Existing == ArchSubtarget.end())
   539:     return nullptr;
   540:   return Existing->second.get();
   541: }
   542: 
   543: FeatureBitset Hexagon_MC::completeHVXFeatures(const FeatureBitset &S) {
   544:   using namespace Hexagon;
   545:   // Make sure that +hvx-length turns hvx on, and that "hvx" alone
   546:   // turns on hvxvNN, corresponding to the existing ArchVNN.
   547:   FeatureBitset FB = S;
   548:   unsigned CpuArch = ArchV5;
   549:   for (unsigned F :
   550:        {ArchV81, ArchV79, ArchV75, ArchV73, ArchV71, ArchV69, ArchV68, ArchV67,
   551:         ArchV66, ArchV65, ArchV62, ArchV60, ArchV55, ArchV5}) {
   552:     if (!FB.test(F))
   553:       continue;
   554:     CpuArch = F;
   555:     break;
   556:   }
   557:   bool UseHvx = false;
   558:   for (unsigned F : {ExtensionHVX, ExtensionHVX64B, ExtensionHVX128B}) {
   559:     if (!FB.test(F))
   560:       continue;
   561:     UseHvx = true;
   562:     break;
   563:   }
   564:   bool HasHvxVer = false;
   565:   for (unsigned F :
   566:        {ExtensionHVXV60, ExtensionHVXV62, ExtensionHVXV65, ExtensionHVXV66,
   567:         ExtensionHVXV67, ExtensionHVXV68, ExtensionHVXV69, ExtensionHVXV71,
   568:         ExtensionHVXV73, ExtensionHVXV75, ExtensionHVXV79, ExtensionHVXV81}) {
   569:     if (!FB.test(F))
   570:       continue;
   571:     HasHvxVer = true;
   572:     UseHvx = true;
   573:     break;
   574:   }
   575: 
   576:   if (!UseHvx || HasHvxVer)
   577:     return FB;
   578: 
   579:   // HasHvxVer is false, and UseHvx is true.
   580:   switch (CpuArch) {
   581:   case ArchV81:
   582:     FB.set(ExtensionHVXV81);
   583:     [[fallthrough]];
   584:   case ArchV79:
   585:     FB.set(ExtensionHVXV79);
   586:     [[fallthrough]];
   587:   case ArchV75:
   588:     FB.set(ExtensionHVXV75);
   589:     [[fallthrough]];
   590:   case ArchV73:
   591:     FB.set(ExtensionHVXV73);
   592:     [[fallthrough]];
   593:   case ArchV71:
   594:     FB.set(ExtensionHVXV71);
   595:     [[fallthrough]];
   596:   case ArchV69:
   597:     FB.set(ExtensionHVXV69);
   598:     [[fallthrough]];
   599:   case ArchV68:
   600:     FB.set(ExtensionHVXV68);
```
- EN: It opens namespaces (MCSubtargetInfo, Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as Case, join, isCPUValid, Hexagon::getCpu, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include Hexagon_MC, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（MCSubtargetInfo, Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 Case, join, isCPUValid, Hexagon::getCpu, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 Hexagon_MC，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601:     [[fallthrough]];
   602:   case ArchV67:
   603:     FB.set(ExtensionHVXV67);
   604:     [[fallthrough]];
   605:   case ArchV66:
   606:     FB.set(ExtensionHVXV66);
   607:     [[fallthrough]];
   608:   case ArchV65:
   609:     FB.set(ExtensionHVXV65);
   610:     [[fallthrough]];
   611:   case ArchV62:
   612:     FB.set(ExtensionHVXV62);
   613:     [[fallthrough]];
   614:   case ArchV60:
   615:     FB.set(ExtensionHVXV60);
   616:     break;
   617:   }
   618:   return FB;
   619: }
   620: 
   621: MCSubtargetInfo *Hexagon_MC::createHexagonMCSubtargetInfo(const Triple &TT,
   622:                                                           StringRef CPU,
   623:                                                           StringRef FS) {
   624:   std::pair<std::string, std::string> Features = selectCPUAndFS(CPU, FS);
   625:   StringRef CPUName = Features.first;
   626:   StringRef ArchFS = Features.second;
   627: 
   628:   MCSubtargetInfo *X = createHexagonMCSubtargetInfoImpl(
   629:       TT, CPUName, /*TuneCPU*/ CPUName, ArchFS);
   630:   if (X != nullptr && (CPUName == "hexagonv67t" || CPUName == "hexagon71t"))
   631:     addArchSubtarget(X, ArchFS);
   632: 
   633:   if (CPU == "help")
   634:     exit(0);
   635: 
   636:   if (!isCPUValid(CPUName.str())) {
   637:     errs() << "error: invalid CPU \"" << CPUName.str().c_str()
   638:            << "\" specified\n";
   639:     return nullptr;
   640:   }
   641: 
   642:   // Add qfloat subtarget feature by default to v68 and above
   643:   // unless explicitly disabled
   644:   if (checkFeature(X, Hexagon::ExtensionHVXV68) &&
   645:       !ArchFS.contains("-hvx-qfloat")) {
   646:     llvm::FeatureBitset Features = X->getFeatureBits();
   647:     X->setFeatureBits(Features.set(Hexagon::ExtensionHVXQFloat));
   648:   }
   649: 
   650:   if (HexagonDisableDuplex) {
   651:     llvm::FeatureBitset Features = X->getFeatureBits();
   652:     X->setFeatureBits(Features.reset(Hexagon::FeatureDuplex));
   653:   }
   654: 
   655:   X->setFeatureBits(completeHVXFeatures(X->getFeatureBits()));
   656: 
   657:   // The Z-buffer instructions are grandfathered in for current
   658:   // architectures but omitted for new ones.  Future instruction
   659:   // sets may introduce new/conflicting z-buffer instructions.
   660:   const bool ZRegOnDefault =
   661:       (CPUName == "hexagonv67") || (CPUName == "hexagonv66");
   662:   if (ZRegOnDefault) {
   663:     llvm::FeatureBitset Features = X->getFeatureBits();
   664:     X->setFeatureBits(Features.set(Hexagon::ExtensionZReg));
   665:   }
   666: 
   667:   return X;
   668: }
   669: 
   670: void Hexagon_MC::addArchSubtarget(MCSubtargetInfo const *STI, StringRef FS) {
   671:   assert(STI != nullptr);
   672:   if (STI->getCPU().contains("t")) {
   673:     auto ArchSTI = createHexagonMCSubtargetInfo(STI->getTargetTriple(),
   674:                                                 STI->getCPU().drop_back(), FS);
   675:     std::lock_guard<std::mutex> Lock(ArchSubtargetMutex);
   676:     ArchSubtarget[std::string(STI->getCPU())] =
   677:         std::unique_ptr<MCSubtargetInfo const>(ArchSTI);
   678:   }
   679: }
   680: 
   681: std::optional<unsigned>
   682: Hexagon_MC::getHVXVersion(const FeatureBitset &Features) {
   683:   for (auto Arch : {Hexagon::ExtensionHVXV81, Hexagon::ExtensionHVXV79,
   684:                     Hexagon::ExtensionHVXV75, Hexagon::ExtensionHVXV73,
   685:                     Hexagon::ExtensionHVXV71, Hexagon::ExtensionHVXV69,
   686:                     Hexagon::ExtensionHVXV68, Hexagon::ExtensionHVXV67,
   687:                     Hexagon::ExtensionHVXV66, Hexagon::ExtensionHVXV65,
   688:                     Hexagon::ExtensionHVXV62, Hexagon::ExtensionHVXV60})
   689:     if (Features.test(Arch))
   690:       return Arch;
   691:   return {};
   692: }
   693: 
   694: unsigned Hexagon_MC::getArchVersion(const FeatureBitset &Features) {
   695:   for (auto Arch :
   696:        {Hexagon::ArchV81, Hexagon::ArchV79, Hexagon::ArchV75, Hexagon::ArchV73,
   697:         Hexagon::ArchV71, Hexagon::ArchV69, Hexagon::ArchV68, Hexagon::ArchV67,
   698:         Hexagon::ArchV66, Hexagon::ArchV65, Hexagon::ArchV62, Hexagon::ArchV60,
   699:         Hexagon::ArchV55, Hexagon::ArchV5})
   700:     if (Features.test(Arch))
```
- EN: It declares or implements routines such as set, Hexagon_MC::createHexagonMCSubtargetInfo, selectCPUAndFS, createHexagonMCSubtargetInfoImpl, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include Hexagon_MC, HexagonDisableDuplex, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 set, Hexagon_MC::createHexagonMCSubtargetInfo, selectCPUAndFS, createHexagonMCSubtargetInfoImpl, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 Hexagon_MC, HexagonDisableDuplex，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701:       return Arch;
   702:   llvm_unreachable("Expected arch v5-v81");
   703:   return 0;
   704: }
   705: 
   706: unsigned Hexagon_MC::GetELFFlags(const MCSubtargetInfo &STI) {
   707:   return StringSwitch<unsigned>(STI.getCPU())
   708:       .Case("generic", llvm::ELF::EF_HEXAGON_MACH_V5)
   709:       .Case("hexagonv5", llvm::ELF::EF_HEXAGON_MACH_V5)
   710:       .Case("hexagonv55", llvm::ELF::EF_HEXAGON_MACH_V55)
   711:       .Case("hexagonv60", llvm::ELF::EF_HEXAGON_MACH_V60)
   712:       .Case("hexagonv62", llvm::ELF::EF_HEXAGON_MACH_V62)
   713:       .Case("hexagonv65", llvm::ELF::EF_HEXAGON_MACH_V65)
   714:       .Case("hexagonv66", llvm::ELF::EF_HEXAGON_MACH_V66)
   715:       .Case("hexagonv67", llvm::ELF::EF_HEXAGON_MACH_V67)
   716:       .Case("hexagonv67t", llvm::ELF::EF_HEXAGON_MACH_V67T)
   717:       .Case("hexagonv68", llvm::ELF::EF_HEXAGON_MACH_V68)
   718:       .Case("hexagonv69", llvm::ELF::EF_HEXAGON_MACH_V69)
   719:       .Case("hexagonv71", llvm::ELF::EF_HEXAGON_MACH_V71)
   720:       .Case("hexagonv71t", llvm::ELF::EF_HEXAGON_MACH_V71T)
   721:       .Case("hexagonv73", llvm::ELF::EF_HEXAGON_MACH_V73)
   722:       .Case("hexagonv75", llvm::ELF::EF_HEXAGON_MACH_V75)
   723:       .Case("hexagonv79", llvm::ELF::EF_HEXAGON_MACH_V79)
   724:       .Case("hexagonv81", llvm::ELF::EF_HEXAGON_MACH_V81);
   725: }
   726: 
   727: llvm::ArrayRef<MCPhysReg> Hexagon_MC::GetVectRegRev() {
   728:   return ArrayRef(VectRegRev);
   729: }
   730: 
   731: namespace {
   732: class HexagonMCInstrAnalysis : public MCInstrAnalysis {
   733: public:
   734:   HexagonMCInstrAnalysis(MCInstrInfo const *Info) : MCInstrAnalysis(Info) {}
   735: 
   736:   bool isUnconditionalBranch(MCInst const &Inst) const override {
   737:     //assert(!HexagonMCInstrInfo::isBundle(Inst));
   738:     return MCInstrAnalysis::isUnconditionalBranch(Inst);
   739:   }
   740: 
   741:   bool isConditionalBranch(MCInst const &Inst) const override {
   742:     //assert(!HexagonMCInstrInfo::isBundle(Inst));
   743:     return MCInstrAnalysis::isConditionalBranch(Inst);
   744:   }
   745: 
   746:   bool evaluateBranch(MCInst const &Inst, uint64_t Addr,
   747:                       uint64_t Size, uint64_t &Target) const override {
   748:     if (!(isCall(Inst) || isUnconditionalBranch(Inst) ||
   749:           isConditionalBranch(Inst)))
   750:       return false;
   751: 
   752:     //assert(!HexagonMCInstrInfo::isBundle(Inst));
   753:     if (!HexagonMCInstrInfo::isExtendable(*Info, Inst))
   754:       return false;
   755:     auto const &Extended(HexagonMCInstrInfo::getExtendableOperand(*Info, Inst));
   756:     assert(Extended.isExpr());
   757:     int64_t Value;
   758:     if (!Extended.getExpr()->evaluateAsAbsolute(Value))
   759:       return false;
   760:     Target = Value;
   761:     return true;
   762:   }
   763: 
   764:   uint32_t getValueFromMask(uint32_t Instruction, uint32_t Mask) const {
   765:     uint32_t Result = 0;
   766:     uint32_t Offset = 0;
   767:     while (Mask) {
   768:       if (Instruction & (Mask & -Mask))
   769:         Result |= (1 << Offset);
   770:       Mask &= (Mask - 1);
   771:       ++Offset;
   772:     }
   773:     return Result;
   774:   }
   775: 
   776:   std::vector<std::pair<uint64_t, uint64_t>>
   777:   findPltEntries(uint64_t PltSectionVA, ArrayRef<uint8_t> PltContents,
   778:                  const MCSubtargetInfo &STI) const override {
   779:     // Do a lightweight parsing of PLT entries.
   780:     std::vector<std::pair<uint64_t, uint64_t>> Result;
   781:     for (uint64_t Byte = 0x0, End = PltContents.size(); Byte < End; Byte += 4) {
   782:       // Recognize immext(##gotpltn)
   783:       uint32_t ImmExt = support::endian::read32le(PltContents.data() + Byte);
   784:       if ((ImmExt & 0x00004000) != 0x00004000)
   785:         continue;
   786:       uint32_t LoadGotPlt =
   787:           support::endian::read32le(PltContents.data() + Byte + 4);
   788:       if ((LoadGotPlt & 0x6a49c00c) != 0x6a49c00c)
   789:         continue;
   790:       uint32_t Address = (getValueFromMask(ImmExt, 0xfff3fff) << 6) +
   791:                          getValueFromMask(LoadGotPlt, 0x1f80) + PltSectionVA +
   792:                          Byte;
   793:       Result.emplace_back(PltSectionVA + Byte, Address);
   794:     }
   795:     return Result;
   796:   }
   797: };
   798: } // namespace
   799: 
   800: static MCInstrAnalysis *createHexagonMCInstrAnalysis(const MCInstrInfo *Info) {
```
- EN: It opens namespaces (static) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonMCInstrAnalysis, which carry the state or API of this component. It defines declarative TableGen records like HexagonMCInstrAnalysis; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as llvm_unreachable, Hexagon_MC::GetELFFlags, StringSwitch<unsigned>, Hexagon_MC::GetVectRegRev, ... (19 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（static），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonMCInstrAnalysis 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonMCInstrAnalysis 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 llvm_unreachable, Hexagon_MC::GetELFFlags, StringSwitch<unsigned>, Hexagon_MC::GetVectRegRev, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 801-852 / 第 801-852 行

```cpp
   801:   return new HexagonMCInstrAnalysis(Info);
   802: }
   803: 
   804: // Force static initialization.
   805: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
   806: LLVMInitializeHexagonTargetMC() {
   807:   // Register the MC asm info.
   808:   RegisterMCAsmInfoFn X(getTheHexagonTarget(), createHexagonMCAsmInfo);
   809: 
   810:   // Register the MC instruction info.
   811:   TargetRegistry::RegisterMCInstrInfo(getTheHexagonTarget(),
   812:                                       createHexagonMCInstrInfo);
   813: 
   814:   // Register the MC register info.
   815:   TargetRegistry::RegisterMCRegInfo(getTheHexagonTarget(),
   816:                                     createHexagonMCRegisterInfo);
   817: 
   818:   // Register the MC subtarget info.
   819:   TargetRegistry::RegisterMCSubtargetInfo(
   820:       getTheHexagonTarget(), Hexagon_MC::createHexagonMCSubtargetInfo);
   821: 
   822:   // Register the MC Code Emitter
   823:   TargetRegistry::RegisterMCCodeEmitter(getTheHexagonTarget(),
   824:                                         createHexagonMCCodeEmitter);
   825: 
   826:   // Register the asm backend
   827:   TargetRegistry::RegisterMCAsmBackend(getTheHexagonTarget(),
   828:                                        createHexagonAsmBackend);
   829: 
   830:   // Register the MC instruction analyzer.
   831:   TargetRegistry::RegisterMCInstrAnalysis(getTheHexagonTarget(),
   832:                                           createHexagonMCInstrAnalysis);
   833: 
   834:   // Register the obj streamer
   835:   TargetRegistry::RegisterELFStreamer(getTheHexagonTarget(), createMCStreamer);
   836: 
   837:   // Register the obj target streamer
   838:   TargetRegistry::RegisterObjectTargetStreamer(
   839:       getTheHexagonTarget(), createHexagonObjectTargetStreamer);
   840: 
   841:   // Register the asm streamer
   842:   TargetRegistry::RegisterAsmTargetStreamer(getTheHexagonTarget(),
   843:                                             createMCAsmTargetStreamer);
   844: 
   845:   // Register the null streamer
   846:   TargetRegistry::RegisterNullTargetStreamer(getTheHexagonTarget(),
   847:                                              createHexagonNullTargetStreamer);
   848: 
   849:   // Register the MC Inst Printer
   850:   TargetRegistry::RegisterMCInstPrinter(getTheHexagonTarget(),
   851:                                         createHexagonMCInstPrinter);
   852: }
```
- EN: It declares or implements routines such as HexagonMCInstrAnalysis, LLVMInitializeHexagonTargetMC, X, TargetRegistry::RegisterMCInstrInfo, ... (14 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonMCInstrAnalysis, Hexagon_MC, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrAnalysis, LLVMInitializeHexagonTargetMC, X, TargetRegistry::RegisterMCInstrInfo, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrAnalysis, Hexagon_MC，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制
- generated metadata tables / 生成的元数据表

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonMCTargetDesc.h, HexagonDepArch.h, HexagonTargetStreamer.h, MCTargetDesc/HexagonInstPrinter.h, MCTargetDesc/HexagonMCAsmInfo.h, MCTargetDesc/HexagonMCELFStreamer.h, MCTargetDesc/HexagonMCInstrInfo.h, TargetInfo/HexagonTargetInfo.h, llvm/ADT/StringExtras.h, llvm/ADT/StringRef.h, ... (37 total)`
- Hexagon symbols / Hexagon 符号: `HexagonMCTargetDesc, HexagonDepArch, HexagonTargetStreamer, HexagonInstPrinter, HexagonMCAsmInfo, HexagonMCELFStreamer, HexagonMCInstrInfo, HexagonTargetInfo, HexagonAttributes, HexagonGenInstrInfo, ... (28 total)`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
