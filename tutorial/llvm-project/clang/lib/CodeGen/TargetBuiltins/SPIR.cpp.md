# SPIR.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/TargetBuiltins/SPIR.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements builtin handling and lowering logic for the SPIR backend.
- **Purpose (CN) / 目的（中文）**: 实现 SPIR 后端的内建函数处理与降级逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: //===--------- SPIR.cpp - Emit LLVM Code for builtins ---------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This contains code to emit Builtin calls as LLVM code.
10: //
11: //===----------------------------------------------------------------------===//
12: 
```
- **EN**: This block documents intent or context for the surrounding target builtin lowering code.
- **CN**: 该代码块说明周围 目标内建函数降级 代码的意图或上下文。

### Lines 13-24
```cpp
13: #include "CGHLSLRuntime.h"
14: #include "CodeGenFunction.h"
15: #include "clang/Basic/TargetBuiltins.h"
16: #include "llvm/IR/Intrinsics.h"
17: 
18: using namespace clang;
19: using namespace CodeGen;
20: using namespace llvm;
21: 
22: Value *CodeGenFunction::EmitSPIRVBuiltinExpr(unsigned BuiltinID,
23:                                              const CallExpr *E) {
24:   switch (BuiltinID) {
```
- **EN**: This block imports local CodeGen headers `CGHLSLRuntime.h`, `CodeGenFunction.h`; Clang headers `clang/Basic/TargetBuiltins.h`; LLVM headers `llvm/IR/Intrinsics.h`; opens or references namespaces `clang`, `CodeGen`, `llvm`; uses control flow (switch) to specialize target builtin lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CGHLSLRuntime.h`, `CodeGenFunction.h`；Clang 头文件 `clang/Basic/TargetBuiltins.h`；LLVM 头文件 `llvm/IR/Intrinsics.h`；打开或引用命名空间 `clang`, `CodeGen`, `llvm`；通过控制流（switch）细化 目标内建函数降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 25-36
```cpp
25:   case SPIRV::BI__builtin_spirv_distance: {
26:     Value *X = EmitScalarExpr(E->getArg(0));
27:     Value *Y = EmitScalarExpr(E->getArg(1));
28:     assert(E->getArg(0)->getType()->hasFloatingRepresentation() &&
29:            E->getArg(1)->getType()->hasFloatingRepresentation() &&
30:            "Distance operands must have a float representation");
31:     assert(E->getArg(0)->getType()->isVectorType() &&
32:            E->getArg(1)->getType()->isVectorType() &&
33:            "Distance operands must be a vector");
34:     return Builder.CreateIntrinsic(
35:         /*ReturnType=*/X->getType()->getScalarType(), Intrinsic::spv_distance,
36:         ArrayRef<Value *>{X, Y}, nullptr, "spv.distance");
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 37-48
```cpp
37:   }
38:   case SPIRV::BI__builtin_spirv_length: {
39:     Value *X = EmitScalarExpr(E->getArg(0));
40:     assert(E->getArg(0)->getType()->hasFloatingRepresentation() &&
41:            "length operand must have a float representation");
42:     assert(E->getArg(0)->getType()->isVectorType() &&
43:            "length operand must be a vector");
44:     return Builder.CreateIntrinsic(
45:         /*ReturnType=*/X->getType()->getScalarType(), Intrinsic::spv_length,
46:         ArrayRef<Value *>{X}, nullptr, "spv.length");
47:   }
48:   case SPIRV::BI__builtin_spirv_reflect: {
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 49-60
```cpp
49:     Value *I = EmitScalarExpr(E->getArg(0));
50:     Value *N = EmitScalarExpr(E->getArg(1));
51:     assert(E->getArg(0)->getType()->hasFloatingRepresentation() &&
52:            E->getArg(1)->getType()->hasFloatingRepresentation() &&
53:            "Reflect operands must have a float representation");
54:     assert(E->getArg(0)->getType()->isVectorType() &&
55:            E->getArg(1)->getType()->isVectorType() &&
56:            "Reflect operands must be a vector");
57:     return Builder.CreateIntrinsic(
58:         /*ReturnType=*/I->getType(), Intrinsic::spv_reflect,
59:         ArrayRef<Value *>{I, N}, nullptr, "spv.reflect");
60:   }
```
- **EN**: This block guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块使用断言或不可达标记保护关键不变量。

### Lines 61-72
```cpp
61:   case SPIRV::BI__builtin_spirv_refract: {
62:     Value *I = EmitScalarExpr(E->getArg(0));
63:     Value *N = EmitScalarExpr(E->getArg(1));
64:     Value *eta = EmitScalarExpr(E->getArg(2));
65:     assert(E->getArg(0)->getType()->hasFloatingRepresentation() &&
66:            E->getArg(1)->getType()->hasFloatingRepresentation() &&
67:            E->getArg(2)->getType()->isFloatingType() &&
68:            "refract operands must have a float representation");
69:     return Builder.CreateIntrinsic(
70:         /*ReturnType=*/I->getType(), Intrinsic::spv_refract,
71:         ArrayRef<Value *>{I, N, eta}, nullptr, "spv.refract");
72:   }
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 73-84
```cpp
73:   case SPIRV::BI__builtin_spirv_smoothstep: {
74:     Value *Min = EmitScalarExpr(E->getArg(0));
75:     Value *Max = EmitScalarExpr(E->getArg(1));
76:     Value *X = EmitScalarExpr(E->getArg(2));
77:     assert(E->getArg(0)->getType()->hasFloatingRepresentation() &&
78:            E->getArg(1)->getType()->hasFloatingRepresentation() &&
79:            E->getArg(2)->getType()->hasFloatingRepresentation() &&
80:            "SmoothStep operands must have a float representation");
81:     return Builder.CreateIntrinsic(
82:         /*ReturnType=*/Min->getType(), Intrinsic::spv_smoothstep,
83:         ArrayRef<Value *>{Min, Max, X}, /*FMFSource=*/nullptr,
84:         "spv.smoothstep");
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 85-96
```cpp
85:   }
86:   case SPIRV::BI__builtin_spirv_faceforward: {
87:     Value *N = EmitScalarExpr(E->getArg(0));
88:     Value *I = EmitScalarExpr(E->getArg(1));
89:     Value *Ng = EmitScalarExpr(E->getArg(2));
90:     assert(E->getArg(0)->getType()->hasFloatingRepresentation() &&
91:            E->getArg(1)->getType()->hasFloatingRepresentation() &&
92:            E->getArg(2)->getType()->hasFloatingRepresentation() &&
93:            "FaceForward operands must have a float representation");
94:     return Builder.CreateIntrinsic(
95:         /*ReturnType=*/N->getType(), Intrinsic::spv_faceforward,
96:         ArrayRef<Value *>{N, I, Ng}, /*FMFSource=*/nullptr, "spv.faceforward");
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 97-108
```cpp
 97:   }
 98:   case SPIRV::BI__builtin_spirv_generic_cast_to_ptr_explicit: {
 99:     Value *Ptr = EmitScalarExpr(E->getArg(0));
100:     assert(E->getArg(0)->getType()->hasPointerRepresentation() &&
101:            E->getArg(1)->getType()->hasIntegerRepresentation() &&
102:            "GenericCastToPtrExplicit takes a pointer and an int");
103:     llvm::Type *Res = getTypes().ConvertType(E->getType());
104:     assert(Res->isPointerTy() &&
105:            "GenericCastToPtrExplicit doesn't return a pointer");
106:     llvm::CallInst *Call = Builder.CreateIntrinsic(
107:         /*ReturnType=*/Res, Intrinsic::spv_generic_cast_to_ptr_explicit,
108:         ArrayRef<Value *>{Ptr}, nullptr, "spv.generic_cast");
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 109-120
```cpp
109:     Call->addRetAttr(llvm::Attribute::AttrKind::NoUndef);
110:     return Call;
111:   }
112:   case SPIRV::BI__builtin_spirv_subgroup_shuffle: {
113:     Value *X = EmitScalarExpr(E->getArg(0));
114:     Value *Y = EmitScalarExpr(E->getArg(1));
115:     assert(E->getArg(1)->getType()->hasIntegerRepresentation());
116:     return Builder.CreateIntrinsic(
117:         /*ReturnType=*/getTypes().ConvertType(E->getArg(0)->getType()),
118:         Intrinsic::spv_wave_readlane, {X, Y}, nullptr, "spv.shuffle");
119:   }
120:   case SPIRV::BI__builtin_spirv_num_workgroups:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 121-132
```cpp
121:     return Builder.CreateIntrinsic(
122:         /*ReturnType=*/getTypes().ConvertType(E->getType()),
123:         Intrinsic::spv_num_workgroups,
124:         ArrayRef<Value *>{EmitScalarExpr(E->getArg(0))}, nullptr,
125:         "spv.num.workgroups");
126:   case SPIRV::BI__builtin_spirv_workgroup_size:
127:     return Builder.CreateIntrinsic(
128:         /*ReturnType=*/getTypes().ConvertType(E->getType()),
129:         Intrinsic::spv_workgroup_size,
130:         ArrayRef<Value *>{EmitScalarExpr(E->getArg(0))}, nullptr,
131:         "spv.workgroup.size");
132:   case SPIRV::BI__builtin_spirv_workgroup_id:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 133-144
```cpp
133:     return Builder.CreateIntrinsic(
134:         /*ReturnType=*/getTypes().ConvertType(E->getType()),
135:         Intrinsic::spv_group_id,
136:         ArrayRef<Value *>{EmitScalarExpr(E->getArg(0))}, nullptr,
137:         "spv.group.id");
138:   case SPIRV::BI__builtin_spirv_local_invocation_id:
139:     return Builder.CreateIntrinsic(
140:         /*ReturnType=*/getTypes().ConvertType(E->getType()),
141:         Intrinsic::spv_thread_id_in_group,
142:         ArrayRef<Value *>{EmitScalarExpr(E->getArg(0))}, nullptr,
143:         "spv.thread.id.in.group");
144:   case SPIRV::BI__builtin_spirv_global_invocation_id:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 145-156
```cpp
145:     return Builder.CreateIntrinsic(
146:         /*ReturnType=*/getTypes().ConvertType(E->getType()),
147:         Intrinsic::spv_thread_id,
148:         ArrayRef<Value *>{EmitScalarExpr(E->getArg(0))}, nullptr,
149:         "spv.thread.id");
150:   case SPIRV::BI__builtin_spirv_global_size:
151:     return Builder.CreateIntrinsic(
152:         /*ReturnType=*/getTypes().ConvertType(E->getType()),
153:         Intrinsic::spv_global_size,
154:         ArrayRef<Value *>{EmitScalarExpr(E->getArg(0))}, nullptr,
155:         "spv.num.workgroups");
156:   case SPIRV::BI__builtin_spirv_global_offset:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 157-168
```cpp
157:     return Builder.CreateIntrinsic(
158:         /*ReturnType=*/getTypes().ConvertType(E->getType()),
159:         Intrinsic::spv_global_offset,
160:         ArrayRef<Value *>{EmitScalarExpr(E->getArg(0))}, nullptr,
161:         "spv.global.offset");
162:   case SPIRV::BI__builtin_spirv_ddx:
163:     return Builder.CreateIntrinsic(
164:         /*ReturnType=*/getTypes().ConvertType(E->getType()), Intrinsic::spv_ddx,
165:         ArrayRef<Value *>{EmitScalarExpr(E->getArg(0))}, nullptr, "spv.ddx");
166:   case SPIRV::BI__builtin_spirv_ddy:
167:     return Builder.CreateIntrinsic(
168:         /*ReturnType=*/getTypes().ConvertType(E->getType()), Intrinsic::spv_ddy,
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 169-177
```cpp
169:         ArrayRef<Value *>{EmitScalarExpr(E->getArg(0))}, nullptr, "spv.ddy");
170:   case SPIRV::BI__builtin_spirv_fwidth:
171:     return Builder.CreateIntrinsic(
172:         /*ReturnType=*/getTypes().ConvertType(E->getType()),
173:         Intrinsic::spv_fwidth, ArrayRef<Value *>{EmitScalarExpr(E->getArg(0))},
174:         nullptr, "spv.fwidth");
175:   }
176:   return nullptr;
177: }
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

## Key Concepts / 关键概念

- **getArg**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **EmitScalarExpr**: Suggests an entry point that materializes IR or helper objects for target builtin lowering. / 暗示其是为 目标内建函数降级 生成 IR 或辅助对象的入口。
- **getType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **SPIRV**: Central symbol in this file's implementation of target builtin lowering. / 是该文件实现 目标内建函数降级 时的核心符号。
- **Builder**: Acts as a construction helper that incrementally assembles target builtin lowering state. / 充当构建辅助器，逐步组装 目标内建函数降级 状态。
- **CreateIntrinsic**: Suggests an entry point that materializes IR or helper objects for target builtin lowering. / 暗示其是为 目标内建函数降级 生成 IR 或辅助对象的入口。
- **ArrayRef**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CGHLSLRuntime.h`, `CodeGenFunction.h`
- **Clang libraries / Clang 库**: `clang/Basic/TargetBuiltins.h`
- **LLVM libraries / LLVM 库**: `llvm/IR/Intrinsics.h`
