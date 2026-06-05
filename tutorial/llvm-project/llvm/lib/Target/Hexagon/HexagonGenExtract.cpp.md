# HexagonGenExtract.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonGenExtract.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon generation of extract operations.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonGenExtract.cpp ----------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "Hexagon.h"
    10: #include "llvm/ADT/APInt.h"
    11: #include "llvm/ADT/GraphTraits.h"
    12: #include "llvm/IR/BasicBlock.h"
    13: #include "llvm/IR/CFG.h"
    14: #include "llvm/IR/Constants.h"
    15: #include "llvm/IR/Dominators.h"
    16: #include "llvm/IR/Function.h"
    17: #include "llvm/IR/IRBuilder.h"
    18: #include "llvm/IR/Instruction.h"
    19: #include "llvm/IR/Instructions.h"
    20: #include "llvm/IR/Intrinsics.h"
    21: #include "llvm/IR/IntrinsicsHexagon.h"
    22: #include "llvm/IR/PatternMatch.h"
    23: #include "llvm/IR/Type.h"
    24: #include "llvm/IR/Value.h"
    25: #include "llvm/InitializePasses.h"
    26: #include "llvm/Pass.h"
    27: #include "llvm/Support/CommandLine.h"
    28: #include <algorithm>
    29: #include <cstdint>
    30: #include <iterator>
    31: 
    32: using namespace llvm;
    33: 
    34: static cl::opt<unsigned> ExtractCutoff("extract-cutoff", cl::init(~0U),
    35:   cl::Hidden, cl::desc("Cutoff for generating \"extract\""
    36:   " instructions"));
    37: 
    38: // This prevents generating extract instructions that have the offset of 0.
    39: // One of the reasons for "extract" is to put a sequence of bits in a regis-
    40: // ter, starting at offset 0 (so that these bits can then be used by an
    41: // "insert"). If the bits are already at offset 0, it is better not to gene-
    42: // rate "extract", since logical bit operations can be merged into compound
    43: // instructions (as opposed to "extract").
    44: static cl::opt<bool> NoSR0("extract-nosr0", cl::init(true), cl::Hidden,
    45:   cl::desc("No extract instruction with offset 0"));
    46: 
    47: namespace {
    48: 
    49:   class HexagonGenExtract : public FunctionPass {
    50:   public:
```
- EN: It imports headers such as Hexagon.h, llvm/ADT/APInt.h, llvm/ADT/GraphTraits.h, llvm/IR/BasicBlock.h, ... (22 total), establishing the LLVM/Hexagon APIs used below. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonGenExtract, which carry the state or API of this component. It defines declarative TableGen records like HexagonGenExtract; these records are consumed by TableGen instead of executed directly.
- CN: 这里引入了 Hexagon.h, llvm/ADT/APInt.h, llvm/ADT/GraphTraits.h, llvm/IR/BasicBlock.h, ... (22 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonGenExtract 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonGenExtract 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 51-100 / 第 51-100 行

```cpp
    51:     static char ID;
    52: 
    53:     HexagonGenExtract() : FunctionPass(ID) {}
    54: 
    55:     StringRef getPassName() const override {
    56:       return "Hexagon generate \"extract\" instructions";
    57:     }
    58: 
    59:     bool runOnFunction(Function &F) override;
    60: 
    61:     void getAnalysisUsage(AnalysisUsage &AU) const override {
    62:       AU.addRequired<DominatorTreeWrapperPass>();
    63:       AU.addPreserved<DominatorTreeWrapperPass>();
    64:       FunctionPass::getAnalysisUsage(AU);
    65:     }
    66: 
    67:   private:
    68:     bool visitBlock(BasicBlock *B);
    69:     bool convert(Instruction *In);
    70: 
    71:     unsigned ExtractCount = 0;
    72:     DominatorTree *DT;
    73:   };
    74: 
    75: } // end anonymous namespace
    76: 
    77: char HexagonGenExtract::ID = 0;
    78: 
    79: INITIALIZE_PASS_BEGIN(HexagonGenExtract, "hextract", "Hexagon generate "
    80:   "\"extract\" instructions", false, false)
    81: INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
    82: INITIALIZE_PASS_END(HexagonGenExtract, "hextract", "Hexagon generate "
    83:   "\"extract\" instructions", false, false)
    84: 
    85: bool HexagonGenExtract::convert(Instruction *In) {
    86:   using namespace PatternMatch;
    87: 
    88:   Value *BF = nullptr;
    89:   ConstantInt *CSL = nullptr, *CSR = nullptr, *CM = nullptr;
    90:   BasicBlock *BB = In->getParent();
    91:   LLVMContext &Ctx = BB->getContext();
    92:   bool LogicalSR;
    93: 
    94:   // (and (shl (lshr x, #sr), #sl), #m)
    95:   LogicalSR = true;
    96:   bool Match = match(In, m_And(m_Shl(m_LShr(m_Value(BF), m_ConstantInt(CSR)),
    97:                                m_ConstantInt(CSL)),
    98:                          m_ConstantInt(CM)));
    99: 
   100:   if (!Match) {
```
- EN: It opens namespaces (char, PatternMatch) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonGenExtract, getPassName, runOnFunction, getAnalysisUsage, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGenExtract, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（char, PatternMatch），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonGenExtract, getPassName, runOnFunction, getAnalysisUsage, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGenExtract，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:     // (and (shl (ashr x, #sr), #sl), #m)
   102:     LogicalSR = false;
   103:     Match = match(In, m_And(m_Shl(m_AShr(m_Value(BF), m_ConstantInt(CSR)),
   104:                             m_ConstantInt(CSL)),
   105:                       m_ConstantInt(CM)));
   106:   }
   107:   if (!Match) {
   108:     // (and (shl x, #sl), #m)
   109:     LogicalSR = true;
   110:     CSR = ConstantInt::get(Type::getInt32Ty(Ctx), 0);
   111:     Match = match(In, m_And(m_Shl(m_Value(BF), m_ConstantInt(CSL)),
   112:                       m_ConstantInt(CM)));
   113:     if (Match && NoSR0)
   114:       return false;
   115:   }
   116:   if (!Match) {
   117:     // (and (lshr x, #sr), #m)
   118:     LogicalSR = true;
   119:     CSL = ConstantInt::get(Type::getInt32Ty(Ctx), 0);
   120:     Match = match(In, m_And(m_LShr(m_Value(BF), m_ConstantInt(CSR)),
   121:                             m_ConstantInt(CM)));
   122:   }
   123:   if (!Match) {
   124:     // (and (ashr x, #sr), #m)
   125:     LogicalSR = false;
   126:     CSL = ConstantInt::get(Type::getInt32Ty(Ctx), 0);
   127:     Match = match(In, m_And(m_AShr(m_Value(BF), m_ConstantInt(CSR)),
   128:                             m_ConstantInt(CM)));
   129:   }
   130:   if (!Match) {
   131:     CM = nullptr;
   132:     // (shl (lshr x, #sr), #sl)
   133:     LogicalSR = true;
   134:     Match = match(In, m_Shl(m_LShr(m_Value(BF), m_ConstantInt(CSR)),
   135:                             m_ConstantInt(CSL)));
   136:   }
   137:   if (!Match) {
   138:     CM = nullptr;
   139:     // (shl (ashr x, #sr), #sl)
   140:     LogicalSR = false;
   141:     Match = match(In, m_Shl(m_AShr(m_Value(BF), m_ConstantInt(CSR)),
   142:                             m_ConstantInt(CSL)));
   143:   }
   144:   if (!Match)
   145:     return false;
   146: 
   147:   Type *Ty = BF->getType();
   148:   if (!Ty->isIntegerTy())
   149:     return false;
   150:   unsigned BW = Ty->getPrimitiveSizeInBits();
```
- EN: It declares or implements routines such as match, ConstantInt::get, getType, getPrimitiveSizeInBits, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 match, ConstantInt::get, getType, getPrimitiveSizeInBits 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   if (BW != 32 && BW != 64)
   152:     return false;
   153: 
   154:   uint32_t SR = CSR->getZExtValue();
   155:   uint32_t SL = CSL->getZExtValue();
   156: 
   157:   if (!CM) {
   158:     // If there was no and, and the shift left did not remove all potential
   159:     // sign bits created by the shift right, then extractu cannot reproduce
   160:     // this value.
   161:     if (!LogicalSR && (SR > SL))
   162:       return false;
   163:     APInt A = APInt(BW, ~0ULL, true).lshr(SR).shl(SL);
   164:     CM = ConstantInt::get(Ctx, A);
   165:   }
   166: 
   167:   // CM is the shifted-left mask. Shift it back right to remove the zero
   168:   // bits on least-significant positions.
   169:   APInt M = CM->getValue().lshr(SL);
   170:   uint32_t T = M.countr_one();
   171: 
   172:   // During the shifts some of the bits will be lost. Calculate how many
   173:   // of the original value will remain after shift right and then left.
   174:   uint32_t U = BW - std::max(SL, SR);
   175:   // The width of the extracted field is the minimum of the original bits
   176:   // that remain after the shifts and the number of contiguous 1s in the mask.
   177:   uint32_t W = std::min(U, T);
   178:   if (W == 0 || W == 1)
   179:     return false;
   180: 
   181:   // Check if the extracted bits are contained within the mask that it is
   182:   // and-ed with. The extract operation will copy these bits, and so the
   183:   // mask cannot any holes in it that would clear any of the bits of the
   184:   // extracted field.
   185:   if (!LogicalSR) {
   186:     // If the shift right was arithmetic, it could have included some 1 bits.
   187:     // It is still ok to generate extract, but only if the mask eliminates
   188:     // those bits (i.e. M does not have any bits set beyond U).
   189:     APInt C = APInt::getHighBitsSet(BW, BW-U);
   190:     if (M.intersects(C) || !M.isMask(W))
   191:       return false;
   192:   } else {
   193:     // Check if M starts with a contiguous sequence of W times 1 bits. Get
   194:     // the low U bits of M (which eliminates the 0 bits shifted in on the
   195:     // left), and check if the result is APInt's "mask":
   196:     if (!M.getLoBits(U).isMask(W))
   197:       return false;
   198:   }
   199: 
   200:   IRBuilder<> IRB(In);
```
- EN: It declares or implements routines such as getZExtValue, APInt, ConstantInt::get, getValue, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getZExtValue, APInt, ConstantInt::get, getValue, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-250 / 第 201-250 行

```cpp
   201:   Intrinsic::ID IntId = (BW == 32) ? Intrinsic::hexagon_S2_extractu
   202:                                    : Intrinsic::hexagon_S2_extractup;
   203:   Value *NewIn =
   204:       IRB.CreateIntrinsic(IntId, {BF, IRB.getInt32(W), IRB.getInt32(SR)});
   205:   if (SL != 0)
   206:     NewIn = IRB.CreateShl(NewIn, SL, CSL->getName());
   207:   In->replaceAllUsesWith(NewIn);
   208:   return true;
   209: }
   210: 
   211: bool HexagonGenExtract::visitBlock(BasicBlock *B) {
   212:   bool Changed = false;
   213: 
   214:   // Depth-first, bottom-up traversal.
   215:   for (auto *DTN : children<DomTreeNode*>(DT->getNode(B)))
   216:     Changed |= visitBlock(DTN->getBlock());
   217: 
   218:   // Allow limiting the number of generated extracts for debugging purposes.
   219:   bool HasCutoff = ExtractCutoff.getPosition();
   220:   unsigned Cutoff = ExtractCutoff;
   221: 
   222:   BasicBlock::iterator I = std::prev(B->end()), NextI, Begin = B->begin();
   223:   while (true) {
   224:     if (HasCutoff && (ExtractCount >= Cutoff))
   225:       return Changed;
   226:     bool Last = (I == Begin);
   227:     if (!Last)
   228:       NextI = std::prev(I);
   229:     Instruction *In = &*I;
   230:     bool Done = convert(In);
   231:     if (HasCutoff && Done)
   232:       ExtractCount++;
   233:     Changed |= Done;
   234:     if (Last)
   235:       break;
   236:     I = NextI;
   237:   }
   238:   return Changed;
   239: }
   240: 
   241: bool HexagonGenExtract::runOnFunction(Function &F) {
   242:   if (skipFunction(F))
   243:     return false;
   244: 
   245:   DT = &getAnalysis<DominatorTreeWrapperPass>().getDomTree();
   246:   bool Changed;
   247: 
   248:   // Traverse the function bottom-up, to see super-expressions before their
   249:   // sub-expressions.
   250:   BasicBlock *Entry = GraphTraits<Function*>::getEntryNode(&F);
```
- EN: It declares or implements routines such as replaceAllUsesWith, HexagonGenExtract::visitBlock, getPosition, std::prev, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGenExtract, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 replaceAllUsesWith, HexagonGenExtract::visitBlock, getPosition, std::prev, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGenExtract，说明了它与同级后端组件的连接关系。

### Lines 251-258 / 第 251-258 行

```cpp
   251:   Changed = visitBlock(Entry);
   252: 
   253:   return Changed;
   254: }
   255: 
   256: FunctionPass *llvm::createHexagonGenExtract() {
   257:   return new HexagonGenExtract();
   258: }
```
- EN: It declares or implements routines such as visitBlock, llvm::createHexagonGenExtract, HexagonGenExtract, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonGenExtract, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 visitBlock, llvm::createHexagonGenExtract, HexagonGenExtract 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonGenExtract，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- pattern-driven instruction selection / 基于模式的指令选择
- intrinsic mapping / Intrinsic 映射

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, llvm/ADT/APInt.h, llvm/ADT/GraphTraits.h, llvm/IR/BasicBlock.h, llvm/IR/CFG.h, llvm/IR/Constants.h, llvm/IR/Dominators.h, llvm/IR/Function.h, llvm/IR/IRBuilder.h, llvm/IR/Instruction.h, ... (22 total)`
- Hexagon symbols / Hexagon 符号: `HexagonGenExtract`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
