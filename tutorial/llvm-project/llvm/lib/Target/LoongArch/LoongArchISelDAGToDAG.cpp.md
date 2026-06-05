# LoongArchISelDAGToDAG.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchISelDAGToDAG.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file selects target instructions from SelectionDAG nodes for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责从 SelectionDAG 节点选择目标指令。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //=- LoongArchISelDAGToDAG.cpp - A dag to dag inst selector for LoongArch -===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines an instruction selector for the LoongArch target.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LoongArchISelDAGToDAG.h"
  14: #include "LoongArchISelLowering.h"
  15: #include "MCTargetDesc/LoongArchMCTargetDesc.h"
  16: #include "MCTargetDesc/LoongArchMatInt.h"
  17: #include "llvm/Support/KnownBits.h"
  18: #include "llvm/Support/raw_ostream.h"
  19: 
  20: using namespace llvm;
  21: 
  22: #define DEBUG_TYPE "loongarch-isel"
  23: #define PASS_NAME "LoongArch DAG->DAG Pattern Instruction Selection"
  24: 
  25: char LoongArchDAGToDAGISelLegacy::ID;
  26: 
  27: LoongArchDAGToDAGISelLegacy::LoongArchDAGToDAGISelLegacy(
  28:     LoongArchTargetMachine &TM, CodeGenOptLevel OptLevel)
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchISelDAGToDAG.h`, `LoongArchISelLowering.h`, `LoongArchMCTargetDesc.h`, `LoongArchMatInt.h`, `KnownBits.h`, `raw_ostream.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchISelDAGToDAG.h`, `LoongArchISelLowering.h`, `LoongArchMCTargetDesc.h`, `LoongArchMatInt.h`, `KnownBits.h`, `raw_ostream.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 29-56 / 第 29-56 行
```cpp
  29:     : SelectionDAGISelLegacy(
  30:           ID, std::make_unique<LoongArchDAGToDAGISel>(TM, OptLevel)) {}
  31: 
  32: INITIALIZE_PASS(LoongArchDAGToDAGISelLegacy, DEBUG_TYPE, PASS_NAME, false,
  33:                 false)
  34: 
  35: void LoongArchDAGToDAGISel::Select(SDNode *Node) {
  36:   // If we have a custom node, we have already selected.
  37:   if (Node->isMachineOpcode()) {
  38:     LLVM_DEBUG(dbgs() << "== "; Node->dump(CurDAG); dbgs() << "\n");
  39:     Node->setNodeId(-1);
  40:     return;
  41:   }
  42: 
  43:   // Instruction Selection not handled by the auto-generated tablegen selection
  44:   // should be handled here.
  45:   unsigned Opcode = Node->getOpcode();
  46:   MVT GRLenVT = Subtarget->getGRLenVT();
  47:   SDLoc DL(Node);
  48:   MVT VT = Node->getSimpleValueType(0);
  49: 
  50:   switch (Opcode) {
  51:   default:
  52:     break;
  53:   case ISD::Constant: {
  54:     int64_t Imm = cast<ConstantSDNode>(Node)->getSExtValue();
  55:     if (Imm == 0 && VT == GRLenVT) {
  56:       SDValue New = CurDAG->getCopyFromReg(CurDAG->getEntryNode(), DL,
```
- **EN**: The range implements or declares functions including `SelectionDAGISelLegacy`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SelectionDAGISelLegacy` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 57-84 / 第 57-84 行
```cpp
  57:                                            LoongArch::R0, GRLenVT);
  58:       ReplaceNode(Node, New.getNode());
  59:       return;
  60:     }
  61:     SDNode *Result = nullptr;
  62:     SDValue SrcReg = CurDAG->getRegister(LoongArch::R0, GRLenVT);
  63:     // The instructions in the sequence are handled here.
  64:     for (LoongArchMatInt::Inst &Inst : LoongArchMatInt::generateInstSeq(Imm)) {
  65:       SDValue SDImm = CurDAG->getSignedTargetConstant(Inst.Imm, DL, GRLenVT);
  66:       switch (Inst.Opc) {
  67:       case LoongArch::LU12I_W:
  68:         Result = CurDAG->getMachineNode(Inst.Opc, DL, GRLenVT, SDImm);
  69:         break;
  70:       case LoongArch::ADDI_W:
  71:       case LoongArch::ORI:
  72:       case LoongArch::LU32I_D:
  73:       case LoongArch::LU52I_D:
  74:         Result = CurDAG->getMachineNode(Inst.Opc, DL, GRLenVT, SrcReg, SDImm);
  75:         break;
  76:       case LoongArch::BSTRINS_D:
  77:         Result = CurDAG->getMachineNode(
  78:             Inst.Opc, DL, GRLenVT,
  79:             {SrcReg, SrcReg,
  80:              CurDAG->getSignedTargetConstant(Inst.Imm >> 32, DL, GRLenVT),
  81:              CurDAG->getTargetConstant(Inst.Imm & 0xFF, DL, GRLenVT)});
  82:         break;
  83:       default:
  84:         llvm_unreachable("unexpected opcode generated by LoongArchMatInt");
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 85-112 / 第 85-112 行
```cpp
  85:       }
  86:       SrcReg = SDValue(Result, 0);
  87:     }
  88: 
  89:     ReplaceNode(Node, Result);
  90:     return;
  91:   }
  92:   case ISD::FrameIndex: {
  93:     SDValue Imm = CurDAG->getTargetConstant(0, DL, GRLenVT);
  94:     int FI = cast<FrameIndexSDNode>(Node)->getIndex();
  95:     SDValue TFI = CurDAG->getTargetFrameIndex(FI, VT);
  96:     unsigned ADDIOp =
  97:         Subtarget->is64Bit() ? LoongArch::ADDI_D : LoongArch::ADDI_W;
  98:     ReplaceNode(Node, CurDAG->getMachineNode(ADDIOp, DL, VT, TFI, Imm));
  99:     return;
 100:   }
 101:   case ISD::BITCAST: {
 102:     if (VT.is128BitVector() || VT.is256BitVector()) {
 103:       ReplaceUses(SDValue(Node, 0), Node->getOperand(0));
 104:       CurDAG->RemoveDeadNode(Node);
 105:       return;
 106:     }
 107:     break;
 108:   }
 109:   case ISD::BUILD_VECTOR: {
 110:     // Select appropriate [x]vrepli.[bhwd] instructions for constant splats of
 111:     // 128/256-bit when LSX/LASX is enabled.
 112:     BuildVectorSDNode *BVN = cast<BuildVectorSDNode>(Node);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 113-140 / 第 113-140 行
```cpp
 113:     APInt SplatValue, SplatUndef;
 114:     unsigned SplatBitSize;
 115:     bool HasAnyUndefs;
 116:     unsigned Op = 0;
 117:     EVT ResTy = BVN->getValueType(0);
 118:     bool Is128Vec = BVN->getValueType(0).is128BitVector();
 119:     bool Is256Vec = BVN->getValueType(0).is256BitVector();
 120:     SDNode *Res;
 121: 
 122:     if (!Subtarget->hasExtLSX() || (!Is128Vec && !Is256Vec))
 123:       break;
 124:     if (!BVN->isConstantSplat(SplatValue, SplatUndef, SplatBitSize,
 125:                               HasAnyUndefs, 8))
 126:       break;
 127: 
 128:     // If we have a signed 10 bit integer, we can splat it directly.
 129:     if (SplatValue.isSignedIntN(10)) {
 130:       switch (SplatBitSize) {
 131:       default:
 132:         break;
 133:       case 8:
 134:         Op = Is256Vec ? LoongArch::PseudoXVREPLI_B : LoongArch::PseudoVREPLI_B;
 135:         break;
 136:       case 16:
 137:         Op = Is256Vec ? LoongArch::PseudoXVREPLI_H : LoongArch::PseudoVREPLI_H;
 138:         break;
 139:       case 32:
 140:         Op = Is256Vec ? LoongArch::PseudoXVREPLI_W : LoongArch::PseudoVREPLI_W;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 141-168 / 第 141-168 行
```cpp
 141:         break;
 142:       case 64:
 143:         Op = Is256Vec ? LoongArch::PseudoXVREPLI_D : LoongArch::PseudoVREPLI_D;
 144:         break;
 145:       }
 146: 
 147:       EVT EleType = ResTy.getVectorElementType();
 148:       APInt Val = SplatValue.sextOrTrunc(EleType.getSizeInBits());
 149:       SDValue Imm = CurDAG->getTargetConstant(Val, DL, EleType);
 150:       Res = CurDAG->getMachineNode(Op, DL, ResTy, Imm);
 151:       ReplaceNode(Node, Res);
 152:       return;
 153:     }
 154: 
 155:     // Select appropriate [x]vldi instructions for some special constant splats,
 156:     // where the immediate value `imm[12] == 1` for used [x]vldi instructions.
 157:     const auto &TLI =
 158:         *static_cast<const LoongArchTargetLowering *>(getTargetLowering());
 159:     std::pair<bool, uint64_t> ConvertVLDI =
 160:         TLI.isImmVLDILegalForMode1(SplatValue, SplatBitSize);
 161:     if (ConvertVLDI.first) {
 162:       Op = Is256Vec ? LoongArch::XVLDI : LoongArch::VLDI;
 163:       SDValue Imm = CurDAG->getSignedTargetConstant(
 164:           SignExtend32<13>(ConvertVLDI.second), DL, MVT::i32);
 165:       Res = CurDAG->getMachineNode(Op, DL, ResTy, Imm);
 166:       ReplaceNode(Node, Res);
 167:       return;
 168:     }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 169-196 / 第 169-196 行
```cpp
 169:     break;
 170:   }
 171:   }
 172: 
 173:   // Select the default instruction.
 174:   SelectCode(Node);
 175: }
 176: 
 177: bool LoongArchDAGToDAGISel::SelectInlineAsmMemoryOperand(
 178:     const SDValue &Op, InlineAsm::ConstraintCode ConstraintID,
 179:     std::vector<SDValue> &OutOps) {
 180:   SDValue Base = Op;
 181:   SDValue Offset =
 182:       CurDAG->getTargetConstant(0, SDLoc(Op), Subtarget->getGRLenVT());
 183:   switch (ConstraintID) {
 184:   default:
 185:     llvm_unreachable("unexpected asm memory constraint");
 186:   // Reg+Reg addressing.
 187:   case InlineAsm::ConstraintCode::k:
 188:     Base = Op.getOperand(0);
 189:     Offset = Op.getOperand(1);
 190:     break;
 191:   // Reg+simm12 addressing.
 192:   case InlineAsm::ConstraintCode::m:
 193:     if (CurDAG->isBaseWithConstantOffset(Op)) {
 194:       ConstantSDNode *CN = dyn_cast<ConstantSDNode>(Op.getOperand(1));
 195:       if (isIntN(12, CN->getSExtValue())) {
 196:         Base = Op.getOperand(0);
```
- **EN**: The range implements or declares functions including `LoongArchDAGToDAGISel::SelectInlineAsmMemoryOperand`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchDAGToDAGISel::SelectInlineAsmMemoryOperand` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 197-224 / 第 197-224 行
```cpp
 197:         Offset = CurDAG->getTargetConstant(CN->getZExtValue(), SDLoc(Op),
 198:                                            Op.getValueType());
 199:       }
 200:     }
 201:     break;
 202:   // Reg+0 addressing.
 203:   case InlineAsm::ConstraintCode::ZB:
 204:     break;
 205:   // Reg+(simm14<<2) addressing.
 206:   case InlineAsm::ConstraintCode::ZC:
 207:     if (CurDAG->isBaseWithConstantOffset(Op)) {
 208:       ConstantSDNode *CN = dyn_cast<ConstantSDNode>(Op.getOperand(1));
 209:       if (isIntN(16, CN->getSExtValue()) &&
 210:           isAligned(Align(4ULL), CN->getZExtValue())) {
 211:         Base = Op.getOperand(0);
 212:         Offset = CurDAG->getTargetConstant(CN->getZExtValue(), SDLoc(Op),
 213:                                            Op.getValueType());
 214:       }
 215:     }
 216:     break;
 217:   }
 218:   OutOps.push_back(Base);
 219:   OutOps.push_back(Offset);
 220:   return false;
 221: }
 222: 
 223: bool LoongArchDAGToDAGISel::SelectBaseAddr(SDValue Addr, SDValue &Base) {
 224:   // If this is FrameIndex, select it directly. Otherwise just let it get
```
- **EN**: The range implements or declares functions including `LoongArchDAGToDAGISel::SelectBaseAddr`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchDAGToDAGISel::SelectBaseAddr` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 225-252 / 第 225-252 行
```cpp
 225:   // selected to a register independently.
 226:   if (auto *FIN = dyn_cast<FrameIndexSDNode>(Addr))
 227:     Base =
 228:         CurDAG->getTargetFrameIndex(FIN->getIndex(), Subtarget->getGRLenVT());
 229:   else
 230:     Base = Addr;
 231:   return true;
 232: }
 233: 
 234: // Fold constant addresses.
 235: bool LoongArchDAGToDAGISel::SelectAddrConstant(SDValue Addr, SDValue &Base,
 236:                                                SDValue &Offset) {
 237:   SDLoc DL(Addr);
 238:   MVT VT = Addr.getSimpleValueType();
 239: 
 240:   if (!isa<ConstantSDNode>(Addr))
 241:     return false;
 242: 
 243:   // If the constant is a simm12, we can fold the whole constant and use R0 as
 244:   // the base.
 245:   int64_t CVal = cast<ConstantSDNode>(Addr)->getSExtValue();
 246:   if (!isInt<12>(CVal))
 247:     return false;
 248:   Base = CurDAG->getRegister(LoongArch::R0, VT);
 249:   Offset = CurDAG->getSignedTargetConstant(SignExtend64<12>(CVal), DL, VT);
 250:   return true;
 251: }
 252: 
```
- **EN**: The range implements or declares functions including `LoongArchDAGToDAGISel::SelectAddrConstant`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchDAGToDAGISel::SelectAddrConstant` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-280 / 第 253-280 行
```cpp
 253: bool LoongArchDAGToDAGISel::selectNonFIBaseAddr(SDValue Addr, SDValue &Base) {
 254:   // If this is FrameIndex, don't select it.
 255:   if (isa<FrameIndexSDNode>(Addr))
 256:     return false;
 257:   Base = Addr;
 258:   return true;
 259: }
 260: 
 261: bool LoongArchDAGToDAGISel::SelectAddrRegImm12(SDValue Addr, SDValue &Base,
 262:                                                SDValue &Offset) {
 263:   SDLoc DL(Addr);
 264:   MVT VT = Addr.getSimpleValueType();
 265: 
 266:   // The address is the result of an ADD. Here we only consider reg+simm12.
 267:   if (CurDAG->isBaseWithConstantOffset(Addr)) {
 268:     int64_t Imm = cast<ConstantSDNode>(Addr.getOperand(1))->getSExtValue();
 269:     if (isInt<12>(Imm)) {
 270:       Base = Addr.getOperand(0);
 271:       Offset = CurDAG->getSignedTargetConstant(SignExtend64<12>(Imm), DL, VT);
 272:       return true;
 273:     }
 274:   }
 275: 
 276:   // Otherwise, we assume Addr as the base address and use constant 0 as the
 277:   // offset.
 278:   Base = Addr;
 279:   Offset = CurDAG->getTargetConstant(0, DL, VT);
 280:   return true;
```
- **EN**: The range implements or declares functions including `LoongArchDAGToDAGISel::selectNonFIBaseAddr`, `LoongArchDAGToDAGISel::SelectAddrRegImm12`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchDAGToDAGISel::selectNonFIBaseAddr`, `LoongArchDAGToDAGISel::SelectAddrRegImm12` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 281-308 / 第 281-308 行
```cpp
 281: }
 282: 
 283: bool LoongArchDAGToDAGISel::selectShiftMask(SDValue N, unsigned ShiftWidth,
 284:                                             SDValue &ShAmt) {
 285:   // Shift instructions on LoongArch only read the lower 5 or 6 bits of the
 286:   // shift amount. If there is an AND on the shift amount, we can bypass it if
 287:   // it doesn't affect any of those bits.
 288:   if (N.getOpcode() == ISD::AND && isa<ConstantSDNode>(N.getOperand(1))) {
 289:     const APInt &AndMask = N->getConstantOperandAPInt(1);
 290: 
 291:     // Since the max shift amount is a power of 2 we can subtract 1 to make a
 292:     // mask that covers the bits needed to represent all shift amounts.
 293:     assert(isPowerOf2_32(ShiftWidth) && "Unexpected max shift amount!");
 294:     APInt ShMask(AndMask.getBitWidth(), ShiftWidth - 1);
 295: 
 296:     if (ShMask.isSubsetOf(AndMask)) {
 297:       ShAmt = N.getOperand(0);
 298:       return true;
 299:     }
 300: 
 301:     // SimplifyDemandedBits may have optimized the mask so try restoring any
 302:     // bits that are known zero.
 303:     KnownBits Known = CurDAG->computeKnownBits(N->getOperand(0));
 304:     if (ShMask.isSubsetOf(AndMask | Known.Zero)) {
 305:       ShAmt = N.getOperand(0);
 306:       return true;
 307:     }
 308:   } else if (N.getOpcode() == LoongArchISD::BSTRPICK) {
```
- **EN**: The range implements or declares functions including `LoongArchDAGToDAGISel::selectShiftMask`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchDAGToDAGISel::selectShiftMask` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 309-336 / 第 309-336 行
```cpp
 309:     // Similar to the above AND, if there is a BSTRPICK on the shift amount, we
 310:     // can bypass it.
 311:     assert(isPowerOf2_32(ShiftWidth) && "Unexpected max shift amount!");
 312:     assert(isa<ConstantSDNode>(N.getOperand(1)) && "Illegal msb operand!");
 313:     assert(isa<ConstantSDNode>(N.getOperand(2)) && "Illegal lsb operand!");
 314:     uint64_t msb = N.getConstantOperandVal(1), lsb = N.getConstantOperandVal(2);
 315:     if (lsb == 0 && Log2_32(ShiftWidth) <= msb + 1) {
 316:       ShAmt = N.getOperand(0);
 317:       return true;
 318:     }
 319:   } else if (N.getOpcode() == ISD::SUB &&
 320:              isa<ConstantSDNode>(N.getOperand(0))) {
 321:     uint64_t Imm = N.getConstantOperandVal(0);
 322:     // If we are shifting by N-X where N == 0 mod Size, then just shift by -X to
 323:     // generate a NEG instead of a SUB of a constant.
 324:     if (Imm != 0 && Imm % ShiftWidth == 0) {
 325:       SDLoc DL(N);
 326:       EVT VT = N.getValueType();
 327:       SDValue Zero =
 328:           CurDAG->getCopyFromReg(CurDAG->getEntryNode(), DL, LoongArch::R0, VT);
 329:       unsigned NegOpc = VT == MVT::i64 ? LoongArch::SUB_D : LoongArch::SUB_W;
 330:       MachineSDNode *Neg =
 331:           CurDAG->getMachineNode(NegOpc, DL, VT, Zero, N.getOperand(1));
 332:       ShAmt = SDValue(Neg, 0);
 333:       return true;
 334:     }
 335:   }
 336: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 337-364 / 第 337-364 行
```cpp
 337:   ShAmt = N;
 338:   return true;
 339: }
 340: 
 341: bool LoongArchDAGToDAGISel::selectSExti32(SDValue N, SDValue &Val) {
 342:   if (N.getOpcode() == ISD::SIGN_EXTEND_INREG &&
 343:       cast<VTSDNode>(N.getOperand(1))->getVT() == MVT::i32) {
 344:     Val = N.getOperand(0);
 345:     return true;
 346:   }
 347:   if (N.getOpcode() == LoongArchISD::BSTRPICK &&
 348:       N.getConstantOperandVal(1) < UINT64_C(0X1F) &&
 349:       N.getConstantOperandVal(2) == UINT64_C(0)) {
 350:     Val = N;
 351:     return true;
 352:   }
 353:   MVT VT = N.getSimpleValueType();
 354:   if (CurDAG->ComputeNumSignBits(N) > (VT.getSizeInBits() - 32)) {
 355:     Val = N;
 356:     return true;
 357:   }
 358: 
 359:   return false;
 360: }
 361: 
 362: bool LoongArchDAGToDAGISel::selectZExti32(SDValue N, SDValue &Val) {
 363:   if (N.getOpcode() == ISD::AND) {
 364:     auto *C = dyn_cast<ConstantSDNode>(N.getOperand(1));
```
- **EN**: The range implements or declares functions including `LoongArchDAGToDAGISel::selectSExti32`, `LoongArchDAGToDAGISel::selectZExti32`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchDAGToDAGISel::selectSExti32`, `LoongArchDAGToDAGISel::selectZExti32` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 365-392 / 第 365-392 行
```cpp
 365:     if (C && C->getZExtValue() == UINT64_C(0xFFFFFFFF)) {
 366:       Val = N.getOperand(0);
 367:       return true;
 368:     }
 369:   }
 370:   MVT VT = N.getSimpleValueType();
 371:   APInt Mask = APInt::getHighBitsSet(VT.getSizeInBits(), 32);
 372:   if (CurDAG->MaskedValueIsZero(N, Mask)) {
 373:     Val = N;
 374:     return true;
 375:   }
 376: 
 377:   return false;
 378: }
 379: 
 380: bool LoongArchDAGToDAGISel::selectVSplat(SDNode *N, APInt &Imm,
 381:                                          unsigned MinSizeInBits) const {
 382:   if (!Subtarget->hasExtLSX())
 383:     return false;
 384: 
 385:   BuildVectorSDNode *Node = dyn_cast<BuildVectorSDNode>(N);
 386: 
 387:   if (!Node)
 388:     return false;
 389: 
 390:   APInt SplatValue, SplatUndef;
 391:   unsigned SplatBitSize;
 392:   bool HasAnyUndefs;
```
- **EN**: The range implements or declares functions including `LoongArchDAGToDAGISel::selectVSplat`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchDAGToDAGISel::selectVSplat` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 393-420 / 第 393-420 行
```cpp
 393: 
 394:   if (!Node->isConstantSplat(SplatValue, SplatUndef, SplatBitSize, HasAnyUndefs,
 395:                              MinSizeInBits, /*IsBigEndian=*/false))
 396:     return false;
 397: 
 398:   Imm = SplatValue;
 399: 
 400:   return true;
 401: }
 402: 
 403: template <unsigned ImmBitSize, unsigned EltBitSize, bool IsSigned>
 404: bool LoongArchDAGToDAGISel::selectVSplatImm(SDValue N, SDValue &SplatVal) {
 405:   APInt ImmValue;
 406:   EVT EltTy = N->getValueType(0).getVectorElementType();
 407:   unsigned EltBitWidth = EltBitSize ? EltBitSize : EltTy.getSizeInBits();
 408: 
 409:   if (N->getOpcode() == ISD::BITCAST)
 410:     N = N->getOperand(0);
 411: 
 412:   if (selectVSplat(N.getNode(), ImmValue, EltBitWidth) &&
 413:       ImmValue.getBitWidth() == EltBitWidth) {
 414:     if (IsSigned && ImmValue.isSignedIntN(ImmBitSize)) {
 415:       SplatVal = CurDAG->getSignedTargetConstant(
 416:           ImmValue.getSExtValue(), SDLoc(N), Subtarget->getGRLenVT());
 417:       return true;
 418:     }
 419:     if (!IsSigned && ImmValue.isIntN(ImmBitSize)) {
 420:       SplatVal = CurDAG->getTargetConstant(ImmValue.getZExtValue(), SDLoc(N),
```
- **EN**: The range implements or declares functions including `LoongArchDAGToDAGISel::selectVSplatImm`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchDAGToDAGISel::selectVSplatImm` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 421-448 / 第 421-448 行
```cpp
 421:                                            Subtarget->getGRLenVT());
 422:       return true;
 423:     }
 424:   }
 425: 
 426:   return false;
 427: }
 428: 
 429: template <unsigned ImmBitSize>
 430: bool LoongArchDAGToDAGISel::selectVSplatImmNeg(SDValue N,
 431:                                                SDValue &SplatVal) const {
 432:   APInt ImmValue;
 433:   EVT EltTy = N->getValueType(0).getVectorElementType();
 434: 
 435:   if (N->getOpcode() == ISD::BITCAST)
 436:     N = N->getOperand(0);
 437: 
 438:   if (selectVSplat(N.getNode(), ImmValue, EltTy.getSizeInBits()) &&
 439:       ImmValue.getBitWidth() == EltTy.getSizeInBits()) {
 440:     if ((-ImmValue).isIntN(ImmBitSize)) {
 441:       SplatVal = CurDAG->getTargetConstant(-ImmValue.getSExtValue(), SDLoc(N),
 442:                                            Subtarget->getGRLenVT());
 443:       return true;
 444:     }
 445:   }
 446: 
 447:   return false;
 448: }
```
- **EN**: The range implements or declares functions including `LoongArchDAGToDAGISel::selectVSplatImmNeg`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchDAGToDAGISel::selectVSplatImmNeg` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 449-476 / 第 449-476 行
```cpp
 449: 
 450: template <unsigned EltBitSize>
 451: bool LoongArchDAGToDAGISel::selectVSplatUimmInvPow2(SDValue N,
 452:                                                     SDValue &SplatImm) const {
 453:   APInt ImmValue;
 454:   EVT EltTy = N->getValueType(0).getVectorElementType();
 455:   unsigned EltBitWidth = EltBitSize ? EltBitSize : EltTy.getSizeInBits();
 456: 
 457:   if (N->getOpcode() == ISD::BITCAST)
 458:     N = N->getOperand(0);
 459: 
 460:   if (selectVSplat(N.getNode(), ImmValue, EltBitWidth) &&
 461:       ImmValue.getBitWidth() == EltBitWidth) {
 462:     int32_t Log2 = (~ImmValue).exactLogBase2();
 463: 
 464:     if (Log2 != -1) {
 465:       SplatImm = CurDAG->getSignedTargetConstant(Log2, SDLoc(N), EltTy);
 466:       return true;
 467:     }
 468:   }
 469: 
 470:   return false;
 471: }
 472: 
 473: template <unsigned EltBitSize>
 474: bool LoongArchDAGToDAGISel::selectVSplatUimmPow2(SDValue N,
 475:                                                  SDValue &SplatImm) const {
 476:   APInt ImmValue;
```
- **EN**: The range implements or declares functions including `LoongArchDAGToDAGISel::selectVSplatUimmInvPow2`, `LoongArchDAGToDAGISel::selectVSplatUimmPow2`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchDAGToDAGISel::selectVSplatUimmInvPow2`, `LoongArchDAGToDAGISel::selectVSplatUimmPow2` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 477-501 / 第 477-501 行
```cpp
 477:   EVT EltTy = N->getValueType(0).getVectorElementType();
 478:   unsigned EltBitWidth = EltBitSize ? EltBitSize : EltTy.getSizeInBits();
 479: 
 480:   if (N->getOpcode() == ISD::BITCAST)
 481:     N = N->getOperand(0);
 482: 
 483:   if (selectVSplat(N.getNode(), ImmValue, EltBitWidth) &&
 484:       ImmValue.getBitWidth() == EltBitWidth) {
 485:     int32_t Log2 = ImmValue.exactLogBase2();
 486: 
 487:     if (Log2 != -1) {
 488:       SplatImm = CurDAG->getSignedTargetConstant(Log2, SDLoc(N), EltTy);
 489:       return true;
 490:     }
 491:   }
 492: 
 493:   return false;
 494: }
 495: 
 496: // This pass converts a legalized DAG into a LoongArch-specific DAG, ready
 497: // for instruction scheduling.
 498: FunctionPass *llvm::createLoongArchISelDag(LoongArchTargetMachine &TM,
 499:                                            CodeGenOptLevel OptLevel) {
 500:   return new LoongArchDAGToDAGISelLegacy(TM, OptLevel);
 501: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `LoongArchISelDAGToDAG.h`
- `LoongArchISelLowering.h`
- `MCTargetDesc/LoongArchMCTargetDesc.h`
- `MCTargetDesc/LoongArchMatInt.h`
- `llvm/Support/KnownBits.h`
- `llvm/Support/raw_ostream.h`
