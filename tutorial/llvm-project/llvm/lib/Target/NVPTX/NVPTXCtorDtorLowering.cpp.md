# NVPTXCtorDtorLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXCtorDtorLowering.cpp`
- Repository: `llvm-project`
- Purpose (EN): This pass creates a unified init and fini kernel with the required metadata.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXCtorDtorLowering.cpp - Handle global ctors and dtors --------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: ///
 9: /// \file
10: /// This pass creates a unified init and fini kernel with the required metadata
11: //===----------------------------------------------------------------------===//
12:
13: #include "NVPTXCtorDtorLowering.h"
14: #include "MCTargetDesc/NVPTXBaseInfo.h"
15: #include "NVPTX.h"
16: #include "llvm/ADT/StringExtras.h"
17: #include "llvm/IR/CallingConv.h"
18: #include "llvm/IR/Constants.h"
19: #include "llvm/IR/Function.h"
20: #include "llvm/IR/GlobalVariable.h"
21: #include "llvm/IR/IRBuilder.h"
22: #include "llvm/IR/Module.h"
23: #include "llvm/IR/Value.h"
24: #include "llvm/Pass.h"
25: #include "llvm/Support/CommandLine.h"
26: #include "llvm/Support/MD5.h"
27: #include "llvm/Support/NVVMAttributes.h"
28: #include "llvm/Transforms/Utils/ModuleUtils.h"
29:
30: using namespace llvm;
31:
32: #define DEBUG_TYPE "nvptx-lower-ctor-dtor"
33:
34: static cl::opt<std::string>
35:     GlobalStr("nvptx-lower-global-ctor-dtor-id",
36:               cl::desc("Override unique ID of ctor/dtor globals."),
37:               cl::init(""), cl::Hidden);
38:
39: static cl::opt<bool>
40:     CreateKernels("nvptx-emit-init-fini-kernel",
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 41-80
```cpp
41:                   cl::desc("Emit kernels to call ctor/dtor globals."),
42:                   cl::init(true), cl::Hidden);
43:
44: namespace {
45:
46: static std::string getHash(StringRef Str) {
47:   llvm::MD5 Hasher;
48:   llvm::MD5::MD5Result Hash;
49:   Hasher.update(Str);
50:   Hasher.final(Hash);
51:   return llvm::utohexstr(Hash.low(), /*LowerCase=*/true);
52: }
53:
54: static void addKernelAttrs(Function *F) {
55:   F->addFnAttr(NVVMAttr::MaxClusterRank, "1");
56:   F->addFnAttr(NVVMAttr::MaxNTID, "1");
57:   F->setCallingConv(CallingConv::PTX_Kernel);
58: }
59:
60: static Function *createInitOrFiniKernelFunction(Module &M, bool IsCtor) {
61:   StringRef InitOrFiniKernelName =
62:       IsCtor ? "nvptx$device$init" : "nvptx$device$fini";
63:   if (M.getFunction(InitOrFiniKernelName))
64:     return nullptr;
65:
66:   Function *InitOrFiniKernel = Function::createWithDefaultAttr(
67:       FunctionType::get(Type::getVoidTy(M.getContext()), false),
68:       GlobalValue::WeakODRLinkage, 0, InitOrFiniKernelName, &M);
69:   addKernelAttrs(InitOrFiniKernel);
70:
71:   return InitOrFiniKernel;
72: }
73:
74: // We create the IR required to call each callback in this section. This is
75: // equivalent to the following code. Normally, the linker would provide us with
76: // the definitions of the init and fini array sections. The 'nvlink' linker does
77: // not do this so initializing these values is done by the runtime.
78: //
79: // extern "C" void **__init_array_start = nullptr;
80: // extern "C" void **__init_array_end = nullptr;
```
- EN: This range implements operational logic in helpers such as cl::init, getHash, update, final, translating backend policy into executable code.
- CN: 这一段实现了 cl::init、getHash、update、final 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81: // extern "C" void **__fini_array_start = nullptr;
 82: // extern "C" void **__fini_array_end = nullptr;
 83: //
 84: // using InitCallback = void();
 85: // using FiniCallback = void();
 86: //
 87: // void call_init_array_callbacks() {
 88: //   for (auto start = __init_array_start; start != __init_array_end; ++start)
 89: //     reinterpret_cast<InitCallback *>(*start)();
 90: // }
 91: //
 92: // void call_fini_array_callbacks() {
 93: //   size_t fini_array_size = __fini_array_end - __fini_array_start;
 94: //   for (size_t i = fini_array_size; i > 0; --i)
 95: //     reinterpret_cast<FiniCallback *>(__fini_array_start[i - 1])();
 96: // }
 97: static void createInitOrFiniCalls(Function &F, bool IsCtor) {
 98:   Module &M = *F.getParent();
 99:   LLVMContext &C = M.getContext();
100:
101:   IRBuilder<> IRB(BasicBlock::Create(C, "entry", &F));
102:   auto *LoopBB = BasicBlock::Create(C, "while.entry", &F);
103:   auto *ExitBB = BasicBlock::Create(C, "while.end", &F);
104:   Type *PtrTy = IRB.getPtrTy(llvm::ADDRESS_SPACE_GLOBAL);
105:
106:   auto *Begin = M.getOrInsertGlobal(
107:       IsCtor ? "__init_array_start" : "__fini_array_start",
108:       PointerType::get(C, 0), [&]() {
109:         auto *GV = new GlobalVariable(
110:             M, PointerType::get(C, 0),
111:             /*isConstant=*/false, GlobalValue::WeakAnyLinkage,
112:             Constant::getNullValue(PointerType::get(C, 0)),
113:             IsCtor ? "__init_array_start" : "__fini_array_start",
114:             /*InsertBefore=*/nullptr, GlobalVariable::NotThreadLocal,
115:             /*AddressSpace=*/llvm::ADDRESS_SPACE_GLOBAL);
116:         GV->setVisibility(GlobalVariable::ProtectedVisibility);
117:         return GV;
118:       });
119:   auto *End = M.getOrInsertGlobal(
120:       IsCtor ? "__init_array_end" : "__fini_array_end", PointerType::get(C, 0),
```
- EN: This range implements operational logic in helpers such as createInitOrFiniCalls, getParent, getContext, IRB, translating backend policy into executable code.
- CN: 这一段实现了 createInitOrFiniCalls、getParent、getContext、IRB 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:       [&]() {
122:         auto *GV = new GlobalVariable(
123:             M, PointerType::get(C, 0),
124:             /*isConstant=*/false, GlobalValue::WeakAnyLinkage,
125:             Constant::getNullValue(PointerType::get(C, 0)),
126:             IsCtor ? "__init_array_end" : "__fini_array_end",
127:             /*InsertBefore=*/nullptr, GlobalVariable::NotThreadLocal,
128:             /*AddressSpace=*/llvm::ADDRESS_SPACE_GLOBAL);
129:         GV->setVisibility(GlobalVariable::ProtectedVisibility);
130:         return GV;
131:       });
132:
133:   // The constructor type is suppoed to allow using the argument vectors, but
134:   // for now we just call them with no arguments.
135:   auto *CallBackTy = FunctionType::get(IRB.getVoidTy(), {});
136:
137:   // The destructor array must be called in reverse order. Get an expression to
138:   // the end of the array and iterate backwards in that case.
139:   Value *BeginVal = IRB.CreateLoad(Begin->getType(), Begin, "begin");
140:   Value *EndVal = IRB.CreateLoad(Begin->getType(), End, "stop");
141:   if (!IsCtor) {
142:     Value *OldBeginVal = BeginVal;
143:     BeginVal =
144:         IRB.CreateInBoundsGEP(PointerType::get(C, 0), EndVal,
145:                               ArrayRef<Value *>(ConstantInt::getAllOnesValue(
146:                                   IntegerType::getInt64Ty(C))),
147:                               "start");
148:     EndVal = OldBeginVal;
149:   }
150:   IRB.CreateCondBr(
151:       IRB.CreateCmp(IsCtor ? ICmpInst::ICMP_NE : ICmpInst::ICMP_UGE, BeginVal,
152:                     EndVal),
153:       LoopBB, ExitBB);
154:   IRB.SetInsertPoint(LoopBB);
155:   auto *CallBackPHI = IRB.CreatePHI(PtrTy, 2, "ptr");
156:   auto *CallBack = IRB.CreateLoad(IRB.getPtrTy(F.getAddressSpace()),
157:                                   CallBackPHI, "callback");
158:   IRB.CreateCall(CallBackTy, CallBack);
159:   auto *NewCallBack =
160:       IRB.CreateConstGEP1_64(PtrTy, CallBackPHI, IsCtor ? 1 : -1, "next");
```
- EN: This range implements operational logic in helpers such as setVisibility, FunctionType::get, CreateLoad, SetInsertPoint, translating backend policy into executable code.
- CN: 这一段实现了 setVisibility、FunctionType::get、CreateLoad、SetInsertPoint 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:   auto *EndCmp = IRB.CreateCmp(IsCtor ? ICmpInst::ICMP_EQ : ICmpInst::ICMP_ULT,
162:                                NewCallBack, EndVal, "end");
163:   CallBackPHI->addIncoming(BeginVal, &F.getEntryBlock());
164:   CallBackPHI->addIncoming(NewCallBack, LoopBB);
165:   IRB.CreateCondBr(EndCmp, ExitBB, LoopBB);
166:   IRB.SetInsertPoint(ExitBB);
167:   IRB.CreateRetVoid();
168: }
169:
170: static bool createInitOrFiniGlobals(Module &M, GlobalVariable *GV,
171:                                     bool IsCtor) {
172:   ConstantArray *GA = dyn_cast<ConstantArray>(GV->getInitializer());
173:   if (!GA || GA->getNumOperands() == 0)
174:     return false;
175:
176:   // NVPTX has no way to emit variables at specific sections or support for
177:   // the traditional constructor sections. Instead, we emit mangled global
178:   // names so the runtime can build the list manually.
179:   for (Value *V : GA->operands()) {
180:     auto *CS = cast<ConstantStruct>(V);
181:     auto *F = cast<Constant>(CS->getOperand(1));
182:     uint64_t Priority = cast<ConstantInt>(CS->getOperand(0))->getSExtValue();
183:     std::string PriorityStr = "." + std::to_string(Priority);
184:     // We append a semi-unique hash and the priority to the global name.
185:     std::string GlobalID =
186:         !GlobalStr.empty() ? GlobalStr : getHash(M.getSourceFileName());
187:     std::string NameStr =
188:         ((IsCtor ? "__init_array_object_" : "__fini_array_object_") +
189:          F->getName() + "_" + GlobalID + "_" + std::to_string(Priority))
190:             .str();
191:     // PTX does not support exported names with '.' in them.
192:     llvm::transform(NameStr, NameStr.begin(),
193:                     [](char c) { return c == '.' ? '_' : c; });
194:
195:     auto *GV = new GlobalVariable(M, F->getType(), /*IsConstant=*/true,
196:                                   GlobalValue::ExternalLinkage, F, NameStr,
197:                                   nullptr, GlobalValue::NotThreadLocal,
198:                                   /*AddressSpace=*/4);
199:     // This isn't respected by Nvidia, simply put here for clarity.
200:     GV->setSection(IsCtor ? ".init_array" + PriorityStr
```
- EN: This range implements operational logic in helpers such as addIncoming, CreateCondBr, SetInsertPoint, CreateRetVoid, translating backend policy into executable code.
- CN: 这一段实现了 addIncoming、CreateCondBr、SetInsertPoint、CreateRetVoid 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-240
```cpp
201:                           : ".fini_array" + PriorityStr);
202:     GV->setVisibility(GlobalVariable::ProtectedVisibility);
203:     appendToUsed(M, {GV});
204:   }
205:
206:   return true;
207: }
208:
209: static bool createInitOrFiniKernel(Module &M, StringRef GlobalName,
210:                                    bool IsCtor) {
211:   GlobalVariable *GV = M.getGlobalVariable(GlobalName);
212:   if (!GV || !GV->hasInitializer())
213:     return false;
214:
215:   if (!createInitOrFiniGlobals(M, GV, IsCtor))
216:     return false;
217:
218:   if (!CreateKernels)
219:     return true;
220:
221:   Function *InitOrFiniKernel = createInitOrFiniKernelFunction(M, IsCtor);
222:   if (!InitOrFiniKernel)
223:     return false;
224:
225:   createInitOrFiniCalls(*InitOrFiniKernel, IsCtor);
226:
227:   GV->eraseFromParent();
228:   return true;
229: }
230:
231: static bool lowerCtorsAndDtors(Module &M) {
232:   bool Modified = false;
233:   Modified |= createInitOrFiniKernel(M, "llvm.global_ctors", /*IsCtor =*/true);
234:   Modified |= createInitOrFiniKernel(M, "llvm.global_dtors", /*IsCtor =*/false);
235:   return Modified;
236: }
237:
238: class NVPTXCtorDtorLoweringLegacy final : public ModulePass {
239: public:
240:   static char ID;
```
- EN: This range defines or declares important types such as setVisibility, appendToUsed, getGlobalVariable, createInitOrFiniKernelFunction, shaping the data model used by NVPTXCtorDtorLowering.cpp.
- CN: 这一段定义或声明了 setVisibility、appendToUsed、getGlobalVariable、createInitOrFiniKernelFunction 等关键类型，构成 NVPTXCtorDtorLowering.cpp 使用的数据模型。

### Lines 241-259
```cpp
241:   NVPTXCtorDtorLoweringLegacy() : ModulePass(ID) {}
242:   bool runOnModule(Module &M) override { return lowerCtorsAndDtors(M); }
243: };
244:
245: } // End anonymous namespace
246:
247: PreservedAnalyses NVPTXCtorDtorLoweringPass::run(Module &M,
248:                                                  ModuleAnalysisManager &AM) {
249:   return lowerCtorsAndDtors(M) ? PreservedAnalyses::none()
250:                                : PreservedAnalyses::all();
251: }
252:
253: char NVPTXCtorDtorLoweringLegacy::ID = 0;
254: INITIALIZE_PASS(NVPTXCtorDtorLoweringLegacy, DEBUG_TYPE,
255:                 "Lower ctors and dtors for NVPTX", false, false)
256:
257: ModulePass *llvm::createNVPTXCtorDtorLoweringLegacyPass() {
258:   return new NVPTXCtorDtorLoweringLegacy();
259: }
```
- EN: This range implements operational logic in helpers such as NVPTXCtorDtorLoweringLegacy, runOnModule, lowerCtorsAndDtors, PreservedAnalyses::all, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXCtorDtorLoweringLegacy、runOnModule、lowerCtorsAndDtors、PreservedAnalyses::all 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include cl::init, getHash, update, final, llvm::utohexstr, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 cl::init, getHash, update, final, llvm::utohexstr，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXCtorDtorLowering.h`
  - `MCTargetDesc/NVPTXBaseInfo.h`
  - `NVPTX.h`
- LLVM infrastructure / LLVM 基础设施:
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
  - `llvm/Support/NVVMAttributes.h`
  - `llvm/Transforms/Utils/ModuleUtils.h`
