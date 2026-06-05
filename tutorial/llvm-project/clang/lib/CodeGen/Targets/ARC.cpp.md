# ARC.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/Targets/ARC.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements target-specific ABI rules and lowering details for ARC.
- **Purpose (CN) / 目的（中文）**: 实现 ARC 目标相关的 ABI 规则与降级细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: //===- ARC.cpp ------------------------------------------------------------===//
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
15: // ARC ABI implementation.
16: namespace {
17: 
18: class ARCABIInfo : public DefaultABIInfo {
19:   struct CCState {
20:     unsigned FreeRegs;
21:   };
22: 
23: public:
24:   using DefaultABIInfo::DefaultABIInfo;
```
- **EN**: This block opens or references namespaces `clang`; introduces declarations such as `ARCABIInfo`, `CCState`.
- **CN**: 该代码块打开或引用命名空间 `clang`；给出诸如 `ARCABIInfo`, `CCState` 的声明。

### Lines 25-36
```cpp
25: 
26: private:
27:   RValue EmitVAArg(CodeGenFunction &CGF, Address VAListAddr, QualType Ty,
28:                    AggValueSlot Slot) const override;
29: 
30:   void updateState(const ABIArgInfo &Info, QualType Ty, CCState &State) const {
31:     if (!State.FreeRegs)
32:       return;
33:     if (Info.isIndirect() && Info.getInReg())
34:       State.FreeRegs--;
35:     else if (Info.isDirect() && Info.getInReg()) {
36:       unsigned sz = (getContext().getTypeSize(Ty) + 31) / 32;
```
- **EN**: This block defines callable entry points like `EmitVAArg`, `updateState`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `EmitVAArg`, `updateState`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 37-48
```cpp
37:       if (sz < State.FreeRegs)
38:         State.FreeRegs -= sz;
39:       else
40:         State.FreeRegs = 0;
41:     }
42:   }
43: 
44:   void computeInfo(CGFunctionInfo &FI) const override {
45:     CCState State;
46:     // ARC uses 8 registers to pass arguments.
47:     State.FreeRegs = 8;
48: 
```
- **EN**: This block defines callable entry points like `computeInfo`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `computeInfo`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 49-60
```cpp
49:     if (!getCXXABI().classifyReturnType(FI))
50:       FI.getReturnInfo() = classifyReturnType(FI.getReturnType());
51:     updateState(FI.getReturnInfo(), FI.getReturnType(), State);
52:     for (auto &I : FI.arguments()) {
53:       I.info = classifyArgumentType(I.type, State.FreeRegs);
54:       updateState(I.info, I.type, State);
55:     }
56:   }
57: 
58:   ABIArgInfo getIndirectByRef(QualType Ty, bool HasFreeRegs) const;
59:   ABIArgInfo getIndirectByValue(QualType Ty) const;
60:   ABIArgInfo classifyArgumentType(QualType Ty, uint8_t FreeRegs) const;
```
- **EN**: This block defines callable entry points like `updateState`, `getIndirectByRef`, `getIndirectByValue`, `classifyArgumentType`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `updateState`, `getIndirectByRef`, `getIndirectByValue`, `classifyArgumentType`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 61-72
```cpp
61:   ABIArgInfo classifyReturnType(QualType RetTy) const;
62: };
63: 
64: class ARCTargetCodeGenInfo : public TargetCodeGenInfo {
65: public:
66:   ARCTargetCodeGenInfo(CodeGenTypes &CGT)
67:       : TargetCodeGenInfo(std::make_unique<ARCABIInfo>(CGT)) {}
68: };
69: 
70: 
71: ABIArgInfo ARCABIInfo::getIndirectByRef(QualType Ty, bool HasFreeRegs) const {
72:   return HasFreeRegs ? getNaturalAlignIndirectInReg(Ty)
```
- **EN**: This block introduces declarations such as `ARCTargetCodeGenInfo`; defines callable entry points like `classifyReturnType`, `ARCTargetCodeGenInfo`, `getIndirectByRef`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块给出诸如 `ARCTargetCodeGenInfo` 的声明；定义可调用入口，例如 `classifyReturnType`, `ARCTargetCodeGenInfo`, `getIndirectByRef`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 73-84
```cpp
73:                      : getNaturalAlignIndirect(
74:                            Ty, getDataLayout().getAllocaAddrSpace(), false);
75: }
76: 
77: ABIArgInfo ARCABIInfo::getIndirectByValue(QualType Ty) const {
78:   // Compute the byval alignment.
79:   const unsigned MinABIStackAlignInBytes = 4;
80:   unsigned TypeAlign = getContext().getTypeAlign(Ty) / 8;
81:   return ABIArgInfo::getIndirect(
82:       CharUnits::fromQuantity(4),
83:       /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
84:       /*ByVal=*/true, TypeAlign > MinABIStackAlignInBytes);
```
- **EN**: This block defines callable entry points like `getNaturalAlignIndirect`, `getIndirectByValue`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `getNaturalAlignIndirect`, `getIndirectByValue`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 85-96
```cpp
85: }
86: 
87: RValue ARCABIInfo::EmitVAArg(CodeGenFunction &CGF, Address VAListAddr,
88:                              QualType Ty, AggValueSlot Slot) const {
89:   return emitVoidPtrVAArg(CGF, VAListAddr, Ty, /*indirect*/ false,
90:                           getContext().getTypeInfoInChars(Ty),
91:                           CharUnits::fromQuantity(4), true, Slot);
92: }
93: 
94: ABIArgInfo ARCABIInfo::classifyArgumentType(QualType Ty,
95:                                             uint8_t FreeRegs) const {
96:   // Handle the generic C++ ABI.
```
- **EN**: This block defines callable entry points like `EmitVAArg`, `emitVoidPtrVAArg`, `classifyArgumentType`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `EmitVAArg`, `emitVoidPtrVAArg`, `classifyArgumentType`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 97-108
```cpp
 97:   const RecordType *RT = Ty->getAsCanonical<RecordType>();
 98:   if (RT) {
 99:     CGCXXABI::RecordArgABI RAA = getRecordArgABI(RT, getCXXABI());
100:     if (RAA == CGCXXABI::RAA_Indirect)
101:       return getIndirectByRef(Ty, FreeRegs > 0);
102: 
103:     if (RAA == CGCXXABI::RAA_DirectInMemory)
104:       return getIndirectByValue(Ty);
105:   }
106: 
107:   // Treat an enum type as its underlying type.
108:   if (const auto *ED = Ty->getAsEnumDecl())
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 109-120
```cpp
109:     Ty = ED->getIntegerType();
110: 
111:   auto SizeInRegs = llvm::alignTo(getContext().getTypeSize(Ty), 32) / 32;
112: 
113:   if (isAggregateTypeForABI(Ty)) {
114:     // Structures with flexible arrays are always indirect.
115:     if (RT && RT->getDecl()->getDefinitionOrSelf()->hasFlexibleArrayMember())
116:       return getIndirectByValue(Ty);
117: 
118:     // Ignore empty structs/unions.
119:     if (isEmptyRecord(getContext(), Ty, true))
120:       return ABIArgInfo::getIgnore();
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 121-132
```cpp
121: 
122:     llvm::LLVMContext &LLVMContext = getVMContext();
123: 
124:     llvm::IntegerType *Int32 = llvm::Type::getInt32Ty(LLVMContext);
125:     SmallVector<llvm::Type *, 3> Elements(SizeInRegs, Int32);
126:     llvm::Type *Result = llvm::StructType::get(LLVMContext, Elements);
127: 
128:     return FreeRegs >= SizeInRegs ?
129:         ABIArgInfo::getDirectInReg(Result) :
130:         ABIArgInfo::getDirect(Result, 0, nullptr, false);
131:   }
132: 
```
- **EN**: This block spells out callable entry points like `Elements`, `getDirectInReg`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块给出可调用入口的声明，例如 `Elements`, `getDirectInReg`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 133-144
```cpp
133:   if (const auto *EIT = Ty->getAs<BitIntType>())
134:     if (EIT->getNumBits() > 64)
135:       return getIndirectByValue(Ty);
136: 
137:   return isPromotableIntegerTypeForABI(Ty)
138:              ? (FreeRegs >= SizeInRegs ? ABIArgInfo::getExtendInReg(Ty)
139:                                        : ABIArgInfo::getExtend(Ty))
140:              : (FreeRegs >= SizeInRegs ? ABIArgInfo::getDirectInReg()
141:                                        : ABIArgInfo::getDirect());
142: }
143: 
144: ABIArgInfo ARCABIInfo::classifyReturnType(QualType RetTy) const {
```
- **EN**: This block defines callable entry points like `isPromotableIntegerTypeForABI`, `classifyReturnType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isPromotableIntegerTypeForABI`, `classifyReturnType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 145-156
```cpp
145:   if (RetTy->isAnyComplexType())
146:     return ABIArgInfo::getDirectInReg();
147: 
148:   // Arguments of size > 4 registers are indirect.
149:   auto RetSize = llvm::alignTo(getContext().getTypeSize(RetTy), 32) / 32;
150:   if (RetSize > 4)
151:     return getIndirectByRef(RetTy, /*HasFreeRegs*/ true);
152: 
153:   return DefaultABIInfo::classifyReturnType(RetTy);
154: }
155: 
156: } // End anonymous namespace.
```
- **EN**: This block spells out callable entry points like `classifyReturnType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出可调用入口的声明，例如 `classifyReturnType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 157-161
```cpp
157: 
158: std::unique_ptr<TargetCodeGenInfo>
159: CodeGen::createARCTargetCodeGenInfo(CodeGenModule &CGM) {
160:   return std::make_unique<ARCTargetCodeGenInfo>(CGM.getTypes());
161: }
```
- **EN**: This block defines callable entry points like `createARCTargetCodeGenInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `createARCTargetCodeGenInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

## Key Concepts / 关键概念

- **ABIArgInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **FreeRegs**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **QualType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **State**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ARCABIInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **getContext**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **RetTy**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Info**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfoImpl.h`, `TargetInfo.h`
