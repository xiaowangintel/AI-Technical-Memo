# ABIInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/ABIInfo.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Defines ABI classification interfaces for argument, return, and calling-convention lowering.
- **Purpose (CN) / 目的（中文）**: 定义参数、返回值与调用约定降级所需的 ABI 分类接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: //===- ABIInfo.cpp --------------------------------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: 
 9: #include "ABIInfo.h"
10: #include "ABIInfoImpl.h"
11: 
12: using namespace clang;
13: using namespace clang::CodeGen;
14: 
15: // Pin the vtable to this file.
16: ABIInfo::~ABIInfo() = default;
```
- **EN**: This block imports local CodeGen headers `ABIInfo.h`, `ABIInfoImpl.h`; opens or references namespaces `clang`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `ABIInfo.h`, `ABIInfoImpl.h`；打开或引用命名空间 `clang`；包含影响本编译单元构建方式的预处理结构。

### Lines 17-32
```cpp
17: 
18: CGCXXABI &ABIInfo::getCXXABI() const { return CGT.getCXXABI(); }
19: 
20: ASTContext &ABIInfo::getContext() const { return CGT.getContext(); }
21: 
22: llvm::LLVMContext &ABIInfo::getVMContext() const {
23:   return CGT.getLLVMContext();
24: }
25: 
26: const llvm::DataLayout &ABIInfo::getDataLayout() const {
27:   return CGT.getDataLayout();
28: }
29: 
30: const TargetInfo &ABIInfo::getTarget() const { return CGT.getTarget(); }
31: 
32: const CodeGenOptions &ABIInfo::getCodeGenOpts() const {
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding ABI lowering implementation.
- **CN**: 该代码块为周围的 ABI 降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 33-48
```cpp
33:   return CGT.getCodeGenOpts();
34: }
35: 
36: bool ABIInfo::isAndroid() const { return getTarget().getTriple().isAndroid(); }
37: 
38: bool ABIInfo::isOHOSFamily() const {
39:   return getTarget().getTriple().isOHOSFamily();
40: }
41: 
42: RValue ABIInfo::EmitMSVAArg(CodeGenFunction &CGF, Address VAListAddr,
43:                             QualType Ty, AggValueSlot Slot) const {
44:   return RValue::getIgnored();
45: }
46: 
47: bool ABIInfo::isHomogeneousAggregateBaseType(QualType Ty) const {
48:   return false;
```
- **EN**: This block defines callable entry points like `isAndroid`, `isOHOSFamily`, `getTarget`, `EmitMSVAArg`, `getIgnored`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `isAndroid`, `isOHOSFamily`, `getTarget`, `EmitMSVAArg`, `getIgnored`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 49-64
```cpp
49: }
50: 
51: bool ABIInfo::isHomogeneousAggregateSmallEnough(const Type *Base,
52:                                                 uint64_t Members) const {
53:   return false;
54: }
55: 
56: bool ABIInfo::isZeroLengthBitfieldPermittedInHomogeneousAggregate() const {
57:   // For compatibility with GCC, ignore empty bitfields in C++ mode.
58:   return getContext().getLangOpts().CPlusPlus;
59: }
60: 
61: bool ABIInfo::isHomogeneousAggregate(QualType Ty, const Type *&Base,
62:                                      uint64_t &Members) const {
63:   if (const ConstantArrayType *AT = getContext().getAsConstantArrayType(Ty)) {
64:     uint64_t NElements = AT->getZExtSize();
```
- **EN**: This block defines callable entry points like `isHomogeneousAggregateSmallEnough`, `isZeroLengthBitfieldPermittedInHomogeneousAggregate`, `isHomogeneousAggregate`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `isHomogeneousAggregateSmallEnough`, `isZeroLengthBitfieldPermittedInHomogeneousAggregate`, `isHomogeneousAggregate`；通过控制流（if）细化 ABI 降级 行为。

### Lines 65-80
```cpp
65:     if (NElements == 0)
66:       return false;
67:     if (!isHomogeneousAggregate(AT->getElementType(), Base, Members))
68:       return false;
69:     Members *= NElements;
70:   } else if (const auto *RD = Ty->getAsRecordDecl()) {
71:     if (RD->hasFlexibleArrayMember())
72:       return false;
73: 
74:     Members = 0;
75: 
76:     // If this is a C++ record, check the properties of the record such as
77:     // bases and ABI specific restrictions
78:     if (const CXXRecordDecl *CXXRD = dyn_cast<CXXRecordDecl>(RD)) {
79:       if (!getCXXABI().isPermittedToBeHomogeneousAggregate(CXXRD))
80:         return false;
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 81-96
```cpp
81: 
82:       for (const auto &I : CXXRD->bases()) {
83:         // Ignore empty records.
84:         if (isEmptyRecord(getContext(), I.getType(), true))
85:           continue;
86: 
87:         uint64_t FldMembers;
88:         if (!isHomogeneousAggregate(I.getType(), Base, FldMembers))
89:           return false;
90: 
91:         Members += FldMembers;
92:       }
93:     }
94: 
95:     for (const auto *FD : RD->fields()) {
96:       // Ignore (non-zero arrays of) empty records.
```
- **EN**: This block uses control flow (if, for) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if, for）细化 ABI 降级 行为。

### Lines 97-112
```cpp
 97:       QualType FT = FD->getType();
 98:       while (const ConstantArrayType *AT =
 99:              getContext().getAsConstantArrayType(FT)) {
100:         if (AT->isZeroSize())
101:           return false;
102:         FT = AT->getElementType();
103:       }
104:       if (isEmptyRecord(getContext(), FT, true))
105:         continue;
106: 
107:       if (isZeroLengthBitfieldPermittedInHomogeneousAggregate() &&
108:           FD->isZeroLengthBitField())
109:         continue;
110: 
111:       uint64_t FldMembers;
112:       if (!isHomogeneousAggregate(FD->getType(), Base, FldMembers))
```
- **EN**: This block uses control flow (if, while) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if, while）细化 ABI 降级 行为。

### Lines 113-128
```cpp
113:         return false;
114: 
115:       Members = (RD->isUnion() ?
116:                  std::max(Members, FldMembers) : Members + FldMembers);
117:     }
118: 
119:     if (!Base)
120:       return false;
121: 
122:     // Ensure there is no padding.
123:     if (getContext().getTypeSize(Base) * Members !=
124:         getContext().getTypeSize(Ty))
125:       return false;
126:   } else {
127:     Members = 1;
128:     if (const ComplexType *CT = Ty->getAs<ComplexType>()) {
```
- **EN**: This block defines callable entry points like `max`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `max`；通过控制流（if）细化 ABI 降级 行为。

### Lines 129-144
```cpp
129:       Members = 2;
130:       Ty = CT->getElementType();
131:     }
132: 
133:     // Most ABIs only support float, double, and some vector type widths.
134:     if (!isHomogeneousAggregateBaseType(Ty))
135:       return false;
136: 
137:     // The base type must be the same for all members.  Types that
138:     // agree in both total size and mode (float vs. vector) are
139:     // treated as being equivalent here.
140:     const Type *TyPtr = Ty.getTypePtr();
141:     if (!Base) {
142:       Base = TyPtr;
143:       // If it's a non-power-of-2 vector, its size is already a power-of-2,
144:       // so make sure to widen it explicitly.
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 145-160
```cpp
145:       if (const VectorType *VT = Base->getAs<VectorType>()) {
146:         QualType EltTy = VT->getElementType();
147:         unsigned NumElements =
148:             getContext().getTypeSize(VT) / getContext().getTypeSize(EltTy);
149:         Base = getContext()
150:                    .getVectorType(EltTy, NumElements, VT->getVectorKind())
151:                    .getTypePtr();
152:       }
153:     }
154: 
155:     if (Base->isVectorType() != TyPtr->isVectorType() ||
156:         getContext().getTypeSize(Base) != getContext().getTypeSize(TyPtr))
157:       return false;
158:   }
159:   return Members > 0 && isHomogeneousAggregateSmallEnough(Base, Members);
160: }
```
- **EN**: This block defines callable entry points like `getContext`, `isHomogeneousAggregateSmallEnough`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getContext`, `isHomogeneousAggregateSmallEnough`；通过控制流（if）细化 ABI 降级 行为。

### Lines 161-176
```cpp
161: 
162: bool ABIInfo::isPromotableIntegerTypeForABI(QualType Ty) const {
163:   if (getContext().isPromotableIntegerType(Ty))
164:     return true;
165: 
166:   if (const auto *EIT = Ty->getAs<BitIntType>())
167:     if (EIT->getNumBits() < getContext().getTypeSize(getContext().IntTy))
168:       return true;
169: 
170:   return false;
171: }
172: 
173: ABIArgInfo ABIInfo::getNaturalAlignIndirect(QualType Ty, unsigned AddrSpace,
174:                                             bool ByVal, bool Realign,
175:                                             llvm::Type *Padding) const {
176:   return ABIArgInfo::getIndirect(getContext().getTypeAlignInChars(Ty),
```
- **EN**: This block defines callable entry points like `isPromotableIntegerTypeForABI`, `getNaturalAlignIndirect`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `isPromotableIntegerTypeForABI`, `getNaturalAlignIndirect`；通过控制流（if）细化 ABI 降级 行为。

### Lines 177-192
```cpp
177:                                  AddrSpace, ByVal, Realign, Padding);
178: }
179: 
180: ABIArgInfo ABIInfo::getNaturalAlignIndirectInReg(QualType Ty,
181:                                                  bool Realign) const {
182:   return ABIArgInfo::getIndirectInReg(getContext().getTypeAlignInChars(Ty),
183:                                       /*ByVal*/ false, Realign);
184: }
185: 
186: void ABIInfo::appendAttributeMangling(TargetAttr *Attr,
187:                                       raw_ostream &Out) const {
188:   if (Attr->isDefaultVersion())
189:     return;
190:   appendAttributeMangling(Attr->getFeaturesStr(), Out);
191: }
192: 
```
- **EN**: This block defines callable entry points like `getNaturalAlignIndirectInReg`, `appendAttributeMangling`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getNaturalAlignIndirectInReg`, `appendAttributeMangling`；通过控制流（if）细化 ABI 降级 行为。

### Lines 193-208
```cpp
193: void ABIInfo::appendAttributeMangling(TargetVersionAttr *Attr,
194:                                       raw_ostream &Out) const {
195:   appendAttributeMangling(Attr->getNamesStr(), Out);
196: }
197: 
198: void ABIInfo::appendAttributeMangling(TargetClonesAttr *Attr, unsigned Index,
199:                                       raw_ostream &Out) const {
200:   appendAttributeMangling(Attr->getFeatureStr(Index), Out);
201:   Out << '.' << Attr->getMangledIndex(Index);
202: }
203: 
204: void ABIInfo::appendAttributeMangling(StringRef AttrStr,
205:                                       raw_ostream &Out) const {
206:   if (AttrStr == "default") {
207:     Out << ".default";
208:     return;
```
- **EN**: This block defines callable entry points like `appendAttributeMangling`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `appendAttributeMangling`；通过控制流（if）细化 ABI 降级 行为。

### Lines 209-224
```cpp
209:   }
210: 
211:   Out << '.';
212:   const TargetInfo &TI = CGT.getTarget();
213:   ParsedTargetAttr Info = TI.parseTargetAttr(AttrStr);
214: 
215:   llvm::sort(Info.Features, [&TI](StringRef LHS, StringRef RHS) {
216:     // Multiversioning doesn't allow "no-${feature}", so we can
217:     // only have "+" prefixes here.
218:     assert(LHS.starts_with("+") && RHS.starts_with("+") &&
219:            "Features should always have a prefix.");
220:     return TI.getFMVPriority({LHS.substr(1)})
221:         .ugt(TI.getFMVPriority({RHS.substr(1)}));
222:   });
223: 
224:   bool IsFirst = true;
```
- **EN**: This block defines callable entry points like `sort`; returns or forwards computed values for the surrounding ABI lowering logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `sort`；为周围的 ABI 降级 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 225-240
```cpp
225:   if (!Info.CPU.empty()) {
226:     IsFirst = false;
227:     Out << "arch_" << Info.CPU;
228:   }
229: 
230:   for (StringRef Feat : Info.Features) {
231:     if (!IsFirst)
232:       Out << '_';
233:     IsFirst = false;
234:     Out << Feat.substr(1);
235:   }
236: }
237: 
238: llvm::FixedVectorType *
239: ABIInfo::getOptimalVectorMemoryType(llvm::FixedVectorType *T,
240:                                     const LangOptions &Opt) const {
```
- **EN**: This block defines callable entry points like `getOptimalVectorMemoryType`; uses control flow (if, for) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getOptimalVectorMemoryType`；通过控制流（if, for）细化 ABI 降级 行为。

### Lines 241-256
```cpp
241:   if (T->getNumElements() == 3 && !Opt.PreserveVec3Type)
242:     return llvm::FixedVectorType::get(T->getElementType(), 4);
243:   return T;
244: }
245: 
246: llvm::Value *ABIInfo::createCoercedLoad(Address SrcAddr, const ABIArgInfo &AI,
247:                                         CodeGenFunction &CGF) const {
248:   return nullptr;
249: }
250: 
251: void ABIInfo::createCoercedStore(llvm::Value *Val, Address DstAddr,
252:                                  const ABIArgInfo &AI, bool DestIsVolatile,
253:                                  CodeGenFunction &CGF) const {}
254: 
255: ABIArgInfo ABIInfo::classifyArgForArm64ECVarArg(QualType Ty) const {
256:   llvm_unreachable("Only implemented for x86");
```
- **EN**: This block defines callable entry points like `createCoercedStore`, `classifyArgForArm64ECVarArg`; uses control flow (if, for) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `createCoercedStore`, `classifyArgForArm64ECVarArg`；通过控制流（if, for）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 257-272
```cpp
257: }
258: 
259: // Pin the vtable to this file.
260: SwiftABIInfo::~SwiftABIInfo() = default;
261: 
262: /// Does the given lowering require more than the given number of
263: /// registers when expanded?
264: ///
265: /// This is intended to be the basis of a reasonable basic implementation
266: /// of should{Pass,Return}Indirectly.
267: ///
268: /// For most targets, a limit of four total registers is reasonable; this
269: /// limits the amount of code required in order to move around the value
270: /// in case it wasn't produced immediately prior to the call by the caller
271: /// (or wasn't produced in exactly the right registers) or isn't used
272: /// immediately within the callee.  But some targets may need to further
```
- **EN**: This block documents intent or context for the surrounding ABI lowering code.
- **CN**: 该代码块说明周围 ABI 降级 代码的意图或上下文。

### Lines 273-288
```cpp
273: /// limit the register count due to an inability to support that many
274: /// return registers.
275: bool SwiftABIInfo::occupiesMoreThan(ArrayRef<llvm::Type *> scalarTypes,
276:                                     unsigned maxAllRegisters) const {
277:   unsigned intCount = 0, fpCount = 0;
278:   for (llvm::Type *type : scalarTypes) {
279:     if (type->isPointerTy()) {
280:       intCount++;
281:     } else if (auto intTy = dyn_cast<llvm::IntegerType>(type)) {
282:       auto ptrWidth = CGT.getTarget().getPointerWidth(LangAS::Default);
283:       intCount += (intTy->getBitWidth() + ptrWidth - 1) / ptrWidth;
284:     } else {
285:       assert(type->isVectorTy() || type->isFloatingPointTy());
286:       fpCount++;
287:     }
288:   }
```
- **EN**: This block defines callable entry points like `occupiesMoreThan`; uses control flow (if, for) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `occupiesMoreThan`；通过控制流（if, for）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 289-303
```cpp
289: 
290:   return (intCount + fpCount > maxAllRegisters);
291: }
292: 
293: bool SwiftABIInfo::shouldPassIndirectly(ArrayRef<llvm::Type *> ComponentTys,
294:                                         bool AsReturnValue) const {
295:   return occupiesMoreThan(ComponentTys, /*total=*/4);
296: }
297: 
298: bool SwiftABIInfo::isLegalVectorType(CharUnits VectorSize, llvm::Type *EltTy,
299:                                      unsigned NumElts) const {
300:   // The default implementation of this assumes that the target guarantees
301:   // 128-bit SIMD support but nothing more.
302:   return (VectorSize.getQuantity() > 8 && VectorSize.getQuantity() <= 16);
303: }
```
- **EN**: This block defines callable entry points like `shouldPassIndirectly`, `occupiesMoreThan`, `isLegalVectorType`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `shouldPassIndirectly`, `occupiesMoreThan`, `isLegalVectorType`；为周围的 ABI 降级 逻辑返回或转发计算结果。

## Key Concepts / 关键概念

- **getContext**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Base**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Members**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Out**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **QualType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGT**: Central symbol in this file's implementation of ABI lowering. / 是该文件实现 ABI 降级 时的核心符号。
- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Attr**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfo.h`, `ABIInfoImpl.h`
