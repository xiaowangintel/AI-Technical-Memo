# RISCV.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/TargetBuiltins/RISCV.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements builtin handling and lowering logic for the RISCV backend.
- **Purpose (CN) / 目的（中文）**: 实现 RISCV 后端的内建函数处理与降级逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: //===-------- RISCV.cpp - Emit LLVM Code for builtins ---------------------===//
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
15: #include "llvm/IR/IntrinsicsRISCV.h"
16: #include "llvm/TargetParser/RISCVISAInfo.h"
17: #include "llvm/TargetParser/RISCVTargetParser.h"
18: 
19: using namespace clang;
20: using namespace CodeGen;
```
- **EN**: This block imports local CodeGen headers `CodeGenFunction.h`; Clang headers `clang/Basic/TargetBuiltins.h`; LLVM headers `llvm/IR/IntrinsicsRISCV.h`, `llvm/TargetParser/RISCVISAInfo.h`, `llvm/TargetParser/RISCVTargetParser.h`; opens or references namespaces `clang`, `CodeGen`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CodeGenFunction.h`；Clang 头文件 `clang/Basic/TargetBuiltins.h`；LLVM 头文件 `llvm/IR/IntrinsicsRISCV.h`, `llvm/TargetParser/RISCVISAInfo.h`, `llvm/TargetParser/RISCVTargetParser.h`；打开或引用命名空间 `clang`, `CodeGen`；包含影响本编译单元构建方式的预处理结构。

### Lines 21-40
```cpp
21: using namespace llvm;
22: 
23: // The 0th bit simulates the `vta` of RVV
24: // The 1st bit simulates the `vma` of RVV
25: static constexpr unsigned RVV_VTA = 0x1;
26: static constexpr unsigned RVV_VMA = 0x2;
27: 
28: // RISC-V Vector builtin helper functions are marked NOINLINE to prevent
29: // excessive inlining in CodeGenFunction::EmitRISCVBuiltinExpr's large switch
30: // statement, which would significantly increase compilation time.
31: static LLVM_ATTRIBUTE_NOINLINE Value *
32: emitRVVVLEFFBuiltin(CodeGenFunction *CGF, const CallExpr *E,
33:                     ReturnValueSlot ReturnValue, llvm::Type *ResultType,
34:                     Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
35:                     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
36:   auto &Builder = CGF->Builder;
37:   auto &CGM = CGF->CGM;
38:   llvm::SmallVector<llvm::Type *, 3> IntrinsicTypes;
39:   if (IsMasked) {
40:     // Move mask to right before vl.
```
- **EN**: This block opens or references namespaces `llvm`; defines callable entry points like `emitRVVVLEFFBuiltin`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块打开或引用命名空间 `llvm`；定义可调用入口，例如 `emitRVVVLEFFBuiltin`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 41-60
```cpp
41:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 1);
42:     if ((PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA))
43:       Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
44:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
45:     IntrinsicTypes = {ResultType, Ops[4]->getType(), Ops[2]->getType()};
46:   } else {
47:     if (PolicyAttrs & RVV_VTA)
48:       Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
49:     IntrinsicTypes = {ResultType, Ops[3]->getType(), Ops[1]->getType()};
50:   }
51:   Value *NewVL = Ops[2];
52:   Ops.erase(Ops.begin() + 2);
53:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
54:   llvm::Value *LoadValue = Builder.CreateCall(F, Ops, "");
55:   llvm::Value *V = Builder.CreateExtractValue(LoadValue, {0});
56:   // Store new_vl.
57:   clang::CharUnits Align;
58:   if (IsMasked)
59:     Align = CGM.getNaturalPointeeTypeAlignment(
60:         E->getArg(E->getNumArgs() - 2)->getType());
```
- **EN**: This block defines callable entry points like `rotate`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `rotate`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 61-80
```cpp
61:   else
62:     Align = CGM.getNaturalPointeeTypeAlignment(E->getArg(1)->getType());
63:   llvm::Value *Val = Builder.CreateExtractValue(LoadValue, {1});
64:   Builder.CreateStore(Val, Address(NewVL, Val->getType(), Align));
65:   return V;
66: }
67: 
68: static LLVM_ATTRIBUTE_NOINLINE Value *
69: emitRVVVSSEBuiltin(CodeGenFunction *CGF, const CallExpr *E,
70:                    ReturnValueSlot ReturnValue, llvm::Type *ResultType,
71:                    Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
72:                    int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
73:   auto &Builder = CGF->Builder;
74:   auto &CGM = CGF->CGM;
75:   llvm::SmallVector<llvm::Type *, 3> IntrinsicTypes;
76:   if (IsMasked) {
77:     // Builtin: (mask, ptr, stride, value, vl). Intrinsic: (value, ptr, stride,
78:     // mask, vl)
79:     std::swap(Ops[0], Ops[3]);
80:   } else {
```
- **EN**: This block defines callable entry points like `emitRVVVSSEBuiltin`, `swap`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitRVVVSSEBuiltin`, `swap`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 81-100
```cpp
 81:     // Builtin: (ptr, stride, value, vl). Intrinsic: (value, ptr, stride, vl)
 82:     std::rotate(Ops.begin(), Ops.begin() + 2, Ops.begin() + 3);
 83:   }
 84:   if (IsMasked)
 85:     IntrinsicTypes = {Ops[0]->getType(), Ops[1]->getType(), Ops[4]->getType()};
 86:   else
 87:     IntrinsicTypes = {Ops[0]->getType(), Ops[1]->getType(), Ops[3]->getType()};
 88:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
 89:   return Builder.CreateCall(F, Ops, "");
 90: }
 91: 
 92: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVIndexedStoreBuiltin(
 93:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
 94:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
 95:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
 96:   auto &Builder = CGF->Builder;
 97:   auto &CGM = CGF->CGM;
 98:   llvm::SmallVector<llvm::Type *, 4> IntrinsicTypes;
 99:   if (IsMasked) {
100:     // Builtin: (mask, ptr, index, value, vl).
```
- **EN**: This block defines callable entry points like `rotate`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `rotate`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 101-120
```cpp
101:     // Intrinsic: (value, ptr, index, mask, vl)
102:     std::swap(Ops[0], Ops[3]);
103:   } else {
104:     // Builtin: (ptr, index, value, vl).
105:     // Intrinsic: (value, ptr, index, vl)
106:     std::rotate(Ops.begin(), Ops.begin() + 2, Ops.begin() + 3);
107:   }
108:   if (IsMasked)
109:     IntrinsicTypes = {Ops[0]->getType(), Ops[1]->getType(), Ops[2]->getType(),
110:                       Ops[4]->getType()};
111:   else
112:     IntrinsicTypes = {Ops[0]->getType(), Ops[1]->getType(), Ops[2]->getType(),
113:                       Ops[3]->getType()};
114:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
115:   return Builder.CreateCall(F, Ops, "");
116: }
117: 
118: static LLVM_ATTRIBUTE_NOINLINE Value *
119: emitRVVPseudoUnaryBuiltin(CodeGenFunction *CGF, const CallExpr *E,
120:                           ReturnValueSlot ReturnValue, llvm::Type *ResultType,
```
- **EN**: This block defines callable entry points like `swap`, `rotate`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `swap`, `rotate`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 121-140
```cpp
121:                           Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
122:                           int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
123:   auto &Builder = CGF->Builder;
124:   auto &CGM = CGF->CGM;
125:   llvm::SmallVector<llvm::Type *, 3> IntrinsicTypes;
126:   if (IsMasked) {
127:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 1);
128:     if ((PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA))
129:       Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
130:   } else {
131:     if (PolicyAttrs & RVV_VTA)
132:       Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
133:   }
134:   auto ElemTy = cast<llvm::VectorType>(ResultType)->getElementType();
135:   Ops.insert(Ops.begin() + 2, llvm::Constant::getNullValue(ElemTy));
136:   if (IsMasked) {
137:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
138:     // maskedoff, op1, op2, mask, vl, policy
139:     IntrinsicTypes = {ResultType, ElemTy, Ops[4]->getType()};
140:   } else {
```
- **EN**: This block defines callable entry points like `rotate`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `rotate`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 141-160
```cpp
141:     // passthru, op1, op2, vl
142:     IntrinsicTypes = {ResultType, ElemTy, Ops[3]->getType()};
143:   }
144:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
145:   return Builder.CreateCall(F, Ops, "");
146: }
147: 
148: static LLVM_ATTRIBUTE_NOINLINE Value *
149: emitRVVPseudoVNotBuiltin(CodeGenFunction *CGF, const CallExpr *E,
150:                          ReturnValueSlot ReturnValue, llvm::Type *ResultType,
151:                          Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
152:                          int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
153:   auto &Builder = CGF->Builder;
154:   auto &CGM = CGF->CGM;
155:   llvm::SmallVector<llvm::Type *, 3> IntrinsicTypes;
156:   if (IsMasked) {
157:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 1);
158:     if ((PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA))
159:       Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
160:   } else {
```
- **EN**: This block defines callable entry points like `emitRVVPseudoVNotBuiltin`, `rotate`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitRVVPseudoVNotBuiltin`, `rotate`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 161-180
```cpp
161:     if (PolicyAttrs & RVV_VTA)
162:       Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
163:   }
164:   auto ElemTy = cast<llvm::VectorType>(ResultType)->getElementType();
165:   Ops.insert(Ops.begin() + 2, llvm::Constant::getAllOnesValue(ElemTy));
166:   if (IsMasked) {
167:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
168:     // maskedoff, op1, po2, mask, vl, policy
169:     IntrinsicTypes = {ResultType, ElemTy, Ops[4]->getType()};
170:   } else {
171:     // passthru, op1, op2, vl
172:     IntrinsicTypes = {ResultType, ElemTy, Ops[3]->getType()};
173:   }
174:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
175:   return Builder.CreateCall(F, Ops, "");
176: }
177: 
178: static LLVM_ATTRIBUTE_NOINLINE Value *
179: emitRVVPseudoMaskBuiltin(CodeGenFunction *CGF, const CallExpr *E,
180:                          ReturnValueSlot ReturnValue, llvm::Type *ResultType,
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 181-200
```cpp
181:                          Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
182:                          int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
183:   auto &Builder = CGF->Builder;
184:   auto &CGM = CGF->CGM;
185:   llvm::SmallVector<llvm::Type *, 3> IntrinsicTypes;
186:   // op1, vl
187:   IntrinsicTypes = {ResultType, Ops[1]->getType()};
188:   Ops.insert(Ops.begin() + 1, Ops[0]);
189:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
190:   return Builder.CreateCall(F, Ops, "");
191: }
192: 
193: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVPseudoVFUnaryBuiltin(
194:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
195:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
196:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
197:   auto &Builder = CGF->Builder;
198:   auto &CGM = CGF->CGM;
199:   llvm::SmallVector<llvm::Type *, 3> IntrinsicTypes;
200:   if (IsMasked) {
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 201-220
```cpp
201:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 1);
202:     if ((PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA))
203:       Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
204:     Ops.insert(Ops.begin() + 2, Ops[1]);
205:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
206:     // maskedoff, op1, op2, mask, vl
207:     IntrinsicTypes = {ResultType, Ops[2]->getType(), Ops.back()->getType()};
208:   } else {
209:     if (PolicyAttrs & RVV_VTA)
210:       Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
211:     // op1, po2, vl
212:     IntrinsicTypes = {ResultType, Ops[1]->getType(), Ops[2]->getType()};
213:     Ops.insert(Ops.begin() + 2, Ops[1]);
214:   }
215:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
216:   return Builder.CreateCall(F, Ops, "");
217: }
218: 
219: static LLVM_ATTRIBUTE_NOINLINE Value *
220: emitRVVPseudoVWCVTBuiltin(CodeGenFunction *CGF, const CallExpr *E,
```
- **EN**: This block defines callable entry points like `rotate`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `rotate`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 221-240
```cpp
221:                           ReturnValueSlot ReturnValue, llvm::Type *ResultType,
222:                           Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
223:                           int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
224:   auto &Builder = CGF->Builder;
225:   auto &CGM = CGF->CGM;
226:   llvm::SmallVector<llvm::Type *, 4> IntrinsicTypes;
227:   if (IsMasked) {
228:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 1);
229:     if ((PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA))
230:       Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
231:   } else {
232:     if (PolicyAttrs & RVV_VTA)
233:       Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
234:   }
235:   auto ElemTy = cast<llvm::VectorType>(Ops[1]->getType())->getElementType();
236:   Ops.insert(Ops.begin() + 2, llvm::Constant::getNullValue(ElemTy));
237:   if (IsMasked) {
238:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
239:     // maskedoff, op1, op2, mask, vl, policy
240:     IntrinsicTypes = {ResultType, Ops[1]->getType(), ElemTy, Ops[4]->getType()};
```
- **EN**: This block defines callable entry points like `rotate`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `rotate`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 241-260
```cpp
241:   } else {
242:     // passtru, op1, op2, vl
243:     IntrinsicTypes = {ResultType, Ops[1]->getType(), ElemTy, Ops[3]->getType()};
244:   }
245:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
246:   return Builder.CreateCall(F, Ops, "");
247: }
248: 
249: static LLVM_ATTRIBUTE_NOINLINE Value *
250: emitRVVPseudoVNCVTBuiltin(CodeGenFunction *CGF, const CallExpr *E,
251:                           ReturnValueSlot ReturnValue, llvm::Type *ResultType,
252:                           Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
253:                           int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
254:   auto &Builder = CGF->Builder;
255:   auto &CGM = CGF->CGM;
256:   llvm::SmallVector<llvm::Type *, 4> IntrinsicTypes;
257:   if (IsMasked) {
258:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 1);
259:     if ((PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA))
260:       Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
```
- **EN**: This block defines callable entry points like `emitRVVPseudoVNCVTBuiltin`, `rotate`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitRVVPseudoVNCVTBuiltin`, `rotate`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 261-280
```cpp
261:   } else {
262:     if (PolicyAttrs & RVV_VTA)
263:       Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
264:   }
265:   Ops.insert(Ops.begin() + 2,
266:              llvm::Constant::getNullValue(Ops.back()->getType()));
267:   if (IsMasked) {
268:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
269:     // maskedoff, op1, xlen, mask, vl
270:     IntrinsicTypes = {ResultType, Ops[1]->getType(), Ops[4]->getType(),
271:                       Ops[4]->getType()};
272:   } else {
273:     // passthru, op1, xlen, vl
274:     IntrinsicTypes = {ResultType, Ops[1]->getType(), Ops[3]->getType(),
275:                       Ops[3]->getType()};
276:   }
277:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
278:   return Builder.CreateCall(F, Ops, "");
279: }
280: 
```
- **EN**: This block defines callable entry points like `getNullValue`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getNullValue`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 281-300
```cpp
281: static LLVM_ATTRIBUTE_NOINLINE Value *
282: emitRVVVlenbBuiltin(CodeGenFunction *CGF, const CallExpr *E,
283:                     ReturnValueSlot ReturnValue, llvm::Type *ResultType,
284:                     Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
285:                     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
286:   auto &Builder = CGF->Builder;
287:   auto &CGM = CGF->CGM;
288:   LLVMContext &Context = CGM.getLLVMContext();
289:   llvm::MDBuilder MDHelper(Context);
290:   llvm::Metadata *OpsMD[] = {llvm::MDString::get(Context, "vlenb")};
291:   llvm::MDNode *RegName = llvm::MDNode::get(Context, OpsMD);
292:   llvm::Value *Metadata = llvm::MetadataAsValue::get(Context, RegName);
293:   llvm::Function *F =
294:       CGM.getIntrinsic(llvm::Intrinsic::read_register, {CGF->SizeTy});
295:   return Builder.CreateCall(F, Metadata);
296: }
297: 
298: static LLVM_ATTRIBUTE_NOINLINE Value *
299: emitRVVVsetvliBuiltin(CodeGenFunction *CGF, const CallExpr *E,
300:                       ReturnValueSlot ReturnValue, llvm::Type *ResultType,
```
- **EN**: This block defines callable entry points like `emitRVVVlenbBuiltin`, `MDHelper`; returns or forwards computed values for the surrounding target builtin lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `emitRVVVlenbBuiltin`, `MDHelper`；为周围的 目标内建函数降级 逻辑返回或转发计算结果。

### Lines 301-320
```cpp
301:                       Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
302:                       int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
303:   auto &Builder = CGF->Builder;
304:   auto &CGM = CGF->CGM;
305:   llvm::Function *F = CGM.getIntrinsic(ID, {ResultType});
306:   return Builder.CreateCall(F, Ops, "");
307: }
308: 
309: static LLVM_ATTRIBUTE_NOINLINE Value *
310: emitRVVVSEMaskBuiltin(CodeGenFunction *CGF, const CallExpr *E,
311:                       ReturnValueSlot ReturnValue, llvm::Type *ResultType,
312:                       Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
313:                       int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
314:   auto &Builder = CGF->Builder;
315:   auto &CGM = CGF->CGM;
316:   llvm::SmallVector<llvm::Type *, 3> IntrinsicTypes;
317:   if (IsMasked) {
318:     // Builtin: (mask, ptr, value, vl).
319:     // Intrinsic: (value, ptr, mask, vl)
320:     std::swap(Ops[0], Ops[2]);
```
- **EN**: This block defines callable entry points like `emitRVVVSEMaskBuiltin`, `swap`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitRVVVSEMaskBuiltin`, `swap`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 321-340
```cpp
321:   } else {
322:     // Builtin: (ptr, value, vl).
323:     // Intrinsic: (value, ptr, vl)
324:     std::swap(Ops[0], Ops[1]);
325:   }
326:   if (IsMasked)
327:     IntrinsicTypes = {Ops[0]->getType(), Ops[1]->getType(), Ops[3]->getType()};
328:   else
329:     IntrinsicTypes = {Ops[0]->getType(), Ops[1]->getType(), Ops[2]->getType()};
330:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
331:   return Builder.CreateCall(F, Ops, "");
332: }
333: 
334: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVUnitStridedSegLoadTupleBuiltin(
335:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
336:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
337:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
338:   auto &Builder = CGF->Builder;
339:   auto &CGM = CGF->CGM;
340:   llvm::SmallVector<llvm::Type *, 4> IntrinsicTypes;
```
- **EN**: This block defines callable entry points like `swap`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `swap`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 341-360
```cpp
341:   bool NoPassthru =
342:       (IsMasked && (PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA)) |
343:       (!IsMasked && (PolicyAttrs & RVV_VTA));
344:   unsigned Offset = IsMasked ? NoPassthru ? 1 : 2 : NoPassthru ? 0 : 1;
345:   if (IsMasked)
346:     IntrinsicTypes = {ResultType, Ops[Offset]->getType(), Ops[0]->getType(),
347:                       Ops.back()->getType()};
348:   else
349:     IntrinsicTypes = {ResultType, Ops[Offset]->getType(),
350:                       Ops.back()->getType()};
351:   if (IsMasked)
352:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 1);
353:   if (NoPassthru)
354:     Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
355:   if (IsMasked)
356:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
357:   Ops.push_back(ConstantInt::get(Ops.back()->getType(), SegInstSEW));
358:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
359:   llvm::Value *LoadValue = Builder.CreateCall(F, Ops, "");
360:   if (ReturnValue.isNull())
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 361-380
```cpp
361:     return LoadValue;
362:   return Builder.CreateStore(LoadValue, ReturnValue.getValue());
363: }
364: 
365: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVUnitStridedSegStoreTupleBuiltin(
366:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
367:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
368:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
369:   auto &Builder = CGF->Builder;
370:   auto &CGM = CGF->CGM;
371:   llvm::SmallVector<llvm::Type *, 4> IntrinsicTypes;
372:   // Masked
373:   // Builtin: (mask, ptr, v_tuple, vl)
374:   // Intrinsic: (tuple, ptr, mask, vl, SegInstSEW)
375:   // Unmasked
376:   // Builtin: (ptr, v_tuple, vl)
377:   // Intrinsic: (tuple, ptr, vl, SegInstSEW)
378:   if (IsMasked)
379:     std::swap(Ops[0], Ops[2]);
380:   else
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 381-400
```cpp
381:     std::swap(Ops[0], Ops[1]);
382:   Ops.push_back(ConstantInt::get(Ops.back()->getType(), SegInstSEW));
383:   if (IsMasked)
384:     IntrinsicTypes = {Ops[0]->getType(), Ops[1]->getType(), Ops[2]->getType(),
385:                       Ops[3]->getType()};
386:   else
387:     IntrinsicTypes = {Ops[0]->getType(), Ops[1]->getType(), Ops[2]->getType()};
388:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
389:   return Builder.CreateCall(F, Ops, "");
390: }
391: 
392: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVUnitStridedSegLoadFFTupleBuiltin(
393:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
394:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
395:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
396:   auto &Builder = CGF->Builder;
397:   auto &CGM = CGF->CGM;
398:   llvm::SmallVector<llvm::Type *, 4> IntrinsicTypes;
399:   bool NoPassthru =
400:       (IsMasked && (PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA)) |
```
- **EN**: This block defines callable entry points like `swap`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `swap`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 401-420
```cpp
401:       (!IsMasked && (PolicyAttrs & RVV_VTA));
402:   unsigned Offset = IsMasked ? NoPassthru ? 1 : 2 : NoPassthru ? 0 : 1;
403:   if (IsMasked)
404:     IntrinsicTypes = {ResultType, Ops.back()->getType(), Ops[Offset]->getType(),
405:                       Ops[0]->getType()};
406:   else
407:     IntrinsicTypes = {ResultType, Ops.back()->getType(),
408:                       Ops[Offset]->getType()};
409:   if (IsMasked)
410:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 1);
411:   if (NoPassthru)
412:     Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
413:   if (IsMasked)
414:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
415:   Ops.push_back(ConstantInt::get(Ops.back()->getType(), SegInstSEW));
416:   Value *NewVL = Ops[2];
417:   Ops.erase(Ops.begin() + 2);
418:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
419:   llvm::Value *LoadValue = Builder.CreateCall(F, Ops, "");
420:   // Get alignment from the new vl operand
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 421-440
```cpp
421:   clang::CharUnits Align =
422:       CGM.getNaturalPointeeTypeAlignment(E->getArg(Offset + 1)->getType());
423:   llvm::Value *ReturnTuple = Builder.CreateExtractValue(LoadValue, 0);
424:   // Store new_vl
425:   llvm::Value *V = Builder.CreateExtractValue(LoadValue, 1);
426:   Builder.CreateStore(V, Address(NewVL, V->getType(), Align));
427:   if (ReturnValue.isNull())
428:     return ReturnTuple;
429:   return Builder.CreateStore(ReturnTuple, ReturnValue.getValue());
430: }
431: 
432: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVStridedSegLoadTupleBuiltin(
433:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
434:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
435:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
436:   auto &Builder = CGF->Builder;
437:   auto &CGM = CGF->CGM;
438:   llvm::SmallVector<llvm::Type *, 4> IntrinsicTypes;
439:   bool NoPassthru =
440:       (IsMasked && (PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA)) |
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 441-460
```cpp
441:       (!IsMasked && (PolicyAttrs & RVV_VTA));
442:   unsigned Offset = IsMasked ? NoPassthru ? 1 : 2 : NoPassthru ? 0 : 1;
443:   if (IsMasked)
444:     IntrinsicTypes = {ResultType, Ops[Offset]->getType(), Ops.back()->getType(),
445:                       Ops[0]->getType()};
446:   else
447:     IntrinsicTypes = {ResultType, Ops[Offset]->getType(),
448:                       Ops.back()->getType()};
449:   if (IsMasked)
450:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 1);
451:   if (NoPassthru)
452:     Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
453:   if (IsMasked)
454:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
455:   Ops.push_back(ConstantInt::get(Ops.back()->getType(), SegInstSEW));
456:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
457:   llvm::Value *LoadValue = Builder.CreateCall(F, Ops, "");
458:   if (ReturnValue.isNull())
459:     return LoadValue;
460:   return Builder.CreateStore(LoadValue, ReturnValue.getValue());
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 461-480
```cpp
461: }
462: 
463: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVStridedSegStoreTupleBuiltin(
464:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
465:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
466:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
467:   auto &Builder = CGF->Builder;
468:   auto &CGM = CGF->CGM;
469:   llvm::SmallVector<llvm::Type *, 4> IntrinsicTypes;
470:   // Masked
471:   // Builtin: (mask, ptr, stride, v_tuple, vl)
472:   // Intrinsic: (tuple, ptr, stride, mask, vl, SegInstSEW)
473:   // Unmasked
474:   // Builtin: (ptr, stride, v_tuple, vl)
475:   // Intrinsic: (tuple, ptr, stride, vl, SegInstSEW)
476:   if (IsMasked)
477:     std::swap(Ops[0], Ops[3]);
478:   else
479:     std::rotate(Ops.begin(), Ops.begin() + 2, Ops.begin() + 3);
480:   Ops.push_back(ConstantInt::get(Ops.back()->getType(), SegInstSEW));
```
- **EN**: This block defines callable entry points like `rotate`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `rotate`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 481-500
```cpp
481:   if (IsMasked)
482:     IntrinsicTypes = {Ops[0]->getType(), Ops[1]->getType(), Ops[4]->getType(),
483:                       Ops[3]->getType()};
484:   else
485:     IntrinsicTypes = {Ops[0]->getType(), Ops[1]->getType(), Ops[3]->getType()};
486:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
487:   return Builder.CreateCall(F, Ops, "");
488: }
489: 
490: static LLVM_ATTRIBUTE_NOINLINE Value *
491: emitRVVAveragingBuiltin(CodeGenFunction *CGF, const CallExpr *E,
492:                         ReturnValueSlot ReturnValue, llvm::Type *ResultType,
493:                         Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
494:                         int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
495:   auto &Builder = CGF->Builder;
496:   auto &CGM = CGF->CGM;
497:   // LLVM intrinsic
498:   // Unmasked: (passthru, op0, op1, round_mode, vl)
499:   // Masked:   (passthru, vector_in, vector_in/scalar_in, mask, vxrm, vl,
500:   // policy)
```
- **EN**: This block defines callable entry points like `emitRVVAveragingBuiltin`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitRVVAveragingBuiltin`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 501-520
```cpp
501: 
502:   bool HasMaskedOff =
503:       !((IsMasked && (PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA)) ||
504:         (!IsMasked && PolicyAttrs & RVV_VTA));
505: 
506:   if (IsMasked)
507:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 2);
508: 
509:   if (!HasMaskedOff)
510:     Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
511: 
512:   if (IsMasked)
513:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
514: 
515:   llvm::Function *F = CGM.getIntrinsic(
516:       ID, {ResultType, Ops[2]->getType(), Ops.back()->getType()});
517:   return Builder.CreateCall(F, Ops, "");
518: }
519: 
520: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVNarrowingClipBuiltin(
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 521-540
```cpp
521:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
522:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
523:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
524:   auto &Builder = CGF->Builder;
525:   auto &CGM = CGF->CGM;
526:   // LLVM intrinsic
527:   // Unmasked: (passthru, op0, op1, round_mode, vl)
528:   // Masked:   (passthru, vector_in, vector_in/scalar_in, mask, vxrm, vl,
529:   // policy)
530: 
531:   bool HasMaskedOff =
532:       !((IsMasked && (PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA)) ||
533:         (!IsMasked && PolicyAttrs & RVV_VTA));
534: 
535:   if (IsMasked)
536:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 2);
537: 
538:   if (!HasMaskedOff)
539:     Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
540: 
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 541-560
```cpp
541:   if (IsMasked)
542:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
543: 
544:   llvm::Function *F =
545:       CGM.getIntrinsic(ID, {ResultType, Ops[1]->getType(), Ops[2]->getType(),
546:                             Ops.back()->getType()});
547:   return Builder.CreateCall(F, Ops, "");
548: }
549: 
550: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVFloatingPointBuiltin(
551:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
552:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
553:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
554:   auto &Builder = CGF->Builder;
555:   auto &CGM = CGF->CGM;
556:   // LLVM intrinsic
557:   // Unmasked: (passthru, op0, op1, round_mode, vl)
558:   // Masked:   (passthru, vector_in, vector_in/scalar_in, mask, frm, vl, policy)
559: 
560:   bool HasMaskedOff =
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 561-580
```cpp
561:       !((IsMasked && (PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA)) ||
562:         (!IsMasked && PolicyAttrs & RVV_VTA));
563:   bool HasRoundModeOp =
564:       IsMasked ? (HasMaskedOff ? Ops.size() == 6 : Ops.size() == 5)
565:                : (HasMaskedOff ? Ops.size() == 5 : Ops.size() == 4);
566: 
567:   if (!HasRoundModeOp)
568:     Ops.insert(Ops.end() - 1,
569:                ConstantInt::get(Ops.back()->getType(), 7)); // frm
570: 
571:   if (IsMasked)
572:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 2);
573: 
574:   if (!HasMaskedOff)
575:     Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
576: 
577:   if (IsMasked)
578:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
579: 
580:   llvm::Function *F = CGM.getIntrinsic(
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 581-600
```cpp
581:       ID, {ResultType, Ops[2]->getType(), Ops.back()->getType()});
582:   return Builder.CreateCall(F, Ops, "");
583: }
584: 
585: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVWideningFloatingPointBuiltin(
586:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
587:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
588:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
589:   auto &Builder = CGF->Builder;
590:   auto &CGM = CGF->CGM;
591:   // LLVM intrinsic
592:   // Unmasked: (passthru, op0, op1, round_mode, vl)
593:   // Masked:   (passthru, vector_in, vector_in/scalar_in, mask, frm, vl, policy)
594: 
595:   bool HasMaskedOff =
596:       !((IsMasked && (PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA)) ||
597:         (!IsMasked && PolicyAttrs & RVV_VTA));
598:   bool HasRoundModeOp =
599:       IsMasked ? (HasMaskedOff ? Ops.size() == 6 : Ops.size() == 5)
600:                : (HasMaskedOff ? Ops.size() == 5 : Ops.size() == 4);
```
- **EN**: This block documents intent or context for the surrounding target builtin lowering code.
- **CN**: 该代码块说明周围 目标内建函数降级 代码的意图或上下文。

### Lines 601-620
```cpp
601: 
602:   if (!HasRoundModeOp)
603:     Ops.insert(Ops.end() - 1,
604:                ConstantInt::get(Ops.back()->getType(), 7)); // frm
605: 
606:   if (IsMasked)
607:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 2);
608: 
609:   if (!HasMaskedOff)
610:     Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
611: 
612:   if (IsMasked)
613:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
614: 
615:   llvm::Function *F =
616:       CGM.getIntrinsic(ID, {ResultType, Ops[1]->getType(), Ops[2]->getType(),
617:                             Ops.back()->getType()});
618:   return Builder.CreateCall(F, Ops, "");
619: }
620: 
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 621-640
```cpp
621: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVIndexedSegLoadTupleBuiltin(
622:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
623:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
624:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
625:   auto &Builder = CGF->Builder;
626:   auto &CGM = CGF->CGM;
627:   llvm::SmallVector<llvm::Type *, 5> IntrinsicTypes;
628: 
629:   bool NoPassthru =
630:       (IsMasked && (PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA)) |
631:       (!IsMasked && (PolicyAttrs & RVV_VTA));
632: 
633:   if (IsMasked)
634:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 1);
635:   if (NoPassthru)
636:     Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
637: 
638:   if (IsMasked)
639:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
640:   Ops.push_back(ConstantInt::get(Ops.back()->getType(), SegInstSEW));
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 641-660
```cpp
641: 
642:   if (IsMasked)
643:     IntrinsicTypes = {ResultType, Ops[1]->getType(), Ops[2]->getType(),
644:                       Ops[3]->getType(), Ops[4]->getType()};
645:   else
646:     IntrinsicTypes = {ResultType, Ops[1]->getType(), Ops[2]->getType(),
647:                       Ops[3]->getType()};
648:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
649:   llvm::Value *LoadValue = Builder.CreateCall(F, Ops, "");
650: 
651:   if (ReturnValue.isNull())
652:     return LoadValue;
653:   return Builder.CreateStore(LoadValue, ReturnValue.getValue());
654: }
655: 
656: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVIndexedSegStoreTupleBuiltin(
657:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
658:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
659:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
660:   auto &Builder = CGF->Builder;
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 661-680
```cpp
661:   auto &CGM = CGF->CGM;
662:   llvm::SmallVector<llvm::Type *, 5> IntrinsicTypes;
663:   // Masked
664:   // Builtin: (mask, ptr, index, v_tuple, vl)
665:   // Intrinsic: (tuple, ptr, index, mask, vl, SegInstSEW)
666:   // Unmasked
667:   // Builtin: (ptr, index, v_tuple, vl)
668:   // Intrinsic: (tuple, ptr, index, vl, SegInstSEW)
669: 
670:   if (IsMasked)
671:     std::swap(Ops[0], Ops[3]);
672:   else
673:     std::rotate(Ops.begin(), Ops.begin() + 2, Ops.begin() + 3);
674: 
675:   Ops.push_back(ConstantInt::get(Ops.back()->getType(), SegInstSEW));
676: 
677:   if (IsMasked)
678:     IntrinsicTypes = {Ops[0]->getType(), Ops[1]->getType(), Ops[2]->getType(),
679:                       Ops[3]->getType(), Ops[4]->getType()};
680:   else
```
- **EN**: This block defines callable entry points like `rotate`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `rotate`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 681-700
```cpp
681:     IntrinsicTypes = {Ops[0]->getType(), Ops[1]->getType(), Ops[2]->getType(),
682:                       Ops[3]->getType()};
683:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
684:   return Builder.CreateCall(F, Ops, "");
685: }
686: 
687: static LLVM_ATTRIBUTE_NOINLINE Value *
688: emitRVVFMABuiltin(CodeGenFunction *CGF, const CallExpr *E,
689:                   ReturnValueSlot ReturnValue, llvm::Type *ResultType,
690:                   Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
691:                   int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
692:   auto &Builder = CGF->Builder;
693:   auto &CGM = CGF->CGM;
694:   // LLVM intrinsic
695:   // Unmasked: (vector_in, vector_in/scalar_in, vector_in, round_mode,
696:   //            vl, policy)
697:   // Masked:   (vector_in, vector_in/scalar_in, vector_in, mask, frm,
698:   //            vl, policy)
699: 
700:   bool HasRoundModeOp = IsMasked ? Ops.size() == 6 : Ops.size() == 5;
```
- **EN**: This block defines callable entry points like `emitRVVFMABuiltin`; returns or forwards computed values for the surrounding target builtin lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `emitRVVFMABuiltin`；为周围的 目标内建函数降级 逻辑返回或转发计算结果。

### Lines 701-720
```cpp
701: 
702:   if (!HasRoundModeOp)
703:     Ops.insert(Ops.end() - 1,
704:                ConstantInt::get(Ops.back()->getType(), 7)); // frm
705: 
706:   if (IsMasked)
707:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 2);
708: 
709:   Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
710: 
711:   llvm::Function *F = CGM.getIntrinsic(
712:       ID, {ResultType, Ops[1]->getType(), Ops.back()->getType()});
713:   return Builder.CreateCall(F, Ops, "");
714: }
715: 
716: static LLVM_ATTRIBUTE_NOINLINE Value *
717: emitRVVWideningFMABuiltin(CodeGenFunction *CGF, const CallExpr *E,
718:                           ReturnValueSlot ReturnValue, llvm::Type *ResultType,
719:                           Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
720:                           int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
```
- **EN**: This block defines callable entry points like `emitRVVWideningFMABuiltin`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitRVVWideningFMABuiltin`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 721-740
```cpp
721:   auto &Builder = CGF->Builder;
722:   auto &CGM = CGF->CGM;
723:   // LLVM intrinsic
724:   // Unmasked: (vector_in, vector_in/scalar_in, vector_in, round_mode, vl,
725:   // policy) Masked:   (vector_in, vector_in/scalar_in, vector_in, mask, frm,
726:   // vl, policy)
727: 
728:   bool HasRoundModeOp = IsMasked ? Ops.size() == 6 : Ops.size() == 5;
729: 
730:   if (!HasRoundModeOp)
731:     Ops.insert(Ops.end() - 1,
732:                ConstantInt::get(Ops.back()->getType(), 7)); // frm
733: 
734:   if (IsMasked)
735:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.begin() + 4);
736: 
737:   Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
738: 
739:   llvm::Function *F =
740:       CGM.getIntrinsic(ID, {ResultType, Ops[1]->getType(), Ops[2]->getType(),
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 741-760
```cpp
741:                             Ops.back()->getType()});
742:   return Builder.CreateCall(F, Ops, "");
743: }
744: 
745: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVFloatingUnaryBuiltin(
746:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
747:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
748:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
749:   auto &Builder = CGF->Builder;
750:   auto &CGM = CGF->CGM;
751:   llvm::SmallVector<llvm::Type *, 3> IntrinsicTypes;
752:   // LLVM intrinsic
753:   // Unmasked: (passthru, op0, round_mode, vl)
754:   // Masked:   (passthru, op0, mask, frm, vl, policy)
755: 
756:   bool HasMaskedOff =
757:       !((IsMasked && (PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA)) ||
758:         (!IsMasked && PolicyAttrs & RVV_VTA));
759:   bool HasRoundModeOp =
760:       IsMasked ? (HasMaskedOff ? Ops.size() == 5 : Ops.size() == 4)
```
- **EN**: This block documents intent or context for the surrounding target builtin lowering code.
- **CN**: 该代码块说明周围 目标内建函数降级 代码的意图或上下文。

### Lines 761-780
```cpp
761:                : (HasMaskedOff ? Ops.size() == 4 : Ops.size() == 3);
762: 
763:   if (!HasRoundModeOp)
764:     Ops.insert(Ops.end() - 1,
765:                ConstantInt::get(Ops.back()->getType(), 7)); // frm
766: 
767:   if (IsMasked)
768:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 2);
769: 
770:   if (!HasMaskedOff)
771:     Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
772: 
773:   if (IsMasked)
774:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
775: 
776:   IntrinsicTypes = {ResultType, Ops.back()->getType()};
777:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
778:   return Builder.CreateCall(F, Ops, "");
779: }
780: 
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 781-800
```cpp
781: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVFloatingConvBuiltin(
782:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
783:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
784:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
785:   auto &Builder = CGF->Builder;
786:   auto &CGM = CGF->CGM;
787:   // LLVM intrinsic
788:   // Unmasked: (passthru, op0, frm, vl)
789:   // Masked:   (passthru, op0, mask, frm, vl, policy)
790:   bool HasMaskedOff =
791:       !((IsMasked && (PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA)) ||
792:         (!IsMasked && PolicyAttrs & RVV_VTA));
793:   bool HasRoundModeOp =
794:       IsMasked ? (HasMaskedOff ? Ops.size() == 5 : Ops.size() == 4)
795:                : (HasMaskedOff ? Ops.size() == 4 : Ops.size() == 3);
796: 
797:   if (!HasRoundModeOp)
798:     Ops.insert(Ops.end() - 1,
799:                ConstantInt::get(Ops.back()->getType(), 7)); // frm
800: 
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 801-820
```cpp
801:   if (IsMasked)
802:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 2);
803: 
804:   if (!HasMaskedOff)
805:     Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
806: 
807:   if (IsMasked)
808:     Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
809: 
810:   llvm::Function *F = CGM.getIntrinsic(
811:       ID, {ResultType, Ops[1]->getType(), Ops.back()->getType()});
812:   return Builder.CreateCall(F, Ops, "");
813: }
814: 
815: static LLVM_ATTRIBUTE_NOINLINE Value *emitRVVFloatingReductionBuiltin(
816:     CodeGenFunction *CGF, const CallExpr *E, ReturnValueSlot ReturnValue,
817:     llvm::Type *ResultType, Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
818:     int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
819:   auto &Builder = CGF->Builder;
820:   auto &CGM = CGF->CGM;
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 821-840
```cpp
821:   // LLVM intrinsic
822:   // Unmasked: (passthru, op0, op1, round_mode, vl)
823:   // Masked:   (passthru, vector_in, vector_in/scalar_in, mask, frm, vl, policy)
824: 
825:   bool HasMaskedOff =
826:       !((IsMasked && (PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA)) ||
827:         (!IsMasked && PolicyAttrs & RVV_VTA));
828:   bool HasRoundModeOp =
829:       IsMasked ? (HasMaskedOff ? Ops.size() == 6 : Ops.size() == 5)
830:                : (HasMaskedOff ? Ops.size() == 5 : Ops.size() == 4);
831: 
832:   if (!HasRoundModeOp)
833:     Ops.insert(Ops.end() - 1,
834:                ConstantInt::get(Ops.back()->getType(), 7)); // frm
835: 
836:   if (IsMasked)
837:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 2);
838: 
839:   if (!HasMaskedOff)
840:     Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 841-860
```cpp
841: 
842:   llvm::Function *F = CGM.getIntrinsic(
843:       ID, {ResultType, Ops[1]->getType(), Ops.back()->getType()});
844:   return Builder.CreateCall(F, Ops, "");
845: }
846: 
847: static LLVM_ATTRIBUTE_NOINLINE Value *
848: emitRVVReinterpretBuiltin(CodeGenFunction *CGF, const CallExpr *E,
849:                           ReturnValueSlot ReturnValue, llvm::Type *ResultType,
850:                           Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
851:                           int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
852:   auto &Builder = CGF->Builder;
853:   auto &CGM = CGF->CGM;
854: 
855:   if (ResultType->isIntOrIntVectorTy(1) ||
856:       Ops[0]->getType()->isIntOrIntVectorTy(1)) {
857:     assert(isa<ScalableVectorType>(ResultType) &&
858:            isa<ScalableVectorType>(Ops[0]->getType()));
859: 
860:     LLVMContext &Context = CGM.getLLVMContext();
```
- **EN**: This block defines callable entry points like `emitRVVReinterpretBuiltin`; uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitRVVReinterpretBuiltin`；通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 861-880
```cpp
861:     ScalableVectorType *Boolean64Ty =
862:         ScalableVectorType::get(llvm::Type::getInt1Ty(Context), 64);
863: 
864:     if (ResultType->isIntOrIntVectorTy(1)) {
865:       // Casting from m1 vector integer -> vector boolean
866:       // Ex: <vscale x 8 x i8>
867:       //     --(bitcast)--------> <vscale x 64 x i1>
868:       //     --(vector_extract)-> <vscale x  8 x i1>
869:       llvm::Value *BitCast = Builder.CreateBitCast(Ops[0], Boolean64Ty);
870:       return Builder.CreateExtractVector(ResultType, BitCast,
871:                                          ConstantInt::get(CGF->Int64Ty, 0));
872:     } else {
873:       // Casting from vector boolean -> m1 vector integer
874:       // Ex: <vscale x  1 x i1>
875:       //       --(vector_insert)-> <vscale x 64 x i1>
876:       //       --(bitcast)-------> <vscale x  8 x i8>
877:       llvm::Value *Boolean64Val = Builder.CreateInsertVector(
878:           Boolean64Ty, llvm::PoisonValue::get(Boolean64Ty), Ops[0],
879:           ConstantInt::get(CGF->Int64Ty, 0));
880:       return Builder.CreateBitCast(Boolean64Val, ResultType);
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 881-900
```cpp
881:     }
882:   }
883:   return Builder.CreateBitCast(Ops[0], ResultType);
884: }
885: 
886: static LLVM_ATTRIBUTE_NOINLINE Value *
887: emitRVVGetBuiltin(CodeGenFunction *CGF, const CallExpr *E,
888:                   ReturnValueSlot ReturnValue, llvm::Type *ResultType,
889:                   Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
890:                   int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
891:   auto &Builder = CGF->Builder;
892:   auto *VecTy = cast<ScalableVectorType>(ResultType);
893:   if (auto *OpVecTy = dyn_cast<ScalableVectorType>(Ops[0]->getType())) {
894:     unsigned MaxIndex =
895:         OpVecTy->getMinNumElements() / VecTy->getMinNumElements();
896:     assert(isPowerOf2_32(MaxIndex));
897:     // Mask to only valid indices.
898:     Ops[1] = Builder.CreateZExt(Ops[1], Builder.getInt64Ty());
899:     Ops[1] = Builder.CreateAnd(Ops[1], MaxIndex - 1);
900:     Ops[1] =
```
- **EN**: This block defines callable entry points like `emitRVVGetBuiltin`; uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitRVVGetBuiltin`；通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 901-920
```cpp
901:         Builder.CreateMul(Ops[1], ConstantInt::get(Ops[1]->getType(),
902:                                                    VecTy->getMinNumElements()));
903:     return Builder.CreateExtractVector(ResultType, Ops[0], Ops[1]);
904:   }
905: 
906:   return Builder.CreateIntrinsic(
907:       Intrinsic::riscv_tuple_extract, {ResultType, Ops[0]->getType()},
908:       {Ops[0], Builder.CreateTrunc(Ops[1], Builder.getInt32Ty())});
909: }
910: 
911: static LLVM_ATTRIBUTE_NOINLINE Value *
912: emitRVVSetBuiltin(CodeGenFunction *CGF, const CallExpr *E,
913:                   ReturnValueSlot ReturnValue, llvm::Type *ResultType,
914:                   Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
915:                   int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
916:   auto &Builder = CGF->Builder;
917:   if (auto *ResVecTy = dyn_cast<ScalableVectorType>(ResultType)) {
918:     auto *VecTy = cast<ScalableVectorType>(Ops[2]->getType());
919:     unsigned MaxIndex =
920:         ResVecTy->getMinNumElements() / VecTy->getMinNumElements();
```
- **EN**: This block defines callable entry points like `emitRVVSetBuiltin`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitRVVSetBuiltin`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 921-940
```cpp
921:     assert(isPowerOf2_32(MaxIndex));
922:     // Mask to only valid indices.
923:     Ops[1] = Builder.CreateZExt(Ops[1], Builder.getInt64Ty());
924:     Ops[1] = Builder.CreateAnd(Ops[1], MaxIndex - 1);
925:     Ops[1] =
926:         Builder.CreateMul(Ops[1], ConstantInt::get(Ops[1]->getType(),
927:                                                    VecTy->getMinNumElements()));
928:     return Builder.CreateInsertVector(ResultType, Ops[0], Ops[2], Ops[1]);
929:   }
930: 
931:   return Builder.CreateIntrinsic(
932:       Intrinsic::riscv_tuple_insert, {ResultType, Ops[2]->getType()},
933:       {Ops[0], Ops[2], Builder.CreateTrunc(Ops[1], Builder.getInt32Ty())});
934: }
935: 
936: static LLVM_ATTRIBUTE_NOINLINE Value *
937: emitRVVCreateBuiltin(CodeGenFunction *CGF, const CallExpr *E,
938:                      ReturnValueSlot ReturnValue, llvm::Type *ResultType,
939:                      Intrinsic::ID ID, SmallVectorImpl<Value *> &Ops,
940:                      int PolicyAttrs, bool IsMasked, unsigned SegInstSEW) {
```
- **EN**: This block defines callable entry points like `emitRVVCreateBuiltin`; returns or forwards computed values for the surrounding target builtin lowering logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitRVVCreateBuiltin`；为周围的 目标内建函数降级 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 941-960
```cpp
941:   auto &Builder = CGF->Builder;
942:   llvm::Value *ReturnVector = llvm::PoisonValue::get(ResultType);
943:   auto *VecTy = cast<ScalableVectorType>(Ops[0]->getType());
944:   for (unsigned I = 0, N = Ops.size(); I < N; ++I) {
945:     if (isa<ScalableVectorType>(ResultType)) {
946:       llvm::Value *Idx = ConstantInt::get(Builder.getInt64Ty(),
947:                                           VecTy->getMinNumElements() * I);
948:       ReturnVector =
949:           Builder.CreateInsertVector(ResultType, ReturnVector, Ops[I], Idx);
950:     } else {
951:       llvm::Value *Idx = ConstantInt::get(Builder.getInt32Ty(), I);
952:       ReturnVector = Builder.CreateIntrinsic(Intrinsic::riscv_tuple_insert,
953:                                              {ResultType, Ops[I]->getType()},
954:                                              {ReturnVector, Ops[I], Idx});
955:     }
956:   }
957:   return ReturnVector;
958: }
959: 
960: Value *CodeGenFunction::EmitRISCVCpuInit() {
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 961-980
```cpp
961:   llvm::FunctionType *FTy = llvm::FunctionType::get(VoidTy, {VoidPtrTy}, false);
962:   llvm::FunctionCallee Func =
963:       CGM.CreateRuntimeFunction(FTy, "__init_riscv_feature_bits");
964:   auto *CalleeGV = cast<llvm::GlobalValue>(Func.getCallee());
965:   CalleeGV->setDSOLocal(true);
966:   CalleeGV->setDLLStorageClass(llvm::GlobalValue::DefaultStorageClass);
967:   return Builder.CreateCall(Func, {llvm::ConstantPointerNull::get(VoidPtrTy)});
968: }
969: 
970: Value *CodeGenFunction::EmitRISCVCpuSupports(const CallExpr *E) {
971: 
972:   const Expr *FeatureExpr = E->getArg(0)->IgnoreParenCasts();
973:   StringRef FeatureStr = cast<StringLiteral>(FeatureExpr)->getString();
974:   if (!getContext().getTargetInfo().validateCpuSupports(FeatureStr))
975:     return Builder.getFalse();
976: 
977:   return EmitRISCVCpuSupports(ArrayRef<StringRef>(FeatureStr));
978: }
979: 
980: static Value *loadRISCVFeatureBits(unsigned Index, CGBuilderTy &Builder,
```
- **EN**: This block defines callable entry points like `EmitRISCVCpuSupports`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitRISCVCpuSupports`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 981-1000
```cpp
 981:                                    CodeGenModule &CGM) {
 982:   llvm::Type *Int32Ty = Builder.getInt32Ty();
 983:   llvm::Type *Int64Ty = Builder.getInt64Ty();
 984:   llvm::ArrayType *ArrayOfInt64Ty =
 985:       llvm::ArrayType::get(Int64Ty, llvm::RISCVISAInfo::FeatureBitSize);
 986:   llvm::Type *StructTy = llvm::StructType::get(Int32Ty, ArrayOfInt64Ty);
 987:   llvm::Constant *RISCVFeaturesBits =
 988:       CGM.CreateRuntimeVariable(StructTy, "__riscv_feature_bits");
 989:   cast<llvm::GlobalValue>(RISCVFeaturesBits)->setDSOLocal(true);
 990:   Value *IndexVal = llvm::ConstantInt::get(Int32Ty, Index);
 991:   llvm::Value *GEPIndices[] = {Builder.getInt32(0), Builder.getInt32(1),
 992:                                IndexVal};
 993:   Value *Ptr =
 994:       Builder.CreateInBoundsGEP(StructTy, RISCVFeaturesBits, GEPIndices);
 995:   Value *FeaturesBit =
 996:       Builder.CreateAlignedLoad(Int64Ty, Ptr, CharUnits::fromQuantity(8));
 997:   return FeaturesBit;
 998: }
 999: 
1000: Value *CodeGenFunction::EmitRISCVCpuSupports(ArrayRef<StringRef> FeaturesStrs) {
```
- **EN**: This block defines callable entry points like `get`; returns or forwards computed values for the surrounding target builtin lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `get`；为周围的 目标内建函数降级 逻辑返回或转发计算结果。

### Lines 1001-1020
```cpp
1001:   const unsigned RISCVFeatureLength = llvm::RISCVISAInfo::FeatureBitSize;
1002:   uint64_t RequireBitMasks[RISCVFeatureLength] = {0};
1003: 
1004:   for (auto Feat : FeaturesStrs) {
1005:     auto [GroupID, BitPos] = RISCVISAInfo::getRISCVFeaturesBitsInfo(Feat);
1006: 
1007:     // If there isn't BitPos for this feature, skip this version.
1008:     // It also report the warning to user during compilation.
1009:     if (BitPos == -1)
1010:       return Builder.getFalse();
1011: 
1012:     RequireBitMasks[GroupID] |= (1ULL << BitPos);
1013:   }
1014: 
1015:   Value *Result = nullptr;
1016:   for (unsigned Idx = 0; Idx < RISCVFeatureLength; Idx++) {
1017:     if (RequireBitMasks[Idx] == 0)
1018:       continue;
1019: 
1020:     Value *Mask = Builder.getInt64(RequireBitMasks[Idx]);
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 1021-1040
```cpp
1021:     Value *Bitset =
1022:         Builder.CreateAnd(loadRISCVFeatureBits(Idx, Builder, CGM), Mask);
1023:     Value *CmpV = Builder.CreateICmpEQ(Bitset, Mask);
1024:     Result = (!Result) ? CmpV : Builder.CreateAnd(Result, CmpV);
1025:   }
1026: 
1027:   assert(Result && "Should have value here.");
1028: 
1029:   return Result;
1030: }
1031: 
1032: Value *CodeGenFunction::EmitRISCVCpuIs(const CallExpr *E) {
1033:   const Expr *CPUExpr = E->getArg(0)->IgnoreParenCasts();
1034:   StringRef CPUStr = cast<clang::StringLiteral>(CPUExpr)->getString();
1035:   return EmitRISCVCpuIs(CPUStr);
1036: }
1037: 
1038: Value *CodeGenFunction::EmitRISCVCpuIs(StringRef CPUStr) {
1039:   llvm::Type *Int32Ty = Builder.getInt32Ty();
1040:   llvm::Type *Int64Ty = Builder.getInt64Ty();
```
- **EN**: This block defines callable entry points like `EmitRISCVCpuIs`; returns or forwards computed values for the surrounding target builtin lowering logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitRISCVCpuIs`；为周围的 目标内建函数降级 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 1041-1060
```cpp
1041:   llvm::StructType *StructTy = llvm::StructType::get(Int32Ty, Int64Ty, Int64Ty);
1042:   llvm::Constant *RISCVCPUModel =
1043:       CGM.CreateRuntimeVariable(StructTy, "__riscv_cpu_model");
1044:   cast<llvm::GlobalValue>(RISCVCPUModel)->setDSOLocal(true);
1045: 
1046:   auto loadRISCVCPUID = [&](unsigned Index) {
1047:     Value *Ptr = Builder.CreateStructGEP(StructTy, RISCVCPUModel, Index);
1048:     Value *CPUID = Builder.CreateAlignedLoad(StructTy->getTypeAtIndex(Index),
1049:                                              Ptr, llvm::MaybeAlign());
1050:     return CPUID;
1051:   };
1052: 
1053:   const llvm::RISCV::CPUModel Model = llvm::RISCV::getCPUModel(CPUStr);
1054: 
1055:   // Compare mvendorid.
1056:   Value *VendorID = loadRISCVCPUID(0);
1057:   Value *Result =
1058:       Builder.CreateICmpEQ(VendorID, Builder.getInt32(Model.MVendorID));
1059: 
1060:   // Compare marchid.
```
- **EN**: This block defines callable entry points like `MaybeAlign`; returns or forwards computed values for the surrounding target builtin lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `MaybeAlign`；为周围的 目标内建函数降级 逻辑返回或转发计算结果。

### Lines 1061-1080
```cpp
1061:   Value *ArchID = loadRISCVCPUID(1);
1062:   Result = Builder.CreateAnd(
1063:       Result, Builder.CreateICmpEQ(ArchID, Builder.getInt64(Model.MArchID)));
1064: 
1065:   // Compare mimpid.
1066:   Value *ImpID = loadRISCVCPUID(2);
1067:   Result = Builder.CreateAnd(
1068:       Result, Builder.CreateICmpEQ(ImpID, Builder.getInt64(Model.MImpID)));
1069: 
1070:   return Result;
1071: }
1072: 
1073: Value *CodeGenFunction::EmitRISCVBuiltinExpr(unsigned BuiltinID,
1074:                                              const CallExpr *E,
1075:                                              ReturnValueSlot ReturnValue) {
1076: 
1077:   if (BuiltinID == Builtin::BI__builtin_cpu_supports)
1078:     return EmitRISCVCpuSupports(E);
1079:   if (BuiltinID == Builtin::BI__builtin_cpu_init)
1080:     return EmitRISCVCpuInit();
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 1081-1100
```cpp
1081:   if (BuiltinID == Builtin::BI__builtin_cpu_is)
1082:     return EmitRISCVCpuIs(E);
1083: 
1084:   SmallVector<Value *, 4> Ops;
1085:   llvm::Type *ResultType = ConvertType(E->getType());
1086: 
1087:   // Find out if any arguments are required to be integer constant expressions.
1088:   unsigned ICEArguments = 0;
1089:   ASTContext::GetBuiltinTypeError Error;
1090:   getContext().GetBuiltinType(BuiltinID, Error, &ICEArguments);
1091:   if (Error == ASTContext::GE_Missing_type) {
1092:     // Vector intrinsics don't have a type string.
1093:     assert(BuiltinID >= clang::RISCV::FirstRVVBuiltin &&
1094:            BuiltinID <= clang::RISCV::LastRVVBuiltin);
1095:     ICEArguments = 0;
1096:     if (BuiltinID == RISCVVector::BI__builtin_rvv_vget_v ||
1097:         BuiltinID == RISCVVector::BI__builtin_rvv_vset_v)
1098:       ICEArguments = 1 << 1;
1099:   } else {
1100:     assert(Error == ASTContext::GE_None && "Unexpected error");
```
- **EN**: This block defines callable entry points like `getContext`; uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getContext`；通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1101-1120
```cpp
1101:   }
1102: 
1103:   if (BuiltinID == RISCV::BI__builtin_riscv_ntl_load)
1104:     ICEArguments |= (1 << 1);
1105:   if (BuiltinID == RISCV::BI__builtin_riscv_ntl_store)
1106:     ICEArguments |= (1 << 2);
1107: 
1108:   for (unsigned i = 0, e = E->getNumArgs(); i != e; i++) {
1109:     // Handle aggregate argument, namely RVV tuple types in segment load/store
1110:     if (hasAggregateEvaluationKind(E->getArg(i)->getType())) {
1111:       LValue L = EmitAggExprToLValue(E->getArg(i));
1112:       llvm::Value *AggValue = Builder.CreateLoad(L.getAddress());
1113:       Ops.push_back(AggValue);
1114:       continue;
1115:     }
1116:     Ops.push_back(EmitScalarOrConstFoldImmArg(ICEArguments, i, E));
1117:   }
1118: 
1119:   Intrinsic::ID ID = Intrinsic::not_intrinsic;
1120:   int PolicyAttrs = 0;
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 1121-1140
```cpp
1121:   bool IsMasked = false;
1122:   // This is used by segment load/store to determine it's llvm type.
1123:   unsigned SegInstSEW = 8;
1124:   // This is used by XSfmm.
1125:   unsigned TWiden = 0;
1126: 
1127:   // Required for overloaded intrinsics.
1128:   llvm::SmallVector<llvm::Type *, 2> IntrinsicTypes;
1129:   switch (BuiltinID) {
1130:   default: llvm_unreachable("unexpected builtin ID");
1131:   case RISCV::BI__builtin_riscv_orc_b_32:
1132:   case RISCV::BI__builtin_riscv_orc_b_64:
1133:   case RISCV::BI__builtin_riscv_clmul_32:
1134:   case RISCV::BI__builtin_riscv_clmul_64:
1135:   case RISCV::BI__builtin_riscv_clmulh_32:
1136:   case RISCV::BI__builtin_riscv_clmulh_64:
1137:   case RISCV::BI__builtin_riscv_clmulr_32:
1138:   case RISCV::BI__builtin_riscv_clmulr_64:
1139:   case RISCV::BI__builtin_riscv_xperm4_32:
1140:   case RISCV::BI__builtin_riscv_xperm4_64:
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1141-1160
```cpp
1141:   case RISCV::BI__builtin_riscv_xperm8_32:
1142:   case RISCV::BI__builtin_riscv_xperm8_64:
1143:   case RISCV::BI__builtin_riscv_brev8_32:
1144:   case RISCV::BI__builtin_riscv_brev8_64:
1145:   case RISCV::BI__builtin_riscv_zip_32:
1146:   case RISCV::BI__builtin_riscv_unzip_32: {
1147:     switch (BuiltinID) {
1148:     default: llvm_unreachable("unexpected builtin ID");
1149:     // Zbb
1150:     case RISCV::BI__builtin_riscv_orc_b_32:
1151:     case RISCV::BI__builtin_riscv_orc_b_64:
1152:       ID = Intrinsic::riscv_orc_b;
1153:       break;
1154: 
1155:     // Zbc
1156:     case RISCV::BI__builtin_riscv_clmul_32:
1157:     case RISCV::BI__builtin_riscv_clmul_64:
1158:       ID = Intrinsic::clmul;
1159:       break;
1160:     case RISCV::BI__builtin_riscv_clmulh_32:
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1161-1180
```cpp
1161:     case RISCV::BI__builtin_riscv_clmulh_64:
1162:       ID = Intrinsic::riscv_clmulh;
1163:       break;
1164:     case RISCV::BI__builtin_riscv_clmulr_32:
1165:     case RISCV::BI__builtin_riscv_clmulr_64:
1166:       ID = Intrinsic::riscv_clmulr;
1167:       break;
1168: 
1169:     // Zbkx
1170:     case RISCV::BI__builtin_riscv_xperm8_32:
1171:     case RISCV::BI__builtin_riscv_xperm8_64:
1172:       ID = Intrinsic::riscv_xperm8;
1173:       break;
1174:     case RISCV::BI__builtin_riscv_xperm4_32:
1175:     case RISCV::BI__builtin_riscv_xperm4_64:
1176:       ID = Intrinsic::riscv_xperm4;
1177:       break;
1178: 
1179:     // Zbkb
1180:     case RISCV::BI__builtin_riscv_brev8_32:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1181-1200
```cpp
1181:     case RISCV::BI__builtin_riscv_brev8_64:
1182:       ID = Intrinsic::riscv_brev8;
1183:       break;
1184:     case RISCV::BI__builtin_riscv_zip_32:
1185:       ID = Intrinsic::riscv_zip;
1186:       break;
1187:     case RISCV::BI__builtin_riscv_unzip_32:
1188:       ID = Intrinsic::riscv_unzip;
1189:       break;
1190:     }
1191: 
1192:     IntrinsicTypes = {ResultType};
1193:     break;
1194:   }
1195: 
1196:   // Zk builtins
1197: 
1198:   // Zknh
1199:   case RISCV::BI__builtin_riscv_sha256sig0:
1200:     ID = Intrinsic::riscv_sha256sig0;
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1201-1220
```cpp
1201:     break;
1202:   case RISCV::BI__builtin_riscv_sha256sig1:
1203:     ID = Intrinsic::riscv_sha256sig1;
1204:     break;
1205:   case RISCV::BI__builtin_riscv_sha256sum0:
1206:     ID = Intrinsic::riscv_sha256sum0;
1207:     break;
1208:   case RISCV::BI__builtin_riscv_sha256sum1:
1209:     ID = Intrinsic::riscv_sha256sum1;
1210:     break;
1211: 
1212:   // Zksed
1213:   case RISCV::BI__builtin_riscv_sm4ks:
1214:     ID = Intrinsic::riscv_sm4ks;
1215:     break;
1216:   case RISCV::BI__builtin_riscv_sm4ed:
1217:     ID = Intrinsic::riscv_sm4ed;
1218:     break;
1219: 
1220:   // Zksh
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1221-1240
```cpp
1221:   case RISCV::BI__builtin_riscv_sm3p0:
1222:     ID = Intrinsic::riscv_sm3p0;
1223:     break;
1224:   case RISCV::BI__builtin_riscv_sm3p1:
1225:     ID = Intrinsic::riscv_sm3p1;
1226:     break;
1227: 
1228:   case RISCV::BI__builtin_riscv_clz_32:
1229:   case RISCV::BI__builtin_riscv_clz_64: {
1230:     Function *F = CGM.getIntrinsic(Intrinsic::ctlz, Ops[0]->getType());
1231:     Value *Result = Builder.CreateCall(F, {Ops[0], Builder.getInt1(false)});
1232:     if (Result->getType() != ResultType)
1233:       Result =
1234:           Builder.CreateIntCast(Result, ResultType, /*isSigned*/ false, "cast");
1235:     return Result;
1236:   }
1237:   case RISCV::BI__builtin_riscv_ctz_32:
1238:   case RISCV::BI__builtin_riscv_ctz_64: {
1239:     Function *F = CGM.getIntrinsic(Intrinsic::cttz, Ops[0]->getType());
1240:     Value *Result = Builder.CreateCall(F, {Ops[0], Builder.getInt1(false)});
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 1241-1260
```cpp
1241:     if (Result->getType() != ResultType)
1242:       Result =
1243:           Builder.CreateIntCast(Result, ResultType, /*isSigned*/ false, "cast");
1244:     return Result;
1245:   }
1246: 
1247:   // Zihintntl
1248:   case RISCV::BI__builtin_riscv_ntl_load: {
1249:     llvm::Type *ResTy = ConvertType(E->getType());
1250:     unsigned DomainVal = 5; // Default __RISCV_NTLH_ALL
1251:     if (Ops.size() == 2)
1252:       DomainVal = cast<ConstantInt>(Ops[1])->getZExtValue();
1253: 
1254:     llvm::MDNode *RISCVDomainNode = llvm::MDNode::get(
1255:         getLLVMContext(),
1256:         llvm::ConstantAsMetadata::get(Builder.getInt32(DomainVal)));
1257:     llvm::MDNode *NontemporalNode = llvm::MDNode::get(
1258:         getLLVMContext(), llvm::ConstantAsMetadata::get(Builder.getInt32(1)));
1259: 
1260:     int Width;
```
- **EN**: This block defines callable entry points like `getLLVMContext`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getLLVMContext`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 1261-1280
```cpp
1261:     if(ResTy->isScalableTy()) {
1262:       const ScalableVectorType *SVTy = cast<ScalableVectorType>(ResTy);
1263:       llvm::Type *ScalarTy = ResTy->getScalarType();
1264:       Width = ScalarTy->getPrimitiveSizeInBits() *
1265:               SVTy->getElementCount().getKnownMinValue();
1266:     } else
1267:       Width = ResTy->getPrimitiveSizeInBits();
1268:     LoadInst *Load = Builder.CreateLoad(
1269:         Address(Ops[0], ResTy, CharUnits::fromQuantity(Width / 8)));
1270: 
1271:     Load->setMetadata(llvm::LLVMContext::MD_nontemporal, NontemporalNode);
1272:     Load->setMetadata(CGM.getModule().getMDKindID("riscv-nontemporal-domain"),
1273:                       RISCVDomainNode);
1274: 
1275:     return Load;
1276:   }
1277:   case RISCV::BI__builtin_riscv_ntl_store: {
1278:     unsigned DomainVal = 5; // Default __RISCV_NTLH_ALL
1279:     if (Ops.size() == 3)
1280:       DomainVal = cast<ConstantInt>(Ops[2])->getZExtValue();
```
- **EN**: This block defines callable entry points like `Address`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `Address`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 1281-1300
```cpp
1281: 
1282:     llvm::MDNode *RISCVDomainNode = llvm::MDNode::get(
1283:         getLLVMContext(),
1284:         llvm::ConstantAsMetadata::get(Builder.getInt32(DomainVal)));
1285:     llvm::MDNode *NontemporalNode = llvm::MDNode::get(
1286:         getLLVMContext(), llvm::ConstantAsMetadata::get(Builder.getInt32(1)));
1287: 
1288:     StoreInst *Store = Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
1289:     Store->setMetadata(llvm::LLVMContext::MD_nontemporal, NontemporalNode);
1290:     Store->setMetadata(CGM.getModule().getMDKindID("riscv-nontemporal-domain"),
1291:                        RISCVDomainNode);
1292: 
1293:     return Store;
1294:   }
1295:   // Zihintpause
1296:   case RISCV::BI__builtin_riscv_pause: {
1297:     llvm::Function *Fn = CGM.getIntrinsic(llvm::Intrinsic::riscv_pause);
1298:     return Builder.CreateCall(Fn, {});
1299:   }
1300: 
```
- **EN**: This block defines callable entry points like `getLLVMContext`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getLLVMContext`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1301-1320
```cpp
1301:   // XCValu
1302:   case RISCV::BI__builtin_riscv_cv_alu_addN:
1303:     ID = Intrinsic::riscv_cv_alu_addN;
1304:     break;
1305:   case RISCV::BI__builtin_riscv_cv_alu_addRN:
1306:     ID = Intrinsic::riscv_cv_alu_addRN;
1307:     break;
1308:   case RISCV::BI__builtin_riscv_cv_alu_adduN:
1309:     ID = Intrinsic::riscv_cv_alu_adduN;
1310:     break;
1311:   case RISCV::BI__builtin_riscv_cv_alu_adduRN:
1312:     ID = Intrinsic::riscv_cv_alu_adduRN;
1313:     break;
1314:   case RISCV::BI__builtin_riscv_cv_alu_clip:
1315:     ID = Intrinsic::riscv_cv_alu_clip;
1316:     break;
1317:   case RISCV::BI__builtin_riscv_cv_alu_clipu:
1318:     ID = Intrinsic::riscv_cv_alu_clipu;
1319:     break;
1320:   case RISCV::BI__builtin_riscv_cv_alu_extbs:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1321-1340
```cpp
1321:     return Builder.CreateSExt(Builder.CreateTrunc(Ops[0], Int8Ty), Int32Ty,
1322:                               "extbs");
1323:   case RISCV::BI__builtin_riscv_cv_alu_extbz:
1324:     return Builder.CreateZExt(Builder.CreateTrunc(Ops[0], Int8Ty), Int32Ty,
1325:                               "extbz");
1326:   case RISCV::BI__builtin_riscv_cv_alu_exths:
1327:     return Builder.CreateSExt(Builder.CreateTrunc(Ops[0], Int16Ty), Int32Ty,
1328:                               "exths");
1329:   case RISCV::BI__builtin_riscv_cv_alu_exthz:
1330:     return Builder.CreateZExt(Builder.CreateTrunc(Ops[0], Int16Ty), Int32Ty,
1331:                               "exthz");
1332:   case RISCV::BI__builtin_riscv_cv_alu_sle:
1333:     return Builder.CreateZExt(Builder.CreateICmpSLE(Ops[0], Ops[1]), Int32Ty,
1334:                               "sle");
1335:   case RISCV::BI__builtin_riscv_cv_alu_sleu:
1336:     return Builder.CreateZExt(Builder.CreateICmpULE(Ops[0], Ops[1]), Int32Ty,
1337:                               "sleu");
1338:   case RISCV::BI__builtin_riscv_cv_alu_subN:
1339:     ID = Intrinsic::riscv_cv_alu_subN;
1340:     break;
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1341-1360
```cpp
1341:   case RISCV::BI__builtin_riscv_cv_alu_subRN:
1342:     ID = Intrinsic::riscv_cv_alu_subRN;
1343:     break;
1344:   case RISCV::BI__builtin_riscv_cv_alu_subuN:
1345:     ID = Intrinsic::riscv_cv_alu_subuN;
1346:     break;
1347:   case RISCV::BI__builtin_riscv_cv_alu_subuRN:
1348:     ID = Intrinsic::riscv_cv_alu_subuRN;
1349:     break;
1350: 
1351:   // XAndesPerf
1352:   case RISCV::BI__builtin_riscv_nds_ffb_32:
1353:   case RISCV::BI__builtin_riscv_nds_ffb_64:
1354:     IntrinsicTypes = {ResultType};
1355:     ID = Intrinsic::riscv_nds_ffb;
1356:     break;
1357:   case RISCV::BI__builtin_riscv_nds_ffzmism_32:
1358:   case RISCV::BI__builtin_riscv_nds_ffzmism_64:
1359:     IntrinsicTypes = {ResultType};
1360:     ID = Intrinsic::riscv_nds_ffzmism;
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1361-1380
```cpp
1361:     break;
1362:   case RISCV::BI__builtin_riscv_nds_ffmism_32:
1363:   case RISCV::BI__builtin_riscv_nds_ffmism_64:
1364:     IntrinsicTypes = {ResultType};
1365:     ID = Intrinsic::riscv_nds_ffmism;
1366:     break;
1367:   case RISCV::BI__builtin_riscv_nds_flmism_32:
1368:   case RISCV::BI__builtin_riscv_nds_flmism_64:
1369:     IntrinsicTypes = {ResultType};
1370:     ID = Intrinsic::riscv_nds_flmism;
1371:     break;
1372: 
1373:   // XAndesBFHCvt
1374:   case RISCV::BI__builtin_riscv_nds_fcvt_s_bf16:
1375:     return Builder.CreateFPExt(Ops[0], FloatTy);
1376:   case RISCV::BI__builtin_riscv_nds_fcvt_bf16_s:
1377:     return Builder.CreateFPTrunc(Ops[0], BFloatTy);
1378: 
1379:     // Vector builtins are handled from here.
1380: #include "clang/Basic/riscv_vector_builtin_cg.inc"
```
- **EN**: This block imports Clang headers `clang/Basic/riscv_vector_builtin_cg.inc`; uses control flow (case) to specialize target builtin lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/Basic/riscv_vector_builtin_cg.inc`；通过控制流（case）细化 目标内建函数降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 1381-1393
```cpp
1381: 
1382:     // SiFive Vector builtins are handled from here.
1383: #include "clang/Basic/riscv_sifive_vector_builtin_cg.inc"
1384: 
1385:     // Andes Vector builtins are handled from here.
1386: #include "clang/Basic/riscv_andes_vector_builtin_cg.inc"
1387:   }
1388: 
1389:   assert(ID != Intrinsic::not_intrinsic);
1390: 
1391:   llvm::Function *F = CGM.getIntrinsic(ID, IntrinsicTypes);
1392:   return Builder.CreateCall(F, Ops, "");
1393: }
```
- **EN**: This block imports Clang headers `clang/Basic/riscv_sifive_vector_builtin_cg.inc`, `clang/Basic/riscv_andes_vector_builtin_cg.inc`; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/Basic/riscv_sifive_vector_builtin_cg.inc`, `clang/Basic/riscv_andes_vector_builtin_cg.inc`；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

## Key Concepts / 关键概念

- **Ops**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Builder**: Acts as a construction helper that incrementally assembles target builtin lowering state. / 充当构建辅助器，逐步组装 目标内建函数降级 状态。
- **getType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **IsMasked**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ResultType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **PolicyAttrs**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGM**: Central symbol in this file's implementation of target builtin lowering. / 是该文件实现 目标内建函数降级 时的核心符号。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CodeGenFunction.h`
- **Clang libraries / Clang 库**: `clang/Basic/TargetBuiltins.h`, `clang/Basic/riscv_vector_builtin_cg.inc`, `clang/Basic/riscv_sifive_vector_builtin_cg.inc`, `clang/Basic/riscv_andes_vector_builtin_cg.inc`
- **LLVM libraries / LLVM 库**: `llvm/IR/IntrinsicsRISCV.h`, `llvm/TargetParser/RISCVISAInfo.h`, `llvm/TargetParser/RISCVTargetParser.h`
