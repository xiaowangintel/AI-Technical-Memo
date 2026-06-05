# CGCUDANV.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/CGCUDANV.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements the CGCUDANV portion of Clang's CodeGen pipeline.
- **Purpose (CN) / 目的（中文）**: 实现 Clang CodeGen 流水线中的 CGCUDANV 部分。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: //===----- CGCUDANV.cpp - Interface to NVIDIA CUDA Runtime ----------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This provides a class for CUDA code generation targeting the NVIDIA CUDA
10: // runtime library.
11: //
12: //===----------------------------------------------------------------------===//
13: 
14: #include "CGCUDARuntime.h"
15: #include "CGCXXABI.h"
16: #include "CodeGenFunction.h"
17: #include "CodeGenModule.h"
18: #include "clang/AST/CharUnits.h"
19: #include "clang/AST/Decl.h"
20: #include "clang/Basic/Cuda.h"
```
- **EN**: This block imports local CodeGen headers `CGCUDARuntime.h`, `CGCXXABI.h`, `CodeGenFunction.h`, and 1 more; Clang headers `clang/AST/CharUnits.h`, `clang/AST/Decl.h`, `clang/Basic/Cuda.h`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CGCUDARuntime.h`, `CGCXXABI.h`, `CodeGenFunction.h`, and 1 more；Clang 头文件 `clang/AST/CharUnits.h`, `clang/AST/Decl.h`, `clang/Basic/Cuda.h`；包含影响本编译单元构建方式的预处理结构。

### Lines 21-40
```cpp
21: #include "clang/CodeGen/CodeGenABITypes.h"
22: #include "clang/CodeGen/ConstantInitBuilder.h"
23: #include "llvm/ADT/StringRef.h"
24: #include "llvm/Frontend/Offloading/Utility.h"
25: #include "llvm/IR/BasicBlock.h"
26: #include "llvm/IR/Constants.h"
27: #include "llvm/IR/DerivedTypes.h"
28: #include "llvm/IR/ReplaceConstant.h"
29: #include "llvm/Support/Format.h"
30: #include "llvm/Support/VirtualFileSystem.h"
31: 
32: using namespace clang;
33: using namespace CodeGen;
34: 
35: namespace {
36: constexpr unsigned CudaFatMagic = 0x466243b1;
37: constexpr unsigned HIPFatMagic = 0x48495046; // "HIPF"
38: 
39: class CGNVCUDARuntime : public CGCUDARuntime {
40: 
```
- **EN**: This block imports Clang headers `clang/CodeGen/CodeGenABITypes.h`, `clang/CodeGen/ConstantInitBuilder.h`; LLVM headers `llvm/ADT/StringRef.h`, `llvm/Frontend/Offloading/Utility.h`, `llvm/IR/BasicBlock.h`, and 5 more; opens or references namespaces `clang`, `CodeGen`; introduces declarations such as `CGNVCUDARuntime`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/CodeGen/CodeGenABITypes.h`, `clang/CodeGen/ConstantInitBuilder.h`；LLVM 头文件 `llvm/ADT/StringRef.h`, `llvm/Frontend/Offloading/Utility.h`, `llvm/IR/BasicBlock.h`, and 5 more；打开或引用命名空间 `clang`, `CodeGen`；给出诸如 `CGNVCUDARuntime` 的声明；包含影响本编译单元构建方式的预处理结构。

### Lines 41-60
```cpp
41:   /// The prefix used for function calls and section names (CUDA, HIP, LLVM)
42:   StringRef Prefix;
43: 
44: private:
45:   llvm::IntegerType *IntTy, *SizeTy;
46:   llvm::Type *VoidTy;
47:   llvm::PointerType *PtrTy;
48: 
49:   /// Convenience reference to LLVM Context
50:   llvm::LLVMContext &Context;
51:   /// Convenience reference to the current module
52:   llvm::Module &TheModule;
53:   /// Keeps track of kernel launch stubs and handles emitted in this module
54:   struct KernelInfo {
55:     llvm::Function *Kernel; // stub function to help launch kernel
56:     const Decl *D;
57:   };
58:   llvm::SmallVector<KernelInfo, 16> EmittedKernels;
59:   // Map a kernel mangled name to a symbol for identifying kernel in host code
60:   // For CUDA, the symbol for identifying the kernel is the same as the device
```
- **EN**: This block introduces declarations such as `KernelInfo`.
- **CN**: 该代码块给出诸如 `KernelInfo` 的声明。

### Lines 61-80
```cpp
61:   // stub function. For HIP, they are different.
62:   llvm::DenseMap<StringRef, llvm::GlobalValue *> KernelHandles;
63:   // Map a kernel handle to the kernel stub.
64:   llvm::DenseMap<llvm::GlobalValue *, llvm::Function *> KernelStubs;
65:   struct VarInfo {
66:     llvm::GlobalVariable *Var;
67:     const VarDecl *D;
68:     DeviceVarFlags Flags;
69:   };
70:   llvm::SmallVector<VarInfo, 16> DeviceVars;
71:   /// Keeps track of variable containing handle of GPU binary. Populated by
72:   /// ModuleCtorFunction() and used to create corresponding cleanup calls in
73:   /// ModuleDtorFunction()
74:   llvm::GlobalVariable *GpuBinaryHandle = nullptr;
75:   /// Whether we generate relocatable device code.
76:   bool RelocatableDeviceCode;
77:   /// Mangle context for device.
78:   std::unique_ptr<MangleContext> DeviceMC;
79: 
80:   llvm::FunctionCallee getSetupArgumentFn() const;
```
- **EN**: This block introduces declarations such as `VarInfo`; defines callable entry points like `getSetupArgumentFn`.
- **CN**: 该代码块给出诸如 `VarInfo` 的声明；定义可调用入口，例如 `getSetupArgumentFn`。

### Lines 81-100
```cpp
 81:   llvm::FunctionCallee getLaunchFn() const;
 82: 
 83:   llvm::FunctionType *getRegisterGlobalsFnTy() const;
 84:   llvm::FunctionType *getCallbackFnTy() const;
 85:   llvm::FunctionType *getRegisterLinkedBinaryFnTy() const;
 86:   std::string addPrefixToName(StringRef FuncName) const;
 87:   std::string addUnderscoredPrefixToName(StringRef FuncName) const;
 88: 
 89:   /// Creates a function to register all kernel stubs generated in this module.
 90:   llvm::Function *makeRegisterGlobalsFn();
 91: 
 92:   /// Helper function that generates a constant string and returns a pointer to
 93:   /// the start of the string.  The result of this function can be used anywhere
 94:   /// where the C code specifies const char*.
 95:   llvm::Constant *makeConstantString(const std::string &Str,
 96:                                      const std::string &Name = "") {
 97:     return CGM.GetAddrOfConstantCString(Str, Name).getPointer();
 98:   }
 99: 
100:   /// Helper function which generates an initialized constant array from Str,
```
- **EN**: This block defines callable entry points like `getLaunchFn`, `addPrefixToName`, `addUnderscoredPrefixToName`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `getLaunchFn`, `addPrefixToName`, `addUnderscoredPrefixToName`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 101-120
```cpp
101:   /// and optionally sets section name and alignment. AddNull specifies whether
102:   /// the array should nave NUL termination.
103:   llvm::Constant *makeConstantArray(StringRef Str,
104:                                     StringRef Name = "",
105:                                     StringRef SectionName = "",
106:                                     unsigned Alignment = 0,
107:                                     bool AddNull = false) {
108:     llvm::Constant *Value =
109:         llvm::ConstantDataArray::getString(Context, Str, AddNull);
110:     auto *GV = new llvm::GlobalVariable(
111:         TheModule, Value->getType(), /*isConstant=*/true,
112:         llvm::GlobalValue::PrivateLinkage, Value, Name);
113:     if (!SectionName.empty()) {
114:       GV->setSection(SectionName);
115:       // Mark the address as used which make sure that this section isn't
116:       // merged and we will really have it in the object file.
117:       GV->setUnnamedAddr(llvm::GlobalValue::UnnamedAddr::None);
118:     }
119:     if (Alignment)
120:       GV->setAlignment(llvm::Align(Alignment));
```
- **EN**: This block defines callable entry points like `getString`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getString`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 121-140
```cpp
121:     return GV;
122:   }
123: 
124:   /// Helper function that generates an empty dummy function returning void.
125:   llvm::Function *makeDummyFunction(llvm::FunctionType *FnTy) {
126:     assert(FnTy->getReturnType()->isVoidTy() &&
127:            "Can only generate dummy functions returning void!");
128:     llvm::Function *DummyFunc = llvm::Function::Create(
129:         FnTy, llvm::GlobalValue::InternalLinkage, "dummy", &TheModule);
130: 
131:     llvm::BasicBlock *DummyBlock =
132:         llvm::BasicBlock::Create(Context, "", DummyFunc);
133:     CGBuilderTy FuncBuilder(CGM, Context);
134:     FuncBuilder.SetInsertPoint(DummyBlock);
135:     FuncBuilder.CreateRetVoid();
136: 
137:     return DummyFunc;
138:   }
139: 
140:   Address prepareKernelArgs(CodeGenFunction &CGF, FunctionArgList &Args);
```
- **EN**: This block defines callable entry points like `Create`, `FuncBuilder`, `prepareKernelArgs`; returns or forwards computed values for the surrounding LLVM IR emission logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `Create`, `FuncBuilder`, `prepareKernelArgs`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 141-160
```cpp
141:   Address prepareKernelArgsLLVMOffload(CodeGenFunction &CGF,
142:                                        FunctionArgList &Args);
143:   void emitDeviceStubBodyLegacy(CodeGenFunction &CGF, FunctionArgList &Args);
144:   void emitDeviceStubBodyNew(CodeGenFunction &CGF, FunctionArgList &Args);
145:   std::string getDeviceSideName(const NamedDecl *ND) override;
146: 
147:   void registerDeviceVar(const VarDecl *VD, llvm::GlobalVariable &Var,
148:                          bool Extern, bool Constant) {
149:     DeviceVars.push_back({&Var,
150:                           VD,
151:                           {DeviceVarFlags::Variable, Extern, Constant,
152:                            VD->hasAttr<HIPManagedAttr>(),
153:                            /*Normalized*/ false, 0}});
154:   }
155:   void registerDeviceSurf(const VarDecl *VD, llvm::GlobalVariable &Var,
156:                           bool Extern, int Type) {
157:     DeviceVars.push_back({&Var,
158:                           VD,
159:                           {DeviceVarFlags::Surface, Extern, /*Constant*/ false,
160:                            /*Managed*/ false,
```
- **EN**: This block defines callable entry points like `prepareKernelArgsLLVMOffload`, `emitDeviceStubBodyLegacy`, `emitDeviceStubBodyNew`, `getDeviceSideName`, `registerDeviceVar`.
- **CN**: 该代码块定义可调用入口，例如 `prepareKernelArgsLLVMOffload`, `emitDeviceStubBodyLegacy`, `emitDeviceStubBodyNew`, `getDeviceSideName`, `registerDeviceVar`。

### Lines 161-180
```cpp
161:                            /*Normalized*/ false, Type}});
162:   }
163:   void registerDeviceTex(const VarDecl *VD, llvm::GlobalVariable &Var,
164:                          bool Extern, int Type, bool Normalized) {
165:     DeviceVars.push_back({&Var,
166:                           VD,
167:                           {DeviceVarFlags::Texture, Extern, /*Constant*/ false,
168:                            /*Managed*/ false, Normalized, Type}});
169:   }
170: 
171:   /// Creates module constructor function
172:   llvm::Function *makeModuleCtorFunction();
173:   /// Creates module destructor function
174:   llvm::Function *makeModuleDtorFunction();
175:   /// Transform managed variables for device compilation.
176:   void transformManagedVars();
177:   /// Create offloading entries to register globals in RDC mode.
178:   void createOffloadingEntries();
179: 
180: public:
```
- **EN**: This block defines callable entry points like `registerDeviceTex`, `transformManagedVars`, `createOffloadingEntries`.
- **CN**: 该代码块定义可调用入口，例如 `registerDeviceTex`, `transformManagedVars`, `createOffloadingEntries`。

### Lines 181-200
```cpp
181:   CGNVCUDARuntime(CodeGenModule &CGM);
182: 
183:   llvm::GlobalValue *getKernelHandle(llvm::Function *F, GlobalDecl GD) override;
184:   llvm::Function *getKernelStub(llvm::GlobalValue *Handle) override {
185:     auto Loc = KernelStubs.find(Handle);
186:     assert(Loc != KernelStubs.end());
187:     return Loc->second;
188:   }
189:   void emitDeviceStub(CodeGenFunction &CGF, FunctionArgList &Args) override;
190:   void handleVarRegistration(const VarDecl *VD,
191:                              llvm::GlobalVariable &Var) override;
192:   void
193:   internalizeDeviceSideVar(const VarDecl *D,
194:                            llvm::GlobalValue::LinkageTypes &Linkage) override;
195: 
196:   llvm::Function *finalizeModule() override;
197: };
198: 
199: } // end anonymous namespace
200: 
```
- **EN**: This block defines callable entry points like `CGNVCUDARuntime`, `emitDeviceStub`, `handleVarRegistration`, `internalizeDeviceSideVar`; returns or forwards computed values for the surrounding LLVM IR emission logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `CGNVCUDARuntime`, `emitDeviceStub`, `handleVarRegistration`, `internalizeDeviceSideVar`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 201-220
```cpp
201: std::string CGNVCUDARuntime::addPrefixToName(StringRef FuncName) const {
202:   return (Prefix + FuncName).str();
203: }
204: std::string
205: CGNVCUDARuntime::addUnderscoredPrefixToName(StringRef FuncName) const {
206:   return ("__" + Prefix + FuncName).str();
207: }
208: 
209: static std::unique_ptr<MangleContext> InitDeviceMC(CodeGenModule &CGM) {
210:   // If the host and device have different C++ ABIs, mark it as the device
211:   // mangle context so that the mangling needs to retrieve the additional
212:   // device lambda mangling number instead of the regular host one.
213:   if (CGM.getContext().getAuxTargetInfo() &&
214:       CGM.getContext().getTargetInfo().getCXXABI().isMicrosoft() &&
215:       CGM.getContext().getAuxTargetInfo()->getCXXABI().isItaniumFamily()) {
216:     return std::unique_ptr<MangleContext>(
217:         CGM.getContext().createDeviceMangleContext(
218:             *CGM.getContext().getAuxTargetInfo()));
219:   }
220: 
```
- **EN**: This block defines callable entry points like `addPrefixToName`, `addUnderscoredPrefixToName`, `InitDeviceMC`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `addPrefixToName`, `addUnderscoredPrefixToName`, `InitDeviceMC`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 221-240
```cpp
221:   return std::unique_ptr<MangleContext>(CGM.getContext().createMangleContext(
222:       CGM.getContext().getAuxTargetInfo()));
223: }
224: 
225: CGNVCUDARuntime::CGNVCUDARuntime(CodeGenModule &CGM)
226:     : CGCUDARuntime(CGM), Context(CGM.getLLVMContext()),
227:       TheModule(CGM.getModule()),
228:       RelocatableDeviceCode(CGM.getLangOpts().GPURelocatableDeviceCode),
229:       DeviceMC(InitDeviceMC(CGM)) {
230:   IntTy = CGM.IntTy;
231:   SizeTy = CGM.SizeTy;
232:   VoidTy = CGM.VoidTy;
233:   PtrTy = CGM.DefaultPtrTy;
234: 
235:   if (CGM.getLangOpts().OffloadViaLLVM)
236:     Prefix = "llvm";
237:   else if (CGM.getLangOpts().HIP)
238:     Prefix = "hip";
239:   else
240:     Prefix = "cuda";
```
- **EN**: This block defines callable entry points like `CGNVCUDARuntime`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `CGNVCUDARuntime`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 241-260
```cpp
241: }
242: 
243: llvm::FunctionCallee CGNVCUDARuntime::getSetupArgumentFn() const {
244:   // cudaError_t cudaSetupArgument(void *, size_t, size_t)
245:   llvm::Type *Params[] = {PtrTy, SizeTy, SizeTy};
246:   return CGM.CreateRuntimeFunction(
247:       llvm::FunctionType::get(IntTy, Params, false),
248:       addPrefixToName("SetupArgument"));
249: }
250: 
251: llvm::FunctionCallee CGNVCUDARuntime::getLaunchFn() const {
252:   if (CGM.getLangOpts().HIP) {
253:     // hipError_t hipLaunchByPtr(char *);
254:     return CGM.CreateRuntimeFunction(
255:         llvm::FunctionType::get(IntTy, PtrTy, false), "hipLaunchByPtr");
256:   }
257:   // cudaError_t cudaLaunch(char *);
258:   return CGM.CreateRuntimeFunction(llvm::FunctionType::get(IntTy, PtrTy, false),
259:                                    "cudaLaunch");
260: }
```
- **EN**: This block defines callable entry points like `getSetupArgumentFn`, `get`, `getLaunchFn`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getSetupArgumentFn`, `get`, `getLaunchFn`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 261-280
```cpp
261: 
262: llvm::FunctionType *CGNVCUDARuntime::getRegisterGlobalsFnTy() const {
263:   return llvm::FunctionType::get(VoidTy, PtrTy, false);
264: }
265: 
266: llvm::FunctionType *CGNVCUDARuntime::getCallbackFnTy() const {
267:   return llvm::FunctionType::get(VoidTy, PtrTy, false);
268: }
269: 
270: llvm::FunctionType *CGNVCUDARuntime::getRegisterLinkedBinaryFnTy() const {
271:   llvm::Type *Params[] = {llvm::PointerType::getUnqual(Context), PtrTy, PtrTy,
272:                           llvm::PointerType::getUnqual(Context)};
273:   return llvm::FunctionType::get(VoidTy, Params, false);
274: }
275: 
276: std::string CGNVCUDARuntime::getDeviceSideName(const NamedDecl *ND) {
277:   GlobalDecl GD;
278:   // D could be either a kernel or a variable.
279:   if (auto *FD = dyn_cast<FunctionDecl>(ND))
280:     GD = GlobalDecl(FD, KernelReferenceKind::Kernel);
```
- **EN**: This block defines callable entry points like `get`, `getDeviceSideName`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getDeviceSideName`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 281-300
```cpp
281:   else
282:     GD = GlobalDecl(ND);
283:   std::string DeviceSideName;
284:   MangleContext *MC;
285:   if (CGM.getLangOpts().CUDAIsDevice)
286:     MC = &CGM.getCXXABI().getMangleContext();
287:   else
288:     MC = DeviceMC.get();
289:   if (MC->shouldMangleDeclName(ND)) {
290:     SmallString<256> Buffer;
291:     llvm::raw_svector_ostream Out(Buffer);
292:     MC->mangleName(GD, Out);
293:     DeviceSideName = std::string(Out.str());
294:   } else
295:     DeviceSideName = std::string(ND->getIdentifier()->getName());
296: 
297:   // Make unique name for device side static file-scope variable for HIP.
298:   if (CGM.getContext().shouldExternalize(ND) &&
299:       CGM.getLangOpts().GPURelocatableDeviceCode) {
300:     SmallString<256> Buffer;
```
- **EN**: This block defines callable entry points like `Out`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `Out`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 301-320
```cpp
301:     llvm::raw_svector_ostream Out(Buffer);
302:     Out << DeviceSideName;
303:     CGM.printPostfixForExternalizedDecl(Out, ND);
304:     DeviceSideName = std::string(Out.str());
305:   }
306:   return DeviceSideName;
307: }
308: 
309: void CGNVCUDARuntime::emitDeviceStub(CodeGenFunction &CGF,
310:                                      FunctionArgList &Args) {
311:   EmittedKernels.push_back({CGF.CurFn, CGF.CurFuncDecl});
312:   if (auto *GV =
313:           dyn_cast<llvm::GlobalVariable>(KernelHandles[CGF.CurFn->getName()])) {
314:     GV->setLinkage(CGF.CurFn->getLinkage());
315:     GV->setInitializer(CGF.CurFn);
316:   }
317:   if (CudaFeatureEnabled(CGM.getTarget().getSDKVersion(),
318:                          CudaFeature::CUDA_USES_NEW_LAUNCH) ||
319:       (CGF.getLangOpts().HIP && CGF.getLangOpts().HIPUseNewLaunchAPI) ||
320:       (CGF.getLangOpts().OffloadViaLLVM))
```
- **EN**: This block defines callable entry points like `Out`, `emitDeviceStub`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `Out`, `emitDeviceStub`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 321-340
```cpp
321:     emitDeviceStubBodyNew(CGF, Args);
322:   else
323:     emitDeviceStubBodyLegacy(CGF, Args);
324: }
325: 
326: /// CUDA passes the arguments with a level of indirection. For example, a
327: /// (void*, short, void*) is passed as {void **, short *, void **} to the launch
328: /// function. For the LLVM/offload launch we flatten the arguments into the
329: /// struct directly. In addition, we include the size of the arguments, thus
330: /// pass {sizeof({void *, short, void *}), ptr to {void *, short, void *},
331: /// nullptr}. The last nullptr needs to be initialized to an array of pointers
332: /// pointing to the arguments if we want to offload to the host.
333: Address CGNVCUDARuntime::prepareKernelArgsLLVMOffload(CodeGenFunction &CGF,
334:                                                       FunctionArgList &Args) {
335:   SmallVector<llvm::Type *> ArgTypes, KernelLaunchParamsTypes;
336:   for (auto &Arg : Args)
337:     ArgTypes.push_back(CGF.ConvertTypeForMem(Arg->getType()));
338:   llvm::StructType *KernelArgsTy = llvm::StructType::create(ArgTypes);
339: 
340:   auto *Int64Ty = CGF.Builder.getInt64Ty();
```
- **EN**: This block defines callable entry points like `emitDeviceStubBodyNew`, `emitDeviceStubBodyLegacy`, `prepareKernelArgsLLVMOffload`; uses control flow (for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitDeviceStubBodyNew`, `emitDeviceStubBodyLegacy`, `prepareKernelArgsLLVMOffload`；通过控制流（for）细化 LLVM IR 生成 行为。

### Lines 341-360
```cpp
341:   KernelLaunchParamsTypes.push_back(Int64Ty);
342:   KernelLaunchParamsTypes.push_back(PtrTy);
343:   KernelLaunchParamsTypes.push_back(PtrTy);
344: 
345:   llvm::StructType *KernelLaunchParamsTy =
346:       llvm::StructType::create(KernelLaunchParamsTypes);
347:   Address KernelArgs = CGF.CreateTempAllocaWithoutCast(
348:       KernelArgsTy, CharUnits::fromQuantity(16), "kernel_args");
349:   Address KernelLaunchParams = CGF.CreateTempAllocaWithoutCast(
350:       KernelLaunchParamsTy, CharUnits::fromQuantity(16),
351:       "kernel_launch_params");
352: 
353:   auto KernelArgsSize = CGM.getDataLayout().getTypeAllocSize(KernelArgsTy);
354:   CGF.Builder.CreateStore(llvm::ConstantInt::get(Int64Ty, KernelArgsSize),
355:                           CGF.Builder.CreateStructGEP(KernelLaunchParams, 0));
356:   CGF.Builder.CreateStore(KernelArgs.emitRawPointer(CGF),
357:                           CGF.Builder.CreateStructGEP(KernelLaunchParams, 1));
358:   CGF.Builder.CreateStore(llvm::Constant::getNullValue(PtrTy),
359:                           CGF.Builder.CreateStructGEP(KernelLaunchParams, 2));
360: 
```
- **EN**: This block spells out callable entry points like `create`, `fromQuantity`.
- **CN**: 该代码块给出可调用入口的声明，例如 `create`, `fromQuantity`。

### Lines 361-380
```cpp
361:   for (unsigned i = 0; i < Args.size(); ++i) {
362:     auto *ArgVal = CGF.Builder.CreateLoad(CGF.GetAddrOfLocalVar(Args[i]));
363:     CGF.Builder.CreateStore(ArgVal, CGF.Builder.CreateStructGEP(KernelArgs, i));
364:   }
365: 
366:   return KernelLaunchParams;
367: }
368: 
369: Address CGNVCUDARuntime::prepareKernelArgs(CodeGenFunction &CGF,
370:                                            FunctionArgList &Args) {
371:   // Calculate amount of space we will need for all arguments.  If we have no
372:   // args, allocate a single pointer so we still have a valid pointer to the
373:   // argument array that we can pass to runtime, even if it will be unused.
374:   Address KernelArgs = CGF.CreateTempAlloca(
375:       PtrTy, LangAS::Default, CharUnits::fromQuantity(16), "kernel_args",
376:       llvm::ConstantInt::get(SizeTy, std::max<size_t>(1, Args.size())));
377:   // Store pointers to the arguments in a locally allocated launch_args.
378:   for (unsigned i = 0; i < Args.size(); ++i) {
379:     llvm::Value *VarPtr = CGF.GetAddrOfLocalVar(Args[i]).emitRawPointer(CGF);
380:     llvm::Value *VoidVarPtr = CGF.Builder.CreatePointerCast(VarPtr, PtrTy);
```
- **EN**: This block defines callable entry points like `prepareKernelArgs`, `fromQuantity`; uses control flow (for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `prepareKernelArgs`, `fromQuantity`；通过控制流（for）细化 LLVM IR 生成 行为。

### Lines 381-400
```cpp
381:     CGF.Builder.CreateDefaultAlignedStore(
382:         VoidVarPtr, CGF.Builder.CreateConstGEP1_32(
383:                         PtrTy, KernelArgs.emitRawPointer(CGF), i));
384:   }
385:   return KernelArgs;
386: }
387: 
388: // CUDA 9.0+ uses new way to launch kernels. Parameters are packed in a local
389: // array and kernels are launched using cudaLaunchKernel().
390: void CGNVCUDARuntime::emitDeviceStubBodyNew(CodeGenFunction &CGF,
391:                                             FunctionArgList &Args) {
392:   // Build the shadow stack entry at the very start of the function.
393:   Address KernelArgs = CGF.getLangOpts().OffloadViaLLVM
394:                            ? prepareKernelArgsLLVMOffload(CGF, Args)
395:                            : prepareKernelArgs(CGF, Args);
396: 
397:   llvm::BasicBlock *EndBlock = CGF.createBasicBlock("setup.end");
398: 
399:   // Lookup cudaLaunchKernel/hipLaunchKernel function.
400:   // HIP kernel launching API name depends on -fgpu-default-stream option. For
```
- **EN**: This block defines callable entry points like `emitDeviceStubBodyNew`, `prepareKernelArgs`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `emitDeviceStubBodyNew`, `prepareKernelArgs`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 401-420
```cpp
401:   // the default value 'legacy', it is hipLaunchKernel. For 'per-thread',
402:   // it is hipLaunchKernel_spt.
403:   // cudaError_t cudaLaunchKernel(const void *func, dim3 gridDim, dim3 blockDim,
404:   //                              void **args, size_t sharedMem,
405:   //                              cudaStream_t stream);
406:   // hipError_t hipLaunchKernel[_spt](const void *func, dim3 gridDim,
407:   //                                  dim3 blockDim, void **args,
408:   //                                  size_t sharedMem, hipStream_t stream);
409:   TranslationUnitDecl *TUDecl = CGM.getContext().getTranslationUnitDecl();
410:   DeclContext *DC = TranslationUnitDecl::castToDeclContext(TUDecl);
411:   std::string KernelLaunchAPI = "LaunchKernel";
412:   if (CGF.getLangOpts().GPUDefaultStream ==
413:       LangOptions::GPUDefaultStreamKind::PerThread) {
414:     if (CGF.getLangOpts().HIP)
415:       KernelLaunchAPI = KernelLaunchAPI + "_spt";
416:     else if (CGF.getLangOpts().CUDA)
417:       KernelLaunchAPI = KernelLaunchAPI + "_ptsz";
418:   }
419:   auto LaunchKernelName = addPrefixToName(KernelLaunchAPI);
420:   const IdentifierInfo &cudaLaunchKernelII =
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 421-440
```cpp
421:       CGM.getContext().Idents.get(LaunchKernelName);
422:   FunctionDecl *cudaLaunchKernelFD = nullptr;
423:   for (auto *Result : DC->lookup(&cudaLaunchKernelII)) {
424:     if (FunctionDecl *FD = dyn_cast<FunctionDecl>(Result))
425:       cudaLaunchKernelFD = FD;
426:   }
427: 
428:   if (cudaLaunchKernelFD == nullptr) {
429:     CGM.Error(CGF.CurFuncDecl->getLocation(),
430:               "Can't find declaration for " + LaunchKernelName);
431:     return;
432:   }
433:   // Create temporary dim3 grid_dim, block_dim.
434:   ParmVarDecl *GridDimParam = cudaLaunchKernelFD->getParamDecl(1);
435:   QualType Dim3Ty = GridDimParam->getType();
436:   Address GridDim = CGF.CreateMemTempWithoutCast(
437:       Dim3Ty, CharUnits::fromQuantity(8), "grid_dim");
438:   Address BlockDim = CGF.CreateMemTempWithoutCast(
439:       Dim3Ty, CharUnits::fromQuantity(8), "block_dim");
440:   Address ShmemSize = CGF.CreateTempAlloca(SizeTy, LangAS::Default,
```
- **EN**: This block defines callable entry points like `fromQuantity`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `fromQuantity`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 441-460
```cpp
441:                                            CGM.getSizeAlign(), "shmem_size");
442:   Address Stream = CGF.CreateTempAlloca(PtrTy, LangAS::Default,
443:                                         CGM.getPointerAlign(), "stream");
444:   llvm::FunctionCallee cudaPopConfigFn = CGM.CreateRuntimeFunction(
445:       llvm::FunctionType::get(IntTy,
446:                               {/*gridDim=*/GridDim.getType(),
447:                                /*blockDim=*/BlockDim.getType(),
448:                                /*ShmemSize=*/ShmemSize.getType(),
449:                                /*Stream=*/Stream.getType()},
450:                               /*isVarArg=*/false),
451:       addUnderscoredPrefixToName("PopCallConfiguration"));
452: 
453:   CGF.EmitRuntimeCallOrInvoke(cudaPopConfigFn, {GridDim.emitRawPointer(CGF),
454:                                                 BlockDim.emitRawPointer(CGF),
455:                                                 ShmemSize.emitRawPointer(CGF),
456:                                                 Stream.emitRawPointer(CGF)});
457: 
458:   // Emit the call to cudaLaunch
459:   llvm::Value *Kernel =
460:       CGF.Builder.CreatePointerCast(KernelHandles[CGF.CurFn->getName()], PtrTy);
```
- **EN**: This block defines callable entry points like `addUnderscoredPrefixToName`.
- **CN**: 该代码块定义可调用入口，例如 `addUnderscoredPrefixToName`。

### Lines 461-480
```cpp
461:   CallArgList LaunchKernelArgs;
462:   LaunchKernelArgs.add(RValue::get(Kernel),
463:                        cudaLaunchKernelFD->getParamDecl(0)->getType());
464:   LaunchKernelArgs.add(RValue::getAggregate(GridDim), Dim3Ty);
465:   LaunchKernelArgs.add(RValue::getAggregate(BlockDim), Dim3Ty);
466:   LaunchKernelArgs.add(RValue::get(KernelArgs, CGF),
467:                        cudaLaunchKernelFD->getParamDecl(3)->getType());
468:   LaunchKernelArgs.add(RValue::get(CGF.Builder.CreateLoad(ShmemSize)),
469:                        cudaLaunchKernelFD->getParamDecl(4)->getType());
470:   LaunchKernelArgs.add(RValue::get(CGF.Builder.CreateLoad(Stream)),
471:                        cudaLaunchKernelFD->getParamDecl(5)->getType());
472: 
473:   QualType QT = cudaLaunchKernelFD->getType();
474:   QualType CQT = QT.getCanonicalType();
475:   llvm::Type *Ty = CGM.getTypes().ConvertType(CQT);
476:   llvm::FunctionType *FTy = cast<llvm::FunctionType>(Ty);
477: 
478:   const CGFunctionInfo &FI =
479:       CGM.getTypes().arrangeFunctionDeclaration(cudaLaunchKernelFD);
480:   llvm::FunctionCallee cudaLaunchKernelFn =
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding LLVM IR emission implementation.
- **CN**: 该代码块为周围的 LLVM IR 生成 实现提供必要的胶水代码、布局或分隔结构。

### Lines 481-500
```cpp
481:       CGM.CreateRuntimeFunction(FTy, LaunchKernelName);
482:   CGF.EmitCall(FI, CGCallee::forDirect(cudaLaunchKernelFn), ReturnValueSlot(),
483:                LaunchKernelArgs);
484: 
485:   // To prevent CUDA device stub functions from being merged by ICF in MSVC
486:   // environment, create an unique global variable for each kernel and write to
487:   // the variable in the device stub.
488:   if (CGM.getContext().getTargetInfo().getCXXABI().isMicrosoft() &&
489:       !CGF.getLangOpts().HIP) {
490:     llvm::Function *KernelFunction = llvm::cast<llvm::Function>(Kernel);
491:     std::string GlobalVarName = (KernelFunction->getName() + ".id").str();
492: 
493:     llvm::GlobalVariable *HandleVar =
494:         CGM.getModule().getNamedGlobal(GlobalVarName);
495:     if (!HandleVar) {
496:       HandleVar = new llvm::GlobalVariable(
497:           CGM.getModule(), CGM.Int8Ty,
498:           /*Constant=*/false, KernelFunction->getLinkage(),
499:           llvm::ConstantInt::get(CGM.Int8Ty, 0), GlobalVarName);
500:       HandleVar->setDSOLocal(KernelFunction->isDSOLocal());
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 501-520
```cpp
501:       HandleVar->setVisibility(KernelFunction->getVisibility());
502:       if (KernelFunction->hasComdat())
503:         HandleVar->setComdat(CGM.getModule().getOrInsertComdat(GlobalVarName));
504:     }
505: 
506:     CGF.Builder.CreateAlignedStore(llvm::ConstantInt::get(CGM.Int8Ty, 1),
507:                                    HandleVar, CharUnits::One(),
508:                                    /*IsVolatile=*/true);
509:   }
510: 
511:   CGF.EmitBranch(EndBlock);
512: 
513:   CGF.EmitBlock(EndBlock);
514: }
515: 
516: void CGNVCUDARuntime::emitDeviceStubBodyLegacy(CodeGenFunction &CGF,
517:                                                FunctionArgList &Args) {
518:   // Emit a call to cudaSetupArgument for each arg in Args.
519:   llvm::FunctionCallee cudaSetupArgFn = getSetupArgumentFn();
520:   llvm::BasicBlock *EndBlock = CGF.createBasicBlock("setup.end");
```
- **EN**: This block defines callable entry points like `emitDeviceStubBodyLegacy`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitDeviceStubBodyLegacy`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 521-540
```cpp
521:   CharUnits Offset = CharUnits::Zero();
522:   for (const VarDecl *A : Args) {
523:     auto TInfo = CGM.getContext().getTypeInfoInChars(A->getType());
524:     Offset = Offset.alignTo(TInfo.Align);
525:     llvm::Value *Args[] = {
526:         CGF.Builder.CreatePointerCast(
527:             CGF.GetAddrOfLocalVar(A).emitRawPointer(CGF), PtrTy),
528:         llvm::ConstantInt::get(SizeTy, TInfo.Width.getQuantity()),
529:         llvm::ConstantInt::get(SizeTy, Offset.getQuantity()),
530:     };
531:     llvm::CallBase *CB = CGF.EmitRuntimeCallOrInvoke(cudaSetupArgFn, Args);
532:     llvm::Constant *Zero = llvm::ConstantInt::get(IntTy, 0);
533:     llvm::Value *CBZero = CGF.Builder.CreateICmpEQ(CB, Zero);
534:     llvm::BasicBlock *NextBlock = CGF.createBasicBlock("setup.next");
535:     CGF.Builder.CreateCondBr(CBZero, NextBlock, EndBlock);
536:     CGF.EmitBlock(NextBlock);
537:     Offset += TInfo.Width;
538:   }
539: 
540:   // Emit the call to cudaLaunch
```
- **EN**: This block uses control flow (for) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（for）细化 LLVM IR 生成 行为。

### Lines 541-560
```cpp
541:   llvm::FunctionCallee cudaLaunchFn = getLaunchFn();
542:   llvm::Value *Arg =
543:       CGF.Builder.CreatePointerCast(KernelHandles[CGF.CurFn->getName()], PtrTy);
544:   CGF.EmitRuntimeCallOrInvoke(cudaLaunchFn, Arg);
545:   CGF.EmitBranch(EndBlock);
546: 
547:   CGF.EmitBlock(EndBlock);
548: }
549: 
550: // Replace the original variable Var with the address loaded from variable
551: // ManagedVar populated by HIP runtime.
552: static void replaceManagedVar(llvm::GlobalVariable *Var,
553:                               llvm::GlobalVariable *ManagedVar) {
554:   SmallVector<SmallVector<llvm::User *, 8>, 8> WorkList;
555:   for (auto &&VarUse : Var->uses()) {
556:     WorkList.push_back({VarUse.getUser()});
557:   }
558:   while (!WorkList.empty()) {
559:     auto &&WorkItem = WorkList.pop_back_val();
560:     auto *U = WorkItem.back();
```
- **EN**: This block defines callable entry points like `replaceManagedVar`; uses control flow (for, while) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `replaceManagedVar`；通过控制流（for, while）细化 LLVM IR 生成 行为。

### Lines 561-580
```cpp
561:     if (isa<llvm::ConstantExpr>(U)) {
562:       for (auto &&UU : U->uses()) {
563:         WorkItem.push_back(UU.getUser());
564:         WorkList.push_back(WorkItem);
565:         WorkItem.pop_back();
566:       }
567:       continue;
568:     }
569:     if (auto *I = dyn_cast<llvm::Instruction>(U)) {
570:       llvm::Value *OldV = Var;
571:       llvm::Instruction *NewV = new llvm::LoadInst(
572:           Var->getType(), ManagedVar, "ld.managed", false,
573:           llvm::Align(Var->getAlignment()), I->getIterator());
574:       WorkItem.pop_back();
575:       // Replace constant expressions directly or indirectly using the managed
576:       // variable with instructions.
577:       for (auto &&Op : WorkItem) {
578:         auto *CE = cast<llvm::ConstantExpr>(Op);
579:         auto *NewInst = CE->getAsInstruction();
580:         NewInst->insertBefore(*I->getParent(), I->getIterator());
```
- **EN**: This block defines callable entry points like `Align`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `Align`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 581-600
```cpp
581:         NewInst->replaceUsesOfWith(OldV, NewV);
582:         OldV = CE;
583:         NewV = NewInst;
584:       }
585:       I->replaceUsesOfWith(OldV, NewV);
586:     } else {
587:       llvm_unreachable("Invalid use of managed variable");
588:     }
589:   }
590: }
591: 
592: /// Creates a function that sets up state on the host side for CUDA objects that
593: /// have a presence on both the host and device sides. Specifically, registers
594: /// the host side of kernel functions and device global variables with the CUDA
595: /// runtime.
596: /// \code
597: /// void __cuda_register_globals(void** GpuBinaryHandle) {
598: ///    __cudaRegisterFunction(GpuBinaryHandle,Kernel0,...);
599: ///    ...
600: ///    __cudaRegisterFunction(GpuBinaryHandle,KernelM,...);
```
- **EN**: This block guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块使用断言或不可达标记保护关键不变量。

### Lines 601-620
```cpp
601: ///    __cudaRegisterVar(GpuBinaryHandle, GlobalVar0, ...);
602: ///    ...
603: ///    __cudaRegisterVar(GpuBinaryHandle, GlobalVarN, ...);
604: /// }
605: /// \endcode
606: llvm::Function *CGNVCUDARuntime::makeRegisterGlobalsFn() {
607:   // No need to register anything
608:   if (EmittedKernels.empty() && DeviceVars.empty())
609:     return nullptr;
610: 
611:   llvm::Function *RegisterKernelsFunc = llvm::Function::Create(
612:       getRegisterGlobalsFnTy(), llvm::GlobalValue::InternalLinkage,
613:       addUnderscoredPrefixToName("_register_globals"), &TheModule);
614:   llvm::BasicBlock *EntryBB =
615:       llvm::BasicBlock::Create(Context, "entry", RegisterKernelsFunc);
616:   CGBuilderTy Builder(CGM, Context);
617:   Builder.SetInsertPoint(EntryBB);
618: 
619:   // void __cudaRegisterFunction(void **, const char *, char *, const char *,
620:   //                             int, uint3*, uint3*, dim3*, dim3*, int*)
```
- **EN**: This block defines callable entry points like `getRegisterGlobalsFnTy`, `Create`, `Builder`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getRegisterGlobalsFnTy`, `Create`, `Builder`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 621-640
```cpp
621:   llvm::Type *RegisterFuncParams[] = {
622:       PtrTy, PtrTy, PtrTy, PtrTy, IntTy,
623:       PtrTy, PtrTy, PtrTy, PtrTy, llvm::PointerType::getUnqual(Context)};
624:   llvm::FunctionCallee RegisterFunc = CGM.CreateRuntimeFunction(
625:       llvm::FunctionType::get(IntTy, RegisterFuncParams, false),
626:       addUnderscoredPrefixToName("RegisterFunction"));
627: 
628:   // Extract GpuBinaryHandle passed as the first argument passed to
629:   // __cuda_register_globals() and generate __cudaRegisterFunction() call for
630:   // each emitted kernel.
631:   llvm::Argument &GpuBinaryHandlePtr = *RegisterKernelsFunc->arg_begin();
632:   for (auto &&I : EmittedKernels) {
633:     llvm::Constant *KernelName =
634:         makeConstantString(getDeviceSideName(cast<NamedDecl>(I.D)));
635:     llvm::Constant *NullPtr = llvm::ConstantPointerNull::get(PtrTy);
636:     llvm::Value *Args[] = {
637:         &GpuBinaryHandlePtr,
638:         KernelHandles[I.Kernel->getName()],
639:         KernelName,
640:         KernelName,
```
- **EN**: This block defines callable entry points like `get`, `makeConstantString`; uses control flow (for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `makeConstantString`；通过控制流（for）细化 LLVM IR 生成 行为。

### Lines 641-660
```cpp
641:         llvm::ConstantInt::getAllOnesValue(IntTy),
642:         NullPtr,
643:         NullPtr,
644:         NullPtr,
645:         NullPtr,
646:         llvm::ConstantPointerNull::get(llvm::PointerType::getUnqual(Context))};
647:     Builder.CreateCall(RegisterFunc, Args);
648:   }
649: 
650:   llvm::Type *VarSizeTy = IntTy;
651:   // For HIP or CUDA 9.0+, device variable size is type of `size_t`.
652:   if (CGM.getLangOpts().HIP ||
653:       ToCudaVersion(CGM.getTarget().getSDKVersion()) >= CudaVersion::CUDA_90)
654:     VarSizeTy = SizeTy;
655: 
656:   // void __cudaRegisterVar(void **, char *, char *, const char *,
657:   //                        int, int, int, int)
658:   llvm::Type *RegisterVarParams[] = {PtrTy, PtrTy,     PtrTy, PtrTy,
659:                                      IntTy, VarSizeTy, IntTy, IntTy};
660:   llvm::FunctionCallee RegisterVar = CGM.CreateRuntimeFunction(
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 661-680
```cpp
661:       llvm::FunctionType::get(VoidTy, RegisterVarParams, false),
662:       addUnderscoredPrefixToName("RegisterVar"));
663:   // void __hipRegisterManagedVar(void **, char *, char *, const char *,
664:   //                              size_t, unsigned)
665:   llvm::Type *RegisterManagedVarParams[] = {PtrTy, PtrTy,     PtrTy,
666:                                             PtrTy, VarSizeTy, IntTy};
667:   llvm::FunctionCallee RegisterManagedVar = CGM.CreateRuntimeFunction(
668:       llvm::FunctionType::get(VoidTy, RegisterManagedVarParams, false),
669:       addUnderscoredPrefixToName("RegisterManagedVar"));
670:   // void __cudaRegisterSurface(void **, const struct surfaceReference *,
671:   //                            const void **, const char *, int, int);
672:   llvm::FunctionCallee RegisterSurf = CGM.CreateRuntimeFunction(
673:       llvm::FunctionType::get(
674:           VoidTy, {PtrTy, PtrTy, PtrTy, PtrTy, IntTy, IntTy}, false),
675:       addUnderscoredPrefixToName("RegisterSurface"));
676:   // void __cudaRegisterTexture(void **, const struct textureReference *,
677:   //                            const void **, const char *, int, int, int)
678:   llvm::FunctionCallee RegisterTex = CGM.CreateRuntimeFunction(
679:       llvm::FunctionType::get(
680:           VoidTy, {PtrTy, PtrTy, PtrTy, PtrTy, IntTy, IntTy, IntTy}, false),
```
- **EN**: This block defines callable entry points like `get`, `addUnderscoredPrefixToName`.
- **CN**: 该代码块定义可调用入口，例如 `get`, `addUnderscoredPrefixToName`。

### Lines 681-700
```cpp
681:       addUnderscoredPrefixToName("RegisterTexture"));
682:   for (auto &&Info : DeviceVars) {
683:     llvm::GlobalVariable *Var = Info.Var;
684:     assert((!Var->isDeclaration() || Info.Flags.isManaged()) &&
685:            "External variables should not show up here, except HIP managed "
686:            "variables");
687:     llvm::Constant *VarName = makeConstantString(getDeviceSideName(Info.D));
688:     switch (Info.Flags.getKind()) {
689:     case DeviceVarFlags::Variable: {
690:       uint64_t VarSize =
691:           CGM.getDataLayout().getTypeAllocSize(Var->getValueType());
692:       if (Info.Flags.isManaged()) {
693:         assert(Var->getName().ends_with(".managed") &&
694:                "HIP managed variables not transformed");
695:         auto *ManagedVar = CGM.getModule().getNamedGlobal(
696:             Var->getName().drop_back(StringRef(".managed").size()));
697:         llvm::Value *Args[] = {
698:             &GpuBinaryHandlePtr,
699:             ManagedVar,
700:             Var,
```
- **EN**: This block defines callable entry points like `addUnderscoredPrefixToName`; uses control flow (if, switch, for, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `addUnderscoredPrefixToName`；通过控制流（if, switch, for, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 701-720
```cpp
701:             VarName,
702:             llvm::ConstantInt::get(VarSizeTy, VarSize),
703:             llvm::ConstantInt::get(IntTy, Var->getAlignment())};
704:         if (!Var->isDeclaration())
705:           Builder.CreateCall(RegisterManagedVar, Args);
706:       } else {
707:         llvm::Value *Args[] = {
708:             &GpuBinaryHandlePtr,
709:             Var,
710:             VarName,
711:             VarName,
712:             llvm::ConstantInt::get(IntTy, Info.Flags.isExtern()),
713:             llvm::ConstantInt::get(VarSizeTy, VarSize),
714:             llvm::ConstantInt::get(IntTy, Info.Flags.isConstant()),
715:             llvm::ConstantInt::get(IntTy, 0)};
716:         Builder.CreateCall(RegisterVar, Args);
717:       }
718:       break;
719:     }
720:     case DeviceVarFlags::Surface:
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 721-740
```cpp
721:       Builder.CreateCall(
722:           RegisterSurf,
723:           {&GpuBinaryHandlePtr, Var, VarName, VarName,
724:            llvm::ConstantInt::get(IntTy, Info.Flags.getSurfTexType()),
725:            llvm::ConstantInt::get(IntTy, Info.Flags.isExtern())});
726:       break;
727:     case DeviceVarFlags::Texture:
728:       Builder.CreateCall(
729:           RegisterTex,
730:           {&GpuBinaryHandlePtr, Var, VarName, VarName,
731:            llvm::ConstantInt::get(IntTy, Info.Flags.getSurfTexType()),
732:            llvm::ConstantInt::get(IntTy, Info.Flags.isNormalized()),
733:            llvm::ConstantInt::get(IntTy, Info.Flags.isExtern())});
734:       break;
735:     }
736:   }
737: 
738:   Builder.CreateRetVoid();
739:   return RegisterKernelsFunc;
740: }
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 741-760
```cpp
741: 
742: /// Creates a global constructor function for the module:
743: ///
744: /// For CUDA:
745: /// \code
746: /// void __cuda_module_ctor() {
747: ///     Handle = __cudaRegisterFatBinary(GpuBinaryBlob);
748: ///     __cuda_register_globals(Handle);
749: /// }
750: /// \endcode
751: ///
752: /// For HIP:
753: /// \code
754: /// void __hip_module_ctor() {
755: ///     if (__hip_gpubin_handle == 0) {
756: ///         __hip_gpubin_handle  = __hipRegisterFatBinary(GpuBinaryBlob);
757: ///         __hip_register_globals(__hip_gpubin_handle);
758: ///     }
759: /// }
760: /// \endcode
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 761-780
```cpp
761: llvm::Function *CGNVCUDARuntime::makeModuleCtorFunction() {
762:   bool IsHIP = CGM.getLangOpts().HIP;
763:   bool IsCUDA = CGM.getLangOpts().CUDA;
764:   // No need to generate ctors/dtors if there is no GPU binary.
765:   StringRef CudaGpuBinaryFileName = CGM.getCodeGenOpts().CudaGpuBinaryFileName;
766:   if (CudaGpuBinaryFileName.empty() && !IsHIP)
767:     return nullptr;
768:   if ((IsHIP || (IsCUDA && !RelocatableDeviceCode)) && EmittedKernels.empty() &&
769:       DeviceVars.empty())
770:     return nullptr;
771: 
772:   // void __{cuda|hip}_register_globals(void* handle);
773:   llvm::Function *RegisterGlobalsFunc = makeRegisterGlobalsFn();
774:   // We always need a function to pass in as callback. Create a dummy
775:   // implementation if we don't need to register anything.
776:   if (RelocatableDeviceCode && !RegisterGlobalsFunc)
777:     RegisterGlobalsFunc = makeDummyFunction(getRegisterGlobalsFnTy());
778: 
779:   // void ** __{cuda|hip}RegisterFatBinary(void *);
780:   llvm::FunctionCallee RegisterFatbinFunc = CGM.CreateRuntimeFunction(
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 781-800
```cpp
781:       llvm::FunctionType::get(PtrTy, PtrTy, false),
782:       addUnderscoredPrefixToName("RegisterFatBinary"));
783:   // struct { int magic, int version, void * gpu_binary, void * dont_care };
784:   llvm::StructType *FatbinWrapperTy =
785:       llvm::StructType::get(IntTy, IntTy, PtrTy, PtrTy);
786: 
787:   // Register GPU binary with the CUDA runtime, store returned handle in a
788:   // global variable and save a reference in GpuBinaryHandle to be cleaned up
789:   // in destructor on exit. Then associate all known kernels with the GPU binary
790:   // handle so CUDA runtime can figure out what to call on the GPU side.
791:   std::unique_ptr<llvm::MemoryBuffer> CudaGpuBinary = nullptr;
792:   if (!CudaGpuBinaryFileName.empty()) {
793:     auto VFS = CGM.getFileSystem();
794:     auto CudaGpuBinaryOrErr =
795:         VFS->getBufferForFile(CudaGpuBinaryFileName, -1, false);
796:     if (std::error_code EC = CudaGpuBinaryOrErr.getError()) {
797:       CGM.getDiags().Report(diag::err_cannot_open_file)
798:           << CudaGpuBinaryFileName << EC.message();
799:       return nullptr;
800:     }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 801-820
```cpp
801:     CudaGpuBinary = std::move(CudaGpuBinaryOrErr.get());
802:   }
803: 
804:   llvm::Function *ModuleCtorFunc = llvm::Function::Create(
805:       llvm::FunctionType::get(VoidTy, false),
806:       llvm::GlobalValue::InternalLinkage,
807:       addUnderscoredPrefixToName("_module_ctor"), &TheModule);
808:   llvm::BasicBlock *CtorEntryBB =
809:       llvm::BasicBlock::Create(Context, "entry", ModuleCtorFunc);
810:   CGBuilderTy CtorBuilder(CGM, Context);
811: 
812:   CtorBuilder.SetInsertPoint(CtorEntryBB);
813: 
814:   const char *FatbinConstantName;
815:   const char *FatbinSectionName;
816:   const char *ModuleIDSectionName;
817:   StringRef ModuleIDPrefix;
818:   llvm::Constant *FatBinStr;
819:   unsigned FatMagic;
820:   if (IsHIP) {
```
- **EN**: This block defines callable entry points like `get`, `Create`, `CtorBuilder`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `Create`, `CtorBuilder`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 821-840
```cpp
821:     // On macOS (Mach-O), section names must be in "segment,section" format.
822:     FatbinConstantName =
823:         CGM.getTriple().isMacOSX() ? "__HIP,__hip_fatbin" : ".hip_fatbin";
824:     FatbinSectionName =
825:         CGM.getTriple().isMacOSX() ? "__HIP,__fatbin" : ".hipFatBinSegment";
826: 
827:     ModuleIDSectionName =
828:         CGM.getTriple().isMacOSX() ? "__HIP,__module_id" : "__hip_module_id";
829:     ModuleIDPrefix = "__hip_";
830: 
831:     if (CudaGpuBinary) {
832:       // If fatbin is available from early finalization, create a string
833:       // literal containing the fat binary loaded from the given file.
834:       const unsigned HIPCodeObjectAlign = 4096;
835:       FatBinStr = makeConstantArray(std::string(CudaGpuBinary->getBuffer()), "",
836:                                     FatbinConstantName, HIPCodeObjectAlign);
837:     } else {
838:       // If fatbin is not available, create an external symbol
839:       // __hip_fatbin in section .hip_fatbin. The external symbol is supposed
840:       // to contain the fat binary but will be populated somewhere else,
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 841-860
```cpp
841:       // e.g. by lld through link script.
842:       FatBinStr = new llvm::GlobalVariable(
843:           CGM.getModule(), CGM.Int8Ty,
844:           /*isConstant=*/true, llvm::GlobalValue::ExternalLinkage, nullptr,
845:           "__hip_fatbin" + (CGM.getLangOpts().CUID.empty()
846:                                 ? ""
847:                                 : "_" + CGM.getContext().getCUIDHash()),
848:           nullptr, llvm::GlobalVariable::NotThreadLocal);
849:       cast<llvm::GlobalVariable>(FatBinStr)->setSection(FatbinConstantName);
850:     }
851: 
852:     FatMagic = HIPFatMagic;
853:   } else {
854:     if (RelocatableDeviceCode)
855:       FatbinConstantName = CGM.getTriple().isMacOSX()
856:                                ? "__NV_CUDA,__nv_relfatbin"
857:                                : "__nv_relfatbin";
858:     else
859:       FatbinConstantName =
860:           CGM.getTriple().isMacOSX() ? "__NV_CUDA,__nv_fatbin" : ".nv_fatbin";
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 861-880
```cpp
861:     // NVIDIA's cuobjdump looks for fatbins in this section.
862:     FatbinSectionName =
863:         CGM.getTriple().isMacOSX() ? "__NV_CUDA,__fatbin" : ".nvFatBinSegment";
864: 
865:     ModuleIDSectionName = CGM.getTriple().isMacOSX()
866:                               ? "__NV_CUDA,__nv_module_id"
867:                               : "__nv_module_id";
868:     ModuleIDPrefix = "__nv_";
869: 
870:     // For CUDA, create a string literal containing the fat binary loaded from
871:     // the given file.
872:     FatBinStr = makeConstantArray(std::string(CudaGpuBinary->getBuffer()), "",
873:                                   FatbinConstantName, 8);
874:     FatMagic = CudaFatMagic;
875:   }
876: 
877:   // Create initialized wrapper structure that points to the loaded GPU binary
878:   ConstantInitBuilder Builder(CGM);
879:   auto Values = Builder.beginStruct(FatbinWrapperTy);
880:   // Fatbin wrapper magic.
```
- **EN**: This block spells out callable entry points like `Builder`.
- **CN**: 该代码块给出可调用入口的声明，例如 `Builder`。

### Lines 881-900
```cpp
881:   Values.addInt(IntTy, FatMagic);
882:   // Fatbin version.
883:   Values.addInt(IntTy, 1);
884:   // Data.
885:   Values.add(FatBinStr);
886:   // Unused in fatbin v1.
887:   Values.add(llvm::ConstantPointerNull::get(PtrTy));
888:   llvm::GlobalVariable *FatbinWrapper = Values.finishAndCreateGlobal(
889:       addUnderscoredPrefixToName("_fatbin_wrapper"), CGM.getPointerAlign(),
890:       /*constant*/ true);
891:   FatbinWrapper->setSection(FatbinSectionName);
892:   CGM.getSanitizerMetadata()->disableSanitizerForGlobal(FatbinWrapper);
893: 
894:   // There is only one HIP fat binary per linked module, however there are
895:   // multiple constructor functions. Make sure the fat binary is registered
896:   // only once. The constructor functions are executed by the dynamic loader
897:   // before the program gains control. The dynamic loader cannot execute the
898:   // constructor functions concurrently since doing that would not guarantee
899:   // thread safety of the loaded program. Therefore we can assume sequential
900:   // execution of constructor functions here.
```
- **EN**: This block spells out callable entry points like `addUnderscoredPrefixToName`.
- **CN**: 该代码块给出可调用入口的声明，例如 `addUnderscoredPrefixToName`。

### Lines 901-920
```cpp
901:   if (IsHIP) {
902:     auto Linkage = RelocatableDeviceCode ? llvm::GlobalValue::ExternalLinkage
903:                                          : llvm::GlobalValue::InternalLinkage;
904:     llvm::BasicBlock *IfBlock =
905:         llvm::BasicBlock::Create(Context, "if", ModuleCtorFunc);
906:     llvm::BasicBlock *ExitBlock =
907:         llvm::BasicBlock::Create(Context, "exit", ModuleCtorFunc);
908:     // The name, size, and initialization pattern of this variable is part
909:     // of HIP ABI.
910:     GpuBinaryHandle = new llvm::GlobalVariable(
911:         TheModule, PtrTy, /*isConstant=*/false, Linkage,
912:         /*Initializer=*/
913:         !RelocatableDeviceCode ? llvm::ConstantPointerNull::get(PtrTy)
914:                                : nullptr,
915:         "__hip_gpubin_handle" + (CGM.getLangOpts().CUID.empty()
916:                                      ? ""
917:                                      : "_" + CGM.getContext().getCUIDHash()));
918:     GpuBinaryHandle->setAlignment(CGM.getPointerAlign().getAsAlign());
919:     // Prevent the weak symbol in different shared libraries being merged.
920:     if (Linkage != llvm::GlobalValue::InternalLinkage)
```
- **EN**: This block defines callable entry points like `Create`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `Create`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 921-940
```cpp
921:       GpuBinaryHandle->setVisibility(llvm::GlobalValue::HiddenVisibility);
922:     Address GpuBinaryAddr(
923:         GpuBinaryHandle, PtrTy,
924:         CharUnits::fromQuantity(GpuBinaryHandle->getAlignment()));
925:     {
926:       auto *HandleValue = CtorBuilder.CreateLoad(GpuBinaryAddr);
927:       llvm::Constant *Zero =
928:           llvm::Constant::getNullValue(HandleValue->getType());
929:       llvm::Value *EQZero = CtorBuilder.CreateICmpEQ(HandleValue, Zero);
930:       CtorBuilder.CreateCondBr(EQZero, IfBlock, ExitBlock);
931:     }
932:     {
933:       CtorBuilder.SetInsertPoint(IfBlock);
934:       // GpuBinaryHandle = __hipRegisterFatBinary(&FatbinWrapper);
935:       llvm::CallInst *RegisterFatbinCall =
936:           CtorBuilder.CreateCall(RegisterFatbinFunc, FatbinWrapper);
937:       CtorBuilder.CreateStore(RegisterFatbinCall, GpuBinaryAddr);
938:       CtorBuilder.CreateBr(ExitBlock);
939:     }
940:     {
```
- **EN**: This block defines callable entry points like `GpuBinaryAddr`, `getNullValue`.
- **CN**: 该代码块定义可调用入口，例如 `GpuBinaryAddr`, `getNullValue`。

### Lines 941-960
```cpp
941:       CtorBuilder.SetInsertPoint(ExitBlock);
942:       // Call __hip_register_globals(GpuBinaryHandle);
943:       if (RegisterGlobalsFunc) {
944:         auto *HandleValue = CtorBuilder.CreateLoad(GpuBinaryAddr);
945:         CtorBuilder.CreateCall(RegisterGlobalsFunc, HandleValue);
946:       }
947:     }
948:   } else if (!RelocatableDeviceCode) {
949:     // Register binary with CUDA runtime. This is substantially different in
950:     // default mode vs. separate compilation!
951:     // GpuBinaryHandle = __cudaRegisterFatBinary(&FatbinWrapper);
952:     llvm::CallInst *RegisterFatbinCall =
953:         CtorBuilder.CreateCall(RegisterFatbinFunc, FatbinWrapper);
954:     GpuBinaryHandle = new llvm::GlobalVariable(
955:         TheModule, PtrTy, false, llvm::GlobalValue::InternalLinkage,
956:         llvm::ConstantPointerNull::get(PtrTy), "__cuda_gpubin_handle");
957:     GpuBinaryHandle->setAlignment(CGM.getPointerAlign().getAsAlign());
958:     CtorBuilder.CreateAlignedStore(RegisterFatbinCall, GpuBinaryHandle,
959:                                    CGM.getPointerAlign());
960: 
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 961-980
```cpp
961:     // Call __cuda_register_globals(GpuBinaryHandle);
962:     if (RegisterGlobalsFunc)
963:       CtorBuilder.CreateCall(RegisterGlobalsFunc, RegisterFatbinCall);
964: 
965:     // Call __cudaRegisterFatBinaryEnd(Handle) if this CUDA version needs it.
966:     if (CudaFeatureEnabled(CGM.getTarget().getSDKVersion(),
967:                            CudaFeature::CUDA_USES_FATBIN_REGISTER_END)) {
968:       // void __cudaRegisterFatBinaryEnd(void **);
969:       llvm::FunctionCallee RegisterFatbinEndFunc = CGM.CreateRuntimeFunction(
970:           llvm::FunctionType::get(VoidTy, PtrTy, false),
971:           "__cudaRegisterFatBinaryEnd");
972:       CtorBuilder.CreateCall(RegisterFatbinEndFunc, RegisterFatbinCall);
973:     }
974:   } else {
975:     // Generate a unique module ID.
976:     SmallString<64> ModuleID;
977:     llvm::raw_svector_ostream OS(ModuleID);
978:     OS << ModuleIDPrefix << llvm::format("%" PRIx64, FatbinWrapper->getGUID());
979:     llvm::Constant *ModuleIDConstant = makeConstantArray(
980:         std::string(ModuleID), "", ModuleIDSectionName, 32, /*AddNull=*/true);
```
- **EN**: This block defines callable entry points like `get`, `OS`, `format`, `string`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `OS`, `format`, `string`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 981-1000
```cpp
 981: 
 982:     // Create an alias for the FatbinWrapper that nvcc will look for.
 983:     llvm::GlobalAlias::create(llvm::GlobalValue::ExternalLinkage,
 984:                               Twine("__fatbinwrap") + ModuleID, FatbinWrapper);
 985: 
 986:     // void __cudaRegisterLinkedBinary%ModuleID%(void (*)(void *), void *,
 987:     // void *, void (*)(void **))
 988:     SmallString<128> RegisterLinkedBinaryName("__cudaRegisterLinkedBinary");
 989:     RegisterLinkedBinaryName += ModuleID;
 990:     llvm::FunctionCallee RegisterLinkedBinaryFunc = CGM.CreateRuntimeFunction(
 991:         getRegisterLinkedBinaryFnTy(), RegisterLinkedBinaryName);
 992: 
 993:     assert(RegisterGlobalsFunc && "Expecting at least dummy function!");
 994:     llvm::Value *Args[] = {RegisterGlobalsFunc, FatbinWrapper, ModuleIDConstant,
 995:                            makeDummyFunction(getCallbackFnTy())};
 996:     CtorBuilder.CreateCall(RegisterLinkedBinaryFunc, Args);
 997:   }
 998: 
 999:   // Create destructor and register it with atexit() the way NVCC does it. Doing
1000:   // it during regular destructor phase worked in CUDA before 9.2 but results in
```
- **EN**: This block defines callable entry points like `create`, `RegisterLinkedBinaryName`, `getRegisterLinkedBinaryFnTy`; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `create`, `RegisterLinkedBinaryName`, `getRegisterLinkedBinaryFnTy`；使用断言或不可达标记保护关键不变量。

### Lines 1001-1020
```cpp
1001:   // double-free in 9.2.
1002:   if (llvm::Function *CleanupFn = makeModuleDtorFunction()) {
1003:     // extern "C" int atexit(void (*f)(void));
1004:     llvm::FunctionType *AtExitTy =
1005:         llvm::FunctionType::get(IntTy, CleanupFn->getType(), false);
1006:     llvm::FunctionCallee AtExitFunc =
1007:         CGM.CreateRuntimeFunction(AtExitTy, "atexit", llvm::AttributeList(),
1008:                                   /*Local=*/true);
1009:     CtorBuilder.CreateCall(AtExitFunc, CleanupFn);
1010:   }
1011: 
1012:   CtorBuilder.CreateRetVoid();
1013:   return ModuleCtorFunc;
1014: }
1015: 
1016: /// Creates a global destructor function that unregisters the GPU code blob
1017: /// registered by constructor.
1018: ///
1019: /// For CUDA:
1020: /// \code
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1021-1040
```cpp
1021: /// void __cuda_module_dtor() {
1022: ///     __cudaUnregisterFatBinary(Handle);
1023: /// }
1024: /// \endcode
1025: ///
1026: /// For HIP:
1027: /// \code
1028: /// void __hip_module_dtor() {
1029: ///     if (__hip_gpubin_handle) {
1030: ///         __hipUnregisterFatBinary(__hip_gpubin_handle);
1031: ///         __hip_gpubin_handle = 0;
1032: ///     }
1033: /// }
1034: /// \endcode
1035: llvm::Function *CGNVCUDARuntime::makeModuleDtorFunction() {
1036:   // No need for destructor if we don't have a handle to unregister.
1037:   if (!GpuBinaryHandle)
1038:     return nullptr;
1039: 
1040:   // void __cudaUnregisterFatBinary(void ** handle);
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1041-1060
```cpp
1041:   llvm::FunctionCallee UnregisterFatbinFunc = CGM.CreateRuntimeFunction(
1042:       llvm::FunctionType::get(VoidTy, PtrTy, false),
1043:       addUnderscoredPrefixToName("UnregisterFatBinary"));
1044: 
1045:   llvm::Function *ModuleDtorFunc = llvm::Function::Create(
1046:       llvm::FunctionType::get(VoidTy, false),
1047:       llvm::GlobalValue::InternalLinkage,
1048:       addUnderscoredPrefixToName("_module_dtor"), &TheModule);
1049: 
1050:   llvm::BasicBlock *DtorEntryBB =
1051:       llvm::BasicBlock::Create(Context, "entry", ModuleDtorFunc);
1052:   CGBuilderTy DtorBuilder(CGM, Context);
1053:   DtorBuilder.SetInsertPoint(DtorEntryBB);
1054: 
1055:   Address GpuBinaryAddr(
1056:       GpuBinaryHandle, GpuBinaryHandle->getValueType(),
1057:       CharUnits::fromQuantity(GpuBinaryHandle->getAlignment()));
1058:   auto *HandleValue = DtorBuilder.CreateLoad(GpuBinaryAddr);
1059:   // There is only one HIP fat binary per linked module, however there are
1060:   // multiple destructor functions. Make sure the fat binary is unregistered
```
- **EN**: This block spells out callable entry points like `get`, `Create`, `DtorBuilder`, `GpuBinaryAddr`.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`, `Create`, `DtorBuilder`, `GpuBinaryAddr`。

### Lines 1061-1080
```cpp
1061:   // only once.
1062:   if (CGM.getLangOpts().HIP) {
1063:     llvm::BasicBlock *IfBlock =
1064:         llvm::BasicBlock::Create(Context, "if", ModuleDtorFunc);
1065:     llvm::BasicBlock *ExitBlock =
1066:         llvm::BasicBlock::Create(Context, "exit", ModuleDtorFunc);
1067:     llvm::Constant *Zero = llvm::Constant::getNullValue(HandleValue->getType());
1068:     llvm::Value *NEZero = DtorBuilder.CreateICmpNE(HandleValue, Zero);
1069:     DtorBuilder.CreateCondBr(NEZero, IfBlock, ExitBlock);
1070: 
1071:     DtorBuilder.SetInsertPoint(IfBlock);
1072:     DtorBuilder.CreateCall(UnregisterFatbinFunc, HandleValue);
1073:     DtorBuilder.CreateStore(Zero, GpuBinaryAddr);
1074:     DtorBuilder.CreateBr(ExitBlock);
1075: 
1076:     DtorBuilder.SetInsertPoint(ExitBlock);
1077:   } else {
1078:     DtorBuilder.CreateCall(UnregisterFatbinFunc, HandleValue);
1079:   }
1080:   DtorBuilder.CreateRetVoid();
```
- **EN**: This block defines callable entry points like `Create`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `Create`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1081-1100
```cpp
1081:   return ModuleDtorFunc;
1082: }
1083: 
1084: CGCUDARuntime *CodeGen::CreateNVCUDARuntime(CodeGenModule &CGM) {
1085:   return new CGNVCUDARuntime(CGM);
1086: }
1087: 
1088: void CGNVCUDARuntime::internalizeDeviceSideVar(
1089:     const VarDecl *D, llvm::GlobalValue::LinkageTypes &Linkage) {
1090:   // For -fno-gpu-rdc, host-side shadows of external declarations of device-side
1091:   // global variables become internal definitions. These have to be internal in
1092:   // order to prevent name conflicts with global host variables with the same
1093:   // name in a different TUs.
1094:   //
1095:   // For -fgpu-rdc, the shadow variables should not be internalized because
1096:   // they may be accessed by different TU.
1097:   if (CGM.getLangOpts().GPURelocatableDeviceCode)
1098:     return;
1099: 
1100:   // __shared__ variables are odd. Shadows do get created, but
```
- **EN**: This block defines callable entry points like `CGNVCUDARuntime`, `internalizeDeviceSideVar`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `CGNVCUDARuntime`, `internalizeDeviceSideVar`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1101-1120
```cpp
1101:   // they are not registered with the CUDA runtime, so they
1102:   // can't really be used to access their device-side
1103:   // counterparts. It's not clear yet whether it's nvcc's bug or
1104:   // a feature, but we've got to do the same for compatibility.
1105:   if (D->hasAttr<CUDADeviceAttr>() || D->hasAttr<CUDAConstantAttr>() ||
1106:       D->hasAttr<CUDASharedAttr>() ||
1107:       D->getType()->isCUDADeviceBuiltinSurfaceType() ||
1108:       D->getType()->isCUDADeviceBuiltinTextureType()) {
1109:     Linkage = llvm::GlobalValue::InternalLinkage;
1110:   }
1111: }
1112: 
1113: void CGNVCUDARuntime::handleVarRegistration(const VarDecl *D,
1114:                                             llvm::GlobalVariable &GV) {
1115:   if (D->hasAttr<CUDADeviceAttr>() || D->hasAttr<CUDAConstantAttr>()) {
1116:     // Shadow variables and their properties must be registered with CUDA
1117:     // runtime. Skip Extern global variables, which will be registered in
1118:     // the TU where they are defined.
1119:     //
1120:     // Don't register a C++17 inline variable. The local symbol can be
```
- **EN**: This block defines callable entry points like `handleVarRegistration`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `handleVarRegistration`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1121-1140
```cpp
1121:     // discarded and referencing a discarded local symbol from outside the
1122:     // comdat (__cuda_register_globals) is disallowed by the ELF spec.
1123:     //
1124:     // HIP managed variables need to be always recorded in device and host
1125:     // compilations for transformation.
1126:     //
1127:     // HIP managed variables and variables in CUDADeviceVarODRUsedByHost are
1128:     // added to llvm.compiler-used, therefore they are safe to be registered.
1129:     if ((!D->hasExternalStorage() && !D->isInline()) ||
1130:         CGM.getContext().CUDADeviceVarODRUsedByHost.contains(D) ||
1131:         D->hasAttr<HIPManagedAttr>()) {
1132:       registerDeviceVar(D, GV, !D->hasDefinition(),
1133:                         D->hasAttr<CUDAConstantAttr>());
1134:     }
1135:   } else if (D->getType()->isCUDADeviceBuiltinSurfaceType() ||
1136:              D->getType()->isCUDADeviceBuiltinTextureType()) {
1137:     // Builtin surfaces and textures and their template arguments are
1138:     // also registered with CUDA runtime.
1139:     const auto *TD = cast<ClassTemplateSpecializationDecl>(
1140:         D->getType()->castAsCXXRecordDecl());
```
- **EN**: This block defines callable entry points like `registerDeviceVar`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `registerDeviceVar`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1141-1160
```cpp
1141:     const TemplateArgumentList &Args = TD->getTemplateArgs();
1142:     if (TD->hasAttr<CUDADeviceBuiltinSurfaceTypeAttr>()) {
1143:       assert(Args.size() == 2 &&
1144:              "Unexpected number of template arguments of CUDA device "
1145:              "builtin surface type.");
1146:       auto SurfType = Args[1].getAsIntegral();
1147:       if (!D->hasExternalStorage())
1148:         registerDeviceSurf(D, GV, !D->hasDefinition(), SurfType.getSExtValue());
1149:     } else {
1150:       assert(Args.size() == 3 &&
1151:              "Unexpected number of template arguments of CUDA device "
1152:              "builtin texture type.");
1153:       auto TexType = Args[1].getAsIntegral();
1154:       auto Normalized = Args[2].getAsIntegral();
1155:       if (!D->hasExternalStorage())
1156:         registerDeviceTex(D, GV, !D->hasDefinition(), TexType.getSExtValue(),
1157:                           Normalized.getZExtValue());
1158:     }
1159:   }
1160: }
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1161-1180
```cpp
1161: 
1162: // Transform managed variables to pointers to managed variables in device code.
1163: // Each use of the original managed variable is replaced by a load from the
1164: // transformed managed variable. The transformed managed variable contains
1165: // the address of managed memory which will be allocated by the runtime.
1166: void CGNVCUDARuntime::transformManagedVars() {
1167:   for (auto &&Info : DeviceVars) {
1168:     llvm::GlobalVariable *Var = Info.Var;
1169:     if (Info.Flags.getKind() == DeviceVarFlags::Variable &&
1170:         Info.Flags.isManaged()) {
1171:       auto *ManagedVar = new llvm::GlobalVariable(
1172:           CGM.getModule(), Var->getType(),
1173:           /*isConstant=*/false, Var->getLinkage(),
1174:           /*Init=*/Var->isDeclaration()
1175:               ? nullptr
1176:               : llvm::ConstantPointerNull::get(Var->getType()),
1177:           /*Name=*/"", /*InsertBefore=*/nullptr,
1178:           llvm::GlobalVariable::NotThreadLocal,
1179:           CGM.getContext().getTargetAddressSpace(CGM.getLangOpts().CUDAIsDevice
1180:                                                      ? LangAS::cuda_device
```
- **EN**: This block defines callable entry points like `transformManagedVars`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `transformManagedVars`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1181-1200
```cpp
1181:                                                      : LangAS::Default));
1182:       ManagedVar->setDSOLocal(Var->isDSOLocal());
1183:       ManagedVar->setVisibility(Var->getVisibility());
1184:       ManagedVar->setExternallyInitialized(true);
1185:       replaceManagedVar(Var, ManagedVar);
1186:       ManagedVar->takeName(Var);
1187:       Var->setName(Twine(ManagedVar->getName()) + ".managed");
1188:       // Keep managed variables even if they are not used in device code since
1189:       // they need to be allocated by the runtime.
1190:       if (CGM.getLangOpts().CUDAIsDevice && !Var->isDeclaration()) {
1191:         assert(!ManagedVar->isDeclaration());
1192:         CGM.addCompilerUsedGlobal(Var);
1193:         CGM.addCompilerUsedGlobal(ManagedVar);
1194:       }
1195:     }
1196:   }
1197: }
1198: 
1199: // Creates offloading entries for all the kernels and globals that must be
1200: // registered. The linker will provide a pointer to this section so we can
```
- **EN**: This block defines callable entry points like `replaceManagedVar`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `replaceManagedVar`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1201-1220
```cpp
1201: // register the symbols with the linked device image.
1202: void CGNVCUDARuntime::createOffloadingEntries() {
1203:   llvm::object::OffloadKind Kind = CGM.getLangOpts().HIP
1204:                                        ? llvm::object::OffloadKind::OFK_HIP
1205:                                        : llvm::object::OffloadKind::OFK_Cuda;
1206:   // For now, just spoof this as OpenMP because that's the runtime it uses.
1207:   if (CGM.getLangOpts().OffloadViaLLVM)
1208:     Kind = llvm::object::OffloadKind::OFK_OpenMP;
1209: 
1210:   llvm::Module &M = CGM.getModule();
1211:   for (KernelInfo &I : EmittedKernels)
1212:     llvm::offloading::emitOffloadingEntry(
1213:         M, Kind, KernelHandles[I.Kernel->getName()],
1214:         getDeviceSideName(cast<NamedDecl>(I.D)), /*Flags=*/0, /*Data=*/0,
1215:         llvm::offloading::OffloadGlobalEntry);
1216: 
1217:   for (VarInfo &I : DeviceVars) {
1218:     uint64_t VarSize =
1219:         CGM.getDataLayout().getTypeAllocSize(I.Var->getValueType());
1220:     int32_t Flags =
```
- **EN**: This block defines callable entry points like `createOffloadingEntries`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `createOffloadingEntries`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1221-1240
```cpp
1221:         (I.Flags.isExtern()
1222:              ? static_cast<int32_t>(llvm::offloading::OffloadGlobalExtern)
1223:              : 0) |
1224:         (I.Flags.isConstant()
1225:              ? static_cast<int32_t>(llvm::offloading::OffloadGlobalConstant)
1226:              : 0) |
1227:         (I.Flags.isNormalized()
1228:              ? static_cast<int32_t>(llvm::offloading::OffloadGlobalNormalized)
1229:              : 0);
1230:     if (I.Flags.getKind() == DeviceVarFlags::Variable) {
1231:       if (I.Flags.isManaged()) {
1232:         assert(I.Var->getName().ends_with(".managed") &&
1233:                "HIP managed variables not transformed");
1234: 
1235:         auto *ManagedVar = M.getNamedGlobal(
1236:             I.Var->getName().drop_back(StringRef(".managed").size()));
1237:         llvm::offloading::emitOffloadingEntry(
1238:             M, Kind, I.Var, getDeviceSideName(I.D), VarSize,
1239:             llvm::offloading::OffloadGlobalManagedEntry | Flags,
1240:             /*Data=*/I.Var->getAlignment(), ManagedVar);
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1241-1260
```cpp
1241:       } else {
1242:         llvm::offloading::emitOffloadingEntry(
1243:             M, Kind, I.Var, getDeviceSideName(I.D), VarSize,
1244:             llvm::offloading::OffloadGlobalEntry | Flags,
1245:             /*Data=*/0);
1246:       }
1247:     } else if (I.Flags.getKind() == DeviceVarFlags::Surface) {
1248:       llvm::offloading::emitOffloadingEntry(
1249:           M, Kind, I.Var, getDeviceSideName(I.D), VarSize,
1250:           llvm::offloading::OffloadGlobalSurfaceEntry | Flags,
1251:           I.Flags.getSurfTexType());
1252:     } else if (I.Flags.getKind() == DeviceVarFlags::Texture) {
1253:       llvm::offloading::emitOffloadingEntry(
1254:           M, Kind, I.Var, getDeviceSideName(I.D), VarSize,
1255:           llvm::offloading::OffloadGlobalTextureEntry | Flags,
1256:           I.Flags.getSurfTexType());
1257:     }
1258:   }
1259: }
1260: 
```
- **EN**: This block defines callable entry points like `emitOffloadingEntry`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitOffloadingEntry`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1261-1280
```cpp
1261: // Returns module constructor to be added.
1262: llvm::Function *CGNVCUDARuntime::finalizeModule() {
1263:   transformManagedVars();
1264:   if (CGM.getLangOpts().CUDAIsDevice) {
1265:     // Mark ODR-used device variables as compiler used to prevent it from being
1266:     // eliminated by optimization. This is necessary for device variables
1267:     // ODR-used by host functions. Sema correctly marks them as ODR-used no
1268:     // matter whether they are ODR-used by device or host functions.
1269:     //
1270:     // We do not need to do this if the variable has used attribute since it
1271:     // has already been added.
1272:     //
1273:     // Static device variables have been externalized at this point, therefore
1274:     // variables with LLVM private or internal linkage need not be added.
1275:     for (auto &&Info : DeviceVars) {
1276:       auto Kind = Info.Flags.getKind();
1277:       if (!Info.Var->isDeclaration() &&
1278:           !llvm::GlobalValue::isLocalLinkage(Info.Var->getLinkage()) &&
1279:           (Kind == DeviceVarFlags::Variable ||
1280:            Kind == DeviceVarFlags::Surface ||
```
- **EN**: This block defines callable entry points like `transformManagedVars`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `transformManagedVars`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1281-1300
```cpp
1281:            Kind == DeviceVarFlags::Texture) &&
1282:           Info.D->isUsed() && !Info.D->hasAttr<UsedAttr>()) {
1283:         CGM.addCompilerUsedGlobal(Info.Var);
1284:       }
1285:     }
1286:     return nullptr;
1287:   }
1288:   if (CGM.getLangOpts().OffloadViaLLVM ||
1289:       (CGM.getLangOpts().OffloadingNewDriver && RelocatableDeviceCode))
1290:     createOffloadingEntries();
1291:   else
1292:     return makeModuleCtorFunction();
1293: 
1294:   return nullptr;
1295: }
1296: 
1297: llvm::GlobalValue *CGNVCUDARuntime::getKernelHandle(llvm::Function *F,
1298:                                                     GlobalDecl GD) {
1299:   auto Loc = KernelHandles.find(F->getName());
1300:   if (Loc != KernelHandles.end()) {
```
- **EN**: This block defines callable entry points like `makeModuleCtorFunction`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `makeModuleCtorFunction`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1301-1320
```cpp
1301:     auto OldHandle = Loc->second;
1302:     if (KernelStubs[OldHandle] == F)
1303:       return OldHandle;
1304: 
1305:     // We've found the function name, but F itself has changed, so we need to
1306:     // update the references.
1307:     if (CGM.getLangOpts().HIP) {
1308:       // For HIP compilation the handle itself does not change, so we only need
1309:       // to update the Stub value.
1310:       KernelStubs[OldHandle] = F;
1311:       return OldHandle;
1312:     }
1313:     // For non-HIP compilation, erase the old Stub and fall-through to creating
1314:     // new entries.
1315:     KernelStubs.erase(OldHandle);
1316:   }
1317: 
1318:   if (!CGM.getLangOpts().HIP) {
1319:     KernelHandles[F->getName()] = F;
1320:     KernelStubs[F] = F;
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1321-1339
```cpp
1321:     return F;
1322:   }
1323: 
1324:   auto *Var = new llvm::GlobalVariable(
1325:       TheModule, F->getType(), /*isConstant=*/true, F->getLinkage(),
1326:       /*Initializer=*/nullptr,
1327:       CGM.getMangledName(
1328:           GD.getWithKernelReferenceKind(KernelReferenceKind::Kernel)));
1329:   Var->setAlignment(CGM.getPointerAlign().getAsAlign());
1330:   Var->setDSOLocal(F->isDSOLocal());
1331:   Var->setVisibility(F->getVisibility());
1332:   auto *FD = cast<FunctionDecl>(GD.getDecl());
1333:   auto *FT = FD->getPrimaryTemplate();
1334:   if (!FT || FT->isThisDeclarationADefinition())
1335:     CGM.maybeSetTrivialComdat(*FD, *Var);
1336:   KernelHandles[F->getName()] = Var;
1337:   KernelStubs[Var] = F;
1338:   return Var;
1339: }
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

## Key Concepts / 关键概念

- **CGM**: Central symbol in this file's implementation of LLVM IR emission. / 是该文件实现 LLVM IR 生成 时的核心符号。
- **CGF**: Central symbol in this file's implementation of LLVM IR emission. / 是该文件实现 LLVM IR 生成 时的核心符号。
- **PtrTy**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Var**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Args**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **IntTy**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Builder**: Acts as a construction helper that incrementally assembles LLVM IR emission state. / 充当构建辅助器，逐步组装 LLVM IR 生成 状态。
- **getLangOpts**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CGCUDARuntime.h`, `CGCXXABI.h`, `CodeGenFunction.h`, `CodeGenModule.h`
- **Clang libraries / Clang 库**: `clang/AST/CharUnits.h`, `clang/AST/Decl.h`, `clang/Basic/Cuda.h`, `clang/CodeGen/CodeGenABITypes.h`, `clang/CodeGen/ConstantInitBuilder.h`
- **LLVM libraries / LLVM 库**: `llvm/ADT/StringRef.h`, `llvm/Frontend/Offloading/Utility.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/ReplaceConstant.h`, `llvm/Support/Format.h`, `llvm/Support/VirtualFileSystem.h`
