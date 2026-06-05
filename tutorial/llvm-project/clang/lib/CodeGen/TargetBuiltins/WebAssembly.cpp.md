# WebAssembly.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/TargetBuiltins/WebAssembly.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements builtin handling and lowering logic for the WebAssembly backend.
- **Purpose (CN) / 目的（中文）**: 实现 WebAssembly 后端的内建函数处理与降级逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: //===-- WebAssembly.cpp - Emit LLVM Code for builtins ---------------------===//
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
13: #include "CGBuiltin.h"
14: #include "clang/Basic/TargetBuiltins.h"
15: #include "llvm/ADT/APInt.h"
16: #include "llvm/IR/Constants.h"
```
- **EN**: This block imports local CodeGen headers `CGBuiltin.h`; Clang headers `clang/Basic/TargetBuiltins.h`; LLVM headers `llvm/ADT/APInt.h`, `llvm/IR/Constants.h`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CGBuiltin.h`；Clang 头文件 `clang/Basic/TargetBuiltins.h`；LLVM 头文件 `llvm/ADT/APInt.h`, `llvm/IR/Constants.h`；包含影响本编译单元构建方式的预处理结构。

### Lines 17-32
```cpp
17: #include "llvm/IR/IntrinsicsWebAssembly.h"
18: #include "llvm/Support/ErrorHandling.h"
19: 
20: using namespace clang;
21: using namespace CodeGen;
22: using namespace llvm;
23: 
24: Value *CodeGenFunction::EmitWebAssemblyBuiltinExpr(unsigned BuiltinID,
25:                                                    const CallExpr *E) {
26:   switch (BuiltinID) {
27:   case WebAssembly::BI__builtin_wasm_memory_size: {
28:     llvm::Type *ResultType = ConvertType(E->getType());
29:     Value *I = EmitScalarExpr(E->getArg(0));
30:     Function *Callee =
31:         CGM.getIntrinsic(Intrinsic::wasm_memory_size, ResultType);
32:     return Builder.CreateCall(Callee, I);
```
- **EN**: This block imports LLVM headers `llvm/IR/IntrinsicsWebAssembly.h`, `llvm/Support/ErrorHandling.h`; opens or references namespaces `clang`, `CodeGen`, `llvm`; uses control flow (switch, case) to specialize target builtin lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 LLVM 头文件 `llvm/IR/IntrinsicsWebAssembly.h`, `llvm/Support/ErrorHandling.h`；打开或引用命名空间 `clang`, `CodeGen`, `llvm`；通过控制流（switch, case）细化 目标内建函数降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 33-48
```cpp
33:   }
34:   case WebAssembly::BI__builtin_wasm_memory_grow: {
35:     llvm::Type *ResultType = ConvertType(E->getType());
36:     Value *Args[] = {EmitScalarExpr(E->getArg(0)),
37:                      EmitScalarExpr(E->getArg(1))};
38:     Function *Callee =
39:         CGM.getIntrinsic(Intrinsic::wasm_memory_grow, ResultType);
40:     return Builder.CreateCall(Callee, Args);
41:   }
42:   case WebAssembly::BI__builtin_wasm_tls_size: {
43:     llvm::Type *ResultType = ConvertType(E->getType());
44:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_tls_size, ResultType);
45:     return Builder.CreateCall(Callee);
46:   }
47:   case WebAssembly::BI__builtin_wasm_tls_align: {
48:     llvm::Type *ResultType = ConvertType(E->getType());
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 49-64
```cpp
49:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_tls_align, ResultType);
50:     return Builder.CreateCall(Callee);
51:   }
52:   case WebAssembly::BI__builtin_wasm_tls_base: {
53:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_tls_base);
54:     return Builder.CreateCall(Callee);
55:   }
56:   case WebAssembly::BI__builtin_wasm_throw: {
57:     Value *Tag = EmitScalarExpr(E->getArg(0));
58:     Value *Obj = EmitScalarExpr(E->getArg(1));
59:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_throw);
60:     return EmitRuntimeCallOrInvoke(Callee, {Tag, Obj});
61:   }
62:   case WebAssembly::BI__builtin_wasm_rethrow: {
63:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_rethrow);
64:     return EmitRuntimeCallOrInvoke(Callee);
```
- **EN**: This block defines callable entry points like `EmitRuntimeCallOrInvoke`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitRuntimeCallOrInvoke`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 65-80
```cpp
65:   }
66:   case WebAssembly::BI__builtin_wasm_memory_atomic_wait32: {
67:     Value *Addr = EmitScalarExpr(E->getArg(0));
68:     Value *Expected = EmitScalarExpr(E->getArg(1));
69:     Value *Timeout = EmitScalarExpr(E->getArg(2));
70:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_memory_atomic_wait32);
71:     return Builder.CreateCall(Callee, {Addr, Expected, Timeout});
72:   }
73:   case WebAssembly::BI__builtin_wasm_memory_atomic_wait64: {
74:     Value *Addr = EmitScalarExpr(E->getArg(0));
75:     Value *Expected = EmitScalarExpr(E->getArg(1));
76:     Value *Timeout = EmitScalarExpr(E->getArg(2));
77:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_memory_atomic_wait64);
78:     return Builder.CreateCall(Callee, {Addr, Expected, Timeout});
79:   }
80:   case WebAssembly::BI__builtin_wasm_memory_atomic_notify: {
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 81-96
```cpp
81:     Value *Addr = EmitScalarExpr(E->getArg(0));
82:     Value *Count = EmitScalarExpr(E->getArg(1));
83:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_memory_atomic_notify);
84:     return Builder.CreateCall(Callee, {Addr, Count});
85:   }
86:   case WebAssembly::BI__builtin_wasm_trunc_s_i32_f32:
87:   case WebAssembly::BI__builtin_wasm_trunc_s_i32_f64:
88:   case WebAssembly::BI__builtin_wasm_trunc_s_i64_f32:
89:   case WebAssembly::BI__builtin_wasm_trunc_s_i64_f64: {
90:     Value *Src = EmitScalarExpr(E->getArg(0));
91:     llvm::Type *ResT = ConvertType(E->getType());
92:     Function *Callee =
93:         CGM.getIntrinsic(Intrinsic::wasm_trunc_signed, {ResT, Src->getType()});
94:     return Builder.CreateCall(Callee, {Src});
95:   }
96:   case WebAssembly::BI__builtin_wasm_trunc_u_i32_f32:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 97-112
```cpp
 97:   case WebAssembly::BI__builtin_wasm_trunc_u_i32_f64:
 98:   case WebAssembly::BI__builtin_wasm_trunc_u_i64_f32:
 99:   case WebAssembly::BI__builtin_wasm_trunc_u_i64_f64: {
100:     Value *Src = EmitScalarExpr(E->getArg(0));
101:     llvm::Type *ResT = ConvertType(E->getType());
102:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_trunc_unsigned,
103:                                         {ResT, Src->getType()});
104:     return Builder.CreateCall(Callee, {Src});
105:   }
106:   case WebAssembly::BI__builtin_wasm_trunc_saturate_s_i32_f32:
107:   case WebAssembly::BI__builtin_wasm_trunc_saturate_s_i32_f64:
108:   case WebAssembly::BI__builtin_wasm_trunc_saturate_s_i64_f32:
109:   case WebAssembly::BI__builtin_wasm_trunc_saturate_s_i64_f64:
110:   case WebAssembly::BI__builtin_wasm_trunc_saturate_s_i16x8_f16x8:
111:   case WebAssembly::BI__builtin_wasm_trunc_saturate_s_i32x4_f32x4: {
112:     Value *Src = EmitScalarExpr(E->getArg(0));
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 113-128
```cpp
113:     llvm::Type *ResT = ConvertType(E->getType());
114:     Function *Callee =
115:         CGM.getIntrinsic(Intrinsic::fptosi_sat, {ResT, Src->getType()});
116:     return Builder.CreateCall(Callee, {Src});
117:   }
118:   case WebAssembly::BI__builtin_wasm_trunc_saturate_u_i32_f32:
119:   case WebAssembly::BI__builtin_wasm_trunc_saturate_u_i32_f64:
120:   case WebAssembly::BI__builtin_wasm_trunc_saturate_u_i64_f32:
121:   case WebAssembly::BI__builtin_wasm_trunc_saturate_u_i64_f64:
122:   case WebAssembly::BI__builtin_wasm_trunc_saturate_u_i16x8_f16x8:
123:   case WebAssembly::BI__builtin_wasm_trunc_saturate_u_i32x4_f32x4: {
124:     Value *Src = EmitScalarExpr(E->getArg(0));
125:     llvm::Type *ResT = ConvertType(E->getType());
126:     Function *Callee =
127:         CGM.getIntrinsic(Intrinsic::fptoui_sat, {ResT, Src->getType()});
128:     return Builder.CreateCall(Callee, {Src});
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 129-144
```cpp
129:   }
130:   case WebAssembly::BI__builtin_wasm_min_f32:
131:   case WebAssembly::BI__builtin_wasm_min_f64:
132:   case WebAssembly::BI__builtin_wasm_min_f16x8:
133:   case WebAssembly::BI__builtin_wasm_min_f32x4:
134:   case WebAssembly::BI__builtin_wasm_min_f64x2: {
135:     Value *LHS = EmitScalarExpr(E->getArg(0));
136:     Value *RHS = EmitScalarExpr(E->getArg(1));
137:     Function *Callee =
138:         CGM.getIntrinsic(Intrinsic::minimum, ConvertType(E->getType()));
139:     return Builder.CreateCall(Callee, {LHS, RHS});
140:   }
141:   case WebAssembly::BI__builtin_wasm_max_f32:
142:   case WebAssembly::BI__builtin_wasm_max_f64:
143:   case WebAssembly::BI__builtin_wasm_max_f16x8:
144:   case WebAssembly::BI__builtin_wasm_max_f32x4:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 145-160
```cpp
145:   case WebAssembly::BI__builtin_wasm_max_f64x2: {
146:     Value *LHS = EmitScalarExpr(E->getArg(0));
147:     Value *RHS = EmitScalarExpr(E->getArg(1));
148:     Function *Callee =
149:         CGM.getIntrinsic(Intrinsic::maximum, ConvertType(E->getType()));
150:     return Builder.CreateCall(Callee, {LHS, RHS});
151:   }
152:   case WebAssembly::BI__builtin_wasm_pmin_f16x8:
153:   case WebAssembly::BI__builtin_wasm_pmin_f32x4:
154:   case WebAssembly::BI__builtin_wasm_pmin_f64x2: {
155:     Value *LHS = EmitScalarExpr(E->getArg(0));
156:     Value *RHS = EmitScalarExpr(E->getArg(1));
157:     Function *Callee =
158:         CGM.getIntrinsic(Intrinsic::wasm_pmin, ConvertType(E->getType()));
159:     return Builder.CreateCall(Callee, {LHS, RHS});
160:   }
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 161-176
```cpp
161:   case WebAssembly::BI__builtin_wasm_pmax_f16x8:
162:   case WebAssembly::BI__builtin_wasm_pmax_f32x4:
163:   case WebAssembly::BI__builtin_wasm_pmax_f64x2: {
164:     Value *LHS = EmitScalarExpr(E->getArg(0));
165:     Value *RHS = EmitScalarExpr(E->getArg(1));
166:     Function *Callee =
167:         CGM.getIntrinsic(Intrinsic::wasm_pmax, ConvertType(E->getType()));
168:     return Builder.CreateCall(Callee, {LHS, RHS});
169:   }
170:   case WebAssembly::BI__builtin_wasm_ceil_f16x8:
171:   case WebAssembly::BI__builtin_wasm_floor_f16x8:
172:   case WebAssembly::BI__builtin_wasm_trunc_f16x8:
173:   case WebAssembly::BI__builtin_wasm_nearest_f16x8:
174:   case WebAssembly::BI__builtin_wasm_ceil_f32x4:
175:   case WebAssembly::BI__builtin_wasm_floor_f32x4:
176:   case WebAssembly::BI__builtin_wasm_trunc_f32x4:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 177-192
```cpp
177:   case WebAssembly::BI__builtin_wasm_nearest_f32x4:
178:   case WebAssembly::BI__builtin_wasm_ceil_f64x2:
179:   case WebAssembly::BI__builtin_wasm_floor_f64x2:
180:   case WebAssembly::BI__builtin_wasm_trunc_f64x2:
181:   case WebAssembly::BI__builtin_wasm_nearest_f64x2: {
182:     unsigned IntNo;
183:     switch (BuiltinID) {
184:     case WebAssembly::BI__builtin_wasm_ceil_f16x8:
185:     case WebAssembly::BI__builtin_wasm_ceil_f32x4:
186:     case WebAssembly::BI__builtin_wasm_ceil_f64x2:
187:       IntNo = Intrinsic::ceil;
188:       break;
189:     case WebAssembly::BI__builtin_wasm_floor_f16x8:
190:     case WebAssembly::BI__builtin_wasm_floor_f32x4:
191:     case WebAssembly::BI__builtin_wasm_floor_f64x2:
192:       IntNo = Intrinsic::floor;
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 193-208
```cpp
193:       break;
194:     case WebAssembly::BI__builtin_wasm_trunc_f16x8:
195:     case WebAssembly::BI__builtin_wasm_trunc_f32x4:
196:     case WebAssembly::BI__builtin_wasm_trunc_f64x2:
197:       IntNo = Intrinsic::trunc;
198:       break;
199:     case WebAssembly::BI__builtin_wasm_nearest_f16x8:
200:     case WebAssembly::BI__builtin_wasm_nearest_f32x4:
201:     case WebAssembly::BI__builtin_wasm_nearest_f64x2:
202:       IntNo = Intrinsic::nearbyint;
203:       break;
204:     default:
205:       llvm_unreachable("unexpected builtin ID");
206:     }
207:     Value *Value = EmitScalarExpr(E->getArg(0));
208:     Function *Callee = CGM.getIntrinsic(IntNo, ConvertType(E->getType()));
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 209-224
```cpp
209:     return Builder.CreateCall(Callee, Value);
210:   }
211:   case WebAssembly::BI__builtin_wasm_ref_null_extern: {
212:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_ref_null_extern);
213:     return Builder.CreateCall(Callee);
214:   }
215:   case WebAssembly::BI__builtin_wasm_ref_is_null_extern: {
216:     Value *Src = EmitScalarExpr(E->getArg(0));
217:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_ref_is_null_extern);
218:     return Builder.CreateCall(Callee, {Src});
219:   }
220:   case WebAssembly::BI__builtin_wasm_ref_null_func: {
221:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_ref_null_func);
222:     return Builder.CreateCall(Callee);
223:   }
224:   case WebAssembly::BI__builtin_wasm_test_function_pointer_signature: {
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 225-240
```cpp
225:     Value *FuncRef = EmitScalarExpr(E->getArg(0));
226: 
227:     // Get the function type from the argument's static type
228:     QualType ArgType = E->getArg(0)->getType();
229:     const PointerType *PtrTy = ArgType->getAs<PointerType>();
230:     assert(PtrTy && "Sema should have ensured this is a function pointer");
231: 
232:     const FunctionType *FuncTy = PtrTy->getPointeeType()->getAs<FunctionType>();
233:     assert(FuncTy && "Sema should have ensured this is a function pointer");
234: 
235:     // In the llvm IR, we won't have access any more to the type of the function
236:     // pointer so we need to insert this type information somehow. The
237:     // @llvm.wasm.ref.test.func takes varargs arguments whose values are unused
238:     // to indicate the type of the function to test for. See the test here:
239:     // llvm/test/CodeGen/WebAssembly/ref-test-func.ll
240:     //
```
- **EN**: This block guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块使用断言或不可达标记保护关键不变量。

### Lines 241-256
```cpp
241:     // The format is: first we include the return types (since this is a C
242:     // function pointer, there will be 0 or one of these) then a token type to
243:     // indicate the boundary between return types and param types, then the
244:     // param types.
245: 
246:     llvm::FunctionType *LLVMFuncTy =
247:         cast<llvm::FunctionType>(ConvertType(QualType(FuncTy, 0)));
248: 
249:     bool VarArg = LLVMFuncTy->isVarArg();
250:     unsigned NParams = LLVMFuncTy->getNumParams();
251:     std::vector<Value *> Args;
252:     Args.reserve(NParams + 3 + VarArg);
253:     // The only real argument is the FuncRef
254:     Args.push_back(FuncRef);
255: 
256:     // Add the type information
```
- **EN**: This block documents intent or context for the surrounding target builtin lowering code.
- **CN**: 该代码块说明周围 目标内建函数降级 代码的意图或上下文。

### Lines 257-272
```cpp
257:     llvm::Type *RetType = LLVMFuncTy->getReturnType();
258:     if (!RetType->isVoidTy()) {
259:       Args.push_back(PoisonValue::get(RetType));
260:     }
261:     // The token type indicates the boundary between return types and param
262:     // types.
263:     Args.push_back(PoisonValue::get(llvm::Type::getTokenTy(getLLVMContext())));
264:     for (unsigned i = 0; i < NParams; i++) {
265:       Args.push_back(PoisonValue::get(LLVMFuncTy->getParamType(i)));
266:     }
267:     if (VarArg) {
268:       Args.push_back(PoisonValue::get(Builder.getPtrTy()));
269:     }
270:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_ref_test_func);
271:     return Builder.CreateCall(Callee, Args);
272:   }
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 273-288
```cpp
273:   case WebAssembly::BI__builtin_wasm_swizzle_i8x16: {
274:     Value *Src = EmitScalarExpr(E->getArg(0));
275:     Value *Indices = EmitScalarExpr(E->getArg(1));
276:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_swizzle);
277:     return Builder.CreateCall(Callee, {Src, Indices});
278:   }
279:   case WebAssembly::BI__builtin_wasm_abs_i8x16:
280:   case WebAssembly::BI__builtin_wasm_abs_i16x8:
281:   case WebAssembly::BI__builtin_wasm_abs_i32x4:
282:   case WebAssembly::BI__builtin_wasm_abs_i64x2: {
283:     Value *Vec = EmitScalarExpr(E->getArg(0));
284:     Value *Neg = Builder.CreateNeg(Vec, "neg");
285:     Constant *Zero = llvm::Constant::getNullValue(Vec->getType());
286:     Value *ICmp = Builder.CreateICmpSLT(Vec, Zero, "abscond");
287:     return Builder.CreateSelect(ICmp, Neg, Vec, "abs");
288:   }
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 289-304
```cpp
289:   case WebAssembly::BI__builtin_wasm_avgr_u_i8x16:
290:   case WebAssembly::BI__builtin_wasm_avgr_u_i16x8: {
291:     Value *LHS = EmitScalarExpr(E->getArg(0));
292:     Value *RHS = EmitScalarExpr(E->getArg(1));
293:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_avgr_unsigned,
294:                                         ConvertType(E->getType()));
295:     return Builder.CreateCall(Callee, {LHS, RHS});
296:   }
297:   case WebAssembly::BI__builtin_wasm_q15mulr_sat_s_i16x8: {
298:     Value *LHS = EmitScalarExpr(E->getArg(0));
299:     Value *RHS = EmitScalarExpr(E->getArg(1));
300:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_q15mulr_sat_signed);
301:     return Builder.CreateCall(Callee, {LHS, RHS});
302:   }
303:   case WebAssembly::BI__builtin_wasm_extadd_pairwise_i8x16_s_i16x8:
304:   case WebAssembly::BI__builtin_wasm_extadd_pairwise_i8x16_u_i16x8:
```
- **EN**: This block defines callable entry points like `ConvertType`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `ConvertType`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 305-320
```cpp
305:   case WebAssembly::BI__builtin_wasm_extadd_pairwise_i16x8_s_i32x4:
306:   case WebAssembly::BI__builtin_wasm_extadd_pairwise_i16x8_u_i32x4: {
307:     Value *Vec = EmitScalarExpr(E->getArg(0));
308:     unsigned IntNo;
309:     switch (BuiltinID) {
310:     case WebAssembly::BI__builtin_wasm_extadd_pairwise_i8x16_s_i16x8:
311:     case WebAssembly::BI__builtin_wasm_extadd_pairwise_i16x8_s_i32x4:
312:       IntNo = Intrinsic::wasm_extadd_pairwise_signed;
313:       break;
314:     case WebAssembly::BI__builtin_wasm_extadd_pairwise_i8x16_u_i16x8:
315:     case WebAssembly::BI__builtin_wasm_extadd_pairwise_i16x8_u_i32x4:
316:       IntNo = Intrinsic::wasm_extadd_pairwise_unsigned;
317:       break;
318:     default:
319:       llvm_unreachable("unexpected builtin ID");
320:     }
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 321-336
```cpp
321: 
322:     Function *Callee = CGM.getIntrinsic(IntNo, ConvertType(E->getType()));
323:     return Builder.CreateCall(Callee, Vec);
324:   }
325:   case WebAssembly::BI__builtin_wasm_bitselect: {
326:     Value *V1 = EmitScalarExpr(E->getArg(0));
327:     Value *V2 = EmitScalarExpr(E->getArg(1));
328:     Value *C = EmitScalarExpr(E->getArg(2));
329:     Function *Callee =
330:         CGM.getIntrinsic(Intrinsic::wasm_bitselect, ConvertType(E->getType()));
331:     return Builder.CreateCall(Callee, {V1, V2, C});
332:   }
333:   case WebAssembly::BI__builtin_wasm_dot_s_i32x4_i16x8: {
334:     Value *LHS = EmitScalarExpr(E->getArg(0));
335:     Value *RHS = EmitScalarExpr(E->getArg(1));
336:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_dot);
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 337-352
```cpp
337:     return Builder.CreateCall(Callee, {LHS, RHS});
338:   }
339:   case WebAssembly::BI__builtin_wasm_any_true_v128:
340:   case WebAssembly::BI__builtin_wasm_all_true_i8x16:
341:   case WebAssembly::BI__builtin_wasm_all_true_i16x8:
342:   case WebAssembly::BI__builtin_wasm_all_true_i32x4:
343:   case WebAssembly::BI__builtin_wasm_all_true_i64x2: {
344:     unsigned IntNo;
345:     switch (BuiltinID) {
346:     case WebAssembly::BI__builtin_wasm_any_true_v128:
347:       IntNo = Intrinsic::wasm_anytrue;
348:       break;
349:     case WebAssembly::BI__builtin_wasm_all_true_i8x16:
350:     case WebAssembly::BI__builtin_wasm_all_true_i16x8:
351:     case WebAssembly::BI__builtin_wasm_all_true_i32x4:
352:     case WebAssembly::BI__builtin_wasm_all_true_i64x2:
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 353-368
```cpp
353:       IntNo = Intrinsic::wasm_alltrue;
354:       break;
355:     default:
356:       llvm_unreachable("unexpected builtin ID");
357:     }
358:     Value *Vec = EmitScalarExpr(E->getArg(0));
359:     Function *Callee = CGM.getIntrinsic(IntNo, Vec->getType());
360:     return Builder.CreateCall(Callee, {Vec});
361:   }
362:   case WebAssembly::BI__builtin_wasm_bitmask_i8x16:
363:   case WebAssembly::BI__builtin_wasm_bitmask_i16x8:
364:   case WebAssembly::BI__builtin_wasm_bitmask_i32x4:
365:   case WebAssembly::BI__builtin_wasm_bitmask_i64x2: {
366:     Value *Vec = EmitScalarExpr(E->getArg(0));
367:     Function *Callee =
368:         CGM.getIntrinsic(Intrinsic::wasm_bitmask, Vec->getType());
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 369-384
```cpp
369:     return Builder.CreateCall(Callee, {Vec});
370:   }
371:   case WebAssembly::BI__builtin_wasm_abs_f16x8:
372:   case WebAssembly::BI__builtin_wasm_abs_f32x4:
373:   case WebAssembly::BI__builtin_wasm_abs_f64x2: {
374:     Value *Vec = EmitScalarExpr(E->getArg(0));
375:     return Builder.CreateFAbs(Vec);
376:   }
377:   case WebAssembly::BI__builtin_wasm_sqrt_f16x8:
378:   case WebAssembly::BI__builtin_wasm_sqrt_f32x4:
379:   case WebAssembly::BI__builtin_wasm_sqrt_f64x2: {
380:     Value *Vec = EmitScalarExpr(E->getArg(0));
381:     Function *Callee = CGM.getIntrinsic(Intrinsic::sqrt, Vec->getType());
382:     return Builder.CreateCall(Callee, {Vec});
383:   }
384:   case WebAssembly::BI__builtin_wasm_narrow_s_i8x16_i16x8:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 385-400
```cpp
385:   case WebAssembly::BI__builtin_wasm_narrow_u_i8x16_i16x8:
386:   case WebAssembly::BI__builtin_wasm_narrow_s_i16x8_i32x4:
387:   case WebAssembly::BI__builtin_wasm_narrow_u_i16x8_i32x4: {
388:     Value *Low = EmitScalarExpr(E->getArg(0));
389:     Value *High = EmitScalarExpr(E->getArg(1));
390:     unsigned IntNo;
391:     switch (BuiltinID) {
392:     case WebAssembly::BI__builtin_wasm_narrow_s_i8x16_i16x8:
393:     case WebAssembly::BI__builtin_wasm_narrow_s_i16x8_i32x4:
394:       IntNo = Intrinsic::wasm_narrow_signed;
395:       break;
396:     case WebAssembly::BI__builtin_wasm_narrow_u_i8x16_i16x8:
397:     case WebAssembly::BI__builtin_wasm_narrow_u_i16x8_i32x4:
398:       IntNo = Intrinsic::wasm_narrow_unsigned;
399:       break;
400:     default:
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 401-416
```cpp
401:       llvm_unreachable("unexpected builtin ID");
402:     }
403:     Function *Callee =
404:         CGM.getIntrinsic(IntNo, {ConvertType(E->getType()), Low->getType()});
405:     return Builder.CreateCall(Callee, {Low, High});
406:   }
407:   case WebAssembly::BI__builtin_wasm_trunc_sat_s_zero_f64x2_i32x4:
408:   case WebAssembly::BI__builtin_wasm_trunc_sat_u_zero_f64x2_i32x4: {
409:     Value *Vec = EmitScalarExpr(E->getArg(0));
410:     unsigned IntNo;
411:     switch (BuiltinID) {
412:     case WebAssembly::BI__builtin_wasm_trunc_sat_s_zero_f64x2_i32x4:
413:       IntNo = Intrinsic::fptosi_sat;
414:       break;
415:     case WebAssembly::BI__builtin_wasm_trunc_sat_u_zero_f64x2_i32x4:
416:       IntNo = Intrinsic::fptoui_sat;
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 417-432
```cpp
417:       break;
418:     default:
419:       llvm_unreachable("unexpected builtin ID");
420:     }
421:     llvm::Type *SrcT = Vec->getType();
422:     llvm::Type *TruncT = SrcT->getWithNewType(Builder.getInt32Ty());
423:     Function *Callee = CGM.getIntrinsic(IntNo, {TruncT, SrcT});
424:     Value *Trunc = Builder.CreateCall(Callee, Vec);
425:     Value *Splat = Constant::getNullValue(TruncT);
426:     return Builder.CreateShuffleVector(Trunc, Splat, {0, 1, 2, 3});
427:   }
428:   case WebAssembly::BI__builtin_wasm_shuffle_i8x16: {
429:     Value *Ops[18];
430:     size_t OpIdx = 0;
431:     Ops[OpIdx++] = EmitScalarExpr(E->getArg(0));
432:     Ops[OpIdx++] = EmitScalarExpr(E->getArg(1));
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 433-448
```cpp
433:     while (OpIdx < 18) {
434:       std::optional<llvm::APSInt> LaneConst =
435:           E->getArg(OpIdx)->getIntegerConstantExpr(getContext());
436:       assert(LaneConst && "Constant arg isn't actually constant?");
437:       Ops[OpIdx++] = llvm::ConstantInt::get(getLLVMContext(), *LaneConst);
438:     }
439:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_shuffle);
440:     return Builder.CreateCall(Callee, Ops);
441:   }
442:   case WebAssembly::BI__builtin_wasm_relaxed_madd_f16x8:
443:   case WebAssembly::BI__builtin_wasm_relaxed_nmadd_f16x8:
444:   case WebAssembly::BI__builtin_wasm_relaxed_madd_f32x4:
445:   case WebAssembly::BI__builtin_wasm_relaxed_nmadd_f32x4:
446:   case WebAssembly::BI__builtin_wasm_relaxed_madd_f64x2:
447:   case WebAssembly::BI__builtin_wasm_relaxed_nmadd_f64x2: {
448:     Value *A = EmitScalarExpr(E->getArg(0));
```
- **EN**: This block uses control flow (while, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（while, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 449-464
```cpp
449:     Value *B = EmitScalarExpr(E->getArg(1));
450:     Value *C = EmitScalarExpr(E->getArg(2));
451:     unsigned IntNo;
452:     switch (BuiltinID) {
453:     case WebAssembly::BI__builtin_wasm_relaxed_madd_f16x8:
454:     case WebAssembly::BI__builtin_wasm_relaxed_madd_f32x4:
455:     case WebAssembly::BI__builtin_wasm_relaxed_madd_f64x2:
456:       IntNo = Intrinsic::wasm_relaxed_madd;
457:       break;
458:     case WebAssembly::BI__builtin_wasm_relaxed_nmadd_f16x8:
459:     case WebAssembly::BI__builtin_wasm_relaxed_nmadd_f32x4:
460:     case WebAssembly::BI__builtin_wasm_relaxed_nmadd_f64x2:
461:       IntNo = Intrinsic::wasm_relaxed_nmadd;
462:       break;
463:     default:
464:       llvm_unreachable("unexpected builtin ID");
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 465-480
```cpp
465:     }
466:     Function *Callee = CGM.getIntrinsic(IntNo, A->getType());
467:     return Builder.CreateCall(Callee, {A, B, C});
468:   }
469:   case WebAssembly::BI__builtin_wasm_relaxed_laneselect_i8x16:
470:   case WebAssembly::BI__builtin_wasm_relaxed_laneselect_i16x8:
471:   case WebAssembly::BI__builtin_wasm_relaxed_laneselect_i32x4:
472:   case WebAssembly::BI__builtin_wasm_relaxed_laneselect_i64x2: {
473:     Value *A = EmitScalarExpr(E->getArg(0));
474:     Value *B = EmitScalarExpr(E->getArg(1));
475:     Value *C = EmitScalarExpr(E->getArg(2));
476:     Function *Callee =
477:         CGM.getIntrinsic(Intrinsic::wasm_relaxed_laneselect, A->getType());
478:     return Builder.CreateCall(Callee, {A, B, C});
479:   }
480:   case WebAssembly::BI__builtin_wasm_relaxed_swizzle_i8x16: {
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 481-496
```cpp
481:     Value *Src = EmitScalarExpr(E->getArg(0));
482:     Value *Indices = EmitScalarExpr(E->getArg(1));
483:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_relaxed_swizzle);
484:     return Builder.CreateCall(Callee, {Src, Indices});
485:   }
486:   case WebAssembly::BI__builtin_wasm_relaxed_min_f32x4:
487:   case WebAssembly::BI__builtin_wasm_relaxed_max_f32x4:
488:   case WebAssembly::BI__builtin_wasm_relaxed_min_f64x2:
489:   case WebAssembly::BI__builtin_wasm_relaxed_max_f64x2: {
490:     Value *LHS = EmitScalarExpr(E->getArg(0));
491:     Value *RHS = EmitScalarExpr(E->getArg(1));
492:     unsigned IntNo;
493:     switch (BuiltinID) {
494:     case WebAssembly::BI__builtin_wasm_relaxed_min_f32x4:
495:     case WebAssembly::BI__builtin_wasm_relaxed_min_f64x2:
496:       IntNo = Intrinsic::wasm_relaxed_min;
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 497-512
```cpp
497:       break;
498:     case WebAssembly::BI__builtin_wasm_relaxed_max_f32x4:
499:     case WebAssembly::BI__builtin_wasm_relaxed_max_f64x2:
500:       IntNo = Intrinsic::wasm_relaxed_max;
501:       break;
502:     default:
503:       llvm_unreachable("unexpected builtin ID");
504:     }
505:     Function *Callee = CGM.getIntrinsic(IntNo, LHS->getType());
506:     return Builder.CreateCall(Callee, {LHS, RHS});
507:   }
508:   case WebAssembly::BI__builtin_wasm_relaxed_trunc_s_i32x4_f32x4:
509:   case WebAssembly::BI__builtin_wasm_relaxed_trunc_u_i32x4_f32x4:
510:   case WebAssembly::BI__builtin_wasm_relaxed_trunc_s_zero_i32x4_f64x2:
511:   case WebAssembly::BI__builtin_wasm_relaxed_trunc_u_zero_i32x4_f64x2: {
512:     Value *Vec = EmitScalarExpr(E->getArg(0));
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 513-528
```cpp
513:     unsigned IntNo;
514:     switch (BuiltinID) {
515:     case WebAssembly::BI__builtin_wasm_relaxed_trunc_s_i32x4_f32x4:
516:       IntNo = Intrinsic::wasm_relaxed_trunc_signed;
517:       break;
518:     case WebAssembly::BI__builtin_wasm_relaxed_trunc_u_i32x4_f32x4:
519:       IntNo = Intrinsic::wasm_relaxed_trunc_unsigned;
520:       break;
521:     case WebAssembly::BI__builtin_wasm_relaxed_trunc_s_zero_i32x4_f64x2:
522:       IntNo = Intrinsic::wasm_relaxed_trunc_signed_zero;
523:       break;
524:     case WebAssembly::BI__builtin_wasm_relaxed_trunc_u_zero_i32x4_f64x2:
525:       IntNo = Intrinsic::wasm_relaxed_trunc_unsigned_zero;
526:       break;
527:     default:
528:       llvm_unreachable("unexpected builtin ID");
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 529-544
```cpp
529:     }
530:     Function *Callee = CGM.getIntrinsic(IntNo);
531:     return Builder.CreateCall(Callee, {Vec});
532:   }
533:   case WebAssembly::BI__builtin_wasm_relaxed_q15mulr_s_i16x8: {
534:     Value *LHS = EmitScalarExpr(E->getArg(0));
535:     Value *RHS = EmitScalarExpr(E->getArg(1));
536:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_relaxed_q15mulr_signed);
537:     return Builder.CreateCall(Callee, {LHS, RHS});
538:   }
539:   case WebAssembly::BI__builtin_wasm_relaxed_dot_i8x16_i7x16_s_i16x8: {
540:     Value *LHS = EmitScalarExpr(E->getArg(0));
541:     Value *RHS = EmitScalarExpr(E->getArg(1));
542:     Function *Callee =
543:         CGM.getIntrinsic(Intrinsic::wasm_relaxed_dot_i8x16_i7x16_signed);
544:     return Builder.CreateCall(Callee, {LHS, RHS});
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 545-560
```cpp
545:   }
546:   case WebAssembly::BI__builtin_wasm_relaxed_dot_i8x16_i7x16_add_s_i32x4: {
547:     Value *LHS = EmitScalarExpr(E->getArg(0));
548:     Value *RHS = EmitScalarExpr(E->getArg(1));
549:     Value *Acc = EmitScalarExpr(E->getArg(2));
550:     Function *Callee =
551:         CGM.getIntrinsic(Intrinsic::wasm_relaxed_dot_i8x16_i7x16_add_signed);
552:     return Builder.CreateCall(Callee, {LHS, RHS, Acc});
553:   }
554:   case WebAssembly::BI__builtin_wasm_relaxed_dot_bf16x8_add_f32_f32x4: {
555:     Value *LHS = EmitScalarExpr(E->getArg(0));
556:     Value *RHS = EmitScalarExpr(E->getArg(1));
557:     Value *Acc = EmitScalarExpr(E->getArg(2));
558:     Function *Callee =
559:         CGM.getIntrinsic(Intrinsic::wasm_relaxed_dot_bf16x8_add_f32);
560:     return Builder.CreateCall(Callee, {LHS, RHS, Acc});
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 561-576
```cpp
561:   }
562:   case WebAssembly::BI__builtin_wasm_loadf16_f32: {
563:     Value *Addr = EmitScalarExpr(E->getArg(0));
564:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_loadf16_f32);
565:     return Builder.CreateCall(Callee, {Addr});
566:   }
567:   case WebAssembly::BI__builtin_wasm_storef16_f32: {
568:     Value *Val = EmitScalarExpr(E->getArg(0));
569:     Value *Addr = EmitScalarExpr(E->getArg(1));
570:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_storef16_f32);
571:     return Builder.CreateCall(Callee, {Val, Addr});
572:   }
573:   case WebAssembly::BI__builtin_wasm_splat_f16x8: {
574:     Value *Val = EmitScalarExpr(E->getArg(0));
575:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_splat_f16x8);
576:     return Builder.CreateCall(Callee, {Val});
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 577-592
```cpp
577:   }
578:   case WebAssembly::BI__builtin_wasm_extract_lane_f16x8: {
579:     Value *Vector = EmitScalarExpr(E->getArg(0));
580:     Value *Index = EmitScalarExpr(E->getArg(1));
581:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_extract_lane_f16x8);
582:     return Builder.CreateCall(Callee, {Vector, Index});
583:   }
584:   case WebAssembly::BI__builtin_wasm_replace_lane_f16x8: {
585:     Value *Vector = EmitScalarExpr(E->getArg(0));
586:     Value *Index = EmitScalarExpr(E->getArg(1));
587:     Value *Val = EmitScalarExpr(E->getArg(2));
588:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_replace_lane_f16x8);
589:     return Builder.CreateCall(Callee, {Vector, Index, Val});
590:   }
591:   case WebAssembly::BI__builtin_wasm_table_get: {
592:     assert(E->getArg(0)->getType()->isArrayType());
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 593-608
```cpp
593:     Value *Table = EmitArrayToPointerDecay(E->getArg(0)).emitRawPointer(*this);
594:     Value *Index = EmitScalarExpr(E->getArg(1));
595:     Function *Callee;
596:     if (E->getType().isWebAssemblyExternrefType())
597:       Callee = CGM.getIntrinsic(Intrinsic::wasm_table_get_externref);
598:     else if (E->getType().isWebAssemblyFuncrefType())
599:       Callee = CGM.getIntrinsic(Intrinsic::wasm_table_get_funcref);
600:     else
601:       llvm_unreachable(
602:           "Unexpected reference type for __builtin_wasm_table_get");
603:     return Builder.CreateCall(Callee, {Table, Index});
604:   }
605:   case WebAssembly::BI__builtin_wasm_table_set: {
606:     assert(E->getArg(0)->getType()->isArrayType());
607:     Value *Table = EmitArrayToPointerDecay(E->getArg(0)).emitRawPointer(*this);
608:     Value *Index = EmitScalarExpr(E->getArg(1));
```
- **EN**: This block uses control flow (if, for, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 609-624
```cpp
609:     Value *Val = EmitScalarExpr(E->getArg(2));
610:     Function *Callee;
611:     if (E->getArg(2)->getType().isWebAssemblyExternrefType())
612:       Callee = CGM.getIntrinsic(Intrinsic::wasm_table_set_externref);
613:     else if (E->getArg(2)->getType().isWebAssemblyFuncrefType())
614:       Callee = CGM.getIntrinsic(Intrinsic::wasm_table_set_funcref);
615:     else
616:       llvm_unreachable(
617:           "Unexpected reference type for __builtin_wasm_table_set");
618:     return Builder.CreateCall(Callee, {Table, Index, Val});
619:   }
620:   case WebAssembly::BI__builtin_wasm_table_size: {
621:     assert(E->getArg(0)->getType()->isArrayType());
622:     Value *Value = EmitArrayToPointerDecay(E->getArg(0)).emitRawPointer(*this);
623:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_table_size);
624:     return Builder.CreateCall(Callee, Value);
```
- **EN**: This block uses control flow (if, for, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 625-640
```cpp
625:   }
626:   case WebAssembly::BI__builtin_wasm_table_grow: {
627:     assert(E->getArg(0)->getType()->isArrayType());
628:     Value *Table = EmitArrayToPointerDecay(E->getArg(0)).emitRawPointer(*this);
629:     Value *Val = EmitScalarExpr(E->getArg(1));
630:     Value *NElems = EmitScalarExpr(E->getArg(2));
631: 
632:     Function *Callee;
633:     if (E->getArg(1)->getType().isWebAssemblyExternrefType())
634:       Callee = CGM.getIntrinsic(Intrinsic::wasm_table_grow_externref);
635:     else if (E->getArg(1)->getType().isWebAssemblyFuncrefType())
636:       Callee = CGM.getIntrinsic(Intrinsic::wasm_table_grow_funcref);
637:     else
638:       llvm_unreachable(
639:           "Unexpected reference type for __builtin_wasm_table_grow");
640: 
```
- **EN**: This block uses control flow (if, for, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 641-656
```cpp
641:     return Builder.CreateCall(Callee, {Table, Val, NElems});
642:   }
643:   case WebAssembly::BI__builtin_wasm_table_fill: {
644:     assert(E->getArg(0)->getType()->isArrayType());
645:     Value *Table = EmitArrayToPointerDecay(E->getArg(0)).emitRawPointer(*this);
646:     Value *Index = EmitScalarExpr(E->getArg(1));
647:     Value *Val = EmitScalarExpr(E->getArg(2));
648:     Value *NElems = EmitScalarExpr(E->getArg(3));
649: 
650:     Function *Callee;
651:     if (E->getArg(2)->getType().isWebAssemblyExternrefType())
652:       Callee = CGM.getIntrinsic(Intrinsic::wasm_table_fill_externref);
653:     else if (E->getArg(2)->getType().isWebAssemblyFuncrefType())
654:       Callee = CGM.getIntrinsic(Intrinsic::wasm_table_fill_funcref);
655:     else
656:       llvm_unreachable(
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 657-672
```cpp
657:           "Unexpected reference type for __builtin_wasm_table_fill");
658: 
659:     return Builder.CreateCall(Callee, {Table, Index, Val, NElems});
660:   }
661:   case WebAssembly::BI__builtin_wasm_table_copy: {
662:     assert(E->getArg(0)->getType()->isArrayType());
663:     Value *TableX = EmitArrayToPointerDecay(E->getArg(0)).emitRawPointer(*this);
664:     Value *TableY = EmitArrayToPointerDecay(E->getArg(1)).emitRawPointer(*this);
665:     Value *DstIdx = EmitScalarExpr(E->getArg(2));
666:     Value *SrcIdx = EmitScalarExpr(E->getArg(3));
667:     Value *NElems = EmitScalarExpr(E->getArg(4));
668: 
669:     Function *Callee = CGM.getIntrinsic(Intrinsic::wasm_table_copy);
670: 
671:     return Builder.CreateCall(Callee, {TableX, TableY, SrcIdx, DstIdx, NElems});
672:   }
```
- **EN**: This block uses control flow (for, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（for, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 673-676
```cpp
673:   default:
674:     return nullptr;
675:   }
676: }
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

## Key Concepts / 关键概念

- **Callee**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **getArg**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **EmitScalarExpr**: Suggests an entry point that materializes IR or helper objects for target builtin lowering. / 暗示其是为 目标内建函数降级 生成 IR 或辅助对象的入口。
- **Intrinsic**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Builder**: Acts as a construction helper that incrementally assembles target builtin lowering state. / 充当构建辅助器，逐步组装 目标内建函数降级 状态。
- **CGM**: Central symbol in this file's implementation of target builtin lowering. / 是该文件实现 目标内建函数降级 时的核心符号。
- **getIntrinsic**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CGBuiltin.h`
- **Clang libraries / Clang 库**: `clang/Basic/TargetBuiltins.h`
- **LLVM libraries / LLVM 库**: `llvm/ADT/APInt.h`, `llvm/IR/Constants.h`, `llvm/IR/IntrinsicsWebAssembly.h`, `llvm/Support/ErrorHandling.h`
