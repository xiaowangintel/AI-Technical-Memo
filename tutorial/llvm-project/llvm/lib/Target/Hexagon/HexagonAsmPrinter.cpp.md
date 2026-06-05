# HexagonAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonAsmPrinter.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file contains a printer that converts from our internal representation
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- HexagonAsmPrinter.cpp - Print machine instrs to Hexagon assembly ---===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file contains a printer that converts from our internal representation
    10: // of machine-dependent LLVM code to Hexagon assembly language. This printer is
    11: // the output mechanism used by `llc'.
    12: //
    13: //===----------------------------------------------------------------------===//
    14: 
    15: #include "HexagonAsmPrinter.h"
    16: #include "HexagonInstrInfo.h"
    17: #include "HexagonRegisterInfo.h"
    18: #include "HexagonSubtarget.h"
    19: #include "MCTargetDesc/HexagonInstPrinter.h"
    20: #include "MCTargetDesc/HexagonMCExpr.h"
    21: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    22: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    23: #include "MCTargetDesc/HexagonTargetStreamer.h"
    24: #include "TargetInfo/HexagonTargetInfo.h"
    25: #include "llvm/ADT/StringExtras.h"
    26: #include "llvm/ADT/StringRef.h"
    27: #include "llvm/ADT/Twine.h"
    28: #include "llvm/BinaryFormat/ELF.h"
    29: #include "llvm/CodeGen/AsmPrinter.h"
    30: #include "llvm/CodeGen/MachineBasicBlock.h"
    31: #include "llvm/CodeGen/MachineFunction.h"
    32: #include "llvm/CodeGen/MachineInstr.h"
    33: #include "llvm/CodeGen/MachineOperand.h"
    34: #include "llvm/CodeGen/TargetRegisterInfo.h"
    35: #include "llvm/CodeGen/TargetSubtargetInfo.h"
    36: #include "llvm/MC/MCContext.h"
    37: #include "llvm/MC/MCDirectives.h"
    38: #include "llvm/MC/MCExpr.h"
    39: #include "llvm/MC/MCInst.h"
    40: #include "llvm/MC/MCRegisterInfo.h"
    41: #include "llvm/MC/MCSectionELF.h"
    42: #include "llvm/MC/MCStreamer.h"
    43: #include "llvm/MC/MCSymbol.h"
    44: #include "llvm/MC/TargetRegistry.h"
    45: #include "llvm/Support/Casting.h"
    46: #include "llvm/Support/Compiler.h"
    47: #include "llvm/Support/ErrorHandling.h"
    48: #include "llvm/Support/raw_ostream.h"
    49: #include "llvm/Target/TargetMachine.h"
    50: #include <cassert>
    51: #include <cstdint>
    52: #include <string>
    53: 
    54: using namespace llvm;
    55: 
    56: namespace llvm {
    57: 
    58: void HexagonLowerToMC(const MCInstrInfo &MCII, const MachineInstr *MI,
    59:                       MCInst &MCB, HexagonAsmPrinter &AP);
    60: 
    61: } // end namespace llvm
    62: 
    63: #define DEBUG_TYPE "asm-printer"
    64: 
    65: // Given a scalar register return its pair.
    66: inline static unsigned getHexagonRegisterPair(unsigned Reg,
    67:       const MCRegisterInfo *RI) {
    68:   assert(Hexagon::IntRegsRegClass.contains(Reg));
    69:   unsigned Pair = *RI->superregs(Reg).begin();
    70:   assert(Hexagon::DoubleRegsRegClass.contains(Pair));
    71:   return Pair;
    72: }
    73: 
    74: void HexagonAsmPrinter::printOperand(const MachineInstr *MI, unsigned OpNo,
    75:                                      raw_ostream &O) {
    76:   const MachineOperand &MO = MI->getOperand(OpNo);
    77: 
    78:   switch (MO.getType()) {
    79:   default:
    80:     llvm_unreachable ("<unknown operand type>");
    81:   case MachineOperand::MO_Register:
    82:     O << HexagonInstPrinter::getRegisterName(MO.getReg());
    83:     return;
    84:   case MachineOperand::MO_Immediate:
    85:     O << MO.getImm();
    86:     return;
    87:   case MachineOperand::MO_MachineBasicBlock:
    88:     MO.getMBB()->getSymbol()->print(O, MAI);
    89:     return;
    90:   case MachineOperand::MO_ConstantPoolIndex:
    91:     GetCPISymbol(MO.getIndex())->print(O, MAI);
    92:     return;
    93:   case MachineOperand::MO_GlobalAddress:
    94:     PrintSymbolOperand(MO, O);
    95:     return;
    96:   }
    97: }
    98: 
    99: // isBlockOnlyReachableByFallthrough - We need to override this since the
   100: // default AsmPrinter does not print labels for any basic block that
```
- EN: It imports headers such as HexagonAsmPrinter.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, ... (38 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonLowerToMC, getHexagonRegisterPair, assert, superregs, ... (12 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonAsmPrinter.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, ... (38 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonLowerToMC, getHexagonRegisterPair, assert, superregs, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-200 / 第 101-200 行

```cpp
   101: // is only reachable by a fall through. That works for all cases except
   102: // for the case in which the basic block is reachable by a fall through but
   103: // through an indirect from a jump table. In this case, the jump table
   104: // will contain a label not defined by AsmPrinter.
   105: bool HexagonAsmPrinter::isBlockOnlyReachableByFallthrough(
   106:       const MachineBasicBlock *MBB) const {
   107:   if (MBB->hasAddressTaken())
   108:     return false;
   109:   return AsmPrinter::isBlockOnlyReachableByFallthrough(MBB);
   110: }
   111: 
   112: /// PrintAsmOperand - Print out an operand for an inline asm expression.
   113: bool HexagonAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
   114:                                         const char *ExtraCode,
   115:                                         raw_ostream &OS) {
   116:   // Does this asm operand have a single letter operand modifier?
   117:   if (ExtraCode && ExtraCode[0]) {
   118:     if (ExtraCode[1] != 0)
   119:       return true; // Unknown modifier.
   120: 
   121:     switch (ExtraCode[0]) {
   122:     default:
   123:       // See if this is a generic print operand
   124:       return AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, OS);
   125:     case 'L':
   126:     case 'H': { // The highest-numbered register of a pair.
   127:       const MachineOperand &MO = MI->getOperand(OpNo);
   128:       const MachineFunction &MF = *MI->getParent()->getParent();
   129:       const TargetRegisterInfo *TRI = MF.getSubtarget().getRegisterInfo();
   130:       if (!MO.isReg())
   131:         return true;
   132:       Register RegNumber = MO.getReg();
   133:       // This should be an assert in the frontend.
   134:       if (Hexagon::DoubleRegsRegClass.contains(RegNumber))
   135:         RegNumber = TRI->getSubReg(RegNumber, ExtraCode[0] == 'L' ?
   136:                                               Hexagon::isub_lo :
   137:                                               Hexagon::isub_hi);
   138:       OS << HexagonInstPrinter::getRegisterName(RegNumber);
   139:       return false;
   140:     }
   141:     case 'I':
   142:       // Write 'i' if an integer constant, otherwise nothing.  Used to print
   143:       // addi vs add, etc.
   144:       if (MI->getOperand(OpNo).isImm())
   145:         OS << "i";
   146:       return false;
   147:     }
   148:   }
   149: 
   150:   printOperand(MI, OpNo, OS);
   151:   return false;
   152: }
   153: 
   154: bool HexagonAsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI,
   155:                                               unsigned OpNo,
   156:                                               const char *ExtraCode,
   157:                                               raw_ostream &O) {
   158:   if (ExtraCode && ExtraCode[0])
   159:     return true; // Unknown modifier.
   160: 
   161:   const MachineOperand &Base  = MI->getOperand(OpNo);
   162:   const MachineOperand &Offset = MI->getOperand(OpNo+1);
   163: 
   164:   if (Base.isReg())
   165:     printOperand(MI, OpNo, O);
   166:   else
   167:     llvm_unreachable("Unimplemented");
   168: 
   169:   if (Offset.isImm()) {
   170:     if (Offset.getImm())
   171:       O << "+#" << Offset.getImm();
   172:   } else {
   173:     llvm_unreachable("Unimplemented");
   174:   }
   175: 
   176:   return false;
   177: }
   178: 
   179: static MCSymbol *smallData(AsmPrinter &AP, const MachineInstr &MI,
   180:                            MCStreamer &OutStreamer, const MCOperand &Imm,
   181:                            int AlignSize, const MCSubtargetInfo& STI) {
   182:   MCSymbol *Sym;
   183:   int64_t Value;
   184:   if (Imm.getExpr()->evaluateAsAbsolute(Value)) {
   185:     StringRef sectionPrefix;
   186:     std::string ImmString;
   187:     StringRef Name;
   188:     if (AlignSize == 8) {
   189:        Name = ".CONST_0000000000000000";
   190:        sectionPrefix = ".gnu.linkonce.l8";
   191:        ImmString = utohexstr(Value);
   192:     } else {
   193:        Name = ".CONST_00000000";
   194:        sectionPrefix = ".gnu.linkonce.l4";
   195:        ImmString = utohexstr(static_cast<uint32_t>(Value));
   196:     }
   197: 
   198:     std::string symbolName =   // Yes, leading zeros are kept.
   199:       Name.drop_back(ImmString.size()).str() + ImmString;
   200:     std::string sectionName = sectionPrefix.str() + symbolName;
```
- EN: It declares or implements routines such as HexagonAsmPrinter::isBlockOnlyReachableByFallthrough, AsmPrinter::isBlockOnlyReachableByFallthrough, HexagonAsmPrinter::PrintAsmOperand, AsmPrinter::PrintAsmOperand, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonAsmPrinter, HexagonInstPrinter, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonAsmPrinter::isBlockOnlyReachableByFallthrough, AsmPrinter::isBlockOnlyReachableByFallthrough, HexagonAsmPrinter::PrintAsmOperand, AsmPrinter::PrintAsmOperand, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonAsmPrinter, HexagonInstPrinter，说明了它与同级后端组件的连接关系。

### Lines 201-300 / 第 201-300 行

```cpp
   201: 
   202:     MCSectionELF *Section = OutStreamer.getContext().getELFSection(
   203:         sectionName, ELF::SHT_PROGBITS, ELF::SHF_WRITE | ELF::SHF_ALLOC);
   204:     OutStreamer.switchSection(Section);
   205: 
   206:     Sym = AP.OutContext.getOrCreateSymbol(Twine(symbolName));
   207:     if (Sym->isUndefined()) {
   208:       OutStreamer.emitLabel(Sym);
   209:       OutStreamer.emitSymbolAttribute(Sym, MCSA_Global);
   210:       OutStreamer.emitIntValue(Value, AlignSize);
   211:       OutStreamer.emitCodeAlignment(Align(AlignSize), &STI);
   212:     }
   213:   } else {
   214:     assert(Imm.isExpr() && "Expected expression and found none");
   215:     const MachineOperand &MO = MI.getOperand(1);
   216:     assert(MO.isGlobal() || MO.isCPI() || MO.isJTI());
   217:     MCSymbol *MOSymbol = nullptr;
   218:     if (MO.isGlobal())
   219:       MOSymbol = AP.getSymbol(MO.getGlobal());
   220:     else if (MO.isCPI())
   221:       MOSymbol = AP.GetCPISymbol(MO.getIndex());
   222:     else if (MO.isJTI())
   223:       MOSymbol = AP.GetJTISymbol(MO.getIndex());
   224:     else
   225:       llvm_unreachable("Unknown operand type!");
   226: 
   227:     StringRef SymbolName = MOSymbol->getName();
   228:     std::string LitaName = ".CONST_" + SymbolName.str();
   229: 
   230:     MCSectionELF *Section = OutStreamer.getContext().getELFSection(
   231:         ".lita", ELF::SHT_PROGBITS, ELF::SHF_WRITE | ELF::SHF_ALLOC);
   232: 
   233:     OutStreamer.switchSection(Section);
   234:     Sym = AP.OutContext.getOrCreateSymbol(Twine(LitaName));
   235:     if (Sym->isUndefined()) {
   236:       OutStreamer.emitLabel(Sym);
   237:       OutStreamer.emitSymbolAttribute(Sym, MCSA_Local);
   238:       OutStreamer.emitValue(Imm.getExpr(), AlignSize);
   239:       OutStreamer.emitCodeAlignment(Align(AlignSize), &STI);
   240:     }
   241:   }
   242:   return Sym;
   243: }
   244: 
   245: static MCInst ScaleVectorOffset(MCInst &Inst, unsigned OpNo,
   246:                                 unsigned VectorSize, MCContext &Ctx) {
   247:   MCInst T;
   248:   T.setOpcode(Inst.getOpcode());
   249:   for (unsigned i = 0, n = Inst.getNumOperands(); i != n; ++i) {
   250:     if (i != OpNo) {
   251:       T.addOperand(Inst.getOperand(i));
   252:       continue;
   253:     }
   254:     MCOperand &ImmOp = Inst.getOperand(i);
   255:     const auto *HE = static_cast<const HexagonMCExpr*>(ImmOp.getExpr());
   256:     int32_t V = cast<MCConstantExpr>(HE->getExpr())->getValue();
   257:     auto *NewCE = MCConstantExpr::create(V / int32_t(VectorSize), Ctx);
   258:     auto *NewHE = HexagonMCExpr::create(NewCE, Ctx);
   259:     T.addOperand(MCOperand::createExpr(NewHE));
   260:   }
   261:   return T;
   262: }
   263: 
   264: void HexagonAsmPrinter::HexagonProcessInstruction(MCInst &Inst,
   265:                                                   const MachineInstr &MI) {
   266:   MCInst &MappedInst = static_cast <MCInst &>(Inst);
   267:   const MCRegisterInfo *RI = OutStreamer->getContext().getRegisterInfo();
   268:   const MachineFunction &MF = *MI.getParent()->getParent();
   269:   auto &HRI = *MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
   270:   unsigned VectorSize = HRI.getRegSizeInBits(Hexagon::HvxVRRegClass) / 8;
   271: 
   272:   switch (Inst.getOpcode()) {
   273:   default:
   274:     return;
   275: 
   276:   case Hexagon::A2_iconst: {
   277:     Inst.setOpcode(Hexagon::A2_addi);
   278:     MCOperand Reg = Inst.getOperand(0);
   279:     MCOperand S16 = Inst.getOperand(1);
   280:     HexagonMCInstrInfo::setMustNotExtend(*S16.getExpr());
   281:     HexagonMCInstrInfo::setS27_2_reloc(*S16.getExpr());
   282:     Inst.clear();
   283:     Inst.addOperand(Reg);
   284:     Inst.addOperand(MCOperand::createReg(Hexagon::R0));
   285:     Inst.addOperand(S16);
   286:     break;
   287:   }
   288: 
   289:   case Hexagon::A2_tfrf: {
   290:     const MCConstantExpr *Zero = MCConstantExpr::create(0, OutContext);
   291:     Inst.setOpcode(Hexagon::A2_paddif);
   292:     Inst.addOperand(MCOperand::createExpr(Zero));
   293:     break;
   294:   }
   295: 
   296:   case Hexagon::A2_tfrt: {
   297:     const MCConstantExpr *Zero = MCConstantExpr::create(0, OutContext);
   298:     Inst.setOpcode(Hexagon::A2_paddit);
   299:     Inst.addOperand(MCOperand::createExpr(Zero));
   300:     break;
```
- EN: It declares or implements routines such as getContext, switchSection, getOrCreateSymbol, emitLabel, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCExpr, HexagonAsmPrinter, HexagonProcessInstruction, HexagonSubtarget, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getContext, switchSection, getOrCreateSymbol, emitLabel, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCExpr, HexagonAsmPrinter, HexagonProcessInstruction, HexagonSubtarget, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 301-400 / 第 301-400 行

```cpp
   301:   }
   302: 
   303:   case Hexagon::A2_tfrfnew: {
   304:     const MCConstantExpr *Zero = MCConstantExpr::create(0, OutContext);
   305:     Inst.setOpcode(Hexagon::A2_paddifnew);
   306:     Inst.addOperand(MCOperand::createExpr(Zero));
   307:     break;
   308:   }
   309: 
   310:   case Hexagon::A2_tfrtnew: {
   311:     const MCConstantExpr *Zero = MCConstantExpr::create(0, OutContext);
   312:     Inst.setOpcode(Hexagon::A2_padditnew);
   313:     Inst.addOperand(MCOperand::createExpr(Zero));
   314:     break;
   315:   }
   316: 
   317:   case Hexagon::A2_zxtb: {
   318:     const MCConstantExpr *C255 = MCConstantExpr::create(255, OutContext);
   319:     Inst.setOpcode(Hexagon::A2_andir);
   320:     Inst.addOperand(MCOperand::createExpr(C255));
   321:     break;
   322:   }
   323: 
   324:   // "$dst = CONST64(#$src1)",
   325:   case Hexagon::CONST64:
   326:     if (!OutStreamer->hasRawTextSupport()) {
   327:       const MCOperand &Imm = MappedInst.getOperand(1);
   328:       MCSectionSubPair Current = OutStreamer->getCurrentSection();
   329: 
   330:       MCSymbol *Sym =
   331:           smallData(*this, MI, *OutStreamer, Imm, 8, getSubtargetInfo());
   332: 
   333:       OutStreamer->switchSection(Current.first, Current.second);
   334:       MCInst TmpInst;
   335:       MCOperand &Reg = MappedInst.getOperand(0);
   336:       TmpInst.setOpcode(Hexagon::L2_loadrdgp);
   337:       TmpInst.addOperand(Reg);
   338:       TmpInst.addOperand(MCOperand::createExpr(
   339:                          MCSymbolRefExpr::create(Sym, OutContext)));
   340:       MappedInst = TmpInst;
   341: 
   342:     }
   343:     break;
   344:   case Hexagon::CONST32:
   345:     if (!OutStreamer->hasRawTextSupport()) {
   346:       MCOperand &Imm = MappedInst.getOperand(1);
   347:       MCSectionSubPair Current = OutStreamer->getCurrentSection();
   348:       MCSymbol *Sym =
   349:           smallData(*this, MI, *OutStreamer, Imm, 4, getSubtargetInfo());
   350:       OutStreamer->switchSection(Current.first, Current.second);
   351:       MCInst TmpInst;
   352:       MCOperand &Reg = MappedInst.getOperand(0);
   353:       TmpInst.setOpcode(Hexagon::L2_loadrigp);
   354:       TmpInst.addOperand(Reg);
   355:       TmpInst.addOperand(MCOperand::createExpr(HexagonMCExpr::create(
   356:           MCSymbolRefExpr::create(Sym, OutContext), OutContext)));
   357:       MappedInst = TmpInst;
   358:     }
   359:     break;
   360: 
   361:   // C2_pxfer_map maps to C2_or instruction. Though, it's possible to use
   362:   // C2_or during instruction selection itself but it results
   363:   // into suboptimal code.
   364:   case Hexagon::C2_pxfer_map: {
   365:     MCOperand &Ps = Inst.getOperand(1);
   366:     MappedInst.setOpcode(Hexagon::C2_or);
   367:     MappedInst.addOperand(Ps);
   368:     return;
   369:   }
   370: 
   371:   // Vector reduce complex multiply by scalar, Rt & 1 map to :hi else :lo
   372:   // The insn is mapped from the 4 operand to the 3 operand raw form taking
   373:   // 3 register pairs.
   374:   case Hexagon::M2_vrcmpys_acc_s1: {
   375:     MCOperand &Rt = Inst.getOperand(3);
   376:     assert(Rt.isReg() && "Expected register and none was found");
   377:     unsigned Reg = RI->getEncodingValue(Rt.getReg());
   378:     if (Reg & 1)
   379:       MappedInst.setOpcode(Hexagon::M2_vrcmpys_acc_s1_h);
   380:     else
   381:       MappedInst.setOpcode(Hexagon::M2_vrcmpys_acc_s1_l);
   382:     Rt.setReg(getHexagonRegisterPair(Rt.getReg(), RI));
   383:     return;
   384:   }
   385:   case Hexagon::M2_vrcmpys_s1: {
   386:     MCOperand &Rt = Inst.getOperand(2);
   387:     assert(Rt.isReg() && "Expected register and none was found");
   388:     unsigned Reg = RI->getEncodingValue(Rt.getReg());
   389:     if (Reg & 1)
   390:       MappedInst.setOpcode(Hexagon::M2_vrcmpys_s1_h);
   391:     else
   392:       MappedInst.setOpcode(Hexagon::M2_vrcmpys_s1_l);
   393:     Rt.setReg(getHexagonRegisterPair(Rt.getReg(), RI));
   394:     return;
   395:   }
   396: 
   397:   case Hexagon::M2_vrcmpys_s1rp: {
   398:     MCOperand &Rt = Inst.getOperand(2);
   399:     assert(Rt.isReg() && "Expected register and none was found");
   400:     unsigned Reg = RI->getEncodingValue(Rt.getReg());
```
- EN: It declares or implements routines such as MCConstantExpr::create, setOpcode, addOperand, CONST64, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCExpr, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 MCConstantExpr::create, setOpcode, addOperand, CONST64, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCExpr，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401:     if (Reg & 1)
   402:       MappedInst.setOpcode(Hexagon::M2_vrcmpys_s1rp_h);
   403:     else
   404:       MappedInst.setOpcode(Hexagon::M2_vrcmpys_s1rp_l);
   405:     Rt.setReg(getHexagonRegisterPair(Rt.getReg(), RI));
   406:     return;
   407:   }
   408: 
   409:   case Hexagon::A4_boundscheck: {
   410:     MCOperand &Rs = Inst.getOperand(1);
   411:     assert(Rs.isReg() && "Expected register and none was found");
   412:     unsigned Reg = RI->getEncodingValue(Rs.getReg());
   413:     if (Reg & 1) // Odd mapped to raw:hi, regpair is rodd:odd-1, like r3:2
   414:       MappedInst.setOpcode(Hexagon::A4_boundscheck_hi);
   415:     else         // raw:lo
   416:       MappedInst.setOpcode(Hexagon::A4_boundscheck_lo);
   417:     Rs.setReg(getHexagonRegisterPair(Rs.getReg(), RI));
   418:     return;
   419:   }
   420: 
   421:   case Hexagon::PS_call_nr:
   422:     Inst.setOpcode(Hexagon::J2_call);
   423:     break;
   424: 
   425:   case Hexagon::PS_readcr:
   426:     Inst.setOpcode(Hexagon::A2_tfrcrr);
   427:     break;
   428: 
   429:   case Hexagon::PS_readcr64:
   430:     Inst.setOpcode(Hexagon::A4_tfrcpp);
   431:     break;
   432: 
   433:   case Hexagon::S5_asrhub_rnd_sat_goodsyntax: {
   434:     MCOperand &MO = MappedInst.getOperand(2);
   435:     int64_t Imm;
   436:     MCExpr const *Expr = MO.getExpr();
   437:     bool Success = Expr->evaluateAsAbsolute(Imm);
   438:     assert(Success && "Expected immediate and none was found");
   439:     (void)Success;
   440:     MCInst TmpInst;
   441:     if (Imm == 0) {
   442:       TmpInst.setOpcode(Hexagon::S2_vsathub);
   443:       TmpInst.addOperand(MappedInst.getOperand(0));
   444:       TmpInst.addOperand(MappedInst.getOperand(1));
   445:       MappedInst = TmpInst;
   446:       return;
   447:     }
   448:     TmpInst.setOpcode(Hexagon::S5_asrhub_rnd_sat);
   449:     TmpInst.addOperand(MappedInst.getOperand(0));
   450:     TmpInst.addOperand(MappedInst.getOperand(1));
   451:     const MCExpr *One = MCConstantExpr::create(1, OutContext);
   452:     const MCExpr *Sub = MCBinaryExpr::createSub(Expr, One, OutContext);
   453:     TmpInst.addOperand(
   454:         MCOperand::createExpr(HexagonMCExpr::create(Sub, OutContext)));
   455:     MappedInst = TmpInst;
   456:     return;
   457:   }
   458: 
   459:   case Hexagon::S5_vasrhrnd_goodsyntax:
   460:   case Hexagon::S2_asr_i_p_rnd_goodsyntax: {
   461:     MCOperand &MO2 = MappedInst.getOperand(2);
   462:     MCExpr const *Expr = MO2.getExpr();
   463:     int64_t Imm;
   464:     bool Success = Expr->evaluateAsAbsolute(Imm);
   465:     assert(Success && "Expected immediate and none was found");
   466:     (void)Success;
   467:     MCInst TmpInst;
   468:     if (Imm == 0) {
   469:       TmpInst.setOpcode(Hexagon::A2_combinew);
   470:       TmpInst.addOperand(MappedInst.getOperand(0));
   471:       MCOperand &MO1 = MappedInst.getOperand(1);
   472:       MCRegister High = RI->getSubReg(MO1.getReg(), Hexagon::isub_hi);
   473:       MCRegister Low = RI->getSubReg(MO1.getReg(), Hexagon::isub_lo);
   474:       // Add a new operand for the second register in the pair.
   475:       TmpInst.addOperand(MCOperand::createReg(High));
   476:       TmpInst.addOperand(MCOperand::createReg(Low));
   477:       MappedInst = TmpInst;
   478:       return;
   479:     }
   480: 
   481:     if (Inst.getOpcode() == Hexagon::S2_asr_i_p_rnd_goodsyntax)
   482:       TmpInst.setOpcode(Hexagon::S2_asr_i_p_rnd);
   483:     else
   484:       TmpInst.setOpcode(Hexagon::S5_vasrhrnd);
   485:     TmpInst.addOperand(MappedInst.getOperand(0));
   486:     TmpInst.addOperand(MappedInst.getOperand(1));
   487:     const MCExpr *One = MCConstantExpr::create(1, OutContext);
   488:     const MCExpr *Sub = MCBinaryExpr::createSub(Expr, One, OutContext);
   489:     TmpInst.addOperand(
   490:         MCOperand::createExpr(HexagonMCExpr::create(Sub, OutContext)));
   491:     MappedInst = TmpInst;
   492:     return;
   493:   }
   494: 
   495:   // if ("#u5==0") Assembler mapped to: "Rd=Rs"; else Rd=asr(Rs,#u5-1):rnd
   496:   case Hexagon::S2_asr_i_r_rnd_goodsyntax: {
   497:     MCOperand &MO = Inst.getOperand(2);
   498:     MCExpr const *Expr = MO.getExpr();
   499:     int64_t Imm;
   500:     bool Success = Expr->evaluateAsAbsolute(Imm);
```
- EN: It declares or implements routines such as setOpcode, setReg, getOperand, assert, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCExpr, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 setOpcode, setReg, getOperand, assert, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCExpr，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501:     assert(Success && "Expected immediate and none was found");
   502:     (void)Success;
   503:     MCInst TmpInst;
   504:     if (Imm == 0) {
   505:       TmpInst.setOpcode(Hexagon::A2_tfr);
   506:       TmpInst.addOperand(MappedInst.getOperand(0));
   507:       TmpInst.addOperand(MappedInst.getOperand(1));
   508:       MappedInst = TmpInst;
   509:       return;
   510:     }
   511:     TmpInst.setOpcode(Hexagon::S2_asr_i_r_rnd);
   512:     TmpInst.addOperand(MappedInst.getOperand(0));
   513:     TmpInst.addOperand(MappedInst.getOperand(1));
   514:     const MCExpr *One = MCConstantExpr::create(1, OutContext);
   515:     const MCExpr *Sub = MCBinaryExpr::createSub(Expr, One, OutContext);
   516:     TmpInst.addOperand(
   517:         MCOperand::createExpr(HexagonMCExpr::create(Sub, OutContext)));
   518:     MappedInst = TmpInst;
   519:     return;
   520:   }
   521: 
   522:   // Translate a "$Rdd = #imm" to "$Rdd = combine(#[-1,0], #imm)"
   523:   case Hexagon::A2_tfrpi: {
   524:     MCInst TmpInst;
   525:     MCOperand &Rdd = MappedInst.getOperand(0);
   526:     MCOperand &MO = MappedInst.getOperand(1);
   527: 
   528:     TmpInst.setOpcode(Hexagon::A2_combineii);
   529:     TmpInst.addOperand(Rdd);
   530:     int64_t Imm;
   531:     bool Success = MO.getExpr()->evaluateAsAbsolute(Imm);
   532:     if (Success && Imm < 0) {
   533:       const MCExpr *MOne = MCConstantExpr::create(-1, OutContext);
   534:       const HexagonMCExpr *E = HexagonMCExpr::create(MOne, OutContext);
   535:       TmpInst.addOperand(MCOperand::createExpr(E));
   536:     } else {
   537:       const MCExpr *Zero = MCConstantExpr::create(0, OutContext);
   538:       const HexagonMCExpr *E = HexagonMCExpr::create(Zero, OutContext);
   539:       TmpInst.addOperand(MCOperand::createExpr(E));
   540:     }
   541:     TmpInst.addOperand(MO);
   542:     MappedInst = TmpInst;
   543:     return;
   544:   }
   545: 
   546:   // Translate a "$Rdd = $Rss" to "$Rdd = combine($Rs, $Rt)"
   547:   case Hexagon::A2_tfrp: {
   548:     MCOperand &MO = MappedInst.getOperand(1);
   549:     MCRegister High = RI->getSubReg(MO.getReg(), Hexagon::isub_hi);
   550:     MCRegister Low = RI->getSubReg(MO.getReg(), Hexagon::isub_lo);
   551:     MO.setReg(High);
   552:     // Add a new operand for the second register in the pair.
   553:     MappedInst.addOperand(MCOperand::createReg(Low));
   554:     MappedInst.setOpcode(Hexagon::A2_combinew);
   555:     return;
   556:   }
   557: 
   558:   case Hexagon::A2_tfrpt:
   559:   case Hexagon::A2_tfrpf: {
   560:     MCOperand &MO = MappedInst.getOperand(2);
   561:     MCRegister High = RI->getSubReg(MO.getReg(), Hexagon::isub_hi);
   562:     MCRegister Low = RI->getSubReg(MO.getReg(), Hexagon::isub_lo);
   563:     MO.setReg(High);
   564:     // Add a new operand for the second register in the pair.
   565:     MappedInst.addOperand(MCOperand::createReg(Low));
   566:     MappedInst.setOpcode((Inst.getOpcode() == Hexagon::A2_tfrpt)
   567:                           ? Hexagon::C2_ccombinewt
   568:                           : Hexagon::C2_ccombinewf);
   569:     return;
   570:   }
   571: 
   572:   case Hexagon::A2_tfrptnew:
   573:   case Hexagon::A2_tfrpfnew: {
   574:     MCOperand &MO = MappedInst.getOperand(2);
   575:     MCRegister High = RI->getSubReg(MO.getReg(), Hexagon::isub_hi);
   576:     MCRegister Low = RI->getSubReg(MO.getReg(), Hexagon::isub_lo);
   577:     MO.setReg(High);
   578:     // Add a new operand for the second register in the pair.
   579:     MappedInst.addOperand(MCOperand::createReg(Low));
   580:     MappedInst.setOpcode(Inst.getOpcode() == Hexagon::A2_tfrptnew
   581:                             ? Hexagon::C2_ccombinewnewt
   582:                             : Hexagon::C2_ccombinewnewf);
   583:     return;
   584:   }
   585: 
   586:   case Hexagon::M2_mpysmi: {
   587:     MCOperand &Imm = MappedInst.getOperand(2);
   588:     MCExpr const *Expr = Imm.getExpr();
   589:     int64_t Value;
   590:     bool Success = Expr->evaluateAsAbsolute(Value);
   591:     assert(Success);
   592:     (void)Success;
   593:     if (Value < 0 && Value > -256) {
   594:       MappedInst.setOpcode(Hexagon::M2_mpysin);
   595:       Imm.setExpr(HexagonMCExpr::create(
   596:           MCUnaryExpr::createMinus(Expr, OutContext), OutContext));
   597:     } else
   598:       MappedInst.setOpcode(Hexagon::M2_mpysip);
   599:     return;
   600:   }
```
- EN: It declares or implements routines such as assert, setOpcode, addOperand, MCConstantExpr::create, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCExpr, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 assert, setOpcode, addOperand, MCConstantExpr::create, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCExpr，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601: 
   602:   case Hexagon::A2_addsp: {
   603:     MCOperand &Rt = Inst.getOperand(1);
   604:     assert(Rt.isReg() && "Expected register and none was found");
   605:     unsigned Reg = RI->getEncodingValue(Rt.getReg());
   606:     if (Reg & 1)
   607:       MappedInst.setOpcode(Hexagon::A2_addsph);
   608:     else
   609:       MappedInst.setOpcode(Hexagon::A2_addspl);
   610:     Rt.setReg(getHexagonRegisterPair(Rt.getReg(), RI));
   611:     return;
   612:   }
   613: 
   614:   case Hexagon::V6_vd0: {
   615:     MCInst TmpInst;
   616:     assert(Inst.getOperand(0).isReg() &&
   617:            "Expected register and none was found");
   618: 
   619:     TmpInst.setOpcode(Hexagon::V6_vxor);
   620:     TmpInst.addOperand(Inst.getOperand(0));
   621:     TmpInst.addOperand(Inst.getOperand(0));
   622:     TmpInst.addOperand(Inst.getOperand(0));
   623:     MappedInst = TmpInst;
   624:     return;
   625:   }
   626: 
   627:   case Hexagon::V6_vdd0: {
   628:     MCInst TmpInst;
   629:     assert (Inst.getOperand(0).isReg() &&
   630:             "Expected register and none was found");
   631: 
   632:     TmpInst.setOpcode(Hexagon::V6_vsubw_dv);
   633:     TmpInst.addOperand(Inst.getOperand(0));
   634:     TmpInst.addOperand(Inst.getOperand(0));
   635:     TmpInst.addOperand(Inst.getOperand(0));
   636:     MappedInst = TmpInst;
   637:     return;
   638:   }
   639: 
   640:   case Hexagon::V6_vL32Ub_pi:
   641:   case Hexagon::V6_vL32b_cur_pi:
   642:   case Hexagon::V6_vL32b_nt_cur_pi:
   643:   case Hexagon::V6_vL32b_pi:
   644:   case Hexagon::V6_vL32b_nt_pi:
   645:   case Hexagon::V6_vL32b_nt_tmp_pi:
   646:   case Hexagon::V6_vL32b_tmp_pi:
   647:     MappedInst = ScaleVectorOffset(Inst, 3, VectorSize, OutContext);
   648:     return;
   649: 
   650:   case Hexagon::V6_vL32Ub_ai:
   651:   case Hexagon::V6_vL32b_ai:
   652:   case Hexagon::V6_vL32b_cur_ai:
   653:   case Hexagon::V6_vL32b_nt_ai:
   654:   case Hexagon::V6_vL32b_nt_cur_ai:
   655:   case Hexagon::V6_vL32b_nt_tmp_ai:
   656:   case Hexagon::V6_vL32b_tmp_ai:
   657:     MappedInst = ScaleVectorOffset(Inst, 2, VectorSize, OutContext);
   658:     return;
   659: 
   660:   case Hexagon::V6_vS32Ub_pi:
   661:   case Hexagon::V6_vS32b_new_pi:
   662:   case Hexagon::V6_vS32b_nt_new_pi:
   663:   case Hexagon::V6_vS32b_nt_pi:
   664:   case Hexagon::V6_vS32b_pi:
   665:     MappedInst = ScaleVectorOffset(Inst, 2, VectorSize, OutContext);
   666:     return;
   667: 
   668:   case Hexagon::V6_vS32Ub_ai:
   669:   case Hexagon::V6_vS32b_ai:
   670:   case Hexagon::V6_vS32b_new_ai:
   671:   case Hexagon::V6_vS32b_nt_ai:
   672:   case Hexagon::V6_vS32b_nt_new_ai:
   673:     MappedInst = ScaleVectorOffset(Inst, 1, VectorSize, OutContext);
   674:     return;
   675: 
   676:   case Hexagon::V6_vL32b_cur_npred_pi:
   677:   case Hexagon::V6_vL32b_cur_pred_pi:
   678:   case Hexagon::V6_vL32b_npred_pi:
   679:   case Hexagon::V6_vL32b_nt_cur_npred_pi:
   680:   case Hexagon::V6_vL32b_nt_cur_pred_pi:
   681:   case Hexagon::V6_vL32b_nt_npred_pi:
   682:   case Hexagon::V6_vL32b_nt_pred_pi:
   683:   case Hexagon::V6_vL32b_nt_tmp_npred_pi:
   684:   case Hexagon::V6_vL32b_nt_tmp_pred_pi:
   685:   case Hexagon::V6_vL32b_pred_pi:
   686:   case Hexagon::V6_vL32b_tmp_npred_pi:
   687:   case Hexagon::V6_vL32b_tmp_pred_pi:
   688:     MappedInst = ScaleVectorOffset(Inst, 4, VectorSize, OutContext);
   689:     return;
   690: 
   691:   case Hexagon::V6_vL32b_cur_npred_ai:
   692:   case Hexagon::V6_vL32b_cur_pred_ai:
   693:   case Hexagon::V6_vL32b_npred_ai:
   694:   case Hexagon::V6_vL32b_nt_cur_npred_ai:
   695:   case Hexagon::V6_vL32b_nt_cur_pred_ai:
   696:   case Hexagon::V6_vL32b_nt_npred_ai:
   697:   case Hexagon::V6_vL32b_nt_pred_ai:
   698:   case Hexagon::V6_vL32b_nt_tmp_npred_ai:
   699:   case Hexagon::V6_vL32b_nt_tmp_pred_ai:
   700:   case Hexagon::V6_vL32b_pred_ai:
```
- EN: It declares or implements routines such as getOperand, assert, getEncodingValue, setOpcode, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 getOperand, assert, getEncodingValue, setOpcode, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 701-800 / 第 701-800 行

```cpp
   701:   case Hexagon::V6_vL32b_tmp_npred_ai:
   702:   case Hexagon::V6_vL32b_tmp_pred_ai:
   703:     MappedInst = ScaleVectorOffset(Inst, 3, VectorSize, OutContext);
   704:     return;
   705: 
   706:   case Hexagon::V6_vS32Ub_npred_pi:
   707:   case Hexagon::V6_vS32Ub_pred_pi:
   708:   case Hexagon::V6_vS32b_new_npred_pi:
   709:   case Hexagon::V6_vS32b_new_pred_pi:
   710:   case Hexagon::V6_vS32b_npred_pi:
   711:   case Hexagon::V6_vS32b_nqpred_pi:
   712:   case Hexagon::V6_vS32b_nt_new_npred_pi:
   713:   case Hexagon::V6_vS32b_nt_new_pred_pi:
   714:   case Hexagon::V6_vS32b_nt_npred_pi:
   715:   case Hexagon::V6_vS32b_nt_nqpred_pi:
   716:   case Hexagon::V6_vS32b_nt_pred_pi:
   717:   case Hexagon::V6_vS32b_nt_qpred_pi:
   718:   case Hexagon::V6_vS32b_pred_pi:
   719:   case Hexagon::V6_vS32b_qpred_pi:
   720:     MappedInst = ScaleVectorOffset(Inst, 3, VectorSize, OutContext);
   721:     return;
   722: 
   723:   case Hexagon::V6_vS32Ub_npred_ai:
   724:   case Hexagon::V6_vS32Ub_pred_ai:
   725:   case Hexagon::V6_vS32b_new_npred_ai:
   726:   case Hexagon::V6_vS32b_new_pred_ai:
   727:   case Hexagon::V6_vS32b_npred_ai:
   728:   case Hexagon::V6_vS32b_nqpred_ai:
   729:   case Hexagon::V6_vS32b_nt_new_npred_ai:
   730:   case Hexagon::V6_vS32b_nt_new_pred_ai:
   731:   case Hexagon::V6_vS32b_nt_npred_ai:
   732:   case Hexagon::V6_vS32b_nt_nqpred_ai:
   733:   case Hexagon::V6_vS32b_nt_pred_ai:
   734:   case Hexagon::V6_vS32b_nt_qpred_ai:
   735:   case Hexagon::V6_vS32b_pred_ai:
   736:   case Hexagon::V6_vS32b_qpred_ai:
   737:     MappedInst = ScaleVectorOffset(Inst, 2, VectorSize, OutContext);
   738:     return;
   739: 
   740:   // V65+
   741:   case Hexagon::V6_vS32b_srls_ai:
   742:     MappedInst = ScaleVectorOffset(Inst, 1, VectorSize, OutContext);
   743:     return;
   744: 
   745:   case Hexagon::V6_vS32b_srls_pi:
   746:     MappedInst = ScaleVectorOffset(Inst, 2, VectorSize, OutContext);
   747:     return;
   748:   }
   749: }
   750: 
   751: /// Print out a single Hexagon MI to the current output stream.
   752: void HexagonAsmPrinter::emitInstruction(const MachineInstr *MI) {
   753:   Hexagon_MC::verifyInstructionPredicates(MI->getOpcode(),
   754:                                           getSubtargetInfo().getFeatureBits());
   755: 
   756:   MCInst MCB;
   757:   MCB.setOpcode(Hexagon::BUNDLE);
   758:   MCB.addOperand(MCOperand::createImm(0));
   759:   const MCInstrInfo &MCII = *Subtarget->getInstrInfo();
   760: 
   761:   if (MI->isBundle()) {
   762:     const MachineBasicBlock* MBB = MI->getParent();
   763:     MachineBasicBlock::const_instr_iterator MII = MI->getIterator();
   764: 
   765:     for (++MII; MII != MBB->instr_end() && MII->isInsideBundle(); ++MII)
   766:       if (!MII->isDebugInstr() && !MII->isImplicitDef())
   767:         HexagonLowerToMC(MCII, &*MII, MCB, *this);
   768:   } else {
   769:     HexagonLowerToMC(MCII, MI, MCB, *this);
   770:   }
   771: 
   772:   const MachineFunction &MF = *MI->getParent()->getParent();
   773:   const auto &HII = *MF.getSubtarget<HexagonSubtarget>().getInstrInfo();
   774:   if (MI->isBundle() && HII.getBundleNoShuf(*MI))
   775:     HexagonMCInstrInfo::setMemReorderDisabled(MCB);
   776: 
   777:   MCContext &Ctx = OutStreamer->getContext();
   778:   bool Ok = HexagonMCInstrInfo::canonicalizePacket(MCII, *Subtarget, Ctx,
   779:                                                    MCB, nullptr);
   780:   assert(Ok); (void)Ok;
   781:   if (HexagonMCInstrInfo::bundleSize(MCB) == 0)
   782:     return;
   783:   OutStreamer->emitInstruction(MCB, getSubtargetInfo());
   784: }
   785: 
   786: void HexagonAsmPrinter::emitStartOfAsmFile(Module &M) {
   787:   if (TM.getTargetTriple().isOSBinFormatELF())
   788:     emitAttributes();
   789: }
   790: 
   791: void HexagonAsmPrinter::emitEndOfAsmFile(Module &M) {
   792:   HexagonTargetStreamer &HTS =
   793:       static_cast<HexagonTargetStreamer &>(*OutStreamer->getTargetStreamer());
   794:   if (TM.getTargetTriple().isOSBinFormatELF())
   795:     HTS.finishAttributeSection();
   796: }
   797: 
   798: void HexagonAsmPrinter::emitAttributes() {
   799:   HexagonTargetStreamer &HTS =
   800:       static_cast<HexagonTargetStreamer &>(*OutStreamer->getTargetStreamer());
```
- EN: It declares or implements routines such as ScaleVectorOffset, HexagonAsmPrinter::emitInstruction, Hexagon_MC::verifyInstructionPredicates, setOpcode, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonAsmPrinter, Hexagon_MC, HexagonLowerToMC, HexagonSubtarget, ... (6 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 ScaleVectorOffset, HexagonAsmPrinter::emitInstruction, Hexagon_MC::verifyInstructionPredicates, setOpcode, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonAsmPrinter, Hexagon_MC, HexagonLowerToMC, HexagonSubtarget, ... (6 total)，说明了它与同级后端组件的连接关系。

### Lines 801-900 / 第 801-900 行

```cpp
   801:   HTS.emitTargetAttributes(TM.getMCSubtargetInfo());
   802: }
   803: 
   804: void HexagonAsmPrinter::LowerPATCHABLE_EVENT_CALL(const MachineInstr &MI,
   805:                                                   bool Typed) {
   806:   auto &O = *OutStreamer;
   807:   MCSymbol *CurSled = OutContext.createTempSymbol("xray_sled_", true);
   808:   O.emitLabel(CurSled);
   809: 
   810:   auto *Sym = MCSymbolRefExpr::create(
   811:       OutContext.getOrCreateSymbol(Typed ? "__xray_TypedEvent"
   812:                                          : "__xray_CustomEvent"),
   813:       OutContext);
   814: 
   815:   // The sled structure:
   816:   //   .Lxray_sled_N:
   817:   //     { jump .Lend }            -- disabled (patched to nop when enabled)
   818:   //     <save args, move operands, call handler, restore args>
   819:   //   .Lend:
   820: 
   821:   MCSymbol *EndSled = OutContext.createTempSymbol();
   822: 
   823:   // Packet 1: jump over the sled (disabled state).
   824:   MCInst *JumpInst = OutContext.createMCInst();
   825:   JumpInst->setOpcode(Hexagon::J2_jump);
   826:   JumpInst->addOperand(MCOperand::createExpr(HexagonMCExpr::create(
   827:       MCSymbolRefExpr::create(EndSled, OutContext), OutContext)));
   828: 
   829:   MCInst JumpPacket;
   830:   JumpPacket.setOpcode(Hexagon::BUNDLE);
   831:   JumpPacket.addOperand(MCOperand::createImm(0));
   832:   JumpPacket.addOperand(MCOperand::createInst(JumpInst));
   833:   EmitToStreamer(O, JumpPacket);
   834: 
   835:   // Packet 2: allocframe to save LR:FP.
   836:   MCInst *AllocInst = OutContext.createMCInst();
   837:   AllocInst->setOpcode(Hexagon::S2_allocframe);
   838:   AllocInst->addOperand(MCOperand::createReg(Hexagon::R29));
   839:   AllocInst->addOperand(MCOperand::createReg(Hexagon::R30));
   840:   AllocInst->addOperand(MCOperand::createExpr(HexagonMCExpr::create(
   841:       MCConstantExpr::create(0, OutContext), OutContext)));
   842: 
   843:   MCInst AllocPacket;
   844:   AllocPacket.setOpcode(Hexagon::BUNDLE);
   845:   AllocPacket.addOperand(MCOperand::createImm(0));
   846:   AllocPacket.addOperand(MCOperand::createInst(AllocInst));
   847:   EmitToStreamer(O, AllocPacket);
   848: 
   849:   // Save argument registers and set up call arguments.
   850:   // Custom event:  2 operands (ptr, size) in MI operands 0,1 -> r0, r1
   851:   // Typed event:   3 operands (type, ptr, size) in MI operands 0,1,2 ->
   852:   // r0,r1,r2
   853:   unsigned NumArgs = Typed ? 3 : 2;
   854: 
   855:   // Save the original argument registers onto the stack.
   856:   // Packet 3: Allocate space and save r0.
   857:   MCInst *SubSpInst = OutContext.createMCInst();
   858:   SubSpInst->setOpcode(Hexagon::A2_addi);
   859:   SubSpInst->addOperand(MCOperand::createReg(Hexagon::R29));
   860:   SubSpInst->addOperand(MCOperand::createReg(Hexagon::R29));
   861:   SubSpInst->addOperand(MCOperand::createExpr(HexagonMCExpr::create(
   862:       MCConstantExpr::create(-(int64_t)(NumArgs * 4), OutContext),
   863:       OutContext)));
   864: 
   865:   MCInst SubSpPacket;
   866:   SubSpPacket.setOpcode(Hexagon::BUNDLE);
   867:   SubSpPacket.addOperand(MCOperand::createImm(0));
   868:   SubSpPacket.addOperand(MCOperand::createInst(SubSpInst));
   869:   EmitToStreamer(O, SubSpPacket);
   870: 
   871:   // Save each argument register.
   872:   for (unsigned I = 0; I < NumArgs; ++I) {
   873:     MCInst *StoreInst = OutContext.createMCInst();
   874:     StoreInst->setOpcode(Hexagon::S2_storeri_io);
   875:     StoreInst->addOperand(MCOperand::createReg(Hexagon::R29));
   876:     StoreInst->addOperand(MCOperand::createExpr(HexagonMCExpr::create(
   877:         MCConstantExpr::create(I * 4, OutContext), OutContext)));
   878:     StoreInst->addOperand(MCOperand::createReg(Hexagon::R0 + I));
   879: 
   880:     MCInst StorePacket;
   881:     StorePacket.setOpcode(Hexagon::BUNDLE);
   882:     StorePacket.addOperand(MCOperand::createImm(0));
   883:     StorePacket.addOperand(MCOperand::createInst(StoreInst));
   884:     EmitToStreamer(O, StorePacket);
   885:   }
   886: 
   887:   // Move operands into argument registers (r0, r1, [r2]).
   888:   // The XRay intrinsic uses i64 for size (and type) parameters. On 32-bit
   889:   // Hexagon these are in DoubleRegs (register pairs). The runtime handler
   890:   // expects 32-bit arguments, so extract the low sub-register.
   891:   //
   892:   // NOTE: Moves are always emitted (even identity moves like r0 = r0) so that
   893:   // the sled has a fixed size. The runtime patching code relies on the sled
   894:   // being a known number of words to encode the correct jump offset for the
   895:   // disabled state.
   896:   //
   897:   // NOTE: When source registers alias destination registers in a conflicting
   898:   // order (e.g., src0 in r1 and src1 in r0), the sequential moves can produce
   899:   // incorrect results. This is the same limitation as AArch64's implementation
   900:   // and is unlikely in practice since the register allocator rarely produces
```
- EN: It declares or implements routines such as emitTargetAttributes, HexagonAsmPrinter::LowerPATCHABLE_EVENT_CALL, createTempSymbol, emitLabel, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonAsmPrinter, HexagonMCExpr, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 emitTargetAttributes, HexagonAsmPrinter::LowerPATCHABLE_EVENT_CALL, createTempSymbol, emitLabel, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonAsmPrinter, HexagonMCExpr，说明了它与同级后端组件的连接关系。

### Lines 901-1000 / 第 901-1000 行

```cpp
   901:   // such assignments for XRay event intrinsics.
   902:   const auto &HRI = *MF->getSubtarget<HexagonSubtarget>().getRegisterInfo();
   903:   for (unsigned I = 0; I < NumArgs; ++I) {
   904:     Register SrcReg = MI.getOperand(I).getReg();
   905:     if (Hexagon::DoubleRegsRegClass.contains(SrcReg))
   906:       SrcReg = HRI.getSubReg(SrcReg, Hexagon::isub_lo);
   907: 
   908:     MCInst *MovInst = OutContext.createMCInst();
   909:     MovInst->setOpcode(Hexagon::A2_tfr);
   910:     MovInst->addOperand(MCOperand::createReg(Hexagon::R0 + I));
   911:     MovInst->addOperand(MCOperand::createReg(SrcReg));
   912: 
   913:     MCInst MovPacket;
   914:     MovPacket.setOpcode(Hexagon::BUNDLE);
   915:     MovPacket.addOperand(MCOperand::createImm(0));
   916:     MovPacket.addOperand(MCOperand::createInst(MovInst));
   917:     EmitToStreamer(O, MovPacket);
   918:   }
   919: 
   920:   // Call the handler.
   921:   MCInst *CallInst = OutContext.createMCInst();
   922:   CallInst->setOpcode(Hexagon::J2_call);
   923:   CallInst->addOperand(
   924:       MCOperand::createExpr(HexagonMCExpr::create(Sym, OutContext)));
   925: 
   926:   MCInst CallPacket;
   927:   CallPacket.setOpcode(Hexagon::BUNDLE);
   928:   CallPacket.addOperand(MCOperand::createImm(0));
   929:   CallPacket.addOperand(MCOperand::createInst(CallInst));
   930:   EmitToStreamer(O, CallPacket);
   931: 
   932:   // Restore argument registers.
   933:   for (unsigned I = 0; I < NumArgs; ++I) {
   934:     MCInst *LoadInst = OutContext.createMCInst();
   935:     LoadInst->setOpcode(Hexagon::L2_loadri_io);
   936:     LoadInst->addOperand(MCOperand::createReg(Hexagon::R0 + I));
   937:     LoadInst->addOperand(MCOperand::createReg(Hexagon::R29));
   938:     LoadInst->addOperand(MCOperand::createExpr(HexagonMCExpr::create(
   939:         MCConstantExpr::create(I * 4, OutContext), OutContext)));
   940: 
   941:     MCInst LoadPacket;
   942:     LoadPacket.setOpcode(Hexagon::BUNDLE);
   943:     LoadPacket.addOperand(MCOperand::createImm(0));
   944:     LoadPacket.addOperand(MCOperand::createInst(LoadInst));
   945:     EmitToStreamer(O, LoadPacket);
   946:   }
   947: 
   948:   // Deallocate saved argument space.
   949:   MCInst *AddSpInst = OutContext.createMCInst();
   950:   AddSpInst->setOpcode(Hexagon::A2_addi);
   951:   AddSpInst->addOperand(MCOperand::createReg(Hexagon::R29));
   952:   AddSpInst->addOperand(MCOperand::createReg(Hexagon::R29));
   953:   AddSpInst->addOperand(MCOperand::createExpr(HexagonMCExpr::create(
   954:       MCConstantExpr::create(NumArgs * 4, OutContext), OutContext)));
   955: 
   956:   MCInst AddSpPacket;
   957:   AddSpPacket.setOpcode(Hexagon::BUNDLE);
   958:   AddSpPacket.addOperand(MCOperand::createImm(0));
   959:   AddSpPacket.addOperand(MCOperand::createInst(AddSpInst));
   960:   EmitToStreamer(O, AddSpPacket);
   961: 
   962:   // Deallocframe to restore LR:FP.
   963:   MCInst *DeallocInst = OutContext.createMCInst();
   964:   DeallocInst->setOpcode(Hexagon::L2_deallocframe);
   965:   DeallocInst->addOperand(MCOperand::createReg(Hexagon::D15));
   966:   DeallocInst->addOperand(MCOperand::createReg(Hexagon::R30));
   967: 
   968:   MCInst DeallocPacket;
   969:   DeallocPacket.setOpcode(Hexagon::BUNDLE);
   970:   DeallocPacket.addOperand(MCOperand::createImm(0));
   971:   DeallocPacket.addOperand(MCOperand::createInst(DeallocInst));
   972:   EmitToStreamer(O, DeallocPacket);
   973: 
   974:   OutStreamer->emitLabel(EndSled);
   975:   recordSled(CurSled, MI,
   976:              Typed ? SledKind::TYPED_EVENT : SledKind::CUSTOM_EVENT, 2);
   977: }
   978: 
   979: void HexagonAsmPrinter::EmitSled(const MachineInstr &MI, SledKind Kind) {
   980:   static const int8_t NoopsInSledCount = 6;
   981:   // We want to emit the following pattern:
   982:   //
   983:   // .L_xray_sled_N:
   984:   // <xray_sled_base>:
   985:   // { jump .Ltmp0 }
   986:   // { nop }
   987:   // { nop }
   988:   // { nop }
   989:   // { nop }
   990:   // { nop }
   991:   // { nop }
   992:   // .Ltmp0:
   993:   //
   994:   // We need the 6 nop words because at runtime, we'd be patching over the
   995:   // full 7 words with the following pattern:
   996:   //
   997:   // <xray_sled_n>:
   998:   // { allocframe(#0) }
   999:   // { immext(#...) // upper 26-bits of func id
  1000:   //   r7 = ##...   // lower  6-bits of func id
```
- EN: It declares or implements routines such as getSubtarget<HexagonSubtarget>, getOperand, createMCInst, setOpcode, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSubtarget, HexagonMCExpr, HexagonAsmPrinter, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getSubtarget<HexagonSubtarget>, getOperand, createMCInst, setOpcode, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget, HexagonMCExpr, HexagonAsmPrinter，说明了它与同级后端组件的连接关系。

### Lines 1001-1055 / 第 1001-1055 行

```cpp
  1001:   //   immext(#...) // upper 26-bits of trampoline
  1002:   //   r6 = ##... } // lower  6-bits of trampoline
  1003:   // { callr r6 }
  1004:   // { deallocframe }
  1005:   //
  1006:   // allocframe saves r31:30 (LR:FP) before the call, and deallocframe
  1007:   // restores them after the trampoline returns, ensuring the caller's
  1008:   // return address in r31 is preserved across the sled.
  1009:   //
  1010:   auto CurSled = OutContext.createTempSymbol("xray_sled_", true);
  1011:   OutStreamer->emitLabel(CurSled);
  1012: 
  1013:   MCInst *SledJump = new (OutContext) MCInst();
  1014:   SledJump->setOpcode(Hexagon::J2_jump);
  1015:   auto PostSled = OutContext.createTempSymbol();
  1016:   SledJump->addOperand(MCOperand::createExpr(HexagonMCExpr::create(
  1017:       MCSymbolRefExpr::create(PostSled, OutContext), OutContext)));
  1018: 
  1019:   // Emit "jump PostSled" instruction, which jumps over the nop series.
  1020:   MCInst SledJumpPacket;
  1021:   SledJumpPacket.setOpcode(Hexagon::BUNDLE);
  1022:   SledJumpPacket.addOperand(MCOperand::createImm(0));
  1023:   SledJumpPacket.addOperand(MCOperand::createInst(SledJump));
  1024: 
  1025:   EmitToStreamer(*OutStreamer, SledJumpPacket);
  1026: 
  1027:   // FIXME: this will emit individual packets, we should
  1028:   // special-case this and combine them into a single packet.
  1029:   emitNops(NoopsInSledCount);
  1030: 
  1031:   OutStreamer->emitLabel(PostSled);
  1032:   recordSled(CurSled, MI, Kind, 2);
  1033: }
  1034: 
  1035: void HexagonAsmPrinter::LowerPATCHABLE_FUNCTION_ENTER(const MachineInstr &MI) {
  1036:   EmitSled(MI, SledKind::FUNCTION_ENTER);
  1037: }
  1038: 
  1039: void HexagonAsmPrinter::LowerPATCHABLE_FUNCTION_EXIT(const MachineInstr &MI) {
  1040:   EmitSled(MI, SledKind::FUNCTION_EXIT);
  1041: }
  1042: 
  1043: void HexagonAsmPrinter::LowerPATCHABLE_TAIL_CALL(const MachineInstr &MI) {
  1044:   EmitSled(MI, SledKind::TAIL_CALL);
  1045: }
  1046: 
  1047: char HexagonAsmPrinter::ID = 0;
  1048: 
  1049: INITIALIZE_PASS(HexagonAsmPrinter, "hexagon-asm-printer",
  1050:                 "Hexagon Assembly Printer", false, false)
  1051: 
  1052: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
  1053: LLVMInitializeHexagonAsmPrinter() {
  1054:   RegisterAsmPrinter<HexagonAsmPrinter> X(getTheHexagonTarget());
  1055: }
```
- EN: It declares or implements routines such as r31:30, emitLabel, new, setOpcode, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCExpr, HexagonAsmPrinter, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 r31:30, emitLabel, new, setOpcode, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCExpr, HexagonAsmPrinter，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- MC-layer target description / MC 层目标描述
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonAsmPrinter.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, MCTargetDesc/HexagonInstPrinter.h, MCTargetDesc/HexagonMCExpr.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, MCTargetDesc/HexagonTargetStreamer.h, TargetInfo/HexagonTargetInfo.h, ... (38 total)`
- Hexagon symbols / Hexagon 符号: `HexagonAsmPrinter, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget, HexagonInstPrinter, HexagonMCExpr, HexagonMCInstrInfo, HexagonMCTargetDesc, HexagonTargetStreamer, HexagonTargetInfo, ... (13 total)`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
