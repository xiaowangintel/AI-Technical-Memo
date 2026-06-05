# SystemZ.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/TargetBuiltins/SystemZ.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements builtin handling and lowering logic for the SystemZ backend.
- **Purpose (CN) / 目的（中文）**: 实现 SystemZ 后端的内建函数处理与降级逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: //===------ SystemZ.cpp - Emit LLVM Code for builtins ---------------------===//
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
15: #include "llvm/IR/IntrinsicsS390.h"
16: 
```
- **EN**: This block imports local CodeGen headers `CodeGenFunction.h`; Clang headers `clang/Basic/TargetBuiltins.h`; LLVM headers `llvm/IR/IntrinsicsS390.h`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CodeGenFunction.h`；Clang 头文件 `clang/Basic/TargetBuiltins.h`；LLVM 头文件 `llvm/IR/IntrinsicsS390.h`；包含影响本编译单元构建方式的预处理结构。

### Lines 17-32
```cpp
17: using namespace clang;
18: using namespace CodeGen;
19: using namespace llvm;
20: 
21: /// Handle a SystemZ function in which the final argument is a pointer
22: /// to an int that receives the post-instruction CC value.  At the LLVM level
23: /// this is represented as a function that returns a {result, cc} pair.
24: static Value *EmitSystemZIntrinsicWithCC(CodeGenFunction &CGF,
25:                                          unsigned IntrinsicID,
26:                                          const CallExpr *E) {
27:   unsigned NumArgs = E->getNumArgs() - 1;
28:   SmallVector<Value *, 8> Args(NumArgs);
29:   for (unsigned I = 0; I < NumArgs; ++I)
30:     Args[I] = CGF.EmitScalarExpr(E->getArg(I));
31:   Address CCPtr = CGF.EmitPointerWithAlignment(E->getArg(NumArgs));
32:   Function *F = CGF.CGM.getIntrinsic(IntrinsicID);
```
- **EN**: This block opens or references namespaces `clang`, `CodeGen`, `llvm`; defines callable entry points like `Args`; uses control flow (for) to specialize target builtin lowering.
- **CN**: 该代码块打开或引用命名空间 `clang`, `CodeGen`, `llvm`；定义可调用入口，例如 `Args`；通过控制流（for）细化 目标内建函数降级 行为。

### Lines 33-48
```cpp
33:   Value *Call = CGF.Builder.CreateCall(F, Args);
34:   Value *CC = CGF.Builder.CreateExtractValue(Call, 1);
35:   CGF.Builder.CreateStore(CC, CCPtr);
36:   return CGF.Builder.CreateExtractValue(Call, 0);
37: }
38: 
39: Value *CodeGenFunction::EmitSystemZBuiltinExpr(unsigned BuiltinID,
40:                                                const CallExpr *E) {
41:   switch (BuiltinID) {
42:   case SystemZ::BI__builtin_tbegin: {
43:     Value *TDB = EmitScalarExpr(E->getArg(0));
44:     Value *Control = llvm::ConstantInt::get(Int32Ty, 0xff0c);
45:     Function *F = CGM.getIntrinsic(Intrinsic::s390_tbegin);
46:     return Builder.CreateCall(F, {TDB, Control});
47:   }
48:   case SystemZ::BI__builtin_tbegin_nofloat: {
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 49-64
```cpp
49:     Value *TDB = EmitScalarExpr(E->getArg(0));
50:     Value *Control = llvm::ConstantInt::get(Int32Ty, 0xff0c);
51:     Function *F = CGM.getIntrinsic(Intrinsic::s390_tbegin_nofloat);
52:     return Builder.CreateCall(F, {TDB, Control});
53:   }
54:   case SystemZ::BI__builtin_tbeginc: {
55:     Value *TDB = llvm::ConstantPointerNull::get(Int8PtrTy);
56:     Value *Control = llvm::ConstantInt::get(Int32Ty, 0xff08);
57:     Function *F = CGM.getIntrinsic(Intrinsic::s390_tbeginc);
58:     return Builder.CreateCall(F, {TDB, Control});
59:   }
60:   case SystemZ::BI__builtin_tabort: {
61:     Value *Data = EmitScalarExpr(E->getArg(0));
62:     Function *F = CGM.getIntrinsic(Intrinsic::s390_tabort);
63:     return Builder.CreateCall(F, Builder.CreateSExt(Data, Int64Ty, "tabort"));
64:   }
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 65-80
```cpp
65:   case SystemZ::BI__builtin_non_tx_store: {
66:     Value *Address = EmitScalarExpr(E->getArg(0));
67:     Value *Data = EmitScalarExpr(E->getArg(1));
68:     Function *F = CGM.getIntrinsic(Intrinsic::s390_ntstg);
69:     return Builder.CreateCall(F, {Data, Address});
70:   }
71: 
72:   // Vector builtins.  Note that most vector builtins are mapped automatically
73:   // to target-specific LLVM intrinsics.  The ones handled specially here can
74:   // be represented via standard LLVM IR, which is preferable to enable common
75:   // LLVM optimizations.
76: 
77:   case SystemZ::BI__builtin_s390_vclzb:
78:   case SystemZ::BI__builtin_s390_vclzh:
79:   case SystemZ::BI__builtin_s390_vclzf:
80:   case SystemZ::BI__builtin_s390_vclzg:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 81-96
```cpp
81:   case SystemZ::BI__builtin_s390_vclzq: {
82:     llvm::Type *ResultType = ConvertType(E->getType());
83:     Value *X = EmitScalarExpr(E->getArg(0));
84:     Value *Undef = ConstantInt::get(Builder.getInt1Ty(), false);
85:     Function *F = CGM.getIntrinsic(Intrinsic::ctlz, ResultType);
86:     return Builder.CreateCall(F, {X, Undef});
87:   }
88: 
89:   case SystemZ::BI__builtin_s390_vctzb:
90:   case SystemZ::BI__builtin_s390_vctzh:
91:   case SystemZ::BI__builtin_s390_vctzf:
92:   case SystemZ::BI__builtin_s390_vctzg:
93:   case SystemZ::BI__builtin_s390_vctzq: {
94:     llvm::Type *ResultType = ConvertType(E->getType());
95:     Value *X = EmitScalarExpr(E->getArg(0));
96:     Value *Undef = ConstantInt::get(Builder.getInt1Ty(), false);
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 97-112
```cpp
 97:     Function *F = CGM.getIntrinsic(Intrinsic::cttz, ResultType);
 98:     return Builder.CreateCall(F, {X, Undef});
 99:   }
100: 
101:   case SystemZ::BI__builtin_s390_verllb:
102:   case SystemZ::BI__builtin_s390_verllh:
103:   case SystemZ::BI__builtin_s390_verllf:
104:   case SystemZ::BI__builtin_s390_verllg: {
105:     llvm::Type *ResultType = ConvertType(E->getType());
106:     llvm::Value *Src = EmitScalarExpr(E->getArg(0));
107:     llvm::Value *Amt = EmitScalarExpr(E->getArg(1));
108:     // Splat scalar rotate amount to vector type.
109:     unsigned NumElts = cast<llvm::FixedVectorType>(ResultType)->getNumElements();
110:     Amt = Builder.CreateIntCast(Amt, ResultType->getScalarType(), false);
111:     Amt = Builder.CreateVectorSplat(NumElts, Amt);
112:     Function *F = CGM.getIntrinsic(Intrinsic::fshl, ResultType);
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 113-128
```cpp
113:     return Builder.CreateCall(F, { Src, Src, Amt });
114:   }
115: 
116:   case SystemZ::BI__builtin_s390_verllvb:
117:   case SystemZ::BI__builtin_s390_verllvh:
118:   case SystemZ::BI__builtin_s390_verllvf:
119:   case SystemZ::BI__builtin_s390_verllvg: {
120:     llvm::Type *ResultType = ConvertType(E->getType());
121:     llvm::Value *Src = EmitScalarExpr(E->getArg(0));
122:     llvm::Value *Amt = EmitScalarExpr(E->getArg(1));
123:     Function *F = CGM.getIntrinsic(Intrinsic::fshl, ResultType);
124:     return Builder.CreateCall(F, { Src, Src, Amt });
125:   }
126: 
127:   case SystemZ::BI__builtin_s390_vfsqsb:
128:   case SystemZ::BI__builtin_s390_vfsqdb: {
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 129-144
```cpp
129:     llvm::Type *ResultType = ConvertType(E->getType());
130:     Value *X = EmitScalarExpr(E->getArg(0));
131:     if (Builder.getIsFPConstrained()) {
132:       Function *F = CGM.getIntrinsic(Intrinsic::experimental_constrained_sqrt, ResultType);
133:       return Builder.CreateConstrainedFPCall(F, { X });
134:     } else {
135:       Function *F = CGM.getIntrinsic(Intrinsic::sqrt, ResultType);
136:       return Builder.CreateCall(F, X);
137:     }
138:   }
139:   case SystemZ::BI__builtin_s390_vfmasb:
140:   case SystemZ::BI__builtin_s390_vfmadb: {
141:     llvm::Type *ResultType = ConvertType(E->getType());
142:     Value *X = EmitScalarExpr(E->getArg(0));
143:     Value *Y = EmitScalarExpr(E->getArg(1));
144:     Value *Z = EmitScalarExpr(E->getArg(2));
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 145-160
```cpp
145:     if (Builder.getIsFPConstrained()) {
146:       Function *F = CGM.getIntrinsic(Intrinsic::experimental_constrained_fma, ResultType);
147:       return Builder.CreateConstrainedFPCall(F, {X, Y, Z});
148:     } else {
149:       Function *F = CGM.getIntrinsic(Intrinsic::fma, ResultType);
150:       return Builder.CreateCall(F, {X, Y, Z});
151:     }
152:   }
153:   case SystemZ::BI__builtin_s390_vfmssb:
154:   case SystemZ::BI__builtin_s390_vfmsdb: {
155:     llvm::Type *ResultType = ConvertType(E->getType());
156:     Value *X = EmitScalarExpr(E->getArg(0));
157:     Value *Y = EmitScalarExpr(E->getArg(1));
158:     Value *Z = EmitScalarExpr(E->getArg(2));
159:     if (Builder.getIsFPConstrained()) {
160:       Function *F = CGM.getIntrinsic(Intrinsic::experimental_constrained_fma, ResultType);
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 161-176
```cpp
161:       return Builder.CreateConstrainedFPCall(F, {X, Y, Builder.CreateFNeg(Z, "neg")});
162:     } else {
163:       Function *F = CGM.getIntrinsic(Intrinsic::fma, ResultType);
164:       return Builder.CreateCall(F, {X, Y, Builder.CreateFNeg(Z, "neg")});
165:     }
166:   }
167:   case SystemZ::BI__builtin_s390_vfnmasb:
168:   case SystemZ::BI__builtin_s390_vfnmadb: {
169:     llvm::Type *ResultType = ConvertType(E->getType());
170:     Value *X = EmitScalarExpr(E->getArg(0));
171:     Value *Y = EmitScalarExpr(E->getArg(1));
172:     Value *Z = EmitScalarExpr(E->getArg(2));
173:     if (Builder.getIsFPConstrained()) {
174:       Function *F = CGM.getIntrinsic(Intrinsic::experimental_constrained_fma, ResultType);
175:       return Builder.CreateFNeg(Builder.CreateConstrainedFPCall(F, {X, Y,  Z}), "neg");
176:     } else {
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 177-192
```cpp
177:       Function *F = CGM.getIntrinsic(Intrinsic::fma, ResultType);
178:       return Builder.CreateFNeg(Builder.CreateCall(F, {X, Y, Z}), "neg");
179:     }
180:   }
181:   case SystemZ::BI__builtin_s390_vfnmssb:
182:   case SystemZ::BI__builtin_s390_vfnmsdb: {
183:     llvm::Type *ResultType = ConvertType(E->getType());
184:     Value *X = EmitScalarExpr(E->getArg(0));
185:     Value *Y = EmitScalarExpr(E->getArg(1));
186:     Value *Z = EmitScalarExpr(E->getArg(2));
187:     if (Builder.getIsFPConstrained()) {
188:       Function *F = CGM.getIntrinsic(Intrinsic::experimental_constrained_fma, ResultType);
189:       Value *NegZ = Builder.CreateFNeg(Z, "sub");
190:       return Builder.CreateFNeg(Builder.CreateConstrainedFPCall(F, {X, Y, NegZ}));
191:     } else {
192:       Function *F = CGM.getIntrinsic(Intrinsic::fma, ResultType);
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 193-208
```cpp
193:       Value *NegZ = Builder.CreateFNeg(Z, "neg");
194:       return Builder.CreateFNeg(Builder.CreateCall(F, {X, Y, NegZ}));
195:     }
196:   }
197:   case SystemZ::BI__builtin_s390_vflpsb:
198:   case SystemZ::BI__builtin_s390_vflpdb: {
199:     Value *X = EmitScalarExpr(E->getArg(0));
200:     return Builder.CreateFAbs(X);
201:   }
202:   case SystemZ::BI__builtin_s390_vflnsb:
203:   case SystemZ::BI__builtin_s390_vflndb: {
204:     Value *X = EmitScalarExpr(E->getArg(0));
205:     return Builder.CreateFNeg(Builder.CreateFAbs(X), "neg");
206:   }
207:   case SystemZ::BI__builtin_s390_vfisb:
208:   case SystemZ::BI__builtin_s390_vfidb: {
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 209-224
```cpp
209:     llvm::Type *ResultType = ConvertType(E->getType());
210:     Value *X = EmitScalarExpr(E->getArg(0));
211:     // Constant-fold the M4 and M5 mask arguments.
212:     llvm::APSInt M4 = *E->getArg(1)->getIntegerConstantExpr(getContext());
213:     llvm::APSInt M5 = *E->getArg(2)->getIntegerConstantExpr(getContext());
214:     // Check whether this instance can be represented via a LLVM standard
215:     // intrinsic.  We only support some combinations of M4 and M5.
216:     Intrinsic::ID ID = Intrinsic::not_intrinsic;
217:     Intrinsic::ID CI;
218:     switch (M4.getZExtValue()) {
219:     default: break;
220:     case 0:  // IEEE-inexact exception allowed
221:       switch (M5.getZExtValue()) {
222:       default: break;
223:       case 0: ID = Intrinsic::rint;
224:               CI = Intrinsic::experimental_constrained_rint; break;
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 225-240
```cpp
225:       }
226:       break;
227:     case 4:  // IEEE-inexact exception suppressed
228:       switch (M5.getZExtValue()) {
229:       default: break;
230:       case 0: ID = Intrinsic::nearbyint;
231:               CI = Intrinsic::experimental_constrained_nearbyint; break;
232:       case 1: ID = Intrinsic::round;
233:               CI = Intrinsic::experimental_constrained_round; break;
234:       case 4: ID = Intrinsic::roundeven;
235:               CI = Intrinsic::experimental_constrained_roundeven; break;
236:       case 5: ID = Intrinsic::trunc;
237:               CI = Intrinsic::experimental_constrained_trunc; break;
238:       case 6: ID = Intrinsic::ceil;
239:               CI = Intrinsic::experimental_constrained_ceil; break;
240:       case 7: ID = Intrinsic::floor;
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 241-256
```cpp
241:               CI = Intrinsic::experimental_constrained_floor; break;
242:       }
243:       break;
244:     }
245:     if (ID != Intrinsic::not_intrinsic) {
246:       if (Builder.getIsFPConstrained()) {
247:         Function *F = CGM.getIntrinsic(CI, ResultType);
248:         return Builder.CreateConstrainedFPCall(F, X);
249:       } else {
250:         Function *F = CGM.getIntrinsic(ID, ResultType);
251:         return Builder.CreateCall(F, X);
252:       }
253:     }
254:     switch (BuiltinID) { // FIXME: constrained version?
255:       case SystemZ::BI__builtin_s390_vfisb: ID = Intrinsic::s390_vfisb; break;
256:       case SystemZ::BI__builtin_s390_vfidb: ID = Intrinsic::s390_vfidb; break;
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 257-272
```cpp
257:       default: llvm_unreachable("Unknown BuiltinID");
258:     }
259:     Function *F = CGM.getIntrinsic(ID);
260:     Value *M4Value = llvm::ConstantInt::get(getLLVMContext(), M4);
261:     Value *M5Value = llvm::ConstantInt::get(getLLVMContext(), M5);
262:     return Builder.CreateCall(F, {X, M4Value, M5Value});
263:   }
264:   case SystemZ::BI__builtin_s390_vfmaxsb:
265:   case SystemZ::BI__builtin_s390_vfmaxdb: {
266:     llvm::Type *ResultType = ConvertType(E->getType());
267:     Value *X = EmitScalarExpr(E->getArg(0));
268:     Value *Y = EmitScalarExpr(E->getArg(1));
269:     // Constant-fold the M4 mask argument.
270:     llvm::APSInt M4 = *E->getArg(2)->getIntegerConstantExpr(getContext());
271:     // Check whether this instance can be represented via a LLVM standard
272:     // intrinsic.  We only support some values of M4.
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 273-288
```cpp
273:     Intrinsic::ID ID = Intrinsic::not_intrinsic;
274:     Intrinsic::ID CI;
275:     switch (M4.getZExtValue()) {
276:     default: break;
277:     case 4: ID = Intrinsic::maxnum;
278:             CI = Intrinsic::experimental_constrained_maxnum; break;
279:     }
280:     if (ID != Intrinsic::not_intrinsic) {
281:       if (Builder.getIsFPConstrained()) {
282:         Function *F = CGM.getIntrinsic(CI, ResultType);
283:         return Builder.CreateConstrainedFPCall(F, {X, Y});
284:       } else {
285:         Function *F = CGM.getIntrinsic(ID, ResultType);
286:         return Builder.CreateCall(F, {X, Y});
287:       }
288:     }
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 289-304
```cpp
289:     switch (BuiltinID) {
290:       case SystemZ::BI__builtin_s390_vfmaxsb: ID = Intrinsic::s390_vfmaxsb; break;
291:       case SystemZ::BI__builtin_s390_vfmaxdb: ID = Intrinsic::s390_vfmaxdb; break;
292:       default: llvm_unreachable("Unknown BuiltinID");
293:     }
294:     Function *F = CGM.getIntrinsic(ID);
295:     Value *M4Value = llvm::ConstantInt::get(getLLVMContext(), M4);
296:     return Builder.CreateCall(F, {X, Y, M4Value});
297:   }
298:   case SystemZ::BI__builtin_s390_vfminsb:
299:   case SystemZ::BI__builtin_s390_vfmindb: {
300:     llvm::Type *ResultType = ConvertType(E->getType());
301:     Value *X = EmitScalarExpr(E->getArg(0));
302:     Value *Y = EmitScalarExpr(E->getArg(1));
303:     // Constant-fold the M4 mask argument.
304:     llvm::APSInt M4 = *E->getArg(2)->getIntegerConstantExpr(getContext());
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 305-320
```cpp
305:     // Check whether this instance can be represented via a LLVM standard
306:     // intrinsic.  We only support some values of M4.
307:     Intrinsic::ID ID = Intrinsic::not_intrinsic;
308:     Intrinsic::ID CI;
309:     switch (M4.getZExtValue()) {
310:     default: break;
311:     case 4: ID = Intrinsic::minnum;
312:             CI = Intrinsic::experimental_constrained_minnum; break;
313:     }
314:     if (ID != Intrinsic::not_intrinsic) {
315:       if (Builder.getIsFPConstrained()) {
316:         Function *F = CGM.getIntrinsic(CI, ResultType);
317:         return Builder.CreateConstrainedFPCall(F, {X, Y});
318:       } else {
319:         Function *F = CGM.getIntrinsic(ID, ResultType);
320:         return Builder.CreateCall(F, {X, Y});
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 321-336
```cpp
321:       }
322:     }
323:     switch (BuiltinID) {
324:       case SystemZ::BI__builtin_s390_vfminsb: ID = Intrinsic::s390_vfminsb; break;
325:       case SystemZ::BI__builtin_s390_vfmindb: ID = Intrinsic::s390_vfmindb; break;
326:       default: llvm_unreachable("Unknown BuiltinID");
327:     }
328:     Function *F = CGM.getIntrinsic(ID);
329:     Value *M4Value = llvm::ConstantInt::get(getLLVMContext(), M4);
330:     return Builder.CreateCall(F, {X, Y, M4Value});
331:   }
332: 
333:   case SystemZ::BI__builtin_s390_vlbrh:
334:   case SystemZ::BI__builtin_s390_vlbrf:
335:   case SystemZ::BI__builtin_s390_vlbrg:
336:   case SystemZ::BI__builtin_s390_vlbrq: {
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 337-352
```cpp
337:     llvm::Type *ResultType = ConvertType(E->getType());
338:     Value *X = EmitScalarExpr(E->getArg(0));
339:     Function *F = CGM.getIntrinsic(Intrinsic::bswap, ResultType);
340:     return Builder.CreateCall(F, X);
341:   }
342: 
343:   // Vector intrinsics that output the post-instruction CC value.
344: 
345: #define INTRINSIC_WITH_CC(NAME) \
346:     case SystemZ::BI__builtin_##NAME: \
347:       return EmitSystemZIntrinsicWithCC(*this, Intrinsic::NAME, E)
348: 
349:   INTRINSIC_WITH_CC(s390_vpkshs);
350:   INTRINSIC_WITH_CC(s390_vpksfs);
351:   INTRINSIC_WITH_CC(s390_vpksgs);
352: 
```
- **EN**: This block spells out callable entry points like `EmitSystemZIntrinsicWithCC`, `INTRINSIC_WITH_CC`; uses control flow (case) to specialize target builtin lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitSystemZIntrinsicWithCC`, `INTRINSIC_WITH_CC`；通过控制流（case）细化 目标内建函数降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 353-368
```cpp
353:   INTRINSIC_WITH_CC(s390_vpklshs);
354:   INTRINSIC_WITH_CC(s390_vpklsfs);
355:   INTRINSIC_WITH_CC(s390_vpklsgs);
356: 
357:   INTRINSIC_WITH_CC(s390_vceqbs);
358:   INTRINSIC_WITH_CC(s390_vceqhs);
359:   INTRINSIC_WITH_CC(s390_vceqfs);
360:   INTRINSIC_WITH_CC(s390_vceqgs);
361:   INTRINSIC_WITH_CC(s390_vceqqs);
362: 
363:   INTRINSIC_WITH_CC(s390_vchbs);
364:   INTRINSIC_WITH_CC(s390_vchhs);
365:   INTRINSIC_WITH_CC(s390_vchfs);
366:   INTRINSIC_WITH_CC(s390_vchgs);
367:   INTRINSIC_WITH_CC(s390_vchqs);
368: 
```
- **EN**: This block spells out callable entry points like `INTRINSIC_WITH_CC`.
- **CN**: 该代码块给出可调用入口的声明，例如 `INTRINSIC_WITH_CC`。

### Lines 369-384
```cpp
369:   INTRINSIC_WITH_CC(s390_vchlbs);
370:   INTRINSIC_WITH_CC(s390_vchlhs);
371:   INTRINSIC_WITH_CC(s390_vchlfs);
372:   INTRINSIC_WITH_CC(s390_vchlgs);
373:   INTRINSIC_WITH_CC(s390_vchlqs);
374: 
375:   INTRINSIC_WITH_CC(s390_vfaebs);
376:   INTRINSIC_WITH_CC(s390_vfaehs);
377:   INTRINSIC_WITH_CC(s390_vfaefs);
378: 
379:   INTRINSIC_WITH_CC(s390_vfaezbs);
380:   INTRINSIC_WITH_CC(s390_vfaezhs);
381:   INTRINSIC_WITH_CC(s390_vfaezfs);
382: 
383:   INTRINSIC_WITH_CC(s390_vfeebs);
384:   INTRINSIC_WITH_CC(s390_vfeehs);
```
- **EN**: This block spells out callable entry points like `INTRINSIC_WITH_CC`.
- **CN**: 该代码块给出可调用入口的声明，例如 `INTRINSIC_WITH_CC`。

### Lines 385-400
```cpp
385:   INTRINSIC_WITH_CC(s390_vfeefs);
386: 
387:   INTRINSIC_WITH_CC(s390_vfeezbs);
388:   INTRINSIC_WITH_CC(s390_vfeezhs);
389:   INTRINSIC_WITH_CC(s390_vfeezfs);
390: 
391:   INTRINSIC_WITH_CC(s390_vfenebs);
392:   INTRINSIC_WITH_CC(s390_vfenehs);
393:   INTRINSIC_WITH_CC(s390_vfenefs);
394: 
395:   INTRINSIC_WITH_CC(s390_vfenezbs);
396:   INTRINSIC_WITH_CC(s390_vfenezhs);
397:   INTRINSIC_WITH_CC(s390_vfenezfs);
398: 
399:   INTRINSIC_WITH_CC(s390_vistrbs);
400:   INTRINSIC_WITH_CC(s390_vistrhs);
```
- **EN**: This block spells out callable entry points like `INTRINSIC_WITH_CC`.
- **CN**: 该代码块给出可调用入口的声明，例如 `INTRINSIC_WITH_CC`。

### Lines 401-416
```cpp
401:   INTRINSIC_WITH_CC(s390_vistrfs);
402: 
403:   INTRINSIC_WITH_CC(s390_vstrcbs);
404:   INTRINSIC_WITH_CC(s390_vstrchs);
405:   INTRINSIC_WITH_CC(s390_vstrcfs);
406: 
407:   INTRINSIC_WITH_CC(s390_vstrczbs);
408:   INTRINSIC_WITH_CC(s390_vstrczhs);
409:   INTRINSIC_WITH_CC(s390_vstrczfs);
410: 
411:   INTRINSIC_WITH_CC(s390_vfcesbs);
412:   INTRINSIC_WITH_CC(s390_vfcedbs);
413:   INTRINSIC_WITH_CC(s390_vfchsbs);
414:   INTRINSIC_WITH_CC(s390_vfchdbs);
415:   INTRINSIC_WITH_CC(s390_vfchesbs);
416:   INTRINSIC_WITH_CC(s390_vfchedbs);
```
- **EN**: This block spells out callable entry points like `INTRINSIC_WITH_CC`.
- **CN**: 该代码块给出可调用入口的声明，例如 `INTRINSIC_WITH_CC`。

### Lines 417-432
```cpp
417: 
418:   INTRINSIC_WITH_CC(s390_vftcisb);
419:   INTRINSIC_WITH_CC(s390_vftcidb);
420: 
421:   INTRINSIC_WITH_CC(s390_vstrsb);
422:   INTRINSIC_WITH_CC(s390_vstrsh);
423:   INTRINSIC_WITH_CC(s390_vstrsf);
424: 
425:   INTRINSIC_WITH_CC(s390_vstrszb);
426:   INTRINSIC_WITH_CC(s390_vstrszh);
427:   INTRINSIC_WITH_CC(s390_vstrszf);
428: 
429: #undef INTRINSIC_WITH_CC
430: 
431:   default:
432:     return nullptr;
```
- **EN**: This block spells out callable entry points like `INTRINSIC_WITH_CC`; returns or forwards computed values for the surrounding target builtin lowering logic; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块给出可调用入口的声明，例如 `INTRINSIC_WITH_CC`；为周围的 目标内建函数降级 逻辑返回或转发计算结果；包含影响本编译单元构建方式的预处理结构。

### Lines 433-434
```cpp
433:   }
434: }
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

## Key Concepts / 关键概念

- **INTRINSIC_WITH_CC**: Central symbol in this file's implementation of target builtin lowering. / 是该文件实现 目标内建函数降级 时的核心符号。
- **Builder**: Acts as a construction helper that incrementally assembles target builtin lowering state. / 充当构建辅助器，逐步组装 目标内建函数降级 状态。
- **Intrinsic**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **getArg**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ResultType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **EmitScalarExpr**: Suggests an entry point that materializes IR or helper objects for target builtin lowering. / 暗示其是为 目标内建函数降级 生成 IR 或辅助对象的入口。
- **Function**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CodeGenFunction.h`
- **Clang libraries / Clang 库**: `clang/Basic/TargetBuiltins.h`
- **LLVM libraries / LLVM 库**: `llvm/IR/IntrinsicsS390.h`
