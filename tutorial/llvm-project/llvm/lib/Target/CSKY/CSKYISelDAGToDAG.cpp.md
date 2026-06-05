# CSKYISelDAGToDAG.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYISelDAGToDAG.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file defines an instruction selector for the CSKY target.
- 目的（中文）: 实现 DAG 到 DAG 的指令选择，将合法化后的 SelectionDAG 节点转换为目标机器指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYISelDAGToDAG.cpp - A dag to dag inst selector for CSKY---------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines an instruction selector for the CSKY target.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "CSKY.h"
  14: #include "CSKYSubtarget.h"
  15: #include "CSKYTargetMachine.h"
  16: #include "MCTargetDesc/CSKYMCTargetDesc.h"
  17: #include "llvm/CodeGen/MachineFrameInfo.h"
  18: #include "llvm/CodeGen/SelectionDAG.h"
  19: #include "llvm/CodeGen/SelectionDAGISel.h"
  20: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 21-40

```cpp
  21: using namespace llvm;
  22: 
  23: #define DEBUG_TYPE "csky-isel"
  24: #define PASS_NAME "CSKY DAG->DAG Pattern Instruction Selection"
  25: 
  26: namespace {
  27: class CSKYDAGToDAGISel : public SelectionDAGISel {
  28:   const CSKYSubtarget *Subtarget;
  29: 
  30: public:
  31:   explicit CSKYDAGToDAGISel(CSKYTargetMachine &TM, CodeGenOptLevel OptLevel)
  32:       : SelectionDAGISel(TM, OptLevel) {}
  33: 
  34:   bool runOnMachineFunction(MachineFunction &MF) override {
  35:     // Reset the subtarget each time through.
  36:     Subtarget = &MF.getSubtarget<CSKYSubtarget>();
  37:     SelectionDAGISel::runOnMachineFunction(MF);
  38:     return true;
  39:   }
  40: 
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as CSKYDAGToDAGISel, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as CSKYDAGToDAGISel contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 CSKYDAGToDAGISel 等接口或数据结构，用于组织该文件暴露的目标专用行为。 CSKYDAGToDAGISel 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 41-60

```cpp
  41:   void Select(SDNode *N) override;
  42:   bool selectAddCarry(SDNode *N);
  43:   bool selectSubCarry(SDNode *N);
  44:   bool selectBITCAST_TO_LOHI(SDNode *N);
  45:   bool selectInlineAsm(SDNode *N);
  46: 
  47:   SDNode *createGPRPairNode(EVT VT, SDValue V0, SDValue V1);
  48: 
  49:   bool SelectInlineAsmMemoryOperand(const SDValue &Op,
  50:                                     InlineAsm::ConstraintCode ConstraintID,
  51:                                     std::vector<SDValue> &OutOps) override;
  52: 
  53: #include "CSKYGenDAGISel.inc"
  54: };
  55: 
  56: class CSKYDAGToDAGISelLegacy : public SelectionDAGISelLegacy {
  57: public:
  58:   static char ID;
  59:   explicit CSKYDAGToDAGISelLegacy(CSKYTargetMachine &TM,
  60:                                   CodeGenOptLevel OptLevel)
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. This chunk introduces interfaces or data structures such as CSKYDAGToDAGISelLegacy, which organize the target-specific behavior exposed by the file. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 这一段引入了 CSKYDAGToDAGISelLegacy 等接口或数据结构，用于组织该文件暴露的目标专用行为。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 61-80

```cpp
  61:       : SelectionDAGISelLegacy(
  62:             ID, std::make_unique<CSKYDAGToDAGISel>(TM, OptLevel)) {}
  63: };
  64: } // namespace
  65: 
  66: char CSKYDAGToDAGISelLegacy::ID = 0;
  67: 
  68: INITIALIZE_PASS(CSKYDAGToDAGISelLegacy, DEBUG_TYPE, PASS_NAME, false, false)
  69: 
  70: void CSKYDAGToDAGISel::Select(SDNode *N) {
  71:   // If we have a custom node, we have already selected
  72:   if (N->isMachineOpcode()) {
  73:     LLVM_DEBUG(dbgs() << "== "; N->dump(CurDAG); dbgs() << "\n");
  74:     N->setNodeId(-1);
  75:     return;
  76:   }
  77: 
  78:   SDLoc Dl(N);
  79:   unsigned Opcode = N->getOpcode();
  80:   bool IsSelected = false;
```

- EN: Function bodies or method definitions such as SelectionDAGISelLegacy, Select contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: SelectionDAGISelLegacy, Select 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 81-100

```cpp
  81: 
  82:   switch (Opcode) {
  83:   default:
  84:     break;
  85:   case ISD::UADDO_CARRY:
  86:     IsSelected = selectAddCarry(N);
  87:     break;
  88:   case ISD::USUBO_CARRY:
  89:     IsSelected = selectSubCarry(N);
  90:     break;
  91:   case ISD::GLOBAL_OFFSET_TABLE: {
  92:     Register GP = Subtarget->getInstrInfo()->getGlobalBaseReg(*MF);
  93:     ReplaceNode(N, CurDAG->getRegister(GP, N->getValueType(0)).getNode());
  94: 
  95:     IsSelected = true;
  96:     break;
  97:   }
  98:   case ISD::FrameIndex: {
  99:     SDValue Imm = CurDAG->getTargetConstant(0, Dl, MVT::i32);
 100:     int FI = cast<FrameIndexSDNode>(N)->getIndex();
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 101-120

```cpp
 101:     SDValue TFI = CurDAG->getTargetFrameIndex(FI, MVT::i32);
 102:     ReplaceNode(N, CurDAG->getMachineNode(Subtarget->hasE2() ? CSKY::ADDI32
 103:                                                              : CSKY::ADDI16XZ,
 104:                                           Dl, MVT::i32, TFI, Imm));
 105: 
 106:     IsSelected = true;
 107:     break;
 108:   }
 109:   case CSKYISD::BITCAST_TO_LOHI:
 110:     IsSelected = selectBITCAST_TO_LOHI(N);
 111:     break;
 112:   case ISD::INLINEASM:
 113:   case ISD::INLINEASM_BR:
 114:     IsSelected = selectInlineAsm(N);
 115:     break;
 116:   }
 117: 
 118:   if (IsSelected)
 119:     return;
 120: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 121-140

```cpp
 121:   // Select the default instruction.
 122:   SelectCode(N);
 123: }
 124: 
 125: bool CSKYDAGToDAGISel::selectInlineAsm(SDNode *N) {
 126:   std::vector<SDValue> AsmNodeOperands;
 127:   InlineAsm::Flag Flag;
 128:   bool Changed = false;
 129:   unsigned NumOps = N->getNumOperands();
 130: 
 131:   // Normally, i64 data is bounded to two arbitrary GRPs for "%r" constraint.
 132:   // However, some instructions (e.g. mula.s32) require GPR pair.
 133:   // Since there is no constraint to explicitly specify a
 134:   // reg pair, we use GPRPair reg class for "%r" for 64-bit data.
 135: 
 136:   SDLoc dl(N);
 137:   SDValue Glue =
 138:       N->getGluedNode() ? N->getOperand(NumOps - 1) : SDValue(nullptr, 0);
 139: 
 140:   SmallVector<bool, 8> OpChanged;
```

- EN: This chunk introduces interfaces or data structures such as for, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as selectInlineAsm contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 这一段引入了 for 等接口或数据结构，用于组织该文件暴露的目标专用行为。 selectInlineAsm 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 141-160

```cpp
 141:   // Glue node will be appended late.
 142:   for (unsigned i = 0, e = N->getGluedNode() ? NumOps - 1 : NumOps; i < e;
 143:        ++i) {
 144:     SDValue op = N->getOperand(i);
 145:     AsmNodeOperands.push_back(op);
 146: 
 147:     if (i < InlineAsm::Op_FirstOperand)
 148:       continue;
 149: 
 150:     if (const auto *C = dyn_cast<ConstantSDNode>(N->getOperand(i)))
 151:       Flag = InlineAsm::Flag(C->getZExtValue());
 152:     else
 153:       continue;
 154: 
 155:     // Immediate operands to inline asm in the SelectionDAG are modeled with
 156:     // two operands. The first is a constant of value InlineAsm::Kind::Imm, and
 157:     // the second is a constant with the value of the immediate. If we get here
 158:     // and we have a Kind::Imm, skip the next operand, and continue.
 159:     if (Flag.isImmKind()) {
 160:       SDValue op = N->getOperand(++i);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 161-180

```cpp
 161:       AsmNodeOperands.push_back(op);
 162:       continue;
 163:     }
 164: 
 165:     const unsigned NumRegs = Flag.getNumOperandRegisters();
 166:     if (NumRegs)
 167:       OpChanged.push_back(false);
 168: 
 169:     unsigned DefIdx = 0;
 170:     bool IsTiedToChangedOp = false;
 171:     // If it's a use that is tied with a previous def, it has no
 172:     // reg class constraint.
 173:     if (Changed && Flag.isUseOperandTiedToDef(DefIdx))
 174:       IsTiedToChangedOp = OpChanged[DefIdx];
 175: 
 176:     // Memory operands to inline asm in the SelectionDAG are modeled with two
 177:     // operands: a constant of value InlineAsm::Kind::Mem followed by the input
 178:     // operand. If we get here and we have a Kind::Mem, skip the next operand
 179:     // (so it doesn't get misinterpreted), and continue. We do this here because
 180:     // it's important to update the OpChanged array correctly before moving on.
```

- EN: This chunk introduces interfaces or data structures such as constraint, which organize the target-specific behavior exposed by the file. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 这一段引入了 constraint 等接口或数据结构，用于组织该文件暴露的目标专用行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 181-200

```cpp
 181:     if (Flag.isMemKind()) {
 182:       SDValue op = N->getOperand(++i);
 183:       AsmNodeOperands.push_back(op);
 184:       continue;
 185:     }
 186: 
 187:     if (!Flag.isRegUseKind() && !Flag.isRegDefKind() &&
 188:         !Flag.isRegDefEarlyClobberKind())
 189:       continue;
 190: 
 191:     unsigned RC;
 192:     const bool HasRC = Flag.hasRegClassConstraint(RC);
 193:     if ((!IsTiedToChangedOp && (!HasRC || RC != CSKY::GPRRegClassID)) ||
 194:         NumRegs != 2)
 195:       continue;
 196: 
 197:     assert((i + 2 < NumOps) && "Invalid number of operands in inline asm");
 198:     SDValue V0 = N->getOperand(i + 1);
 199:     SDValue V1 = N->getOperand(i + 2);
 200:     unsigned Reg0 = cast<RegisterSDNode>(V0)->getReg();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 201-220

```cpp
 201:     unsigned Reg1 = cast<RegisterSDNode>(V1)->getReg();
 202:     SDValue PairedReg;
 203:     MachineRegisterInfo &MRI = MF->getRegInfo();
 204: 
 205:     if (Flag.isRegDefKind() || Flag.isRegDefEarlyClobberKind()) {
 206:       // Replace the two GPRs with 1 GPRPair and copy values from GPRPair to
 207:       // the original GPRs.
 208: 
 209:       Register GPVR = MRI.createVirtualRegister(&CSKY::GPRPairRegClass);
 210:       PairedReg = CurDAG->getRegister(GPVR, MVT::i64);
 211:       SDValue Chain = SDValue(N, 0);
 212: 
 213:       SDNode *GU = N->getGluedUser();
 214:       SDValue RegCopy =
 215:           CurDAG->getCopyFromReg(Chain, dl, GPVR, MVT::i64, Chain.getValue(1));
 216: 
 217:       // Extract values from a GPRPair reg and copy to the original GPR reg.
 218:       SDValue Sub0 =
 219:           CurDAG->getTargetExtractSubreg(CSKY::sub32_0, dl, MVT::i32, RegCopy);
 220:       SDValue Sub1 =
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 221-240

```cpp
 221:           CurDAG->getTargetExtractSubreg(CSKY::sub32_32, dl, MVT::i32, RegCopy);
 222:       SDValue T0 =
 223:           CurDAG->getCopyToReg(Sub0, dl, Reg0, Sub0, RegCopy.getValue(1));
 224:       SDValue T1 = CurDAG->getCopyToReg(Sub1, dl, Reg1, Sub1, T0.getValue(1));
 225: 
 226:       // Update the original glue user.
 227:       std::vector<SDValue> Ops(GU->op_begin(), GU->op_end() - 1);
 228:       Ops.push_back(T1.getValue(1));
 229:       CurDAG->UpdateNodeOperands(GU, Ops);
 230:     } else {
 231:       // For Kind  == InlineAsm::Kind::RegUse, we first copy two GPRs into a
 232:       // GPRPair and then pass the GPRPair to the inline asm.
 233:       SDValue Chain = AsmNodeOperands[InlineAsm::Op_InputChain];
 234: 
 235:       // As REG_SEQ doesn't take RegisterSDNode, we copy them first.
 236:       SDValue T0 =
 237:           CurDAG->getCopyFromReg(Chain, dl, Reg0, MVT::i32, Chain.getValue(1));
 238:       SDValue T1 =
 239:           CurDAG->getCopyFromReg(Chain, dl, Reg1, MVT::i32, T0.getValue(1));
 240:       SDValue Pair = SDValue(createGPRPairNode(MVT::i64, T0, T1), 0);
```

- EN: At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 241-260

```cpp
 241: 
 242:       // Copy REG_SEQ into a GPRPair-typed VR and replace the original two
 243:       // i32 VRs of inline asm with it.
 244:       Register GPVR = MRI.createVirtualRegister(&CSKY::GPRPairRegClass);
 245:       PairedReg = CurDAG->getRegister(GPVR, MVT::i64);
 246:       Chain = CurDAG->getCopyToReg(T1, dl, GPVR, Pair, T1.getValue(1));
 247: 
 248:       AsmNodeOperands[InlineAsm::Op_InputChain] = Chain;
 249:       Glue = Chain.getValue(1);
 250:     }
 251: 
 252:     Changed = true;
 253: 
 254:     if (PairedReg.getNode()) {
 255:       OpChanged[OpChanged.size() - 1] = true;
 256:       // TODO: maybe a setter for getNumOperandRegisters?
 257:       Flag = InlineAsm::Flag(Flag.getKind(), 1 /* RegNum*/);
 258:       if (IsTiedToChangedOp)
 259:         Flag.setMatchingOp(DefIdx);
 260:       else
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 261-280

```cpp
 261:         Flag.setRegClass(CSKY::GPRPairRegClassID);
 262:       // Replace the current flag.
 263:       AsmNodeOperands[AsmNodeOperands.size() - 1] =
 264:           CurDAG->getTargetConstant(Flag, dl, MVT::i32);
 265:       // Add the new register node and skip the original two GPRs.
 266:       AsmNodeOperands.push_back(PairedReg);
 267:       // Skip the next two GPRs.
 268:       i += 2;
 269:     }
 270:   }
 271: 
 272:   if (Glue.getNode())
 273:     AsmNodeOperands.push_back(Glue);
 274:   if (!Changed)
 275:     return false;
 276: 
 277:   SDValue New = CurDAG->getNode(N->getOpcode(), SDLoc(N),
 278:                                 CurDAG->getVTList(MVT::Other, MVT::Glue),
 279:                                 AsmNodeOperands);
 280:   New->setNodeId(-1);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 281-300

```cpp
 281:   ReplaceNode(N, New.getNode());
 282:   return true;
 283: }
 284: 
 285: bool CSKYDAGToDAGISel::selectBITCAST_TO_LOHI(SDNode *N) {
 286:   SDLoc Dl(N);
 287:   auto VT = N->getValueType(0);
 288:   auto V = N->getOperand(0);
 289: 
 290:   if (!Subtarget->hasFPUv2DoubleFloat())
 291:     return false;
 292: 
 293:   SDValue V1 = SDValue(CurDAG->getMachineNode(CSKY::FMFVRL_D, Dl, VT, V), 0);
 294:   SDValue V2 = SDValue(CurDAG->getMachineNode(CSKY::FMFVRH_D, Dl, VT, V), 0);
 295: 
 296:   ReplaceUses(SDValue(N, 0), V1);
 297:   ReplaceUses(SDValue(N, 1), V2);
 298:   CurDAG->RemoveDeadNode(N);
 299: 
 300:   return true;
```

- EN: Function bodies or method definitions such as selectBITCAST_TO_LOHI contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: selectBITCAST_TO_LOHI 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 301-320

```cpp
 301: }
 302: 
 303: bool CSKYDAGToDAGISel::selectAddCarry(SDNode *N) {
 304:   MachineSDNode *NewNode = nullptr;
 305:   auto Type0 = N->getValueType(0);
 306:   auto Type1 = N->getValueType(1);
 307:   auto Op0 = N->getOperand(0);
 308:   auto Op1 = N->getOperand(1);
 309:   auto Op2 = N->getOperand(2);
 310: 
 311:   SDLoc Dl(N);
 312: 
 313:   if (isNullConstant(Op2)) {
 314:     auto *CA = CurDAG->getMachineNode(
 315:         Subtarget->has2E3() ? CSKY::CLRC32 : CSKY::CLRC16, Dl, Type1);
 316:     NewNode = CurDAG->getMachineNode(
 317:         Subtarget->has2E3() ? CSKY::ADDC32 : CSKY::ADDC16, Dl, {Type0, Type1},
 318:         {Op0, Op1, SDValue(CA, 0)});
 319:   } else if (isOneConstant(Op2)) {
 320:     auto *CA = CurDAG->getMachineNode(
```

- EN: Function bodies or method definitions such as selectAddCarry contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: selectAddCarry 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 321-340

```cpp
 321:         Subtarget->has2E3() ? CSKY::SETC32 : CSKY::SETC16, Dl, Type1);
 322:     NewNode = CurDAG->getMachineNode(
 323:         Subtarget->has2E3() ? CSKY::ADDC32 : CSKY::ADDC16, Dl, {Type0, Type1},
 324:         {Op0, Op1, SDValue(CA, 0)});
 325:   } else {
 326:     NewNode = CurDAG->getMachineNode(Subtarget->has2E3() ? CSKY::ADDC32
 327:                                                          : CSKY::ADDC16,
 328:                                      Dl, {Type0, Type1}, {Op0, Op1, Op2});
 329:   }
 330:   ReplaceNode(N, NewNode);
 331:   return true;
 332: }
 333: 
 334: static SDValue InvertCarryFlag(const CSKYSubtarget *Subtarget,
 335:                                SelectionDAG *DAG, SDLoc Dl, SDValue OldCarry) {
 336:   auto NewCarryReg =
 337:       DAG->getMachineNode(Subtarget->has2E3() ? CSKY::MVCV32 : CSKY::MVCV16, Dl,
 338:                           MVT::i32, OldCarry);
 339:   auto NewCarry =
 340:       DAG->getMachineNode(Subtarget->hasE2() ? CSKY::BTSTI32 : CSKY::BTSTI16,
```

- EN: Function bodies or method definitions such as InvertCarryFlag contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: InvertCarryFlag 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 341-360

```cpp
 341:                           Dl, OldCarry.getValueType(), SDValue(NewCarryReg, 0),
 342:                           DAG->getTargetConstant(0, Dl, MVT::i32));
 343:   return SDValue(NewCarry, 0);
 344: }
 345: 
 346: bool CSKYDAGToDAGISel::selectSubCarry(SDNode *N) {
 347:   MachineSDNode *NewNode = nullptr;
 348:   auto Type0 = N->getValueType(0);
 349:   auto Type1 = N->getValueType(1);
 350:   auto Op0 = N->getOperand(0);
 351:   auto Op1 = N->getOperand(1);
 352:   auto Op2 = N->getOperand(2);
 353: 
 354:   SDLoc Dl(N);
 355: 
 356:   if (isNullConstant(Op2)) {
 357:     auto *CA = CurDAG->getMachineNode(
 358:         Subtarget->has2E3() ? CSKY::SETC32 : CSKY::SETC16, Dl, Type1);
 359:     NewNode = CurDAG->getMachineNode(
 360:         Subtarget->has2E3() ? CSKY::SUBC32 : CSKY::SUBC16, Dl, {Type0, Type1},
```

- EN: Function bodies or method definitions such as selectSubCarry contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: selectSubCarry 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 361-380

```cpp
 361:         {Op0, Op1, SDValue(CA, 0)});
 362:   } else if (isOneConstant(Op2)) {
 363:     auto *CA = CurDAG->getMachineNode(
 364:         Subtarget->has2E3() ? CSKY::CLRC32 : CSKY::CLRC16, Dl, Type1);
 365:     NewNode = CurDAG->getMachineNode(
 366:         Subtarget->has2E3() ? CSKY::SUBC32 : CSKY::SUBC16, Dl, {Type0, Type1},
 367:         {Op0, Op1, SDValue(CA, 0)});
 368:   } else {
 369:     auto CarryIn = InvertCarryFlag(Subtarget, CurDAG, Dl, Op2);
 370:     NewNode = CurDAG->getMachineNode(Subtarget->has2E3() ? CSKY::SUBC32
 371:                                                          : CSKY::SUBC16,
 372:                                      Dl, {Type0, Type1}, {Op0, Op1, CarryIn});
 373:   }
 374:   auto CarryOut = InvertCarryFlag(Subtarget, CurDAG, Dl, SDValue(NewNode, 1));
 375: 
 376:   ReplaceUses(SDValue(N, 0), SDValue(NewNode, 0));
 377:   ReplaceUses(SDValue(N, 1), CarryOut);
 378:   CurDAG->RemoveDeadNode(N);
 379: 
 380:   return true;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 381-400

```cpp
 381: }
 382: 
 383: SDNode *CSKYDAGToDAGISel::createGPRPairNode(EVT VT, SDValue V0, SDValue V1) {
 384:   SDLoc dl(V0.getNode());
 385:   SDValue RegClass =
 386:       CurDAG->getTargetConstant(CSKY::GPRPairRegClassID, dl, MVT::i32);
 387:   SDValue SubReg0 = CurDAG->getTargetConstant(CSKY::sub32_0, dl, MVT::i32);
 388:   SDValue SubReg1 = CurDAG->getTargetConstant(CSKY::sub32_32, dl, MVT::i32);
 389:   const SDValue Ops[] = {RegClass, V0, SubReg0, V1, SubReg1};
 390:   return CurDAG->getMachineNode(TargetOpcode::REG_SEQUENCE, dl, VT, Ops);
 391: }
 392: 
 393: bool CSKYDAGToDAGISel::SelectInlineAsmMemoryOperand(
 394:     const SDValue &Op, const InlineAsm::ConstraintCode ConstraintID,
 395:     std::vector<SDValue> &OutOps) {
 396:   switch (ConstraintID) {
 397:   case InlineAsm::ConstraintCode::m:
 398:     // We just support simple memory operands that have a single address
 399:     // operand and need no special handling.
 400:     OutOps.push_back(Op);
```

- EN: Function bodies or method definitions such as SelectInlineAsmMemoryOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: SelectInlineAsmMemoryOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

### Lines 401-412

```cpp
 401:     return false;
 402:   default:
 403:     break;
 404:   }
 405: 
 406:   return true;
 407: }
 408: 
 409: FunctionPass *llvm::createCSKYISelDag(CSKYTargetMachine &TM,
 410:                                       CodeGenOptLevel OptLevel) {
 411:   return new CSKYDAGToDAGISelLegacy(TM, OptLevel);
 412: }
```

- EN: At the backend level, this chunk supports instruction-selection decisions over legalized DAG nodes.
- 中文: 在后端层面，这一段支持对合法化 DAG 节点进行指令选择。

## Key Concepts / 关键概念

- Pattern-based instruction selection / 基于模式的指令选择
- Target DAG nodes / 目标 DAG 节点
- SelectionDAG lowering / SelectionDAG 降级
- MachineFunction state / MachineFunction 状态
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKY.h`, `CSKYSubtarget.h`, `CSKYTargetMachine.h`, `MCTargetDesc/CSKYMCTargetDesc.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/SelectionDAGISel.h`, `CSKYGenDAGISel.inc`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Generated or companion files / 生成或配套文件: `CSKYGenDAGISel.inc`
