# RDFCopy.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/RDFCopy.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements a Hexagon backend component.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- RDFCopy.cpp --------------------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // RDF-based copy propagation.
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #include "RDFCopy.h"
    14: #include "llvm/CodeGen/MachineDominators.h"
    15: #include "llvm/CodeGen/MachineInstr.h"
    16: #include "llvm/CodeGen/MachineOperand.h"
    17: #include "llvm/CodeGen/RDFGraph.h"
    18: #include "llvm/CodeGen/RDFLiveness.h"
    19: #include "llvm/CodeGen/RDFRegisters.h"
    20: #include "llvm/CodeGen/TargetOpcodes.h"
    21: #include "llvm/CodeGen/TargetRegisterInfo.h"
    22: #include "llvm/MC/MCRegisterInfo.h"
    23: #include "llvm/Support/CommandLine.h"
    24: #include "llvm/Support/Debug.h"
    25: #include "llvm/Support/ErrorHandling.h"
    26: #include "llvm/Support/raw_ostream.h"
    27: #include <cassert>
    28: #include <cstdint>
    29: 
    30: using namespace llvm;
    31: using namespace rdf;
    32: 
    33: #ifndef NDEBUG
    34: static cl::opt<unsigned> CpLimit("rdf-cp-limit", cl::init(0), cl::Hidden);
    35: static unsigned CpCount = 0;
    36: #endif
    37: 
    38: bool CopyPropagation::interpretAsCopy(const MachineInstr *MI, EqualityMap &EM) {
    39:   unsigned Opc = MI->getOpcode();
    40:   switch (Opc) {
    41:     case TargetOpcode::COPY: {
    42:       const MachineOperand &Dst = MI->getOperand(0);
    43:       const MachineOperand &Src = MI->getOperand(1);
    44:       RegisterRef DstR = DFG.makeRegRef(Dst.getReg(), Dst.getSubReg());
    45:       RegisterRef SrcR = DFG.makeRegRef(Src.getReg(), Src.getSubReg());
    46:       assert(DstR.asMCReg().isPhysical());
    47:       assert(SrcR.asMCReg().isPhysical());
    48:       const TargetRegisterInfo &TRI = DFG.getTRI();
    49:       if (TRI.getMinimalPhysRegClass(DstR.asMCReg()) !=
    50:           TRI.getMinimalPhysRegClass(SrcR.asMCReg()))
```
- EN: It imports headers such as RDFCopy.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, ... (16 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, rdf) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as CpLimit, CopyPropagation::interpretAsCopy, getOpcode, getOperand, ... (7 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 RDFCopy.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, ... (16 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, rdf），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 CpLimit, CopyPropagation::interpretAsCopy, getOpcode, getOperand, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-100 / 第 51-100 行

```cpp
    51:         return false;
    52:       if (!DFG.isTracked(SrcR) || !DFG.isTracked(DstR))
    53:         return false;
    54:       EM.insert(std::make_pair(DstR, SrcR));
    55:       return true;
    56:     }
    57:     case TargetOpcode::REG_SEQUENCE:
    58:       llvm_unreachable("Unexpected REG_SEQUENCE");
    59:   }
    60:   return false;
    61: }
    62: 
    63: void CopyPropagation::recordCopy(NodeAddr<StmtNode*> SA, EqualityMap &EM) {
    64:   CopyMap.insert(std::make_pair(SA.Id, EM));
    65:   Copies.push_back(SA.Id);
    66: 
    67:   for (auto I : EM) {
    68:     auto FS = DefM.find(I.second.Id);
    69:     if (FS == DefM.end() || FS->second.empty())
    70:       continue; // Undefined source
    71:     RDefMap[I.second][SA.Id] = FS->second.top()->Id;
    72:     // Insert DstR into the map.
    73:     RDefMap[I.first];
    74:   }
    75: }
    76: 
    77: 
    78: void CopyPropagation::updateMap(NodeAddr<InstrNode*> IA) {
    79:   RegisterSet RRs(DFG.getPRI());
    80:   for (NodeAddr<RefNode*> RA : IA.Addr->members(DFG))
    81:     RRs.insert(RA.Addr->getRegRef(DFG));
    82:   bool Common = false;
    83:   for (auto &R : RDefMap) {
    84:     if (!RRs.count(R.first))
    85:       continue;
    86:     Common = true;
    87:     break;
    88:   }
    89:   if (!Common)
    90:     return;
    91: 
    92:   for (auto &R : RDefMap) {
    93:     if (!RRs.count(R.first))
    94:       continue;
    95:     auto F = DefM.find(R.first.Id);
    96:     if (F == DefM.end() || F->second.empty())
    97:       continue;
    98:     R.second[IA.Id] = F->second.top()->Id;
    99:   }
   100: }
```
- EN: It declares or implements routines such as insert, llvm_unreachable, CopyPropagation::recordCopy, push_back, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 insert, llvm_unreachable, CopyPropagation::recordCopy, push_back, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 101-150 / 第 101-150 行

```cpp
   101: 
   102: bool CopyPropagation::scanBlock(MachineBasicBlock *B) {
   103:   bool Changed = false;
   104:   NodeAddr<BlockNode*> BA = DFG.findBlock(B);
   105:   DFG.markBlock(BA.Id, DefM);
   106: 
   107:   for (NodeAddr<InstrNode*> IA : BA.Addr->members(DFG)) {
   108:     if (DFG.IsCode<NodeAttrs::Stmt>(IA)) {
   109:       NodeAddr<StmtNode*> SA = IA;
   110:       EqualityMap EM(RegisterRefLess(DFG.getPRI()));
   111:       if (interpretAsCopy(SA.Addr->getCode(), EM))
   112:         recordCopy(SA, EM);
   113:     }
   114: 
   115:     updateMap(IA);
   116:     DFG.pushAllDefs(IA, DefM);
   117:   }
   118: 
   119:   MachineDomTreeNode *N = MDT.getNode(B);
   120:   for (auto *I : *N)
   121:     Changed |= scanBlock(I->getBlock());
   122: 
   123:   DFG.releaseBlock(BA.Id, DefM);
   124:   return Changed;
   125: }
   126: 
   127: bool CopyPropagation::run() {
   128:   scanBlock(&DFG.getMF().front());
   129: 
   130:   if (trace()) {
   131:     dbgs() << "Copies:\n";
   132:     for (NodeId I : Copies) {
   133:       dbgs() << "Instr: " << *DFG.addr<StmtNode*>(I).Addr->getCode();
   134:       dbgs() << "   eq: {";
   135:       if (auto It = CopyMap.find(I); It != CopyMap.end()) {
   136:         for (auto J : It->second)
   137:           dbgs() << ' ' << Print<RegisterRef>(J.first, DFG) << '='
   138:                  << Print<RegisterRef>(J.second, DFG);
   139:       }
   140:       dbgs() << " }\n";
   141:     }
   142:     dbgs() << "\nRDef map:\n";
   143:     for (auto R : RDefMap) {
   144:       dbgs() << Print<RegisterRef>(R.first, DFG) << " -> {";
   145:       for (auto &M : R.second)
   146:         dbgs() << ' ' << Print<NodeId>(M.first, DFG) << ':'
   147:                << Print<NodeId>(M.second, DFG);
   148:       dbgs() << " }\n";
   149:     }
   150:   }
```
- EN: It declares or implements routines such as CopyPropagation::scanBlock, findBlock, markBlock, EM, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 CopyPropagation::scanBlock, findBlock, markBlock, EM, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-200 / 第 151-200 行

```cpp
   151: 
   152:   bool Changed = false;
   153: #ifndef NDEBUG
   154:   bool HasLimit = CpLimit.getNumOccurrences() > 0;
   155: #endif
   156: 
   157:   auto MinPhysReg = [this](RegisterRef RR) -> MCRegister {
   158:     const TargetRegisterInfo &TRI = DFG.getTRI();
   159:     const TargetRegisterClass &RC = *TRI.getMinimalPhysRegClass(RR.asMCReg());
   160:     if ((RC.LaneMask & RR.Mask) == RC.LaneMask)
   161:       return RR.asMCReg();
   162:     for (MCSubRegIndexIterator S(RR.asMCReg(), &TRI); S.isValid(); ++S)
   163:       if (RR.Mask == TRI.getSubRegIndexLaneMask(S.getSubRegIndex()))
   164:         return S.getSubReg();
   165:     llvm_unreachable("Should have found a register");
   166:     return MCRegister();
   167:   };
   168: 
   169:   const PhysicalRegisterInfo &PRI = DFG.getPRI();
   170: 
   171:   for (NodeId C : Copies) {
   172: #ifndef NDEBUG
   173:     if (HasLimit && CpCount >= CpLimit)
   174:       break;
   175: #endif
   176:     auto SA = DFG.addr<InstrNode*>(C);
   177:     auto FS = CopyMap.find(SA.Id);
   178:     if (FS == CopyMap.end())
   179:       continue;
   180: 
   181:     EqualityMap &EM = FS->second;
   182:     for (NodeAddr<DefNode*> DA : SA.Addr->members_if(DFG.IsDef, DFG)) {
   183:       RegisterRef DR = DA.Addr->getRegRef(DFG);
   184:       auto FR = EM.find(DR);
   185:       if (FR == EM.end())
   186:         continue;
   187:       RegisterRef SR = FR->second;
   188:       if (PRI.equal_to(DR, SR))
   189:         continue;
   190: 
   191:       auto &RDefSR = RDefMap[SR];
   192:       NodeId RDefSR_SA = RDefSR[SA.Id];
   193: 
   194:       for (NodeId N = DA.Addr->getReachedUse(), NextN; N; N = NextN) {
   195:         auto UA = DFG.addr<UseNode*>(N);
   196:         NextN = UA.Addr->getSibling();
   197:         uint16_t F = UA.Addr->getFlags();
   198:         if ((F & NodeAttrs::PhiRef) || (F & NodeAttrs::Fixed))
   199:           continue;
   200:         if (!PRI.equal_to(UA.Addr->getRegRef(DFG), DR))
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as getTRI, getMinimalPhysRegClass, isValid, llvm_unreachable, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 getTRI, getMinimalPhysRegClass, isValid, llvm_unreachable, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-250 / 第 201-250 行

```cpp
   201:           continue;
   202: 
   203:         NodeAddr<InstrNode*> IA = UA.Addr->getOwner(DFG);
   204:         assert(DFG.IsCode<NodeAttrs::Stmt>(IA));
   205:         if (RDefSR[IA.Id] != RDefSR_SA)
   206:           continue;
   207: 
   208:         MachineOperand &Op = UA.Addr->getOp();
   209:         if (Op.isTied())
   210:           continue;
   211:         if (trace()) {
   212:           dbgs() << "Can replace " << Print<RegisterRef>(DR, DFG)
   213:                  << " with " << Print<RegisterRef>(SR, DFG) << " in "
   214:                  << *NodeAddr<StmtNode*>(IA).Addr->getCode();
   215:         }
   216: 
   217:         MCRegister NewReg = MinPhysReg(SR);
   218:         Op.setReg(NewReg);
   219:         Op.setSubReg(0);
   220:         DFG.unlinkUse(UA, false);
   221:         if (RDefSR_SA != 0) {
   222:           UA.Addr->linkToDef(UA.Id, DFG.addr<DefNode*>(RDefSR_SA));
   223:         } else {
   224:           UA.Addr->setReachingDef(0);
   225:           UA.Addr->setSibling(0);
   226:         }
   227: 
   228:         Changed = true;
   229:   #ifndef NDEBUG
   230:         if (HasLimit && CpCount >= CpLimit)
   231:           break;
   232:         CpCount++;
   233:   #endif
   234: 
   235:         auto FC = CopyMap.find(IA.Id);
   236:         if (FC != CopyMap.end()) {
   237:           // Update the EM map in the copy's entry.
   238:           auto &M = FC->second;
   239:           for (auto &J : M) {
   240:             if (!PRI.equal_to(J.second, DR))
   241:               continue;
   242:             J.second = SR;
   243:             break;
   244:           }
   245:         }
   246:       } // for (N in reached-uses)
   247:     } // for (DA in defs)
   248:   } // for (C in Copies)
   249: 
   250:   return Changed;
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as getOwner, assert, getOp, dbgs, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 getOwner, assert, getOp, dbgs, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 251-251 / 第 251-251 行

```cpp
   251: }
```
- EN: This range contains executable implementation details for a Hexagon backend subsystem.
- CN: 这一段包含了某个 Hexagon 后端子系统的可执行实现细节。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述

## Dependencies / 依赖关系

- Direct includes / 直接包含: `RDFCopy.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, llvm/CodeGen/RDFGraph.h, llvm/CodeGen/RDFLiveness.h, llvm/CodeGen/RDFRegisters.h, llvm/CodeGen/TargetOpcodes.h, llvm/CodeGen/TargetRegisterInfo.h, llvm/MC/MCRegisterInfo.h, ... (16 total)`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
