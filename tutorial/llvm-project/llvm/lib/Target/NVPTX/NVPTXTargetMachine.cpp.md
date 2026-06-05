# NVPTXTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXTargetMachine.cpp`
- Repository: `llvm-project`
- Purpose (EN): Top-level implementation for the NVPTX target.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===-- NVPTXTargetMachine.cpp - Define TargetMachine for NVPTX -----------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // Top-level implementation for the NVPTX target.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "NVPTXTargetMachine.h"
14: #include "NVPTX.h"
15: #include "NVPTXAliasAnalysis.h"
16: #include "NVPTXAllocaHoisting.h"
17: #include "NVPTXAtomicLower.h"
18: #include "NVPTXCtorDtorLowering.h"
19: #include "NVPTXLowerAggrCopies.h"
20: #include "NVPTXMachineFunctionInfo.h"
21: #include "NVPTXTargetObjectFile.h"
22: #include "NVPTXTargetTransformInfo.h"
23: #include "TargetInfo/NVPTXTargetInfo.h"
24: #include "llvm/Analysis/KernelInfo.h"
25: #include "llvm/Analysis/TargetTransformInfo.h"
26: #include "llvm/CodeGen/Passes.h"
27: #include "llvm/CodeGen/TargetPassConfig.h"
28: #include "llvm/IR/IntrinsicsNVPTX.h"
29: #include "llvm/MC/TargetRegistry.h"
30: #include "llvm/Pass.h"
31: #include "llvm/Passes/PassBuilder.h"
32: #include "llvm/Support/CommandLine.h"
33: #include "llvm/Support/Compiler.h"
34: #include "llvm/Target/TargetMachine.h"
35: #include "llvm/Target/TargetOptions.h"
36: #include "llvm/TargetParser/Triple.h"
37: #include "llvm/Transforms/IPO/ExpandVariadics.h"
38: #include "llvm/Transforms/Scalar.h"
39: #include "llvm/Transforms/Scalar/GVN.h"
40: #include "llvm/Transforms/Vectorize/LoadStoreVectorizer.h"
41: #include <cassert>
42: #include <optional>
43: #include <string>
44:
45: using namespace llvm;
46:
47: // LSV is still relatively new; this switch lets us turn it off in case we
48: // encounter (or suspect) a bug.
49: static cl::opt<bool>
50:     DisableLoadStoreVectorizer("disable-nvptx-load-store-vectorizer",
51:                                cl::desc("Disable load/store vectorizer"),
52:                                cl::init(false), cl::Hidden);
53:
54: // NVPTX IR Peephole is a new pass; this option will lets us turn it off in case
55: // we encounter some issues.
56: static cl::opt<bool>
57:     DisableNVPTXIRPeephole("disable-nvptx-ir-peephole",
58:                            cl::desc("Disable NVPTX IR Peephole"),
59:                            cl::init(false), cl::Hidden);
60:
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 61-120
```cpp
 61: // TODO: Remove this flag when we are confident with no regressions.
 62: static cl::opt<bool> DisableRequireStructuredCFG(
 63:     "disable-nvptx-require-structured-cfg",
 64:     cl::desc("Transitional flag to turn off NVPTX's requirement on preserving "
 65:              "structured CFG. The requirement should be disabled only when "
 66:              "unexpected regressions happen."),
 67:     cl::init(false), cl::Hidden);
 68:
 69: static cl::opt<bool> UseShortPointersOpt(
 70:     "nvptx-short-ptr",
 71:     cl::desc(
 72:         "Use 32-bit pointers for accessing const/local/shared address spaces."),
 73:     cl::init(false), cl::Hidden);
 74:
 75: // byval arguments in NVPTX are special. We're only allowed to read from them
 76: // using a special instruction, and if we ever need to write to them or take an
 77: // address, we must make a local copy and use it, instead.
 78: //
 79: // The problem is that local copies are very expensive, and we create them very
 80: // late in the compilation pipeline, so LLVM does not have much of a chance to
 81: // eliminate them, if they turn out to be unnecessary.
 82: //
 83: // One way around that is to create such copies early on, and let them percolate
 84: // through the optimizations. The copying itself will never trigger creation of
 85: // another copy later on, as the reads are allowed. If LLVM can eliminate it,
 86: // it's a win. It the full optimization pipeline can't remove the copy, that's
 87: // as good as it gets in terms of the effort we could've done, and it's
 88: // certainly a much better effort than what we do now.
 89: //
 90: // This early injection of the copies has potential to create undesireable
 91: // side-effects, so it's disabled by default, for now, until it sees more
 92: // testing.
 93: static cl::opt<bool> EarlyByValArgsCopy(
 94:     "nvptx-early-byval-copy",
 95:     cl::desc("Create a copy of byval function arguments early."),
 96:     cl::init(false), cl::Hidden);
 97:
 98: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeNVPTXTarget() {
 99:   // Register the target.
100:   RegisterTargetMachine<NVPTXTargetMachine32> X(getTheNVPTXTarget32());
101:   RegisterTargetMachine<NVPTXTargetMachine64> Y(getTheNVPTXTarget64());
102:
103:   PassRegistry &PR = *PassRegistry::getPassRegistry();
104:   // FIXME: This pass is really intended to be invoked during IR optimization,
105:   // but it's very NVPTX-specific.
106:   initializeNVVMReflectLegacyPassPass(PR);
107:   initializeNVVMIntrRangePass(PR);
108:   initializeGenericToNVVMLegacyPassPass(PR);
109:   initializeNVPTXAllocaHoistingPass(PR);
110:   initializeNVPTXAsmPrinterPass(PR);
111:   initializeNVPTXAssignValidGlobalNamesPass(PR);
112:   initializeNVPTXAtomicLowerPass(PR);
113:   initializeNVPTXLowerArgsLegacyPassPass(PR);
114:   initializeNVPTXSetByValParamAlignLegacyPassPass(PR);
115:   initializeNVPTXMarkKernelPtrsGlobalLegacyPassPass(PR);
116:   initializeNVPTXLowerAllocaPass(PR);
117:   initializeNVPTXLowerUnreachablePass(PR);
118:   initializeNVPTXCtorDtorLoweringLegacyPass(PR);
119:   initializeNVPTXLowerAggrCopiesPass(PR);
120:   initializeNVPTXProxyRegErasurePass(PR);
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 121-180
```cpp
121:   initializeNVPTXForwardParamsPassPass(PR);
122:   initializeNVPTXDAGToDAGISelLegacyPass(PR);
123:   initializeNVPTXAAWrapperPassPass(PR);
124:   initializeNVPTXExternalAAWrapperPass(PR);
125:   initializeNVPTXPeepholePass(PR);
126:   initializeNVPTXTagInvariantLoadLegacyPassPass(PR);
127:   initializeNVPTXIRPeepholePass(PR);
128:   initializeNVPTXPrologEpilogPassPass(PR);
129: }
130:
131: NVPTXTargetMachine::NVPTXTargetMachine(const Target &T, const Triple &TT,
132:                                        StringRef CPU, StringRef FS,
133:                                        const TargetOptions &Options,
134:                                        std::optional<Reloc::Model> RM,
135:                                        std::optional<CodeModel::Model> CM,
136:                                        CodeGenOptLevel OL, bool is64bit)
137:     // The pic relocation model is used regardless of what the client has
138:     // specified, as it is the only relocation model currently supported.
139:     : CodeGenTargetMachineImpl(
140:           T, TT.computeDataLayout(UseShortPointersOpt ? "shortptr" : ""), TT,
141:           CPU, FS, Options, Reloc::PIC_,
142:           getEffectiveCodeModel(CM, CodeModel::Small), OL),
143:       is64bit(is64bit), TLOF(std::make_unique<NVPTXTargetObjectFile>()),
144:       Subtarget(TT, std::string(CPU), std::string(FS), *this),
145:       StrPool(StrAlloc) {
146:   if (TT.getOS() == Triple::NVCL)
147:     drvInterface = NVPTX::NVCL;
148:   else
149:     drvInterface = NVPTX::CUDA;
150:   if (!DisableRequireStructuredCFG)
151:     setRequiresStructuredCFG(true);
152:   initAsmInfo();
153: }
154:
155: NVPTXTargetMachine::~NVPTXTargetMachine() = default;
156:
157: void NVPTXTargetMachine32::anchor() {}
158:
159: NVPTXTargetMachine32::NVPTXTargetMachine32(const Target &T, const Triple &TT,
160:                                            StringRef CPU, StringRef FS,
161:                                            const TargetOptions &Options,
162:                                            std::optional<Reloc::Model> RM,
163:                                            std::optional<CodeModel::Model> CM,
164:                                            CodeGenOptLevel OL, bool JIT)
165:     : NVPTXTargetMachine(T, TT, CPU, FS, Options, RM, CM, OL, false) {}
166:
167: void NVPTXTargetMachine64::anchor() {}
168:
169: NVPTXTargetMachine64::NVPTXTargetMachine64(const Target &T, const Triple &TT,
170:                                            StringRef CPU, StringRef FS,
171:                                            const TargetOptions &Options,
172:                                            std::optional<Reloc::Model> RM,
173:                                            std::optional<CodeModel::Model> CM,
174:                                            CodeGenOptLevel OL, bool JIT)
175:     : NVPTXTargetMachine(T, TT, CPU, FS, Options, RM, CM, OL, true) {}
176:
177: namespace {
178:
179: class NVPTXPassConfig : public TargetPassConfig {
180: public:
```
- EN: This range defines or declares important types such as initializeNVPTXForwardParamsPassPass, initializeNVPTXDAGToDAGISelLegacyPass, initializeNVPTXAAWrapperPassPass, initializeNVPTXExternalAAWrapperPass, shaping the data model used by NVPTXTargetMachine.cpp.
- CN: 这一段定义或声明了 initializeNVPTXForwardParamsPassPass、initializeNVPTXDAGToDAGISelLegacyPass、initializeNVPTXAAWrapperPassPass、initializeNVPTXExternalAAWrapperPass 等关键类型，构成 NVPTXTargetMachine.cpp 使用的数据模型。

### Lines 181-240
```cpp
181:   NVPTXPassConfig(NVPTXTargetMachine &TM, PassManagerBase &PM)
182:       : TargetPassConfig(TM, PM) {}
183:
184:   NVPTXTargetMachine &getNVPTXTargetMachine() const {
185:     return getTM<NVPTXTargetMachine>();
186:   }
187:
188:   void addIRPasses() override;
189:   bool addInstSelector() override;
190:   void addPreRegAlloc() override;
191:   void addPostRegAlloc() override;
192:   void addMachineSSAOptimization() override;
193:
194:   FunctionPass *createTargetRegisterAllocator(bool) override;
195:   void addFastRegAlloc() override;
196:   void addOptimizedRegAlloc() override;
197:
198:   bool addRegAssignAndRewriteFast() override {
199:     llvm_unreachable("should not be used");
200:   }
201:
202:   bool addRegAssignAndRewriteOptimized() override {
203:     llvm_unreachable("should not be used");
204:   }
205:
206: private:
207:   // If the opt level is aggressive, add GVN; otherwise, add EarlyCSE. This
208:   // function is only called in opt mode.
209:   void addEarlyCSEOrGVNPass();
210:
211:   // Add passes that propagate special memory spaces.
212:   void addAddressSpaceInferencePasses();
213:
214:   // Add passes that perform straight-line scalar optimizations.
215:   void addStraightLineScalarOptimizationPasses();
216: };
217:
218: } // end anonymous namespace
219:
220: TargetPassConfig *NVPTXTargetMachine::createPassConfig(PassManagerBase &PM) {
221:   return new NVPTXPassConfig(*this, PM);
222: }
223:
224: MachineFunctionInfo *NVPTXTargetMachine::createMachineFunctionInfo(
225:     BumpPtrAllocator &Allocator, const Function &F,
226:     const TargetSubtargetInfo *STI) const {
227:   return NVPTXMachineFunctionInfo::create<NVPTXMachineFunctionInfo>(Allocator,
228:                                                                     F, STI);
229: }
230:
231: void NVPTXTargetMachine::registerEarlyDefaultAliasAnalyses(AAManager &AAM) {
232:   AAM.registerFunctionAnalysis<NVPTXAA>();
233: }
234:
235: void NVPTXTargetMachine::registerPassBuilderCallbacks(PassBuilder &PB) {
236: #define GET_PASS_REGISTRY "NVPTXPassRegistry.def"
237: #include "llvm/Passes/TargetPassRegistry.inc"
238:
239:   PB.registerPipelineStartEPCallback(
240:       [this](ModulePassManager &PM, OptimizationLevel Level) {
```
- EN: This range implements operational logic in helpers such as NVPTXPassConfig, TargetPassConfig, getNVPTXTargetMachine, addIRPasses, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXPassConfig、TargetPassConfig、getNVPTXTargetMachine、addIRPasses 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:         // We do not want to fold out calls to nvvm.reflect early if the user
242:         // has not provided a target architecture just yet.
243:         if (Subtarget.hasTargetName())
244:           PM.addPass(NVVMReflectPass(Subtarget.getSmVersion()));
245:
246:         FunctionPassManager FPM;
247:         // Note: NVVMIntrRangePass was causing numerical discrepancies at one
248:         // point, if issues crop up, consider disabling.
249:         FPM.addPass(NVVMIntrRangePass());
250:         if (EarlyByValArgsCopy)
251:           FPM.addPass(NVPTXCopyByValArgsPass());
252:         PM.addPass(createModuleToFunctionPassAdaptor(std::move(FPM)));
253:       });
254:
255:   if (!NoKernelInfoEndLTO) {
256:     PB.registerFullLinkTimeOptimizationLastEPCallback(
257:         [this](ModulePassManager &PM, OptimizationLevel Level) {
258:           FunctionPassManager FPM;
259:           FPM.addPass(KernelInfoPrinter(this));
260:           PM.addPass(createModuleToFunctionPassAdaptor(std::move(FPM)));
261:         });
262:   }
263: }
264:
265: TargetTransformInfo
266: NVPTXTargetMachine::getTargetTransformInfo(const Function &F) const {
267:   return TargetTransformInfo(std::make_unique<NVPTXTTIImpl>(this, F));
268: }
269:
270: std::pair<const Value *, unsigned>
271: NVPTXTargetMachine::getPredicatedAddrSpace(const Value *V) const {
272:   if (auto *II = dyn_cast<IntrinsicInst>(V)) {
273:     switch (II->getIntrinsicID()) {
274:     case Intrinsic::nvvm_isspacep_const:
275:       return std::make_pair(II->getArgOperand(0), llvm::ADDRESS_SPACE_CONST);
276:     case Intrinsic::nvvm_isspacep_global:
277:       return std::make_pair(II->getArgOperand(0), llvm::ADDRESS_SPACE_GLOBAL);
278:     case Intrinsic::nvvm_isspacep_local:
279:       return std::make_pair(II->getArgOperand(0), llvm::ADDRESS_SPACE_LOCAL);
280:     case Intrinsic::nvvm_isspacep_shared:
281:       return std::make_pair(II->getArgOperand(0), llvm::ADDRESS_SPACE_SHARED);
282:     case Intrinsic::nvvm_isspacep_shared_cluster:
283:       return std::make_pair(II->getArgOperand(0),
284:                             llvm::ADDRESS_SPACE_SHARED_CLUSTER);
285:     default:
286:       break;
287:     }
288:   }
289:   return std::make_pair(nullptr, -1);
290: }
291:
292: void NVPTXPassConfig::addEarlyCSEOrGVNPass() {
293:   if (getOptLevel() == CodeGenOptLevel::Aggressive)
294:     // Disable scalar PRE due to Register Pressure increase
295:     addPass(createGVNPass(/*ScalarPRE=*/false));
296:   else
297:     addPass(createEarlyCSEPass());
298: }
299:
300: void NVPTXPassConfig::addAddressSpaceInferencePasses() {
```
- EN: This range implements operational logic in helpers such as addPass, NVPTXTargetMachine::getTargetTransformInfo, TargetTransformInfo, NVPTXTargetMachine::getPredicatedAddrSpace, translating backend policy into executable code.
- CN: 这一段实现了 addPass、NVPTXTargetMachine::getTargetTransformInfo、TargetTransformInfo、NVPTXTargetMachine::getPredicatedAddrSpace 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:   // NVPTXLowerArgs emits alloca for byval parameters which can often
302:   // be eliminated by SROA.
303:   addPass(createSROAPass());
304:   addPass(createNVPTXLowerAllocaPass());
305:   // TODO: Consider running InferAddressSpaces during opt, earlier in the
306:   // compilation flow.
307:   addPass(createInferAddressSpacesPass());
308:   addPass(createNVPTXAtomicLowerPass());
309: }
310:
311: void NVPTXPassConfig::addStraightLineScalarOptimizationPasses() {
312:   addPass(createSeparateConstOffsetFromGEPPass());
313:   addPass(createSpeculativeExecutionPass());
314:   // ReassociateGEPs exposes more opportunites for SLSR. See
315:   // the example in reassociate-geps-and-slsr.ll.
316:   addPass(createStraightLineStrengthReducePass());
317:   // SeparateConstOffsetFromGEP and SLSR creates common expressions which GVN or
318:   // EarlyCSE can reuse. GVN generates significantly better code than EarlyCSE
319:   // for some of our benchmarks.
320:   addEarlyCSEOrGVNPass();
321:   // Run NaryReassociate after EarlyCSE/GVN to be more effective.
322:   addPass(createNaryReassociatePass());
323:   // NaryReassociate on GEPs creates redundant common expressions, so run
324:   // EarlyCSE after it.
325:   addPass(createEarlyCSEPass());
326: }
327:
328: void NVPTXPassConfig::addIRPasses() {
329:   // The following passes are known to not play well with virtual regs hanging
330:   // around after register allocation (which in our case, is *all* registers).
331:   // We explicitly disable them here.  We do, however, need some functionality
332:   // of the PrologEpilogCodeInserter pass, so we emulate that behavior in the
333:   // NVPTXPrologEpilog pass (see NVPTXPrologEpilogPass.cpp).
334:   disablePass(&PrologEpilogCodeInserterID);
335:   disablePass(&MachineLateInstrsCleanupID);
336:   disablePass(&MachineCopyPropagationID);
337:   disablePass(&TailDuplicateLegacyID);
338:   disablePass(&StackMapLivenessID);
339:   disablePass(&PostRAMachineSinkingID);
340:   disablePass(&PostRASchedulerID);
341:   disablePass(&FuncletLayoutID);
342:   disablePass(&PatchableFunctionID);
343:   disablePass(&ShrinkWrapID);
344:   disablePass(&RemoveLoadsIntoFakeUsesID);
345:
346:   addPass(createNVPTXAAWrapperPass());
347:   addPass(createNVPTXExternalAAWrapperPass());
348:
349:   // NVVMReflectPass is added in addEarlyAsPossiblePasses, so hopefully running
350:   // it here does nothing.  But since we need it for correctness when lowering
351:   // to NVPTX, run it here too, in case whoever built our pass pipeline didn't
352:   // call addEarlyAsPossiblePasses.
353:   const NVPTXSubtarget &ST = *getTM<NVPTXTargetMachine>().getSubtargetImpl();
354:   addPass(createNVVMReflectPass(ST.getSmVersion()));
355:
356:   if (getOptLevel() != CodeGenOptLevel::None)
357:     addPass(createNVPTXImageOptimizerPass());
358:   addPass(createNVPTXAssignValidGlobalNamesPass());
359:   addPass(createGenericToNVVMLegacyPass());
360:
```
- EN: This range implements operational logic in helpers such as addPass, NVPTXPassConfig::addStraightLineScalarOptimizationPasses, addEarlyCSEOrGVNPass, NVPTXPassConfig::addIRPasses, translating backend policy into executable code.
- CN: 这一段实现了 addPass、NVPTXPassConfig::addStraightLineScalarOptimizationPasses、addEarlyCSEOrGVNPass、NVPTXPassConfig::addIRPasses 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:   // NVPTXLowerArgs is required for correctness and should be run right
362:   // before the address space inference passes.
363:   if (getNVPTXTargetMachine().getDrvInterface() == NVPTX::CUDA)
364:     addPass(createNVPTXMarkKernelPtrsGlobalPass());
365:   addPass(createNVPTXSetByValParamAlignPass());
366:   addPass(createNVPTXLowerArgsPass());
367:   if (getOptLevel() != CodeGenOptLevel::None) {
368:     addAddressSpaceInferencePasses();
369:     addStraightLineScalarOptimizationPasses();
370:   }
371:
372:   addPass(createAtomicExpandLegacyPass());
373:   addPass(createExpandVariadicsPass(ExpandVariadicsMode::Lowering));
374:   addPass(createNVPTXCtorDtorLoweringLegacyPass());
375:
376:   // === LSR and other generic IR passes ===
377:   TargetPassConfig::addIRPasses();
378:   // EarlyCSE is not always strong enough to clean up what LSR produces. For
379:   // example, GVN can combine
380:   //
381:   //   %0 = add %a, %b
382:   //   %1 = add %b, %a
383:   //
384:   // and
385:   //
386:   //   %0 = shl nsw %a, 2
387:   //   %1 = shl %a, 2
388:   //
389:   // but EarlyCSE can do neither of them.
390:   if (getOptLevel() != CodeGenOptLevel::None) {
391:     addEarlyCSEOrGVNPass();
392:     if (!DisableLoadStoreVectorizer)
393:       addPass(createLoadStoreVectorizerPass());
394:     addPass(createSROAPass());
395:     addPass(createNVPTXTagInvariantLoadsPass());
396:     if (!DisableNVPTXIRPeephole)
397:       addPass(createNVPTXIRPeepholePass());
398:   }
399:
400:   if (ST.hasPTXASUnreachableBug()) {
401:     // Run LowerUnreachable to WAR a ptxas bug. See the commit description of
402:     // 1ee4d880e8760256c606fe55b7af85a4f70d006d for more details.
403:     const auto &Options = getNVPTXTargetMachine().Options;
404:     addPass(createNVPTXLowerUnreachablePass(Options.TrapUnreachable,
405:                                             Options.NoTrapAfterNoreturn));
406:   }
407: }
408:
409: bool NVPTXPassConfig::addInstSelector() {
410:   addPass(createLowerAggrCopies());
411:   addPass(createAllocaHoisting());
412:   addPass(createNVPTXISelDag(getNVPTXTargetMachine(), getOptLevel()));
413:   addPass(createNVPTXReplaceImageHandlesPass());
414:
415:   return false;
416: }
417:
418: void NVPTXPassConfig::addPreRegAlloc() {
419:   addPass(createNVPTXForwardParamsPass());
420:   // Remove Proxy Register pseudo instructions used to keep `callseq_end` alive.
```
- EN: This range implements operational logic in helpers such as addPass, addAddressSpaceInferencePasses, addStraightLineScalarOptimizationPasses, TargetPassConfig::addIRPasses, translating backend policy into executable code.
- CN: 这一段实现了 addPass、addAddressSpaceInferencePasses、addStraightLineScalarOptimizationPasses、TargetPassConfig::addIRPasses 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-480
```cpp
421:   addPass(createNVPTXProxyRegErasurePass());
422: }
423:
424: void NVPTXPassConfig::addPostRegAlloc() {
425:   addPass(createNVPTXPrologEpilogPass());
426:   if (getOptLevel() != CodeGenOptLevel::None) {
427:     // NVPTXPrologEpilogPass calculates frame object offset and replace frame
428:     // index with VRFrame register. NVPTXPeephole need to be run after that and
429:     // will replace VRFrame with VRFrameLocal when possible.
430:     addPass(createNVPTXPeephole());
431:   }
432: }
433:
434: FunctionPass *NVPTXPassConfig::createTargetRegisterAllocator(bool) {
435:   return nullptr; // No reg alloc
436: }
437:
438: void NVPTXPassConfig::addFastRegAlloc() {
439:   addPass(&PHIEliminationID);
440:   addPass(&TwoAddressInstructionPassID);
441: }
442:
443: void NVPTXPassConfig::addOptimizedRegAlloc() {
444:   addPass(&ProcessImplicitDefsID);
445:   addPass(&LiveVariablesID);
446:   addPass(&MachineLoopInfoID);
447:   addPass(&PHIEliminationID);
448:
449:   addPass(&TwoAddressInstructionPassID);
450:   addPass(&RegisterCoalescerID);
451:
452:   // PreRA instruction scheduling.
453:   if (addPass(&MachineSchedulerID))
454:     printAndVerify("After Machine Scheduling");
455:
456:   addPass(&StackSlotColoringID);
457:
458:   // FIXME: Needs physical registers
459:   // addPass(&MachineLICMID);
460:
461:   printAndVerify("After StackSlotColoring");
462: }
463:
464: void NVPTXPassConfig::addMachineSSAOptimization() {
465:   // Pre-ra tail duplication.
466:   if (addPass(&EarlyTailDuplicateLegacyID))
467:     printAndVerify("After Pre-RegAlloc TailDuplicate");
468:
469:   // Optimize PHIs before DCE: removing dead PHI cycles may make more
470:   // instructions dead.
471:   addPass(&OptimizePHIsLegacyID);
472:
473:   // This pass merges large allocas. StackSlotColoring is a different pass
474:   // which merges spill slots.
475:   addPass(&StackColoringLegacyID);
476:
477:   // If the target requests it, assign local variables to stack slots relative
478:   // to one another and simplify frame index references where possible.
479:   addPass(&LocalStackSlotAllocationID);
480:
```
- EN: This range implements operational logic in helpers such as addPass, NVPTXPassConfig::addPostRegAlloc, NVPTXPassConfig::createTargetRegisterAllocator, NVPTXPassConfig::addFastRegAlloc, translating backend policy into executable code.
- CN: 这一段实现了 addPass、NVPTXPassConfig::addPostRegAlloc、NVPTXPassConfig::createTargetRegisterAllocator、NVPTXPassConfig::addFastRegAlloc 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-502
```cpp
481:   // With optimization, dead code should already be eliminated. However
482:   // there is one known exception: lowered code for arguments that are only
483:   // used by tail calls, where the tail calls reuse the incoming stack
484:   // arguments directly (see t11 in test/CodeGen/X86/sibcall.ll).
485:   addPass(&DeadMachineInstructionElimID);
486:   printAndVerify("After codegen DCE pass");
487:
488:   // Allow targets to insert passes that improve instruction level parallelism,
489:   // like if-conversion. Such passes will typically need dominator trees and
490:   // loop info, just like LICM and CSE below.
491:   if (addILPOpts())
492:     printAndVerify("After ILP optimizations");
493:
494:   addPass(&EarlyMachineLICMID);
495:   addPass(&MachineCSELegacyID);
496:
497:   addPass(&MachineSinkingLegacyID);
498:   printAndVerify("After Machine LICM, CSE and Sinking passes");
499:
500:   addPass(&PeepholeOptimizerLegacyID);
501:   printAndVerify("After codegen peephole optimization pass");
502: }
```
- EN: This range implements operational logic in helpers such as addPass, printAndVerify, translating backend policy into executable code.
- CN: 这一段实现了 addPass、printAndVerify 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Target machine objects capture global code-generation policy, data layout, and pass-pipeline construction.
  - CN: TargetMachine 对象负责记录全局代码生成策略、数据布局以及 pass 流水线构建方式。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include cl::init, LLVMInitializeNVPTXTarget, X, Y, PassRegistry::getPassRegistry, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 cl::init, LLVMInitializeNVPTXTarget, X, Y, PassRegistry::getPassRegistry，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXTargetMachine.h`
  - `NVPTX.h`
  - `NVPTXAliasAnalysis.h`
  - `NVPTXAllocaHoisting.h`
  - `NVPTXAtomicLower.h`
  - `NVPTXCtorDtorLowering.h`
  - `NVPTXLowerAggrCopies.h`
  - `NVPTXMachineFunctionInfo.h`
  - `NVPTXTargetObjectFile.h`
  - `NVPTXTargetTransformInfo.h`
  - `TargetInfo/NVPTXTargetInfo.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Analysis/KernelInfo.h`
  - `llvm/Analysis/TargetTransformInfo.h`
  - `llvm/CodeGen/Passes.h`
  - `llvm/CodeGen/TargetPassConfig.h`
  - `llvm/IR/IntrinsicsNVPTX.h`
  - `llvm/MC/TargetRegistry.h`
  - `llvm/Pass.h`
  - `llvm/Passes/PassBuilder.h`
  - `llvm/Support/CommandLine.h`
  - `llvm/Support/Compiler.h`
  - `llvm/Target/TargetMachine.h`
  - `llvm/Target/TargetOptions.h`
  - `llvm/TargetParser/Triple.h`
  - `llvm/Transforms/IPO/ExpandVariadics.h`
  - `llvm/Transforms/Scalar.h`
  - `llvm/Transforms/Scalar/GVN.h`
- System/standard headers / 系统或标准头文件:
  - `cassert`
  - `optional`
  - `string`
