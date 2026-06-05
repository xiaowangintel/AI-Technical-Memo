# AArch64.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/Targets/AArch64.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements target-specific ABI rules and lowering details for AArch64.
- **Purpose (CN) / 目的（中文）**: 实现 AArch64 目标相关的 ABI 规则与降级细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: //===- AArch64.cpp --------------------------------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: 
 9: #include "ABIInfoImpl.h"
10: #include "TargetInfo.h"
11: #include "clang/AST/Decl.h"
12: #include "clang/Basic/DiagnosticFrontend.h"
13: #include "llvm/TargetParser/AArch64TargetParser.h"
14: 
15: using namespace clang;
16: using namespace clang::CodeGen;
17: 
18: //===----------------------------------------------------------------------===//
19: // AArch64 ABI Implementation
20: //===----------------------------------------------------------------------===//
```
- **EN**: This block imports local CodeGen headers `ABIInfoImpl.h`, `TargetInfo.h`; Clang headers `clang/AST/Decl.h`, `clang/Basic/DiagnosticFrontend.h`; LLVM headers `llvm/TargetParser/AArch64TargetParser.h`; opens or references namespaces `clang`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `ABIInfoImpl.h`, `TargetInfo.h`；Clang 头文件 `clang/AST/Decl.h`, `clang/Basic/DiagnosticFrontend.h`；LLVM 头文件 `llvm/TargetParser/AArch64TargetParser.h`；打开或引用命名空间 `clang`；包含影响本编译单元构建方式的预处理结构。

### Lines 21-40
```cpp
21: 
22: namespace {
23: 
24: class AArch64ABIInfo : public ABIInfo {
25:   AArch64ABIKind Kind;
26: 
27:   std::unique_ptr<TargetCodeGenInfo> WinX86_64CodegenInfo;
28: 
29: public:
30:   AArch64ABIInfo(CodeGenModule &CGM, AArch64ABIKind Kind)
31:       : ABIInfo(CGM.getTypes()), Kind(Kind) {
32:     if (getTarget().getTriple().isWindowsArm64EC()) {
33:       WinX86_64CodegenInfo =
34:           createWinX86_64TargetCodeGenInfo(CGM, X86AVXABILevel::None);
35:     }
36:   }
37: 
38:   bool isSoftFloat() const { return Kind == AArch64ABIKind::AAPCSSoft; }
39: 
40: private:
```
- **EN**: This block introduces declarations such as `AArch64ABIInfo`; defines callable entry points like `AArch64ABIInfo`, `createWinX86_64TargetCodeGenInfo`, `isSoftFloat`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出诸如 `AArch64ABIInfo` 的声明；定义可调用入口，例如 `AArch64ABIInfo`, `createWinX86_64TargetCodeGenInfo`, `isSoftFloat`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 41-60
```cpp
41:   AArch64ABIKind getABIKind() const { return Kind; }
42:   bool isDarwinPCS() const { return Kind == AArch64ABIKind::DarwinPCS; }
43: 
44:   ABIArgInfo classifyReturnType(QualType RetTy, bool IsVariadicFn) const;
45:   ABIArgInfo classifyArgumentType(QualType RetTy, bool IsVariadicFn,
46:                                   bool IsNamedArg, unsigned CallingConvention,
47:                                   unsigned &NSRN, unsigned &NPRN) const;
48:   llvm::Type *convertFixedToScalableVectorType(const VectorType *VT) const;
49:   ABIArgInfo coerceIllegalVector(QualType Ty, unsigned &NSRN,
50:                                  unsigned &NPRN) const;
51:   ABIArgInfo coerceAndExpandPureScalableAggregate(
52:       QualType Ty, bool IsNamedArg, unsigned NVec, unsigned NPred,
53:       const SmallVectorImpl<llvm::Type *> &UnpaddedCoerceToSeq, unsigned &NSRN,
54:       unsigned &NPRN) const;
55:   bool isHomogeneousAggregateBaseType(QualType Ty) const override;
56:   bool isHomogeneousAggregateSmallEnough(const Type *Ty,
57:                                          uint64_t Members) const override;
58:   bool isZeroLengthBitfieldPermittedInHomogeneousAggregate() const override;
59: 
60:   bool isIllegalVectorType(QualType Ty) const;
```
- **EN**: This block defines callable entry points like `getABIKind`, `isDarwinPCS`, `classifyReturnType`, `classifyArgumentType`, `coerceIllegalVector`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `getABIKind`, `isDarwinPCS`, `classifyReturnType`, `classifyArgumentType`, `coerceIllegalVector`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 61-80
```cpp
61: 
62:   bool passAsAggregateType(QualType Ty) const;
63:   bool passAsPureScalableType(QualType Ty, unsigned &NV, unsigned &NP,
64:                               SmallVectorImpl<llvm::Type *> &CoerceToSeq) const;
65: 
66:   void flattenType(llvm::Type *Ty,
67:                    SmallVectorImpl<llvm::Type *> &Flattened) const;
68: 
69:   void computeInfo(CGFunctionInfo &FI) const override {
70:     if (!::classifyReturnType(getCXXABI(), FI, *this))
71:       FI.getReturnInfo() =
72:           classifyReturnType(FI.getReturnType(), FI.isVariadic());
73: 
74:     unsigned ArgNo = 0;
75:     unsigned NSRN = 0, NPRN = 0;
76:     for (auto &it : FI.arguments()) {
77:       const bool IsNamedArg =
78:           !FI.isVariadic() || ArgNo < FI.getRequiredArgs().getNumRequiredArgs();
79:       ++ArgNo;
80:       it.info = classifyArgumentType(it.type, FI.isVariadic(), IsNamedArg,
```
- **EN**: This block defines callable entry points like `passAsAggregateType`, `passAsPureScalableType`, `flattenType`, `computeInfo`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `passAsAggregateType`, `passAsPureScalableType`, `flattenType`, `computeInfo`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 81-100
```cpp
 81:                                      FI.getCallingConvention(), NSRN, NPRN);
 82:     }
 83:   }
 84: 
 85:   RValue EmitDarwinVAArg(Address VAListAddr, QualType Ty, CodeGenFunction &CGF,
 86:                          AggValueSlot Slot) const;
 87: 
 88:   RValue EmitAAPCSVAArg(Address VAListAddr, QualType Ty, CodeGenFunction &CGF,
 89:                         AArch64ABIKind Kind, AggValueSlot Slot) const;
 90: 
 91:   RValue EmitVAArg(CodeGenFunction &CGF, Address VAListAddr, QualType Ty,
 92:                    AggValueSlot Slot) const override {
 93:     llvm::Type *BaseTy = CGF.ConvertType(Ty);
 94:     if (isa<llvm::ScalableVectorType>(BaseTy))
 95:       llvm::report_fatal_error("Passing SVE types to variadic functions is "
 96:                                "currently not supported");
 97: 
 98:     return Kind == AArch64ABIKind::Win64
 99:                ? EmitMSVAArg(CGF, VAListAddr, Ty, Slot)
100:            : isDarwinPCS() ? EmitDarwinVAArg(VAListAddr, Ty, CGF, Slot)
```
- **EN**: This block defines callable entry points like `EmitDarwinVAArg`, `EmitAAPCSVAArg`, `EmitVAArg`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `EmitDarwinVAArg`, `EmitAAPCSVAArg`, `EmitVAArg`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 101-120
```cpp
101:                            : EmitAAPCSVAArg(VAListAddr, Ty, CGF, Kind, Slot);
102:   }
103: 
104:   RValue EmitMSVAArg(CodeGenFunction &CGF, Address VAListAddr, QualType Ty,
105:                      AggValueSlot Slot) const override;
106: 
107:   bool allowBFloatArgsAndRet() const override {
108:     return getTarget().hasBFloat16Type();
109:   }
110: 
111:   using ABIInfo::appendAttributeMangling;
112:   void appendAttributeMangling(TargetClonesAttr *Attr, unsigned Index,
113:                                raw_ostream &Out) const override;
114:   void appendAttributeMangling(StringRef AttrStr,
115:                                raw_ostream &Out) const override;
116: };
117: 
118: class AArch64SwiftABIInfo : public SwiftABIInfo {
119: public:
120:   explicit AArch64SwiftABIInfo(CodeGenTypes &CGT)
```
- **EN**: This block introduces declarations such as `AArch64SwiftABIInfo`; defines callable entry points like `EmitAAPCSVAArg`, `EmitMSVAArg`, `allowBFloatArgsAndRet`, `getTarget`, `appendAttributeMangling`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块给出诸如 `AArch64SwiftABIInfo` 的声明；定义可调用入口，例如 `EmitAAPCSVAArg`, `EmitMSVAArg`, `allowBFloatArgsAndRet`, `getTarget`, `appendAttributeMangling`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 121-140
```cpp
121:       : SwiftABIInfo(CGT, /*SwiftErrorInRegister=*/true) {}
122: 
123:   bool isLegalVectorType(CharUnits VectorSize, llvm::Type *EltTy,
124:                          unsigned NumElts) const override;
125: };
126: 
127: class AArch64TargetCodeGenInfo : public TargetCodeGenInfo {
128: public:
129:   AArch64TargetCodeGenInfo(CodeGenModule &CGM, AArch64ABIKind Kind)
130:       : TargetCodeGenInfo(std::make_unique<AArch64ABIInfo>(CGM, Kind)) {
131:     SwiftInfo = std::make_unique<AArch64SwiftABIInfo>(CGM.getTypes());
132:   }
133: 
134:   StringRef getARCRetainAutoreleasedReturnValueMarker() const override {
135:     return "mov\tfp, fp\t\t// marker for objc_retainAutoreleaseReturnValue";
136:   }
137: 
138:   int getDwarfEHStackPointer(CodeGen::CodeGenModule &M) const override {
139:     return 31;
140:   }
```
- **EN**: This block introduces declarations such as `AArch64TargetCodeGenInfo`; defines callable entry points like `SwiftABIInfo`, `isLegalVectorType`, `AArch64TargetCodeGenInfo`, `getARCRetainAutoreleasedReturnValueMarker`, `getDwarfEHStackPointer`; uses control flow (for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出诸如 `AArch64TargetCodeGenInfo` 的声明；定义可调用入口，例如 `SwiftABIInfo`, `isLegalVectorType`, `AArch64TargetCodeGenInfo`, `getARCRetainAutoreleasedReturnValueMarker`, `getDwarfEHStackPointer`；通过控制流（for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 141-160
```cpp
141: 
142:   bool doesReturnSlotInterfereWithArgs() const override { return false; }
143: 
144:   void setTargetAttributes(const Decl *D, llvm::GlobalValue *GV,
145:                            CodeGen::CodeGenModule &CGM) const override {
146:     auto *Fn = dyn_cast<llvm::Function>(GV);
147:     if (!Fn)
148:       return;
149: 
150:     const auto *FD = dyn_cast_or_null<FunctionDecl>(D);
151:     TargetInfo::BranchProtectionInfo BPI(CGM.getLangOpts());
152: 
153:     if (FD && FD->hasAttr<TargetAttr>()) {
154:       const auto *TA = FD->getAttr<TargetAttr>();
155:       ParsedTargetAttr Attr =
156:           CGM.getTarget().parseTargetAttr(TA->getFeaturesStr());
157:       if (!Attr.BranchProtection.empty()) {
158:         StringRef Error;
159:         (void)CGM.getTarget().validateBranchProtection(
160:             Attr.BranchProtection, Attr.CPU, BPI, CGM.getLangOpts(), Error);
```
- **EN**: This block defines callable entry points like `doesReturnSlotInterfereWithArgs`, `setTargetAttributes`, `BPI`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `doesReturnSlotInterfereWithArgs`, `setTargetAttributes`, `BPI`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 161-180
```cpp
161:         assert(Error.empty());
162:       }
163:     }
164:     setBranchProtectionFnAttributes(BPI, *Fn);
165:     setPointerAuthFnAttributes(CGM.getCodeGenOpts().PointerAuth, *Fn);
166:   }
167: 
168:   bool isScalarizableAsmOperand(CodeGen::CodeGenFunction &CGF,
169:                                 llvm::Type *Ty) const override {
170:     if (CGF.getTarget().hasFeature("ls64")) {
171:       auto *ST = dyn_cast<llvm::StructType>(Ty);
172:       if (ST && ST->getNumElements() == 1) {
173:         auto *AT = dyn_cast<llvm::ArrayType>(ST->getElementType(0));
174:         if (AT && AT->getNumElements() == 8 &&
175:             AT->getElementType()->isIntegerTy(64))
176:           return true;
177:       }
178:     }
179:     return TargetCodeGenInfo::isScalarizableAsmOperand(CGF, Ty);
180:   }
```
- **EN**: This block defines callable entry points like `setBranchProtectionFnAttributes`, `setPointerAuthFnAttributes`, `isScalarizableAsmOperand`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `setBranchProtectionFnAttributes`, `setPointerAuthFnAttributes`, `isScalarizableAsmOperand`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 181-200
```cpp
181: 
182:   void checkFunctionABI(CodeGenModule &CGM,
183:                         const FunctionDecl *Decl) const override;
184: 
185:   void checkFunctionCallABI(CodeGenModule &CGM, SourceLocation CallLoc,
186:                             const FunctionDecl *Caller,
187:                             const FunctionDecl *Callee, const CallArgList &Args,
188:                             QualType ReturnType) const override;
189: 
190:   bool wouldInliningViolateFunctionCallABI(
191:       const FunctionDecl *Caller, const FunctionDecl *Callee) const override;
192: 
193: private:
194:   // Diagnose calls between functions with incompatible Streaming SVE
195:   // attributes.
196:   void checkFunctionCallABIStreaming(CodeGenModule &CGM, SourceLocation CallLoc,
197:                                      const FunctionDecl *Caller,
198:                                      const FunctionDecl *Callee) const;
199:   // Diagnose calls which must pass arguments in floating-point registers when
200:   // the selected target does not have floating-point registers.
```
- **EN**: This block spells out callable entry points like `checkFunctionABI`, `checkFunctionCallABI`, `wouldInliningViolateFunctionCallABI`, `checkFunctionCallABIStreaming`.
- **CN**: 该代码块给出可调用入口的声明，例如 `checkFunctionABI`, `checkFunctionCallABI`, `wouldInliningViolateFunctionCallABI`, `checkFunctionCallABIStreaming`。

### Lines 201-220
```cpp
201:   void checkFunctionCallABISoftFloat(CodeGenModule &CGM, SourceLocation CallLoc,
202:                                      const FunctionDecl *Caller,
203:                                      const FunctionDecl *Callee,
204:                                      const CallArgList &Args,
205:                                      QualType ReturnType) const;
206: };
207: 
208: class WindowsAArch64TargetCodeGenInfo : public AArch64TargetCodeGenInfo {
209: public:
210:   WindowsAArch64TargetCodeGenInfo(CodeGenModule &CGM, AArch64ABIKind K)
211:       : AArch64TargetCodeGenInfo(CGM, K) {}
212: 
213:   void setTargetAttributes(const Decl *D, llvm::GlobalValue *GV,
214:                            CodeGen::CodeGenModule &CGM) const override;
215: 
216:   void getDependentLibraryOption(llvm::StringRef Lib,
217:                                  llvm::SmallString<24> &Opt) const override {
218:     Opt = "/DEFAULTLIB:" + qualifyWindowsLibrary(Lib);
219:   }
220: 
```
- **EN**: This block introduces declarations such as `WindowsAArch64TargetCodeGenInfo`; defines callable entry points like `checkFunctionCallABISoftFloat`, `WindowsAArch64TargetCodeGenInfo`, `setTargetAttributes`, `getDependentLibraryOption`.
- **CN**: 该代码块给出诸如 `WindowsAArch64TargetCodeGenInfo` 的声明；定义可调用入口，例如 `checkFunctionCallABISoftFloat`, `WindowsAArch64TargetCodeGenInfo`, `setTargetAttributes`, `getDependentLibraryOption`。

### Lines 221-240
```cpp
221:   void getDetectMismatchOption(llvm::StringRef Name, llvm::StringRef Value,
222:                                llvm::SmallString<32> &Opt) const override {
223:     Opt = "/FAILIFMISMATCH:\"" + Name.str() + "=" + Value.str() + "\"";
224:   }
225: };
226: 
227: void WindowsAArch64TargetCodeGenInfo::setTargetAttributes(
228:     const Decl *D, llvm::GlobalValue *GV, CodeGen::CodeGenModule &CGM) const {
229:   AArch64TargetCodeGenInfo::setTargetAttributes(D, GV, CGM);
230:   if (GV->isDeclaration())
231:     return;
232:   addStackProbeTargetAttributes(D, GV, CGM);
233: }
234: }
235: 
236: llvm::Type *
237: AArch64ABIInfo::convertFixedToScalableVectorType(const VectorType *VT) const {
238:   assert(VT->getElementType()->isBuiltinType() && "expected builtin type!");
239: 
240:   if (VT->getVectorKind() == VectorKind::SveFixedLengthPredicate) {
```
- **EN**: This block defines callable entry points like `getDetectMismatchOption`, `setTargetAttributes`, `addStackProbeTargetAttributes`, `convertFixedToScalableVectorType`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getDetectMismatchOption`, `setTargetAttributes`, `addStackProbeTargetAttributes`, `convertFixedToScalableVectorType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 241-260
```cpp
241:     assert(VT->getElementType()->castAs<BuiltinType>()->getKind() ==
242:                BuiltinType::UChar &&
243:            "unexpected builtin type for SVE predicate!");
244:     return llvm::ScalableVectorType::get(llvm::Type::getInt1Ty(getVMContext()),
245:                                          16);
246:   }
247: 
248:   if (VT->getVectorKind() == VectorKind::SveFixedLengthData) {
249:     const auto *BT = VT->getElementType()->castAs<BuiltinType>();
250:     switch (BT->getKind()) {
251:     default:
252:       llvm_unreachable("unexpected builtin type for SVE vector!");
253: 
254:     case BuiltinType::SChar:
255:     case BuiltinType::UChar:
256:     case BuiltinType::MFloat8:
257:       return llvm::ScalableVectorType::get(
258:           llvm::Type::getInt8Ty(getVMContext()), 16);
259: 
260:     case BuiltinType::Short:
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, switch, for, case) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, switch, for, case）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 261-280
```cpp
261:     case BuiltinType::UShort:
262:       return llvm::ScalableVectorType::get(
263:           llvm::Type::getInt16Ty(getVMContext()), 8);
264: 
265:     case BuiltinType::Int:
266:     case BuiltinType::UInt:
267:       return llvm::ScalableVectorType::get(
268:           llvm::Type::getInt32Ty(getVMContext()), 4);
269: 
270:     case BuiltinType::Long:
271:     case BuiltinType::ULong:
272:       return llvm::ScalableVectorType::get(
273:           llvm::Type::getInt64Ty(getVMContext()), 2);
274: 
275:     case BuiltinType::Half:
276:       return llvm::ScalableVectorType::get(
277:           llvm::Type::getHalfTy(getVMContext()), 8);
278: 
279:     case BuiltinType::Float:
280:       return llvm::ScalableVectorType::get(
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 281-300
```cpp
281:           llvm::Type::getFloatTy(getVMContext()), 4);
282: 
283:     case BuiltinType::Double:
284:       return llvm::ScalableVectorType::get(
285:           llvm::Type::getDoubleTy(getVMContext()), 2);
286: 
287:     case BuiltinType::BFloat16:
288:       return llvm::ScalableVectorType::get(
289:           llvm::Type::getBFloatTy(getVMContext()), 8);
290:     }
291:   }
292: 
293:   llvm_unreachable("expected fixed-length SVE vector");
294: }
295: 
296: ABIArgInfo AArch64ABIInfo::coerceIllegalVector(QualType Ty, unsigned &NSRN,
297:                                                unsigned &NPRN) const {
298:   assert(Ty->isVectorType() && "expected vector type!");
299: 
300:   const auto *VT = Ty->castAs<VectorType>();
```
- **EN**: This block defines callable entry points like `getFloatTy`, `get`, `coerceIllegalVector`; uses control flow (case) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getFloatTy`, `get`, `coerceIllegalVector`；通过控制流（case）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 301-320
```cpp
301:   if (VT->getVectorKind() == VectorKind::SveFixedLengthPredicate) {
302:     assert(VT->getElementType()->isBuiltinType() && "expected builtin type!");
303:     assert(VT->getElementType()->castAs<BuiltinType>()->getKind() ==
304:                BuiltinType::UChar &&
305:            "unexpected builtin type for SVE predicate!");
306:     NPRN = std::min(NPRN + 1, 4u);
307:     return ABIArgInfo::getDirect(llvm::ScalableVectorType::get(
308:         llvm::Type::getInt1Ty(getVMContext()), 16));
309:   }
310: 
311:   if (VT->getVectorKind() == VectorKind::SveFixedLengthData) {
312:     NSRN = std::min(NSRN + 1, 8u);
313:     return ABIArgInfo::getDirect(convertFixedToScalableVectorType(VT));
314:   }
315: 
316:   uint64_t Size = getContext().getTypeSize(Ty);
317:   // Android promotes <2 x i8> to i16, not i32
318:   if ((isAndroid() || isOHOSFamily()) && (Size <= 16)) {
319:     llvm::Type *ResType = llvm::Type::getInt16Ty(getVMContext());
320:     return ABIArgInfo::getDirect(ResType);
```
- **EN**: This block defines callable entry points like `getDirect`; uses control flow (if, for) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 321-340
```cpp
321:   }
322:   if (Size <= 32) {
323:     llvm::Type *ResType = llvm::Type::getInt32Ty(getVMContext());
324:     return ABIArgInfo::getDirect(ResType);
325:   }
326:   if (Size == 64) {
327:     NSRN = std::min(NSRN + 1, 8u);
328:     auto *ResType =
329:         llvm::FixedVectorType::get(llvm::Type::getInt32Ty(getVMContext()), 2);
330:     return ABIArgInfo::getDirect(ResType);
331:   }
332:   if (Size == 128) {
333:     NSRN = std::min(NSRN + 1, 8u);
334:     auto *ResType =
335:         llvm::FixedVectorType::get(llvm::Type::getInt32Ty(getVMContext()), 4);
336:     return ABIArgInfo::getDirect(ResType);
337:   }
338: 
339:   return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
340:                                  /*ByVal=*/false);
```
- **EN**: This block defines callable entry points like `getDirect`, `get`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `get`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 341-360
```cpp
341: }
342: 
343: ABIArgInfo AArch64ABIInfo::coerceAndExpandPureScalableAggregate(
344:     QualType Ty, bool IsNamedArg, unsigned NVec, unsigned NPred,
345:     const SmallVectorImpl<llvm::Type *> &UnpaddedCoerceToSeq, unsigned &NSRN,
346:     unsigned &NPRN) const {
347:   if (!IsNamedArg || NSRN + NVec > 8 || NPRN + NPred > 4)
348:     return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
349:                                    /*ByVal=*/false);
350:   NSRN += NVec;
351:   NPRN += NPred;
352: 
353:   // Handle SVE vector tuples.
354:   if (Ty->isSVESizelessBuiltinType())
355:     return ABIArgInfo::getDirect();
356: 
357:   llvm::Type *UnpaddedCoerceToType =
358:       UnpaddedCoerceToSeq.size() == 1
359:           ? UnpaddedCoerceToSeq[0]
360:           : llvm::StructType::get(CGT.getLLVMContext(), UnpaddedCoerceToSeq,
```
- **EN**: This block defines callable entry points like `coerceAndExpandPureScalableAggregate`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `coerceAndExpandPureScalableAggregate`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 361-380
```cpp
361:                                   true);
362: 
363:   SmallVector<llvm::Type *> CoerceToSeq;
364:   flattenType(CGT.ConvertType(Ty), CoerceToSeq);
365:   auto *CoerceToType =
366:       llvm::StructType::get(CGT.getLLVMContext(), CoerceToSeq, false);
367: 
368:   return ABIArgInfo::getCoerceAndExpand(CoerceToType, UnpaddedCoerceToType);
369: }
370: 
371: ABIArgInfo AArch64ABIInfo::classifyArgumentType(QualType Ty, bool IsVariadicFn,
372:                                                 bool IsNamedArg,
373:                                                 unsigned CallingConvention,
374:                                                 unsigned &NSRN,
375:                                                 unsigned &NPRN) const {
376:   Ty = useFirstFieldIfTransparentUnion(Ty);
377: 
378:   if (IsVariadicFn && getTarget().getTriple().isWindowsArm64EC()) {
379:     // Arm64EC varargs functions use the x86_64 classification rules,
380:     // not the AArch64 ABI rules.
```
- **EN**: This block defines callable entry points like `flattenType`, `get`, `getCoerceAndExpand`, `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `flattenType`, `get`, `getCoerceAndExpand`, `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 381-400
```cpp
381:     return WinX86_64CodegenInfo->getABIInfo().classifyArgForArm64ECVarArg(Ty);
382:   }
383: 
384:   // Handle illegal vector types here.
385:   if (isIllegalVectorType(Ty))
386:     return coerceIllegalVector(Ty, NSRN, NPRN);
387: 
388:   if (!passAsAggregateType(Ty)) {
389:     // Treat an enum type as its underlying type.
390:     if (const auto *ED = Ty->getAsEnumDecl())
391:       Ty = ED->getIntegerType();
392: 
393:     if (const auto *EIT = Ty->getAs<BitIntType>())
394:       if (EIT->getNumBits() > 128)
395:         return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
396:                                        false);
397: 
398:     if (Ty->isVectorType())
399:       NSRN = std::min(NSRN + 1, 8u);
400:     else if (const auto *BT = Ty->getAs<BuiltinType>()) {
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 401-420
```cpp
401:       if (BT->isFloatingPoint())
402:         NSRN = std::min(NSRN + 1, 8u);
403:       else {
404:         switch (BT->getKind()) {
405:         case BuiltinType::SveBool:
406:         case BuiltinType::SveCount:
407:           NPRN = std::min(NPRN + 1, 4u);
408:           break;
409:         case BuiltinType::SveBoolx2:
410:           NPRN = std::min(NPRN + 2, 4u);
411:           break;
412:         case BuiltinType::SveBoolx4:
413:           NPRN = std::min(NPRN + 4, 4u);
414:           break;
415:         case BuiltinType::MFloat8:
416:           NSRN = std::min(NSRN + 1, 8u);
417:           break;
418:         default:
419:           if (BT->isSVESizelessBuiltinType())
420:             NSRN = std::min(
```
- **EN**: This block uses control flow (if, switch, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 421-440
```cpp
421:                 NSRN + getContext().getBuiltinVectorTypeInfo(BT).NumVectors,
422:                 8u);
423:         }
424:       }
425:     }
426: 
427:     return (isPromotableIntegerTypeForABI(Ty) && isDarwinPCS()
428:                 ? ABIArgInfo::getExtend(Ty, CGT.ConvertType(Ty))
429:                 : ABIArgInfo::getDirect());
430:   }
431: 
432:   // Structures with either a non-trivial destructor or a non-trivial
433:   // copy constructor are always indirect.
434:   if (CGCXXABI::RecordArgABI RAA = getRecordArgABI(Ty, getCXXABI())) {
435:     return getNaturalAlignIndirect(
436:         Ty, /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
437:         /*ByVal=*/RAA == CGCXXABI::RAA_DirectInMemory);
438:   }
439: 
440:   // Empty records:
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 441-460
```cpp
441:   // AAPCS64 does not say that empty records are ignored as arguments,
442:   // but other compilers do so in certain situations, and we copy that behavior.
443:   // Those situations are in fact language-mode-specific, which seems really
444:   // unfortunate, but it's something we just have to accept. If this doesn't
445:   // apply, just fall through to the standard argument-handling path.
446:   // Darwin overrides the psABI here to ignore all empty records in all modes.
447:   uint64_t Size = getContext().getTypeSize(Ty);
448:   bool IsEmpty = isEmptyRecord(getContext(), Ty, true);
449:   if (!Ty->isSVESizelessBuiltinType() && (IsEmpty || Size == 0)) {
450:     // Empty records are ignored in C mode, and in C++ on Darwin.
451:     if (!getContext().getLangOpts().CPlusPlus || isDarwinPCS())
452:       return ABIArgInfo::getIgnore();
453: 
454:     // In C++ mode, arguments which have sizeof() == 0 (which are non-standard
455:     // C++) are ignored. This isn't defined by any standard, so we copy GCC's
456:     // behaviour here.
457:     if (Size == 0)
458:       return ABIArgInfo::getIgnore();
459:   }
460: 
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 461-480
```cpp
461:   // Homogeneous Floating-point Aggregates (HFAs) need to be expanded.
462:   const Type *Base = nullptr;
463:   uint64_t Members = 0;
464:   bool IsWin64 = Kind == AArch64ABIKind::Win64 ||
465:                  CallingConvention == llvm::CallingConv::Win64;
466:   bool IsWinVariadic = IsWin64 && IsVariadicFn;
467:   // In variadic functions on Windows, all composite types are treated alike,
468:   // no special handling of HFAs/HVAs.
469:   if (!IsWinVariadic && isHomogeneousAggregate(Ty, Base, Members)) {
470:     NSRN = std::min(NSRN + Members, uint64_t(8));
471:     if (Kind != AArch64ABIKind::AAPCS)
472:       return ABIArgInfo::getDirect(
473:           llvm::ArrayType::get(CGT.ConvertType(QualType(Base, 0)), Members));
474: 
475:     // For HFAs/HVAs, cap the argument alignment to 16, otherwise
476:     // set it to 8 according to the AAPCS64 document.
477:     unsigned Align =
478:         getContext().getTypeUnadjustedAlignInChars(Ty).getQuantity();
479:     Align = (Align >= 16) ? 16 : 8;
480:     return ABIArgInfo::getDirect(
```
- **EN**: This block defines callable entry points like `getContext`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getContext`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 481-500
```cpp
481:         llvm::ArrayType::get(CGT.ConvertType(QualType(Base, 0)), Members), 0,
482:         nullptr, true, Align);
483:   }
484: 
485:   // In AAPCS named arguments of a Pure Scalable Type are passed expanded in
486:   // registers, or indirectly if there are not enough registers.
487:   if (Kind == AArch64ABIKind::AAPCS) {
488:     unsigned NVec = 0, NPred = 0;
489:     SmallVector<llvm::Type *> UnpaddedCoerceToSeq;
490:     if (passAsPureScalableType(Ty, NVec, NPred, UnpaddedCoerceToSeq) &&
491:         (NVec + NPred) > 0)
492:       return coerceAndExpandPureScalableAggregate(
493:           Ty, IsNamedArg, NVec, NPred, UnpaddedCoerceToSeq, NSRN, NPRN);
494:   }
495: 
496:   // Aggregates <= 16 bytes are passed directly in registers or on the stack.
497:   if (Size <= 128) {
498:     unsigned Alignment;
499:     if (Kind == AArch64ABIKind::AAPCS) {
500:       Alignment = getContext().getTypeUnadjustedAlign(Ty);
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 501-520
```cpp
501:       Alignment = Alignment < 128 ? 64 : 128;
502:     } else {
503:       Alignment =
504:           std::max(getContext().getTypeAlign(Ty),
505:                    (unsigned)getTarget().getPointerWidth(LangAS::Default));
506:     }
507:     Size = llvm::alignTo(Size, Alignment);
508: 
509:     // If the Aggregate is made up of pointers, use an array of pointers for the
510:     // coerced type. This prevents having to convert ptr2int->int2ptr through
511:     // the call, allowing alias analysis to produce better code.
512:     auto ContainsOnlyPointers = [&](const auto &Self, QualType Ty) {
513:       if (isEmptyRecord(getContext(), Ty, true))
514:         return false;
515:       const auto *RD = Ty->getAsRecordDecl();
516:       if (!RD)
517:         return false;
518:       if (const CXXRecordDecl *CXXRD = dyn_cast<CXXRecordDecl>(RD)) {
519:         for (const auto &I : CXXRD->bases())
520:           if (!Self(Self, I.getType()))
```
- **EN**: This block defines callable entry points like `max`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `max`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 521-540
```cpp
521:             return false;
522:       }
523:       return all_of(RD->fields(), [&](FieldDecl *FD) {
524:         QualType FDTy = FD->getType();
525:         if (FDTy->isArrayType())
526:           FDTy = getContext().getBaseElementType(FDTy);
527:         return (FDTy->isPointerOrReferenceType() &&
528:                 getContext().getTypeSize(FDTy) == 64 &&
529:                 !FDTy->getPointeeType().hasAddressSpace()) ||
530:                Self(Self, FDTy);
531:       });
532:     };
533: 
534:     // We use a pair of i64 for 16-byte aggregate with 8-byte alignment.
535:     // For aggregates with 16-byte alignment, we use i128.
536:     llvm::Type *BaseTy = llvm::Type::getIntNTy(getVMContext(), Alignment);
537:     if ((Size == 64 || Size == 128) && Alignment == 64 &&
538:         ContainsOnlyPointers(ContainsOnlyPointers, Ty))
539:       BaseTy = llvm::PointerType::getUnqual(getVMContext());
540:     return ABIArgInfo::getDirect(
```
- **EN**: This block defines callable entry points like `all_of`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `all_of`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 541-560
```cpp
541:         Size == Alignment ? BaseTy
542:                           : llvm::ArrayType::get(BaseTy, Size / Alignment));
543:   }
544: 
545:   return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
546:                                  /*ByVal=*/false);
547: }
548: 
549: ABIArgInfo AArch64ABIInfo::classifyReturnType(QualType RetTy,
550:                                               bool IsVariadicFn) const {
551:   if (RetTy->isVoidType())
552:     return ABIArgInfo::getIgnore();
553: 
554:   if (const auto *VT = RetTy->getAs<VectorType>()) {
555:     if (VT->getVectorKind() == VectorKind::SveFixedLengthData ||
556:         VT->getVectorKind() == VectorKind::SveFixedLengthPredicate) {
557:       unsigned NSRN = 0, NPRN = 0;
558:       return coerceIllegalVector(RetTy, NSRN, NPRN);
559:     }
560:   }
```
- **EN**: This block defines callable entry points like `get`, `classifyReturnType`, `coerceIllegalVector`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `get`, `classifyReturnType`, `coerceIllegalVector`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 561-580
```cpp
561: 
562:   // Large vector types should be returned via memory.
563:   if (RetTy->isVectorType() && getContext().getTypeSize(RetTy) > 128)
564:     return getNaturalAlignIndirect(RetTy, getDataLayout().getAllocaAddrSpace());
565: 
566:   if (!passAsAggregateType(RetTy)) {
567:     // Treat an enum type as its underlying type.
568:     if (const auto *ED = RetTy->getAsEnumDecl())
569:       RetTy = ED->getIntegerType();
570: 
571:     if (const auto *EIT = RetTy->getAs<BitIntType>())
572:       if (EIT->getNumBits() > 128)
573:         return getNaturalAlignIndirect(RetTy,
574:                                        getDataLayout().getAllocaAddrSpace());
575: 
576:     return (isPromotableIntegerTypeForABI(RetTy) && isDarwinPCS()
577:                 ? ABIArgInfo::getExtend(RetTy)
578:                 : ABIArgInfo::getDirect());
579:   }
580: 
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 581-600
```cpp
581:   uint64_t Size = getContext().getTypeSize(RetTy);
582:   if (!RetTy->isSVESizelessBuiltinType() &&
583:       (isEmptyRecord(getContext(), RetTy, true) || Size == 0))
584:     return ABIArgInfo::getIgnore();
585: 
586:   const Type *Base = nullptr;
587:   uint64_t Members = 0;
588:   if (isHomogeneousAggregate(RetTy, Base, Members) &&
589:       !(getTarget().getTriple().getArch() == llvm::Triple::aarch64_32 &&
590:         IsVariadicFn))
591:     // Homogeneous Floating-point Aggregates (HFAs) are returned directly.
592:     return ABIArgInfo::getDirect();
593: 
594:   // In AAPCS return values of a Pure Scalable type are treated as a single
595:   // named argument and passed expanded in registers, or indirectly if there are
596:   // not enough registers.
597:   if (Kind == AArch64ABIKind::AAPCS) {
598:     unsigned NSRN = 0, NPRN = 0;
599:     unsigned NVec = 0, NPred = 0;
600:     SmallVector<llvm::Type *> UnpaddedCoerceToSeq;
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 601-620
```cpp
601:     if (passAsPureScalableType(RetTy, NVec, NPred, UnpaddedCoerceToSeq) &&
602:         (NVec + NPred) > 0)
603:       return coerceAndExpandPureScalableAggregate(
604:           RetTy, /* IsNamedArg */ true, NVec, NPred, UnpaddedCoerceToSeq, NSRN,
605:           NPRN);
606:   }
607: 
608:   // Aggregates <= 16 bytes are returned directly in registers or on the stack.
609:   if (Size <= 128) {
610:     if (Size <= 64 && getDataLayout().isLittleEndian()) {
611:       // Composite types are returned in lower bits of a 64-bit register for LE,
612:       // and in higher bits for BE. However, integer types are always returned
613:       // in lower bits for both LE and BE, and they are not rounded up to
614:       // 64-bits. We can skip rounding up of composite types for LE, but not for
615:       // BE, otherwise composite types will be indistinguishable from integer
616:       // types.
617:       return ABIArgInfo::getDirect(
618:           llvm::IntegerType::get(getVMContext(), Size));
619:     }
620: 
```
- **EN**: This block defines callable entry points like `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 621-640
```cpp
621:     unsigned Alignment = getContext().getTypeAlign(RetTy);
622:     Size = llvm::alignTo(Size, 64); // round up to multiple of 8 bytes
623: 
624:     // We use a pair of i64 for 16-byte aggregate with 8-byte alignment.
625:     // For aggregates with 16-byte alignment, we use i128.
626:     if (Alignment < 128 && Size == 128) {
627:       llvm::Type *BaseTy = llvm::Type::getInt64Ty(getVMContext());
628:       return ABIArgInfo::getDirect(llvm::ArrayType::get(BaseTy, Size / 64));
629:     }
630:     return ABIArgInfo::getDirect(llvm::IntegerType::get(getVMContext(), Size));
631:   }
632: 
633:   return getNaturalAlignIndirect(RetTy, getDataLayout().getAllocaAddrSpace());
634: }
635: 
636: /// isIllegalVectorType - check whether the vector type is legal for AArch64.
637: bool AArch64ABIInfo::isIllegalVectorType(QualType Ty) const {
638:   if (const VectorType *VT = Ty->getAs<VectorType>()) {
639:     // Check whether VT is a fixed-length SVE vector. These types are
640:     // represented as scalable vectors in function args/return and must be
```
- **EN**: This block defines callable entry points like `getDirect`, `getNaturalAlignIndirect`, `isIllegalVectorType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `getNaturalAlignIndirect`, `isIllegalVectorType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 641-660
```cpp
641:     // coerced from fixed vectors.
642:     if (VT->getVectorKind() == VectorKind::SveFixedLengthData ||
643:         VT->getVectorKind() == VectorKind::SveFixedLengthPredicate)
644:       return true;
645: 
646:     // Check whether VT is legal.
647:     unsigned NumElements = VT->getNumElements();
648:     uint64_t Size = getContext().getTypeSize(VT);
649:     // NumElements should be power of 2.
650:     if (!llvm::isPowerOf2_32(NumElements))
651:       return true;
652: 
653:     // arm64_32 has to be compatible with the ARM logic here, which allows huge
654:     // vectors for some reason.
655:     llvm::Triple Triple = getTarget().getTriple();
656:     if (Triple.getArch() == llvm::Triple::aarch64_32 &&
657:         Triple.isOSBinFormatMachO())
658:       return Size <= 32;
659: 
660:     return Size != 64 && (Size != 128 || NumElements == 1);
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 661-680
```cpp
661:   }
662:   return false;
663: }
664: 
665: bool AArch64SwiftABIInfo::isLegalVectorType(CharUnits VectorSize,
666:                                             llvm::Type *EltTy,
667:                                             unsigned NumElts) const {
668:   if (!llvm::isPowerOf2_32(NumElts))
669:     return false;
670:   if (VectorSize.getQuantity() != 8 &&
671:       (VectorSize.getQuantity() != 16 || NumElts == 1))
672:     return false;
673:   return true;
674: }
675: 
676: bool AArch64ABIInfo::isHomogeneousAggregateBaseType(QualType Ty) const {
677:   // For the soft-float ABI variant, no types are considered to be homogeneous
678:   // aggregates.
679:   if (isSoftFloat())
680:     return false;
```
- **EN**: This block defines callable entry points like `isLegalVectorType`, `isHomogeneousAggregateBaseType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isLegalVectorType`, `isHomogeneousAggregateBaseType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 681-700
```cpp
681: 
682:   // Homogeneous aggregates for AAPCS64 must have base types of a floating
683:   // point type or a short-vector type. This is the same as the 32-bit ABI,
684:   // but with the difference that any floating-point type is allowed,
685:   // including __fp16.
686:   if (const BuiltinType *BT = Ty->getAs<BuiltinType>()) {
687:     if (BT->isFloatingPoint())
688:       return true;
689:   } else if (const VectorType *VT = Ty->getAs<VectorType>()) {
690:     if (auto Kind = VT->getVectorKind();
691:         Kind == VectorKind::SveFixedLengthData ||
692:         Kind == VectorKind::SveFixedLengthPredicate)
693:       return false;
694: 
695:     unsigned VecSize = getContext().getTypeSize(VT);
696:     if (VecSize == 64 || VecSize == 128)
697:       return true;
698:   }
699:   return false;
700: }
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 701-720
```cpp
701: 
702: bool AArch64ABIInfo::isHomogeneousAggregateSmallEnough(const Type *Base,
703:                                                        uint64_t Members) const {
704:   return Members <= 4;
705: }
706: 
707: bool AArch64ABIInfo::isZeroLengthBitfieldPermittedInHomogeneousAggregate()
708:     const {
709:   // AAPCS64 says that the rule for whether something is a homogeneous
710:   // aggregate is applied to the output of the data layout decision. So
711:   // anything that doesn't affect the data layout also does not affect
712:   // homogeneity. In particular, zero-length bitfields don't stop a struct
713:   // being homogeneous.
714:   return true;
715: }
716: 
717: bool AArch64ABIInfo::passAsAggregateType(QualType Ty) const {
718:   if (Kind == AArch64ABIKind::AAPCS && Ty->isSVESizelessBuiltinType()) {
719:     const auto *BT = Ty->castAs<BuiltinType>();
720:     return !BT->isSVECount() &&
```
- **EN**: This block defines callable entry points like `isHomogeneousAggregateSmallEnough`, `isZeroLengthBitfieldPermittedInHomogeneousAggregate`, `passAsAggregateType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isHomogeneousAggregateSmallEnough`, `isZeroLengthBitfieldPermittedInHomogeneousAggregate`, `passAsAggregateType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 721-740
```cpp
721:            getContext().getBuiltinVectorTypeInfo(BT).NumVectors > 1;
722:   }
723:   return isAggregateTypeForABI(Ty);
724: }
725: 
726: // Check if a type needs to be passed in registers as a Pure Scalable Type (as
727: // defined by AAPCS64). Return the number of data vectors and the number of
728: // predicate vectors in the type, into `NVec` and `NPred`, respectively. Upon
729: // return `CoerceToSeq` contains an expanded sequence of LLVM IR types, one
730: // element for each non-composite member. For practical purposes, limit the
731: // length of `CoerceToSeq` to about 12 (the maximum that could possibly fit
732: // in registers) and return false, the effect of which will be to  pass the
733: // argument under the rules for a large (> 128 bytes) composite.
734: bool AArch64ABIInfo::passAsPureScalableType(
735:     QualType Ty, unsigned &NVec, unsigned &NPred,
736:     SmallVectorImpl<llvm::Type *> &CoerceToSeq) const {
737:   if (const ConstantArrayType *AT = getContext().getAsConstantArrayType(Ty)) {
738:     uint64_t NElt = AT->getZExtSize();
739:     if (NElt == 0)
740:       return false;
```
- **EN**: This block defines callable entry points like `isAggregateTypeForABI`, `passAsPureScalableType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isAggregateTypeForABI`, `passAsPureScalableType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 741-760
```cpp
741: 
742:     unsigned NV = 0, NP = 0;
743:     SmallVector<llvm::Type *> EltCoerceToSeq;
744:     if (!passAsPureScalableType(AT->getElementType(), NV, NP, EltCoerceToSeq))
745:       return false;
746: 
747:     if (CoerceToSeq.size() + NElt * EltCoerceToSeq.size() > 12)
748:       return false;
749: 
750:     for (uint64_t I = 0; I < NElt; ++I)
751:       llvm::append_range(CoerceToSeq, EltCoerceToSeq);
752: 
753:     NVec += NElt * NV;
754:     NPred += NElt * NP;
755:     return true;
756:   }
757: 
758:   if (const RecordType *RT = Ty->getAsCanonical<RecordType>()) {
759:     // If the record cannot be passed in registers, then it's not a PST.
760:     if (CGCXXABI::RecordArgABI RAA = getRecordArgABI(RT, getCXXABI());
```
- **EN**: This block defines callable entry points like `append_range`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `append_range`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 761-780
```cpp
761:         RAA != CGCXXABI::RAA_Default)
762:       return false;
763: 
764:     // Pure scalable types are never unions and never contain unions.
765:     const RecordDecl *RD = RT->getDecl()->getDefinitionOrSelf();
766:     if (RD->isUnion())
767:       return false;
768: 
769:     // If this is a C++ record, check the bases.
770:     if (const CXXRecordDecl *CXXRD = dyn_cast<CXXRecordDecl>(RD)) {
771:       for (const auto &I : CXXRD->bases()) {
772:         if (isEmptyRecord(getContext(), I.getType(), true))
773:           continue;
774:         if (!passAsPureScalableType(I.getType(), NVec, NPred, CoerceToSeq))
775:           return false;
776:       }
777:     }
778: 
779:     // Check members.
780:     for (const auto *FD : RD->fields()) {
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 781-800
```cpp
781:       QualType FT = FD->getType();
782:       if (isEmptyField(getContext(), FD, /* AllowArrays */ true))
783:         continue;
784:       if (!passAsPureScalableType(FT, NVec, NPred, CoerceToSeq))
785:         return false;
786:     }
787: 
788:     return true;
789:   }
790: 
791:   if (const auto *VT = Ty->getAs<VectorType>()) {
792:     if (VT->getVectorKind() == VectorKind::SveFixedLengthPredicate) {
793:       ++NPred;
794:       if (CoerceToSeq.size() + 1 > 12)
795:         return false;
796:       CoerceToSeq.push_back(convertFixedToScalableVectorType(VT));
797:       return true;
798:     }
799: 
800:     if (VT->getVectorKind() == VectorKind::SveFixedLengthData) {
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 801-820
```cpp
801:       ++NVec;
802:       if (CoerceToSeq.size() + 1 > 12)
803:         return false;
804:       CoerceToSeq.push_back(convertFixedToScalableVectorType(VT));
805:       return true;
806:     }
807: 
808:     return false;
809:   }
810: 
811:   if (!Ty->isBuiltinType())
812:     return false;
813: 
814:   bool isPredicate;
815:   switch (Ty->castAs<BuiltinType>()->getKind()) {
816: #define SVE_VECTOR_TYPE(Name, MangledName, Id, SingletonId)                    \
817:   case BuiltinType::Id:                                                        \
818:     isPredicate = false;                                                       \
819:     break;
820: #define SVE_PREDICATE_TYPE(Name, MangledName, Id, SingletonId)                 \
```
- **EN**: This block uses control flow (if, switch, case) to specialize target-specific ABI and code generation; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标相关的 ABI 与代码生成 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 821-840
```cpp
821:   case BuiltinType::Id:                                                        \
822:     isPredicate = true;                                                        \
823:     break;
824: #include "clang/Basic/AArch64ACLETypes.def"
825:   default:
826:     return false;
827:   }
828: 
829:   ASTContext::BuiltinVectorTypeInfo Info =
830:       getContext().getBuiltinVectorTypeInfo(cast<BuiltinType>(Ty));
831:   assert(Info.NumVectors > 0 && Info.NumVectors <= 4 &&
832:          "Expected 1, 2, 3 or 4 vectors!");
833:   if (isPredicate)
834:     NPred += Info.NumVectors;
835:   else
836:     NVec += Info.NumVectors;
837:   llvm::Type *EltTy = Info.ElementType->isMFloat8Type()
838:                           ? llvm::Type::getInt8Ty(getVMContext())
839:                           : CGT.ConvertType(Info.ElementType);
840:   auto *VTy = llvm::ScalableVectorType::get(EltTy, Info.EC.getKnownMinValue());
```
- **EN**: This block imports Clang headers `clang/Basic/AArch64ACLETypes.def`; spells out callable entry points like `getContext`; uses control flow (if, case) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/Basic/AArch64ACLETypes.def`；给出可调用入口的声明，例如 `getContext`；通过控制流（if, case）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

### Lines 841-860
```cpp
841: 
842:   if (CoerceToSeq.size() + Info.NumVectors > 12)
843:     return false;
844:   std::fill_n(std::back_inserter(CoerceToSeq), Info.NumVectors, VTy);
845: 
846:   return true;
847: }
848: 
849: // Expand an LLVM IR type into a sequence with a element for each non-struct,
850: // non-array member of the type, with the exception of the padding types, which
851: // are retained.
852: void AArch64ABIInfo::flattenType(
853:     llvm::Type *Ty, SmallVectorImpl<llvm::Type *> &Flattened) const {
854: 
855:   if (ABIArgInfo::isPaddingForCoerceAndExpand(Ty)) {
856:     Flattened.push_back(Ty);
857:     return;
858:   }
859: 
860:   if (const auto *AT = dyn_cast<llvm::ArrayType>(Ty)) {
```
- **EN**: This block defines callable entry points like `fill_n`, `flattenType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `fill_n`, `flattenType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 861-880
```cpp
861:     uint64_t NElt = AT->getNumElements();
862:     if (NElt == 0)
863:       return;
864: 
865:     SmallVector<llvm::Type *> EltFlattened;
866:     flattenType(AT->getElementType(), EltFlattened);
867: 
868:     for (uint64_t I = 0; I < NElt; ++I)
869:       llvm::append_range(Flattened, EltFlattened);
870:     return;
871:   }
872: 
873:   if (const auto *ST = dyn_cast<llvm::StructType>(Ty)) {
874:     for (auto *ET : ST->elements())
875:       flattenType(ET, Flattened);
876:     return;
877:   }
878: 
879:   Flattened.push_back(Ty);
880: }
```
- **EN**: This block defines callable entry points like `flattenType`, `append_range`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `flattenType`, `append_range`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 881-900
```cpp
881: 
882: RValue AArch64ABIInfo::EmitAAPCSVAArg(Address VAListAddr, QualType Ty,
883:                                       CodeGenFunction &CGF, AArch64ABIKind Kind,
884:                                       AggValueSlot Slot) const {
885:   // These numbers are not used for variadic arguments, hence it doesn't matter
886:   // they don't retain their values across multiple calls to
887:   // `classifyArgumentType` here.
888:   unsigned NSRN = 0, NPRN = 0;
889:   ABIArgInfo AI =
890:       classifyArgumentType(Ty, /*IsVariadicFn=*/true, /* IsNamedArg */ false,
891:                            CGF.CurFnInfo->getCallingConvention(), NSRN, NPRN);
892:   // Empty records are ignored for parameter passing purposes.
893:   if (AI.isIgnore())
894:     return Slot.asRValue();
895: 
896:   bool IsIndirect = AI.isIndirect();
897: 
898:   llvm::Type *BaseTy = CGF.ConvertType(Ty);
899:   if (IsIndirect)
900:     BaseTy = llvm::PointerType::getUnqual(BaseTy->getContext());
```
- **EN**: This block defines callable entry points like `EmitAAPCSVAArg`, `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `EmitAAPCSVAArg`, `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 901-920
```cpp
901:   else if (AI.getCoerceToType())
902:     BaseTy = AI.getCoerceToType();
903: 
904:   unsigned NumRegs = 1;
905:   if (llvm::ArrayType *ArrTy = dyn_cast<llvm::ArrayType>(BaseTy)) {
906:     BaseTy = ArrTy->getElementType();
907:     NumRegs = ArrTy->getNumElements();
908:   }
909:   bool IsFPR =
910:       !isSoftFloat() && (BaseTy->isFloatingPointTy() || BaseTy->isVectorTy());
911: 
912:   // The AArch64 va_list type and handling is specified in the Procedure Call
913:   // Standard, section B.4:
914:   //
915:   // struct {
916:   //   void *__stack;
917:   //   void *__gr_top;
918:   //   void *__vr_top;
919:   //   int __gr_offs;
920:   //   int __vr_offs;
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 921-940
```cpp
921:   // };
922: 
923:   llvm::BasicBlock *MaybeRegBlock = CGF.createBasicBlock("vaarg.maybe_reg");
924:   llvm::BasicBlock *InRegBlock = CGF.createBasicBlock("vaarg.in_reg");
925:   llvm::BasicBlock *OnStackBlock = CGF.createBasicBlock("vaarg.on_stack");
926:   llvm::BasicBlock *ContBlock = CGF.createBasicBlock("vaarg.end");
927: 
928:   CharUnits TySize = getContext().getTypeSizeInChars(Ty);
929:   CharUnits TyAlign = getContext().getTypeUnadjustedAlignInChars(Ty);
930: 
931:   Address reg_offs_p = Address::invalid();
932:   llvm::Value *reg_offs = nullptr;
933:   int reg_top_index;
934:   int RegSize = IsIndirect ? 8 : TySize.getQuantity();
935:   if (!IsFPR) {
936:     // 3 is the field number of __gr_offs
937:     reg_offs_p = CGF.Builder.CreateStructGEP(VAListAddr, 3, "gr_offs_p");
938:     reg_offs = CGF.Builder.CreateLoad(reg_offs_p, "gr_offs");
939:     reg_top_index = 1; // field number for __gr_top
940:     RegSize = llvm::alignTo(RegSize, 8);
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 941-960
```cpp
941:   } else {
942:     // 4 is the field number of __vr_offs.
943:     reg_offs_p = CGF.Builder.CreateStructGEP(VAListAddr, 4, "vr_offs_p");
944:     reg_offs = CGF.Builder.CreateLoad(reg_offs_p, "vr_offs");
945:     reg_top_index = 2; // field number for __vr_top
946:     RegSize = 16 * NumRegs;
947:   }
948: 
949:   //=======================================
950:   // Find out where argument was passed
951:   //=======================================
952: 
953:   // If reg_offs >= 0 we're already using the stack for this type of
954:   // argument. We don't want to keep updating reg_offs (in case it overflows,
955:   // though anyone passing 2GB of arguments, each at most 16 bytes, deserves
956:   // whatever they get).
957:   llvm::Value *UsingStack = nullptr;
958:   UsingStack = CGF.Builder.CreateICmpSGE(
959:       reg_offs, llvm::ConstantInt::get(CGF.Int32Ty, 0));
960: 
```
- **EN**: This block defines callable entry points like `get`; uses control flow (for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 961-980
```cpp
961:   CGF.Builder.CreateCondBr(UsingStack, OnStackBlock, MaybeRegBlock);
962: 
963:   // Otherwise, at least some kind of argument could go in these registers, the
964:   // question is whether this particular type is too big.
965:   CGF.EmitBlock(MaybeRegBlock);
966: 
967:   // Integer arguments may need to correct register alignment (for example a
968:   // "struct { __int128 a; };" gets passed in x_2N, x_{2N+1}). In this case we
969:   // align __gr_offs to calculate the potential address.
970:   if (!IsFPR && !IsIndirect && TyAlign.getQuantity() > 8) {
971:     int Align = TyAlign.getQuantity();
972: 
973:     reg_offs = CGF.Builder.CreateAdd(
974:         reg_offs, llvm::ConstantInt::get(CGF.Int32Ty, Align - 1),
975:         "align_regoffs");
976:     reg_offs = CGF.Builder.CreateAnd(
977:         reg_offs, llvm::ConstantInt::getSigned(CGF.Int32Ty, -Align),
978:         "aligned_regoffs");
979:   }
980: 
```
- **EN**: This block defines callable entry points like `get`, `getSigned`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getSigned`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 981-1000
```cpp
 981:   // Update the gr_offs/vr_offs pointer for next call to va_arg on this va_list.
 982:   // The fact that this is done unconditionally reflects the fact that
 983:   // allocating an argument to the stack also uses up all the remaining
 984:   // registers of the appropriate kind.
 985:   llvm::Value *NewOffset = nullptr;
 986:   NewOffset = CGF.Builder.CreateAdd(
 987:       reg_offs, llvm::ConstantInt::get(CGF.Int32Ty, RegSize), "new_reg_offs");
 988:   CGF.Builder.CreateStore(NewOffset, reg_offs_p);
 989: 
 990:   // Now we're in a position to decide whether this argument really was in
 991:   // registers or not.
 992:   llvm::Value *InRegs = nullptr;
 993:   InRegs = CGF.Builder.CreateICmpSLE(
 994:       NewOffset, llvm::ConstantInt::get(CGF.Int32Ty, 0), "inreg");
 995: 
 996:   CGF.Builder.CreateCondBr(InRegs, InRegBlock, OnStackBlock);
 997: 
 998:   //=======================================
 999:   // Argument was in registers
1000:   //=======================================
```
- **EN**: This block spells out callable entry points like `get`.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`。

### Lines 1001-1020
```cpp
1001: 
1002:   // Now we emit the code for if the argument was originally passed in
1003:   // registers. First start the appropriate block:
1004:   CGF.EmitBlock(InRegBlock);
1005: 
1006:   llvm::Value *reg_top = nullptr;
1007:   Address reg_top_p =
1008:       CGF.Builder.CreateStructGEP(VAListAddr, reg_top_index, "reg_top_p");
1009:   reg_top = CGF.Builder.CreateLoad(reg_top_p, "reg_top");
1010:   Address BaseAddr(CGF.Builder.CreateInBoundsGEP(CGF.Int8Ty, reg_top, reg_offs),
1011:                    CGF.Int8Ty, CharUnits::fromQuantity(IsFPR ? 16 : 8));
1012:   Address RegAddr = Address::invalid();
1013:   llvm::Type *MemTy = CGF.ConvertTypeForMem(Ty), *ElementTy = MemTy;
1014: 
1015:   if (IsIndirect) {
1016:     // If it's been passed indirectly (actually a struct), whatever we find from
1017:     // stored registers or on the stack will actually be a struct **.
1018:     MemTy = llvm::PointerType::getUnqual(MemTy->getContext());
1019:   }
1020: 
```
- **EN**: This block defines callable entry points like `BaseAddr`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `BaseAddr`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1021-1040
```cpp
1021:   const Type *Base = nullptr;
1022:   uint64_t NumMembers = 0;
1023:   bool IsHFA = isHomogeneousAggregate(Ty, Base, NumMembers);
1024:   if (IsHFA && NumMembers > 1) {
1025:     // Homogeneous aggregates passed in registers will have their elements split
1026:     // and stored 16-bytes apart regardless of size (they're notionally in qN,
1027:     // qN+1, ...). We reload and store into a temporary local variable
1028:     // contiguously.
1029:     assert(!IsIndirect && "Homogeneous aggregates should be passed directly");
1030:     auto BaseTyInfo = getContext().getTypeInfoInChars(QualType(Base, 0));
1031:     llvm::Type *BaseTy = CGF.ConvertType(QualType(Base, 0));
1032:     llvm::Type *HFATy = llvm::ArrayType::get(BaseTy, NumMembers);
1033:     Address Tmp = CGF.CreateTempAlloca(HFATy,
1034:                                        std::max(TyAlign, BaseTyInfo.Align));
1035: 
1036:     // On big-endian platforms, the value will be right-aligned in its slot.
1037:     int Offset = 0;
1038:     if (CGF.CGM.getDataLayout().isBigEndian() &&
1039:         BaseTyInfo.Width.getQuantity() < 16)
1040:       Offset = 16 - BaseTyInfo.Width.getQuantity();
```
- **EN**: This block defines callable entry points like `max`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `max`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1041-1060
```cpp
1041: 
1042:     for (unsigned i = 0; i < NumMembers; ++i) {
1043:       CharUnits BaseOffset = CharUnits::fromQuantity(16 * i + Offset);
1044:       Address LoadAddr =
1045:         CGF.Builder.CreateConstInBoundsByteGEP(BaseAddr, BaseOffset);
1046:       LoadAddr = LoadAddr.withElementType(BaseTy);
1047: 
1048:       Address StoreAddr = CGF.Builder.CreateConstArrayGEP(Tmp, i);
1049: 
1050:       llvm::Value *Elem = CGF.Builder.CreateLoad(LoadAddr);
1051:       CGF.Builder.CreateStore(Elem, StoreAddr);
1052:     }
1053: 
1054:     RegAddr = Tmp.withElementType(MemTy);
1055:   } else {
1056:     // Otherwise the object is contiguous in memory.
1057: 
1058:     // It might be right-aligned in its slot.
1059:     CharUnits SlotSize = BaseAddr.getAlignment();
1060:     if (CGF.CGM.getDataLayout().isBigEndian() && !IsIndirect &&
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1061-1080
```cpp
1061:         (IsHFA || !isAggregateTypeForABI(Ty)) &&
1062:         TySize < SlotSize) {
1063:       CharUnits Offset = SlotSize - TySize;
1064:       BaseAddr = CGF.Builder.CreateConstInBoundsByteGEP(BaseAddr, Offset);
1065:     }
1066: 
1067:     RegAddr = BaseAddr.withElementType(MemTy);
1068:   }
1069: 
1070:   CGF.EmitBranch(ContBlock);
1071: 
1072:   //=======================================
1073:   // Argument was on the stack
1074:   //=======================================
1075:   CGF.EmitBlock(OnStackBlock);
1076: 
1077:   Address stack_p = CGF.Builder.CreateStructGEP(VAListAddr, 0, "stack_p");
1078:   llvm::Value *OnStackPtr = CGF.Builder.CreateLoad(stack_p, "stack");
1079: 
1080:   // Again, stack arguments may need realignment. In this case both integer and
```
- **EN**: This block documents intent or context for the surrounding target-specific ABI and code generation code.
- **CN**: 该代码块说明周围 目标相关的 ABI 与代码生成 代码的意图或上下文。

### Lines 1081-1100
```cpp
1081:   // floating-point ones might be affected.
1082:   if (!IsIndirect && TyAlign.getQuantity() > 8) {
1083:     OnStackPtr = emitRoundPointerUpToAlignment(CGF, OnStackPtr, TyAlign);
1084:   }
1085:   Address OnStackAddr = Address(OnStackPtr, CGF.Int8Ty,
1086:                                 std::max(CharUnits::fromQuantity(8), TyAlign));
1087: 
1088:   // All stack slots are multiples of 8 bytes.
1089:   CharUnits StackSlotSize = CharUnits::fromQuantity(8);
1090:   CharUnits StackSize;
1091:   if (IsIndirect)
1092:     StackSize = StackSlotSize;
1093:   else
1094:     StackSize = TySize.alignTo(StackSlotSize);
1095: 
1096:   llvm::Value *StackSizeC = CGF.Builder.getSize(StackSize);
1097:   llvm::Value *NewStack = CGF.Builder.CreateInBoundsGEP(
1098:       CGF.Int8Ty, OnStackPtr, StackSizeC, "new_stack");
1099: 
1100:   // Write the new value of __stack for the next call to va_arg
```
- **EN**: This block defines callable entry points like `max`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `max`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1101-1120
```cpp
1101:   CGF.Builder.CreateStore(NewStack, stack_p);
1102: 
1103:   if (CGF.CGM.getDataLayout().isBigEndian() && !isAggregateTypeForABI(Ty) &&
1104:       TySize < StackSlotSize) {
1105:     CharUnits Offset = StackSlotSize - TySize;
1106:     OnStackAddr = CGF.Builder.CreateConstInBoundsByteGEP(OnStackAddr, Offset);
1107:   }
1108: 
1109:   OnStackAddr = OnStackAddr.withElementType(MemTy);
1110: 
1111:   CGF.EmitBranch(ContBlock);
1112: 
1113:   //=======================================
1114:   // Tidy up
1115:   //=======================================
1116:   CGF.EmitBlock(ContBlock);
1117: 
1118:   Address ResAddr = emitMergePHI(CGF, RegAddr, InRegBlock, OnStackAddr,
1119:                                  OnStackBlock, "vaargs.addr");
1120: 
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1121-1140
```cpp
1121:   if (IsIndirect)
1122:     return CGF.EmitLoadOfAnyValue(
1123:         CGF.MakeAddrLValue(
1124:             Address(CGF.Builder.CreateLoad(ResAddr, "vaarg.addr"), ElementTy,
1125:                     TyAlign),
1126:             Ty),
1127:         Slot);
1128: 
1129:   return CGF.EmitLoadOfAnyValue(CGF.MakeAddrLValue(ResAddr, Ty), Slot);
1130: }
1131: 
1132: RValue AArch64ABIInfo::EmitDarwinVAArg(Address VAListAddr, QualType Ty,
1133:                                        CodeGenFunction &CGF,
1134:                                        AggValueSlot Slot) const {
1135:   // The backend's lowering doesn't support va_arg for aggregates or
1136:   // illegal vector types.  Lower VAArg here for these cases and use
1137:   // the LLVM va_arg instruction for everything else.
1138:   if (!isAggregateTypeForABI(Ty) && !isIllegalVectorType(Ty))
1139:     return CGF.EmitLoadOfAnyValue(
1140:         CGF.MakeAddrLValue(
```
- **EN**: This block defines callable entry points like `EmitDarwinVAArg`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `EmitDarwinVAArg`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1141-1160
```cpp
1141:             EmitVAArgInstr(CGF, VAListAddr, Ty, ABIArgInfo::getDirect()), Ty),
1142:         Slot);
1143: 
1144:   uint64_t PointerSize = getTarget().getPointerWidth(LangAS::Default) / 8;
1145:   CharUnits SlotSize = CharUnits::fromQuantity(PointerSize);
1146: 
1147:   // Empty records are ignored for parameter passing purposes.
1148:   if (isEmptyRecord(getContext(), Ty, true))
1149:     return Slot.asRValue();
1150: 
1151:   // The size of the actual thing passed, which might end up just
1152:   // being a pointer for indirect types.
1153:   auto TyInfo = getContext().getTypeInfoInChars(Ty);
1154: 
1155:   // Arguments bigger than 16 bytes which aren't homogeneous
1156:   // aggregates should be passed indirectly.
1157:   bool IsIndirect = false;
1158:   if (TyInfo.Width.getQuantity() > 16) {
1159:     const Type *Base = nullptr;
1160:     uint64_t Members = 0;
```
- **EN**: This block defines callable entry points like `EmitVAArgInstr`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `EmitVAArgInstr`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1161-1180
```cpp
1161:     IsIndirect = !isHomogeneousAggregate(Ty, Base, Members);
1162:   }
1163: 
1164:   return emitVoidPtrVAArg(CGF, VAListAddr, Ty, IsIndirect, TyInfo, SlotSize,
1165:                           /*AllowHigherAlign*/ true, Slot);
1166: }
1167: 
1168: RValue AArch64ABIInfo::EmitMSVAArg(CodeGenFunction &CGF, Address VAListAddr,
1169:                                    QualType Ty, AggValueSlot Slot) const {
1170:   bool IsIndirect = false;
1171: 
1172:   if (getTarget().getTriple().isWindowsArm64EC()) {
1173:     // MS x64 ABI requirement: "Any argument that doesn't fit in 8 bytes, or is
1174:     // not 1, 2, 4, or 8 bytes, must be passed by reference."
1175:     uint64_t Width = getContext().getTypeSize(Ty);
1176:     IsIndirect = Width > 64 || !llvm::isPowerOf2_64(Width);
1177:   } else {
1178:     // Composites larger than 16 bytes are passed by reference.
1179:     if (isAggregateTypeForABI(Ty) && getContext().getTypeSize(Ty) > 128)
1180:       IsIndirect = true;
```
- **EN**: This block defines callable entry points like `EmitMSVAArg`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `EmitMSVAArg`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1181-1200
```cpp
1181:   }
1182: 
1183:   return emitVoidPtrVAArg(CGF, VAListAddr, Ty, IsIndirect,
1184:                           CGF.getContext().getTypeInfoInChars(Ty),
1185:                           CharUnits::fromQuantity(8),
1186:                           /*allowHigherAlign*/ false, Slot);
1187: }
1188: 
1189: static bool isStreamingCompatible(const FunctionDecl *F) {
1190:   if (const auto *T = F->getType()->getAs<FunctionProtoType>())
1191:     return T->getAArch64SMEAttributes() &
1192:            FunctionType::SME_PStateSMCompatibleMask;
1193:   return false;
1194: }
1195: 
1196: // Report an error if an argument or return value of type Ty would need to be
1197: // passed in a floating-point register.
1198: static void diagnoseIfNeedsFPReg(DiagnosticsEngine &Diags,
1199:                                  const StringRef ABIName,
1200:                                  const AArch64ABIInfo &ABIInfo,
```
- **EN**: This block defines callable entry points like `isStreamingCompatible`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isStreamingCompatible`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1201-1220
```cpp
1201:                                  const QualType &Ty, const NamedDecl *D,
1202:                                  SourceLocation loc) {
1203:   const Type *HABase = nullptr;
1204:   uint64_t HAMembers = 0;
1205:   if (Ty->isFloatingType() || Ty->isVectorType() ||
1206:       ABIInfo.isHomogeneousAggregate(Ty, HABase, HAMembers)) {
1207:     Diags.Report(loc, diag::err_target_unsupported_type_for_abi)
1208:         << D->getDeclName() << Ty << ABIName;
1209:   }
1210: }
1211: 
1212: // If we are using a hard-float ABI, but do not have floating point registers,
1213: // then report an error for any function arguments or returns which would be
1214: // passed in floating-pint registers.
1215: void AArch64TargetCodeGenInfo::checkFunctionABI(
1216:     CodeGenModule &CGM, const FunctionDecl *FuncDecl) const {
1217:   const AArch64ABIInfo &ABIInfo = getABIInfo<AArch64ABIInfo>();
1218:   const TargetInfo &TI = ABIInfo.getContext().getTargetInfo();
1219: 
1220:   if (!TI.hasFeature("fp") && !ABIInfo.isSoftFloat()) {
```
- **EN**: This block defines callable entry points like `checkFunctionABI`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `checkFunctionABI`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1221-1240
```cpp
1221:     diagnoseIfNeedsFPReg(CGM.getDiags(), TI.getABI(), ABIInfo,
1222:                          FuncDecl->getReturnType(), FuncDecl,
1223:                          FuncDecl->getLocation());
1224:     for (ParmVarDecl *PVD : FuncDecl->parameters()) {
1225:       diagnoseIfNeedsFPReg(CGM.getDiags(), TI.getABI(), ABIInfo, PVD->getType(),
1226:                            PVD, FuncDecl->getLocation());
1227:     }
1228:   }
1229: }
1230: 
1231: enum class ArmSMEInlinability : uint8_t {
1232:   Ok = 0,
1233:   ErrorCalleeRequiresNewZA = 1 << 0,
1234:   ErrorCalleeRequiresNewZT0 = 1 << 1,
1235:   WarnIncompatibleStreamingModes = 1 << 2,
1236:   ErrorIncompatibleStreamingModes = 1 << 3,
1237: 
1238:   IncompatibleStreamingModes =
1239:       WarnIncompatibleStreamingModes | ErrorIncompatibleStreamingModes,
1240: 
```
- **EN**: This block introduces declarations such as `ArmSMEInlinability`; defines callable entry points like `diagnoseIfNeedsFPReg`; uses control flow (for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出诸如 `ArmSMEInlinability` 的声明；定义可调用入口，例如 `diagnoseIfNeedsFPReg`；通过控制流（for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1241-1260
```cpp
1241:   LLVM_MARK_AS_BITMASK_ENUM(/*LargestValue=*/ErrorIncompatibleStreamingModes),
1242: };
1243: 
1244: /// Determines if there are any Arm SME ABI issues with inlining \p Callee into
1245: /// \p Caller. Returns the issue (if any) in the ArmSMEInlinability bit enum.
1246: static ArmSMEInlinability GetArmSMEInlinability(const FunctionDecl *Caller,
1247:                                                 const FunctionDecl *Callee) {
1248:   bool CallerIsStreaming =
1249:       IsArmStreamingFunction(Caller, /*IncludeLocallyStreaming=*/true);
1250:   bool CalleeIsStreaming =
1251:       IsArmStreamingFunction(Callee, /*IncludeLocallyStreaming=*/true);
1252:   bool CallerIsStreamingCompatible = isStreamingCompatible(Caller);
1253:   bool CalleeIsStreamingCompatible = isStreamingCompatible(Callee);
1254: 
1255:   ArmSMEInlinability Inlinability = ArmSMEInlinability::Ok;
1256: 
1257:   if (!CalleeIsStreamingCompatible &&
1258:       (CallerIsStreaming != CalleeIsStreaming || CallerIsStreamingCompatible)) {
1259:     if (CalleeIsStreaming)
1260:       Inlinability |= ArmSMEInlinability::ErrorIncompatibleStreamingModes;
```
- **EN**: This block defines callable entry points like `GetArmSMEInlinability`, `IsArmStreamingFunction`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `GetArmSMEInlinability`, `IsArmStreamingFunction`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1261-1280
```cpp
1261:     else
1262:       Inlinability |= ArmSMEInlinability::WarnIncompatibleStreamingModes;
1263:   }
1264:   if (auto *NewAttr = Callee->getAttr<ArmNewAttr>()) {
1265:     if (NewAttr->isNewZA())
1266:       Inlinability |= ArmSMEInlinability::ErrorCalleeRequiresNewZA;
1267:     if (NewAttr->isNewZT0())
1268:       Inlinability |= ArmSMEInlinability::ErrorCalleeRequiresNewZT0;
1269:   }
1270: 
1271:   return Inlinability;
1272: }
1273: 
1274: void AArch64TargetCodeGenInfo::checkFunctionCallABIStreaming(
1275:     CodeGenModule &CGM, SourceLocation CallLoc, const FunctionDecl *Caller,
1276:     const FunctionDecl *Callee) const {
1277:   if (!Caller || !Callee || !Callee->hasAttr<AlwaysInlineAttr>())
1278:     return;
1279: 
1280:   ArmSMEInlinability Inlinability = GetArmSMEInlinability(Caller, Callee);
```
- **EN**: This block defines callable entry points like `checkFunctionCallABIStreaming`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `checkFunctionCallABIStreaming`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1281-1300
```cpp
1281: 
1282:   if ((Inlinability & ArmSMEInlinability::IncompatibleStreamingModes) !=
1283:       ArmSMEInlinability::Ok)
1284:     CGM.getDiags().Report(
1285:         CallLoc,
1286:         (Inlinability & ArmSMEInlinability::ErrorIncompatibleStreamingModes) ==
1287:                 ArmSMEInlinability::ErrorIncompatibleStreamingModes
1288:             ? diag::err_function_always_inline_attribute_mismatch
1289:             : diag::warn_function_always_inline_attribute_mismatch)
1290:         << Caller->getDeclName() << Callee->getDeclName() << "streaming";
1291: 
1292:   if ((Inlinability & ArmSMEInlinability::ErrorCalleeRequiresNewZA) ==
1293:       ArmSMEInlinability::ErrorCalleeRequiresNewZA)
1294:     CGM.getDiags().Report(CallLoc, diag::err_function_always_inline_new_za)
1295:         << Callee->getDeclName();
1296: 
1297:   if ((Inlinability & ArmSMEInlinability::ErrorCalleeRequiresNewZT0) ==
1298:       ArmSMEInlinability::ErrorCalleeRequiresNewZT0)
1299:     CGM.getDiags().Report(CallLoc, diag::err_function_always_inline_new_zt0)
1300:         << Callee->getDeclName();
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1301-1320
```cpp
1301: }
1302: 
1303: // If the target does not have floating-point registers, but we are using a
1304: // hard-float ABI, there is no way to pass floating-point, vector or HFA values
1305: // to functions, so we report an error.
1306: void AArch64TargetCodeGenInfo::checkFunctionCallABISoftFloat(
1307:     CodeGenModule &CGM, SourceLocation CallLoc, const FunctionDecl *Caller,
1308:     const FunctionDecl *Callee, const CallArgList &Args,
1309:     QualType ReturnType) const {
1310:   const AArch64ABIInfo &ABIInfo = getABIInfo<AArch64ABIInfo>();
1311:   const TargetInfo &TI = ABIInfo.getContext().getTargetInfo();
1312: 
1313:   if (!Caller || TI.hasFeature("fp") || ABIInfo.isSoftFloat())
1314:     return;
1315: 
1316:   diagnoseIfNeedsFPReg(CGM.getDiags(), TI.getABI(), ABIInfo, ReturnType,
1317:                        Callee ? Callee : Caller, CallLoc);
1318: 
1319:   for (const CallArg &Arg : Args)
1320:     diagnoseIfNeedsFPReg(CGM.getDiags(), TI.getABI(), ABIInfo, Arg.getType(),
```
- **EN**: This block defines callable entry points like `checkFunctionCallABISoftFloat`, `diagnoseIfNeedsFPReg`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `checkFunctionCallABISoftFloat`, `diagnoseIfNeedsFPReg`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1321-1340
```cpp
1321:                          Callee ? Callee : Caller, CallLoc);
1322: }
1323: 
1324: void AArch64TargetCodeGenInfo::checkFunctionCallABI(CodeGenModule &CGM,
1325:                                                     SourceLocation CallLoc,
1326:                                                     const FunctionDecl *Caller,
1327:                                                     const FunctionDecl *Callee,
1328:                                                     const CallArgList &Args,
1329:                                                     QualType ReturnType) const {
1330:   checkFunctionCallABIStreaming(CGM, CallLoc, Caller, Callee);
1331:   checkFunctionCallABISoftFloat(CGM, CallLoc, Caller, Callee, Args, ReturnType);
1332: }
1333: 
1334: bool AArch64TargetCodeGenInfo::wouldInliningViolateFunctionCallABI(
1335:     const FunctionDecl *Caller, const FunctionDecl *Callee) const {
1336:   return Caller && Callee &&
1337:          GetArmSMEInlinability(Caller, Callee) != ArmSMEInlinability::Ok;
1338: }
1339: 
1340: void AArch64ABIInfo::appendAttributeMangling(TargetClonesAttr *Attr,
```
- **EN**: This block defines callable entry points like `checkFunctionCallABI`, `checkFunctionCallABIStreaming`, `checkFunctionCallABISoftFloat`, `wouldInliningViolateFunctionCallABI`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `checkFunctionCallABI`, `checkFunctionCallABIStreaming`, `checkFunctionCallABISoftFloat`, `wouldInliningViolateFunctionCallABI`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 1341-1360
```cpp
1341:                                              unsigned Index,
1342:                                              raw_ostream &Out) const {
1343:   appendAttributeMangling(Attr->getFeatureStr(Index), Out);
1344: }
1345: 
1346: void AArch64ABIInfo::appendAttributeMangling(StringRef AttrStr,
1347:                                              raw_ostream &Out) const {
1348:   if (AttrStr == "default") {
1349:     Out << ".default";
1350:     return;
1351:   }
1352: 
1353:   Out << "._";
1354:   SmallVector<StringRef, 8> Features;
1355:   AttrStr.split(Features, "+");
1356:   for (auto &Feat : Features)
1357:     Feat = Feat.trim();
1358: 
1359:   llvm::sort(Features, [](const StringRef LHS, const StringRef RHS) {
1360:     return LHS.compare(RHS) < 0;
```
- **EN**: This block defines callable entry points like `appendAttributeMangling`, `sort`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `appendAttributeMangling`, `sort`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1361-1380
```cpp
1361:   });
1362: 
1363:   llvm::SmallDenseSet<StringRef, 8> UniqueFeats;
1364:   for (auto &Feat : Features)
1365:     if (getTarget().doesFeatureAffectCodeGen(Feat))
1366:       if (auto Ext = llvm::AArch64::parseFMVExtension(Feat))
1367:         if (UniqueFeats.insert(Ext->Name).second)
1368:           Out << 'M' << Ext->Name;
1369: }
1370: 
1371: std::unique_ptr<TargetCodeGenInfo>
1372: CodeGen::createAArch64TargetCodeGenInfo(CodeGenModule &CGM,
1373:                                         AArch64ABIKind Kind) {
1374:   return std::make_unique<AArch64TargetCodeGenInfo>(CGM, Kind);
1375: }
1376: 
1377: std::unique_ptr<TargetCodeGenInfo>
1378: CodeGen::createWindowsAArch64TargetCodeGenInfo(CodeGenModule &CGM,
1379:                                                AArch64ABIKind K) {
1380:   return std::make_unique<WindowsAArch64TargetCodeGenInfo>(CGM, K);
```
- **EN**: This block defines callable entry points like `createAArch64TargetCodeGenInfo`, `createWindowsAArch64TargetCodeGenInfo`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `createAArch64TargetCodeGenInfo`, `createWindowsAArch64TargetCodeGenInfo`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1381-1381
```cpp
1381: }
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target-specific ABI and code generation implementation.
- **CN**: 该代码块为周围的 目标相关的 ABI 与代码生成 实现提供必要的胶水代码、布局或分隔结构。

## Key Concepts / 关键概念

- **CGF**: Central symbol in this file's implementation of target-specific ABI and code generation. / 是该文件实现 目标相关的 ABI 与代码生成 时的核心符号。
- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGM**: Central symbol in this file's implementation of target-specific ABI and code generation. / 是该文件实现 目标相关的 ABI 与代码生成 时的核心符号。
- **QualType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ABIArgInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **NSRN**: Central symbol in this file's implementation of target-specific ABI and code generation. / 是该文件实现 目标相关的 ABI 与代码生成 时的核心符号。
- **getContext**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **BuiltinType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfoImpl.h`, `TargetInfo.h`
- **Clang libraries / Clang 库**: `clang/AST/Decl.h`, `clang/Basic/DiagnosticFrontend.h`, `clang/Basic/AArch64ACLETypes.def`
- **LLVM libraries / LLVM 库**: `llvm/TargetParser/AArch64TargetParser.h`
