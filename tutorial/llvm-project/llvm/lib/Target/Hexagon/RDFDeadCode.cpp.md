# RDFDeadCode.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/RDFDeadCode.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements a Hexagon backend component.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===--- RDFDeadCode.cpp --------------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // RDF-based generic dead code elimination.
    10: 
    11: #include "RDFDeadCode.h"
    12: 
    13: #include "llvm/ADT/SetVector.h"
    14: #include "llvm/CodeGen/MachineBasicBlock.h"
    15: #include "llvm/CodeGen/MachineFunction.h"
    16: #include "llvm/CodeGen/MachineRegisterInfo.h"
    17: #include "llvm/CodeGen/RDFGraph.h"
    18: #include "llvm/CodeGen/RDFLiveness.h"
    19: #include "llvm/Support/Debug.h"
    20: 
    21: #include <queue>
    22: 
    23: using namespace llvm;
    24: using namespace rdf;
    25: 
    26: // This drastically improves execution time in "collect" over using
    27: // SetVector as a work queue, and popping the first element from it.
    28: template<typename T> struct DeadCodeElimination::SetQueue {
    29:   SetQueue() : Set(), Queue() {}
    30: 
    31:   bool empty() const {
    32:     return Queue.empty();
    33:   }
    34:   T pop_front() {
    35:     T V = Queue.front();
    36:     Queue.pop();
    37:     Set.erase(V);
    38:     return V;
    39:   }
    40:   void push_back(T V) {
    41:     if (Set.count(V))
    42:       return;
    43:     Queue.push(V);
    44:     Set.insert(V);
    45:   }
    46: 
    47: private:
    48:   DenseSet<T> Set;
    49:   std::queue<T> Queue;
    50: };
```
- EN: It imports headers such as RDFDeadCode.h, llvm/ADT/SetVector.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, ... (9 total), establishing the LLVM/Hexagon APIs used below. It opens namespaces (llvm, rdf) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as SetQueue, empty, pop_front, front, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里引入了 RDFDeadCode.h, llvm/ADT/SetVector.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, ... (9 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里打开了命名空间（llvm, rdf），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 SetQueue, empty, pop_front, front, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 51-100 / 第 51-100 行

```cpp
    51: 
    52: 
    53: // Check if the given instruction has observable side-effects, i.e. if
    54: // it should be considered "live". It is safe for this function to be
    55: // overly conservative (i.e. return "true" for all instructions), but it
    56: // is not safe to return "false" for an instruction that should not be
    57: // considered removable.
    58: bool DeadCodeElimination::isLiveInstr(NodeAddr<StmtNode *> S) const {
    59:   const MachineInstr *MI = S.Addr->getCode();
    60:   if (MI->mayStore() || MI->isBranch() || MI->isCall() || MI->isReturn())
    61:     return true;
    62:   if (MI->hasOrderedMemoryRef() || MI->hasUnmodeledSideEffects() ||
    63:       MI->isPosition())
    64:     return true;
    65:   if (MI->isPHI())
    66:     return false;
    67:   for (auto &Op : MI->operands()) {
    68:     if (Op.isReg() && MRI.isReserved(Op.getReg()))
    69:       return true;
    70:     if (Op.isRegMask()) {
    71:       const uint32_t *BM = Op.getRegMask();
    72:       for (unsigned R = 0, RN = DFG.getTRI().getNumRegs(); R != RN; ++R) {
    73:         if (BM[R/32] & (1u << (R%32)))
    74:           continue;
    75:         if (MRI.isReserved(R))
    76:           return true;
    77:       }
    78:     }
    79:   }
    80:   return false;
    81: }
    82: 
    83: void DeadCodeElimination::scanInstr(NodeAddr<InstrNode*> IA,
    84:       SetQueue<NodeId> &WorkQ) {
    85:   if (!DFG.IsCode<NodeAttrs::Stmt>(IA))
    86:     return;
    87:   if (!isLiveInstr(IA))
    88:     return;
    89:   for (NodeAddr<RefNode*> RA : IA.Addr->members(DFG)) {
    90:     if (!LiveNodes.count(RA.Id))
    91:       WorkQ.push_back(RA.Id);
    92:   }
    93: }
    94: 
    95: void DeadCodeElimination::processDef(NodeAddr<DefNode*> DA,
    96:       SetQueue<NodeId> &WorkQ) {
    97:   NodeAddr<InstrNode*> IA = DA.Addr->getOwner(DFG);
    98:   for (NodeAddr<UseNode*> UA : IA.Addr->members_if(DFG.IsUse, DFG)) {
    99:     if (!LiveNodes.count(UA.Id))
   100:       WorkQ.push_back(UA.Id);
```
- EN: It declares or implements routines such as conservative, getCode, getRegMask, DeadCodeElimination::scanInstr, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 conservative, getCode, getRegMask, DeadCodeElimination::scanInstr, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 101-150 / 第 101-150 行

```cpp
   101:   }
   102:   for (NodeAddr<DefNode*> TA : DFG.getRelatedRefs(IA, DA))
   103:     LiveNodes.insert(TA.Id);
   104: }
   105: 
   106: void DeadCodeElimination::processUse(NodeAddr<UseNode*> UA,
   107:       SetQueue<NodeId> &WorkQ) {
   108:   for (NodeAddr<DefNode*> DA : LV.getAllReachingDefs(UA)) {
   109:     if (!LiveNodes.count(DA.Id))
   110:       WorkQ.push_back(DA.Id);
   111:   }
   112: }
   113: 
   114: // Traverse the DFG and collect the set dead RefNodes and the set of
   115: // dead instructions. Return "true" if any of these sets is non-empty,
   116: // "false" otherwise.
   117: bool DeadCodeElimination::collect() {
   118:   // This function works by first finding all live nodes. The dead nodes
   119:   // are then the complement of the set of live nodes.
   120:   //
   121:   // Assume that all nodes are dead. Identify instructions which must be
   122:   // considered live, i.e. instructions with observable side-effects, such
   123:   // as calls and stores. All arguments of such instructions are considered
   124:   // live. For each live def, all operands used in the corresponding
   125:   // instruction are considered live. For each live use, all its reaching
   126:   // defs are considered live.
   127:   LiveNodes.clear();
   128:   SetQueue<NodeId> WorkQ;
   129:   for (NodeAddr<BlockNode*> BA : DFG.getFunc().Addr->members(DFG))
   130:     for (NodeAddr<InstrNode*> IA : BA.Addr->members(DFG))
   131:       scanInstr(IA, WorkQ);
   132: 
   133:   while (!WorkQ.empty()) {
   134:     NodeId N = WorkQ.pop_front();
   135:     LiveNodes.insert(N);
   136:     auto RA = DFG.addr<RefNode*>(N);
   137:     if (DFG.IsDef(RA))
   138:       processDef(RA, WorkQ);
   139:     else
   140:       processUse(RA, WorkQ);
   141:   }
   142: 
   143:   if (trace()) {
   144:     dbgs() << "Live nodes:\n";
   145:     for (NodeId N : LiveNodes) {
   146:       auto RA = DFG.addr<RefNode*>(N);
   147:       dbgs() << PrintNode<RefNode*>(RA, DFG) << "\n";
   148:     }
   149:   }
   150: 
```
- EN: It declares or implements routines such as DeadCodeElimination::processUse, DeadCodeElimination::collect, clear, pop_front, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 DeadCodeElimination::processUse, DeadCodeElimination::collect, clear, pop_front, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   auto IsDead = [this] (NodeAddr<InstrNode*> IA) -> bool {
   152:     for (NodeAddr<DefNode*> DA : IA.Addr->members_if(DFG.IsDef, DFG))
   153:       if (LiveNodes.count(DA.Id))
   154:         return false;
   155:     return true;
   156:   };
   157: 
   158:   for (NodeAddr<BlockNode*> BA : DFG.getFunc().Addr->members(DFG)) {
   159:     for (NodeAddr<InstrNode*> IA : BA.Addr->members(DFG)) {
   160:       for (NodeAddr<RefNode*> RA : IA.Addr->members(DFG))
   161:         if (!LiveNodes.count(RA.Id))
   162:           DeadNodes.insert(RA.Id);
   163:       if (DFG.IsCode<NodeAttrs::Stmt>(IA))
   164:         if (isLiveInstr(IA) || DFG.hasUntrackedRef(IA))
   165:           continue;
   166:       if (IsDead(IA)) {
   167:         DeadInstrs.insert(IA.Id);
   168:         if (trace())
   169:           dbgs() << "Dead instr: " << PrintNode<InstrNode*>(IA, DFG) << "\n";
   170:       }
   171:     }
   172:   }
   173: 
   174:   return !DeadNodes.empty();
   175: }
   176: 
   177: // Erase the nodes given in the Nodes set from DFG. In addition to removing
   178: // them from the DFG, if a node corresponds to a statement, the corresponding
   179: // machine instruction is erased from the function.
   180: bool DeadCodeElimination::erase(const SetVector<NodeId> &Nodes) {
   181:   if (Nodes.empty())
   182:     return false;
   183: 
   184:   // Prepare the actual set of ref nodes to remove: ref nodes from Nodes
   185:   // are included directly, for each InstrNode in Nodes, include the set
   186:   // of all RefNodes from it.
   187:   NodeList DRNs, DINs;
   188:   for (auto I : Nodes) {
   189:     auto BA = DFG.addr<NodeBase*>(I);
   190:     uint16_t Type = BA.Addr->getType();
   191:     if (Type == NodeAttrs::Ref) {
   192:       DRNs.push_back(DFG.addr<RefNode*>(I));
   193:       continue;
   194:     }
   195: 
   196:     // If it's a code node, add all ref nodes from it.
   197:     uint16_t Kind = BA.Addr->getKind();
   198:     if (Kind == NodeAttrs::Stmt || Kind == NodeAttrs::Phi) {
   199:       append_range(DRNs, NodeAddr<CodeNode*>(BA).Addr->members(DFG));
   200:       DINs.push_back(DFG.addr<InstrNode*>(I));
```
- EN: It declares or implements routines such as insert, empty, DeadCodeElimination::erase, getType, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 insert, empty, DeadCodeElimination::erase, getType, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-243 / 第 201-243 行

```cpp
   201:     } else {
   202:       llvm_unreachable("Unexpected code node");
   203:       return false;
   204:     }
   205:   }
   206: 
   207:   // Sort the list so that use nodes are removed first. This makes the
   208:   // "unlink" functions a bit faster.
   209:   auto UsesFirst = [] (NodeAddr<RefNode*> A, NodeAddr<RefNode*> B) -> bool {
   210:     uint16_t KindA = A.Addr->getKind(), KindB = B.Addr->getKind();
   211:     if (KindA == NodeAttrs::Use && KindB == NodeAttrs::Def)
   212:       return true;
   213:     if (KindA == NodeAttrs::Def && KindB == NodeAttrs::Use)
   214:       return false;
   215:     return A.Id < B.Id;
   216:   };
   217:   llvm::sort(DRNs, UsesFirst);
   218: 
   219:   if (trace())
   220:     dbgs() << "Removing dead ref nodes:\n";
   221:   for (NodeAddr<RefNode*> RA : DRNs) {
   222:     if (trace())
   223:       dbgs() << "  " << PrintNode<RefNode*>(RA, DFG) << '\n';
   224:     if (DFG.IsUse(RA))
   225:       DFG.unlinkUse(RA, true);
   226:     else if (DFG.IsDef(RA))
   227:       DFG.unlinkDef(RA, true);
   228:   }
   229: 
   230:   // Now, remove all dead instruction nodes.
   231:   for (NodeAddr<InstrNode*> IA : DINs) {
   232:     NodeAddr<BlockNode*> BA = IA.Addr->getOwner(DFG);
   233:     BA.Addr->removeMember(IA, DFG);
   234:     if (!DFG.IsCode<NodeAttrs::Stmt>(IA))
   235:       continue;
   236: 
   237:     MachineInstr *MI = NodeAddr<StmtNode*>(IA).Addr->getCode();
   238:     if (trace())
   239:       dbgs() << "erasing: " << *MI;
   240:     MI->eraseFromParent();
   241:   }
   242:   return true;
   243: }
```
- EN: It declares or implements routines such as llvm_unreachable, getKind, llvm::sort, getOwner, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 llvm_unreachable, getKind, llvm::sort, getOwner, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义

## Dependencies / 依赖关系

- Direct includes / 直接包含: `RDFDeadCode.h, llvm/ADT/SetVector.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/CodeGen/RDFGraph.h, llvm/CodeGen/RDFLiveness.h, llvm/Support/Debug.h, queue`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
