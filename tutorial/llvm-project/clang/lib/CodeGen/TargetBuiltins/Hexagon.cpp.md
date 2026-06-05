# Hexagon.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/TargetBuiltins/Hexagon.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements builtin handling and lowering logic for the Hexagon backend.
- **Purpose (CN) / 目的（中文）**: 实现 Hexagon 后端的内建函数处理与降级逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: //===------ Hexagon.cpp - Emit LLVM Code for builtins ---------------------===//
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
13: #include "CodeGenFunction.h"
14: #include "clang/Basic/TargetBuiltins.h"
15: #include "llvm/IR/IntrinsicsHexagon.h"
16: 
```
- **EN**: This block imports local CodeGen headers `CodeGenFunction.h`; Clang headers `clang/Basic/TargetBuiltins.h`; LLVM headers `llvm/IR/IntrinsicsHexagon.h`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CodeGenFunction.h`；Clang 头文件 `clang/Basic/TargetBuiltins.h`；LLVM 头文件 `llvm/IR/IntrinsicsHexagon.h`；包含影响本编译单元构建方式的预处理结构。

### Lines 17-32
```cpp
17: using namespace clang;
18: using namespace CodeGen;
19: using namespace llvm;
20: 
21: static std::pair<Intrinsic::ID, unsigned>
22: getIntrinsicForHexagonNonClangBuiltin(unsigned BuiltinID) {
23:   struct Info {
24:     unsigned BuiltinID;
25:     Intrinsic::ID IntrinsicID;
26:     unsigned VecLen;
27:   };
28:   static Info Infos[] = {
29: #define CUSTOM_BUILTIN_MAPPING(x,s) \
30:   { Hexagon::BI__builtin_HEXAGON_##x, Intrinsic::hexagon_##x, s },
31:     CUSTOM_BUILTIN_MAPPING(L2_loadrub_pci, 0)
32:     CUSTOM_BUILTIN_MAPPING(L2_loadrb_pci, 0)
```
- **EN**: This block opens or references namespaces `clang`, `CodeGen`, `llvm`; introduces declarations such as `Info`; defines callable entry points like `getIntrinsicForHexagonNonClangBuiltin`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块打开或引用命名空间 `clang`, `CodeGen`, `llvm`；给出诸如 `Info` 的声明；定义可调用入口，例如 `getIntrinsicForHexagonNonClangBuiltin`；包含影响本编译单元构建方式的预处理结构。

### Lines 33-48
```cpp
33:     CUSTOM_BUILTIN_MAPPING(L2_loadruh_pci, 0)
34:     CUSTOM_BUILTIN_MAPPING(L2_loadrh_pci, 0)
35:     CUSTOM_BUILTIN_MAPPING(L2_loadri_pci, 0)
36:     CUSTOM_BUILTIN_MAPPING(L2_loadrd_pci, 0)
37:     CUSTOM_BUILTIN_MAPPING(L2_loadrub_pcr, 0)
38:     CUSTOM_BUILTIN_MAPPING(L2_loadrb_pcr, 0)
39:     CUSTOM_BUILTIN_MAPPING(L2_loadruh_pcr, 0)
40:     CUSTOM_BUILTIN_MAPPING(L2_loadrh_pcr, 0)
41:     CUSTOM_BUILTIN_MAPPING(L2_loadri_pcr, 0)
42:     CUSTOM_BUILTIN_MAPPING(L2_loadrd_pcr, 0)
43:     CUSTOM_BUILTIN_MAPPING(S2_storerb_pci, 0)
44:     CUSTOM_BUILTIN_MAPPING(S2_storerh_pci, 0)
45:     CUSTOM_BUILTIN_MAPPING(S2_storerf_pci, 0)
46:     CUSTOM_BUILTIN_MAPPING(S2_storeri_pci, 0)
47:     CUSTOM_BUILTIN_MAPPING(S2_storerd_pci, 0)
48:     CUSTOM_BUILTIN_MAPPING(S2_storerb_pcr, 0)
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 49-64
```cpp
49:     CUSTOM_BUILTIN_MAPPING(S2_storerh_pcr, 0)
50:     CUSTOM_BUILTIN_MAPPING(S2_storerf_pcr, 0)
51:     CUSTOM_BUILTIN_MAPPING(S2_storeri_pcr, 0)
52:     CUSTOM_BUILTIN_MAPPING(S2_storerd_pcr, 0)
53:     // Legacy builtins that take a vector in place of a vector predicate.
54:     CUSTOM_BUILTIN_MAPPING(V6_vmaskedstoreq, 64)
55:     CUSTOM_BUILTIN_MAPPING(V6_vmaskedstorenq, 64)
56:     CUSTOM_BUILTIN_MAPPING(V6_vmaskedstorentq, 64)
57:     CUSTOM_BUILTIN_MAPPING(V6_vmaskedstorentnq, 64)
58:     CUSTOM_BUILTIN_MAPPING(V6_vmaskedstoreq_128B, 128)
59:     CUSTOM_BUILTIN_MAPPING(V6_vmaskedstorenq_128B, 128)
60:     CUSTOM_BUILTIN_MAPPING(V6_vmaskedstorentq_128B, 128)
61:     CUSTOM_BUILTIN_MAPPING(V6_vmaskedstorentnq_128B, 128)
62: #include "clang/Basic/BuiltinsHexagonMapCustomDep.def"
63: #undef CUSTOM_BUILTIN_MAPPING
64:   };
```
- **EN**: This block imports Clang headers `clang/Basic/BuiltinsHexagonMapCustomDep.def`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/Basic/BuiltinsHexagonMapCustomDep.def`；包含影响本编译单元构建方式的预处理结构。

### Lines 65-80
```cpp
65: 
66:   auto CmpInfo = [] (Info A, Info B) { return A.BuiltinID < B.BuiltinID; };
67:   static const bool SortOnce = (llvm::sort(Infos, CmpInfo), true);
68:   (void)SortOnce;
69: 
70:   const Info *F = llvm::lower_bound(Infos, Info{BuiltinID, 0, 0}, CmpInfo);
71:   if (F == std::end(Infos) || F->BuiltinID != BuiltinID)
72:     return {Intrinsic::not_intrinsic, 0};
73: 
74:   return {F->IntrinsicID, F->VecLen};
75: }
76: 
77: Value *CodeGenFunction::EmitHexagonBuiltinExpr(unsigned BuiltinID,
78:                                                const CallExpr *E) {
79:   Intrinsic::ID ID;
80:   unsigned VecLen;
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 81-96
```cpp
81:   std::tie(ID, VecLen) = getIntrinsicForHexagonNonClangBuiltin(BuiltinID);
82: 
83:   auto MakeCircOp = [this, E](unsigned IntID, bool IsLoad) {
84:     // The base pointer is passed by address, so it needs to be loaded.
85:     Address A = EmitPointerWithAlignment(E->getArg(0));
86:     Address BP = Address(A.emitRawPointer(*this), Int8PtrTy, A.getAlignment());
87:     llvm::Value *Base = Builder.CreateLoad(BP);
88:     // The treatment of both loads and stores is the same: the arguments for
89:     // the builtin are the same as the arguments for the intrinsic.
90:     // Load:
91:     //   builtin(Base, Inc, Mod, Start) -> intr(Base, Inc, Mod, Start)
92:     //   builtin(Base, Mod, Start)      -> intr(Base, Mod, Start)
93:     // Store:
94:     //   builtin(Base, Inc, Mod, Val, Start) -> intr(Base, Inc, Mod, Val, Start)
95:     //   builtin(Base, Mod, Val, Start)      -> intr(Base, Mod, Val, Start)
96:     SmallVector<llvm::Value*,5> Ops = { Base };
```
- **EN**: This block defines callable entry points like `tie`.
- **CN**: 该代码块定义可调用入口，例如 `tie`。

### Lines 97-112
```cpp
 97:     for (unsigned i = 1, e = E->getNumArgs(); i != e; ++i)
 98:       Ops.push_back(EmitScalarExpr(E->getArg(i)));
 99: 
100:     llvm::Value *Result = Builder.CreateCall(CGM.getIntrinsic(IntID), Ops);
101:     // The load intrinsics generate two results (Value, NewBase), stores
102:     // generate one (NewBase). The new base address needs to be stored.
103:     llvm::Value *NewBase = IsLoad ? Builder.CreateExtractValue(Result, 1)
104:                                   : Result;
105:     llvm::Value *LV = EmitScalarExpr(E->getArg(0));
106:     Address Dest = EmitPointerWithAlignment(E->getArg(0));
107:     llvm::Value *RetVal =
108:         Builder.CreateAlignedStore(NewBase, LV, Dest.getAlignment());
109:     if (IsLoad)
110:       RetVal = Builder.CreateExtractValue(Result, 0);
111:     return RetVal;
112:   };
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 113-128
```cpp
113: 
114:   // Handle the conversion of bit-reverse load intrinsics to bit code.
115:   // The intrinsic call after this function only reads from memory and the
116:   // write to memory is dealt by the store instruction.
117:   auto MakeBrevLd = [this, E](unsigned IntID, llvm::Type *DestTy) {
118:     // The intrinsic generates one result, which is the new value for the base
119:     // pointer. It needs to be returned. The result of the load instruction is
120:     // passed to intrinsic by address, so the value needs to be stored.
121:     llvm::Value *BaseAddress = EmitScalarExpr(E->getArg(0));
122: 
123:     // Expressions like &(*pt++) will be incremented per evaluation.
124:     // EmitPointerWithAlignment and EmitScalarExpr evaluates the expression
125:     // per call.
126:     Address DestAddr = EmitPointerWithAlignment(E->getArg(1));
127:     DestAddr = DestAddr.withElementType(Int8Ty);
128:     llvm::Value *DestAddress = DestAddr.emitRawPointer(*this);
```
- **EN**: This block documents intent or context for the surrounding target builtin lowering code.
- **CN**: 该代码块说明周围 目标内建函数降级 代码的意图或上下文。

### Lines 129-144
```cpp
129: 
130:     // Operands are Base, Dest, Modifier.
131:     // The intrinsic format in LLVM IR is defined as
132:     // { ValueType, i8* } (i8*, i32).
133:     llvm::Value *Result = Builder.CreateCall(
134:         CGM.getIntrinsic(IntID), {BaseAddress, EmitScalarExpr(E->getArg(2))});
135: 
136:     // The value needs to be stored as the variable is passed by reference.
137:     llvm::Value *DestVal = Builder.CreateExtractValue(Result, 0);
138: 
139:     // The store needs to be truncated to fit the destination type.
140:     // While i32 and i64 are natively supported on Hexagon, i8 and i16 needs
141:     // to be handled with stores of respective destination type.
142:     DestVal = Builder.CreateTrunc(DestVal, DestTy);
143: 
144:     Builder.CreateAlignedStore(DestVal, DestAddress, DestAddr.getAlignment());
```
- **EN**: This block documents intent or context for the surrounding target builtin lowering code.
- **CN**: 该代码块说明周围 目标内建函数降级 代码的意图或上下文。

### Lines 145-160
```cpp
145:     // The updated value of the base pointer is returned.
146:     return Builder.CreateExtractValue(Result, 1);
147:   };
148: 
149:   auto V2Q = [this, VecLen] (llvm::Value *Vec) {
150:     Intrinsic::ID ID = VecLen == 128 ? Intrinsic::hexagon_V6_vandvrt_128B
151:                                      : Intrinsic::hexagon_V6_vandvrt;
152:     return Builder.CreateCall(CGM.getIntrinsic(ID),
153:                               {Vec, Builder.getInt32(-1)});
154:   };
155:   auto Q2V = [this, VecLen] (llvm::Value *Pred) {
156:     Intrinsic::ID ID = VecLen == 128 ? Intrinsic::hexagon_V6_vandqrt_128B
157:                                      : Intrinsic::hexagon_V6_vandqrt;
158:     return Builder.CreateCall(CGM.getIntrinsic(ID),
159:                               {Pred, Builder.getInt32(-1)});
160:   };
```
- **EN**: This block documents intent or context for the surrounding target builtin lowering code.
- **CN**: 该代码块说明周围 目标内建函数降级 代码的意图或上下文。

### Lines 161-176
```cpp
161: 
162:   switch (BuiltinID) {
163:   // These intrinsics return a tuple {Vector, VectorPred} in LLVM IR,
164:   // and the corresponding C/C++ builtins use loads/stores to update
165:   // the predicate.
166:   case Hexagon::BI__builtin_HEXAGON_V6_vaddcarry:
167:   case Hexagon::BI__builtin_HEXAGON_V6_vaddcarry_128B:
168:   case Hexagon::BI__builtin_HEXAGON_V6_vsubcarry:
169:   case Hexagon::BI__builtin_HEXAGON_V6_vsubcarry_128B: {
170:     // Get the type from the 0-th argument.
171:     llvm::Type *VecType = ConvertType(E->getArg(0)->getType());
172:     Address PredAddr =
173:         EmitPointerWithAlignment(E->getArg(2)).withElementType(VecType);
174:     llvm::Value *PredIn = V2Q(Builder.CreateLoad(PredAddr));
175:     llvm::Value *Result = Builder.CreateCall(CGM.getIntrinsic(ID),
176:         {EmitScalarExpr(E->getArg(0)), EmitScalarExpr(E->getArg(1)), PredIn});
```
- **EN**: This block defines callable entry points like `EmitPointerWithAlignment`; uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitPointerWithAlignment`；通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 177-192
```cpp
177: 
178:     llvm::Value *PredOut = Builder.CreateExtractValue(Result, 1);
179:     Builder.CreateAlignedStore(Q2V(PredOut), PredAddr.emitRawPointer(*this),
180:                                PredAddr.getAlignment());
181:     return Builder.CreateExtractValue(Result, 0);
182:   }
183:   // These are identical to the builtins above, except they don't consume
184:   // input carry, only generate carry-out. Since they still produce two
185:   // outputs, generate the store of the predicate, but no load.
186:   case Hexagon::BI__builtin_HEXAGON_V6_vaddcarryo:
187:   case Hexagon::BI__builtin_HEXAGON_V6_vaddcarryo_128B:
188:   case Hexagon::BI__builtin_HEXAGON_V6_vsubcarryo:
189:   case Hexagon::BI__builtin_HEXAGON_V6_vsubcarryo_128B: {
190:     // Get the type from the 0-th argument.
191:     llvm::Type *VecType = ConvertType(E->getArg(0)->getType());
192:     Address PredAddr =
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 193-208
```cpp
193:         EmitPointerWithAlignment(E->getArg(2)).withElementType(VecType);
194:     llvm::Value *Result = Builder.CreateCall(CGM.getIntrinsic(ID),
195:         {EmitScalarExpr(E->getArg(0)), EmitScalarExpr(E->getArg(1))});
196: 
197:     llvm::Value *PredOut = Builder.CreateExtractValue(Result, 1);
198:     Builder.CreateAlignedStore(Q2V(PredOut), PredAddr.emitRawPointer(*this),
199:                                PredAddr.getAlignment());
200:     return Builder.CreateExtractValue(Result, 0);
201:   }
202: 
203:   case Hexagon::BI__builtin_HEXAGON_V6_vmaskedstoreq:
204:   case Hexagon::BI__builtin_HEXAGON_V6_vmaskedstorenq:
205:   case Hexagon::BI__builtin_HEXAGON_V6_vmaskedstorentq:
206:   case Hexagon::BI__builtin_HEXAGON_V6_vmaskedstorentnq:
207:   case Hexagon::BI__builtin_HEXAGON_V6_vmaskedstoreq_128B:
208:   case Hexagon::BI__builtin_HEXAGON_V6_vmaskedstorenq_128B:
```
- **EN**: This block defines callable entry points like `EmitPointerWithAlignment`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitPointerWithAlignment`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 209-224
```cpp
209:   case Hexagon::BI__builtin_HEXAGON_V6_vmaskedstorentq_128B:
210:   case Hexagon::BI__builtin_HEXAGON_V6_vmaskedstorentnq_128B: {
211:     SmallVector<llvm::Value*,4> Ops;
212:     const Expr *PredOp = E->getArg(0);
213:     // There will be an implicit cast to a boolean vector. Strip it.
214:     if (auto *Cast = dyn_cast<ImplicitCastExpr>(PredOp)) {
215:       if (Cast->getCastKind() == CK_BitCast)
216:         PredOp = Cast->getSubExpr();
217:       Ops.push_back(V2Q(EmitScalarExpr(PredOp)));
218:     }
219:     for (int i = 1, e = E->getNumArgs(); i != e; ++i)
220:       Ops.push_back(EmitScalarExpr(E->getArg(i)));
221:     return Builder.CreateCall(CGM.getIntrinsic(ID), Ops);
222:   }
223: 
224:   case Hexagon::BI__builtin_HEXAGON_L2_loadrub_pci:
```
- **EN**: This block uses control flow (if, for, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, for, case）细化 目标内建函数降级 行为。

### Lines 225-240
```cpp
225:   case Hexagon::BI__builtin_HEXAGON_L2_loadrb_pci:
226:   case Hexagon::BI__builtin_HEXAGON_L2_loadruh_pci:
227:   case Hexagon::BI__builtin_HEXAGON_L2_loadrh_pci:
228:   case Hexagon::BI__builtin_HEXAGON_L2_loadri_pci:
229:   case Hexagon::BI__builtin_HEXAGON_L2_loadrd_pci:
230:   case Hexagon::BI__builtin_HEXAGON_L2_loadrub_pcr:
231:   case Hexagon::BI__builtin_HEXAGON_L2_loadrb_pcr:
232:   case Hexagon::BI__builtin_HEXAGON_L2_loadruh_pcr:
233:   case Hexagon::BI__builtin_HEXAGON_L2_loadrh_pcr:
234:   case Hexagon::BI__builtin_HEXAGON_L2_loadri_pcr:
235:   case Hexagon::BI__builtin_HEXAGON_L2_loadrd_pcr:
236:     return MakeCircOp(ID, /*IsLoad=*/true);
237:   case Hexagon::BI__builtin_HEXAGON_S2_storerb_pci:
238:   case Hexagon::BI__builtin_HEXAGON_S2_storerh_pci:
239:   case Hexagon::BI__builtin_HEXAGON_S2_storerf_pci:
240:   case Hexagon::BI__builtin_HEXAGON_S2_storeri_pci:
```
- **EN**: This block spells out callable entry points like `MakeCircOp`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeCircOp`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 241-256
```cpp
241:   case Hexagon::BI__builtin_HEXAGON_S2_storerd_pci:
242:   case Hexagon::BI__builtin_HEXAGON_S2_storerb_pcr:
243:   case Hexagon::BI__builtin_HEXAGON_S2_storerh_pcr:
244:   case Hexagon::BI__builtin_HEXAGON_S2_storerf_pcr:
245:   case Hexagon::BI__builtin_HEXAGON_S2_storeri_pcr:
246:   case Hexagon::BI__builtin_HEXAGON_S2_storerd_pcr:
247:     return MakeCircOp(ID, /*IsLoad=*/false);
248:   case Hexagon::BI__builtin_brev_ldub:
249:     return MakeBrevLd(Intrinsic::hexagon_L2_loadrub_pbr, Int8Ty);
250:   case Hexagon::BI__builtin_brev_ldb:
251:     return MakeBrevLd(Intrinsic::hexagon_L2_loadrb_pbr, Int8Ty);
252:   case Hexagon::BI__builtin_brev_lduh:
253:     return MakeBrevLd(Intrinsic::hexagon_L2_loadruh_pbr, Int16Ty);
254:   case Hexagon::BI__builtin_brev_ldh:
255:     return MakeBrevLd(Intrinsic::hexagon_L2_loadrh_pbr, Int16Ty);
256:   case Hexagon::BI__builtin_brev_ldw:
```
- **EN**: This block spells out callable entry points like `MakeCircOp`, `MakeBrevLd`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeCircOp`, `MakeBrevLd`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 257-263
```cpp
257:     return MakeBrevLd(Intrinsic::hexagon_L2_loadri_pbr, Int32Ty);
258:   case Hexagon::BI__builtin_brev_ldd:
259:     return MakeBrevLd(Intrinsic::hexagon_L2_loadrd_pbr, Int64Ty);
260:   } // switch
261: 
262:   return nullptr;
263: }
```
- **EN**: This block spells out callable entry points like `MakeBrevLd`; uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeBrevLd`；通过控制流（switch, case）细化 目标内建函数降级 行为。

## Key Concepts / 关键概念

- **CUSTOM_BUILTIN_MAPPING**: Central symbol in this file's implementation of target builtin lowering. / 是该文件实现 目标内建函数降级 时的核心符号。
- **Builder**: Acts as a construction helper that incrementally assembles target builtin lowering state. / 充当构建辅助器，逐步组装 目标内建函数降级 状态。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Intrinsic**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **getArg**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Result**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **BuiltinID**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **EmitScalarExpr**: Suggests an entry point that materializes IR or helper objects for target builtin lowering. / 暗示其是为 目标内建函数降级 生成 IR 或辅助对象的入口。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CodeGenFunction.h`
- **Clang libraries / Clang 库**: `clang/Basic/TargetBuiltins.h`, `clang/Basic/BuiltinsHexagonMapCustomDep.def`
- **LLVM libraries / LLVM 库**: `llvm/IR/IntrinsicsHexagon.h`
