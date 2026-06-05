# DirectXTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DirectXTargetMachine.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains DirectX target initializer.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- DirectXTargetMachine.cpp - DirectX Target Implementation -*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: ///
 9: /// \file
10: /// This file contains DirectX target initializer.
11: ///
12: //===----------------------------------------------------------------------===//
13:
14: #include "DirectXTargetMachine.h"
15: #include "DXILCBufferAccess.h"
16: #include "DXILDataScalarization.h"
17: #include "DXILFinalizeLinkage.h"
18: #include "DXILFlattenArrays.h"
19: #include "DXILForwardHandleAccesses.h"
20: #include "DXILIntrinsicExpansion.h"
21: #include "DXILLegalizePass.h"
22: #include "DXILMemIntrinsics.h"
23: #include "DXILOpLowering.h"
24: #include "DXILPostOptimizationValidation.h"
25: #include "DXILPrettyPrinter.h"
26: #include "DXILResourceAccess.h"
27: #include "DXILResourceImplicitBinding.h"
28: #include "DXILRootSignature.h"
29: #include "DXILShaderFlags.h"
30: #include "DXILTranslateMetadata.h"
31: #include "DXILWriter/DXILWriterPass.h"
32: #include "DirectX.h"
33: #include "DirectXSubtarget.h"
34: #include "DirectXTargetTransformInfo.h"
35: #include "TargetInfo/DirectXTargetInfo.h"
36: #include "llvm/CodeGen/MachineModuleInfo.h"
37: #include "llvm/CodeGen/Passes.h"
38: #include "llvm/CodeGen/TargetPassConfig.h"
39: #include "llvm/IR/IRPrintingPasses.h"
40: #include "llvm/IR/LegacyPassManager.h"
```
- EN: This range pulls in local backend headers and core LLVM infrastructure needed by the rest of the file.
- CN: 这一段引入本地后端头文件以及后续实现所依赖的 LLVM 基础设施。

### Lines 41-80
```cpp
41: #include "llvm/InitializePasses.h"
42: #include "llvm/MC/MCSectionDXContainer.h"
43: #include "llvm/MC/SectionKind.h"
44: #include "llvm/MC/TargetRegistry.h"
45: #include "llvm/Passes/PassBuilder.h"
46: #include "llvm/Support/CodeGen.h"
47: #include "llvm/Support/Compiler.h"
48: #include "llvm/Support/ErrorHandling.h"
49: #include "llvm/Target/TargetLoweringObjectFile.h"
50: #include "llvm/Transforms/IPO/GlobalDCE.h"
51: #include "llvm/Transforms/Scalar.h"
52: #include "llvm/Transforms/Scalar/Scalarizer.h"
53: #include "llvm/Transforms/Utils.h"
54: #include <optional>
55:
56: using namespace llvm;
57:
58: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
59: LLVMInitializeDirectXTarget() {
60:   RegisterTargetMachine<DirectXTargetMachine> X(getTheDirectXTarget());
61:   auto *PR = PassRegistry::getPassRegistry();
62:   initializeDXILIntrinsicExpansionLegacyPass(*PR);
63:   initializeDXILMemIntrinsicsLegacyPass(*PR);
64:   initializeDXILDataScalarizationLegacyPass(*PR);
65:   initializeDXILFlattenArraysLegacyPass(*PR);
66:   initializeScalarizerLegacyPassPass(*PR);
67:   initializeDXILLegalizeLegacyPass(*PR);
68:   initializeDXILPrepareModulePass(*PR);
69:   initializeEmbedDXILPassPass(*PR);
70:   initializeWriteDXILPassPass(*PR);
71:   initializeDXContainerGlobalsPass(*PR);
72:   initializeGlobalDCELegacyPassPass(*PR);
73:   initializeDXILOpLoweringLegacyPass(*PR);
74:   initializeDXILResourceAccessLegacyPass(*PR);
75:   initializeDXILResourceImplicitBindingLegacyPass(*PR);
76:   initializeDXILTranslateMetadataLegacyPass(*PR);
77:   initializeDXILPostOptimizationValidationLegacyPass(*PR);
78:   initializeShaderFlagsAnalysisWrapperPass(*PR);
79:   initializeRootSignatureAnalysisWrapperPass(*PR);
80:   initializeDXILFinalizeLinkageLegacyPass(*PR);
```
- EN: This range implements operational logic in helpers such as LLVMInitializeDirectXTarget, X, PassRegistry::getPassRegistry, initializeDXILIntrinsicExpansionLegacyPass, translating backend policy into executable code.
- CN: 这一段实现了 LLVMInitializeDirectXTarget、X、PassRegistry::getPassRegistry、initializeDXILIntrinsicExpansionLegacyPass 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:   initializeDXILPrettyPrinterLegacyPass(*PR);
 82:   initializeDXILForwardHandleAccessesLegacyPass(*PR);
 83:   initializeDSELegacyPassPass(*PR);
 84:   initializeDXILCBufferAccessLegacyPass(*PR);
 85:   initializeStripConvergenceIntrinsicsLegacyPassPass(*PR);
 86: }
 87:
 88: class DXILTargetObjectFile : public TargetLoweringObjectFile {
 89: public:
 90:   DXILTargetObjectFile() = default;
 91:
 92:   MCSection *getExplicitSectionGlobal(const GlobalObject *GO, SectionKind Kind,
 93:                                       const TargetMachine &TM) const override {
 94:     return getContext().getDXContainerSection(GO->getSection(), Kind);
 95:   }
 96:
 97: protected:
 98:   MCSection *SelectSectionForGlobal(const GlobalObject *GO, SectionKind Kind,
 99:                                     const TargetMachine &TM) const override {
100:     llvm_unreachable("Not supported!");
101:   }
102: };
103:
104: class DirectXPassConfig : public TargetPassConfig {
105: public:
106:   DirectXPassConfig(DirectXTargetMachine &TM, PassManagerBase &PM)
107:       : TargetPassConfig(TM, PM) {}
108:
109:   DirectXTargetMachine &getDirectXTargetMachine() const {
110:     return getTM<DirectXTargetMachine>();
111:   }
112:
113:   FunctionPass *createTargetRegisterAllocator(bool) override { return nullptr; }
114:   void addCodeGenPrepare() override {
115:     addPass(createStripConvergenceIntrinsicsPass());
116:     addPass(createDXILFinalizeLinkageLegacyPass());
117:     addPass(createGlobalDCEPass());
118:     addPass(createDXILMemIntrinsicsLegacyPass());
119:     addPass(createDXILCBufferAccessLegacyPass());
120:     addPass(createDXILResourceAccessLegacyPass());
```
- EN: This range defines or declares important types such as initializeDXILPrettyPrinterLegacyPass, initializeDXILForwardHandleAccessesLegacyPass, initializeDSELegacyPassPass, initializeDXILCBufferAccessLegacyPass, shaping the data model used by DirectXTargetMachine.cpp.
- CN: 这一段定义或声明了 initializeDXILPrettyPrinterLegacyPass、initializeDXILForwardHandleAccessesLegacyPass、initializeDSELegacyPassPass、initializeDXILCBufferAccessLegacyPass 等关键类型，构成 DirectXTargetMachine.cpp 使用的数据模型。

### Lines 121-160
```cpp
121:     addPass(createDXILIntrinsicExpansionLegacyPass());
122:     addPass(createDXILDataScalarizationLegacyPass());
123:     ScalarizerPassOptions DxilScalarOptions;
124:     DxilScalarOptions.ScalarizeLoadStore = true;
125:     addPass(createScalarizerPass(DxilScalarOptions));
126:     addPass(createDXILFlattenArraysLegacyPass());
127:     addPass(createDXILForwardHandleAccessesLegacyPass());
128:     addPass(createDeadStoreEliminationPass());
129:     addPass(createDXILLegalizeLegacyPass());
130:     addPass(createDXILResourceImplicitBindingLegacyPass());
131:     addPass(createDXILTranslateMetadataLegacyPass());
132:     addPass(createDXILPostOptimizationValidationLegacyPass());
133:     addPass(createDXILOpLoweringLegacyPass());
134:     addPass(createDXILPrepareModulePass());
135:   }
136: };
137:
138: DirectXTargetMachine::DirectXTargetMachine(const Target &T, const Triple &TT,
139:                                            StringRef CPU, StringRef FS,
140:                                            const TargetOptions &Options,
141:                                            std::optional<Reloc::Model> RM,
142:                                            std::optional<CodeModel::Model> CM,
143:                                            CodeGenOptLevel OL, bool JIT)
144:     : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, CPU, FS, Options,
145:                                Reloc::Static, CodeModel::Small, OL),
146:       TLOF(std::make_unique<DXILTargetObjectFile>()),
147:       Subtarget(std::make_unique<DirectXSubtarget>(TT, CPU, FS, *this)) {
148:   initAsmInfo();
149: }
150:
151: DirectXTargetMachine::~DirectXTargetMachine() {}
152:
153: void DirectXTargetMachine::registerPassBuilderCallbacks(PassBuilder &PB) {
154: #define GET_PASS_REGISTRY "DirectXPassRegistry.def"
155: #include "llvm/Passes/TargetPassRegistry.inc"
156: }
157:
158: bool DirectXTargetMachine::addPassesToEmitFile(
159:     PassManagerBase &PM, raw_pwrite_stream &Out, raw_pwrite_stream *DwoOut,
160:     CodeGenFileType FileType, bool DisableVerify,
```
- EN: This range implements operational logic in helpers such as addPass, Subtarget, initAsmInfo, DirectXTargetMachine::~DirectXTargetMachine, translating backend policy into executable code.
- CN: 这一段实现了 addPass、Subtarget、initAsmInfo、DirectXTargetMachine::~DirectXTargetMachine 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:     MachineModuleInfoWrapperPass *MMIWP) {
162:   TargetPassConfig *PassConfig = createPassConfig(PM);
163:   PassConfig->addCodeGenPrepare();
164:
165:   switch (FileType) {
166:   case CodeGenFileType::AssemblyFile:
167:     PM.add(createDXILPrettyPrinterLegacyPass(Out));
168:     PM.add(createPrintModulePass(Out, "", true));
169:     break;
170:   case CodeGenFileType::ObjectFile:
171:     if (TargetPassConfig::willCompleteCodeGenPipeline()) {
172:       PM.add(createDXILEmbedderPass());
173:       // We embed the other DXContainer globals after embedding DXIL so that the
174:       // globals don't pollute the DXIL.
175:       PM.add(createDXContainerGlobalsPass());
176:
177:       if (!MMIWP)
178:         MMIWP = new MachineModuleInfoWrapperPass(this);
179:       PM.add(MMIWP);
180:       if (addAsmPrinter(PM, Out, DwoOut, FileType,
181:                         MMIWP->getMMI().getContext()))
182:         return true;
183:     } else
184:       PM.add(createDXILWriterPass(Out));
185:     break;
186:   case CodeGenFileType::Null:
187:     break;
188:   }
189:   return false;
190: }
191:
192: bool DirectXTargetMachine::addPassesToEmitMC(PassManagerBase &PM,
193:                                              MCContext *&Ctx,
194:                                              raw_pwrite_stream &Out,
195:                                              bool DisableVerify) {
196:   return true;
197: }
198:
199: TargetPassConfig *DirectXTargetMachine::createPassConfig(PassManagerBase &PM) {
200:   return new DirectXPassConfig(*this, PM);
```
- EN: This range implements operational logic in helpers such as createPassConfig, addCodeGenPrepare, add, MachineModuleInfoWrapperPass, translating backend policy into executable code.
- CN: 这一段实现了 createPassConfig、addCodeGenPrepare、add、MachineModuleInfoWrapperPass 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-218
```cpp
201: }
202:
203: const DirectXSubtarget *
204: DirectXTargetMachine::getSubtargetImpl(const Function &) const {
205:   return Subtarget.get();
206: }
207:
208: TargetTransformInfo
209: DirectXTargetMachine::getTargetTransformInfo(const Function &F) const {
210:   return TargetTransformInfo(std::make_unique<DirectXTTIImpl>(this, F));
211: }
212:
213: DirectXTargetLowering::DirectXTargetLowering(const DirectXTargetMachine &TM,
214:                                              const DirectXSubtarget &STI)
215:     : TargetLowering(TM, STI) {
216:   addRegisterClass(MVT::i32, &dxil::DXILClassRegClass);
217:   computeRegisterProperties(STI.getRegisterInfo());
218: }
```
- EN: This range implements operational logic in helpers such as DirectXTargetMachine::getSubtargetImpl, get, DirectXTargetMachine::getTargetTransformInfo, TargetTransformInfo, translating backend policy into executable code.
- CN: 这一段实现了 DirectXTargetMachine::getSubtargetImpl、get、DirectXTargetMachine::getTargetTransformInfo、TargetTransformInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Target machine objects capture global code-generation policy, data layout, and pass-pipeline construction.
  - CN: TargetMachine 对象负责记录全局代码生成策略、数据布局以及 pass 流水线构建方式。
- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include LLVMInitializeDirectXTarget, X, PassRegistry::getPassRegistry, initializeDXILIntrinsicExpansionLegacyPass, initializeDXILMemIntrinsicsLegacyPass, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 LLVMInitializeDirectXTarget, X, PassRegistry::getPassRegistry, initializeDXILIntrinsicExpansionLegacyPass, initializeDXILMemIntrinsicsLegacyPass，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DirectXTargetMachine.h`
  - `DXILCBufferAccess.h`
  - `DXILDataScalarization.h`
  - `DXILFinalizeLinkage.h`
  - `DXILFlattenArrays.h`
  - `DXILForwardHandleAccesses.h`
  - `DXILIntrinsicExpansion.h`
  - `DXILLegalizePass.h`
  - `DXILMemIntrinsics.h`
  - `DXILOpLowering.h`
  - `DXILPostOptimizationValidation.h`
  - `DXILPrettyPrinter.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/MachineModuleInfo.h`
  - `llvm/CodeGen/Passes.h`
  - `llvm/CodeGen/TargetPassConfig.h`
  - `llvm/IR/IRPrintingPasses.h`
  - `llvm/IR/LegacyPassManager.h`
  - `llvm/InitializePasses.h`
  - `llvm/MC/MCSectionDXContainer.h`
  - `llvm/MC/SectionKind.h`
  - `llvm/MC/TargetRegistry.h`
  - `llvm/Passes/PassBuilder.h`
  - `llvm/Support/CodeGen.h`
  - `llvm/Support/Compiler.h`
  - `llvm/Support/ErrorHandling.h`
  - `llvm/Target/TargetLoweringObjectFile.h`
  - `llvm/Transforms/IPO/GlobalDCE.h`
  - `llvm/Transforms/Scalar.h`
- System/standard headers / 系统或标准头文件:
  - `optional`
