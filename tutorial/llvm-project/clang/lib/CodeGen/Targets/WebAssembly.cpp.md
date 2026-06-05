# WebAssembly.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/Targets/WebAssembly.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements target-specific ABI rules and lowering details for WebAssembly.
- **Purpose (CN) / 目的（中文）**: 实现 WebAssembly 目标相关的 ABI 规则与降级细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: //===- WebAssembly.cpp ----------------------------------------------------===//
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
16: // WebAssembly ABI Implementation
17: //
18: // This is a very simple ABI that relies a lot on DefaultABIInfo.
19: //===----------------------------------------------------------------------===//
20: 
21: class WebAssemblyABIInfo final : public ABIInfo {
22:   DefaultABIInfo defaultInfo;
23:   WebAssemblyABIKind Kind;
24: 
```
- **EN**: This block opens or references namespaces `clang`; introduces declarations such as `WebAssemblyABIInfo`.
- **CN**: 该代码块打开或引用命名空间 `clang`；给出诸如 `WebAssemblyABIInfo` 的声明。

### Lines 25-36
```cpp
25: public:
26:   explicit WebAssemblyABIInfo(CodeGen::CodeGenTypes &CGT,
27:                               WebAssemblyABIKind Kind)
28:       : ABIInfo(CGT), defaultInfo(CGT), Kind(Kind) {}
29: 
30: private:
31:   ABIArgInfo classifyReturnType(QualType RetTy) const;
32:   ABIArgInfo classifyArgumentType(QualType Ty) const;
33: 
34:   // DefaultABIInfo's classifyReturnType and classifyArgumentType are
35:   // non-virtual, but computeInfo and EmitVAArg are virtual, so we
36:   // overload them.
```
- **EN**: This block defines callable entry points like `WebAssemblyABIInfo`, `classifyReturnType`, `classifyArgumentType`.
- **CN**: 该代码块定义可调用入口，例如 `WebAssemblyABIInfo`, `classifyReturnType`, `classifyArgumentType`。

### Lines 37-48
```cpp
37:   void computeInfo(CGFunctionInfo &FI) const override {
38:     if (!getCXXABI().classifyReturnType(FI))
39:       FI.getReturnInfo() = classifyReturnType(FI.getReturnType());
40:     for (auto &Arg : FI.arguments())
41:       Arg.info = classifyArgumentType(Arg.type);
42:   }
43: 
44:   RValue EmitVAArg(CodeGenFunction &CGF, Address VAListAddr, QualType Ty,
45:                    AggValueSlot Slot) const override;
46: };
47: 
48: class WebAssemblyTargetCodeGenInfo final : public TargetCodeGenInfo {
```
- **EN**: This block introduces declarations such as `WebAssemblyTargetCodeGenInfo`; defines callable entry points like `computeInfo`, `EmitVAArg`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出诸如 `WebAssemblyTargetCodeGenInfo` 的声明；定义可调用入口，例如 `computeInfo`, `EmitVAArg`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 49-60
```cpp
49: public:
50:   explicit WebAssemblyTargetCodeGenInfo(CodeGen::CodeGenTypes &CGT,
51:                                         WebAssemblyABIKind K)
52:       : TargetCodeGenInfo(std::make_unique<WebAssemblyABIInfo>(CGT, K)) {
53:     SwiftInfo =
54:         std::make_unique<SwiftABIInfo>(CGT, /*SwiftErrorInRegister=*/false);
55:   }
56: 
57:   void setTargetAttributes(const Decl *D, llvm::GlobalValue *GV,
58:                            CodeGen::CodeGenModule &CGM) const override {
59:     TargetCodeGenInfo::setTargetAttributes(D, GV, CGM);
60:     if (const auto *FD = dyn_cast_or_null<FunctionDecl>(D)) {
```
- **EN**: This block defines callable entry points like `WebAssemblyTargetCodeGenInfo`, `setTargetAttributes`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `WebAssemblyTargetCodeGenInfo`, `setTargetAttributes`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 61-72
```cpp
61:       if (const auto *Attr = FD->getAttr<WebAssemblyImportModuleAttr>()) {
62:         llvm::Function *Fn = cast<llvm::Function>(GV);
63:         llvm::AttrBuilder B(GV->getContext());
64:         B.addAttribute("wasm-import-module", Attr->getImportModule());
65:         Fn->addFnAttrs(B);
66:       }
67:       if (const auto *Attr = FD->getAttr<WebAssemblyImportNameAttr>()) {
68:         llvm::Function *Fn = cast<llvm::Function>(GV);
69:         llvm::AttrBuilder B(GV->getContext());
70:         B.addAttribute("wasm-import-name", Attr->getImportName());
71:         Fn->addFnAttrs(B);
72:       }
```
- **EN**: This block defines callable entry points like `B`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `B`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 73-84
```cpp
73:       if (const auto *Attr = FD->getAttr<WebAssemblyExportNameAttr>()) {
74:         llvm::Function *Fn = cast<llvm::Function>(GV);
75:         llvm::AttrBuilder B(GV->getContext());
76:         B.addAttribute("wasm-export-name", Attr->getExportName());
77:         Fn->addFnAttrs(B);
78:       }
79:     }
80: 
81:     if (auto *FD = dyn_cast_or_null<FunctionDecl>(D)) {
82:       llvm::Function *Fn = cast<llvm::Function>(GV);
83:       if (!FD->doesThisDeclarationHaveABody() && !FD->hasPrototype())
84:         Fn->addFnAttr("no-prototype");
```
- **EN**: This block defines callable entry points like `B`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `B`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 85-96
```cpp
85:     }
86:   }
87: 
88:   /// Return the WebAssembly externref reference type.
89:   virtual llvm::Type *getWasmExternrefReferenceType() const override {
90:     return llvm::Type::getWasm_ExternrefTy(getABIInfo().getVMContext());
91:   }
92:   /// Return the WebAssembly funcref reference type.
93:   virtual llvm::Type *getWasmFuncrefReferenceType() const override {
94:     return llvm::Type::getWasm_FuncrefTy(getABIInfo().getVMContext());
95:   }
96: };
```
- **EN**: This block defines callable entry points like `getWasm_ExternrefTy`, `getWasm_FuncrefTy`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `getWasm_ExternrefTy`, `getWasm_FuncrefTy`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 97-108
```cpp
 97: 
 98: /// Classify argument of given type \p Ty.
 99: ABIArgInfo WebAssemblyABIInfo::classifyArgumentType(QualType Ty) const {
100:   Ty = useFirstFieldIfTransparentUnion(Ty);
101: 
102:   if (isAggregateTypeForABI(Ty)) {
103:     // Records with non-trivial destructors/copy-constructors should not be
104:     // passed by value.
105:     if (auto RAA = getRecordArgABI(Ty, getCXXABI()))
106:       return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
107:                                      RAA == CGCXXABI::RAA_DirectInMemory);
108:     // Ignore empty structs/unions.
```
- **EN**: This block defines callable entry points like `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 109-120
```cpp
109:     if (isEmptyRecord(getContext(), Ty, true))
110:       return ABIArgInfo::getIgnore();
111:     // Lower single-element structs to just pass a regular value. TODO: We
112:     // could do reasonable-size multiple-element structs too, using getExpand(),
113:     // though watch out for things like bitfields.
114:     if (const Type *SeltTy = isSingleElementStruct(Ty, getContext()))
115:       return ABIArgInfo::getDirect(CGT.ConvertType(QualType(SeltTy, 0)));
116:     // For the experimental multivalue ABI, fully expand all other aggregates
117:     if (Kind == WebAssemblyABIKind::ExperimentalMV) {
118:       const auto *RD = Ty->castAsRecordDecl();
119:       bool HasBitField = false;
120:       for (auto *Field : RD->fields()) {
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 121-132
```cpp
121:         if (Field->isBitField()) {
122:           HasBitField = true;
123:           break;
124:         }
125:       }
126:       if (!HasBitField)
127:         return ABIArgInfo::getExpand();
128:     }
129:   }
130: 
131:   // Otherwise just do the default thing.
132:   return defaultInfo.classifyArgumentType(Ty);
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 133-144
```cpp
133: }
134: 
135: ABIArgInfo WebAssemblyABIInfo::classifyReturnType(QualType RetTy) const {
136:   if (isAggregateTypeForABI(RetTy)) {
137:     // Records with non-trivial destructors/copy-constructors should not be
138:     // returned by value.
139:     if (!getRecordArgABI(RetTy, getCXXABI())) {
140:       // Ignore empty structs/unions.
141:       if (isEmptyRecord(getContext(), RetTy, true))
142:         return ABIArgInfo::getIgnore();
143:       // Lower single-element structs to just return a regular value. TODO: We
144:       // could do reasonable-size multiple-element structs too, using
```
- **EN**: This block defines callable entry points like `classifyReturnType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyReturnType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 145-156
```cpp
145:       // ABIArgInfo::getDirect().
146:       if (const Type *SeltTy = isSingleElementStruct(RetTy, getContext()))
147:         return ABIArgInfo::getDirect(CGT.ConvertType(QualType(SeltTy, 0)));
148:       // For the experimental multivalue ABI, return all other aggregates
149:       if (Kind == WebAssemblyABIKind::ExperimentalMV)
150:         return ABIArgInfo::getDirect();
151:     }
152:   }
153: 
154:   // Otherwise just do the default thing.
155:   return defaultInfo.classifyReturnType(RetTy);
156: }
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 157-168
```cpp
157: 
158: RValue WebAssemblyABIInfo::EmitVAArg(CodeGenFunction &CGF, Address VAListAddr,
159:                                      QualType Ty, AggValueSlot Slot) const {
160:   bool IsIndirect = isAggregateTypeForABI(Ty) &&
161:                     !isEmptyRecord(getContext(), Ty, true) &&
162:                     !isSingleElementStruct(Ty, getContext());
163:   return emitVoidPtrVAArg(CGF, VAListAddr, Ty, IsIndirect,
164:                           getContext().getTypeInfoInChars(Ty),
165:                           CharUnits::fromQuantity(4),
166:                           /*AllowHigherAlign=*/true, Slot);
167: }
168: 
```
- **EN**: This block defines callable entry points like `EmitVAArg`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `EmitVAArg`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 169-173
```cpp
169: std::unique_ptr<TargetCodeGenInfo>
170: CodeGen::createWebAssemblyTargetCodeGenInfo(CodeGenModule &CGM,
171:                                             WebAssemblyABIKind K) {
172:   return std::make_unique<WebAssemblyTargetCodeGenInfo>(CGM.getTypes(), K);
173: }
```
- **EN**: This block defines callable entry points like `createWebAssemblyTargetCodeGenInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `createWebAssemblyTargetCodeGenInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

## Key Concepts / 关键概念

- **ABIArgInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **getContext**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGT**: Central symbol in this file's implementation of target-specific ABI and code generation. / 是该文件实现 目标相关的 ABI 与代码生成 时的核心符号。
- **QualType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Function**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **RetTy**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **WebAssemblyABIInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **WebAssemblyABIKind**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfoImpl.h`, `TargetInfo.h`
