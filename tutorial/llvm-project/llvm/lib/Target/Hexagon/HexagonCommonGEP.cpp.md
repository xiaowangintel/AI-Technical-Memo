# HexagonCommonGEP.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonCommonGEP.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon common GEP simplification and reuse.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- HexagonCommonGEP.cpp -----------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "Hexagon.h"
    10: 
    11: #include "llvm/ADT/ArrayRef.h"
    12: #include "llvm/ADT/FoldingSet.h"
    13: #include "llvm/ADT/GraphTraits.h"
    14: #include "llvm/ADT/STLExtras.h"
    15: #include "llvm/ADT/SetVector.h"
    16: #include "llvm/ADT/SmallVector.h"
    17: #include "llvm/ADT/StringRef.h"
    18: #include "llvm/Analysis/LoopInfo.h"
    19: #include "llvm/Analysis/PostDominators.h"
    20: #include "llvm/IR/BasicBlock.h"
    21: #include "llvm/IR/Constant.h"
    22: #include "llvm/IR/Constants.h"
    23: #include "llvm/IR/DerivedTypes.h"
    24: #include "llvm/IR/Dominators.h"
    25: #include "llvm/IR/Function.h"
    26: #include "llvm/IR/Instruction.h"
    27: #include "llvm/IR/Instructions.h"
    28: #include "llvm/IR/Type.h"
    29: #include "llvm/IR/Use.h"
    30: #include "llvm/IR/User.h"
    31: #include "llvm/IR/Value.h"
    32: #include "llvm/IR/Verifier.h"
    33: #include "llvm/InitializePasses.h"
    34: #include "llvm/Pass.h"
    35: #include "llvm/Support/Allocator.h"
    36: #include "llvm/Support/Casting.h"
    37: #include "llvm/Support/CommandLine.h"
    38: #include "llvm/Support/Compiler.h"
    39: #include "llvm/Support/Debug.h"
    40: #include "llvm/Support/raw_ostream.h"
    41: #include "llvm/Transforms/Utils/Local.h"
    42: #include <cassert>
    43: #include <cstddef>
    44: #include <cstdint>
    45: #include <iterator>
    46: #include <map>
    47: #include <set>
    48: #include <utility>
    49: #include <vector>
    50: 
    51: #define DEBUG_TYPE "commgep"
    52: 
    53: using namespace llvm;
    54: 
    55: static cl::opt<bool> OptSpeculate("commgep-speculate", cl::init(true),
    56:                                   cl::Hidden);
    57: 
    58: static cl::opt<bool> OptEnableInv("commgep-inv", cl::init(true), cl::Hidden);
    59: 
    60: static cl::opt<bool> OptEnableConst("commgep-const", cl::init(true),
    61:                                     cl::Hidden);
    62: 
    63: namespace {
    64: 
    65:   struct GepNode;
    66:   using NodeSet = std::set<GepNode *>;
    67:   using NodeToValueMap = std::map<GepNode *, Value *>;
    68:   using NodeVect = std::vector<GepNode *>;
    69:   using NodeChildrenMap = std::map<GepNode *, NodeVect>;
    70:   using UseSet = SetVector<Use *>;
    71:   using NodeToUsesMap = std::map<GepNode *, UseSet>;
    72: 
    73:   // Numbering map for gep nodes. Used to keep track of ordering for
    74:   // gep nodes.
    75:   struct NodeOrdering {
    76:     NodeOrdering() = default;
    77: 
    78:     void insert(const GepNode *N) { Map.insert(std::make_pair(N, ++LastNum)); }
    79:     void clear() { Map.clear(); }
    80: 
    81:     bool operator()(const GepNode *N1, const GepNode *N2) const {
    82:       auto F1 = Map.find(N1), F2 = Map.find(N2);
    83:       assert(F1 != Map.end() && F2 != Map.end());
    84:       return F1->second < F2->second;
    85:     }
    86: 
    87:   private:
    88:     std::map<const GepNode *, unsigned> Map;
    89:     unsigned LastNum = 0;
    90:   };
    91: 
    92:   class HexagonCommonGEP : public FunctionPass {
    93:   public:
    94:     static char ID;
    95: 
    96:     HexagonCommonGEP() : FunctionPass(ID) {}
    97: 
    98:     bool runOnFunction(Function &F) override;
    99:     StringRef getPassName() const override { return "Hexagon Common GEP"; }
   100: 
```
- EN: It imports headers such as Hexagon.h, llvm/ADT/ArrayRef.h, llvm/ADT/FoldingSet.h, llvm/ADT/GraphTraits.h, ... (40 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as GepNode, NodeOrdering, HexagonCommonGEP, which carry the state or API of this component.
- CN: 这里引入了 Hexagon.h, llvm/ADT/ArrayRef.h, llvm/ADT/FoldingSet.h, llvm/ADT/GraphTraits.h, ... (40 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 GepNode, NodeOrdering, HexagonCommonGEP 等类型，用来承载该组件的状态或接口。

### Lines 101-200 / 第 101-200 行

```cpp
   101:     void getAnalysisUsage(AnalysisUsage &AU) const override {
   102:       AU.addRequired<DominatorTreeWrapperPass>();
   103:       AU.addPreserved<DominatorTreeWrapperPass>();
   104:       AU.addRequired<PostDominatorTreeWrapperPass>();
   105:       AU.addPreserved<PostDominatorTreeWrapperPass>();
   106:       AU.addRequired<LoopInfoWrapperPass>();
   107:       AU.addPreserved<LoopInfoWrapperPass>();
   108:       FunctionPass::getAnalysisUsage(AU);
   109:     }
   110: 
   111:   private:
   112:     using ValueToNodeMap = std::map<Value *, GepNode *>;
   113:     using ValueVect = std::vector<Value *>;
   114:     using NodeToValuesMap = std::map<GepNode *, ValueVect>;
   115: 
   116:     void getBlockTraversalOrder(BasicBlock *Root, ValueVect &Order);
   117:     bool isHandledGepForm(GetElementPtrInst *GepI);
   118:     void processGepInst(GetElementPtrInst *GepI, ValueToNodeMap &NM);
   119:     void collect();
   120:     void common();
   121: 
   122:     BasicBlock *recalculatePlacement(GepNode *Node, NodeChildrenMap &NCM,
   123:                                      NodeToValueMap &Loc);
   124:     BasicBlock *recalculatePlacementRec(GepNode *Node, NodeChildrenMap &NCM,
   125:                                         NodeToValueMap &Loc);
   126:     bool isInvariantIn(Value *Val, Loop *L);
   127:     bool isInvariantIn(GepNode *Node, Loop *L);
   128:     bool isInMainPath(BasicBlock *B, Loop *L);
   129:     BasicBlock *adjustForInvariance(GepNode *Node, NodeChildrenMap &NCM,
   130:                                     NodeToValueMap &Loc);
   131:     void separateChainForNode(GepNode *Node, Use *U, NodeToValueMap &Loc);
   132:     void separateConstantChains(GepNode *Node, NodeChildrenMap &NCM,
   133:                                 NodeToValueMap &Loc);
   134:     void computeNodePlacement(NodeToValueMap &Loc);
   135: 
   136:     Value *fabricateGEP(NodeVect &NA, BasicBlock::iterator At,
   137:                         BasicBlock *LocB);
   138:     void getAllUsersForNode(GepNode *Node, ValueVect &Values,
   139:                             NodeChildrenMap &NCM);
   140:     void materialize(NodeToValueMap &Loc);
   141: 
   142:     void removeDeadCode();
   143: 
   144:     NodeVect Nodes;
   145:     NodeToUsesMap Uses;
   146:     NodeOrdering NodeOrder;   // Node ordering, for deterministic behavior.
   147:     SpecificBumpPtrAllocator<GepNode> *Mem;
   148:     LLVMContext *Ctx;
   149:     LoopInfo *LI;
   150:     DominatorTree *DT;
   151:     PostDominatorTree *PDT;
   152:     Function *Fn;
   153:   };
   154: 
   155: } // end anonymous namespace
   156: 
   157: char HexagonCommonGEP::ID = 0;
   158: 
   159: INITIALIZE_PASS_BEGIN(HexagonCommonGEP, "hcommgep", "Hexagon Common GEP",
   160:       false, false)
   161: INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
   162: INITIALIZE_PASS_DEPENDENCY(PostDominatorTreeWrapperPass)
   163: INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass)
   164: INITIALIZE_PASS_END(HexagonCommonGEP, "hcommgep", "Hexagon Common GEP",
   165:       false, false)
   166: 
   167: namespace {
   168: 
   169:   struct GepNode {
   170:     enum {
   171:       None      = 0,
   172:       Root      = 0x01,
   173:       Internal  = 0x02,
   174:       Used      = 0x04,
   175:       InBounds  = 0x08,
   176:       Pointer   = 0x10,   // See note below.
   177:     };
   178:     // Note: GEP indices generally traverse nested types, and so a GepNode
   179:     // (representing a single index) can be associated with some composite
   180:     // type. The exception is the GEP input, which is a pointer, and not
   181:     // a composite type (at least not in the sense of having sub-types).
   182:     // Also, the corresponding index plays a different role as well: it is
   183:     // simply added to the input pointer. Since pointer types are becoming
   184:     // opaque (i.e. are no longer going to include the pointee type), the
   185:     // two pieces of information (1) the fact that it's a pointer, and
   186:     // (2) the pointee type, need to be stored separately. The pointee type
   187:     // will be stored in the PTy member, while the fact that the node
   188:     // operates on a pointer will be reflected by the flag "Pointer".
   189: 
   190:     uint32_t Flags = 0;
   191:     union {
   192:       GepNode *Parent;
   193:       Value *BaseVal;
   194:     };
   195:     Value *Idx = nullptr;
   196:     Type *PTy = nullptr;    // Type indexed by this node. For pointer nodes
   197:                             // this is the "pointee" type, and indexing a
   198:                             // pointer does not change the type.
   199: 
   200:     GepNode() : Parent(nullptr) {}
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as GepNode, which carry the state or API of this component. It declares or implements routines such as getAnalysisUsage, addRequired<DominatorTreeWrapperPass>, addPreserved<DominatorTreeWrapperPass>, addRequired<PostDominatorTreeWrapperPass>, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 GepNode 等类型，用来承载该组件的状态或接口。 这里声明或实现了 getAnalysisUsage, addRequired<DominatorTreeWrapperPass>, addPreserved<DominatorTreeWrapperPass>, addRequired<PostDominatorTreeWrapperPass>, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-300 / 第 201-300 行

```cpp
   201:     GepNode(const GepNode *N) : Flags(N->Flags), Idx(N->Idx), PTy(N->PTy) {
   202:       if (Flags & Root)
   203:         BaseVal = N->BaseVal;
   204:       else
   205:         Parent = N->Parent;
   206:     }
   207: 
   208:     friend raw_ostream &operator<< (raw_ostream &OS, const GepNode &GN);
   209:   };
   210: 
   211:   raw_ostream &operator<< (raw_ostream &OS, const GepNode &GN) {
   212:     OS << "{ {";
   213:     bool Comma = false;
   214:     if (GN.Flags & GepNode::Root) {
   215:       OS << "root";
   216:       Comma = true;
   217:     }
   218:     if (GN.Flags & GepNode::Internal) {
   219:       if (Comma)
   220:         OS << ',';
   221:       OS << "internal";
   222:       Comma = true;
   223:     }
   224:     if (GN.Flags & GepNode::Used) {
   225:       if (Comma)
   226:         OS << ',';
   227:       OS << "used";
   228:     }
   229:     if (GN.Flags & GepNode::InBounds) {
   230:       if (Comma)
   231:         OS << ',';
   232:       OS << "inbounds";
   233:     }
   234:     if (GN.Flags & GepNode::Pointer) {
   235:       if (Comma)
   236:         OS << ',';
   237:       OS << "pointer";
   238:     }
   239:     OS << "} ";
   240:     if (GN.Flags & GepNode::Root)
   241:       OS << "BaseVal:" << GN.BaseVal->getName() << '(' << GN.BaseVal << ')';
   242:     else
   243:       OS << "Parent:" << GN.Parent;
   244: 
   245:     OS << " Idx:";
   246:     if (ConstantInt *CI = dyn_cast<ConstantInt>(GN.Idx))
   247:       OS << CI->getValue().getSExtValue();
   248:     else if (GN.Idx->hasName())
   249:       OS << GN.Idx->getName();
   250:     else
   251:       OS << "<anon> =" << *GN.Idx;
   252: 
   253:     OS << " PTy:";
   254:     if (GN.PTy->isStructTy()) {
   255:       StructType *STy = cast<StructType>(GN.PTy);
   256:       if (!STy->isLiteral())
   257:         OS << GN.PTy->getStructName();
   258:       else
   259:         OS << "<anon-struct>:" << *STy;
   260:     }
   261:     else
   262:       OS << *GN.PTy;
   263:     OS << " }";
   264:     return OS;
   265:   }
   266: 
   267:   template <typename NodeContainer>
   268:   void dump_node_container(raw_ostream &OS, const NodeContainer &S) {
   269:     using const_iterator = typename NodeContainer::const_iterator;
   270: 
   271:     for (const_iterator I = S.begin(), E = S.end(); I != E; ++I)
   272:       OS << *I << ' ' << **I << '\n';
   273:   }
   274: 
   275:   [[maybe_unused]] raw_ostream &operator<<(raw_ostream &OS, const NodeVect &S);
   276:   raw_ostream &operator<< (raw_ostream &OS, const NodeVect &S) {
   277:     dump_node_container(OS, S);
   278:     return OS;
   279:   }
   280: 
   281:   [[maybe_unused]] raw_ostream &operator<<(raw_ostream &OS,
   282:                                            const NodeToUsesMap &M);
   283:   raw_ostream &operator<< (raw_ostream &OS, const NodeToUsesMap &M){
   284:     for (const auto &I : M) {
   285:       const UseSet &Us = I.second;
   286:       OS << I.first << " -> #" << Us.size() << '{';
   287:       for (const Use *U : Us) {
   288:         User *R = U->getUser();
   289:         if (R->hasName())
   290:           OS << ' ' << R->getName();
   291:         else
   292:           OS << " <?>(" << *R << ')';
   293:       }
   294:       OS << " }\n";
   295:     }
   296:     return OS;
   297:   }
   298: 
   299:   struct in_set {
   300:     in_set(const NodeSet &S) : NS(S) {}
```
- EN: It declares types such as in_set, which carry the state or API of this component. It declares or implements routines such as GepNode, cast<StructType>, dump_node_container, getUser, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 in_set 等类型，用来承载该组件的状态或接口。 这里声明或实现了 GepNode, cast<StructType>, dump_node_container, getUser, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-400 / 第 301-400 行

```cpp
   301: 
   302:     bool operator() (GepNode *N) const {
   303:       return NS.find(N) != NS.end();
   304:     }
   305: 
   306:   private:
   307:     const NodeSet &NS;
   308:   };
   309: 
   310: } // end anonymous namespace
   311: 
   312: inline void *operator new(size_t, SpecificBumpPtrAllocator<GepNode> &A) {
   313:   return A.Allocate();
   314: }
   315: 
   316: void HexagonCommonGEP::getBlockTraversalOrder(BasicBlock *Root,
   317:       ValueVect &Order) {
   318:   // Compute block ordering for a typical DT-based traversal of the flow
   319:   // graph: "before visiting a block, all of its dominators must have been
   320:   // visited".
   321: 
   322:   Order.push_back(Root);
   323:   for (auto *DTN : children<DomTreeNode*>(DT->getNode(Root)))
   324:     getBlockTraversalOrder(DTN->getBlock(), Order);
   325: }
   326: 
   327: bool HexagonCommonGEP::isHandledGepForm(GetElementPtrInst *GepI) {
   328:   // No vector GEPs.
   329:   if (!GepI->getType()->isPointerTy())
   330:     return false;
   331:   // No GEPs without any indices.  (Is this possible?)
   332:   if (GepI->indices().empty())
   333:     return false;
   334:   return true;
   335: }
   336: 
   337: void HexagonCommonGEP::processGepInst(GetElementPtrInst *GepI,
   338:       ValueToNodeMap &NM) {
   339:   LLVM_DEBUG(dbgs() << "Visiting GEP: " << *GepI << '\n');
   340:   GepNode *N = new (*Mem) GepNode;
   341:   Value *PtrOp = GepI->getPointerOperand();
   342:   uint32_t InBounds = GepI->isInBounds() ? GepNode::InBounds : 0;
   343:   ValueToNodeMap::iterator F = NM.find(PtrOp);
   344:   if (F == NM.end()) {
   345:     N->BaseVal = PtrOp;
   346:     N->Flags |= GepNode::Root | InBounds;
   347:   } else {
   348:     // If PtrOp was a GEP instruction, it must have already been processed.
   349:     // The ValueToNodeMap entry for it is the last gep node in the generated
   350:     // chain. Link to it here.
   351:     N->Parent = F->second;
   352:   }
   353:   N->PTy = GepI->getSourceElementType();
   354:   N->Flags |= GepNode::Pointer;
   355:   N->Idx = *GepI->idx_begin();
   356: 
   357:   // Collect the list of users of this GEP instruction. Will add it to the
   358:   // last node created for it.
   359:   UseSet Us;
   360:   for (Value::user_iterator UI = GepI->user_begin(), UE = GepI->user_end();
   361:        UI != UE; ++UI) {
   362:     // Check if this gep is used by anything other than other geps that
   363:     // we will process.
   364:     if (isa<GetElementPtrInst>(*UI)) {
   365:       GetElementPtrInst *UserG = cast<GetElementPtrInst>(*UI);
   366:       if (isHandledGepForm(UserG))
   367:         continue;
   368:     }
   369:     Us.insert(&UI.getUse());
   370:   }
   371:   Nodes.push_back(N);
   372:   NodeOrder.insert(N);
   373: 
   374:   // Skip the first index operand, since it was already handled above. This
   375:   // dereferences the pointer operand.
   376:   GepNode *PN = N;
   377:   Type *PtrTy = GepI->getSourceElementType();
   378:   for (Use &U : llvm::drop_begin(GepI->indices())) {
   379:     Value *Op = U;
   380:     GepNode *Nx = new (*Mem) GepNode;
   381:     Nx->Parent = PN;  // Link Nx to the previous node.
   382:     Nx->Flags |= GepNode::Internal | InBounds;
   383:     Nx->PTy = PtrTy;
   384:     Nx->Idx = Op;
   385:     Nodes.push_back(Nx);
   386:     NodeOrder.insert(Nx);
   387:     PN = Nx;
   388: 
   389:     PtrTy = GetElementPtrInst::getTypeAtIndex(PtrTy, Op);
   390:   }
   391: 
   392:   // After last node has been created, update the use information.
   393:   if (!Us.empty()) {
   394:     PN->Flags |= GepNode::Used;
   395:     Uses[PN].insert_range(Us);
   396:   }
   397: 
   398:   // Link the last node with the originating GEP instruction. This is to
   399:   // help with linking chained GEP instructions.
   400:   NM.insert(std::make_pair(GepI, PN));
```
- EN: It opens namespaces (inline) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as find, new, Allocate, HexagonCommonGEP::getBlockTraversalOrder, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCommonGEP, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（inline），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 find, new, Allocate, HexagonCommonGEP::getBlockTraversalOrder, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCommonGEP，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401: }
   402: 
   403: void HexagonCommonGEP::collect() {
   404:   // Establish depth-first traversal order of the dominator tree.
   405:   ValueVect BO;
   406:   getBlockTraversalOrder(&Fn->front(), BO);
   407: 
   408:   // The creation of gep nodes requires DT-traversal. When processing a GEP
   409:   // instruction that uses another GEP instruction as the base pointer, the
   410:   // gep node for the base pointer should already exist.
   411:   ValueToNodeMap NM;
   412:   for (Value *I : BO) {
   413:     BasicBlock *B = cast<BasicBlock>(I);
   414:     for (Instruction &J : *B)
   415:       if (auto *GepI = dyn_cast<GetElementPtrInst>(&J))
   416:         if (isHandledGepForm(GepI))
   417:           processGepInst(GepI, NM);
   418:   }
   419: 
   420:   LLVM_DEBUG(dbgs() << "Gep nodes after initial collection:\n" << Nodes);
   421: }
   422: 
   423: static void invert_find_roots(const NodeVect &Nodes, NodeChildrenMap &NCM,
   424:                               NodeVect &Roots) {
   425:   for (GepNode *N : Nodes) {
   426:     if (N->Flags & GepNode::Root) {
   427:       Roots.push_back(N);
   428:       continue;
   429:     }
   430:     GepNode *PN = N->Parent;
   431:     NCM[PN].push_back(N);
   432:   }
   433: }
   434: 
   435: static void nodes_for_root(GepNode *Root, NodeChildrenMap &NCM,
   436:                            NodeSet &Nodes) {
   437:     NodeVect Work;
   438:     Work.push_back(Root);
   439:     Nodes.insert(Root);
   440: 
   441:     while (!Work.empty()) {
   442:       NodeVect::iterator First = Work.begin();
   443:       GepNode *N = *First;
   444:       Work.erase(First);
   445:       NodeChildrenMap::iterator CF = NCM.find(N);
   446:       if (CF != NCM.end()) {
   447:         llvm::append_range(Work, CF->second);
   448:         Nodes.insert(CF->second.begin(), CF->second.end());
   449:       }
   450:     }
   451: }
   452: 
   453: namespace {
   454: 
   455:   using NodeSymRel = std::set<NodeSet>;
   456:   using NodePair = std::pair<GepNode *, GepNode *>;
   457:   using NodePairSet = std::set<NodePair>;
   458: 
   459: } // end anonymous namespace
   460: 
   461: static const NodeSet *node_class(GepNode *N, NodeSymRel &Rel) {
   462:   for (const NodeSet &S : Rel)
   463:     if (S.count(N))
   464:       return &S;
   465:   return nullptr;
   466: }
   467: 
   468:   // Create an ordered pair of GepNode pointers. The pair will be used in
   469:   // determining equality. The only purpose of the ordering is to eliminate
   470:   // duplication due to the commutativity of equality/non-equality.
   471: static NodePair node_pair(GepNode *N1, GepNode *N2) {
   472:   uintptr_t P1 = reinterpret_cast<uintptr_t>(N1);
   473:   uintptr_t P2 = reinterpret_cast<uintptr_t>(N2);
   474:   if (P1 <= P2)
   475:     return std::make_pair(N1, N2);
   476:   return std::make_pair(N2, N1);
   477: }
   478: 
   479: static unsigned node_hash(GepNode *N) {
   480:     // Include everything except flags and parent.
   481:     FoldingSetNodeID ID;
   482:     ID.AddPointer(N->Idx);
   483:     ID.AddPointer(N->PTy);
   484:     return ID.ComputeHash();
   485: }
   486: 
   487: static bool node_eq(GepNode *N1, GepNode *N2, NodePairSet &Eq,
   488:                     NodePairSet &Ne) {
   489:     // Don't cache the result for nodes with different hashes. The hash
   490:     // comparison is fast enough.
   491:     if (node_hash(N1) != node_hash(N2))
   492:       return false;
   493: 
   494:     NodePair NP = node_pair(N1, N2);
   495:     NodePairSet::iterator FEq = Eq.find(NP);
   496:     if (FEq != Eq.end())
   497:       return true;
   498:     NodePairSet::iterator FNe = Ne.find(NP);
   499:     if (FNe != Ne.end())
   500:       return false;
```
- EN: It opens namespaces (static) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonCommonGEP::collect, getBlockTraversalOrder, cast<BasicBlock>, LLVM_DEBUG, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCommonGEP, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（static），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonCommonGEP::collect, getBlockTraversalOrder, cast<BasicBlock>, LLVM_DEBUG, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCommonGEP，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501:     // Not previously compared.
   502:     bool Root1 = N1->Flags & GepNode::Root;
   503:     uint32_t CmpFlags = GepNode::Root | GepNode::Pointer;
   504:     bool Different = (N1->Flags & CmpFlags) != (N2->Flags & CmpFlags);
   505:     NodePair P = node_pair(N1, N2);
   506:     // If the root/pointer flags have different values, the nodes are
   507:     // different.
   508:     // If both nodes are root nodes, but their base pointers differ,
   509:     // they are different.
   510:     if (Different || (Root1 && N1->BaseVal != N2->BaseVal)) {
   511:       Ne.insert(P);
   512:       return false;
   513:     }
   514:     // Here the root/pointer flags are identical, and for root nodes the
   515:     // base pointers are equal, so the root nodes are equal.
   516:     // For non-root nodes, compare their parent nodes.
   517:     if (Root1 || node_eq(N1->Parent, N2->Parent, Eq, Ne)) {
   518:       Eq.insert(P);
   519:       return true;
   520:     }
   521:     return false;
   522: }
   523: 
   524: void HexagonCommonGEP::common() {
   525:   // The essence of this commoning is finding gep nodes that are equal.
   526:   // To do this we need to compare all pairs of nodes. To save time,
   527:   // first, partition the set of all nodes into sets of potentially equal
   528:   // nodes, and then compare pairs from within each partition.
   529:   using NodeSetMap = std::map<unsigned, NodeSet>;
   530:   NodeSetMap MaybeEq;
   531: 
   532:   for (GepNode *N : Nodes) {
   533:     unsigned H = node_hash(N);
   534:     MaybeEq[H].insert(N);
   535:   }
   536: 
   537:   // Compute the equivalence relation for the gep nodes.  Use two caches,
   538:   // one for equality and the other for non-equality.
   539:   NodeSymRel EqRel;  // Equality relation (as set of equivalence classes).
   540:   NodePairSet Eq, Ne;  // Caches.
   541:   for (auto &I : MaybeEq) {
   542:     NodeSet &S = I.second;
   543:     for (NodeSet::iterator NI = S.begin(), NE = S.end(); NI != NE; ++NI) {
   544:       GepNode *N = *NI;
   545:       // If node already has a class, then the class must have been created
   546:       // in a prior iteration of this loop. Since equality is transitive,
   547:       // nothing more will be added to that class, so skip it.
   548:       if (node_class(N, EqRel))
   549:         continue;
   550: 
   551:       // Create a new class candidate now.
   552:       NodeSet C;
   553:       for (NodeSet::iterator NJ = std::next(NI); NJ != NE; ++NJ)
   554:         if (node_eq(N, *NJ, Eq, Ne))
   555:           C.insert(*NJ);
   556:       // If Tmp is empty, N would be the only element in it. Don't bother
   557:       // creating a class for it then.
   558:       if (!C.empty()) {
   559:         C.insert(N);  // Finalize the set before adding it to the relation.
   560:         std::pair<NodeSymRel::iterator, bool> Ins = EqRel.insert(C);
   561:         (void)Ins;
   562:         assert(Ins.second && "Cannot add a class");
   563:       }
   564:     }
   565:   }
   566: 
   567:   LLVM_DEBUG({
   568:     dbgs() << "Gep node equality:\n";
   569:     for (NodePairSet::iterator I = Eq.begin(), E = Eq.end(); I != E; ++I)
   570:       dbgs() << "{ " << I->first << ", " << I->second << " }\n";
   571: 
   572:     dbgs() << "Gep equivalence classes:\n";
   573:     for (const NodeSet &S : EqRel) {
   574:       dbgs() << '{';
   575:       for (NodeSet::const_iterator J = S.begin(), F = S.end(); J != F; ++J) {
   576:         if (J != S.begin())
   577:           dbgs() << ',';
   578:         dbgs() << ' ' << *J;
   579:       }
   580:       dbgs() << " }\n";
   581:     }
   582:   });
   583: 
   584:   // Create a projection from a NodeSet to the minimal element in it.
   585:   using ProjMap = std::map<const NodeSet *, GepNode *>;
   586:   ProjMap PM;
   587:   for (const NodeSet &S : EqRel) {
   588:     GepNode *Min = *llvm::min_element(S, NodeOrder);
   589:     std::pair<ProjMap::iterator,bool> Ins = PM.insert(std::make_pair(&S, Min));
   590:     (void)Ins;
   591:     assert(Ins.second && "Cannot add minimal element");
   592: 
   593:     // Update the min element's flags, and user list.
   594:     uint32_t Flags = 0;
   595:     UseSet &MinUs = Uses[Min];
   596:     for (GepNode *N : S) {
   597:       uint32_t NF = N->Flags;
   598:       // If N is used, append all original values of N to the list of
   599:       // original values of Min.
   600:       if (NF & GepNode::Used) {
```
- EN: It declares or implements routines such as node_pair, insert, HexagonCommonGEP::common, node_hash, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonCommonGEP, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 node_pair, insert, HexagonCommonGEP::common, node_hash, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonCommonGEP，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601:         auto &U = Uses[N];
   602:         MinUs.insert_range(U);
   603:       }
   604:       Flags |= NF;
   605:     }
   606:     if (MinUs.empty())
   607:       Uses.erase(Min);
   608: 
   609:     // The collected flags should include all the flags from the min element.
   610:     assert((Min->Flags & Flags) == Min->Flags);
   611:     Min->Flags = Flags;
   612:   }
   613: 
   614:   // Commoning: for each non-root gep node, replace "Parent" with the
   615:   // selected (minimum) node from the corresponding equivalence class.
   616:   // If a given parent does not have an equivalence class, leave it
   617:   // unchanged (it means that it's the only element in its class).
   618:   for (GepNode *N : Nodes) {
   619:     if (N->Flags & GepNode::Root)
   620:       continue;
   621:     const NodeSet *PC = node_class(N->Parent, EqRel);
   622:     if (!PC)
   623:       continue;
   624:     ProjMap::iterator F = PM.find(PC);
   625:     if (F == PM.end())
   626:       continue;
   627:     // Found a replacement, use it.
   628:     GepNode *Rep = F->second;
   629:     N->Parent = Rep;
   630:   }
   631: 
   632:   LLVM_DEBUG(dbgs() << "Gep nodes after commoning:\n" << Nodes);
   633: 
   634:   // Finally, erase the nodes that are no longer used.
   635:   NodeSet Erase;
   636:   for (GepNode *N : Nodes) {
   637:     const NodeSet *PC = node_class(N, EqRel);
   638:     if (!PC)
   639:       continue;
   640:     ProjMap::iterator F = PM.find(PC);
   641:     if (F == PM.end())
   642:       continue;
   643:     if (N == F->second)
   644:       continue;
   645:     // Node for removal.
   646:     Erase.insert(N);
   647:   }
   648:   erase_if(Nodes, in_set(Erase));
   649: 
   650:   LLVM_DEBUG(dbgs() << "Gep nodes after post-commoning cleanup:\n" << Nodes);
   651: }
   652: 
   653: template <typename T>
   654: static BasicBlock *nearest_common_dominator(DominatorTree *DT, T &Blocks) {
   655:   LLVM_DEBUG({
   656:     dbgs() << "NCD of {";
   657:     for (typename T::iterator I = Blocks.begin(), E = Blocks.end(); I != E;
   658:          ++I) {
   659:       if (!*I)
   660:         continue;
   661:       BasicBlock *B = cast<BasicBlock>(*I);
   662:       dbgs() << ' ' << B->getName();
   663:     }
   664:     dbgs() << " }\n";
   665:   });
   666: 
   667:   // Allow null basic blocks in Blocks.  In such cases, return nullptr.
   668:   typename T::iterator I = Blocks.begin(), E = Blocks.end();
   669:   if (I == E || !*I)
   670:     return nullptr;
   671:   BasicBlock *Dom = cast<BasicBlock>(*I);
   672:   while (++I != E) {
   673:     BasicBlock *B = cast_or_null<BasicBlock>(*I);
   674:     Dom = B ? DT->findNearestCommonDominator(Dom, B) : nullptr;
   675:     if (!Dom)
   676:       return nullptr;
   677:     }
   678:     LLVM_DEBUG(dbgs() << "computed:" << Dom->getName() << '\n');
   679:     return Dom;
   680: }
   681: 
   682: template <typename T>
   683: static BasicBlock *nearest_common_dominatee(DominatorTree *DT, T &Blocks) {
   684:     // If two blocks, A and B, dominate a block C, then A dominates B,
   685:     // or B dominates A.
   686:     typename T::iterator I = Blocks.begin(), E = Blocks.end();
   687:     // Find the first non-null block.
   688:     while (I != E && !*I)
   689:       ++I;
   690:     if (I == E)
   691:       return DT->getRoot();
   692:     BasicBlock *DomB = cast<BasicBlock>(*I);
   693:     while (++I != E) {
   694:       if (!*I)
   695:         continue;
   696:       BasicBlock *B = cast<BasicBlock>(*I);
   697:       if (DT->dominates(B, DomB))
   698:         continue;
   699:       if (!DT->dominates(DomB, B))
   700:         return nullptr;
```
- EN: It declares or implements routines such as insert_range, assert, selected, node_class, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 insert_range, assert, selected, node_class, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 701-800 / 第 701-800 行

```cpp
   701:       DomB = B;
   702:     }
   703:     return DomB;
   704: }
   705: 
   706: // Find the first use in B of any value from Values. If no such use,
   707: // return B->end().
   708: template <typename T>
   709: static BasicBlock::iterator first_use_of_in_block(T &Values, BasicBlock *B) {
   710:     BasicBlock::iterator FirstUse = B->end(), BEnd = B->end();
   711: 
   712:     using iterator = typename T::iterator;
   713: 
   714:     for (iterator I = Values.begin(), E = Values.end(); I != E; ++I) {
   715:       Value *V = *I;
   716:       // If V is used in a PHI node, the use belongs to the incoming block,
   717:       // not the block with the PHI node. In the incoming block, the use
   718:       // would be considered as being at the end of it, so it cannot
   719:       // influence the position of the first use (which is assumed to be
   720:       // at the end to start with).
   721:       if (isa<PHINode>(V))
   722:         continue;
   723:       if (!isa<Instruction>(V))
   724:         continue;
   725:       Instruction *In = cast<Instruction>(V);
   726:       if (In->getParent() != B)
   727:         continue;
   728:       BasicBlock::iterator It = In->getIterator();
   729:       if (std::distance(FirstUse, BEnd) < std::distance(It, BEnd))
   730:         FirstUse = It;
   731:     }
   732:     return FirstUse;
   733: }
   734: 
   735: static bool is_empty(const BasicBlock *B) {
   736:     return B->empty() || (&*B->begin() == B->getTerminator());
   737: }
   738: 
   739: BasicBlock *HexagonCommonGEP::recalculatePlacement(GepNode *Node,
   740:       NodeChildrenMap &NCM, NodeToValueMap &Loc) {
   741:   LLVM_DEBUG(dbgs() << "Loc for node:" << Node << '\n');
   742:   // Recalculate the placement for Node, assuming that the locations of
   743:   // its children in Loc are valid.
   744:   // Return nullptr if there is no valid placement for Node (for example, it
   745:   // uses an index value that is not available at the location required
   746:   // to dominate all children, etc.).
   747: 
   748:   // Find the nearest common dominator for:
   749:   // - all users, if the node is used, and
   750:   // - all children.
   751:   ValueVect Bs;
   752:   if (Node->Flags & GepNode::Used) {
   753:     // Append all blocks with uses of the original values to the
   754:     // block vector Bs.
   755:     NodeToUsesMap::iterator UF = Uses.find(Node);
   756:     assert(UF != Uses.end() && "Used node with no use information");
   757:     UseSet &Us = UF->second;
   758:     for (Use *U : Us) {
   759:       User *R = U->getUser();
   760:       if (!isa<Instruction>(R))
   761:         continue;
   762:       BasicBlock *PB = isa<PHINode>(R)
   763:           ? cast<PHINode>(R)->getIncomingBlock(*U)
   764:           : cast<Instruction>(R)->getParent();
   765:       Bs.push_back(PB);
   766:     }
   767:   }
   768:   // Append the location of each child.
   769:   NodeChildrenMap::iterator CF = NCM.find(Node);
   770:   if (CF != NCM.end()) {
   771:     NodeVect &Cs = CF->second;
   772:     for (GepNode *CN : Cs) {
   773:       NodeToValueMap::iterator LF = Loc.find(CN);
   774:       // If the child is only used in GEP instructions (i.e. is not used in
   775:       // non-GEP instructions), the nearest dominator computed for it may
   776:       // have been null. In such case it won't have a location available.
   777:       if (LF == Loc.end())
   778:         continue;
   779:       Bs.push_back(LF->second);
   780:     }
   781:   }
   782: 
   783:   BasicBlock *DomB = nearest_common_dominator(DT, Bs);
   784:   if (!DomB)
   785:     return nullptr;
   786:   // Check if the index used by Node dominates the computed dominator.
   787:   Instruction *IdxI = dyn_cast<Instruction>(Node->Idx);
   788:   if (IdxI && !DT->dominates(IdxI->getParent(), DomB))
   789:     return nullptr;
   790: 
   791:   // Avoid putting nodes into empty blocks.
   792:   while (is_empty(DomB)) {
   793:     DomTreeNode *N = (*DT)[DomB]->getIDom();
   794:     if (!N)
   795:       break;
   796:     DomB = N->getBlock();
   797:   }
   798: 
   799:   // Otherwise, DomB is fine. Update the location map.
   800:   Loc[Node] = DomB;
```
- EN: It declares or implements routines such as end, cast<Instruction>, getIterator, is_empty, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonCommonGEP, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 end, cast<Instruction>, getIterator, is_empty, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonCommonGEP，说明了它与同级后端组件的连接关系。

### Lines 801-900 / 第 801-900 行

```cpp
   801:   return DomB;
   802: }
   803: 
   804: BasicBlock *HexagonCommonGEP::recalculatePlacementRec(GepNode *Node,
   805:       NodeChildrenMap &NCM, NodeToValueMap &Loc) {
   806:   LLVM_DEBUG(dbgs() << "LocRec begin for node:" << Node << '\n');
   807:   // Recalculate the placement of Node, after recursively recalculating the
   808:   // placements of all its children.
   809:   NodeChildrenMap::iterator CF = NCM.find(Node);
   810:   if (CF != NCM.end()) {
   811:     NodeVect &Cs = CF->second;
   812:     for (GepNode *C : Cs)
   813:       recalculatePlacementRec(C, NCM, Loc);
   814:   }
   815:   BasicBlock *LB = recalculatePlacement(Node, NCM, Loc);
   816:   LLVM_DEBUG(dbgs() << "LocRec end for node:" << Node << '\n');
   817:   return LB;
   818: }
   819: 
   820: bool HexagonCommonGEP::isInvariantIn(Value *Val, Loop *L) {
   821:   if (isa<Constant>(Val) || isa<Argument>(Val))
   822:     return true;
   823:   Instruction *In = dyn_cast<Instruction>(Val);
   824:   if (!In)
   825:     return false;
   826:   BasicBlock *HdrB = L->getHeader(), *DefB = In->getParent();
   827:   return DT->properlyDominates(DefB, HdrB);
   828: }
   829: 
   830: bool HexagonCommonGEP::isInvariantIn(GepNode *Node, Loop *L) {
   831:   if (Node->Flags & GepNode::Root)
   832:     if (!isInvariantIn(Node->BaseVal, L))
   833:       return false;
   834:   return isInvariantIn(Node->Idx, L);
   835: }
   836: 
   837: bool HexagonCommonGEP::isInMainPath(BasicBlock *B, Loop *L) {
   838:   BasicBlock *HB = L->getHeader();
   839:   BasicBlock *LB = L->getLoopLatch();
   840:   // B must post-dominate the loop header or dominate the loop latch.
   841:   if (PDT->dominates(B, HB))
   842:     return true;
   843:   if (LB && DT->dominates(B, LB))
   844:     return true;
   845:   return false;
   846: }
   847: 
   848: static BasicBlock *preheader(DominatorTree *DT, Loop *L) {
   849:   if (BasicBlock *PH = L->getLoopPreheader())
   850:     return PH;
   851:   if (!OptSpeculate)
   852:     return nullptr;
   853:   DomTreeNode *DN = DT->getNode(L->getHeader());
   854:   if (!DN)
   855:     return nullptr;
   856:   return DN->getIDom()->getBlock();
   857: }
   858: 
   859: BasicBlock *HexagonCommonGEP::adjustForInvariance(GepNode *Node,
   860:       NodeChildrenMap &NCM, NodeToValueMap &Loc) {
   861:   // Find the "topmost" location for Node: it must be dominated by both,
   862:   // its parent (or the BaseVal, if it's a root node), and by the index
   863:   // value.
   864:   ValueVect Bs;
   865:   if (Node->Flags & GepNode::Root) {
   866:     if (Instruction *PIn = dyn_cast<Instruction>(Node->BaseVal))
   867:       Bs.push_back(PIn->getParent());
   868:   } else {
   869:     Bs.push_back(Loc[Node->Parent]);
   870:   }
   871:   if (Instruction *IIn = dyn_cast<Instruction>(Node->Idx))
   872:     Bs.push_back(IIn->getParent());
   873:   BasicBlock *TopB = nearest_common_dominatee(DT, Bs);
   874: 
   875:   // Traverse the loop nest upwards until we find a loop in which Node
   876:   // is no longer invariant, or until we get to the upper limit of Node's
   877:   // placement. The traversal will also stop when a suitable "preheader"
   878:   // cannot be found for a given loop. The "preheader" may actually be
   879:   // a regular block outside of the loop (i.e. not guarded), in which case
   880:   // the Node will be speculated.
   881:   // For nodes that are not in the main path of the containing loop (i.e.
   882:   // are not executed in each iteration), do not move them out of the loop.
   883:   BasicBlock *LocB = cast_or_null<BasicBlock>(Loc[Node]);
   884:   if (LocB) {
   885:     Loop *Lp = LI->getLoopFor(LocB);
   886:     while (Lp) {
   887:       if (!isInvariantIn(Node, Lp) || !isInMainPath(LocB, Lp))
   888:         break;
   889:       BasicBlock *NewLoc = preheader(DT, Lp);
   890:       if (!NewLoc || !DT->dominates(TopB, NewLoc))
   891:         break;
   892:       Lp = Lp->getParentLoop();
   893:       LocB = NewLoc;
   894:     }
   895:   }
   896:   Loc[Node] = LocB;
   897: 
   898:   // Recursively compute the locations of all children nodes.
   899:   NodeChildrenMap::iterator CF = NCM.find(Node);
   900:   if (CF != NCM.end()) {
```
- EN: It declares or implements routines such as HexagonCommonGEP::recalculatePlacementRec, LLVM_DEBUG, find, recalculatePlacement, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCommonGEP, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonCommonGEP::recalculatePlacementRec, LLVM_DEBUG, find, recalculatePlacement, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCommonGEP，说明了它与同级后端组件的连接关系。

### Lines 901-1000 / 第 901-1000 行

```cpp
   901:     NodeVect &Cs = CF->second;
   902:     for (GepNode *C : Cs)
   903:       adjustForInvariance(C, NCM, Loc);
   904:   }
   905:   return LocB;
   906: }
   907: 
   908: namespace {
   909: 
   910:   struct LocationAsBlock {
   911:     LocationAsBlock(const NodeToValueMap &L) : Map(L) {}
   912: 
   913:     const NodeToValueMap &Map;
   914:   };
   915: 
   916:   [[maybe_unused]] raw_ostream &operator<<(raw_ostream &OS,
   917:                                            const LocationAsBlock &Loc) {
   918:     for (const auto &I : Loc.Map) {
   919:       OS << I.first << " -> ";
   920:       if (BasicBlock *B = cast_or_null<BasicBlock>(I.second))
   921:         OS << B->getName() << '(' << B << ')';
   922:       else
   923:         OS << "<null-block>";
   924:       OS << '\n';
   925:     }
   926:     return OS;
   927:   }
   928: 
   929:   inline bool is_constant(GepNode *N) {
   930:     return isa<ConstantInt>(N->Idx);
   931:   }
   932: 
   933: } // end anonymous namespace
   934: 
   935: void HexagonCommonGEP::separateChainForNode(GepNode *Node, Use *U,
   936:       NodeToValueMap &Loc) {
   937:   User *R = U->getUser();
   938:   LLVM_DEBUG(dbgs() << "Separating chain for node (" << Node << ") user: " << *R
   939:                     << '\n');
   940:   BasicBlock *PB = cast<Instruction>(R)->getParent();
   941: 
   942:   GepNode *N = Node;
   943:   GepNode *C = nullptr, *NewNode = nullptr;
   944:   while (is_constant(N) && !(N->Flags & GepNode::Root)) {
   945:     // XXX if (single-use) dont-replicate;
   946:     GepNode *NewN = new (*Mem) GepNode(N);
   947:     Nodes.push_back(NewN);
   948:     Loc[NewN] = PB;
   949: 
   950:     if (N == Node)
   951:       NewNode = NewN;
   952:     NewN->Flags &= ~GepNode::Used;
   953:     if (C)
   954:       C->Parent = NewN;
   955:     C = NewN;
   956:     N = N->Parent;
   957:   }
   958:   if (!NewNode)
   959:     return;
   960: 
   961:   // Move over all uses that share the same user as U from Node to NewNode.
   962:   NodeToUsesMap::iterator UF = Uses.find(Node);
   963:   assert(UF != Uses.end());
   964:   UseSet &Us = UF->second;
   965:   UseSet NewUs;
   966:   for (Use *U : Us) {
   967:     if (U->getUser() == R)
   968:       NewUs.insert(U);
   969:   }
   970:   for (Use *U : NewUs)
   971:     Us.remove(U); // erase takes an iterator.
   972: 
   973:   if (Us.empty()) {
   974:     Node->Flags &= ~GepNode::Used;
   975:     Uses.erase(UF);
   976:   }
   977: 
   978:   // Should at least have U in NewUs.
   979:   NewNode->Flags |= GepNode::Used;
   980:   LLVM_DEBUG(dbgs() << "new node: " << NewNode << "  " << *NewNode << '\n');
   981:   assert(!NewUs.empty());
   982:   Uses[NewNode] = NewUs;
   983: }
   984: 
   985: void HexagonCommonGEP::separateConstantChains(GepNode *Node,
   986:       NodeChildrenMap &NCM, NodeToValueMap &Loc) {
   987:   // First approximation: extract all chains.
   988:   NodeSet Ns;
   989:   nodes_for_root(Node, NCM, Ns);
   990: 
   991:   LLVM_DEBUG(dbgs() << "Separating constant chains for node: " << Node << '\n');
   992:   // Collect all used nodes together with the uses from loads and stores,
   993:   // where the GEP node could be folded into the load/store instruction.
   994:   NodeToUsesMap FNs; // Foldable nodes.
   995:   for (GepNode *N : Ns) {
   996:     if (!(N->Flags & GepNode::Used))
   997:       continue;
   998:     NodeToUsesMap::iterator UF = Uses.find(N);
   999:     assert(UF != Uses.end());
  1000:     UseSet &Us = UF->second;
```
- EN: It opens namespaces (void) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as LocationAsBlock, which carry the state or API of this component. It declares or implements routines such as LocationAsBlock, is_constant, isa<ConstantInt>, HexagonCommonGEP::separateChainForNode, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（void），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 LocationAsBlock 等类型，用来承载该组件的状态或接口。 这里声明或实现了 LocationAsBlock, is_constant, isa<ConstantInt>, HexagonCommonGEP::separateChainForNode, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1001-1100 / 第 1001-1100 行

```cpp
  1001:     // Loads/stores that use the node N.
  1002:     UseSet LSs;
  1003:     for (Use *U : Us) {
  1004:       User *R = U->getUser();
  1005:       // We're interested in uses that provide the address. It can happen
  1006:       // that the value may also be provided via GEP, but we won't handle
  1007:       // those cases here for now.
  1008:       if (LoadInst *Ld = dyn_cast<LoadInst>(R)) {
  1009:         unsigned PtrX = LoadInst::getPointerOperandIndex();
  1010:         if (&Ld->getOperandUse(PtrX) == U)
  1011:           LSs.insert(U);
  1012:       } else if (StoreInst *St = dyn_cast<StoreInst>(R)) {
  1013:         unsigned PtrX = StoreInst::getPointerOperandIndex();
  1014:         if (&St->getOperandUse(PtrX) == U)
  1015:           LSs.insert(U);
  1016:       }
  1017:     }
  1018:     // Even if the total use count is 1, separating the chain may still be
  1019:     // beneficial, since the constant chain may be longer than the GEP alone
  1020:     // would be (e.g. if the parent node has a constant index and also has
  1021:     // other children).
  1022:     if (!LSs.empty())
  1023:       FNs.insert(std::make_pair(N, LSs));
  1024:   }
  1025: 
  1026:   LLVM_DEBUG(dbgs() << "Nodes with foldable users:\n" << FNs);
  1027: 
  1028:   for (auto &FN : FNs) {
  1029:     GepNode *N = FN.first;
  1030:     UseSet &Us = FN.second;
  1031:     for (Use *U : Us)
  1032:       separateChainForNode(N, U, Loc);
  1033:   }
  1034: }
  1035: 
  1036: void HexagonCommonGEP::computeNodePlacement(NodeToValueMap &Loc) {
  1037:   // Compute the inverse of the Node.Parent links. Also, collect the set
  1038:   // of root nodes.
  1039:   NodeChildrenMap NCM;
  1040:   NodeVect Roots;
  1041:   invert_find_roots(Nodes, NCM, Roots);
  1042: 
  1043:   // Compute the initial placement determined by the users' locations, and
  1044:   // the locations of the child nodes.
  1045:   for (GepNode *Root : Roots)
  1046:     recalculatePlacementRec(Root, NCM, Loc);
  1047: 
  1048:   LLVM_DEBUG(dbgs() << "Initial node placement:\n" << LocationAsBlock(Loc));
  1049: 
  1050:   if (OptEnableInv) {
  1051:     for (GepNode *Root : Roots)
  1052:       adjustForInvariance(Root, NCM, Loc);
  1053: 
  1054:     LLVM_DEBUG(dbgs() << "Node placement after adjustment for invariance:\n"
  1055:                       << LocationAsBlock(Loc));
  1056:   }
  1057:   if (OptEnableConst) {
  1058:     for (GepNode *Root : Roots)
  1059:       separateConstantChains(Root, NCM, Loc);
  1060:   }
  1061:   LLVM_DEBUG(dbgs() << "Node use information:\n" << Uses);
  1062: 
  1063:   // At the moment, there is no further refinement of the initial placement.
  1064:   // Such a refinement could include splitting the nodes if they are placed
  1065:   // too far from some of its users.
  1066: 
  1067:   LLVM_DEBUG(dbgs() << "Final node placement:\n" << LocationAsBlock(Loc));
  1068: }
  1069: 
  1070: Value *HexagonCommonGEP::fabricateGEP(NodeVect &NA, BasicBlock::iterator At,
  1071:       BasicBlock *LocB) {
  1072:   LLVM_DEBUG(dbgs() << "Fabricating GEP in " << LocB->getName()
  1073:                     << " for nodes:\n"
  1074:                     << NA);
  1075:   unsigned Num = NA.size();
  1076:   GepNode *RN = NA[0];
  1077:   assert((RN->Flags & GepNode::Root) && "Creating GEP for non-root");
  1078: 
  1079:   GetElementPtrInst *NewInst = nullptr;
  1080:   Value *Input = RN->BaseVal;
  1081:   Type *InpTy = RN->PTy;
  1082: 
  1083:   unsigned Idx = 0;
  1084:   do {
  1085:     SmallVector<Value*, 4> IdxList;
  1086:     // If the type of the input of the first node is not a pointer,
  1087:     // we need to add an artificial i32 0 to the indices (because the
  1088:     // actual input in the IR will be a pointer).
  1089:     if (!(NA[Idx]->Flags & GepNode::Pointer)) {
  1090:       Type *Int32Ty = Type::getInt32Ty(*Ctx);
  1091:       IdxList.push_back(ConstantInt::get(Int32Ty, 0));
  1092:     }
  1093: 
  1094:     // Keep adding indices from NA until we have to stop and generate
  1095:     // an "intermediate" GEP.
  1096:     while (++Idx <= Num) {
  1097:       GepNode *N = NA[Idx-1];
  1098:       IdxList.push_back(N->Idx);
  1099:       if (Idx < Num) {
  1100:         // We have to stop if we reach a pointer.
```
- EN: It declares or implements routines such as getUser, LoadInst::getPointerOperandIndex, StoreInst::getPointerOperandIndex, be, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonCommonGEP, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getUser, LoadInst::getPointerOperandIndex, StoreInst::getPointerOperandIndex, be, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonCommonGEP，说明了它与同级后端组件的连接关系。

### Lines 1101-1200 / 第 1101-1200 行

```cpp
  1101:         if (NA[Idx]->Flags & GepNode::Pointer)
  1102:           break;
  1103:       }
  1104:     }
  1105:     NewInst = GetElementPtrInst::Create(InpTy, Input, IdxList, "cgep", At);
  1106:     NewInst->setIsInBounds(RN->Flags & GepNode::InBounds);
  1107:     LLVM_DEBUG(dbgs() << "new GEP: " << *NewInst << '\n');
  1108:     if (Idx < Num) {
  1109:       Input = NewInst;
  1110:       InpTy = NA[Idx]->PTy;
  1111:     }
  1112:   } while (Idx <= Num);
  1113: 
  1114:   return NewInst;
  1115: }
  1116: 
  1117: void HexagonCommonGEP::getAllUsersForNode(GepNode *Node, ValueVect &Values,
  1118:       NodeChildrenMap &NCM) {
  1119:   NodeVect Work;
  1120:   Work.push_back(Node);
  1121: 
  1122:   while (!Work.empty()) {
  1123:     NodeVect::iterator First = Work.begin();
  1124:     GepNode *N = *First;
  1125:     Work.erase(First);
  1126:     if (N->Flags & GepNode::Used) {
  1127:       NodeToUsesMap::iterator UF = Uses.find(N);
  1128:       assert(UF != Uses.end() && "No use information for used node");
  1129:       UseSet &Us = UF->second;
  1130:       for (const auto &U : Us)
  1131:         Values.push_back(U->getUser());
  1132:     }
  1133:     NodeChildrenMap::iterator CF = NCM.find(N);
  1134:     if (CF != NCM.end()) {
  1135:       NodeVect &Cs = CF->second;
  1136:       llvm::append_range(Work, Cs);
  1137:     }
  1138:   }
  1139: }
  1140: 
  1141: void HexagonCommonGEP::materialize(NodeToValueMap &Loc) {
  1142:   LLVM_DEBUG(dbgs() << "Nodes before materialization:\n" << Nodes << '\n');
  1143:   NodeChildrenMap NCM;
  1144:   NodeVect Roots;
  1145:   // Compute the inversion again, since computing placement could alter
  1146:   // "parent" relation between nodes.
  1147:   invert_find_roots(Nodes, NCM, Roots);
  1148: 
  1149:   while (!Roots.empty()) {
  1150:     NodeVect::iterator First = Roots.begin();
  1151:     GepNode *Root = *First, *Last = *First;
  1152:     Roots.erase(First);
  1153: 
  1154:     NodeVect NA;  // Nodes to assemble.
  1155:     // Append to NA all child nodes up to (and including) the first child
  1156:     // that:
  1157:     // (1) has more than 1 child, or
  1158:     // (2) is used, or
  1159:     // (3) has a child located in a different block.
  1160:     bool LastUsed = false;
  1161:     unsigned LastCN = 0;
  1162:     // The location may be null if the computation failed (it can legitimately
  1163:     // happen for nodes created from dead GEPs).
  1164:     Value *LocV = Loc[Last];
  1165:     if (!LocV)
  1166:       continue;
  1167:     BasicBlock *LastB = cast<BasicBlock>(LocV);
  1168:     do {
  1169:       NA.push_back(Last);
  1170:       LastUsed = (Last->Flags & GepNode::Used);
  1171:       if (LastUsed)
  1172:         break;
  1173:       NodeChildrenMap::iterator CF = NCM.find(Last);
  1174:       LastCN = (CF != NCM.end()) ? CF->second.size() : 0;
  1175:       if (LastCN != 1)
  1176:         break;
  1177:       GepNode *Child = CF->second.front();
  1178:       BasicBlock *ChildB = cast_or_null<BasicBlock>(Loc[Child]);
  1179:       if (ChildB != nullptr && LastB != ChildB)
  1180:         break;
  1181:       Last = Child;
  1182:     } while (true);
  1183: 
  1184:     BasicBlock::iterator InsertAt = LastB->getTerminator()->getIterator();
  1185:     if (LastUsed || LastCN > 0) {
  1186:       ValueVect Urs;
  1187:       getAllUsersForNode(Root, Urs, NCM);
  1188:       BasicBlock::iterator FirstUse = first_use_of_in_block(Urs, LastB);
  1189:       if (FirstUse != LastB->end())
  1190:         InsertAt = FirstUse;
  1191:     }
  1192: 
  1193:     // Generate a new instruction for NA.
  1194:     Value *NewInst = fabricateGEP(NA, InsertAt, LastB);
  1195: 
  1196:     // Convert all the children of Last node into roots, and append them
  1197:     // to the Roots list.
  1198:     if (LastCN > 0) {
  1199:       NodeVect &Cs = NCM[Last];
  1200:       for (GepNode *CN : Cs) {
```
- EN: It declares or implements routines such as GetElementPtrInst::Create, setIsInBounds, LLVM_DEBUG, HexagonCommonGEP::getAllUsersForNode, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonCommonGEP, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 GetElementPtrInst::Create, setIsInBounds, LLVM_DEBUG, HexagonCommonGEP::getAllUsersForNode, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonCommonGEP，说明了它与同级后端组件的连接关系。

### Lines 1201-1288 / 第 1201-1288 行

```cpp
  1201:         CN->Flags &= ~GepNode::Internal;
  1202:         CN->Flags |= GepNode::Root;
  1203:         CN->BaseVal = NewInst;
  1204:         Roots.push_back(CN);
  1205:       }
  1206:     }
  1207: 
  1208:     // Lastly, if the Last node was used, replace all uses with the new GEP.
  1209:     // The uses reference the original GEP values.
  1210:     if (LastUsed) {
  1211:       NodeToUsesMap::iterator UF = Uses.find(Last);
  1212:       assert(UF != Uses.end() && "No use information found");
  1213:       UseSet &Us = UF->second;
  1214:       for (Use *U : Us)
  1215:         U->set(NewInst);
  1216:     }
  1217:   }
  1218: }
  1219: 
  1220: void HexagonCommonGEP::removeDeadCode() {
  1221:   ValueVect BO;
  1222:   BO.push_back(&Fn->front());
  1223: 
  1224:   for (unsigned i = 0; i < BO.size(); ++i) {
  1225:     BasicBlock *B = cast<BasicBlock>(BO[i]);
  1226:     for (auto *DTN : children<DomTreeNode *>(DT->getNode(B)))
  1227:       BO.push_back(DTN->getBlock());
  1228:   }
  1229: 
  1230:   for (Value *V : llvm::reverse(BO)) {
  1231:     BasicBlock *B = cast<BasicBlock>(V);
  1232:     ValueVect Ins;
  1233:     for (Instruction &I : llvm::reverse(*B))
  1234:       Ins.push_back(&I);
  1235:     for (Value *I : Ins) {
  1236:       Instruction *In = cast<Instruction>(I);
  1237:       if (isInstructionTriviallyDead(In))
  1238:         In->eraseFromParent();
  1239:     }
  1240:   }
  1241: }
  1242: 
  1243: bool HexagonCommonGEP::runOnFunction(Function &F) {
  1244:   if (skipFunction(F))
  1245:     return false;
  1246: 
  1247:   // For now bail out on C++ exception handling.
  1248:   for (const BasicBlock &BB : F)
  1249:     for (const Instruction &I : BB)
  1250:       if (isa<InvokeInst>(I) || isa<LandingPadInst>(I))
  1251:         return false;
  1252: 
  1253:   Fn = &F;
  1254:   DT = &getAnalysis<DominatorTreeWrapperPass>().getDomTree();
  1255:   PDT = &getAnalysis<PostDominatorTreeWrapperPass>().getPostDomTree();
  1256:   LI = &getAnalysis<LoopInfoWrapperPass>().getLoopInfo();
  1257:   Ctx = &F.getContext();
  1258: 
  1259:   Nodes.clear();
  1260:   Uses.clear();
  1261:   NodeOrder.clear();
  1262: 
  1263:   SpecificBumpPtrAllocator<GepNode> Allocator;
  1264:   Mem = &Allocator;
  1265: 
  1266:   collect();
  1267:   common();
  1268: 
  1269:   NodeToValueMap Loc;
  1270:   computeNodePlacement(Loc);
  1271:   materialize(Loc);
  1272:   removeDeadCode();
  1273: 
  1274: #ifdef EXPENSIVE_CHECKS
  1275:   // Run this only when expensive checks are enabled.
  1276:   if (verifyFunction(F, &dbgs()))
  1277:     report_fatal_error("Broken function");
  1278: #endif
  1279:   return true;
  1280: }
  1281: 
  1282: namespace llvm {
  1283: 
  1284:   FunctionPass *createHexagonCommonGEP() {
  1285:     return new HexagonCommonGEP();
  1286:   }
  1287: 
  1288: } // end namespace llvm
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as push_back, find, assert, HexagonCommonGEP::removeDeadCode, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 push_back, find, assert, HexagonCommonGEP::removeDeadCode, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- instruction semantics / 指令语义

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, llvm/ADT/ArrayRef.h, llvm/ADT/FoldingSet.h, llvm/ADT/GraphTraits.h, llvm/ADT/STLExtras.h, llvm/ADT/SetVector.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/Analysis/LoopInfo.h, llvm/Analysis/PostDominators.h, ... (40 total)`
- Hexagon symbols / Hexagon 符号: `HexagonCommonGEP`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
