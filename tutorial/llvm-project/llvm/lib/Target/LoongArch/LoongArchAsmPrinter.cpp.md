# LoongArchAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchAsmPrinter.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file emits textual assembly from LLVM machine instructions for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责将 LLVM 机器指令输出为文本汇编。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===- LoongArchAsmPrinter.cpp - LoongArch LLVM Assembly Printer -*- C++ -*--=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains a printer that converts from our internal representation
  10: // of machine-dependent LLVM code to GAS-format LoongArch assembly language.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "LoongArchAsmPrinter.h"
  15: #include "LoongArch.h"
  16: #include "LoongArchMachineFunctionInfo.h"
  17: #include "MCTargetDesc/LoongArchInstPrinter.h"
  18: #include "MCTargetDesc/LoongArchMCTargetDesc.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchAsmPrinter.h`, `LoongArch.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchInstPrinter.h`, `LoongArchMCTargetDesc.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchAsmPrinter.h`, `LoongArch.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchInstPrinter.h`, `LoongArchMCTargetDesc.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "TargetInfo/LoongArchTargetInfo.h"
  20: #include "llvm/CodeGen/AsmPrinter.h"
  21: #include "llvm/CodeGen/MachineJumpTableInfo.h"
  22: #include "llvm/CodeGen/MachineModuleInfoImpls.h"
  23: #include "llvm/MC/MCAsmInfo.h"
  24: #include "llvm/MC/MCContext.h"
  25: #include "llvm/MC/MCInstBuilder.h"
  26: #include "llvm/MC/MCSectionELF.h"
  27: #include "llvm/MC/TargetRegistry.h"
  28: #include "llvm/Support/Compiler.h"
  29: 
  30: using namespace llvm;
  31: 
  32: #define DEBUG_TYPE "loongarch-asm-printer"
  33: 
  34: cl::opt<bool> LArchAnnotateTableJump(
  35:     "loongarch-annotate-tablejump", cl::Hidden,
  36:     cl::desc(
```
- **EN**: It imports dependencies such as `LoongArchTargetInfo.h`, `AsmPrinter.h`, `MachineJumpTableInfo.h`, `MachineModuleInfoImpls.h`, `MCAsmInfo.h`, `MCContext.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `LoongArchTargetInfo.h`, `AsmPrinter.h`, `MachineJumpTableInfo.h`, `MachineModuleInfoImpls.h`, `MCAsmInfo.h`, `MCContext.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37:         "Annotate table jump instruction to correlate it with the jump table."),
  38:     cl::init(false));
  39: 
  40: // Simple pseudo-instructions have their lowering (with expansion to real
  41: // instructions) auto-generated.
  42: #include "LoongArchGenMCPseudoLowering.inc"
  43: 
  44: void LoongArchAsmPrinter::emitInstruction(const MachineInstr *MI) {
  45:   LoongArch_MC::verifyInstructionPredicates(
  46:       MI->getOpcode(), getSubtargetInfo().getFeatureBits());
  47: 
  48:   // Do any auto-generated pseudo lowerings.
  49:   if (MCInst OutInst; lowerPseudoInstExpansion(MI, OutInst)) {
  50:     EmitToStreamer(*OutStreamer, OutInst);
  51:     return;
  52:   }
  53: 
  54:   switch (MI->getOpcode()) {
```
- **EN**: It imports dependencies such as `LoongArchGenMCPseudoLowering.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `LoongArchAsmPrinter::emitInstruction`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 它引入了 `LoongArchGenMCPseudoLowering.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `LoongArchAsmPrinter::emitInstruction` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   case TargetOpcode::STATEPOINT:
  56:     LowerSTATEPOINT(*MI);
  57:     return;
  58:   case TargetOpcode::PATCHABLE_FUNCTION_ENTER:
  59:     LowerPATCHABLE_FUNCTION_ENTER(*MI);
  60:     return;
  61:   case TargetOpcode::PATCHABLE_FUNCTION_EXIT:
  62:     LowerPATCHABLE_FUNCTION_EXIT(*MI);
  63:     return;
  64:   case TargetOpcode::PATCHABLE_TAIL_CALL:
  65:     LowerPATCHABLE_TAIL_CALL(*MI);
  66:     return;
  67:   }
  68: 
  69:   MCInst TmpInst;
  70:   if (!lowerLoongArchMachineInstrToMCInst(MI, TmpInst, *this))
  71:     EmitToStreamer(*OutStreamer, TmpInst);
  72: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 73-90 / 第 73-90 行
```cpp
  73: 
  74: bool LoongArchAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
  75:                                           const char *ExtraCode,
  76:                                           raw_ostream &OS) {
  77:   // First try the generic code, which knows about modifiers like 'c' and 'n'.
  78:   if (!AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, OS))
  79:     return false;
  80: 
  81:   const MachineOperand &MO = MI->getOperand(OpNo);
  82:   if (ExtraCode && ExtraCode[0]) {
  83:     if (ExtraCode[1] != 0)
  84:       return true; // Unknown modifier.
  85: 
  86:     switch (ExtraCode[0]) {
  87:     default:
  88:       return true; // Unknown modifier.
  89:     case 'z':      // Print $zero register if zero, regular printing otherwise.
  90:       if (MO.isImm() && MO.getImm() == 0) {
```
- **EN**: The range implements or declares functions including `LoongArchAsmPrinter::PrintAsmOperand`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmPrinter::PrintAsmOperand` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91:         OS << '$' << LoongArchInstPrinter::getRegisterName(LoongArch::R0);
  92:         return false;
  93:       }
  94:       break;
  95:     case 'u': // Print LASX registers.
  96:     case 'w': // Print LSX registers.
  97:     {
  98:       // If the operand is an LASX, LSX or floating point register, print the
  99:       // name of LASX or LSX register with the same index in that register
 100:       // class.
 101:       unsigned RegID = MO.getReg().id(), FirstReg;
 102:       if (RegID >= LoongArch::XR0 && RegID <= LoongArch::XR31)
 103:         FirstReg = LoongArch::XR0;
 104:       else if (RegID >= LoongArch::VR0 && RegID <= LoongArch::VR31)
 105:         FirstReg = LoongArch::VR0;
 106:       else if (RegID >= LoongArch::F0_64 && RegID <= LoongArch::F31_64)
 107:         FirstReg = LoongArch::F0_64;
 108:       else if (RegID >= LoongArch::F0 && RegID <= LoongArch::F31)
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:         FirstReg = LoongArch::F0;
 110:       else
 111:         return true;
 112:       OS << '$'
 113:          << LoongArchInstPrinter::getRegisterName(
 114:                 RegID - FirstReg +
 115:                 (ExtraCode[0] == 'u' ? LoongArch::XR0 : LoongArch::VR0));
 116:       return false;
 117:     }
 118:       // TODO: handle other extra codes if any.
 119:     }
 120:   }
 121: 
 122:   switch (MO.getType()) {
 123:   case MachineOperand::MO_Immediate:
 124:     OS << MO.getImm();
 125:     return false;
 126:   case MachineOperand::MO_Register:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:     OS << '$' << LoongArchInstPrinter::getRegisterName(MO.getReg());
 128:     return false;
 129:   case MachineOperand::MO_GlobalAddress:
 130:     PrintSymbolOperand(MO, OS);
 131:     return false;
 132:   default:
 133:     llvm_unreachable("not implemented");
 134:   }
 135: 
 136:   return true;
 137: }
 138: 
 139: bool LoongArchAsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI,
 140:                                                 unsigned OpNo,
 141:                                                 const char *ExtraCode,
 142:                                                 raw_ostream &OS) {
 143:   // TODO: handle extra code.
 144:   if (ExtraCode)
```
- **EN**: The range implements or declares functions including `LoongArchAsmPrinter::PrintAsmMemoryOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchAsmPrinter::PrintAsmMemoryOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 145-162 / 第 145-162 行
```cpp
 145:     return true;
 146: 
 147:   // We only support memory operands like "Base + Offset", where base must be a
 148:   // register, and offset can be a register or an immediate value.
 149:   const MachineOperand &BaseMO = MI->getOperand(OpNo);
 150:   // Base address must be a register.
 151:   if (!BaseMO.isReg())
 152:     return true;
 153:   // Print the base address register.
 154:   OS << "$" << LoongArchInstPrinter::getRegisterName(BaseMO.getReg());
 155:   // Print the offset operand.
 156:   const MachineOperand &OffsetMO = MI->getOperand(OpNo + 1);
 157:   MCOperand MCO;
 158:   if (!lowerOperand(OffsetMO, MCO))
 159:     return true;
 160:   if (OffsetMO.isReg())
 161:     OS << ", $" << LoongArchInstPrinter::getRegisterName(OffsetMO.getReg());
 162:   else if (OffsetMO.isImm())
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```cpp
 163:     OS << ", " << OffsetMO.getImm();
 164:   else if (OffsetMO.isGlobal() || OffsetMO.isBlockAddress() ||
 165:            OffsetMO.isMCSymbol() || OffsetMO.isCPI()) {
 166:     OS << ", ";
 167:     MAI.printExpr(OS, *MCO.getExpr());
 168:   } else
 169:     return true;
 170: 
 171:   return false;
 172: }
 173: 
 174: void LoongArchAsmPrinter::LowerSTATEPOINT(const MachineInstr &MI) {
 175:   StatepointOpers SOpers(&MI);
 176:   if (unsigned PatchBytes = SOpers.getNumPatchBytes()) {
 177:     assert(PatchBytes % 4 == 0 && "Invalid number of NOP bytes requested!");
 178:     emitNops(PatchBytes / 4);
 179:   } else {
 180:     // Lower call target and choose correct opcode.
```
- **EN**: The range implements or declares functions including `LoongArchAsmPrinter::LowerSTATEPOINT`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchAsmPrinter::LowerSTATEPOINT` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 181-198 / 第 181-198 行
```cpp
 181:     const MachineOperand &CallTarget = SOpers.getCallTarget();
 182:     MCOperand CallTargetMCOp;
 183:     switch (CallTarget.getType()) {
 184:     case MachineOperand::MO_GlobalAddress:
 185:     case MachineOperand::MO_ExternalSymbol:
 186:       lowerOperand(CallTarget, CallTargetMCOp);
 187:       EmitToStreamer(*OutStreamer,
 188:                      MCInstBuilder(LoongArch::BL).addOperand(CallTargetMCOp));
 189:       break;
 190:     case MachineOperand::MO_Immediate:
 191:       CallTargetMCOp = MCOperand::createImm(CallTarget.getImm());
 192:       EmitToStreamer(*OutStreamer,
 193:                      MCInstBuilder(LoongArch::BL).addOperand(CallTargetMCOp));
 194:       break;
 195:     case MachineOperand::MO_Register:
 196:       CallTargetMCOp = MCOperand::createReg(CallTarget.getReg());
 197:       EmitToStreamer(*OutStreamer, MCInstBuilder(LoongArch::JIRL)
 198:                                        .addReg(LoongArch::R1)
```
- **EN**: The range implements or declares functions including `EmitToStreamer`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `EmitToStreamer` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 199-216 / 第 199-216 行
```cpp
 199:                                        .addOperand(CallTargetMCOp)
 200:                                        .addImm(0));
 201:       break;
 202:     default:
 203:       llvm_unreachable("Unsupported operand type in statepoint call target");
 204:       break;
 205:     }
 206:   }
 207: 
 208:   auto &Ctx = OutStreamer->getContext();
 209:   MCSymbol *MILabel = Ctx.createTempSymbol();
 210:   OutStreamer->emitLabel(MILabel);
 211:   SM.recordStatepoint(*MILabel, MI);
 212: }
 213: 
 214: void LoongArchAsmPrinter::LowerPATCHABLE_FUNCTION_ENTER(
 215:     const MachineInstr &MI) {
 216:   const Function &F = MF->getFunction();
```
- **EN**: The range implements or declares functions including `LoongArchAsmPrinter::LowerPATCHABLE_FUNCTION_ENTER`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchAsmPrinter::LowerPATCHABLE_FUNCTION_ENTER` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 217-234 / 第 217-234 行
```cpp
 217:   if (F.hasFnAttribute("patchable-function-entry")) {
 218:     unsigned Num = F.getFnAttributeAsParsedInteger("patchable-function-entry");
 219:     emitNops(Num);
 220:     return;
 221:   }
 222: 
 223:   emitSled(MI, SledKind::FUNCTION_ENTER);
 224: }
 225: 
 226: void LoongArchAsmPrinter::LowerPATCHABLE_FUNCTION_EXIT(const MachineInstr &MI) {
 227:   emitSled(MI, SledKind::FUNCTION_EXIT);
 228: }
 229: 
 230: void LoongArchAsmPrinter::LowerPATCHABLE_TAIL_CALL(const MachineInstr &MI) {
 231:   emitSled(MI, SledKind::TAIL_CALL);
 232: }
 233: 
 234: void LoongArchAsmPrinter::emitSled(const MachineInstr &MI, SledKind Kind) {
```
- **EN**: The range implements or declares functions including `LoongArchAsmPrinter::LowerPATCHABLE_FUNCTION_EXIT`, `LoongArchAsmPrinter::LowerPATCHABLE_TAIL_CALL`, `LoongArchAsmPrinter::emitSled`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmPrinter::LowerPATCHABLE_FUNCTION_EXIT`, `LoongArchAsmPrinter::LowerPATCHABLE_TAIL_CALL`, `LoongArchAsmPrinter::emitSled` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235:   // For loongarch64 we want to emit the following pattern:
 236:   //
 237:   // .Lxray_sled_beginN:
 238:   //   B .Lxray_sled_endN
 239:   //   11 NOPs (44 bytes)
 240:   // .Lxray_sled_endN:
 241:   //
 242:   // We need the extra bytes because at runtime they may be used for the
 243:   // actual pattern defined at compiler-rt/lib/xray/xray_loongarch64.cpp.
 244:   // The count here should be adjusted accordingly if the implementation
 245:   // changes.
 246:   const int8_t NoopsInSledCount = 11;
 247:   OutStreamer->emitCodeAlignment(Align(4), &getSubtargetInfo());
 248:   MCSymbol *BeginOfSled = OutContext.createTempSymbol("xray_sled_begin");
 249:   MCSymbol *EndOfSled = OutContext.createTempSymbol("xray_sled_end");
 250:   OutStreamer->emitLabel(BeginOfSled);
 251:   EmitToStreamer(*OutStreamer,
 252:                  MCInstBuilder(LoongArch::B)
```
- **EN**: The range implements or declares functions including `EmitToStreamer`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `EmitToStreamer` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-270 / 第 253-270 行
```cpp
 253:                      .addExpr(MCSymbolRefExpr::create(EndOfSled, OutContext)));
 254:   emitNops(NoopsInSledCount);
 255:   OutStreamer->emitLabel(EndOfSled);
 256:   recordSled(BeginOfSled, MI, Kind, 2);
 257: }
 258: 
 259: void LoongArchAsmPrinter::emitJumpTableInfo() {
 260:   AsmPrinter::emitJumpTableInfo();
 261: 
 262:   if (!LArchAnnotateTableJump)
 263:     return;
 264: 
 265:   assert(TM.getTargetTriple().isOSBinFormatELF());
 266: 
 267:   auto *LAFI = MF->getInfo<LoongArchMachineFunctionInfo>();
 268:   unsigned EntrySize = LAFI->getJumpInfoSize();
 269:   auto JTI = MF->getJumpTableInfo();
 270: 
```
- **EN**: The range implements or declares functions including `LoongArchAsmPrinter::emitJumpTableInfo`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchAsmPrinter::emitJumpTableInfo` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 271-288 / 第 271-288 行
```cpp
 271:   if (!JTI || 0 == EntrySize)
 272:     return;
 273: 
 274:   unsigned Size = getDataLayout().getPointerSize();
 275:   const auto &JT = JTI->getJumpTables();
 276: 
 277:   // Emit an additional section to store the correlation info as pairs of
 278:   // addresses, each pair contains the address of a jump instruction (jr) and
 279:   // the address of the jump table.
 280:   OutStreamer->switchSection(MMI->getContext().getELFSection(
 281:       ".discard.tablejump_annotate", ELF::SHT_PROGBITS, 0));
 282: 
 283:   for (unsigned Idx = 0; Idx < EntrySize; ++Idx) {
 284:     int JTIIdx = LAFI->getJumpInfoJTIIndex(Idx);
 285:     if (JT[JTIIdx].MBBs.empty())
 286:       continue;
 287:     OutStreamer->emitValue(
 288:         MCSymbolRefExpr::create(LAFI->getJumpInfoJrMI(Idx)->getPreInstrSymbol(),
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 289-306 / 第 289-306 行
```cpp
 289:                                 OutContext),
 290:         Size);
 291:     OutStreamer->emitValue(
 292:         MCSymbolRefExpr::create(GetJTISymbol(JTIIdx), OutContext), Size);
 293:   }
 294: }
 295: 
 296: bool LoongArchAsmPrinter::runOnMachineFunction(MachineFunction &MF) {
 297:   AsmPrinter::runOnMachineFunction(MF);
 298:   // Emit the XRay table for this function.
 299:   emitXRayTable();
 300:   return true;
 301: }
 302: 
 303: char LoongArchAsmPrinter::ID = 0;
 304: 
 305: INITIALIZE_PASS(LoongArchAsmPrinter, "loongarch-asm-printer",
 306:                 "LoongArch Assembly Printer", false, false)
```
- **EN**: The range implements or declares functions including `LoongArchAsmPrinter::runOnMachineFunction`.
- **CN**: 这一段实现或声明了 `LoongArchAsmPrinter::runOnMachineFunction` 等函数。

### Lines 307-313 / 第 307-313 行
```cpp
 307: 
 308: // Force static initialization.
 309: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
 310: LLVMInitializeLoongArchAsmPrinter() {
 311:   RegisterAsmPrinter<LoongArchAsmPrinter> X(getTheLoongArch32Target());
 312:   RegisterAsmPrinter<LoongArchAsmPrinter> Y(getTheLoongArch64Target());
 313: }
```
- **EN**: This span continues the file's main responsibility: this file emits textual assembly from LLVM machine instructions for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Assembly emission**: Prints target instructions and directives in textual assembly form. / 以文本汇编形式输出目标指令和伪指令。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `LoongArchAsmPrinter.h`
- `LoongArch.h`
- `LoongArchMachineFunctionInfo.h`
- `MCTargetDesc/LoongArchInstPrinter.h`
- `MCTargetDesc/LoongArchMCTargetDesc.h`
- `TargetInfo/LoongArchTargetInfo.h`
- `llvm/CodeGen/AsmPrinter.h`
- `llvm/CodeGen/MachineJumpTableInfo.h`
- `llvm/CodeGen/MachineModuleInfoImpls.h`
- `llvm/MC/MCAsmInfo.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCInstBuilder.h`
- `llvm/MC/MCSectionELF.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/Compiler.h`
- `LoongArchGenMCPseudoLowering.inc`
