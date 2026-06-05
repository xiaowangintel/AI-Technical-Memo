# NVPTXImageOptimizer.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXImageOptimizer.cpp`
- Repository: `llvm-project`
- Purpose (EN): This pass implements IR-level optimizations of image access code, including:.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXImageOptimizer.cpp - Image optimization pass -----------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This pass implements IR-level optimizations of image access code,
10: // including:
11: //
12: // 1. Eliminate istypep intrinsics when image access qualifier is known
13: //
14: //===----------------------------------------------------------------------===//
15:
16: #include "NVPTX.h"
17: #include "NVVMProperties.h"
18: #include "llvm/Analysis/ConstantFolding.h"
19: #include "llvm/IR/Constants.h"
20: #include "llvm/IR/Instructions.h"
21: #include "llvm/IR/Intrinsics.h"
22: #include "llvm/IR/IntrinsicsNVPTX.h"
23: #include "llvm/Pass.h"
24:
25: using namespace llvm;
26:
27: namespace {
28: class NVPTXImageOptimizer : public FunctionPass {
29: private:
30:   static char ID;
31:   SmallVector<Instruction*, 4> InstrToDelete;
32:
33: public:
34:   NVPTXImageOptimizer();
35:
36:   bool runOnFunction(Function &F) override;
37:
38:   StringRef getPassName() const override { return "NVPTX Image Optimizer"; }
39:
40: private:
```
- EN: This range defines or declares important types such as NVPTXImageOptimizer, runOnFunction, getPassName, shaping the data model used by NVPTXImageOptimizer.cpp.
- CN: 这一段定义或声明了 NVPTXImageOptimizer、runOnFunction、getPassName 等关键类型，构成 NVPTXImageOptimizer.cpp 使用的数据模型。

### Lines 41-80
```cpp
41:   bool replaceIsTypeP(Instruction &I, PTXOpaqueType Expected);
42:   Value *cleanupValue(Value *V);
43:   void replaceWith(Instruction *From, ConstantInt *To);
44: };
45: }
46:
47: char NVPTXImageOptimizer::ID = 0;
48:
49: NVPTXImageOptimizer::NVPTXImageOptimizer()
50:   : FunctionPass(ID) {}
51:
52: bool NVPTXImageOptimizer::runOnFunction(Function &F) {
53:   if (skipFunction(F))
54:     return false;
55:
56:   bool Changed = false;
57:   InstrToDelete.clear();
58:
59:   // Look for call instructions in the function
60:   for (BasicBlock &BB : F) {
61:     for (Instruction &Instr : BB) {
62:       if (CallInst *CI = dyn_cast<CallInst>(&Instr)) {
63:         Function *CalledF = CI->getCalledFunction();
64:         if (CalledF && CalledF->isIntrinsic()) {
65:           // This is an intrinsic function call, check if its an istypep
66:           switch (CalledF->getIntrinsicID()) {
67:           default: break;
68:           case Intrinsic::nvvm_istypep_sampler:
69:             Changed |= replaceIsTypeP(Instr, PTXOpaqueType::Sampler);
70:             break;
71:           case Intrinsic::nvvm_istypep_surface:
72:             Changed |= replaceIsTypeP(Instr, PTXOpaqueType::Surface);
73:             break;
74:           case Intrinsic::nvvm_istypep_texture:
75:             Changed |= replaceIsTypeP(Instr, PTXOpaqueType::Texture);
76:             break;
77:           }
78:         }
79:       }
80:     }
```
- EN: This range implements operational logic in helpers such as replaceIsTypeP, cleanupValue, replaceWith, NVPTXImageOptimizer::NVPTXImageOptimizer, translating backend policy into executable code.
- CN: 这一段实现了 replaceIsTypeP、cleanupValue、replaceWith、NVPTXImageOptimizer::NVPTXImageOptimizer 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:   }
 82:
 83:   // Delete any istypep instances we replaced in the IR
 84:   for (Instruction *I : InstrToDelete)
 85:     I->eraseFromParent();
 86:
 87:   return Changed;
 88: }
 89:
 90: bool NVPTXImageOptimizer::replaceIsTypeP(Instruction &I,
 91:                                          PTXOpaqueType Expected) {
 92:   PTXOpaqueType OT = getPTXOpaqueType(*cleanupValue(I.getOperand(0)));
 93:   if (OT == PTXOpaqueType::None)
 94:     return false;
 95:   replaceWith(&I, ConstantInt::getBool(I.getContext(), OT == Expected));
 96:   return true;
 97: }
 98:
 99: void NVPTXImageOptimizer::replaceWith(Instruction *From, ConstantInt *To) {
100:   // We implement "poor man's DCE" here to make sure any code that is no longer
101:   // live is actually unreachable and can be trivially eliminated by the
102:   // unreachable block elimination pass.
103:   for (Use &U : From->uses()) {
104:     if (CondBrInst *BI = dyn_cast<CondBrInst>(U)) {
105:       BasicBlock *Dest = BI->getSuccessor(To->isZero() ? 1 : 0);
106:       UncondBrInst::Create(Dest, BI->getIterator());
107:       InstrToDelete.push_back(BI);
108:     }
109:   }
110:   From->replaceAllUsesWith(To);
111:   InstrToDelete.push_back(From);
112: }
113:
114: Value *NVPTXImageOptimizer::cleanupValue(Value *V) {
115:   if (ExtractValueInst *EVI = dyn_cast<ExtractValueInst>(V)) {
116:     return cleanupValue(EVI->getAggregateOperand());
117:   }
118:   return V;
119: }
120:
```
- EN: This range implements operational logic in helpers such as eraseFromParent, getPTXOpaqueType, replaceWith, NVPTXImageOptimizer::replaceWith, translating backend policy into executable code.
- CN: 这一段实现了 eraseFromParent、getPTXOpaqueType、replaceWith、NVPTXImageOptimizer::replaceWith 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-123
```cpp
121: FunctionPass *llvm::createNVPTXImageOptimizerPass() {
122:   return new NVPTXImageOptimizer();
123: }
```
- EN: This range implements operational logic in helpers such as llvm::createNVPTXImageOptimizerPass, NVPTXImageOptimizer, translating backend policy into executable code.
- CN: 这一段实现了 llvm::createNVPTXImageOptimizerPass、NVPTXImageOptimizer 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXImageOptimizer, runOnFunction, getPassName, replaceIsTypeP, cleanupValue, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXImageOptimizer, runOnFunction, getPassName, replaceIsTypeP, cleanupValue，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
  - `NVVMProperties.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Analysis/ConstantFolding.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsNVPTX.h`
  - `llvm/Pass.h`
