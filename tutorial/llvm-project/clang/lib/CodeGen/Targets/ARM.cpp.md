# ARM.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/Targets/ARM.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements target-specific ABI rules and lowering details for ARM.
- **Purpose (CN) / 目的（中文）**: 实现 ARM 目标相关的 ABI 规则与降级细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: //===- ARM.cpp ------------------------------------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: 
 9: #include "ABIInfoImpl.h"
10: #include "TargetInfo.h"
11: 
12: using namespace clang;
13: using namespace clang::CodeGen;
14: 
15: //===----------------------------------------------------------------------===//
16: // ARM ABI Implementation
17: //===----------------------------------------------------------------------===//
18: 
19: namespace {
20: 
```
- **EN**: This block imports local CodeGen headers `ABIInfoImpl.h`, `TargetInfo.h`; opens or references namespaces `clang`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `ABIInfoImpl.h`, `TargetInfo.h`；打开或引用命名空间 `clang`；包含影响本编译单元构建方式的预处理结构。

### Lines 21-40
```cpp
21: class ARMABIInfo : public ABIInfo {
22:   ARMABIKind Kind;
23:   bool IsFloatABISoftFP;
24: 
25: public:
26:   ARMABIInfo(CodeGenTypes &CGT, ARMABIKind Kind) : ABIInfo(CGT), Kind(Kind) {
27:     setCCs();
28:     IsFloatABISoftFP = CGT.getCodeGenOpts().FloatABI == "softfp" ||
29:         CGT.getCodeGenOpts().FloatABI == ""; // default
30:   }
31: 
32:   bool isEABI() const {
33:     switch (getTarget().getTriple().getEnvironment()) {
34:     case llvm::Triple::Android:
35:     case llvm::Triple::EABI:
36:     case llvm::Triple::EABIHF:
37:     case llvm::Triple::GNUEABI:
38:     case llvm::Triple::GNUEABIT64:
39:     case llvm::Triple::GNUEABIHF:
40:     case llvm::Triple::GNUEABIHFT64:
```
- **EN**: This block introduces declarations such as `ARMABIInfo`; defines callable entry points like `ARMABIInfo`, `setCCs`, `isEABI`; uses control flow (switch, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出诸如 `ARMABIInfo` 的声明；定义可调用入口，例如 `ARMABIInfo`, `setCCs`, `isEABI`；通过控制流（switch, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 41-60
```cpp
41:     case llvm::Triple::MuslEABI:
42:     case llvm::Triple::MuslEABIHF:
43:       return true;
44:     default:
45:       return getTarget().getTriple().isOHOSFamily();
46:     }
47:   }
48: 
49:   bool isEABIHF() const {
50:     switch (getTarget().getTriple().getEnvironment()) {
51:     case llvm::Triple::EABIHF:
52:     case llvm::Triple::GNUEABIHF:
53:     case llvm::Triple::GNUEABIHFT64:
54:     case llvm::Triple::MuslEABIHF:
55:       return true;
56:     default:
57:       return false;
58:     }
59:   }
60: 
```
- **EN**: This block defines callable entry points like `getTarget`, `isEABIHF`; uses control flow (switch, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getTarget`, `isEABIHF`；通过控制流（switch, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 61-80
```cpp
61:   ARMABIKind getABIKind() const { return Kind; }
62: 
63:   bool allowBFloatArgsAndRet() const override {
64:     return !IsFloatABISoftFP && getTarget().hasBFloat16Type();
65:   }
66: 
67: private:
68:   ABIArgInfo classifyReturnType(QualType RetTy, bool isVariadic,
69:                                 unsigned functionCallConv) const;
70:   ABIArgInfo classifyArgumentType(QualType RetTy, bool isVariadic,
71:                                   unsigned functionCallConv) const;
72:   ABIArgInfo classifyHomogeneousAggregate(QualType Ty, const Type *Base,
73:                                           uint64_t Members) const;
74:   bool shouldIgnoreEmptyArg(QualType Ty) const;
75:   ABIArgInfo coerceIllegalVector(QualType Ty) const;
76:   bool isIllegalVectorType(QualType Ty) const;
77:   bool containsAnyFP16Vectors(QualType Ty) const;
78: 
79:   bool isHomogeneousAggregateBaseType(QualType Ty) const override;
80:   bool isHomogeneousAggregateSmallEnough(const Type *Ty,
```
- **EN**: This block defines callable entry points like `getABIKind`, `allowBFloatArgsAndRet`, `classifyReturnType`, `classifyArgumentType`, `classifyHomogeneousAggregate`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `getABIKind`, `allowBFloatArgsAndRet`, `classifyReturnType`, `classifyArgumentType`, `classifyHomogeneousAggregate`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 81-100
```cpp
 81:                                          uint64_t Members) const override;
 82:   bool isZeroLengthBitfieldPermittedInHomogeneousAggregate() const override;
 83: 
 84:   bool isEffectivelyAAPCS_VFP(unsigned callConvention, bool acceptHalf) const;
 85: 
 86:   void computeInfo(CGFunctionInfo &FI) const override;
 87: 
 88:   RValue EmitVAArg(CodeGenFunction &CGF, Address VAListAddr, QualType Ty,
 89:                    AggValueSlot Slot) const override;
 90: 
 91:   llvm::CallingConv::ID getLLVMDefaultCC() const;
 92:   llvm::CallingConv::ID getABIDefaultCC() const;
 93:   void setCCs();
 94: };
 95: 
 96: class ARMSwiftABIInfo : public SwiftABIInfo {
 97: public:
 98:   explicit ARMSwiftABIInfo(CodeGenTypes &CGT)
 99:       : SwiftABIInfo(CGT, /*SwiftErrorInRegister=*/true) {}
100: 
```
- **EN**: This block introduces declarations such as `ARMSwiftABIInfo`; defines callable entry points like `isZeroLengthBitfieldPermittedInHomogeneousAggregate`, `isEffectivelyAAPCS_VFP`, `computeInfo`, `EmitVAArg`, `getLLVMDefaultCC`.
- **CN**: 该代码块给出诸如 `ARMSwiftABIInfo` 的声明；定义可调用入口，例如 `isZeroLengthBitfieldPermittedInHomogeneousAggregate`, `isEffectivelyAAPCS_VFP`, `computeInfo`, `EmitVAArg`, `getLLVMDefaultCC`。

### Lines 101-120
```cpp
101:   bool isLegalVectorType(CharUnits VectorSize, llvm::Type *EltTy,
102:                          unsigned NumElts) const override;
103: };
104: 
105: class ARMTargetCodeGenInfo : public TargetCodeGenInfo {
106: public:
107:   ARMTargetCodeGenInfo(CodeGenTypes &CGT, ARMABIKind K)
108:       : TargetCodeGenInfo(std::make_unique<ARMABIInfo>(CGT, K)) {
109:     SwiftInfo = std::make_unique<ARMSwiftABIInfo>(CGT);
110:   }
111: 
112:   int getDwarfEHStackPointer(CodeGen::CodeGenModule &M) const override {
113:     return 13;
114:   }
115: 
116:   StringRef getARCRetainAutoreleasedReturnValueMarker() const override {
117:     return "mov\tr7, r7\t\t// marker for objc_retainAutoreleaseReturnValue";
118:   }
119: 
120:   bool initDwarfEHRegSizeTable(CodeGen::CodeGenFunction &CGF,
```
- **EN**: This block introduces declarations such as `ARMTargetCodeGenInfo`; defines callable entry points like `isLegalVectorType`, `ARMTargetCodeGenInfo`, `getDwarfEHStackPointer`, `getARCRetainAutoreleasedReturnValueMarker`; uses control flow (for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出诸如 `ARMTargetCodeGenInfo` 的声明；定义可调用入口，例如 `isLegalVectorType`, `ARMTargetCodeGenInfo`, `getDwarfEHStackPointer`, `getARCRetainAutoreleasedReturnValueMarker`；通过控制流（for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 121-140
```cpp
121:                                llvm::Value *Address) const override {
122:     llvm::Value *Four8 = llvm::ConstantInt::get(CGF.Int8Ty, 4);
123: 
124:     // 0-15 are the 16 integer registers.
125:     AssignToArrayRange(CGF.Builder, Address, Four8, 0, 15);
126:     return false;
127:   }
128: 
129:   unsigned getSizeOfUnwindException() const override {
130:     if (getABIInfo<ARMABIInfo>().isEABI())
131:       return 88;
132:     return TargetCodeGenInfo::getSizeOfUnwindException();
133:   }
134: 
135:   void setTargetAttributes(const Decl *D, llvm::GlobalValue *GV,
136:                            CodeGen::CodeGenModule &CGM) const override {
137:     auto *Fn = dyn_cast<llvm::Function>(GV);
138:     if (!Fn)
139:       return;
140:     const auto *FD = dyn_cast_or_null<FunctionDecl>(D);
```
- **EN**: This block defines callable entry points like `AssignToArrayRange`, `getSizeOfUnwindException`, `setTargetAttributes`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `AssignToArrayRange`, `getSizeOfUnwindException`, `setTargetAttributes`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 141-160
```cpp
141: 
142:     if (FD && FD->hasAttr<TargetAttr>()) {
143:       const auto *TA = FD->getAttr<TargetAttr>();
144:       ParsedTargetAttr Attr =
145:           CGM.getTarget().parseTargetAttr(TA->getFeaturesStr());
146:       if (!Attr.BranchProtection.empty()) {
147:         TargetInfo::BranchProtectionInfo BPI{};
148:         StringRef DiagMsg;
149:         StringRef Arch =
150:             Attr.CPU.empty() ? CGM.getTarget().getTargetOpts().CPU : Attr.CPU;
151:         if (!CGM.getTarget().validateBranchProtection(
152:                 Attr.BranchProtection, Arch, BPI, CGM.getLangOpts(), DiagMsg)) {
153:           CGM.getDiags().Report(
154:               D->getLocation(),
155:               diag::warn_target_unsupported_branch_protection_attribute)
156:               << Arch;
157:         } else
158:           setBranchProtectionFnAttributes(BPI, (*Fn));
159:       } else if (CGM.getLangOpts().BranchTargetEnforcement ||
160:                  CGM.getLangOpts().hasSignReturnAddress()) {
```
- **EN**: This block defines callable entry points like `setBranchProtectionFnAttributes`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `setBranchProtectionFnAttributes`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 161-180
```cpp
161:         // If the Branch Protection attribute is missing, validate the target
162:         // Architecture attribute against Branch Protection command line
163:         // settings.
164:         if (!CGM.getTarget().isBranchProtectionSupportedArch(Attr.CPU))
165:           CGM.getDiags().Report(
166:               D->getLocation(),
167:               diag::warn_target_unsupported_branch_protection_attribute)
168:               << Attr.CPU;
169:       }
170:     } else if (CGM.getTarget().isBranchProtectionSupportedArch(
171:                    CGM.getTarget().getTargetOpts().CPU)) {
172:       TargetInfo::BranchProtectionInfo BPI(CGM.getLangOpts());
173:       setBranchProtectionFnAttributes(BPI, (*Fn));
174:     }
175: 
176:     if (!FD || !FD->hasAttr<ARMInterruptAttr>())
177:       return;
178: 
179:     const ARMInterruptAttr *Attr = FD->getAttr<ARMInterruptAttr>();
180:     const char *Kind;
```
- **EN**: This block defines callable entry points like `BPI`, `setBranchProtectionFnAttributes`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `BPI`, `setBranchProtectionFnAttributes`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 181-200
```cpp
181:     switch (Attr->getInterrupt()) {
182:     case ARMInterruptAttr::Generic: Kind = ""; break;
183:     case ARMInterruptAttr::IRQ:     Kind = "IRQ"; break;
184:     case ARMInterruptAttr::FIQ:     Kind = "FIQ"; break;
185:     case ARMInterruptAttr::SWI:     Kind = "SWI"; break;
186:     case ARMInterruptAttr::ABORT:   Kind = "ABORT"; break;
187:     case ARMInterruptAttr::UNDEF:   Kind = "UNDEF"; break;
188:     }
189: 
190:     Fn->addFnAttr("interrupt", Kind);
191: 
192:     // Note: the ARMSaveFPAttr can only exist if we also have an interrupt
193:     // attribute
194:     const ARMSaveFPAttr *SaveFPAttr = FD->getAttr<ARMSaveFPAttr>();
195:     if (SaveFPAttr)
196:       Fn->addFnAttr("save-fp");
197: 
198:     ARMABIKind ABI = getABIInfo<ARMABIInfo>().getABIKind();
199:     if (ABI == ARMABIKind::APCS)
200:       return;
```
- **EN**: This block uses control flow (if, switch, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 201-220
```cpp
201: 
202:     // AAPCS guarantees that sp will be 8-byte aligned on any public interface,
203:     // however this is not necessarily true on taking any interrupt. Instruct
204:     // the backend to perform a realignment as part of the function prologue.
205:     llvm::AttrBuilder B(Fn->getContext());
206:     B.addStackAlignmentAttr(8);
207:     Fn->addFnAttrs(B);
208:   }
209: };
210: 
211: class WindowsARMTargetCodeGenInfo : public ARMTargetCodeGenInfo {
212: public:
213:   WindowsARMTargetCodeGenInfo(CodeGenTypes &CGT, ARMABIKind K)
214:       : ARMTargetCodeGenInfo(CGT, K) {}
215: 
216:   void setTargetAttributes(const Decl *D, llvm::GlobalValue *GV,
217:                            CodeGen::CodeGenModule &CGM) const override;
218: 
219:   void getDependentLibraryOption(llvm::StringRef Lib,
220:                                  llvm::SmallString<24> &Opt) const override {
```
- **EN**: This block introduces declarations such as `WindowsARMTargetCodeGenInfo`; defines callable entry points like `B`, `WindowsARMTargetCodeGenInfo`, `setTargetAttributes`, `getDependentLibraryOption`.
- **CN**: 该代码块给出诸如 `WindowsARMTargetCodeGenInfo` 的声明；定义可调用入口，例如 `B`, `WindowsARMTargetCodeGenInfo`, `setTargetAttributes`, `getDependentLibraryOption`。

### Lines 221-240
```cpp
221:     Opt = "/DEFAULTLIB:" + qualifyWindowsLibrary(Lib);
222:   }
223: 
224:   void getDetectMismatchOption(llvm::StringRef Name, llvm::StringRef Value,
225:                                llvm::SmallString<32> &Opt) const override {
226:     Opt = "/FAILIFMISMATCH:\"" + Name.str() + "=" + Value.str() + "\"";
227:   }
228: };
229: 
230: void WindowsARMTargetCodeGenInfo::setTargetAttributes(
231:     const Decl *D, llvm::GlobalValue *GV, CodeGen::CodeGenModule &CGM) const {
232:   ARMTargetCodeGenInfo::setTargetAttributes(D, GV, CGM);
233:   if (GV->isDeclaration())
234:     return;
235:   addStackProbeTargetAttributes(D, GV, CGM);
236: }
237: }
238: 
239: void ARMABIInfo::computeInfo(CGFunctionInfo &FI) const {
240:   if (!::classifyReturnType(getCXXABI(), FI, *this))
```
- **EN**: This block defines callable entry points like `getDetectMismatchOption`, `setTargetAttributes`, `addStackProbeTargetAttributes`, `computeInfo`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDetectMismatchOption`, `setTargetAttributes`, `addStackProbeTargetAttributes`, `computeInfo`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 241-260
```cpp
241:     FI.getReturnInfo() = classifyReturnType(FI.getReturnType(), FI.isVariadic(),
242:                                             FI.getCallingConvention());
243: 
244:   for (auto &I : FI.arguments())
245:     I.info = classifyArgumentType(I.type, FI.isVariadic(),
246:                                   FI.getCallingConvention());
247: 
248: 
249:   // Always honor user-specified calling convention.
250:   if (FI.getCallingConvention() != llvm::CallingConv::C)
251:     return;
252: 
253:   llvm::CallingConv::ID cc = getRuntimeCC();
254:   if (cc != llvm::CallingConv::C)
255:     FI.setEffectiveCallingConvention(cc);
256: }
257: 
258: /// Return the default calling convention that LLVM will use.
259: llvm::CallingConv::ID ARMABIInfo::getLLVMDefaultCC() const {
260:   // The default calling convention that LLVM will infer.
```
- **EN**: This block defines callable entry points like `getLLVMDefaultCC`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getLLVMDefaultCC`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 261-280
```cpp
261:   if (isEABIHF() || getTarget().getTriple().isWatchABI())
262:     return llvm::CallingConv::ARM_AAPCS_VFP;
263:   else if (isEABI())
264:     return llvm::CallingConv::ARM_AAPCS;
265:   else
266:     return llvm::CallingConv::ARM_APCS;
267: }
268: 
269: /// Return the calling convention that our ABI would like us to use
270: /// as the C calling convention.
271: llvm::CallingConv::ID ARMABIInfo::getABIDefaultCC() const {
272:   switch (getABIKind()) {
273:   case ARMABIKind::APCS:
274:     return llvm::CallingConv::ARM_APCS;
275:   case ARMABIKind::AAPCS:
276:     return llvm::CallingConv::ARM_AAPCS;
277:   case ARMABIKind::AAPCS_VFP:
278:     return llvm::CallingConv::ARM_AAPCS_VFP;
279:   case ARMABIKind::AAPCS16_VFP:
280:     return llvm::CallingConv::ARM_AAPCS_VFP;
```
- **EN**: This block defines callable entry points like `getABIDefaultCC`; uses control flow (if, switch, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getABIDefaultCC`；通过控制流（if, switch, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 281-300
```cpp
281:   }
282:   llvm_unreachable("bad ABI kind");
283: }
284: 
285: void ARMABIInfo::setCCs() {
286:   assert(getRuntimeCC() == llvm::CallingConv::C);
287: 
288:   // Don't muddy up the IR with a ton of explicit annotations if
289:   // they'd just match what LLVM will infer from the triple.
290:   llvm::CallingConv::ID abiCC = getABIDefaultCC();
291:   if (abiCC != getLLVMDefaultCC())
292:     RuntimeCC = abiCC;
293: }
294: 
295: ABIArgInfo ARMABIInfo::coerceIllegalVector(QualType Ty) const {
296:   uint64_t Size = getContext().getTypeSize(Ty);
297:   if (Size <= 32) {
298:     llvm::Type *ResType =
299:         llvm::Type::getInt32Ty(getVMContext());
300:     return ABIArgInfo::getDirect(ResType);
```
- **EN**: This block defines callable entry points like `setCCs`, `coerceIllegalVector`, `getInt32Ty`, `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `setCCs`, `coerceIllegalVector`, `getInt32Ty`, `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 301-320
```cpp
301:   }
302:   if (Size == 64 || Size == 128) {
303:     auto *ResType = llvm::FixedVectorType::get(
304:         llvm::Type::getInt32Ty(getVMContext()), Size / 32);
305:     return ABIArgInfo::getDirect(ResType);
306:   }
307:   return getNaturalAlignIndirect(
308:       Ty, /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
309:       /*ByVal=*/false);
310: }
311: 
312: ABIArgInfo ARMABIInfo::classifyHomogeneousAggregate(QualType Ty,
313:                                                     const Type *Base,
314:                                                     uint64_t Members) const {
315:   assert(Base && "Base class should be set for homogeneous aggregate");
316:   // Base can be a floating-point or a vector.
317:   if (const VectorType *VT = Base->getAs<VectorType>()) {
318:     // FP16 vectors should be converted to integer vectors
319:     if (!getTarget().hasFastHalfType() && containsAnyFP16Vectors(Ty)) {
320:       uint64_t Size = getContext().getTypeSize(VT);
```
- **EN**: This block introduces declarations such as `should`; defines callable entry points like `getInt32Ty`, `getDirect`, `classifyHomogeneousAggregate`; uses control flow (if, for) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `should` 的声明；定义可调用入口，例如 `getInt32Ty`, `getDirect`, `classifyHomogeneousAggregate`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 321-340
```cpp
321:       auto *NewVecTy = llvm::FixedVectorType::get(
322:           llvm::Type::getInt32Ty(getVMContext()), Size / 32);
323:       llvm::Type *Ty = llvm::ArrayType::get(NewVecTy, Members);
324:       return ABIArgInfo::getDirect(Ty, 0, nullptr, false);
325:     }
326:   }
327:   unsigned Align = 0;
328:   if (getABIKind() == ARMABIKind::AAPCS ||
329:       getABIKind() == ARMABIKind::AAPCS_VFP) {
330:     // For alignment adjusted HFAs, cap the argument alignment to 8, leave it
331:     // default otherwise.
332:     Align = getContext().getTypeUnadjustedAlignInChars(Ty).getQuantity();
333:     unsigned BaseAlign = getContext().getTypeAlignInChars(Base).getQuantity();
334:     Align = (Align > BaseAlign && Align >= 8) ? 8 : 0;
335:   }
336:   return ABIArgInfo::getDirect(nullptr, 0, nullptr, false, Align);
337: }
338: 
339: bool ARMABIInfo::shouldIgnoreEmptyArg(QualType Ty) const {
340:   uint64_t Size = getContext().getTypeSize(Ty);
```
- **EN**: This block defines callable entry points like `getInt32Ty`, `getDirect`, `shouldIgnoreEmptyArg`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getInt32Ty`, `getDirect`, `shouldIgnoreEmptyArg`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 341-360
```cpp
341:   assert((isEmptyRecord(getContext(), Ty, true) || Size == 0) &&
342:          "Arg is not empty");
343: 
344:   // Empty records are ignored in C mode, and in C++ on WatchOS.
345:   if (!getContext().getLangOpts().CPlusPlus ||
346:       getABIKind() == ARMABIKind::AAPCS16_VFP)
347:     return true;
348: 
349:   // In C++ mode, arguments which have sizeof() == 0 are ignored. This is not a
350:   // situation which is defined by any C++ standard or ABI, but this matches
351:   // GCC's de facto ABI.
352:   if (Size == 0)
353:     return true;
354: 
355:   // Clang 19.0 and earlier always ignored empty struct arguments in C++ mode.
356:   if (getContext().getLangOpts().getClangABICompat() <=
357:       LangOptions::ClangABI::Ver19)
358:     return true;
359: 
360:   // Otherwise, they are passed as if they have a size of 1 byte.
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 361-380
```cpp
361:   return false;
362: }
363: 
364: ABIArgInfo ARMABIInfo::classifyArgumentType(QualType Ty, bool isVariadic,
365:                                             unsigned functionCallConv) const {
366:   // 6.1.2.1 The following argument types are VFP CPRCs:
367:   //   A single-precision floating-point type (including promoted
368:   //   half-precision types); A double-precision floating-point type;
369:   //   A 64-bit or 128-bit containerized vector type; Homogeneous Aggregate
370:   //   with a Base Type of a single- or double-precision floating-point type,
371:   //   64-bit containerized vectors or 128-bit containerized vectors with one
372:   //   to four Elements.
373:   // Variadic functions should always marshal to the base standard.
374:   bool IsAAPCS_VFP =
375:       !isVariadic && isEffectivelyAAPCS_VFP(functionCallConv, /* AAPCS16 */ false);
376: 
377:   Ty = useFirstFieldIfTransparentUnion(Ty);
378: 
379:   // Handle illegal vector types here.
380:   if (isIllegalVectorType(Ty))
```
- **EN**: This block defines callable entry points like `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 381-400
```cpp
381:     return coerceIllegalVector(Ty);
382: 
383:   if (!isAggregateTypeForABI(Ty)) {
384:     // Treat an enum type as its underlying type.
385:     if (const auto *ED = Ty->getAsEnumDecl()) {
386:       Ty = ED->getIntegerType();
387:     }
388: 
389:     if (const auto *EIT = Ty->getAs<BitIntType>())
390:       if (EIT->getNumBits() > 64)
391:         return getNaturalAlignIndirect(
392:             Ty, /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
393:             /*ByVal=*/true);
394: 
395:     return (isPromotableIntegerTypeForABI(Ty)
396:                 ? ABIArgInfo::getExtend(Ty, CGT.ConvertType(Ty))
397:                 : ABIArgInfo::getDirect());
398:   }
399: 
400:   if (CGCXXABI::RecordArgABI RAA = getRecordArgABI(Ty, getCXXABI())) {
```
- **EN**: This block defines callable entry points like `coerceIllegalVector`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `coerceIllegalVector`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 401-420
```cpp
401:     return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
402:                                    RAA == CGCXXABI::RAA_DirectInMemory);
403:   }
404: 
405:   // Empty records are either ignored completely or passed as if they were a
406:   // 1-byte object, depending on the ABI and language standard.
407:   if (isEmptyRecord(getContext(), Ty, true) ||
408:       getContext().getTypeSize(Ty) == 0) {
409:     if (shouldIgnoreEmptyArg(Ty))
410:       return ABIArgInfo::getIgnore();
411:     else
412:       return ABIArgInfo::getDirect(llvm::Type::getInt8Ty(getVMContext()));
413:   }
414: 
415:   if (IsAAPCS_VFP) {
416:     // Homogeneous Aggregates need to be expanded when we can fit the aggregate
417:     // into VFP registers.
418:     const Type *Base = nullptr;
419:     uint64_t Members = 0;
420:     if (isHomogeneousAggregate(Ty, Base, Members))
```
- **EN**: This block defines callable entry points like `getNaturalAlignIndirect`, `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getNaturalAlignIndirect`, `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 421-440
```cpp
421:       return classifyHomogeneousAggregate(Ty, Base, Members);
422:   } else if (getABIKind() == ARMABIKind::AAPCS16_VFP) {
423:     // WatchOS does have homogeneous aggregates. Note that we intentionally use
424:     // this convention even for a variadic function: the backend will use GPRs
425:     // if needed.
426:     const Type *Base = nullptr;
427:     uint64_t Members = 0;
428:     if (isHomogeneousAggregate(Ty, Base, Members)) {
429:       assert(Base && Members <= 4 && "unexpected homogeneous aggregate");
430:       llvm::Type *Ty =
431:         llvm::ArrayType::get(CGT.ConvertType(QualType(Base, 0)), Members);
432:       return ABIArgInfo::getDirect(Ty, 0, nullptr, false);
433:     }
434:   }
435: 
436:   if (getABIKind() == ARMABIKind::AAPCS16_VFP &&
437:       getContext().getTypeSizeInChars(Ty) > CharUnits::fromQuantity(16)) {
438:     // WatchOS is adopting the 64-bit AAPCS rule on composite types: if they're
439:     // bigger than 128-bits, they get placed in space allocated by the caller,
440:     // and a pointer is passed.
```
- **EN**: This block defines callable entry points like `classifyHomogeneousAggregate`, `get`, `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `classifyHomogeneousAggregate`, `get`, `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 441-460
```cpp
441:     return ABIArgInfo::getIndirect(
442:         CharUnits::fromQuantity(getContext().getTypeAlign(Ty) / 8),
443:         getDataLayout().getAllocaAddrSpace(), false);
444:   }
445: 
446:   // Support byval for ARM.
447:   // The ABI alignment for APCS is 4-byte and for AAPCS at least 4-byte and at
448:   // most 8-byte. We realign the indirect argument if type alignment is bigger
449:   // than ABI alignment.
450:   uint64_t ABIAlign = 4;
451:   uint64_t TyAlign;
452:   if (getABIKind() == ARMABIKind::AAPCS_VFP ||
453:       getABIKind() == ARMABIKind::AAPCS) {
454:     TyAlign = getContext().getTypeUnadjustedAlignInChars(Ty).getQuantity();
455:     ABIAlign = std::clamp(TyAlign, (uint64_t)4, (uint64_t)8);
456:   } else {
457:     TyAlign = getContext().getTypeAlignInChars(Ty).getQuantity();
458:   }
459:   if (getContext().getTypeSizeInChars(Ty) > CharUnits::fromQuantity(64)) {
460:     assert(getABIKind() != ARMABIKind::AAPCS16_VFP && "unexpected byval");
```
- **EN**: This block defines callable entry points like `getIndirect`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getIndirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 461-480
```cpp
461:     return ABIArgInfo::getIndirect(
462:         CharUnits::fromQuantity(ABIAlign),
463:         /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
464:         /*ByVal=*/true, /*Realign=*/TyAlign > ABIAlign);
465:   }
466: 
467:   // Otherwise, pass by coercing to a structure of the appropriate size.
468:   llvm::Type* ElemTy;
469:   unsigned SizeRegs;
470:   // FIXME: Try to match the types of the arguments more accurately where
471:   // we can.
472:   if (TyAlign <= 4) {
473:     ElemTy = llvm::Type::getInt32Ty(getVMContext());
474:     SizeRegs = (getContext().getTypeSize(Ty) + 31) / 32;
475:   } else {
476:     ElemTy = llvm::Type::getInt64Ty(getVMContext());
477:     SizeRegs = (getContext().getTypeSize(Ty) + 63) / 64;
478:   }
479: 
480:   return ABIArgInfo::getDirect(llvm::ArrayType::get(ElemTy, SizeRegs));
```
- **EN**: This block defines callable entry points like `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 481-500
```cpp
481: }
482: 
483: static bool isIntegerLikeType(QualType Ty, ASTContext &Context,
484:                               llvm::LLVMContext &VMContext) {
485:   // APCS, C Language Calling Conventions, Non-Simple Return Values: A structure
486:   // is called integer-like if its size is less than or equal to one word, and
487:   // the offset of each of its addressable sub-fields is zero.
488: 
489:   uint64_t Size = Context.getTypeSize(Ty);
490: 
491:   // Check that the type fits in a word.
492:   if (Size > 32)
493:     return false;
494: 
495:   // FIXME: Handle vector types!
496:   if (Ty->isVectorType())
497:     return false;
498: 
499:   // Float types are never treated as "integer like".
500:   if (Ty->isRealFloatingType())
```
- **EN**: This block defines callable entry points like `isIntegerLikeType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isIntegerLikeType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 501-520
```cpp
501:     return false;
502: 
503:   // If this is a builtin or pointer type then it is ok.
504:   if (Ty->getAs<BuiltinType>() || Ty->isPointerType())
505:     return true;
506: 
507:   // Small complex integer types are "integer like".
508:   if (const ComplexType *CT = Ty->getAs<ComplexType>())
509:     return isIntegerLikeType(CT->getElementType(), Context, VMContext);
510: 
511:   // Single element and zero sized arrays should be allowed, by the definition
512:   // above, but they are not.
513: 
514:   // Otherwise, it must be a record type.
515:   const RecordType *RT = Ty->getAsCanonical<RecordType>();
516:   if (!RT) return false;
517: 
518:   // Ignore records with flexible arrays.
519:   const RecordDecl *RD = RT->getDecl()->getDefinitionOrSelf();
520:   if (RD->hasFlexibleArrayMember())
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 521-540
```cpp
521:     return false;
522: 
523:   // Check that all sub-fields are at offset 0, and are themselves "integer
524:   // like".
525:   const ASTRecordLayout &Layout = Context.getASTRecordLayout(RD);
526: 
527:   bool HadField = false;
528:   unsigned idx = 0;
529:   for (RecordDecl::field_iterator i = RD->field_begin(), e = RD->field_end();
530:        i != e; ++i, ++idx) {
531:     const FieldDecl *FD = *i;
532: 
533:     // Bit-fields are not addressable, we only need to verify they are "integer
534:     // like". We still have to disallow a subsequent non-bitfield, for example:
535:     //   struct { int : 0; int x }
536:     // is non-integer like according to gcc.
537:     if (FD->isBitField()) {
538:       if (!RD->isUnion())
539:         HadField = true;
540: 
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 541-560
```cpp
541:       if (!isIntegerLikeType(FD->getType(), Context, VMContext))
542:         return false;
543: 
544:       continue;
545:     }
546: 
547:     // Check if this field is at offset 0.
548:     if (Layout.getFieldOffset(idx) != 0)
549:       return false;
550: 
551:     if (!isIntegerLikeType(FD->getType(), Context, VMContext))
552:       return false;
553: 
554:     // Only allow at most one field in a structure. This doesn't match the
555:     // wording above, but follows gcc in situations with a field following an
556:     // empty structure.
557:     if (!RD->isUnion()) {
558:       if (HadField)
559:         return false;
560: 
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 561-580
```cpp
561:       HadField = true;
562:     }
563:   }
564: 
565:   return true;
566: }
567: 
568: ABIArgInfo ARMABIInfo::classifyReturnType(QualType RetTy, bool isVariadic,
569:                                           unsigned functionCallConv) const {
570: 
571:   // Variadic functions should always marshal to the base standard.
572:   bool IsAAPCS_VFP =
573:       !isVariadic && isEffectivelyAAPCS_VFP(functionCallConv, /* AAPCS16 */ true);
574: 
575:   if (RetTy->isVoidType())
576:     return ABIArgInfo::getIgnore();
577: 
578:   if (const VectorType *VT = RetTy->getAs<VectorType>()) {
579:     // Large vector types should be returned via memory.
580:     if (getContext().getTypeSize(RetTy) > 128)
```
- **EN**: This block defines callable entry points like `classifyReturnType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyReturnType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 581-600
```cpp
581:       return getNaturalAlignIndirect(RetTy,
582:                                      getDataLayout().getAllocaAddrSpace());
583:     // TODO: FP16/BF16 vectors should be converted to integer vectors
584:     // This check is similar  to isIllegalVectorType - refactor?
585:     if ((!getTarget().hasFastHalfType() &&
586:         (VT->getElementType()->isFloat16Type() ||
587:          VT->getElementType()->isHalfType())) ||
588:         (IsFloatABISoftFP &&
589:          VT->getElementType()->isBFloat16Type()))
590:       return coerceIllegalVector(RetTy);
591:   }
592: 
593:   if (!isAggregateTypeForABI(RetTy)) {
594:     // Treat an enum type as its underlying type.
595:     if (const auto *ED = RetTy->getAsEnumDecl())
596:       RetTy = ED->getIntegerType();
597: 
598:     if (const auto *EIT = RetTy->getAs<BitIntType>())
599:       if (EIT->getNumBits() > 64)
600:         return getNaturalAlignIndirect(
```
- **EN**: This block defines callable entry points like `getNaturalAlignIndirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getNaturalAlignIndirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 601-620
```cpp
601:             RetTy, /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
602:             /*ByVal=*/false);
603: 
604:     return isPromotableIntegerTypeForABI(RetTy) ? ABIArgInfo::getExtend(RetTy)
605:                                                 : ABIArgInfo::getDirect();
606:   }
607: 
608:   // Are we following APCS?
609:   if (getABIKind() == ARMABIKind::APCS) {
610:     if (isEmptyRecord(getContext(), RetTy, false))
611:       return ABIArgInfo::getIgnore();
612: 
613:     // Complex types are all returned as packed integers.
614:     //
615:     // FIXME: Consider using 2 x vector types if the back end handles them
616:     // correctly.
617:     if (RetTy->isAnyComplexType())
618:       return ABIArgInfo::getDirect(llvm::IntegerType::get(
619:           getVMContext(), getContext().getTypeSize(RetTy)));
620: 
```
- **EN**: This block defines callable entry points like `isPromotableIntegerTypeForABI`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isPromotableIntegerTypeForABI`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 621-640
```cpp
621:     // Integer like structures are returned in r0.
622:     if (isIntegerLikeType(RetTy, getContext(), getVMContext())) {
623:       // Return in the smallest viable integer type.
624:       uint64_t Size = getContext().getTypeSize(RetTy);
625:       if (Size <= 8)
626:         return ABIArgInfo::getDirect(llvm::Type::getInt8Ty(getVMContext()));
627:       if (Size <= 16)
628:         return ABIArgInfo::getDirect(llvm::Type::getInt16Ty(getVMContext()));
629:       return ABIArgInfo::getDirect(llvm::Type::getInt32Ty(getVMContext()));
630:     }
631: 
632:     // Otherwise return in memory.
633:     return getNaturalAlignIndirect(RetTy, getDataLayout().getAllocaAddrSpace());
634:   }
635: 
636:   // Otherwise this is an AAPCS variant.
637: 
638:   if (isEmptyRecord(getContext(), RetTy, true) ||
639:       getContext().getTypeSize(RetTy) == 0)
640:     return ABIArgInfo::getIgnore();
```
- **EN**: This block defines callable entry points like `getDirect`, `getNaturalAlignIndirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `getNaturalAlignIndirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 641-660
```cpp
641: 
642:   // Check for homogeneous aggregates with AAPCS-VFP.
643:   if (IsAAPCS_VFP) {
644:     const Type *Base = nullptr;
645:     uint64_t Members = 0;
646:     if (isHomogeneousAggregate(RetTy, Base, Members))
647:       return classifyHomogeneousAggregate(RetTy, Base, Members);
648:   }
649: 
650:   // Aggregates <= 4 bytes are returned in r0; other aggregates
651:   // are returned indirectly.
652:   uint64_t Size = getContext().getTypeSize(RetTy);
653:   if (Size <= 32) {
654:     if (getDataLayout().isBigEndian())
655:       // Return in 32 bit integer integer type (as if loaded by LDR, AAPCS 5.4)
656:       return ABIArgInfo::getDirect(llvm::Type::getInt32Ty(getVMContext()));
657: 
658:     // Return in the smallest viable integer type.
659:     if (Size <= 8)
660:       return ABIArgInfo::getDirect(llvm::Type::getInt8Ty(getVMContext()));
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 661-680
```cpp
661:     if (Size <= 16)
662:       return ABIArgInfo::getDirect(llvm::Type::getInt16Ty(getVMContext()));
663:     return ABIArgInfo::getDirect(llvm::Type::getInt32Ty(getVMContext()));
664:   } else if (Size <= 128 && getABIKind() == ARMABIKind::AAPCS16_VFP) {
665:     llvm::Type *Int32Ty = llvm::Type::getInt32Ty(getVMContext());
666:     llvm::Type *CoerceTy =
667:         llvm::ArrayType::get(Int32Ty, llvm::alignTo(Size, 32) / 32);
668:     return ABIArgInfo::getDirect(CoerceTy);
669:   }
670: 
671:   return getNaturalAlignIndirect(RetTy, getDataLayout().getAllocaAddrSpace());
672: }
673: 
674: /// isIllegalVector - check whether Ty is an illegal vector type.
675: bool ARMABIInfo::isIllegalVectorType(QualType Ty) const {
676:   if (const VectorType *VT = Ty->getAs<VectorType> ()) {
677:     // On targets that don't support half, fp16 or bfloat, they are expanded
678:     // into float, and we don't want the ABI to depend on whether or not they
679:     // are supported in hardware. Thus return false to coerce vectors of these
680:     // types into integer vectors.
```
- **EN**: This block defines callable entry points like `getDirect`, `get`, `getNaturalAlignIndirect`, `isIllegalVectorType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `get`, `getNaturalAlignIndirect`, `isIllegalVectorType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 681-700
```cpp
681:     // We do not depend on hasFastHalfType for bfloat as it is a
682:     // separate IR type.
683:     if ((!getTarget().hasFastHalfType() &&
684:         (VT->getElementType()->isFloat16Type() ||
685:          VT->getElementType()->isHalfType())) ||
686:         (IsFloatABISoftFP &&
687:          VT->getElementType()->isBFloat16Type()))
688:       return true;
689:     if (isAndroid()) {
690:       // Android shipped using Clang 3.1, which supported a slightly different
691:       // vector ABI. The primary differences were that 3-element vector types
692:       // were legal, and so were sub 32-bit vectors (i.e. <2 x i8>). This path
693:       // accepts that legacy behavior for Android only.
694:       // Check whether VT is legal.
695:       unsigned NumElements = VT->getNumElements();
696:       // NumElements should be power of 2 or equal to 3.
697:       if (!llvm::isPowerOf2_32(NumElements) && NumElements != 3)
698:         return true;
699:     } else {
700:       // Check whether VT is legal.
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 701-720
```cpp
701:       unsigned NumElements = VT->getNumElements();
702:       uint64_t Size = getContext().getTypeSize(VT);
703:       // NumElements should be power of 2.
704:       if (!llvm::isPowerOf2_32(NumElements))
705:         return true;
706:       // Size should be greater than 32 bits.
707:       return Size <= 32;
708:     }
709:   }
710:   return false;
711: }
712: 
713: /// Return true if a type contains any 16-bit floating point vectors
714: bool ARMABIInfo::containsAnyFP16Vectors(QualType Ty) const {
715:   if (const ConstantArrayType *AT = getContext().getAsConstantArrayType(Ty)) {
716:     uint64_t NElements = AT->getZExtSize();
717:     if (NElements == 0)
718:       return false;
719:     return containsAnyFP16Vectors(AT->getElementType());
720:   }
```
- **EN**: This block defines callable entry points like `containsAnyFP16Vectors`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `containsAnyFP16Vectors`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 721-740
```cpp
721:   if (const auto *RD = Ty->getAsRecordDecl()) {
722:     // If this is a C++ record, check the bases first.
723:     if (const CXXRecordDecl *CXXRD = dyn_cast<CXXRecordDecl>(RD))
724:       if (llvm::any_of(CXXRD->bases(), [this](const CXXBaseSpecifier &B) {
725:             return containsAnyFP16Vectors(B.getType());
726:           }))
727:         return true;
728: 
729:     if (llvm::any_of(RD->fields(), [this](FieldDecl *FD) {
730:           return FD && containsAnyFP16Vectors(FD->getType());
731:         }))
732:       return true;
733: 
734:     return false;
735:   } else {
736:     if (const VectorType *VT = Ty->getAs<VectorType>())
737:       return (VT->getElementType()->isFloat16Type() ||
738:               VT->getElementType()->isBFloat16Type() ||
739:               VT->getElementType()->isHalfType());
740:     return false;
```
- **EN**: This block defines callable entry points like `containsAnyFP16Vectors`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `containsAnyFP16Vectors`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 741-760
```cpp
741:   }
742: }
743: 
744: bool ARMSwiftABIInfo::isLegalVectorType(CharUnits VectorSize, llvm::Type *EltTy,
745:                                         unsigned NumElts) const {
746:   if (!llvm::isPowerOf2_32(NumElts))
747:     return false;
748:   unsigned size = CGT.getDataLayout().getTypeStoreSizeInBits(EltTy);
749:   if (size > 64)
750:     return false;
751:   if (VectorSize.getQuantity() != 8 &&
752:       (VectorSize.getQuantity() != 16 || NumElts == 1))
753:     return false;
754:   return true;
755: }
756: 
757: bool ARMABIInfo::isHomogeneousAggregateBaseType(QualType Ty) const {
758:   // Homogeneous aggregates for AAPCS-VFP must have base types of float,
759:   // double, or 64-bit or 128-bit vectors.
760:   if (const BuiltinType *BT = Ty->getAs<BuiltinType>()) {
```
- **EN**: This block defines callable entry points like `isLegalVectorType`, `isHomogeneousAggregateBaseType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isLegalVectorType`, `isHomogeneousAggregateBaseType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 761-780
```cpp
761:     if (BT->getKind() == BuiltinType::Float ||
762:         BT->getKind() == BuiltinType::Double ||
763:         BT->getKind() == BuiltinType::LongDouble)
764:       return true;
765:   } else if (const VectorType *VT = Ty->getAs<VectorType>()) {
766:     unsigned VecSize = getContext().getTypeSize(VT);
767:     if (VecSize == 64 || VecSize == 128)
768:       return true;
769:   }
770:   return false;
771: }
772: 
773: bool ARMABIInfo::isHomogeneousAggregateSmallEnough(const Type *Base,
774:                                                    uint64_t Members) const {
775:   return Members <= 4;
776: }
777: 
778: bool ARMABIInfo::isZeroLengthBitfieldPermittedInHomogeneousAggregate() const {
779:   // AAPCS32 says that the rule for whether something is a homogeneous
780:   // aggregate is applied to the output of the data layout decision. So
```
- **EN**: This block defines callable entry points like `isHomogeneousAggregateSmallEnough`, `isZeroLengthBitfieldPermittedInHomogeneousAggregate`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isHomogeneousAggregateSmallEnough`, `isZeroLengthBitfieldPermittedInHomogeneousAggregate`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 781-800
```cpp
781:   // anything that doesn't affect the data layout also does not affect
782:   // homogeneity. In particular, zero-length bitfields don't stop a struct
783:   // being homogeneous.
784:   return true;
785: }
786: 
787: bool ARMABIInfo::isEffectivelyAAPCS_VFP(unsigned callConvention,
788:                                         bool acceptHalf) const {
789:   // Give precedence to user-specified calling conventions.
790:   if (callConvention != llvm::CallingConv::C)
791:     return (callConvention == llvm::CallingConv::ARM_AAPCS_VFP);
792:   else
793:     return (getABIKind() == ARMABIKind::AAPCS_VFP) ||
794:            (acceptHalf && (getABIKind() == ARMABIKind::AAPCS16_VFP));
795: }
796: 
797: RValue ARMABIInfo::EmitVAArg(CodeGenFunction &CGF, Address VAListAddr,
798:                              QualType Ty, AggValueSlot Slot) const {
799:   CharUnits SlotSize = CharUnits::fromQuantity(4);
800: 
```
- **EN**: This block defines callable entry points like `isEffectivelyAAPCS_VFP`, `EmitVAArg`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isEffectivelyAAPCS_VFP`, `EmitVAArg`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 801-820
```cpp
801:   // Empty records are ignored for parameter passing purposes.
802:   uint64_t Size = getContext().getTypeSize(Ty);
803:   bool IsEmpty = isEmptyRecord(getContext(), Ty, true);
804:   if ((IsEmpty || Size == 0) && shouldIgnoreEmptyArg(Ty))
805:     return Slot.asRValue();
806: 
807:   CharUnits TySize = getContext().getTypeSizeInChars(Ty);
808:   CharUnits TyAlignForABI = getContext().getTypeUnadjustedAlignInChars(Ty);
809: 
810:   // Use indirect if size of the illegal vector is bigger than 16 bytes.
811:   bool IsIndirect = false;
812:   const Type *Base = nullptr;
813:   uint64_t Members = 0;
814:   if (TySize > CharUnits::fromQuantity(16) && isIllegalVectorType(Ty)) {
815:     IsIndirect = true;
816: 
817:   // ARMv7k passes structs bigger than 16 bytes indirectly, in space
818:   // allocated by the caller.
819:   } else if (TySize > CharUnits::fromQuantity(16) &&
820:              getABIKind() == ARMABIKind::AAPCS16_VFP &&
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 821-840
```cpp
821:              !isHomogeneousAggregate(Ty, Base, Members)) {
822:     IsIndirect = true;
823: 
824:   // Otherwise, bound the type's ABI alignment.
825:   // The ABI alignment for 64-bit or 128-bit vectors is 8 for AAPCS and 4 for
826:   // APCS. For AAPCS, the ABI alignment is at least 4-byte and at most 8-byte.
827:   // Our callers should be prepared to handle an under-aligned address.
828:   } else if (getABIKind() == ARMABIKind::AAPCS_VFP ||
829:              getABIKind() == ARMABIKind::AAPCS) {
830:     TyAlignForABI = std::max(TyAlignForABI, CharUnits::fromQuantity(4));
831:     TyAlignForABI = std::min(TyAlignForABI, CharUnits::fromQuantity(8));
832:   } else if (getABIKind() == ARMABIKind::AAPCS16_VFP) {
833:     // ARMv7k allows type alignment up to 16 bytes.
834:     TyAlignForABI = std::max(TyAlignForABI, CharUnits::fromQuantity(4));
835:     TyAlignForABI = std::min(TyAlignForABI, CharUnits::fromQuantity(16));
836:   } else {
837:     TyAlignForABI = CharUnits::fromQuantity(4);
838:   }
839: 
840:   TypeInfoChars TyInfo(TySize, TyAlignForABI, AlignRequirementKind::None);
```
- **EN**: This block defines callable entry points like `getABIKind`, `TyInfo`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getABIKind`, `TyInfo`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 841-853
```cpp
841:   return emitVoidPtrVAArg(CGF, VAListAddr, Ty, IsIndirect, TyInfo, SlotSize,
842:                           /*AllowHigherAlign*/ true, Slot);
843: }
844: 
845: std::unique_ptr<TargetCodeGenInfo>
846: CodeGen::createARMTargetCodeGenInfo(CodeGenModule &CGM, ARMABIKind Kind) {
847:   return std::make_unique<ARMTargetCodeGenInfo>(CGM.getTypes(), Kind);
848: }
849: 
850: std::unique_ptr<TargetCodeGenInfo>
851: CodeGen::createWindowsARMTargetCodeGenInfo(CodeGenModule &CGM, ARMABIKind K) {
852:   return std::make_unique<WindowsARMTargetCodeGenInfo>(CGM.getTypes(), K);
853: }
```
- **EN**: This block defines callable entry points like `createARMTargetCodeGenInfo`, `createWindowsARMTargetCodeGenInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `createARMTargetCodeGenInfo`, `createWindowsARMTargetCodeGenInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

## Key Concepts / 关键概念

- **ABIArgInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **getContext**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ARMABIKind**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **RetTy**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Size**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ARMABIInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **CGM**: Central symbol in this file's implementation of target-specific ABI and code generation. / 是该文件实现 目标相关的 ABI 与代码生成 时的核心符号。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfoImpl.h`, `TargetInfo.h`
