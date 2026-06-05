# TargetInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/TargetInfo.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Provides target-specific CodeGen interfaces, factories, and ABI hooks.
- **Purpose (CN) / 目的（中文）**: 提供目标相关的 CodeGen 接口、工厂函数与 ABI 钩子。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: //===---- TargetInfo.cpp - Encapsulate target details -----------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // These classes wrap the information about a call or function
10: // definition used to handle ABI compliancy.
11: //
12: //===----------------------------------------------------------------------===//
13: 
14: #include "TargetInfo.h"
15: #include "ABIInfo.h"
16: #include "ABIInfoImpl.h"
```
- **EN**: This block imports local CodeGen headers `TargetInfo.h`, `ABIInfo.h`, `ABIInfoImpl.h`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `TargetInfo.h`, `ABIInfo.h`, `ABIInfoImpl.h`；包含影响本编译单元构建方式的预处理结构。

### Lines 17-32
```cpp
17: #include "CodeGenFunction.h"
18: #include "clang/Basic/CodeGenOptions.h"
19: #include "clang/CodeGen/CGFunctionInfo.h"
20: #include "llvm/ADT/StringExtras.h"
21: #include "llvm/ADT/Twine.h"
22: #include "llvm/IR/Function.h"
23: #include "llvm/IR/Type.h"
24: #include "llvm/Support/raw_ostream.h"
25: 
26: using namespace clang;
27: using namespace CodeGen;
28: 
29: LLVM_DUMP_METHOD void ABIArgInfo::dump() const {
30:   raw_ostream &OS = llvm::errs();
31:   OS << "(ABIArgInfo Kind=";
32:   switch (TheKind) {
```
- **EN**: This block imports local CodeGen headers `CodeGenFunction.h`; Clang headers `clang/Basic/CodeGenOptions.h`, `clang/CodeGen/CGFunctionInfo.h`; LLVM headers `llvm/ADT/StringExtras.h`, `llvm/ADT/Twine.h`, `llvm/IR/Function.h`, and 2 more; opens or references namespaces `clang`, `CodeGen`; defines callable entry points like `dump`; uses control flow (switch) to specialize Clang CodeGen support; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CodeGenFunction.h`；Clang 头文件 `clang/Basic/CodeGenOptions.h`, `clang/CodeGen/CGFunctionInfo.h`；LLVM 头文件 `llvm/ADT/StringExtras.h`, `llvm/ADT/Twine.h`, `llvm/IR/Function.h`, and 2 more；打开或引用命名空间 `clang`, `CodeGen`；定义可调用入口，例如 `dump`；通过控制流（switch）细化 Clang CodeGen 支撑逻辑 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 33-48
```cpp
33:   case Direct:
34:     OS << "Direct Type=";
35:     if (llvm::Type *Ty = getCoerceToType())
36:       Ty->print(OS);
37:     else
38:       OS << "null";
39:     break;
40:   case Extend:
41:     OS << "Extend";
42:     break;
43:   case Ignore:
44:     OS << "Ignore";
45:     break;
46:   case InAlloca:
47:     OS << "InAlloca Offset=" << getInAllocaFieldIndex();
48:     break;
```
- **EN**: This block uses control flow (if, case) to specialize Clang CodeGen support.
- **CN**: 该代码块通过控制流（if, case）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 49-64
```cpp
49:   case Indirect:
50:     OS << "Indirect Align=" << getIndirectAlign().getQuantity()
51:        << " ByVal=" << getIndirectByVal()
52:        << " Realign=" << getIndirectRealign();
53:     break;
54:   case IndirectAliased:
55:     OS << "Indirect Align=" << getIndirectAlign().getQuantity()
56:        << " AadrSpace=" << getIndirectAddrSpace()
57:        << " Realign=" << getIndirectRealign();
58:     break;
59:   case Expand:
60:     OS << "Expand";
61:     break;
62:   case CoerceAndExpand:
63:     OS << "CoerceAndExpand Type=";
64:     getCoerceAndExpandType()->print(OS);
```
- **EN**: This block spells out callable entry points like `getCoerceAndExpandType`; uses control flow (case) to specialize Clang CodeGen support.
- **CN**: 该代码块给出可调用入口的声明，例如 `getCoerceAndExpandType`；通过控制流（case）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 65-80
```cpp
65:     break;
66:   case TargetSpecific:
67:     OS << "TargetSpecific Type=";
68:     if (llvm::Type *Ty = getCoerceToType())
69:       Ty->print(OS);
70:     else
71:       OS << "null";
72:     break;
73:   }
74:   OS << ")\n";
75: }
76: 
77: TargetCodeGenInfo::TargetCodeGenInfo(std::unique_ptr<ABIInfo> Info)
78:     : Info(std::move(Info)) {}
79: 
80: TargetCodeGenInfo::~TargetCodeGenInfo() = default;
```
- **EN**: This block defines callable entry points like `TargetCodeGenInfo`; uses control flow (if, case) to specialize Clang CodeGen support.
- **CN**: 该代码块定义可调用入口，例如 `TargetCodeGenInfo`；通过控制流（if, case）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 81-96
```cpp
81: 
82: // If someone can figure out a general rule for this, that would be great.
83: // It's probably just doomed to be platform-dependent, though.
84: unsigned TargetCodeGenInfo::getSizeOfUnwindException() const {
85:   if (getABIInfo().getCodeGenOpts().hasSEHExceptions())
86:     return getABIInfo().getDataLayout().getPointerSizeInBits() > 32 ? 64 : 48;
87:   // Verified for:
88:   //   x86-64     FreeBSD, Linux, Darwin
89:   //   x86-32     FreeBSD, Linux, Darwin
90:   //   PowerPC    Linux
91:   //   ARM        Darwin (*not* EABI)
92:   //   AArch64    Linux
93:   return 32;
94: }
95: 
96: bool TargetCodeGenInfo::isNoProtoCallVariadic(const CallArgList &args,
```
- **EN**: This block defines callable entry points like `getSizeOfUnwindException`; uses control flow (if) to specialize Clang CodeGen support.
- **CN**: 该代码块定义可调用入口，例如 `getSizeOfUnwindException`；通过控制流（if）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 97-112
```cpp
 97:                                      const FunctionNoProtoType *fnType) const {
 98:   // The following conventions are known to require this to be false:
 99:   //   x86_stdcall
100:   //   MIPS
101:   // For everything else, we just prefer false unless we opt out.
102:   return false;
103: }
104: 
105: void
106: TargetCodeGenInfo::getDependentLibraryOption(llvm::StringRef Lib,
107:                                              llvm::SmallString<24> &Opt) const {
108:   // This assumes the user is passing a library name like "rt" instead of a
109:   // filename like "librt.a/so", and that they don't care whether it's static or
110:   // dynamic.
111:   Opt = "-l";
112:   Opt += Lib;
```
- **EN**: This block defines callable entry points like `getDependentLibraryOption`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `getDependentLibraryOption`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 113-128
```cpp
113: }
114: 
115: unsigned TargetCodeGenInfo::getDeviceKernelCallingConv() const {
116:   // Device kernels are called via an explicit runtime API with arguments,
117:   // such as set with clSetKernelArg() for OpenCL, not as normal
118:   // sub-functions.  This uses a modified version of the C calling convention
119:   // which simplifies the treatment of non-scalar types.  (The rule adjustment
120:   // hapens in CodeGenTypes::arrangeLLVMFunctionInfo.)
121:   //
122:   // Outside of OpenCL, kernels currently do not exist for CPU targets.
123:   assert(getABIInfo().getContext().getLangOpts().OpenCL &&
124:          "Kernel calling convention only defined for OpenCL");
125:   return llvm::CallingConv::C;
126: }
127: 
128: void TargetCodeGenInfo::setOCLKernelStubCallingConvention(
```
- **EN**: This block defines callable entry points like `getDeviceKernelCallingConv`; uses control flow (for) to specialize Clang CodeGen support; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getDeviceKernelCallingConv`；通过控制流（for）细化 Clang CodeGen 支撑逻辑 行为；使用断言或不可达标记保护关键不变量。

### Lines 129-144
```cpp
129:     const FunctionType *&FT) const {
130:   FT = getABIInfo().getContext().adjustFunctionType(
131:       FT, FT->getExtInfo().withCallingConv(CC_C));
132: }
133: 
134: llvm::Constant *TargetCodeGenInfo::getNullPointer(const CodeGen::CodeGenModule &CGM,
135:     llvm::PointerType *T, QualType QT) const {
136:   return llvm::ConstantPointerNull::get(T);
137: }
138: 
139: LangAS TargetCodeGenInfo::getGlobalVarAddressSpace(CodeGenModule &CGM,
140:                                                    const VarDecl *D) const {
141:   assert(!CGM.getLangOpts().OpenCL &&
142:          !(CGM.getLangOpts().CUDA && CGM.getLangOpts().CUDAIsDevice) &&
143:          "Address space agnostic languages only");
144:   return D ? D->getType().getAddressSpace() : LangAS::Default;
```
- **EN**: This block defines callable entry points like `get`, `getGlobalVarAddressSpace`; returns or forwards computed values for the surrounding Clang CodeGen support logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getGlobalVarAddressSpace`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 145-160
```cpp
145: }
146: 
147: StringRef
148: TargetCodeGenInfo::getLLVMSyncScopeStr(const LangOptions &LangOpts,
149:                                        SyncScope Scope,
150:                                        llvm::AtomicOrdering Ordering) const {
151:   return ""; /* default sync scope */
152: }
153: 
154: llvm::SyncScope::ID
155: TargetCodeGenInfo::getLLVMSyncScopeID(const LangOptions &LangOpts,
156:                                       SyncScope Scope,
157:                                       llvm::AtomicOrdering Ordering,
158:                                       llvm::LLVMContext &Ctx) const {
159:   return Ctx.getOrInsertSyncScopeID(
160:       getLLVMSyncScopeStr(LangOpts, Scope, Ordering));
```
- **EN**: This block defines callable entry points like `getLLVMSyncScopeStr`, `getLLVMSyncScopeID`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `getLLVMSyncScopeStr`, `getLLVMSyncScopeID`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 161-176
```cpp
161: }
162: 
163: void TargetCodeGenInfo::addStackProbeTargetAttributes(
164:     const Decl *D, llvm::GlobalValue *GV, CodeGen::CodeGenModule &CGM) const {
165:   if (llvm::Function *Fn = dyn_cast_or_null<llvm::Function>(GV)) {
166:     if (CGM.getCodeGenOpts().StackProbeSize != 4096)
167:       Fn->addFnAttr("stack-probe-size",
168:                     llvm::utostr(CGM.getCodeGenOpts().StackProbeSize));
169:     if (CGM.getCodeGenOpts().NoStackArgProbe)
170:       Fn->addFnAttr("no-stack-arg-probe");
171:   }
172: }
173: 
174: /// Create an OpenCL kernel for an enqueued block.
175: ///
176: /// The kernel has the same function type as the block invoke function. Its
```
- **EN**: This block defines callable entry points like `addStackProbeTargetAttributes`; uses control flow (if) to specialize Clang CodeGen support.
- **CN**: 该代码块定义可调用入口，例如 `addStackProbeTargetAttributes`；通过控制流（if）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 177-192
```cpp
177: /// name is the name of the block invoke function postfixed with "_kernel".
178: /// It simply calls the block invoke function then returns.
179: llvm::Value *TargetCodeGenInfo::createEnqueuedBlockKernel(
180:     CodeGenFunction &CGF, llvm::Function *Invoke, llvm::Type *BlockTy) const {
181:   auto *InvokeFT = Invoke->getFunctionType();
182:   auto &C = CGF.getLLVMContext();
183:   std::string Name = Invoke->getName().str() + "_kernel";
184:   auto *FT = llvm::FunctionType::get(llvm::Type::getVoidTy(C),
185:                                      InvokeFT->params(), false);
186:   auto *F = llvm::Function::Create(FT, llvm::GlobalValue::ExternalLinkage, Name,
187:                                    &CGF.CGM.getModule());
188:   llvm::CallingConv::ID KernelCC =
189:       CGF.getTypes().ClangCallConvToLLVMCallConv(CallingConv::CC_DeviceKernel);
190:   F->setCallingConv(KernelCC);
191: 
192:   llvm::AttrBuilder KernelAttrs(C);
```
- **EN**: This block defines callable entry points like `KernelAttrs`.
- **CN**: 该代码块定义可调用入口，例如 `KernelAttrs`。

### Lines 193-208
```cpp
193: 
194:   // FIXME: This is missing setTargetAttributes
195:   CGF.CGM.addDefaultFunctionDefinitionAttributes(KernelAttrs);
196:   F->addFnAttrs(KernelAttrs);
197: 
198:   auto IP = CGF.Builder.saveIP();
199:   auto *BB = llvm::BasicBlock::Create(C, "entry", F);
200:   auto &Builder = CGF.Builder;
201:   Builder.SetInsertPoint(BB);
202:   llvm::SmallVector<llvm::Value *, 2> Args(llvm::make_pointer_range(F->args()));
203:   llvm::CallInst *Call = Builder.CreateCall(Invoke, Args);
204:   Call->setCallingConv(Invoke->getCallingConv());
205: 
206:   Builder.CreateRetVoid();
207:   Builder.restoreIP(IP);
208:   return F;
```
- **EN**: This block spells out callable entry points like `Args`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块给出可调用入口的声明，例如 `Args`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 209-224
```cpp
209: }
210: 
211: void TargetCodeGenInfo::setBranchProtectionFnAttributes(
212:     const TargetInfo::BranchProtectionInfo &BPI, llvm::Function &F) {
213:   // Called on already created and initialized function where attributes already
214:   // set from command line attributes but some might need to be removed as the
215:   // actual BPI is different.
216:   if (BPI.SignReturnAddr != LangOptions::SignReturnAddressScopeKind::None) {
217:     F.addFnAttr("sign-return-address", BPI.getSignReturnAddrStr());
218:     F.addFnAttr("sign-return-address-key", BPI.getSignKeyStr());
219:   } else {
220:     if (F.hasFnAttribute("sign-return-address"))
221:       F.removeFnAttr("sign-return-address");
222:     if (F.hasFnAttribute("sign-return-address-key"))
223:       F.removeFnAttr("sign-return-address-key");
224:   }
```
- **EN**: This block defines callable entry points like `setBranchProtectionFnAttributes`; uses control flow (if) to specialize Clang CodeGen support.
- **CN**: 该代码块定义可调用入口，例如 `setBranchProtectionFnAttributes`；通过控制流（if）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 225-240
```cpp
225: 
226:   auto AddRemoveAttributeAsSet = [&](bool Set, const StringRef &ModAttr) {
227:     if (Set)
228:       F.addFnAttr(ModAttr);
229:     else if (F.hasFnAttribute(ModAttr))
230:       F.removeFnAttr(ModAttr);
231:   };
232: 
233:   AddRemoveAttributeAsSet(BPI.BranchTargetEnforcement,
234:                           "branch-target-enforcement");
235:   AddRemoveAttributeAsSet(BPI.BranchProtectionPAuthLR,
236:                           "branch-protection-pauth-lr");
237:   AddRemoveAttributeAsSet(BPI.GuardedControlStack, "guarded-control-stack");
238: }
239: 
240: void TargetCodeGenInfo::initBranchProtectionFnAttributes(
```
- **EN**: This block defines callable entry points like `AddRemoveAttributeAsSet`; uses control flow (if) to specialize Clang CodeGen support.
- **CN**: 该代码块定义可调用入口，例如 `AddRemoveAttributeAsSet`；通过控制流（if）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 241-256
```cpp
241:     const TargetInfo::BranchProtectionInfo &BPI, llvm::AttrBuilder &FuncAttrs) {
242:   // Only used for initializing attributes in the AttrBuilder, which will not
243:   // contain any of these attributes so no need to remove anything.
244:   if (BPI.SignReturnAddr != LangOptions::SignReturnAddressScopeKind::None) {
245:     FuncAttrs.addAttribute("sign-return-address", BPI.getSignReturnAddrStr());
246:     FuncAttrs.addAttribute("sign-return-address-key", BPI.getSignKeyStr());
247:   }
248:   if (BPI.BranchTargetEnforcement)
249:     FuncAttrs.addAttribute("branch-target-enforcement");
250:   if (BPI.BranchProtectionPAuthLR)
251:     FuncAttrs.addAttribute("branch-protection-pauth-lr");
252:   if (BPI.GuardedControlStack)
253:     FuncAttrs.addAttribute("guarded-control-stack");
254: }
255: 
256: void TargetCodeGenInfo::setPointerAuthFnAttributes(
```
- **EN**: This block uses control flow (if) to specialize Clang CodeGen support.
- **CN**: 该代码块通过控制流（if）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 257-272
```cpp
257:     const PointerAuthOptions &Opts, llvm::Function &F) {
258:   auto UpdateAttr = [&F](bool AttrShouldExist, StringRef AttrName) {
259:     if (AttrShouldExist && !F.hasFnAttribute(AttrName))
260:       F.addFnAttr(AttrName);
261:     if (!AttrShouldExist && F.hasFnAttribute(AttrName))
262:       F.removeFnAttr(AttrName);
263:   };
264:   UpdateAttr(Opts.ReturnAddresses, "ptrauth-returns");
265:   UpdateAttr((bool)Opts.FunctionPointers, "ptrauth-calls");
266:   UpdateAttr(Opts.AuthTraps, "ptrauth-auth-traps");
267:   UpdateAttr(Opts.IndirectGotos, "ptrauth-indirect-gotos");
268:   UpdateAttr(Opts.AArch64JumpTableHardening, "aarch64-jump-table-hardening");
269: }
270: 
271: void TargetCodeGenInfo::initPointerAuthFnAttributes(
272:     const PointerAuthOptions &Opts, llvm::AttrBuilder &FuncAttrs) {
```
- **EN**: This block defines callable entry points like `UpdateAttr`, `initPointerAuthFnAttributes`; uses control flow (if) to specialize Clang CodeGen support.
- **CN**: 该代码块定义可调用入口，例如 `UpdateAttr`, `initPointerAuthFnAttributes`；通过控制流（if）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 273-288
```cpp
273:   if (Opts.ReturnAddresses)
274:     FuncAttrs.addAttribute("ptrauth-returns");
275:   if (Opts.FunctionPointers)
276:     FuncAttrs.addAttribute("ptrauth-calls");
277:   if (Opts.AuthTraps)
278:     FuncAttrs.addAttribute("ptrauth-auth-traps");
279:   if (Opts.IndirectGotos)
280:     FuncAttrs.addAttribute("ptrauth-indirect-gotos");
281:   if (Opts.AArch64JumpTableHardening)
282:     FuncAttrs.addAttribute("aarch64-jump-table-hardening");
283: }
284: 
285: namespace {
286: class DefaultTargetCodeGenInfo : public TargetCodeGenInfo {
287: public:
288:   DefaultTargetCodeGenInfo(CodeGen::CodeGenTypes &CGT)
```
- **EN**: This block introduces declarations such as `DefaultTargetCodeGenInfo`; uses control flow (if) to specialize Clang CodeGen support.
- **CN**: 该代码块给出诸如 `DefaultTargetCodeGenInfo` 的声明；通过控制流（if）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 289-296
```cpp
289:       : TargetCodeGenInfo(std::make_unique<DefaultABIInfo>(CGT)) {}
290: };
291: } // namespace
292: 
293: std::unique_ptr<TargetCodeGenInfo>
294: CodeGen::createDefaultTargetCodeGenInfo(CodeGenModule &CGM) {
295:   return std::make_unique<DefaultTargetCodeGenInfo>(CGM.getTypes());
296: }
```
- **EN**: This block opens or references namespaces `std`; defines callable entry points like `TargetCodeGenInfo`, `createDefaultTargetCodeGenInfo`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块打开或引用命名空间 `std`；定义可调用入口，例如 `TargetCodeGenInfo`, `createDefaultTargetCodeGenInfo`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

## Key Concepts / 关键概念

- **TargetCodeGenInfo**: Likely stores or computes descriptive metadata that drives Clang CodeGen support. / 很可能用于保存或计算驱动 Clang CodeGen 支撑逻辑 的描述性元数据。
- **BPI**: Central symbol in this file's implementation of Clang CodeGen support. / 是该文件实现 Clang CodeGen 支撑逻辑 时的核心符号。
- **CGM**: Central symbol in this file's implementation of Clang CodeGen support. / 是该文件实现 Clang CodeGen 支撑逻辑 时的核心符号。
- **FuncAttrs**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Opts**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **addAttribute**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Function**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `TargetInfo.h`, `ABIInfo.h`, `ABIInfoImpl.h`, `CodeGenFunction.h`
- **Clang libraries / Clang 库**: `clang/Basic/CodeGenOptions.h`, `clang/CodeGen/CGFunctionInfo.h`
- **LLVM libraries / LLVM 库**: `llvm/ADT/StringExtras.h`, `llvm/ADT/Twine.h`, `llvm/IR/Function.h`, `llvm/IR/Type.h`, `llvm/Support/raw_ostream.h`
