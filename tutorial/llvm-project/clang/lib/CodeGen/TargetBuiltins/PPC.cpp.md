# PPC.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/TargetBuiltins/PPC.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements builtin handling and lowering logic for the PPC backend.
- **Purpose (CN) / 目的（中文）**: 实现 PPC 后端的内建函数处理与降级逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: //===---------- PPC.cpp - Emit LLVM Code for builtins ---------------------===//
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
15: #include "llvm/IR/InlineAsm.h"
16: #include "llvm/IR/IntrinsicsPowerPC.h"
17: #include "llvm/Support/ScopedPrinter.h"
18: 
19: using namespace clang;
20: using namespace CodeGen;
```
- **EN**: This block imports local CodeGen headers `CGBuiltin.h`; Clang headers `clang/Basic/TargetBuiltins.h`; LLVM headers `llvm/IR/InlineAsm.h`, `llvm/IR/IntrinsicsPowerPC.h`, `llvm/Support/ScopedPrinter.h`; opens or references namespaces `clang`, `CodeGen`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CGBuiltin.h`；Clang 头文件 `clang/Basic/TargetBuiltins.h`；LLVM 头文件 `llvm/IR/InlineAsm.h`, `llvm/IR/IntrinsicsPowerPC.h`, `llvm/Support/ScopedPrinter.h`；打开或引用命名空间 `clang`, `CodeGen`；包含影响本编译单元构建方式的预处理结构。

### Lines 21-40
```cpp
21: using namespace llvm;
22: 
23: static llvm::Value *emitPPCLoadReserveIntrinsic(CodeGenFunction &CGF,
24:                                                 unsigned BuiltinID,
25:                                                 const CallExpr *E) {
26:   Value *Addr = CGF.EmitScalarExpr(E->getArg(0));
27: 
28:   SmallString<64> Asm;
29:   raw_svector_ostream AsmOS(Asm);
30:   llvm::IntegerType *RetType = CGF.Int32Ty;
31: 
32:   switch (BuiltinID) {
33:   case clang::PPC::BI__builtin_ppc_ldarx:
34:     AsmOS << "ldarx ";
35:     RetType = CGF.Int64Ty;
36:     break;
37:   case clang::PPC::BI__builtin_ppc_lwarx:
38:     AsmOS << "lwarx ";
39:     RetType = CGF.Int32Ty;
40:     break;
```
- **EN**: This block opens or references namespaces `llvm`; defines callable entry points like `AsmOS`; uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块打开或引用命名空间 `llvm`；定义可调用入口，例如 `AsmOS`；通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 41-60
```cpp
41:   case clang::PPC::BI__builtin_ppc_lharx:
42:     AsmOS << "lharx ";
43:     RetType = CGF.Int16Ty;
44:     break;
45:   case clang::PPC::BI__builtin_ppc_lbarx:
46:     AsmOS << "lbarx ";
47:     RetType = CGF.Int8Ty;
48:     break;
49:   default:
50:     llvm_unreachable("Expected only PowerPC load reserve intrinsics");
51:   }
52: 
53:   AsmOS << "$0, ${1:y}";
54: 
55:   std::string Constraints = "=r,*Z,~{memory}";
56:   std::string_view MachineClobbers = CGF.getTarget().getClobbers();
57:   if (!MachineClobbers.empty()) {
58:     Constraints += ',';
59:     Constraints += MachineClobbers;
60:   }
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 61-80
```cpp
61: 
62:   llvm::Type *PtrType = CGF.DefaultPtrTy;
63:   llvm::FunctionType *FTy = llvm::FunctionType::get(RetType, {PtrType}, false);
64: 
65:   llvm::InlineAsm *IA =
66:       llvm::InlineAsm::get(FTy, Asm, Constraints, /*hasSideEffects=*/true);
67:   llvm::CallInst *CI = CGF.Builder.CreateCall(IA, {Addr});
68:   CI->addParamAttr(
69:       0, Attribute::get(CGF.getLLVMContext(), Attribute::ElementType, RetType));
70:   return CI;
71: }
72: 
73: Value *CodeGenFunction::EmitPPCBuiltinCpu(unsigned BuiltinID,
74:                                           llvm::Type *ReturnType,
75:                                           StringRef CPUStr) {
76:   assert(BuiltinID == Builtin::BI__builtin_cpu_is ||
77:          BuiltinID == Builtin::BI__builtin_cpu_supports);
78: 
79: #include "llvm/TargetParser/PPCTargetParser.def"
80:   auto GenAIXPPCBuiltinCpuExpr = [&](unsigned SupportMethod, unsigned FieldIdx,
```
- **EN**: This block imports LLVM headers `llvm/TargetParser/PPCTargetParser.def`; defines callable entry points like `get`; returns or forwards computed values for the surrounding target builtin lowering logic; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 LLVM 头文件 `llvm/TargetParser/PPCTargetParser.def`；定义可调用入口，例如 `get`；为周围的 目标内建函数降级 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

### Lines 81-100
```cpp
 81:                                      unsigned Mask, CmpInst::Predicate CompOp,
 82:                                      unsigned OpValue) -> Value * {
 83:     if (SupportMethod == BUILTIN_PPC_FALSE)
 84:       return llvm::ConstantInt::getFalse(ReturnType);
 85: 
 86:     if (SupportMethod == BUILTIN_PPC_TRUE)
 87:       return llvm::ConstantInt::getTrue(ReturnType);
 88: 
 89:     assert(SupportMethod <= SYS_CALL && "Invalid value for SupportMethod.");
 90: 
 91:     llvm::Value *FieldValue = nullptr;
 92:     if (SupportMethod == USE_SYS_CONF) {
 93:       llvm::Type *STy = llvm::StructType::get(PPC_SYSTEMCONFIG_TYPE);
 94:       llvm::Constant *SysConf =
 95:           CGM.CreateRuntimeVariable(STy, "_system_configuration");
 96: 
 97:       // Grab the appropriate field from _system_configuration.
 98:       llvm::Value *Idxs[] = {ConstantInt::get(Int32Ty, 0),
 99:                              ConstantInt::get(Int32Ty, FieldIdx)};
100: 
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 101-120
```cpp
101:       FieldValue = Builder.CreateInBoundsGEP(STy, SysConf, Idxs);
102:       FieldValue = Builder.CreateAlignedLoad(Int32Ty, FieldValue,
103:                                              CharUnits::fromQuantity(4));
104:     } else if (SupportMethod == SYS_CALL) {
105:       llvm::FunctionType *FTy =
106:           llvm::FunctionType::get(Int64Ty, Int32Ty, false);
107:       llvm::FunctionCallee Func =
108:           CGM.CreateRuntimeFunction(FTy, "getsystemcfg");
109: 
110:       FieldValue =
111:           Builder.CreateCall(Func, {ConstantInt::get(Int32Ty, FieldIdx)});
112:     }
113:     assert(FieldValue &&
114:            "SupportMethod value is not defined in PPCTargetParser.def.");
115: 
116:     if (Mask)
117:       FieldValue = Builder.CreateAnd(FieldValue, Mask);
118: 
119:     llvm::Type *ValueType = FieldValue->getType();
120:     bool IsValueType64Bit = ValueType->isIntegerTy(64);
```
- **EN**: This block defines callable entry points like `fromQuantity`, `get`; uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `fromQuantity`, `get`；通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 121-140
```cpp
121:     assert(
122:         (IsValueType64Bit || ValueType->isIntegerTy(32)) &&
123:         "Only 32/64-bit integers are supported in GenAIXPPCBuiltinCpuExpr().");
124: 
125:     return Builder.CreateICmp(
126:         CompOp, FieldValue,
127:         ConstantInt::get(IsValueType64Bit ? Int64Ty : Int32Ty, OpValue));
128:   };
129: 
130:   if (BuiltinID == Builtin::BI__builtin_cpu_is) {
131:     llvm::Triple Triple = getTarget().getTriple();
132: 
133:     typedef std::tuple<unsigned, unsigned, unsigned, unsigned> CPUInfo;
134: 
135:     auto [LinuxSupportMethod, LinuxIDValue, AIXSupportMethod, AIXIDValue] =
136:         static_cast<CPUInfo>(StringSwitch<CPUInfo>(CPUStr)
137: #define PPC_CPU(NAME, Linux_SUPPORT_METHOD, LinuxID, AIX_SUPPORT_METHOD,       \
138:                 AIXID)                                                         \
139:   .Case(NAME, {Linux_SUPPORT_METHOD, LinuxID, AIX_SUPPORT_METHOD, AIXID})
140: #include "llvm/TargetParser/PPCTargetParser.def"
```
- **EN**: This block imports LLVM headers `llvm/TargetParser/PPCTargetParser.def`; defines callable entry points like `get`; uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 LLVM 头文件 `llvm/TargetParser/PPCTargetParser.def`；定义可调用入口，例如 `get`；通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

### Lines 141-160
```cpp
141:                                  .Default({BUILTIN_PPC_UNSUPPORTED, 0,
142:                                            BUILTIN_PPC_UNSUPPORTED, 0}));
143: 
144:     if (Triple.isOSAIX()) {
145:       assert((AIXSupportMethod != BUILTIN_PPC_UNSUPPORTED) &&
146:              "Invalid CPU name. Missed by SemaChecking?");
147:       return GenAIXPPCBuiltinCpuExpr(AIXSupportMethod, AIX_SYSCON_IMPL_IDX, 0,
148:                                      ICmpInst::ICMP_EQ, AIXIDValue);
149:     }
150: 
151:     assert(Triple.isOSLinux() &&
152:            "__builtin_cpu_is() is only supported for AIX and Linux.");
153: 
154:     assert((LinuxSupportMethod != BUILTIN_PPC_UNSUPPORTED) &&
155:            "Invalid CPU name. Missed by SemaChecking?");
156: 
157:     if (LinuxSupportMethod == BUILTIN_PPC_FALSE)
158:       return llvm::ConstantInt::getFalse(ReturnType);
159: 
160:     Value *Op0 = llvm::ConstantInt::get(Int32Ty, PPC_FAWORD_CPUID);
```
- **EN**: This block defines callable entry points like `GenAIXPPCBuiltinCpuExpr`; uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `GenAIXPPCBuiltinCpuExpr`；通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 161-180
```cpp
161:     llvm::Function *F = CGM.getIntrinsic(Intrinsic::ppc_fixed_addr_ld);
162:     Value *TheCall = Builder.CreateCall(F, {Op0}, "cpu_is");
163:     return Builder.CreateICmpEQ(TheCall,
164:                                 llvm::ConstantInt::get(Int32Ty, LinuxIDValue));
165:   }
166:   // else BuiltinID == Builtin::BI__builtin_cpu_supports
167:   llvm::Triple Triple = getTarget().getTriple();
168:   if (Triple.isOSAIX()) {
169:     typedef std::tuple<unsigned, unsigned, unsigned, CmpInst::Predicate,
170:                        unsigned>
171:         CPUSupportType;
172:     auto [SupportMethod, FieldIdx, Mask, CompOp, Value] =
173:         static_cast<CPUSupportType>(
174:             StringSwitch<CPUSupportType>(CPUStr)
175: #define PPC_AIX_FEATURE(NAME, DESC, SUPPORT_METHOD, INDEX, MASK, COMP_OP,      \
176:                         VALUE)                                                 \
177:   .Case(NAME, {SUPPORT_METHOD, INDEX, MASK, COMP_OP, VALUE})
178: #include "llvm/TargetParser/PPCTargetParser.def"
179:                 .Default({BUILTIN_PPC_FALSE, 0, 0, CmpInst::Predicate(), 0}));
180:     return GenAIXPPCBuiltinCpuExpr(SupportMethod, FieldIdx, Mask, CompOp,
```
- **EN**: This block imports LLVM headers `llvm/TargetParser/PPCTargetParser.def`; defines callable entry points like `get`; uses control flow (if) to specialize target builtin lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 LLVM 头文件 `llvm/TargetParser/PPCTargetParser.def`；定义可调用入口，例如 `get`；通过控制流（if）细化 目标内建函数降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 181-200
```cpp
181:                                    Value);
182:   }
183: 
184:   assert(Triple.isOSLinux() &&
185:          "__builtin_cpu_supports() is only supported for AIX and Linux.");
186:   auto [FeatureWord, BitMask] =
187:       StringSwitch<std::pair<unsigned, unsigned>>(CPUStr)
188: #define PPC_LNX_FEATURE(Name, Description, EnumName, Bitmask, FA_WORD)         \
189:   .Case(Name, {FA_WORD, Bitmask})
190: #include "llvm/TargetParser/PPCTargetParser.def"
191:           .Default({0, 0});
192:   if (!BitMask)
193:     return Builder.getFalse();
194:   Value *Op0 = llvm::ConstantInt::get(Int32Ty, FeatureWord);
195:   llvm::Function *F = CGM.getIntrinsic(Intrinsic::ppc_fixed_addr_ld);
196:   Value *TheCall = Builder.CreateCall(F, {Op0}, "cpu_supports");
197:   Value *Mask =
198:       Builder.CreateAnd(TheCall, llvm::ConstantInt::get(Int32Ty, BitMask));
199:   return Builder.CreateICmpNE(Mask, llvm::Constant::getNullValue(Int32Ty));
200: #undef PPC_FAWORD_HWCAP
```
- **EN**: This block imports LLVM headers `llvm/TargetParser/PPCTargetParser.def`; uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 LLVM 头文件 `llvm/TargetParser/PPCTargetParser.def`；通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

### Lines 201-220
```cpp
201: #undef PPC_FAWORD_HWCAP2
202: #undef PPC_FAWORD_CPUID
203: }
204: 
205: Value *CodeGenFunction::EmitPPCBuiltinExpr(unsigned BuiltinID,
206:                                            const CallExpr *E) {
207:   // Do not emit the builtin arguments in the arguments of a function call,
208:   // because the evaluation order of function arguments is not specified in C++.
209:   // This is important when testing to ensure the arguments are emitted in the
210:   // same order every time. Eg:
211:   // Instead of:
212:   //   return Builder.CreateFDiv(EmitScalarExpr(E->getArg(0)),
213:   //                             EmitScalarExpr(E->getArg(1)), "swdiv");
214:   // Use:
215:   //   Value *Op0 = EmitScalarExpr(E->getArg(0));
216:   //   Value *Op1 = EmitScalarExpr(E->getArg(1));
217:   //   return Builder.CreateFDiv(Op0, Op1, "swdiv")
218: 
219:   Intrinsic::ID ID = Intrinsic::not_intrinsic;
220: 
```
- **EN**: This block contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块包含影响本编译单元构建方式的预处理结构。

### Lines 221-240
```cpp
221:   switch (BuiltinID) {
222:   default:
223:     return nullptr;
224: 
225:   case Builtin::BI__builtin_cpu_is:
226:   case Builtin::BI__builtin_cpu_supports: {
227:     const Expr *CPUExpr = E->getArg(0)->IgnoreParenCasts();
228:     StringRef CPUStr = cast<clang::StringLiteral>(CPUExpr)->getString();
229:     return EmitPPCBuiltinCpu(BuiltinID, ConvertType(E->getType()), CPUStr);
230:   }
231:   // __builtin_ppc_get_timebase is GCC 4.8+'s PowerPC-specific name for what we
232:   // call __builtin_readcyclecounter.
233:   case PPC::BI__builtin_ppc_get_timebase:
234:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::readcyclecounter));
235: 
236:   // vec_ld, vec_xl_be, vec_lvsl, vec_lvsr
237:   case PPC::BI__builtin_altivec_lvx:
238:   case PPC::BI__builtin_altivec_lvxl:
239:   case PPC::BI__builtin_altivec_lvebx:
240:   case PPC::BI__builtin_altivec_lvehx:
```
- **EN**: This block defines callable entry points like `EmitPPCBuiltinCpu`; uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitPPCBuiltinCpu`；通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 241-260
```cpp
241:   case PPC::BI__builtin_altivec_lvewx:
242:   case PPC::BI__builtin_altivec_lvsl:
243:   case PPC::BI__builtin_altivec_lvsr:
244:   case PPC::BI__builtin_vsx_lxvd2x:
245:   case PPC::BI__builtin_vsx_lxvw4x:
246:   case PPC::BI__builtin_vsx_lxvd2x_be:
247:   case PPC::BI__builtin_vsx_lxvw4x_be:
248:   case PPC::BI__builtin_vsx_lxvl:
249:   case PPC::BI__builtin_vsx_lxvll:
250:   {
251:     SmallVector<Value *, 2> Ops;
252:     Ops.push_back(EmitScalarExpr(E->getArg(0)));
253:     Ops.push_back(EmitScalarExpr(E->getArg(1)));
254:     if (!(BuiltinID == PPC::BI__builtin_vsx_lxvl ||
255:           BuiltinID == PPC::BI__builtin_vsx_lxvll)) {
256:       Ops[0] = Builder.CreateGEP(Int8Ty, Ops[1], Ops[0]);
257:       Ops.pop_back();
258:     }
259: 
260:     switch (BuiltinID) {
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 261-280
```cpp
261:     default: llvm_unreachable("Unsupported ld/lvsl/lvsr intrinsic!");
262:     case PPC::BI__builtin_altivec_lvx:
263:       ID = Intrinsic::ppc_altivec_lvx;
264:       break;
265:     case PPC::BI__builtin_altivec_lvxl:
266:       ID = Intrinsic::ppc_altivec_lvxl;
267:       break;
268:     case PPC::BI__builtin_altivec_lvebx:
269:       ID = Intrinsic::ppc_altivec_lvebx;
270:       break;
271:     case PPC::BI__builtin_altivec_lvehx:
272:       ID = Intrinsic::ppc_altivec_lvehx;
273:       break;
274:     case PPC::BI__builtin_altivec_lvewx:
275:       ID = Intrinsic::ppc_altivec_lvewx;
276:       break;
277:     case PPC::BI__builtin_altivec_lvsl:
278:       ID = Intrinsic::ppc_altivec_lvsl;
279:       break;
280:     case PPC::BI__builtin_altivec_lvsr:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 281-300
```cpp
281:       ID = Intrinsic::ppc_altivec_lvsr;
282:       break;
283:     case PPC::BI__builtin_vsx_lxvd2x:
284:       ID = Intrinsic::ppc_vsx_lxvd2x;
285:       break;
286:     case PPC::BI__builtin_vsx_lxvw4x:
287:       ID = Intrinsic::ppc_vsx_lxvw4x;
288:       break;
289:     case PPC::BI__builtin_vsx_lxvd2x_be:
290:       ID = Intrinsic::ppc_vsx_lxvd2x_be;
291:       break;
292:     case PPC::BI__builtin_vsx_lxvw4x_be:
293:       ID = Intrinsic::ppc_vsx_lxvw4x_be;
294:       break;
295:     case PPC::BI__builtin_vsx_lxvl:
296:       ID = Intrinsic::ppc_vsx_lxvl;
297:       break;
298:     case PPC::BI__builtin_vsx_lxvll:
299:       ID = Intrinsic::ppc_vsx_lxvll;
300:       break;
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 301-320
```cpp
301:     }
302:     llvm::Function *F = CGM.getIntrinsic(ID);
303:     return Builder.CreateCall(F, Ops, "");
304:   }
305: 
306:   // vec_st, vec_xst_be
307:   case PPC::BI__builtin_altivec_stvx:
308:   case PPC::BI__builtin_altivec_stvxl:
309:   case PPC::BI__builtin_altivec_stvebx:
310:   case PPC::BI__builtin_altivec_stvehx:
311:   case PPC::BI__builtin_altivec_stvewx:
312:   case PPC::BI__builtin_vsx_stxvd2x:
313:   case PPC::BI__builtin_vsx_stxvw4x:
314:   case PPC::BI__builtin_vsx_stxvd2x_be:
315:   case PPC::BI__builtin_vsx_stxvw4x_be:
316:   case PPC::BI__builtin_vsx_stxvl:
317:   case PPC::BI__builtin_vsx_stxvll:
318:   {
319:     SmallVector<Value *, 3> Ops;
320:     Ops.push_back(EmitScalarExpr(E->getArg(0)));
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 321-340
```cpp
321:     Ops.push_back(EmitScalarExpr(E->getArg(1)));
322:     Ops.push_back(EmitScalarExpr(E->getArg(2)));
323:     if (!(BuiltinID == PPC::BI__builtin_vsx_stxvl ||
324:           BuiltinID == PPC::BI__builtin_vsx_stxvll)) {
325:       Ops[1] = Builder.CreateGEP(Int8Ty, Ops[2], Ops[1]);
326:       Ops.pop_back();
327:     }
328: 
329:     switch (BuiltinID) {
330:     default: llvm_unreachable("Unsupported st intrinsic!");
331:     case PPC::BI__builtin_altivec_stvx:
332:       ID = Intrinsic::ppc_altivec_stvx;
333:       break;
334:     case PPC::BI__builtin_altivec_stvxl:
335:       ID = Intrinsic::ppc_altivec_stvxl;
336:       break;
337:     case PPC::BI__builtin_altivec_stvebx:
338:       ID = Intrinsic::ppc_altivec_stvebx;
339:       break;
340:     case PPC::BI__builtin_altivec_stvehx:
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 341-360
```cpp
341:       ID = Intrinsic::ppc_altivec_stvehx;
342:       break;
343:     case PPC::BI__builtin_altivec_stvewx:
344:       ID = Intrinsic::ppc_altivec_stvewx;
345:       break;
346:     case PPC::BI__builtin_vsx_stxvd2x:
347:       ID = Intrinsic::ppc_vsx_stxvd2x;
348:       break;
349:     case PPC::BI__builtin_vsx_stxvw4x:
350:       ID = Intrinsic::ppc_vsx_stxvw4x;
351:       break;
352:     case PPC::BI__builtin_vsx_stxvd2x_be:
353:       ID = Intrinsic::ppc_vsx_stxvd2x_be;
354:       break;
355:     case PPC::BI__builtin_vsx_stxvw4x_be:
356:       ID = Intrinsic::ppc_vsx_stxvw4x_be;
357:       break;
358:     case PPC::BI__builtin_vsx_stxvl:
359:       ID = Intrinsic::ppc_vsx_stxvl;
360:       break;
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 361-380
```cpp
361:     case PPC::BI__builtin_vsx_stxvll:
362:       ID = Intrinsic::ppc_vsx_stxvll;
363:       break;
364:     }
365:     llvm::Function *F = CGM.getIntrinsic(ID);
366:     return Builder.CreateCall(F, Ops, "");
367:   }
368:   case PPC::BI__builtin_vsx_ldrmb: {
369:     // Essentially boils down to performing an unaligned VMX load sequence so
370:     // as to avoid crossing a page boundary and then shuffling the elements
371:     // into the right side of the vector register.
372:     Value *Op0 = EmitScalarExpr(E->getArg(0));
373:     Value *Op1 = EmitScalarExpr(E->getArg(1));
374:     int64_t NumBytes = cast<ConstantInt>(Op1)->getZExtValue();
375:     llvm::Type *ResTy = ConvertType(E->getType());
376:     bool IsLE = getTarget().isLittleEndian();
377: 
378:     // If the user wants the entire vector, just load the entire vector.
379:     if (NumBytes == 16) {
380:       Value *LD =
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 381-400
```cpp
381:           Builder.CreateLoad(Address(Op0, ResTy, CharUnits::fromQuantity(1)));
382:       if (!IsLE)
383:         return LD;
384: 
385:       // Reverse the bytes on LE.
386:       SmallVector<int, 16> RevMask;
387:       for (int Idx = 0; Idx < 16; Idx++)
388:         RevMask.push_back(15 - Idx);
389:       return Builder.CreateShuffleVector(LD, LD, RevMask);
390:     }
391: 
392:     llvm::Function *Lvx = CGM.getIntrinsic(Intrinsic::ppc_altivec_lvx);
393:     llvm::Function *Lvs = CGM.getIntrinsic(IsLE ? Intrinsic::ppc_altivec_lvsr
394:                                                 : Intrinsic::ppc_altivec_lvsl);
395:     llvm::Function *Vperm = CGM.getIntrinsic(Intrinsic::ppc_altivec_vperm);
396:     Value *HiMem = Builder.CreateGEP(
397:         Int8Ty, Op0, ConstantInt::get(Op1->getType(), NumBytes - 1));
398:     Value *LoLd = Builder.CreateCall(Lvx, Op0, "ld.lo");
399:     Value *HiLd = Builder.CreateCall(Lvx, HiMem, "ld.hi");
400:     Value *Mask1 = Builder.CreateCall(Lvs, Op0, "mask1");
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 401-420
```cpp
401: 
402:     Op0 = IsLE ? HiLd : LoLd;
403:     Op1 = IsLE ? LoLd : HiLd;
404:     Value *AllElts = Builder.CreateCall(Vperm, {Op0, Op1, Mask1}, "shuffle1");
405:     Constant *Zero = llvm::Constant::getNullValue(IsLE ? ResTy : AllElts->getType());
406: 
407:     if (IsLE) {
408:       SmallVector<int, 16> Consts;
409:       for (int Idx = 0; Idx < 16; Idx++) {
410:         int Val = (NumBytes - Idx - 1 >= 0) ? (NumBytes - Idx - 1)
411:                                             : 16 - (NumBytes - Idx);
412:         Consts.push_back(Val);
413:       }
414:       return Builder.CreateShuffleVector(Builder.CreateBitCast(AllElts, ResTy),
415:                                          Zero, Consts);
416:     }
417:     SmallVector<Constant *, 16> Consts;
418:     for (int Idx = 0; Idx < 16; Idx++)
419:       Consts.push_back(Builder.getInt8(NumBytes + Idx));
420:     Value *Mask2 = ConstantVector::get(Consts);
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 421-440
```cpp
421:     return Builder.CreateBitCast(
422:         Builder.CreateCall(Vperm, {Zero, AllElts, Mask2}, "shuffle2"), ResTy);
423:   }
424:   case PPC::BI__builtin_vsx_strmb: {
425:     Value *Op0 = EmitScalarExpr(E->getArg(0));
426:     Value *Op1 = EmitScalarExpr(E->getArg(1));
427:     Value *Op2 = EmitScalarExpr(E->getArg(2));
428:     int64_t NumBytes = cast<ConstantInt>(Op1)->getZExtValue();
429:     bool IsLE = getTarget().isLittleEndian();
430:     auto StoreSubVec = [&](unsigned Width, unsigned Offset, unsigned EltNo) {
431:       // Storing the whole vector, simply store it on BE and reverse bytes and
432:       // store on LE.
433:       if (Width == 16) {
434:         Value *StVec = Op2;
435:         if (IsLE) {
436:           SmallVector<int, 16> RevMask;
437:           for (int Idx = 0; Idx < 16; Idx++)
438:             RevMask.push_back(15 - Idx);
439:           StVec = Builder.CreateShuffleVector(Op2, Op2, RevMask);
440:         }
```
- **EN**: This block uses control flow (if, for, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, for, case）细化 目标内建函数降级 行为。

### Lines 441-460
```cpp
441:         return Builder.CreateStore(
442:             StVec, Address(Op0, Op2->getType(), CharUnits::fromQuantity(1)));
443:       }
444:       auto *ConvTy = Int64Ty;
445:       unsigned NumElts = 0;
446:       switch (Width) {
447:       default:
448:         llvm_unreachable("width for stores must be a power of 2");
449:       case 8:
450:         ConvTy = Int64Ty;
451:         NumElts = 2;
452:         break;
453:       case 4:
454:         ConvTy = Int32Ty;
455:         NumElts = 4;
456:         break;
457:       case 2:
458:         ConvTy = Int16Ty;
459:         NumElts = 8;
460:         break;
```
- **EN**: This block defines callable entry points like `Address`; uses control flow (switch, for, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `Address`；通过控制流（switch, for, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 461-480
```cpp
461:       case 1:
462:         ConvTy = Int8Ty;
463:         NumElts = 16;
464:         break;
465:       }
466:       Value *Vec = Builder.CreateBitCast(
467:           Op2, llvm::FixedVectorType::get(ConvTy, NumElts));
468:       Value *Ptr =
469:           Builder.CreateGEP(Int8Ty, Op0, ConstantInt::get(Int64Ty, Offset));
470:       Value *Elt = Builder.CreateExtractElement(Vec, EltNo);
471:       if (IsLE && Width > 1) {
472:         Function *F = CGM.getIntrinsic(Intrinsic::bswap, ConvTy);
473:         Elt = Builder.CreateCall(F, Elt);
474:       }
475:       return Builder.CreateStore(
476:           Elt, Address(Ptr, ConvTy, CharUnits::fromQuantity(1)));
477:     };
478:     unsigned Stored = 0;
479:     unsigned RemainingBytes = NumBytes;
480:     Value *Result;
```
- **EN**: This block defines callable entry points like `get`, `Address`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`, `Address`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 481-500
```cpp
481:     if (NumBytes == 16)
482:       return StoreSubVec(16, 0, 0);
483:     if (NumBytes >= 8) {
484:       Result = StoreSubVec(8, NumBytes - 8, IsLE ? 0 : 1);
485:       RemainingBytes -= 8;
486:       Stored += 8;
487:     }
488:     if (RemainingBytes >= 4) {
489:       Result = StoreSubVec(4, NumBytes - Stored - 4,
490:                            IsLE ? (Stored >> 2) : 3 - (Stored >> 2));
491:       RemainingBytes -= 4;
492:       Stored += 4;
493:     }
494:     if (RemainingBytes >= 2) {
495:       Result = StoreSubVec(2, NumBytes - Stored - 2,
496:                            IsLE ? (Stored >> 1) : 7 - (Stored >> 1));
497:       RemainingBytes -= 2;
498:       Stored += 2;
499:     }
500:     if (RemainingBytes)
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 501-520
```cpp
501:       Result =
502:           StoreSubVec(1, NumBytes - Stored - 1, IsLE ? Stored : 15 - Stored);
503:     return Result;
504:   }
505:   // Square root
506:   case PPC::BI__builtin_vsx_xvsqrtsp:
507:   case PPC::BI__builtin_vsx_xvsqrtdp: {
508:     llvm::Type *ResultType = ConvertType(E->getType());
509:     Value *X = EmitScalarExpr(E->getArg(0));
510:     if (Builder.getIsFPConstrained()) {
511:       llvm::Function *F = CGM.getIntrinsic(
512:           Intrinsic::experimental_constrained_sqrt, ResultType);
513:       return Builder.CreateConstrainedFPCall(F, X);
514:     } else {
515:       llvm::Function *F = CGM.getIntrinsic(Intrinsic::sqrt, ResultType);
516:       return Builder.CreateCall(F, X);
517:     }
518:   }
519:   // Count leading zeros
520:   case PPC::BI__builtin_altivec_vclzb:
```
- **EN**: This block defines callable entry points like `StoreSubVec`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `StoreSubVec`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 521-540
```cpp
521:   case PPC::BI__builtin_altivec_vclzh:
522:   case PPC::BI__builtin_altivec_vclzw:
523:   case PPC::BI__builtin_altivec_vclzd: {
524:     llvm::Type *ResultType = ConvertType(E->getType());
525:     Value *X = EmitScalarExpr(E->getArg(0));
526:     Value *Undef = ConstantInt::get(Builder.getInt1Ty(), false);
527:     Function *F = CGM.getIntrinsic(Intrinsic::ctlz, ResultType);
528:     return Builder.CreateCall(F, {X, Undef});
529:   }
530:   case PPC::BI__builtin_altivec_vctzb:
531:   case PPC::BI__builtin_altivec_vctzh:
532:   case PPC::BI__builtin_altivec_vctzw:
533:   case PPC::BI__builtin_altivec_vctzd: {
534:     llvm::Type *ResultType = ConvertType(E->getType());
535:     Value *X = EmitScalarExpr(E->getArg(0));
536:     Value *Undef = ConstantInt::get(Builder.getInt1Ty(), false);
537:     Function *F = CGM.getIntrinsic(Intrinsic::cttz, ResultType);
538:     return Builder.CreateCall(F, {X, Undef});
539:   }
540:   case PPC::BI__builtin_altivec_vinsd:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 541-560
```cpp
541:   case PPC::BI__builtin_altivec_vinsw:
542:   case PPC::BI__builtin_altivec_vinsd_elt:
543:   case PPC::BI__builtin_altivec_vinsw_elt: {
544:     llvm::Type *ResultType = ConvertType(E->getType());
545:     Value *Op0 = EmitScalarExpr(E->getArg(0));
546:     Value *Op1 = EmitScalarExpr(E->getArg(1));
547:     Value *Op2 = EmitScalarExpr(E->getArg(2));
548: 
549:     bool IsUnaligned = (BuiltinID == PPC::BI__builtin_altivec_vinsw ||
550:                         BuiltinID == PPC::BI__builtin_altivec_vinsd);
551: 
552:     bool Is32bit = (BuiltinID == PPC::BI__builtin_altivec_vinsw ||
553:                     BuiltinID == PPC::BI__builtin_altivec_vinsw_elt);
554: 
555:     // The third argument must be a compile time constant.
556:     ConstantInt *ArgCI = dyn_cast<ConstantInt>(Op2);
557:     assert(ArgCI &&
558:            "Third Arg to vinsw/vinsd intrinsic must be a constant integer!");
559: 
560:     // Valid value for the third argument is dependent on the input type and
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 561-580
```cpp
561:     // builtin called.
562:     int ValidMaxValue = 0;
563:     if (IsUnaligned)
564:       ValidMaxValue = (Is32bit) ? 12 : 8;
565:     else
566:       ValidMaxValue = (Is32bit) ? 3 : 1;
567: 
568:     // Get value of third argument.
569:     int64_t ConstArg = ArgCI->getSExtValue();
570: 
571:     // Compose range checking error message.
572:     std::string RangeErrMsg = IsUnaligned ? "byte" : "element";
573:     RangeErrMsg += " number " + llvm::to_string(ConstArg);
574:     RangeErrMsg += " is outside of the valid range [0, ";
575:     RangeErrMsg += llvm::to_string(ValidMaxValue) + "]";
576: 
577:     // Issue error if third argument is not within the valid range.
578:     if (ConstArg < 0 || ConstArg > ValidMaxValue)
579:       CGM.Error(E->getExprLoc(), RangeErrMsg);
580: 
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 581-600
```cpp
581:     // Input to vec_replace_elt is an element index, convert to byte index.
582:     if (!IsUnaligned) {
583:       ConstArg *= Is32bit ? 4 : 8;
584:       // Fix the constant according to endianess.
585:       if (getTarget().isLittleEndian())
586:         ConstArg = (Is32bit ? 12 : 8) - ConstArg;
587:     }
588: 
589:     ID = Is32bit ? Intrinsic::ppc_altivec_vinsw : Intrinsic::ppc_altivec_vinsd;
590:     Op2 = ConstantInt::getSigned(Int32Ty, ConstArg);
591:     // Casting input to vector int as per intrinsic definition.
592:     Op0 =
593:         Is32bit
594:             ? Builder.CreateBitCast(Op0, llvm::FixedVectorType::get(Int32Ty, 4))
595:             : Builder.CreateBitCast(Op0,
596:                                     llvm::FixedVectorType::get(Int64Ty, 2));
597:     return Builder.CreateBitCast(
598:         Builder.CreateCall(CGM.getIntrinsic(ID), {Op0, Op1, Op2}), ResultType);
599:   }
600:   case PPC::BI__builtin_altivec_vadduqm:
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 601-620
```cpp
601:   case PPC::BI__builtin_altivec_vsubuqm: {
602:     Value *Op0 = EmitScalarExpr(E->getArg(0));
603:     Value *Op1 = EmitScalarExpr(E->getArg(1));
604:     llvm::Type *Int128Ty = llvm::IntegerType::get(getLLVMContext(), 128);
605:     Op0 = Builder.CreateBitCast(Op0, llvm::FixedVectorType::get(Int128Ty, 1));
606:     Op1 = Builder.CreateBitCast(Op1, llvm::FixedVectorType::get(Int128Ty, 1));
607:     if (BuiltinID == PPC::BI__builtin_altivec_vadduqm)
608:       return Builder.CreateAdd(Op0, Op1, "vadduqm");
609:     else
610:       return Builder.CreateSub(Op0, Op1, "vsubuqm");
611:   }
612:   case PPC::BI__builtin_altivec_vaddcuq_c:
613:   case PPC::BI__builtin_altivec_vsubcuq_c: {
614:     SmallVector<Value *, 2> Ops;
615:     Value *Op0 = EmitScalarExpr(E->getArg(0));
616:     Value *Op1 = EmitScalarExpr(E->getArg(1));
617:     llvm::Type *V1I128Ty = llvm::FixedVectorType::get(
618:         llvm::IntegerType::get(getLLVMContext(), 128), 1);
619:     Ops.push_back(Builder.CreateBitCast(Op0, V1I128Ty));
620:     Ops.push_back(Builder.CreateBitCast(Op1, V1I128Ty));
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 621-640
```cpp
621:     ID = (BuiltinID == PPC::BI__builtin_altivec_vaddcuq_c)
622:              ? Intrinsic::ppc_altivec_vaddcuq
623:              : Intrinsic::ppc_altivec_vsubcuq;
624:     return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
625:   }
626:   case PPC::BI__builtin_altivec_vaddeuqm_c:
627:   case PPC::BI__builtin_altivec_vaddecuq_c:
628:   case PPC::BI__builtin_altivec_vsubeuqm_c:
629:   case PPC::BI__builtin_altivec_vsubecuq_c: {
630:     SmallVector<Value *, 3> Ops;
631:     Value *Op0 = EmitScalarExpr(E->getArg(0));
632:     Value *Op1 = EmitScalarExpr(E->getArg(1));
633:     Value *Op2 = EmitScalarExpr(E->getArg(2));
634:     llvm::Type *V1I128Ty = llvm::FixedVectorType::get(
635:         llvm::IntegerType::get(getLLVMContext(), 128), 1);
636:     Ops.push_back(Builder.CreateBitCast(Op0, V1I128Ty));
637:     Ops.push_back(Builder.CreateBitCast(Op1, V1I128Ty));
638:     Ops.push_back(Builder.CreateBitCast(Op2, V1I128Ty));
639:     switch (BuiltinID) {
640:     default:
```
- **EN**: This block defines callable entry points like `get`; uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 641-660
```cpp
641:       llvm_unreachable("Unsupported intrinsic!");
642:     case PPC::BI__builtin_altivec_vaddeuqm_c:
643:       ID = Intrinsic::ppc_altivec_vaddeuqm;
644:       break;
645:     case PPC::BI__builtin_altivec_vaddecuq_c:
646:       ID = Intrinsic::ppc_altivec_vaddecuq;
647:       break;
648:     case PPC::BI__builtin_altivec_vsubeuqm_c:
649:       ID = Intrinsic::ppc_altivec_vsubeuqm;
650:       break;
651:     case PPC::BI__builtin_altivec_vsubecuq_c:
652:       ID = Intrinsic::ppc_altivec_vsubecuq;
653:       break;
654:     }
655:     return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
656:   }
657:   case PPC::BI__builtin_ppc_rldimi:
658:   case PPC::BI__builtin_ppc_rlwimi: {
659:     Value *Op0 = EmitScalarExpr(E->getArg(0));
660:     Value *Op1 = EmitScalarExpr(E->getArg(1));
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 661-680
```cpp
661:     Value *Op2 = EmitScalarExpr(E->getArg(2));
662:     Value *Op3 = EmitScalarExpr(E->getArg(3));
663:     // rldimi is 64-bit instruction, expand the intrinsic before isel to
664:     // leverage peephole and avoid legalization efforts.
665:     if (BuiltinID == PPC::BI__builtin_ppc_rldimi &&
666:         !getTarget().getTriple().isPPC64()) {
667:       Function *F = CGM.getIntrinsic(Intrinsic::fshl, Op0->getType());
668:       Op2 = Builder.CreateZExt(Op2, Int64Ty);
669:       Value *Shift = Builder.CreateCall(F, {Op0, Op0, Op2});
670:       return Builder.CreateOr(Builder.CreateAnd(Shift, Op3),
671:                               Builder.CreateAnd(Op1, Builder.CreateNot(Op3)));
672:     }
673:     return Builder.CreateCall(
674:         CGM.getIntrinsic(BuiltinID == PPC::BI__builtin_ppc_rldimi
675:                              ? Intrinsic::ppc_rldimi
676:                              : Intrinsic::ppc_rlwimi),
677:         {Op0, Op1, Op2, Op3});
678:   }
679:   case PPC::BI__builtin_ppc_rlwnm: {
680:     Value *Op0 = EmitScalarExpr(E->getArg(0));
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 681-700
```cpp
681:     Value *Op1 = EmitScalarExpr(E->getArg(1));
682:     Value *Op2 = EmitScalarExpr(E->getArg(2));
683:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_rlwnm),
684:                               {Op0, Op1, Op2});
685:   }
686:   case PPC::BI__builtin_ppc_poppar4:
687:   case PPC::BI__builtin_ppc_poppar8: {
688:     Value *Op0 = EmitScalarExpr(E->getArg(0));
689:     llvm::Type *ArgType = Op0->getType();
690:     Function *F = CGM.getIntrinsic(Intrinsic::ctpop, ArgType);
691:     Value *Tmp = Builder.CreateCall(F, Op0);
692: 
693:     llvm::Type *ResultType = ConvertType(E->getType());
694:     Value *Result = Builder.CreateAnd(Tmp, llvm::ConstantInt::get(ArgType, 1));
695:     if (Result->getType() != ResultType)
696:       Result = Builder.CreateIntCast(Result, ResultType, /*isSigned*/true,
697:                                      "cast");
698:     return Result;
699:   }
700:   case PPC::BI__builtin_ppc_cmpb: {
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 701-720
```cpp
701:     Value *Op0 = EmitScalarExpr(E->getArg(0));
702:     Value *Op1 = EmitScalarExpr(E->getArg(1));
703:     if (getTarget().getTriple().isPPC64()) {
704:       Function *F =
705:           CGM.getIntrinsic(Intrinsic::ppc_cmpb, {Int64Ty, Int64Ty, Int64Ty});
706:       return Builder.CreateCall(F, {Op0, Op1}, "cmpb");
707:     }
708:     // For 32 bit, emit the code as below:
709:     // %conv = trunc i64 %a to i32
710:     // %conv1 = trunc i64 %b to i32
711:     // %shr = lshr i64 %a, 32
712:     // %conv2 = trunc i64 %shr to i32
713:     // %shr3 = lshr i64 %b, 32
714:     // %conv4 = trunc i64 %shr3 to i32
715:     // %0 = tail call i32 @llvm.ppc.cmpb32(i32 %conv, i32 %conv1)
716:     // %conv5 = zext i32 %0 to i64
717:     // %1 = tail call i32 @llvm.ppc.cmpb32(i32 %conv2, i32 %conv4)
718:     // %conv614 = zext i32 %1 to i64
719:     // %shl = shl nuw i64 %conv614, 32
720:     // %or = or i64 %shl, %conv5
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 721-740
```cpp
721:     // ret i64 %or
722:     Function *F =
723:         CGM.getIntrinsic(Intrinsic::ppc_cmpb, {Int32Ty, Int32Ty, Int32Ty});
724:     Value *ArgOneLo = Builder.CreateTrunc(Op0, Int32Ty);
725:     Value *ArgTwoLo = Builder.CreateTrunc(Op1, Int32Ty);
726:     Constant *ShiftAmt = ConstantInt::get(Int64Ty, 32);
727:     Value *ArgOneHi =
728:         Builder.CreateTrunc(Builder.CreateLShr(Op0, ShiftAmt), Int32Ty);
729:     Value *ArgTwoHi =
730:         Builder.CreateTrunc(Builder.CreateLShr(Op1, ShiftAmt), Int32Ty);
731:     Value *ResLo = Builder.CreateZExt(
732:         Builder.CreateCall(F, {ArgOneLo, ArgTwoLo}, "cmpb"), Int64Ty);
733:     Value *ResHiShift = Builder.CreateZExt(
734:         Builder.CreateCall(F, {ArgOneHi, ArgTwoHi}, "cmpb"), Int64Ty);
735:     Value *ResHi = Builder.CreateShl(ResHiShift, ShiftAmt);
736:     return Builder.CreateOr(ResLo, ResHi);
737:   }
738:   // Copy sign
739:   case PPC::BI__builtin_vsx_xvcpsgnsp:
740:   case PPC::BI__builtin_vsx_xvcpsgndp: {
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 741-760
```cpp
741:     llvm::Type *ResultType = ConvertType(E->getType());
742:     Value *X = EmitScalarExpr(E->getArg(0));
743:     Value *Y = EmitScalarExpr(E->getArg(1));
744:     ID = Intrinsic::copysign;
745:     llvm::Function *F = CGM.getIntrinsic(ID, ResultType);
746:     return Builder.CreateCall(F, {X, Y});
747:   }
748:   // Rounding/truncation
749:   case PPC::BI__builtin_vsx_xvrspip:
750:   case PPC::BI__builtin_vsx_xvrdpip:
751:   case PPC::BI__builtin_vsx_xvrdpim:
752:   case PPC::BI__builtin_vsx_xvrspim:
753:   case PPC::BI__builtin_vsx_xvrdpi:
754:   case PPC::BI__builtin_vsx_xvrspi:
755:   case PPC::BI__builtin_vsx_xvrdpic:
756:   case PPC::BI__builtin_vsx_xvrspic:
757:   case PPC::BI__builtin_vsx_xvrdpiz:
758:   case PPC::BI__builtin_vsx_xvrspiz: {
759:     llvm::Type *ResultType = ConvertType(E->getType());
760:     Value *X = EmitScalarExpr(E->getArg(0));
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 761-780
```cpp
761:     if (BuiltinID == PPC::BI__builtin_vsx_xvrdpim ||
762:         BuiltinID == PPC::BI__builtin_vsx_xvrspim)
763:       ID = Builder.getIsFPConstrained()
764:                ? Intrinsic::experimental_constrained_floor
765:                : Intrinsic::floor;
766:     else if (BuiltinID == PPC::BI__builtin_vsx_xvrdpi ||
767:              BuiltinID == PPC::BI__builtin_vsx_xvrspi)
768:       ID = Builder.getIsFPConstrained()
769:                ? Intrinsic::experimental_constrained_round
770:                : Intrinsic::round;
771:     else if (BuiltinID == PPC::BI__builtin_vsx_xvrdpic ||
772:              BuiltinID == PPC::BI__builtin_vsx_xvrspic)
773:       ID = Builder.getIsFPConstrained()
774:                ? Intrinsic::experimental_constrained_rint
775:                : Intrinsic::rint;
776:     else if (BuiltinID == PPC::BI__builtin_vsx_xvrdpip ||
777:              BuiltinID == PPC::BI__builtin_vsx_xvrspip)
778:       ID = Builder.getIsFPConstrained()
779:                ? Intrinsic::experimental_constrained_ceil
780:                : Intrinsic::ceil;
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 781-800
```cpp
781:     else if (BuiltinID == PPC::BI__builtin_vsx_xvrdpiz ||
782:              BuiltinID == PPC::BI__builtin_vsx_xvrspiz)
783:       ID = Builder.getIsFPConstrained()
784:                ? Intrinsic::experimental_constrained_trunc
785:                : Intrinsic::trunc;
786:     llvm::Function *F = CGM.getIntrinsic(ID, ResultType);
787:     return Builder.getIsFPConstrained() ? Builder.CreateConstrainedFPCall(F, X)
788:                                         : Builder.CreateCall(F, X);
789:   }
790: 
791:   // Absolute value
792:   case PPC::BI__builtin_vsx_xvabsdp:
793:   case PPC::BI__builtin_vsx_xvabssp: {
794:     Value *X = EmitScalarExpr(E->getArg(0));
795:     return Builder.CreateFAbs(X);
796:   }
797: 
798:   // Fastmath by default
799:   case PPC::BI__builtin_ppc_recipdivf:
800:   case PPC::BI__builtin_ppc_recipdivd:
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 801-820
```cpp
801:   case PPC::BI__builtin_ppc_rsqrtf:
802:   case PPC::BI__builtin_ppc_rsqrtd: {
803:     FastMathFlags FMF = Builder.getFastMathFlags();
804:     Builder.getFastMathFlags().setFast();
805:     llvm::Type *ResultType = ConvertType(E->getType());
806:     Value *X = EmitScalarExpr(E->getArg(0));
807: 
808:     if (BuiltinID == PPC::BI__builtin_ppc_recipdivf ||
809:         BuiltinID == PPC::BI__builtin_ppc_recipdivd) {
810:       Value *Y = EmitScalarExpr(E->getArg(1));
811:       Value *FDiv = Builder.CreateFDiv(X, Y, "recipdiv");
812:       Builder.getFastMathFlags() &= (FMF);
813:       return FDiv;
814:     }
815:     auto *One = ConstantFP::get(ResultType, 1.0);
816:     llvm::Function *F = CGM.getIntrinsic(Intrinsic::sqrt, ResultType);
817:     Value *FDiv = Builder.CreateFDiv(One, Builder.CreateCall(F, X), "rsqrt");
818:     Builder.getFastMathFlags() &= (FMF);
819:     return FDiv;
820:   }
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 821-840
```cpp
821:   case PPC::BI__builtin_ppc_alignx: {
822:     Value *Op0 = EmitScalarExpr(E->getArg(0));
823:     Value *Op1 = EmitScalarExpr(E->getArg(1));
824:     ConstantInt *AlignmentCI = cast<ConstantInt>(Op0);
825:     if (AlignmentCI->getValue().ugt(llvm::Value::MaximumAlignment))
826:       AlignmentCI = ConstantInt::get(AlignmentCI->getIntegerType(),
827:                                      llvm::Value::MaximumAlignment);
828: 
829:     emitAlignmentAssumption(Op1, E->getArg(1),
830:                             /*The expr loc is sufficient.*/ SourceLocation(),
831:                             AlignmentCI, nullptr);
832:     return Op1;
833:   }
834:   case PPC::BI__builtin_ppc_rdlam: {
835:     Value *Op0 = EmitScalarExpr(E->getArg(0));
836:     Value *Op1 = EmitScalarExpr(E->getArg(1));
837:     Value *Op2 = EmitScalarExpr(E->getArg(2));
838:     llvm::Type *Ty = Op0->getType();
839:     Value *ShiftAmt = Builder.CreateIntCast(Op1, Ty, false);
840:     Function *F = CGM.getIntrinsic(Intrinsic::fshl, Ty);
```
- **EN**: This block defines callable entry points like `emitAlignmentAssumption`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitAlignmentAssumption`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 841-860
```cpp
841:     Value *Rotate = Builder.CreateCall(F, {Op0, Op0, ShiftAmt});
842:     return Builder.CreateAnd(Rotate, Op2);
843:   }
844:   case PPC::BI__builtin_ppc_load2r: {
845:     Function *F = CGM.getIntrinsic(Intrinsic::ppc_load2r);
846:     Value *Op0 = EmitScalarExpr(E->getArg(0));
847:     Value *LoadIntrinsic = Builder.CreateCall(F, {Op0});
848:     return Builder.CreateTrunc(LoadIntrinsic, Int16Ty);
849:   }
850:   // FMA variations
851:   case PPC::BI__builtin_ppc_fnmsub:
852:   case PPC::BI__builtin_ppc_fnmsubs:
853:   case PPC::BI__builtin_vsx_xvmaddadp:
854:   case PPC::BI__builtin_vsx_xvmaddasp:
855:   case PPC::BI__builtin_vsx_xvnmaddadp:
856:   case PPC::BI__builtin_vsx_xvnmaddasp:
857:   case PPC::BI__builtin_vsx_xvmsubadp:
858:   case PPC::BI__builtin_vsx_xvmsubasp:
859:   case PPC::BI__builtin_vsx_xvnmsubadp:
860:   case PPC::BI__builtin_vsx_xvnmsubasp: {
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 861-880
```cpp
861:     llvm::Type *ResultType = ConvertType(E->getType());
862:     Value *X = EmitScalarExpr(E->getArg(0));
863:     Value *Y = EmitScalarExpr(E->getArg(1));
864:     Value *Z = EmitScalarExpr(E->getArg(2));
865:     llvm::Function *F;
866:     if (Builder.getIsFPConstrained())
867:       F = CGM.getIntrinsic(Intrinsic::experimental_constrained_fma, ResultType);
868:     else
869:       F = CGM.getIntrinsic(Intrinsic::fma, ResultType);
870:     switch (BuiltinID) {
871:       case PPC::BI__builtin_vsx_xvmaddadp:
872:       case PPC::BI__builtin_vsx_xvmaddasp:
873:         if (Builder.getIsFPConstrained())
874:           return Builder.CreateConstrainedFPCall(F, {X, Y, Z});
875:         else
876:           return Builder.CreateCall(F, {X, Y, Z});
877:       case PPC::BI__builtin_vsx_xvnmaddadp:
878:       case PPC::BI__builtin_vsx_xvnmaddasp:
879:         if (Builder.getIsFPConstrained())
880:           return Builder.CreateFNeg(
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 881-900
```cpp
881:               Builder.CreateConstrainedFPCall(F, {X, Y, Z}), "neg");
882:         else
883:           return Builder.CreateFNeg(Builder.CreateCall(F, {X, Y, Z}), "neg");
884:       case PPC::BI__builtin_vsx_xvmsubadp:
885:       case PPC::BI__builtin_vsx_xvmsubasp:
886:         if (Builder.getIsFPConstrained())
887:           return Builder.CreateConstrainedFPCall(
888:               F, {X, Y, Builder.CreateFNeg(Z, "neg")});
889:         else
890:           return Builder.CreateCall(F, {X, Y, Builder.CreateFNeg(Z, "neg")});
891:       case PPC::BI__builtin_ppc_fnmsub:
892:       case PPC::BI__builtin_ppc_fnmsubs:
893:       case PPC::BI__builtin_vsx_xvnmsubadp:
894:       case PPC::BI__builtin_vsx_xvnmsubasp:
895:         if (Builder.getIsFPConstrained())
896:           return Builder.CreateFNeg(
897:               Builder.CreateConstrainedFPCall(
898:                   F, {X, Y, Builder.CreateFNeg(Z, "neg")}),
899:               "neg");
900:         else
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 901-920
```cpp
901:           return Builder.CreateCall(
902:               CGM.getIntrinsic(Intrinsic::ppc_fnmsub, ResultType), {X, Y, Z});
903:       }
904:     llvm_unreachable("Unknown FMA operation");
905:     return nullptr; // Suppress no-return warning
906:   }
907: 
908:   case PPC::BI__builtin_vsx_insertword: {
909:     Value *Op0 = EmitScalarExpr(E->getArg(0));
910:     Value *Op1 = EmitScalarExpr(E->getArg(1));
911:     Value *Op2 = EmitScalarExpr(E->getArg(2));
912:     llvm::Function *F = CGM.getIntrinsic(Intrinsic::ppc_vsx_xxinsertw);
913: 
914:     // Third argument is a compile time constant int. It must be clamped to
915:     // to the range [0, 12].
916:     ConstantInt *ArgCI = dyn_cast<ConstantInt>(Op2);
917:     assert(ArgCI &&
918:            "Third arg to xxinsertw intrinsic must be constant integer");
919:     const int64_t MaxIndex = 12;
920:     int64_t Index = std::clamp(ArgCI->getSExtValue(), (int64_t)0, MaxIndex);
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 921-940
```cpp
921: 
922:     // The builtin semantics don't exactly match the xxinsertw instructions
923:     // semantics (which ppc_vsx_xxinsertw follows). The builtin extracts the
924:     // word from the first argument, and inserts it in the second argument. The
925:     // instruction extracts the word from its second input register and inserts
926:     // it into its first input register, so swap the first and second arguments.
927:     std::swap(Op0, Op1);
928: 
929:     // Need to cast the second argument from a vector of unsigned int to a
930:     // vector of long long.
931:     Op1 = Builder.CreateBitCast(Op1, llvm::FixedVectorType::get(Int64Ty, 2));
932: 
933:     if (getTarget().isLittleEndian()) {
934:       // Reverse the double words in the vector we will extract from.
935:       Op0 = Builder.CreateBitCast(Op0, llvm::FixedVectorType::get(Int64Ty, 2));
936:       Op0 = Builder.CreateShuffleVector(Op0, Op0, {1, 0});
937: 
938:       // Reverse the index.
939:       Index = MaxIndex - Index;
940:     }
```
- **EN**: This block defines callable entry points like `swap`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `swap`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 941-960
```cpp
941: 
942:     // Intrinsic expects the first arg to be a vector of int.
943:     Op0 = Builder.CreateBitCast(Op0, llvm::FixedVectorType::get(Int32Ty, 4));
944:     Op2 = ConstantInt::getSigned(Int32Ty, Index);
945:     return Builder.CreateCall(F, {Op0, Op1, Op2});
946:   }
947: 
948:   case PPC::BI__builtin_vsx_extractuword: {
949:     Value *Op0 = EmitScalarExpr(E->getArg(0));
950:     Value *Op1 = EmitScalarExpr(E->getArg(1));
951:     llvm::Function *F = CGM.getIntrinsic(Intrinsic::ppc_vsx_xxextractuw);
952: 
953:     // Intrinsic expects the first argument to be a vector of doublewords.
954:     Op0 = Builder.CreateBitCast(Op0, llvm::FixedVectorType::get(Int64Ty, 2));
955: 
956:     // The second argument is a compile time constant int that needs to
957:     // be clamped to the range [0, 12].
958:     ConstantInt *ArgCI = dyn_cast<ConstantInt>(Op1);
959:     assert(ArgCI &&
960:            "Second Arg to xxextractuw intrinsic must be a constant integer!");
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 961-980
```cpp
961:     const int64_t MaxIndex = 12;
962:     int64_t Index = std::clamp(ArgCI->getSExtValue(), (int64_t)0, MaxIndex);
963: 
964:     if (getTarget().isLittleEndian()) {
965:       // Reverse the index.
966:       Index = MaxIndex - Index;
967:       Op1 = ConstantInt::getSigned(Int32Ty, Index);
968: 
969:       // Emit the call, then reverse the double words of the results vector.
970:       Value *Call = Builder.CreateCall(F, {Op0, Op1});
971: 
972:       Value *ShuffleCall =
973:           Builder.CreateShuffleVector(Call, Call, {1, 0});
974:       return ShuffleCall;
975:     } else {
976:       Op1 = ConstantInt::getSigned(Int32Ty, Index);
977:       return Builder.CreateCall(F, {Op0, Op1});
978:     }
979:   }
980: 
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 981-1000
```cpp
 981:   case PPC::BI__builtin_vsx_xxpermdi: {
 982:     Value *Op0 = EmitScalarExpr(E->getArg(0));
 983:     Value *Op1 = EmitScalarExpr(E->getArg(1));
 984:     Value *Op2 = EmitScalarExpr(E->getArg(2));
 985:     ConstantInt *ArgCI = dyn_cast<ConstantInt>(Op2);
 986:     assert(ArgCI && "Third arg must be constant integer!");
 987: 
 988:     unsigned Index = ArgCI->getZExtValue();
 989:     Op0 = Builder.CreateBitCast(Op0, llvm::FixedVectorType::get(Int64Ty, 2));
 990:     Op1 = Builder.CreateBitCast(Op1, llvm::FixedVectorType::get(Int64Ty, 2));
 991: 
 992:     // Account for endianness by treating this as just a shuffle. So we use the
 993:     // same indices for both LE and BE in order to produce expected results in
 994:     // both cases.
 995:     int ElemIdx0 = (Index & 2) >> 1;
 996:     int ElemIdx1 = 2 + (Index & 1);
 997: 
 998:     int ShuffleElts[2] = {ElemIdx0, ElemIdx1};
 999:     Value *ShuffleCall = Builder.CreateShuffleVector(Op0, Op1, ShuffleElts);
1000:     QualType BIRetType = E->getType();
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1001-1020
```cpp
1001:     auto RetTy = ConvertType(BIRetType);
1002:     return Builder.CreateBitCast(ShuffleCall, RetTy);
1003:   }
1004: 
1005:   case PPC::BI__builtin_vsx_xxsldwi: {
1006:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1007:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1008:     Value *Op2 = EmitScalarExpr(E->getArg(2));
1009:     ConstantInt *ArgCI = dyn_cast<ConstantInt>(Op2);
1010:     assert(ArgCI && "Third argument must be a compile time constant");
1011:     unsigned Index = ArgCI->getZExtValue() & 0x3;
1012:     Op0 = Builder.CreateBitCast(Op0, llvm::FixedVectorType::get(Int32Ty, 4));
1013:     Op1 = Builder.CreateBitCast(Op1, llvm::FixedVectorType::get(Int32Ty, 4));
1014: 
1015:     // Create a shuffle mask
1016:     int ElemIdx0;
1017:     int ElemIdx1;
1018:     int ElemIdx2;
1019:     int ElemIdx3;
1020:     if (getTarget().isLittleEndian()) {
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1021-1040
```cpp
1021:       // Little endian element N comes from element 8+N-Index of the
1022:       // concatenated wide vector (of course, using modulo arithmetic on
1023:       // the total number of elements).
1024:       ElemIdx0 = (8 - Index) % 8;
1025:       ElemIdx1 = (9 - Index) % 8;
1026:       ElemIdx2 = (10 - Index) % 8;
1027:       ElemIdx3 = (11 - Index) % 8;
1028:     } else {
1029:       // Big endian ElemIdx<N> = Index + N
1030:       ElemIdx0 = Index;
1031:       ElemIdx1 = Index + 1;
1032:       ElemIdx2 = Index + 2;
1033:       ElemIdx3 = Index + 3;
1034:     }
1035: 
1036:     int ShuffleElts[4] = {ElemIdx0, ElemIdx1, ElemIdx2, ElemIdx3};
1037:     Value *ShuffleCall = Builder.CreateShuffleVector(Op0, Op1, ShuffleElts);
1038:     QualType BIRetType = E->getType();
1039:     auto RetTy = ConvertType(BIRetType);
1040:     return Builder.CreateBitCast(ShuffleCall, RetTy);
```
- **EN**: This block documents intent or context for the surrounding target builtin lowering code.
- **CN**: 该代码块说明周围 目标内建函数降级 代码的意图或上下文。

### Lines 1041-1060
```cpp
1041:   }
1042: 
1043:   case PPC::BI__builtin_pack_vector_int128: {
1044:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1045:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1046:     bool isLittleEndian = getTarget().isLittleEndian();
1047:     Value *PoisonValue =
1048:         llvm::PoisonValue::get(llvm::FixedVectorType::get(Op0->getType(), 2));
1049:     Value *Res = Builder.CreateInsertElement(
1050:         PoisonValue, Op0, (uint64_t)(isLittleEndian ? 1 : 0));
1051:     Res = Builder.CreateInsertElement(Res, Op1,
1052:                                       (uint64_t)(isLittleEndian ? 0 : 1));
1053:     return Builder.CreateBitCast(Res, ConvertType(E->getType()));
1054:   }
1055: 
1056:   case PPC::BI__builtin_unpack_vector_int128: {
1057:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1058:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1059:     ConstantInt *Index = cast<ConstantInt>(Op1);
1060:     Value *Unpacked = Builder.CreateBitCast(
```
- **EN**: This block defines callable entry points like `get`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1061-1080
```cpp
1061:         Op0, llvm::FixedVectorType::get(ConvertType(E->getType()), 2));
1062: 
1063:     if (getTarget().isLittleEndian())
1064:       Index =
1065:           ConstantInt::get(Index->getIntegerType(), 1 - Index->getZExtValue());
1066: 
1067:     return Builder.CreateExtractElement(Unpacked, Index);
1068:   }
1069: 
1070:   case PPC::BI__builtin_ppc_sthcx: {
1071:     llvm::Function *F = CGM.getIntrinsic(Intrinsic::ppc_sthcx);
1072:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1073:     Value *Op1 = Builder.CreateSExt(EmitScalarExpr(E->getArg(1)), Int32Ty);
1074:     return Builder.CreateCall(F, {Op0, Op1});
1075:   }
1076: 
1077:   // The PPC MMA builtins take a pointer to a __vector_quad as an argument.
1078:   // Some of the MMA instructions accumulate their result into an existing
1079:   // accumulator whereas the others generate a new accumulator. So we need to
1080:   // use custom code generation to expand a builtin call with a pointer to a
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 1081-1100
```cpp
1081:   // load (if the corresponding instruction accumulates its result) followed by
1082:   // the call to the intrinsic and a store of the result.
1083: #define CUSTOM_BUILTIN(Name, Intr, Types, Accumulate, Feature) \
1084:   case PPC::BI__builtin_##Name:
1085: #include "clang/Basic/BuiltinsPPC.def"
1086:   {
1087:     SmallVector<Value *, 4> Ops;
1088:     for (unsigned i = 0, e = E->getNumArgs(); i != e; i++)
1089:       if (E->getArg(i)->getType()->isArrayType())
1090:         Ops.push_back(
1091:             EmitArrayToPointerDecay(E->getArg(i)).emitRawPointer(*this));
1092:       else
1093:         Ops.push_back(EmitScalarExpr(E->getArg(i)));
1094:     // The first argument of these two builtins is a pointer used to store their
1095:     // result. However, the llvm intrinsics return their result in multiple
1096:     // return values. So, here we emit code extracting these values from the
1097:     // intrinsic results and storing them using that pointer.
1098:     if (BuiltinID == PPC::BI__builtin_mma_disassemble_acc ||
1099:         BuiltinID == PPC::BI__builtin_vsx_disassemble_pair ||
1100:         BuiltinID == PPC::BI__builtin_mma_disassemble_pair) {
```
- **EN**: This block imports Clang headers `clang/Basic/BuiltinsPPC.def`; uses control flow (if, for, case) to specialize target builtin lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/Basic/BuiltinsPPC.def`；通过控制流（if, for, case）细化 目标内建函数降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 1101-1120
```cpp
1101:       unsigned NumVecs = 2;
1102:       auto Intrinsic = Intrinsic::ppc_vsx_disassemble_pair;
1103:       if (BuiltinID == PPC::BI__builtin_mma_disassemble_acc) {
1104:         NumVecs = 4;
1105:         Intrinsic = Intrinsic::ppc_mma_disassemble_acc;
1106:       }
1107:       llvm::Function *F = CGM.getIntrinsic(Intrinsic);
1108:       Address Addr = EmitPointerWithAlignment(E->getArg(1));
1109:       Value *Vec = Builder.CreateLoad(Addr);
1110:       Value *Call = Builder.CreateCall(F, {Vec});
1111:       llvm::Type *VTy = llvm::FixedVectorType::get(Int8Ty, 16);
1112:       Value *Ptr = Ops[0];
1113:       for (unsigned i=0; i<NumVecs; i++) {
1114:         Value *Vec = Builder.CreateExtractValue(Call, i);
1115:         llvm::ConstantInt* Index = llvm::ConstantInt::get(IntTy, i);
1116:         Value *GEP = Builder.CreateInBoundsGEP(VTy, Ptr, Index);
1117:         Builder.CreateAlignedStore(Vec, GEP, MaybeAlign(16));
1118:       }
1119:       return Call;
1120:     }
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 1121-1140
```cpp
1121:     if (BuiltinID == PPC::BI__builtin_vsx_build_pair ||
1122:         BuiltinID == PPC::BI__builtin_mma_build_acc) {
1123:       // Reverse the order of the operands for LE, so the
1124:       // same builtin call can be used on both LE and BE
1125:       // without the need for the programmer to swap operands.
1126:       // The operands are reversed starting from the second argument,
1127:       // the first operand is the pointer to the pair/accumulator
1128:       // that is being built.
1129:       if (getTarget().isLittleEndian())
1130:         std::reverse(Ops.begin() + 1, Ops.end());
1131:     }
1132:     bool Accumulate;
1133:     switch (BuiltinID) {
1134:   #define CUSTOM_BUILTIN(Name, Intr, Types, Acc, Feature) \
1135:     case PPC::BI__builtin_##Name: \
1136:       ID = Intrinsic::ppc_##Intr; \
1137:       Accumulate = Acc; \
1138:       break;
1139:   #include "clang/Basic/BuiltinsPPC.def"
1140:     }
```
- **EN**: This block imports Clang headers `clang/Basic/BuiltinsPPC.def`; uses control flow (if, switch, case) to specialize target builtin lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/Basic/BuiltinsPPC.def`；通过控制流（if, switch, case）细化 目标内建函数降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 1141-1160
```cpp
1141:     // Handle custom builtins that return early without using the common
1142:     // store-back pattern.
1143:     switch (BuiltinID) {
1144:     case PPC::BI__builtin_vsx_lxvp:
1145:     case PPC::BI__builtin_mma_lxvp:
1146:       Ops[0] = Builder.CreateGEP(Int8Ty, Ops[1], Ops[0]);
1147:       Ops.pop_back();
1148:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1149:     case PPC::BI__builtin_vsx_stxvp:
1150:     case PPC::BI__builtin_mma_stxvp:
1151:       Ops[1] = Builder.CreateGEP(Int8Ty, Ops[2], Ops[1]);
1152:       Ops.pop_back();
1153:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1154:     case PPC::BI__builtin_disassemble_dmr:
1155:       Ops[1] = Builder.CreateLoad(EmitPointerWithAlignment(E->getArg(1)));
1156:       return Builder.CreateAlignedStore(Ops[1], Ops[0], MaybeAlign());
1157: 
1158:     // Handle AES encrypt paired builtins - they return a value directly.
1159:     // For variant builtins, add the appropriate immediate value.
1160:     case PPC::BI__builtin_aes128_encrypt_paired:
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 1161-1180
```cpp
1161:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, 0));
1162:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1163:     case PPC::BI__builtin_aes192_encrypt_paired:
1164:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, 1));
1165:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1166:     case PPC::BI__builtin_aes256_encrypt_paired:
1167:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, 2));
1168:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1169:     case PPC::BI__builtin_aes_encrypt_paired:
1170:       // For base builtin, Ops already has all 3 arguments.
1171:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1172: 
1173:     // Handle AES decrypt paired builtins - they return a value directly.
1174:     // For variant builtins, add the appropriate immediate value.
1175:     case PPC::BI__builtin_aes128_decrypt_paired:
1176:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, 0));
1177:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1178:     case PPC::BI__builtin_aes192_decrypt_paired:
1179:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, 1));
1180:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1181-1200
```cpp
1181:     case PPC::BI__builtin_aes256_decrypt_paired:
1182:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, 2));
1183:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1184:     case PPC::BI__builtin_aes_decrypt_paired:
1185:       // For base builtin, Ops already has all 3 arguments.
1186:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1187: 
1188:     // Handle AES genlastkey paired builtins.
1189:     case PPC::BI__builtin_aes128_genlastkey_paired:
1190:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, 0));
1191:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1192:     case PPC::BI__builtin_aes192_genlastkey_paired:
1193:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, 1));
1194:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1195:     case PPC::BI__builtin_aes256_genlastkey_paired:
1196:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, 2));
1197:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1198:     case PPC::BI__builtin_aes_genlastkey_paired:
1199:       // For base builtin, Ops already has all 2 arguments.
1200:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1201-1220
```cpp
1201: 
1202:     // Handle Galois Field multiplication builtins.
1203:     case PPC::BI__builtin_galois_field_mult_gcm:
1204:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, 0));
1205:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1206:     case PPC::BI__builtin_galois_field_mult_xts:
1207:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, 1));
1208:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1209:     case PPC::BI__builtin_galois_field_mult:
1210:       // For base builtin, Ops already has all 3 arguments.
1211:       return Builder.CreateCall(CGM.getIntrinsic(ID), Ops, "");
1212: 
1213:     default:
1214:       break;
1215:     }
1216: 
1217:     SmallVector<Value*, 4> CallOps;
1218: 
1219:     // Accumulate = true, are used for builtins where the hardware instruction
1220:     // reads the old destination value, performs an operation with it, and
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1221-1240
```cpp
1221:     // writes the result back.
1222:     // Load the existing value from the first argument (destination pointer) and
1223:     // add it to CallOps as the first intrinsic operand.
1224:     if (Accumulate) {
1225:       Address Addr = EmitPointerWithAlignment(E->getArg(0));
1226:       Value *Acc = Builder.CreateLoad(Addr);
1227:       CallOps.push_back(Acc);
1228:     }
1229: 
1230:     // Handles builtins that need special argument handling such as:
1231:     // - Dereferencing pointer arguments to load actual register values.
1232:     // - Adding implicit operands required by the intrinsic.
1233:     // - Transforming or reordering operands.
1234:     // After preprocessing, the loop at end copies Ops[1..n] into CallOps,
1235:     // skipping Ops[0] which is the destination pointer for result storage.
1236:     switch (BuiltinID) {
1237:     case PPC::BI__builtin_dmmr:
1238:     case PPC::BI__builtin_dmxor:
1239:     case PPC::BI__builtin_dmsha2hash: {
1240:       Address Addr = EmitPointerWithAlignment(E->getArg(1));
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 1241-1260
```cpp
1241:       Ops[1] = Builder.CreateLoad(Addr);
1242:       break;
1243:     }
1244:     case PPC::BI__builtin_dmsha256hash:
1245:     case PPC::BI__builtin_dmsha512hash: {
1246:       Ops[1] = Builder.CreateLoad(EmitPointerWithAlignment(E->getArg(1)));
1247:       int Imm = (BuiltinID == PPC::BI__builtin_dmsha256hash) ? 0 : 1;
1248:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, Imm));
1249:       break;
1250:     }
1251:     case PPC::BI__builtin_dmsha3dw:
1252:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, 0));
1253:       break;
1254:     case PPC::BI__builtin_dmcryshash:
1255:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, 12));
1256:       break;
1257:     case PPC::BI__builtin_dmxxsha384512pad:
1258:     case PPC::BI__builtin_dmxxsha224256pad: {
1259:       int Imm = (BuiltinID == PPC::BI__builtin_dmxxsha384512pad) ? 2 : 3;
1260:       Ops.push_back(ConstantInt::get(Int32Ty, Imm));
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1261-1280
```cpp
1261:       Ops.push_back(ConstantInt::get(Int32Ty, 0));
1262:       Ops.push_back(ConstantInt::get(Int32Ty, 0));
1263:       break;
1264:     }
1265:     case PPC::BI__builtin_dmxxsha3512pad:
1266:     case PPC::BI__builtin_dmxxsha3384pad:
1267:     case PPC::BI__builtin_dmxxsha3256pad:
1268:     case PPC::BI__builtin_dmxxsha3224pad:
1269:     case PPC::BI__builtin_dmxxshake256pad:
1270:     case PPC::BI__builtin_dmxxshake128pad: {
1271:       Value *E_val = Ops[2];
1272:       int ID, BL;
1273:       switch (BuiltinID) {
1274:       case PPC::BI__builtin_dmxxsha3512pad:
1275:         ID = 0;
1276:         BL = 0;
1277:         break;
1278:       case PPC::BI__builtin_dmxxsha3384pad:
1279:         ID = 0;
1280:         BL = 1;
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 1281-1300
```cpp
1281:         break;
1282:       case PPC::BI__builtin_dmxxsha3256pad:
1283:         ID = 0;
1284:         BL = 2;
1285:         break;
1286:       case PPC::BI__builtin_dmxxsha3224pad:
1287:         ID = 0;
1288:         BL = 3;
1289:         break;
1290:       case PPC::BI__builtin_dmxxshake256pad:
1291:         ID = 1;
1292:         BL = 0;
1293:         break;
1294:       case PPC::BI__builtin_dmxxshake128pad:
1295:         ID = 1;
1296:         BL = 1;
1297:         break;
1298:       }
1299:       Ops[2] = ConstantInt::get(Int32Ty, ID);
1300:       Ops.push_back(E_val);
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1301-1320
```cpp
1301:       Ops.push_back(ConstantInt::get(Int32Ty, BL));
1302:       break;
1303:     }
1304:     }
1305:     for (unsigned i=1; i<Ops.size(); i++)
1306:       CallOps.push_back(Ops[i]);
1307:     llvm::Function *F = CGM.getIntrinsic(ID);
1308:     Value *Call = Builder.CreateCall(F, CallOps);
1309:     return Builder.CreateAlignedStore(Call, Ops[0], MaybeAlign());
1310:   }
1311: 
1312:   case PPC::BI__builtin_ppc_compare_and_swap:
1313:   case PPC::BI__builtin_ppc_compare_and_swaplp: {
1314:     Address Addr = EmitPointerWithAlignment(E->getArg(0));
1315:     Address OldValAddr = EmitPointerWithAlignment(E->getArg(1));
1316:     Value *OldVal = Builder.CreateLoad(OldValAddr);
1317:     QualType AtomicTy = E->getArg(0)->getType()->getPointeeType();
1318:     LValue LV = MakeAddrLValue(Addr, AtomicTy);
1319:     Value *Op2 = EmitScalarExpr(E->getArg(2));
1320:     auto Pair = EmitAtomicCompareExchange(
```
- **EN**: This block uses control flow (for, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（for, case）细化 目标内建函数降级 行为。

### Lines 1321-1340
```cpp
1321:         LV, RValue::get(OldVal), RValue::get(Op2), E->getExprLoc(),
1322:         llvm::AtomicOrdering::Monotonic, llvm::AtomicOrdering::Monotonic, true);
1323:     // Unlike c11's atomic_compare_exchange, according to
1324:     // https://www.ibm.com/docs/en/xl-c-and-cpp-aix/16.1?topic=functions-compare-swap-compare-swaplp
1325:     // > In either case, the contents of the memory location specified by addr
1326:     // > are copied into the memory location specified by old_val_addr.
1327:     // But it hasn't specified storing to OldValAddr is atomic or not and
1328:     // which order to use. Now following XL's codegen, treat it as a normal
1329:     // store.
1330:     Value *LoadedVal = Pair.first.getScalarVal();
1331:     Builder.CreateStore(LoadedVal, OldValAddr);
1332:     return Builder.CreateZExt(Pair.second, Builder.getInt32Ty());
1333:   }
1334:   case PPC::BI__builtin_ppc_fetch_and_add:
1335:   case PPC::BI__builtin_ppc_fetch_and_addlp: {
1336:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Add, E,
1337:                                  llvm::AtomicOrdering::Monotonic);
1338:   }
1339:   case PPC::BI__builtin_ppc_fetch_and_and:
1340:   case PPC::BI__builtin_ppc_fetch_and_andlp: {
```
- **EN**: This block defines callable entry points like `get`, `MakeBinaryAtomicValue`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`, `MakeBinaryAtomicValue`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1341-1360
```cpp
1341:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::And, E,
1342:                                  llvm::AtomicOrdering::Monotonic);
1343:   }
1344: 
1345:   case PPC::BI__builtin_ppc_fetch_and_or:
1346:   case PPC::BI__builtin_ppc_fetch_and_orlp: {
1347:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Or, E,
1348:                                  llvm::AtomicOrdering::Monotonic);
1349:   }
1350:   case PPC::BI__builtin_ppc_fetch_and_swap:
1351:   case PPC::BI__builtin_ppc_fetch_and_swaplp: {
1352:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Xchg, E,
1353:                                  llvm::AtomicOrdering::Monotonic);
1354:   }
1355:   case PPC::BI__builtin_ppc_ldarx:
1356:   case PPC::BI__builtin_ppc_lwarx:
1357:   case PPC::BI__builtin_ppc_lharx:
1358:   case PPC::BI__builtin_ppc_lbarx:
1359:     return emitPPCLoadReserveIntrinsic(*this, BuiltinID, E);
1360:   case PPC::BI__builtin_ppc_mfspr: {
```
- **EN**: This block defines callable entry points like `MakeBinaryAtomicValue`, `emitPPCLoadReserveIntrinsic`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `MakeBinaryAtomicValue`, `emitPPCLoadReserveIntrinsic`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1361-1380
```cpp
1361:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1362:     llvm::Type *RetType = CGM.getDataLayout().getTypeSizeInBits(VoidPtrTy) == 32
1363:                               ? Int32Ty
1364:                               : Int64Ty;
1365:     Function *F = CGM.getIntrinsic(Intrinsic::ppc_mfspr, RetType);
1366:     return Builder.CreateCall(F, {Op0});
1367:   }
1368:   case PPC::BI__builtin_ppc_mtspr: {
1369:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1370:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1371:     llvm::Type *RetType = CGM.getDataLayout().getTypeSizeInBits(VoidPtrTy) == 32
1372:                               ? Int32Ty
1373:                               : Int64Ty;
1374:     Function *F = CGM.getIntrinsic(Intrinsic::ppc_mtspr, RetType);
1375:     return Builder.CreateCall(F, {Op0, Op1});
1376:   }
1377:   case PPC::BI__builtin_ppc_popcntb: {
1378:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
1379:     llvm::Type *ArgType = ArgValue->getType();
1380:     Function *F = CGM.getIntrinsic(Intrinsic::ppc_popcntb, {ArgType, ArgType});
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1381-1400
```cpp
1381:     return Builder.CreateCall(F, {ArgValue}, "popcntb");
1382:   }
1383:   case PPC::BI__builtin_ppc_mtfsf: {
1384:     // The builtin takes a uint32 that needs to be cast to an
1385:     // f64 to be passed to the intrinsic.
1386:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1387:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1388:     Value *Cast = Builder.CreateUIToFP(Op1, DoubleTy);
1389:     llvm::Function *F = CGM.getIntrinsic(Intrinsic::ppc_mtfsf);
1390:     return Builder.CreateCall(F, {Op0, Cast}, "");
1391:   }
1392: 
1393:   case PPC::BI__builtin_ppc_swdiv_nochk:
1394:   case PPC::BI__builtin_ppc_swdivs_nochk: {
1395:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1396:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1397:     FastMathFlags FMF = Builder.getFastMathFlags();
1398:     Builder.getFastMathFlags().setFast();
1399:     Value *FDiv = Builder.CreateFDiv(Op0, Op1, "swdiv_nochk");
1400:     Builder.getFastMathFlags() &= (FMF);
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1401-1420
```cpp
1401:     return FDiv;
1402:   }
1403:   case PPC::BI__builtin_ppc_fric:
1404:     return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(
1405:                            *this, E, Intrinsic::rint,
1406:                            Intrinsic::experimental_constrained_rint))
1407:         .getScalarVal();
1408:   case PPC::BI__builtin_ppc_frim:
1409:   case PPC::BI__builtin_ppc_frims:
1410:     return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(
1411:                            *this, E, Intrinsic::floor,
1412:                            Intrinsic::experimental_constrained_floor))
1413:         .getScalarVal();
1414:   case PPC::BI__builtin_ppc_frin:
1415:   case PPC::BI__builtin_ppc_frins:
1416:     return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(
1417:                            *this, E, Intrinsic::round,
1418:                            Intrinsic::experimental_constrained_round))
1419:         .getScalarVal();
1420:   case PPC::BI__builtin_ppc_frip:
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1421-1440
```cpp
1421:   case PPC::BI__builtin_ppc_frips:
1422:     return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(
1423:                            *this, E, Intrinsic::ceil,
1424:                            Intrinsic::experimental_constrained_ceil))
1425:         .getScalarVal();
1426:   case PPC::BI__builtin_ppc_friz:
1427:   case PPC::BI__builtin_ppc_frizs:
1428:     return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(
1429:                            *this, E, Intrinsic::trunc,
1430:                            Intrinsic::experimental_constrained_trunc))
1431:         .getScalarVal();
1432:   case PPC::BI__builtin_ppc_fsqrt:
1433:   case PPC::BI__builtin_ppc_fsqrts:
1434:     return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(
1435:                            *this, E, Intrinsic::sqrt,
1436:                            Intrinsic::experimental_constrained_sqrt))
1437:         .getScalarVal();
1438:   case PPC::BI__builtin_ppc_test_data_class: {
1439:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1440:     Value *Op1 = EmitScalarExpr(E->getArg(1));
```
- **EN**: This block defines callable entry points like `get`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1441-1460
```cpp
1441:     return Builder.CreateCall(
1442:         CGM.getIntrinsic(Intrinsic::ppc_test_data_class, Op0->getType()),
1443:         {Op0, Op1}, "test_data_class");
1444:   }
1445:   case PPC::BI__builtin_ppc_maxfe: {
1446:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1447:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1448:     Value *Op2 = EmitScalarExpr(E->getArg(2));
1449:     Value *Op3 = EmitScalarExpr(E->getArg(3));
1450:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_maxfe),
1451:                               {Op0, Op1, Op2, Op3});
1452:   }
1453:   case PPC::BI__builtin_ppc_maxfl: {
1454:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1455:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1456:     Value *Op2 = EmitScalarExpr(E->getArg(2));
1457:     Value *Op3 = EmitScalarExpr(E->getArg(3));
1458:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_maxfl),
1459:                               {Op0, Op1, Op2, Op3});
1460:   }
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1461-1480
```cpp
1461:   case PPC::BI__builtin_ppc_maxfs: {
1462:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1463:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1464:     Value *Op2 = EmitScalarExpr(E->getArg(2));
1465:     Value *Op3 = EmitScalarExpr(E->getArg(3));
1466:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_maxfs),
1467:                               {Op0, Op1, Op2, Op3});
1468:   }
1469:   case PPC::BI__builtin_ppc_minfe: {
1470:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1471:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1472:     Value *Op2 = EmitScalarExpr(E->getArg(2));
1473:     Value *Op3 = EmitScalarExpr(E->getArg(3));
1474:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_minfe),
1475:                               {Op0, Op1, Op2, Op3});
1476:   }
1477:   case PPC::BI__builtin_ppc_minfl: {
1478:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1479:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1480:     Value *Op2 = EmitScalarExpr(E->getArg(2));
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1481-1500
```cpp
1481:     Value *Op3 = EmitScalarExpr(E->getArg(3));
1482:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_minfl),
1483:                               {Op0, Op1, Op2, Op3});
1484:   }
1485:   case PPC::BI__builtin_ppc_minfs: {
1486:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1487:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1488:     Value *Op2 = EmitScalarExpr(E->getArg(2));
1489:     Value *Op3 = EmitScalarExpr(E->getArg(3));
1490:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_minfs),
1491:                               {Op0, Op1, Op2, Op3});
1492:   }
1493:   case PPC::BI__builtin_ppc_swdiv:
1494:   case PPC::BI__builtin_ppc_swdivs: {
1495:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1496:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1497:     return Builder.CreateFDiv(Op0, Op1, "swdiv");
1498:   }
1499:   case PPC::BI__builtin_ppc_set_fpscr_rn:
1500:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_setrnd),
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1501-1520
```cpp
1501:                               {EmitScalarExpr(E->getArg(0))});
1502:   case PPC::BI__builtin_ppc_mffs:
1503:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_readflm));
1504: 
1505:   case PPC::BI__builtin_amo_lwat_s: {
1506:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1507:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1508:     Value *Op2 = EmitScalarExpr(E->getArg(2));
1509:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_amo_lwat),
1510:                               {Op0, Op1, Op2});
1511:   }
1512:   case PPC::BI__builtin_amo_ldat_s: {
1513:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1514:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1515:     Value *Op2 = EmitScalarExpr(E->getArg(2));
1516:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_amo_ldat),
1517:                               {Op0, Op1, Op2});
1518:   }
1519:   case PPC::BI__builtin_amo_lwat_cond_s: {
1520:     Value *Op0 = EmitScalarExpr(E->getArg(0));
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1521-1540
```cpp
1521:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1522:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_amo_lwat_cond),
1523:                               {Op0, Op1});
1524:   }
1525:   case PPC::BI__builtin_amo_ldat_cond_s: {
1526:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1527:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1528:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_amo_ldat_cond),
1529:                               {Op0, Op1});
1530:   }
1531:   case PPC::BI__builtin_amo_lwat_csne_s: {
1532:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1533:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1534:     Value *Op2 = EmitScalarExpr(E->getArg(2));
1535:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_amo_lwat_csne),
1536:                               {Op0, Op1, Op2});
1537:   }
1538:   case PPC::BI__builtin_amo_ldat_csne_s: {
1539:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1540:     Value *Op1 = EmitScalarExpr(E->getArg(1));
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1541-1560
```cpp
1541:     Value *Op2 = EmitScalarExpr(E->getArg(2));
1542:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_amo_ldat_csne),
1543:                               {Op0, Op1, Op2});
1544:   }
1545:   case PPC::BI__builtin_amo_stwat_s: {
1546:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1547:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1548:     Value *Op2 = EmitScalarExpr(E->getArg(2));
1549:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_amo_stwat),
1550:                               {Op0, Op1, Op2});
1551:   }
1552:   case PPC::BI__builtin_amo_stdat_s: {
1553:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1554:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1555:     Value *Op2 = EmitScalarExpr(E->getArg(2));
1556:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::ppc_amo_stdat),
1557:                               {Op0, Op1, Op2});
1558:   }
1559:   }
1560: }
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

## Key Concepts / 关键概念

- **Builder**: Acts as a construction helper that incrementally assembles target builtin lowering state. / 充当构建辅助器，逐步组装 目标内建函数降级 状态。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **getArg**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **EmitScalarExpr**: Suggests an entry point that materializes IR or helper objects for target builtin lowering. / 暗示其是为 目标内建函数降级 生成 IR 或辅助对象的入口。
- **Op0**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Intrinsic**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Op1**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Ops**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CGBuiltin.h`
- **Clang libraries / Clang 库**: `clang/Basic/TargetBuiltins.h`, `clang/Basic/BuiltinsPPC.def`, `clang/Basic/BuiltinsPPC.def`
- **LLVM libraries / LLVM 库**: `llvm/IR/InlineAsm.h`, `llvm/IR/IntrinsicsPowerPC.h`, `llvm/Support/ScopedPrinter.h`, `llvm/TargetParser/PPCTargetParser.def`, `llvm/TargetParser/PPCTargetParser.def`, `llvm/TargetParser/PPCTargetParser.def`, `llvm/TargetParser/PPCTargetParser.def`
