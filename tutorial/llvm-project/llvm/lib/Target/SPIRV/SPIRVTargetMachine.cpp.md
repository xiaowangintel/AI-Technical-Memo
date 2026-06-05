# SPIRVTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVTargetMachine.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements the info about SPIR-V target spec.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- SPIRVTargetMachine.cpp - Define TargetMachine for SPIR-V -*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // Implements the info about SPIR-V target spec.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "SPIRVTargetMachine.h"
14: #include "Analysis/SPIRVConvergenceRegionAnalysis.h"
15: #include "SPIRV.h"
16: #include "SPIRVCBufferAccess.h"
17: #include "SPIRVCtorDtorLowering.h"
18: #include "SPIRVEmitIntrinsics.h"
19: #include "SPIRVGlobalRegistry.h"
20: #include "SPIRVLegalizeImplicitBinding.h"
21: #include "SPIRVLegalizePointerCast.h"
22: #include "SPIRVLegalizeZeroSizeArrays.h"
23: #include "SPIRVLegalizerInfo.h"
24: #include "SPIRVMergeRegionExitTargets.h"
25: #include "SPIRVPrepareFunctions.h"
26: #include "SPIRVPrepareGlobals.h"
27: #include "SPIRVPushConstantAccess.h"
28: #include "SPIRVRegularizer.h"
29: #include "SPIRVStructurizerWrapper.h"
30: #include "SPIRVTargetObjectFile.h"
31: #include "SPIRVTargetTransformInfo.h"
32: #include "TargetInfo/SPIRVTargetInfo.h"
33: #include "llvm/CodeGen/GlobalISel/IRTranslator.h"
34: #include "llvm/CodeGen/GlobalISel/InstructionSelect.h"
35: #include "llvm/CodeGen/GlobalISel/Legalizer.h"
36: #include "llvm/CodeGen/GlobalISel/RegBankSelect.h"
37: #include "llvm/CodeGen/Passes.h"
38: #include "llvm/CodeGen/TargetPassConfig.h"
39: #include "llvm/InitializePasses.h"
40: #include "llvm/MC/TargetRegistry.h"
```
- EN: This range pulls in local backend headers and core LLVM infrastructure needed by the rest of the file.
- CN: 这一段引入本地后端头文件以及后续实现所依赖的 LLVM 基础设施。

### Lines 41-80
```cpp
41: #include "llvm/Pass.h"
42: #include "llvm/Passes/PassBuilder.h"
43: #include "llvm/Support/Compiler.h"
44: #include "llvm/Target/TargetOptions.h"
45: #include "llvm/Transforms/IPO/ExpandVariadics.h"
46: #include "llvm/Transforms/Scalar.h"
47: #include "llvm/Transforms/Utils.h"
48: #include <optional>
49:
50: using namespace llvm;
51:
52: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeSPIRVTarget() {
53:   // Register the target.
54:   RegisterTargetMachine<SPIRVTargetMachine> X(getTheSPIRV32Target());
55:   RegisterTargetMachine<SPIRVTargetMachine> Y(getTheSPIRV64Target());
56:   RegisterTargetMachine<SPIRVTargetMachine> Z(getTheSPIRVLogicalTarget());
57:
58:   PassRegistry &PR = *PassRegistry::getPassRegistry();
59:   initializeGlobalISel(PR);
60:   initializeSPIRVModuleAnalysisPass(PR);
61:   initializeSPIRVAsmPrinterPass(PR);
62:   initializeSPIRVConvergenceRegionAnalysisWrapperPassPass(PR);
63:   initializeSPIRVStructurizerPass(PR);
64:   initializeSPIRVCBufferAccessLegacyPass(PR);
65:   initializeSPIRVPushConstantAccessLegacyPass(PR);
66:   initializeSPIRVPreLegalizerCombinerPass(PR);
67:   initializeSPIRVLegalizePointerCastLegacyPass(PR);
68:   initializeSPIRVLegalizeZeroSizeArraysLegacyPass(PR);
69:   initializeSPIRVRegularizerLegacyPass(PR);
70:   initializeSPIRVPreLegalizerPass(PR);
71:   initializeSPIRVPostLegalizerPass(PR);
72:   initializeSPIRVMergeRegionExitTargetsLegacyPass(PR);
73:   initializeSPIRVEmitIntrinsicsPass(PR);
74:   initializeSPIRVPrepareFunctionsLegacyPass(PR);
75:   initializeSPIRVPrepareGlobalsLegacyPass(PR);
76:   initializeSPIRVLegalizeImplicitBindingLegacyPass(PR);
77:   initializeSPIRVCtorDtorLoweringLegacyPass(PR);
78: }
79:
80: static Reloc::Model getEffectiveRelocModel(std::optional<Reloc::Model> RM) {
```
- EN: This range implements operational logic in helpers such as LLVMInitializeSPIRVTarget, X, Y, Z, translating backend policy into executable code.
- CN: 这一段实现了 LLVMInitializeSPIRVTarget、X、Y、Z 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:   if (!RM)
 82:     return Reloc::PIC_;
 83:   return *RM;
 84: }
 85:
 86: // Pin SPIRVTargetObjectFile's vtables to this file.
 87: SPIRVTargetObjectFile::~SPIRVTargetObjectFile() = default;
 88:
 89: SPIRVTargetMachine::SPIRVTargetMachine(const Target &T, const Triple &TT,
 90:                                        StringRef CPU, StringRef FS,
 91:                                        const TargetOptions &Options,
 92:                                        std::optional<Reloc::Model> RM,
 93:                                        std::optional<CodeModel::Model> CM,
 94:                                        CodeGenOptLevel OL, bool JIT)
 95:     : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, CPU, FS, Options,
 96:                                getEffectiveRelocModel(RM),
 97:                                getEffectiveCodeModel(CM, CodeModel::Small), OL),
 98:       TLOF(std::make_unique<SPIRVTargetObjectFile>()),
 99:       Subtarget(TT, CPU.str(), FS.str(), *this) {
100:   initAsmInfo();
101:   setGlobalISel(true);
102:   setFastISel(false);
103:   setO0WantsFastISel(false);
104:   setRequiresStructuredCFG(false);
105: }
106:
107: void SPIRVTargetMachine::registerPassBuilderCallbacks(PassBuilder &PB) {
108: #define GET_PASS_REGISTRY "SPIRVPassRegistry.def"
109: #include "llvm/Passes/TargetPassRegistry.inc"
110: }
111:
112: namespace {
113: // SPIR-V Code Generator Pass Configuration Options.
114: class SPIRVPassConfig : public TargetPassConfig {
115: public:
116:   SPIRVPassConfig(SPIRVTargetMachine &TM, PassManagerBase &PM)
117:       : TargetPassConfig(TM, PM), TM(TM) {}
118:
119:   SPIRVTargetMachine &getSPIRVTargetMachine() const {
120:     return getTM<SPIRVTargetMachine>();
```
- EN: This range defines or declares important types such as Subtarget, initAsmInfo, setGlobalISel, setFastISel, shaping the data model used by SPIRVTargetMachine.cpp.
- CN: 这一段定义或声明了 Subtarget、initAsmInfo、setGlobalISel、setFastISel 等关键类型，构成 SPIRVTargetMachine.cpp 使用的数据模型。

### Lines 121-160
```cpp
121:   }
122:   void addMachineSSAOptimization() override;
123:   void addIRPasses() override;
124:   void addISelPrepare() override;
125:
126:   bool addIRTranslator() override;
127:   void addPreLegalizeMachineIR() override;
128:   bool addLegalizeMachineIR() override;
129:   bool addRegBankSelect() override;
130:   bool addGlobalInstructionSelect() override;
131:
132:   FunctionPass *createTargetRegisterAllocator(bool) override;
133:   void addFastRegAlloc() override {}
134:   void addOptimizedRegAlloc() override {}
135:
136:   void addPostRegAlloc() override;
137:
138: private:
139:   const SPIRVTargetMachine &TM;
140: };
141: } // namespace
142:
143: // We do not use physical registers, and maintain virtual registers throughout
144: // the entire pipeline, so return nullptr to disable register allocation.
145: FunctionPass *SPIRVPassConfig::createTargetRegisterAllocator(bool) {
146:   return nullptr;
147: }
148:
149: // A place to disable passes that may break CFG.
150: void SPIRVPassConfig::addMachineSSAOptimization() {
151:   TargetPassConfig::addMachineSSAOptimization();
152: }
153:
154: // Disable passes that break from assuming no virtual registers exist.
155: void SPIRVPassConfig::addPostRegAlloc() {
156:   // Do not work with vregs instead of physical regs.
157:   disablePass(&MachineCopyPropagationID);
158:   disablePass(&PostRAMachineSinkingID);
159:   disablePass(&PostRASchedulerID);
160:   disablePass(&FuncletLayoutID);
```
- EN: This range implements operational logic in helpers such as addMachineSSAOptimization, addIRPasses, addISelPrepare, addIRTranslator, translating backend policy into executable code.
- CN: 这一段实现了 addMachineSSAOptimization、addIRPasses、addISelPrepare、addIRTranslator 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:   disablePass(&StackMapLivenessID);
162:   disablePass(&PatchableFunctionID);
163:   disablePass(&ShrinkWrapID);
164:   disablePass(&LiveDebugValuesID);
165:   disablePass(&MachineLateInstrsCleanupID);
166:   disablePass(&RemoveLoadsIntoFakeUsesID);
167:
168:   // Do not work with OpPhi.
169:   disablePass(&BranchFolderPassID);
170:   disablePass(&MachineBlockPlacementID);
171:
172:   TargetPassConfig::addPostRegAlloc();
173: }
174:
175: TargetTransformInfo
176: SPIRVTargetMachine::getTargetTransformInfo(const Function &F) const {
177:   return TargetTransformInfo(std::make_unique<SPIRVTTIImpl>(this, F));
178: }
179:
180: TargetPassConfig *SPIRVTargetMachine::createPassConfig(PassManagerBase &PM) {
181:   return new SPIRVPassConfig(*this, PM);
182: }
183:
184: void SPIRVPassConfig::addIRPasses() {
185:   addPass(createAtomicExpandLegacyPass());
186:
187:   TargetPassConfig::addIRPasses();
188:
189:   // Variadic function calls aren't supported in shader code.
190:   // This needs to come before SPIRVPrepareFunctions because this
191:   // may introduce intrinsic calls.
192:   if (!TM.getSubtargetImpl()->isShader()) {
193:     addPass(createExpandVariadicsPass(ExpandVariadicsMode::Lowering));
194:   }
195:
196:   addPass(createSPIRVRegularizerPass());
197:   addPass(createSPIRVCtorDtorLoweringLegacyPass());
198:   addPass(createSPIRVPrepareFunctionsPass(TM));
199:   addPass(createSPIRVPrepareGlobalsPass());
200: }
```
- EN: This range implements operational logic in helpers such as disablePass, TargetPassConfig::addPostRegAlloc, SPIRVTargetMachine::getTargetTransformInfo, TargetTransformInfo, translating backend policy into executable code.
- CN: 这一段实现了 disablePass、TargetPassConfig::addPostRegAlloc、SPIRVTargetMachine::getTargetTransformInfo、TargetTransformInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-240
```cpp
201:
202: void SPIRVPassConfig::addISelPrepare() {
203:   if (TM.getSubtargetImpl()->isShader()) {
204:     // Vulkan does not allow address space casts. This pass is run to remove
205:     // address space casts that can be removed.
206:     // If an address space cast is not removed while targeting Vulkan, lowering
207:     // will fail during MIR lowering.
208:     addPass(createInferAddressSpacesPass());
209:
210:     // 1.  Simplify loop for subsequent transformations. After this steps, loops
211:     // have the following properties:
212:     //  - loops have a single entry edge (pre-header to loop header).
213:     //  - all loop exits are dominated by the loop pre-header.
214:     //  - loops have a single back-edge.
215:     addPass(createLoopSimplifyPass());
216:
217:     // 2. Removes registers whose lifetime spans across basic blocks. Also
218:     // removes phi nodes. This will greatly simplify the next steps.
219:     addPass(createRegToMemWrapperPass());
220:
221:     // 3. Merge the convergence region exit nodes into one. After this step,
222:     // regions are single-entry, single-exit. This will help determine the
223:     // correct merge block.
224:     addPass(createSPIRVMergeRegionExitTargetsPass());
225:
226:     // 4. Structurize.
227:     addPass(createSPIRVStructurizerPass());
228:
229:     // 5. Reduce the amount of variables required by pushing some operations
230:     // back to virtual registers.
231:     addPass(createPromoteMemoryToRegisterPass());
232:   } else {
233:     // Canonicalize loops so they have a single latch and preheader.
234:     // This enables OpLoopMerge emission for non-shader targets.
235:     addPass(createLoopSimplifyPass());
236:   }
237:   SPIRVTargetMachine &TM = getTM<SPIRVTargetMachine>();
238:   addPass(createStripConvergenceIntrinsicsPass());
239:   addPass(createSPIRVLegalizeImplicitBindingPass());
240:   addPass(createSPIRVLegalizeZeroSizeArraysPass(TM));
```
- EN: This range implements operational logic in helpers such as SPIRVPassConfig::addISelPrepare, addPass, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVPassConfig::addISelPrepare、addPass 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-280
```cpp
241:   addPass(createSPIRVCBufferAccessLegacyPass());
242:   addPass(createSPIRVPushConstantAccessLegacyPass(&TM));
243:   addPass(createSPIRVEmitIntrinsicsPass(TM));
244:   if (TM.getSubtargetImpl()->isLogicalSPIRV())
245:     addPass(createSPIRVLegalizePointerCastPass(&TM));
246:   TargetPassConfig::addISelPrepare();
247: }
248:
249: bool SPIRVPassConfig::addIRTranslator() {
250:   addPass(new IRTranslator(getOptLevel()));
251:   return false;
252: }
253:
254: void SPIRVPassConfig::addPreLegalizeMachineIR() {
255:   addPass(createSPIRVPreLegalizerCombiner());
256:   addPass(createSPIRVPreLegalizerPass());
257: }
258:
259: // Use the default legalizer.
260: bool SPIRVPassConfig::addLegalizeMachineIR() {
261:   addPass(new Legalizer());
262:   addPass(createSPIRVPostLegalizerPass());
263:   return false;
264: }
265:
266: // Do not add the RegBankSelect pass, as we only ever need virtual registers.
267: bool SPIRVPassConfig::addRegBankSelect() {
268:   disablePass(&RegBankSelect::ID);
269:   return false;
270: }
271:
272: // Deprecated flag kept for backward compatibility. NSDI emission is now handled
273: // by SPIRVNonSemanticDebugHandler, registered in SPIRVAsmPrinter::
274: // doInitialization() when the module contains debug info (llvm.dbg.cu).
275: // TODO: Remove this option after a deprecation period. Callers that used
276: // -spv-emit-nonsemantic-debug-info should switch to -g.
277: static cl::opt<bool> SPVEnableNonSemanticDI(
278:     "spv-emit-nonsemantic-debug-info",
279:     cl::desc("Deprecated. Use -g to emit SPIR-V NonSemantic.Shader.DebugInfo "
280:              "instructions"),
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 281-298
```cpp
281:     cl::Optional, cl::init(false));
282:
283: namespace {
284: // A custom subclass of InstructionSelect, which is mostly the same except from
285: // not requiring RegBankSelect to occur previously.
286: class SPIRVInstructionSelect : public InstructionSelect {
287:   // We don't use register banks, so unset the requirement for them
288:   MachineFunctionProperties getRequiredProperties() const override {
289:     return InstructionSelect::getRequiredProperties().resetRegBankSelected();
290:   }
291: };
292: } // namespace
293:
294: // Add the custom SPIRVInstructionSelect from above.
295: bool SPIRVPassConfig::addGlobalInstructionSelect() {
296:   addPass(new SPIRVInstructionSelect());
297:   return false;
298: }
```
- EN: This range defines or declares important types such as cl::init, SPIRVInstructionSelect, getRequiredProperties, InstructionSelect::getRequiredProperties, shaping the data model used by SPIRVTargetMachine.cpp.
- CN: 这一段定义或声明了 cl::init、SPIRVInstructionSelect、getRequiredProperties、InstructionSelect::getRequiredProperties 等关键类型，构成 SPIRVTargetMachine.cpp 使用的数据模型。

## Key Concepts / 关键概念

- EN: Target machine objects capture global code-generation policy, data layout, and pass-pipeline construction.
  - CN: TargetMachine 对象负责记录全局代码生成策略、数据布局以及 pass 流水线构建方式。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include LLVMInitializeSPIRVTarget, X, Y, Z, PassRegistry::getPassRegistry, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 LLVMInitializeSPIRVTarget, X, Y, Z, PassRegistry::getPassRegistry，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVTargetMachine.h`
  - `Analysis/SPIRVConvergenceRegionAnalysis.h`
  - `SPIRV.h`
  - `SPIRVCBufferAccess.h`
  - `SPIRVCtorDtorLowering.h`
  - `SPIRVEmitIntrinsics.h`
  - `SPIRVGlobalRegistry.h`
  - `SPIRVLegalizeImplicitBinding.h`
  - `SPIRVLegalizePointerCast.h`
  - `SPIRVLegalizeZeroSizeArrays.h`
  - `SPIRVLegalizerInfo.h`
  - `SPIRVMergeRegionExitTargets.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/GlobalISel/IRTranslator.h`
  - `llvm/CodeGen/GlobalISel/InstructionSelect.h`
  - `llvm/CodeGen/GlobalISel/Legalizer.h`
  - `llvm/CodeGen/GlobalISel/RegBankSelect.h`
  - `llvm/CodeGen/Passes.h`
  - `llvm/CodeGen/TargetPassConfig.h`
  - `llvm/InitializePasses.h`
  - `llvm/MC/TargetRegistry.h`
  - `llvm/Pass.h`
  - `llvm/Passes/PassBuilder.h`
  - `llvm/Support/Compiler.h`
  - `llvm/Target/TargetOptions.h`
  - `llvm/Transforms/IPO/ExpandVariadics.h`
  - `llvm/Transforms/Scalar.h`
  - `llvm/Transforms/Utils.h`
  - `llvm/Passes/TargetPassRegistry.inc`
- System/standard headers / 系统或标准头文件:
  - `optional`
