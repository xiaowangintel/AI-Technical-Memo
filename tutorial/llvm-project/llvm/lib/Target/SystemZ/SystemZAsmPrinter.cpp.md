# SystemZAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZAsmPrinter.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file emits textual assembly from LLVM machine instructions for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将 LLVM 机器指令输出为文本汇编。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```cpp
   1: //===-- SystemZAsmPrinter.cpp - SystemZ LLVM assembly printer -------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // Streams SystemZ assembly language and associated data, in the form of
  10: // MCInsts and MCExprs respectively.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "SystemZAsmPrinter.h"
  15: #include "MCTargetDesc/SystemZGNUInstPrinter.h"
  16: #include "MCTargetDesc/SystemZHLASMInstPrinter.h"
  17: #include "MCTargetDesc/SystemZMCAsmInfo.h"
  18: #include "MCTargetDesc/SystemZMCTargetDesc.h"
  19: #include "SystemZConstantPoolValue.h"
  20: #include "SystemZMCInstLower.h"
  21: #include "TargetInfo/SystemZTargetInfo.h"
  22: #include "llvm/ADT/StringExtras.h"
  23: #include "llvm/BinaryFormat/ELF.h"
  24: #include "llvm/BinaryFormat/GOFF.h"
  25: #include "llvm/CodeGen/MachineModuleInfoImpls.h"
  26: #include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
  27: #include "llvm/IR/GlobalVariable.h"
  28: #include "llvm/IR/Mangler.h"
  29: #include "llvm/IR/Module.h"
  30: #include "llvm/MC/MCDirectives.h"
  31: #include "llvm/MC/MCExpr.h"
  32: #include "llvm/MC/MCInstBuilder.h"
  33: #include "llvm/MC/MCSectionELF.h"
  34: #include "llvm/MC/MCStreamer.h"
  35: #include "llvm/MC/MCSymbolGOFF.h"
  36: #include "llvm/MC/TargetRegistry.h"
  37: #include "llvm/Support/Chrono.h"
  38: #include "llvm/Support/Compiler.h"
  39: #include "llvm/Support/ConvertEBCDIC.h"
  40: #include "llvm/Support/FormatVariadic.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZAsmPrinter.h`, `SystemZGNUInstPrinter.h`, `SystemZHLASMInstPrinter.h`, `SystemZMCAsmInfo.h`, `SystemZMCTargetDesc.h`, `SystemZConstantPoolValue.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZAsmPrinter.h`, `SystemZGNUInstPrinter.h`, `SystemZHLASMInstPrinter.h`, `SystemZMCAsmInfo.h`, `SystemZMCTargetDesc.h`, `SystemZConstantPoolValue.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 41-80 / 第 41-80 行
```cpp
  41: 
  42: using namespace llvm;
  43: 
  44: // Return an RI instruction like MI with opcode Opcode, but with the
  45: // GR64 register operands turned into GR32s.
  46: static MCInst lowerRILow(const MachineInstr *MI, unsigned Opcode) {
  47:   if (MI->isCompare())
  48:     return MCInstBuilder(Opcode)
  49:       .addReg(SystemZMC::getRegAsGR32(MI->getOperand(0).getReg()))
  50:       .addImm(MI->getOperand(1).getImm());
  51:   else
  52:     return MCInstBuilder(Opcode)
  53:       .addReg(SystemZMC::getRegAsGR32(MI->getOperand(0).getReg()))
  54:       .addReg(SystemZMC::getRegAsGR32(MI->getOperand(1).getReg()))
  55:       .addImm(MI->getOperand(2).getImm());
  56: }
  57: 
  58: // Return an RI instruction like MI with opcode Opcode, but with the
  59: // GR64 register operands turned into GRH32s.
  60: static MCInst lowerRIHigh(const MachineInstr *MI, unsigned Opcode) {
  61:   if (MI->isCompare())
  62:     return MCInstBuilder(Opcode)
  63:       .addReg(SystemZMC::getRegAsGRH32(MI->getOperand(0).getReg()))
  64:       .addImm(MI->getOperand(1).getImm());
  65:   else
  66:     return MCInstBuilder(Opcode)
  67:       .addReg(SystemZMC::getRegAsGRH32(MI->getOperand(0).getReg()))
  68:       .addReg(SystemZMC::getRegAsGRH32(MI->getOperand(1).getReg()))
  69:       .addImm(MI->getOperand(2).getImm());
  70: }
  71: 
  72: // Return an RI instruction like MI with opcode Opcode, but with the
  73: // R2 register turned into a GR64.
  74: static MCInst lowerRIEfLow(const MachineInstr *MI, unsigned Opcode) {
  75:   return MCInstBuilder(Opcode)
  76:     .addReg(MI->getOperand(0).getReg())
  77:     .addReg(MI->getOperand(1).getReg())
  78:     .addReg(SystemZMC::getRegAsGR64(MI->getOperand(2).getReg()))
  79:     .addImm(MI->getOperand(3).getImm())
  80:     .addImm(MI->getOperand(4).getImm())
```
- **EN**: The range implements or declares functions including `lowerRILow`, `MCInstBuilder`, `lowerRIHigh`, `MCInstBuilder`, `lowerRIEfLow`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `lowerRILow`, `MCInstBuilder`, `lowerRIHigh`, `MCInstBuilder`, `lowerRIEfLow` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 81-120 / 第 81-120 行
```cpp
  81:     .addImm(MI->getOperand(5).getImm());
  82: }
  83: 
  84: static const MCSymbolRefExpr *getTLSGetOffset(MCContext &Context) {
  85:   StringRef Name = "__tls_get_offset";
  86:   return MCSymbolRefExpr::create(Context.getOrCreateSymbol(Name),
  87:                                  SystemZ::S_PLT, Context);
  88: }
  89: 
  90: static const MCSymbolRefExpr *getGlobalOffsetTable(MCContext &Context) {
  91:   StringRef Name = "_GLOBAL_OFFSET_TABLE_";
  92:   return MCSymbolRefExpr::create(Context.getOrCreateSymbol(Name),
  93:                                  Context);
  94: }
  95: 
  96: // MI is an instruction that accepts an optional alignment hint,
  97: // and which was already lowered to LoweredMI.  If the alignment
  98: // of the original memory operand is known, update LoweredMI to
  99: // an instruction with the corresponding hint set.
 100: static void lowerAlignmentHint(const MachineInstr *MI, MCInst &LoweredMI,
 101:                                unsigned Opcode) {
 102:   if (MI->memoperands_empty())
 103:     return;
 104: 
 105:   Align Alignment = Align(16);
 106:   for (MachineInstr::mmo_iterator MMOI = MI->memoperands_begin(),
 107:          EE = MI->memoperands_end(); MMOI != EE; ++MMOI)
 108:     if ((*MMOI)->getAlign() < Alignment)
 109:       Alignment = (*MMOI)->getAlign();
 110: 
 111:   unsigned AlignmentHint = 0;
 112:   if (Alignment >= Align(16))
 113:     AlignmentHint = 4;
 114:   else if (Alignment >= Align(8))
 115:     AlignmentHint = 3;
 116:   if (AlignmentHint == 0)
 117:     return;
 118: 
 119:   LoweredMI.setOpcode(Opcode);
 120:   LoweredMI.addOperand(MCOperand::createImm(AlignmentHint));
```
- **EN**: The range implements or declares functions including `lowerAlignmentHint`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerAlignmentHint` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 121-160 / 第 121-160 行
```cpp
 121: }
 122: 
 123: // MI loads the high part of a vector from memory.  Return an instruction
 124: // that uses replicating vector load Opcode to do the same thing.
 125: static MCInst lowerSubvectorLoad(const MachineInstr *MI, unsigned Opcode) {
 126:   return MCInstBuilder(Opcode)
 127:     .addReg(SystemZMC::getRegAsVR128(MI->getOperand(0).getReg()))
 128:     .addReg(MI->getOperand(1).getReg())
 129:     .addImm(MI->getOperand(2).getImm())
 130:     .addReg(MI->getOperand(3).getReg());
 131: }
 132: 
 133: // MI stores the high part of a vector to memory.  Return an instruction
 134: // that uses elemental vector store Opcode to do the same thing.
 135: static MCInst lowerSubvectorStore(const MachineInstr *MI, unsigned Opcode) {
 136:   return MCInstBuilder(Opcode)
 137:     .addReg(SystemZMC::getRegAsVR128(MI->getOperand(0).getReg()))
 138:     .addReg(MI->getOperand(1).getReg())
 139:     .addImm(MI->getOperand(2).getImm())
 140:     .addReg(MI->getOperand(3).getReg())
 141:     .addImm(0);
 142: }
 143: 
 144: // MI extracts the first element of the source vector.
 145: static MCInst lowerVecEltExtraction(const MachineInstr *MI, unsigned Opcode) {
 146:   return MCInstBuilder(Opcode)
 147:       .addReg(SystemZMC::getRegAsGR64(MI->getOperand(0).getReg()))
 148:       .addReg(SystemZMC::getRegAsVR128(MI->getOperand(1).getReg()))
 149:       .addReg(0)
 150:       .addImm(0);
 151: }
 152: 
 153: // MI inserts value into the first element of the destination vector.
 154: static MCInst lowerVecEltInsertion(const MachineInstr *MI, unsigned Opcode) {
 155:   return MCInstBuilder(Opcode)
 156:       .addReg(SystemZMC::getRegAsVR128(MI->getOperand(0).getReg()))
 157:       .addReg(SystemZMC::getRegAsVR128(MI->getOperand(0).getReg()))
 158:       .addReg(MI->getOperand(1).getReg())
 159:       .addReg(0)
 160:       .addImm(0);
```
- **EN**: The range implements or declares functions including `lowerSubvectorLoad`, `lowerSubvectorStore`, `lowerVecEltExtraction`, `lowerVecEltInsertion`.
- **CN**: 这一段实现或声明了 `lowerSubvectorLoad`, `lowerSubvectorStore`, `lowerVecEltExtraction`, `lowerVecEltInsertion` 等函数。

### Lines 161-200 / 第 161-200 行
```cpp
 161: }
 162: 
 163: bool SystemZAsmPrinter::doInitialization(Module &M) {
 164:   SM.reset();
 165: 
 166:   // In HLASM, the only way to represent aliases is to use the
 167:   // extra-label-at-definition strategy. This is similar to the AIX
 168:   // implementation with the additional caveat that all symbol attributes must
 169:   // be emitted before the label is emitted.
 170:   if (TM.getTargetTriple().isOSzOS()) {
 171:     // Construct an aliasing list for each GlobalObject.
 172:     for (const auto &Alias : M.aliases()) {
 173:       const GlobalObject *Aliasee = Alias.getAliaseeObject();
 174:       if (!Aliasee)
 175:         OutContext.reportError(
 176:             {}, "Alias without a base object is not yet supported on z/OS.");
 177: 
 178:       bool IsFunc = isa<Function>(Aliasee->stripPointerCasts());
 179:       if (IsFunc) {
 180:         if (Alias.hasWeakLinkage() || Alias.hasLinkOnceLinkage())
 181:           OutContext.reportError({},
 182:                                  "Weak alias/reference not supported on z/OS");
 183: 
 184:         GOAliasMap[Aliasee].push_back(&Alias);
 185:       } else
 186:         OutContext.reportError(
 187:             {}, "Only aliases to functions is supported in GOFF.");
 188:     }
 189:   }
 190:   return AsmPrinter::doInitialization(M);
 191: }
 192: 
 193: // The XPLINK ABI requires that a no-op encoding the call type is emitted after
 194: // each call to a subroutine. This information can be used by the called
 195: // function to determine its entry point, e.g. for generating a backtrace. The
 196: // call type is encoded as a register number in the bcr instruction. See
 197: // enumeration CallType for the possible values.
 198: void SystemZAsmPrinter::emitCallInformation(CallType CT) {
 199:   EmitToStreamer(*OutStreamer,
 200:                  MCInstBuilder(SystemZ::BCRAsm)
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::doInitialization`, `SystemZAsmPrinter::emitCallInformation`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::doInitialization`, `SystemZAsmPrinter::emitCallInformation` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 201-240 / 第 201-240 行
```cpp
 201:                      .addImm(0)
 202:                      .addReg(SystemZMC::GR64Regs[static_cast<unsigned>(CT)]));
 203: }
 204: 
 205: uint32_t SystemZAsmPrinter::AssociatedDataAreaTable::insert(const MCSymbol *Sym,
 206:                                                             unsigned SlotKind) {
 207:   auto Key = std::make_pair(Sym, SlotKind);
 208:   auto It = Displacements.find(Key);
 209: 
 210:   if (It != Displacements.end())
 211:     return (*It).second;
 212: 
 213:   // Determine length of descriptor.
 214:   uint32_t Length;
 215:   switch (SlotKind) {
 216:   case SystemZII::MO_ADA_DIRECT_FUNC_DESC:
 217:     Length = 2 * PointerSize;
 218:     break;
 219:   default:
 220:     Length = PointerSize;
 221:     break;
 222:   }
 223: 
 224:   uint32_t Displacement = NextDisplacement;
 225:   Displacements[std::make_pair(Sym, SlotKind)] = NextDisplacement;
 226:   NextDisplacement += Length;
 227: 
 228:   return Displacement;
 229: }
 230: 
 231: uint32_t
 232: SystemZAsmPrinter::AssociatedDataAreaTable::insert(const MachineOperand MO) {
 233:   MCSymbol *Sym;
 234:   if (MO.getType() == MachineOperand::MO_GlobalAddress) {
 235:     const GlobalValue *GV = MO.getGlobal();
 236:     Sym = MO.getParent()->getMF()->getTarget().getSymbol(GV);
 237:     assert(Sym && "No symbol");
 238:   } else if (MO.getType() == MachineOperand::MO_ExternalSymbol) {
 239:     const char *SymName = MO.getSymbolName();
 240:     Sym = MO.getParent()->getMF()->getContext().getOrCreateSymbol(SymName);
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::AssociatedDataAreaTable::insert`, `SystemZAsmPrinter::AssociatedDataAreaTable::insert`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::AssociatedDataAreaTable::insert`, `SystemZAsmPrinter::AssociatedDataAreaTable::insert` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 241-280 / 第 241-280 行
```cpp
 241:     assert(Sym && "No symbol");
 242:   } else
 243:     llvm_unreachable("Unexpected operand type");
 244: 
 245:   unsigned ADAslotType = MO.getTargetFlags();
 246:   return insert(Sym, ADAslotType);
 247: }
 248: 
 249: void SystemZAsmPrinter::emitInstruction(const MachineInstr *MI) {
 250:   SystemZ_MC::verifyInstructionPredicates(MI->getOpcode(),
 251:                                           getSubtargetInfo().getFeatureBits());
 252: 
 253:   SystemZMCInstLower Lower(MF->getContext(), *this);
 254:   MCInst LoweredMI;
 255:   switch (MI->getOpcode()) {
 256:   case SystemZ::Return:
 257:     LoweredMI = MCInstBuilder(SystemZ::BR)
 258:       .addReg(SystemZ::R14D);
 259:     break;
 260: 
 261:   case SystemZ::Return_XPLINK:
 262:     LoweredMI = MCInstBuilder(SystemZ::B)
 263:       .addReg(SystemZ::R7D)
 264:       .addImm(2)
 265:       .addReg(0);
 266:     break;
 267: 
 268:   case SystemZ::CondReturn:
 269:     LoweredMI = MCInstBuilder(SystemZ::BCR)
 270:       .addImm(MI->getOperand(0).getImm())
 271:       .addImm(MI->getOperand(1).getImm())
 272:       .addReg(SystemZ::R14D);
 273:     break;
 274: 
 275:   case SystemZ::CondReturn_XPLINK:
 276:     LoweredMI = MCInstBuilder(SystemZ::BC)
 277:       .addImm(MI->getOperand(0).getImm())
 278:       .addImm(MI->getOperand(1).getImm())
 279:       .addReg(SystemZ::R7D)
 280:       .addImm(2)
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::emitInstruction`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::emitInstruction` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 281-320 / 第 281-320 行
```cpp
 281:       .addReg(0);
 282:     break;
 283: 
 284:   case SystemZ::CRBReturn:
 285:     LoweredMI = MCInstBuilder(SystemZ::CRB)
 286:       .addReg(MI->getOperand(0).getReg())
 287:       .addReg(MI->getOperand(1).getReg())
 288:       .addImm(MI->getOperand(2).getImm())
 289:       .addReg(SystemZ::R14D)
 290:       .addImm(0);
 291:     break;
 292: 
 293:   case SystemZ::CGRBReturn:
 294:     LoweredMI = MCInstBuilder(SystemZ::CGRB)
 295:       .addReg(MI->getOperand(0).getReg())
 296:       .addReg(MI->getOperand(1).getReg())
 297:       .addImm(MI->getOperand(2).getImm())
 298:       .addReg(SystemZ::R14D)
 299:       .addImm(0);
 300:     break;
 301: 
 302:   case SystemZ::CIBReturn:
 303:     LoweredMI = MCInstBuilder(SystemZ::CIB)
 304:       .addReg(MI->getOperand(0).getReg())
 305:       .addImm(MI->getOperand(1).getImm())
 306:       .addImm(MI->getOperand(2).getImm())
 307:       .addReg(SystemZ::R14D)
 308:       .addImm(0);
 309:     break;
 310: 
 311:   case SystemZ::CGIBReturn:
 312:     LoweredMI = MCInstBuilder(SystemZ::CGIB)
 313:       .addReg(MI->getOperand(0).getReg())
 314:       .addImm(MI->getOperand(1).getImm())
 315:       .addImm(MI->getOperand(2).getImm())
 316:       .addReg(SystemZ::R14D)
 317:       .addImm(0);
 318:     break;
 319: 
 320:   case SystemZ::CLRBReturn:
```
- **EN**: This span continues the file's main responsibility: this file emits textual assembly from LLVM machine instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 321-360 / 第 321-360 行
```cpp
 321:     LoweredMI = MCInstBuilder(SystemZ::CLRB)
 322:       .addReg(MI->getOperand(0).getReg())
 323:       .addReg(MI->getOperand(1).getReg())
 324:       .addImm(MI->getOperand(2).getImm())
 325:       .addReg(SystemZ::R14D)
 326:       .addImm(0);
 327:     break;
 328: 
 329:   case SystemZ::CLGRBReturn:
 330:     LoweredMI = MCInstBuilder(SystemZ::CLGRB)
 331:       .addReg(MI->getOperand(0).getReg())
 332:       .addReg(MI->getOperand(1).getReg())
 333:       .addImm(MI->getOperand(2).getImm())
 334:       .addReg(SystemZ::R14D)
 335:       .addImm(0);
 336:     break;
 337: 
 338:   case SystemZ::CLIBReturn:
 339:     LoweredMI = MCInstBuilder(SystemZ::CLIB)
 340:       .addReg(MI->getOperand(0).getReg())
 341:       .addImm(MI->getOperand(1).getImm())
 342:       .addImm(MI->getOperand(2).getImm())
 343:       .addReg(SystemZ::R14D)
 344:       .addImm(0);
 345:     break;
 346: 
 347:   case SystemZ::CLGIBReturn:
 348:     LoweredMI = MCInstBuilder(SystemZ::CLGIB)
 349:       .addReg(MI->getOperand(0).getReg())
 350:       .addImm(MI->getOperand(1).getImm())
 351:       .addImm(MI->getOperand(2).getImm())
 352:       .addReg(SystemZ::R14D)
 353:       .addImm(0);
 354:     break;
 355: 
 356:   case SystemZ::CallBRASL_XPLINK64:
 357:     EmitToStreamer(*OutStreamer, MCInstBuilder(SystemZ::BRASL)
 358:                                      .addReg(SystemZ::R7D)
 359:                                      .addExpr(Lower.getExpr(MI->getOperand(0),
 360:                                                             SystemZ::S_None)));
```
- **EN**: The range implements or declares functions including `EmitToStreamer`.
- **CN**: 这一段实现或声明了 `EmitToStreamer` 等函数。

### Lines 361-400 / 第 361-400 行
```cpp
 361:     emitCallInformation(CallType::BRASL7);
 362:     return;
 363: 
 364:   case SystemZ::CallBASR_XPLINK64:
 365:     EmitToStreamer(*OutStreamer, MCInstBuilder(SystemZ::BASR)
 366:                                      .addReg(SystemZ::R7D)
 367:                                      .addReg(MI->getOperand(0).getReg()));
 368:     emitCallInformation(CallType::BASR76);
 369:     return;
 370: 
 371:   case SystemZ::CallBASR_STACKEXT:
 372:     EmitToStreamer(*OutStreamer, MCInstBuilder(SystemZ::BASR)
 373:                                      .addReg(SystemZ::R3D)
 374:                                      .addReg(MI->getOperand(0).getReg()));
 375:     emitCallInformation(CallType::BASR33);
 376:     return;
 377: 
 378:   case SystemZ::ADA_ENTRY_VALUE:
 379:   case SystemZ::ADA_ENTRY: {
 380:     const SystemZSubtarget &Subtarget = MF->getSubtarget<SystemZSubtarget>();
 381:     const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
 382:     uint32_t Disp = ADATable.insert(MI->getOperand(1));
 383:     Register TargetReg = MI->getOperand(0).getReg();
 384: 
 385:     Register ADAReg = MI->getOperand(2).getReg();
 386:     Disp += MI->getOperand(3).getImm();
 387:     bool LoadAddr = MI->getOpcode() == SystemZ::ADA_ENTRY;
 388: 
 389:     unsigned Op0 = LoadAddr ? SystemZ::LA : SystemZ::LG;
 390:     unsigned Op = TII->getOpcodeForOffset(Op0, Disp);
 391: 
 392:     Register IndexReg = 0;
 393:     if (!Op) {
 394:       if (TargetReg != ADAReg) {
 395:         IndexReg = TargetReg;
 396:         // Use TargetReg to store displacement.
 397:         EmitToStreamer(
 398:             *OutStreamer,
 399:             MCInstBuilder(SystemZ::LLILF).addReg(TargetReg).addImm(Disp));
 400:       } else
```
- **EN**: The range implements or declares functions including `EmitToStreamer`, `EmitToStreamer`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `EmitToStreamer`, `EmitToStreamer` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 401-440 / 第 401-440 行
```cpp
 401:         EmitToStreamer(*OutStreamer, MCInstBuilder(SystemZ::ALGFI)
 402:                                          .addReg(TargetReg)
 403:                                          .addReg(TargetReg)
 404:                                          .addImm(Disp));
 405:       Disp = 0;
 406:       Op = Op0;
 407:     }
 408:     EmitToStreamer(*OutStreamer, MCInstBuilder(Op)
 409:                                      .addReg(TargetReg)
 410:                                      .addReg(ADAReg)
 411:                                      .addImm(Disp)
 412:                                      .addReg(IndexReg));
 413: 
 414:     return;
 415:   }
 416:   case SystemZ::CallBRASL:
 417:     LoweredMI = MCInstBuilder(SystemZ::BRASL)
 418:                     .addReg(SystemZ::R14D)
 419:                     .addExpr(Lower.getExpr(MI->getOperand(0), SystemZ::S_PLT));
 420:     break;
 421: 
 422:   case SystemZ::CallBASR:
 423:     LoweredMI = MCInstBuilder(SystemZ::BASR)
 424:       .addReg(SystemZ::R14D)
 425:       .addReg(MI->getOperand(0).getReg());
 426:     break;
 427: 
 428:   case SystemZ::CallJG:
 429:     LoweredMI = MCInstBuilder(SystemZ::JG)
 430:                     .addExpr(Lower.getExpr(MI->getOperand(0), SystemZ::S_PLT));
 431:     break;
 432: 
 433:   case SystemZ::CallBRCL:
 434:     LoweredMI = MCInstBuilder(SystemZ::BRCL)
 435:                     .addImm(MI->getOperand(0).getImm())
 436:                     .addImm(MI->getOperand(1).getImm())
 437:                     .addExpr(Lower.getExpr(MI->getOperand(2), SystemZ::S_PLT));
 438:     break;
 439: 
 440:   case SystemZ::CallBR:
```
- **EN**: The range implements or declares functions including `EmitToStreamer`, `EmitToStreamer`.
- **CN**: 这一段实现或声明了 `EmitToStreamer`, `EmitToStreamer` 等函数。

### Lines 441-480 / 第 441-480 行
```cpp
 441:     LoweredMI = MCInstBuilder(SystemZ::BR)
 442:       .addReg(MI->getOperand(0).getReg());
 443:     break;
 444: 
 445:   case SystemZ::CallBCR:
 446:     LoweredMI = MCInstBuilder(SystemZ::BCR)
 447:       .addImm(MI->getOperand(0).getImm())
 448:       .addImm(MI->getOperand(1).getImm())
 449:       .addReg(MI->getOperand(2).getReg());
 450:     break;
 451: 
 452:   case SystemZ::CRBCall:
 453:     LoweredMI = MCInstBuilder(SystemZ::CRB)
 454:       .addReg(MI->getOperand(0).getReg())
 455:       .addReg(MI->getOperand(1).getReg())
 456:       .addImm(MI->getOperand(2).getImm())
 457:       .addReg(MI->getOperand(3).getReg())
 458:       .addImm(0);
 459:     break;
 460: 
 461:   case SystemZ::CGRBCall:
 462:     LoweredMI = MCInstBuilder(SystemZ::CGRB)
 463:       .addReg(MI->getOperand(0).getReg())
 464:       .addReg(MI->getOperand(1).getReg())
 465:       .addImm(MI->getOperand(2).getImm())
 466:       .addReg(MI->getOperand(3).getReg())
 467:       .addImm(0);
 468:     break;
 469: 
 470:   case SystemZ::CIBCall:
 471:     LoweredMI = MCInstBuilder(SystemZ::CIB)
 472:       .addReg(MI->getOperand(0).getReg())
 473:       .addImm(MI->getOperand(1).getImm())
 474:       .addImm(MI->getOperand(2).getImm())
 475:       .addReg(MI->getOperand(3).getReg())
 476:       .addImm(0);
 477:     break;
 478: 
 479:   case SystemZ::CGIBCall:
 480:     LoweredMI = MCInstBuilder(SystemZ::CGIB)
```
- **EN**: This span continues the file's main responsibility: this file emits textual assembly from LLVM machine instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 481-520 / 第 481-520 行
```cpp
 481:       .addReg(MI->getOperand(0).getReg())
 482:       .addImm(MI->getOperand(1).getImm())
 483:       .addImm(MI->getOperand(2).getImm())
 484:       .addReg(MI->getOperand(3).getReg())
 485:       .addImm(0);
 486:     break;
 487: 
 488:   case SystemZ::CLRBCall:
 489:     LoweredMI = MCInstBuilder(SystemZ::CLRB)
 490:       .addReg(MI->getOperand(0).getReg())
 491:       .addReg(MI->getOperand(1).getReg())
 492:       .addImm(MI->getOperand(2).getImm())
 493:       .addReg(MI->getOperand(3).getReg())
 494:       .addImm(0);
 495:     break;
 496: 
 497:   case SystemZ::CLGRBCall:
 498:     LoweredMI = MCInstBuilder(SystemZ::CLGRB)
 499:       .addReg(MI->getOperand(0).getReg())
 500:       .addReg(MI->getOperand(1).getReg())
 501:       .addImm(MI->getOperand(2).getImm())
 502:       .addReg(MI->getOperand(3).getReg())
 503:       .addImm(0);
 504:     break;
 505: 
 506:   case SystemZ::CLIBCall:
 507:     LoweredMI = MCInstBuilder(SystemZ::CLIB)
 508:       .addReg(MI->getOperand(0).getReg())
 509:       .addImm(MI->getOperand(1).getImm())
 510:       .addImm(MI->getOperand(2).getImm())
 511:       .addReg(MI->getOperand(3).getReg())
 512:       .addImm(0);
 513:     break;
 514: 
 515:   case SystemZ::CLGIBCall:
 516:     LoweredMI = MCInstBuilder(SystemZ::CLGIB)
 517:       .addReg(MI->getOperand(0).getReg())
 518:       .addImm(MI->getOperand(1).getImm())
 519:       .addImm(MI->getOperand(2).getImm())
 520:       .addReg(MI->getOperand(3).getReg())
```
- **EN**: This span continues the file's main responsibility: this file emits textual assembly from LLVM machine instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 521-560 / 第 521-560 行
```cpp
 521:       .addImm(0);
 522:     break;
 523: 
 524:   case SystemZ::TLS_GDCALL:
 525:     LoweredMI =
 526:         MCInstBuilder(SystemZ::BRASL)
 527:             .addReg(SystemZ::R14D)
 528:             .addExpr(getTLSGetOffset(MF->getContext()))
 529:             .addExpr(Lower.getExpr(MI->getOperand(0), SystemZ::S_TLSGD));
 530:     break;
 531: 
 532:   case SystemZ::TLS_LDCALL:
 533:     LoweredMI =
 534:         MCInstBuilder(SystemZ::BRASL)
 535:             .addReg(SystemZ::R14D)
 536:             .addExpr(getTLSGetOffset(MF->getContext()))
 537:             .addExpr(Lower.getExpr(MI->getOperand(0), SystemZ::S_TLSLDM));
 538:     break;
 539: 
 540:   case SystemZ::GOT:
 541:     LoweredMI = MCInstBuilder(SystemZ::LARL)
 542:       .addReg(MI->getOperand(0).getReg())
 543:       .addExpr(getGlobalOffsetTable(MF->getContext()));
 544:     break;
 545: 
 546:   case SystemZ::IILF64:
 547:     LoweredMI = MCInstBuilder(SystemZ::IILF)
 548:       .addReg(SystemZMC::getRegAsGR32(MI->getOperand(0).getReg()))
 549:       .addImm(MI->getOperand(2).getImm());
 550:     break;
 551: 
 552:   case SystemZ::IIHF64:
 553:     LoweredMI = MCInstBuilder(SystemZ::IIHF)
 554:       .addReg(SystemZMC::getRegAsGRH32(MI->getOperand(0).getReg()))
 555:       .addImm(MI->getOperand(2).getImm());
 556:     break;
 557: 
 558:   case SystemZ::RISBHH:
 559:   case SystemZ::RISBHL:
 560:     LoweredMI = lowerRIEfLow(MI, SystemZ::RISBHG);
```
- **EN**: The range implements or declares functions including `MCInstBuilder`, `MCInstBuilder`.
- **CN**: 这一段实现或声明了 `MCInstBuilder`, `MCInstBuilder` 等函数。

### Lines 561-600 / 第 561-600 行
```cpp
 561:     break;
 562: 
 563:   case SystemZ::RISBLH:
 564:   case SystemZ::RISBLL:
 565:     LoweredMI = lowerRIEfLow(MI, SystemZ::RISBLG);
 566:     break;
 567: 
 568:   case SystemZ::VLVGP32:
 569:     LoweredMI = MCInstBuilder(SystemZ::VLVGP)
 570:       .addReg(MI->getOperand(0).getReg())
 571:       .addReg(SystemZMC::getRegAsGR64(MI->getOperand(1).getReg()))
 572:       .addReg(SystemZMC::getRegAsGR64(MI->getOperand(2).getReg()));
 573:     break;
 574: 
 575:   case SystemZ::VLR16:
 576:   case SystemZ::VLR32:
 577:   case SystemZ::VLR64:
 578:     LoweredMI = MCInstBuilder(SystemZ::VLR)
 579:       .addReg(SystemZMC::getRegAsVR128(MI->getOperand(0).getReg()))
 580:       .addReg(SystemZMC::getRegAsVR128(MI->getOperand(1).getReg()));
 581:     break;
 582: 
 583:   case SystemZ::VL:
 584:     Lower.lower(MI, LoweredMI);
 585:     lowerAlignmentHint(MI, LoweredMI, SystemZ::VLAlign);
 586:     break;
 587: 
 588:   case SystemZ::VST:
 589:     Lower.lower(MI, LoweredMI);
 590:     lowerAlignmentHint(MI, LoweredMI, SystemZ::VSTAlign);
 591:     break;
 592: 
 593:   case SystemZ::VLM:
 594:     Lower.lower(MI, LoweredMI);
 595:     lowerAlignmentHint(MI, LoweredMI, SystemZ::VLMAlign);
 596:     break;
 597: 
 598:   case SystemZ::VSTM:
 599:     Lower.lower(MI, LoweredMI);
 600:     lowerAlignmentHint(MI, LoweredMI, SystemZ::VSTMAlign);
```
- **EN**: This span continues the file's main responsibility: this file emits textual assembly from LLVM machine instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 601-640 / 第 601-640 行
```cpp
 601:     break;
 602: 
 603:   case SystemZ::VL16:
 604:     LoweredMI = lowerSubvectorLoad(MI, SystemZ::VLREPH);
 605:     break;
 606: 
 607:   case SystemZ::VL32:
 608:     LoweredMI = lowerSubvectorLoad(MI, SystemZ::VLREPF);
 609:     break;
 610: 
 611:   case SystemZ::VL64:
 612:     LoweredMI = lowerSubvectorLoad(MI, SystemZ::VLREPG);
 613:     break;
 614: 
 615:   case SystemZ::VST16:
 616:     LoweredMI = lowerSubvectorStore(MI, SystemZ::VSTEH);
 617:     break;
 618: 
 619:   case SystemZ::VST32:
 620:     LoweredMI = lowerSubvectorStore(MI, SystemZ::VSTEF);
 621:     break;
 622: 
 623:   case SystemZ::VST64:
 624:     LoweredMI = lowerSubvectorStore(MI, SystemZ::VSTEG);
 625:     break;
 626: 
 627:   case SystemZ::LFER:
 628:     LoweredMI = lowerVecEltExtraction(MI, SystemZ::VLGVF);
 629:     break;
 630: 
 631:   case SystemZ::LFER_16:
 632:     LoweredMI = lowerVecEltExtraction(MI, SystemZ::VLGVH);
 633:     break;
 634: 
 635:   case SystemZ::LEFR:
 636:     LoweredMI = lowerVecEltInsertion(MI, SystemZ::VLVGF);
 637:     break;
 638: 
 639:   case SystemZ::LEFR_16:
 640:     LoweredMI = lowerVecEltInsertion(MI, SystemZ::VLVGH);
```
- **EN**: This span continues the file's main responsibility: this file emits textual assembly from LLVM machine instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 641-680 / 第 641-680 行
```cpp
 641:     break;
 642: 
 643: #define LOWER_LOW(NAME)                                                 \
 644:   case SystemZ::NAME##64: LoweredMI = lowerRILow(MI, SystemZ::NAME); break
 645: 
 646:   LOWER_LOW(IILL);
 647:   LOWER_LOW(IILH);
 648:   LOWER_LOW(TMLL);
 649:   LOWER_LOW(TMLH);
 650:   LOWER_LOW(NILL);
 651:   LOWER_LOW(NILH);
 652:   LOWER_LOW(NILF);
 653:   LOWER_LOW(OILL);
 654:   LOWER_LOW(OILH);
 655:   LOWER_LOW(OILF);
 656:   LOWER_LOW(XILF);
 657: 
 658: #undef LOWER_LOW
 659: 
 660: #define LOWER_HIGH(NAME) \
 661:   case SystemZ::NAME##64: LoweredMI = lowerRIHigh(MI, SystemZ::NAME); break
 662: 
 663:   LOWER_HIGH(IIHL);
 664:   LOWER_HIGH(IIHH);
 665:   LOWER_HIGH(TMHL);
 666:   LOWER_HIGH(TMHH);
 667:   LOWER_HIGH(NIHL);
 668:   LOWER_HIGH(NIHH);
 669:   LOWER_HIGH(NIHF);
 670:   LOWER_HIGH(OIHL);
 671:   LOWER_HIGH(OIHH);
 672:   LOWER_HIGH(OIHF);
 673:   LOWER_HIGH(XIHF);
 674: 
 675: #undef LOWER_HIGH
 676: 
 677:   case SystemZ::Serialize:
 678:     if (MF->getSubtarget<SystemZSubtarget>().hasFastSerialization())
 679:       LoweredMI = MCInstBuilder(SystemZ::BCRAsm)
 680:         .addImm(14).addReg(SystemZ::R0D);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 681-720 / 第 681-720 行
```cpp
 681:     else
 682:       LoweredMI = MCInstBuilder(SystemZ::BCRAsm)
 683:         .addImm(15).addReg(SystemZ::R0D);
 684:     break;
 685: 
 686:   // We want to emit "j .+2" for traps, jumping to the relative immediate field
 687:   // of the jump instruction, which is an illegal instruction. We cannot emit a
 688:   // "." symbol, so create and emit a temp label before the instruction and use
 689:   // that instead.
 690:   case SystemZ::Trap: {
 691:     MCSymbol *DotSym = OutContext.createTempSymbol();
 692:     OutStreamer->emitLabel(DotSym);
 693: 
 694:     const MCSymbolRefExpr *Expr = MCSymbolRefExpr::create(DotSym, OutContext);
 695:     const MCConstantExpr *ConstExpr = MCConstantExpr::create(2, OutContext);
 696:     LoweredMI = MCInstBuilder(SystemZ::J)
 697:       .addExpr(MCBinaryExpr::createAdd(Expr, ConstExpr, OutContext));
 698:     }
 699:     break;
 700: 
 701:   // Conditional traps will create a branch on condition instruction that jumps
 702:   // to the relative immediate field of the jump instruction. (eg. "jo .+2")
 703:   case SystemZ::CondTrap: {
 704:     MCSymbol *DotSym = OutContext.createTempSymbol();
 705:     OutStreamer->emitLabel(DotSym);
 706: 
 707:     const MCSymbolRefExpr *Expr = MCSymbolRefExpr::create(DotSym, OutContext);
 708:     const MCConstantExpr *ConstExpr = MCConstantExpr::create(2, OutContext);
 709:     LoweredMI = MCInstBuilder(SystemZ::BRC)
 710:       .addImm(MI->getOperand(0).getImm())
 711:       .addImm(MI->getOperand(1).getImm())
 712:       .addExpr(MCBinaryExpr::createAdd(Expr, ConstExpr, OutContext));
 713:     }
 714:     break;
 715: 
 716:   case TargetOpcode::FENTRY_CALL:
 717:     LowerFENTRY_CALL(*MI, Lower);
 718:     return;
 719: 
 720:   case TargetOpcode::STACKMAP:
```
- **EN**: This span continues the file's main responsibility: this file emits textual assembly from LLVM machine instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 721-760 / 第 721-760 行
```cpp
 721:     LowerSTACKMAP(*MI);
 722:     return;
 723: 
 724:   case TargetOpcode::PATCHPOINT:
 725:     LowerPATCHPOINT(*MI, Lower);
 726:     return;
 727: 
 728:   case TargetOpcode::PATCHABLE_FUNCTION_ENTER:
 729:     LowerPATCHABLE_FUNCTION_ENTER(*MI, Lower);
 730:     return;
 731: 
 732:   case TargetOpcode::PATCHABLE_RET:
 733:     LowerPATCHABLE_RET(*MI, Lower);
 734:     return;
 735: 
 736:   case TargetOpcode::PATCHABLE_FUNCTION_EXIT:
 737:     llvm_unreachable("PATCHABLE_FUNCTION_EXIT should never be emitted");
 738: 
 739:   case TargetOpcode::PATCHABLE_TAIL_CALL:
 740:     // TODO: Define a trampoline `__xray_FunctionTailExit` and differentiate a
 741:     // normal function exit from a tail exit.
 742:     llvm_unreachable("Tail call is handled in the normal case. See comments "
 743:                      "around this assert.");
 744: 
 745:   case SystemZ::EXRL_Pseudo: {
 746:     unsigned TargetInsOpc = MI->getOperand(0).getImm();
 747:     Register LenMinus1Reg = MI->getOperand(1).getReg();
 748:     Register DestReg = MI->getOperand(2).getReg();
 749:     int64_t DestDisp = MI->getOperand(3).getImm();
 750:     Register SrcReg = MI->getOperand(4).getReg();
 751:     int64_t SrcDisp = MI->getOperand(5).getImm();
 752: 
 753:     SystemZTargetStreamer *TS = getTargetStreamer();
 754:     MCInst ET = MCInstBuilder(TargetInsOpc)
 755:                     .addReg(DestReg)
 756:                     .addImm(DestDisp)
 757:                     .addImm(1)
 758:                     .addReg(SrcReg)
 759:                     .addImm(SrcDisp);
 760:     SystemZTargetStreamer::MCInstSTIPair ET_STI(ET, &MF->getSubtarget());
```
- **EN**: The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 761-800 / 第 761-800 行
```cpp
 761:     auto [It, Inserted] = TS->EXRLTargets2Sym.try_emplace(ET_STI);
 762:     if (Inserted)
 763:       It->second = OutContext.createTempSymbol();
 764:     MCSymbol *DotSym = It->second;
 765:     const MCSymbolRefExpr *Dot = MCSymbolRefExpr::create(DotSym, OutContext);
 766:     EmitToStreamer(
 767:         *OutStreamer,
 768:         MCInstBuilder(SystemZ::EXRL).addReg(LenMinus1Reg).addExpr(Dot));
 769:     return;
 770:   }
 771: 
 772:   // EH_SjLj_Setup is a dummy terminator instruction of size 0.
 773:   // It is used to handle the clobber register for builtin setjmp.
 774:   case SystemZ::EH_SjLj_Setup:
 775:     return;
 776: 
 777:   default:
 778:     Lower.lower(MI, LoweredMI);
 779:     break;
 780:   }
 781:   EmitToStreamer(*OutStreamer, LoweredMI);
 782: }
 783: 
 784: // Emit the largest nop instruction smaller than or equal to NumBytes
 785: // bytes.  Return the size of nop emitted.
 786: static unsigned EmitNop(MCContext &OutContext, MCStreamer &OutStreamer,
 787:                         unsigned NumBytes, const MCSubtargetInfo &STI) {
 788:   if (NumBytes < 2) {
 789:     llvm_unreachable("Zero nops?");
 790:     return 0;
 791:   }
 792:   else if (NumBytes < 4) {
 793:     OutStreamer.emitInstruction(
 794:         MCInstBuilder(SystemZ::BCRAsm).addImm(0).addReg(SystemZ::R0D), STI);
 795:     return 2;
 796:   }
 797:   else if (NumBytes < 6) {
 798:     OutStreamer.emitInstruction(
 799:         MCInstBuilder(SystemZ::BCAsm).addImm(0).addReg(0).addImm(0).addReg(0),
 800:         STI);
```
- **EN**: The range implements or declares functions including `EmitNop`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `EmitNop` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 801-840 / 第 801-840 行
```cpp
 801:     return 4;
 802:   }
 803:   else {
 804:     MCSymbol *DotSym = OutContext.createTempSymbol();
 805:     const MCSymbolRefExpr *Dot = MCSymbolRefExpr::create(DotSym, OutContext);
 806:     OutStreamer.emitLabel(DotSym);
 807:     OutStreamer.emitInstruction(
 808:         MCInstBuilder(SystemZ::BRCLAsm).addImm(0).addExpr(Dot), STI);
 809:     return 6;
 810:   }
 811: }
 812: 
 813: void SystemZAsmPrinter::LowerFENTRY_CALL(const MachineInstr &MI,
 814:                                          SystemZMCInstLower &Lower) {
 815:   MCContext &Ctx = MF->getContext();
 816:   if (MF->getFunction().hasFnAttribute("mrecord-mcount")) {
 817:     MCSymbol *DotSym = OutContext.createTempSymbol();
 818:     OutStreamer->pushSection();
 819:     OutStreamer->switchSection(
 820:         Ctx.getELFSection("__mcount_loc", ELF::SHT_PROGBITS, ELF::SHF_ALLOC));
 821:     OutStreamer->emitSymbolValue(DotSym, 8);
 822:     OutStreamer->popSection();
 823:     OutStreamer->emitLabel(DotSym);
 824:   }
 825: 
 826:   if (MF->getFunction().hasFnAttribute("mnop-mcount")) {
 827:     EmitNop(Ctx, *OutStreamer, 6, getSubtargetInfo());
 828:     return;
 829:   }
 830: 
 831:   MCSymbol *fentry = Ctx.getOrCreateSymbol("__fentry__");
 832:   const MCSymbolRefExpr *Op =
 833:       MCSymbolRefExpr::create(fentry, SystemZ::S_PLT, Ctx);
 834:   OutStreamer->emitInstruction(
 835:       MCInstBuilder(SystemZ::BRASL).addReg(SystemZ::R0D).addExpr(Op),
 836:       getSubtargetInfo());
 837: }
 838: 
 839: void SystemZAsmPrinter::LowerSTACKMAP(const MachineInstr &MI) {
 840:   auto *TII = MF->getSubtarget<SystemZSubtarget>().getInstrInfo();
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::LowerFENTRY_CALL`, `SystemZAsmPrinter::LowerSTACKMAP`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::LowerFENTRY_CALL`, `SystemZAsmPrinter::LowerSTACKMAP` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 841-880 / 第 841-880 行
```cpp
 841: 
 842:   unsigned NumNOPBytes = MI.getOperand(1).getImm();
 843: 
 844:   auto &Ctx = OutStreamer->getContext();
 845:   MCSymbol *MILabel = Ctx.createTempSymbol();
 846:   OutStreamer->emitLabel(MILabel);
 847:   
 848:   SM.recordStackMap(*MILabel, MI);
 849:   assert(NumNOPBytes % 2 == 0 && "Invalid number of NOP bytes requested!");
 850: 
 851:   // Scan ahead to trim the shadow.
 852:   unsigned ShadowBytes = 0;
 853:   const MachineBasicBlock &MBB = *MI.getParent();
 854:   MachineBasicBlock::const_iterator MII(MI);
 855:   ++MII;
 856:   while (ShadowBytes < NumNOPBytes) {
 857:     if (MII == MBB.end() ||
 858:         MII->getOpcode() == TargetOpcode::PATCHPOINT ||
 859:         MII->getOpcode() == TargetOpcode::STACKMAP)
 860:       break;
 861:     ShadowBytes += TII->getInstSizeInBytes(*MII);
 862:     if (MII->isCall())
 863:       break;
 864:     ++MII;
 865:   }
 866: 
 867:   // Emit nops.
 868:   while (ShadowBytes < NumNOPBytes)
 869:     ShadowBytes += EmitNop(OutContext, *OutStreamer, NumNOPBytes - ShadowBytes,
 870:                            getSubtargetInfo());
 871: }
 872: 
 873: // Lower a patchpoint of the form:
 874: // [<def>], <id>, <numBytes>, <target>, <numArgs>
 875: void SystemZAsmPrinter::LowerPATCHPOINT(const MachineInstr &MI,
 876:                                         SystemZMCInstLower &Lower) {
 877:   auto &Ctx = OutStreamer->getContext();
 878:   MCSymbol *MILabel = Ctx.createTempSymbol();
 879:   OutStreamer->emitLabel(MILabel);
 880: 
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::LowerPATCHPOINT`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::LowerPATCHPOINT` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 881-920 / 第 881-920 行
```cpp
 881:   SM.recordPatchPoint(*MILabel, MI);
 882:   PatchPointOpers Opers(&MI);
 883: 
 884:   unsigned EncodedBytes = 0;
 885:   const MachineOperand &CalleeMO = Opers.getCallTarget();
 886: 
 887:   if (CalleeMO.isImm()) {
 888:     uint64_t CallTarget = CalleeMO.getImm();
 889:     if (CallTarget) {
 890:       unsigned ScratchIdx = -1;
 891:       unsigned ScratchReg = 0;
 892:       do {
 893:         ScratchIdx = Opers.getNextScratchIdx(ScratchIdx + 1);
 894:         ScratchReg = MI.getOperand(ScratchIdx).getReg();
 895:       } while (ScratchReg == SystemZ::R0D);
 896: 
 897:       // Materialize the call target address
 898:       EmitToStreamer(*OutStreamer, MCInstBuilder(SystemZ::LLILF)
 899:                                       .addReg(ScratchReg)
 900:                                       .addImm(CallTarget & 0xFFFFFFFF));
 901:       EncodedBytes += 6;
 902:       if (CallTarget >> 32) {
 903:         EmitToStreamer(*OutStreamer, MCInstBuilder(SystemZ::IIHF)
 904:                                         .addReg(ScratchReg)
 905:                                         .addImm(CallTarget >> 32));
 906:         EncodedBytes += 6;
 907:       }
 908: 
 909:       EmitToStreamer(*OutStreamer, MCInstBuilder(SystemZ::BASR)
 910:                                      .addReg(SystemZ::R14D)
 911:                                      .addReg(ScratchReg));
 912:       EncodedBytes += 2;
 913:     }
 914:   } else if (CalleeMO.isGlobal()) {
 915:     const MCExpr *Expr = Lower.getExpr(CalleeMO, SystemZ::S_PLT);
 916:     EmitToStreamer(*OutStreamer, MCInstBuilder(SystemZ::BRASL)
 917:                                    .addReg(SystemZ::R14D)
 918:                                    .addExpr(Expr));
 919:     EncodedBytes += 6;
 920:   }
```
- **EN**: The range implements or declares functions including `EmitToStreamer`, `EmitToStreamer`, `EmitToStreamer`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `EmitToStreamer`, `EmitToStreamer`, `EmitToStreamer` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 921-960 / 第 921-960 行
```cpp
 921: 
 922:   // Emit padding.
 923:   unsigned NumBytes = Opers.getNumPatchBytes();
 924:   assert(NumBytes >= EncodedBytes &&
 925:          "Patchpoint can't request size less than the length of a call.");
 926:   assert((NumBytes - EncodedBytes) % 2 == 0 &&
 927:          "Invalid number of NOP bytes requested!");
 928:   while (EncodedBytes < NumBytes)
 929:     EncodedBytes += EmitNop(OutContext, *OutStreamer, NumBytes - EncodedBytes,
 930:                             getSubtargetInfo());
 931: }
 932: 
 933: void SystemZAsmPrinter::LowerPATCHABLE_FUNCTION_ENTER(
 934:     const MachineInstr &MI, SystemZMCInstLower &Lower) {
 935: 
 936:   const MachineFunction &MF = *(MI.getParent()->getParent());
 937:   const Function &F = MF.getFunction();
 938: 
 939:   // If patchable-function-entry is set, emit in-function nops here.
 940:   if (F.hasFnAttribute("patchable-function-entry")) {
 941:     // get M-N from function attribute (CodeGenFunction subtracts N
 942:     // from M to yield the correct patchable-function-entry).
 943:     unsigned Num = F.getFnAttributeAsParsedInteger("patchable-function-entry");
 944:     // Emit M-N 2-byte nops. Use getNop() here instead of emitNops()
 945:     // to keep it aligned with the common code implementation emitting
 946:     // the prefix nops.
 947:     for (unsigned I = 0; I < Num; ++I)
 948:       EmitToStreamer(*OutStreamer, MF.getSubtarget().getInstrInfo()->getNop());
 949:     return;
 950:   }
 951:   // Otherwise, emit xray sled.
 952:   // .begin:
 953:   //   j .end    # -> stmg    %r2, %r15, 16(%r15)
 954:   //   nop
 955:   //   llilf   %2, FuncID
 956:   //   brasl   %r14, __xray_FunctionEntry@GOT
 957:   // .end:
 958:   //
 959:   // Update compiler-rt/lib/xray/xray_s390x.cpp accordingly when number
 960:   // of instructions change.
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::LowerPATCHABLE_FUNCTION_ENTER`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::LowerPATCHABLE_FUNCTION_ENTER` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 961-1000 / 第 961-1000 行
```cpp
 961:   bool HasVectorFeature =
 962:       TM.getMCSubtargetInfo().hasFeature(SystemZ::FeatureVector) &&
 963:       !TM.getMCSubtargetInfo().hasFeature(SystemZ::FeatureSoftFloat);
 964:   MCSymbol *FuncEntry = OutContext.getOrCreateSymbol(
 965:       HasVectorFeature ? "__xray_FunctionEntryVec" : "__xray_FunctionEntry");
 966:   MCSymbol *BeginOfSled = OutContext.createTempSymbol("xray_sled_", true);
 967:   MCSymbol *EndOfSled = OutContext.createTempSymbol();
 968:   OutStreamer->emitLabel(BeginOfSled);
 969:   EmitToStreamer(*OutStreamer,
 970:                  MCInstBuilder(SystemZ::J)
 971:                      .addExpr(MCSymbolRefExpr::create(EndOfSled, OutContext)));
 972:   EmitNop(OutContext, *OutStreamer, 2, getSubtargetInfo());
 973:   EmitToStreamer(*OutStreamer,
 974:                  MCInstBuilder(SystemZ::LLILF).addReg(SystemZ::R2D).addImm(0));
 975:   EmitToStreamer(*OutStreamer, MCInstBuilder(SystemZ::BRASL)
 976:                                    .addReg(SystemZ::R14D)
 977:                                    .addExpr(MCSymbolRefExpr::create(
 978:                                        FuncEntry, SystemZ::S_PLT, OutContext)));
 979:   OutStreamer->emitLabel(EndOfSled);
 980:   recordSled(BeginOfSled, MI, SledKind::FUNCTION_ENTER, 2);
 981: }
 982: 
 983: void SystemZAsmPrinter::LowerPATCHABLE_RET(const MachineInstr &MI,
 984:                                            SystemZMCInstLower &Lower) {
 985:   unsigned OpCode = MI.getOperand(0).getImm();
 986:   MCSymbol *FallthroughLabel = nullptr;
 987:   if (OpCode == SystemZ::CondReturn) {
 988:     FallthroughLabel = OutContext.createTempSymbol();
 989:     int64_t Cond0 = MI.getOperand(1).getImm();
 990:     int64_t Cond1 = MI.getOperand(2).getImm();
 991:     EmitToStreamer(*OutStreamer, MCInstBuilder(SystemZ::BRC)
 992:                                      .addImm(Cond0)
 993:                                      .addImm(Cond1 ^ Cond0)
 994:                                      .addExpr(MCSymbolRefExpr::create(
 995:                                          FallthroughLabel, OutContext)));
 996:   }
 997:   // .begin:
 998:   //   br %r14    # -> stmg    %r2, %r15, 24(%r15)
 999:   //   nop
1000:   //   nop
```
- **EN**: The range implements or declares functions including `EmitToStreamer`, `EmitToStreamer`, `SystemZAsmPrinter::LowerPATCHABLE_RET`, `EmitToStreamer`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `EmitToStreamer`, `EmitToStreamer`, `SystemZAsmPrinter::LowerPATCHABLE_RET`, `EmitToStreamer` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1001-1040 / 第 1001-1040 行
```cpp
1001:   //   llilf   %2,FuncID
1002:   //   j       __xray_FunctionExit@GOT
1003:   //
1004:   // Update compiler-rt/lib/xray/xray_s390x.cpp accordingly when number
1005:   // of instructions change.
1006:   bool HasVectorFeature =
1007:       TM.getMCSubtargetInfo().hasFeature(SystemZ::FeatureVector) &&
1008:       !TM.getMCSubtargetInfo().hasFeature(SystemZ::FeatureSoftFloat);
1009:   MCSymbol *FuncExit = OutContext.getOrCreateSymbol(
1010:       HasVectorFeature ? "__xray_FunctionExitVec" : "__xray_FunctionExit");
1011:   MCSymbol *BeginOfSled = OutContext.createTempSymbol("xray_sled_", true);
1012:   OutStreamer->emitLabel(BeginOfSled);
1013:   EmitToStreamer(*OutStreamer,
1014:                  MCInstBuilder(SystemZ::BR).addReg(SystemZ::R14D));
1015:   EmitNop(OutContext, *OutStreamer, 4, getSubtargetInfo());
1016:   EmitToStreamer(*OutStreamer,
1017:                  MCInstBuilder(SystemZ::LLILF).addReg(SystemZ::R2D).addImm(0));
1018:   EmitToStreamer(*OutStreamer, MCInstBuilder(SystemZ::J)
1019:                                    .addExpr(MCSymbolRefExpr::create(
1020:                                        FuncExit, SystemZ::S_PLT, OutContext)));
1021:   if (FallthroughLabel)
1022:     OutStreamer->emitLabel(FallthroughLabel);
1023:   recordSled(BeginOfSled, MI, SledKind::FUNCTION_EXIT, 2);
1024: }
1025: 
1026: // The *alignment* of 128-bit vector types is different between the software
1027: // and hardware vector ABIs. If the there is an externally visible use of a
1028: // vector type in the module it should be annotated with an attribute.
1029: void SystemZAsmPrinter::emitAttributes(Module &M) {
1030:   if (M.getModuleFlag("s390x-visible-vector-ABI")) {
1031:     bool HasVectorFeature =
1032:         TM.getMCSubtargetInfo().hasFeature(SystemZ::FeatureVector);
1033:     OutStreamer->emitGNUAttribute(8, HasVectorFeature ? 2 : 1);
1034:   }
1035: }
1036: 
1037: // Convert a SystemZ-specific constant pool modifier into the associated
1038: // specifier.
1039: static uint8_t getSpecifierFromModifier(SystemZCP::SystemZCPModifier Modifier) {
1040:   switch (Modifier) {
```
- **EN**: The range implements or declares functions including `EmitToStreamer`, `SystemZAsmPrinter::emitAttributes`, `getSpecifierFromModifier`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `EmitToStreamer`, `SystemZAsmPrinter::emitAttributes`, `getSpecifierFromModifier` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1041-1080 / 第 1041-1080 行
```cpp
1041:   case SystemZCP::TLSGD:
1042:     return SystemZ::S_TLSGD;
1043:   case SystemZCP::TLSLDM:
1044:     return SystemZ::S_TLSLDM;
1045:   case SystemZCP::DTPOFF:
1046:     return SystemZ::S_DTPOFF;
1047:   case SystemZCP::NTPOFF:
1048:     return SystemZ::S_NTPOFF;
1049:   }
1050:   llvm_unreachable("Invalid SystemCPModifier!");
1051: }
1052: 
1053: void SystemZAsmPrinter::emitMachineConstantPoolValue(
1054:     MachineConstantPoolValue *MCPV) {
1055:   auto *ZCPV = static_cast<SystemZConstantPoolValue*>(MCPV);
1056: 
1057:   const MCExpr *Expr = MCSymbolRefExpr::create(
1058:       getSymbol(ZCPV->getGlobalValue()),
1059:       getSpecifierFromModifier(ZCPV->getModifier()), OutContext);
1060:   uint64_t Size = getDataLayout().getTypeAllocSize(ZCPV->getType());
1061: 
1062:   OutStreamer->emitValue(Expr, Size);
1063: }
1064: 
1065: // Emit the ctor or dtor list taking into account the init priority.
1066: void SystemZAsmPrinter::emitXXStructorList(const DataLayout &DL,
1067:                                            const Constant *List, bool IsCtor) {
1068:   if (!TM.getTargetTriple().isOSBinFormatGOFF())
1069:     return AsmPrinter::emitXXStructorList(DL, List, IsCtor);
1070: 
1071:   SmallVector<Structor, 8> Structors;
1072:   preprocessXXStructorList(DL, List, Structors);
1073:   if (Structors.empty())
1074:     return;
1075: 
1076:   const Align Align = llvm::Align(4);
1077:   const TargetLoweringObjectFileGOFF &Obj =
1078:       static_cast<const TargetLoweringObjectFileGOFF &>(getObjFileLowering());
1079:   for (Structor &S : Structors) {
1080:     MCSectionGOFF *Section =
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::emitMachineConstantPoolValue`, `SystemZAsmPrinter::emitXXStructorList`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::emitMachineConstantPoolValue`, `SystemZAsmPrinter::emitXXStructorList` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1081-1120 / 第 1081-1120 行
```cpp
1081:         static_cast<MCSectionGOFF *>(Obj.getStaticXtorSection(S.Priority));
1082:     OutStreamer->switchSection(Section);
1083:     if (OutStreamer->getCurrentSection() != OutStreamer->getPreviousSection())
1084:       emitAlignment(Align);
1085: 
1086:     // The priority is provided as an input to getStaticXtorSection(), and is
1087:     // recalculated within that function as `Prio` going to going into the
1088:     // PR section.
1089:     // This priority retrieved via the `SortKey` below is the recalculated
1090:     // Priority.
1091:     uint32_t XtorPriority = Section->getPRAttributes().SortKey;
1092: 
1093:     const GlobalValue *GV = dyn_cast<GlobalValue>(S.Func->stripPointerCasts());
1094:     assert(GV && "C++ xxtor pointer was not a GlobalValue!");
1095:     MCSymbolGOFF *Symbol = static_cast<MCSymbolGOFF *>(getSymbol(GV));
1096: 
1097:     // @@SQINIT entry: { unsigned prio; void (*ctor)();  void (*dtor)(); }
1098: 
1099:     unsigned PointerSizeInBytes = DL.getPointerSize();
1100: 
1101:     auto &Ctx = OutStreamer->getContext();
1102:     const MCExpr *ADAFuncRefExpr;
1103:     unsigned SlotKind = SystemZII::MO_ADA_DIRECT_FUNC_DESC;
1104: 
1105:     MCSectionGOFF *ADASection =
1106:         static_cast<MCSectionGOFF *>(Obj.getADASection());
1107:     assert(ADASection && "ADA section must exist for GOFF targets!");
1108:     const MCSymbol *ADASym = ADASection->getBeginSymbol();
1109:     assert(ADASym && "ADA symbol should already be set!");
1110: 
1111:     ADAFuncRefExpr = MCBinaryExpr::createAdd(
1112:         MCSpecifierExpr::create(MCSymbolRefExpr::create(ADASym, OutContext),
1113:                                 SystemZ::S_QCon, OutContext),
1114:         MCConstantExpr::create(ADATable.insert(Symbol, SlotKind), Ctx), Ctx);
1115: 
1116:     emitInt32(XtorPriority);
1117:     if (IsCtor) {
1118:       OutStreamer->emitValue(ADAFuncRefExpr, PointerSizeInBytes);
1119:       OutStreamer->emitIntValue(0, PointerSizeInBytes);
1120:     } else {
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1121-1160 / 第 1121-1160 行
```cpp
1121:       OutStreamer->emitIntValue(0, PointerSizeInBytes);
1122:       OutStreamer->emitValue(ADAFuncRefExpr, PointerSizeInBytes);
1123:     }
1124:   }
1125: }
1126: 
1127: static void printFormattedRegName(const MCAsmInfo *MAI, unsigned RegNo,
1128:                                   raw_ostream &OS) {
1129:   const char *RegName;
1130:   if (MAI->getAssemblerDialect() == AD_HLASM) {
1131:     RegName = SystemZHLASMInstPrinter::getRegisterName(RegNo);
1132:     // Skip register prefix so that only register number is left
1133:     assert(isalpha(RegName[0]) && isdigit(RegName[1]));
1134:     OS << (RegName + 1);
1135:   } else {
1136:     RegName = SystemZGNUInstPrinter::getRegisterName(RegNo);
1137:     OS << '%' << RegName;
1138:   }
1139: }
1140: 
1141: static void printReg(unsigned Reg, const MCAsmInfo *MAI, raw_ostream &OS) {
1142:   if (!Reg)
1143:     OS << '0';
1144:   else
1145:     printFormattedRegName(MAI, Reg, OS);
1146: }
1147: 
1148: static void printOperand(const MCOperand &MCOp, const MCAsmInfo *MAI,
1149:                          raw_ostream &OS) {
1150:   if (MCOp.isReg())
1151:     printReg(MCOp.getReg(), MAI, OS);
1152:   else if (MCOp.isImm())
1153:     OS << MCOp.getImm();
1154:   else if (MCOp.isExpr())
1155:     MAI->printExpr(OS, *MCOp.getExpr());
1156:   else
1157:     llvm_unreachable("Invalid operand");
1158: }
1159: 
1160: static void printAddress(const MCAsmInfo *MAI, unsigned Base,
```
- **EN**: The range implements or declares functions including `printFormattedRegName`, `printReg`, `printOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `printFormattedRegName`, `printReg`, `printOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1161-1200 / 第 1161-1200 行
```cpp
1161:                          const MCOperand &DispMO, unsigned Index,
1162:                          raw_ostream &OS) {
1163:   printOperand(DispMO, MAI, OS);
1164:   if (Base || Index) {
1165:     OS << '(';
1166:     if (Index) {
1167:       printFormattedRegName(MAI, Index, OS);
1168:       if (Base)
1169:         OS << ',';
1170:     }
1171:     if (Base)
1172:       printFormattedRegName(MAI, Base, OS);
1173:     OS << ')';
1174:   }
1175: }
1176: 
1177: bool SystemZAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
1178:                                         const char *ExtraCode,
1179:                                         raw_ostream &OS) {
1180:   const MCRegisterInfo &MRI = TM.getMCRegisterInfo();
1181:   const MachineOperand &MO = MI->getOperand(OpNo);
1182:   MCOperand MCOp;
1183:   if (ExtraCode) {
1184:     if (ExtraCode[0] == 'N' && !ExtraCode[1] && MO.isReg() &&
1185:         SystemZ::GR128BitRegClass.contains(MO.getReg()))
1186:       MCOp =
1187:           MCOperand::createReg(MRI.getSubReg(MO.getReg(), SystemZ::subreg_l64));
1188:     else
1189:       return AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, OS);
1190:   } else {
1191:     SystemZMCInstLower Lower(MF->getContext(), *this);
1192:     MCOp = Lower.lowerOperand(MO);
1193:   }
1194:   printOperand(MCOp, &MAI, OS);
1195:   return false;
1196: }
1197: 
1198: bool SystemZAsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI,
1199:                                               unsigned OpNo,
1200:                                               const char *ExtraCode,
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::PrintAsmOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::PrintAsmOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1201-1240 / 第 1201-1240 行
```cpp
1201:                                               raw_ostream &OS) {
1202:   if (ExtraCode && ExtraCode[0] && !ExtraCode[1]) {
1203:     switch (ExtraCode[0]) {
1204:     case 'A':
1205:       // Unlike EmitMachineNode(), EmitSpecialNode(INLINEASM) does not call
1206:       // setMemRefs(), so MI->memoperands() is empty and the alignment
1207:       // information is not available.
1208:       return false;
1209:     case 'O':
1210:       OS << MI->getOperand(OpNo + 1).getImm();
1211:       return false;
1212:     case 'R':
1213:       ::printReg(MI->getOperand(OpNo).getReg(), &MAI, OS);
1214:       return false;
1215:     }
1216:   }
1217:   printAddress(&MAI, MI->getOperand(OpNo).getReg(),
1218:                MCOperand::createImm(MI->getOperand(OpNo + 1).getImm()),
1219:                MI->getOperand(OpNo + 2).getReg(), OS);
1220:   return false;
1221: }
1222: 
1223: void SystemZAsmPrinter::emitEndOfAsmFile(Module &M) {
1224:   auto TT = OutContext.getTargetTriple();
1225:   if (TT.isOSzOS()) {
1226:     OutStreamer->switchSection(getObjFileLowering().getTextSection());
1227:     for (auto &Info : DeferredPPA1)
1228:       emitPPA1(Info);
1229:     emitADASection();
1230:     emitIDRLSection(M);
1231:     // On z/OS, we need to associate an external data reference with an ED
1232:     // symbol, for which we use the the ED of the ADA. We also need to mark the
1233:     // reference as being to data, otherwise we cannot bind with code generated
1234:     // by XL.
1235:     for (auto &GO : M.global_objects()) {
1236:       if (auto *GV = dyn_cast<GlobalVariable>(&GO)) {
1237:         if (!GV->hasInitializer()) {
1238:           MCSymbol *Sym = getSymbol(GV);
1239:           getTargetStreamer()->emitADA(
1240:               Sym, OutContext.getObjectFileInfo()->getADASection());
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::emitEndOfAsmFile`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::emitEndOfAsmFile` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1241-1280 / 第 1241-1280 行
```cpp
1241:           OutStreamer->emitSymbolAttribute(Sym, MCSA_ELF_TypeObject);
1242:         }
1243:       }
1244:     }
1245:   }
1246:   emitAttributes(M);
1247: }
1248: 
1249: void SystemZAsmPrinter::emitADASection() {
1250:   OutStreamer->pushSection();
1251: 
1252:   const unsigned PointerSize = getDataLayout().getPointerSize();
1253:   OutStreamer->switchSection(getObjFileLowering().getADASection());
1254: 
1255:   unsigned EmittedBytes = 0;
1256:   for (auto &Entry : ADATable.getTable()) {
1257:     const MCSymbol *Sym;
1258:     unsigned SlotKind;
1259:     std::tie(Sym, SlotKind) = Entry.first;
1260:     unsigned Offset = Entry.second;
1261:     assert(Offset == EmittedBytes && "Offset not as expected");
1262:     (void)EmittedBytes;
1263: #define EMIT_COMMENT(Str)                                                      \
1264:   OutStreamer->AddComment(Twine("Offset ")                                     \
1265:                               .concat(utostr(Offset))                          \
1266:                               .concat(" " Str " ")                             \
1267:                               .concat(Sym->getName()));
1268:     switch (SlotKind) {
1269:     case SystemZII::MO_ADA_DIRECT_FUNC_DESC:
1270:       // Language Environment DLL logic requires function descriptors, for
1271:       // imported functions, that are placed in the ADA to be 8 byte aligned.
1272:       EMIT_COMMENT("function descriptor of");
1273:       OutStreamer->emitValue(
1274:           MCSpecifierExpr::create(MCSymbolRefExpr::create(Sym, OutContext),
1275:                                   SystemZ::S_RCon, OutContext),
1276:           PointerSize);
1277:       OutStreamer->emitValue(
1278:           MCSpecifierExpr::create(MCSymbolRefExpr::create(Sym, OutContext),
1279:                                   SystemZ::S_VCon, OutContext),
1280:           PointerSize);
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::emitADASection`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::emitADASection` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1281-1320 / 第 1281-1320 行
```cpp
1281:       EmittedBytes += PointerSize * 2;
1282:       break;
1283:     case SystemZII::MO_ADA_DATA_SYMBOL_ADDR:
1284:       EMIT_COMMENT("pointer to data symbol");
1285:       OutStreamer->emitValue(
1286:           MCSpecifierExpr::create(MCSymbolRefExpr::create(Sym, OutContext),
1287:                                   SystemZ::S_None, OutContext),
1288:           PointerSize);
1289:       EmittedBytes += PointerSize;
1290:       break;
1291:     case SystemZII::MO_ADA_INDIRECT_FUNC_DESC: {
1292:       MCSymbol *Alias = OutContext.getOrCreateSymbol(
1293:           Twine(Sym->getName()).concat("@indirect"));
1294:       OutStreamer->emitSymbolAttribute(Alias, MCSA_IndirectSymbol);
1295:       OutStreamer->emitSymbolAttribute(Alias, MCSA_ELF_TypeFunction);
1296:       OutStreamer->emitSymbolAttribute(Alias, MCSA_Global);
1297:       OutStreamer->emitSymbolAttribute(Alias, MCSA_Extern);
1298:       MCSymbolGOFF *GOFFSym =
1299:           static_cast<llvm::MCSymbolGOFF *>(const_cast<llvm::MCSymbol *>(Sym));
1300:       getTargetStreamer()->emitExternalName(Alias, GOFFSym->getExternalName());
1301:       EMIT_COMMENT("pointer to function descriptor");
1302:       OutStreamer->emitValue(
1303:           MCSpecifierExpr::create(MCSymbolRefExpr::create(Alias, OutContext),
1304:                                   SystemZ::S_VCon, OutContext),
1305:           PointerSize);
1306:       EmittedBytes += PointerSize;
1307:       break;
1308:     }
1309:     default:
1310:       llvm_unreachable("Unexpected slot kind");
1311:     }
1312: #undef EMIT_COMMENT
1313:   }
1314:   OutStreamer->popSection();
1315: }
1316: 
1317: static std::string getProductID(Module &M) {
1318:   std::string ProductID;
1319:   if (auto *MD = M.getModuleFlag("zos_product_id"))
1320:     ProductID = cast<MDString>(MD)->getString().str();
```
- **EN**: The range implements or declares functions including `getProductID`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `getProductID` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1321-1360 / 第 1321-1360 行
```cpp
1321:   if (ProductID.empty())
1322:     ProductID = "LLVM";
1323:   return ProductID;
1324: }
1325: 
1326: static uint32_t getProductVersion(Module &M) {
1327:   if (auto *VersionVal = mdconst::extract_or_null<ConstantInt>(
1328:           M.getModuleFlag("zos_product_major_version")))
1329:     return VersionVal->getZExtValue();
1330:   return LLVM_VERSION_MAJOR;
1331: }
1332: 
1333: static uint32_t getProductRelease(Module &M) {
1334:   if (auto *ReleaseVal = mdconst::extract_or_null<ConstantInt>(
1335:           M.getModuleFlag("zos_product_minor_version")))
1336:     return ReleaseVal->getZExtValue();
1337:   return LLVM_VERSION_MINOR;
1338: }
1339: 
1340: static uint32_t getProductPatch(Module &M) {
1341:   if (auto *PatchVal = mdconst::extract_or_null<ConstantInt>(
1342:           M.getModuleFlag("zos_product_patchlevel")))
1343:     return PatchVal->getZExtValue();
1344:   return LLVM_VERSION_PATCH;
1345: }
1346: 
1347: static time_t getTranslationTime(Module &M) {
1348:   std::time_t Time = 0;
1349:   if (auto *Val = mdconst::extract_or_null<ConstantInt>(
1350:           M.getModuleFlag("zos_translation_time"))) {
1351:     long SecondsSinceEpoch = Val->getSExtValue();
1352:     Time = static_cast<time_t>(SecondsSinceEpoch);
1353:   }
1354:   return Time;
1355: }
1356: 
1357: void SystemZAsmPrinter::emitIDRLSection(Module &M) {
1358:   OutStreamer->pushSection();
1359:   OutStreamer->switchSection(getObjFileLowering().getIDRLSection());
1360:   constexpr unsigned IDRLDataLength = 30;
```
- **EN**: The range implements or declares functions including `getProductVersion`, `getProductRelease`, `getProductPatch`, `getTranslationTime`, `SystemZAsmPrinter::emitIDRLSection`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getProductVersion`, `getProductRelease`, `getProductPatch`, `getTranslationTime`, `SystemZAsmPrinter::emitIDRLSection` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1361-1400 / 第 1361-1400 行
```cpp
1361:   std::time_t Time = getTranslationTime(M);
1362: 
1363:   uint32_t ProductVersion = getProductVersion(M);
1364:   uint32_t ProductRelease = getProductRelease(M);
1365: 
1366:   std::string ProductID = getProductID(M);
1367: 
1368:   SmallString<IDRLDataLength + 1> TempStr;
1369:   raw_svector_ostream O(TempStr);
1370:   O << formatv("{0,-10}{1,0-2:d}{2,0-2:d}{3:%Y%m%d%H%M%S}{4,0-2}",
1371:                ProductID.substr(0, 10).c_str(), ProductVersion, ProductRelease,
1372:                llvm::sys::toUtcTime(Time), "0");
1373:   SmallString<IDRLDataLength> Data;
1374:   ConverterEBCDIC::convertToEBCDIC(TempStr, Data);
1375: 
1376:   OutStreamer->emitInt8(0);               // Reserved.
1377:   OutStreamer->emitInt8(3);               // Format.
1378:   OutStreamer->emitInt16(IDRLDataLength); // Length.
1379:   OutStreamer->emitBytes(Data.str());
1380:   OutStreamer->popSection();
1381: }
1382: 
1383: void SystemZAsmPrinter::emitFunctionBodyEnd() {
1384:   if (TM.getTargetTriple().isOSzOS()) {
1385:     // Emit symbol for the end of function if the z/OS target streamer
1386:     // is used. This is needed to calculate the size of the function.
1387:     OutStreamer->emitLabel(DeferredPPA1.back().FnEnd);
1388:   }
1389: }
1390: 
1391: static void emitPPA1Flags(std::unique_ptr<MCStreamer> &OutStreamer, bool VarArg,
1392:                           bool StackProtector, bool FPRMask, bool VRMask,
1393:                           bool EHBlock, bool HasArgAreaLength, bool HasName) {
1394:   enum class PPA1Flag1 : uint8_t {
1395:     DSA64Bit = (0x80 >> 0),
1396:     VarArg = (0x80 >> 7),
1397:     LLVM_MARK_AS_BITMASK_ENUM(DSA64Bit)
1398:   };
1399:   enum class PPA1Flag2 : uint8_t {
1400:     ExternalProcedure = (0x80 >> 0),
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::emitFunctionBodyEnd`, `emitPPA1Flags`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::emitFunctionBodyEnd`, `emitPPA1Flags` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1401-1440 / 第 1401-1440 行
```cpp
1401:     STACKPROTECTOR = (0x80 >> 3),
1402:     LLVM_MARK_AS_BITMASK_ENUM(ExternalProcedure)
1403:   };
1404:   enum class PPA1Flag3 : uint8_t {
1405:     HasArgAreaLength = (0x80 >> 1),
1406:     FPRMask = (0x80 >> 2),
1407:     LLVM_MARK_AS_BITMASK_ENUM(HasArgAreaLength)
1408:   };
1409:   enum class PPA1Flag4 : uint8_t {
1410:     EPMOffsetPresent = (0x80 >> 0),
1411:     VRMask = (0x80 >> 2),
1412:     EHBlock = (0x80 >> 3),
1413:     ProcedureNamePresent = (0x80 >> 7),
1414:     LLVM_MARK_AS_BITMASK_ENUM(EPMOffsetPresent)
1415:   };
1416: 
1417:   // Declare optional section flags that can be modified.
1418:   auto Flags1 = PPA1Flag1(0);
1419:   auto Flags2 = PPA1Flag2::ExternalProcedure;
1420:   auto Flags3 = PPA1Flag3(0);
1421:   auto Flags4 = PPA1Flag4::EPMOffsetPresent;
1422: 
1423:   Flags1 |= PPA1Flag1::DSA64Bit;
1424: 
1425:   if (VarArg)
1426:     Flags1 |= PPA1Flag1::VarArg;
1427: 
1428:   if (StackProtector)
1429:     Flags2 |= PPA1Flag2::STACKPROTECTOR;
1430: 
1431:   if (HasArgAreaLength)
1432:     Flags3 |= PPA1Flag3::HasArgAreaLength; // Add emit ArgAreaLength flag.
1433: 
1434:   // SavedGPRMask, SavedFPRMask, and SavedVRMask are precomputed in.
1435:   if (FPRMask)
1436:     Flags3 |= PPA1Flag3::FPRMask; // Add emit FPR mask flag.
1437: 
1438:   if (VRMask)
1439:     Flags4 |= PPA1Flag4::VRMask; // Add emit VR mask flag.
1440: 
```
- **EN**: The range implements or declares functions including `LLVM_MARK_AS_BITMASK_ENUM`, `LLVM_MARK_AS_BITMASK_ENUM`, `LLVM_MARK_AS_BITMASK_ENUM`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LLVM_MARK_AS_BITMASK_ENUM`, `LLVM_MARK_AS_BITMASK_ENUM`, `LLVM_MARK_AS_BITMASK_ENUM` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1441-1480 / 第 1441-1480 行
```cpp
1441:   if (EHBlock)
1442:     Flags4 |= PPA1Flag4::EHBlock; // Add optional EH block.
1443: 
1444:   if (HasName)
1445:     Flags4 |= PPA1Flag4::ProcedureNamePresent; // Add optional name block.
1446: 
1447:   OutStreamer->AddComment("PPA1 Flags 1");
1448:   OutStreamer->AddComment("  Bit 0: 1 = 64-bit DSA");
1449:   if ((Flags1 & PPA1Flag1::VarArg) == PPA1Flag1::VarArg)
1450:     OutStreamer->AddComment("  Bit 7: 1 = Vararg function");
1451:   OutStreamer->emitInt8(static_cast<uint8_t>(Flags1)); // Flags 1.
1452: 
1453:   OutStreamer->AddComment("PPA1 Flags 2");
1454:   if ((Flags2 & PPA1Flag2::ExternalProcedure) == PPA1Flag2::ExternalProcedure)
1455:     OutStreamer->AddComment("  Bit 0: 1 = External procedure");
1456:   if ((Flags2 & PPA1Flag2::STACKPROTECTOR) == PPA1Flag2::STACKPROTECTOR)
1457:     OutStreamer->AddComment("  Bit 3: 1 = STACKPROTECT is enabled");
1458:   else
1459:     OutStreamer->AddComment("  Bit 3: 0 = STACKPROTECT is not enabled");
1460:   OutStreamer->emitInt8(static_cast<uint8_t>(Flags2)); // Flags 2.
1461: 
1462:   OutStreamer->AddComment("PPA1 Flags 3");
1463:   if ((Flags3 & PPA1Flag3::HasArgAreaLength) == PPA1Flag3::HasArgAreaLength)
1464:     OutStreamer->AddComment(
1465:         "  Bit 1: 1 = Argument Area Length is in optional area");
1466:   if ((Flags3 & PPA1Flag3::FPRMask) == PPA1Flag3::FPRMask)
1467:     OutStreamer->AddComment("  Bit 2: 1 = FP Reg Mask is in optional area");
1468:   OutStreamer->emitInt8(
1469:       static_cast<uint8_t>(Flags3)); // Flags 3 (optional sections).
1470: 
1471:   OutStreamer->AddComment("PPA1 Flags 4");
1472:   if ((Flags4 & PPA1Flag4::VRMask) == PPA1Flag4::VRMask)
1473:     OutStreamer->AddComment("  Bit 2: 1 = Vector Reg Mask is in optional area");
1474:   if ((Flags4 & PPA1Flag4::EHBlock) == PPA1Flag4::EHBlock)
1475:     OutStreamer->AddComment("  Bit 3: 1 = C++ EH block");
1476:   if ((Flags4 & PPA1Flag4::ProcedureNamePresent) ==
1477:       PPA1Flag4::ProcedureNamePresent)
1478:     OutStreamer->AddComment("  Bit 7: 1 = Name Length and Name");
1479:   OutStreamer->emitInt8(static_cast<uint8_t>(
1480:       Flags4)); // Flags 4 (optional sections, always emit these).
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1481-1520 / 第 1481-1520 行
```cpp
1481: }
1482: 
1483: static void emitPPA1Name(std::unique_ptr<MCStreamer> &OutStreamer,
1484:                          StringRef OutName) {
1485:   size_t NameSize = OutName.size();
1486:   uint16_t OutSize;
1487:   if (NameSize < UINT16_MAX) {
1488:     OutSize = static_cast<uint16_t>(NameSize);
1489:   } else {
1490:     OutName = OutName.substr(0, UINT16_MAX);
1491:     OutSize = UINT16_MAX;
1492:   }
1493:   // Emit padding to ensure that the next optional field word-aligned.
1494:   uint8_t ExtraZeros = 4 - ((2 + OutSize) % 4);
1495: 
1496:   SmallString<512> OutnameConv;
1497:   ConverterEBCDIC::convertToEBCDIC(OutName, OutnameConv);
1498:   OutName = OutnameConv.str();
1499: 
1500:   OutStreamer->AddComment("Length of Name");
1501:   OutStreamer->emitInt16(OutSize);
1502:   OutStreamer->AddComment("Name of Function");
1503:   OutStreamer->emitBytes(OutName);
1504:   OutStreamer->emitZeros(ExtraZeros);
1505: }
1506: 
1507: void SystemZAsmPrinter::emitPPA1(PPA1Info &Info) {
1508:   assert(PPA2Sym != nullptr && "PPA2 Symbol not defined");
1509: 
1510:   // Optional Argument Area Length.
1511:   // Note: This represents the length of the argument area that we reserve
1512:   //       in our stack for setting up arguments for calls to other
1513:   //       routines. If this optional field is not set, LE will reserve
1514:   //       128 bytes for the argument area. This optional field is
1515:   //       created if greater than 128 bytes is required - to guarantee
1516:   //       the required space is reserved on stack extension in the new
1517:   //       extension.  This optional field is also created if the
1518:   //       routine has alloca(). This may reduce stack space
1519:   //       if alloca() call causes a stack extension.
1520:   bool HasArgAreaLength = (Info.AllocaReg != 0) || (Info.CallFrameSize > 128);
```
- **EN**: The range implements or declares functions including `emitPPA1Name`, `SystemZAsmPrinter::emitPPA1`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `emitPPA1Name`, `SystemZAsmPrinter::emitPPA1` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1521-1560 / 第 1521-1560 行
```cpp
1521: 
1522:   // Emit PPA1 section.
1523:   OutStreamer->AddComment("PPA1");
1524:   OutStreamer->emitLabel(Info.PPA1);
1525:   OutStreamer->AddComment("Version");
1526:   OutStreamer->emitInt8(0x02); // Version.
1527:   OutStreamer->AddComment("LE Signature X'CE'");
1528:   OutStreamer->emitInt8(0xCE); // CEL signature.
1529:   OutStreamer->AddComment("Saved GPR Mask");
1530:   OutStreamer->emitInt16(Info.SavedGPRMask);
1531:   OutStreamer->AddComment("Offset to PPA2");
1532:   OutStreamer->emitAbsoluteSymbolDiff(PPA2Sym, Info.PPA1, 4);
1533: 
1534:   emitPPA1Flags(OutStreamer, Info.IsVarArg, Info.HasStackProtector,
1535:                 Info.SavedFPRMask != 0, Info.SavedVRMask != 0,
1536:                 Info.PersonalityRoutine != nullptr, HasArgAreaLength,
1537:                 Info.Name.size() > 0);
1538: 
1539:   OutStreamer->AddComment("Length/4 of Parms");
1540:   OutStreamer->emitInt16(
1541:       static_cast<uint16_t>(Info.SizeOfFnParams / 4)); // Parms/4.
1542:   OutStreamer->AddComment("Length of Code");
1543:   OutStreamer->emitAbsoluteSymbolDiff(Info.FnEnd, Info.EPMarker, 4);
1544: 
1545:   if (HasArgAreaLength) {
1546:     OutStreamer->AddComment("Argument Area Length");
1547:     OutStreamer->emitInt32(Info.CallFrameSize);
1548:   }
1549: 
1550:   // Emit saved FPR mask and offset to FPR save area (0x20 of flags 3).
1551:   if (Info.SavedFPRMask) {
1552:     OutStreamer->AddComment("FPR mask");
1553:     OutStreamer->emitInt16(Info.SavedFPRMask);
1554:     OutStreamer->AddComment("AR mask");
1555:     OutStreamer->emitInt16(0); // AR Mask, unused currently.
1556:     OutStreamer->AddComment("FPR Save Area Locator");
1557:     uint64_t FPRSaveAreaOffset = Info.OffsetFPR;
1558:     assert(FPRSaveAreaOffset < 0x10000000 && "Offset out of range");
1559:     FPRSaveAreaOffset &= 0x0FFFFFFF; // Lose top 4 bits.
1560:     OutStreamer->AddComment(
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1561-1600 / 第 1561-1600 行
```cpp
1561:         Twine("  Bit 0-3: Register R").concat(utostr(Info.FrameReg)).str());
1562:     OutStreamer->AddComment(
1563:         Twine("  Bit 4-31: Offset ").concat(utostr(FPRSaveAreaOffset)).str());
1564:     OutStreamer->emitInt32(FPRSaveAreaOffset |
1565:                            (Info.FrameReg << 28)); // Offset to FPR save area
1566:                                                    // with register to add
1567:                                                    // value to (alloca reg).
1568:   }
1569: 
1570:   // Emit saved VR mask to VR save area.
1571:   if (Info.SavedVRMask) {
1572:     OutStreamer->AddComment("VR mask");
1573:     OutStreamer->emitInt8(Info.SavedVRMask);
1574:     OutStreamer->emitInt8(0);  // Reserved.
1575:     OutStreamer->emitInt16(0); // Also reserved.
1576:     uint64_t VRSaveAreaOffset = Info.OffsetVR;
1577:     assert(VRSaveAreaOffset < 0x10000000 && "Offset out of range");
1578:     VRSaveAreaOffset &= 0x0FFFFFFF; // Lose top 4 bits.
1579:     OutStreamer->AddComment("VR Save Area Locator");
1580:     OutStreamer->AddComment(
1581:         Twine("  Bit 0-3: Register R").concat(utostr(Info.FrameReg)).str());
1582:     OutStreamer->AddComment(
1583:         Twine("  Bit 4-31: Offset ").concat(utostr(VRSaveAreaOffset)).str());
1584:     OutStreamer->emitInt32(VRSaveAreaOffset | (Info.FrameReg << 28));
1585:   }
1586: 
1587:   // Emit C++ EH information block.
1588:   if (Info.PersonalityRoutine) {
1589:     OutStreamer->AddComment("Version");
1590:     OutStreamer->emitInt32(1);
1591:     OutStreamer->AddComment("Flags");
1592:     OutStreamer->emitInt32(0); // LSDA field is a WAS offset
1593:     OutStreamer->AddComment("Personality routine");
1594:     OutStreamer->emitInt64(ADATable.insert(
1595:         Info.PersonalityRoutine, SystemZII::MO_ADA_INDIRECT_FUNC_DESC));
1596:     OutStreamer->AddComment("LSDA location");
1597:     OutStreamer->emitInt64(
1598:         ADATable.insert(Info.GCCEH, SystemZII::MO_ADA_DATA_SYMBOL_ADDR));
1599:   }
1600: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1601-1640 / 第 1601-1640 行
```cpp
1601:   // Emit name length and name optional section (0x01 of flags 4)
1602:   if (Info.Name.size())
1603:     emitPPA1Name(OutStreamer, Info.Name);
1604: 
1605:   // Emit offset to entry point optional section (0x80 of flags 4).
1606:   OutStreamer->emitAbsoluteSymbolDiff(Info.EPMarker, Info.PPA1, 4);
1607: }
1608: 
1609: void SystemZAsmPrinter::calculatePPA1() {
1610:   assert(PPA2Sym != nullptr && "PPA2 Symbol not defined");
1611: 
1612:   PPA1Info Info;
1613: 
1614:   const TargetRegisterInfo *TRI = MF->getRegInfo().getTargetRegisterInfo();
1615:   const SystemZSubtarget &Subtarget = MF->getSubtarget<SystemZSubtarget>();
1616: 
1617:   const SystemZMachineFunctionInfo *ZFI =
1618:       MF->getInfo<SystemZMachineFunctionInfo>();
1619:   const auto *ZFL = static_cast<const SystemZXPLINKFrameLowering *>(
1620:       Subtarget.getFrameLowering());
1621:   const MachineFrameInfo &MFFrame = MF->getFrameInfo();
1622: 
1623:   // Get saved GPR/FPR/VPR masks.
1624:   const std::vector<CalleeSavedInfo> &CSI = MFFrame.getCalleeSavedInfo();
1625:   uint16_t SavedGPRMask = 0;
1626:   uint16_t SavedFPRMask = 0;
1627:   uint8_t SavedVRMask = 0;
1628:   int64_t OffsetFPR = 0;
1629:   int64_t OffsetVR = 0;
1630:   const int64_t TopOfStack =
1631:       MFFrame.getOffsetAdjustment() + MFFrame.getStackSize();
1632: 
1633:   // Loop over the spilled registers. The CalleeSavedInfo can't be used because
1634:   // it does not contain all spilled registers.
1635:   for (unsigned I = ZFI->getSpillGPRRegs().LowGPR,
1636:                 E = ZFI->getSpillGPRRegs().HighGPR;
1637:        I && E && I <= E; ++I) {
1638:     unsigned V = TRI->getEncodingValue((Register)I);
1639:     assert(V < 16 && "GPR index out of range");
1640:     SavedGPRMask |= 1 << (15 - V);
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::calculatePPA1`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::calculatePPA1` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1641-1680 / 第 1641-1680 行
```cpp
1641:   }
1642: 
1643:   for (auto &CS : CSI) {
1644:     unsigned Reg = CS.getReg();
1645:     unsigned I = TRI->getEncodingValue(Reg);
1646: 
1647:     if (SystemZ::FP64BitRegClass.contains(Reg)) {
1648:       assert(I < 16 && "FPR index out of range");
1649:       SavedFPRMask |= 1 << (15 - I);
1650:       int64_t Temp = MFFrame.getObjectOffset(CS.getFrameIdx());
1651:       if (Temp < OffsetFPR)
1652:         OffsetFPR = Temp;
1653:     } else if (SystemZ::VR128BitRegClass.contains(Reg)) {
1654:       assert(I >= 16 && I <= 23 && "VPR index out of range");
1655:       unsigned BitNum = I - 16;
1656:       SavedVRMask |= 1 << (7 - BitNum);
1657:       int64_t Temp = MFFrame.getObjectOffset(CS.getFrameIdx());
1658:       if (Temp < OffsetVR)
1659:         OffsetVR = Temp;
1660:     }
1661:   }
1662: 
1663:   // Adjust the offset.
1664:   OffsetFPR += (OffsetFPR < 0) ? TopOfStack : 0;
1665:   OffsetVR += (OffsetVR < 0) ? TopOfStack : 0;
1666: 
1667:   // Get alloca register.
1668:   uint8_t FrameReg = TRI->getEncodingValue(TRI->getFrameRegister(*MF));
1669:   uint8_t AllocaReg = ZFL->hasFP(*MF) ? FrameReg : 0;
1670:   assert(AllocaReg < 16 && "Can't have alloca register larger than 15");
1671: 
1672:   MCSymbol *PersonalityRoutine = nullptr;
1673:   MCSymbol *GCCEH = nullptr;
1674:   if (!MF->getLandingPads().empty()) {
1675:     const Function *Per = dyn_cast<Function>(
1676:         MF->getFunction().getPersonalityFn()->stripPointerCasts());
1677:     PersonalityRoutine = Per ? MF->getTarget().getSymbol(Per) : nullptr;
1678:     assert(PersonalityRoutine && "Missing personality routine");
1679: 
1680:     GCCEH = MF->getContext().getOrCreateSymbol(Twine("GCC_except_table") +
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1681-1720 / 第 1681-1720 行
```cpp
1681:                                                Twine(MF->getFunctionNumber()));
1682:   }
1683: 
1684:   // Get the name of the function, with suffix _.
1685:   std::string N(MF->getFunction().hasName()
1686:                     ? Twine(MF->getFunction().getName()).concat("_").str()
1687:                     : "");
1688: 
1689:   // Save the calculated values.
1690:   if (MF->getFunction().hasName())
1691:     Info.Name = MF->getFunction().getName();
1692:   Info.PPA1 = OutContext.createTempSymbol(Twine("PPA1_").concat(N).str(), true);
1693:   Info.EPMarker =
1694:       OutContext.createTempSymbol(Twine("EPM_").concat(N).str(), true);
1695:   Info.FnEnd = OutContext.createTempSymbol(Twine(N).concat("end_").str());
1696:   Info.PersonalityRoutine = PersonalityRoutine;
1697:   Info.GCCEH = GCCEH;
1698:   Info.OffsetFPR = OffsetFPR;
1699:   Info.OffsetVR = OffsetVR;
1700:   Info.CallFrameSize = MFFrame.getMaxCallFrameSize();
1701:   Info.SizeOfFnParams = ZFI->getSizeOfFnParams();
1702:   Info.SavedGPRMask = SavedGPRMask;
1703:   Info.SavedFPRMask = SavedFPRMask;
1704:   Info.SavedVRMask = SavedVRMask;
1705:   Info.FrameReg = FrameReg;
1706:   Info.AllocaReg = AllocaReg;
1707:   Info.IsVarArg = MF->getFunction().isVarArg();
1708:   Info.HasStackProtector = MFFrame.hasStackProtectorIndex();
1709: 
1710:   DeferredPPA1.push_back(Info);
1711: }
1712: 
1713: void SystemZAsmPrinter::emitStartOfAsmFile(Module &M) {
1714:   if (TM.getTargetTriple().isOSzOS())
1715:     emitPPA2(M);
1716:   AsmPrinter::emitStartOfAsmFile(M);
1717: }
1718: 
1719: void SystemZAsmPrinter::emitPPA2(Module &M) {
1720:   OutStreamer->pushSection();
```
- **EN**: The range implements or declares functions including `N`, `SystemZAsmPrinter::emitStartOfAsmFile`, `SystemZAsmPrinter::emitPPA2`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `N`, `SystemZAsmPrinter::emitStartOfAsmFile`, `SystemZAsmPrinter::emitPPA2` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1721-1760 / 第 1721-1760 行
```cpp
1721:   OutStreamer->switchSection(getObjFileLowering().getTextSection());
1722:   MCContext &OutContext = OutStreamer->getContext();
1723:   // Make CELQSTRT symbol.
1724:   const char *StartSymbolName = "CELQSTRT";
1725:   MCSymbol *CELQSTRT = OutContext.getOrCreateSymbol(StartSymbolName);
1726:   OutStreamer->emitSymbolAttribute(CELQSTRT, MCSA_OSLinkage);
1727:   OutStreamer->emitSymbolAttribute(CELQSTRT, MCSA_Global);
1728: 
1729:   // Create symbol and assign to class field for use in PPA1.
1730:   PPA2Sym = OutContext.createTempSymbol("PPA2", false);
1731:   MCSymbol *DateVersionSym = OutContext.createTempSymbol("DVS", false);
1732: 
1733:   std::time_t Time = getTranslationTime(M);
1734:   SmallString<14> CompilationTimeEBCDIC, CompilationTime;
1735:   CompilationTime = formatv("{0:%Y%m%d%H%M%S}", llvm::sys::toUtcTime(Time));
1736: 
1737:   uint32_t ProductVersion = getProductVersion(M),
1738:            ProductRelease = getProductRelease(M),
1739:            ProductPatch = getProductPatch(M);
1740: 
1741:   SmallString<6> VersionEBCDIC, Version;
1742:   Version = formatv("{0,0-2:d}{1,0-2:d}{2,0-2:d}", ProductVersion,
1743:                     ProductRelease, ProductPatch);
1744: 
1745:   ConverterEBCDIC::convertToEBCDIC(CompilationTime, CompilationTimeEBCDIC);
1746:   ConverterEBCDIC::convertToEBCDIC(Version, VersionEBCDIC);
1747: 
1748:   enum class PPA2MemberId : uint8_t {
1749:     // See z/OS Language Environment Vendor Interfaces v2r5, p.23, for
1750:     // complete list. Only the C runtime is supported by this backend.
1751:     LE_C_Runtime = 3,
1752:   };
1753:   enum class PPA2MemberSubId : uint8_t {
1754:     // List of languages using the LE C runtime implementation.
1755:     C = 0x00,
1756:     CXX = 0x01,
1757:     Swift = 0x03,
1758:     Go = 0x60,
1759:     LLVMBasedLang = 0xe7,
1760:   };
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 1761-1800 / 第 1761-1800 行
```cpp
1761:   // PPA2 Flags
1762:   enum class PPA2Flags : uint8_t {
1763:     CompileForBinaryFloatingPoint = 0x80,
1764:     CompiledWithXPLink = 0x01,
1765:     CompiledUnitASCII = 0x04,
1766:     HasServiceInfo = 0x20,
1767:   };
1768: 
1769:   PPA2MemberSubId MemberSubId = PPA2MemberSubId::LLVMBasedLang;
1770:   if (auto *MD = M.getModuleFlag("zos_cu_language")) {
1771:     StringRef Language = cast<MDString>(MD)->getString();
1772:     MemberSubId = StringSwitch<PPA2MemberSubId>(Language)
1773:                       .Case("C", PPA2MemberSubId::C)
1774:                       .Case("C++", PPA2MemberSubId::CXX)
1775:                       .Case("Swift", PPA2MemberSubId::Swift)
1776:                       .Case("Go", PPA2MemberSubId::Go)
1777:                       .Default(PPA2MemberSubId::LLVMBasedLang);
1778:   }
1779: 
1780:   // Emit PPA2 section.
1781:   OutStreamer->emitLabel(PPA2Sym);
1782:   OutStreamer->emitInt8(static_cast<uint8_t>(PPA2MemberId::LE_C_Runtime));
1783:   OutStreamer->emitInt8(static_cast<uint8_t>(MemberSubId));
1784:   OutStreamer->emitInt8(0x22); // Member defined, c370_plist+c370_env
1785:   OutStreamer->emitInt8(0x04); // Control level 4 (XPLink)
1786:   OutStreamer->emitAbsoluteSymbolDiff(CELQSTRT, PPA2Sym, 4);
1787:   OutStreamer->emitInt32(0x00000000);
1788:   OutStreamer->emitAbsoluteSymbolDiff(DateVersionSym, PPA2Sym, 4);
1789:   OutStreamer->emitInt32(
1790:       0x00000000); // Offset to main entry point, always 0 (so says TR).
1791:   uint8_t Flgs = static_cast<uint8_t>(PPA2Flags::CompileForBinaryFloatingPoint);
1792:   Flgs |= static_cast<uint8_t>(PPA2Flags::CompiledWithXPLink);
1793: 
1794:   bool IsASCII = true;
1795:   if (auto *MD = M.getModuleFlag("zos_le_char_mode")) {
1796:     const StringRef &CharMode = cast<MDString>(MD)->getString();
1797:     if (CharMode == "ebcdic")
1798:       IsASCII = false;
1799:     else if (CharMode != "ascii")
1800:       OutContext.reportError(
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1801-1840 / 第 1801-1840 行
```cpp
1801:           {}, "Only ascii or ebcdic are allowed for zos_le_char_mode");
1802:   }
1803:   if (IsASCII)
1804:     Flgs |= static_cast<uint8_t>(
1805:         PPA2Flags::CompiledUnitASCII); // Setting bit for ASCII char. mode.
1806: 
1807:   OutStreamer->emitInt8(Flgs);
1808:   OutStreamer->emitInt8(0x00);    // Reserved.
1809:                                   // No MD5 signature before timestamp.
1810:                                   // No FLOAT(AFP(VOLATILE)).
1811:                                   // Remaining 5 flag bits reserved.
1812:   OutStreamer->emitInt16(0x0000); // 16 Reserved flag bits.
1813: 
1814:   // Emit date and version section.
1815:   OutStreamer->emitLabel(DateVersionSym);
1816:   OutStreamer->emitBytes(CompilationTimeEBCDIC.str());
1817:   OutStreamer->emitBytes(VersionEBCDIC.str());
1818: 
1819:   OutStreamer->emitInt16(0x0000); // Service level string length.
1820: 
1821:   // The binder requires that the offset to the PPA2 be emitted in a different,
1822:   // specially-named section.
1823:   OutStreamer->switchSection(getObjFileLowering().getPPA2ListSection());
1824:   // Emit 8 byte alignment.
1825:   // Emit pointer to PPA2 label.
1826:   OutStreamer->AddComment("A(PPA2-CELQSTRT)");
1827:   OutStreamer->emitAbsoluteSymbolDiff(PPA2Sym, CELQSTRT, 8);
1828:   OutStreamer->popSection();
1829: }
1830: 
1831: void SystemZAsmPrinter::emitGlobalAlias(const Module &M,
1832:                                         const GlobalAlias &GA) {
1833:   if (!TM.getTargetTriple().isOSzOS())
1834:     return AsmPrinter::emitGlobalAlias(M, GA);
1835: 
1836:   // Aliased function labels have already been emitted for z/OS
1837: }
1838: 
1839: const MCExpr *SystemZAsmPrinter::lowerConstant(const Constant *CV,
1840:                                                const Constant *BaseCV,
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::emitGlobalAlias`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::emitGlobalAlias` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1841-1880 / 第 1841-1880 行
```cpp
1841:                                                uint64_t Offset) {
1842:   const Triple &TargetTriple = TM.getTargetTriple();
1843: 
1844:   if (TargetTriple.isOSzOS()) {
1845:     const GlobalAlias *GA = dyn_cast<GlobalAlias>(CV);
1846:     const GlobalVariable *GV = dyn_cast<GlobalVariable>(CV);
1847:     const Function *FV = dyn_cast<Function>(CV);
1848:     bool IsFunc = !GV && (FV || (GA && isa<Function>(GA->getAliaseeObject())));
1849: 
1850:     MCSymbol *Sym = NULL;
1851: 
1852:     if (GA)
1853:       Sym = getSymbol(GA);
1854:     else if (IsFunc)
1855:       Sym = getSymbol(FV);
1856:     else if (GV)
1857:       Sym = getSymbol(GV);
1858: 
1859:     if (IsFunc) {
1860:       OutStreamer->emitSymbolAttribute(Sym, MCSA_ELF_TypeFunction);
1861:       if (FV->hasExternalLinkage())
1862:         return MCSpecifierExpr::create(MCSymbolRefExpr::create(Sym, OutContext),
1863:                                        SystemZ::S_VCon, OutContext);
1864:       // Trigger creation of function descriptor in ADA for internal
1865:       // functions.
1866:       unsigned Disp = ADATable.insert(Sym, SystemZII::MO_ADA_DIRECT_FUNC_DESC);
1867:       return MCBinaryExpr::createAdd(
1868:           MCSpecifierExpr::create(
1869:               MCSymbolRefExpr::create(
1870:                   getObjFileLowering().getADASection()->getBeginSymbol(),
1871:                   OutContext),
1872:               SystemZ::S_None, OutContext),
1873:           MCConstantExpr::create(Disp, OutContext), OutContext);
1874:     }
1875:     if (Sym) {
1876:       OutStreamer->emitSymbolAttribute(Sym, MCSA_ELF_TypeObject);
1877:       return MCSymbolRefExpr::create(Sym, OutContext);
1878:     }
1879:   }
1880:   return AsmPrinter::lowerConstant(CV);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1881-1920 / 第 1881-1920 行
```cpp
1881: }
1882: 
1883: void SystemZAsmPrinter::emitFunctionEntryLabel() {
1884:   const SystemZSubtarget &Subtarget = MF->getSubtarget<SystemZSubtarget>();
1885: 
1886:   if (Subtarget.getTargetTriple().isOSzOS()) {
1887:     calculatePPA1();
1888: 
1889:     // EntryPoint Marker
1890:     const MachineFrameInfo &MFFrame = MF->getFrameInfo();
1891:     bool IsUsingAlloca = MFFrame.hasVarSizedObjects();
1892:     uint32_t DSASize = MFFrame.getStackSize();
1893:     bool IsLeaf = DSASize == 0 && MFFrame.getCalleeSavedInfo().empty();
1894: 
1895:     // Set Flags.
1896:     uint8_t Flags = 0;
1897:     if (IsLeaf)
1898:       Flags |= 0x08;
1899:     if (IsUsingAlloca)
1900:       Flags |= 0x04;
1901: 
1902:     // Combine into top 27 bits of DSASize and bottom 5 bits of Flags.
1903:     uint32_t DSAAndFlags = DSASize & 0xFFFFFFE0; // (x/32) << 5
1904:     DSAAndFlags |= Flags;
1905: 
1906:     // Emit entry point marker section.
1907:     OutStreamer->AddComment("XPLINK Routine Layout Entry");
1908:     OutStreamer->emitLabel(DeferredPPA1.back().EPMarker);
1909:     OutStreamer->AddComment("Eyecatcher 0x00C300C500C500");
1910:     OutStreamer->emitIntValueInHex(0x00C300C500C500, 7); // Eyecatcher.
1911:     OutStreamer->AddComment("Mark Type C'1'");
1912:     OutStreamer->emitInt8(0xF1); // Mark Type.
1913:     OutStreamer->AddComment("Offset to PPA1");
1914:     OutStreamer->emitAbsoluteSymbolDiff(DeferredPPA1.back().PPA1,
1915:                                         DeferredPPA1.back().EPMarker, 4);
1916:     if (OutStreamer->isVerboseAsm()) {
1917:       OutStreamer->AddComment("DSA Size 0x" + Twine::utohexstr(DSASize));
1918:       OutStreamer->AddComment("Entry Flags");
1919:       if (Flags & 0x08)
1920:         OutStreamer->AddComment("  Bit 1: 1 = Leaf function");
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter::emitFunctionEntryLabel`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter::emitFunctionEntryLabel` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1921-1958 / 第 1921-1958 行
```cpp
1921:       else
1922:         OutStreamer->AddComment("  Bit 1: 0 = Non-leaf function");
1923:       if (Flags & 0x04)
1924:         OutStreamer->AddComment("  Bit 2: 1 = Uses alloca");
1925:       else
1926:         OutStreamer->AddComment("  Bit 2: 0 = Does not use alloca");
1927:     }
1928:     OutStreamer->emitInt32(DSAAndFlags);
1929: 
1930:     getTargetStreamer()->emitADA(CurrentFnSym,
1931:                                  getObjFileLowering().getADASection());
1932:   }
1933: 
1934:   AsmPrinter::emitFunctionEntryLabel();
1935: 
1936:   if (Subtarget.getTargetTriple().isOSzOS()) {
1937:     const Function *F = &MF->getFunction();
1938:     // Emit aliasing label for function entry point label.
1939:     for (const GlobalAlias *Alias : GOAliasMap[F]) {
1940:       MCSymbol *Sym = getSymbol(Alias);
1941:       OutStreamer->emitSymbolAttribute(Sym, MCSA_ELF_TypeFunction);
1942:       emitVisibility(Sym, Alias->getVisibility());
1943:       emitLinkage(Alias, Sym);
1944:       OutStreamer->emitLabel(Sym);
1945:     }
1946:   }
1947: }
1948: 
1949: char SystemZAsmPrinter::ID = 0;
1950: 
1951: INITIALIZE_PASS(SystemZAsmPrinter, "systemz-asm-printer",
1952:                 "SystemZ Assembly Printer", false, false)
1953: 
1954: // Force static initialization.
1955: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
1956: LLVMInitializeSystemZAsmPrinter() {
1957:   RegisterAsmPrinter<SystemZAsmPrinter> X(getTheSystemZTarget());
1958: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **MC streaming**: Emits sections, symbols, expressions, and encoded bytes to object or assembly output. / 向目标文件或汇编输出节、符号、表达式和编码字节。
- **Assembly emission**: Prints target instructions and directives in textual assembly form. / 以文本汇编形式输出目标指令和伪指令。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- `SystemZAsmPrinter.h`
- `MCTargetDesc/SystemZGNUInstPrinter.h`
- `MCTargetDesc/SystemZHLASMInstPrinter.h`
- `MCTargetDesc/SystemZMCAsmInfo.h`
- `MCTargetDesc/SystemZMCTargetDesc.h`
- `SystemZConstantPoolValue.h`
- `SystemZMCInstLower.h`
- `TargetInfo/SystemZTargetInfo.h`
- `llvm/ADT/StringExtras.h`
- `llvm/BinaryFormat/ELF.h`
- `llvm/BinaryFormat/GOFF.h`
- `llvm/CodeGen/MachineModuleInfoImpls.h`
- `llvm/CodeGen/TargetLoweringObjectFileImpl.h`
- `llvm/IR/GlobalVariable.h`
- `llvm/IR/Mangler.h`
- `llvm/IR/Module.h`
- `...` (11 more include dependencies omitted for brevity / 其余 11 个 include 依赖已省略)
