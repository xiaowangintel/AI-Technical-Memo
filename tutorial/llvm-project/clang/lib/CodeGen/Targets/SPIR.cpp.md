# SPIR.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/Targets/SPIR.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements target-specific ABI rules and lowering details for SPIR.
- **Purpose (CN) / 目的（中文）**: 实现 SPIR 目标相关的 ABI 规则与降级细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: //===- SPIR.cpp -----------------------------------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: 
 9: #include "ABIInfoImpl.h"
10: #include "HLSLBufferLayoutBuilder.h"
11: #include "TargetInfo.h"
12: #include "clang/AST/DeclCXX.h"
13: #include "clang/Basic/LangOptions.h"
14: #include "llvm/IR/DerivedTypes.h"
15: 
16: #include <stdint.h>
17: #include <utility>
18: 
19: using namespace clang;
20: using namespace clang::CodeGen;
```
- **EN**: This block imports local CodeGen headers `ABIInfoImpl.h`, `HLSLBufferLayoutBuilder.h`, `TargetInfo.h`, and 1 more; Clang headers `clang/AST/DeclCXX.h`, `clang/Basic/LangOptions.h`; LLVM headers `llvm/IR/DerivedTypes.h`; other headers `utility`; opens or references namespaces `clang`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `ABIInfoImpl.h`, `HLSLBufferLayoutBuilder.h`, `TargetInfo.h`, and 1 more；Clang 头文件 `clang/AST/DeclCXX.h`, `clang/Basic/LangOptions.h`；LLVM 头文件 `llvm/IR/DerivedTypes.h`；其他头文件 `utility`；打开或引用命名空间 `clang`；包含影响本编译单元构建方式的预处理结构。

### Lines 21-40
```cpp
21: 
22: //===----------------------------------------------------------------------===//
23: // Base ABI and target codegen info implementation common between SPIR and
24: // SPIR-V.
25: //===----------------------------------------------------------------------===//
26: 
27: namespace {
28: class CommonSPIRABIInfo : public DefaultABIInfo {
29: public:
30:   CommonSPIRABIInfo(CodeGenTypes &CGT) : DefaultABIInfo(CGT) { setCCs(); }
31: 
32: private:
33:   void setCCs();
34: };
35: 
36: class SPIRVABIInfo : public CommonSPIRABIInfo {
37: public:
38:   SPIRVABIInfo(CodeGenTypes &CGT) : CommonSPIRABIInfo(CGT) {}
39:   void computeInfo(CGFunctionInfo &FI) const override;
40:   RValue EmitVAArg(CodeGenFunction &CGF, Address VAListAddr, QualType Ty,
```
- **EN**: This block introduces declarations such as `CommonSPIRABIInfo`, `SPIRVABIInfo`; defines callable entry points like `CommonSPIRABIInfo`, `setCCs`, `SPIRVABIInfo`, `computeInfo`.
- **CN**: 该代码块给出诸如 `CommonSPIRABIInfo`, `SPIRVABIInfo` 的声明；定义可调用入口，例如 `CommonSPIRABIInfo`, `setCCs`, `SPIRVABIInfo`, `computeInfo`。

### Lines 41-60
```cpp
41:                    AggValueSlot Slot) const override;
42: 
43:   llvm::FixedVectorType *
44:   getOptimalVectorMemoryType(llvm::FixedVectorType *Ty,
45:                              const LangOptions &LangOpt) const override;
46: 
47: private:
48:   ABIArgInfo classifyKernelArgumentType(QualType Ty) const;
49: };
50: 
51: class AMDGCNSPIRVABIInfo : public SPIRVABIInfo {
52:   // TODO: this should be unified / shared with AMDGPU, ideally we'd like to
53:   //       re-use AMDGPUABIInfo eventually, rather than duplicate.
54:   static constexpr unsigned MaxNumRegsForArgsRet = 16; // 16 32-bit registers
55:   mutable unsigned NumRegsLeft = 0;
56: 
57:   uint64_t numRegsForType(QualType Ty) const;
58: 
59:   bool isHomogeneousAggregateBaseType(QualType Ty) const override {
60:     return true;
```
- **EN**: This block introduces declarations such as `AMDGCNSPIRVABIInfo`; defines callable entry points like `getOptimalVectorMemoryType`, `classifyKernelArgumentType`, `numRegsForType`, `isHomogeneousAggregateBaseType`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块给出诸如 `AMDGCNSPIRVABIInfo` 的声明；定义可调用入口，例如 `getOptimalVectorMemoryType`, `classifyKernelArgumentType`, `numRegsForType`, `isHomogeneousAggregateBaseType`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 61-80
```cpp
61:   }
62:   bool isHomogeneousAggregateSmallEnough(const Type *Base,
63:                                          uint64_t Members) const override {
64:     uint32_t NumRegs = (getContext().getTypeSize(Base) + 31) / 32;
65: 
66:     // Homogeneous Aggregates may occupy at most 16 registers.
67:     return Members * NumRegs <= MaxNumRegsForArgsRet;
68:   }
69: 
70:   // Coerce HIP scalar pointer arguments from generic pointers to global ones.
71:   llvm::Type *coerceKernelArgumentType(llvm::Type *Ty, unsigned FromAS,
72:                                        unsigned ToAS) const;
73: 
74:   ABIArgInfo classifyReturnType(QualType RetTy) const;
75:   ABIArgInfo classifyKernelArgumentType(QualType Ty) const;
76:   ABIArgInfo classifyArgumentType(QualType Ty) const;
77: 
78: public:
79:   AMDGCNSPIRVABIInfo(CodeGenTypes &CGT) : SPIRVABIInfo(CGT) {}
80:   void computeInfo(CGFunctionInfo &FI) const override;
```
- **EN**: This block defines callable entry points like `isHomogeneousAggregateSmallEnough`, `classifyReturnType`, `classifyKernelArgumentType`, `classifyArgumentType`, `AMDGCNSPIRVABIInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `isHomogeneousAggregateSmallEnough`, `classifyReturnType`, `classifyKernelArgumentType`, `classifyArgumentType`, `AMDGCNSPIRVABIInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 81-100
```cpp
 81: 
 82:   llvm::FixedVectorType *
 83:   getOptimalVectorMemoryType(llvm::FixedVectorType *Ty,
 84:                              const LangOptions &LangOpt) const override;
 85: };
 86: } // end anonymous namespace
 87: namespace {
 88: class CommonSPIRTargetCodeGenInfo : public TargetCodeGenInfo {
 89: public:
 90:   CommonSPIRTargetCodeGenInfo(CodeGen::CodeGenTypes &CGT)
 91:       : TargetCodeGenInfo(std::make_unique<CommonSPIRABIInfo>(CGT)) {}
 92:   CommonSPIRTargetCodeGenInfo(std::unique_ptr<ABIInfo> ABIInfo)
 93:       : TargetCodeGenInfo(std::move(ABIInfo)) {}
 94: 
 95:   unsigned getDeviceKernelCallingConv() const override;
 96:   llvm::Type *getOpenCLType(CodeGenModule &CGM, const Type *T) const override;
 97:   llvm::Type *getHLSLType(CodeGenModule &CGM, const Type *Ty,
 98:                           const CGHLSLOffsetInfo &OffsetInfo) const override;
 99: 
100:   llvm::Type *getHLSLPadding(CodeGenModule &CGM,
```
- **EN**: This block opens or references namespaces `namespace`; introduces declarations such as `CommonSPIRTargetCodeGenInfo`; defines callable entry points like `getOptimalVectorMemoryType`, `CommonSPIRTargetCodeGenInfo`, `getDeviceKernelCallingConv`.
- **CN**: 该代码块打开或引用命名空间 `namespace`；给出诸如 `CommonSPIRTargetCodeGenInfo` 的声明；定义可调用入口，例如 `getOptimalVectorMemoryType`, `CommonSPIRTargetCodeGenInfo`, `getDeviceKernelCallingConv`。

### Lines 101-120
```cpp
101:                              CharUnits NumBytes) const override {
102:     unsigned Size = NumBytes.getQuantity();
103:     return llvm::TargetExtType::get(CGM.getLLVMContext(), "spirv.Padding", {},
104:                                     {Size});
105:   }
106: 
107:   bool isHLSLPadding(llvm::Type *Ty) const override {
108:     if (auto *TET = dyn_cast<llvm::TargetExtType>(Ty))
109:       return TET->getName() == "spirv.Padding";
110:     return false;
111:   }
112: 
113:   llvm::Type *getSPIRVImageTypeFromHLSLResource(
114:       const HLSLAttributedResourceType::Attributes &attributes,
115:       QualType SampledType, CodeGenModule &CGM) const;
116:   void
117:   setOCLKernelStubCallingConvention(const FunctionType *&FT) const override;
118:   llvm::Constant *getNullPointer(const CodeGen::CodeGenModule &CGM,
119:                                  llvm::PointerType *T,
120:                                  QualType QT) const override;
```
- **EN**: This block defines callable entry points like `isHLSLPadding`, `setOCLKernelStubCallingConvention`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isHLSLPadding`, `setOCLKernelStubCallingConvention`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 121-140
```cpp
121: };
122: class SPIRVTargetCodeGenInfo : public CommonSPIRTargetCodeGenInfo {
123: public:
124:   SPIRVTargetCodeGenInfo(CodeGen::CodeGenTypes &CGT)
125:       : CommonSPIRTargetCodeGenInfo(
126:             (CGT.getTarget().getTriple().getVendor() == llvm::Triple::AMD)
127:                 ? std::make_unique<AMDGCNSPIRVABIInfo>(CGT)
128:                 : std::make_unique<SPIRVABIInfo>(CGT)) {}
129:   void setCUDAKernelCallingConvention(const FunctionType *&FT) const override;
130:   LangAS getGlobalVarAddressSpace(CodeGenModule &CGM,
131:                                   const VarDecl *D) const override;
132:   void setTargetAttributes(const Decl *D, llvm::GlobalValue *GV,
133:                            CodeGen::CodeGenModule &M) const override;
134:   StringRef getLLVMSyncScopeStr(const LangOptions &LangOpts, SyncScope Scope,
135:                                 llvm::AtomicOrdering Ordering) const override;
136:   void setTargetAtomicMetadata(CodeGenFunction &CGF,
137:                                llvm::Instruction &AtomicInst,
138:                                const AtomicExpr *Expr = nullptr) const override;
139:   bool supportsLibCall() const override {
140:     return getABIInfo().getTarget().getTriple().getVendor() !=
```
- **EN**: This block introduces declarations such as `SPIRVTargetCodeGenInfo`; defines callable entry points like `SPIRVTargetCodeGenInfo`, `setCUDAKernelCallingConvention`, `getGlobalVarAddressSpace`, `setTargetAttributes`, `getLLVMSyncScopeStr`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块给出诸如 `SPIRVTargetCodeGenInfo` 的声明；定义可调用入口，例如 `SPIRVTargetCodeGenInfo`, `setCUDAKernelCallingConvention`, `getGlobalVarAddressSpace`, `setTargetAttributes`, `getLLVMSyncScopeStr`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 141-160
```cpp
141:            llvm::Triple::AMD;
142:   }
143: 
144:   LangAS getSRetAddrSpace(const CXXRecordDecl *RD) const override;
145: };
146: } // End anonymous namespace.
147: 
148: void CommonSPIRABIInfo::setCCs() {
149:   assert(getRuntimeCC() == llvm::CallingConv::C);
150:   RuntimeCC = llvm::CallingConv::SPIR_FUNC;
151: }
152: 
153: ABIArgInfo SPIRVABIInfo::classifyKernelArgumentType(QualType Ty) const {
154:   // Coerce pointer arguments with default address space to CrossWorkGroup
155:   // pointers as default address space kernel
156:   // arguments are not allowed. We use the opencl_global language address
157:   // space which always maps to CrossWorkGroup.
158:   llvm::Type *LTy = CGT.ConvertType(Ty);
159:   auto DefaultAS = getContext().getTargetAddressSpace(LangAS::Default);
160:   auto GlobalAS = getContext().getTargetAddressSpace(LangAS::opencl_global);
```
- **EN**: This block defines callable entry points like `getSRetAddrSpace`, `setCCs`, `classifyKernelArgumentType`; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getSRetAddrSpace`, `setCCs`, `classifyKernelArgumentType`；使用断言或不可达标记保护关键不变量。

### Lines 161-180
```cpp
161:   auto *PtrTy = llvm::dyn_cast<llvm::PointerType>(LTy);
162:   if (PtrTy && PtrTy->getAddressSpace() == DefaultAS) {
163:     LTy = llvm::PointerType::get(PtrTy->getContext(), GlobalAS);
164:     return ABIArgInfo::getDirect(LTy, 0, nullptr, false);
165:   }
166: 
167:   if (getContext().getLangOpts().isTargetDevice() &&
168:       isAggregateTypeForABI(Ty)) {
169:     // Force copying aggregate type in kernel arguments by value when
170:     // compiling CUDA targeting SPIR-V. This is required for the object
171:     // copied to be valid on the device.
172:     // This behavior follows the CUDA spec
173:     // https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#global-function-argument-processing,
174:     // and matches the NVPTX implementation. TODO: hardcoding to 0 should be
175:     // revisited if HIPSPV / byval starts making use of the AS of an indirect
176:     // arg.
177:     return getNaturalAlignIndirect(Ty, /*AddrSpace=*/0, /*byval=*/true);
178:   }
179:   return classifyArgumentType(Ty);
180: }
```
- **EN**: This block defines callable entry points like `getDirect`, `getNaturalAlignIndirect`, `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `getNaturalAlignIndirect`, `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 181-200
```cpp
181: 
182: void SPIRVABIInfo::computeInfo(CGFunctionInfo &FI) const {
183:   // The logic is same as in DefaultABIInfo with an exception on the kernel
184:   // arguments handling.
185:   llvm::CallingConv::ID CC = FI.getCallingConvention();
186: 
187:   for (auto &&[ArgumentsCount, I] : llvm::enumerate(FI.arguments()))
188:     I.info = ArgumentsCount < FI.getNumRequiredArgs()
189:                  ? classifyArgumentType(I.type)
190:                  : ABIArgInfo::getDirect();
191: 
192:   if (!getCXXABI().classifyReturnType(FI))
193:     FI.getReturnInfo() = classifyReturnType(FI.getReturnType());
194: 
195:   for (auto &I : FI.arguments()) {
196:     if (CC == llvm::CallingConv::SPIR_KERNEL) {
197:       I.info = classifyKernelArgumentType(I.type);
198:     } else {
199:       I.info = classifyArgumentType(I.type);
200:     }
```
- **EN**: This block defines callable entry points like `computeInfo`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `computeInfo`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 201-220
```cpp
201:   }
202: }
203: 
204: RValue SPIRVABIInfo::EmitVAArg(CodeGenFunction &CGF, Address VAListAddr,
205:                                QualType Ty, AggValueSlot Slot) const {
206:   return emitVoidPtrVAArg(CGF, VAListAddr, Ty, /*IsIndirect=*/false,
207:                           getContext().getTypeInfoInChars(Ty),
208:                           CharUnits::fromQuantity(1),
209:                           /*AllowHigherAlign=*/true, Slot);
210: }
211: 
212: uint64_t AMDGCNSPIRVABIInfo::numRegsForType(QualType Ty) const {
213:   // This duplicates the AMDGPUABI computation.
214:   uint64_t NumRegs = 0;
215: 
216:   if (const VectorType *VT = Ty->getAs<VectorType>()) {
217:     // Compute from the number of elements. The reported size is based on the
218:     // in-memory size, which includes the padding 4th element for 3-vectors.
219:     QualType EltTy = VT->getElementType();
220:     uint64_t EltSize = getContext().getTypeSize(EltTy);
```
- **EN**: This block defines callable entry points like `EmitVAArg`, `numRegsForType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `EmitVAArg`, `numRegsForType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 221-240
```cpp
221: 
222:     // 16-bit element vectors should be passed as packed.
223:     if (EltSize == 16)
224:       return (VT->getNumElements() + 1) / 2;
225: 
226:     uint64_t EltNumRegs = (EltSize + 31) / 32;
227:     return EltNumRegs * VT->getNumElements();
228:   }
229: 
230:   if (const auto *RD = Ty->getAsRecordDecl()) {
231:     assert(!RD->hasFlexibleArrayMember());
232: 
233:     for (const FieldDecl *Field : RD->fields()) {
234:       QualType FieldTy = Field->getType();
235:       NumRegs += numRegsForType(FieldTy);
236:     }
237: 
238:     return NumRegs;
239:   }
240: 
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 241-260
```cpp
241:   return (getContext().getTypeSize(Ty) + 31) / 32;
242: }
243: 
244: llvm::Type *AMDGCNSPIRVABIInfo::coerceKernelArgumentType(llvm::Type *Ty,
245:                                                          unsigned FromAS,
246:                                                          unsigned ToAS) const {
247:   // Single value types.
248:   auto *PtrTy = llvm::dyn_cast<llvm::PointerType>(Ty);
249:   if (PtrTy && PtrTy->getAddressSpace() == FromAS)
250:     return llvm::PointerType::get(Ty->getContext(), ToAS);
251:   return Ty;
252: }
253: 
254: ABIArgInfo AMDGCNSPIRVABIInfo::classifyReturnType(QualType RetTy) const {
255:   if (!isAggregateTypeForABI(RetTy) || getRecordArgABI(RetTy, getCXXABI()))
256:     return DefaultABIInfo::classifyReturnType(RetTy);
257: 
258:   // Ignore empty structs/unions.
259:   if (isEmptyRecord(getContext(), RetTy, true))
260:     return ABIArgInfo::getIgnore();
```
- **EN**: This block defines callable entry points like `classifyReturnType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyReturnType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 261-280
```cpp
261: 
262:   // Lower single-element structs to just return a regular value.
263:   if (const Type *SeltTy = isSingleElementStruct(RetTy, getContext()))
264:     return ABIArgInfo::getDirect(CGT.ConvertType(QualType(SeltTy, 0)));
265: 
266:   if (const auto *RD = RetTy->getAsRecordDecl();
267:       RD && RD->hasFlexibleArrayMember())
268:     return DefaultABIInfo::classifyReturnType(RetTy);
269: 
270:   // Pack aggregates <= 4 bytes into single VGPR or pair.
271:   uint64_t Size = getContext().getTypeSize(RetTy);
272:   if (Size <= 16)
273:     return ABIArgInfo::getDirect(llvm::Type::getInt16Ty(getVMContext()));
274: 
275:   if (Size <= 32)
276:     return ABIArgInfo::getDirect(llvm::Type::getInt32Ty(getVMContext()));
277: 
278:   // TODO: This carried over from AMDGPU oddity, we retain it to
279:   //       ensure consistency, but it might be reasonable to return Int64.
280:   if (Size <= 64) {
```
- **EN**: This block defines callable entry points like `classifyReturnType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyReturnType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 281-300
```cpp
281:     llvm::Type *I32Ty = llvm::Type::getInt32Ty(getVMContext());
282:     return ABIArgInfo::getDirect(llvm::ArrayType::get(I32Ty, 2));
283:   }
284: 
285:   if (numRegsForType(RetTy) <= MaxNumRegsForArgsRet)
286:     return ABIArgInfo::getDirect();
287:   return DefaultABIInfo::classifyReturnType(RetTy);
288: }
289: 
290: /// For kernels all parameters are really passed in a special buffer. It doesn't
291: /// make sense to pass anything byval, so everything must be direct.
292: ABIArgInfo AMDGCNSPIRVABIInfo::classifyKernelArgumentType(QualType Ty) const {
293:   Ty = useFirstFieldIfTransparentUnion(Ty);
294: 
295:   // TODO: Can we omit empty structs?
296: 
297:   if (const Type *SeltTy = isSingleElementStruct(Ty, getContext()))
298:     Ty = QualType(SeltTy, 0);
299: 
300:   llvm::Type *OrigLTy = CGT.ConvertType(Ty);
```
- **EN**: This block defines callable entry points like `getDirect`, `classifyReturnType`, `classifyKernelArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `classifyReturnType`, `classifyKernelArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 301-320
```cpp
301:   llvm::Type *LTy = OrigLTy;
302:   if (getContext().getLangOpts().isTargetDevice()) {
303:     LTy = coerceKernelArgumentType(
304:         OrigLTy, /*FromAS=*/getContext().getTargetAddressSpace(LangAS::Default),
305:         /*ToAS=*/getContext().getTargetAddressSpace(LangAS::opencl_global));
306:   }
307: 
308:   // FIXME: This doesn't apply the optimization of coercing pointers in structs
309:   // to global address space when using byref. This would require implementing a
310:   // new kind of coercion of the in-memory type when for indirect arguments.
311:   if (LTy == OrigLTy && isAggregateTypeForABI(Ty)) {
312:     return ABIArgInfo::getIndirectAliased(
313:         getContext().getTypeAlignInChars(Ty),
314:         getContext().getTargetAddressSpace(LangAS::opencl_constant),
315:         false /*Realign*/, nullptr /*Padding*/);
316:   }
317: 
318:   // TODO: inhibiting flattening is an AMDGPU workaround for Clover, which might
319:   //       be vestigial and should be revisited.
320:   return ABIArgInfo::getDirect(LTy, 0, nullptr, false);
```
- **EN**: This block defines callable entry points like `getIndirectAliased`, `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getIndirectAliased`, `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 321-340
```cpp
321: }
322: 
323: ABIArgInfo AMDGCNSPIRVABIInfo::classifyArgumentType(QualType Ty) const {
324:   assert(NumRegsLeft <= MaxNumRegsForArgsRet && "register estimate underflow");
325: 
326:   Ty = useFirstFieldIfTransparentUnion(Ty);
327: 
328:   // TODO: support for variadics.
329: 
330:   if (!isAggregateTypeForABI(Ty)) {
331:     ABIArgInfo ArgInfo = DefaultABIInfo::classifyArgumentType(Ty);
332:     if (!ArgInfo.isIndirect()) {
333:       uint64_t NumRegs = numRegsForType(Ty);
334:       NumRegsLeft -= std::min(NumRegs, uint64_t{NumRegsLeft});
335:     }
336: 
337:     return ArgInfo;
338:   }
339: 
340:   // Records with non-trivial destructors/copy-constructors should not be
```
- **EN**: This block defines callable entry points like `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 341-360
```cpp
341:   // passed by value.
342:   if (auto RAA = getRecordArgABI(Ty, getCXXABI()))
343:     return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
344:                                    RAA == CGCXXABI::RAA_DirectInMemory);
345: 
346:   // Ignore empty structs/unions.
347:   if (isEmptyRecord(getContext(), Ty, true))
348:     return ABIArgInfo::getIgnore();
349: 
350:   // Lower single-element structs to just pass a regular value. TODO: We
351:   // could do reasonable-size multiple-element structs too, using getExpand(),
352:   // though watch out for things like bitfields.
353:   if (const Type *SeltTy = isSingleElementStruct(Ty, getContext()))
354:     return ABIArgInfo::getDirect(CGT.ConvertType(QualType(SeltTy, 0)));
355: 
356:   if (const auto *RD = Ty->getAsRecordDecl();
357:       RD && RD->hasFlexibleArrayMember())
358:     return DefaultABIInfo::classifyArgumentType(Ty);
359: 
360:   uint64_t Size = getContext().getTypeSize(Ty);
```
- **EN**: This block spells out callable entry points like `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出可调用入口的声明，例如 `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 361-380
```cpp
361:   if (Size <= 64) {
362:     // Pack aggregates <= 8 bytes into single VGPR or pair.
363:     unsigned NumRegs = (Size + 31) / 32;
364:     NumRegsLeft -= std::min(NumRegsLeft, NumRegs);
365: 
366:     if (Size <= 16)
367:       return ABIArgInfo::getDirect(llvm::Type::getInt16Ty(getVMContext()));
368: 
369:     if (Size <= 32)
370:       return ABIArgInfo::getDirect(llvm::Type::getInt32Ty(getVMContext()));
371: 
372:     // TODO: This is an AMDGPU oddity, and might be vestigial, we retain it to
373:     //       ensure consistency, but it should be revisited.
374:     llvm::Type *I32Ty = llvm::Type::getInt32Ty(getVMContext());
375:     return ABIArgInfo::getDirect(llvm::ArrayType::get(I32Ty, 2));
376:   }
377: 
378:   if (NumRegsLeft > 0) {
379:     uint64_t NumRegs = numRegsForType(Ty);
380:     if (NumRegsLeft >= NumRegs) {
```
- **EN**: This block defines callable entry points like `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 381-400
```cpp
381:       NumRegsLeft -= NumRegs;
382:       return ABIArgInfo::getDirect();
383:     }
384:   }
385: 
386:   // Use pass-by-reference in stead of pass-by-value for struct arguments in
387:   // function ABI.
388:   return ABIArgInfo::getIndirectAliased(
389:       getContext().getTypeAlignInChars(Ty),
390:       getContext().getTargetAddressSpace(LangAS::opencl_private));
391: }
392: 
393: void AMDGCNSPIRVABIInfo::computeInfo(CGFunctionInfo &FI) const {
394:   llvm::CallingConv::ID CC = FI.getCallingConvention();
395: 
396:   if (!getCXXABI().classifyReturnType(FI))
397:     FI.getReturnInfo() = classifyReturnType(FI.getReturnType());
398: 
399:   NumRegsLeft = MaxNumRegsForArgsRet;
400:   for (auto &I : FI.arguments()) {
```
- **EN**: This block defines callable entry points like `getDirect`, `getIndirectAliased`, `computeInfo`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `getIndirectAliased`, `computeInfo`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 401-420
```cpp
401:     if (CC == llvm::CallingConv::SPIR_KERNEL)
402:       I.info = classifyKernelArgumentType(I.type);
403:     else
404:       I.info = classifyArgumentType(I.type);
405:   }
406: }
407: 
408: llvm::FixedVectorType *
409: SPIRVABIInfo::getOptimalVectorMemoryType(llvm::FixedVectorType *Ty,
410:                                          const LangOptions &LangOpt) const {
411:   // For Logical SPIR-V, we don't know the underlying hardware or layout.
412:   // This means we don't know which vector size is better, and also cannot
413:   // assume a smaller vector size is stored in a larger vector size.
414:   if (getTarget().getTriple().isSPIRVLogical())
415:     return Ty;
416:   return DefaultABIInfo::getOptimalVectorMemoryType(Ty, LangOpt);
417: }
418: 
419: llvm::FixedVectorType *AMDGCNSPIRVABIInfo::getOptimalVectorMemoryType(
420:     llvm::FixedVectorType *Ty, const LangOptions &LangOpt) const {
```
- **EN**: This block defines callable entry points like `getOptimalVectorMemoryType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getOptimalVectorMemoryType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 421-440
```cpp
421:   // AMDGPU has legal instructions for 96-bit so 3x32 can be supported.
422:   if (Ty->getNumElements() == 3 && getDataLayout().getTypeSizeInBits(Ty) == 96)
423:     return Ty;
424:   return DefaultABIInfo::getOptimalVectorMemoryType(Ty, LangOpt);
425: }
426: 
427: namespace clang {
428: namespace CodeGen {
429: void computeSPIRKernelABIInfo(CodeGenModule &CGM, CGFunctionInfo &FI) {
430:   if (CGM.getTarget().getTriple().isSPIRV()) {
431:     if (CGM.getTarget().getTriple().getVendor() == llvm::Triple::AMD)
432:       AMDGCNSPIRVABIInfo(CGM.getTypes()).computeInfo(FI);
433:     else
434:       SPIRVABIInfo(CGM.getTypes()).computeInfo(FI);
435:   } else {
436:     CommonSPIRABIInfo(CGM.getTypes()).computeInfo(FI);
437:   }
438: }
439: }
440: }
```
- **EN**: This block opens or references namespaces `clang`, `CodeGen`; defines callable entry points like `getOptimalVectorMemoryType`, `computeSPIRKernelABIInfo`, `SPIRVABIInfo`, `CommonSPIRABIInfo`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块打开或引用命名空间 `clang`, `CodeGen`；定义可调用入口，例如 `getOptimalVectorMemoryType`, `computeSPIRKernelABIInfo`, `SPIRVABIInfo`, `CommonSPIRABIInfo`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 441-460
```cpp
441: 
442: unsigned CommonSPIRTargetCodeGenInfo::getDeviceKernelCallingConv() const {
443:   return llvm::CallingConv::SPIR_KERNEL;
444: }
445: 
446: LangAS SPIRVTargetCodeGenInfo::getSRetAddrSpace(const CXXRecordDecl *RD) const {
447:   // Types with no viable copy/move must be constructed in-place, use the
448:   // default AS so the sret pointer matches the "this" convention.
449:   if (RD && !RD->canPassInRegisters())
450:     return LangAS::Default;
451:   return getLangASFromTargetAS(
452:       getABIInfo().getDataLayout().getAllocaAddrSpace());
453: }
454: 
455: void SPIRVTargetCodeGenInfo::setCUDAKernelCallingConvention(
456:     const FunctionType *&FT) const {
457:   // Convert HIP kernels to SPIR-V kernels.
458:   if (getABIInfo().getContext().getLangOpts().HIP) {
459:     FT = getABIInfo().getContext().adjustFunctionType(
460:         FT, FT->getExtInfo().withCallingConv(CC_DeviceKernel));
```
- **EN**: This block defines callable entry points like `getDeviceKernelCallingConv`, `getSRetAddrSpace`, `getLangASFromTargetAS`, `setCUDAKernelCallingConvention`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDeviceKernelCallingConv`, `getSRetAddrSpace`, `getLangASFromTargetAS`, `setCUDAKernelCallingConvention`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 461-480
```cpp
461:     return;
462:   }
463: }
464: 
465: void CommonSPIRTargetCodeGenInfo::setOCLKernelStubCallingConvention(
466:     const FunctionType *&FT) const {
467:   FT = getABIInfo().getContext().adjustFunctionType(
468:       FT, FT->getExtInfo().withCallingConv(CC_SpirFunction));
469: }
470: 
471: // LLVM currently assumes a null pointer has the bit pattern 0, but some GPU
472: // targets use a non-zero encoding for null in certain address spaces.
473: // Because SPIR(-V) is a generic target and the bit pattern of null in
474: // non-generic AS is unspecified, materialize null in non-generic AS via an
475: // addrspacecast from null in generic AS. This allows later lowering to
476: // substitute the target's real sentinel value.
477: llvm::Constant *
478: CommonSPIRTargetCodeGenInfo::getNullPointer(const CodeGen::CodeGenModule &CGM,
479:                                             llvm::PointerType *PT,
480:                                             QualType QT) const {
```
- **EN**: This block defines callable entry points like `setOCLKernelStubCallingConvention`, `getNullPointer`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `setOCLKernelStubCallingConvention`, `getNullPointer`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 481-500
```cpp
481:   LangAS AS = QT->getUnqualifiedDesugaredType()->isNullPtrType()
482:                   ? LangAS::Default
483:                   : QT->getPointeeType().getAddressSpace();
484:   unsigned ASAsInt = static_cast<unsigned>(AS);
485:   unsigned FirstTargetASAsInt =
486:       static_cast<unsigned>(LangAS::FirstTargetAddressSpace);
487:   unsigned CodeSectionINTELAS = FirstTargetASAsInt + 9;
488:   // As per SPV_INTEL_function_pointers, it is illegal to addrspacecast
489:   // function pointers to/from the generic AS.
490:   bool IsFunctionPtrAS =
491:       CGM.getTriple().isSPIRV() && ASAsInt == CodeSectionINTELAS;
492:   if (AS == LangAS::Default || AS == LangAS::opencl_generic ||
493:       AS == LangAS::opencl_constant || IsFunctionPtrAS)
494:     return llvm::ConstantPointerNull::get(PT);
495: 
496:   auto &Ctx = CGM.getContext();
497:   auto NPT = llvm::PointerType::get(
498:       PT->getContext(), Ctx.getTargetAddressSpace(LangAS::opencl_generic));
499:   return llvm::ConstantExpr::getAddrSpaceCast(
500:       llvm::ConstantPointerNull::get(NPT), PT);
```
- **EN**: This block spells out callable entry points like `getAddrSpaceCast`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出可调用入口的声明，例如 `getAddrSpaceCast`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 501-520
```cpp
501: }
502: 
503: LangAS
504: SPIRVTargetCodeGenInfo::getGlobalVarAddressSpace(CodeGenModule &CGM,
505:                                                  const VarDecl *D) const {
506:   assert(!CGM.getLangOpts().OpenCL &&
507:          !(CGM.getLangOpts().CUDA && CGM.getLangOpts().CUDAIsDevice) &&
508:          "Address space agnostic languages only");
509:   // If we're here it means that we're using the SPIRDefIsGen ASMap, hence for
510:   // the global AS we can rely on either cuda_device or sycl_global to be
511:   // correct; however, since this is not a CUDA Device context, we use
512:   // sycl_global to prevent confusion with the assertion.
513:   LangAS DefaultGlobalAS = getLangASFromTargetAS(
514:       CGM.getContext().getTargetAddressSpace(LangAS::sycl_global));
515:   if (!D)
516:     return DefaultGlobalAS;
517: 
518:   LangAS AddrSpace = D->getType().getAddressSpace();
519:   if (AddrSpace != LangAS::Default)
520:     return AddrSpace;
```
- **EN**: This block defines callable entry points like `getGlobalVarAddressSpace`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getGlobalVarAddressSpace`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 521-540
```cpp
521: 
522:   return DefaultGlobalAS;
523: }
524: 
525: void SPIRVTargetCodeGenInfo::setTargetAttributes(
526:     const Decl *D, llvm::GlobalValue *GV, CodeGen::CodeGenModule &M) const {
527:   if (GV->isDeclaration())
528:     return;
529: 
530:   const FunctionDecl *FD = dyn_cast_or_null<FunctionDecl>(D);
531:   if (!FD)
532:     return;
533: 
534:   llvm::Function *F = dyn_cast<llvm::Function>(GV);
535:   assert(F && "Expected GlobalValue to be a Function");
536: 
537:   if (!M.getLangOpts().HIP ||
538:       M.getTarget().getTriple().getVendor() != llvm::Triple::AMD)
539:     return;
540: 
```
- **EN**: This block defines callable entry points like `setTargetAttributes`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `setTargetAttributes`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 541-560
```cpp
541:   if (!FD->hasAttr<CUDAGlobalAttr>())
542:     return;
543: 
544:   unsigned N = M.getLangOpts().GPUMaxThreadsPerBlock;
545:   if (auto FlatWGS = FD->getAttr<AMDGPUFlatWorkGroupSizeAttr>())
546:     N = FlatWGS->getMax()->EvaluateKnownConstInt(M.getContext()).getExtValue();
547: 
548:   // We encode the maximum flat WG size in the first component of the 3D
549:   // max_work_group_size attribute, which will get reverse translated into the
550:   // original AMDGPU attribute when targeting AMDGPU.
551:   auto Int32Ty = llvm::IntegerType::getInt32Ty(M.getLLVMContext());
552:   llvm::Metadata *AttrMDArgs[] = {
553:       llvm::ConstantAsMetadata::get(llvm::ConstantInt::get(Int32Ty, N)),
554:       llvm::ConstantAsMetadata::get(llvm::ConstantInt::get(Int32Ty, 1)),
555:       llvm::ConstantAsMetadata::get(llvm::ConstantInt::get(Int32Ty, 1))};
556: 
557:   F->setMetadata("max_work_group_size",
558:                  llvm::MDNode::get(M.getLLVMContext(), AttrMDArgs));
559: }
560: 
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 561-580
```cpp
561: StringRef SPIRVTargetCodeGenInfo::getLLVMSyncScopeStr(
562:     const LangOptions &, SyncScope Scope, llvm::AtomicOrdering) const {
563:   switch (Scope) {
564:   case SyncScope::HIPSingleThread:
565:   case SyncScope::SingleScope:
566:     return "singlethread";
567:   case SyncScope::HIPWavefront:
568:   case SyncScope::OpenCLSubGroup:
569:   case SyncScope::WavefrontScope:
570:     return "subgroup";
571:   case SyncScope::HIPCluster:
572:   case SyncScope::ClusterScope:
573:   case SyncScope::HIPWorkgroup:
574:   case SyncScope::OpenCLWorkGroup:
575:   case SyncScope::WorkgroupScope:
576:     return "workgroup";
577:   case SyncScope::HIPAgent:
578:   case SyncScope::OpenCLDevice:
579:   case SyncScope::DeviceScope:
580:     return "device";
```
- **EN**: This block defines callable entry points like `getLLVMSyncScopeStr`; uses control flow (switch, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getLLVMSyncScopeStr`；通过控制流（switch, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 581-600
```cpp
581:   case SyncScope::SystemScope:
582:   case SyncScope::HIPSystem:
583:   case SyncScope::OpenCLAllSVMDevices:
584:     return "";
585:   }
586:   return "";
587: }
588: 
589: void SPIRVTargetCodeGenInfo::setTargetAtomicMetadata(
590:     CodeGenFunction &CGF, llvm::Instruction &AtomicInst,
591:     const AtomicExpr *AE) const {
592:   if (CGF.CGM.getTriple().getVendor() != llvm::Triple::VendorType::AMD)
593:     return;
594: 
595:   auto *RMW = dyn_cast<llvm::AtomicRMWInst>(&AtomicInst);
596:   if (!RMW)
597:     return;
598: 
599:   AtomicOptions AO = CGF.CGM.getAtomicOpts();
600:   llvm::MDNode *Empty = llvm::MDNode::get(CGF.getLLVMContext(), {});
```
- **EN**: This block defines callable entry points like `setTargetAtomicMetadata`; uses control flow (if, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `setTargetAtomicMetadata`；通过控制流（if, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 601-620
```cpp
601:   if (!AO.getOption(clang::AtomicOptionKind::FineGrainedMemory))
602:     RMW->setMetadata("amdgpu.no.fine.grained.memory", Empty);
603:   if (!AO.getOption(clang::AtomicOptionKind::RemoteMemory))
604:     RMW->setMetadata("amdgpu.no.remote.memory", Empty);
605:   if (AO.getOption(clang::AtomicOptionKind::IgnoreDenormalMode) &&
606:       RMW->getOperation() == llvm::AtomicRMWInst::FAdd &&
607:       RMW->getType()->isFloatTy())
608:     RMW->setMetadata("amdgpu.ignore.denormal.mode", Empty);
609: }
610: 
611: /// Construct a SPIR-V target extension type for the given OpenCL image type.
612: static llvm::Type *getSPIRVImageType(llvm::LLVMContext &Ctx, StringRef BaseType,
613:                                      StringRef OpenCLName,
614:                                      unsigned AccessQualifier) {
615:   // These parameters compare to the operands of OpTypeImage (see
616:   // https://registry.khronos.org/SPIR-V/specs/unified1/SPIRV.html#OpTypeImage
617:   // for more details). The first 6 integer parameters all default to 0, and
618:   // will be changed to 1 only for the image type(s) that set the parameter to
619:   // one. The 7th integer parameter is the access qualifier, which is tacked on
620:   // at the end.
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 621-640
```cpp
621:   SmallVector<unsigned, 7> IntParams = {0, 0, 0, 0, 0, 0};
622: 
623:   // Choose the dimension of the image--this corresponds to the Dim enum in
624:   // SPIR-V (first integer parameter of OpTypeImage).
625:   if (OpenCLName.starts_with("image2d"))
626:     IntParams[0] = 1;
627:   else if (OpenCLName.starts_with("image3d"))
628:     IntParams[0] = 2;
629:   else if (OpenCLName == "image1d_buffer")
630:     IntParams[0] = 5; // Buffer
631:   else
632:     assert(OpenCLName.starts_with("image1d") && "Unknown image type");
633: 
634:   // Set the other integer parameters of OpTypeImage if necessary. Note that the
635:   // OpenCL image types don't provide any information for the Sampled or
636:   // Image Format parameters.
637:   if (OpenCLName.contains("_depth"))
638:     IntParams[1] = 1;
639:   if (OpenCLName.contains("_array"))
640:     IntParams[2] = 1;
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 641-660
```cpp
641:   if (OpenCLName.contains("_msaa"))
642:     IntParams[3] = 1;
643: 
644:   // Access qualifier
645:   IntParams.push_back(AccessQualifier);
646: 
647:   return llvm::TargetExtType::get(Ctx, BaseType, {llvm::Type::getVoidTy(Ctx)},
648:                                   IntParams);
649: }
650: 
651: llvm::Type *CommonSPIRTargetCodeGenInfo::getOpenCLType(CodeGenModule &CGM,
652:                                                        const Type *Ty) const {
653:   llvm::LLVMContext &Ctx = CGM.getLLVMContext();
654:   if (auto *PipeTy = dyn_cast<PipeType>(Ty))
655:     return llvm::TargetExtType::get(Ctx, "spirv.Pipe", {},
656:                                     {!PipeTy->isReadOnly()});
657:   if (auto *BuiltinTy = dyn_cast<BuiltinType>(Ty)) {
658:     enum AccessQualifier : unsigned { AQ_ro = 0, AQ_wo = 1, AQ_rw = 2 };
659:     switch (BuiltinTy->getKind()) {
660: #define IMAGE_TYPE(ImgType, Id, SingletonId, Access, Suffix)                   \
```
- **EN**: This block introduces declarations such as `AccessQualifier`; uses control flow (if, switch) to specialize target-specific ABI and code generation; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块给出诸如 `AccessQualifier` 的声明；通过控制流（if, switch）细化 目标相关的 ABI 与代码生成 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 661-680
```cpp
661:     case BuiltinType::Id:                                                      \
662:       return getSPIRVImageType(Ctx, "spirv.Image", #ImgType, AQ_##Suffix);
663: #include "clang/Basic/OpenCLImageTypes.def"
664:     case BuiltinType::OCLSampler:
665:       return llvm::TargetExtType::get(Ctx, "spirv.Sampler");
666:     case BuiltinType::OCLEvent:
667:       return llvm::TargetExtType::get(Ctx, "spirv.Event");
668:     case BuiltinType::OCLClkEvent:
669:       return llvm::TargetExtType::get(Ctx, "spirv.DeviceEvent");
670:     case BuiltinType::OCLQueue:
671:       return llvm::TargetExtType::get(Ctx, "spirv.Queue");
672:     case BuiltinType::OCLReserveID:
673:       return llvm::TargetExtType::get(Ctx, "spirv.ReserveId");
674: #define INTEL_SUBGROUP_AVC_TYPE(Name, Id)                                      \
675:     case BuiltinType::OCLIntelSubgroupAVC##Id:                                 \
676:       return llvm::TargetExtType::get(Ctx, "spirv.Avc" #Id "INTEL");
677: #include "clang/Basic/OpenCLExtensionTypes.def"
678:     default:
679:       return nullptr;
680:     }
```
- **EN**: This block imports Clang headers `clang/Basic/OpenCLImageTypes.def`, `clang/Basic/OpenCLExtensionTypes.def`; spells out callable entry points like `getSPIRVImageType`, `get`; uses control flow (case) to specialize target-specific ABI and code generation; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/Basic/OpenCLImageTypes.def`, `clang/Basic/OpenCLExtensionTypes.def`；给出可调用入口的声明，例如 `getSPIRVImageType`, `get`；通过控制流（case）细化 目标相关的 ABI 与代码生成 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 681-700
```cpp
681:   }
682: 
683:   return nullptr;
684: }
685: 
686: // Gets a spirv.IntegralConstant or spirv.Literal. If IntegralType is present,
687: // returns an IntegralConstant, otherwise returns a Literal.
688: static llvm::Type *getInlineSpirvConstant(CodeGenModule &CGM,
689:                                           llvm::Type *IntegralType,
690:                                           llvm::APInt Value) {
691:   llvm::LLVMContext &Ctx = CGM.getLLVMContext();
692: 
693:   // Convert the APInt value to an array of uint32_t words
694:   llvm::SmallVector<uint32_t> Words;
695: 
696:   while (Value.ugt(0)) {
697:     uint32_t Word = Value.trunc(32).getZExtValue();
698:     Value.lshrInPlace(32);
699: 
700:     Words.push_back(Word);
```
- **EN**: This block uses control flow (while) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（while）细化 目标相关的 ABI 与代码生成 行为。

### Lines 701-720
```cpp
701:   }
702:   if (Words.size() == 0)
703:     Words.push_back(0);
704: 
705:   if (IntegralType)
706:     return llvm::TargetExtType::get(Ctx, "spirv.IntegralConstant",
707:                                     {IntegralType}, Words);
708:   return llvm::TargetExtType::get(Ctx, "spirv.Literal", {}, Words);
709: }
710: 
711: static llvm::Type *getInlineSpirvType(CodeGenModule &CGM,
712:                                       const HLSLInlineSpirvType *SpirvType) {
713:   llvm::LLVMContext &Ctx = CGM.getLLVMContext();
714: 
715:   llvm::SmallVector<llvm::Type *> Operands;
716: 
717:   for (auto &Operand : SpirvType->getOperands()) {
718:     using SpirvOperandKind = SpirvOperand::SpirvOperandKind;
719: 
720:     llvm::Type *Result = nullptr;
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 721-740
```cpp
721:     switch (Operand.getKind()) {
722:     case SpirvOperandKind::ConstantId: {
723:       llvm::Type *IntegralType =
724:           CGM.getTypes().ConvertType(Operand.getResultType());
725: 
726:       Result = getInlineSpirvConstant(CGM, IntegralType, Operand.getValue());
727:       break;
728:     }
729:     case SpirvOperandKind::Literal: {
730:       Result = getInlineSpirvConstant(CGM, nullptr, Operand.getValue());
731:       break;
732:     }
733:     case SpirvOperandKind::TypeId: {
734:       QualType TypeOperand = Operand.getResultType();
735:       if (const auto *RD = TypeOperand->getAsRecordDecl()) {
736:         assert(RD->isCompleteDefinition() &&
737:                "Type completion should have been required in Sema");
738: 
739:         const FieldDecl *HandleField = RD->findFirstNamedDataMember();
740:         if (HandleField) {
```
- **EN**: This block uses control flow (if, switch, case) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 741-760
```cpp
741:           QualType ResourceType = HandleField->getType();
742:           if (ResourceType->getAs<HLSLAttributedResourceType>()) {
743:             TypeOperand = ResourceType;
744:           }
745:         }
746:       }
747:       Result = CGM.getTypes().ConvertType(TypeOperand);
748:       break;
749:     }
750:     default:
751:       llvm_unreachable("HLSLInlineSpirvType had invalid operand!");
752:       break;
753:     }
754: 
755:     assert(Result);
756:     Operands.push_back(Result);
757:   }
758: 
759:   return llvm::TargetExtType::get(Ctx, "spirv.Type", Operands,
760:                                   {SpirvType->getOpcode(), SpirvType->getSize(),
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 761-780
```cpp
761:                                    SpirvType->getAlignment()});
762: }
763: 
764: llvm::Type *CommonSPIRTargetCodeGenInfo::getHLSLType(
765:     CodeGenModule &CGM, const Type *Ty,
766:     const CGHLSLOffsetInfo &OffsetInfo) const {
767:   llvm::LLVMContext &Ctx = CGM.getLLVMContext();
768: 
769:   if (auto *SpirvType = dyn_cast<HLSLInlineSpirvType>(Ty))
770:     return getInlineSpirvType(CGM, SpirvType);
771: 
772:   auto *ResType = dyn_cast<HLSLAttributedResourceType>(Ty);
773:   if (!ResType)
774:     return nullptr;
775: 
776:   const HLSLAttributedResourceType::Attributes &ResAttrs = ResType->getAttrs();
777:   switch (ResAttrs.ResourceClass) {
778:   case llvm::dxil::ResourceClass::UAV:
779:   case llvm::dxil::ResourceClass::SRV: {
780:     // TypedBuffer and RawBuffer both need element type
```
- **EN**: This block uses control flow (if, switch, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 781-800
```cpp
781:     QualType ContainedTy = ResType->getContainedType();
782:     if (ContainedTy.isNull())
783:       return nullptr;
784: 
785:     assert(!ResAttrs.IsROV &&
786:            "Rasterizer order views not implemented for SPIR-V yet");
787: 
788:     if (!ResAttrs.RawBuffer) {
789:       // convert element type
790:       return getSPIRVImageTypeFromHLSLResource(ResAttrs, ContainedTy, CGM);
791:     }
792: 
793:     if (ResAttrs.IsCounter) {
794:       llvm::Type *ElemType = llvm::Type::getInt32Ty(Ctx);
795:       uint32_t StorageClass = /* StorageBuffer storage class */ 12;
796:       return llvm::TargetExtType::get(Ctx, "spirv.VulkanBuffer", {ElemType},
797:                                       {StorageClass, true});
798:     }
799:     llvm::Type *ElemType = CGM.getTypes().ConvertTypeForMem(ContainedTy);
800:     llvm::ArrayType *RuntimeArrayType = llvm::ArrayType::get(ElemType, 0);
```
- **EN**: This block defines callable entry points like `getSPIRVImageTypeFromHLSLResource`; uses control flow (if, for) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getSPIRVImageTypeFromHLSLResource`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 801-820
```cpp
801:     uint32_t StorageClass = /* StorageBuffer storage class */ 12;
802:     bool IsWritable = ResAttrs.ResourceClass == llvm::dxil::ResourceClass::UAV;
803:     return llvm::TargetExtType::get(Ctx, "spirv.VulkanBuffer",
804:                                     {RuntimeArrayType},
805:                                     {StorageClass, IsWritable});
806:   }
807:   case llvm::dxil::ResourceClass::CBuffer: {
808:     QualType ContainedTy = ResType->getContainedType();
809:     if (ContainedTy.isNull() || !ContainedTy->isStructureType())
810:       return nullptr;
811: 
812:     llvm::StructType *BufferLayoutTy =
813:         HLSLBufferLayoutBuilder(CGM).layOutStruct(
814:             ContainedTy->getAsCanonical<RecordType>(), OffsetInfo);
815:     uint32_t StorageClass = /* Uniform storage class */ 2;
816:     return llvm::TargetExtType::get(Ctx, "spirv.VulkanBuffer", {BufferLayoutTy},
817:                                     {StorageClass, false});
818:     break;
819:   }
820:   case llvm::dxil::ResourceClass::Sampler:
```
- **EN**: This block defines callable entry points like `HLSLBufferLayoutBuilder`; uses control flow (if, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `HLSLBufferLayoutBuilder`；通过控制流（if, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 821-840
```cpp
821:     return llvm::TargetExtType::get(Ctx, "spirv.Sampler");
822:   }
823:   return nullptr;
824: }
825: 
826: static unsigned
827: getImageFormat(const LangOptions &LangOpts,
828:                const HLSLAttributedResourceType::Attributes &attributes,
829:                llvm::Type *SampledType, QualType Ty, unsigned NumChannels) {
830:   // For images with `Sampled` operand equal to 2, there are restrictions on
831:   // using the Unknown image format. To avoid these restrictions in common
832:   // cases, we guess an image format for them based on the sampled type and the
833:   // number of channels. This is intended to match the behaviour of DXC.
834:   if (LangOpts.HLSLSpvUseUnknownImageFormat ||
835:       attributes.ResourceClass != llvm::dxil::ResourceClass::UAV) {
836:     return 0; // Unknown
837:   }
838: 
839:   if (SampledType->isIntegerTy(32)) {
840:     if (Ty->isSignedIntegerType()) {
```
- **EN**: This block defines callable entry points like `get`, `getImageFormat`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getImageFormat`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 841-860
```cpp
841:       if (NumChannels == 1)
842:         return 24; // R32i
843:       if (NumChannels == 2)
844:         return 25; // Rg32i
845:       if (NumChannels == 4)
846:         return 21; // Rgba32i
847:     } else {
848:       if (NumChannels == 1)
849:         return 33; // R32ui
850:       if (NumChannels == 2)
851:         return 35; // Rg32ui
852:       if (NumChannels == 4)
853:         return 30; // Rgba32ui
854:     }
855:   } else if (SampledType->isIntegerTy(64)) {
856:     if (NumChannels == 1) {
857:       if (Ty->isSignedIntegerType()) {
858:         return 41; // R64i
859:       }
860:       return 40; // R64ui
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 861-880
```cpp
861:     }
862:   } else if (SampledType->isFloatTy()) {
863:     if (NumChannels == 1)
864:       return 3; // R32f
865:     if (NumChannels == 2)
866:       return 6; // Rg32f
867:     if (NumChannels == 4)
868:       return 1; // Rgba32f
869:   }
870: 
871:   return 0; // Unknown
872: }
873: 
874: llvm::Type *CommonSPIRTargetCodeGenInfo::getSPIRVImageTypeFromHLSLResource(
875:     const HLSLAttributedResourceType::Attributes &attributes, QualType Ty,
876:     CodeGenModule &CGM) const {
877:   llvm::LLVMContext &Ctx = CGM.getLLVMContext();
878: 
879:   unsigned NumChannels = 1;
880:   Ty = Ty->getCanonicalTypeUnqualified();
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 881-900
```cpp
881:   if (const VectorType *V = dyn_cast<VectorType>(Ty)) {
882:     NumChannels = V->getNumElements();
883:     Ty = V->getElementType();
884:   }
885:   assert(!Ty->isVectorType() && "We still have a vector type.");
886: 
887:   llvm::Type *SampledType = CGM.getTypes().ConvertTypeForMem(Ty);
888: 
889:   assert((SampledType->isIntegerTy() || SampledType->isFloatingPointTy()) &&
890:          "The element type for a SPIR-V resource must be a scalar integer or "
891:          "floating point type.");
892: 
893:   // These parameters correspond to the operands to the OpTypeImage SPIR-V
894:   // instruction. See
895:   // https://registry.khronos.org/SPIR-V/specs/unified1/SPIRV.html#OpTypeImage.
896:   SmallVector<unsigned, 6> IntParams(6, 0);
897: 
898:   const char *Name =
899:       Ty->isSignedIntegerType() ? "spirv.SignedImage" : "spirv.Image";
900: 
```
- **EN**: This block defines callable entry points like `IntParams`; uses control flow (if, for) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `IntParams`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 901-920
```cpp
901:   // Dim
902:   switch (attributes.ResourceDimension) {
903:   case llvm::dxil::ResourceDimension::Dim1D:
904:     IntParams[0] = 0;
905:     break;
906:   case llvm::dxil::ResourceDimension::Dim2D:
907:     IntParams[0] = 1;
908:     break;
909:   case llvm::dxil::ResourceDimension::Dim3D:
910:     IntParams[0] = 2;
911:     break;
912:   case llvm::dxil::ResourceDimension::Cube:
913:     IntParams[0] = 3;
914:     break;
915:   case llvm::dxil::ResourceDimension::Unknown:
916:     IntParams[0] = 5;
917:     break;
918:   }
919: 
920:   // Depth
```
- **EN**: This block uses control flow (switch, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（switch, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 921-940
```cpp
921:   // HLSL does not indicate if it is a depth texture or not, so we use unknown.
922:   IntParams[1] = 2;
923: 
924:   // Arrayed
925:   IntParams[2] = 0;
926: 
927:   // MS
928:   IntParams[3] = 0;
929: 
930:   // Sampled
931:   IntParams[4] =
932:       attributes.ResourceClass == llvm::dxil::ResourceClass::UAV ? 2 : 1;
933: 
934:   // Image format.
935:   IntParams[5] = getImageFormat(CGM.getLangOpts(), attributes, SampledType, Ty,
936:                                 NumChannels);
937: 
938:   llvm::TargetExtType *ImageType =
939:       llvm::TargetExtType::get(Ctx, Name, {SampledType}, IntParams);
940:   return ImageType;
```
- **EN**: This block documents intent or context for the surrounding target-specific ABI and code generation code.
- **CN**: 该代码块说明周围 目标相关的 ABI 与代码生成 代码的意图或上下文。

### Lines 941-951
```cpp
941: }
942: 
943: std::unique_ptr<TargetCodeGenInfo>
944: CodeGen::createCommonSPIRTargetCodeGenInfo(CodeGenModule &CGM) {
945:   return std::make_unique<CommonSPIRTargetCodeGenInfo>(CGM.getTypes());
946: }
947: 
948: std::unique_ptr<TargetCodeGenInfo>
949: CodeGen::createSPIRVTargetCodeGenInfo(CodeGenModule &CGM) {
950:   return std::make_unique<SPIRVTargetCodeGenInfo>(CGM.getTypes());
951: }
```
- **EN**: This block defines callable entry points like `createCommonSPIRTargetCodeGenInfo`, `createSPIRVTargetCodeGenInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `createCommonSPIRTargetCodeGenInfo`, `createSPIRVTargetCodeGenInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

## Key Concepts / 关键概念

- **CGM**: Central symbol in this file's implementation of target-specific ABI and code generation. / 是该文件实现 目标相关的 ABI 与代码生成 时的核心符号。
- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **getContext**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **QualType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Ctx**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ABIArgInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **LangAS**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **IntParams**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfoImpl.h`, `HLSLBufferLayoutBuilder.h`, `TargetInfo.h`, `stdint.h`
- **Clang libraries / Clang 库**: `clang/AST/DeclCXX.h`, `clang/Basic/LangOptions.h`, `clang/Basic/OpenCLImageTypes.def`, `clang/Basic/OpenCLExtensionTypes.def`
- **LLVM libraries / LLVM 库**: `llvm/IR/DerivedTypes.h`
- **Other headers / 其他头文件**: `utility`
