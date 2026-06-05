# CSKY.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/Targets/CSKY.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements target-specific ABI rules and lowering details for CSKY.
- **Purpose (CN) / 目的（中文）**: 实现 CSKY 目标相关的 ABI 规则与降级细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: //===- CSKY.cpp -----------------------------------------------------------===//
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
```
- **EN**: This block imports local CodeGen headers `ABIInfoImpl.h`, `TargetInfo.h`; opens or references namespaces `clang`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `ABIInfoImpl.h`, `TargetInfo.h`；打开或引用命名空间 `clang`；包含影响本编译单元构建方式的预处理结构。

### Lines 13-24
```cpp
13: using namespace clang::CodeGen;
14: 
15: //===----------------------------------------------------------------------===//
16: // CSKY ABI Implementation
17: //===----------------------------------------------------------------------===//
18: namespace {
19: class CSKYABIInfo : public DefaultABIInfo {
20:   static const int NumArgGPRs = 4;
21:   static const int NumArgFPRs = 4;
22: 
23:   static const unsigned XLen = 32;
24:   unsigned FLen;
```
- **EN**: This block opens or references namespaces `clang`; introduces declarations such as `CSKYABIInfo`.
- **CN**: 该代码块打开或引用命名空间 `clang`；给出诸如 `CSKYABIInfo` 的声明。

### Lines 25-36
```cpp
25: 
26: public:
27:   CSKYABIInfo(CodeGen::CodeGenTypes &CGT, unsigned FLen)
28:       : DefaultABIInfo(CGT), FLen(FLen) {}
29: 
30:   void computeInfo(CGFunctionInfo &FI) const override;
31:   ABIArgInfo classifyArgumentType(QualType Ty, int &ArgGPRsLeft,
32:                                   int &ArgFPRsLeft,
33:                                   bool isReturnType = false) const;
34:   ABIArgInfo classifyReturnType(QualType RetTy) const;
35: 
36:   RValue EmitVAArg(CodeGenFunction &CGF, Address VAListAddr, QualType Ty,
```
- **EN**: This block defines callable entry points like `CSKYABIInfo`, `computeInfo`, `classifyArgumentType`, `classifyReturnType`.
- **CN**: 该代码块定义可调用入口，例如 `CSKYABIInfo`, `computeInfo`, `classifyArgumentType`, `classifyReturnType`。

### Lines 37-48
```cpp
37:                    AggValueSlot Slot) const override;
38: };
39: 
40: } // end anonymous namespace
41: 
42: void CSKYABIInfo::computeInfo(CGFunctionInfo &FI) const {
43:   QualType RetTy = FI.getReturnType();
44:   if (!getCXXABI().classifyReturnType(FI))
45:     FI.getReturnInfo() = classifyReturnType(RetTy);
46: 
47:   bool IsRetIndirect = FI.getReturnInfo().getKind() == ABIArgInfo::Indirect;
48: 
```
- **EN**: This block opens or references namespaces `void`; defines callable entry points like `computeInfo`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块打开或引用命名空间 `void`；定义可调用入口，例如 `computeInfo`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 49-60
```cpp
49:   // We must track the number of GPRs used in order to conform to the CSKY
50:   // ABI, as integer scalars passed in registers should have signext/zeroext
51:   // when promoted.
52:   int ArgGPRsLeft = IsRetIndirect ? NumArgGPRs - 1 : NumArgGPRs;
53:   int ArgFPRsLeft = FLen ? NumArgFPRs : 0;
54: 
55:   for (auto &ArgInfo : FI.arguments()) {
56:     ArgInfo.info = classifyArgumentType(ArgInfo.type, ArgGPRsLeft, ArgFPRsLeft);
57:   }
58: }
59: 
60: RValue CSKYABIInfo::EmitVAArg(CodeGenFunction &CGF, Address VAListAddr,
```
- **EN**: This block uses control flow (for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 61-72
```cpp
61:                               QualType Ty, AggValueSlot Slot) const {
62:   CharUnits SlotSize = CharUnits::fromQuantity(XLen / 8);
63: 
64:   // Empty records are ignored for parameter passing purposes.
65:   if (isEmptyRecord(getContext(), Ty, true))
66:     return Slot.asRValue();
67: 
68:   auto TInfo = getContext().getTypeInfoInChars(Ty);
69: 
70:   return emitVoidPtrVAArg(CGF, VAListAddr, Ty, false, TInfo, SlotSize,
71:                           /*AllowHigherAlign=*/true, Slot);
72: }
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 73-84
```cpp
73: 
74: ABIArgInfo CSKYABIInfo::classifyArgumentType(QualType Ty, int &ArgGPRsLeft,
75:                                              int &ArgFPRsLeft,
76:                                              bool isReturnType) const {
77:   assert(ArgGPRsLeft <= NumArgGPRs && "Arg GPR tracking underflow");
78:   Ty = useFirstFieldIfTransparentUnion(Ty);
79: 
80:   // Structures with either a non-trivial destructor or a non-trivial
81:   // copy constructor are always passed indirectly.
82:   if (CGCXXABI::RecordArgABI RAA = getRecordArgABI(Ty, getCXXABI())) {
83:     if (ArgGPRsLeft)
84:       ArgGPRsLeft -= 1;
```
- **EN**: This block defines callable entry points like `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 85-96
```cpp
85:     return getNaturalAlignIndirect(
86:         Ty, /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
87:         /*ByVal=*/RAA == CGCXXABI::RAA_DirectInMemory);
88:   }
89: 
90:   // Ignore empty structs/unions.
91:   if (isEmptyRecord(getContext(), Ty, true))
92:     return ABIArgInfo::getIgnore();
93: 
94:   if (!Ty->isUnionType())
95:     if (const Type *SeltTy = isSingleElementStruct(Ty, getContext()))
96:       return ABIArgInfo::getDirect(CGT.ConvertType(QualType(SeltTy, 0)));
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 97-108
```cpp
 97: 
 98:   uint64_t Size = getContext().getTypeSize(Ty);
 99:   // Pass floating point values via FPRs if possible.
100:   if (Ty->isFloatingType() && !Ty->isComplexType() && FLen >= Size &&
101:       ArgFPRsLeft) {
102:     ArgFPRsLeft--;
103:     return ABIArgInfo::getDirect();
104:   }
105: 
106:   // Complex types for the hard float ABI must be passed direct rather than
107:   // using CoerceAndExpand.
108:   if (Ty->isComplexType() && FLen && !isReturnType) {
```
- **EN**: This block defines callable entry points like `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 109-120
```cpp
109:     QualType EltTy = Ty->castAs<ComplexType>()->getElementType();
110:     if (getContext().getTypeSize(EltTy) <= FLen) {
111:       ArgFPRsLeft -= 2;
112:       return ABIArgInfo::getDirect();
113:     }
114:   }
115: 
116:   if (!isAggregateTypeForABI(Ty)) {
117:     // Treat an enum type as its underlying type.
118:     if (const auto *ED = Ty->getAsEnumDecl())
119:       Ty = ED->getIntegerType();
120: 
```
- **EN**: This block defines callable entry points like `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 121-132
```cpp
121:     // All integral types are promoted to XLen width, unless passed on the
122:     // stack.
123:     if (Size < XLen && Ty->isIntegralOrEnumerationType())
124:       return ABIArgInfo::getExtend(Ty);
125: 
126:     if (const auto *EIT = Ty->getAs<BitIntType>()) {
127:       if (EIT->getNumBits() < XLen)
128:         return ABIArgInfo::getExtend(Ty);
129:     }
130: 
131:     return ABIArgInfo::getDirect();
132:   }
```
- **EN**: This block defines callable entry points like `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 133-144
```cpp
133: 
134:   // For argument type, the first 4*XLen parts of aggregate will be passed
135:   // in registers, and the rest will be passed in stack.
136:   // So we can coerce to integers directly and let backend handle it correctly.
137:   // For return type, aggregate which <= 2*XLen will be returned in registers.
138:   // Otherwise, aggregate will be returned indirectly.
139:   if (!isReturnType || (isReturnType && Size <= 2 * XLen)) {
140:     if (Size <= XLen) {
141:       return ABIArgInfo::getDirect(
142:           llvm::IntegerType::get(getVMContext(), XLen));
143:     } else {
144:       return ABIArgInfo::getDirect(llvm::ArrayType::get(
```
- **EN**: This block defines callable entry points like `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 145-156
```cpp
145:           llvm::IntegerType::get(getVMContext(), XLen), (Size + 31) / XLen));
146:     }
147:   }
148:   return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
149:                                  /*ByVal=*/false);
150: }
151: 
152: ABIArgInfo CSKYABIInfo::classifyReturnType(QualType RetTy) const {
153:   if (RetTy->isVoidType())
154:     return ABIArgInfo::getIgnore();
155: 
156:   int ArgGPRsLeft = 2;
```
- **EN**: This block defines callable entry points like `get`, `classifyReturnType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `get`, `classifyReturnType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 157-168
```cpp
157:   int ArgFPRsLeft = FLen ? 1 : 0;
158: 
159:   // The rules for return and argument types are the same, so defer to
160:   // classifyArgumentType.
161:   return classifyArgumentType(RetTy, ArgGPRsLeft, ArgFPRsLeft, true);
162: }
163: 
164: namespace {
165: class CSKYTargetCodeGenInfo : public TargetCodeGenInfo {
166: public:
167:   CSKYTargetCodeGenInfo(CodeGen::CodeGenTypes &CGT, unsigned FLen)
168:       : TargetCodeGenInfo(std::make_unique<CSKYABIInfo>(CGT, FLen)) {}
```
- **EN**: This block introduces declarations such as `CSKYTargetCodeGenInfo`; defines callable entry points like `classifyArgumentType`, `CSKYTargetCodeGenInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块给出诸如 `CSKYTargetCodeGenInfo` 的声明；定义可调用入口，例如 `classifyArgumentType`, `CSKYTargetCodeGenInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 169-175
```cpp
169: };
170: } // end anonymous namespace
171: 
172: std::unique_ptr<TargetCodeGenInfo>
173: CodeGen::createCSKYTargetCodeGenInfo(CodeGenModule &CGM, unsigned FLen) {
174:   return std::make_unique<CSKYTargetCodeGenInfo>(CGM.getTypes(), FLen);
175: }
```
- **EN**: This block opens or references namespaces `std`; defines callable entry points like `createCSKYTargetCodeGenInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块打开或引用命名空间 `std`；定义可调用入口，例如 `createCSKYTargetCodeGenInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

## Key Concepts / 关键概念

- **ABIArgInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **FLen**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **XLen**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **QualType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ArgGPRsLeft**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ArgFPRsLeft**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CSKYABIInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **RetTy**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfoImpl.h`, `TargetInfo.h`
