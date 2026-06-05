# DXILCBufferAccess.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILCBufferAccess.cpp`
- Repository: `llvm-project`
- Purpose (EN): DXILCBufferAccess support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- DXILCBufferAccess.cpp - Translate CBuffer Loads --------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8:
 9: #include "DXILCBufferAccess.h"
10: #include "DirectX.h"
11: #include "llvm/Analysis/DXILResource.h"
12: #include "llvm/Frontend/HLSL/CBuffer.h"
13: #include "llvm/Frontend/HLSL/HLSLResource.h"
14: #include "llvm/IR/IRBuilder.h"
15: #include "llvm/IR/IntrinsicInst.h"
16: #include "llvm/IR/IntrinsicsDirectX.h"
17: #include "llvm/IR/ReplaceConstant.h"
18: #include "llvm/InitializePasses.h"
19: #include "llvm/Pass.h"
20: #include "llvm/Support/FormatVariadic.h"
21: #include "llvm/Transforms/Utils/Local.h"
22:
23: #define DEBUG_TYPE "dxil-cbuffer-access"
24: using namespace llvm;
25:
26: static void replaceUsersOfGlobal(GlobalVariable *Global,
27:                                  GlobalVariable *HandleGV, size_t Offset) {
28:   for (Use &U : make_early_inc_range(Global->uses())) {
29:     auto UseInst = dyn_cast<Instruction>(U.getUser());
30:     // TODO: Constants? Metadata?
31:     assert(UseInst && "Non-instruction use of cbuffer");
32:
33:     IRBuilder<> Builder(UseInst);
34:     LoadInst *Handle = Builder.CreateLoad(HandleGV->getValueType(), HandleGV,
35:                                           HandleGV->getName());
36:     Value *Ptr = Builder.CreateIntrinsic(
37:         Global->getType(), Intrinsic::dx_resource_getpointer,
38:         ArrayRef<Value *>{Handle,
39:                           ConstantInt::get(Builder.getInt32Ty(), Offset)});
40:     U.set(Ptr);
```
- EN: This range implements operational logic in helpers such as getUser, assert, Builder, getName, translating backend policy into executable code.
- CN: 这一段实现了 getUser、assert、Builder、getName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:   }
42:
43:   Global->removeFromParent();
44: }
45:
46: static bool replaceCBufferAccesses(Module &M) {
47:   std::optional<hlsl::CBufferMetadata> CBufMD = hlsl::CBufferMetadata::get(
48:       M, [](Type *Ty) { return isa<llvm::dxil::PaddingExtType>(Ty); });
49:   if (!CBufMD)
50:     return false;
51:
52:   SmallVector<Constant *> CBufferGlobals;
53:   for (const hlsl::CBufferMapping &Mapping : *CBufMD)
54:     for (const hlsl::CBufferMember &Member : Mapping.Members)
55:       CBufferGlobals.push_back(Member.GV);
56:   convertUsersOfConstantsToInstructions(CBufferGlobals);
57:
58:   for (const hlsl::CBufferMapping &Mapping : *CBufMD)
59:     for (const hlsl::CBufferMember &Member : Mapping.Members)
60:       replaceUsersOfGlobal(Member.GV, Mapping.Handle, Member.Offset);
61:
62:   CBufMD->eraseFromModule();
63:   return true;
64: }
65:
66: PreservedAnalyses DXILCBufferAccess::run(Module &M, ModuleAnalysisManager &AM) {
67:   PreservedAnalyses PA;
68:   bool Changed = replaceCBufferAccesses(M);
69:
70:   if (!Changed)
71:     return PreservedAnalyses::all();
72:   return PA;
73: }
74:
75: namespace {
76: class DXILCBufferAccessLegacy : public ModulePass {
77: public:
78:   bool runOnModule(Module &M) override { return replaceCBufferAccesses(M); }
79:   StringRef getPassName() const override { return "DXIL CBuffer Access"; }
80:   DXILCBufferAccessLegacy() : ModulePass(ID) {}
```
- EN: This range defines or declares important types such as removeFromParent, replaceCBufferAccesses, push_back, convertUsersOfConstantsToInstructions, shaping the data model used by DXILCBufferAccess.cpp.
- CN: 这一段定义或声明了 removeFromParent、replaceCBufferAccesses、push_back、convertUsersOfConstantsToInstructions 等关键类型，构成 DXILCBufferAccess.cpp 使用的数据模型。

### Lines 81-92
```cpp
81:
82:   static char ID; // Pass identification.
83: };
84: char DXILCBufferAccessLegacy::ID = 0;
85: } // end anonymous namespace
86:
87: INITIALIZE_PASS(DXILCBufferAccessLegacy, DEBUG_TYPE, "DXIL CBuffer Access",
88:                 false, false)
89:
90: ModulePass *llvm::createDXILCBufferAccessLegacyPass() {
91:   return new DXILCBufferAccessLegacy();
92: }
```
- EN: This range implements operational logic in helpers such as llvm::createDXILCBufferAccessLegacyPass, DXILCBufferAccessLegacy, translating backend policy into executable code.
- CN: 这一段实现了 llvm::createDXILCBufferAccessLegacyPass、DXILCBufferAccessLegacy 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include getUser, assert, Builder, getName, ConstantInt::get, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 getUser, assert, Builder, getName, ConstantInt::get，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILCBufferAccess.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Analysis/DXILResource.h`
  - `llvm/Frontend/HLSL/CBuffer.h`
  - `llvm/Frontend/HLSL/HLSLResource.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/IntrinsicsDirectX.h`
  - `llvm/IR/ReplaceConstant.h`
  - `llvm/InitializePasses.h`
  - `llvm/Pass.h`
  - `llvm/Support/FormatVariadic.h`
  - `llvm/Transforms/Utils/Local.h`
