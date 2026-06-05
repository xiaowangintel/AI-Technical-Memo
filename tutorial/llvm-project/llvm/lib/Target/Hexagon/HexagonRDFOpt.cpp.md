# HexagonRDFOpt.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonRDFOpt.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon register-dataflow-based optimization.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及寄存器模型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonRDFOpt.cpp --------------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "Hexagon.h"
    10: #include "HexagonInstrInfo.h"
    11: #include "HexagonSubtarget.h"
    12: #include "MCTargetDesc/HexagonBaseInfo.h"
    13: #include "RDFCopy.h"
    14: #include "RDFDeadCode.h"
    15: #include "llvm/ADT/DenseMap.h"
    16: #include "llvm/ADT/STLExtras.h"
    17: #include "llvm/ADT/SetVector.h"
    18: #include "llvm/CodeGen/MachineDominanceFrontier.h"
    19: #include "llvm/CodeGen/MachineDominators.h"
    20: #include "llvm/CodeGen/MachineFunction.h"
    21: #include "llvm/CodeGen/MachineFunctionPass.h"
    22: #include "llvm/CodeGen/MachineInstr.h"
    23: #include "llvm/CodeGen/MachineOperand.h"
    24: #include "llvm/CodeGen/MachineRegisterInfo.h"
    25: #include "llvm/CodeGen/RDFGraph.h"
    26: #include "llvm/CodeGen/RDFLiveness.h"
    27: #include "llvm/CodeGen/RDFRegisters.h"
    28: #include "llvm/InitializePasses.h"
    29: #include "llvm/Pass.h"
    30: #include "llvm/Support/CommandLine.h"
    31: #include "llvm/Support/Compiler.h"
    32: #include "llvm/Support/Debug.h"
    33: #include "llvm/Support/ErrorHandling.h"
    34: #include "llvm/Support/raw_ostream.h"
    35: #include <cassert>
    36: #include <limits>
    37: 
    38: using namespace llvm;
    39: using namespace rdf;
    40: 
    41: static unsigned RDFCount = 0;
    42: 
    43: static cl::opt<unsigned>
    44:     RDFLimit("hexagon-rdf-limit",
    45:              cl::init(std::numeric_limits<unsigned>::max()));
    46: 
    47: extern cl::opt<unsigned> RDFFuncBlockLimit;
    48: 
    49: static cl::opt<bool> RDFDump("hexagon-rdf-dump", cl::Hidden);
    50: static cl::opt<bool> RDFTrackReserved("hexagon-rdf-track-reserved", cl::Hidden);
```
- EN: It imports headers such as Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, MCTargetDesc/HexagonBaseInfo.h, ... (28 total), establishing the LLVM/Hexagon APIs used below. It opens namespaces (llvm, rdf) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as RDFLimit, RDFDump, RDFTrackReserved, translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior.
- CN: 这里引入了 Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, MCTargetDesc/HexagonBaseInfo.h, ... (28 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里打开了命名空间（llvm, rdf），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 RDFLimit, RDFDump, RDFTrackReserved 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。

### Lines 51-100 / 第 51-100 行

```cpp
    51: 
    52: namespace {
    53: 
    54:   class HexagonRDFOpt : public MachineFunctionPass {
    55:   public:
    56:     HexagonRDFOpt() : MachineFunctionPass(ID) {}
    57: 
    58:     void getAnalysisUsage(AnalysisUsage &AU) const override {
    59:       AU.addRequired<MachineDominatorTreeWrapperPass>();
    60:       AU.addRequired<MachineDominanceFrontierWrapperPass>();
    61:       AU.setPreservesAll();
    62:       MachineFunctionPass::getAnalysisUsage(AU);
    63:     }
    64: 
    65:     StringRef getPassName() const override {
    66:       return "Hexagon RDF optimizations";
    67:     }
    68: 
    69:     bool runOnMachineFunction(MachineFunction &MF) override;
    70: 
    71:     MachineFunctionProperties getRequiredProperties() const override {
    72:       return MachineFunctionProperties().setNoVRegs();
    73:     }
    74: 
    75:     static char ID;
    76: 
    77:   private:
    78:     MachineDominatorTree *MDT;
    79:     MachineRegisterInfo *MRI;
    80:   };
    81: 
    82: struct HexagonCP : public CopyPropagation {
    83:   HexagonCP(DataFlowGraph &G) : CopyPropagation(G) {}
    84: 
    85:   bool interpretAsCopy(const MachineInstr *MI, EqualityMap &EM) override;
    86: };
    87: 
    88: struct HexagonDCE : public DeadCodeElimination {
    89:   HexagonDCE(DataFlowGraph &G, MachineRegisterInfo &MRI)
    90:     : DeadCodeElimination(G, MRI) {}
    91: 
    92:   bool rewrite(NodeAddr<InstrNode*> IA, SetVector<NodeId> &Remove);
    93:   void removeOperand(NodeAddr<InstrNode*> IA, unsigned OpNum);
    94: 
    95:   bool run();
    96: };
    97: 
    98: } // end anonymous namespace
    99: 
   100: char HexagonRDFOpt::ID = 0;
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonRDFOpt, HexagonCP, HexagonDCE, which carry the state or API of this component. It defines declarative TableGen records like HexagonRDFOpt; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as HexagonRDFOpt, getAnalysisUsage, addRequired<MachineDominatorTreeWrapperPass>, addRequired<MachineDominanceFrontierWrapperPass>, ... (16 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonRDFOpt, HexagonCP, HexagonDCE 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonRDFOpt 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 HexagonRDFOpt, getAnalysisUsage, addRequired<MachineDominatorTreeWrapperPass>, addRequired<MachineDominanceFrontierWrapperPass>, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-150 / 第 101-150 行

```cpp
   101: 
   102: INITIALIZE_PASS_BEGIN(HexagonRDFOpt, "hexagon-rdf-opt",
   103:       "Hexagon RDF optimizations", false, false)
   104: INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
   105: INITIALIZE_PASS_DEPENDENCY(MachineDominanceFrontierWrapperPass)
   106: INITIALIZE_PASS_END(HexagonRDFOpt, "hexagon-rdf-opt",
   107:       "Hexagon RDF optimizations", false, false)
   108: 
   109: bool HexagonCP::interpretAsCopy(const MachineInstr *MI, EqualityMap &EM) {
   110:   auto mapRegs = [&EM] (RegisterRef DstR, RegisterRef SrcR) -> void {
   111:     EM.insert(std::make_pair(DstR, SrcR));
   112:   };
   113: 
   114:   DataFlowGraph &DFG = getDFG();
   115:   unsigned Opc = MI->getOpcode();
   116:   switch (Opc) {
   117:     case Hexagon::A2_combinew: {
   118:       const MachineOperand &DstOp = MI->getOperand(0);
   119:       const MachineOperand &HiOp = MI->getOperand(1);
   120:       const MachineOperand &LoOp = MI->getOperand(2);
   121:       assert(DstOp.getSubReg() == 0 && "Unexpected subregister");
   122:       mapRegs(DFG.makeRegRef(DstOp.getReg(), Hexagon::isub_hi),
   123:               DFG.makeRegRef(HiOp.getReg(),  HiOp.getSubReg()));
   124:       mapRegs(DFG.makeRegRef(DstOp.getReg(), Hexagon::isub_lo),
   125:               DFG.makeRegRef(LoOp.getReg(), LoOp.getSubReg()));
   126:       return true;
   127:     }
   128:     case Hexagon::A2_addi: {
   129:       const MachineOperand &A = MI->getOperand(2);
   130:       if (!A.isImm() || A.getImm() != 0)
   131:         return false;
   132:       [[fallthrough]];
   133:     }
   134:     case Hexagon::A2_tfr: {
   135:       const MachineOperand &DstOp = MI->getOperand(0);
   136:       const MachineOperand &SrcOp = MI->getOperand(1);
   137:       mapRegs(DFG.makeRegRef(DstOp.getReg(), DstOp.getSubReg()),
   138:               DFG.makeRegRef(SrcOp.getReg(), SrcOp.getSubReg()));
   139:       return true;
   140:     }
   141:   }
   142: 
   143:   return CopyPropagation::interpretAsCopy(MI, EM);
   144: }
   145: 
   146: bool HexagonDCE::run() {
   147:   bool Collected = collect();
   148:   if (!Collected)
   149:     return false;
   150: 
```
- EN: It declares or implements routines such as INITIALIZE_PASS_BEGIN, insert, getDFG, getOpcode, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonRDFOpt, HexagonCP, HexagonDCE, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 INITIALIZE_PASS_BEGIN, insert, getDFG, getOpcode, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonRDFOpt, HexagonCP, HexagonDCE，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   const SetVector<NodeId> &DeadNodes = getDeadNodes();
   152:   const SetVector<NodeId> &DeadInstrs = getDeadInstrs();
   153: 
   154:   using RefToInstrMap = DenseMap<NodeId, NodeId>;
   155: 
   156:   RefToInstrMap R2I;
   157:   SetVector<NodeId> PartlyDead;
   158:   DataFlowGraph &DFG = getDFG();
   159: 
   160:   for (NodeAddr<BlockNode*> BA : DFG.getFunc().Addr->members(DFG)) {
   161:     for (auto TA : BA.Addr->members_if(DFG.IsCode<NodeAttrs::Stmt>, DFG)) {
   162:       NodeAddr<StmtNode*> SA = TA;
   163:       for (NodeAddr<RefNode*> RA : SA.Addr->members(DFG)) {
   164:         R2I.insert(std::make_pair(RA.Id, SA.Id));
   165:         if (DFG.IsDef(RA) && DeadNodes.count(RA.Id))
   166:           if (!DeadInstrs.count(SA.Id))
   167:             PartlyDead.insert(SA.Id);
   168:       }
   169:     }
   170:   }
   171: 
   172:   // Nodes to remove.
   173:   SetVector<NodeId> Remove = DeadInstrs;
   174: 
   175:   bool Changed = false;
   176:   for (NodeId N : PartlyDead) {
   177:     auto SA = DFG.addr<StmtNode*>(N);
   178:     if (trace())
   179:       dbgs() << "Partly dead: " << *SA.Addr->getCode();
   180:     Changed |= rewrite(SA, Remove);
   181:   }
   182: 
   183:   return erase(Remove) || Changed;
   184: }
   185: 
   186: void HexagonDCE::removeOperand(NodeAddr<InstrNode*> IA, unsigned OpNum) {
   187:   MachineInstr *MI = NodeAddr<StmtNode*>(IA).Addr->getCode();
   188: 
   189:   auto getOpNum = [MI] (MachineOperand &Op) -> unsigned {
   190:     for (unsigned i = 0, n = MI->getNumOperands(); i != n; ++i)
   191:       if (&MI->getOperand(i) == &Op)
   192:         return i;
   193:     llvm_unreachable("Invalid operand");
   194:   };
   195:   DenseMap<NodeId,unsigned> OpMap;
   196:   DataFlowGraph &DFG = getDFG();
   197:   NodeList Refs = IA.Addr->members(DFG);
   198:   for (NodeAddr<RefNode*> RA : Refs)
   199:     OpMap.insert(std::make_pair(RA.Id, getOpNum(RA.Addr->getOp())));
   200: 
```
- EN: It declares or implements routines such as getDeadNodes, getDeadInstrs, getDFG, insert, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonDCE, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getDeadNodes, getDeadInstrs, getDFG, insert, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonDCE，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201:   MI->removeOperand(OpNum);
   202: 
   203:   for (NodeAddr<RefNode*> RA : Refs) {
   204:     unsigned N = OpMap[RA.Id];
   205:     if (N < OpNum)
   206:       RA.Addr->setRegRef(&MI->getOperand(N), DFG);
   207:     else if (N > OpNum)
   208:       RA.Addr->setRegRef(&MI->getOperand(N-1), DFG);
   209:   }
   210: }
   211: 
   212: bool HexagonDCE::rewrite(NodeAddr<InstrNode*> IA, SetVector<NodeId> &Remove) {
   213:   if (!getDFG().IsCode<NodeAttrs::Stmt>(IA))
   214:     return false;
   215:   DataFlowGraph &DFG = getDFG();
   216:   MachineInstr &MI = *NodeAddr<StmtNode*>(IA).Addr->getCode();
   217:   auto &HII = static_cast<const HexagonInstrInfo&>(DFG.getTII());
   218:   if (HII.getAddrMode(MI) != HexagonII::PostInc)
   219:     return false;
   220:   unsigned Opc = MI.getOpcode();
   221:   unsigned OpNum, NewOpc;
   222:   switch (Opc) {
   223:     case Hexagon::L2_loadri_pi:
   224:       NewOpc = Hexagon::L2_loadri_io;
   225:       OpNum = 1;
   226:       break;
   227:     case Hexagon::L2_loadrd_pi:
   228:       NewOpc = Hexagon::L2_loadrd_io;
   229:       OpNum = 1;
   230:       break;
   231:     case Hexagon::V6_vL32b_pi:
   232:       NewOpc = Hexagon::V6_vL32b_ai;
   233:       OpNum = 1;
   234:       break;
   235:     case Hexagon::S2_storeri_pi:
   236:       NewOpc = Hexagon::S2_storeri_io;
   237:       OpNum = 0;
   238:       break;
   239:     case Hexagon::S2_storerd_pi:
   240:       NewOpc = Hexagon::S2_storerd_io;
   241:       OpNum = 0;
   242:       break;
   243:     case Hexagon::V6_vS32b_pi:
   244:       NewOpc = Hexagon::V6_vS32b_ai;
   245:       OpNum = 0;
   246:       break;
   247:     default:
   248:       return false;
   249:   }
   250:   auto IsDead = [this] (NodeAddr<DefNode*> DA) -> bool {
```
- EN: It declares or implements routines such as removeOperand, HexagonDCE::rewrite, getDFG, getCode, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonDCE, HexagonInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 removeOperand, HexagonDCE::rewrite, getDFG, getCode, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonDCE, HexagonInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:     return getDeadNodes().count(DA.Id);
   252:   };
   253:   NodeList Defs;
   254:   MachineOperand &Op = MI.getOperand(OpNum);
   255:   for (NodeAddr<DefNode*> DA : IA.Addr->members_if(DFG.IsDef, DFG)) {
   256:     if (&DA.Addr->getOp() != &Op)
   257:       continue;
   258:     Defs = DFG.getRelatedRefs(IA, DA);
   259:     if (!llvm::all_of(Defs, IsDead))
   260:       return false;
   261:     break;
   262:   }
   263: 
   264:   // Mark all nodes in Defs for removal.
   265:   for (auto D : Defs)
   266:     Remove.insert(D.Id);
   267: 
   268:   if (trace())
   269:     dbgs() << "Rewriting: " << MI;
   270:   MI.setDesc(HII.get(NewOpc));
   271:   MI.getOperand(OpNum+2).setImm(0);
   272:   removeOperand(IA, OpNum);
   273:   if (trace())
   274:     dbgs() << "       to: " << MI;
   275: 
   276:   return true;
   277: }
   278: 
   279: bool HexagonRDFOpt::runOnMachineFunction(MachineFunction &MF) {
   280:   if (skipFunction(MF.getFunction()))
   281:     return false;
   282: 
   283:   // Perform RDF optimizations only if number of basic blocks in the
   284:   // function is less than the limit
   285:   if (MF.size() > RDFFuncBlockLimit) {
   286:     if (RDFDump)
   287:       dbgs() << "Skipping " << getPassName() << ": too many basic blocks\n";
   288:     return false;
   289:   }
   290: 
   291:   if (RDFLimit.getPosition()) {
   292:     if (RDFCount >= RDFLimit)
   293:       return false;
   294:     RDFCount++;
   295:   }
   296: 
   297:   MDT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
   298:   const auto &MDF = getAnalysis<MachineDominanceFrontierWrapperPass>().getMDF();
   299:   const auto &HII = *MF.getSubtarget<HexagonSubtarget>().getInstrInfo();
   300:   const auto &HRI = *MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
```
- EN: It declares or implements routines such as getDeadNodes, getOperand, getRelatedRefs, setDesc, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonRDFOpt, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getDeadNodes, getOperand, getRelatedRefs, setDesc, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonRDFOpt, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 301-350 / 第 301-350 行

```cpp
   301:   MRI = &MF.getRegInfo();
   302:   bool Changed;
   303: 
   304:   if (RDFDump)
   305:     MF.print(dbgs() << "Before " << getPassName() << "\n", nullptr);
   306: 
   307:   DataFlowGraph G(MF, HII, HRI, *MDT, MDF);
   308:   // Dead phi nodes are necessary for copy propagation: we can add a use
   309:   // of a register in a block where it would need a phi node, but which
   310:   // was dead (and removed) during the graph build time.
   311:   DataFlowGraph::Config Cfg;
   312:   Cfg.Options = RDFTrackReserved
   313:                     ? BuildOptions::KeepDeadPhis
   314:                     : BuildOptions::KeepDeadPhis | BuildOptions::OmitReserved;
   315:   G.build(Cfg);
   316: 
   317:   if (RDFDump)
   318:     dbgs() << "Starting copy propagation on: " << MF.getName() << '\n'
   319:            << PrintNode<FuncNode*>(G.getFunc(), G) << '\n';
   320:   HexagonCP CP(G);
   321:   CP.trace(RDFDump);
   322:   Changed = CP.run();
   323: 
   324:   if (RDFDump)
   325:     dbgs() << "Starting dead code elimination on: " << MF.getName() << '\n'
   326:            << PrintNode<FuncNode*>(G.getFunc(), G) << '\n';
   327:   HexagonDCE DCE(G, *MRI);
   328:   DCE.trace(RDFDump);
   329:   Changed |= DCE.run();
   330: 
   331:   if (Changed) {
   332:     if (RDFDump) {
   333:       dbgs() << "Starting liveness recomputation on: " << MF.getName() << '\n'
   334:              << PrintNode<FuncNode*>(G.getFunc(), G) << '\n';
   335:     }
   336:     Liveness LV(*MRI, G);
   337:     LV.trace(RDFDump);
   338:     LV.computeLiveIns();
   339:     LV.resetLiveIns();
   340:     LV.resetKills();
   341:   }
   342: 
   343:   if (RDFDump)
   344:     MF.print(dbgs() << "After " << getPassName() << "\n", nullptr);
   345: 
   346:   return false;
   347: }
   348: 
   349: FunctionPass *llvm::createHexagonRDFOpt() {
   350:   return new HexagonRDFOpt();
```
- EN: It declares or implements routines such as getRegInfo, G, build, CP, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCP, HexagonDCE, HexagonRDFOpt, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getRegInfo, G, build, CP, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCP, HexagonDCE, HexagonRDFOpt，说明了它与同级后端组件的连接关系。

### Lines 351-351 / 第 351-351 行

```cpp
   351: }
```
- EN: This range contains executable implementation details for a Hexagon backend subsystem.
- CN: 这一段包含了某个 Hexagon 后端子系统的可执行实现细节。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, MCTargetDesc/HexagonBaseInfo.h, RDFCopy.h, RDFDeadCode.h, llvm/ADT/DenseMap.h, llvm/ADT/STLExtras.h, llvm/ADT/SetVector.h, llvm/CodeGen/MachineDominanceFrontier.h, ... (28 total)`
- Hexagon symbols / Hexagon 符号: `HexagonRDFOpt, HexagonInstrInfo, HexagonSubtarget, HexagonBaseInfo, HexagonCP, HexagonDCE, HexagonII`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
