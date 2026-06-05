# NVPTXIRPeephole.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXIRPeephole.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements IR-level peephole optimizations.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===------ NVPTXIRPeephole.cpp - NVPTX IR Peephole --------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file implements IR-level peephole optimizations. These transformations
10: // run late in the NVPTX IR pass pipeline just before the instruction selection.
11: //
12: // Currently, it implements the following transformation(s):
13: // 1. FMA folding (float/double types):
14: //    Transforms FMUL+FADD/FSUB sequences into FMA intrinsics when the
15: //    'contract' fast-math flag is present. Supported patterns:
16: //    - fadd(fmul(a, b), c) => fma(a, b, c)
17: //    - fadd(c, fmul(a, b)) => fma(a, b, c)
18: //    - fadd(fmul(a, b), fmul(c, d)) => fma(a, b, fmul(c, d))
19: //    - fsub(fmul(a, b), c) => fma(a, b, fneg(c))
20: //    - fsub(a, fmul(b, c)) => fma(fneg(b), c, a)
21: //    - fsub(fmul(a, b), fmul(c, d)) => fma(a, b, fneg(fmul(c, d)))
22: //
23: //===----------------------------------------------------------------------===//
24:
25: #include "NVPTXUtilities.h"
26: #include "llvm/IR/IRBuilder.h"
27: #include "llvm/IR/InstIterator.h"
28: #include "llvm/IR/Instructions.h"
29: #include "llvm/IR/Intrinsics.h"
30:
31: #define DEBUG_TYPE "nvptx-ir-peephole"
32:
33: using namespace llvm;
34:
35: static bool tryFoldBinaryFMul(BinaryOperator *BI) {
36:   Value *Op0 = BI->getOperand(0);
37:   Value *Op1 = BI->getOperand(1);
38:
39:   auto *FMul0 = dyn_cast<BinaryOperator>(Op0);
40:   auto *FMul1 = dyn_cast<BinaryOperator>(Op1);
```
- EN: This range implements operational logic in helpers such as tryFoldBinaryFMul, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 tryFoldBinaryFMul、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:
42:   BinaryOperator *FMul = nullptr;
43:   Value *OtherOperand = nullptr;
44:   bool IsFirstOperand = false;
45:
46:   // Either Op0 or Op1 should be a valid FMul
47:   if (FMul0 && FMul0->getOpcode() == Instruction::FMul && FMul0->hasOneUse() &&
48:       FMul0->hasAllowContract()) {
49:     FMul = FMul0;
50:     OtherOperand = Op1;
51:     IsFirstOperand = true;
52:   } else if (FMul1 && FMul1->getOpcode() == Instruction::FMul &&
53:              FMul1->hasOneUse() && FMul1->hasAllowContract()) {
54:     FMul = FMul1;
55:     OtherOperand = Op0;
56:     IsFirstOperand = false;
57:   } else {
58:     return false;
59:   }
60:
61:   bool IsFSub = BI->getOpcode() == Instruction::FSub;
62:   LLVM_DEBUG({
63:     const char *OpName = IsFSub ? "FSub" : "FAdd";
64:     dbgs() << "Found " << OpName << " with FMul (single use) as "
65:            << (IsFirstOperand ? "first" : "second") << " operand: " << *BI
66:            << "\n";
67:   });
68:
69:   Value *MulOp0 = FMul->getOperand(0);
70:   Value *MulOp1 = FMul->getOperand(1);
71:   IRBuilder<> Builder(BI);
72:   Value *FMA = nullptr;
73:
74:   if (!IsFSub) {
75:     // fadd(fmul(a, b), c) => fma(a, b, c)
76:     // fadd(c, fmul(a, b)) => fma(a, b, c)
77:     FMA = Builder.CreateIntrinsic(Intrinsic::fma, {BI->getType()},
78:                                   {MulOp0, MulOp1, OtherOperand});
79:   } else {
80:     if (IsFirstOperand) {
```
- EN: This range implements operational logic in helpers such as hasAllowContract, hasOneUse, getOperand, Builder, translating backend policy into executable code.
- CN: 这一段实现了 hasAllowContract、hasOneUse、getOperand、Builder 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:       // fsub(fmul(a, b), c) => fma(a, b, fneg(c))
 82:       Value *NegOtherOp =
 83:           Builder.CreateFNegFMF(OtherOperand, BI->getFastMathFlags());
 84:       FMA = Builder.CreateIntrinsic(Intrinsic::fma, {BI->getType()},
 85:                                     {MulOp0, MulOp1, NegOtherOp});
 86:     } else {
 87:       // fsub(a, fmul(b, c)) => fma(fneg(b), c, a)
 88:       Value *NegMulOp0 =
 89:           Builder.CreateFNegFMF(MulOp0, FMul->getFastMathFlags());
 90:       FMA = Builder.CreateIntrinsic(Intrinsic::fma, {BI->getType()},
 91:                                     {NegMulOp0, MulOp1, OtherOperand});
 92:     }
 93:   }
 94:
 95:   // Combine fast-math flags from the original instructions
 96:   auto *FMAInst = cast<Instruction>(FMA);
 97:   FastMathFlags BinaryFMF = BI->getFastMathFlags();
 98:   FastMathFlags FMulFMF = FMul->getFastMathFlags();
 99:   FastMathFlags NewFMF = FastMathFlags::intersectRewrite(BinaryFMF, FMulFMF) |
100:                          FastMathFlags::unionValue(BinaryFMF, FMulFMF);
101:   FMAInst->setFastMathFlags(NewFMF);
102:
103:   LLVM_DEBUG({
104:     const char *OpName = IsFSub ? "FSub" : "FAdd";
105:     dbgs() << "Replacing " << OpName << " with FMA: " << *FMA << "\n";
106:   });
107:   BI->replaceAllUsesWith(FMA);
108:   BI->eraseFromParent();
109:   FMul->eraseFromParent();
110:   return true;
111: }
112:
113: static bool foldFMA(Function &F) {
114:   bool Changed = false;
115:
116:   // Iterate and process float/double FAdd/FSub instructions with allow-contract
117:   for (auto &I : llvm::make_early_inc_range(instructions(F))) {
118:     if (auto *BI = dyn_cast<BinaryOperator>(&I)) {
119:       // Only FAdd and FSub are supported.
120:       if (BI->getOpcode() != Instruction::FAdd &&
```
- EN: This range implements operational logic in helpers such as CreateFNegFMF, getFastMathFlags, FastMathFlags::unionValue, setFastMathFlags, translating backend policy into executable code.
- CN: 这一段实现了 CreateFNegFMF、getFastMathFlags、FastMathFlags::unionValue、setFastMathFlags 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:           BI->getOpcode() != Instruction::FSub)
122:         continue;
123:
124:       // At minimum, the instruction should have allow-contract.
125:       if (!BI->hasAllowContract())
126:         continue;
127:
128:       // Only float and double are supported.
129:       if (!BI->getType()->isFloatTy() && !BI->getType()->isDoubleTy())
130:         continue;
131:
132:       if (tryFoldBinaryFMul(BI))
133:         Changed = true;
134:     }
135:   }
136:   return Changed;
137: }
138:
139: namespace {
140:
141: struct NVPTXIRPeephole : public FunctionPass {
142:   static char ID;
143:   NVPTXIRPeephole() : FunctionPass(ID) {}
144:   bool runOnFunction(Function &F) override;
145: };
146:
147: } // namespace
148:
149: char NVPTXIRPeephole::ID = 0;
150: INITIALIZE_PASS(NVPTXIRPeephole, "nvptx-ir-peephole", "NVPTX IR Peephole",
151:                 false, false)
152:
153: bool NVPTXIRPeephole::runOnFunction(Function &F) { return foldFMA(F); }
154:
155: FunctionPass *llvm::createNVPTXIRPeepholePass() {
156:   return new NVPTXIRPeephole();
157: }
158:
159: PreservedAnalyses NVPTXIRPeepholePass::run(Function &F,
160:                                            FunctionAnalysisManager &) {
```
- EN: This range defines or declares important types such as getOpcode, NVPTXIRPeephole, runOnFunction, NVPTXIRPeephole::runOnFunction, shaping the data model used by NVPTXIRPeephole.cpp.
- CN: 这一段定义或声明了 getOpcode、NVPTXIRPeephole、runOnFunction、NVPTXIRPeephole::runOnFunction 等关键类型，构成 NVPTXIRPeephole.cpp 使用的数据模型。

### Lines 161-167
```cpp
161:   if (!foldFMA(F))
162:     return PreservedAnalyses::all();
163:
164:   PreservedAnalyses PA;
165:   PA.preserveSet<CFGAnalyses>();
166:   return PA;
167: }
```
- EN: This range implements operational logic in helpers such as PreservedAnalyses::all, translating backend policy into executable code.
- CN: 这一段实现了 PreservedAnalyses::all 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include tryFoldBinaryFMul, getOperand, hasAllowContract, hasOneUse, Builder, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 tryFoldBinaryFMul, getOperand, hasAllowContract, hasOneUse, Builder，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXUtilities.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/InstIterator.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/Intrinsics.h`
