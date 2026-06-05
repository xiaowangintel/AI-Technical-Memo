# AVR.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/Targets/AVR.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements target-specific ABI rules and lowering details for AVR.
- **Purpose (CN) / 目的（中文）**: 实现 AVR 目标相关的 ABI 规则与降级细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: //===- AVR.cpp ------------------------------------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: 
 9: #include "ABIInfoImpl.h"
10: #include "TargetInfo.h"
11: #include "clang/Basic/DiagnosticFrontend.h"
12: 
```
- **EN**: This block imports local CodeGen headers `ABIInfoImpl.h`, `TargetInfo.h`; Clang headers `clang/Basic/DiagnosticFrontend.h`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `ABIInfoImpl.h`, `TargetInfo.h`；Clang 头文件 `clang/Basic/DiagnosticFrontend.h`；包含影响本编译单元构建方式的预处理结构。

### Lines 13-24
```cpp
13: using namespace clang;
14: using namespace clang::CodeGen;
15: 
16: //===----------------------------------------------------------------------===//
17: // AVR ABI Implementation. Documented at
18: // https://gcc.gnu.org/wiki/avr-gcc#Calling_Convention
19: // https://gcc.gnu.org/wiki/avr-gcc#Reduced_Tiny
20: //===----------------------------------------------------------------------===//
21: 
22: namespace {
23: class AVRABIInfo : public DefaultABIInfo {
24: private:
```
- **EN**: This block opens or references namespaces `clang`; introduces declarations such as `AVRABIInfo`.
- **CN**: 该代码块打开或引用命名空间 `clang`；给出诸如 `AVRABIInfo` 的声明。

### Lines 25-36
```cpp
25:   // The total amount of registers can be used to pass parameters. It is 18 on
26:   // AVR, or 6 on AVRTiny.
27:   const unsigned ParamRegs;
28:   // The total amount of registers can be used to pass return value. It is 8 on
29:   // AVR, or 4 on AVRTiny.
30:   const unsigned RetRegs;
31: 
32: public:
33:   AVRABIInfo(CodeGenTypes &CGT, unsigned NPR, unsigned NRR)
34:       : DefaultABIInfo(CGT), ParamRegs(NPR), RetRegs(NRR) {}
35: 
36:   ABIArgInfo classifyReturnType(QualType Ty, bool &LargeRet) const {
```
- **EN**: This block defines callable entry points like `AVRABIInfo`, `classifyReturnType`.
- **CN**: 该代码块定义可调用入口，例如 `AVRABIInfo`, `classifyReturnType`。

### Lines 37-48
```cpp
37:     // On AVR, a return struct with size less than or equals to 8 bytes is
38:     // returned directly via registers R18-R25. On AVRTiny, a return struct
39:     // with size less than or equals to 4 bytes is returned directly via
40:     // registers R22-R25.
41:     if (isAggregateTypeForABI(Ty) &&
42:         getContext().getTypeSize(Ty) <= RetRegs * 8)
43:       return ABIArgInfo::getDirect();
44:     // A return value (struct or scalar) with larger size is returned via a
45:     // stack slot, along with a pointer as the function's implicit argument.
46:     if (getContext().getTypeSize(Ty) > RetRegs * 8) {
47:       LargeRet = true;
48:       return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace());
```
- **EN**: This block defines callable entry points like `getNaturalAlignIndirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getNaturalAlignIndirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 49-60
```cpp
49:     }
50:     // An i8 return value should not be extended to i16, since AVR has 8-bit
51:     // registers.
52:     if (Ty->isIntegralOrEnumerationType() && getContext().getTypeSize(Ty) <= 8)
53:       return ABIArgInfo::getDirect();
54:     // Otherwise we follow the default way which is compatible.
55:     return DefaultABIInfo::classifyReturnType(Ty);
56:   }
57: 
58:   ABIArgInfo classifyArgumentType(QualType Ty, unsigned &NumRegs) const {
59:     unsigned TySize = getContext().getTypeSize(Ty);
60: 
```
- **EN**: This block defines callable entry points like `classifyReturnType`, `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyReturnType`, `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 61-72
```cpp
61:     // An int8 type argument always costs two registers like an int16.
62:     if (TySize == 8 && NumRegs >= 2 && Ty->isIntegralOrEnumerationType()) {
63:       NumRegs -= 2;
64:       return ABIArgInfo::getExtend(Ty);
65:     }
66: 
67:     // If the argument size is an odd number of bytes, round up the size
68:     // to the next even number.
69:     TySize = llvm::alignTo(TySize, 16);
70: 
71:     // Any type including an array/struct type can be passed in rgisters,
72:     // if there are enough registers left.
```
- **EN**: This block defines callable entry points like `getExtend`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getExtend`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 73-84
```cpp
73:     if (TySize <= NumRegs * 8) {
74:       NumRegs -= TySize / 8;
75:       return ABIArgInfo::getDirect();
76:     }
77: 
78:     // An argument is passed either completely in registers or completely in
79:     // memory. Since there are not enough registers left, current argument
80:     // and all other unprocessed arguments should be passed in memory.
81:     // However we still need to return `ABIArgInfo::getDirect()` other than
82:     // `ABIInfo::getNaturalAlignIndirect(Ty)`, otherwise an extra stack slot
83:     // will be allocated, so the stack frame layout will be incompatible with
84:     // avr-gcc.
```
- **EN**: This block defines callable entry points like `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 85-96
```cpp
85:     NumRegs = 0;
86:     return ABIArgInfo::getDirect();
87:   }
88: 
89:   void computeInfo(CGFunctionInfo &FI) const override {
90:     // Decide the return type.
91:     bool LargeRet = false;
92:     if (!getCXXABI().classifyReturnType(FI))
93:       FI.getReturnInfo() = classifyReturnType(FI.getReturnType(), LargeRet);
94: 
95:     // Decide each argument type. The total number of registers can be used for
96:     // arguments depends on several factors:
```
- **EN**: This block defines callable entry points like `getDirect`, `computeInfo`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `computeInfo`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 97-108
```cpp
 97:     // 1. Arguments of varargs functions are passed on the stack. This applies
 98:     //    even to the named arguments. So no register can be used.
 99:     // 2. Total 18 registers can be used on avr and 6 ones on avrtiny.
100:     // 3. If the return type is a struct with too large size, two registers
101:     //    (out of 18/6) will be cost as an implicit pointer argument.
102:     unsigned NumRegs = ParamRegs;
103:     if (FI.isVariadic())
104:       NumRegs = 0;
105:     else if (LargeRet)
106:       NumRegs -= 2;
107:     for (auto &I : FI.arguments())
108:       I.info = classifyArgumentType(I.type, NumRegs);
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 109-120
```cpp
109:   }
110: };
111: 
112: class AVRTargetCodeGenInfo : public TargetCodeGenInfo {
113: public:
114:   AVRTargetCodeGenInfo(CodeGenTypes &CGT, unsigned NPR, unsigned NRR)
115:       : TargetCodeGenInfo(std::make_unique<AVRABIInfo>(CGT, NPR, NRR)) {}
116: 
117:   LangAS getGlobalVarAddressSpace(CodeGenModule &CGM,
118:                                   const VarDecl *D) const override {
119:     // Check if global/static variable is defined in address space
120:     // 1~6 (__flash, __flash1, __flash2, __flash3, __flash4, __flash5)
```
- **EN**: This block introduces declarations such as `AVRTargetCodeGenInfo`; defines callable entry points like `AVRTargetCodeGenInfo`, `getGlobalVarAddressSpace`.
- **CN**: 该代码块给出诸如 `AVRTargetCodeGenInfo` 的声明；定义可调用入口，例如 `AVRTargetCodeGenInfo`, `getGlobalVarAddressSpace`。

### Lines 121-132
```cpp
121:     // but not constant.
122:     if (D) {
123:       LangAS AS = D->getType().getAddressSpace();
124:       if (isTargetAddressSpace(AS) && 1 <= toTargetAddressSpace(AS) &&
125:           toTargetAddressSpace(AS) <= 6 && !D->getType().isConstQualified())
126:         CGM.getDiags().Report(D->getLocation(),
127:                               diag::err_verify_nonconst_addrspace)
128:             << "__flash*";
129:     }
130:     return TargetCodeGenInfo::getGlobalVarAddressSpace(CGM, D);
131:   }
132: 
```
- **EN**: This block defines callable entry points like `getGlobalVarAddressSpace`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getGlobalVarAddressSpace`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 133-144
```cpp
133:   void setTargetAttributes(const Decl *D, llvm::GlobalValue *GV,
134:                            CodeGen::CodeGenModule &CGM) const override {
135:     if (GV->isDeclaration())
136:       return;
137:     const auto *FD = dyn_cast_or_null<FunctionDecl>(D);
138:     if (!FD) return;
139:     auto *Fn = cast<llvm::Function>(GV);
140: 
141:     if (FD->getAttr<AVRInterruptAttr>())
142:       Fn->addFnAttr("interrupt");
143: 
144:     if (FD->getAttr<AVRSignalAttr>())
```
- **EN**: This block defines callable entry points like `setTargetAttributes`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `setTargetAttributes`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 145-154
```cpp
145:       Fn->addFnAttr("signal");
146:   }
147: };
148: }
149: 
150: std::unique_ptr<TargetCodeGenInfo>
151: CodeGen::createAVRTargetCodeGenInfo(CodeGenModule &CGM, unsigned NPR,
152:                                     unsigned NRR) {
153:   return std::make_unique<AVRTargetCodeGenInfo>(CGM.getTypes(), NPR, NRR);
154: }
```
- **EN**: This block defines callable entry points like `createAVRTargetCodeGenInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `createAVRTargetCodeGenInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

## Key Concepts / 关键概念

- **NumRegs**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ABIArgInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **NPR**: Central symbol in this file's implementation of target-specific ABI and code generation. / 是该文件实现 目标相关的 ABI 与代码生成 时的核心符号。
- **NRR**: Central symbol in this file's implementation of target-specific ABI and code generation. / 是该文件实现 目标相关的 ABI 与代码生成 时的核心符号。
- **TySize**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGM**: Central symbol in this file's implementation of target-specific ABI and code generation. / 是该文件实现 目标相关的 ABI 与代码生成 时的核心符号。
- **LargeRet**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **RetRegs**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfoImpl.h`, `TargetInfo.h`
- **Clang libraries / Clang 库**: `clang/Basic/DiagnosticFrontend.h`
