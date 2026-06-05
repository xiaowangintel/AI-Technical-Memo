# SPIRVCtorDtorLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVCtorDtorLowering.cpp`
- Repository: `llvm-project`
- Purpose (EN): This pass creates a unified init and fini kernel with the required metadata to call global constructors and destructors on SPIR-V targets.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- SPIRVCtorDtorLowering.cpp - Handle global ctors and dtors --------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This pass creates a unified init and fini kernel with the required metadata
10: // to call global constructors and destructors on SPIR-V targets.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "SPIRVCtorDtorLowering.h"
15: #include "MCTargetDesc/SPIRVBaseInfo.h"
16: #include "SPIRV.h"
17: #include "llvm/ADT/STLFunctionalExtras.h"
18: #include "llvm/ADT/StringExtras.h"
19: #include "llvm/IR/CallingConv.h"
20: #include "llvm/IR/Constants.h"
21: #include "llvm/IR/Function.h"
22: #include "llvm/IR/GlobalVariable.h"
23: #include "llvm/IR/IRBuilder.h"
24: #include "llvm/IR/Module.h"
25: #include "llvm/IR/Value.h"
26: #include "llvm/Pass.h"
27: #include "llvm/Support/CommandLine.h"
28: #include "llvm/Support/MD5.h"
29: #include "llvm/Transforms/IPO/OpenMPOpt.h"
30: #include "llvm/Transforms/Utils/ModuleUtils.h"
31:
32: using namespace llvm;
33:
34: #define DEBUG_TYPE "spirv-lower-ctor-dtor"
35:
36: static cl::opt<std::string>
37:     GlobalStr("spirv-lower-global-ctor-dtor-id",
38:               cl::desc("Override unique ID of ctor/dtor globals."),
39:               cl::init(""), cl::Hidden);
40:
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 41-80
```cpp
41: static cl::opt<bool>
42:     CreateKernels("spirv-emit-init-fini-kernel",
43:                   cl::desc("Emit kernels to call ctor/dtor globals."),
44:                   cl::init(true), cl::Hidden);
45:
46: namespace {
47: constexpr int SPIRV_GLOBAL_AS = 1;
48:
49: std::string getHash(StringRef Str) {
50:   llvm::MD5 Hasher;
51:   llvm::MD5::MD5Result Hash;
52:   Hasher.update(Str);
53:   Hasher.final(Hash);
54:   return llvm::utohexstr(Hash.low(), /*LowerCase=*/true);
55: }
56:
57: void addKernelAttrs(Function *F) {
58:   F->setCallingConv(CallingConv::SPIR_KERNEL);
59:   F->addFnAttr("uniform-work-group-size", "true");
60: }
61:
62: Function *createInitOrFiniKernelFunction(Module &M, bool IsCtor) {
63:   StringRef InitOrFiniKernelName =
64:       IsCtor ? "spirv$device$init" : "spirv$device$fini";
65:   if (M.getFunction(InitOrFiniKernelName))
66:     return nullptr;
67:
68:   Function *InitOrFiniKernel = Function::createWithDefaultAttr(
69:       FunctionType::get(Type::getVoidTy(M.getContext()), false),
70:       GlobalValue::WeakODRLinkage, 0, InitOrFiniKernelName, &M);
71:   addKernelAttrs(InitOrFiniKernel);
72:
73:   return InitOrFiniKernel;
74: }
75:
76: // We create the IR required to call each callback in this section. This is
77: // equivalent to the following code. Normally, the linker would provide us with
78: // the definitions of the init and fini array sections. The 'spirv-link' linker
79: // does not do this so initializing these values is done by the offload runtime.
80: //
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 81-120
```cpp
 81: // extern "C" void **__init_array_start = nullptr;
 82: // extern "C" void **__init_array_end = nullptr;
 83: // extern "C" void **__fini_array_start = nullptr;
 84: // extern "C" void **__fini_array_end = nullptr;
 85: //
 86: // using InitCallback = void();
 87: // using FiniCallback = void();
 88: //
 89: // void call_init_array_callbacks() {
 90: //   for (auto start = __init_array_start; start != __init_array_end; ++start)
 91: //     reinterpret_cast<InitCallback *>(*start)();
 92: // }
 93: //
 94: // void call_fini_array_callbacks() {
 95: //   size_t fini_array_size = __fini_array_end - __fini_array_start;
 96: //   for (size_t i = fini_array_size; i > 0; --i)
 97: //     reinterpret_cast<FiniCallback *>(__fini_array_start[i - 1])();
 98: // }
 99: void createInitOrFiniCalls(Function &F, bool IsCtor) {
100:   Module &M = *F.getParent();
101:   LLVMContext &C = M.getContext();
102:
103:   IRBuilder<> IRB(BasicBlock::Create(C, "entry", &F));
104:   auto *LoopBB = BasicBlock::Create(C, "while.entry", &F);
105:   auto *ExitBB = BasicBlock::Create(C, "while.end", &F);
106:   Type *PtrTy = IRB.getPtrTy(SPIRV_GLOBAL_AS);
107:
108:   auto CreateGlobal = [&](const char *Name) -> GlobalVariable * {
109:     auto *GV = new GlobalVariable(
110:         M, PointerType::getUnqual(C),
111:         /*isConstant=*/false, GlobalValue::WeakAnyLinkage,
112:         Constant::getNullValue(PointerType::getUnqual(C)), Name,
113:         /*InsertBefore=*/nullptr, GlobalVariable::NotThreadLocal,
114:         /*AddressSpace=*/SPIRV_GLOBAL_AS);
115:     GV->setVisibility(GlobalVariable::ProtectedVisibility);
116:     return GV;
117:   };
118:
119:   auto *Begin = M.getOrInsertGlobal(
120:       IsCtor ? "__init_array_start" : "__fini_array_start",
```
- EN: This range implements operational logic in helpers such as createInitOrFiniCalls, getParent, getContext, IRB, translating backend policy into executable code.
- CN: 这一段实现了 createInitOrFiniCalls、getParent、getContext、IRB 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:       PointerType::getUnqual(C), function_ref<GlobalVariable *()>([&]() {
122:         return CreateGlobal(IsCtor ? "__init_array_start"
123:                                    : "__fini_array_start");
124:       }));
125:   auto *End = M.getOrInsertGlobal(
126:       IsCtor ? "__init_array_end" : "__fini_array_end",
127:       PointerType::getUnqual(C), function_ref<GlobalVariable *()>([&]() {
128:         return CreateGlobal(IsCtor ? "__init_array_end" : "__fini_array_end");
129:       }));
130:   auto *CallBackTy = FunctionType::get(IRB.getVoidTy(), {});
131:
132:   // The destructor array must be called in reverse order. Get an expression to
133:   // the end of the array and iterate backwards in that case.
134:   Value *BeginVal = IRB.CreateLoad(Begin->getType(), Begin, "begin");
135:   Value *EndVal = IRB.CreateLoad(Begin->getType(), End, "stop");
136:   if (!IsCtor) {
137:     Value *OldBeginVal = BeginVal;
138:     BeginVal =
139:         IRB.CreateInBoundsGEP(PointerType::getUnqual(C), EndVal,
140:                               ArrayRef<Value *>(ConstantInt::getAllOnesValue(
141:                                   IntegerType::getInt64Ty(C))),
142:                               "start");
143:     EndVal = OldBeginVal;
144:   }
145:   IRB.CreateCondBr(
146:       IRB.CreateCmp(IsCtor ? ICmpInst::ICMP_NE : ICmpInst::ICMP_UGE, BeginVal,
147:                     EndVal),
148:       LoopBB, ExitBB);
149:   IRB.SetInsertPoint(LoopBB);
150:   auto *CallBackPHI = IRB.CreatePHI(PtrTy, 2, "ptr");
151:   auto *CallBack = IRB.CreateLoad(IRB.getPtrTy(F.getAddressSpace()),
152:                                   CallBackPHI, "callback");
153:   IRB.CreateCall(CallBackTy, CallBack);
154:   auto *NewCallBack =
155:       IRB.CreateConstGEP1_64(PtrTy, CallBackPHI, IsCtor ? 1 : -1, "next");
156:   auto *EndCmp = IRB.CreateCmp(IsCtor ? ICmpInst::ICMP_EQ : ICmpInst::ICMP_ULT,
157:                                NewCallBack, EndVal, "end");
158:   CallBackPHI->addIncoming(BeginVal, &F.getEntryBlock());
159:   CallBackPHI->addIncoming(NewCallBack, LoopBB);
160:   IRB.CreateCondBr(EndCmp, ExitBB, LoopBB);
```
- EN: This range implements operational logic in helpers such as PointerType::getUnqual, CreateGlobal, FunctionType::get, CreateLoad, translating backend policy into executable code.
- CN: 这一段实现了 PointerType::getUnqual、CreateGlobal、FunctionType::get、CreateLoad 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:   IRB.SetInsertPoint(ExitBB);
162:   IRB.CreateRetVoid();
163: }
164:
165: bool createInitOrFiniGlobals(Module &M, GlobalVariable *GV, bool IsCtor) {
166:   ConstantArray *GA = dyn_cast<ConstantArray>(GV->getInitializer());
167:   if (!GA || GA->getNumOperands() == 0)
168:     return false;
169:
170:   // SPIR-V has no way to emit variables at specific sections or support for
171:   // the traditional constructor sections. Instead, we emit mangled global
172:   // names so the runtime can build the list manually.
173:   for (Value *V : GA->operands()) {
174:     auto *CS = cast<ConstantStruct>(V);
175:     auto *F = cast<Constant>(CS->getOperand(1));
176:     uint64_t Priority = cast<ConstantInt>(CS->getOperand(0))->getSExtValue();
177:     std::string PriorityStr = "." + std::to_string(Priority);
178:     // We append a semi-unique hash and the priority to the global name.
179:     std::string GlobalID =
180:         !GlobalStr.empty() ? GlobalStr : getHash(M.getSourceFileName());
181:     std::string NameStr =
182:         ((IsCtor ? "__init_array_object_" : "__fini_array_object_") +
183:          F->getName() + "_" + GlobalID + "_" + std::to_string(Priority))
184:             .str();
185:     llvm::transform(NameStr, NameStr.begin(),
186:                     [](char c) { return c == '.' ? '_' : c; });
187:
188:     auto *GV = new GlobalVariable(M, F->getType(), /*IsConstant=*/true,
189:                                   GlobalValue::ExternalLinkage, F, NameStr,
190:                                   nullptr, GlobalValue::NotThreadLocal,
191:                                   /*AddressSpace=*/SPIRV_GLOBAL_AS);
192:     GV->setSection(IsCtor ? ".init_array" + PriorityStr
193:                           : ".fini_array" + PriorityStr);
194:     GV->setVisibility(GlobalVariable::ProtectedVisibility);
195:   }
196:
197:   return true;
198: }
199:
200: bool createInitOrFiniKernel(Module &M, StringRef GlobalName, bool IsCtor) {
```
- EN: This range implements operational logic in helpers such as SetInsertPoint, CreateRetVoid, createInitOrFiniGlobals, getInitializer, translating backend policy into executable code.
- CN: 这一段实现了 SetInsertPoint、CreateRetVoid、createInitOrFiniGlobals、getInitializer 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-240
```cpp
201:   GlobalVariable *GV = M.getGlobalVariable(GlobalName);
202:   if (!GV || !GV->hasInitializer())
203:     return false;
204:
205:   if (!createInitOrFiniGlobals(M, GV, IsCtor))
206:     return false;
207:
208:   if (!CreateKernels)
209:     return true;
210:
211:   Function *InitOrFiniKernel = createInitOrFiniKernelFunction(M, IsCtor);
212:   if (!InitOrFiniKernel)
213:     return false;
214:
215:   createInitOrFiniCalls(*InitOrFiniKernel, IsCtor);
216:
217:   GV->eraseFromParent();
218:   return true;
219: }
220:
221: bool lowerCtorsAndDtors(Module &M) {
222:   // Only run this pass for OpenMP offload compilation
223:   if (!llvm::omp::isOpenMPDevice(M))
224:     return false;
225:
226:   bool Modified = false;
227:   Modified |= createInitOrFiniKernel(M, "llvm.global_ctors", /*IsCtor =*/true);
228:   Modified |= createInitOrFiniKernel(M, "llvm.global_dtors", /*IsCtor =*/false);
229:   return Modified;
230: }
231:
232: class SPIRVCtorDtorLoweringLegacy final : public ModulePass {
233: public:
234:   static char ID;
235:   SPIRVCtorDtorLoweringLegacy() : ModulePass(ID) {}
236:   bool runOnModule(Module &M) override { return lowerCtorsAndDtors(M); }
237: };
238:
239: } // End anonymous namespace
240:
```
- EN: This range defines or declares important types such as getGlobalVariable, createInitOrFiniKernelFunction, createInitOrFiniCalls, eraseFromParent, shaping the data model used by SPIRVCtorDtorLowering.cpp.
- CN: 这一段定义或声明了 getGlobalVariable、createInitOrFiniKernelFunction、createInitOrFiniCalls、eraseFromParent 等关键类型，构成 SPIRVCtorDtorLowering.cpp 使用的数据模型。

### Lines 241-253
```cpp
241: PreservedAnalyses SPIRVCtorDtorLoweringPass::run(Module &M,
242:                                                  ModuleAnalysisManager &AM) {
243:   return lowerCtorsAndDtors(M) ? PreservedAnalyses::none()
244:                                : PreservedAnalyses::all();
245: }
246:
247: char SPIRVCtorDtorLoweringLegacy::ID = 0;
248: INITIALIZE_PASS(SPIRVCtorDtorLoweringLegacy, DEBUG_TYPE,
249:                 "SPIRV lower ctors and dtors", false, false)
250:
251: ModulePass *llvm::createSPIRVCtorDtorLoweringLegacyPass() {
252:   return new SPIRVCtorDtorLoweringLegacy();
253: }
```
- EN: This range implements operational logic in helpers such as lowerCtorsAndDtors, PreservedAnalyses::all, llvm::createSPIRVCtorDtorLoweringLegacyPass, SPIRVCtorDtorLoweringLegacy, translating backend policy into executable code.
- CN: 这一段实现了 lowerCtorsAndDtors、PreservedAnalyses::all、llvm::createSPIRVCtorDtorLoweringLegacyPass、SPIRVCtorDtorLoweringLegacy 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include cl::init, getHash, update, final, llvm::utohexstr, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 cl::init, getHash, update, final, llvm::utohexstr，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVCtorDtorLowering.h`
  - `MCTargetDesc/SPIRVBaseInfo.h`
  - `SPIRV.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/STLFunctionalExtras.h`
  - `llvm/ADT/StringExtras.h`
  - `llvm/IR/CallingConv.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/GlobalVariable.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/Module.h`
  - `llvm/IR/Value.h`
  - `llvm/Pass.h`
  - `llvm/Support/CommandLine.h`
  - `llvm/Support/MD5.h`
  - `llvm/Transforms/IPO/OpenMPOpt.h`
  - `llvm/Transforms/Utils/ModuleUtils.h`
