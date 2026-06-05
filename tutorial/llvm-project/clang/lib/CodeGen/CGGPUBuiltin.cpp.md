# CGGPUBuiltin.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/CGGPUBuiltin.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements the CGGPUBuiltin portion of Clang's CodeGen pipeline.
- **Purpose (CN) / 目的（中文）**: 实现 Clang CodeGen 流水线中的 CGGPUBuiltin 部分。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: //===------ CGGPUBuiltin.cpp - Codegen for GPU builtins -------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // Generates code for built-in GPU calls which are not runtime-specific.
10: // (Runtime-specific codegen lives in programming model specific files.)
11: //
12: //===----------------------------------------------------------------------===//
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 13-24
```cpp
13: 
14: #include "CodeGenFunction.h"
15: #include "clang/Basic/Builtins.h"
16: #include "llvm/IR/DataLayout.h"
17: #include "llvm/IR/Instruction.h"
18: #include "llvm/Transforms/Utils/AMDGPUEmitPrintf.h"
19: 
20: using namespace clang;
21: using namespace CodeGen;
22: 
23: namespace {
24: llvm::Function *GetVprintfDeclaration(llvm::Module &M) {
```
- **EN**: This block imports local CodeGen headers `CodeGenFunction.h`; Clang headers `clang/Basic/Builtins.h`; LLVM headers `llvm/IR/DataLayout.h`, `llvm/IR/Instruction.h`, `llvm/Transforms/Utils/AMDGPUEmitPrintf.h`; opens or references namespaces `clang`, `CodeGen`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CodeGenFunction.h`；Clang 头文件 `clang/Basic/Builtins.h`；LLVM 头文件 `llvm/IR/DataLayout.h`, `llvm/IR/Instruction.h`, `llvm/Transforms/Utils/AMDGPUEmitPrintf.h`；打开或引用命名空间 `clang`, `CodeGen`；包含影响本编译单元构建方式的预处理结构。

### Lines 25-36
```cpp
25:   llvm::Type *ArgTypes[] = {llvm::PointerType::getUnqual(M.getContext()),
26:                             llvm::PointerType::getUnqual(M.getContext())};
27:   llvm::FunctionType *VprintfFuncType = llvm::FunctionType::get(
28:       llvm::Type::getInt32Ty(M.getContext()), ArgTypes, false);
29: 
30:   if (auto *F = M.getFunction("vprintf")) {
31:     // Our CUDA system header declares vprintf with the right signature, so
32:     // nobody else should have been able to declare vprintf with a bogus
33:     // signature.
34:     assert(F->getFunctionType() == VprintfFuncType);
35:     return F;
36:   }
```
- **EN**: This block defines callable entry points like `getInt32Ty`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getInt32Ty`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 37-48
```cpp
37: 
38:   // vprintf doesn't already exist; create a declaration and insert it into the
39:   // module.
40:   return llvm::Function::Create(
41:       VprintfFuncType, llvm::GlobalVariable::ExternalLinkage, "vprintf", &M);
42: }
43: 
44: // Transforms a call to printf into a call to the NVPTX vprintf syscall (which
45: // isn't particularly special; it's invoked just like a regular function).
46: // vprintf takes two args: A format string, and a pointer to a buffer containing
47: // the varargs.
48: //
```
- **EN**: This block spells out callable entry points like `Create`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块给出可调用入口的声明，例如 `Create`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 49-60
```cpp
49: // For example, the call
50: //
51: //   printf("format string", arg1, arg2, arg3);
52: //
53: // is converted into something resembling
54: //
55: //   struct Tmp {
56: //     Arg1 a1;
57: //     Arg2 a2;
58: //     Arg3 a3;
59: //   };
60: //   char* buf = alloca(sizeof(Tmp));
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 61-72
```cpp
61: //   *(Tmp*)buf = {a1, a2, a3};
62: //   vprintf("format string", buf);
63: //
64: // buf is aligned to the max of {alignof(Arg1), ...}.  Furthermore, each of the
65: // args is itself aligned to its preferred alignment.
66: //
67: // Note that by the time this function runs, E's args have already undergone the
68: // standard C vararg promotion (short -> int, float -> double, etc.).
69: 
70: std::pair<llvm::Value *, llvm::TypeSize>
71: packArgsIntoNVPTXFormatBuffer(CodeGenFunction *CGF, const CallArgList &Args) {
72:   const llvm::DataLayout &DL = CGF->CGM.getDataLayout();
```
- **EN**: This block defines callable entry points like `packArgsIntoNVPTXFormatBuffer`.
- **CN**: 该代码块定义可调用入口，例如 `packArgsIntoNVPTXFormatBuffer`。

### Lines 73-84
```cpp
73:   llvm::LLVMContext &Ctx = CGF->CGM.getLLVMContext();
74:   CGBuilderTy &Builder = CGF->Builder;
75: 
76:   // Construct and fill the args buffer that we'll pass to vprintf.
77:   if (Args.size() <= 1) {
78:     // If there are no args, pass a null pointer and size 0
79:     llvm::Value *BufferPtr =
80:         llvm::ConstantPointerNull::get(llvm::PointerType::getUnqual(Ctx));
81:     return {BufferPtr, llvm::TypeSize::getFixed(0)};
82:   } else {
83:     llvm::SmallVector<llvm::Type *, 8> ArgTypes;
84:     for (unsigned I = 1, NumArgs = Args.size(); I < NumArgs; ++I)
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 85-96
```cpp
85:       ArgTypes.push_back(Args[I].getRValue(*CGF).getScalarVal()->getType());
86: 
87:     // Using llvm::StructType is correct only because printf doesn't accept
88:     // aggregates.  If we had to handle aggregates here, we'd have to manually
89:     // compute the offsets within the alloca -- we wouldn't be able to assume
90:     // that the alignment of the llvm type was the same as the alignment of the
91:     // clang type.
92:     llvm::Type *AllocaTy = llvm::StructType::create(ArgTypes, "printf_args");
93:     llvm::Value *Alloca = CGF->CreateTempAlloca(AllocaTy);
94: 
95:     for (unsigned I = 1, NumArgs = Args.size(); I < NumArgs; ++I) {
96:       llvm::Value *P = Builder.CreateStructGEP(AllocaTy, Alloca, I - 1);
```
- **EN**: This block uses control flow (for) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（for）细化 LLVM IR 生成 行为。

### Lines 97-108
```cpp
 97:       llvm::Value *Arg = Args[I].getRValue(*CGF).getScalarVal();
 98:       Builder.CreateAlignedStore(Arg, P, DL.getPrefTypeAlign(Arg->getType()));
 99:     }
100:     llvm::Value *BufferPtr =
101:         Builder.CreatePointerCast(Alloca, llvm::PointerType::getUnqual(Ctx));
102:     return {BufferPtr, DL.getTypeAllocSize(AllocaTy)};
103:   }
104: }
105: 
106: bool containsNonScalarVarargs(CodeGenFunction *CGF, const CallArgList &Args) {
107:   return llvm::any_of(llvm::drop_begin(Args), [&](const CallArg &A) {
108:     return !A.getRValue(*CGF).isScalar();
```
- **EN**: This block defines callable entry points like `containsNonScalarVarargs`, `any_of`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `containsNonScalarVarargs`, `any_of`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 109-120
```cpp
109:   });
110: }
111: 
112: RValue EmitDevicePrintfCallExpr(const CallExpr *E, CodeGenFunction *CGF,
113:                                 llvm::Function *Decl, bool WithSizeArg) {
114:   CodeGenModule &CGM = CGF->CGM;
115:   CGBuilderTy &Builder = CGF->Builder;
116:   assert(E->getBuiltinCallee() == Builtin::BIprintf ||
117:          E->getBuiltinCallee() == Builtin::BI__builtin_printf);
118:   assert(E->getNumArgs() >= 1); // printf always has at least one arg.
119: 
120:   // Uses the same format as nvptx for the argument packing, but also passes
```
- **EN**: This block defines callable entry points like `EmitDevicePrintfCallExpr`; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitDevicePrintfCallExpr`；使用断言或不可达标记保护关键不变量。

### Lines 121-132
```cpp
121:   // an i32 for the total size of the passed pointer
122:   CallArgList Args;
123:   CGF->EmitCallArgs(Args,
124:                     E->getDirectCallee()->getType()->getAs<FunctionProtoType>(),
125:                     E->arguments(), E->getDirectCallee(),
126:                     /* ParamsToSkip = */ 0);
127: 
128:   // We don't know how to emit non-scalar varargs.
129:   if (containsNonScalarVarargs(CGF, Args)) {
130:     CGM.ErrorUnsupported(E, "non-scalar arg to printf");
131:     return RValue::get(llvm::ConstantInt::get(CGF->IntTy, 0));
132:   }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 133-144
```cpp
133: 
134:   auto r = packArgsIntoNVPTXFormatBuffer(CGF, Args);
135:   llvm::Value *BufferPtr = r.first;
136: 
137:   llvm::SmallVector<llvm::Value *, 3> Vec = {
138:       Args[0].getRValue(*CGF).getScalarVal(), BufferPtr};
139:   if (WithSizeArg) {
140:     // Passing > 32bit of data as a local alloca doesn't work for nvptx or
141:     // amdgpu
142:     llvm::Constant *Size =
143:         llvm::ConstantInt::get(llvm::Type::getInt32Ty(CGM.getLLVMContext()),
144:                                static_cast<uint32_t>(r.second.getFixedValue()));
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 145-156
```cpp
145: 
146:     Vec.push_back(Size);
147:   }
148:   return RValue::get(Builder.CreateCall(Decl, Vec));
149: }
150: } // namespace
151: 
152: RValue CodeGenFunction::EmitNVPTXDevicePrintfCallExpr(const CallExpr *E) {
153:   assert(getTarget().getTriple().isNVPTX());
154:   return EmitDevicePrintfCallExpr(
155:       E, this, GetVprintfDeclaration(CGM.getModule()), false);
156: }
```
- **EN**: This block opens or references namespaces `RValue`; defines callable entry points like `get`, `EmitNVPTXDevicePrintfCallExpr`, `EmitDevicePrintfCallExpr`; returns or forwards computed values for the surrounding LLVM IR emission logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块打开或引用命名空间 `RValue`；定义可调用入口，例如 `get`, `EmitNVPTXDevicePrintfCallExpr`, `EmitDevicePrintfCallExpr`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 157-168
```cpp
157: 
158: RValue CodeGenFunction::EmitAMDGPUDevicePrintfCallExpr(const CallExpr *E) {
159:   assert(getTarget().getTriple().isAMDGCN() ||
160:          (getTarget().getTriple().isSPIRV() &&
161:           getTarget().getTriple().getVendor() == llvm::Triple::AMD));
162:   assert(E->getBuiltinCallee() == Builtin::BIprintf ||
163:          E->getBuiltinCallee() == Builtin::BI__builtin_printf);
164:   assert(E->getNumArgs() >= 1); // printf always has at least one arg.
165: 
166:   CallArgList CallArgs;
167:   EmitCallArgs(CallArgs,
168:                E->getDirectCallee()->getType()->getAs<FunctionProtoType>(),
```
- **EN**: This block defines callable entry points like `EmitAMDGPUDevicePrintfCallExpr`; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitAMDGPUDevicePrintfCallExpr`；使用断言或不可达标记保护关键不变量。

### Lines 169-180
```cpp
169:                E->arguments(), E->getDirectCallee(),
170:                /* ParamsToSkip = */ 0);
171: 
172:   SmallVector<llvm::Value *, 8> Args;
173:   for (const auto &A : CallArgs) {
174:     // We don't know how to emit non-scalar varargs.
175:     if (!A.getRValue(*this).isScalar()) {
176:       CGM.ErrorUnsupported(E, "non-scalar arg to printf");
177:       return RValue::get(llvm::ConstantInt::getAllOnesValue(IntTy));
178:     }
179: 
180:     llvm::Value *Arg = A.getRValue(*this).getScalarVal();
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 181-192
```cpp
181:     Args.push_back(Arg);
182:   }
183: 
184:   llvm::IRBuilder<> IRB(Builder.GetInsertBlock(), Builder.GetInsertPoint());
185:   IRB.SetCurrentDebugLocation(Builder.getCurrentDebugLocation());
186: 
187:   bool isBuffered = (CGM.getTarget().getTargetOpts().AMDGPUPrintfKindVal ==
188:                      clang::TargetOptions::AMDGPUPrintfKind::Buffered);
189:   auto Printf = llvm::emitAMDGPUPrintfCall(IRB, Args, isBuffered);
190:   Builder.SetInsertPoint(IRB.GetInsertBlock(), IRB.GetInsertPoint());
191:   return RValue::get(Printf);
192: }
```
- **EN**: This block spells out callable entry points like `IRB`, `get`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块给出可调用入口的声明，例如 `IRB`, `get`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

## Key Concepts / 关键概念

- **CGF**: Central symbol in this file's implementation of LLVM IR emission. / 是该文件实现 LLVM IR 生成 时的核心符号。
- **Args**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Builder**: Acts as a construction helper that incrementally assembles LLVM IR emission state. / 充当构建辅助器，逐步组装 LLVM IR 生成 状态。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGM**: Central symbol in this file's implementation of LLVM IR emission. / 是该文件实现 LLVM IR 生成 时的核心符号。
- **RValue**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CodeGenFunction**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **BufferPtr**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CodeGenFunction.h`
- **Clang libraries / Clang 库**: `clang/Basic/Builtins.h`
- **LLVM libraries / LLVM 库**: `llvm/IR/DataLayout.h`, `llvm/IR/Instruction.h`, `llvm/Transforms/Utils/AMDGPUEmitPrintf.h`
