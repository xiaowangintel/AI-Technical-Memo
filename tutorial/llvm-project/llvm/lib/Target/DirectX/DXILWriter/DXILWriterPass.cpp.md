# DXILWriterPass.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILWriter/DXILWriterPass.cpp`
- Repository: `llvm-project`
- Purpose (EN): DXILWriterPass support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- DXILWriterPass.cpp - Bitcode writing pass --------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // DXILWriterPass implementation.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "DXILWriterPass.h"
14: #include "DXILBitcodeWriter.h"
15: #include "DirectXIRPasses/DXILDebugInfo.h"
16: #include "llvm/ADT/DenseMap.h"
17: #include "llvm/ADT/STLExtras.h"
18: #include "llvm/ADT/StringRef.h"
19: #include "llvm/Analysis/ModuleSummaryAnalysis.h"
20: #include "llvm/IR/Constants.h"
21: #include "llvm/IR/DerivedTypes.h"
22: #include "llvm/IR/GlobalVariable.h"
23: #include "llvm/IR/IntrinsicInst.h"
24: #include "llvm/IR/Intrinsics.h"
25: #include "llvm/IR/LLVMContext.h"
26: #include "llvm/IR/Module.h"
27: #include "llvm/IR/PassManager.h"
28: #include "llvm/InitializePasses.h"
29: #include "llvm/Pass.h"
30: #include "llvm/Support/Alignment.h"
31: #include "llvm/Transforms/Utils/ModuleUtils.h"
32:
33: using namespace llvm;
34: using namespace llvm::dxil;
35:
36: namespace {
37: class WriteDXILPass : public llvm::ModulePass {
38:   raw_ostream &OS; // raw_ostream to print on
39:
40: public:
```
- EN: This range defines or declares important types such as WriteDXILPass, shaping the data model used by DXILWriterPass.cpp.
- CN: 这一段定义或声明了 WriteDXILPass 等关键类型，构成 DXILWriterPass.cpp 使用的数据模型。

### Lines 41-80
```cpp
41:   static char ID; // Pass identification, replacement for typeid
42:   WriteDXILPass() : ModulePass(ID), OS(dbgs()) {
43:     initializeWriteDXILPassPass(*PassRegistry::getPassRegistry());
44:   }
45:
46:   explicit WriteDXILPass(raw_ostream &o) : ModulePass(ID), OS(o) {
47:     initializeWriteDXILPassPass(*PassRegistry::getPassRegistry());
48:   }
49:
50:   StringRef getPassName() const override { return "Bitcode Writer"; }
51:
52:   bool runOnModule(Module &M) override {
53:     const auto DIMap = DXILDebugInfoPass::run(M);
54:     WriteDXILToFile(M, OS, DIMap);
55:     return false;
56:   }
57:   void getAnalysisUsage(AnalysisUsage &AU) const override {
58:     AU.setPreservesAll();
59:   }
60: };
61:
62: static void legalizeLifetimeIntrinsics(Module &M) {
63:   LLVMContext &Ctx = M.getContext();
64:   Type *I64Ty = IntegerType::get(Ctx, 64);
65:   Type *PtrTy = PointerType::get(Ctx, 0);
66:   Intrinsic::ID LifetimeIIDs[2] = {Intrinsic::lifetime_start,
67:                                    Intrinsic::lifetime_end};
68:   for (Intrinsic::ID &IID : LifetimeIIDs) {
69:     Function *F = M.getFunction(Intrinsic::getName(IID, {PtrTy}, &M));
70:     if (!F)
71:       continue;
72:
73:     // Get or insert an LLVM 3.7-compliant lifetime intrinsic function of the
74:     // form `void @llvm.lifetime.[start/end](i64, ptr)` with the NoUnwind
75:     // attribute
76:     AttributeList Attr;
77:     Attr = Attr.addFnAttribute(Ctx, Attribute::NoUnwind);
78:     FunctionCallee LifetimeCallee = M.getOrInsertFunction(
79:         Intrinsic::getBaseName(IID), Attr, Type::getVoidTy(Ctx), I64Ty, PtrTy);
80:
```
- EN: This range implements operational logic in helpers such as WriteDXILPass, initializeWriteDXILPassPass, getPassName, runOnModule, translating backend policy into executable code.
- CN: 这一段实现了 WriteDXILPass、initializeWriteDXILPassPass、getPassName、runOnModule 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:     // Replace all calls to lifetime intrinsics with calls to the
 82:     // LLVM 3.7-compliant version of the lifetime intrinsic
 83:     for (User *U : make_early_inc_range(F->users())) {
 84:       CallInst *CI = dyn_cast<CallInst>(U);
 85:       assert(CI &&
 86:              "Expected user of a lifetime intrinsic function to be a CallInst");
 87:
 88:       // LLVM 3.7 lifetime intrinics require an i8* operand, so we insert
 89:       // a bitcast to ensure that is the case
 90:       Value *PtrOperand = CI->getArgOperand(0);
 91:       PointerType *PtrOpPtrTy = cast<PointerType>(PtrOperand->getType());
 92:       Value *NoOpBitCast = CastInst::Create(Instruction::BitCast, PtrOperand,
 93:                                             PtrOpPtrTy, "", CI->getIterator());
 94:
 95:       // LLVM 3.7 lifetime intrinsics have an explicit size operand, whose value
 96:       // we can obtain from the pointer operand which must be an AllocaInst (as
 97:       // of https://github.com/llvm/llvm-project/pull/149310)
 98:       AllocaInst *AI = dyn_cast<AllocaInst>(PtrOperand);
 99:       assert(AI &&
100:              "The pointer operand of a lifetime intrinsic call must be an "
101:              "AllocaInst");
102:       std::optional<TypeSize> AllocSize =
103:           AI->getAllocationSize(CI->getDataLayout());
104:       assert(AllocSize.has_value() &&
105:              "Expected the allocation size of AllocaInst to be known");
106:       CallInst *NewCI = CallInst::Create(
107:           LifetimeCallee,
108:           {ConstantInt::get(I64Ty, AllocSize.value().getFixedValue()),
109:            NoOpBitCast},
110:           "", CI->getIterator());
111:       for (Attribute ParamAttr : CI->getParamAttributes(0))
112:         NewCI->addParamAttr(1, ParamAttr);
113:
114:       CI->eraseFromParent();
115:     }
116:
117:     F->eraseFromParent();
118:   }
119: }
120:
```
- EN: This range implements operational logic in helpers such as getArgOperand, getType, getIterator, getAllocationSize, translating backend policy into executable code.
- CN: 这一段实现了 getArgOperand、getType、getIterator、getAllocationSize 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121: static void removeLifetimeIntrinsics(Module &M) {
122:   Intrinsic::ID LifetimeIIDs[2] = {Intrinsic::lifetime_start,
123:                                    Intrinsic::lifetime_end};
124:   for (Intrinsic::ID &IID : LifetimeIIDs) {
125:     Function *F = M.getFunction(Intrinsic::getBaseName(IID));
126:     if (!F)
127:       continue;
128:
129:     for (User *U : make_early_inc_range(F->users())) {
130:       CallInst *CI = dyn_cast<CallInst>(U);
131:       assert(CI && "Expected user of lifetime function to be a CallInst");
132:       BitCastInst *BCI = dyn_cast<BitCastInst>(CI->getArgOperand(1));
133:       assert(BCI && "Expected pointer operand of CallInst to be a BitCastInst");
134:       CI->eraseFromParent();
135:       BCI->eraseFromParent();
136:     }
137:     F->eraseFromParent();
138:   }
139: }
140:
141: class EmbedDXILPass : public llvm::ModulePass {
142: public:
143:   static char ID; // Pass identification, replacement for typeid
144:   EmbedDXILPass() : ModulePass(ID) {
145:     initializeEmbedDXILPassPass(*PassRegistry::getPassRegistry());
146:   }
147:
148:   StringRef getPassName() const override { return "DXIL Embedder"; }
149:
150:   bool runOnModule(Module &M) override {
151:     std::string Data;
152:     llvm::raw_string_ostream OS(Data);
153:
154:     // Perform late legalization of lifetime intrinsics that would otherwise
155:     // fail the Module Verifier if performed in an earlier pass
156:     legalizeLifetimeIntrinsics(M);
157:
158:     const auto DIMap = DXILDebugInfoPass::run(M);
159:     WriteDXILToFile(M, OS, DIMap);
160:
```
- EN: This range defines or declares important types such as removeLifetimeIntrinsics, getFunction, assert, getArgOperand, shaping the data model used by DXILWriterPass.cpp.
- CN: 这一段定义或声明了 removeLifetimeIntrinsics、getFunction、assert、getArgOperand 等关键类型，构成 DXILWriterPass.cpp 使用的数据模型。

### Lines 161-197
```cpp
161:     // We no longer need lifetime intrinsics after bitcode serialization, so we
162:     // simply remove them to keep the Module Verifier happy after our
163:     // not-so-legal legalizations
164:     removeLifetimeIntrinsics(M);
165:
166:     Constant *ModuleConstant =
167:         ConstantDataArray::get(M.getContext(), arrayRefFromStringRef(Data));
168:     auto *GV = new llvm::GlobalVariable(M, ModuleConstant->getType(), true,
169:                                         GlobalValue::PrivateLinkage,
170:                                         ModuleConstant, "dx.dxil");
171:     GV->setSection("DXIL");
172:     GV->setAlignment(Align(4));
173:     appendToCompilerUsed(M, {GV});
174:     return true;
175:   }
176:
177:   void getAnalysisUsage(AnalysisUsage &AU) const override {
178:     AU.setPreservesAll();
179:   }
180: };
181: } // namespace
182:
183: char WriteDXILPass::ID = 0;
184: INITIALIZE_PASS_BEGIN(WriteDXILPass, "dxil-write-bitcode", "Write Bitcode",
185:                       false, true)
186: INITIALIZE_PASS_DEPENDENCY(ModuleSummaryIndexWrapperPass)
187: INITIALIZE_PASS_END(WriteDXILPass, "dxil-write-bitcode", "Write Bitcode", false,
188:                     true)
189:
190: ModulePass *llvm::createDXILWriterPass(raw_ostream &Str) {
191:   return new WriteDXILPass(Str);
192: }
193:
194: char EmbedDXILPass::ID = 0;
195: INITIALIZE_PASS(EmbedDXILPass, "dxil-embed", "Embed DXIL", false, true)
196:
197: ModulePass *llvm::createDXILEmbedderPass() { return new EmbedDXILPass(); }
```
- EN: This range implements operational logic in helpers such as removeLifetimeIntrinsics, ConstantDataArray::get, setSection, setAlignment, translating backend policy into executable code.
- CN: 这一段实现了 removeLifetimeIntrinsics、ConstantDataArray::get、setSection、setAlignment 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include WriteDXILPass, initializeWriteDXILPassPass, getPassName, runOnModule, DXILDebugInfoPass::run, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 WriteDXILPass, initializeWriteDXILPassPass, getPassName, runOnModule, DXILDebugInfoPass::run，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILWriterPass.h`
  - `DXILBitcodeWriter.h`
  - `DirectXIRPasses/DXILDebugInfo.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/DenseMap.h`
  - `llvm/ADT/STLExtras.h`
  - `llvm/ADT/StringRef.h`
  - `llvm/Analysis/ModuleSummaryAnalysis.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/DerivedTypes.h`
  - `llvm/IR/GlobalVariable.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/LLVMContext.h`
  - `llvm/IR/Module.h`
  - `llvm/IR/PassManager.h`
  - `llvm/InitializePasses.h`
  - `llvm/Pass.h`
  - `llvm/Support/Alignment.h`
  - `llvm/Transforms/Utils/ModuleUtils.h`
