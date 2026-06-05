# BPFAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFAsmPrinter.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains a printer that converts from our internal representation of machine-dependent LLVM code to the BPF assembly language.
- 目的（中文）: 实现汇编打印器，把 LLVM 机器指令降级为文本汇编或 MC 层输出。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- BPFAsmPrinter.cpp - BPF LLVM assembly writer ----------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains a printer that converts from our internal representation
  10: // of machine-dependent LLVM code to the BPF assembly language.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "BPFAsmPrinter.h"
  15: #include "BPF.h"
  16: #include "BPFInstrInfo.h"
  17: #include "BPFMCInstLower.h"
  18: #include "BTFDebug.h"
  19: #include "MCTargetDesc/BPFInstPrinter.h"
  20: #include "TargetInfo/BPFTargetInfo.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 21-40

```cpp
  21: #include "llvm/BinaryFormat/ELF.h"
  22: #include "llvm/CodeGen/AsmPrinter.h"
  23: #include "llvm/CodeGen/MachineConstantPool.h"
  24: #include "llvm/CodeGen/MachineInstr.h"
  25: #include "llvm/CodeGen/MachineJumpTableInfo.h"
  26: #include "llvm/CodeGen/MachineModuleInfo.h"
  27: #include "llvm/CodeGen/TargetLowering.h"
  28: #include "llvm/IR/DiagnosticInfo.h"
  29: #include "llvm/IR/Module.h"
  30: #include "llvm/MC/MCAsmInfo.h"
  31: #include "llvm/MC/MCExpr.h"
  32: #include "llvm/MC/MCInst.h"
  33: #include "llvm/MC/MCSectionELF.h"
  34: #include "llvm/MC/MCStreamer.h"
  35: #include "llvm/MC/MCSymbol.h"
  36: #include "llvm/MC/MCSymbolELF.h"
  37: #include "llvm/MC/TargetRegistry.h"
  38: #include "llvm/Support/Compiler.h"
  39: #include "llvm/Support/raw_ostream.h"
  40: #include "llvm/Target/TargetLoweringObjectFile.h"
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 41-60

```cpp
  41: using namespace llvm;
  42: 
  43: #define DEBUG_TYPE "asm-printer"
  44: 
  45: BPFAsmPrinter::BPFAsmPrinter(TargetMachine &TM,
  46:                              std::unique_ptr<MCStreamer> Streamer)
  47:     : AsmPrinter(TM, std::move(Streamer), ID), BTF(nullptr), TM(TM) {}
  48: 
  49: BPFAsmPrinter::~BPFAsmPrinter() = default;
  50: 
  51: bool BPFAsmPrinter::doInitialization(Module &M) {
  52:   AsmPrinter::doInitialization(M);
  53: 
  54:   // Only emit BTF when debuginfo available.
  55:   if (MAI.doesSupportDebugInformation() && !M.debug_compile_units().empty()) {
  56:     BTF = new BTFDebug(this);
  57:     Handlers.push_back(std::unique_ptr<BTFDebug>(BTF));
  58:   }
  59: 
  60:   return false;
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as AsmPrinter, doInitialization contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 AsmPrinter, doInitialization 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 61-80

```cpp
  61: }
  62: 
  63: const BPFTargetMachine &BPFAsmPrinter::getBTM() const {
  64:   return static_cast<const BPFTargetMachine &>(TM);
  65: }
  66: 
  67: bool BPFAsmPrinter::doFinalization(Module &M) {
  68:   // Remove unused globals which are previously used for jump table.
  69:   const BPFSubtarget *Subtarget = getBTM().getSubtargetImpl();
  70:   if (Subtarget->hasGotox()) {
  71:     std::vector<GlobalVariable *> Targets;
  72:     for (GlobalVariable &Global : M.globals()) {
  73:       if (Global.getLinkage() != GlobalValue::PrivateLinkage)
  74:         continue;
  75:       if (!Global.isConstant() || !Global.hasInitializer())
  76:         continue;
  77: 
  78:       Constant *CV = dyn_cast<Constant>(Global.getInitializer());
  79:       if (!CV)
  80:         continue;
```

- EN: Function bodies or method definitions such as doFinalization contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: doFinalization 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 81-100

```cpp
  81:       ConstantArray *CA = dyn_cast<ConstantArray>(CV);
  82:       if (!CA)
  83:         continue;
  84: 
  85:       for (unsigned i = 1, e = CA->getNumOperands(); i != e; ++i) {
  86:         if (!dyn_cast<BlockAddress>(CA->getOperand(i)))
  87:           continue;
  88:       }
  89:       Targets.push_back(&Global);
  90:     }
  91: 
  92:     for (GlobalVariable *GV : Targets) {
  93:       GV->replaceAllUsesWith(PoisonValue::get(GV->getType()));
  94:       GV->dropAllReferences();
  95:       GV->eraseFromParent();
  96:     }
  97:   }
  98: 
  99:   for (GlobalObject &GO : M.global_objects()) {
 100:     if (!GO.hasExternalWeakLinkage())
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 101-120

```cpp
 101:       continue;
 102: 
 103:     if (!SawTrapCall && GO.getName() == BPF_TRAP) {
 104:       GO.eraseFromParent();
 105:       break;
 106:     }
 107:   }
 108: 
 109:   return AsmPrinter::doFinalization(M);
 110: }
 111: 
 112: void BPFAsmPrinter::printOperand(const MachineInstr *MI, int OpNum,
 113:                                  raw_ostream &O) {
 114:   const MachineOperand &MO = MI->getOperand(OpNum);
 115: 
 116:   switch (MO.getType()) {
 117:   case MachineOperand::MO_Register:
 118:     O << BPFInstPrinter::getRegisterName(MO.getReg());
 119:     break;
 120: 
```

- EN: Function bodies or method definitions such as printOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: printOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 121-140

```cpp
 121:   case MachineOperand::MO_Immediate:
 122:     O << MO.getImm();
 123:     break;
 124: 
 125:   case MachineOperand::MO_MachineBasicBlock:
 126:     O << *MO.getMBB()->getSymbol();
 127:     break;
 128: 
 129:   case MachineOperand::MO_GlobalAddress:
 130:     O << *getSymbol(MO.getGlobal());
 131:     break;
 132: 
 133:   case MachineOperand::MO_BlockAddress: {
 134:     MCSymbol *BA = GetBlockAddressSymbol(MO.getBlockAddress());
 135:     O << BA->getName();
 136:     break;
 137:   }
 138: 
 139:   case MachineOperand::MO_ExternalSymbol:
 140:     O << *GetExternalSymbolSymbol(MO.getSymbolName());
```

- EN: In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 141-160

```cpp
 141:     break;
 142: 
 143:   case MachineOperand::MO_JumpTableIndex:
 144:   case MachineOperand::MO_ConstantPoolIndex:
 145:   default:
 146:     llvm_unreachable("<unknown operand type>");
 147:   }
 148: }
 149: 
 150: bool BPFAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
 151:                                     const char *ExtraCode, raw_ostream &O) {
 152:   if (ExtraCode && ExtraCode[0])
 153:     return AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, O);
 154: 
 155:   printOperand(MI, OpNo, O);
 156:   return false;
 157: }
 158: 
 159: bool BPFAsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI,
 160:                                           unsigned OpNum, const char *ExtraCode,
```

- EN: Function bodies or method definitions such as PrintAsmOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: PrintAsmOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 161-180

```cpp
 161:                                           raw_ostream &O) {
 162:   assert(OpNum + 1 < MI->getNumOperands() && "Insufficient operands");
 163:   const MachineOperand &BaseMO = MI->getOperand(OpNum);
 164:   const MachineOperand &OffsetMO = MI->getOperand(OpNum + 1);
 165:   assert(BaseMO.isReg() && "Unexpected base pointer for inline asm memory operand.");
 166:   assert(OffsetMO.isImm() && "Unexpected offset for inline asm memory operand.");
 167:   int Offset = OffsetMO.getImm();
 168: 
 169:   if (ExtraCode)
 170:     return true; // Unknown modifier.
 171: 
 172:   if (Offset < 0)
 173:     O << "(" << BPFInstPrinter::getRegisterName(BaseMO.getReg()) << " - " << -Offset << ")";
 174:   else
 175:     O << "(" << BPFInstPrinter::getRegisterName(BaseMO.getReg()) << " + " << Offset << ")";
 176: 
 177:   return false;
 178: }
 179: 
 180: void BPFAsmPrinter::emitInstruction(const MachineInstr *MI) {
```

- EN: Function bodies or method definitions such as emitInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: emitInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 181-200

```cpp
 181:   if (MI->isCall()) {
 182:     for (const MachineOperand &Op : MI->operands()) {
 183:       if (Op.isGlobal()) {
 184:         if (const GlobalValue *GV = Op.getGlobal())
 185:           if (GV->getName() == BPF_TRAP)
 186:             SawTrapCall = true;
 187:       }
 188:     }
 189:   }
 190: 
 191:   BPF_MC::verifyInstructionPredicates(MI->getOpcode(),
 192:                                       getSubtargetInfo().getFeatureBits());
 193: 
 194:   MCInst TmpInst;
 195: 
 196:   if (!BTF || !BTF->InstLower(MI, TmpInst)) {
 197:     BPFMCInstLower MCInstLowering(OutContext, *this);
 198:     MCInstLowering.Lower(MI, TmpInst);
 199:   }
 200:   EmitToStreamer(*OutStreamer, TmpInst);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 201-220

```cpp
 201: }
 202: 
 203: void BPFAsmPrinter::emitFunctionBodyEnd() {
 204:   // Emit .bpf_cleanup section with a flat table of
 205:   // (call_site, landing_pad) pairs.
 206:   const std::vector<LandingPadInfo> &LandingPads = MF->getLandingPads();
 207:   if (LandingPads.empty())
 208:     return;
 209: 
 210:   MCContext &Ctx = OutContext;
 211:   auto *CleanupSec =
 212:       Ctx.getELFSection(".bpf_cleanup", ELF::SHT_PROGBITS, ELF::SHF_ALLOC);
 213:   OutStreamer->switchSection(CleanupSec);
 214: 
 215:   const auto &TypeInfos = MF->getTypeInfos();
 216:   const Function &F = MF->getFunction();
 217:   LLVMContext &LLVMCtx = F.getContext();
 218: 
 219:   // Each landing pad has BeginLabels/EndLabels marking the invoke
 220:   // call sites that unwind to it.
```

- EN: Function bodies or method definitions such as emitFunctionBodyEnd contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: emitFunctionBodyEnd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 221-240

```cpp
 221:   for (const LandingPadInfo &LP : LandingPads) {
 222:     // BPF treats all landing pads as catch-all: the kernel redirects to
 223:     // the landing pad regardless of exception type. Reject type-specific
 224:     // catches and filters which would silently misbehave.
 225:     for (int TId : LP.TypeIds) {
 226:       if (TId > 0 && TypeInfos[TId - 1] != nullptr) {
 227:         LLVMCtx.diagnose(DiagnosticInfoUnsupported(
 228:             F, "BPF does not support type-specific exception catches yet"));
 229:         return;
 230:       }
 231:       if (TId < 0) {
 232:         LLVMCtx.diagnose(DiagnosticInfoUnsupported(
 233:             F, "BPF does not support exception filters yet"));
 234:         return;
 235:       }
 236:     }
 237: 
 238:     MCSymbol *LPLabel = LP.LandingPadLabel;
 239:     if (!LPLabel)
 240:       continue;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 241-260

```cpp
 241:     for (unsigned i = 0, e = LP.BeginLabels.size(); i != e; ++i) {
 242:       MCSymbol *Begin = LP.BeginLabels[i];
 243:       MCSymbol *End = LP.EndLabels[i];
 244: 
 245:       // Each entry is 3 x 4 bytes: begin, end, landing_pad.
 246:       // The invoke region [begin, end) may include argument setup
 247:       // before the call. The runtime checks begin <= PC < end.
 248:       OutStreamer->emitSymbolValue(Begin, 4);
 249:       OutStreamer->emitSymbolValue(End, 4);
 250:       OutStreamer->emitSymbolValue(LPLabel, 4);
 251:     }
 252:   }
 253: 
 254:   // Switch back to the function's section.
 255:   OutStreamer->switchSection(MF->getSection());
 256: }
 257: 
 258: MCSymbol *BPFAsmPrinter::getJTPublicSymbol(unsigned JTI) {
 259:   SmallString<60> Name;
 260:   raw_svector_ostream(Name)
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 261-280

```cpp
 261:       << "BPF.JT." << MF->getFunctionNumber() << '.' << JTI;
 262:   MCSymbol *S = OutContext.getOrCreateSymbol(Name);
 263:   if (auto *ES = static_cast<MCSymbolELF *>(S)) {
 264:     ES->setBinding(ELF::STB_GLOBAL);
 265:     ES->setType(ELF::STT_OBJECT);
 266:   }
 267:   return S;
 268: }
 269: 
 270: void BPFAsmPrinter::emitJumpTableInfo() {
 271:   const MachineJumpTableInfo *MJTI = MF->getJumpTableInfo();
 272:   if (!MJTI)
 273:     return;
 274: 
 275:   const std::vector<MachineJumpTableEntry> &JT = MJTI->getJumpTables();
 276:   if (JT.empty())
 277:     return;
 278: 
 279:   const TargetLoweringObjectFile &TLOF = getObjFileLowering();
 280:   const Function &F = MF->getFunction();
```

- EN: Function bodies or method definitions such as emitJumpTableInfo contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: emitJumpTableInfo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 281-300

```cpp
 281: 
 282:   MCSection *Sec = OutStreamer->getCurrentSectionOnly();
 283:   MCSymbol *SecStart = Sec->getBeginSymbol();
 284: 
 285:   MCSection *JTS = TLOF.getSectionForJumpTable(F, TM);
 286:   assert(MJTI->getEntryKind() == MachineJumpTableInfo::EK_BlockAddress);
 287:   unsigned EntrySize = MJTI->getEntrySize(getDataLayout());
 288:   OutStreamer->switchSection(JTS);
 289:   for (unsigned JTI = 0; JTI < JT.size(); JTI++) {
 290:     ArrayRef<MachineBasicBlock *> JTBBs = JT[JTI].MBBs;
 291:     if (JTBBs.empty())
 292:       continue;
 293: 
 294:     MCSymbol *JTStart = getJTPublicSymbol(JTI);
 295:     OutStreamer->emitLabel(JTStart);
 296:     for (const MachineBasicBlock *MBB : JTBBs) {
 297:       const MCExpr *Diff = MCBinaryExpr::createSub(
 298:           MCSymbolRefExpr::create(MBB->getSymbol(), OutContext),
 299:           MCSymbolRefExpr::create(SecStart, OutContext), OutContext);
 300:       OutStreamer->emitValue(Diff, EntrySize);
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 301-319

```cpp
 301:     }
 302:     const MCExpr *JTSize =
 303:         MCConstantExpr::create(JTBBs.size() * EntrySize, OutContext);
 304:     OutStreamer->emitELFSize(JTStart, JTSize);
 305:   }
 306: }
 307: 
 308: char BPFAsmPrinter::ID = 0;
 309: 
 310: INITIALIZE_PASS(BPFAsmPrinter, "bpf-asm-printer", "BPF Assembly Printer", false,
 311:                 false)
 312: 
 313: // Force static initialization.
 314: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
 315: LLVMInitializeBPFAsmPrinter() {
 316:   RegisterAsmPrinter<BPFAsmPrinter> X(getTheBPFleTarget());
 317:   RegisterAsmPrinter<BPFAsmPrinter> Y(getTheBPFbeTarget());
 318:   RegisterAsmPrinter<BPFAsmPrinter> Z(getTheBPFTarget());
 319: }
```

- EN: In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 从文件角色看，它参与将机器指令打印或降级为汇编输出。

## Key Concepts / 关键概念

- MachineInstr to MC lowering / MachineInstr 到 MC 降级
- Assembly syntax emission / 汇编语法输出
- Machine instruction manipulation / 机器指令操作
- Basic block level transformation / 基本块级转换
- MC instruction representation / MC 指令表示
- MC streaming and emission / MC 流式输出
- IR to target lowering / IR 到目标降级
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPFAsmPrinter.h`, `BPF.h`, `BPFInstrInfo.h`, `BPFMCInstLower.h`, `BTFDebug.h`, `MCTargetDesc/BPFInstPrinter.h`, `TargetInfo/BPFTargetInfo.h`, `llvm/BinaryFormat/ELF.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, LLVM MC, SelectionDAG
- Local companions / 本地配套文件: `BPFAsmPrinter.h`
