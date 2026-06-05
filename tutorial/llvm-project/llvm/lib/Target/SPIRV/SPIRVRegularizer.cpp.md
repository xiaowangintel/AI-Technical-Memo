# SPIRVRegularizer.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVRegularizer.cpp`
- Repository: `llvm-project`
- Purpose (EN): This pass implements regularization of LLVM IR for SPIR-V.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- SPIRVRegularizer.cpp - regularize IR for SPIR-V ---------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This pass implements regularization of LLVM IR for SPIR-V. The prototype of
10: // the pass was taken from SPIRV-LLVM translator.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "SPIRVRegularizer.h"
15: #include "SPIRV.h"
16: #include "llvm/IR/Constants.h"
17: #include "llvm/IR/IRBuilder.h"
18: #include "llvm/IR/InstIterator.h"
19: #include "llvm/IR/Instructions.h"
20: #include "llvm/IR/PassManager.h"
21:
22: #include <list>
23:
24: #define DEBUG_TYPE "spirv-regularizer"
25:
26: using namespace llvm;
27:
28: static bool runImpl(Function &F);
29:
30: namespace {
31: struct SPIRVRegularizerLegacy : public FunctionPass {
32: public:
33:   static char ID;
34:   SPIRVRegularizerLegacy() : FunctionPass(ID) {}
35:   bool runOnFunction(Function &F) override { return runImpl(F); }
36:   StringRef getPassName() const override { return "SPIR-V Regularizer"; }
37:
38:   void getAnalysisUsage(AnalysisUsage &AU) const override {
39:     FunctionPass::getAnalysisUsage(AU);
40:   }
```
- EN: This range defines or declares important types such as runImpl, SPIRVRegularizerLegacy, runOnFunction, getPassName, shaping the data model used by SPIRVRegularizer.cpp.
- CN: 这一段定义或声明了 runImpl、SPIRVRegularizerLegacy、runOnFunction、getPassName 等关键类型，构成 SPIRVRegularizer.cpp 使用的数据模型。

### Lines 41-80
```cpp
41: };
42: } // namespace
43:
44: char SPIRVRegularizerLegacy::ID = 0;
45:
46: INITIALIZE_PASS(SPIRVRegularizerLegacy, DEBUG_TYPE, "SPIR-V Regularizer", false,
47:                 false)
48:
49: // Since SPIR-V cannot represent constant expression, constant expressions
50: // in LLVM IR need to be lowered to instructions. For each function,
51: // the constant expressions used by instructions of the function are replaced
52: // by instructions placed in the entry block since it dominates all other BBs.
53: // Each constant expression only needs to be lowered once in each function
54: // and all uses of it by instructions in that function are replaced by
55: // one instruction.
56: // TODO: remove redundant instructions for common subexpression.
57: static void runLowerConstExpr(Function &F) {
58:   LLVMContext &Ctx = F.getContext();
59:   std::list<Instruction *> WorkList;
60:   for (auto &II : instructions(F))
61:     WorkList.push_back(&II);
62:
63:   auto FBegin = F.begin();
64:   while (!WorkList.empty()) {
65:     Instruction *II = WorkList.front();
66:
67:     auto LowerOp = [&II, &FBegin, &F](Value *V) -> Value * {
68:       if (isa<Function>(V))
69:         return V;
70:       auto *CE = cast<ConstantExpr>(V);
71:       LLVM_DEBUG(dbgs() << "[lowerConstantExpressions] " << *CE);
72:       auto ReplInst = CE->getAsInstruction();
73:       auto InsPoint = II->getParent() == &*FBegin ? II : &FBegin->back();
74:       ReplInst->insertBefore(InsPoint->getIterator());
75:       LLVM_DEBUG(dbgs() << " -> " << *ReplInst << '\n');
76:       std::vector<Instruction *> Users;
77:       // Do not replace use during iteration of use. Do it in another loop.
78:       for (auto U : CE->users()) {
79:         LLVM_DEBUG(dbgs() << "[lowerConstantExpressions] Use: " << *U << '\n');
80:         auto InstUser = dyn_cast<Instruction>(U);
```
- EN: This range implements operational logic in helpers such as runLowerConstExpr, getContext, push_back, begin, translating backend policy into executable code.
- CN: 这一段实现了 runLowerConstExpr、getContext、push_back、begin 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:         // Only replace users in scope of current function.
 82:         if (InstUser && InstUser->getParent()->getParent() == &F)
 83:           Users.push_back(InstUser);
 84:       }
 85:       for (auto &User : Users) {
 86:         if (ReplInst->getParent() == User->getParent() &&
 87:             User->comesBefore(ReplInst))
 88:           ReplInst->moveBefore(User->getIterator());
 89:         User->replaceUsesOfWith(CE, ReplInst);
 90:       }
 91:       return ReplInst;
 92:     };
 93:
 94:     WorkList.pop_front();
 95:     auto LowerConstantVec = [&II, &LowerOp, &WorkList,
 96:                              &Ctx](ConstantVector *Vec,
 97:                                    unsigned NumOfOp) -> Value * {
 98:       if (std::all_of(Vec->op_begin(), Vec->op_end(), [](Value *V) {
 99:             return isa<ConstantExpr>(V) || isa<Function>(V);
100:           })) {
101:         // Expand a vector of constexprs and construct it back with
102:         // series of insertelement instructions.
103:         std::list<Value *> OpList;
104:         std::transform(Vec->op_begin(), Vec->op_end(),
105:                        std::back_inserter(OpList),
106:                        [LowerOp](Value *V) { return LowerOp(V); });
107:         Value *Repl = nullptr;
108:         unsigned Idx = 0;
109:         auto *PhiII = dyn_cast<PHINode>(II);
110:         Instruction *InsPoint =
111:             PhiII ? &PhiII->getIncomingBlock(NumOfOp)->back() : II;
112:         std::list<Instruction *> ReplList;
113:         for (auto V : OpList) {
114:           if (auto *Inst = dyn_cast<Instruction>(V))
115:             ReplList.push_back(Inst);
116:           Repl = InsertElementInst::Create(
117:               (Repl ? Repl : PoisonValue::get(Vec->getType())), V,
118:               ConstantInt::get(Type::getInt32Ty(Ctx), Idx++), "",
119:               InsPoint->getIterator());
120:         }
```
- EN: This range implements operational logic in helpers such as push_back, comesBefore, moveBefore, replaceUsesOfWith, translating backend policy into executable code.
- CN: 这一段实现了 push_back、comesBefore、moveBefore、replaceUsesOfWith 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:         WorkList.splice(WorkList.begin(), ReplList);
122:         return Repl;
123:       }
124:       return nullptr;
125:     };
126:     for (unsigned OI = 0, OE = II->getNumOperands(); OI != OE; ++OI) {
127:       auto *Op = II->getOperand(OI);
128:       if (auto *Vec = dyn_cast<ConstantVector>(Op)) {
129:         Value *ReplInst = LowerConstantVec(Vec, OI);
130:         if (ReplInst)
131:           II->replaceUsesOfWith(Op, ReplInst);
132:       } else if (auto CE = dyn_cast<ConstantExpr>(Op)) {
133:         WorkList.push_front(cast<Instruction>(LowerOp(CE)));
134:       } else if (auto MDAsVal = dyn_cast<MetadataAsValue>(Op)) {
135:         auto ConstMD = dyn_cast<ConstantAsMetadata>(MDAsVal->getMetadata());
136:         if (!ConstMD)
137:           continue;
138:         Constant *C = ConstMD->getValue();
139:         Value *ReplInst = nullptr;
140:         if (auto *Vec = dyn_cast<ConstantVector>(C))
141:           ReplInst = LowerConstantVec(Vec, OI);
142:         if (auto *CE = dyn_cast<ConstantExpr>(C))
143:           ReplInst = LowerOp(CE);
144:         if (!ReplInst)
145:           continue;
146:         Metadata *RepMD = ValueAsMetadata::get(ReplInst);
147:         Value *RepMDVal = MetadataAsValue::get(Ctx, RepMD);
148:         II->setOperand(OI, RepMDVal);
149:         WorkList.push_front(cast<Instruction>(ReplInst));
150:       }
151:     }
152:   }
153: }
154:
155: // Lower i1 comparisons with certain predicates to logical operations.
156: // The backend treats i1 as boolean values, and SPIR-V only allows logical
157: // operations for boolean values. This function lowers i1 comparisons with
158: // certain predicates to logical operations to generate valid SPIR-V.
159: static void runLowerI1Comparisons(Function &F) {
160:   for (auto &I : make_early_inc_range(instructions(F))) {
```
- EN: This range implements operational logic in helpers such as splice, getOperand, LowerConstantVec, replaceUsesOfWith, translating backend policy into executable code.
- CN: 这一段实现了 splice、getOperand、LowerConstantVec、replaceUsesOfWith 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:     auto *Cmp = dyn_cast<ICmpInst>(&I);
162:     if (!Cmp)
163:       continue;
164:
165:     bool IsI1 = Cmp->getOperand(0)->getType()->isIntegerTy(1);
166:     if (!IsI1)
167:       continue;
168:
169:     auto Pred = Cmp->getPredicate();
170:     bool IsTargetPred =
171:         Pred >= ICmpInst::ICMP_UGT && Pred <= ICmpInst::ICMP_SLE;
172:     if (!IsTargetPred)
173:       continue;
174:
175:     Value *P = Cmp->getOperand(0);
176:     Value *Q = Cmp->getOperand(1);
177:
178:     IRBuilder<> Builder(Cmp);
179:     Value *Result = nullptr;
180:     switch (Pred) {
181:     case ICmpInst::ICMP_UGT:
182:     case ICmpInst::ICMP_SLT:
183:       // Result = p & !q
184:       Result = Builder.CreateAnd(P, Builder.CreateNot(Q));
185:       break;
186:     case ICmpInst::ICMP_ULT:
187:     case ICmpInst::ICMP_SGT:
188:       // Result = q & !p
189:       Result = Builder.CreateAnd(Q, Builder.CreateNot(P));
190:       break;
191:     case ICmpInst::ICMP_ULE:
192:     case ICmpInst::ICMP_SGE:
193:       // Result = q | !p
194:       Result = Builder.CreateOr(Q, Builder.CreateNot(P));
195:       break;
196:     case ICmpInst::ICMP_UGE:
197:     case ICmpInst::ICMP_SLE:
198:       // Result = p | !q
199:       Result = Builder.CreateOr(P, Builder.CreateNot(Q));
200:       break;
```
- EN: This range implements operational logic in helpers such as getOperand, getPredicate, Builder, CreateAnd, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getPredicate、Builder、CreateAnd 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-224
```cpp
201:     default:
202:       llvm_unreachable("Unexpected predicate");
203:     }
204:
205:     Result->takeName(Cmp);
206:     Cmp->replaceAllUsesWith(Result);
207:     Cmp->eraseFromParent();
208:   }
209: }
210:
211: static bool runImpl(Function &F) {
212:   runLowerI1Comparisons(F);
213:   runLowerConstExpr(F);
214:   return true;
215: }
216:
217: PreservedAnalyses SPIRVRegularizer::run(Function &F,
218:                                         FunctionAnalysisManager &AM) {
219:   return runImpl(F) ? PreservedAnalyses::none() : PreservedAnalyses::all();
220: }
221:
222: FunctionPass *llvm::createSPIRVRegularizerPass() {
223:   return new SPIRVRegularizerLegacy();
224: }
```
- EN: This range implements operational logic in helpers such as llvm_unreachable, takeName, replaceAllUsesWith, eraseFromParent, translating backend policy into executable code.
- CN: 这一段实现了 llvm_unreachable、takeName、replaceAllUsesWith、eraseFromParent 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include runImpl, SPIRVRegularizerLegacy, runOnFunction, getPassName, getAnalysisUsage, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 runImpl, SPIRVRegularizerLegacy, runOnFunction, getPassName, getAnalysisUsage，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVRegularizer.h`
  - `SPIRV.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/IR/Constants.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/InstIterator.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/PassManager.h`
- System/standard headers / 系统或标准头文件:
  - `list`
