# CSKYAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYAsmPrinter.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains a printer that converts from our internal representation of machine-dependent LLVM code to the CSKY assembly language.
- 目的（中文）: 实现汇编打印器，把 LLVM 机器指令降级为文本汇编或 MC 层输出。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYAsmPrinter.cpp - CSKY LLVM assembly writer --------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains a printer that converts from our internal representation
  10: // of machine-dependent LLVM code to the CSKY assembly language.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: #include "CSKYAsmPrinter.h"
  14: #include "CSKY.h"
  15: #include "CSKYConstantPoolValue.h"
  16: #include "CSKYTargetMachine.h"
  17: #include "MCTargetDesc/CSKYInstPrinter.h"
  18: #include "MCTargetDesc/CSKYMCAsmInfo.h"
  19: #include "MCTargetDesc/CSKYTargetStreamer.h"
  20: #include "TargetInfo/CSKYTargetInfo.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 21-40

```cpp
  21: #include "llvm/ADT/Statistic.h"
  22: #include "llvm/CodeGen/AsmPrinter.h"
  23: #include "llvm/CodeGen/MachineConstantPool.h"
  24: #include "llvm/CodeGen/MachineFrameInfo.h"
  25: #include "llvm/IR/DataLayout.h"
  26: #include "llvm/MC/MCAsmInfo.h"
  27: #include "llvm/MC/MCContext.h"
  28: #include "llvm/MC/MCInstBuilder.h"
  29: #include "llvm/MC/MCStreamer.h"
  30: #include "llvm/MC/TargetRegistry.h"
  31: 
  32: using namespace llvm;
  33: 
  34: #define DEBUG_TYPE "csky-asm-printer"
  35: 
  36: STATISTIC(CSKYNumInstrsCompressed,
  37:           "Number of C-SKY Compressed instructions emitted");
  38: 
  39: CSKYAsmPrinter::CSKYAsmPrinter(llvm::TargetMachine &TM,
  40:                                std::unique_ptr<llvm::MCStreamer> Streamer)
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 41-60

```cpp
  41:     : AsmPrinter(TM, std::move(Streamer)), MCInstLowering(OutContext, *this) {}
  42: 
  43: bool CSKYAsmPrinter::runOnMachineFunction(MachineFunction &MF) {
  44:   MCP = MF.getConstantPool();
  45:   TII = MF.getSubtarget().getInstrInfo();
  46: 
  47:   // Set the current MCSubtargetInfo to a copy which has the correct
  48:   // feature bits for the current MachineFunction
  49:   MCSubtargetInfo &NewSTI =
  50:       OutStreamer->getContext().getSubtargetCopy(TM.getMCSubtargetInfo());
  51:   NewSTI.setFeatureBits(MF.getSubtarget().getFeatureBits());
  52:   Subtarget = &NewSTI;
  53: 
  54:   return AsmPrinter::runOnMachineFunction(MF);
  55: }
  56: 
  57: #define GEN_COMPRESS_INSTR
  58: #include "CSKYGenCompressInstEmitter.inc"
  59: void CSKYAsmPrinter::EmitToStreamer(MCStreamer &S, const MCInst &Inst) {
  60:   MCInst CInst;
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as AsmPrinter, runOnMachineFunction, EmitToStreamer contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 AsmPrinter, runOnMachineFunction, EmitToStreamer 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 61-80

```cpp
  61:   bool Res = compressInst(CInst, Inst, *Subtarget);
  62:   if (Res)
  63:     ++CSKYNumInstrsCompressed;
  64:   AsmPrinter::EmitToStreamer(*OutStreamer, Res ? CInst : Inst);
  65: }
  66: 
  67: // Simple pseudo-instructions have their lowering (with expansion to real
  68: // instructions) auto-generated.
  69: #include "CSKYGenMCPseudoLowering.inc"
  70: 
  71: void CSKYAsmPrinter::expandTLSLA(const MachineInstr *MI) {
  72:   DebugLoc DL = MI->getDebugLoc();
  73: 
  74:   MCSymbol *PCLabel = OutContext.getOrCreateSymbol(
  75:       Twine(MAI.getInternalSymbolPrefix()) + "PC" + Twine(getFunctionNumber()) +
  76:       "_" + Twine(MI->getOperand(3).getImm()));
  77: 
  78:   OutStreamer->emitLabel(PCLabel);
  79: 
  80:   auto Instr = BuildMI(*MF, DL, TII->get(CSKY::LRW32))
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as expandTLSLA contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 expandTLSLA 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 81-100

```cpp
  81:                    .add(MI->getOperand(0))
  82:                    .add(MI->getOperand(2));
  83:   MCInst LRWInst;
  84:   MCInstLowering.Lower(Instr, LRWInst);
  85:   EmitToStreamer(*OutStreamer, LRWInst);
  86: 
  87:   Instr = BuildMI(*MF, DL, TII->get(CSKY::GRS32))
  88:               .add(MI->getOperand(1))
  89:               .addSym(PCLabel);
  90:   MCInst GRSInst;
  91:   MCInstLowering.Lower(Instr, GRSInst);
  92:   EmitToStreamer(*OutStreamer, GRSInst);
  93:   return;
  94: }
  95: 
  96: void CSKYAsmPrinter::emitCustomConstantPool(const MachineInstr *MI) {
  97: 
  98:   // This instruction represents a floating constant pool in the function.
  99:   // The first operand is the ID# for this instruction, the second is the
 100:   // index into the MachineConstantPool that this is, the third is the size
```

- EN: Function bodies or method definitions such as emitCustomConstantPool contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: emitCustomConstantPool 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 101-120

```cpp
 101:   // in bytes of this constant pool entry.
 102:   // The required alignment is specified on the basic block holding this MI.
 103:   unsigned LabelId = (unsigned)MI->getOperand(0).getImm();
 104:   unsigned CPIdx = (unsigned)MI->getOperand(1).getIndex();
 105: 
 106:   // If this is the first entry of the pool, mark it.
 107:   if (!InConstantPool) {
 108:     OutStreamer->emitValueToAlignment(Align(4));
 109:     InConstantPool = true;
 110:   }
 111: 
 112:   OutStreamer->emitLabel(GetCPISymbol(LabelId));
 113: 
 114:   const MachineConstantPoolEntry &MCPE = MCP->getConstants()[CPIdx];
 115:   if (MCPE.isMachineConstantPoolEntry())
 116:     emitMachineConstantPoolValue(MCPE.Val.MachineCPVal);
 117:   else
 118:     emitGlobalConstant(MF->getDataLayout(), MCPE.Val.ConstVal);
 119:   return;
 120: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 121-140

```cpp
 121: 
 122: void CSKYAsmPrinter::emitFunctionBodyEnd() {
 123:   // Make sure to terminate any constant pools that were at the end
 124:   // of the function.
 125:   if (!InConstantPool)
 126:     return;
 127:   InConstantPool = false;
 128: }
 129: 
 130: void CSKYAsmPrinter::emitStartOfAsmFile(Module &M) {
 131:   if (TM.getTargetTriple().isOSBinFormatELF())
 132:     emitAttributes();
 133: }
 134: 
 135: void CSKYAsmPrinter::emitEndOfAsmFile(Module &M) {
 136:   CSKYTargetStreamer &CTS =
 137:       static_cast<CSKYTargetStreamer &>(*OutStreamer->getTargetStreamer());
 138: 
 139:   if (TM.getTargetTriple().isOSBinFormatELF())
 140:     CTS.finishAttributeSection();
```

- EN: Function bodies or method definitions such as emitFunctionBodyEnd, emitStartOfAsmFile, emitEndOfAsmFile contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: emitFunctionBodyEnd, emitStartOfAsmFile, emitEndOfAsmFile 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 141-160

```cpp
 141: }
 142: 
 143: void CSKYAsmPrinter::emitInstruction(const MachineInstr *MI) {
 144:   CSKY_MC::verifyInstructionPredicates(MI->getOpcode(),
 145:                                        getSubtargetInfo().getFeatureBits());
 146: 
 147:   // Do any auto-generated pseudo lowerings.
 148:   if (MCInst OutInst; lowerPseudoInstExpansion(MI, OutInst)) {
 149:     EmitToStreamer(*OutStreamer, OutInst);
 150:     return;
 151:   }
 152: 
 153:   // If we just ended a constant pool, mark it as such.
 154:   if (InConstantPool && MI->getOpcode() != CSKY::CONSTPOOL_ENTRY) {
 155:     InConstantPool = false;
 156:   }
 157: 
 158:   if (MI->getOpcode() == CSKY::PseudoTLSLA32)
 159:     return expandTLSLA(MI);
 160: 
```

- EN: Function bodies or method definitions such as emitInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: emitInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 161-180

```cpp
 161:   if (MI->getOpcode() == CSKY::CONSTPOOL_ENTRY)
 162:     return emitCustomConstantPool(MI);
 163: 
 164:   MCInst TmpInst;
 165:   MCInstLowering.Lower(MI, TmpInst);
 166:   EmitToStreamer(*OutStreamer, TmpInst);
 167: }
 168: 
 169: // Convert a CSKY-specific constant pool modifier into the associated
 170: // MCSymbolRefExpr variant kind.
 171: static CSKY::Specifier getModifierVariantKind(CSKYCP::CSKYCPModifier Modifier) {
 172:   switch (Modifier) {
 173:   case CSKYCP::NO_MOD:
 174:     return CSKY::S_None;
 175:   case CSKYCP::ADDR:
 176:     return CSKY::S_ADDR;
 177:   case CSKYCP::GOT:
 178:     return CSKY::S_GOT;
 179:   case CSKYCP::GOTOFF:
 180:     return CSKY::S_GOTOFF;
```

- EN: Function bodies or method definitions such as getModifierVariantKind contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: getModifierVariantKind 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 181-200

```cpp
 181:   case CSKYCP::PLT:
 182:     return CSKY::S_PLT;
 183:   case CSKYCP::TLSGD:
 184:     return CSKY::S_TLSGD;
 185:   case CSKYCP::TLSLE:
 186:     return CSKY::S_TLSLE;
 187:   case CSKYCP::TLSIE:
 188:     return CSKY::S_TLSIE;
 189:   }
 190:   llvm_unreachable("Invalid CSKYCPModifier!");
 191: }
 192: 
 193: void CSKYAsmPrinter::emitMachineConstantPoolValue(
 194:     MachineConstantPoolValue *MCPV) {
 195:   int Size = getDataLayout().getTypeAllocSize(MCPV->getType());
 196:   CSKYConstantPoolValue *CCPV = static_cast<CSKYConstantPoolValue *>(MCPV);
 197:   MCSymbol *MCSym;
 198: 
 199:   if (CCPV->isBlockAddress()) {
 200:     const BlockAddress *BA =
```

- EN: Function bodies or method definitions such as emitMachineConstantPoolValue contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: emitMachineConstantPoolValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 201-220

```cpp
 201:         cast<CSKYConstantPoolConstant>(CCPV)->getBlockAddress();
 202:     MCSym = GetBlockAddressSymbol(BA);
 203:   } else if (CCPV->isGlobalValue()) {
 204:     const GlobalValue *GV = cast<CSKYConstantPoolConstant>(CCPV)->getGV();
 205:     MCSym = getSymbol(GV);
 206:   } else if (CCPV->isMachineBasicBlock()) {
 207:     const MachineBasicBlock *MBB = cast<CSKYConstantPoolMBB>(CCPV)->getMBB();
 208:     MCSym = MBB->getSymbol();
 209:   } else if (CCPV->isJT()) {
 210:     signed JTI = cast<CSKYConstantPoolJT>(CCPV)->getJTI();
 211:     MCSym = GetJTISymbol(JTI);
 212:   } else if (CCPV->isConstPool()) {
 213:     const Constant *C = cast<CSKYConstantPoolConstant>(CCPV)->getConstantPool();
 214:     MCSym = GetCPISymbol(MCP->getConstantPoolIndex(C, Align(4)));
 215:   } else {
 216:     assert(CCPV->isExtSymbol() && "unrecognized constant pool value");
 217:     StringRef Sym = cast<CSKYConstantPoolSymbol>(CCPV)->getSymbol();
 218:     MCSym = GetExternalSymbolSymbol(Sym);
 219:   }
 220:   // Create an MCSymbol for the reference.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 221-240

```cpp
 221:   const MCExpr *Expr = MCSymbolRefExpr::create(MCSym, OutContext);
 222: 
 223:   if (CCPV->getPCAdjustment()) {
 224: 
 225:     MCSymbol *PCLabel = OutContext.getOrCreateSymbol(
 226:         Twine(MAI.getInternalSymbolPrefix()) + "PC" +
 227:         Twine(getFunctionNumber()) + "_" + Twine(CCPV->getLabelID()));
 228: 
 229:     const MCExpr *PCRelExpr = MCSymbolRefExpr::create(PCLabel, OutContext);
 230:     if (CCPV->mustAddCurrentAddress()) {
 231:       // We want "(<expr> - .)", but MC doesn't have a concept of the '.'
 232:       // label, so just emit a local label end reference that instead.
 233:       MCSymbol *DotSym = OutContext.createTempSymbol();
 234:       OutStreamer->emitLabel(DotSym);
 235:       const MCExpr *DotExpr = MCSymbolRefExpr::create(DotSym, OutContext);
 236:       PCRelExpr = MCBinaryExpr::createSub(PCRelExpr, DotExpr, OutContext);
 237:     }
 238:     Expr = MCBinaryExpr::createSub(Expr, PCRelExpr, OutContext);
 239:   }
 240: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 241-260

```cpp
 241:   // Create an MCSymbol for the reference.
 242:   Expr = MCSpecifierExpr::create(
 243:       Expr, getModifierVariantKind(CCPV->getModifier()), OutContext);
 244: 
 245:   OutStreamer->emitValue(Expr, Size);
 246: }
 247: 
 248: void CSKYAsmPrinter::emitAttributes() {
 249:   CSKYTargetStreamer &CTS =
 250:       static_cast<CSKYTargetStreamer &>(*OutStreamer->getTargetStreamer());
 251: 
 252:   const Triple &TT = TM.getTargetTriple();
 253:   StringRef CPU = TM.getTargetCPU();
 254:   StringRef FS = TM.getTargetFeatureString();
 255:   const CSKYTargetMachine &CTM = static_cast<const CSKYTargetMachine &>(TM);
 256:   /* TuneCPU doesn't impact emission of ELF attributes, ELF attributes only
 257:      care about arch related features, so we can set TuneCPU as CPU.  */
 258:   const CSKYSubtarget STI(TT, CPU, /*TuneCPU=*/CPU, FS, CTM);
 259: 
 260:   CTS.emitTargetAttributes(STI);
```

- EN: Function bodies or method definitions such as emitAttributes contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: emitAttributes 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 261-280

```cpp
 261: }
 262: 
 263: bool CSKYAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
 264:                                      const char *ExtraCode, raw_ostream &OS) {
 265:   // First try the generic code, which knows about modifiers like 'c' and 'n'.
 266:   if (!AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, OS))
 267:     return false;
 268: 
 269:   const MachineOperand &MO = MI->getOperand(OpNo);
 270:   if (ExtraCode && ExtraCode[0]) {
 271:     if (ExtraCode[1] != 0)
 272:       return true; // Unknown modifier.
 273: 
 274:     switch (ExtraCode[0]) {
 275:     default:
 276:       return true; // Unknown modifier.
 277:     case 'R':
 278:       if (MO.getType() == MachineOperand::MO_Register) {
 279:         OS << CSKYInstPrinter::getRegisterName(MO.getReg() + 1);
 280:         return false;
```

- EN: Function bodies or method definitions such as PrintAsmOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: PrintAsmOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 281-300

```cpp
 281:       }
 282:     }
 283:   }
 284: 
 285:   switch (MO.getType()) {
 286:   case MachineOperand::MO_Immediate:
 287:     OS << MO.getImm();
 288:     return false;
 289:   case MachineOperand::MO_Register:
 290:     if (MO.getReg() == CSKY::C)
 291:       return false;
 292:     OS << CSKYInstPrinter::getRegisterName(MO.getReg());
 293:     return false;
 294:   case MachineOperand::MO_GlobalAddress:
 295:     PrintSymbolOperand(MO, OS);
 296:     return false;
 297:   case MachineOperand::MO_BlockAddress: {
 298:     MCSymbol *Sym = GetBlockAddressSymbol(MO.getBlockAddress());
 299:     Sym->print(OS, MAI);
 300:     return false;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 301-320

```cpp
 301:   }
 302:   default:
 303:     break;
 304:   }
 305: 
 306:   return true;
 307: }
 308: 
 309: bool CSKYAsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI,
 310:                                            unsigned OpNo, const char *ExtraCode,
 311:                                            raw_ostream &OS) {
 312:   if (!ExtraCode) {
 313:     const MachineOperand &MO = MI->getOperand(OpNo);
 314:     // For now, we only support register memory operands in registers and
 315:     // assume there is no addend
 316:     if (!MO.isReg())
 317:       return true;
 318: 
 319:     OS << "(" << CSKYInstPrinter::getRegisterName(MO.getReg()) << ", 0)";
 320:     return false;
```

- EN: Function bodies or method definitions such as PrintAsmMemoryOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: PrintAsmMemoryOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 321-328

```cpp
 321:   }
 322: 
 323:   return AsmPrinter::PrintAsmMemoryOperand(MI, OpNo, ExtraCode, OS);
 324: }
 325: 
 326: extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeCSKYAsmPrinter() {
 327:   RegisterAsmPrinter<CSKYAsmPrinter> X(getTheCSKYTarget());
 328: }
```

- EN: In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 从文件角色看，它参与将机器指令打印或降级为汇编输出。

## Key Concepts / 关键概念

- MachineInstr to MC lowering / MachineInstr 到 MC 降级
- Assembly syntax emission / 汇编语法输出
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- MC instruction representation / MC 指令表示
- MC streaming and emission / MC 流式输出
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYAsmPrinter.h`, `CSKY.h`, `CSKYConstantPoolValue.h`, `CSKYTargetMachine.h`, `MCTargetDesc/CSKYInstPrinter.h`, `MCTargetDesc/CSKYMCAsmInfo.h`, `MCTargetDesc/CSKYTargetStreamer.h`, `TargetInfo/CSKYTargetInfo.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, LLVM MC
- Generated or companion files / 生成或配套文件: `CSKYGenCompressInstEmitter.inc`, `CSKYGenMCPseudoLowering.inc`
- Local companions / 本地配套文件: `CSKYAsmPrinter.h`
