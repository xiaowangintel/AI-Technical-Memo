# CGBuiltin.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/CGBuiltin.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements the CGBuiltin portion of Clang's CodeGen pipeline.
- **Purpose (CN) / 目的（中文）**: 实现 Clang CodeGen 流水线中的 CGBuiltin 部分。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
 1: //===---- CGBuiltin.cpp - Emit LLVM Code for builtins ---------------------===//
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
14: #include "ABIInfo.h"
15: #include "CGCUDARuntime.h"
16: #include "CGCXXABI.h"
17: #include "CGDebugInfo.h"
18: #include "CGObjCRuntime.h"
19: #include "CGOpenCLRuntime.h"
20: #include "CGRecordLayout.h"
21: #include "CGValue.h"
22: #include "CodeGenFunction.h"
23: #include "CodeGenModule.h"
24: #include "ConstantEmitter.h"
25: #include "PatternInit.h"
26: #include "TargetInfo.h"
27: #include "clang/AST/OSLog.h"
28: #include "clang/AST/StmtVisitor.h"
29: #include "clang/Basic/DiagnosticFrontend.h"
30: #include "clang/Basic/TargetInfo.h"
```
- **EN**: This block imports local CodeGen headers `CGBuiltin.h`, `ABIInfo.h`, `CGCUDARuntime.h`, and 11 more; Clang headers `clang/AST/OSLog.h`, `clang/AST/StmtVisitor.h`, `clang/Basic/DiagnosticFrontend.h`, and 1 more; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CGBuiltin.h`, `ABIInfo.h`, `CGCUDARuntime.h`, and 11 more；Clang 头文件 `clang/AST/OSLog.h`, `clang/AST/StmtVisitor.h`, `clang/Basic/DiagnosticFrontend.h`, and 1 more；包含影响本编译单元构建方式的预处理结构。

### Lines 31-60
```cpp
31: #include "llvm/IR/InlineAsm.h"
32: #include "llvm/IR/Instruction.h"
33: #include "llvm/IR/Intrinsics.h"
34: #include "llvm/IR/IntrinsicsX86.h"
35: #include "llvm/IR/MatrixBuilder.h"
36: #include "llvm/Support/ConvertUTF.h"
37: #include "llvm/Support/ScopedPrinter.h"
38: #include <optional>
39: #include <utility>
40: 
41: using namespace clang;
42: using namespace CodeGen;
43: using namespace llvm;
44: 
45: /// Some builtins do not have library implementation on some targets and
46: /// are instead emitted as LLVM IRs by some target builtin emitters.
47: /// FIXME: Remove this when library support is added
48: static bool shouldEmitBuiltinAsIR(unsigned BuiltinID,
49:                                   const Builtin::Context &BI,
50:                                   const CodeGenFunction &CGF) {
51:   if (!CGF.CGM.getLangOpts().MathErrno &&
52:       CGF.CurFPFeatures.getExceptionMode() ==
53:           LangOptions::FPExceptionModeKind::FPE_Ignore &&
54:       !CGF.CGM.getTargetCodeGenInfo().supportsLibCall()) {
55:     switch (BuiltinID) {
56:     default:
57:       return false;
58:     case Builtin::BIlogbf:
59:     case Builtin::BI__builtin_logbf:
60:     case Builtin::BIlogb:
```
- **EN**: This block imports LLVM headers `llvm/IR/InlineAsm.h`, `llvm/IR/Instruction.h`, `llvm/IR/Intrinsics.h`, and 4 more; other headers `optional`, `utility`; opens or references namespaces `clang`, `CodeGen`, `llvm`; defines callable entry points like `shouldEmitBuiltinAsIR`; uses control flow (if, switch, case) to specialize LLVM IR emission; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 LLVM 头文件 `llvm/IR/InlineAsm.h`, `llvm/IR/Instruction.h`, `llvm/IR/Intrinsics.h`, and 4 more；其他头文件 `optional`, `utility`；打开或引用命名空间 `clang`, `CodeGen`, `llvm`；定义可调用入口，例如 `shouldEmitBuiltinAsIR`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 61-90
```cpp
61:     case Builtin::BI__builtin_logb:
62:     case Builtin::BIscalbnf:
63:     case Builtin::BI__builtin_scalbnf:
64:     case Builtin::BIscalbn:
65:     case Builtin::BI__builtin_scalbn:
66:       return true;
67:     }
68:   }
69:   return false;
70: }
71: 
72: static Value *EmitTargetArchBuiltinExpr(CodeGenFunction *CGF,
73:                                         unsigned BuiltinID, const CallExpr *E,
74:                                         ReturnValueSlot ReturnValue,
75:                                         llvm::Triple::ArchType Arch) {
76:   // When compiling in HipStdPar mode we have to be conservative in rejecting
77:   // target specific features in the FE, and defer the possible error to the
78:   // AcceleratorCodeSelection pass, wherein iff an unsupported target builtin is
79:   // referenced by an accelerator executable function, we emit an error.
80:   // Returning nullptr here leads to the builtin being handled in
81:   // EmitStdParUnsupportedBuiltin.
82:   if (CGF->getLangOpts().HIPStdPar && CGF->getLangOpts().CUDAIsDevice &&
83:       Arch != CGF->getTarget().getTriple().getArch())
84:     return nullptr;
85: 
86:   switch (Arch) {
87:   case llvm::Triple::arm:
88:   case llvm::Triple::armeb:
89:   case llvm::Triple::thumb:
90:   case llvm::Triple::thumbeb:
```
- **EN**: This block uses control flow (if, switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, switch, case）细化 LLVM IR 生成 行为。

### Lines 91-120
```cpp
 91:     return CGF->EmitARMBuiltinExpr(BuiltinID, E, ReturnValue, Arch);
 92:   case llvm::Triple::aarch64:
 93:   case llvm::Triple::aarch64_32:
 94:   case llvm::Triple::aarch64_be:
 95:     return CGF->EmitAArch64BuiltinExpr(BuiltinID, E, Arch);
 96:   case llvm::Triple::bpfeb:
 97:   case llvm::Triple::bpfel:
 98:     return CGF->EmitBPFBuiltinExpr(BuiltinID, E);
 99:   case llvm::Triple::dxil:
100:     return CGF->EmitDirectXBuiltinExpr(BuiltinID, E);
101:   case llvm::Triple::x86:
102:   case llvm::Triple::x86_64:
103:     return CGF->EmitX86BuiltinExpr(BuiltinID, E);
104:   case llvm::Triple::ppc:
105:   case llvm::Triple::ppcle:
106:   case llvm::Triple::ppc64:
107:   case llvm::Triple::ppc64le:
108:     return CGF->EmitPPCBuiltinExpr(BuiltinID, E);
109:   case llvm::Triple::r600:
110:   case llvm::Triple::amdgcn:
111:     return CGF->EmitAMDGPUBuiltinExpr(BuiltinID, E);
112:   case llvm::Triple::systemz:
113:     return CGF->EmitSystemZBuiltinExpr(BuiltinID, E);
114:   case llvm::Triple::nvptx:
115:   case llvm::Triple::nvptx64:
116:     return CGF->EmitNVPTXBuiltinExpr(BuiltinID, E);
117:   case llvm::Triple::wasm32:
118:   case llvm::Triple::wasm64:
119:     return CGF->EmitWebAssemblyBuiltinExpr(BuiltinID, E);
120:   case llvm::Triple::hexagon:
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 121-150
```cpp
121:     return CGF->EmitHexagonBuiltinExpr(BuiltinID, E);
122:   case llvm::Triple::riscv32:
123:   case llvm::Triple::riscv64:
124:   case llvm::Triple::riscv32be:
125:   case llvm::Triple::riscv64be:
126:     return CGF->EmitRISCVBuiltinExpr(BuiltinID, E, ReturnValue);
127:   case llvm::Triple::spirv32:
128:   case llvm::Triple::spirv64:
129:     if (CGF->getTarget().getTriple().getOS() == llvm::Triple::OSType::AMDHSA)
130:       return CGF->EmitAMDGPUBuiltinExpr(BuiltinID, E);
131:     [[fallthrough]];
132:   case llvm::Triple::spirv:
133:     return CGF->EmitSPIRVBuiltinExpr(BuiltinID, E);
134:   default:
135:     return nullptr;
136:   }
137: }
138: 
139: Value *CodeGenFunction::EmitTargetBuiltinExpr(unsigned BuiltinID,
140:                                               const CallExpr *E,
141:                                               ReturnValueSlot ReturnValue) {
142:   if (getContext().BuiltinInfo.isAuxBuiltinID(BuiltinID)) {
143:     assert(getContext().getAuxTargetInfo() && "Missing aux target info");
144:     return EmitTargetArchBuiltinExpr(
145:         this, getContext().BuiltinInfo.getAuxBuiltinID(BuiltinID), E,
146:         ReturnValue, getContext().getAuxTargetInfo()->getTriple().getArch());
147:   }
148: 
149:   return EmitTargetArchBuiltinExpr(this, BuiltinID, E, ReturnValue,
150:                                    getTarget().getTriple().getArch());
```
- **EN**: This block defines callable entry points like `EmitTargetArchBuiltinExpr`; uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitTargetArchBuiltinExpr`；通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 151-180
```cpp
151: }
152: 
153: static void initializeAlloca(CodeGenFunction &CGF, AllocaInst *AI, Value *Size,
154:                              Align AlignmentInBytes) {
155:   ConstantInt *Byte;
156:   switch (CGF.getLangOpts().getTrivialAutoVarInit()) {
157:   case LangOptions::TrivialAutoVarInitKind::Uninitialized:
158:     // Nothing to initialize.
159:     return;
160:   case LangOptions::TrivialAutoVarInitKind::Zero:
161:     Byte = CGF.Builder.getInt8(0x00);
162:     break;
163:   case LangOptions::TrivialAutoVarInitKind::Pattern: {
164:     llvm::Type *Int8 = llvm::IntegerType::getInt8Ty(CGF.CGM.getLLVMContext());
165:     Byte = llvm::dyn_cast<llvm::ConstantInt>(
166:         initializationPatternFor(CGF.CGM, Int8));
167:     break;
168:   }
169:   }
170:   if (CGF.CGM.stopAutoInit())
171:     return;
172:   auto *I = CGF.Builder.CreateMemSet(AI, Byte, Size, AlignmentInBytes);
173:   I->addAnnotationMetadata("auto-init");
174: }
175: 
176: /// getBuiltinLibFunction - Given a builtin id for a function like
177: /// "__builtin_fabsf", return a Function* for "fabsf".
178: llvm::Constant *CodeGenModule::getBuiltinLibFunction(const FunctionDecl *FD,
179:                                                      unsigned BuiltinID) {
180:   assert(Context.BuiltinInfo.isLibFunction(BuiltinID));
```
- **EN**: This block defines callable entry points like `initializeAlloca`, `initializationPatternFor`; uses control flow (if, switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `initializeAlloca`, `initializationPatternFor`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 181-210
```cpp
181: 
182:   // Get the name, skip over the __builtin_ prefix (if necessary). We may have
183:   // to build this up so provide a small stack buffer to handle the vast
184:   // majority of names.
185:   llvm::SmallString<64> Name;
186:   GlobalDecl D(FD);
187: 
188:   // TODO: This list should be expanded or refactored after all GCC-compatible
189:   // std libcall builtins are implemented.
190:   static const SmallDenseMap<unsigned, StringRef, 64> F128Builtins{
191:       {Builtin::BI__builtin___fprintf_chk, "__fprintf_chkieee128"},
192:       {Builtin::BI__builtin___printf_chk, "__printf_chkieee128"},
193:       {Builtin::BI__builtin___snprintf_chk, "__snprintf_chkieee128"},
194:       {Builtin::BI__builtin___sprintf_chk, "__sprintf_chkieee128"},
195:       {Builtin::BI__builtin___vfprintf_chk, "__vfprintf_chkieee128"},
196:       {Builtin::BI__builtin___vprintf_chk, "__vprintf_chkieee128"},
197:       {Builtin::BI__builtin___vsnprintf_chk, "__vsnprintf_chkieee128"},
198:       {Builtin::BI__builtin___vsprintf_chk, "__vsprintf_chkieee128"},
199:       {Builtin::BI__builtin_fprintf, "__fprintfieee128"},
200:       {Builtin::BI__builtin_printf, "__printfieee128"},
201:       {Builtin::BI__builtin_snprintf, "__snprintfieee128"},
202:       {Builtin::BI__builtin_sprintf, "__sprintfieee128"},
203:       {Builtin::BI__builtin_vfprintf, "__vfprintfieee128"},
204:       {Builtin::BI__builtin_vprintf, "__vprintfieee128"},
205:       {Builtin::BI__builtin_vsnprintf, "__vsnprintfieee128"},
206:       {Builtin::BI__builtin_vsprintf, "__vsprintfieee128"},
207:       {Builtin::BI__builtin_fscanf, "__fscanfieee128"},
208:       {Builtin::BI__builtin_scanf, "__scanfieee128"},
209:       {Builtin::BI__builtin_sscanf, "__sscanfieee128"},
210:       {Builtin::BI__builtin_vfscanf, "__vfscanfieee128"},
```
- **EN**: This block defines callable entry points like `D`.
- **CN**: 该代码块定义可调用入口，例如 `D`。

### Lines 211-240
```cpp
211:       {Builtin::BI__builtin_vscanf, "__vscanfieee128"},
212:       {Builtin::BI__builtin_vsscanf, "__vsscanfieee128"},
213:       {Builtin::BI__builtin_nexttowardf128, "__nexttowardieee128"},
214:   };
215: 
216:   // The AIX library functions frexpl, ldexpl, and modfl are for 128-bit
217:   // IBM 'long double' (i.e. __ibm128). Map to the 'double' versions
218:   // if it is 64-bit 'long double' mode.
219:   static const SmallDenseMap<unsigned, StringRef, 4> AIXLongDouble64Builtins{
220:       {Builtin::BI__builtin_frexpl, "frexp"},
221:       {Builtin::BI__builtin_ldexpl, "ldexp"},
222:       {Builtin::BI__builtin_modfl, "modf"},
223:   };
224: 
225:   // If the builtin has been declared explicitly with an assembler label,
226:   // use the mangled name. This differs from the plain label on platforms
227:   // that prefix labels.
228:   if (FD->hasAttr<AsmLabelAttr>())
229:     Name = getMangledName(D);
230:   else {
231:     // TODO: This mutation should also be applied to other targets other than
232:     // PPC, after backend supports IEEE 128-bit style libcalls.
233:     if (getTriple().isPPC64() &&
234:         &getTarget().getLongDoubleFormat() == &llvm::APFloat::IEEEquad() &&
235:         F128Builtins.contains(BuiltinID))
236:       Name = F128Builtins.lookup(BuiltinID);
237:     else if (getTriple().isOSAIX() &&
238:              &getTarget().getLongDoubleFormat() ==
239:                  &llvm::APFloat::IEEEdouble() &&
240:              AIXLongDouble64Builtins.contains(BuiltinID))
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 241-270
```cpp
241:       Name = AIXLongDouble64Builtins.lookup(BuiltinID);
242:     else
243:       Name = Context.BuiltinInfo.getName(BuiltinID).substr(10);
244:   }
245: 
246:   llvm::FunctionType *Ty =
247:     cast<llvm::FunctionType>(getTypes().ConvertType(FD->getType()));
248: 
249:   return GetOrCreateLLVMFunction(Name, Ty, D, /*ForVTable=*/false);
250: }
251: 
252: /// Emit the conversions required to turn the given value into an
253: /// integer of the given size.
254: Value *EmitToInt(CodeGenFunction &CGF, llvm::Value *V,
255:                         QualType T, llvm::IntegerType *IntType) {
256:   V = CGF.EmitToMemory(V, T);
257: 
258:   if (V->getType()->isPointerTy())
259:     return CGF.Builder.CreatePtrToInt(V, IntType);
260: 
261:   assert(V->getType() == IntType);
262:   return V;
263: }
264: 
265: Value *EmitFromInt(CodeGenFunction &CGF, llvm::Value *V,
266:                           QualType T, llvm::Type *ResultType) {
267:   V = CGF.EmitFromMemory(V, T);
268: 
269:   if (ResultType->isPointerTy())
270:     return CGF.Builder.CreateIntToPtr(V, ResultType);
```
- **EN**: This block defines callable entry points like `GetOrCreateLLVMFunction`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `GetOrCreateLLVMFunction`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 271-300
```cpp
271: 
272:   assert(V->getType() == ResultType);
273:   return V;
274: }
275: 
276: Address CheckAtomicAlignment(CodeGenFunction &CGF, const CallExpr *E) {
277:   ASTContext &Ctx = CGF.getContext();
278:   Address Ptr = CGF.EmitPointerWithAlignment(E->getArg(0));
279:   const llvm::DataLayout &DL = CGF.CGM.getDataLayout();
280:   unsigned Bytes = Ptr.getElementType()->isPointerTy()
281:                        ? Ctx.getTypeSizeInChars(Ctx.VoidPtrTy).getQuantity()
282:                        : DL.getTypeStoreSize(Ptr.getElementType());
283:   unsigned Align = Ptr.getAlignment().getQuantity();
284:   if (Align % Bytes != 0) {
285:     DiagnosticsEngine &Diags = CGF.CGM.getDiags();
286:     Diags.Report(E->getBeginLoc(), diag::warn_sync_op_misaligned);
287:     // Force address to be at least naturally-aligned.
288:     return Ptr.withAlignment(CharUnits::fromQuantity(Bytes));
289:   }
290:   return Ptr;
291: }
292: 
293: /// Utility to insert an atomic instruction based on Intrinsic::ID
294: /// and the expression node.
295: Value *MakeBinaryAtomicValue(
296:     CodeGenFunction &CGF, llvm::AtomicRMWInst::BinOp Kind, const CallExpr *E,
297:     AtomicOrdering Ordering) {
298: 
299:   QualType T = E->getType();
300:   assert(E->getArg(0)->getType()->isPointerType());
```
- **EN**: This block defines callable entry points like `CheckAtomicAlignment`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `CheckAtomicAlignment`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 301-330
```cpp
301:   assert(CGF.getContext().hasSameUnqualifiedType(T,
302:                                   E->getArg(0)->getType()->getPointeeType()));
303:   assert(CGF.getContext().hasSameUnqualifiedType(T, E->getArg(1)->getType()));
304: 
305:   Address DestAddr = CheckAtomicAlignment(CGF, E);
306: 
307:   llvm::IntegerType *IntType = llvm::IntegerType::get(
308:       CGF.getLLVMContext(), CGF.getContext().getTypeSize(T));
309: 
310:   llvm::Value *Val = CGF.EmitScalarExpr(E->getArg(1));
311:   llvm::Type *ValueType = Val->getType();
312:   Val = EmitToInt(CGF, Val, T, IntType);
313: 
314:   llvm::Value *Result =
315:       CGF.Builder.CreateAtomicRMW(Kind, DestAddr, Val, Ordering);
316:   return EmitFromInt(CGF, Result, T, ValueType);
317: }
318: 
319: static Value *EmitNontemporalStore(CodeGenFunction &CGF, const CallExpr *E) {
320:   Value *Val = CGF.EmitScalarExpr(E->getArg(0));
321:   Address Addr = CGF.EmitPointerWithAlignment(E->getArg(1));
322: 
323:   Val = CGF.EmitToMemory(Val, E->getArg(0)->getType());
324:   LValue LV = CGF.MakeAddrLValue(Addr, E->getArg(0)->getType());
325:   LV.setNontemporal(true);
326:   CGF.EmitStoreOfScalar(Val, LV, false);
327:   return nullptr;
328: }
329: 
330: static Value *EmitNontemporalLoad(CodeGenFunction &CGF, const CallExpr *E) {
```
- **EN**: This block defines callable entry points like `EmitFromInt`; returns or forwards computed values for the surrounding LLVM IR emission logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitFromInt`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 331-360
```cpp
331:   Address Addr = CGF.EmitPointerWithAlignment(E->getArg(0));
332: 
333:   LValue LV = CGF.MakeAddrLValue(Addr, E->getType());
334:   LV.setNontemporal(true);
335:   return CGF.EmitLoadOfScalar(LV, E->getExprLoc());
336: }
337: 
338: static RValue EmitBinaryAtomic(CodeGenFunction &CGF,
339:                                llvm::AtomicRMWInst::BinOp Kind,
340:                                const CallExpr *E) {
341:   return RValue::get(MakeBinaryAtomicValue(CGF, Kind, E));
342: }
343: 
344: /// Utility to insert an atomic instruction based Intrinsic::ID and
345: /// the expression node, where the return value is the result of the
346: /// operation.
347: static RValue EmitBinaryAtomicPost(CodeGenFunction &CGF,
348:                                    llvm::AtomicRMWInst::BinOp Kind,
349:                                    const CallExpr *E,
350:                                    Instruction::BinaryOps Op,
351:                                    bool Invert = false) {
352:   QualType T = E->getType();
353:   assert(E->getArg(0)->getType()->isPointerType());
354:   assert(CGF.getContext().hasSameUnqualifiedType(T,
355:                                   E->getArg(0)->getType()->getPointeeType()));
356:   assert(CGF.getContext().hasSameUnqualifiedType(T, E->getArg(1)->getType()));
357: 
358:   Address DestAddr = CheckAtomicAlignment(CGF, E);
359: 
360:   llvm::IntegerType *IntType = llvm::IntegerType::get(
```
- **EN**: This block defines callable entry points like `EmitBinaryAtomic`, `get`, `EmitBinaryAtomicPost`; returns or forwards computed values for the surrounding LLVM IR emission logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitBinaryAtomic`, `get`, `EmitBinaryAtomicPost`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 361-390
```cpp
361:       CGF.getLLVMContext(), CGF.getContext().getTypeSize(T));
362: 
363:   llvm::Value *Val = CGF.EmitScalarExpr(E->getArg(1));
364:   llvm::Type *ValueType = Val->getType();
365:   Val = EmitToInt(CGF, Val, T, IntType);
366: 
367:   llvm::Value *Result = CGF.Builder.CreateAtomicRMW(
368:       Kind, DestAddr, Val, llvm::AtomicOrdering::SequentiallyConsistent);
369:   Result = CGF.Builder.CreateBinOp(Op, Result, Val);
370:   if (Invert)
371:     Result =
372:         CGF.Builder.CreateBinOp(llvm::Instruction::Xor, Result,
373:                                 llvm::ConstantInt::getAllOnesValue(IntType));
374:   Result = EmitFromInt(CGF, Result, T, ValueType);
375:   return RValue::get(Result);
376: }
377: 
378: /// Utility to insert an atomic cmpxchg instruction.
379: ///
380: /// @param CGF The current codegen function.
381: /// @param E   Builtin call expression to convert to cmpxchg.
382: ///            arg0 - address to operate on
383: ///            arg1 - value to compare with
384: ///            arg2 - new value
385: /// @param ReturnBool Specifies whether to return success flag of
386: ///                   cmpxchg result or the old value.
387: ///
388: /// @returns result of cmpxchg, according to ReturnBool
389: ///
390: /// Note: In order to lower Microsoft's _InterlockedCompareExchange* intrinsics
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 391-420
```cpp
391: /// invoke the function EmitAtomicCmpXchgForMSIntrin.
392: Value *MakeAtomicCmpXchgValue(CodeGenFunction &CGF, const CallExpr *E,
393:                               bool ReturnBool,
394:                               llvm::AtomicOrdering SuccessOrdering,
395:                               llvm::AtomicOrdering FailureOrdering) {
396:   QualType T = ReturnBool ? E->getArg(1)->getType() : E->getType();
397:   Address DestAddr = CheckAtomicAlignment(CGF, E);
398: 
399:   llvm::IntegerType *IntType = llvm::IntegerType::get(
400:       CGF.getLLVMContext(), CGF.getContext().getTypeSize(T));
401: 
402:   Value *Cmp = CGF.EmitScalarExpr(E->getArg(1));
403:   llvm::Type *ValueType = Cmp->getType();
404:   Cmp = EmitToInt(CGF, Cmp, T, IntType);
405:   Value *New = EmitToInt(CGF, CGF.EmitScalarExpr(E->getArg(2)), T, IntType);
406: 
407:   Value *Pair = CGF.Builder.CreateAtomicCmpXchg(
408:       DestAddr, Cmp, New, SuccessOrdering, FailureOrdering);
409:   if (ReturnBool)
410:     // Extract boolean success flag and zext it to int.
411:     return CGF.Builder.CreateZExt(CGF.Builder.CreateExtractValue(Pair, 1),
412:                                   CGF.ConvertType(E->getType()));
413:   else
414:     // Extract old value and emit it using the same type as compare value.
415:     return EmitFromInt(CGF, CGF.Builder.CreateExtractValue(Pair, 0), T,
416:                        ValueType);
417: }
418: 
419: /// This function should be invoked to emit atomic cmpxchg for Microsoft's
420: /// _InterlockedCompareExchange* intrinsics which have the following signature:
```
- **EN**: This block defines callable entry points like `EmitFromInt`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitFromInt`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 421-450
```cpp
421: /// T _InterlockedCompareExchange(T volatile *Destination,
422: ///                               T Exchange,
423: ///                               T Comparand);
424: ///
425: /// Whereas the llvm 'cmpxchg' instruction has the following syntax:
426: /// cmpxchg *Destination, Comparand, Exchange.
427: /// So we need to swap Comparand and Exchange when invoking
428: /// CreateAtomicCmpXchg. That is the reason we could not use the above utility
429: /// function MakeAtomicCmpXchgValue since it expects the arguments to be
430: /// already swapped.
431: 
432: static
433: Value *EmitAtomicCmpXchgForMSIntrin(CodeGenFunction &CGF, const CallExpr *E,
434:     AtomicOrdering SuccessOrdering = AtomicOrdering::SequentiallyConsistent) {
435:   assert(E->getArg(0)->getType()->isPointerType());
436:   assert(CGF.getContext().hasSameUnqualifiedType(
437:       E->getType(), E->getArg(0)->getType()->getPointeeType()));
438:   assert(CGF.getContext().hasSameUnqualifiedType(E->getType(),
439:                                                  E->getArg(1)->getType()));
440:   assert(CGF.getContext().hasSameUnqualifiedType(E->getType(),
441:                                                  E->getArg(2)->getType()));
442: 
443:   Address DestAddr = CheckAtomicAlignment(CGF, E);
444: 
445:   auto *Exchange = CGF.EmitScalarExpr(E->getArg(1));
446:   auto *RTy = Exchange->getType();
447: 
448:   auto *Comparand = CGF.EmitScalarExpr(E->getArg(2));
449: 
450:   if (RTy->isPointerTy()) {
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 451-480
```cpp
451:     Exchange = CGF.Builder.CreatePtrToInt(Exchange, CGF.IntPtrTy);
452:     Comparand = CGF.Builder.CreatePtrToInt(Comparand, CGF.IntPtrTy);
453:   }
454: 
455:   // For Release ordering, the failure ordering should be Monotonic.
456:   auto FailureOrdering = SuccessOrdering == AtomicOrdering::Release ?
457:                          AtomicOrdering::Monotonic :
458:                          SuccessOrdering;
459: 
460:   // The atomic instruction is marked volatile for consistency with MSVC. This
461:   // blocks the few atomics optimizations that LLVM has. If we want to optimize
462:   // _Interlocked* operations in the future, we will have to remove the volatile
463:   // marker.
464:   auto *CmpXchg = CGF.Builder.CreateAtomicCmpXchg(
465:       DestAddr, Comparand, Exchange, SuccessOrdering, FailureOrdering);
466:   CmpXchg->setVolatile(true);
467: 
468:   auto *Result = CGF.Builder.CreateExtractValue(CmpXchg, 0);
469:   if (RTy->isPointerTy()) {
470:     Result = CGF.Builder.CreateIntToPtr(Result, RTy);
471:   }
472: 
473:   return Result;
474: }
475: 
476: // 64-bit Microsoft platforms support 128 bit cmpxchg operations. They are
477: // prototyped like this:
478: //
479: // unsigned char _InterlockedCompareExchange128...(
480: //     __int64 volatile * _Destination,
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 481-510
```cpp
481: //     __int64 _ExchangeHigh,
482: //     __int64 _ExchangeLow,
483: //     __int64 * _ComparandResult);
484: //
485: // Note that Destination is assumed to be at least 16-byte aligned, despite
486: // being typed int64.
487: 
488: static Value *EmitAtomicCmpXchg128ForMSIntrin(CodeGenFunction &CGF,
489:                                               const CallExpr *E,
490:                                               AtomicOrdering SuccessOrdering) {
491:   assert(E->getNumArgs() == 4);
492:   llvm::Value *DestPtr = CGF.EmitScalarExpr(E->getArg(0));
493:   llvm::Value *ExchangeHigh = CGF.EmitScalarExpr(E->getArg(1));
494:   llvm::Value *ExchangeLow = CGF.EmitScalarExpr(E->getArg(2));
495:   Address ComparandAddr = CGF.EmitPointerWithAlignment(E->getArg(3));
496: 
497:   assert(DestPtr->getType()->isPointerTy());
498:   assert(!ExchangeHigh->getType()->isPointerTy());
499:   assert(!ExchangeLow->getType()->isPointerTy());
500: 
501:   // For Release ordering, the failure ordering should be Monotonic.
502:   auto FailureOrdering = SuccessOrdering == AtomicOrdering::Release
503:                              ? AtomicOrdering::Monotonic
504:                              : SuccessOrdering;
505: 
506:   // Convert to i128 pointers and values. Alignment is also overridden for
507:   // destination pointer.
508:   llvm::Type *Int128Ty = llvm::IntegerType::get(CGF.getLLVMContext(), 128);
509:   Address DestAddr(DestPtr, Int128Ty,
510:                    CGF.getContext().toCharUnitsFromBits(128));
```
- **EN**: This block defines callable entry points like `DestAddr`; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `DestAddr`；使用断言或不可达标记保护关键不变量。

### Lines 511-540
```cpp
511:   ComparandAddr = ComparandAddr.withElementType(Int128Ty);
512: 
513:   // (((i128)hi) << 64) | ((i128)lo)
514:   ExchangeHigh = CGF.Builder.CreateZExt(ExchangeHigh, Int128Ty);
515:   ExchangeLow = CGF.Builder.CreateZExt(ExchangeLow, Int128Ty);
516:   ExchangeHigh =
517:       CGF.Builder.CreateShl(ExchangeHigh, llvm::ConstantInt::get(Int128Ty, 64));
518:   llvm::Value *Exchange = CGF.Builder.CreateOr(ExchangeHigh, ExchangeLow);
519: 
520:   // Load the comparand for the instruction.
521:   llvm::Value *Comparand = CGF.Builder.CreateLoad(ComparandAddr);
522: 
523:   auto *CXI = CGF.Builder.CreateAtomicCmpXchg(DestAddr, Comparand, Exchange,
524:                                               SuccessOrdering, FailureOrdering);
525: 
526:   // The atomic instruction is marked volatile for consistency with MSVC. This
527:   // blocks the few atomics optimizations that LLVM has. If we want to optimize
528:   // _Interlocked* operations in the future, we will have to remove the volatile
529:   // marker.
530:   CXI->setVolatile(true);
531: 
532:   // Store the result as an outparameter.
533:   CGF.Builder.CreateStore(CGF.Builder.CreateExtractValue(CXI, 0),
534:                           ComparandAddr);
535: 
536:   // Get the success boolean and zero extend it to i8.
537:   Value *Success = CGF.Builder.CreateExtractValue(CXI, 1);
538:   return CGF.Builder.CreateZExt(Success, CGF.Int8Ty);
539: }
540: 
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 541-570
```cpp
541: static Value *EmitAtomicIncrementValue(CodeGenFunction &CGF, const CallExpr *E,
542:     AtomicOrdering Ordering = AtomicOrdering::SequentiallyConsistent) {
543:   assert(E->getArg(0)->getType()->isPointerType());
544: 
545:   auto *IntTy = CGF.ConvertType(E->getType());
546:   Address DestAddr = CheckAtomicAlignment(CGF, E);
547:   auto *Result = CGF.Builder.CreateAtomicRMW(
548:       AtomicRMWInst::Add, DestAddr, ConstantInt::get(IntTy, 1), Ordering);
549:   return CGF.Builder.CreateAdd(Result, ConstantInt::get(IntTy, 1));
550: }
551: 
552: static Value *EmitAtomicDecrementValue(
553:     CodeGenFunction &CGF, const CallExpr *E,
554:     AtomicOrdering Ordering = AtomicOrdering::SequentiallyConsistent) {
555:   assert(E->getArg(0)->getType()->isPointerType());
556: 
557:   auto *IntTy = CGF.ConvertType(E->getType());
558:   Address DestAddr = CheckAtomicAlignment(CGF, E);
559:   auto *Result = CGF.Builder.CreateAtomicRMW(
560:       AtomicRMWInst::Sub, DestAddr, ConstantInt::get(IntTy, 1), Ordering);
561:   return CGF.Builder.CreateSub(Result, ConstantInt::get(IntTy, 1));
562: }
563: 
564: // Build a plain volatile load.
565: static Value *EmitISOVolatileLoad(CodeGenFunction &CGF, const CallExpr *E) {
566:   Value *Ptr = CGF.EmitScalarExpr(E->getArg(0));
567:   QualType ElTy = E->getArg(0)->getType()->getPointeeType();
568:   CharUnits LoadSize = CGF.getContext().getTypeSizeInChars(ElTy);
569:   llvm::Type *ITy =
570:       llvm::IntegerType::get(CGF.getLLVMContext(), LoadSize.getQuantity() * 8);
```
- **EN**: This block defines callable entry points like `get`; returns or forwards computed values for the surrounding LLVM IR emission logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 571-600
```cpp
571:   llvm::LoadInst *Load = CGF.Builder.CreateAlignedLoad(ITy, Ptr, LoadSize);
572:   Load->setAtomic(llvm::AtomicOrdering::Monotonic);
573:   Load->setVolatile(true);
574:   return Load;
575: }
576: 
577: // Build a plain volatile store.
578: static Value *EmitISOVolatileStore(CodeGenFunction &CGF, const CallExpr *E) {
579:   Value *Ptr = CGF.EmitScalarExpr(E->getArg(0));
580:   Value *Value = CGF.EmitScalarExpr(E->getArg(1));
581:   QualType ElTy = E->getArg(0)->getType()->getPointeeType();
582:   CharUnits StoreSize = CGF.getContext().getTypeSizeInChars(ElTy);
583:   llvm::StoreInst *Store =
584:       CGF.Builder.CreateAlignedStore(Value, Ptr, StoreSize);
585:   Store->setAtomic(llvm::AtomicOrdering::Monotonic);
586:   Store->setVolatile(true);
587:   return Store;
588: }
589: 
590: // Emit a simple mangled intrinsic that has 1 argument and a return type
591: // matching the argument type. Depending on mode, this may be a constrained
592: // floating-point intrinsic.
593: Value *emitUnaryMaybeConstrainedFPBuiltin(CodeGenFunction &CGF,
594:                                 const CallExpr *E, unsigned IntrinsicID,
595:                                 unsigned ConstrainedIntrinsicID) {
596:   llvm::Value *Src0 = CGF.EmitScalarExpr(E->getArg(0));
597: 
598:   CodeGenFunction::CGFPOptionsRAII FPOptsRAII(CGF, E);
599:   if (CGF.Builder.getIsFPConstrained()) {
600:     Function *F = CGF.CGM.getIntrinsic(ConstrainedIntrinsicID, Src0->getType());
```
- **EN**: This block defines callable entry points like `FPOptsRAII`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `FPOptsRAII`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 601-630
```cpp
601:     return CGF.Builder.CreateConstrainedFPCall(F, { Src0 });
602:   } else {
603:     Function *F = CGF.CGM.getIntrinsic(IntrinsicID, Src0->getType());
604:     return CGF.Builder.CreateCall(F, Src0);
605:   }
606: }
607: 
608: // Emit an intrinsic that has 2 operands of the same type as its result.
609: // Depending on mode, this may be a constrained floating-point intrinsic.
610: static Value *emitBinaryMaybeConstrainedFPBuiltin(CodeGenFunction &CGF,
611:                                 const CallExpr *E, unsigned IntrinsicID,
612:                                 unsigned ConstrainedIntrinsicID) {
613:   llvm::Value *Src0 = CGF.EmitScalarExpr(E->getArg(0));
614:   llvm::Value *Src1 = CGF.EmitScalarExpr(E->getArg(1));
615: 
616:   CodeGenFunction::CGFPOptionsRAII FPOptsRAII(CGF, E);
617:   if (CGF.Builder.getIsFPConstrained()) {
618:     Function *F = CGF.CGM.getIntrinsic(ConstrainedIntrinsicID, Src0->getType());
619:     return CGF.Builder.CreateConstrainedFPCall(F, { Src0, Src1 });
620:   } else {
621:     Function *F = CGF.CGM.getIntrinsic(IntrinsicID, Src0->getType());
622:     return CGF.Builder.CreateCall(F, { Src0, Src1 });
623:   }
624: }
625: 
626: // Has second type mangled argument.
627: static Value *
628: emitBinaryExpMaybeConstrainedFPBuiltin(CodeGenFunction &CGF, const CallExpr *E,
629:                                        Intrinsic::ID IntrinsicID,
630:                                        Intrinsic::ID ConstrainedIntrinsicID) {
```
- **EN**: This block defines callable entry points like `FPOptsRAII`, `emitBinaryExpMaybeConstrainedFPBuiltin`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `FPOptsRAII`, `emitBinaryExpMaybeConstrainedFPBuiltin`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 631-660
```cpp
631:   llvm::Value *Src0 = CGF.EmitScalarExpr(E->getArg(0));
632:   llvm::Value *Src1 = CGF.EmitScalarExpr(E->getArg(1));
633: 
634:   CodeGenFunction::CGFPOptionsRAII FPOptsRAII(CGF, E);
635:   if (CGF.Builder.getIsFPConstrained()) {
636:     Function *F = CGF.CGM.getIntrinsic(ConstrainedIntrinsicID,
637:                                        {Src0->getType(), Src1->getType()});
638:     return CGF.Builder.CreateConstrainedFPCall(F, {Src0, Src1});
639:   }
640: 
641:   Function *F =
642:       CGF.CGM.getIntrinsic(IntrinsicID, {Src0->getType(), Src1->getType()});
643:   return CGF.Builder.CreateCall(F, {Src0, Src1});
644: }
645: 
646: // Emit an intrinsic that has 3 operands of the same type as its result.
647: // Depending on mode, this may be a constrained floating-point intrinsic.
648: static Value *emitTernaryMaybeConstrainedFPBuiltin(CodeGenFunction &CGF,
649:                                  const CallExpr *E, unsigned IntrinsicID,
650:                                  unsigned ConstrainedIntrinsicID) {
651:   llvm::Value *Src0 = CGF.EmitScalarExpr(E->getArg(0));
652:   llvm::Value *Src1 = CGF.EmitScalarExpr(E->getArg(1));
653:   llvm::Value *Src2 = CGF.EmitScalarExpr(E->getArg(2));
654: 
655:   CodeGenFunction::CGFPOptionsRAII FPOptsRAII(CGF, E);
656:   if (CGF.Builder.getIsFPConstrained()) {
657:     Function *F = CGF.CGM.getIntrinsic(ConstrainedIntrinsicID, Src0->getType());
658:     return CGF.Builder.CreateConstrainedFPCall(F, { Src0, Src1, Src2 });
659:   } else {
660:     Function *F = CGF.CGM.getIntrinsic(IntrinsicID, Src0->getType());
```
- **EN**: This block defines callable entry points like `FPOptsRAII`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `FPOptsRAII`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 661-690
```cpp
661:     return CGF.Builder.CreateCall(F, { Src0, Src1, Src2 });
662:   }
663: }
664: 
665: // Emit an intrinsic that has overloaded integer result and fp operand.
666: static Value *
667: emitMaybeConstrainedFPToIntRoundBuiltin(CodeGenFunction &CGF, const CallExpr *E,
668:                                         unsigned IntrinsicID,
669:                                         unsigned ConstrainedIntrinsicID) {
670:   llvm::Type *ResultType = CGF.ConvertType(E->getType());
671:   llvm::Value *Src0 = CGF.EmitScalarExpr(E->getArg(0));
672: 
673:   if (CGF.Builder.getIsFPConstrained()) {
674:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(CGF, E);
675:     Function *F = CGF.CGM.getIntrinsic(ConstrainedIntrinsicID,
676:                                        {ResultType, Src0->getType()});
677:     return CGF.Builder.CreateConstrainedFPCall(F, {Src0});
678:   } else {
679:     Function *F =
680:         CGF.CGM.getIntrinsic(IntrinsicID, {ResultType, Src0->getType()});
681:     return CGF.Builder.CreateCall(F, Src0);
682:   }
683: }
684: 
685: static Value *emitFrexpBuiltin(CodeGenFunction &CGF, const CallExpr *E,
686:                                Intrinsic::ID IntrinsicID) {
687:   llvm::Value *Src0 = CGF.EmitScalarExpr(E->getArg(0));
688:   llvm::Value *Src1 = CGF.EmitScalarExpr(E->getArg(1));
689: 
690:   QualType IntPtrTy = E->getArg(1)->getType()->getPointeeType();
```
- **EN**: This block defines callable entry points like `emitMaybeConstrainedFPToIntRoundBuiltin`, `FPOptsRAII`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitMaybeConstrainedFPToIntRoundBuiltin`, `FPOptsRAII`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 691-720
```cpp
691:   llvm::Type *IntTy = CGF.ConvertType(IntPtrTy);
692:   llvm::Function *F =
693:       CGF.CGM.getIntrinsic(IntrinsicID, {Src0->getType(), IntTy});
694:   llvm::Value *Call = CGF.Builder.CreateCall(F, Src0);
695: 
696:   llvm::Value *Exp = CGF.Builder.CreateExtractValue(Call, 1);
697:   LValue LV = CGF.MakeNaturalAlignAddrLValue(Src1, IntPtrTy);
698:   CGF.EmitStoreOfScalar(Exp, LV);
699: 
700:   return CGF.Builder.CreateExtractValue(Call, 0);
701: }
702: 
703: static void emitSincosBuiltin(CodeGenFunction &CGF, const CallExpr *E,
704:                               Intrinsic::ID IntrinsicID) {
705:   llvm::Value *Val = CGF.EmitScalarExpr(E->getArg(0));
706:   llvm::Value *Dest0 = CGF.EmitScalarExpr(E->getArg(1));
707:   llvm::Value *Dest1 = CGF.EmitScalarExpr(E->getArg(2));
708: 
709:   llvm::Function *F = CGF.CGM.getIntrinsic(IntrinsicID, {Val->getType()});
710:   llvm::Value *Call = CGF.Builder.CreateCall(F, Val);
711: 
712:   llvm::Value *SinResult = CGF.Builder.CreateExtractValue(Call, 0);
713:   llvm::Value *CosResult = CGF.Builder.CreateExtractValue(Call, 1);
714: 
715:   QualType DestPtrType = E->getArg(1)->getType()->getPointeeType();
716:   LValue SinLV = CGF.MakeNaturalAlignAddrLValue(Dest0, DestPtrType);
717:   LValue CosLV = CGF.MakeNaturalAlignAddrLValue(Dest1, DestPtrType);
718: 
719:   llvm::StoreInst *StoreSin =
720:       CGF.Builder.CreateStore(SinResult, SinLV.getAddress());
```
- **EN**: This block defines callable entry points like `emitSincosBuiltin`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `emitSincosBuiltin`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 721-750
```cpp
721:   llvm::StoreInst *StoreCos =
722:       CGF.Builder.CreateStore(CosResult, CosLV.getAddress());
723: 
724:   // Mark the two stores as non-aliasing with each other. The order of stores
725:   // emitted by this builtin is arbitrary, enforcing a particular order will
726:   // prevent optimizations later on.
727:   llvm::MDBuilder MDHelper(CGF.getLLVMContext());
728:   MDNode *Domain = MDHelper.createAnonymousAliasScopeDomain();
729:   MDNode *AliasScope = MDHelper.createAnonymousAliasScope(Domain);
730:   MDNode *AliasScopeList = MDNode::get(Call->getContext(), AliasScope);
731:   StoreSin->setMetadata(LLVMContext::MD_alias_scope, AliasScopeList);
732:   StoreCos->setMetadata(LLVMContext::MD_noalias, AliasScopeList);
733: }
734: 
735: static llvm::Value *emitModfBuiltin(CodeGenFunction &CGF, const CallExpr *E,
736:                                     Intrinsic::ID IntrinsicID) {
737:   llvm::Value *Val = CGF.EmitScalarExpr(E->getArg(0));
738:   llvm::Value *IntPartDest = CGF.EmitScalarExpr(E->getArg(1));
739: 
740:   llvm::Value *Call =
741:       CGF.Builder.CreateIntrinsic(IntrinsicID, {Val->getType()}, Val);
742: 
743:   llvm::Value *FractionalResult = CGF.Builder.CreateExtractValue(Call, 0);
744:   llvm::Value *IntegralResult = CGF.Builder.CreateExtractValue(Call, 1);
745: 
746:   QualType DestPtrType = E->getArg(1)->getType()->getPointeeType();
747:   LValue IntegralLV = CGF.MakeNaturalAlignAddrLValue(IntPartDest, DestPtrType);
748:   CGF.EmitStoreOfScalar(IntegralResult, IntegralLV);
749: 
750:   return FractionalResult;
```
- **EN**: This block defines callable entry points like `MDHelper`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `MDHelper`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 751-780
```cpp
751: }
752: 
753: /// EmitFAbs - Emit a call to @llvm.fabs().
754: static Value *EmitFAbs(CodeGenFunction &CGF, Value *V) {
755:   llvm::CallInst *Call = CGF.Builder.CreateFAbs(V);
756:   Call->setDoesNotAccessMemory();
757:   return Call;
758: }
759: 
760: /// Emit the computation of the sign bit for a floating point value. Returns
761: /// the i1 sign bit value.
762: static Value *EmitSignBit(CodeGenFunction &CGF, Value *V) {
763:   LLVMContext &C = CGF.CGM.getLLVMContext();
764: 
765:   llvm::Type *Ty = V->getType();
766:   int Width = Ty->getPrimitiveSizeInBits();
767:   llvm::Type *IntTy = llvm::IntegerType::get(C, Width);
768:   V = CGF.Builder.CreateBitCast(V, IntTy);
769:   if (Ty->isPPC_FP128Ty()) {
770:     // We want the sign bit of the higher-order double. The bitcast we just
771:     // did works as if the double-double was stored to memory and then
772:     // read as an i128. The "store" will put the higher-order double in the
773:     // lower address in both little- and big-Endian modes, but the "load"
774:     // will treat those bits as a different part of the i128: the low bits in
775:     // little-Endian, the high bits in big-Endian. Therefore, on big-Endian
776:     // we need to shift the high bits down to the low before truncating.
777:     Width >>= 1;
778:     if (CGF.getTarget().isBigEndian()) {
779:       Value *ShiftCst = llvm::ConstantInt::get(IntTy, Width);
780:       V = CGF.Builder.CreateLShr(V, ShiftCst);
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 781-810
```cpp
781:     }
782:     // We are truncating value in order to extract the higher-order
783:     // double, which we will be using to extract the sign from.
784:     IntTy = llvm::IntegerType::get(C, Width);
785:     V = CGF.Builder.CreateTrunc(V, IntTy);
786:   }
787:   Value *Zero = llvm::Constant::getNullValue(IntTy);
788:   return CGF.Builder.CreateICmpSLT(V, Zero);
789: }
790: 
791: /// Checks no arguments or results are passed indirectly in the ABI (i.e. via a
792: /// hidden pointer). This is used to check annotating FP libcalls (that could
793: /// set `errno`) with "int" TBAA metadata is safe. If any floating-point
794: /// arguments are passed indirectly, setup for the call could be incorrectly
795: /// optimized out.
796: static bool HasNoIndirectArgumentsOrResults(CGFunctionInfo const &FnInfo) {
797:   auto IsIndirect = [&](ABIArgInfo const &info) {
798:     return info.isIndirect() || info.isIndirectAliased() || info.isInAlloca();
799:   };
800:   return !IsIndirect(FnInfo.getReturnInfo()) &&
801:          llvm::none_of(FnInfo.arguments(),
802:                        [&](CGFunctionInfoArgInfo const &ArgInfo) {
803:                          return IsIndirect(ArgInfo.info);
804:                        });
805: }
806: 
807: static RValue emitLibraryCall(CodeGenFunction &CGF, const FunctionDecl *FD,
808:                               const CallExpr *E, llvm::Constant *calleeValue) {
809:   CodeGenFunction::CGFPOptionsRAII FPOptsRAII(CGF, E);
810:   CGCallee callee = CGCallee::forDirect(calleeValue, GlobalDecl(FD));
```
- **EN**: This block defines callable entry points like `HasNoIndirectArgumentsOrResults`, `none_of`, `IsIndirect`, `emitLibraryCall`, `FPOptsRAII`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `HasNoIndirectArgumentsOrResults`, `none_of`, `IsIndirect`, `emitLibraryCall`, `FPOptsRAII`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 811-840
```cpp
811:   llvm::CallBase *callOrInvoke = nullptr;
812:   CGFunctionInfo const *FnInfo = nullptr;
813:   RValue Call =
814:       CGF.EmitCall(E->getCallee()->getType(), callee, E, ReturnValueSlot(),
815:                    /*Chain=*/nullptr, &callOrInvoke, &FnInfo);
816: 
817:   if (unsigned BuiltinID = FD->getBuiltinID()) {
818:     // Check whether a FP math builtin function, such as BI__builtin_expf
819:     ASTContext &Context = CGF.getContext();
820:     bool ConstWithoutErrnoAndExceptions =
821:         Context.BuiltinInfo.isConstWithoutErrnoAndExceptions(BuiltinID);
822:     // Restrict to target with errno, for example, MacOS doesn't set errno.
823:     // TODO: Support builtin function with complex type returned, eg: cacosh
824:     if (ConstWithoutErrnoAndExceptions && CGF.CGM.getLangOpts().MathErrno &&
825:         !CGF.Builder.getIsFPConstrained() && Call.isScalar() &&
826:         HasNoIndirectArgumentsOrResults(*FnInfo)) {
827:       // Emit "int" TBAA metadata on FP math libcalls.
828:       clang::QualType IntTy = Context.IntTy;
829:       TBAAAccessInfo TBAAInfo = CGF.CGM.getTBAAAccessInfo(IntTy);
830:       CGF.CGM.DecorateInstructionWithTBAA(callOrInvoke, TBAAInfo);
831:     }
832:   }
833:   return Call;
834: }
835: 
836: /// Emit a call to llvm.{sadd,uadd,ssub,usub,smul,umul}.with.overflow.*
837: /// depending on IntrinsicID.
838: ///
839: /// \arg CGF The current codegen function.
840: /// \arg IntrinsicID The ID for the Intrinsic we wish to generate.
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 841-870
```cpp
841: /// \arg X The first argument to the llvm.*.with.overflow.*.
842: /// \arg Y The second argument to the llvm.*.with.overflow.*.
843: /// \arg Carry The carry returned by the llvm.*.with.overflow.*.
844: /// \returns The result (i.e. sum/product) returned by the intrinsic.
845: llvm::Value *EmitOverflowIntrinsic(CodeGenFunction &CGF,
846:                                    const Intrinsic::ID IntrinsicID,
847:                                    llvm::Value *X, llvm::Value *Y,
848:                                    llvm::Value *&Carry) {
849:   // Make sure we have integers of the same width.
850:   assert(X->getType() == Y->getType() &&
851:          "Arguments must be the same type. (Did you forget to make sure both "
852:          "arguments have the same integer width?)");
853: 
854:   Function *Callee = CGF.CGM.getIntrinsic(IntrinsicID, X->getType());
855:   llvm::Value *Tmp = CGF.Builder.CreateCall(Callee, {X, Y});
856:   Carry = CGF.Builder.CreateExtractValue(Tmp, 1);
857:   return CGF.Builder.CreateExtractValue(Tmp, 0);
858: }
859: 
860: namespace {
861:   struct WidthAndSignedness {
862:     unsigned Width;
863:     bool Signed;
864:   };
865: }
866: 
867: static WidthAndSignedness
868: getIntegerWidthAndSignedness(const clang::ASTContext &context,
869:                              const clang::QualType Type) {
870:   assert(Type->isIntegerType() && "Given type is not an integer.");
```
- **EN**: This block introduces declarations such as `WidthAndSignedness`; defines callable entry points like `getIntegerWidthAndSignedness`; returns or forwards computed values for the surrounding LLVM IR emission logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `WidthAndSignedness` 的声明；定义可调用入口，例如 `getIntegerWidthAndSignedness`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 871-900
```cpp
871:   unsigned Width = context.getIntWidth(Type);
872:   bool Signed = Type->isSignedIntegerType();
873:   return {Width, Signed};
874: }
875: 
876: // Given one or more integer types, this function produces an integer type that
877: // encompasses them: any value in one of the given types could be expressed in
878: // the encompassing type.
879: static struct WidthAndSignedness
880: EncompassingIntegerType(ArrayRef<struct WidthAndSignedness> Types) {
881:   assert(Types.size() > 0 && "Empty list of types.");
882: 
883:   // If any of the given types is signed, we must return a signed type.
884:   bool Signed = false;
885:   for (const auto &Type : Types) {
886:     Signed |= Type.Signed;
887:   }
888: 
889:   // The encompassing type must have a width greater than or equal to the width
890:   // of the specified types.  Additionally, if the encompassing type is signed,
891:   // its width must be strictly greater than the width of any unsigned types
892:   // given.
893:   unsigned Width = 0;
894:   for (const auto &Type : Types) {
895:     unsigned MinWidth = Type.Width + (Signed && !Type.Signed);
896:     if (Width < MinWidth) {
897:       Width = MinWidth;
898:     }
899:   }
900: 
```
- **EN**: This block introduces declarations such as `WidthAndSignedness`; defines callable entry points like `EncompassingIntegerType`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `WidthAndSignedness` 的声明；定义可调用入口，例如 `EncompassingIntegerType`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 901-930
```cpp
901:   return {Width, Signed};
902: }
903: 
904: Value *CodeGenFunction::EmitVAStartEnd(Value *ArgValue, bool IsStart) {
905:   Intrinsic::ID inst = IsStart ? Intrinsic::vastart : Intrinsic::vaend;
906:   return Builder.CreateCall(CGM.getIntrinsic(inst, {ArgValue->getType()}),
907:                             ArgValue);
908: }
909: 
910: /// Checks if using the result of __builtin_object_size(p, @p From) in place of
911: /// __builtin_object_size(p, @p To) is correct
912: static bool areBOSTypesCompatible(int From, int To) {
913:   // Note: Our __builtin_object_size implementation currently treats Type=0 and
914:   // Type=2 identically. Encoding this implementation detail here may make
915:   // improving __builtin_object_size difficult in the future, so it's omitted.
916:   return From == To || (From == 0 && To == 1) || (From == 3 && To == 2);
917: }
918: 
919: static llvm::Value *
920: getDefaultBuiltinObjectSizeResult(unsigned Type, llvm::IntegerType *ResType) {
921:   return ConstantInt::get(ResType, (Type & 2) ? 0 : -1, /*isSigned=*/true);
922: }
923: 
924: llvm::Value *
925: CodeGenFunction::evaluateOrEmitBuiltinObjectSize(const Expr *E, unsigned Type,
926:                                                  llvm::IntegerType *ResType,
927:                                                  llvm::Value *EmittedE,
928:                                                  bool IsDynamic) {
929:   if (std::optional<uint64_t> ObjectSize =
930:           E->tryEvaluateObjectSize(getContext(), Type))
```
- **EN**: This block defines callable entry points like `areBOSTypesCompatible`, `getDefaultBuiltinObjectSizeResult`, `get`, `evaluateOrEmitBuiltinObjectSize`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `areBOSTypesCompatible`, `getDefaultBuiltinObjectSizeResult`, `get`, `evaluateOrEmitBuiltinObjectSize`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 931-960
```cpp
931:     return ConstantInt::get(ResType, *ObjectSize, /*isSigned=*/true);
932:   return emitBuiltinObjectSize(E, Type, ResType, EmittedE, IsDynamic);
933: }
934: 
935: namespace {
936: 
937: /// StructFieldAccess is a simple visitor class to grab the first MemberExpr
938: /// from an Expr. It records any ArraySubscriptExpr we meet along the way.
939: class StructFieldAccess
940:     : public ConstStmtVisitor<StructFieldAccess, const Expr *> {
941:   bool AddrOfSeen = false;
942: 
943: public:
944:   const Expr *ArrayIndex = nullptr;
945:   QualType ArrayElementTy;
946: 
947:   const Expr *VisitMemberExpr(const MemberExpr *E) {
948:     if (AddrOfSeen && E->getType()->isArrayType())
949:       // Avoid forms like '&ptr->array'.
950:       return nullptr;
951:     return E;
952:   }
953: 
954:   const Expr *VisitArraySubscriptExpr(const ArraySubscriptExpr *E) {
955:     if (ArrayIndex)
956:       // We don't support multiple subscripts.
957:       return nullptr;
958: 
959:     AddrOfSeen = false; // '&ptr->array[idx]' is okay.
960:     ArrayIndex = E->getIdx();
```
- **EN**: This block introduces declarations such as `StructFieldAccess`; defines callable entry points like `get`, `emitBuiltinObjectSize`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块给出诸如 `StructFieldAccess` 的声明；定义可调用入口，例如 `get`, `emitBuiltinObjectSize`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 961-990
```cpp
961:     ArrayElementTy = E->getBase()->getType();
962:     return Visit(E->getBase());
963:   }
964:   const Expr *VisitCastExpr(const CastExpr *E) {
965:     if (E->getCastKind() == CK_LValueToRValue)
966:       return E;
967:     return Visit(E->getSubExpr());
968:   }
969:   const Expr *VisitParenExpr(const ParenExpr *E) {
970:     return Visit(E->getSubExpr());
971:   }
972:   const Expr *VisitUnaryAddrOf(const clang::UnaryOperator *E) {
973:     AddrOfSeen = true;
974:     return Visit(E->getSubExpr());
975:   }
976:   const Expr *VisitUnaryDeref(const clang::UnaryOperator *E) {
977:     AddrOfSeen = false;
978:     return Visit(E->getSubExpr());
979:   }
980:   const Expr *VisitBinaryOperator(const clang::BinaryOperator *Op) {
981:     return Op->isCommaOp() ? Visit(Op->getRHS()) : nullptr;
982:   }
983: };
984: 
985: } // end anonymous namespace
986: 
987: /// Find a struct's flexible array member. It may be embedded inside multiple
988: /// sub-structs, but must still be the last field.
989: static const FieldDecl *FindFlexibleArrayMemberField(CodeGenFunction &CGF,
990:                                                      ASTContext &Ctx,
```
- **EN**: This block opens or references namespaces `static`; defines callable entry points like `Visit`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块打开或引用命名空间 `static`；定义可调用入口，例如 `Visit`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 991-1020
```cpp
 991:                                                      const RecordDecl *RD) {
 992:   const LangOptions::StrictFlexArraysLevelKind StrictFlexArraysLevel =
 993:       CGF.getLangOpts().getStrictFlexArraysLevel();
 994: 
 995:   if (RD->isImplicit())
 996:     return nullptr;
 997: 
 998:   for (const FieldDecl *FD : RD->fields()) {
 999:     if (Decl::isFlexibleArrayMemberLike(
1000:             Ctx, FD, FD->getType(), StrictFlexArraysLevel,
1001:             /*IgnoreTemplateOrMacroSubstitution=*/true))
1002:       return FD;
1003: 
1004:     if (const auto *RD = FD->getType()->getAsRecordDecl())
1005:       if (const FieldDecl *FD = FindFlexibleArrayMemberField(CGF, Ctx, RD))
1006:         return FD;
1007:   }
1008: 
1009:   return nullptr;
1010: }
1011: 
1012: /// Calculate the offset of a struct field. It may be embedded inside multiple
1013: /// sub-structs.
1014: static bool GetFieldOffset(ASTContext &Ctx, const RecordDecl *RD,
1015:                            const FieldDecl *FD, int64_t &Offset) {
1016:   if (RD->isImplicit())
1017:     return false;
1018: 
1019:   // Keep track of the field number ourselves, because the other methods
1020:   // (CGRecordLayout::getLLVMFieldNo) aren't always equivalent to how the AST
```
- **EN**: This block defines callable entry points like `GetFieldOffset`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `GetFieldOffset`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1021-1050
```cpp
1021:   // is laid out.
1022:   uint32_t FieldNo = 0;
1023:   const ASTRecordLayout &Layout = Ctx.getASTRecordLayout(RD);
1024: 
1025:   for (const FieldDecl *Field : RD->fields()) {
1026:     if (Field == FD) {
1027:       Offset += Layout.getFieldOffset(FieldNo);
1028:       return true;
1029:     }
1030: 
1031:     if (const auto *RD = Field->getType()->getAsRecordDecl()) {
1032:       if (GetFieldOffset(Ctx, RD, FD, Offset)) {
1033:         Offset += Layout.getFieldOffset(FieldNo);
1034:         return true;
1035:       }
1036:     }
1037: 
1038:     if (!RD->isUnion())
1039:       ++FieldNo;
1040:   }
1041: 
1042:   return false;
1043: }
1044: 
1045: static std::optional<int64_t>
1046: GetFieldOffset(ASTContext &Ctx, const RecordDecl *RD, const FieldDecl *FD) {
1047:   int64_t Offset = 0;
1048: 
1049:   if (GetFieldOffset(Ctx, RD, FD, Offset))
1050:     return std::optional<int64_t>(Offset);
```
- **EN**: This block defines callable entry points like `GetFieldOffset`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `GetFieldOffset`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1051-1080
```cpp
1051: 
1052:   return std::nullopt;
1053: }
1054: 
1055: llvm::Value *CodeGenFunction::emitCountedBySize(const Expr *E,
1056:                                                 llvm::Value *EmittedE,
1057:                                                 unsigned Type,
1058:                                                 llvm::IntegerType *ResType) {
1059:   // Note: If the whole struct is specificed in the __bdos (i.e. Visitor
1060:   // returns a DeclRefExpr). The calculation of the whole size of the structure
1061:   // with a flexible array member can be done in two ways:
1062:   //
1063:   //     1) sizeof(struct S) + count * sizeof(typeof(fam))
1064:   //     2) offsetof(struct S, fam) + count * sizeof(typeof(fam))
1065:   //
1066:   // The first will add additional padding after the end of the array
1067:   // allocation while the second method is more precise, but not quite expected
1068:   // from programmers. See
1069:   // https://lore.kernel.org/lkml/ZvV6X5FPBBW7CO1f@archlinux/ for a discussion
1070:   // of the topic.
1071:   //
1072:   // GCC isn't (currently) able to calculate __bdos on a pointer to the whole
1073:   // structure. Therefore, because of the above issue, we choose to match what
1074:   // GCC does for consistency's sake.
1075: 
1076:   StructFieldAccess Visitor;
1077:   E = Visitor.Visit(E);
1078:   if (!E)
1079:     return nullptr;
1080: 
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1081-1110
```cpp
1081:   const Expr *Idx = Visitor.ArrayIndex;
1082:   if (Idx) {
1083:     if (Idx->HasSideEffects(getContext()))
1084:       // We can't have side-effects.
1085:       return getDefaultBuiltinObjectSizeResult(Type, ResType);
1086: 
1087:     if (const auto *IL = dyn_cast<IntegerLiteral>(Idx)) {
1088:       int64_t Val = IL->getValue().getSExtValue();
1089:       if (Val < 0)
1090:         return getDefaultBuiltinObjectSizeResult(Type, ResType);
1091: 
1092:       // The index is 0, so we don't need to take it into account.
1093:       if (Val == 0)
1094:         Idx = nullptr;
1095:     }
1096:   }
1097: 
1098:   // __counted_by on either a flexible array member or a pointer into a struct
1099:   // with a flexible array member.
1100:   if (const auto *ME = dyn_cast<MemberExpr>(E))
1101:     return emitCountedByMemberSize(ME, Idx, EmittedE, Visitor.ArrayElementTy,
1102:                                    Type, ResType);
1103: 
1104:   // __counted_by on a pointer in a struct.
1105:   if (const auto *ICE = dyn_cast<ImplicitCastExpr>(E);
1106:       ICE && ICE->getCastKind() == CK_LValueToRValue)
1107:     return emitCountedByPointerSize(ICE, Idx, EmittedE, Visitor.ArrayElementTy,
1108:                                     Type, ResType);
1109: 
1110:   return nullptr;
```
- **EN**: This block defines callable entry points like `emitCountedByPointerSize`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitCountedByPointerSize`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1111-1140
```cpp
1111: }
1112: 
1113: static llvm::Value *EmitPositiveResultOrZero(CodeGenFunction &CGF,
1114:                                              llvm::Value *Res,
1115:                                              llvm::Value *Index,
1116:                                              llvm::IntegerType *ResType,
1117:                                              bool IsSigned) {
1118:   //  cmp = (array_size >= 0)
1119:   Value *Cmp = CGF.Builder.CreateIsNotNeg(Res);
1120:   if (Index)
1121:     //  cmp = (cmp && index >= 0)
1122:     Cmp = CGF.Builder.CreateAnd(CGF.Builder.CreateIsNotNeg(Index), Cmp);
1123: 
1124:   //  return cmp ? result : 0
1125:   return CGF.Builder.CreateSelect(Cmp, Res,
1126:                                   ConstantInt::get(ResType, 0, IsSigned));
1127: }
1128: 
1129: static std::pair<llvm::Value *, llvm::Value *>
1130: GetCountFieldAndIndex(CodeGenFunction &CGF, const MemberExpr *ME,
1131:                       const FieldDecl *ArrayFD, const FieldDecl *CountFD,
1132:                       const Expr *Idx, llvm::IntegerType *ResType,
1133:                       bool IsSigned) {
1134:   //  count = ptr->count;
1135:   Value *Count = CGF.EmitLoadOfCountedByField(ME, ArrayFD, CountFD);
1136:   if (!Count)
1137:     return std::make_pair<Value *>(nullptr, nullptr);
1138:   Count = CGF.Builder.CreateIntCast(Count, ResType, IsSigned, "count");
1139: 
1140:   //  index = ptr->index;
```
- **EN**: This block defines callable entry points like `get`, `GetCountFieldAndIndex`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `GetCountFieldAndIndex`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1141-1170
```cpp
1141:   Value *Index = nullptr;
1142:   if (Idx) {
1143:     bool IdxSigned = Idx->getType()->isSignedIntegerType();
1144:     Index = CGF.EmitScalarExpr(Idx);
1145:     Index = CGF.Builder.CreateIntCast(Index, ResType, IdxSigned, "index");
1146:   }
1147: 
1148:   return std::make_pair(Count, Index);
1149: }
1150: 
1151: llvm::Value *CodeGenFunction::emitCountedByPointerSize(
1152:     const ImplicitCastExpr *E, const Expr *Idx, llvm::Value *EmittedE,
1153:     QualType CastedArrayElementTy, unsigned Type, llvm::IntegerType *ResType) {
1154:   assert(E->getCastKind() == CK_LValueToRValue &&
1155:          "must be an LValue to RValue cast");
1156: 
1157:   const MemberExpr *ME =
1158:       dyn_cast<MemberExpr>(E->getSubExpr()->IgnoreParenNoopCasts(getContext()));
1159:   if (!ME)
1160:     return nullptr;
1161: 
1162:   const auto *ArrayBaseFD = dyn_cast<FieldDecl>(ME->getMemberDecl());
1163:   if (!ArrayBaseFD || !ArrayBaseFD->getType()->isPointerType() ||
1164:       !ArrayBaseFD->getType()->isCountAttributedType())
1165:     return nullptr;
1166: 
1167:   // Get the 'count' FieldDecl.
1168:   const FieldDecl *CountFD = ArrayBaseFD->findCountedByField();
1169:   if (!CountFD)
1170:     // Can't find the field referenced by the "counted_by" attribute.
```
- **EN**: This block defines callable entry points like `make_pair`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `make_pair`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1171-1200
```cpp
1171:     return nullptr;
1172: 
1173:   // Calculate the array's object size using these formulae. (Note: if the
1174:   // calculation is negative, we return 0.):
1175:   //
1176:   //      struct p;
1177:   //      struct s {
1178:   //          /* ... */
1179:   //          struct p **array __attribute__((counted_by(count)));
1180:   //          int count;
1181:   //      };
1182:   //
1183:   // 1) 'ptr->array':
1184:   //
1185:   //    count = ptr->count;
1186:   //
1187:   //    array_element_size = sizeof (*ptr->array);
1188:   //    array_size = count * array_element_size;
1189:   //
1190:   //    result = array_size;
1191:   //
1192:   //    cmp = (result >= 0)
1193:   //    return cmp ? result : 0;
1194:   //
1195:   // 2) '&((cast) ptr->array)[idx]':
1196:   //
1197:   //    count = ptr->count;
1198:   //    index = idx;
1199:   //
1200:   //    array_element_size = sizeof (*ptr->array);
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1201-1230
```cpp
1201:   //    array_size = count * array_element_size;
1202:   //
1203:   //    casted_array_element_size = sizeof (*((cast) ptr->array));
1204:   //
1205:   //    index_size = index * casted_array_element_size;
1206:   //    result = array_size - index_size;
1207:   //
1208:   //    cmp = (result >= 0)
1209:   //    if (index)
1210:   //        cmp  = (cmp && index > 0)
1211:   //    return cmp ? result : 0;
1212: 
1213:   auto GetElementBaseSize = [&](QualType ElementTy) {
1214:     CharUnits ElementSize =
1215:         getContext().getTypeSizeInChars(ElementTy->getPointeeType());
1216: 
1217:     if (ElementSize.isZero()) {
1218:       // This might be a __sized_by (or __counted_by) on a
1219:       // 'void *', which counts bytes, not elements.
1220:       [[maybe_unused]] auto *CAT = ElementTy->getAs<CountAttributedType>();
1221:       assert(CAT && "must have an CountAttributedType");
1222: 
1223:       ElementSize = CharUnits::One();
1224:     }
1225: 
1226:     return std::optional<CharUnits>(ElementSize);
1227:   };
1228: 
1229:   // Get the sizes of the original array element and the casted array element,
1230:   // if different.
```
- **EN**: This block defines callable entry points like `getContext`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getContext`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1231-1260
```cpp
1231:   std::optional<CharUnits> ArrayElementBaseSize =
1232:       GetElementBaseSize(ArrayBaseFD->getType());
1233:   if (!ArrayElementBaseSize)
1234:     return nullptr;
1235: 
1236:   std::optional<CharUnits> CastedArrayElementBaseSize = ArrayElementBaseSize;
1237:   if (!CastedArrayElementTy.isNull() && CastedArrayElementTy->isPointerType()) {
1238:     CastedArrayElementBaseSize = GetElementBaseSize(CastedArrayElementTy);
1239:     if (!CastedArrayElementBaseSize)
1240:       return nullptr;
1241:   }
1242: 
1243:   bool IsSigned = CountFD->getType()->isSignedIntegerType();
1244: 
1245:   //  count = ptr->count;
1246:   //  index = ptr->index;
1247:   Value *Count, *Index;
1248:   std::tie(Count, Index) = GetCountFieldAndIndex(
1249:       *this, ME, ArrayBaseFD, CountFD, Idx, ResType, IsSigned);
1250:   if (!Count)
1251:     return nullptr;
1252: 
1253:   //  array_element_size = sizeof (*ptr->array)
1254:   auto *ArrayElementSize = llvm::ConstantInt::get(
1255:       ResType, ArrayElementBaseSize->getQuantity(), IsSigned);
1256: 
1257:   //  casted_array_element_size = sizeof (*((cast) ptr->array));
1258:   auto *CastedArrayElementSize = llvm::ConstantInt::get(
1259:       ResType, CastedArrayElementBaseSize->getQuantity(), IsSigned);
1260: 
```
- **EN**: This block defines callable entry points like `GetElementBaseSize`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `GetElementBaseSize`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1261-1290
```cpp
1261:   //  array_size = count * array_element_size;
1262:   Value *ArraySize = Builder.CreateMul(Count, ArrayElementSize, "array_size",
1263:                                        !IsSigned, IsSigned);
1264: 
1265:   // Option (1) 'ptr->array'
1266:   //  result = array_size
1267:   Value *Result = ArraySize;
1268: 
1269:   if (Idx) { // Option (2) '&((cast) ptr->array)[idx]'
1270:     //  index_size = index * casted_array_element_size;
1271:     Value *IndexSize = Builder.CreateMul(Index, CastedArrayElementSize,
1272:                                          "index_size", !IsSigned, IsSigned);
1273: 
1274:     //  result = result - index_size;
1275:     Result =
1276:         Builder.CreateSub(Result, IndexSize, "result", !IsSigned, IsSigned);
1277:   }
1278: 
1279:   return EmitPositiveResultOrZero(*this, Result, Index, ResType, IsSigned);
1280: }
1281: 
1282: llvm::Value *CodeGenFunction::emitCountedByMemberSize(
1283:     const MemberExpr *ME, const Expr *Idx, llvm::Value *EmittedE,
1284:     QualType CastedArrayElementTy, unsigned Type, llvm::IntegerType *ResType) {
1285:   const auto *FD = dyn_cast<FieldDecl>(ME->getMemberDecl());
1286:   if (!FD)
1287:     return nullptr;
1288: 
1289:   // Find the flexible array member and check that it has the __counted_by
1290:   // attribute.
```
- **EN**: This block defines callable entry points like `EmitPositiveResultOrZero`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitPositiveResultOrZero`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1291-1320
```cpp
1291:   ASTContext &Ctx = getContext();
1292:   const RecordDecl *RD = FD->getDeclContext()->getOuterLexicalRecordContext();
1293:   const FieldDecl *FlexibleArrayMemberFD = nullptr;
1294: 
1295:   if (Decl::isFlexibleArrayMemberLike(
1296:           Ctx, FD, FD->getType(), getLangOpts().getStrictFlexArraysLevel(),
1297:           /*IgnoreTemplateOrMacroSubstitution=*/true))
1298:     FlexibleArrayMemberFD = FD;
1299:   else
1300:     FlexibleArrayMemberFD = FindFlexibleArrayMemberField(*this, Ctx, RD);
1301: 
1302:   if (!FlexibleArrayMemberFD ||
1303:       !FlexibleArrayMemberFD->getType()->isCountAttributedType())
1304:     return nullptr;
1305: 
1306:   // Get the 'count' FieldDecl.
1307:   const FieldDecl *CountFD = FlexibleArrayMemberFD->findCountedByField();
1308:   if (!CountFD)
1309:     // Can't find the field referenced by the "counted_by" attribute.
1310:     return nullptr;
1311: 
1312:   // Calculate the flexible array member's object size using these formulae.
1313:   // (Note: if the calculation is negative, we return 0.):
1314:   //
1315:   //      struct p;
1316:   //      struct s {
1317:   //          /* ... */
1318:   //          int count;
1319:   //          struct p *array[] __attribute__((counted_by(count)));
1320:   //      };
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1321-1350
```cpp
1321:   //
1322:   // 1) 'ptr->array':
1323:   //
1324:   //    count = ptr->count;
1325:   //
1326:   //    flexible_array_member_element_size = sizeof (*ptr->array);
1327:   //    flexible_array_member_size =
1328:   //        count * flexible_array_member_element_size;
1329:   //
1330:   //    result = flexible_array_member_size;
1331:   //
1332:   //    cmp = (result >= 0)
1333:   //    return cmp ? result : 0;
1334:   //
1335:   // 2) '&((cast) ptr->array)[idx]':
1336:   //
1337:   //    count = ptr->count;
1338:   //    index = idx;
1339:   //
1340:   //    flexible_array_member_element_size = sizeof (*ptr->array);
1341:   //    flexible_array_member_size =
1342:   //        count * flexible_array_member_element_size;
1343:   //
1344:   //    casted_flexible_array_member_element_size =
1345:   //        sizeof (*((cast) ptr->array));
1346:   //    index_size = index * casted_flexible_array_member_element_size;
1347:   //
1348:   //    result = flexible_array_member_size - index_size;
1349:   //
1350:   //    cmp = (result >= 0)
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1351-1380
```cpp
1351:   //    if (index != 0)
1352:   //        cmp = (cmp && index >= 0)
1353:   //    return cmp ? result : 0;
1354:   //
1355:   // 3) '&ptr->field':
1356:   //
1357:   //    count = ptr->count;
1358:   //    sizeof_struct = sizeof (struct s);
1359:   //
1360:   //    flexible_array_member_element_size = sizeof (*ptr->array);
1361:   //    flexible_array_member_size =
1362:   //        count * flexible_array_member_element_size;
1363:   //
1364:   //    field_offset = offsetof (struct s, field);
1365:   //    offset_diff = sizeof_struct - field_offset;
1366:   //
1367:   //    result = offset_diff + flexible_array_member_size;
1368:   //
1369:   //    cmp = (result >= 0)
1370:   //    return cmp ? result : 0;
1371:   //
1372:   // 4) '&((cast) ptr->field_array)[idx]':
1373:   //
1374:   //    count = ptr->count;
1375:   //    index = idx;
1376:   //    sizeof_struct = sizeof (struct s);
1377:   //
1378:   //    flexible_array_member_element_size = sizeof (*ptr->array);
1379:   //    flexible_array_member_size =
1380:   //        count * flexible_array_member_element_size;
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1381-1410
```cpp
1381:   //
1382:   //    casted_field_element_size = sizeof (*((cast) ptr->field_array));
1383:   //    field_offset = offsetof (struct s, field)
1384:   //    field_offset += index * casted_field_element_size;
1385:   //
1386:   //    offset_diff = sizeof_struct - field_offset;
1387:   //
1388:   //    result = offset_diff + flexible_array_member_size;
1389:   //
1390:   //    cmp = (result >= 0)
1391:   //    if (index != 0)
1392:   //        cmp = (cmp && index >= 0)
1393:   //    return cmp ? result : 0;
1394: 
1395:   bool IsSigned = CountFD->getType()->isSignedIntegerType();
1396: 
1397:   QualType FlexibleArrayMemberTy = FlexibleArrayMemberFD->getType();
1398: 
1399:   // Explicit cast because otherwise the CharWidth will promote an i32's into
1400:   // u64's leading to overflows.
1401:   int64_t CharWidth = static_cast<int64_t>(CGM.getContext().getCharWidth());
1402: 
1403:   //  field_offset = offsetof (struct s, field);
1404:   Value *FieldOffset = nullptr;
1405:   if (FlexibleArrayMemberFD != FD) {
1406:     std::optional<int64_t> Offset = GetFieldOffset(Ctx, RD, FD);
1407:     if (!Offset)
1408:       return nullptr;
1409:     FieldOffset =
1410:         llvm::ConstantInt::get(ResType, *Offset / CharWidth, IsSigned);
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1411-1440
```cpp
1411:   }
1412: 
1413:   //  count = ptr->count;
1414:   //  index = ptr->index;
1415:   Value *Count, *Index;
1416:   std::tie(Count, Index) = GetCountFieldAndIndex(
1417:       *this, ME, FlexibleArrayMemberFD, CountFD, Idx, ResType, IsSigned);
1418:   if (!Count)
1419:     return nullptr;
1420: 
1421:   //  flexible_array_member_element_size = sizeof (*ptr->array);
1422:   const ArrayType *ArrayTy = Ctx.getAsArrayType(FlexibleArrayMemberTy);
1423:   CharUnits BaseSize = Ctx.getTypeSizeInChars(ArrayTy->getElementType());
1424:   auto *FlexibleArrayMemberElementSize =
1425:       llvm::ConstantInt::get(ResType, BaseSize.getQuantity(), IsSigned);
1426: 
1427:   //  flexible_array_member_size = count * flexible_array_member_element_size;
1428:   Value *FlexibleArrayMemberSize =
1429:       Builder.CreateMul(Count, FlexibleArrayMemberElementSize,
1430:                         "flexible_array_member_size", !IsSigned, IsSigned);
1431: 
1432:   Value *Result = nullptr;
1433:   if (FlexibleArrayMemberFD == FD) {
1434:     if (Idx) { // Option (2) '&((cast) ptr->array)[idx]'
1435:       //  casted_flexible_array_member_element_size =
1436:       //      sizeof (*((cast) ptr->array));
1437:       llvm::ConstantInt *CastedFlexibleArrayMemberElementSize =
1438:           FlexibleArrayMemberElementSize;
1439:       if (!CastedArrayElementTy.isNull() &&
1440:           CastedArrayElementTy->isPointerType()) {
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1441-1470
```cpp
1441:         CharUnits BaseSize =
1442:             Ctx.getTypeSizeInChars(CastedArrayElementTy->getPointeeType());
1443:         CastedFlexibleArrayMemberElementSize =
1444:             llvm::ConstantInt::get(ResType, BaseSize.getQuantity(), IsSigned);
1445:       }
1446: 
1447:       //  index_size = index * casted_flexible_array_member_element_size;
1448:       Value *IndexSize =
1449:           Builder.CreateMul(Index, CastedFlexibleArrayMemberElementSize,
1450:                             "index_size", !IsSigned, IsSigned);
1451: 
1452:       //  result = flexible_array_member_size - index_size;
1453:       Result = Builder.CreateSub(FlexibleArrayMemberSize, IndexSize, "result",
1454:                                  !IsSigned, IsSigned);
1455:     } else { // Option (1) 'ptr->array'
1456:       //  result = flexible_array_member_size;
1457:       Result = FlexibleArrayMemberSize;
1458:     }
1459:   } else {
1460:     //  sizeof_struct = sizeof (struct s);
1461:     llvm::StructType *StructTy = getTypes().getCGRecordLayout(RD).getLLVMType();
1462:     const llvm::DataLayout &Layout = CGM.getDataLayout();
1463:     TypeSize Size = Layout.getTypeSizeInBits(StructTy);
1464:     Value *SizeofStruct =
1465:         llvm::ConstantInt::get(ResType, Size.getKnownMinValue() / CharWidth);
1466: 
1467:     if (Idx) { // Option (4) '&((cast) ptr->field_array)[idx]'
1468:       //  casted_field_element_size = sizeof (*((cast) ptr->field_array));
1469:       CharUnits BaseSize;
1470:       if (!CastedArrayElementTy.isNull() &&
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1471-1500
```cpp
1471:           CastedArrayElementTy->isPointerType()) {
1472:         BaseSize =
1473:             Ctx.getTypeSizeInChars(CastedArrayElementTy->getPointeeType());
1474:       } else {
1475:         const ArrayType *ArrayTy = Ctx.getAsArrayType(FD->getType());
1476:         BaseSize = Ctx.getTypeSizeInChars(ArrayTy->getElementType());
1477:       }
1478: 
1479:       llvm::ConstantInt *CastedFieldElementSize =
1480:           llvm::ConstantInt::get(ResType, BaseSize.getQuantity(), IsSigned);
1481: 
1482:       //  field_offset += index * casted_field_element_size;
1483:       Value *Mul = Builder.CreateMul(Index, CastedFieldElementSize,
1484:                                      "field_offset", !IsSigned, IsSigned);
1485:       FieldOffset = Builder.CreateAdd(FieldOffset, Mul);
1486:     }
1487:     // Option (3) '&ptr->field', and Option (4) continuation.
1488:     //  offset_diff = flexible_array_member_offset - field_offset;
1489:     Value *OffsetDiff = Builder.CreateSub(SizeofStruct, FieldOffset,
1490:                                           "offset_diff", !IsSigned, IsSigned);
1491: 
1492:     //  result = offset_diff + flexible_array_member_size;
1493:     Result = Builder.CreateAdd(FlexibleArrayMemberSize, OffsetDiff, "result");
1494:   }
1495: 
1496:   return EmitPositiveResultOrZero(*this, Result, Index, ResType, IsSigned);
1497: }
1498: 
1499: /// Returns a Value corresponding to the size of the given expression.
1500: /// This Value may be either of the following:
```
- **EN**: This block defines callable entry points like `get`, `EmitPositiveResultOrZero`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `get`, `EmitPositiveResultOrZero`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 1501-1530
```cpp
1501: ///   - A llvm::Argument (if E is a param with the pass_object_size attribute on
1502: ///     it)
1503: ///   - A call to the @llvm.objectsize intrinsic
1504: ///
1505: /// EmittedE is the result of emitting `E` as a scalar expr. If it's non-null
1506: /// and we wouldn't otherwise try to reference a pass_object_size parameter,
1507: /// we'll call @llvm.objectsize on EmittedE, rather than emitting E.
1508: llvm::Value *
1509: CodeGenFunction::emitBuiltinObjectSize(const Expr *E, unsigned Type,
1510:                                        llvm::IntegerType *ResType,
1511:                                        llvm::Value *EmittedE, bool IsDynamic) {
1512:   // We need to reference an argument if the pointer is a parameter with the
1513:   // pass_object_size attribute.
1514:   if (auto *D = dyn_cast<DeclRefExpr>(E->IgnoreParenImpCasts())) {
1515:     auto *Param = dyn_cast<ParmVarDecl>(D->getDecl());
1516:     auto *PS = D->getDecl()->getAttr<PassObjectSizeAttr>();
1517:     if (Param != nullptr && PS != nullptr &&
1518:         areBOSTypesCompatible(PS->getType(), Type)) {
1519:       auto Iter = SizeArguments.find(Param);
1520:       assert(Iter != SizeArguments.end());
1521: 
1522:       const ImplicitParamDecl *D = Iter->second;
1523:       auto DIter = LocalDeclMap.find(D);
1524:       assert(DIter != LocalDeclMap.end());
1525: 
1526:       return EmitLoadOfScalar(DIter->second, /*Volatile=*/false,
1527:                               getContext().getSizeType(), E->getBeginLoc());
1528:     }
1529:   }
1530: 
```
- **EN**: This block defines callable entry points like `emitBuiltinObjectSize`, `EmitLoadOfScalar`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitBuiltinObjectSize`, `EmitLoadOfScalar`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1531-1560
```cpp
1531:   // LLVM can't handle Type=3 appropriately, and __builtin_object_size shouldn't
1532:   // evaluate E for side-effects. In either case, we shouldn't lower to
1533:   // @llvm.objectsize.
1534:   if (Type == 3 || (!EmittedE && E->HasSideEffects(getContext())))
1535:     return getDefaultBuiltinObjectSizeResult(Type, ResType);
1536: 
1537:   Value *Ptr = EmittedE ? EmittedE : EmitScalarExpr(E);
1538:   assert(Ptr->getType()->isPointerTy() &&
1539:          "Non-pointer passed to __builtin_object_size?");
1540: 
1541:   if (IsDynamic)
1542:     // Emit special code for a flexible array member with the "counted_by"
1543:     // attribute.
1544:     if (Value *V = emitCountedBySize(E, Ptr, Type, ResType))
1545:       return V;
1546: 
1547:   Function *F =
1548:       CGM.getIntrinsic(Intrinsic::objectsize, {ResType, Ptr->getType()});
1549: 
1550:   // LLVM only supports 0 and 2, make sure that we pass along that as a boolean.
1551:   Value *Min = Builder.getInt1((Type & 2) != 0);
1552:   // For GCC compatibility, __builtin_object_size treat NULL as unknown size.
1553:   Value *NullIsUnknown = Builder.getTrue();
1554:   Value *Dynamic = Builder.getInt1(IsDynamic);
1555:   return Builder.CreateCall(F, {Ptr, Min, NullIsUnknown, Dynamic});
1556: }
1557: 
1558: namespace {
1559: /// A struct to generically describe a bit test intrinsic.
1560: struct BitTest {
```
- **EN**: This block introduces declarations such as `BitTest`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `BitTest` 的声明；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1561-1590
```cpp
1561:   enum ActionKind : uint8_t { TestOnly, Complement, Reset, Set };
1562:   enum InterlockingKind : uint8_t {
1563:     Unlocked,
1564:     Sequential,
1565:     Acquire,
1566:     Release,
1567:     NoFence
1568:   };
1569: 
1570:   ActionKind Action;
1571:   InterlockingKind Interlocking;
1572:   bool Is64Bit;
1573: 
1574:   static BitTest decodeBitTestBuiltin(unsigned BuiltinID);
1575: };
1576: 
1577: } // namespace
1578: 
1579: BitTest BitTest::decodeBitTestBuiltin(unsigned BuiltinID) {
1580:   switch (BuiltinID) {
1581:     // Main portable variants.
1582:   case Builtin::BI_bittest:
1583:     return {TestOnly, Unlocked, false};
1584:   case Builtin::BI_bittestandcomplement:
1585:     return {Complement, Unlocked, false};
1586:   case Builtin::BI_bittestandreset:
1587:     return {Reset, Unlocked, false};
1588:   case Builtin::BI_bittestandset:
1589:     return {Set, Unlocked, false};
1590:   case Builtin::BI_interlockedbittestandreset:
```
- **EN**: This block opens or references namespaces `BitTest`; introduces declarations such as `ActionKind`, `InterlockingKind`; defines callable entry points like `decodeBitTestBuiltin`; uses control flow (switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块打开或引用命名空间 `BitTest`；给出诸如 `ActionKind`, `InterlockingKind` 的声明；定义可调用入口，例如 `decodeBitTestBuiltin`；通过控制流（switch, case）细化 LLVM IR 生成 行为。

### Lines 1591-1620
```cpp
1591:     return {Reset, Sequential, false};
1592:   case Builtin::BI_interlockedbittestandset:
1593:     return {Set, Sequential, false};
1594: 
1595:     // 64-bit variants.
1596:   case Builtin::BI_bittest64:
1597:     return {TestOnly, Unlocked, true};
1598:   case Builtin::BI_bittestandcomplement64:
1599:     return {Complement, Unlocked, true};
1600:   case Builtin::BI_bittestandreset64:
1601:     return {Reset, Unlocked, true};
1602:   case Builtin::BI_bittestandset64:
1603:     return {Set, Unlocked, true};
1604:   case Builtin::BI_interlockedbittestandreset64:
1605:     return {Reset, Sequential, true};
1606:   case Builtin::BI_interlockedbittestandset64:
1607:     return {Set, Sequential, true};
1608: 
1609:     // ARM/AArch64-specific ordering variants.
1610:   case Builtin::BI_interlockedbittestandset_acq:
1611:     return {Set, Acquire, false};
1612:   case Builtin::BI_interlockedbittestandset_rel:
1613:     return {Set, Release, false};
1614:   case Builtin::BI_interlockedbittestandset_nf:
1615:     return {Set, NoFence, false};
1616:   case Builtin::BI_interlockedbittestandreset_acq:
1617:     return {Reset, Acquire, false};
1618:   case Builtin::BI_interlockedbittestandreset_rel:
1619:     return {Reset, Release, false};
1620:   case Builtin::BI_interlockedbittestandreset_nf:
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 1621-1650
```cpp
1621:     return {Reset, NoFence, false};
1622:   case Builtin::BI_interlockedbittestandreset64_acq:
1623:     return {Reset, Acquire, false};
1624:   case Builtin::BI_interlockedbittestandreset64_rel:
1625:     return {Reset, Release, false};
1626:   case Builtin::BI_interlockedbittestandreset64_nf:
1627:     return {Reset, NoFence, false};
1628:   case Builtin::BI_interlockedbittestandset64_acq:
1629:     return {Set, Acquire, false};
1630:   case Builtin::BI_interlockedbittestandset64_rel:
1631:     return {Set, Release, false};
1632:   case Builtin::BI_interlockedbittestandset64_nf:
1633:     return {Set, NoFence, false};
1634:   }
1635:   llvm_unreachable("expected only bittest intrinsics");
1636: }
1637: 
1638: static char bitActionToX86BTCode(BitTest::ActionKind A) {
1639:   switch (A) {
1640:   case BitTest::TestOnly:   return '\0';
1641:   case BitTest::Complement: return 'c';
1642:   case BitTest::Reset:      return 'r';
1643:   case BitTest::Set:        return 's';
1644:   }
1645:   llvm_unreachable("invalid action");
1646: }
1647: 
1648: static llvm::Value *EmitX86BitTestIntrinsic(CodeGenFunction &CGF,
1649:                                             BitTest BT,
1650:                                             const CallExpr *E, Value *BitBase,
```
- **EN**: This block defines callable entry points like `bitActionToX86BTCode`; uses control flow (switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `bitActionToX86BTCode`；通过控制流（switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1651-1680
```cpp
1651:                                             Value *BitPos) {
1652:   char Action = bitActionToX86BTCode(BT.Action);
1653:   char SizeSuffix = BT.Is64Bit ? 'q' : 'l';
1654: 
1655:   // Build the assembly.
1656:   SmallString<64> Asm;
1657:   raw_svector_ostream AsmOS(Asm);
1658:   if (BT.Interlocking != BitTest::Unlocked)
1659:     AsmOS << "lock ";
1660:   AsmOS << "bt";
1661:   if (Action)
1662:     AsmOS << Action;
1663:   AsmOS << SizeSuffix << " $2, ($1)";
1664: 
1665:   // Build the constraints. FIXME: We should support immediates when possible.
1666:   std::string Constraints = "={@ccc},r,r,~{cc},~{memory}";
1667:   std::string_view MachineClobbers = CGF.getTarget().getClobbers();
1668:   if (!MachineClobbers.empty()) {
1669:     Constraints += ',';
1670:     Constraints += MachineClobbers;
1671:   }
1672:   llvm::IntegerType *IntType = llvm::IntegerType::get(
1673:       CGF.getLLVMContext(),
1674:       CGF.getContext().getTypeSize(E->getArg(1)->getType()));
1675:   llvm::FunctionType *FTy =
1676:       llvm::FunctionType::get(CGF.Int8Ty, {CGF.DefaultPtrTy, IntType}, false);
1677: 
1678:   llvm::InlineAsm *IA =
1679:       llvm::InlineAsm::get(FTy, Asm, Constraints, /*hasSideEffects=*/true);
1680:   return CGF.Builder.CreateCall(IA, {BitBase, BitPos});
```
- **EN**: This block defines callable entry points like `AsmOS`, `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `AsmOS`, `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1681-1710
```cpp
1681: }
1682: 
1683: static llvm::AtomicOrdering
1684: getBitTestAtomicOrdering(BitTest::InterlockingKind I) {
1685:   switch (I) {
1686:   case BitTest::Unlocked:   return llvm::AtomicOrdering::NotAtomic;
1687:   case BitTest::Sequential: return llvm::AtomicOrdering::SequentiallyConsistent;
1688:   case BitTest::Acquire:    return llvm::AtomicOrdering::Acquire;
1689:   case BitTest::Release:    return llvm::AtomicOrdering::Release;
1690:   case BitTest::NoFence:    return llvm::AtomicOrdering::Monotonic;
1691:   }
1692:   llvm_unreachable("invalid interlocking");
1693: }
1694: 
1695: static llvm::Value *EmitBitCountExpr(CodeGenFunction &CGF, const Expr *E) {
1696:   llvm::Value *ArgValue = CGF.EmitScalarExpr(E);
1697:   llvm::Type *ArgType = ArgValue->getType();
1698: 
1699:   // Boolean vectors can be casted directly to its bitfield representation. We
1700:   // intentionally do not round up to the next power of two size and let LLVM
1701:   // handle the trailing bits.
1702:   if (auto *VT = dyn_cast<llvm::FixedVectorType>(ArgType);
1703:       VT && VT->getElementType()->isIntegerTy(1)) {
1704:     llvm::Type *StorageType =
1705:         llvm::Type::getIntNTy(CGF.getLLVMContext(), VT->getNumElements());
1706:     ArgValue = CGF.Builder.CreateBitCast(ArgValue, StorageType);
1707:   }
1708: 
1709:   return ArgValue;
1710: }
```
- **EN**: This block defines callable entry points like `getBitTestAtomicOrdering`, `getIntNTy`; uses control flow (if, switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getBitTestAtomicOrdering`, `getIntNTy`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1711-1740
```cpp
1711: 
1712: /// Emit a _bittest* intrinsic. These intrinsics take a pointer to an array of
1713: /// bits and a bit position and read and optionally modify the bit at that
1714: /// position. The position index can be arbitrarily large, i.e. it can be larger
1715: /// than 31 or 63, so we need an indexed load in the general case.
1716: static llvm::Value *EmitBitTestIntrinsic(CodeGenFunction &CGF,
1717:                                          unsigned BuiltinID,
1718:                                          const CallExpr *E) {
1719:   Value *BitBase = CGF.EmitScalarExpr(E->getArg(0));
1720:   Value *BitPos = CGF.EmitScalarExpr(E->getArg(1));
1721: 
1722:   BitTest BT = BitTest::decodeBitTestBuiltin(BuiltinID);
1723: 
1724:   // X86 has special BT, BTC, BTR, and BTS instructions that handle the array
1725:   // indexing operation internally. Use them if possible.
1726:   if (CGF.getTarget().getTriple().isX86())
1727:     return EmitX86BitTestIntrinsic(CGF, BT, E, BitBase, BitPos);
1728: 
1729:   // Otherwise, use generic code to load one byte and test the bit. Use all but
1730:   // the bottom three bits as the array index, and the bottom three bits to form
1731:   // a mask.
1732:   // Bit = BitBaseI8[BitPos >> 3] & (1 << (BitPos & 0x7)) != 0;
1733:   Value *ByteIndex = CGF.Builder.CreateAShr(
1734:       BitPos, llvm::ConstantInt::get(BitPos->getType(), 3), "bittest.byteidx");
1735:   Address ByteAddr(CGF.Builder.CreateInBoundsGEP(CGF.Int8Ty, BitBase, ByteIndex,
1736:                                                  "bittest.byteaddr"),
1737:                    CGF.Int8Ty, CharUnits::One());
1738:   Value *PosLow =
1739:       CGF.Builder.CreateAnd(CGF.Builder.CreateTrunc(BitPos, CGF.Int8Ty),
1740:                             llvm::ConstantInt::get(CGF.Int8Ty, 0x7));
```
- **EN**: This block defines callable entry points like `get`, `ByteAddr`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `ByteAddr`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1741-1770
```cpp
1741: 
1742:   // The updating instructions will need a mask.
1743:   Value *Mask = nullptr;
1744:   if (BT.Action != BitTest::TestOnly) {
1745:     Mask = CGF.Builder.CreateShl(llvm::ConstantInt::get(CGF.Int8Ty, 1), PosLow,
1746:                                  "bittest.mask");
1747:   }
1748: 
1749:   // Check the action and ordering of the interlocked intrinsics.
1750:   llvm::AtomicOrdering Ordering = getBitTestAtomicOrdering(BT.Interlocking);
1751: 
1752:   Value *OldByte = nullptr;
1753:   if (Ordering != llvm::AtomicOrdering::NotAtomic) {
1754:     // Emit a combined atomicrmw load/store operation for the interlocked
1755:     // intrinsics.
1756:     llvm::AtomicRMWInst::BinOp RMWOp = llvm::AtomicRMWInst::Or;
1757:     if (BT.Action == BitTest::Reset) {
1758:       Mask = CGF.Builder.CreateNot(Mask);
1759:       RMWOp = llvm::AtomicRMWInst::And;
1760:     }
1761:     OldByte = CGF.Builder.CreateAtomicRMW(RMWOp, ByteAddr, Mask, Ordering);
1762:   } else {
1763:     // Emit a plain load for the non-interlocked intrinsics.
1764:     OldByte = CGF.Builder.CreateLoad(ByteAddr, "bittest.byte");
1765:     Value *NewByte = nullptr;
1766:     switch (BT.Action) {
1767:     case BitTest::TestOnly:
1768:       // Don't store anything.
1769:       break;
1770:     case BitTest::Complement:
```
- **EN**: This block uses control flow (if, switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, switch, case）细化 LLVM IR 生成 行为。

### Lines 1771-1800
```cpp
1771:       NewByte = CGF.Builder.CreateXor(OldByte, Mask);
1772:       break;
1773:     case BitTest::Reset:
1774:       NewByte = CGF.Builder.CreateAnd(OldByte, CGF.Builder.CreateNot(Mask));
1775:       break;
1776:     case BitTest::Set:
1777:       NewByte = CGF.Builder.CreateOr(OldByte, Mask);
1778:       break;
1779:     }
1780:     if (NewByte)
1781:       CGF.Builder.CreateStore(NewByte, ByteAddr);
1782:   }
1783: 
1784:   // However we loaded the old byte, either by plain load or atomicrmw, shift
1785:   // the bit into the low position and mask it to 0 or 1.
1786:   Value *ShiftedByte = CGF.Builder.CreateLShr(OldByte, PosLow, "bittest.shr");
1787:   return CGF.Builder.CreateAnd(
1788:       ShiftedByte, llvm::ConstantInt::get(CGF.Int8Ty, 1), "bittest.res");
1789: }
1790: 
1791: namespace {
1792: enum class MSVCSetJmpKind {
1793:   _setjmpex,
1794:   _setjmp3,
1795:   _setjmp
1796: };
1797: }
1798: 
1799: /// MSVC handles setjmp a bit differently on different platforms. On every
1800: /// architecture except 32-bit x86, the frame address is passed. On x86, extra
```
- **EN**: This block introduces declarations such as `MSVCSetJmpKind`; defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块给出诸如 `MSVCSetJmpKind` 的声明；定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 1801-1830
```cpp
1801: /// parameters can be passed as variadic arguments, but we always pass none.
1802: static RValue EmitMSVCRTSetJmp(CodeGenFunction &CGF, MSVCSetJmpKind SJKind,
1803:                                const CallExpr *E) {
1804:   llvm::Value *Arg1 = nullptr;
1805:   llvm::Type *Arg1Ty = nullptr;
1806:   StringRef Name;
1807:   bool IsVarArg = false;
1808:   if (SJKind == MSVCSetJmpKind::_setjmp3) {
1809:     Name = "_setjmp3";
1810:     Arg1Ty = CGF.Int32Ty;
1811:     Arg1 = llvm::ConstantInt::get(CGF.IntTy, 0);
1812:     IsVarArg = true;
1813:   } else {
1814:     Name = SJKind == MSVCSetJmpKind::_setjmp ? "_setjmp" : "_setjmpex";
1815:     Arg1Ty = CGF.Int8PtrTy;
1816:     if (CGF.getTarget().getTriple().getArch() == llvm::Triple::aarch64) {
1817:       Arg1 = CGF.Builder.CreateCall(
1818:           CGF.CGM.getIntrinsic(Intrinsic::sponentry, CGF.AllocaInt8PtrTy));
1819:     } else
1820:       Arg1 = CGF.Builder.CreateCall(
1821:           CGF.CGM.getIntrinsic(Intrinsic::frameaddress, CGF.AllocaInt8PtrTy),
1822:           llvm::ConstantInt::get(CGF.Int32Ty, 0));
1823:   }
1824: 
1825:   // Mark the call site and declaration with ReturnsTwice.
1826:   llvm::Type *ArgTypes[2] = {CGF.Int8PtrTy, Arg1Ty};
1827:   llvm::AttributeList ReturnsTwiceAttr = llvm::AttributeList::get(
1828:       CGF.getLLVMContext(), llvm::AttributeList::FunctionIndex,
1829:       llvm::Attribute::ReturnsTwice);
1830:   llvm::FunctionCallee SetJmpFn = CGF.CGM.CreateRuntimeFunction(
```
- **EN**: This block defines callable entry points like `EmitMSVCRTSetJmp`, `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitMSVCRTSetJmp`, `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1831-1860
```cpp
1831:       llvm::FunctionType::get(CGF.IntTy, ArgTypes, IsVarArg), Name,
1832:       ReturnsTwiceAttr, /*Local=*/true);
1833: 
1834:   llvm::Value *Buf = CGF.Builder.CreateBitOrPointerCast(
1835:       CGF.EmitScalarExpr(E->getArg(0)), CGF.Int8PtrTy);
1836:   llvm::Value *Args[] = {Buf, Arg1};
1837:   llvm::CallBase *CB = CGF.EmitRuntimeCallOrInvoke(SetJmpFn, Args);
1838:   CB->setAttributes(ReturnsTwiceAttr);
1839:   return RValue::get(CB);
1840: }
1841: 
1842: // Emit an MSVC intrinsic. Assumes that arguments have *not* been evaluated.
1843: Value *CodeGenFunction::EmitMSVCBuiltinExpr(MSVCIntrin BuiltinID,
1844:                                             const CallExpr *E) {
1845:   switch (BuiltinID) {
1846:   case MSVCIntrin::_BitScanForward:
1847:   case MSVCIntrin::_BitScanReverse: {
1848:     Address IndexAddress(EmitPointerWithAlignment(E->getArg(0)));
1849:     Value *ArgValue = EmitScalarExpr(E->getArg(1));
1850: 
1851:     llvm::Type *ArgType = ArgValue->getType();
1852:     llvm::Type *IndexType = IndexAddress.getElementType();
1853:     llvm::Type *ResultType = ConvertType(E->getType());
1854: 
1855:     Value *ArgZero = llvm::Constant::getNullValue(ArgType);
1856:     Value *ResZero = llvm::Constant::getNullValue(ResultType);
1857:     Value *ResOne = llvm::ConstantInt::get(ResultType, 1);
1858: 
1859:     BasicBlock *Begin = Builder.GetInsertBlock();
1860:     BasicBlock *End = createBasicBlock("bitscan_end", this->CurFn);
```
- **EN**: This block defines callable entry points like `get`, `IndexAddress`; uses control flow (switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `IndexAddress`；通过控制流（switch, case）细化 LLVM IR 生成 行为。

### Lines 1861-1890
```cpp
1861:     Builder.SetInsertPoint(End);
1862:     PHINode *Result = Builder.CreatePHI(ResultType, 2, "bitscan_result");
1863: 
1864:     Builder.SetInsertPoint(Begin);
1865:     Value *IsZero = Builder.CreateICmpEQ(ArgValue, ArgZero);
1866:     BasicBlock *NotZero = createBasicBlock("bitscan_not_zero", this->CurFn);
1867:     Builder.CreateCondBr(IsZero, End, NotZero);
1868:     Result->addIncoming(ResZero, Begin);
1869: 
1870:     Builder.SetInsertPoint(NotZero);
1871: 
1872:     if (BuiltinID == MSVCIntrin::_BitScanForward) {
1873:       Function *F = CGM.getIntrinsic(Intrinsic::cttz, ArgType);
1874:       Value *ZeroCount = Builder.CreateCall(F, {ArgValue, Builder.getTrue()});
1875:       ZeroCount = Builder.CreateIntCast(ZeroCount, IndexType, false);
1876:       Builder.CreateStore(ZeroCount, IndexAddress, false);
1877:     } else {
1878:       unsigned ArgWidth = cast<llvm::IntegerType>(ArgType)->getBitWidth();
1879:       Value *ArgTypeLastIndex = llvm::ConstantInt::get(IndexType, ArgWidth - 1);
1880: 
1881:       Function *F = CGM.getIntrinsic(Intrinsic::ctlz, ArgType);
1882:       Value *ZeroCount = Builder.CreateCall(F, {ArgValue, Builder.getTrue()});
1883:       ZeroCount = Builder.CreateIntCast(ZeroCount, IndexType, false);
1884:       Value *Index = Builder.CreateNSWSub(ArgTypeLastIndex, ZeroCount);
1885:       Builder.CreateStore(Index, IndexAddress, false);
1886:     }
1887:     Builder.CreateBr(End);
1888:     Result->addIncoming(ResOne, NotZero);
1889: 
1890:     Builder.SetInsertPoint(End);
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1891-1920
```cpp
1891:     return Result;
1892:   }
1893:   case MSVCIntrin::_InterlockedAnd:
1894:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::And, E);
1895:   case MSVCIntrin::_InterlockedExchange:
1896:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Xchg, E);
1897:   case MSVCIntrin::_InterlockedExchangeAdd:
1898:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Add, E);
1899:   case MSVCIntrin::_InterlockedExchangeSub:
1900:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Sub, E);
1901:   case MSVCIntrin::_InterlockedOr:
1902:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Or, E);
1903:   case MSVCIntrin::_InterlockedXor:
1904:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Xor, E);
1905:   case MSVCIntrin::_InterlockedExchangeAdd_acq:
1906:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Add, E,
1907:                                  AtomicOrdering::Acquire);
1908:   case MSVCIntrin::_InterlockedExchangeAdd_rel:
1909:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Add, E,
1910:                                  AtomicOrdering::Release);
1911:   case MSVCIntrin::_InterlockedExchangeAdd_nf:
1912:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Add, E,
1913:                                  AtomicOrdering::Monotonic);
1914:   case MSVCIntrin::_InterlockedExchange_acq:
1915:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Xchg, E,
1916:                                  AtomicOrdering::Acquire);
1917:   case MSVCIntrin::_InterlockedExchange_rel:
1918:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Xchg, E,
1919:                                  AtomicOrdering::Release);
1920:   case MSVCIntrin::_InterlockedExchange_nf:
```
- **EN**: This block spells out callable entry points like `MakeBinaryAtomicValue`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeBinaryAtomicValue`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 1921-1950
```cpp
1921:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Xchg, E,
1922:                                  AtomicOrdering::Monotonic);
1923:   case MSVCIntrin::_InterlockedCompareExchange:
1924:     return EmitAtomicCmpXchgForMSIntrin(*this, E);
1925:   case MSVCIntrin::_InterlockedCompareExchange_acq:
1926:     return EmitAtomicCmpXchgForMSIntrin(*this, E, AtomicOrdering::Acquire);
1927:   case MSVCIntrin::_InterlockedCompareExchange_rel:
1928:     return EmitAtomicCmpXchgForMSIntrin(*this, E, AtomicOrdering::Release);
1929:   case MSVCIntrin::_InterlockedCompareExchange_nf:
1930:     return EmitAtomicCmpXchgForMSIntrin(*this, E, AtomicOrdering::Monotonic);
1931:   case MSVCIntrin::_InterlockedCompareExchange128:
1932:     return EmitAtomicCmpXchg128ForMSIntrin(
1933:         *this, E, AtomicOrdering::SequentiallyConsistent);
1934:   case MSVCIntrin::_InterlockedCompareExchange128_acq:
1935:     return EmitAtomicCmpXchg128ForMSIntrin(*this, E, AtomicOrdering::Acquire);
1936:   case MSVCIntrin::_InterlockedCompareExchange128_rel:
1937:     return EmitAtomicCmpXchg128ForMSIntrin(*this, E, AtomicOrdering::Release);
1938:   case MSVCIntrin::_InterlockedCompareExchange128_nf:
1939:     return EmitAtomicCmpXchg128ForMSIntrin(*this, E, AtomicOrdering::Monotonic);
1940:   case MSVCIntrin::_InterlockedOr_acq:
1941:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Or, E,
1942:                                  AtomicOrdering::Acquire);
1943:   case MSVCIntrin::_InterlockedOr_rel:
1944:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Or, E,
1945:                                  AtomicOrdering::Release);
1946:   case MSVCIntrin::_InterlockedOr_nf:
1947:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Or, E,
1948:                                  AtomicOrdering::Monotonic);
1949:   case MSVCIntrin::_InterlockedXor_acq:
1950:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Xor, E,
```
- **EN**: This block spells out callable entry points like `MakeBinaryAtomicValue`, `EmitAtomicCmpXchgForMSIntrin`, `EmitAtomicCmpXchg128ForMSIntrin`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeBinaryAtomicValue`, `EmitAtomicCmpXchgForMSIntrin`, `EmitAtomicCmpXchg128ForMSIntrin`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 1951-1980
```cpp
1951:                                  AtomicOrdering::Acquire);
1952:   case MSVCIntrin::_InterlockedXor_rel:
1953:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Xor, E,
1954:                                  AtomicOrdering::Release);
1955:   case MSVCIntrin::_InterlockedXor_nf:
1956:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::Xor, E,
1957:                                  AtomicOrdering::Monotonic);
1958:   case MSVCIntrin::_InterlockedAnd_acq:
1959:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::And, E,
1960:                                  AtomicOrdering::Acquire);
1961:   case MSVCIntrin::_InterlockedAnd_rel:
1962:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::And, E,
1963:                                  AtomicOrdering::Release);
1964:   case MSVCIntrin::_InterlockedAnd_nf:
1965:     return MakeBinaryAtomicValue(*this, AtomicRMWInst::And, E,
1966:                                  AtomicOrdering::Monotonic);
1967:   case MSVCIntrin::_InterlockedIncrement_acq:
1968:     return EmitAtomicIncrementValue(*this, E, AtomicOrdering::Acquire);
1969:   case MSVCIntrin::_InterlockedIncrement_rel:
1970:     return EmitAtomicIncrementValue(*this, E, AtomicOrdering::Release);
1971:   case MSVCIntrin::_InterlockedIncrement_nf:
1972:     return EmitAtomicIncrementValue(*this, E, AtomicOrdering::Monotonic);
1973:   case MSVCIntrin::_InterlockedDecrement_acq:
1974:     return EmitAtomicDecrementValue(*this, E, AtomicOrdering::Acquire);
1975:   case MSVCIntrin::_InterlockedDecrement_rel:
1976:     return EmitAtomicDecrementValue(*this, E, AtomicOrdering::Release);
1977:   case MSVCIntrin::_InterlockedDecrement_nf:
1978:     return EmitAtomicDecrementValue(*this, E, AtomicOrdering::Monotonic);
1979: 
1980:   case MSVCIntrin::_InterlockedDecrement:
```
- **EN**: This block spells out callable entry points like `MakeBinaryAtomicValue`, `EmitAtomicIncrementValue`, `EmitAtomicDecrementValue`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeBinaryAtomicValue`, `EmitAtomicIncrementValue`, `EmitAtomicDecrementValue`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 1981-2010
```cpp
1981:     return EmitAtomicDecrementValue(*this, E);
1982:   case MSVCIntrin::_InterlockedIncrement:
1983:     return EmitAtomicIncrementValue(*this, E);
1984: 
1985:   case MSVCIntrin::__fastfail: {
1986:     // Request immediate process termination from the kernel. The instruction
1987:     // sequences to do this are documented on MSDN:
1988:     // https://msdn.microsoft.com/en-us/library/dn774154.aspx
1989:     llvm::Triple::ArchType ISA = getTarget().getTriple().getArch();
1990:     StringRef Asm, Constraints;
1991:     switch (ISA) {
1992:     default:
1993:       ErrorUnsupported(E, "__fastfail call for this architecture");
1994:       break;
1995:     case llvm::Triple::x86:
1996:     case llvm::Triple::x86_64:
1997:       Asm = "int $$0x29";
1998:       Constraints = "{cx}";
1999:       break;
2000:     case llvm::Triple::thumb:
2001:       Asm = "udf #251";
2002:       Constraints = "{r0}";
2003:       break;
2004:     case llvm::Triple::aarch64:
2005:       Asm = "brk #0xF003";
2006:       Constraints = "{w0}";
2007:     }
2008:     llvm::FunctionType *FTy = llvm::FunctionType::get(VoidTy, {Int32Ty}, false);
2009:     llvm::InlineAsm *IA =
2010:         llvm::InlineAsm::get(FTy, Asm, Constraints, /*hasSideEffects=*/true);
```
- **EN**: This block defines callable entry points like `EmitAtomicDecrementValue`, `EmitAtomicIncrementValue`, `ErrorUnsupported`, `get`; uses control flow (switch, for, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitAtomicDecrementValue`, `EmitAtomicIncrementValue`, `ErrorUnsupported`, `get`；通过控制流（switch, for, case）细化 LLVM IR 生成 行为。

### Lines 2011-2040
```cpp
2011:     llvm::AttributeList NoReturnAttr = llvm::AttributeList::get(
2012:         getLLVMContext(), llvm::AttributeList::FunctionIndex,
2013:         llvm::Attribute::NoReturn);
2014:     llvm::CallInst *CI = Builder.CreateCall(IA, EmitScalarExpr(E->getArg(0)));
2015:     CI->setAttributes(NoReturnAttr);
2016:     return CI;
2017:   }
2018:   }
2019:   llvm_unreachable("Incorrect MSVC intrinsic!");
2020: }
2021: 
2022: namespace {
2023: // ARC cleanup for __builtin_os_log_format
2024: struct CallObjCArcUse final : EHScopeStack::Cleanup {
2025:   CallObjCArcUse(llvm::Value *object) : object(object) {}
2026:   llvm::Value *object;
2027: 
2028:   void Emit(CodeGenFunction &CGF, Flags flags) override {
2029:     CGF.EmitARCIntrinsicUse(object);
2030:   }
2031: };
2032: }
2033: 
2034: Value *CodeGenFunction::EmitCheckedArgForBuiltin(const Expr *E,
2035:                                                  BuiltinCheckKind Kind) {
2036:   assert((Kind == BCK_CLZPassedZero || Kind == BCK_CTZPassedZero) &&
2037:          "Unsupported builtin check kind");
2038: 
2039:   Value *ArgValue = EmitBitCountExpr(*this, E);
2040:   if (!SanOpts.has(SanitizerKind::Builtin))
```
- **EN**: This block introduces declarations such as `CallObjCArcUse`; defines callable entry points like `getLLVMContext`, `CallObjCArcUse`, `Emit`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `CallObjCArcUse` 的声明；定义可调用入口，例如 `getLLVMContext`, `CallObjCArcUse`, `Emit`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 2041-2070
```cpp
2041:     return ArgValue;
2042: 
2043:   auto CheckOrdinal = SanitizerKind::SO_Builtin;
2044:   auto CheckHandler = SanitizerHandler::InvalidBuiltin;
2045:   SanitizerDebugLocation SanScope(this, {CheckOrdinal}, CheckHandler);
2046:   Value *Cond = Builder.CreateICmpNE(
2047:       ArgValue, llvm::Constant::getNullValue(ArgValue->getType()));
2048:   EmitCheck(std::make_pair(Cond, CheckOrdinal), CheckHandler,
2049:             {EmitCheckSourceLocation(E->getExprLoc()),
2050:              llvm::ConstantInt::get(Builder.getInt8Ty(), Kind)},
2051:             {});
2052:   return ArgValue;
2053: }
2054: 
2055: Value *CodeGenFunction::EmitCheckedArgForAssume(const Expr *E) {
2056:   Value *ArgValue = EvaluateExprAsBool(E);
2057:   if (!SanOpts.has(SanitizerKind::Builtin))
2058:     return ArgValue;
2059: 
2060:   auto CheckOrdinal = SanitizerKind::SO_Builtin;
2061:   auto CheckHandler = SanitizerHandler::InvalidBuiltin;
2062:   SanitizerDebugLocation SanScope(this, {CheckOrdinal}, CheckHandler);
2063:   EmitCheck(
2064:       std::make_pair(ArgValue, CheckOrdinal), CheckHandler,
2065:       {EmitCheckSourceLocation(E->getExprLoc()),
2066:        llvm::ConstantInt::get(Builder.getInt8Ty(), BCK_AssumePassedFalse)},
2067:       {});
2068:   return ArgValue;
2069: }
2070: 
```
- **EN**: This block defines callable entry points like `getNullValue`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getNullValue`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 2071-2100
```cpp
2071: static Value *EmitAbs(CodeGenFunction &CGF, Value *ArgValue, bool HasNSW) {
2072:   return CGF.Builder.CreateBinaryIntrinsic(
2073:       Intrinsic::abs, ArgValue,
2074:       ConstantInt::get(CGF.Builder.getInt1Ty(), HasNSW));
2075: }
2076: 
2077: static Value *EmitOverflowCheckedAbs(CodeGenFunction &CGF, const CallExpr *E,
2078:                                      bool SanitizeOverflow) {
2079:   Value *ArgValue = CGF.EmitScalarExpr(E->getArg(0));
2080: 
2081:   // Try to eliminate overflow check.
2082:   if (const auto *VCI = dyn_cast<llvm::ConstantInt>(ArgValue)) {
2083:     if (!VCI->isMinSignedValue())
2084:       return EmitAbs(CGF, ArgValue, true);
2085:   }
2086: 
2087:   SmallVector<SanitizerKind::SanitizerOrdinal, 1> Ordinals;
2088:   SanitizerHandler CheckHandler;
2089:   if (SanitizeOverflow) {
2090:     Ordinals.push_back(SanitizerKind::SO_SignedIntegerOverflow);
2091:     CheckHandler = SanitizerHandler::NegateOverflow;
2092:   } else
2093:     CheckHandler = SanitizerHandler::SubOverflow;
2094: 
2095:   SanitizerDebugLocation SanScope(&CGF, Ordinals, CheckHandler);
2096: 
2097:   Constant *Zero = Constant::getNullValue(ArgValue->getType());
2098:   Value *ResultAndOverflow = CGF.Builder.CreateBinaryIntrinsic(
2099:       Intrinsic::ssub_with_overflow, Zero, ArgValue);
2100:   Value *Result = CGF.Builder.CreateExtractValue(ResultAndOverflow, 0);
```
- **EN**: This block defines callable entry points like `get`, `SanScope`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `SanScope`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 2101-2130
```cpp
2101:   Value *NotOverflow = CGF.Builder.CreateNot(
2102:       CGF.Builder.CreateExtractValue(ResultAndOverflow, 1));
2103: 
2104:   // TODO: support -ftrapv-handler.
2105:   if (SanitizeOverflow) {
2106:     CGF.EmitCheck({{NotOverflow, SanitizerKind::SO_SignedIntegerOverflow}},
2107:                   CheckHandler,
2108:                   {CGF.EmitCheckSourceLocation(E->getArg(0)->getExprLoc()),
2109:                    CGF.EmitCheckTypeDescriptor(E->getType())},
2110:                   {ArgValue});
2111:   } else
2112:     CGF.EmitTrapCheck(NotOverflow, CheckHandler);
2113: 
2114:   Value *CmpResult = CGF.Builder.CreateICmpSLT(ArgValue, Zero, "abscond");
2115:   return CGF.Builder.CreateSelect(CmpResult, Result, ArgValue, "abs");
2116: }
2117: 
2118: /// Get the argument type for arguments to os_log_helper.
2119: static CanQualType getOSLogArgType(ASTContext &C, int Size) {
2120:   QualType UnsignedTy = C.getIntTypeForBitwidth(Size * 8, /*Signed=*/false);
2121:   return C.getCanonicalType(UnsignedTy);
2122: }
2123: 
2124: llvm::Function *CodeGenFunction::generateBuiltinOSLogHelperFunction(
2125:     const analyze_os_log::OSLogBufferLayout &Layout,
2126:     CharUnits BufferAlignment) {
2127:   ASTContext &Ctx = getContext();
2128: 
2129:   llvm::SmallString<64> Name;
2130:   {
```
- **EN**: This block defines callable entry points like `getOSLogArgType`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getOSLogArgType`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 2131-2160
```cpp
2131:     raw_svector_ostream OS(Name);
2132:     OS << "__os_log_helper";
2133:     OS << "_" << BufferAlignment.getQuantity();
2134:     OS << "_" << int(Layout.getSummaryByte());
2135:     OS << "_" << int(Layout.getNumArgsByte());
2136:     for (const auto &Item : Layout.Items)
2137:       OS << "_" << int(Item.getSizeByte()) << "_"
2138:          << int(Item.getDescriptorByte());
2139:   }
2140: 
2141:   if (llvm::Function *F = CGM.getModule().getFunction(Name))
2142:     return F;
2143: 
2144:   llvm::SmallVector<QualType, 4> ArgTys;
2145:   FunctionArgList Args;
2146:   Args.push_back(ImplicitParamDecl::Create(
2147:       Ctx, nullptr, SourceLocation(), &Ctx.Idents.get("buffer"), Ctx.VoidPtrTy,
2148:       ImplicitParamKind::Other));
2149:   ArgTys.emplace_back(Ctx.VoidPtrTy);
2150: 
2151:   for (unsigned int I = 0, E = Layout.Items.size(); I < E; ++I) {
2152:     char Size = Layout.Items[I].getSizeByte();
2153:     if (!Size)
2154:       continue;
2155: 
2156:     QualType ArgTy = getOSLogArgType(Ctx, Size);
2157:     Args.push_back(ImplicitParamDecl::Create(
2158:         Ctx, nullptr, SourceLocation(),
2159:         &Ctx.Idents.get(std::string("arg") + llvm::to_string(I)), ArgTy,
2160:         ImplicitParamKind::Other));
```
- **EN**: This block defines callable entry points like `OS`, `SourceLocation`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `OS`, `SourceLocation`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 2161-2190
```cpp
2161:     ArgTys.emplace_back(ArgTy);
2162:   }
2163: 
2164:   QualType ReturnTy = Ctx.VoidTy;
2165: 
2166:   // The helper function has linkonce_odr linkage to enable the linker to merge
2167:   // identical functions. To ensure the merging always happens, 'noinline' is
2168:   // attached to the function when compiling with -Oz.
2169:   const CGFunctionInfo &FI =
2170:       CGM.getTypes().arrangeBuiltinFunctionDeclaration(ReturnTy, Args);
2171:   llvm::FunctionType *FuncTy = CGM.getTypes().GetFunctionType(FI);
2172:   llvm::Function *Fn = llvm::Function::Create(
2173:       FuncTy, llvm::GlobalValue::LinkOnceODRLinkage, Name, &CGM.getModule());
2174:   Fn->setVisibility(llvm::GlobalValue::HiddenVisibility);
2175:   CGM.SetLLVMFunctionAttributes(GlobalDecl(), FI, Fn, /*IsThunk=*/false);
2176:   CGM.SetLLVMFunctionAttributesForDefinition(nullptr, Fn);
2177:   Fn->setDoesNotThrow();
2178: 
2179:   // Attach 'noinline' at -Oz.
2180:   if (CGM.getCodeGenOpts().OptimizeSize == 2)
2181:     Fn->addFnAttr(llvm::Attribute::NoInline);
2182: 
2183:   auto NL = ApplyDebugLocation::CreateEmpty(*this);
2184:   StartFunction(GlobalDecl(), ReturnTy, Fn, FI, Args);
2185: 
2186:   // Create a scope with an artificial location for the body of this function.
2187:   auto AL = ApplyDebugLocation::CreateArtificial(*this);
2188: 
2189:   CharUnits Offset;
2190:   Address BufAddr = makeNaturalAddressForPointer(
```
- **EN**: This block spells out callable entry points like `StartFunction`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `StartFunction`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 2191-2220
```cpp
2191:       Builder.CreateLoad(GetAddrOfLocalVar(Args[0]), "buf"), Ctx.VoidTy,
2192:       BufferAlignment);
2193:   Builder.CreateStore(Builder.getInt8(Layout.getSummaryByte()),
2194:                       Builder.CreateConstByteGEP(BufAddr, Offset++, "summary"));
2195:   Builder.CreateStore(Builder.getInt8(Layout.getNumArgsByte()),
2196:                       Builder.CreateConstByteGEP(BufAddr, Offset++, "numArgs"));
2197: 
2198:   unsigned I = 1;
2199:   for (const auto &Item : Layout.Items) {
2200:     Builder.CreateStore(
2201:         Builder.getInt8(Item.getDescriptorByte()),
2202:         Builder.CreateConstByteGEP(BufAddr, Offset++, "argDescriptor"));
2203:     Builder.CreateStore(
2204:         Builder.getInt8(Item.getSizeByte()),
2205:         Builder.CreateConstByteGEP(BufAddr, Offset++, "argSize"));
2206: 
2207:     CharUnits Size = Item.size();
2208:     if (!Size.getQuantity())
2209:       continue;
2210: 
2211:     Address Arg = GetAddrOfLocalVar(Args[I]);
2212:     Address Addr = Builder.CreateConstByteGEP(BufAddr, Offset, "argData");
2213:     Addr = Addr.withElementType(Arg.getElementType());
2214:     Builder.CreateStore(Builder.CreateLoad(Arg), Addr);
2215:     Offset += Size;
2216:     ++I;
2217:   }
2218: 
2219:   FinishFunction();
2220: 
```
- **EN**: This block defines callable entry points like `FinishFunction`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `FinishFunction`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 2221-2250
```cpp
2221:   return Fn;
2222: }
2223: 
2224: RValue CodeGenFunction::emitBuiltinOSLogFormat(const CallExpr &E) {
2225:   assert(E.getNumArgs() >= 2 &&
2226:          "__builtin_os_log_format takes at least 2 arguments");
2227:   ASTContext &Ctx = getContext();
2228:   analyze_os_log::OSLogBufferLayout Layout;
2229:   analyze_os_log::computeOSLogBufferLayout(Ctx, &E, Layout);
2230:   Address BufAddr = EmitPointerWithAlignment(E.getArg(0));
2231: 
2232:   // Ignore argument 1, the format string. It is not currently used.
2233:   CallArgList Args;
2234:   Args.add(RValue::get(BufAddr.emitRawPointer(*this)), Ctx.VoidPtrTy);
2235: 
2236:   for (const auto &Item : Layout.Items) {
2237:     int Size = Item.getSizeByte();
2238:     if (!Size)
2239:       continue;
2240: 
2241:     llvm::Value *ArgVal;
2242: 
2243:     if (Item.getKind() == analyze_os_log::OSLogBufferItem::MaskKind) {
2244:       uint64_t Val = 0;
2245:       for (unsigned I = 0, E = Item.getMaskType().size(); I < E; ++I)
2246:         Val |= ((uint64_t)Item.getMaskType()[I]) << I * 8;
2247:       ArgVal = llvm::Constant::getIntegerValue(Int64Ty, llvm::APInt(64, Val));
2248:     } else if (const Expr *TheExpr = Item.getExpr()) {
2249:       ArgVal = EmitScalarExpr(TheExpr, /*Ignore*/ false);
2250: 
```
- **EN**: This block defines callable entry points like `emitBuiltinOSLogFormat`, `computeOSLogBufferLayout`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitBuiltinOSLogFormat`, `computeOSLogBufferLayout`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 2251-2280
```cpp
2251:       // If a temporary object that requires destruction after the full
2252:       // expression is passed, push a lifetime-extended cleanup to extend its
2253:       // lifetime to the end of the enclosing block scope.
2254:       auto LifetimeExtendObject = [&](const Expr *E) {
2255:         E = E->IgnoreParenCasts();
2256:         // Extend lifetimes of objects returned by function calls and message
2257:         // sends.
2258: 
2259:         // FIXME: We should do this in other cases in which temporaries are
2260:         //        created including arguments of non-ARC types (e.g., C++
2261:         //        temporaries).
2262:         if (isa<CallExpr>(E) || isa<ObjCMessageExpr>(E))
2263:           return true;
2264:         return false;
2265:       };
2266: 
2267:       if (TheExpr->getType()->isObjCRetainableType() &&
2268:           getLangOpts().ObjCAutoRefCount && LifetimeExtendObject(TheExpr)) {
2269:         assert(getEvaluationKind(TheExpr->getType()) == TEK_Scalar &&
2270:                "Only scalar can be a ObjC retainable type");
2271:         if (!isa<Constant>(ArgVal)) {
2272:           CleanupKind Cleanup = getARCCleanupKind();
2273:           QualType Ty = TheExpr->getType();
2274:           RawAddress Alloca = CreateMemTempWithoutCast(Ty, "os.log.arg");
2275:           ArgVal = EmitARCRetain(Ty, ArgVal);
2276:           Builder.CreateStore(ArgVal, Alloca);
2277:           pushLifetimeExtendedDestroy(Cleanup, Alloca, Ty,
2278:                                       CodeGenFunction::destroyARCStrongPrecise,
2279:                                       Cleanup & EHCleanup);
2280: 
```
- **EN**: This block defines callable entry points like `pushLifetimeExtendedDestroy`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `pushLifetimeExtendedDestroy`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 2281-2310
```cpp
2281:           // Push a clang.arc.use call to ensure ARC optimizer knows that the
2282:           // argument has to be alive.
2283:           if (CGM.getCodeGenOpts().OptimizationLevel != 0)
2284:             pushCleanupAfterFullExpr<CallObjCArcUse>(Cleanup, ArgVal);
2285:         }
2286:       }
2287:     } else {
2288:       ArgVal = Builder.getInt32(Item.getConstValue().getQuantity());
2289:     }
2290: 
2291:     unsigned ArgValSize =
2292:         CGM.getDataLayout().getTypeSizeInBits(ArgVal->getType());
2293:     llvm::IntegerType *IntTy = llvm::Type::getIntNTy(getLLVMContext(),
2294:                                                      ArgValSize);
2295:     ArgVal = Builder.CreateBitOrPointerCast(ArgVal, IntTy);
2296:     CanQualType ArgTy = getOSLogArgType(Ctx, Size);
2297:     // If ArgVal has type x86_fp80, zero-extend ArgVal.
2298:     ArgVal = Builder.CreateZExtOrBitCast(ArgVal, ConvertType(ArgTy));
2299:     Args.add(RValue::get(ArgVal), ArgTy);
2300:   }
2301: 
2302:   const CGFunctionInfo &FI =
2303:       CGM.getTypes().arrangeBuiltinFunctionCall(Ctx.VoidTy, Args);
2304:   llvm::Function *F = CodeGenFunction(CGM).generateBuiltinOSLogHelperFunction(
2305:       Layout, BufAddr.getAlignment());
2306:   EmitCall(FI, CGCallee::forDirect(F), ReturnValueSlot(), Args);
2307:   return RValue::get(BufAddr, *this);
2308: }
2309: 
2310: static bool isSpecialUnsignedMultiplySignedResult(
```
- **EN**: This block defines callable entry points like `EmitCall`, `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitCall`, `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 2311-2340
```cpp
2311:     unsigned BuiltinID, WidthAndSignedness Op1Info, WidthAndSignedness Op2Info,
2312:     WidthAndSignedness ResultInfo) {
2313:   return BuiltinID == Builtin::BI__builtin_mul_overflow &&
2314:          Op1Info.Width == Op2Info.Width && Op2Info.Width == ResultInfo.Width &&
2315:          !Op1Info.Signed && !Op2Info.Signed && ResultInfo.Signed;
2316: }
2317: 
2318: static RValue EmitCheckedUnsignedMultiplySignedResult(
2319:     CodeGenFunction &CGF, const clang::Expr *Op1, WidthAndSignedness Op1Info,
2320:     const clang::Expr *Op2, WidthAndSignedness Op2Info,
2321:     const clang::Expr *ResultArg, QualType ResultQTy,
2322:     WidthAndSignedness ResultInfo) {
2323:   assert(isSpecialUnsignedMultiplySignedResult(
2324:              Builtin::BI__builtin_mul_overflow, Op1Info, Op2Info, ResultInfo) &&
2325:          "Cannot specialize this multiply");
2326: 
2327:   llvm::Value *V1 = CGF.EmitScalarExpr(Op1);
2328:   llvm::Value *V2 = CGF.EmitScalarExpr(Op2);
2329: 
2330:   llvm::Value *HasOverflow;
2331:   llvm::Value *Result = EmitOverflowIntrinsic(
2332:       CGF, Intrinsic::umul_with_overflow, V1, V2, HasOverflow);
2333: 
2334:   // The intrinsic call will detect overflow when the value is > UINT_MAX,
2335:   // however, since the original builtin had a signed result, we need to report
2336:   // an overflow when the result is greater than INT_MAX.
2337:   auto IntMax = llvm::APInt::getSignedMaxValue(ResultInfo.Width);
2338:   llvm::Value *IntMaxValue = llvm::ConstantInt::get(Result->getType(), IntMax);
2339: 
2340:   llvm::Value *IntMaxOverflow = CGF.Builder.CreateICmpUGT(Result, IntMaxValue);
```
- **EN**: This block defines callable entry points like `EmitCheckedUnsignedMultiplySignedResult`; returns or forwards computed values for the surrounding LLVM IR emission logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitCheckedUnsignedMultiplySignedResult`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 2341-2370
```cpp
2341:   HasOverflow = CGF.Builder.CreateOr(HasOverflow, IntMaxOverflow);
2342: 
2343:   bool isVolatile =
2344:       ResultArg->getType()->getPointeeType().isVolatileQualified();
2345:   Address ResultPtr = CGF.EmitPointerWithAlignment(ResultArg);
2346:   CGF.Builder.CreateStore(CGF.EmitToMemory(Result, ResultQTy), ResultPtr,
2347:                           isVolatile);
2348:   return RValue::get(HasOverflow);
2349: }
2350: 
2351: /// Determine if a binop is a checked mixed-sign multiply we can specialize.
2352: static bool isSpecialMixedSignMultiply(unsigned BuiltinID,
2353:                                        WidthAndSignedness Op1Info,
2354:                                        WidthAndSignedness Op2Info,
2355:                                        WidthAndSignedness ResultInfo) {
2356:   return BuiltinID == Builtin::BI__builtin_mul_overflow &&
2357:          std::max(Op1Info.Width, Op2Info.Width) >= ResultInfo.Width &&
2358:          Op1Info.Signed != Op2Info.Signed;
2359: }
2360: 
2361: /// Emit a checked mixed-sign multiply. This is a cheaper specialization of
2362: /// the generic checked-binop irgen.
2363: static RValue
2364: EmitCheckedMixedSignMultiply(CodeGenFunction &CGF, const clang::Expr *Op1,
2365:                              WidthAndSignedness Op1Info, const clang::Expr *Op2,
2366:                              WidthAndSignedness Op2Info,
2367:                              const clang::Expr *ResultArg, QualType ResultQTy,
2368:                              WidthAndSignedness ResultInfo) {
2369:   assert(isSpecialMixedSignMultiply(Builtin::BI__builtin_mul_overflow, Op1Info,
2370:                                     Op2Info, ResultInfo) &&
```
- **EN**: This block defines callable entry points like `get`, `isSpecialMixedSignMultiply`, `EmitCheckedMixedSignMultiply`; returns or forwards computed values for the surrounding LLVM IR emission logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`, `isSpecialMixedSignMultiply`, `EmitCheckedMixedSignMultiply`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 2371-2400
```cpp
2371:          "Not a mixed-sign multipliction we can specialize");
2372: 
2373:   // Emit the signed and unsigned operands.
2374:   const clang::Expr *SignedOp = Op1Info.Signed ? Op1 : Op2;
2375:   const clang::Expr *UnsignedOp = Op1Info.Signed ? Op2 : Op1;
2376:   llvm::Value *Signed = CGF.EmitScalarExpr(SignedOp);
2377:   llvm::Value *Unsigned = CGF.EmitScalarExpr(UnsignedOp);
2378:   unsigned SignedOpWidth = Op1Info.Signed ? Op1Info.Width : Op2Info.Width;
2379:   unsigned UnsignedOpWidth = Op1Info.Signed ? Op2Info.Width : Op1Info.Width;
2380: 
2381:   // One of the operands may be smaller than the other. If so, [s|z]ext it.
2382:   if (SignedOpWidth < UnsignedOpWidth)
2383:     Signed = CGF.Builder.CreateSExt(Signed, Unsigned->getType(), "op.sext");
2384:   if (UnsignedOpWidth < SignedOpWidth)
2385:     Unsigned = CGF.Builder.CreateZExt(Unsigned, Signed->getType(), "op.zext");
2386: 
2387:   llvm::Type *OpTy = Signed->getType();
2388:   llvm::Value *Zero = llvm::Constant::getNullValue(OpTy);
2389:   Address ResultPtr = CGF.EmitPointerWithAlignment(ResultArg);
2390:   llvm::Type *ResTy = CGF.getTypes().ConvertType(ResultQTy);
2391:   unsigned OpWidth = std::max(Op1Info.Width, Op2Info.Width);
2392: 
2393:   // Take the absolute value of the signed operand.
2394:   llvm::Value *IsNegative = CGF.Builder.CreateICmpSLT(Signed, Zero);
2395:   llvm::Value *AbsOfNegative = CGF.Builder.CreateSub(Zero, Signed);
2396:   llvm::Value *AbsSigned =
2397:       CGF.Builder.CreateSelect(IsNegative, AbsOfNegative, Signed);
2398: 
2399:   // Perform a checked unsigned multiplication.
2400:   llvm::Value *UnsignedOverflow;
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 2401-2430
```cpp
2401:   llvm::Value *UnsignedResult =
2402:       EmitOverflowIntrinsic(CGF, Intrinsic::umul_with_overflow, AbsSigned,
2403:                             Unsigned, UnsignedOverflow);
2404: 
2405:   llvm::Value *Overflow, *Result;
2406:   if (ResultInfo.Signed) {
2407:     // Signed overflow occurs if the result is greater than INT_MAX or lesser
2408:     // than INT_MIN, i.e when |Result| > (INT_MAX + IsNegative).
2409:     auto IntMax =
2410:         llvm::APInt::getSignedMaxValue(ResultInfo.Width).zext(OpWidth);
2411:     llvm::Value *MaxResult =
2412:         CGF.Builder.CreateAdd(llvm::ConstantInt::get(OpTy, IntMax),
2413:                               CGF.Builder.CreateZExt(IsNegative, OpTy));
2414:     llvm::Value *SignedOverflow =
2415:         CGF.Builder.CreateICmpUGT(UnsignedResult, MaxResult);
2416:     Overflow = CGF.Builder.CreateOr(UnsignedOverflow, SignedOverflow);
2417: 
2418:     // Prepare the signed result (possibly by negating it).
2419:     llvm::Value *NegativeResult = CGF.Builder.CreateNeg(UnsignedResult);
2420:     llvm::Value *SignedResult =
2421:         CGF.Builder.CreateSelect(IsNegative, NegativeResult, UnsignedResult);
2422:     Result = CGF.Builder.CreateTrunc(SignedResult, ResTy);
2423:   } else {
2424:     // Unsigned overflow occurs if the result is < 0 or greater than UINT_MAX.
2425:     llvm::Value *Underflow = CGF.Builder.CreateAnd(
2426:         IsNegative, CGF.Builder.CreateIsNotNull(UnsignedResult));
2427:     Overflow = CGF.Builder.CreateOr(UnsignedOverflow, Underflow);
2428:     if (ResultInfo.Width < OpWidth) {
2429:       auto IntMax =
2430:           llvm::APInt::getMaxValue(ResultInfo.Width).zext(OpWidth);
```
- **EN**: This block defines callable entry points like `EmitOverflowIntrinsic`, `getSignedMaxValue`, `getMaxValue`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitOverflowIntrinsic`, `getSignedMaxValue`, `getMaxValue`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 2431-2460
```cpp
2431:       llvm::Value *TruncOverflow = CGF.Builder.CreateICmpUGT(
2432:           UnsignedResult, llvm::ConstantInt::get(OpTy, IntMax));
2433:       Overflow = CGF.Builder.CreateOr(Overflow, TruncOverflow);
2434:     }
2435: 
2436:     // Negate the product if it would be negative in infinite precision.
2437:     Result = CGF.Builder.CreateSelect(
2438:         IsNegative, CGF.Builder.CreateNeg(UnsignedResult), UnsignedResult);
2439: 
2440:     Result = CGF.Builder.CreateTrunc(Result, ResTy);
2441:   }
2442:   assert(Overflow && Result && "Missing overflow or result");
2443: 
2444:   bool isVolatile =
2445:       ResultArg->getType()->getPointeeType().isVolatileQualified();
2446:   CGF.Builder.CreateStore(CGF.EmitToMemory(Result, ResultQTy), ResultPtr,
2447:                           isVolatile);
2448:   return RValue::get(Overflow);
2449: }
2450: 
2451: /// Determine if the specified type requires laundering by checking if it is a
2452: /// dynamic class type or contains a subobject which is a dynamic class type.
2453: static bool TypeRequiresBuiltinLaunder(CodeGenModule &CGM, QualType Ty) {
2454:   if (!CGM.getCodeGenOpts().StrictVTablePointers)
2455:     return false;
2456:   return Ty.requiresBuiltinLaunder(CGM.getContext());
2457: }
2458: 
2459: RValue CodeGenFunction::emitRotate(const CallExpr *E, bool IsRotateRight) {
2460:   llvm::Value *Src = EmitScalarExpr(E->getArg(0));
```
- **EN**: This block defines callable entry points like `get`, `TypeRequiresBuiltinLaunder`, `emitRotate`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`, `TypeRequiresBuiltinLaunder`, `emitRotate`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 2461-2490
```cpp
2461:   llvm::Value *ShiftAmt = EmitScalarExpr(E->getArg(1));
2462: 
2463:   // The builtin's shift arg may have a different type than the source arg and
2464:   // result, but the LLVM intrinsic uses the same type for all values.
2465:   llvm::Type *Ty = Src->getType();
2466:   llvm::Type *ShiftTy = ShiftAmt->getType();
2467: 
2468:   unsigned BitWidth = Ty->getIntegerBitWidth();
2469: 
2470:   // Normalize shift amount to [0, BitWidth) range to match runtime behavior.
2471:   // This matches the algorithm in ExprConstant.cpp for constant evaluation.
2472:   if (BitWidth == 1) {
2473:     // Rotating a 1-bit value is always a no-op
2474:     ShiftAmt = ConstantInt::get(ShiftTy, 0);
2475:   } else if (BitWidth == 2) {
2476:     // For 2-bit values: rotation amount is 0 or 1 based on
2477:     // whether the amount is even or odd. We can't use srem here because
2478:     // the divisor (2) would be misinterpreted as -2 in 2-bit signed arithmetic.
2479:     llvm::Value *One = ConstantInt::get(ShiftTy, 1);
2480:     ShiftAmt = Builder.CreateAnd(ShiftAmt, One);
2481:   } else {
2482:     unsigned ShiftAmtBitWidth = ShiftTy->getIntegerBitWidth();
2483:     bool ShiftAmtIsSigned = E->getArg(1)->getType()->isSignedIntegerType();
2484: 
2485:     // Choose the wider type for the divisor to avoid truncation
2486:     llvm::Type *DivisorTy = ShiftAmtBitWidth > BitWidth ? ShiftTy : Ty;
2487:     llvm::Value *Divisor = ConstantInt::get(DivisorTy, BitWidth);
2488: 
2489:     // Extend ShiftAmt to match Divisor width if needed
2490:     if (ShiftAmtBitWidth < DivisorTy->getIntegerBitWidth()) {
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 2491-2520
```cpp
2491:       ShiftAmt = Builder.CreateIntCast(ShiftAmt, DivisorTy, ShiftAmtIsSigned);
2492:     }
2493: 
2494:     // Normalize to [0, BitWidth)
2495:     llvm::Value *RemResult;
2496:     if (ShiftAmtIsSigned) {
2497:       RemResult = Builder.CreateSRem(ShiftAmt, Divisor);
2498:       // Signed remainder can be negative, convert to positive equivalent
2499:       llvm::Value *Zero = ConstantInt::get(DivisorTy, 0);
2500:       llvm::Value *IsNegative = Builder.CreateICmpSLT(RemResult, Zero);
2501:       llvm::Value *PositiveShift = Builder.CreateAdd(RemResult, Divisor);
2502:       ShiftAmt = Builder.CreateSelect(IsNegative, PositiveShift, RemResult);
2503:     } else {
2504:       ShiftAmt = Builder.CreateURem(ShiftAmt, Divisor);
2505:     }
2506:   }
2507: 
2508:   // Convert to the source type if needed
2509:   if (ShiftAmt->getType() != Ty) {
2510:     ShiftAmt = Builder.CreateIntCast(ShiftAmt, Ty, false);
2511:   }
2512: 
2513:   // Rotate is a special case of LLVM funnel shift - 1st 2 args are the same.
2514:   unsigned IID = IsRotateRight ? Intrinsic::fshr : Intrinsic::fshl;
2515:   Function *F = CGM.getIntrinsic(IID, Ty);
2516:   return RValue::get(Builder.CreateCall(F, {Src, Src, ShiftAmt}));
2517: }
2518: 
2519: // Map math builtins for long-double to f128 version.
2520: static unsigned mutateLongDoubleBuiltin(unsigned BuiltinID) {
```
- **EN**: This block defines callable entry points like `mutateLongDoubleBuiltin`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `mutateLongDoubleBuiltin`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 2521-2550
```cpp
2521:   switch (BuiltinID) {
2522: #define MUTATE_LDBL(func) \
2523:   case Builtin::BI__builtin_##func##l: \
2524:     return Builtin::BI__builtin_##func##f128;
2525:   MUTATE_LDBL(sqrt)
2526:   MUTATE_LDBL(cbrt)
2527:   MUTATE_LDBL(fabs)
2528:   MUTATE_LDBL(log)
2529:   MUTATE_LDBL(log2)
2530:   MUTATE_LDBL(log10)
2531:   MUTATE_LDBL(log1p)
2532:   MUTATE_LDBL(logb)
2533:   MUTATE_LDBL(exp)
2534:   MUTATE_LDBL(exp2)
2535:   MUTATE_LDBL(expm1)
2536:   MUTATE_LDBL(fdim)
2537:   MUTATE_LDBL(hypot)
2538:   MUTATE_LDBL(ilogb)
2539:   MUTATE_LDBL(pow)
2540:   MUTATE_LDBL(fmin)
2541:   MUTATE_LDBL(fmax)
2542:   MUTATE_LDBL(ceil)
2543:   MUTATE_LDBL(trunc)
2544:   MUTATE_LDBL(rint)
2545:   MUTATE_LDBL(nearbyint)
2546:   MUTATE_LDBL(round)
2547:   MUTATE_LDBL(floor)
2548:   MUTATE_LDBL(lround)
2549:   MUTATE_LDBL(llround)
2550:   MUTATE_LDBL(lrint)
```
- **EN**: This block uses control flow (switch, case) to specialize LLVM IR emission; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块通过控制流（switch, case）细化 LLVM IR 生成 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 2551-2580
```cpp
2551:   MUTATE_LDBL(llrint)
2552:   MUTATE_LDBL(fmod)
2553:   MUTATE_LDBL(modf)
2554:   MUTATE_LDBL(nan)
2555:   MUTATE_LDBL(nans)
2556:   MUTATE_LDBL(inf)
2557:   MUTATE_LDBL(fma)
2558:   MUTATE_LDBL(sin)
2559:   MUTATE_LDBL(cos)
2560:   MUTATE_LDBL(tan)
2561:   MUTATE_LDBL(sinh)
2562:   MUTATE_LDBL(cosh)
2563:   MUTATE_LDBL(tanh)
2564:   MUTATE_LDBL(asin)
2565:   MUTATE_LDBL(acos)
2566:   MUTATE_LDBL(atan)
2567:   MUTATE_LDBL(asinh)
2568:   MUTATE_LDBL(acosh)
2569:   MUTATE_LDBL(atanh)
2570:   MUTATE_LDBL(atan2)
2571:   MUTATE_LDBL(erf)
2572:   MUTATE_LDBL(erfc)
2573:   MUTATE_LDBL(ldexp)
2574:   MUTATE_LDBL(frexp)
2575:   MUTATE_LDBL(huge_val)
2576:   MUTATE_LDBL(copysign)
2577:   MUTATE_LDBL(nextafter)
2578:   MUTATE_LDBL(nexttoward)
2579:   MUTATE_LDBL(remainder)
2580:   MUTATE_LDBL(remquo)
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding LLVM IR emission implementation.
- **CN**: 该代码块为周围的 LLVM IR 生成 实现提供必要的胶水代码、布局或分隔结构。

### Lines 2581-2610
```cpp
2581:   MUTATE_LDBL(scalbln)
2582:   MUTATE_LDBL(scalbn)
2583:   MUTATE_LDBL(tgamma)
2584:   MUTATE_LDBL(lgamma)
2585: #undef MUTATE_LDBL
2586:   default:
2587:     return BuiltinID;
2588:   }
2589: }
2590: 
2591: static Value *tryUseTestFPKind(CodeGenFunction &CGF, unsigned BuiltinID,
2592:                                Value *V) {
2593:   if (CGF.Builder.getIsFPConstrained() &&
2594:       CGF.Builder.getDefaultConstrainedExcept() != fp::ebIgnore) {
2595:     if (Value *Result =
2596:             CGF.getTargetHooks().testFPKind(V, BuiltinID, CGF.Builder, CGF.CGM))
2597:       return Result;
2598:   }
2599:   return nullptr;
2600: }
2601: 
2602: static RValue EmitHipStdParUnsupportedBuiltin(CodeGenFunction *CGF,
2603:                                               const FunctionDecl *FD) {
2604:   auto Name = FD->getNameAsString() + "__hipstdpar_unsupported";
2605:   auto FnTy = CGF->CGM.getTypes().GetFunctionType(FD);
2606:   auto UBF = CGF->CGM.getModule().getOrInsertFunction(Name, FnTy);
2607: 
2608:   SmallVector<Value *, 16> Args;
2609:   for (auto &&FormalTy : FnTy->params())
2610:     Args.push_back(llvm::PoisonValue::get(FormalTy));
```
- **EN**: This block defines callable entry points like `EmitHipStdParUnsupportedBuiltin`; uses control flow (if, for) to specialize LLVM IR emission; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块定义可调用入口，例如 `EmitHipStdParUnsupportedBuiltin`；通过控制流（if, for）细化 LLVM IR 生成 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 2611-2640
```cpp
2611: 
2612:   return RValue::get(CGF->Builder.CreateCall(UBF, Args));
2613: }
2614: 
2615: // stdc_{leading,trailing}_{zeros,ones} and stdc_count_ones: counts bits using
2616: // ctlz, cttz, or ctpop (IsPop). InvertArg flips the input to count the
2617: // opposite bit value.
2618: RValue CodeGenFunction::emitStdcCountIntrinsic(const CallExpr *E,
2619:                                                Intrinsic::ID IntID,
2620:                                                bool InvertArg, bool IsPop) {
2621:   Value *ArgValue = EmitScalarExpr(E->getArg(0));
2622:   llvm::Type *ArgType = ArgValue->getType();
2623:   llvm::Type *ResultType = ConvertType(E->getType());
2624:   Value *ActualArg = InvertArg ? Builder.CreateNot(ArgValue) : ArgValue;
2625:   Function *F = CGM.getIntrinsic(IntID, ArgType);
2626:   Value *Result = IsPop
2627:                       ? Builder.CreateCall(F, ActualArg)
2628:                       : Builder.CreateCall(F, {ActualArg, Builder.getFalse()});
2629:   if (Result->getType() != ResultType)
2630:     Result = Builder.CreateIntCast(Result, ResultType, false);
2631:   return RValue::get(Result);
2632: }
2633: 
2634: // stdc_count_zeros (BitWidth - ctpop) and stdc_bit_width (BitWidth - ctlz).
2635: // IsPop selects ctpop; otherwise ctlz is used.
2636: RValue CodeGenFunction::emitStdcBitWidthMinus(const CallExpr *E,
2637:                                               Intrinsic::ID IntID, bool IsPop) {
2638:   Value *ArgValue = EmitScalarExpr(E->getArg(0));
2639:   llvm::Type *ArgType = ArgValue->getType();
2640:   llvm::Type *ResultType = ConvertType(E->getType());
```
- **EN**: This block defines callable entry points like `get`, `emitStdcCountIntrinsic`, `emitStdcBitWidthMinus`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `emitStdcCountIntrinsic`, `emitStdcBitWidthMinus`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 2641-2670
```cpp
2641:   unsigned BitWidth = ArgType->getIntegerBitWidth();
2642:   Function *F = CGM.getIntrinsic(IntID, ArgType);
2643:   Value *Cnt = IsPop ? Builder.CreateCall(F, ArgValue)
2644:                      : Builder.CreateCall(F, {ArgValue, Builder.getFalse()});
2645:   Value *Result = Builder.CreateSub(ConstantInt::get(ArgType, BitWidth), Cnt);
2646:   if (Result->getType() != ResultType)
2647:     Result = Builder.CreateIntCast(Result, ResultType, false);
2648:   return RValue::get(Result);
2649: }
2650: 
2651: // stdc_first_{leading,trailing}_{zero,one}: returns the 1-based position of
2652: // the first matching bit, or 0 if no such bit exists. InvertArg flips the
2653: // input to search for zeros instead of ones.
2654: RValue CodeGenFunction::emitStdcFirstBit(const CallExpr *E, Intrinsic::ID IntID,
2655:                                          bool InvertArg) {
2656:   Value *ArgValue = EmitScalarExpr(E->getArg(0));
2657:   llvm::Type *ArgType = ArgValue->getType();
2658:   llvm::Type *ResultType = ConvertType(E->getType());
2659:   Value *Zero = ConstantInt::get(ArgType, 0);
2660:   Value *One = ConstantInt::get(ArgType, 1);
2661:   Value *ActualArg = InvertArg ? Builder.CreateNot(ArgValue) : ArgValue;
2662:   Function *F = CGM.getIntrinsic(IntID, ArgType);
2663:   Value *Cnt = Builder.CreateCall(F, {ActualArg, Builder.getFalse()});
2664:   Value *Tmp = Builder.CreateAdd(Cnt, One);
2665:   Value *IsZero = Builder.CreateICmpEQ(ActualArg, Zero);
2666:   Value *Result = Builder.CreateSelect(IsZero, Zero, Tmp);
2667:   if (Result->getType() != ResultType)
2668:     Result = Builder.CreateIntCast(Result, ResultType, false);
2669:   return RValue::get(Result);
2670: }
```
- **EN**: This block defines callable entry points like `get`, `emitStdcFirstBit`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `emitStdcFirstBit`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 2671-2700
```cpp
2671: 
2672: RValue CodeGenFunction::EmitBuiltinExpr(const GlobalDecl GD, unsigned BuiltinID,
2673:                                         const CallExpr *E,
2674:                                         ReturnValueSlot ReturnValue) {
2675:   assert(!getContext().BuiltinInfo.isImmediate(BuiltinID) &&
2676:          "Should not codegen for consteval builtins");
2677: 
2678:   const FunctionDecl *FD = GD.getDecl()->getAsFunction();
2679:   // See if we can constant fold this builtin.  If so, don't emit it at all.
2680:   // TODO: Extend this handling to all builtin calls that we can constant-fold.
2681:   Expr::EvalResult Result;
2682:   if (E->isPRValue() && E->EvaluateAsRValue(Result, CGM.getContext()) &&
2683:       !Result.hasSideEffects()) {
2684:     if (Result.Val.isInt())
2685:       return RValue::get(llvm::ConstantInt::get(getLLVMContext(),
2686:                                                 Result.Val.getInt()));
2687:     if (Result.Val.isFloat())
2688:       return RValue::get(llvm::ConstantFP::get(getLLVMContext(),
2689:                                                Result.Val.getFloat()));
2690:   }
2691: 
2692:   // If current long-double semantics is IEEE 128-bit, replace math builtins
2693:   // of long-double with f128 equivalent.
2694:   // TODO: This mutation should also be applied to other targets other than PPC,
2695:   // after backend supports IEEE 128-bit style libcalls.
2696:   if (getTarget().getTriple().isPPC64() &&
2697:       &getTarget().getLongDoubleFormat() == &llvm::APFloat::IEEEquad())
2698:     BuiltinID = mutateLongDoubleBuiltin(BuiltinID);
2699: 
2700:   // If the builtin has been declared explicitly with an assembler label,
```
- **EN**: This block defines callable entry points like `EmitBuiltinExpr`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitBuiltinExpr`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 2701-2730
```cpp
2701:   // disable the specialized emitting below. Ideally we should communicate the
2702:   // rename in IR, or at least avoid generating the intrinsic calls that are
2703:   // likely to get lowered to the renamed library functions.
2704:   const unsigned BuiltinIDIfNoAsmLabel =
2705:       FD->hasAttr<AsmLabelAttr>() ? 0 : BuiltinID;
2706: 
2707:   std::optional<bool> ErrnoOverriden;
2708:   // ErrnoOverriden is true if math-errno is overriden via the
2709:   // '#pragma float_control(precise, on)'. This pragma disables fast-math,
2710:   // which implies math-errno.
2711:   if (E->hasStoredFPFeatures()) {
2712:     FPOptionsOverride OP = E->getFPFeatures();
2713:     if (OP.hasMathErrnoOverride())
2714:       ErrnoOverriden = OP.getMathErrnoOverride();
2715:   }
2716:   // True if 'attribute__((optnone))' is used. This attribute overrides
2717:   // fast-math which implies math-errno.
2718:   bool OptNone = CurFuncDecl && CurFuncDecl->hasAttr<OptimizeNoneAttr>();
2719: 
2720:   bool IsOptimizationEnabled = CGM.getCodeGenOpts().OptimizationLevel != 0;
2721: 
2722:   bool GenerateFPMathIntrinsics =
2723:       getContext().BuiltinInfo.shouldGenerateFPMathIntrinsic(
2724:           BuiltinID, CGM.getTriple(), ErrnoOverriden, getLangOpts().MathErrno,
2725:           OptNone, IsOptimizationEnabled);
2726: 
2727:   if (GenerateFPMathIntrinsics) {
2728:     switch (BuiltinIDIfNoAsmLabel) {
2729:     case Builtin::BIacos:
2730:     case Builtin::BIacosf:
```
- **EN**: This block defines callable entry points like `getContext`; uses control flow (if, switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getContext`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为。

### Lines 2731-2760
```cpp
2731:     case Builtin::BIacosl:
2732:     case Builtin::BI__builtin_acos:
2733:     case Builtin::BI__builtin_acosf:
2734:     case Builtin::BI__builtin_acosf16:
2735:     case Builtin::BI__builtin_acosl:
2736:     case Builtin::BI__builtin_acosf128:
2737:     case Builtin::BI__builtin_elementwise_acos:
2738:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(
2739:           *this, E, Intrinsic::acos, Intrinsic::experimental_constrained_acos));
2740: 
2741:     case Builtin::BIasin:
2742:     case Builtin::BIasinf:
2743:     case Builtin::BIasinl:
2744:     case Builtin::BI__builtin_asin:
2745:     case Builtin::BI__builtin_asinf:
2746:     case Builtin::BI__builtin_asinf16:
2747:     case Builtin::BI__builtin_asinl:
2748:     case Builtin::BI__builtin_asinf128:
2749:     case Builtin::BI__builtin_elementwise_asin:
2750:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(
2751:           *this, E, Intrinsic::asin, Intrinsic::experimental_constrained_asin));
2752: 
2753:     case Builtin::BIatan:
2754:     case Builtin::BIatanf:
2755:     case Builtin::BIatanl:
2756:     case Builtin::BI__builtin_atan:
2757:     case Builtin::BI__builtin_atanf:
2758:     case Builtin::BI__builtin_atanf16:
2759:     case Builtin::BI__builtin_atanl:
2760:     case Builtin::BI__builtin_atanf128:
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 2761-2790
```cpp
2761:     case Builtin::BI__builtin_elementwise_atan:
2762:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(
2763:           *this, E, Intrinsic::atan, Intrinsic::experimental_constrained_atan));
2764: 
2765:     case Builtin::BIatan2:
2766:     case Builtin::BIatan2f:
2767:     case Builtin::BIatan2l:
2768:     case Builtin::BI__builtin_atan2:
2769:     case Builtin::BI__builtin_atan2f:
2770:     case Builtin::BI__builtin_atan2f16:
2771:     case Builtin::BI__builtin_atan2l:
2772:     case Builtin::BI__builtin_atan2f128:
2773:     case Builtin::BI__builtin_elementwise_atan2:
2774:       return RValue::get(emitBinaryMaybeConstrainedFPBuiltin(
2775:           *this, E, Intrinsic::atan2,
2776:           Intrinsic::experimental_constrained_atan2));
2777: 
2778:     case Builtin::BIceil:
2779:     case Builtin::BIceilf:
2780:     case Builtin::BIceill:
2781:     case Builtin::BI__builtin_ceil:
2782:     case Builtin::BI__builtin_ceilf:
2783:     case Builtin::BI__builtin_ceilf16:
2784:     case Builtin::BI__builtin_ceill:
2785:     case Builtin::BI__builtin_ceilf128:
2786:     case Builtin::BI__builtin_elementwise_ceil:
2787:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
2788:                                    Intrinsic::ceil,
2789:                                    Intrinsic::experimental_constrained_ceil));
2790: 
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 2791-2820
```cpp
2791:     case Builtin::BIcopysign:
2792:     case Builtin::BIcopysignf:
2793:     case Builtin::BIcopysignl:
2794:     case Builtin::BI__builtin_copysign:
2795:     case Builtin::BI__builtin_copysignf:
2796:     case Builtin::BI__builtin_copysignf16:
2797:     case Builtin::BI__builtin_copysignl:
2798:     case Builtin::BI__builtin_copysignf128:
2799:       return RValue::get(
2800:           emitBuiltinWithOneOverloadedType<2>(*this, E, Intrinsic::copysign));
2801: 
2802:     case Builtin::BIcos:
2803:     case Builtin::BIcosf:
2804:     case Builtin::BIcosl:
2805:     case Builtin::BI__builtin_cos:
2806:     case Builtin::BI__builtin_cosf:
2807:     case Builtin::BI__builtin_cosf16:
2808:     case Builtin::BI__builtin_cosl:
2809:     case Builtin::BI__builtin_cosf128:
2810:     case Builtin::BI__builtin_elementwise_cos:
2811:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
2812:                                    Intrinsic::cos,
2813:                                    Intrinsic::experimental_constrained_cos));
2814: 
2815:     case Builtin::BIcosh:
2816:     case Builtin::BIcoshf:
2817:     case Builtin::BIcoshl:
2818:     case Builtin::BI__builtin_cosh:
2819:     case Builtin::BI__builtin_coshf:
2820:     case Builtin::BI__builtin_coshf16:
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 2821-2850
```cpp
2821:     case Builtin::BI__builtin_coshl:
2822:     case Builtin::BI__builtin_coshf128:
2823:     case Builtin::BI__builtin_elementwise_cosh:
2824:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(
2825:           *this, E, Intrinsic::cosh, Intrinsic::experimental_constrained_cosh));
2826: 
2827:     case Builtin::BIexp:
2828:     case Builtin::BIexpf:
2829:     case Builtin::BIexpl:
2830:     case Builtin::BI__builtin_exp:
2831:     case Builtin::BI__builtin_expf:
2832:     case Builtin::BI__builtin_expf16:
2833:     case Builtin::BI__builtin_expl:
2834:     case Builtin::BI__builtin_expf128:
2835:     case Builtin::BI__builtin_elementwise_exp:
2836:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
2837:                                    Intrinsic::exp,
2838:                                    Intrinsic::experimental_constrained_exp));
2839: 
2840:     case Builtin::BIexp2:
2841:     case Builtin::BIexp2f:
2842:     case Builtin::BIexp2l:
2843:     case Builtin::BI__builtin_exp2:
2844:     case Builtin::BI__builtin_exp2f:
2845:     case Builtin::BI__builtin_exp2f16:
2846:     case Builtin::BI__builtin_exp2l:
2847:     case Builtin::BI__builtin_exp2f128:
2848:     case Builtin::BI__builtin_elementwise_exp2:
2849:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
2850:                                    Intrinsic::exp2,
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 2851-2880
```cpp
2851:                                    Intrinsic::experimental_constrained_exp2));
2852:     case Builtin::BI__builtin_exp10:
2853:     case Builtin::BI__builtin_exp10f:
2854:     case Builtin::BI__builtin_exp10f16:
2855:     case Builtin::BI__builtin_exp10l:
2856:     case Builtin::BI__builtin_exp10f128:
2857:     case Builtin::BI__builtin_elementwise_exp10: {
2858:       // TODO: strictfp support
2859:       if (Builder.getIsFPConstrained())
2860:         break;
2861:       return RValue::get(
2862:           emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::exp10));
2863:     }
2864:     case Builtin::BIfabs:
2865:     case Builtin::BIfabsf:
2866:     case Builtin::BIfabsl:
2867:     case Builtin::BI__builtin_fabs:
2868:     case Builtin::BI__builtin_fabsf:
2869:     case Builtin::BI__builtin_fabsf16:
2870:     case Builtin::BI__builtin_fabsl:
2871:     case Builtin::BI__builtin_fabsf128:
2872:       return RValue::get(
2873:           emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::fabs));
2874: 
2875:     case Builtin::BIfloor:
2876:     case Builtin::BIfloorf:
2877:     case Builtin::BIfloorl:
2878:     case Builtin::BI__builtin_floor:
2879:     case Builtin::BI__builtin_floorf:
2880:     case Builtin::BI__builtin_floorf16:
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 2881-2910
```cpp
2881:     case Builtin::BI__builtin_floorl:
2882:     case Builtin::BI__builtin_floorf128:
2883:     case Builtin::BI__builtin_elementwise_floor:
2884:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
2885:                                    Intrinsic::floor,
2886:                                    Intrinsic::experimental_constrained_floor));
2887: 
2888:     case Builtin::BIfma:
2889:     case Builtin::BIfmaf:
2890:     case Builtin::BIfmal:
2891:     case Builtin::BI__builtin_fma:
2892:     case Builtin::BI__builtin_fmaf:
2893:     case Builtin::BI__builtin_fmaf16:
2894:     case Builtin::BI__builtin_fmal:
2895:     case Builtin::BI__builtin_fmaf128:
2896:     case Builtin::BI__builtin_elementwise_fma:
2897:       return RValue::get(emitTernaryMaybeConstrainedFPBuiltin(*this, E,
2898:                                    Intrinsic::fma,
2899:                                    Intrinsic::experimental_constrained_fma));
2900: 
2901:     case Builtin::BIfmax:
2902:     case Builtin::BIfmaxf:
2903:     case Builtin::BIfmaxl:
2904:     case Builtin::BI__builtin_fmax:
2905:     case Builtin::BI__builtin_fmaxf:
2906:     case Builtin::BI__builtin_fmaxf16:
2907:     case Builtin::BI__builtin_fmaxl:
2908:     case Builtin::BI__builtin_fmaxf128: {
2909:       IRBuilder<>::FastMathFlagGuard FMFGuard(Builder);
2910:       Builder.getFastMathFlags().setNoSignedZeros();
```
- **EN**: This block defines callable entry points like `get`, `FMFGuard`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `FMFGuard`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 2911-2940
```cpp
2911:       return RValue::get(emitBinaryMaybeConstrainedFPBuiltin(
2912:           *this, E, Intrinsic::maxnum,
2913:           Intrinsic::experimental_constrained_maxnum));
2914:     }
2915: 
2916:     case Builtin::BIfmin:
2917:     case Builtin::BIfminf:
2918:     case Builtin::BIfminl:
2919:     case Builtin::BI__builtin_fmin:
2920:     case Builtin::BI__builtin_fminf:
2921:     case Builtin::BI__builtin_fminf16:
2922:     case Builtin::BI__builtin_fminl:
2923:     case Builtin::BI__builtin_fminf128: {
2924:       IRBuilder<>::FastMathFlagGuard FMFGuard(Builder);
2925:       Builder.getFastMathFlags().setNoSignedZeros();
2926:       return RValue::get(emitBinaryMaybeConstrainedFPBuiltin(
2927:           *this, E, Intrinsic::minnum,
2928:           Intrinsic::experimental_constrained_minnum));
2929:     }
2930: 
2931:     case Builtin::BIfmaximum_num:
2932:     case Builtin::BIfmaximum_numf:
2933:     case Builtin::BIfmaximum_numl:
2934:     case Builtin::BI__builtin_fmaximum_num:
2935:     case Builtin::BI__builtin_fmaximum_numf:
2936:     case Builtin::BI__builtin_fmaximum_numf16:
2937:     case Builtin::BI__builtin_fmaximum_numl:
2938:     case Builtin::BI__builtin_fmaximum_numf128:
2939:       return RValue::get(
2940:           emitBuiltinWithOneOverloadedType<2>(*this, E, Intrinsic::maximumnum));
```
- **EN**: This block defines callable entry points like `get`, `FMFGuard`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `FMFGuard`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 2941-2970
```cpp
2941: 
2942:     case Builtin::BIfminimum_num:
2943:     case Builtin::BIfminimum_numf:
2944:     case Builtin::BIfminimum_numl:
2945:     case Builtin::BI__builtin_fminimum_num:
2946:     case Builtin::BI__builtin_fminimum_numf:
2947:     case Builtin::BI__builtin_fminimum_numf16:
2948:     case Builtin::BI__builtin_fminimum_numl:
2949:     case Builtin::BI__builtin_fminimum_numf128:
2950:       return RValue::get(
2951:           emitBuiltinWithOneOverloadedType<2>(*this, E, Intrinsic::minimumnum));
2952: 
2953:     // fmod() is a special-case. It maps to the frem instruction rather than an
2954:     // LLVM intrinsic.
2955:     case Builtin::BIfmod:
2956:     case Builtin::BIfmodf:
2957:     case Builtin::BIfmodl:
2958:     case Builtin::BI__builtin_fmod:
2959:     case Builtin::BI__builtin_fmodf:
2960:     case Builtin::BI__builtin_fmodf16:
2961:     case Builtin::BI__builtin_fmodl:
2962:     case Builtin::BI__builtin_fmodf128:
2963:     case Builtin::BI__builtin_elementwise_fmod: {
2964:       CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
2965:       Value *Arg1 = EmitScalarExpr(E->getArg(0));
2966:       Value *Arg2 = EmitScalarExpr(E->getArg(1));
2967:       if (Builder.getIsFPConstrained()) {
2968:         Function *F = CGM.getIntrinsic(Intrinsic::experimental_constrained_frem,
2969:                                        Arg1->getType());
2970:         return RValue::get(Builder.CreateConstrainedFPCall(F, {Arg1, Arg2}));
```
- **EN**: This block defines callable entry points like `get`, `FPOptsRAII`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `FPOptsRAII`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 2971-3000
```cpp
2971:       } else {
2972:         return RValue::get(Builder.CreateFRem(Arg1, Arg2, "fmod"));
2973:       }
2974:     }
2975: 
2976:     case Builtin::BIlog:
2977:     case Builtin::BIlogf:
2978:     case Builtin::BIlogl:
2979:     case Builtin::BI__builtin_log:
2980:     case Builtin::BI__builtin_logf:
2981:     case Builtin::BI__builtin_logf16:
2982:     case Builtin::BI__builtin_logl:
2983:     case Builtin::BI__builtin_logf128:
2984:     case Builtin::BI__builtin_elementwise_log:
2985:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
2986:                                    Intrinsic::log,
2987:                                    Intrinsic::experimental_constrained_log));
2988: 
2989:     case Builtin::BIlog10:
2990:     case Builtin::BIlog10f:
2991:     case Builtin::BIlog10l:
2992:     case Builtin::BI__builtin_log10:
2993:     case Builtin::BI__builtin_log10f:
2994:     case Builtin::BI__builtin_log10f16:
2995:     case Builtin::BI__builtin_log10l:
2996:     case Builtin::BI__builtin_log10f128:
2997:     case Builtin::BI__builtin_elementwise_log10:
2998:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
2999:                                    Intrinsic::log10,
3000:                                    Intrinsic::experimental_constrained_log10));
```
- **EN**: This block defines callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3001-3030
```cpp
3001: 
3002:     case Builtin::BIlog2:
3003:     case Builtin::BIlog2f:
3004:     case Builtin::BIlog2l:
3005:     case Builtin::BI__builtin_log2:
3006:     case Builtin::BI__builtin_log2f:
3007:     case Builtin::BI__builtin_log2f16:
3008:     case Builtin::BI__builtin_log2l:
3009:     case Builtin::BI__builtin_log2f128:
3010:     case Builtin::BI__builtin_elementwise_log2:
3011:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
3012:                                    Intrinsic::log2,
3013:                                    Intrinsic::experimental_constrained_log2));
3014: 
3015:     case Builtin::BInearbyint:
3016:     case Builtin::BInearbyintf:
3017:     case Builtin::BInearbyintl:
3018:     case Builtin::BI__builtin_nearbyint:
3019:     case Builtin::BI__builtin_nearbyintf:
3020:     case Builtin::BI__builtin_nearbyintl:
3021:     case Builtin::BI__builtin_nearbyintf128:
3022:     case Builtin::BI__builtin_elementwise_nearbyint:
3023:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
3024:                                 Intrinsic::nearbyint,
3025:                                 Intrinsic::experimental_constrained_nearbyint));
3026: 
3027:     case Builtin::BIpow:
3028:     case Builtin::BIpowf:
3029:     case Builtin::BIpowl:
3030:     case Builtin::BI__builtin_pow:
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3031-3060
```cpp
3031:     case Builtin::BI__builtin_powf:
3032:     case Builtin::BI__builtin_powf16:
3033:     case Builtin::BI__builtin_powl:
3034:     case Builtin::BI__builtin_powf128:
3035:     case Builtin::BI__builtin_elementwise_pow:
3036:       return RValue::get(emitBinaryMaybeConstrainedFPBuiltin(*this, E,
3037:                                    Intrinsic::pow,
3038:                                    Intrinsic::experimental_constrained_pow));
3039: 
3040:     case Builtin::BIrint:
3041:     case Builtin::BIrintf:
3042:     case Builtin::BIrintl:
3043:     case Builtin::BI__builtin_rint:
3044:     case Builtin::BI__builtin_rintf:
3045:     case Builtin::BI__builtin_rintf16:
3046:     case Builtin::BI__builtin_rintl:
3047:     case Builtin::BI__builtin_rintf128:
3048:     case Builtin::BI__builtin_elementwise_rint:
3049:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
3050:                                    Intrinsic::rint,
3051:                                    Intrinsic::experimental_constrained_rint));
3052: 
3053:     case Builtin::BIround:
3054:     case Builtin::BIroundf:
3055:     case Builtin::BIroundl:
3056:     case Builtin::BI__builtin_round:
3057:     case Builtin::BI__builtin_roundf:
3058:     case Builtin::BI__builtin_roundf16:
3059:     case Builtin::BI__builtin_roundl:
3060:     case Builtin::BI__builtin_roundf128:
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3061-3090
```cpp
3061:     case Builtin::BI__builtin_elementwise_round:
3062:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
3063:                                    Intrinsic::round,
3064:                                    Intrinsic::experimental_constrained_round));
3065: 
3066:     case Builtin::BIroundeven:
3067:     case Builtin::BIroundevenf:
3068:     case Builtin::BIroundevenl:
3069:     case Builtin::BI__builtin_roundeven:
3070:     case Builtin::BI__builtin_roundevenf:
3071:     case Builtin::BI__builtin_roundevenf16:
3072:     case Builtin::BI__builtin_roundevenl:
3073:     case Builtin::BI__builtin_roundevenf128:
3074:     case Builtin::BI__builtin_elementwise_roundeven:
3075:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
3076:                                    Intrinsic::roundeven,
3077:                                    Intrinsic::experimental_constrained_roundeven));
3078: 
3079:     case Builtin::BIsin:
3080:     case Builtin::BIsinf:
3081:     case Builtin::BIsinl:
3082:     case Builtin::BI__builtin_sin:
3083:     case Builtin::BI__builtin_sinf:
3084:     case Builtin::BI__builtin_sinf16:
3085:     case Builtin::BI__builtin_sinl:
3086:     case Builtin::BI__builtin_sinf128:
3087:     case Builtin::BI__builtin_elementwise_sin:
3088:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
3089:                                    Intrinsic::sin,
3090:                                    Intrinsic::experimental_constrained_sin));
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3091-3120
```cpp
3091: 
3092:     case Builtin::BIsinh:
3093:     case Builtin::BIsinhf:
3094:     case Builtin::BIsinhl:
3095:     case Builtin::BI__builtin_sinh:
3096:     case Builtin::BI__builtin_sinhf:
3097:     case Builtin::BI__builtin_sinhf16:
3098:     case Builtin::BI__builtin_sinhl:
3099:     case Builtin::BI__builtin_sinhf128:
3100:     case Builtin::BI__builtin_elementwise_sinh:
3101:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(
3102:           *this, E, Intrinsic::sinh, Intrinsic::experimental_constrained_sinh));
3103: 
3104:     case Builtin::BI__builtin_sincospi:
3105:     case Builtin::BI__builtin_sincospif:
3106:     case Builtin::BI__builtin_sincospil:
3107:       if (Builder.getIsFPConstrained())
3108:         break; // TODO: Emit constrained sincospi intrinsic once one exists.
3109:       emitSincosBuiltin(*this, E, Intrinsic::sincospi);
3110:       return RValue::get(nullptr);
3111: 
3112:     case Builtin::BIsincos:
3113:     case Builtin::BIsincosf:
3114:     case Builtin::BIsincosl:
3115:     case Builtin::BI__builtin_sincos:
3116:     case Builtin::BI__builtin_sincosf:
3117:     case Builtin::BI__builtin_sincosf16:
3118:     case Builtin::BI__builtin_sincosl:
3119:     case Builtin::BI__builtin_sincosf128:
3120:       if (Builder.getIsFPConstrained())
```
- **EN**: This block spells out callable entry points like `emitSincosBuiltin`, `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `emitSincosBuiltin`, `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3121-3150
```cpp
3121:         break; // TODO: Emit constrained sincos intrinsic once one exists.
3122:       emitSincosBuiltin(*this, E, Intrinsic::sincos);
3123:       return RValue::get(nullptr);
3124: 
3125:     case Builtin::BIsqrt:
3126:     case Builtin::BIsqrtf:
3127:     case Builtin::BIsqrtl:
3128:     case Builtin::BI__builtin_sqrt:
3129:     case Builtin::BI__builtin_sqrtf:
3130:     case Builtin::BI__builtin_sqrtf16:
3131:     case Builtin::BI__builtin_sqrtl:
3132:     case Builtin::BI__builtin_sqrtf128:
3133:     case Builtin::BI__builtin_elementwise_sqrt: {
3134:       llvm::Value *Call = emitUnaryMaybeConstrainedFPBuiltin(
3135:           *this, E, Intrinsic::sqrt, Intrinsic::experimental_constrained_sqrt);
3136:       SetSqrtFPAccuracy(Call);
3137:       return RValue::get(Call);
3138:     }
3139: 
3140:     case Builtin::BItan:
3141:     case Builtin::BItanf:
3142:     case Builtin::BItanl:
3143:     case Builtin::BI__builtin_tan:
3144:     case Builtin::BI__builtin_tanf:
3145:     case Builtin::BI__builtin_tanf16:
3146:     case Builtin::BI__builtin_tanl:
3147:     case Builtin::BI__builtin_tanf128:
3148:     case Builtin::BI__builtin_elementwise_tan:
3149:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(
3150:           *this, E, Intrinsic::tan, Intrinsic::experimental_constrained_tan));
```
- **EN**: This block defines callable entry points like `emitSincosBuiltin`, `get`, `SetSqrtFPAccuracy`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitSincosBuiltin`, `get`, `SetSqrtFPAccuracy`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3151-3180
```cpp
3151: 
3152:     case Builtin::BItanh:
3153:     case Builtin::BItanhf:
3154:     case Builtin::BItanhl:
3155:     case Builtin::BI__builtin_tanh:
3156:     case Builtin::BI__builtin_tanhf:
3157:     case Builtin::BI__builtin_tanhf16:
3158:     case Builtin::BI__builtin_tanhl:
3159:     case Builtin::BI__builtin_tanhf128:
3160:     case Builtin::BI__builtin_elementwise_tanh:
3161:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(
3162:           *this, E, Intrinsic::tanh, Intrinsic::experimental_constrained_tanh));
3163: 
3164:     case Builtin::BItrunc:
3165:     case Builtin::BItruncf:
3166:     case Builtin::BItruncl:
3167:     case Builtin::BI__builtin_trunc:
3168:     case Builtin::BI__builtin_truncf:
3169:     case Builtin::BI__builtin_truncf16:
3170:     case Builtin::BI__builtin_truncl:
3171:     case Builtin::BI__builtin_truncf128:
3172:     case Builtin::BI__builtin_elementwise_trunc:
3173:       return RValue::get(emitUnaryMaybeConstrainedFPBuiltin(*this, E,
3174:                                    Intrinsic::trunc,
3175:                                    Intrinsic::experimental_constrained_trunc));
3176: 
3177:     case Builtin::BIlround:
3178:     case Builtin::BIlroundf:
3179:     case Builtin::BIlroundl:
3180:     case Builtin::BI__builtin_lround:
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3181-3210
```cpp
3181:     case Builtin::BI__builtin_lroundf:
3182:     case Builtin::BI__builtin_lroundl:
3183:     case Builtin::BI__builtin_lroundf128:
3184:       return RValue::get(emitMaybeConstrainedFPToIntRoundBuiltin(
3185:           *this, E, Intrinsic::lround,
3186:           Intrinsic::experimental_constrained_lround));
3187: 
3188:     case Builtin::BIllround:
3189:     case Builtin::BIllroundf:
3190:     case Builtin::BIllroundl:
3191:     case Builtin::BI__builtin_llround:
3192:     case Builtin::BI__builtin_llroundf:
3193:     case Builtin::BI__builtin_llroundl:
3194:     case Builtin::BI__builtin_llroundf128:
3195:       return RValue::get(emitMaybeConstrainedFPToIntRoundBuiltin(
3196:           *this, E, Intrinsic::llround,
3197:           Intrinsic::experimental_constrained_llround));
3198: 
3199:     case Builtin::BIlrint:
3200:     case Builtin::BIlrintf:
3201:     case Builtin::BIlrintl:
3202:     case Builtin::BI__builtin_lrint:
3203:     case Builtin::BI__builtin_lrintf:
3204:     case Builtin::BI__builtin_lrintl:
3205:     case Builtin::BI__builtin_lrintf128:
3206:       return RValue::get(emitMaybeConstrainedFPToIntRoundBuiltin(
3207:           *this, E, Intrinsic::lrint,
3208:           Intrinsic::experimental_constrained_lrint));
3209: 
3210:     case Builtin::BIllrint:
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3211-3240
```cpp
3211:     case Builtin::BIllrintf:
3212:     case Builtin::BIllrintl:
3213:     case Builtin::BI__builtin_llrint:
3214:     case Builtin::BI__builtin_llrintf:
3215:     case Builtin::BI__builtin_llrintl:
3216:     case Builtin::BI__builtin_llrintf128:
3217:       return RValue::get(emitMaybeConstrainedFPToIntRoundBuiltin(
3218:           *this, E, Intrinsic::llrint,
3219:           Intrinsic::experimental_constrained_llrint));
3220:     case Builtin::BI__builtin_ldexp:
3221:     case Builtin::BI__builtin_ldexpf:
3222:     case Builtin::BI__builtin_ldexpl:
3223:     case Builtin::BI__builtin_ldexpf16:
3224:     case Builtin::BI__builtin_ldexpf128:
3225:     case Builtin::BI__builtin_elementwise_ldexp:
3226:       return RValue::get(emitBinaryExpMaybeConstrainedFPBuiltin(
3227:           *this, E, Intrinsic::ldexp,
3228:           Intrinsic::experimental_constrained_ldexp));
3229:     default:
3230:       break;
3231:     }
3232:   }
3233: 
3234:   // Check NonnullAttribute/NullabilityArg and Alignment.
3235:   auto EmitArgCheck = [&](TypeCheckKind Kind, Address A, const Expr *Arg,
3236:                           unsigned ParmNum) {
3237:     Value *Val = A.emitRawPointer(*this);
3238:     EmitNonNullArgCheck(RValue::get(Val), Arg->getType(), Arg->getExprLoc(), FD,
3239:                         ParmNum);
3240: 
```
- **EN**: This block defines callable entry points like `get`, `EmitNonNullArgCheck`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `EmitNonNullArgCheck`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3241-3270
```cpp
3241:     if (SanOpts.has(SanitizerKind::Alignment)) {
3242:       SanitizerSet SkippedChecks;
3243:       SkippedChecks.set(SanitizerKind::All);
3244:       SkippedChecks.clear(SanitizerKind::Alignment);
3245:       SourceLocation Loc = Arg->getExprLoc();
3246:       // Strip an implicit cast.
3247:       if (auto *CE = dyn_cast<ImplicitCastExpr>(Arg))
3248:         if (CE->getCastKind() == CK_BitCast)
3249:           Arg = CE->getSubExpr();
3250:       EmitTypeCheck(Kind, Loc, Val, Arg->getType(), A.getAlignment(),
3251:                     SkippedChecks);
3252:     }
3253:   };
3254: 
3255:   switch (BuiltinIDIfNoAsmLabel) {
3256:   default: break;
3257:   case Builtin::BI__builtin___CFStringMakeConstantString:
3258:   case Builtin::BI__builtin___NSStringMakeConstantString:
3259:     return RValue::get(ConstantEmitter(*this).emitAbstract(E, E->getType()));
3260:   case Builtin::BI__builtin_stdarg_start:
3261:   case Builtin::BI__builtin_va_start:
3262:   case Builtin::BI__va_start:
3263:   case Builtin::BI__builtin_c23_va_start:
3264:   case Builtin::BI__builtin_va_end:
3265:     EmitVAStartEnd(BuiltinID == Builtin::BI__va_start
3266:                        ? EmitScalarExpr(E->getArg(0))
3267:                        : EmitVAListRef(E->getArg(0)).emitRawPointer(*this),
3268:                    BuiltinID != Builtin::BI__builtin_va_end);
3269:     return RValue::get(nullptr);
3270:   case Builtin::BI__builtin_va_copy: {
```
- **EN**: This block defines callable entry points like `EmitTypeCheck`, `get`, `EmitVAStartEnd`; uses control flow (if, switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitTypeCheck`, `get`, `EmitVAStartEnd`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为。

### Lines 3271-3300
```cpp
3271:     Value *DstPtr = EmitVAListRef(E->getArg(0)).emitRawPointer(*this);
3272:     Value *SrcPtr = EmitVAListRef(E->getArg(1)).emitRawPointer(*this);
3273:     Builder.CreateCall(CGM.getIntrinsic(Intrinsic::vacopy, {DstPtr->getType()}),
3274:                        {DstPtr, SrcPtr});
3275:     return RValue::get(nullptr);
3276:   }
3277:   case Builtin::BIabs:
3278:   case Builtin::BIlabs:
3279:   case Builtin::BIllabs:
3280:   case Builtin::BI__builtin_abs:
3281:   case Builtin::BI__builtin_labs:
3282:   case Builtin::BI__builtin_llabs: {
3283:     bool SanitizeOverflow = SanOpts.has(SanitizerKind::SignedIntegerOverflow);
3284: 
3285:     Value *Result;
3286:     switch (getLangOpts().getSignedOverflowBehavior()) {
3287:     case LangOptions::SOB_Defined:
3288:       Result = EmitAbs(*this, EmitScalarExpr(E->getArg(0)), false);
3289:       break;
3290:     case LangOptions::SOB_Undefined:
3291:       if (!SanitizeOverflow) {
3292:         Result = EmitAbs(*this, EmitScalarExpr(E->getArg(0)), true);
3293:         break;
3294:       }
3295:       [[fallthrough]];
3296:     case LangOptions::SOB_Trapping:
3297:       // TODO: Somehow handle the corner case when the address of abs is taken.
3298:       Result = EmitOverflowCheckedAbs(*this, E, SanitizeOverflow);
3299:       break;
3300:     }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为。

### Lines 3301-3330
```cpp
3301:     return RValue::get(Result);
3302:   }
3303:   case Builtin::BI__builtin_complex: {
3304:     Value *Real = EmitScalarExpr(E->getArg(0));
3305:     Value *Imag = EmitScalarExpr(E->getArg(1));
3306:     return RValue::getComplex({Real, Imag});
3307:   }
3308:   case Builtin::BI__builtin_conj:
3309:   case Builtin::BI__builtin_conjf:
3310:   case Builtin::BI__builtin_conjl:
3311:   case Builtin::BIconj:
3312:   case Builtin::BIconjf:
3313:   case Builtin::BIconjl: {
3314:     ComplexPairTy ComplexVal = EmitComplexExpr(E->getArg(0));
3315:     Value *Real = ComplexVal.first;
3316:     Value *Imag = ComplexVal.second;
3317:     Imag = Builder.CreateFNeg(Imag, "neg");
3318:     return RValue::getComplex(std::make_pair(Real, Imag));
3319:   }
3320:   case Builtin::BI__builtin_creal:
3321:   case Builtin::BI__builtin_crealf:
3322:   case Builtin::BI__builtin_creall:
3323:   case Builtin::BIcreal:
3324:   case Builtin::BIcrealf:
3325:   case Builtin::BIcreall: {
3326:     ComplexPairTy ComplexVal = EmitComplexExpr(E->getArg(0));
3327:     return RValue::get(ComplexVal.first);
3328:   }
3329: 
3330:   case Builtin::BI__builtin_preserve_access_index: {
```
- **EN**: This block defines callable entry points like `get`, `getComplex`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getComplex`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3331-3360
```cpp
3331:     // Only enabled preserved access index region when debuginfo
3332:     // is available as debuginfo is needed to preserve user-level
3333:     // access pattern.
3334:     if (!getDebugInfo()) {
3335:       CGM.Error(E->getExprLoc(), "using builtin_preserve_access_index() without -g");
3336:       return RValue::get(EmitScalarExpr(E->getArg(0)));
3337:     }
3338: 
3339:     // Nested builtin_preserve_access_index() not supported
3340:     if (IsInPreservedAIRegion) {
3341:       CGM.Error(E->getExprLoc(), "nested builtin_preserve_access_index() not supported");
3342:       return RValue::get(EmitScalarExpr(E->getArg(0)));
3343:     }
3344: 
3345:     IsInPreservedAIRegion = true;
3346:     Value *Res = EmitScalarExpr(E->getArg(0));
3347:     IsInPreservedAIRegion = false;
3348:     return RValue::get(Res);
3349:   }
3350: 
3351:   case Builtin::BI__builtin_cimag:
3352:   case Builtin::BI__builtin_cimagf:
3353:   case Builtin::BI__builtin_cimagl:
3354:   case Builtin::BIcimag:
3355:   case Builtin::BIcimagf:
3356:   case Builtin::BIcimagl: {
3357:     ComplexPairTy ComplexVal = EmitComplexExpr(E->getArg(0));
3358:     return RValue::get(ComplexVal.second);
3359:   }
3360: 
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3361-3390
```cpp
3361:   case Builtin::BI__builtin_clrsb:
3362:   case Builtin::BI__builtin_clrsbl:
3363:   case Builtin::BI__builtin_clrsbll: {
3364:     // clrsb(x) -> clz(x < 0 ? ~x : x) - 1 or
3365:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
3366: 
3367:     llvm::Type *ArgType = ArgValue->getType();
3368:     Function *F = CGM.getIntrinsic(Intrinsic::ctlz, ArgType);
3369: 
3370:     llvm::Type *ResultType = ConvertType(E->getType());
3371:     Value *Zero = llvm::Constant::getNullValue(ArgType);
3372:     Value *IsNeg = Builder.CreateICmpSLT(ArgValue, Zero, "isneg");
3373:     Value *Inverse = Builder.CreateNot(ArgValue, "not");
3374:     Value *Tmp = Builder.CreateSelect(IsNeg, Inverse, ArgValue);
3375:     Value *Ctlz = Builder.CreateCall(F, {Tmp, Builder.getFalse()});
3376:     Value *Result =
3377:         Builder.CreateNUWSub(Ctlz, llvm::ConstantInt::get(ArgType, 1));
3378:     Result = Builder.CreateIntCast(Result, ResultType, /*isSigned*/true,
3379:                                    "cast");
3380:     return RValue::get(Result);
3381:   }
3382:   case Builtin::BI__builtin_ctzs:
3383:   case Builtin::BI__builtin_ctz:
3384:   case Builtin::BI__builtin_ctzl:
3385:   case Builtin::BI__builtin_ctzll:
3386:   case Builtin::BI__builtin_ctzg:
3387:   case Builtin::BI__builtin_elementwise_ctzg: {
3388:     bool HasFallback =
3389:         (BuiltinIDIfNoAsmLabel == Builtin::BI__builtin_ctzg ||
3390:          BuiltinIDIfNoAsmLabel == Builtin::BI__builtin_elementwise_ctzg) &&
```
- **EN**: This block defines callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3391-3420
```cpp
3391:         E->getNumArgs() > 1;
3392: 
3393:     Value *ArgValue =
3394:         HasFallback ? EmitBitCountExpr(*this, E->getArg(0))
3395:                     : EmitCheckedArgForBuiltin(E->getArg(0), BCK_CTZPassedZero);
3396: 
3397:     llvm::Type *ArgType = ArgValue->getType();
3398:     Function *F = CGM.getIntrinsic(Intrinsic::cttz, ArgType);
3399: 
3400:     llvm::Type *ResultType = ConvertType(E->getType());
3401:     // The elementwise builtins always exhibit zero-is-undef behaviour
3402:     Value *ZeroUndef = Builder.getInt1(
3403:         HasFallback || getTarget().isCLZForZeroUndef() ||
3404:         BuiltinIDIfNoAsmLabel == Builtin::BI__builtin_elementwise_ctzg);
3405:     Value *Result = Builder.CreateCall(F, {ArgValue, ZeroUndef});
3406:     if (Result->getType() != ResultType)
3407:       Result =
3408:           Builder.CreateIntCast(Result, ResultType, /*isSigned*/ false, "cast");
3409:     if (!HasFallback)
3410:       return RValue::get(Result);
3411: 
3412:     Value *Zero = Constant::getNullValue(ArgType);
3413:     Value *IsZero = Builder.CreateICmpEQ(ArgValue, Zero, "iszero");
3414:     Value *FallbackValue = EmitScalarExpr(E->getArg(1));
3415:     Value *ResultOrFallback =
3416:         Builder.CreateSelect(IsZero, FallbackValue, Result, "ctzg");
3417:     return RValue::get(ResultOrFallback);
3418:   }
3419:   case Builtin::BI__builtin_clzs:
3420:   case Builtin::BI__builtin_clz:
```
- **EN**: This block defines callable entry points like `EmitCheckedArgForBuiltin`, `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitCheckedArgForBuiltin`, `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3421-3450
```cpp
3421:   case Builtin::BI__builtin_clzl:
3422:   case Builtin::BI__builtin_clzll:
3423:   case Builtin::BI__builtin_clzg:
3424:   case Builtin::BI__builtin_elementwise_clzg: {
3425:     bool HasFallback =
3426:         (BuiltinIDIfNoAsmLabel == Builtin::BI__builtin_clzg ||
3427:          BuiltinIDIfNoAsmLabel == Builtin::BI__builtin_elementwise_clzg) &&
3428:         E->getNumArgs() > 1;
3429: 
3430:     Value *ArgValue =
3431:         HasFallback ? EmitBitCountExpr(*this, E->getArg(0))
3432:                     : EmitCheckedArgForBuiltin(E->getArg(0), BCK_CLZPassedZero);
3433: 
3434:     llvm::Type *ArgType = ArgValue->getType();
3435:     Function *F = CGM.getIntrinsic(Intrinsic::ctlz, ArgType);
3436: 
3437:     llvm::Type *ResultType = ConvertType(E->getType());
3438:     // The elementwise builtins always exhibit zero-is-undef behaviour
3439:     Value *ZeroUndef = Builder.getInt1(
3440:         HasFallback || getTarget().isCLZForZeroUndef() ||
3441:         BuiltinIDIfNoAsmLabel == Builtin::BI__builtin_elementwise_clzg);
3442:     Value *Result = Builder.CreateCall(F, {ArgValue, ZeroUndef});
3443:     if (Result->getType() != ResultType)
3444:       Result =
3445:           Builder.CreateIntCast(Result, ResultType, /*isSigned*/ false, "cast");
3446:     if (!HasFallback)
3447:       return RValue::get(Result);
3448: 
3449:     Value *Zero = Constant::getNullValue(ArgType);
3450:     Value *IsZero = Builder.CreateICmpEQ(ArgValue, Zero, "iszero");
```
- **EN**: This block defines callable entry points like `EmitCheckedArgForBuiltin`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitCheckedArgForBuiltin`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3451-3480
```cpp
3451:     Value *FallbackValue = EmitScalarExpr(E->getArg(1));
3452:     Value *ResultOrFallback =
3453:         Builder.CreateSelect(IsZero, FallbackValue, Result, "clzg");
3454:     return RValue::get(ResultOrFallback);
3455:   }
3456:   case Builtin::BI__builtin_ffs:
3457:   case Builtin::BI__builtin_ffsl:
3458:   case Builtin::BI__builtin_ffsll: {
3459:     // ffs(x) -> x ? cttz(x) + 1 : 0
3460:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
3461: 
3462:     llvm::Type *ArgType = ArgValue->getType();
3463:     Function *F = CGM.getIntrinsic(Intrinsic::cttz, ArgType);
3464: 
3465:     llvm::Type *ResultType = ConvertType(E->getType());
3466:     Value *Tmp =
3467:         Builder.CreateAdd(Builder.CreateCall(F, {ArgValue, Builder.getTrue()}),
3468:                           llvm::ConstantInt::get(ArgType, 1));
3469:     Value *Zero = llvm::Constant::getNullValue(ArgType);
3470:     Value *IsZero = Builder.CreateICmpEQ(ArgValue, Zero, "iszero");
3471:     Value *Result = Builder.CreateSelect(IsZero, Zero, Tmp, "ffs");
3472:     if (Result->getType() != ResultType)
3473:       Result = Builder.CreateIntCast(Result, ResultType, /*isSigned*/true,
3474:                                      "cast");
3475:     return RValue::get(Result);
3476:   }
3477:   case Builtin::BI__builtin_parity:
3478:   case Builtin::BI__builtin_parityl:
3479:   case Builtin::BI__builtin_parityll: {
3480:     // parity(x) -> ctpop(x) & 1
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3481-3510
```cpp
3481:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
3482: 
3483:     llvm::Type *ArgType = ArgValue->getType();
3484:     Function *F = CGM.getIntrinsic(Intrinsic::ctpop, ArgType);
3485: 
3486:     llvm::Type *ResultType = ConvertType(E->getType());
3487:     Value *Tmp = Builder.CreateCall(F, ArgValue);
3488:     Value *Result = Builder.CreateAnd(Tmp, llvm::ConstantInt::get(ArgType, 1));
3489:     if (Result->getType() != ResultType)
3490:       Result = Builder.CreateIntCast(Result, ResultType, /*isSigned*/true,
3491:                                      "cast");
3492:     return RValue::get(Result);
3493:   }
3494:   case Builtin::BI__lzcnt16:
3495:   case Builtin::BI__lzcnt:
3496:   case Builtin::BI__lzcnt64: {
3497:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
3498: 
3499:     llvm::Type *ArgType = ArgValue->getType();
3500:     Function *F = CGM.getIntrinsic(Intrinsic::ctlz, ArgType);
3501: 
3502:     llvm::Type *ResultType = ConvertType(E->getType());
3503:     Value *Result = Builder.CreateCall(F, {ArgValue, Builder.getFalse()});
3504:     if (Result->getType() != ResultType)
3505:       Result = Builder.CreateIntCast(Result, ResultType, /*isSigned*/true,
3506:                                      "cast");
3507:     return RValue::get(Result);
3508:   }
3509:   case Builtin::BI__popcnt16:
3510:   case Builtin::BI__popcnt:
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3511-3540
```cpp
3511:   case Builtin::BI__popcnt64:
3512:   case Builtin::BI__builtin_popcount:
3513:   case Builtin::BI__builtin_popcountl:
3514:   case Builtin::BI__builtin_popcountll:
3515:   case Builtin::BI__builtin_popcountg: {
3516:     Value *ArgValue = EmitBitCountExpr(*this, E->getArg(0));
3517: 
3518:     llvm::Type *ArgType = ArgValue->getType();
3519:     Function *F = CGM.getIntrinsic(Intrinsic::ctpop, ArgType);
3520: 
3521:     llvm::Type *ResultType = ConvertType(E->getType());
3522:     Value *Result = Builder.CreateCall(F, ArgValue);
3523:     if (Result->getType() != ResultType)
3524:       Result =
3525:           Builder.CreateIntCast(Result, ResultType, /*isSigned*/ false, "cast");
3526:     return RValue::get(Result);
3527:   }
3528:   case Builtin::BI__builtin_unpredictable: {
3529:     // Always return the argument of __builtin_unpredictable. LLVM does not
3530:     // handle this builtin. Metadata for this builtin should be added directly
3531:     // to instructions such as branches or switches that use it.
3532:     return RValue::get(EmitScalarExpr(E->getArg(0)));
3533:   }
3534:   case Builtin::BI__builtin_expect: {
3535:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
3536:     llvm::Type *ArgType = ArgValue->getType();
3537: 
3538:     Value *ExpectedValue = EmitScalarExpr(E->getArg(1));
3539:     // Don't generate llvm.expect on -O0 as the backend won't use it for
3540:     // anything.
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3541-3570
```cpp
3541:     // Note, we still IRGen ExpectedValue because it could have side-effects.
3542:     if (CGM.getCodeGenOpts().OptimizationLevel == 0)
3543:       return RValue::get(ArgValue);
3544: 
3545:     Function *FnExpect = CGM.getIntrinsic(Intrinsic::expect, ArgType);
3546:     Value *Result =
3547:         Builder.CreateCall(FnExpect, {ArgValue, ExpectedValue}, "expval");
3548:     return RValue::get(Result);
3549:   }
3550:   case Builtin::BI__builtin_expect_with_probability: {
3551:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
3552:     llvm::Type *ArgType = ArgValue->getType();
3553: 
3554:     Value *ExpectedValue = EmitScalarExpr(E->getArg(1));
3555:     llvm::APFloat Probability(0.0);
3556:     const Expr *ProbArg = E->getArg(2);
3557:     bool EvalSucceed = ProbArg->EvaluateAsFloat(Probability, CGM.getContext());
3558:     assert(EvalSucceed && "probability should be able to evaluate as float");
3559:     (void)EvalSucceed;
3560:     bool LoseInfo = false;
3561:     Probability.convert(llvm::APFloat::IEEEdouble(),
3562:                         llvm::RoundingMode::Dynamic, &LoseInfo);
3563:     llvm::Type *Ty = ConvertType(ProbArg->getType());
3564:     Constant *Confidence = ConstantFP::get(Ty, Probability);
3565:     // Don't generate llvm.expect.with.probability on -O0 as the backend
3566:     // won't use it for anything.
3567:     // Note, we still IRGen ExpectedValue because it could have side-effects.
3568:     if (CGM.getCodeGenOpts().OptimizationLevel == 0)
3569:       return RValue::get(ArgValue);
3570: 
```
- **EN**: This block defines callable entry points like `get`, `Probability`; uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`, `Probability`；通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 3571-3600
```cpp
3571:     Function *FnExpect =
3572:         CGM.getIntrinsic(Intrinsic::expect_with_probability, ArgType);
3573:     Value *Result = Builder.CreateCall(
3574:         FnExpect, {ArgValue, ExpectedValue, Confidence}, "expval");
3575:     return RValue::get(Result);
3576:   }
3577:   case Builtin::BI__builtin_assume_aligned: {
3578:     const Expr *Ptr = E->getArg(0);
3579:     Value *PtrValue = EmitScalarExpr(Ptr);
3580:     Value *OffsetValue =
3581:       (E->getNumArgs() > 2) ? EmitScalarExpr(E->getArg(2)) : nullptr;
3582: 
3583:     Value *AlignmentValue = EmitScalarExpr(E->getArg(1));
3584:     ConstantInt *AlignmentCI = cast<ConstantInt>(AlignmentValue);
3585:     if (AlignmentCI->getValue().ugt(llvm::Value::MaximumAlignment))
3586:       AlignmentCI = ConstantInt::get(AlignmentCI->getIntegerType(),
3587:                                      llvm::Value::MaximumAlignment);
3588: 
3589:     emitAlignmentAssumption(PtrValue, Ptr,
3590:                             /*The expr loc is sufficient.*/ SourceLocation(),
3591:                             AlignmentCI, OffsetValue);
3592:     return RValue::get(PtrValue);
3593:   }
3594:   case Builtin::BI__builtin_assume_dereferenceable: {
3595:     const Expr *Ptr = E->getArg(0);
3596:     const Expr *Size = E->getArg(1);
3597:     Value *PtrValue = EmitScalarExpr(Ptr);
3598:     Value *SizeValue = EmitScalarExpr(Size);
3599:     if (SizeValue->getType() != IntPtrTy)
3600:       SizeValue =
```
- **EN**: This block defines callable entry points like `get`, `emitAlignmentAssumption`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `emitAlignmentAssumption`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3601-3630
```cpp
3601:           Builder.CreateIntCast(SizeValue, IntPtrTy, false, "casted.size");
3602:     Builder.CreateDereferenceableAssumption(PtrValue, SizeValue);
3603:     return RValue::get(nullptr);
3604:   }
3605:   case Builtin::BI__assume:
3606:   case Builtin::BI__builtin_assume: {
3607:     if (E->getArg(0)->HasSideEffects(getContext()))
3608:       return RValue::get(nullptr);
3609: 
3610:     Value *ArgValue = EmitCheckedArgForAssume(E->getArg(0));
3611:     Function *FnAssume = CGM.getIntrinsic(Intrinsic::assume);
3612:     Builder.CreateCall(FnAssume, ArgValue);
3613:     return RValue::get(nullptr);
3614:   }
3615:   case Builtin::BI__builtin_assume_separate_storage: {
3616:     const Expr *Arg0 = E->getArg(0);
3617:     const Expr *Arg1 = E->getArg(1);
3618: 
3619:     Value *Value0 = EmitScalarExpr(Arg0);
3620:     Value *Value1 = EmitScalarExpr(Arg1);
3621: 
3622:     Value *Values[] = {Value0, Value1};
3623:     OperandBundleDefT<Value *> OBD("separate_storage", Values);
3624:     Builder.CreateAssumption({OBD});
3625:     return RValue::get(nullptr);
3626:   }
3627:   case Builtin::BI__builtin_allow_runtime_check: {
3628:     StringRef Kind =
3629:         cast<StringLiteral>(E->getArg(0)->IgnoreParenCasts())->getString();
3630:     LLVMContext &Ctx = CGM.getLLVMContext();
```
- **EN**: This block defines callable entry points like `get`, `OBD`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `OBD`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3631-3660
```cpp
3631:     llvm::Value *Allow = Builder.CreateCall(
3632:         CGM.getIntrinsic(Intrinsic::allow_runtime_check),
3633:         llvm::MetadataAsValue::get(Ctx, llvm::MDString::get(Ctx, Kind)));
3634:     return RValue::get(Allow);
3635:   }
3636:   case Builtin::BI__builtin_allow_sanitize_check: {
3637:     Intrinsic::ID IntrID = Intrinsic::not_intrinsic;
3638:     StringRef Name =
3639:         cast<StringLiteral>(E->getArg(0)->IgnoreParenCasts())->getString();
3640: 
3641:     // We deliberately allow the use of kernel- and non-kernel names
3642:     // interchangably, even when one or the other is enabled. This is consistent
3643:     // with the no_sanitize-attribute, which allows either kernel- or non-kernel
3644:     // name to disable instrumentation (see CodeGenFunction::StartFunction).
3645:     if (getLangOpts().Sanitize.hasOneOf(SanitizerKind::Address |
3646:                                         SanitizerKind::KernelAddress) &&
3647:         (Name == "address" || Name == "kernel-address")) {
3648:       IntrID = Intrinsic::allow_sanitize_address;
3649:     } else if (getLangOpts().Sanitize.has(SanitizerKind::Thread) &&
3650:                Name == "thread") {
3651:       IntrID = Intrinsic::allow_sanitize_thread;
3652:     } else if (getLangOpts().Sanitize.hasOneOf(SanitizerKind::Memory |
3653:                                                SanitizerKind::KernelMemory) &&
3654:                (Name == "memory" || Name == "kernel-memory")) {
3655:       IntrID = Intrinsic::allow_sanitize_memory;
3656:     } else if (getLangOpts().Sanitize.hasOneOf(
3657:                    SanitizerKind::HWAddress | SanitizerKind::KernelHWAddress) &&
3658:                (Name == "hwaddress" || Name == "kernel-hwaddress")) {
3659:       IntrID = Intrinsic::allow_sanitize_hwaddress;
3660:     }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3661-3690
```cpp
3661: 
3662:     if (IntrID != Intrinsic::not_intrinsic) {
3663:       llvm::Value *Allow = Builder.CreateCall(CGM.getIntrinsic(IntrID));
3664:       return RValue::get(Allow);
3665:     }
3666:     // If the checked sanitizer is not enabled, we can safely lower to false
3667:     // right away. This is also more efficient, since the LowerAllowCheckPass
3668:     // must not always be enabled if none of the above sanitizers are enabled.
3669:     return RValue::get(Builder.getFalse());
3670:   }
3671:   case Builtin::BI__arithmetic_fence: {
3672:     // Create the builtin call if FastMath is selected, and the target
3673:     // supports the builtin, otherwise just return the argument.
3674:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
3675:     llvm::FastMathFlags FMF = Builder.getFastMathFlags();
3676:     bool isArithmeticFenceEnabled =
3677:         FMF.allowReassoc() &&
3678:         getContext().getTargetInfo().checkArithmeticFenceSupported();
3679:     QualType ArgType = E->getArg(0)->getType();
3680:     if (ArgType->isComplexType()) {
3681:       if (isArithmeticFenceEnabled) {
3682:         QualType ElementType = ArgType->castAs<ComplexType>()->getElementType();
3683:         ComplexPairTy ComplexVal = EmitComplexExpr(E->getArg(0));
3684:         Value *Real = Builder.CreateArithmeticFence(ComplexVal.first,
3685:                                                     ConvertType(ElementType));
3686:         Value *Imag = Builder.CreateArithmeticFence(ComplexVal.second,
3687:                                                     ConvertType(ElementType));
3688:         return RValue::getComplex(std::make_pair(Real, Imag));
3689:       }
3690:       ComplexPairTy ComplexVal = EmitComplexExpr(E->getArg(0));
```
- **EN**: This block defines callable entry points like `get`, `FPOptsRAII`, `getContext`, `ConvertType`, `getComplex`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `FPOptsRAII`, `getContext`, `ConvertType`, `getComplex`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3691-3720
```cpp
3691:       Value *Real = ComplexVal.first;
3692:       Value *Imag = ComplexVal.second;
3693:       return RValue::getComplex(std::make_pair(Real, Imag));
3694:     }
3695:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
3696:     if (isArithmeticFenceEnabled)
3697:       return RValue::get(
3698:           Builder.CreateArithmeticFence(ArgValue, ConvertType(ArgType)));
3699:     return RValue::get(ArgValue);
3700:   }
3701:   case Builtin::BI__builtin_bswapg: {
3702:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
3703:     llvm::IntegerType *IntTy = cast<llvm::IntegerType>(ArgValue->getType());
3704:     assert(IntTy && "LLVM's __builtin_bswapg only supports integer variants");
3705:     if (IntTy->getBitWidth() == 1 || IntTy->getBitWidth() == 8)
3706:       return RValue::get(ArgValue);
3707:     assert(((IntTy->getBitWidth() % 16 == 0 && IntTy->getBitWidth() != 0)) &&
3708:            "LLVM's __builtin_bswapg only supports integer variants that has a "
3709:            "multiple of 16 bits as well as a single byte");
3710:     return RValue::get(
3711:         emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::bswap));
3712:   }
3713:   case Builtin::BI__builtin_bswap16:
3714:   case Builtin::BI__builtin_bswap32:
3715:   case Builtin::BI__builtin_bswap64:
3716:   case Builtin::BI_byteswap_ushort:
3717:   case Builtin::BI_byteswap_ulong:
3718:   case Builtin::BI_byteswap_uint64: {
3719:     return RValue::get(
3720:         emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::bswap));
```
- **EN**: This block defines callable entry points like `getComplex`, `get`; uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getComplex`, `get`；通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 3721-3750
```cpp
3721:   }
3722:   case Builtin::BI__builtin_bitreverseg: {
3723:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
3724:     llvm::IntegerType *IntTy = cast<llvm::IntegerType>(ArgValue->getType());
3725:     assert(IntTy &&
3726:            "LLVM's __builtin_bitreverseg only support integer variants");
3727:     if (IntTy->getBitWidth() == 1)
3728:       return RValue::get(ArgValue);
3729:     return RValue::get(
3730:         emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::bitreverse));
3731:   }
3732:   case Builtin::BI__builtin_bitreverse8:
3733:   case Builtin::BI__builtin_bitreverse16:
3734:   case Builtin::BI__builtin_bitreverse32:
3735:   case Builtin::BI__builtin_bitreverse64: {
3736:     return RValue::get(
3737:         emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::bitreverse));
3738:   }
3739:   case Builtin::BI__builtin_rotateleft8:
3740:   case Builtin::BI__builtin_rotateleft16:
3741:   case Builtin::BI__builtin_rotateleft32:
3742:   case Builtin::BI__builtin_rotateleft64:
3743:   case Builtin::BI__builtin_stdc_rotate_left:
3744:   case Builtin::BIstdc_rotate_left_uc:
3745:   case Builtin::BIstdc_rotate_left_us:
3746:   case Builtin::BIstdc_rotate_left_ui:
3747:   case Builtin::BIstdc_rotate_left_ul:
3748:   case Builtin::BIstdc_rotate_left_ull:
3749:   case Builtin::BI_rotl8: // Microsoft variants of rotate left
3750:   case Builtin::BI_rotl16:
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 3751-3780
```cpp
3751:   case Builtin::BI_rotl:
3752:   case Builtin::BI_lrotl:
3753:   case Builtin::BI_rotl64:
3754:     return emitRotate(E, false);
3755: 
3756:   case Builtin::BI__builtin_rotateright8:
3757:   case Builtin::BI__builtin_rotateright16:
3758:   case Builtin::BI__builtin_rotateright32:
3759:   case Builtin::BI__builtin_rotateright64:
3760:   case Builtin::BI__builtin_stdc_rotate_right:
3761:   case Builtin::BIstdc_rotate_right_uc:
3762:   case Builtin::BIstdc_rotate_right_us:
3763:   case Builtin::BIstdc_rotate_right_ui:
3764:   case Builtin::BIstdc_rotate_right_ul:
3765:   case Builtin::BIstdc_rotate_right_ull:
3766:   case Builtin::BI_rotr8: // Microsoft variants of rotate right
3767:   case Builtin::BI_rotr16:
3768:   case Builtin::BI_rotr:
3769:   case Builtin::BI_lrotr:
3770:   case Builtin::BI_rotr64:
3771:     return emitRotate(E, true);
3772: 
3773:   case Builtin::BIstdc_leading_zeros_uc:
3774:   case Builtin::BIstdc_leading_zeros_us:
3775:   case Builtin::BIstdc_leading_zeros_ui:
3776:   case Builtin::BIstdc_leading_zeros_ul:
3777:   case Builtin::BIstdc_leading_zeros_ull:
3778:   case Builtin::BI__builtin_stdc_leading_zeros:
3779:     return emitStdcCountIntrinsic(E, Intrinsic::ctlz, /*InvertArg=*/false);
3780:   case Builtin::BIstdc_leading_ones_uc:
```
- **EN**: This block spells out callable entry points like `emitRotate`, `emitStdcCountIntrinsic`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `emitRotate`, `emitStdcCountIntrinsic`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3781-3810
```cpp
3781:   case Builtin::BIstdc_leading_ones_us:
3782:   case Builtin::BIstdc_leading_ones_ui:
3783:   case Builtin::BIstdc_leading_ones_ul:
3784:   case Builtin::BIstdc_leading_ones_ull:
3785:   case Builtin::BI__builtin_stdc_leading_ones:
3786:     return emitStdcCountIntrinsic(E, Intrinsic::ctlz, /*InvertArg=*/true);
3787:   case Builtin::BIstdc_trailing_zeros_uc:
3788:   case Builtin::BIstdc_trailing_zeros_us:
3789:   case Builtin::BIstdc_trailing_zeros_ui:
3790:   case Builtin::BIstdc_trailing_zeros_ul:
3791:   case Builtin::BIstdc_trailing_zeros_ull:
3792:   case Builtin::BI__builtin_stdc_trailing_zeros:
3793:     return emitStdcCountIntrinsic(E, Intrinsic::cttz, /*InvertArg=*/false);
3794:   case Builtin::BIstdc_trailing_ones_uc:
3795:   case Builtin::BIstdc_trailing_ones_us:
3796:   case Builtin::BIstdc_trailing_ones_ui:
3797:   case Builtin::BIstdc_trailing_ones_ul:
3798:   case Builtin::BIstdc_trailing_ones_ull:
3799:   case Builtin::BI__builtin_stdc_trailing_ones:
3800:     return emitStdcCountIntrinsic(E, Intrinsic::cttz, /*InvertArg=*/true);
3801:   case Builtin::BIstdc_first_leading_zero_uc:
3802:   case Builtin::BIstdc_first_leading_zero_us:
3803:   case Builtin::BIstdc_first_leading_zero_ui:
3804:   case Builtin::BIstdc_first_leading_zero_ul:
3805:   case Builtin::BIstdc_first_leading_zero_ull:
3806:   case Builtin::BI__builtin_stdc_first_leading_zero:
3807:     return emitStdcFirstBit(E, Intrinsic::ctlz, /*InvertArg=*/true);
3808:   case Builtin::BIstdc_first_leading_one_uc:
3809:   case Builtin::BIstdc_first_leading_one_us:
3810:   case Builtin::BIstdc_first_leading_one_ui:
```
- **EN**: This block spells out callable entry points like `emitStdcCountIntrinsic`, `emitStdcFirstBit`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `emitStdcCountIntrinsic`, `emitStdcFirstBit`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3811-3840
```cpp
3811:   case Builtin::BIstdc_first_leading_one_ul:
3812:   case Builtin::BIstdc_first_leading_one_ull:
3813:   case Builtin::BI__builtin_stdc_first_leading_one:
3814:     return emitStdcFirstBit(E, Intrinsic::ctlz, /*InvertArg=*/false);
3815:   case Builtin::BIstdc_first_trailing_zero_uc:
3816:   case Builtin::BIstdc_first_trailing_zero_us:
3817:   case Builtin::BIstdc_first_trailing_zero_ui:
3818:   case Builtin::BIstdc_first_trailing_zero_ul:
3819:   case Builtin::BIstdc_first_trailing_zero_ull:
3820:   case Builtin::BI__builtin_stdc_first_trailing_zero:
3821:     return emitStdcFirstBit(E, Intrinsic::cttz, /*InvertArg=*/true);
3822:   case Builtin::BIstdc_first_trailing_one_uc:
3823:   case Builtin::BIstdc_first_trailing_one_us:
3824:   case Builtin::BIstdc_first_trailing_one_ui:
3825:   case Builtin::BIstdc_first_trailing_one_ul:
3826:   case Builtin::BIstdc_first_trailing_one_ull:
3827:   case Builtin::BI__builtin_stdc_first_trailing_one:
3828:     return emitStdcFirstBit(E, Intrinsic::cttz, /*InvertArg=*/false);
3829:   case Builtin::BIstdc_count_zeros_uc:
3830:   case Builtin::BIstdc_count_zeros_us:
3831:   case Builtin::BIstdc_count_zeros_ui:
3832:   case Builtin::BIstdc_count_zeros_ul:
3833:   case Builtin::BIstdc_count_zeros_ull:
3834:   case Builtin::BI__builtin_stdc_count_zeros:
3835:     return emitStdcBitWidthMinus(E, Intrinsic::ctpop, /*IsPop=*/true);
3836:   case Builtin::BIstdc_count_ones_uc:
3837:   case Builtin::BIstdc_count_ones_us:
3838:   case Builtin::BIstdc_count_ones_ui:
3839:   case Builtin::BIstdc_count_ones_ul:
3840:   case Builtin::BIstdc_count_ones_ull:
```
- **EN**: This block spells out callable entry points like `emitStdcFirstBit`, `emitStdcBitWidthMinus`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `emitStdcFirstBit`, `emitStdcBitWidthMinus`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3841-3870
```cpp
3841:   case Builtin::BI__builtin_stdc_count_ones:
3842:     return emitStdcCountIntrinsic(E, Intrinsic::ctpop, /*InvertArg=*/false,
3843:                                   /*IsPop=*/true);
3844:   case Builtin::BIstdc_has_single_bit_uc:
3845:   case Builtin::BIstdc_has_single_bit_us:
3846:   case Builtin::BIstdc_has_single_bit_ui:
3847:   case Builtin::BIstdc_has_single_bit_ul:
3848:   case Builtin::BIstdc_has_single_bit_ull:
3849:   case Builtin::BI__builtin_stdc_has_single_bit: {
3850:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
3851:     llvm::Type *ArgType = ArgValue->getType();
3852:     Value *One = ConstantInt::get(ArgType, 1);
3853:     Function *F = CGM.getIntrinsic(Intrinsic::ctpop, ArgType);
3854:     Value *PopCnt = Builder.CreateCall(F, ArgValue);
3855:     return RValue::get(Builder.CreateICmpEQ(PopCnt, One));
3856:   }
3857:   case Builtin::BIstdc_bit_width_uc:
3858:   case Builtin::BIstdc_bit_width_us:
3859:   case Builtin::BIstdc_bit_width_ui:
3860:   case Builtin::BIstdc_bit_width_ul:
3861:   case Builtin::BIstdc_bit_width_ull:
3862:   case Builtin::BI__builtin_stdc_bit_width:
3863:     return emitStdcBitWidthMinus(E, Intrinsic::ctlz, /*IsPop=*/false);
3864:   case Builtin::BIstdc_bit_floor_uc:
3865:   case Builtin::BIstdc_bit_floor_us:
3866:   case Builtin::BIstdc_bit_floor_ui:
3867:   case Builtin::BIstdc_bit_floor_ul:
3868:   case Builtin::BIstdc_bit_floor_ull:
3869:   case Builtin::BI__builtin_stdc_bit_floor: {
3870:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
```
- **EN**: This block defines callable entry points like `get`, `emitStdcBitWidthMinus`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `emitStdcBitWidthMinus`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3871-3900
```cpp
3871:     llvm::Type *ArgType = ArgValue->getType();
3872:     unsigned BitWidth = ArgType->getIntegerBitWidth();
3873:     Value *Zero = ConstantInt::get(ArgType, 0);
3874:     Value *One = ConstantInt::get(ArgType, 1);
3875:     Function *F = CGM.getIntrinsic(Intrinsic::ctlz, ArgType);
3876:     Value *LZ = Builder.CreateCall(F, {ArgValue, Builder.getTrue()});
3877:     Value *ShiftAmt =
3878:         Builder.CreateSub(ConstantInt::get(ArgType, BitWidth - 1), LZ);
3879:     Value *Shifted = Builder.CreateShl(One, ShiftAmt);
3880:     Value *IsZero = Builder.CreateICmpEQ(ArgValue, Zero);
3881:     Value *Result = Builder.CreateSelect(IsZero, Zero, Shifted);
3882:     return RValue::get(Result);
3883:   }
3884:   case Builtin::BIstdc_bit_ceil_uc:
3885:   case Builtin::BIstdc_bit_ceil_us:
3886:   case Builtin::BIstdc_bit_ceil_ui:
3887:   case Builtin::BIstdc_bit_ceil_ul:
3888:   case Builtin::BIstdc_bit_ceil_ull:
3889:   case Builtin::BI__builtin_stdc_bit_ceil: {
3890:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
3891:     llvm::Type *ArgType = ArgValue->getType();
3892:     unsigned BitWidth = ArgType->getIntegerBitWidth();
3893:     Value *One = ConstantInt::get(ArgType, 1);
3894:     Value *Two = ConstantInt::get(ArgType, 2);
3895: 
3896:     Value *IsLEOne = Builder.CreateICmpULE(ArgValue, One, "isleone");
3897: 
3898:     BasicBlock *EntryBB = Builder.GetInsertBlock();
3899:     BasicBlock *CalcBB = createBasicBlock("bitceil.calc", CurFn);
3900:     BasicBlock *MergeBB = createBasicBlock("bitceil.merge", CurFn);
```
- **EN**: This block defines callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 3901-3930
```cpp
3901: 
3902:     Builder.CreateCondBr(IsLEOne, MergeBB, CalcBB);
3903: 
3904:     Builder.SetInsertPoint(CalcBB);
3905:     Function *F = CGM.getIntrinsic(Intrinsic::ctlz, ArgType);
3906:     Value *ArgMinusOne = Builder.CreateSub(ArgValue, One);
3907:     Value *LZ = Builder.CreateCall(F, {ArgMinusOne, Builder.getFalse()});
3908:     // 2<<(BitWidth-1-LZ) to get the next power of two. The shift
3909:     // amount is always in [0, BitWidth-1], so when LZ==0 (argument has its MSB
3910:     // set), the result wraps to 0
3911:     Value *ShiftAmt =
3912:         Builder.CreateSub(ConstantInt::get(ArgType, BitWidth - 1), LZ);
3913:     Value *Tmp = Builder.CreateShl(Two, ShiftAmt);
3914:     Builder.CreateBr(MergeBB);
3915: 
3916:     Builder.SetInsertPoint(MergeBB);
3917:     PHINode *Phi = Builder.CreatePHI(ArgType, 2);
3918:     Phi->addIncoming(One, EntryBB);
3919:     Phi->addIncoming(Tmp, CalcBB);
3920:     return RValue::get(Phi);
3921:   }
3922: 
3923:   case Builtin::BI__builtin_constant_p: {
3924:     llvm::Type *ResultType = ConvertType(E->getType());
3925: 
3926:     const Expr *Arg = E->getArg(0);
3927:     QualType ArgType = Arg->getType();
3928:     // FIXME: The allowance for Obj-C pointers and block pointers is historical
3929:     // and likely a mistake.
3930:     if (!ArgType->isIntegralOrEnumerationType() && !ArgType->isFloatingType() &&
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3931-3960
```cpp
3931:         !ArgType->isObjCObjectPointerType() && !ArgType->isBlockPointerType())
3932:       // Per the GCC documentation, only numeric constants are recognized after
3933:       // inlining.
3934:       return RValue::get(ConstantInt::get(ResultType, 0));
3935: 
3936:     if (Arg->HasSideEffects(getContext()))
3937:       // The argument is unevaluated, so be conservative if it might have
3938:       // side-effects.
3939:       return RValue::get(ConstantInt::get(ResultType, 0));
3940: 
3941:     Value *ArgValue = EmitScalarExpr(Arg);
3942:     if (ArgType->isObjCObjectPointerType()) {
3943:       // Convert Objective-C objects to id because we cannot distinguish between
3944:       // LLVM types for Obj-C classes as they are opaque.
3945:       ArgType = CGM.getContext().getObjCIdType();
3946:       ArgValue = Builder.CreateBitCast(ArgValue, ConvertType(ArgType));
3947:     }
3948:     Function *F =
3949:         CGM.getIntrinsic(Intrinsic::is_constant, ConvertType(ArgType));
3950:     Value *Result = Builder.CreateCall(F, ArgValue);
3951:     if (Result->getType() != ResultType)
3952:       Result = Builder.CreateIntCast(Result, ResultType, /*isSigned*/false);
3953:     return RValue::get(Result);
3954:   }
3955:   case Builtin::BI__builtin_dynamic_object_size:
3956:   case Builtin::BI__builtin_object_size: {
3957:     unsigned Type =
3958:         E->getArg(1)->EvaluateKnownConstInt(getContext()).getZExtValue();
3959:     auto *ResType = cast<llvm::IntegerType>(ConvertType(E->getType()));
3960: 
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3961-3990
```cpp
3961:     // We pass this builtin onto the optimizer so that it can figure out the
3962:     // object size in more complex cases.
3963:     bool IsDynamic = BuiltinID == Builtin::BI__builtin_dynamic_object_size;
3964:     return RValue::get(emitBuiltinObjectSize(E->getArg(0), Type, ResType,
3965:                                              /*EmittedE=*/nullptr, IsDynamic));
3966:   }
3967:   case Builtin::BI__builtin_counted_by_ref: {
3968:     // Default to returning '(void *) 0'.
3969:     llvm::Value *Result = llvm::ConstantPointerNull::get(
3970:         llvm::PointerType::getUnqual(getLLVMContext()));
3971: 
3972:     const Expr *Arg = E->getArg(0)->IgnoreParenImpCasts();
3973: 
3974:     if (auto *UO = dyn_cast<UnaryOperator>(Arg);
3975:         UO && UO->getOpcode() == UO_AddrOf) {
3976:       Arg = UO->getSubExpr()->IgnoreParenImpCasts();
3977: 
3978:       if (auto *ASE = dyn_cast<ArraySubscriptExpr>(Arg))
3979:         Arg = ASE->getBase()->IgnoreParenImpCasts();
3980:     }
3981: 
3982:     if (const MemberExpr *ME = dyn_cast_if_present<MemberExpr>(Arg)) {
3983:       if (auto *CATy =
3984:               ME->getMemberDecl()->getType()->getAs<CountAttributedType>();
3985:           CATy && CATy->getKind() == CountAttributedType::CountedBy) {
3986:         const auto *MemberDecl = cast<FieldDecl>(ME->getMemberDecl());
3987:         if (const FieldDecl *CountFD = MemberDecl->findCountedByField())
3988:           Result = GetCountedByFieldExprGEP(Arg, MemberDecl, CountFD);
3989:         else
3990:           llvm::report_fatal_error("Cannot find the counted_by 'count' field");
```
- **EN**: This block defines callable entry points like `getUnqual`, `report_fatal_error`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getUnqual`, `report_fatal_error`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 3991-4020
```cpp
3991:       }
3992:     }
3993: 
3994:     return RValue::get(Result);
3995:   }
3996:   case Builtin::BI__builtin_prefetch: {
3997:     Value *Locality, *RW, *Address = EmitScalarExpr(E->getArg(0));
3998:     // FIXME: Technically these constants should of type 'int', yes?
3999:     RW = (E->getNumArgs() > 1) ? EmitScalarExpr(E->getArg(1)) :
4000:       llvm::ConstantInt::get(Int32Ty, 0);
4001:     Locality = (E->getNumArgs() > 2) ? EmitScalarExpr(E->getArg(2)) :
4002:       llvm::ConstantInt::get(Int32Ty, 3);
4003:     Value *Data = llvm::ConstantInt::get(Int32Ty, 1);
4004:     Function *F = CGM.getIntrinsic(Intrinsic::prefetch, Address->getType());
4005:     Builder.CreateCall(F, {Address, RW, Locality, Data});
4006:     return RValue::get(nullptr);
4007:   }
4008:   case Builtin::BI__builtin_readcyclecounter: {
4009:     Function *F = CGM.getIntrinsic(Intrinsic::readcyclecounter);
4010:     return RValue::get(Builder.CreateCall(F));
4011:   }
4012:   case Builtin::BI__builtin_readsteadycounter: {
4013:     Function *F = CGM.getIntrinsic(Intrinsic::readsteadycounter);
4014:     return RValue::get(Builder.CreateCall(F));
4015:   }
4016:   case Builtin::BI__builtin___clear_cache: {
4017:     Value *Begin = EmitScalarExpr(E->getArg(0));
4018:     Value *End = EmitScalarExpr(E->getArg(1));
4019:     Function *F = CGM.getIntrinsic(Intrinsic::clear_cache, {CGM.DefaultPtrTy});
4020:     return RValue::get(Builder.CreateCall(F, {Begin, End}));
```
- **EN**: This block defines callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 4021-4050
```cpp
4021:   }
4022:   case Builtin::BI__builtin_trap:
4023:     EmitTrapCall(Intrinsic::trap);
4024:     return RValue::get(nullptr);
4025:   case Builtin::BI__builtin_verbose_trap: {
4026:     llvm::DILocation *TrapLocation = Builder.getCurrentDebugLocation();
4027:     if (getDebugInfo()) {
4028:       TrapLocation = getDebugInfo()->CreateTrapFailureMessageFor(
4029:           TrapLocation, *E->getArg(0)->tryEvaluateString(getContext()),
4030:           *E->getArg(1)->tryEvaluateString(getContext()));
4031:     }
4032:     ApplyDebugLocation ApplyTrapDI(*this, TrapLocation);
4033:     // Currently no attempt is made to prevent traps from being merged.
4034:     EmitTrapCall(Intrinsic::trap);
4035:     return RValue::get(nullptr);
4036:   }
4037:   case Builtin::BI__debugbreak:
4038:     EmitTrapCall(Intrinsic::debugtrap);
4039:     return RValue::get(nullptr);
4040:   case Builtin::BI__builtin_unreachable: {
4041:     EmitUnreachable(E->getExprLoc());
4042: 
4043:     // We do need to preserve an insertion point.
4044:     EmitBlock(createBasicBlock("unreachable.cont"));
4045: 
4046:     return RValue::get(nullptr);
4047:   }
4048: 
4049:   case Builtin::BI__builtin_powi:
4050:   case Builtin::BI__builtin_powif:
```
- **EN**: This block defines callable entry points like `EmitTrapCall`, `get`, `ApplyTrapDI`, `EmitUnreachable`, `EmitBlock`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitTrapCall`, `get`, `ApplyTrapDI`, `EmitUnreachable`, `EmitBlock`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4051-4080
```cpp
4051:   case Builtin::BI__builtin_powil: {
4052:     llvm::Value *Src0 = EmitScalarExpr(E->getArg(0));
4053:     llvm::Value *Src1 = EmitScalarExpr(E->getArg(1));
4054: 
4055:     if (Builder.getIsFPConstrained()) {
4056:       // FIXME: llvm.powi has 2 mangling types,
4057:       // llvm.experimental.constrained.powi has one.
4058:       CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
4059:       Function *F = CGM.getIntrinsic(Intrinsic::experimental_constrained_powi,
4060:                                      Src0->getType());
4061:       return RValue::get(Builder.CreateConstrainedFPCall(F, { Src0, Src1 }));
4062:     }
4063: 
4064:     Function *F = CGM.getIntrinsic(Intrinsic::powi,
4065:                                    { Src0->getType(), Src1->getType() });
4066:     return RValue::get(Builder.CreateCall(F, { Src0, Src1 }));
4067:   }
4068:   case Builtin::BI__builtin_frexpl: {
4069:     // Linux PPC will not be adding additional PPCDoubleDouble support.
4070:     // WIP to switch default to IEEE long double. Will emit libcall for
4071:     // frexpl instead of legalizing this type in the BE.
4072:     if (&getTarget().getLongDoubleFormat() == &llvm::APFloat::PPCDoubleDouble())
4073:       break;
4074:     [[fallthrough]];
4075:   }
4076:   case Builtin::BI__builtin_frexp:
4077:   case Builtin::BI__builtin_frexpf:
4078:   case Builtin::BI__builtin_frexpf128:
4079:   case Builtin::BI__builtin_frexpf16:
4080:     return RValue::get(emitFrexpBuiltin(*this, E, Intrinsic::frexp));
```
- **EN**: This block defines callable entry points like `FPOptsRAII`, `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `FPOptsRAII`, `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4081-4110
```cpp
4081:   case Builtin::BImodf:
4082:   case Builtin::BImodff:
4083:   case Builtin::BImodfl:
4084:   case Builtin::BI__builtin_modf:
4085:   case Builtin::BI__builtin_modff:
4086:   case Builtin::BI__builtin_modfl:
4087:     if (Builder.getIsFPConstrained())
4088:       break; // TODO: Emit constrained modf intrinsic once one exists.
4089:     return RValue::get(emitModfBuiltin(*this, E, Intrinsic::modf));
4090:   case Builtin::BI__builtin_isgreater:
4091:   case Builtin::BI__builtin_isgreaterequal:
4092:   case Builtin::BI__builtin_isless:
4093:   case Builtin::BI__builtin_islessequal:
4094:   case Builtin::BI__builtin_islessgreater:
4095:   case Builtin::BI__builtin_isunordered: {
4096:     // Ordered comparisons: we know the arguments to these are matching scalar
4097:     // floating point values.
4098:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
4099:     Value *LHS = EmitScalarExpr(E->getArg(0));
4100:     Value *RHS = EmitScalarExpr(E->getArg(1));
4101: 
4102:     switch (BuiltinID) {
4103:     default: llvm_unreachable("Unknown ordered comparison");
4104:     case Builtin::BI__builtin_isgreater:
4105:       LHS = Builder.CreateFCmpOGT(LHS, RHS, "cmp");
4106:       break;
4107:     case Builtin::BI__builtin_isgreaterequal:
4108:       LHS = Builder.CreateFCmpOGE(LHS, RHS, "cmp");
4109:       break;
4110:     case Builtin::BI__builtin_isless:
```
- **EN**: This block defines callable entry points like `get`, `FPOptsRAII`; uses control flow (if, switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`, `FPOptsRAII`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 4111-4140
```cpp
4111:       LHS = Builder.CreateFCmpOLT(LHS, RHS, "cmp");
4112:       break;
4113:     case Builtin::BI__builtin_islessequal:
4114:       LHS = Builder.CreateFCmpOLE(LHS, RHS, "cmp");
4115:       break;
4116:     case Builtin::BI__builtin_islessgreater:
4117:       LHS = Builder.CreateFCmpONE(LHS, RHS, "cmp");
4118:       break;
4119:     case Builtin::BI__builtin_isunordered:
4120:       LHS = Builder.CreateFCmpUNO(LHS, RHS, "cmp");
4121:       break;
4122:     }
4123:     // ZExt bool to int type.
4124:     return RValue::get(Builder.CreateZExt(LHS, ConvertType(E->getType())));
4125:   }
4126: 
4127:   case Builtin::BI__builtin_isnan: {
4128:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
4129:     Value *V = EmitScalarExpr(E->getArg(0));
4130:     if (Value *Result = tryUseTestFPKind(*this, BuiltinID, V))
4131:       return RValue::get(Result);
4132:     return RValue::get(
4133:         Builder.CreateZExt(Builder.createIsFPClass(V, FPClassTest::fcNan),
4134:                            ConvertType(E->getType())));
4135:   }
4136: 
4137:   case Builtin::BI__builtin_issignaling: {
4138:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
4139:     Value *V = EmitScalarExpr(E->getArg(0));
4140:     return RValue::get(
```
- **EN**: This block defines callable entry points like `get`, `FPOptsRAII`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `FPOptsRAII`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4141-4170
```cpp
4141:         Builder.CreateZExt(Builder.createIsFPClass(V, FPClassTest::fcSNan),
4142:                            ConvertType(E->getType())));
4143:   }
4144: 
4145:   case Builtin::BI__builtin_isinf: {
4146:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
4147:     Value *V = EmitScalarExpr(E->getArg(0));
4148:     if (Value *Result = tryUseTestFPKind(*this, BuiltinID, V))
4149:       return RValue::get(Result);
4150:     return RValue::get(
4151:         Builder.CreateZExt(Builder.createIsFPClass(V, FPClassTest::fcInf),
4152:                            ConvertType(E->getType())));
4153:   }
4154: 
4155:   case Builtin::BIfinite:
4156:   case Builtin::BI__finite:
4157:   case Builtin::BIfinitef:
4158:   case Builtin::BI__finitef:
4159:   case Builtin::BIfinitel:
4160:   case Builtin::BI__finitel:
4161:   case Builtin::BI__builtin_isfinite: {
4162:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
4163:     Value *V = EmitScalarExpr(E->getArg(0));
4164:     if (Value *Result = tryUseTestFPKind(*this, BuiltinID, V))
4165:       return RValue::get(Result);
4166:     return RValue::get(
4167:         Builder.CreateZExt(Builder.createIsFPClass(V, FPClassTest::fcFinite),
4168:                            ConvertType(E->getType())));
4169:   }
4170: 
```
- **EN**: This block defines callable entry points like `ConvertType`, `FPOptsRAII`, `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `ConvertType`, `FPOptsRAII`, `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4171-4200
```cpp
4171:   case Builtin::BI__builtin_isnormal: {
4172:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
4173:     Value *V = EmitScalarExpr(E->getArg(0));
4174:     return RValue::get(
4175:         Builder.CreateZExt(Builder.createIsFPClass(V, FPClassTest::fcNormal),
4176:                            ConvertType(E->getType())));
4177:   }
4178: 
4179:   case Builtin::BI__builtin_issubnormal: {
4180:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
4181:     Value *V = EmitScalarExpr(E->getArg(0));
4182:     return RValue::get(
4183:         Builder.CreateZExt(Builder.createIsFPClass(V, FPClassTest::fcSubnormal),
4184:                            ConvertType(E->getType())));
4185:   }
4186: 
4187:   case Builtin::BI__builtin_iszero: {
4188:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
4189:     Value *V = EmitScalarExpr(E->getArg(0));
4190:     return RValue::get(
4191:         Builder.CreateZExt(Builder.createIsFPClass(V, FPClassTest::fcZero),
4192:                            ConvertType(E->getType())));
4193:   }
4194: 
4195:   case Builtin::BI__builtin_isfpclass: {
4196:     Expr::EvalResult Result;
4197:     if (!E->getArg(1)->EvaluateAsInt(Result, CGM.getContext()))
4198:       break;
4199:     uint64_t Test = Result.Val.getInt().getLimitedValue();
4200:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
```
- **EN**: This block defines callable entry points like `FPOptsRAII`, `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `FPOptsRAII`, `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4201-4230
```cpp
4201:     Value *V = EmitScalarExpr(E->getArg(0));
4202:     return RValue::get(Builder.CreateZExt(Builder.createIsFPClass(V, Test),
4203:                                           ConvertType(E->getType())));
4204:   }
4205: 
4206:   case Builtin::BI__builtin_nondeterministic_value: {
4207:     llvm::Type *Ty = ConvertType(E->getArg(0)->getType());
4208: 
4209:     Value *Result = PoisonValue::get(Ty);
4210:     Result = Builder.CreateFreeze(Result);
4211: 
4212:     return RValue::get(Result);
4213:   }
4214: 
4215:   case Builtin::BI__builtin_elementwise_abs: {
4216:     Value *Result;
4217:     QualType QT = E->getArg(0)->getType();
4218: 
4219:     if (auto *VecTy = QT->getAs<VectorType>())
4220:       QT = VecTy->getElementType();
4221:     if (QT->isIntegerType())
4222:       Result = Builder.CreateBinaryIntrinsic(
4223:           Intrinsic::abs, EmitScalarExpr(E->getArg(0)), Builder.getFalse(),
4224:           nullptr, "elt.abs");
4225:     else
4226:       Result = emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::fabs,
4227:                                                    "elt.abs");
4228: 
4229:     return RValue::get(Result);
4230:   }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4231-4260
```cpp
4231:   case Builtin::BI__builtin_elementwise_bitreverse:
4232:     return RValue::get(emitBuiltinWithOneOverloadedType<1>(
4233:         *this, E, Intrinsic::bitreverse, "elt.bitreverse"));
4234:   case Builtin::BI__builtin_elementwise_popcount:
4235:     return RValue::get(emitBuiltinWithOneOverloadedType<1>(
4236:         *this, E, Intrinsic::ctpop, "elt.ctpop"));
4237:   case Builtin::BI__builtin_elementwise_canonicalize:
4238:     return RValue::get(emitBuiltinWithOneOverloadedType<1>(
4239:         *this, E, Intrinsic::canonicalize, "elt.canonicalize"));
4240:   case Builtin::BI__builtin_elementwise_copysign:
4241:     return RValue::get(
4242:         emitBuiltinWithOneOverloadedType<2>(*this, E, Intrinsic::copysign));
4243:   case Builtin::BI__builtin_elementwise_fshl:
4244:     return RValue::get(
4245:         emitBuiltinWithOneOverloadedType<3>(*this, E, Intrinsic::fshl));
4246:   case Builtin::BI__builtin_elementwise_fshr:
4247:     return RValue::get(
4248:         emitBuiltinWithOneOverloadedType<3>(*this, E, Intrinsic::fshr));
4249: 
4250:   case Builtin::BI__builtin_elementwise_add_sat:
4251:   case Builtin::BI__builtin_elementwise_sub_sat: {
4252:     Value *Op0 = EmitScalarExpr(E->getArg(0));
4253:     Value *Op1 = EmitScalarExpr(E->getArg(1));
4254:     Value *Result;
4255:     assert(Op0->getType()->isIntOrIntVectorTy() && "integer type expected");
4256:     QualType Ty = E->getArg(0)->getType();
4257:     if (auto *VecTy = Ty->getAs<VectorType>())
4258:       Ty = VecTy->getElementType();
4259:     bool IsSigned = Ty->isSignedIntegerType();
4260:     unsigned Opc;
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 4261-4290
```cpp
4261:     if (BuiltinIDIfNoAsmLabel == Builtin::BI__builtin_elementwise_add_sat)
4262:       Opc = IsSigned ? Intrinsic::sadd_sat : Intrinsic::uadd_sat;
4263:     else
4264:       Opc = IsSigned ? Intrinsic::ssub_sat : Intrinsic::usub_sat;
4265:     Result = Builder.CreateBinaryIntrinsic(Opc, Op0, Op1, nullptr, "elt.sat");
4266:     return RValue::get(Result);
4267:   }
4268: 
4269:   case Builtin::BI__builtin_elementwise_max: {
4270:     Value *Op0 = EmitScalarExpr(E->getArg(0));
4271:     Value *Op1 = EmitScalarExpr(E->getArg(1));
4272:     Value *Result;
4273:     if (Op0->getType()->isIntOrIntVectorTy()) {
4274:       QualType Ty = E->getArg(0)->getType();
4275:       if (auto *VecTy = Ty->getAs<VectorType>())
4276:         Ty = VecTy->getElementType();
4277:       Result = Builder.CreateBinaryIntrinsic(
4278:           Ty->isSignedIntegerType() ? Intrinsic::smax : Intrinsic::umax, Op0,
4279:           Op1, nullptr, "elt.max");
4280:     } else
4281:       Result = Builder.CreateMaxNum(Op0, Op1, /*FMFSource=*/nullptr, "elt.max");
4282:     return RValue::get(Result);
4283:   }
4284:   case Builtin::BI__builtin_elementwise_min: {
4285:     Value *Op0 = EmitScalarExpr(E->getArg(0));
4286:     Value *Op1 = EmitScalarExpr(E->getArg(1));
4287:     Value *Result;
4288:     if (Op0->getType()->isIntOrIntVectorTy()) {
4289:       QualType Ty = E->getArg(0)->getType();
4290:       if (auto *VecTy = Ty->getAs<VectorType>())
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4291-4320
```cpp
4291:         Ty = VecTy->getElementType();
4292:       Result = Builder.CreateBinaryIntrinsic(
4293:           Ty->isSignedIntegerType() ? Intrinsic::smin : Intrinsic::umin, Op0,
4294:           Op1, nullptr, "elt.min");
4295:     } else
4296:       Result = Builder.CreateMinNum(Op0, Op1, /*FMFSource=*/nullptr, "elt.min");
4297:     return RValue::get(Result);
4298:   }
4299: 
4300:   case Builtin::BI__builtin_elementwise_maxnum: {
4301:     Value *Op0 = EmitScalarExpr(E->getArg(0));
4302:     Value *Op1 = EmitScalarExpr(E->getArg(1));
4303:     Value *Result = Builder.CreateBinaryIntrinsic(llvm::Intrinsic::maxnum, Op0,
4304:                                                   Op1, nullptr, "elt.maxnum");
4305:     return RValue::get(Result);
4306:   }
4307: 
4308:   case Builtin::BI__builtin_elementwise_minnum: {
4309:     Value *Op0 = EmitScalarExpr(E->getArg(0));
4310:     Value *Op1 = EmitScalarExpr(E->getArg(1));
4311:     Value *Result = Builder.CreateBinaryIntrinsic(llvm::Intrinsic::minnum, Op0,
4312:                                                   Op1, nullptr, "elt.minnum");
4313:     return RValue::get(Result);
4314:   }
4315: 
4316:   case Builtin::BI__builtin_elementwise_maximum: {
4317:     Value *Op0 = EmitScalarExpr(E->getArg(0));
4318:     Value *Op1 = EmitScalarExpr(E->getArg(1));
4319:     Value *Result = Builder.CreateBinaryIntrinsic(Intrinsic::maximum, Op0, Op1,
4320:                                                   nullptr, "elt.maximum");
```
- **EN**: This block defines callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 4321-4350
```cpp
4321:     return RValue::get(Result);
4322:   }
4323: 
4324:   case Builtin::BI__builtin_elementwise_minimum: {
4325:     Value *Op0 = EmitScalarExpr(E->getArg(0));
4326:     Value *Op1 = EmitScalarExpr(E->getArg(1));
4327:     Value *Result = Builder.CreateBinaryIntrinsic(Intrinsic::minimum, Op0, Op1,
4328:                                                   nullptr, "elt.minimum");
4329:     return RValue::get(Result);
4330:   }
4331: 
4332:   case Builtin::BI__builtin_elementwise_maximumnum: {
4333:     Value *Op0 = EmitScalarExpr(E->getArg(0));
4334:     Value *Op1 = EmitScalarExpr(E->getArg(1));
4335:     Value *Result = Builder.CreateBinaryIntrinsic(
4336:         Intrinsic::maximumnum, Op0, Op1, nullptr, "elt.maximumnum");
4337:     return RValue::get(Result);
4338:   }
4339: 
4340:   case Builtin::BI__builtin_elementwise_minimumnum: {
4341:     Value *Op0 = EmitScalarExpr(E->getArg(0));
4342:     Value *Op1 = EmitScalarExpr(E->getArg(1));
4343:     Value *Result = Builder.CreateBinaryIntrinsic(
4344:         Intrinsic::minimumnum, Op0, Op1, nullptr, "elt.minimumnum");
4345:     return RValue::get(Result);
4346:   }
4347: 
4348:   case Builtin::BI__builtin_reduce_max: {
4349:     auto GetIntrinsicID = [this](QualType QT) {
4350:       if (auto *VecTy = QT->getAs<VectorType>())
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4351-4380
```cpp
4351:         QT = VecTy->getElementType();
4352:       else if (QT->isSizelessVectorType())
4353:         QT = QT->getSizelessVectorEltType(CGM.getContext());
4354: 
4355:       if (QT->isSignedIntegerType())
4356:         return Intrinsic::vector_reduce_smax;
4357:       if (QT->isUnsignedIntegerType())
4358:         return Intrinsic::vector_reduce_umax;
4359:       assert(QT->isFloatingType() && "must have a float here");
4360:       return Intrinsic::vector_reduce_fmax;
4361:     };
4362:     return RValue::get(emitBuiltinWithOneOverloadedType<1>(
4363:         *this, E, GetIntrinsicID(E->getArg(0)->getType()), "rdx.min"));
4364:   }
4365: 
4366:   case Builtin::BI__builtin_reduce_min: {
4367:     auto GetIntrinsicID = [this](QualType QT) {
4368:       if (auto *VecTy = QT->getAs<VectorType>())
4369:         QT = VecTy->getElementType();
4370:       else if (QT->isSizelessVectorType())
4371:         QT = QT->getSizelessVectorEltType(CGM.getContext());
4372: 
4373:       if (QT->isSignedIntegerType())
4374:         return Intrinsic::vector_reduce_smin;
4375:       if (QT->isUnsignedIntegerType())
4376:         return Intrinsic::vector_reduce_umin;
4377:       assert(QT->isFloatingType() && "must have a float here");
4378:       return Intrinsic::vector_reduce_fmin;
4379:     };
4380: 
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 4381-4410
```cpp
4381:     return RValue::get(emitBuiltinWithOneOverloadedType<1>(
4382:         *this, E, GetIntrinsicID(E->getArg(0)->getType()), "rdx.min"));
4383:   }
4384: 
4385:   case Builtin::BI__builtin_reduce_add:
4386:     return RValue::get(emitBuiltinWithOneOverloadedType<1>(
4387:         *this, E, Intrinsic::vector_reduce_add, "rdx.add"));
4388:   case Builtin::BI__builtin_reduce_mul:
4389:     return RValue::get(emitBuiltinWithOneOverloadedType<1>(
4390:         *this, E, Intrinsic::vector_reduce_mul, "rdx.mul"));
4391:   case Builtin::BI__builtin_reduce_xor:
4392:     return RValue::get(emitBuiltinWithOneOverloadedType<1>(
4393:         *this, E, Intrinsic::vector_reduce_xor, "rdx.xor"));
4394:   case Builtin::BI__builtin_reduce_or:
4395:     return RValue::get(emitBuiltinWithOneOverloadedType<1>(
4396:         *this, E, Intrinsic::vector_reduce_or, "rdx.or"));
4397:   case Builtin::BI__builtin_reduce_and:
4398:     return RValue::get(emitBuiltinWithOneOverloadedType<1>(
4399:         *this, E, Intrinsic::vector_reduce_and, "rdx.and"));
4400:   case Builtin::BI__builtin_reduce_maximum:
4401:     return RValue::get(emitBuiltinWithOneOverloadedType<1>(
4402:         *this, E, Intrinsic::vector_reduce_fmaximum, "rdx.maximum"));
4403:   case Builtin::BI__builtin_reduce_minimum:
4404:     return RValue::get(emitBuiltinWithOneOverloadedType<1>(
4405:         *this, E, Intrinsic::vector_reduce_fminimum, "rdx.minimum"));
4406:   case Builtin::BI__builtin_reduce_assoc_fadd:
4407:   case Builtin::BI__builtin_reduce_in_order_fadd: {
4408:     llvm::Value *Vector = EmitScalarExpr(E->getArg(0));
4409:     llvm::Type *ScalarTy = Vector->getType()->getScalarType();
4410:     llvm::Value *StartValue = nullptr;
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 4411-4440
```cpp
4411:     if (E->getNumArgs() == 2)
4412:       StartValue = Builder.CreateFPCast(EmitScalarExpr(E->getArg(1)), ScalarTy);
4413:     llvm::Value *Args[] = {/*start_value=*/StartValue
4414:                                ? StartValue
4415:                                : llvm::ConstantFP::get(ScalarTy, -0.0F),
4416:                            /*vector=*/Vector};
4417:     llvm::Function *F =
4418:         CGM.getIntrinsic(Intrinsic::vector_reduce_fadd, Vector->getType());
4419:     llvm::CallBase *Reduce = Builder.CreateCall(F, Args, "rdx.addf");
4420:     if (BuiltinIDIfNoAsmLabel == Builtin::BI__builtin_reduce_assoc_fadd) {
4421:       // `__builtin_reduce_assoc_fadd` is an associative reduction which
4422:       // requires the reassoc FMF flag.
4423:       llvm::FastMathFlags FMF;
4424:       FMF.setAllowReassoc();
4425:       cast<llvm::CallBase>(Reduce)->setFastMathFlags(FMF);
4426:     }
4427:     return RValue::get(Reduce);
4428:   }
4429: 
4430:   case Builtin::BI__builtin_matrix_transpose: {
4431:     auto *MatrixTy = E->getArg(0)->getType()->castAs<ConstantMatrixType>();
4432:     Value *MatValue = EmitScalarExpr(E->getArg(0));
4433:     MatrixBuilder MB(Builder);
4434:     Value *Result = MB.CreateMatrixTranspose(MatValue, MatrixTy->getNumRows(),
4435:                                              MatrixTy->getNumColumns());
4436:     return RValue::get(Result);
4437:   }
4438: 
4439:   case Builtin::BI__builtin_matrix_column_major_load: {
4440:     MatrixBuilder MB(Builder);
```
- **EN**: This block defines callable entry points like `get`, `MB`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `MB`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4441-4470
```cpp
4441:     // Emit everything that isn't dependent on the first parameter type
4442:     Value *Stride = EmitScalarExpr(E->getArg(3));
4443:     const auto *ResultTy = E->getType()->getAs<ConstantMatrixType>();
4444:     auto *PtrTy = E->getArg(0)->getType()->getAs<PointerType>();
4445:     assert(PtrTy && "arg0 must be of pointer type");
4446:     bool IsVolatile = PtrTy->getPointeeType().isVolatileQualified();
4447: 
4448:     Address Src = EmitPointerWithAlignment(E->getArg(0));
4449:     EmitNonNullArgCheck(RValue::get(Src.emitRawPointer(*this)),
4450:                         E->getArg(0)->getType(), E->getArg(0)->getExprLoc(), FD,
4451:                         0);
4452:     Value *Result = MB.CreateColumnMajorLoad(
4453:         Src.getElementType(), Src.emitRawPointer(*this),
4454:         Align(Src.getAlignment().getQuantity()), Stride, IsVolatile,
4455:         ResultTy->getNumRows(), ResultTy->getNumColumns(), "matrix");
4456:     return RValue::get(Result);
4457:   }
4458: 
4459:   case Builtin::BI__builtin_matrix_column_major_store: {
4460:     MatrixBuilder MB(Builder);
4461:     Value *Matrix = EmitScalarExpr(E->getArg(0));
4462:     Address Dst = EmitPointerWithAlignment(E->getArg(1));
4463:     Value *Stride = EmitScalarExpr(E->getArg(2));
4464: 
4465:     const auto *MatrixTy = E->getArg(0)->getType()->getAs<ConstantMatrixType>();
4466:     auto *PtrTy = E->getArg(1)->getType()->getAs<PointerType>();
4467:     assert(PtrTy && "arg1 must be of pointer type");
4468:     bool IsVolatile = PtrTy->getPointeeType().isVolatileQualified();
4469: 
4470:     EmitNonNullArgCheck(RValue::get(Dst.emitRawPointer(*this)),
```
- **EN**: This block defines callable entry points like `EmitNonNullArgCheck`, `Align`, `get`, `MB`; uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitNonNullArgCheck`, `Align`, `get`, `MB`；通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 4471-4500
```cpp
4471:                         E->getArg(1)->getType(), E->getArg(1)->getExprLoc(), FD,
4472:                         0);
4473:     Value *Result = MB.CreateColumnMajorStore(
4474:         Matrix, Dst.emitRawPointer(*this),
4475:         Align(Dst.getAlignment().getQuantity()), Stride, IsVolatile,
4476:         MatrixTy->getNumRows(), MatrixTy->getNumColumns());
4477:     addInstToNewSourceAtom(cast<Instruction>(Result), Matrix);
4478:     return RValue::get(Result);
4479:   }
4480: 
4481:   case Builtin::BI__builtin_masked_load:
4482:   case Builtin::BI__builtin_masked_expand_load: {
4483:     llvm::Value *Mask = EmitScalarExpr(E->getArg(0));
4484:     llvm::Value *Ptr = EmitScalarExpr(E->getArg(1));
4485: 
4486:     llvm::Type *RetTy = CGM.getTypes().ConvertType(E->getType());
4487:     llvm::Value *PassThru = llvm::PoisonValue::get(RetTy);
4488:     if (E->getNumArgs() > 2)
4489:       PassThru = EmitScalarExpr(E->getArg(2));
4490: 
4491:     CharUnits Align = CGM.getNaturalTypeAlignment(
4492:         E->getType()->getAs<VectorType>()->getElementType(), nullptr);
4493: 
4494:     llvm::Value *Result;
4495:     if (BuiltinID == Builtin::BI__builtin_masked_load) {
4496:       Result = Builder.CreateMaskedLoad(RetTy, Ptr, Align.getAsAlign(), Mask,
4497:                                         PassThru, "masked_load");
4498:     } else {
4499:       Function *F = CGM.getIntrinsic(Intrinsic::masked_expandload, {RetTy});
4500:       Result =
```
- **EN**: This block defines callable entry points like `Align`, `addInstToNewSourceAtom`, `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `Align`, `addInstToNewSourceAtom`, `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4501-4530
```cpp
4501:           Builder.CreateCall(F, {Ptr, Mask, PassThru}, "masked_expand_load");
4502:     }
4503:     return RValue::get(Result);
4504:   };
4505:   case Builtin::BI__builtin_masked_gather: {
4506:     llvm::Value *Mask = EmitScalarExpr(E->getArg(0));
4507:     llvm::Value *Idx = EmitScalarExpr(E->getArg(1));
4508:     llvm::Value *Ptr = EmitScalarExpr(E->getArg(2));
4509: 
4510:     llvm::Type *RetTy = CGM.getTypes().ConvertType(E->getType());
4511:     CharUnits Align = CGM.getNaturalTypeAlignment(
4512:         E->getType()->getAs<VectorType>()->getElementType(), nullptr);
4513: 
4514:     llvm::Value *PassThru = llvm::PoisonValue::get(RetTy);
4515:     if (E->getNumArgs() > 3)
4516:       PassThru = EmitScalarExpr(E->getArg(3));
4517: 
4518:     llvm::Type *ElemTy = CGM.getTypes().ConvertType(
4519:         E->getType()->getAs<VectorType>()->getElementType());
4520:     llvm::Value *PtrVec = Builder.CreateGEP(ElemTy, Ptr, Idx);
4521: 
4522:     llvm::Value *Result = Builder.CreateMaskedGather(
4523:         RetTy, PtrVec, Align.getAsAlign(), Mask, PassThru, "masked_gather");
4524:     return RValue::get(Result);
4525:   }
4526:   case Builtin::BI__builtin_masked_store:
4527:   case Builtin::BI__builtin_masked_compress_store: {
4528:     llvm::Value *Mask = EmitScalarExpr(E->getArg(0));
4529:     llvm::Value *Val = EmitScalarExpr(E->getArg(1));
4530:     llvm::Value *Ptr = EmitScalarExpr(E->getArg(2));
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4531-4560
```cpp
4531: 
4532:     QualType ValTy = E->getArg(1)->getType();
4533:     llvm::Type *ValLLTy = CGM.getTypes().ConvertType(ValTy);
4534: 
4535:     CharUnits Align = CGM.getNaturalTypeAlignment(
4536:         E->getArg(1)->getType()->getAs<VectorType>()->getElementType(),
4537:         nullptr);
4538: 
4539:     if (BuiltinID == Builtin::BI__builtin_masked_store) {
4540:       Builder.CreateMaskedStore(Val, Ptr, Align.getAsAlign(), Mask);
4541:     } else {
4542:       llvm::Function *F =
4543:           CGM.getIntrinsic(llvm::Intrinsic::masked_compressstore, {ValLLTy});
4544:       Builder.CreateCall(F, {Val, Ptr, Mask});
4545:     }
4546:     return RValue::get(nullptr);
4547:   }
4548:   case Builtin::BI__builtin_masked_scatter: {
4549:     llvm::Value *Mask = EmitScalarExpr(E->getArg(0));
4550:     llvm::Value *Idx = EmitScalarExpr(E->getArg(1));
4551:     llvm::Value *Val = EmitScalarExpr(E->getArg(2));
4552:     llvm::Value *Ptr = EmitScalarExpr(E->getArg(3));
4553: 
4554:     CharUnits Align = CGM.getNaturalTypeAlignment(
4555:         E->getArg(2)->getType()->getAs<VectorType>()->getElementType(),
4556:         nullptr);
4557: 
4558:     llvm::Type *ElemTy = CGM.getTypes().ConvertType(
4559:         E->getArg(1)->getType()->getAs<VectorType>()->getElementType());
4560:     llvm::Value *PtrVec = Builder.CreateGEP(ElemTy, Ptr, Idx);
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4561-4590
```cpp
4561: 
4562:     Builder.CreateMaskedScatter(Val, PtrVec, Align.getAsAlign(), Mask);
4563:     return RValue();
4564:   }
4565:   case Builtin::BI__builtin_isinf_sign: {
4566:     // isinf_sign(x) -> fabs(x) == infinity ? (signbit(x) ? -1 : 1) : 0
4567:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
4568:     // FIXME: for strictfp/IEEE-754 we need to not trap on SNaN here.
4569:     Value *Arg = EmitScalarExpr(E->getArg(0));
4570:     Value *AbsArg = EmitFAbs(*this, Arg);
4571:     Value *IsInf = Builder.CreateFCmpOEQ(
4572:         AbsArg, ConstantFP::getInfinity(Arg->getType()), "isinf");
4573:     Value *IsNeg = EmitSignBit(*this, Arg);
4574: 
4575:     llvm::Type *IntTy = ConvertType(E->getType());
4576:     Value *Zero = Constant::getNullValue(IntTy);
4577:     Value *One = ConstantInt::get(IntTy, 1);
4578:     Value *NegativeOne = ConstantInt::getAllOnesValue(IntTy);
4579:     Value *SignResult = Builder.CreateSelect(IsNeg, NegativeOne, One);
4580:     Value *Result = Builder.CreateSelect(IsInf, SignResult, Zero);
4581:     return RValue::get(Result);
4582:   }
4583: 
4584:   case Builtin::BI__builtin_flt_rounds: {
4585:     Function *F = CGM.getIntrinsic(Intrinsic::get_rounding);
4586: 
4587:     llvm::Type *ResultType = ConvertType(E->getType());
4588:     Value *Result = Builder.CreateCall(F);
4589:     if (Result->getType() != ResultType)
4590:       Result = Builder.CreateIntCast(Result, ResultType, /*isSigned*/true,
```
- **EN**: This block defines callable entry points like `RValue`, `FPOptsRAII`, `getInfinity`, `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `RValue`, `FPOptsRAII`, `getInfinity`, `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4591-4620
```cpp
4591:                                      "cast");
4592:     return RValue::get(Result);
4593:   }
4594: 
4595:   case Builtin::BI__builtin_set_flt_rounds: {
4596:     Function *F = CGM.getIntrinsic(Intrinsic::set_rounding);
4597: 
4598:     Value *V = EmitScalarExpr(E->getArg(0));
4599:     Builder.CreateCall(F, V);
4600:     return RValue::get(nullptr);
4601:   }
4602: 
4603:   case Builtin::BI__builtin_fpclassify: {
4604:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
4605:     // FIXME: for strictfp/IEEE-754 we need to not trap on SNaN here.
4606:     Value *V = EmitScalarExpr(E->getArg(5));
4607:     llvm::Type *Ty = ConvertType(E->getArg(5)->getType());
4608: 
4609:     // Create Result
4610:     BasicBlock *Begin = Builder.GetInsertBlock();
4611:     BasicBlock *End = createBasicBlock("fpclassify_end", this->CurFn);
4612:     Builder.SetInsertPoint(End);
4613:     PHINode *Result =
4614:       Builder.CreatePHI(ConvertType(E->getArg(0)->getType()), 4,
4615:                         "fpclassify_result");
4616: 
4617:     // if (V==0) return FP_ZERO
4618:     Builder.SetInsertPoint(Begin);
4619:     Value *IsZero = Builder.CreateFCmpOEQ(V, Constant::getNullValue(Ty),
4620:                                           "iszero");
```
- **EN**: This block defines callable entry points like `get`, `FPOptsRAII`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `FPOptsRAII`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 4621-4650
```cpp
4621:     Value *ZeroLiteral = EmitScalarExpr(E->getArg(4));
4622:     BasicBlock *NotZero = createBasicBlock("fpclassify_not_zero", this->CurFn);
4623:     Builder.CreateCondBr(IsZero, End, NotZero);
4624:     Result->addIncoming(ZeroLiteral, Begin);
4625: 
4626:     // if (V != V) return FP_NAN
4627:     Builder.SetInsertPoint(NotZero);
4628:     Value *IsNan = Builder.CreateFCmpUNO(V, V, "cmp");
4629:     Value *NanLiteral = EmitScalarExpr(E->getArg(0));
4630:     BasicBlock *NotNan = createBasicBlock("fpclassify_not_nan", this->CurFn);
4631:     Builder.CreateCondBr(IsNan, End, NotNan);
4632:     Result->addIncoming(NanLiteral, NotZero);
4633: 
4634:     // if (fabs(V) == infinity) return FP_INFINITY
4635:     Builder.SetInsertPoint(NotNan);
4636:     Value *VAbs = EmitFAbs(*this, V);
4637:     Value *IsInf =
4638:       Builder.CreateFCmpOEQ(VAbs, ConstantFP::getInfinity(V->getType()),
4639:                             "isinf");
4640:     Value *InfLiteral = EmitScalarExpr(E->getArg(1));
4641:     BasicBlock *NotInf = createBasicBlock("fpclassify_not_inf", this->CurFn);
4642:     Builder.CreateCondBr(IsInf, End, NotInf);
4643:     Result->addIncoming(InfLiteral, NotNan);
4644: 
4645:     // if (fabs(V) >= MIN_NORMAL) return FP_NORMAL else FP_SUBNORMAL
4646:     Builder.SetInsertPoint(NotInf);
4647:     APFloat Smallest = APFloat::getSmallestNormalized(
4648:         getContext().getFloatTypeSemantics(E->getArg(5)->getType()));
4649:     Value *IsNormal =
4650:       Builder.CreateFCmpUGE(VAbs, ConstantFP::get(V->getContext(), Smallest),
```
- **EN**: This block spells out callable entry points like `getContext`.
- **CN**: 该代码块给出可调用入口的声明，例如 `getContext`。

### Lines 4651-4680
```cpp
4651:                             "isnormal");
4652:     Value *NormalResult =
4653:       Builder.CreateSelect(IsNormal, EmitScalarExpr(E->getArg(2)),
4654:                            EmitScalarExpr(E->getArg(3)));
4655:     Builder.CreateBr(End);
4656:     Result->addIncoming(NormalResult, NotInf);
4657: 
4658:     // return Result
4659:     Builder.SetInsertPoint(End);
4660:     return RValue::get(Result);
4661:   }
4662: 
4663:   // An alloca will always return a pointer to the alloca (stack) address
4664:   // space. This address space need not be the same as the AST / Language
4665:   // default (e.g. in C / C++ auto vars are in the generic address space). At
4666:   // the AST level this is handled within CreateTempAlloca et al., but for the
4667:   // builtin / dynamic alloca we have to handle it here. We use an explicit cast
4668:   // instead of passing an AS to CreateAlloca so as to not inhibit optimisation.
4669:   case Builtin::BIalloca:
4670:   case Builtin::BI_alloca:
4671:   case Builtin::BI__builtin_alloca_uninitialized:
4672:   case Builtin::BI__builtin_alloca: {
4673:     Value *Size = EmitScalarExpr(E->getArg(0));
4674:     const TargetInfo &TI = getContext().getTargetInfo();
4675:     // The alignment of the alloca should correspond to __BIGGEST_ALIGNMENT__.
4676:     const Align SuitableAlignmentInBytes =
4677:         CGM.getContext()
4678:             .toCharUnitsFromBits(TI.getSuitableAlign())
4679:             .getAsAlign();
4680:     AllocaInst *AI = Builder.CreateAlloca(Builder.getInt8Ty(), Size);
```
- **EN**: This block defines callable entry points like `EmitScalarExpr`, `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitScalarExpr`, `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 4681-4710
```cpp
4681:     AI->setAlignment(SuitableAlignmentInBytes);
4682:     if (BuiltinID != Builtin::BI__builtin_alloca_uninitialized)
4683:       initializeAlloca(*this, AI, Size, SuitableAlignmentInBytes);
4684:     if (AI->getAddressSpace() !=
4685:         CGM.getContext().getTargetAddressSpace(
4686:             E->getType()->getPointeeType().getAddressSpace())) {
4687:       llvm::Type *Ty = CGM.getTypes().ConvertType(E->getType());
4688:       return RValue::get(performAddrSpaceCast(AI, Ty));
4689:     }
4690:     return RValue::get(AI);
4691:   }
4692: 
4693:   case Builtin::BI__builtin_alloca_with_align_uninitialized:
4694:   case Builtin::BI__builtin_alloca_with_align: {
4695:     Value *Size = EmitScalarExpr(E->getArg(0));
4696:     Value *AlignmentInBitsValue = EmitScalarExpr(E->getArg(1));
4697:     auto *AlignmentInBitsCI = cast<ConstantInt>(AlignmentInBitsValue);
4698:     unsigned AlignmentInBits = AlignmentInBitsCI->getZExtValue();
4699:     const Align AlignmentInBytes =
4700:         CGM.getContext().toCharUnitsFromBits(AlignmentInBits).getAsAlign();
4701:     AllocaInst *AI = Builder.CreateAlloca(Builder.getInt8Ty(), Size);
4702:     AI->setAlignment(AlignmentInBytes);
4703:     if (BuiltinID != Builtin::BI__builtin_alloca_with_align_uninitialized)
4704:       initializeAlloca(*this, AI, Size, AlignmentInBytes);
4705:     if (AI->getAddressSpace() !=
4706:         CGM.getContext().getTargetAddressSpace(
4707:             E->getType()->getPointeeType().getAddressSpace())) {
4708:       llvm::Type *Ty = CGM.getTypes().ConvertType(E->getType());
4709:       return RValue::get(performAddrSpaceCast(AI, Ty));
4710:     }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4711-4740
```cpp
4711:     return RValue::get(AI);
4712:   }
4713: 
4714:   case Builtin::BI__builtin_infer_alloc_token: {
4715:     llvm::MDNode *MDN = buildAllocToken(E);
4716:     llvm::Value *MDV = MetadataAsValue::get(getLLVMContext(), MDN);
4717:     llvm::Function *F =
4718:         CGM.getIntrinsic(llvm::Intrinsic::alloc_token_id, {IntPtrTy});
4719:     llvm::CallBase *TokenID = Builder.CreateCall(F, MDV);
4720:     return RValue::get(TokenID);
4721:   }
4722: 
4723:   case Builtin::BIbzero:
4724:   case Builtin::BI__builtin_bzero: {
4725:     Address Dest = EmitPointerWithAlignment(E->getArg(0));
4726:     Value *SizeVal = EmitScalarExpr(E->getArg(1));
4727:     EmitNonNullArgCheck(Dest, E->getArg(0)->getType(),
4728:                         E->getArg(0)->getExprLoc(), FD, 0);
4729:     auto *I = Builder.CreateMemSet(Dest, Builder.getInt8(0), SizeVal, false);
4730:     addInstToNewSourceAtom(I, nullptr);
4731:     return RValue::get(nullptr);
4732:   }
4733: 
4734:   case Builtin::BIbcopy:
4735:   case Builtin::BI__builtin_bcopy: {
4736:     Address Src = EmitPointerWithAlignment(E->getArg(0));
4737:     Address Dest = EmitPointerWithAlignment(E->getArg(1));
4738:     Value *SizeVal = EmitScalarExpr(E->getArg(2));
4739:     EmitNonNullArgCheck(RValue::get(Src.emitRawPointer(*this)),
4740:                         E->getArg(0)->getType(), E->getArg(0)->getExprLoc(), FD,
```
- **EN**: This block defines callable entry points like `get`, `EmitNonNullArgCheck`, `addInstToNewSourceAtom`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `EmitNonNullArgCheck`, `addInstToNewSourceAtom`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 4741-4770
```cpp
4741:                         0);
4742:     EmitNonNullArgCheck(RValue::get(Dest.emitRawPointer(*this)),
4743:                         E->getArg(1)->getType(), E->getArg(1)->getExprLoc(), FD,
4744:                         0);
4745:     auto *I = Builder.CreateMemMove(Dest, Src, SizeVal, false);
4746:     addInstToNewSourceAtom(I, nullptr);
4747:     return RValue::get(nullptr);
4748:   }
4749: 
4750:   case Builtin::BImemcpy:
4751:   case Builtin::BI__builtin_memcpy:
4752:   case Builtin::BImempcpy:
4753:   case Builtin::BI__builtin_mempcpy: {
4754:     Address Dest = EmitPointerWithAlignment(E->getArg(0));
4755:     Address Src = EmitPointerWithAlignment(E->getArg(1));
4756:     Value *SizeVal = EmitScalarExpr(E->getArg(2));
4757:     EmitArgCheck(TCK_Store, Dest, E->getArg(0), 0);
4758:     EmitArgCheck(TCK_Load, Src, E->getArg(1), 1);
4759:     auto *I = Builder.CreateMemCpy(Dest, Src, SizeVal, false);
4760:     addInstToNewSourceAtom(I, nullptr);
4761:     if (BuiltinID == Builtin::BImempcpy ||
4762:         BuiltinID == Builtin::BI__builtin_mempcpy)
4763:       return RValue::get(Builder.CreateInBoundsGEP(
4764:           Dest.getElementType(), Dest.emitRawPointer(*this), SizeVal));
4765:     else
4766:       return RValue::get(Dest, *this);
4767:   }
4768: 
4769:   case Builtin::BI__builtin_memcpy_inline: {
4770:     Address Dest = EmitPointerWithAlignment(E->getArg(0));
```
- **EN**: This block defines callable entry points like `EmitNonNullArgCheck`, `addInstToNewSourceAtom`, `get`, `EmitArgCheck`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitNonNullArgCheck`, `addInstToNewSourceAtom`, `get`, `EmitArgCheck`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4771-4800
```cpp
4771:     Address Src = EmitPointerWithAlignment(E->getArg(1));
4772:     uint64_t Size =
4773:         E->getArg(2)->EvaluateKnownConstInt(getContext()).getZExtValue();
4774:     EmitArgCheck(TCK_Store, Dest, E->getArg(0), 0);
4775:     EmitArgCheck(TCK_Load, Src, E->getArg(1), 1);
4776:     auto *I = Builder.CreateMemCpyInline(Dest, Src, Size);
4777:     addInstToNewSourceAtom(I, nullptr);
4778:     return RValue::get(nullptr);
4779:   }
4780: 
4781:   case Builtin::BI__builtin_char_memchr:
4782:     BuiltinID = Builtin::BI__builtin_memchr;
4783:     break;
4784: 
4785:   case Builtin::BI__builtin___memcpy_chk: {
4786:     // fold __builtin_memcpy_chk(x, y, cst1, cst2) to memcpy iff cst1<=cst2.
4787:     Expr::EvalResult SizeResult, DstSizeResult;
4788:     if (!E->getArg(2)->EvaluateAsInt(SizeResult, CGM.getContext()) ||
4789:         !E->getArg(3)->EvaluateAsInt(DstSizeResult, CGM.getContext()))
4790:       break;
4791:     llvm::APSInt Size = SizeResult.Val.getInt();
4792:     llvm::APSInt DstSize = DstSizeResult.Val.getInt();
4793:     if (Size.ugt(DstSize))
4794:       break;
4795:     Address Dest = EmitPointerWithAlignment(E->getArg(0));
4796:     Address Src = EmitPointerWithAlignment(E->getArg(1));
4797:     Value *SizeVal = llvm::ConstantInt::get(Builder.getContext(), Size);
4798:     auto *I = Builder.CreateMemCpy(Dest, Src, SizeVal, false);
4799:     addInstToNewSourceAtom(I, nullptr);
4800:     return RValue::get(Dest, *this);
```
- **EN**: This block defines callable entry points like `EmitArgCheck`, `addInstToNewSourceAtom`, `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitArgCheck`, `addInstToNewSourceAtom`, `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4801-4830
```cpp
4801:   }
4802: 
4803:   case Builtin::BI__builtin_objc_memmove_collectable: {
4804:     Address DestAddr = EmitPointerWithAlignment(E->getArg(0));
4805:     Address SrcAddr = EmitPointerWithAlignment(E->getArg(1));
4806:     Value *SizeVal = EmitScalarExpr(E->getArg(2));
4807:     CGM.getObjCRuntime().EmitGCMemmoveCollectable(*this,
4808:                                                   DestAddr, SrcAddr, SizeVal);
4809:     return RValue::get(DestAddr, *this);
4810:   }
4811: 
4812:   case Builtin::BI__builtin___memmove_chk: {
4813:     // fold __builtin_memmove_chk(x, y, cst1, cst2) to memmove iff cst1<=cst2.
4814:     Expr::EvalResult SizeResult, DstSizeResult;
4815:     if (!E->getArg(2)->EvaluateAsInt(SizeResult, CGM.getContext()) ||
4816:         !E->getArg(3)->EvaluateAsInt(DstSizeResult, CGM.getContext()))
4817:       break;
4818:     llvm::APSInt Size = SizeResult.Val.getInt();
4819:     llvm::APSInt DstSize = DstSizeResult.Val.getInt();
4820:     if (Size.ugt(DstSize))
4821:       break;
4822:     Address Dest = EmitPointerWithAlignment(E->getArg(0));
4823:     Address Src = EmitPointerWithAlignment(E->getArg(1));
4824:     Value *SizeVal = llvm::ConstantInt::get(Builder.getContext(), Size);
4825:     auto *I = Builder.CreateMemMove(Dest, Src, SizeVal, false);
4826:     addInstToNewSourceAtom(I, nullptr);
4827:     return RValue::get(Dest, *this);
4828:   }
4829: 
4830:   case Builtin::BI__builtin_trivially_relocate:
```
- **EN**: This block defines callable entry points like `get`, `addInstToNewSourceAtom`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `addInstToNewSourceAtom`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4831-4860
```cpp
4831:   case Builtin::BImemmove:
4832:   case Builtin::BI__builtin_memmove: {
4833:     Address Dest = EmitPointerWithAlignment(E->getArg(0));
4834:     Address Src = EmitPointerWithAlignment(E->getArg(1));
4835:     Value *SizeVal = EmitScalarExpr(E->getArg(2));
4836:     if (BuiltinIDIfNoAsmLabel == Builtin::BI__builtin_trivially_relocate)
4837:       SizeVal = Builder.CreateMul(
4838:           SizeVal,
4839:           ConstantInt::get(
4840:               SizeVal->getType(),
4841:               getContext()
4842:                   .getTypeSizeInChars(E->getArg(0)->getType()->getPointeeType())
4843:                   .getQuantity()));
4844:     EmitArgCheck(TCK_Store, Dest, E->getArg(0), 0);
4845:     EmitArgCheck(TCK_Load, Src, E->getArg(1), 1);
4846:     auto *I = Builder.CreateMemMove(Dest, Src, SizeVal, false);
4847:     addInstToNewSourceAtom(I, nullptr);
4848:     return RValue::get(Dest, *this);
4849:   }
4850:   case Builtin::BImemset:
4851:   case Builtin::BI__builtin_memset: {
4852:     Address Dest = EmitPointerWithAlignment(E->getArg(0));
4853:     Value *ByteVal = Builder.CreateTrunc(EmitScalarExpr(E->getArg(1)),
4854:                                          Builder.getInt8Ty());
4855:     Value *SizeVal = EmitScalarExpr(E->getArg(2));
4856:     EmitNonNullArgCheck(Dest, E->getArg(0)->getType(),
4857:                         E->getArg(0)->getExprLoc(), FD, 0);
4858:     auto *I = Builder.CreateMemSet(Dest, ByteVal, SizeVal, false);
4859:     addInstToNewSourceAtom(I, ByteVal);
4860:     return RValue::get(Dest, *this);
```
- **EN**: This block defines callable entry points like `EmitArgCheck`, `addInstToNewSourceAtom`, `get`, `EmitNonNullArgCheck`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitArgCheck`, `addInstToNewSourceAtom`, `get`, `EmitNonNullArgCheck`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4861-4890
```cpp
4861:   }
4862:   case Builtin::BI__builtin_memset_inline: {
4863:     Address Dest = EmitPointerWithAlignment(E->getArg(0));
4864:     Value *ByteVal =
4865:         Builder.CreateTrunc(EmitScalarExpr(E->getArg(1)), Builder.getInt8Ty());
4866:     uint64_t Size =
4867:         E->getArg(2)->EvaluateKnownConstInt(getContext()).getZExtValue();
4868:     EmitNonNullArgCheck(RValue::get(Dest.emitRawPointer(*this)),
4869:                         E->getArg(0)->getType(), E->getArg(0)->getExprLoc(), FD,
4870:                         0);
4871:     auto *I = Builder.CreateMemSetInline(Dest, ByteVal, Size);
4872:     addInstToNewSourceAtom(I, nullptr);
4873:     return RValue::get(nullptr);
4874:   }
4875:   case Builtin::BI__builtin___memset_chk: {
4876:     // fold __builtin_memset_chk(x, y, cst1, cst2) to memset iff cst1<=cst2.
4877:     Expr::EvalResult SizeResult, DstSizeResult;
4878:     if (!E->getArg(2)->EvaluateAsInt(SizeResult, CGM.getContext()) ||
4879:         !E->getArg(3)->EvaluateAsInt(DstSizeResult, CGM.getContext()))
4880:       break;
4881:     llvm::APSInt Size = SizeResult.Val.getInt();
4882:     llvm::APSInt DstSize = DstSizeResult.Val.getInt();
4883:     if (Size.ugt(DstSize))
4884:       break;
4885:     Address Dest = EmitPointerWithAlignment(E->getArg(0));
4886:     Value *ByteVal = Builder.CreateTrunc(EmitScalarExpr(E->getArg(1)),
4887:                                          Builder.getInt8Ty());
4888:     Value *SizeVal = llvm::ConstantInt::get(Builder.getContext(), Size);
4889:     auto *I = Builder.CreateMemSet(Dest, ByteVal, SizeVal, false);
4890:     addInstToNewSourceAtom(I, nullptr);
```
- **EN**: This block defines callable entry points like `EmitNonNullArgCheck`, `addInstToNewSourceAtom`, `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitNonNullArgCheck`, `addInstToNewSourceAtom`, `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4891-4920
```cpp
4891:     return RValue::get(Dest, *this);
4892:   }
4893:   case Builtin::BI__builtin_wmemchr: {
4894:     // The MSVC runtime library does not provide a definition of wmemchr, so we
4895:     // need an inline implementation.
4896:     if (!getTarget().getTriple().isOSMSVCRT())
4897:       break;
4898: 
4899:     llvm::Type *WCharTy = ConvertType(getContext().WCharTy);
4900:     Value *Str = EmitScalarExpr(E->getArg(0));
4901:     Value *Chr = EmitScalarExpr(E->getArg(1));
4902:     Value *Size = EmitScalarExpr(E->getArg(2));
4903: 
4904:     BasicBlock *Entry = Builder.GetInsertBlock();
4905:     BasicBlock *CmpEq = createBasicBlock("wmemchr.eq");
4906:     BasicBlock *Next = createBasicBlock("wmemchr.next");
4907:     BasicBlock *Exit = createBasicBlock("wmemchr.exit");
4908:     Value *SizeEq0 = Builder.CreateICmpEQ(Size, ConstantInt::get(SizeTy, 0));
4909:     Builder.CreateCondBr(SizeEq0, Exit, CmpEq);
4910: 
4911:     EmitBlock(CmpEq);
4912:     PHINode *StrPhi = Builder.CreatePHI(Str->getType(), 2);
4913:     StrPhi->addIncoming(Str, Entry);
4914:     PHINode *SizePhi = Builder.CreatePHI(SizeTy, 2);
4915:     SizePhi->addIncoming(Size, Entry);
4916:     CharUnits WCharAlign =
4917:         getContext().getTypeAlignInChars(getContext().WCharTy);
4918:     Value *StrCh = Builder.CreateAlignedLoad(WCharTy, StrPhi, WCharAlign);
4919:     Value *FoundChr = Builder.CreateConstInBoundsGEP1_32(WCharTy, StrPhi, 0);
4920:     Value *StrEqChr = Builder.CreateICmpEQ(StrCh, Chr);
```
- **EN**: This block defines callable entry points like `get`, `EmitBlock`, `getContext`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `EmitBlock`, `getContext`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4921-4950
```cpp
4921:     Builder.CreateCondBr(StrEqChr, Exit, Next);
4922: 
4923:     EmitBlock(Next);
4924:     Value *NextStr = Builder.CreateConstInBoundsGEP1_32(WCharTy, StrPhi, 1);
4925:     Value *NextSize = Builder.CreateSub(SizePhi, ConstantInt::get(SizeTy, 1));
4926:     Value *NextSizeEq0 =
4927:         Builder.CreateICmpEQ(NextSize, ConstantInt::get(SizeTy, 0));
4928:     Builder.CreateCondBr(NextSizeEq0, Exit, CmpEq);
4929:     StrPhi->addIncoming(NextStr, Next);
4930:     SizePhi->addIncoming(NextSize, Next);
4931: 
4932:     EmitBlock(Exit);
4933:     PHINode *Ret = Builder.CreatePHI(Str->getType(), 3);
4934:     Ret->addIncoming(llvm::Constant::getNullValue(Str->getType()), Entry);
4935:     Ret->addIncoming(llvm::Constant::getNullValue(Str->getType()), Next);
4936:     Ret->addIncoming(FoundChr, CmpEq);
4937:     return RValue::get(Ret);
4938:   }
4939:   case Builtin::BI__builtin_wmemcmp: {
4940:     // The MSVC runtime library does not provide a definition of wmemcmp, so we
4941:     // need an inline implementation.
4942:     if (!getTarget().getTriple().isOSMSVCRT())
4943:       break;
4944: 
4945:     llvm::Type *WCharTy = ConvertType(getContext().WCharTy);
4946: 
4947:     Value *Dst = EmitScalarExpr(E->getArg(0));
4948:     Value *Src = EmitScalarExpr(E->getArg(1));
4949:     Value *Size = EmitScalarExpr(E->getArg(2));
4950: 
```
- **EN**: This block defines callable entry points like `EmitBlock`, `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitBlock`, `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 4951-4980
```cpp
4951:     BasicBlock *Entry = Builder.GetInsertBlock();
4952:     BasicBlock *CmpGT = createBasicBlock("wmemcmp.gt");
4953:     BasicBlock *CmpLT = createBasicBlock("wmemcmp.lt");
4954:     BasicBlock *Next = createBasicBlock("wmemcmp.next");
4955:     BasicBlock *Exit = createBasicBlock("wmemcmp.exit");
4956:     Value *SizeEq0 = Builder.CreateICmpEQ(Size, ConstantInt::get(SizeTy, 0));
4957:     Builder.CreateCondBr(SizeEq0, Exit, CmpGT);
4958: 
4959:     EmitBlock(CmpGT);
4960:     PHINode *DstPhi = Builder.CreatePHI(Dst->getType(), 2);
4961:     DstPhi->addIncoming(Dst, Entry);
4962:     PHINode *SrcPhi = Builder.CreatePHI(Src->getType(), 2);
4963:     SrcPhi->addIncoming(Src, Entry);
4964:     PHINode *SizePhi = Builder.CreatePHI(SizeTy, 2);
4965:     SizePhi->addIncoming(Size, Entry);
4966:     CharUnits WCharAlign =
4967:         getContext().getTypeAlignInChars(getContext().WCharTy);
4968:     Value *DstCh = Builder.CreateAlignedLoad(WCharTy, DstPhi, WCharAlign);
4969:     Value *SrcCh = Builder.CreateAlignedLoad(WCharTy, SrcPhi, WCharAlign);
4970:     Value *DstGtSrc = Builder.CreateICmpUGT(DstCh, SrcCh);
4971:     Builder.CreateCondBr(DstGtSrc, Exit, CmpLT);
4972: 
4973:     EmitBlock(CmpLT);
4974:     Value *DstLtSrc = Builder.CreateICmpULT(DstCh, SrcCh);
4975:     Builder.CreateCondBr(DstLtSrc, Exit, Next);
4976: 
4977:     EmitBlock(Next);
4978:     Value *NextDst = Builder.CreateConstInBoundsGEP1_32(WCharTy, DstPhi, 1);
4979:     Value *NextSrc = Builder.CreateConstInBoundsGEP1_32(WCharTy, SrcPhi, 1);
4980:     Value *NextSize = Builder.CreateSub(SizePhi, ConstantInt::get(SizeTy, 1));
```
- **EN**: This block spells out callable entry points like `EmitBlock`, `getContext`.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitBlock`, `getContext`。

### Lines 4981-5010
```cpp
4981:     Value *NextSizeEq0 =
4982:         Builder.CreateICmpEQ(NextSize, ConstantInt::get(SizeTy, 0));
4983:     Builder.CreateCondBr(NextSizeEq0, Exit, CmpGT);
4984:     DstPhi->addIncoming(NextDst, Next);
4985:     SrcPhi->addIncoming(NextSrc, Next);
4986:     SizePhi->addIncoming(NextSize, Next);
4987: 
4988:     EmitBlock(Exit);
4989:     PHINode *Ret = Builder.CreatePHI(IntTy, 4);
4990:     Ret->addIncoming(ConstantInt::get(IntTy, 0), Entry);
4991:     Ret->addIncoming(ConstantInt::get(IntTy, 1), CmpGT);
4992:     Ret->addIncoming(ConstantInt::getAllOnesValue(IntTy), CmpLT);
4993:     Ret->addIncoming(ConstantInt::get(IntTy, 0), Next);
4994:     return RValue::get(Ret);
4995:   }
4996:   case Builtin::BI__builtin_dwarf_cfa: {
4997:     // The offset in bytes from the first argument to the CFA.
4998:     //
4999:     // Why on earth is this in the frontend?  Is there any reason at
5000:     // all that the backend can't reasonably determine this while
5001:     // lowering llvm.eh.dwarf.cfa()?
5002:     //
5003:     // TODO: If there's a satisfactory reason, add a target hook for
5004:     // this instead of hard-coding 0, which is correct for most targets.
5005:     int32_t Offset = 0;
5006: 
5007:     Function *F = CGM.getIntrinsic(Intrinsic::eh_dwarf_cfa);
5008:     return RValue::get(Builder.CreateCall(F,
5009:                                       llvm::ConstantInt::get(Int32Ty, Offset)));
5010:   }
```
- **EN**: This block defines callable entry points like `EmitBlock`, `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitBlock`, `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5011-5040
```cpp
5011:   case Builtin::BI__builtin_return_address: {
5012:     Value *Depth = ConstantEmitter(*this).emitAbstract(E->getArg(0),
5013:                                                    getContext().UnsignedIntTy);
5014:     Function *F =
5015:         CGM.getIntrinsic(Intrinsic::returnaddress, {CGM.ProgramPtrTy});
5016:     return RValue::get(Builder.CreateCall(F, Depth));
5017:   }
5018:   case Builtin::BI_ReturnAddress: {
5019:     Function *F =
5020:         CGM.getIntrinsic(Intrinsic::returnaddress, {CGM.ProgramPtrTy});
5021:     return RValue::get(Builder.CreateCall(F, Builder.getInt32(0)));
5022:   }
5023:   case Builtin::BI__builtin_frame_address: {
5024:     Value *Depth = ConstantEmitter(*this).emitAbstract(E->getArg(0),
5025:                                                    getContext().UnsignedIntTy);
5026:     Function *F = CGM.getIntrinsic(Intrinsic::frameaddress, AllocaInt8PtrTy);
5027:     return RValue::get(Builder.CreateCall(F, Depth));
5028:   }
5029:   case Builtin::BI__builtin_stack_address: {
5030:     return RValue::get(Builder.CreateCall(
5031:         CGM.getIntrinsic(Intrinsic::stackaddress, AllocaInt8PtrTy)));
5032:   }
5033:   case Builtin::BI__builtin_extract_return_addr: {
5034:     Value *Address = EmitScalarExpr(E->getArg(0));
5035:     Value *Result = getTargetHooks().decodeReturnAddress(*this, Address);
5036:     return RValue::get(Result);
5037:   }
5038:   case Builtin::BI__builtin_frob_return_addr: {
5039:     Value *Address = EmitScalarExpr(E->getArg(0));
5040:     Value *Result = getTargetHooks().encodeReturnAddress(*this, Address);
```
- **EN**: This block defines callable entry points like `getContext`, `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getContext`, `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5041-5070
```cpp
5041:     return RValue::get(Result);
5042:   }
5043:   case Builtin::BI__builtin_dwarf_sp_column: {
5044:     llvm::IntegerType *Ty
5045:       = cast<llvm::IntegerType>(ConvertType(E->getType()));
5046:     int Column = getTargetHooks().getDwarfEHStackPointer(CGM);
5047:     if (Column == -1) {
5048:       CGM.ErrorUnsupported(E, "__builtin_dwarf_sp_column");
5049:       return RValue::get(llvm::UndefValue::get(Ty));
5050:     }
5051:     return RValue::get(llvm::ConstantInt::get(Ty, Column, true));
5052:   }
5053:   case Builtin::BI__builtin_init_dwarf_reg_size_table: {
5054:     Value *Address = EmitScalarExpr(E->getArg(0));
5055:     if (getTargetHooks().initDwarfEHRegSizeTable(*this, Address))
5056:       CGM.ErrorUnsupported(E, "__builtin_init_dwarf_reg_size_table");
5057:     return RValue::get(llvm::UndefValue::get(ConvertType(E->getType())));
5058:   }
5059:   case Builtin::BI__builtin_eh_return: {
5060:     Value *Int = EmitScalarExpr(E->getArg(0));
5061:     Value *Ptr = EmitScalarExpr(E->getArg(1));
5062: 
5063:     llvm::IntegerType *IntTy = cast<llvm::IntegerType>(Int->getType());
5064:     assert((IntTy->getBitWidth() == 32 || IntTy->getBitWidth() == 64) &&
5065:            "LLVM's __builtin_eh_return only supports 32- and 64-bit variants");
5066:     Function *F =
5067:         CGM.getIntrinsic(IntTy->getBitWidth() == 32 ? Intrinsic::eh_return_i32
5068:                                                     : Intrinsic::eh_return_i64);
5069:     Builder.CreateCall(F, {Int, Ptr});
5070:     Builder.CreateUnreachable();
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 5071-5100
```cpp
5071: 
5072:     // We do need to preserve an insertion point.
5073:     EmitBlock(createBasicBlock("builtin_eh_return.cont"));
5074: 
5075:     return RValue::get(nullptr);
5076:   }
5077:   case Builtin::BI__builtin_unwind_init: {
5078:     Function *F = CGM.getIntrinsic(Intrinsic::eh_unwind_init);
5079:     Builder.CreateCall(F);
5080:     return RValue::get(nullptr);
5081:   }
5082:   case Builtin::BI__builtin_extend_pointer: {
5083:     // Extends a pointer to the size of an _Unwind_Word, which is
5084:     // uint64_t on all platforms.  Generally this gets poked into a
5085:     // register and eventually used as an address, so if the
5086:     // addressing registers are wider than pointers and the platform
5087:     // doesn't implicitly ignore high-order bits when doing
5088:     // addressing, we need to make sure we zext / sext based on
5089:     // the platform's expectations.
5090:     //
5091:     // See: http://gcc.gnu.org/ml/gcc-bugs/2002-02/msg00237.html
5092: 
5093:     // Cast the pointer to intptr_t.
5094:     Value *Ptr = EmitScalarExpr(E->getArg(0));
5095:     Value *Result = Builder.CreatePtrToInt(Ptr, IntPtrTy, "extend.cast");
5096: 
5097:     // If that's 64 bits, we're done.
5098:     if (IntPtrTy->getBitWidth() == 64)
5099:       return RValue::get(Result);
5100: 
```
- **EN**: This block defines callable entry points like `EmitBlock`, `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitBlock`, `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 5101-5130
```cpp
5101:     // Otherwise, ask the codegen data what to do.
5102:     if (getTargetHooks().extendPointerWithSExt())
5103:       return RValue::get(Builder.CreateSExt(Result, Int64Ty, "extend.sext"));
5104:     else
5105:       return RValue::get(Builder.CreateZExt(Result, Int64Ty, "extend.zext"));
5106:   }
5107:   case Builtin::BI__builtin_setjmp: {
5108:     // Buffer is a void**.
5109:     Address Buf = EmitPointerWithAlignment(E->getArg(0));
5110: 
5111:     if (getTarget().getTriple().getArch() == llvm::Triple::systemz) {
5112:       // On this target, the back end fills in the context buffer completely.
5113:       // It doesn't really matter if the frontend stores to the buffer before
5114:       // calling setjmp, the back-end is going to overwrite them anyway.
5115:       Function *F = CGM.getIntrinsic(Intrinsic::eh_sjlj_setjmp);
5116:       return RValue::get(Builder.CreateCall(F, Buf.emitRawPointer(*this)));
5117:     }
5118: 
5119:     // Store the frame pointer to the setjmp buffer.
5120:     Value *FrameAddr = Builder.CreateCall(
5121:         CGM.getIntrinsic(Intrinsic::frameaddress, AllocaInt8PtrTy),
5122:         ConstantInt::get(Int32Ty, 0));
5123:     Builder.CreateStore(FrameAddr, Buf);
5124: 
5125:     // Store the stack pointer to the setjmp buffer.
5126:     Value *StackAddr = Builder.CreateStackSave();
5127:     assert(Buf.emitRawPointer(*this)->getType() == StackAddr->getType());
5128: 
5129:     Address StackSaveSlot = Builder.CreateConstInBoundsGEP(Buf, 2);
5130:     Builder.CreateStore(StackAddr, StackSaveSlot);
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 5131-5160
```cpp
5131: 
5132:     // Call LLVM's EH setjmp, which is lightweight.
5133:     Function *F = CGM.getIntrinsic(Intrinsic::eh_sjlj_setjmp);
5134:     return RValue::get(Builder.CreateCall(F, Buf.emitRawPointer(*this)));
5135:   }
5136:   case Builtin::BI__builtin_longjmp: {
5137:     Value *Buf = EmitScalarExpr(E->getArg(0));
5138: 
5139:     // Call LLVM's EH longjmp, which is lightweight.
5140:     Builder.CreateCall(CGM.getIntrinsic(Intrinsic::eh_sjlj_longjmp), Buf);
5141: 
5142:     // longjmp doesn't return; mark this as unreachable.
5143:     Builder.CreateUnreachable();
5144: 
5145:     // We do need to preserve an insertion point.
5146:     EmitBlock(createBasicBlock("longjmp.cont"));
5147: 
5148:     return RValue::get(nullptr);
5149:   }
5150:   case Builtin::BI__builtin_launder: {
5151:     const Expr *Arg = E->getArg(0);
5152:     QualType ArgTy = Arg->getType()->getPointeeType();
5153:     Value *Ptr = EmitScalarExpr(Arg);
5154:     if (TypeRequiresBuiltinLaunder(CGM, ArgTy))
5155:       Ptr = Builder.CreateLaunderInvariantGroup(Ptr);
5156: 
5157:     return RValue::get(Ptr);
5158:   }
5159:   case Builtin::BI__sync_fetch_and_add:
5160:   case Builtin::BI__sync_fetch_and_sub:
```
- **EN**: This block defines callable entry points like `get`, `EmitBlock`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `EmitBlock`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 5161-5190
```cpp
5161:   case Builtin::BI__sync_fetch_and_or:
5162:   case Builtin::BI__sync_fetch_and_and:
5163:   case Builtin::BI__sync_fetch_and_xor:
5164:   case Builtin::BI__sync_fetch_and_nand:
5165:   case Builtin::BI__sync_add_and_fetch:
5166:   case Builtin::BI__sync_sub_and_fetch:
5167:   case Builtin::BI__sync_and_and_fetch:
5168:   case Builtin::BI__sync_or_and_fetch:
5169:   case Builtin::BI__sync_xor_and_fetch:
5170:   case Builtin::BI__sync_nand_and_fetch:
5171:   case Builtin::BI__sync_val_compare_and_swap:
5172:   case Builtin::BI__sync_bool_compare_and_swap:
5173:   case Builtin::BI__sync_lock_test_and_set:
5174:   case Builtin::BI__sync_lock_release:
5175:   case Builtin::BI__sync_swap:
5176:     llvm_unreachable("Shouldn't make it through sema");
5177:   case Builtin::BI__sync_fetch_and_add_1:
5178:   case Builtin::BI__sync_fetch_and_add_2:
5179:   case Builtin::BI__sync_fetch_and_add_4:
5180:   case Builtin::BI__sync_fetch_and_add_8:
5181:   case Builtin::BI__sync_fetch_and_add_16:
5182:     return EmitBinaryAtomic(*this, llvm::AtomicRMWInst::Add, E);
5183:   case Builtin::BI__sync_fetch_and_sub_1:
5184:   case Builtin::BI__sync_fetch_and_sub_2:
5185:   case Builtin::BI__sync_fetch_and_sub_4:
5186:   case Builtin::BI__sync_fetch_and_sub_8:
5187:   case Builtin::BI__sync_fetch_and_sub_16:
5188:     return EmitBinaryAtomic(*this, llvm::AtomicRMWInst::Sub, E);
5189:   case Builtin::BI__sync_fetch_and_or_1:
5190:   case Builtin::BI__sync_fetch_and_or_2:
```
- **EN**: This block spells out callable entry points like `EmitBinaryAtomic`; uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitBinaryAtomic`；通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 5191-5220
```cpp
5191:   case Builtin::BI__sync_fetch_and_or_4:
5192:   case Builtin::BI__sync_fetch_and_or_8:
5193:   case Builtin::BI__sync_fetch_and_or_16:
5194:     return EmitBinaryAtomic(*this, llvm::AtomicRMWInst::Or, E);
5195:   case Builtin::BI__sync_fetch_and_and_1:
5196:   case Builtin::BI__sync_fetch_and_and_2:
5197:   case Builtin::BI__sync_fetch_and_and_4:
5198:   case Builtin::BI__sync_fetch_and_and_8:
5199:   case Builtin::BI__sync_fetch_and_and_16:
5200:     return EmitBinaryAtomic(*this, llvm::AtomicRMWInst::And, E);
5201:   case Builtin::BI__sync_fetch_and_xor_1:
5202:   case Builtin::BI__sync_fetch_and_xor_2:
5203:   case Builtin::BI__sync_fetch_and_xor_4:
5204:   case Builtin::BI__sync_fetch_and_xor_8:
5205:   case Builtin::BI__sync_fetch_and_xor_16:
5206:     return EmitBinaryAtomic(*this, llvm::AtomicRMWInst::Xor, E);
5207:   case Builtin::BI__sync_fetch_and_nand_1:
5208:   case Builtin::BI__sync_fetch_and_nand_2:
5209:   case Builtin::BI__sync_fetch_and_nand_4:
5210:   case Builtin::BI__sync_fetch_and_nand_8:
5211:   case Builtin::BI__sync_fetch_and_nand_16:
5212:     return EmitBinaryAtomic(*this, llvm::AtomicRMWInst::Nand, E);
5213: 
5214:   // Clang extensions: not overloaded yet.
5215:   case Builtin::BI__sync_fetch_and_min:
5216:     return EmitBinaryAtomic(*this, llvm::AtomicRMWInst::Min, E);
5217:   case Builtin::BI__sync_fetch_and_max:
5218:     return EmitBinaryAtomic(*this, llvm::AtomicRMWInst::Max, E);
5219:   case Builtin::BI__sync_fetch_and_umin:
5220:     return EmitBinaryAtomic(*this, llvm::AtomicRMWInst::UMin, E);
```
- **EN**: This block spells out callable entry points like `EmitBinaryAtomic`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitBinaryAtomic`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5221-5250
```cpp
5221:   case Builtin::BI__sync_fetch_and_umax:
5222:     return EmitBinaryAtomic(*this, llvm::AtomicRMWInst::UMax, E);
5223: 
5224:   case Builtin::BI__sync_add_and_fetch_1:
5225:   case Builtin::BI__sync_add_and_fetch_2:
5226:   case Builtin::BI__sync_add_and_fetch_4:
5227:   case Builtin::BI__sync_add_and_fetch_8:
5228:   case Builtin::BI__sync_add_and_fetch_16:
5229:     return EmitBinaryAtomicPost(*this, llvm::AtomicRMWInst::Add, E,
5230:                                 llvm::Instruction::Add);
5231:   case Builtin::BI__sync_sub_and_fetch_1:
5232:   case Builtin::BI__sync_sub_and_fetch_2:
5233:   case Builtin::BI__sync_sub_and_fetch_4:
5234:   case Builtin::BI__sync_sub_and_fetch_8:
5235:   case Builtin::BI__sync_sub_and_fetch_16:
5236:     return EmitBinaryAtomicPost(*this, llvm::AtomicRMWInst::Sub, E,
5237:                                 llvm::Instruction::Sub);
5238:   case Builtin::BI__sync_and_and_fetch_1:
5239:   case Builtin::BI__sync_and_and_fetch_2:
5240:   case Builtin::BI__sync_and_and_fetch_4:
5241:   case Builtin::BI__sync_and_and_fetch_8:
5242:   case Builtin::BI__sync_and_and_fetch_16:
5243:     return EmitBinaryAtomicPost(*this, llvm::AtomicRMWInst::And, E,
5244:                                 llvm::Instruction::And);
5245:   case Builtin::BI__sync_or_and_fetch_1:
5246:   case Builtin::BI__sync_or_and_fetch_2:
5247:   case Builtin::BI__sync_or_and_fetch_4:
5248:   case Builtin::BI__sync_or_and_fetch_8:
5249:   case Builtin::BI__sync_or_and_fetch_16:
5250:     return EmitBinaryAtomicPost(*this, llvm::AtomicRMWInst::Or, E,
```
- **EN**: This block spells out callable entry points like `EmitBinaryAtomic`, `EmitBinaryAtomicPost`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitBinaryAtomic`, `EmitBinaryAtomicPost`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5251-5280
```cpp
5251:                                 llvm::Instruction::Or);
5252:   case Builtin::BI__sync_xor_and_fetch_1:
5253:   case Builtin::BI__sync_xor_and_fetch_2:
5254:   case Builtin::BI__sync_xor_and_fetch_4:
5255:   case Builtin::BI__sync_xor_and_fetch_8:
5256:   case Builtin::BI__sync_xor_and_fetch_16:
5257:     return EmitBinaryAtomicPost(*this, llvm::AtomicRMWInst::Xor, E,
5258:                                 llvm::Instruction::Xor);
5259:   case Builtin::BI__sync_nand_and_fetch_1:
5260:   case Builtin::BI__sync_nand_and_fetch_2:
5261:   case Builtin::BI__sync_nand_and_fetch_4:
5262:   case Builtin::BI__sync_nand_and_fetch_8:
5263:   case Builtin::BI__sync_nand_and_fetch_16:
5264:     return EmitBinaryAtomicPost(*this, llvm::AtomicRMWInst::Nand, E,
5265:                                 llvm::Instruction::And, true);
5266: 
5267:   case Builtin::BI__sync_val_compare_and_swap_1:
5268:   case Builtin::BI__sync_val_compare_and_swap_2:
5269:   case Builtin::BI__sync_val_compare_and_swap_4:
5270:   case Builtin::BI__sync_val_compare_and_swap_8:
5271:   case Builtin::BI__sync_val_compare_and_swap_16:
5272:     return RValue::get(MakeAtomicCmpXchgValue(
5273:         *this, E, false, AtomicOrdering::SequentiallyConsistent,
5274:         AtomicOrdering::SequentiallyConsistent));
5275: 
5276:   case Builtin::BI__sync_bool_compare_and_swap_1:
5277:   case Builtin::BI__sync_bool_compare_and_swap_2:
5278:   case Builtin::BI__sync_bool_compare_and_swap_4:
5279:   case Builtin::BI__sync_bool_compare_and_swap_8:
5280:   case Builtin::BI__sync_bool_compare_and_swap_16:
```
- **EN**: This block spells out callable entry points like `EmitBinaryAtomicPost`, `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitBinaryAtomicPost`, `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5281-5310
```cpp
5281:     return RValue::get(MakeAtomicCmpXchgValue(
5282:         *this, E, true, AtomicOrdering::SequentiallyConsistent,
5283:         AtomicOrdering::SequentiallyConsistent));
5284: 
5285:   case Builtin::BI__sync_swap_1:
5286:   case Builtin::BI__sync_swap_2:
5287:   case Builtin::BI__sync_swap_4:
5288:   case Builtin::BI__sync_swap_8:
5289:   case Builtin::BI__sync_swap_16:
5290:     return EmitBinaryAtomic(*this, llvm::AtomicRMWInst::Xchg, E);
5291: 
5292:   case Builtin::BI__sync_lock_test_and_set_1:
5293:   case Builtin::BI__sync_lock_test_and_set_2:
5294:   case Builtin::BI__sync_lock_test_and_set_4:
5295:   case Builtin::BI__sync_lock_test_and_set_8:
5296:   case Builtin::BI__sync_lock_test_and_set_16:
5297:     return EmitBinaryAtomic(*this, llvm::AtomicRMWInst::Xchg, E);
5298: 
5299:   case Builtin::BI__sync_lock_release_1:
5300:   case Builtin::BI__sync_lock_release_2:
5301:   case Builtin::BI__sync_lock_release_4:
5302:   case Builtin::BI__sync_lock_release_8:
5303:   case Builtin::BI__sync_lock_release_16: {
5304:     Address Ptr = CheckAtomicAlignment(*this, E);
5305:     QualType ElTy = E->getArg(0)->getType()->getPointeeType();
5306: 
5307:     llvm::Type *ITy = llvm::IntegerType::get(getLLVMContext(),
5308:                                              getContext().getTypeSize(ElTy));
5309:     llvm::StoreInst *Store =
5310:         Builder.CreateStore(llvm::Constant::getNullValue(ITy), Ptr);
```
- **EN**: This block defines callable entry points like `get`, `EmitBinaryAtomic`, `getContext`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `EmitBinaryAtomic`, `getContext`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5311-5340
```cpp
5311:     Store->setAtomic(llvm::AtomicOrdering::Release);
5312:     return RValue::get(nullptr);
5313:   }
5314: 
5315:   case Builtin::BI__sync_synchronize: {
5316:     // We assume this is supposed to correspond to a C++0x-style
5317:     // sequentially-consistent fence (i.e. this is only usable for
5318:     // synchronization, not device I/O or anything like that). This intrinsic
5319:     // is really badly designed in the sense that in theory, there isn't
5320:     // any way to safely use it... but in practice, it mostly works
5321:     // to use it with non-atomic loads and stores to get acquire/release
5322:     // semantics.
5323:     Builder.CreateFence(llvm::AtomicOrdering::SequentiallyConsistent);
5324:     return RValue::get(nullptr);
5325:   }
5326: 
5327:   case Builtin::BI__builtin_nontemporal_load:
5328:     return RValue::get(EmitNontemporalLoad(*this, E));
5329:   case Builtin::BI__builtin_nontemporal_store:
5330:     return RValue::get(EmitNontemporalStore(*this, E));
5331:   case Builtin::BI__c11_atomic_is_lock_free:
5332:   case Builtin::BI__atomic_is_lock_free: {
5333:     // Call "bool __atomic_is_lock_free(size_t size, void *ptr)". For the
5334:     // __c11 builtin, ptr is 0 (indicating a properly-aligned object), since
5335:     // _Atomic(T) is always properly-aligned.
5336:     const char *LibCallName = "__atomic_is_lock_free";
5337:     CallArgList Args;
5338:     Args.add(RValue::get(EmitScalarExpr(E->getArg(0))),
5339:              getContext().getSizeType());
5340:     if (BuiltinID == Builtin::BI__atomic_is_lock_free)
```
- **EN**: This block defines callable entry points like `get`, `getContext`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getContext`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 5341-5370
```cpp
5341:       Args.add(RValue::get(EmitScalarExpr(E->getArg(1))),
5342:                getContext().VoidPtrTy);
5343:     else
5344:       Args.add(RValue::get(llvm::Constant::getNullValue(VoidPtrTy)),
5345:                getContext().VoidPtrTy);
5346:     const CGFunctionInfo &FuncInfo =
5347:         CGM.getTypes().arrangeBuiltinFunctionCall(E->getType(), Args);
5348:     llvm::FunctionType *FTy = CGM.getTypes().GetFunctionType(FuncInfo);
5349:     llvm::FunctionCallee Func = CGM.CreateRuntimeFunction(FTy, LibCallName);
5350:     return EmitCall(FuncInfo, CGCallee::forDirect(Func),
5351:                     ReturnValueSlot(), Args);
5352:   }
5353: 
5354:   case Builtin::BI__atomic_thread_fence:
5355:   case Builtin::BI__atomic_signal_fence:
5356:   case Builtin::BI__c11_atomic_thread_fence:
5357:   case Builtin::BI__c11_atomic_signal_fence: {
5358:     llvm::SyncScope::ID SSID;
5359:     if (BuiltinID == Builtin::BI__atomic_signal_fence ||
5360:         BuiltinID == Builtin::BI__c11_atomic_signal_fence)
5361:       SSID = llvm::SyncScope::SingleThread;
5362:     else
5363:       SSID = llvm::SyncScope::System;
5364:     Value *Order = EmitScalarExpr(E->getArg(0));
5365:     if (isa<llvm::ConstantInt>(Order)) {
5366:       int ord = cast<llvm::ConstantInt>(Order)->getZExtValue();
5367:       switch (ord) {
5368:       case 0:  // memory_order_relaxed
5369:       default: // invalid order
5370:         break;
```
- **EN**: This block defines callable entry points like `getContext`, `EmitCall`; uses control flow (if, switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getContext`, `EmitCall`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为。

### Lines 5371-5400
```cpp
5371:       case 1:  // memory_order_consume
5372:       case 2:  // memory_order_acquire
5373:         Builder.CreateFence(llvm::AtomicOrdering::Acquire, SSID);
5374:         break;
5375:       case 3:  // memory_order_release
5376:         Builder.CreateFence(llvm::AtomicOrdering::Release, SSID);
5377:         break;
5378:       case 4:  // memory_order_acq_rel
5379:         Builder.CreateFence(llvm::AtomicOrdering::AcquireRelease, SSID);
5380:         break;
5381:       case 5:  // memory_order_seq_cst
5382:         Builder.CreateFence(llvm::AtomicOrdering::SequentiallyConsistent, SSID);
5383:         break;
5384:       }
5385:       return RValue::get(nullptr);
5386:     }
5387: 
5388:     llvm::BasicBlock *AcquireBB, *ReleaseBB, *AcqRelBB, *SeqCstBB;
5389:     AcquireBB = createBasicBlock("acquire", CurFn);
5390:     ReleaseBB = createBasicBlock("release", CurFn);
5391:     AcqRelBB = createBasicBlock("acqrel", CurFn);
5392:     SeqCstBB = createBasicBlock("seqcst", CurFn);
5393:     llvm::BasicBlock *ContBB = createBasicBlock("atomic.continue", CurFn);
5394: 
5395:     Order = Builder.CreateIntCast(Order, Builder.getInt32Ty(), false);
5396:     llvm::SwitchInst *SI = Builder.CreateSwitch(Order, ContBB);
5397: 
5398:     Builder.SetInsertPoint(AcquireBB);
5399:     Builder.CreateFence(llvm::AtomicOrdering::Acquire, SSID);
5400:     Builder.CreateBr(ContBB);
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5401-5430
```cpp
5401:     SI->addCase(Builder.getInt32(1), AcquireBB);
5402:     SI->addCase(Builder.getInt32(2), AcquireBB);
5403: 
5404:     Builder.SetInsertPoint(ReleaseBB);
5405:     Builder.CreateFence(llvm::AtomicOrdering::Release, SSID);
5406:     Builder.CreateBr(ContBB);
5407:     SI->addCase(Builder.getInt32(3), ReleaseBB);
5408: 
5409:     Builder.SetInsertPoint(AcqRelBB);
5410:     Builder.CreateFence(llvm::AtomicOrdering::AcquireRelease, SSID);
5411:     Builder.CreateBr(ContBB);
5412:     SI->addCase(Builder.getInt32(4), AcqRelBB);
5413: 
5414:     Builder.SetInsertPoint(SeqCstBB);
5415:     Builder.CreateFence(llvm::AtomicOrdering::SequentiallyConsistent, SSID);
5416:     Builder.CreateBr(ContBB);
5417:     SI->addCase(Builder.getInt32(5), SeqCstBB);
5418: 
5419:     Builder.SetInsertPoint(ContBB);
5420:     return RValue::get(nullptr);
5421:   }
5422:   case Builtin::BI__scoped_atomic_thread_fence: {
5423:     auto ScopeModel = AtomicScopeModel::create(AtomicScopeModelKind::Generic);
5424: 
5425:     Value *Order = EmitScalarExpr(E->getArg(0));
5426:     Value *Scope = EmitScalarExpr(E->getArg(1));
5427:     auto Ord = dyn_cast<llvm::ConstantInt>(Order);
5428:     auto Scp = dyn_cast<llvm::ConstantInt>(Scope);
5429:     if (Ord && Scp) {
5430:       SyncScope SS = ScopeModel->isValid(Scp->getZExtValue())
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 5431-5460
```cpp
5431:                          ? ScopeModel->map(Scp->getZExtValue())
5432:                          : ScopeModel->map(ScopeModel->getFallBackValue());
5433:       switch (Ord->getZExtValue()) {
5434:       case 0:  // memory_order_relaxed
5435:       default: // invalid order
5436:         break;
5437:       case 1: // memory_order_consume
5438:       case 2: // memory_order_acquire
5439:         Builder.CreateFence(
5440:             llvm::AtomicOrdering::Acquire,
5441:             getTargetHooks().getLLVMSyncScopeID(getLangOpts(), SS,
5442:                                                 llvm::AtomicOrdering::Acquire,
5443:                                                 getLLVMContext()));
5444:         break;
5445:       case 3: // memory_order_release
5446:         Builder.CreateFence(
5447:             llvm::AtomicOrdering::Release,
5448:             getTargetHooks().getLLVMSyncScopeID(getLangOpts(), SS,
5449:                                                 llvm::AtomicOrdering::Release,
5450:                                                 getLLVMContext()));
5451:         break;
5452:       case 4: // memory_order_acq_rel
5453:         Builder.CreateFence(llvm::AtomicOrdering::AcquireRelease,
5454:                             getTargetHooks().getLLVMSyncScopeID(
5455:                                 getLangOpts(), SS,
5456:                                 llvm::AtomicOrdering::AcquireRelease,
5457:                                 getLLVMContext()));
5458:         break;
5459:       case 5: // memory_order_seq_cst
5460:         Builder.CreateFence(llvm::AtomicOrdering::SequentiallyConsistent,
```
- **EN**: This block defines callable entry points like `getTargetHooks`; uses control flow (switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getTargetHooks`；通过控制流（switch, case）细化 LLVM IR 生成 行为。

### Lines 5461-5490
```cpp
5461:                             getTargetHooks().getLLVMSyncScopeID(
5462:                                 getLangOpts(), SS,
5463:                                 llvm::AtomicOrdering::SequentiallyConsistent,
5464:                                 getLLVMContext()));
5465:         break;
5466:       }
5467:       return RValue::get(nullptr);
5468:     }
5469: 
5470:     llvm::BasicBlock *ContBB = createBasicBlock("atomic.scope.continue", CurFn);
5471: 
5472:     llvm::SmallVector<std::pair<llvm::BasicBlock *, llvm::AtomicOrdering>>
5473:         OrderBBs;
5474:     if (Ord) {
5475:       switch (Ord->getZExtValue()) {
5476:       case 0:  // memory_order_relaxed
5477:       default: // invalid order
5478:         ContBB->eraseFromParent();
5479:         return RValue::get(nullptr);
5480:       case 1: // memory_order_consume
5481:       case 2: // memory_order_acquire
5482:         OrderBBs.emplace_back(Builder.GetInsertBlock(),
5483:                               llvm::AtomicOrdering::Acquire);
5484:         break;
5485:       case 3: // memory_order_release
5486:         OrderBBs.emplace_back(Builder.GetInsertBlock(),
5487:                               llvm::AtomicOrdering::Release);
5488:         break;
5489:       case 4: // memory_order_acq_rel
5490:         OrderBBs.emplace_back(Builder.GetInsertBlock(),
```
- **EN**: This block defines callable entry points like `getTargetHooks`, `get`; uses control flow (if, switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getTargetHooks`, `get`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为。

### Lines 5491-5520
```cpp
5491:                               llvm::AtomicOrdering::AcquireRelease);
5492:         break;
5493:       case 5: // memory_order_seq_cst
5494:         OrderBBs.emplace_back(Builder.GetInsertBlock(),
5495:                               llvm::AtomicOrdering::SequentiallyConsistent);
5496:         break;
5497:       }
5498:     } else {
5499:       llvm::BasicBlock *AcquireBB = createBasicBlock("acquire", CurFn);
5500:       llvm::BasicBlock *ReleaseBB = createBasicBlock("release", CurFn);
5501:       llvm::BasicBlock *AcqRelBB = createBasicBlock("acqrel", CurFn);
5502:       llvm::BasicBlock *SeqCstBB = createBasicBlock("seqcst", CurFn);
5503: 
5504:       Order = Builder.CreateIntCast(Order, Builder.getInt32Ty(), false);
5505:       llvm::SwitchInst *SI = Builder.CreateSwitch(Order, ContBB);
5506:       SI->addCase(Builder.getInt32(1), AcquireBB);
5507:       SI->addCase(Builder.getInt32(2), AcquireBB);
5508:       SI->addCase(Builder.getInt32(3), ReleaseBB);
5509:       SI->addCase(Builder.getInt32(4), AcqRelBB);
5510:       SI->addCase(Builder.getInt32(5), SeqCstBB);
5511: 
5512:       OrderBBs.emplace_back(AcquireBB, llvm::AtomicOrdering::Acquire);
5513:       OrderBBs.emplace_back(ReleaseBB, llvm::AtomicOrdering::Release);
5514:       OrderBBs.emplace_back(AcqRelBB, llvm::AtomicOrdering::AcquireRelease);
5515:       OrderBBs.emplace_back(SeqCstBB,
5516:                             llvm::AtomicOrdering::SequentiallyConsistent);
5517:     }
5518: 
5519:     for (auto &[OrderBB, Ordering] : OrderBBs) {
5520:       Builder.SetInsertPoint(OrderBB);
```
- **EN**: This block uses control flow (for, case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（for, case）细化 LLVM IR 生成 行为。

### Lines 5521-5550
```cpp
5521:       if (Scp) {
5522:         SyncScope SS = ScopeModel->isValid(Scp->getZExtValue())
5523:                            ? ScopeModel->map(Scp->getZExtValue())
5524:                            : ScopeModel->map(ScopeModel->getFallBackValue());
5525:         Builder.CreateFence(Ordering,
5526:                             getTargetHooks().getLLVMSyncScopeID(
5527:                                 getLangOpts(), SS, Ordering, getLLVMContext()));
5528:         Builder.CreateBr(ContBB);
5529:       } else {
5530:         llvm::DenseMap<unsigned, llvm::BasicBlock *> BBs;
5531:         for (unsigned Scp : ScopeModel->getRuntimeValues())
5532:           BBs[Scp] = createBasicBlock(getAsString(ScopeModel->map(Scp)), CurFn);
5533: 
5534:         auto *SC = Builder.CreateIntCast(Scope, Builder.getInt32Ty(), false);
5535:         llvm::SwitchInst *SI = Builder.CreateSwitch(SC, ContBB);
5536:         for (unsigned Scp : ScopeModel->getRuntimeValues()) {
5537:           auto *B = BBs[Scp];
5538:           SI->addCase(Builder.getInt32(Scp), B);
5539: 
5540:           Builder.SetInsertPoint(B);
5541:           Builder.CreateFence(Ordering, getTargetHooks().getLLVMSyncScopeID(
5542:                                             getLangOpts(), ScopeModel->map(Scp),
5543:                                             Ordering, getLLVMContext()));
5544:           Builder.CreateBr(ContBB);
5545:         }
5546:       }
5547:     }
5548: 
5549:     Builder.SetInsertPoint(ContBB);
5550:     return RValue::get(nullptr);
```
- **EN**: This block defines callable entry points like `getTargetHooks`, `getLangOpts`, `get`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getTargetHooks`, `getLangOpts`, `get`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 5551-5580
```cpp
5551:   }
5552: 
5553:   case Builtin::BI__builtin_signbit:
5554:   case Builtin::BI__builtin_signbitf:
5555:   case Builtin::BI__builtin_signbitl: {
5556:     return RValue::get(
5557:         Builder.CreateZExt(EmitSignBit(*this, EmitScalarExpr(E->getArg(0))),
5558:                            ConvertType(E->getType())));
5559:   }
5560:   case Builtin::BI__warn_memset_zero_len:
5561:     return RValue::getIgnored();
5562:   case Builtin::BI__annotation: {
5563:     // Re-encode each wide string to UTF8 and make an MDString.
5564:     SmallVector<Metadata *, 1> Strings;
5565:     for (const Expr *Arg : E->arguments()) {
5566:       const auto *Str = cast<StringLiteral>(Arg->IgnoreParenCasts());
5567:       assert(Str->getCharByteWidth() == 2 || Str->getCharByteWidth() == 4);
5568:       StringRef WideBytes = Str->getBytes();
5569:       std::string StrUtf8;
5570:       bool Converted =
5571:           (Str->getCharByteWidth() == 2)
5572:               ? convertUTF16ToUTF8String(
5573:                     ArrayRef(WideBytes.data(), WideBytes.size()), StrUtf8)
5574:               : convertUTF32ToUTF8String(
5575:                     ArrayRef(WideBytes.data(), WideBytes.size()), StrUtf8);
5576:       if (!Converted) {
5577:         CGM.ErrorUnsupported(E, "non-Unicode __annotation argument");
5578:         continue;
5579:       }
5580:       Strings.push_back(llvm::MDString::get(getLLVMContext(), StrUtf8));
```
- **EN**: This block defines callable entry points like `get`, `getIgnored`, `ArrayRef`; uses control flow (if, for, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getIgnored`, `ArrayRef`；通过控制流（if, for, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 5581-5610
```cpp
5581:     }
5582: 
5583:     // Build and MDTuple of MDStrings and emit the intrinsic call.
5584:     llvm::Function *F = CGM.getIntrinsic(Intrinsic::codeview_annotation, {});
5585:     MDTuple *StrTuple = MDTuple::get(getLLVMContext(), Strings);
5586:     Builder.CreateCall(F, MetadataAsValue::get(getLLVMContext(), StrTuple));
5587:     return RValue::getIgnored();
5588:   }
5589:   case Builtin::BI__builtin_annotation: {
5590:     llvm::Value *AnnVal = EmitScalarExpr(E->getArg(0));
5591:     llvm::Function *F = CGM.getIntrinsic(
5592:         Intrinsic::annotation, {AnnVal->getType(), CGM.ConstGlobalsPtrTy});
5593: 
5594:     // Get the annotation string, go through casts. Sema requires this to be a
5595:     // non-wide string literal, potentially casted, so the cast<> is safe.
5596:     const Expr *AnnotationStrExpr = E->getArg(1)->IgnoreParenCasts();
5597:     StringRef Str = cast<StringLiteral>(AnnotationStrExpr)->getString();
5598:     return RValue::get(
5599:         EmitAnnotationCall(F, AnnVal, Str, E->getExprLoc(), nullptr));
5600:   }
5601:   case Builtin::BI__builtin_addcb:
5602:   case Builtin::BI__builtin_addcs:
5603:   case Builtin::BI__builtin_addc:
5604:   case Builtin::BI__builtin_addcl:
5605:   case Builtin::BI__builtin_addcll:
5606:   case Builtin::BI__builtin_subcb:
5607:   case Builtin::BI__builtin_subcs:
5608:   case Builtin::BI__builtin_subc:
5609:   case Builtin::BI__builtin_subcl:
5610:   case Builtin::BI__builtin_subcll: {
```
- **EN**: This block defines callable entry points like `getIgnored`, `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getIgnored`, `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5611-5640
```cpp
5611: 
5612:     // We translate all of these builtins from expressions of the form:
5613:     //   int x = ..., y = ..., carryin = ..., carryout, result;
5614:     //   result = __builtin_addc(x, y, carryin, &carryout);
5615:     //
5616:     // to LLVM IR of the form:
5617:     //
5618:     //   %tmp1 = call {i32, i1} @llvm.uadd.with.overflow.i32(i32 %x, i32 %y)
5619:     //   %tmpsum1 = extractvalue {i32, i1} %tmp1, 0
5620:     //   %carry1 = extractvalue {i32, i1} %tmp1, 1
5621:     //   %tmp2 = call {i32, i1} @llvm.uadd.with.overflow.i32(i32 %tmpsum1,
5622:     //                                                       i32 %carryin)
5623:     //   %result = extractvalue {i32, i1} %tmp2, 0
5624:     //   %carry2 = extractvalue {i32, i1} %tmp2, 1
5625:     //   %tmp3 = or i1 %carry1, %carry2
5626:     //   %tmp4 = zext i1 %tmp3 to i32
5627:     //   store i32 %tmp4, i32* %carryout
5628: 
5629:     // Scalarize our inputs.
5630:     llvm::Value *X = EmitScalarExpr(E->getArg(0));
5631:     llvm::Value *Y = EmitScalarExpr(E->getArg(1));
5632:     llvm::Value *Carryin = EmitScalarExpr(E->getArg(2));
5633:     Address CarryOutPtr = EmitPointerWithAlignment(E->getArg(3));
5634: 
5635:     // Decide if we are lowering to a uadd.with.overflow or usub.with.overflow.
5636:     Intrinsic::ID IntrinsicId;
5637:     switch (BuiltinID) {
5638:     default: llvm_unreachable("Unknown multiprecision builtin id.");
5639:     case Builtin::BI__builtin_addcb:
5640:     case Builtin::BI__builtin_addcs:
```
- **EN**: This block uses control flow (switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 5641-5670
```cpp
5641:     case Builtin::BI__builtin_addc:
5642:     case Builtin::BI__builtin_addcl:
5643:     case Builtin::BI__builtin_addcll:
5644:       IntrinsicId = Intrinsic::uadd_with_overflow;
5645:       break;
5646:     case Builtin::BI__builtin_subcb:
5647:     case Builtin::BI__builtin_subcs:
5648:     case Builtin::BI__builtin_subc:
5649:     case Builtin::BI__builtin_subcl:
5650:     case Builtin::BI__builtin_subcll:
5651:       IntrinsicId = Intrinsic::usub_with_overflow;
5652:       break;
5653:     }
5654: 
5655:     // Construct our resulting LLVM IR expression.
5656:     llvm::Value *Carry1;
5657:     llvm::Value *Sum1 = EmitOverflowIntrinsic(*this, IntrinsicId,
5658:                                               X, Y, Carry1);
5659:     llvm::Value *Carry2;
5660:     llvm::Value *Sum2 = EmitOverflowIntrinsic(*this, IntrinsicId,
5661:                                               Sum1, Carryin, Carry2);
5662:     llvm::Value *CarryOut = Builder.CreateZExt(Builder.CreateOr(Carry1, Carry2),
5663:                                                X->getType());
5664:     Builder.CreateStore(CarryOut, CarryOutPtr);
5665:     return RValue::get(Sum2);
5666:   }
5667: 
5668:   case Builtin::BI__builtin_add_overflow:
5669:   case Builtin::BI__builtin_sub_overflow:
5670:   case Builtin::BI__builtin_mul_overflow: {
```
- **EN**: This block defines callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5671-5700
```cpp
5671:     const clang::Expr *LeftArg = E->getArg(0);
5672:     const clang::Expr *RightArg = E->getArg(1);
5673:     const clang::Expr *ResultArg = E->getArg(2);
5674: 
5675:     clang::QualType ResultQTy =
5676:         ResultArg->getType()->castAs<PointerType>()->getPointeeType();
5677: 
5678:     WidthAndSignedness LeftInfo =
5679:         getIntegerWidthAndSignedness(CGM.getContext(), LeftArg->getType());
5680:     WidthAndSignedness RightInfo =
5681:         getIntegerWidthAndSignedness(CGM.getContext(), RightArg->getType());
5682:     WidthAndSignedness ResultInfo =
5683:         getIntegerWidthAndSignedness(CGM.getContext(), ResultQTy);
5684: 
5685:     // Handle mixed-sign multiplication as a special case, because adding
5686:     // runtime or backend support for our generic irgen would be too expensive.
5687:     if (isSpecialMixedSignMultiply(BuiltinID, LeftInfo, RightInfo, ResultInfo))
5688:       return EmitCheckedMixedSignMultiply(*this, LeftArg, LeftInfo, RightArg,
5689:                                           RightInfo, ResultArg, ResultQTy,
5690:                                           ResultInfo);
5691: 
5692:     if (isSpecialUnsignedMultiplySignedResult(BuiltinID, LeftInfo, RightInfo,
5693:                                               ResultInfo))
5694:       return EmitCheckedUnsignedMultiplySignedResult(
5695:           *this, LeftArg, LeftInfo, RightArg, RightInfo, ResultArg, ResultQTy,
5696:           ResultInfo);
5697: 
5698:     WidthAndSignedness EncompassingInfo =
5699:         EncompassingIntegerType({LeftInfo, RightInfo, ResultInfo});
5700: 
```
- **EN**: This block defines callable entry points like `getIntegerWidthAndSignedness`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getIntegerWidthAndSignedness`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 5701-5730
```cpp
5701:     llvm::Type *EncompassingLLVMTy =
5702:         llvm::IntegerType::get(CGM.getLLVMContext(), EncompassingInfo.Width);
5703: 
5704:     llvm::Type *ResultLLVMTy = CGM.getTypes().ConvertType(ResultQTy);
5705: 
5706:     Intrinsic::ID IntrinsicId;
5707:     switch (BuiltinID) {
5708:     default:
5709:       llvm_unreachable("Unknown overflow builtin id.");
5710:     case Builtin::BI__builtin_add_overflow:
5711:       IntrinsicId = EncompassingInfo.Signed ? Intrinsic::sadd_with_overflow
5712:                                             : Intrinsic::uadd_with_overflow;
5713:       break;
5714:     case Builtin::BI__builtin_sub_overflow:
5715:       IntrinsicId = EncompassingInfo.Signed ? Intrinsic::ssub_with_overflow
5716:                                             : Intrinsic::usub_with_overflow;
5717:       break;
5718:     case Builtin::BI__builtin_mul_overflow:
5719:       IntrinsicId = EncompassingInfo.Signed ? Intrinsic::smul_with_overflow
5720:                                             : Intrinsic::umul_with_overflow;
5721:       break;
5722:     }
5723: 
5724:     llvm::Value *Left = EmitScalarExpr(LeftArg);
5725:     llvm::Value *Right = EmitScalarExpr(RightArg);
5726:     Address ResultPtr = EmitPointerWithAlignment(ResultArg);
5727: 
5728:     // Extend each operand to the encompassing type.
5729:     Left = Builder.CreateIntCast(Left, EncompassingLLVMTy, LeftInfo.Signed);
5730:     Right = Builder.CreateIntCast(Right, EncompassingLLVMTy, RightInfo.Signed);
```
- **EN**: This block defines callable entry points like `get`; uses control flow (switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 5731-5760
```cpp
5731: 
5732:     // Perform the operation on the extended values.
5733:     llvm::Value *Overflow, *Result;
5734:     Result = EmitOverflowIntrinsic(*this, IntrinsicId, Left, Right, Overflow);
5735: 
5736:     if (EncompassingInfo.Width > ResultInfo.Width) {
5737:       // The encompassing type is wider than the result type, so we need to
5738:       // truncate it.
5739:       llvm::Value *ResultTrunc = Builder.CreateTrunc(Result, ResultLLVMTy);
5740: 
5741:       // To see if the truncation caused an overflow, we will extend
5742:       // the result and then compare it to the original result.
5743:       llvm::Value *ResultTruncExt = Builder.CreateIntCast(
5744:           ResultTrunc, EncompassingLLVMTy, ResultInfo.Signed);
5745:       llvm::Value *TruncationOverflow =
5746:           Builder.CreateICmpNE(Result, ResultTruncExt);
5747: 
5748:       Overflow = Builder.CreateOr(Overflow, TruncationOverflow);
5749:       Result = ResultTrunc;
5750:     }
5751: 
5752:     // Finally, store the result using the pointer.
5753:     bool isVolatile =
5754:       ResultArg->getType()->getPointeeType().isVolatileQualified();
5755:     Builder.CreateStore(EmitToMemory(Result, ResultQTy), ResultPtr, isVolatile);
5756: 
5757:     return RValue::get(Overflow);
5758:   }
5759: 
5760:   case Builtin::BI__builtin_uadd_overflow:
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 5761-5790
```cpp
5761:   case Builtin::BI__builtin_uaddl_overflow:
5762:   case Builtin::BI__builtin_uaddll_overflow:
5763:   case Builtin::BI__builtin_usub_overflow:
5764:   case Builtin::BI__builtin_usubl_overflow:
5765:   case Builtin::BI__builtin_usubll_overflow:
5766:   case Builtin::BI__builtin_umul_overflow:
5767:   case Builtin::BI__builtin_umull_overflow:
5768:   case Builtin::BI__builtin_umulll_overflow:
5769:   case Builtin::BI__builtin_sadd_overflow:
5770:   case Builtin::BI__builtin_saddl_overflow:
5771:   case Builtin::BI__builtin_saddll_overflow:
5772:   case Builtin::BI__builtin_ssub_overflow:
5773:   case Builtin::BI__builtin_ssubl_overflow:
5774:   case Builtin::BI__builtin_ssubll_overflow:
5775:   case Builtin::BI__builtin_smul_overflow:
5776:   case Builtin::BI__builtin_smull_overflow:
5777:   case Builtin::BI__builtin_smulll_overflow: {
5778: 
5779:     // We translate all of these builtins directly to the relevant llvm IR node.
5780: 
5781:     // Scalarize our inputs.
5782:     llvm::Value *X = EmitScalarExpr(E->getArg(0));
5783:     llvm::Value *Y = EmitScalarExpr(E->getArg(1));
5784:     Address SumOutPtr = EmitPointerWithAlignment(E->getArg(2));
5785: 
5786:     // Decide which of the overflow intrinsics we are lowering to:
5787:     Intrinsic::ID IntrinsicId;
5788:     switch (BuiltinID) {
5789:     default: llvm_unreachable("Unknown overflow builtin id.");
5790:     case Builtin::BI__builtin_uadd_overflow:
```
- **EN**: This block uses control flow (switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 5791-5820
```cpp
5791:     case Builtin::BI__builtin_uaddl_overflow:
5792:     case Builtin::BI__builtin_uaddll_overflow:
5793:       IntrinsicId = Intrinsic::uadd_with_overflow;
5794:       break;
5795:     case Builtin::BI__builtin_usub_overflow:
5796:     case Builtin::BI__builtin_usubl_overflow:
5797:     case Builtin::BI__builtin_usubll_overflow:
5798:       IntrinsicId = Intrinsic::usub_with_overflow;
5799:       break;
5800:     case Builtin::BI__builtin_umul_overflow:
5801:     case Builtin::BI__builtin_umull_overflow:
5802:     case Builtin::BI__builtin_umulll_overflow:
5803:       IntrinsicId = Intrinsic::umul_with_overflow;
5804:       break;
5805:     case Builtin::BI__builtin_sadd_overflow:
5806:     case Builtin::BI__builtin_saddl_overflow:
5807:     case Builtin::BI__builtin_saddll_overflow:
5808:       IntrinsicId = Intrinsic::sadd_with_overflow;
5809:       break;
5810:     case Builtin::BI__builtin_ssub_overflow:
5811:     case Builtin::BI__builtin_ssubl_overflow:
5812:     case Builtin::BI__builtin_ssubll_overflow:
5813:       IntrinsicId = Intrinsic::ssub_with_overflow;
5814:       break;
5815:     case Builtin::BI__builtin_smul_overflow:
5816:     case Builtin::BI__builtin_smull_overflow:
5817:     case Builtin::BI__builtin_smulll_overflow:
5818:       IntrinsicId = Intrinsic::smul_with_overflow;
5819:       break;
5820:     }
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5821-5850
```cpp
5821: 
5822: 
5823:     llvm::Value *Carry;
5824:     llvm::Value *Sum = EmitOverflowIntrinsic(*this, IntrinsicId, X, Y, Carry);
5825:     Builder.CreateStore(Sum, SumOutPtr);
5826: 
5827:     return RValue::get(Carry);
5828:   }
5829:   case Builtin::BIaddressof:
5830:   case Builtin::BI__addressof:
5831:   case Builtin::BI__builtin_addressof:
5832:     return RValue::get(EmitLValue(E->getArg(0)).getPointer(*this));
5833:   case Builtin::BI__builtin_function_start:
5834:     return RValue::get(CGM.GetFunctionStart(
5835:         E->getArg(0)->getAsBuiltinConstantDeclRef(CGM.getContext())));
5836:   case Builtin::BI__builtin_operator_new:
5837:     return EmitBuiltinNewDeleteCall(
5838:         E->getCallee()->getType()->castAs<FunctionProtoType>(), E, false);
5839:   case Builtin::BI__builtin_operator_delete:
5840:     EmitBuiltinNewDeleteCall(
5841:         E->getCallee()->getType()->castAs<FunctionProtoType>(), E, true);
5842:     return RValue::get(nullptr);
5843: 
5844:   case Builtin::BI__builtin_is_aligned:
5845:     return EmitBuiltinIsAligned(E);
5846:   case Builtin::BI__builtin_align_up:
5847:     return EmitBuiltinAlignTo(E, true);
5848:   case Builtin::BI__builtin_align_down:
5849:     return EmitBuiltinAlignTo(E, false);
5850: 
```
- **EN**: This block spells out callable entry points like `get`, `EmitBuiltinNewDeleteCall`, `EmitBuiltinIsAligned`, `EmitBuiltinAlignTo`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`, `EmitBuiltinNewDeleteCall`, `EmitBuiltinIsAligned`, `EmitBuiltinAlignTo`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5851-5880
```cpp
5851:   case Builtin::BI__noop:
5852:     // __noop always evaluates to an integer literal zero.
5853:     return RValue::get(ConstantInt::get(IntTy, 0));
5854:   case Builtin::BI__builtin_call_with_static_chain: {
5855:     const CallExpr *Call = cast<CallExpr>(E->getArg(0));
5856:     const Expr *Chain = E->getArg(1);
5857:     return EmitCall(Call->getCallee()->getType(),
5858:                     EmitCallee(Call->getCallee()), Call, ReturnValue,
5859:                     EmitScalarExpr(Chain));
5860:   }
5861:   case Builtin::BI_InterlockedExchange8:
5862:   case Builtin::BI_InterlockedExchange16:
5863:   case Builtin::BI_InterlockedExchange:
5864:   case Builtin::BI_InterlockedExchangePointer:
5865:     return RValue::get(
5866:         EmitMSVCBuiltinExpr(MSVCIntrin::_InterlockedExchange, E));
5867:   case Builtin::BI_InterlockedCompareExchangePointer:
5868:     return RValue::get(
5869:         EmitMSVCBuiltinExpr(MSVCIntrin::_InterlockedCompareExchange, E));
5870:   case Builtin::BI_InterlockedCompareExchangePointer_nf:
5871:     return RValue::get(
5872:         EmitMSVCBuiltinExpr(MSVCIntrin::_InterlockedCompareExchange_nf, E));
5873:   case Builtin::BI_InterlockedCompareExchange8:
5874:   case Builtin::BI_InterlockedCompareExchange16:
5875:   case Builtin::BI_InterlockedCompareExchange:
5876:   case Builtin::BI_InterlockedCompareExchange64:
5877:     return RValue::get(EmitAtomicCmpXchgForMSIntrin(*this, E));
5878:   case Builtin::BI_InterlockedIncrement16:
5879:   case Builtin::BI_InterlockedIncrement:
5880:     return RValue::get(
```
- **EN**: This block defines callable entry points like `get`, `EmitCall`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `EmitCall`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5881-5910
```cpp
5881:         EmitMSVCBuiltinExpr(MSVCIntrin::_InterlockedIncrement, E));
5882:   case Builtin::BI_InterlockedDecrement16:
5883:   case Builtin::BI_InterlockedDecrement:
5884:     return RValue::get(
5885:         EmitMSVCBuiltinExpr(MSVCIntrin::_InterlockedDecrement, E));
5886:   case Builtin::BI_InterlockedAnd8:
5887:   case Builtin::BI_InterlockedAnd16:
5888:   case Builtin::BI_InterlockedAnd:
5889:     return RValue::get(EmitMSVCBuiltinExpr(MSVCIntrin::_InterlockedAnd, E));
5890:   case Builtin::BI_InterlockedExchangeAdd8:
5891:   case Builtin::BI_InterlockedExchangeAdd16:
5892:   case Builtin::BI_InterlockedExchangeAdd:
5893:     return RValue::get(
5894:         EmitMSVCBuiltinExpr(MSVCIntrin::_InterlockedExchangeAdd, E));
5895:   case Builtin::BI_InterlockedExchangeSub8:
5896:   case Builtin::BI_InterlockedExchangeSub16:
5897:   case Builtin::BI_InterlockedExchangeSub:
5898:     return RValue::get(
5899:         EmitMSVCBuiltinExpr(MSVCIntrin::_InterlockedExchangeSub, E));
5900:   case Builtin::BI_InterlockedOr8:
5901:   case Builtin::BI_InterlockedOr16:
5902:   case Builtin::BI_InterlockedOr:
5903:     return RValue::get(EmitMSVCBuiltinExpr(MSVCIntrin::_InterlockedOr, E));
5904:   case Builtin::BI_InterlockedXor8:
5905:   case Builtin::BI_InterlockedXor16:
5906:   case Builtin::BI_InterlockedXor:
5907:     return RValue::get(EmitMSVCBuiltinExpr(MSVCIntrin::_InterlockedXor, E));
5908: 
5909:   case Builtin::BI_bittest64:
5910:   case Builtin::BI_bittest:
```
- **EN**: This block spells out callable entry points like `EmitMSVCBuiltinExpr`, `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitMSVCBuiltinExpr`, `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5911-5940
```cpp
5911:   case Builtin::BI_bittestandcomplement64:
5912:   case Builtin::BI_bittestandcomplement:
5913:   case Builtin::BI_bittestandreset64:
5914:   case Builtin::BI_bittestandreset:
5915:   case Builtin::BI_bittestandset64:
5916:   case Builtin::BI_bittestandset:
5917:   case Builtin::BI_interlockedbittestandreset:
5918:   case Builtin::BI_interlockedbittestandreset64:
5919:   case Builtin::BI_interlockedbittestandreset64_acq:
5920:   case Builtin::BI_interlockedbittestandreset64_rel:
5921:   case Builtin::BI_interlockedbittestandreset64_nf:
5922:   case Builtin::BI_interlockedbittestandset64:
5923:   case Builtin::BI_interlockedbittestandset64_acq:
5924:   case Builtin::BI_interlockedbittestandset64_rel:
5925:   case Builtin::BI_interlockedbittestandset64_nf:
5926:   case Builtin::BI_interlockedbittestandset:
5927:   case Builtin::BI_interlockedbittestandset_acq:
5928:   case Builtin::BI_interlockedbittestandset_rel:
5929:   case Builtin::BI_interlockedbittestandset_nf:
5930:   case Builtin::BI_interlockedbittestandreset_acq:
5931:   case Builtin::BI_interlockedbittestandreset_rel:
5932:   case Builtin::BI_interlockedbittestandreset_nf:
5933:     return RValue::get(EmitBitTestIntrinsic(*this, BuiltinID, E));
5934: 
5935:     // These builtins exist to emit regular volatile loads and stores not
5936:     // affected by the -fms-volatile setting.
5937:   case Builtin::BI__iso_volatile_load8:
5938:   case Builtin::BI__iso_volatile_load16:
5939:   case Builtin::BI__iso_volatile_load32:
5940:   case Builtin::BI__iso_volatile_load64:
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 5941-5970
```cpp
5941:     return RValue::get(EmitISOVolatileLoad(*this, E));
5942:   case Builtin::BI__iso_volatile_store8:
5943:   case Builtin::BI__iso_volatile_store16:
5944:   case Builtin::BI__iso_volatile_store32:
5945:   case Builtin::BI__iso_volatile_store64:
5946:     return RValue::get(EmitISOVolatileStore(*this, E));
5947: 
5948:   case Builtin::BI__builtin_ptrauth_sign_constant:
5949:     return RValue::get(ConstantEmitter(*this).emitAbstract(E, E->getType()));
5950: 
5951:   case Builtin::BI__builtin_ptrauth_auth:
5952:   case Builtin::BI__builtin_ptrauth_auth_and_resign:
5953:   case Builtin::BI__builtin_ptrauth_auth_load_relative_and_sign:
5954:   case Builtin::BI__builtin_ptrauth_blend_discriminator:
5955:   case Builtin::BI__builtin_ptrauth_sign_generic_data:
5956:   case Builtin::BI__builtin_ptrauth_sign_unauthenticated:
5957:   case Builtin::BI__builtin_ptrauth_strip: {
5958:     // Emit the arguments.
5959:     SmallVector<llvm::Value *, 6> Args;
5960:     for (auto argExpr : E->arguments())
5961:       Args.push_back(EmitScalarExpr(argExpr));
5962: 
5963:     // Cast the value to intptr_t, saving its original type.
5964:     llvm::Type *OrigValueType = Args[0]->getType();
5965:     if (OrigValueType->isPointerTy())
5966:       Args[0] = Builder.CreatePtrToInt(Args[0], IntPtrTy);
5967: 
5968:     switch (BuiltinID) {
5969:     case Builtin::BI__builtin_ptrauth_auth_and_resign:
5970:     case Builtin::BI__builtin_ptrauth_auth_load_relative_and_sign:
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, switch, for, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, switch, for, case）细化 LLVM IR 生成 行为。

### Lines 5971-6000
```cpp
5971:       if (Args[4]->getType()->isPointerTy())
5972:         Args[4] = Builder.CreatePtrToInt(Args[4], IntPtrTy);
5973:       [[fallthrough]];
5974: 
5975:     case Builtin::BI__builtin_ptrauth_auth:
5976:     case Builtin::BI__builtin_ptrauth_sign_unauthenticated:
5977:       if (Args[2]->getType()->isPointerTy())
5978:         Args[2] = Builder.CreatePtrToInt(Args[2], IntPtrTy);
5979:       break;
5980: 
5981:     case Builtin::BI__builtin_ptrauth_sign_generic_data:
5982:       if (Args[1]->getType()->isPointerTy())
5983:         Args[1] = Builder.CreatePtrToInt(Args[1], IntPtrTy);
5984:       break;
5985: 
5986:     case Builtin::BI__builtin_ptrauth_blend_discriminator:
5987:     case Builtin::BI__builtin_ptrauth_strip:
5988:       break;
5989:     }
5990: 
5991:     // Call the intrinsic.
5992:     auto IntrinsicID = [&]() -> unsigned {
5993:       switch (BuiltinID) {
5994:       case Builtin::BI__builtin_ptrauth_auth:
5995:         return Intrinsic::ptrauth_auth;
5996:       case Builtin::BI__builtin_ptrauth_auth_and_resign:
5997:         return Intrinsic::ptrauth_resign;
5998:       case Builtin::BI__builtin_ptrauth_auth_load_relative_and_sign:
5999:         return Intrinsic::ptrauth_resign_load_relative;
6000:       case Builtin::BI__builtin_ptrauth_blend_discriminator:
```
- **EN**: This block uses control flow (if, switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, switch, case）细化 LLVM IR 生成 行为。

### Lines 6001-6030
```cpp
6001:         return Intrinsic::ptrauth_blend;
6002:       case Builtin::BI__builtin_ptrauth_sign_generic_data:
6003:         return Intrinsic::ptrauth_sign_generic;
6004:       case Builtin::BI__builtin_ptrauth_sign_unauthenticated:
6005:         return Intrinsic::ptrauth_sign;
6006:       case Builtin::BI__builtin_ptrauth_strip:
6007:         return Intrinsic::ptrauth_strip;
6008:       }
6009:       llvm_unreachable("bad ptrauth intrinsic");
6010:     }();
6011:     auto Intrinsic = CGM.getIntrinsic(IntrinsicID);
6012:     llvm::Value *Result = EmitRuntimeCall(Intrinsic, Args);
6013: 
6014:     if (BuiltinID != Builtin::BI__builtin_ptrauth_sign_generic_data &&
6015:         BuiltinID != Builtin::BI__builtin_ptrauth_blend_discriminator &&
6016:         OrigValueType->isPointerTy()) {
6017:       Result = Builder.CreateIntToPtr(Result, OrigValueType);
6018:     }
6019:     return RValue::get(Result);
6020:   }
6021: 
6022:   case Builtin::BI__builtin_get_vtable_pointer: {
6023:     const Expr *Target = E->getArg(0);
6024:     QualType TargetType = Target->getType();
6025:     const CXXRecordDecl *Decl = TargetType->getPointeeCXXRecordDecl();
6026:     assert(Decl);
6027:     auto ThisAddress = EmitPointerWithAlignment(Target);
6028:     assert(ThisAddress.isValid());
6029:     llvm::Value *VTablePointer =
6030:         GetVTablePtr(ThisAddress, Int8PtrTy, Decl, VTableAuthMode::MustTrap);
```
- **EN**: This block defines callable entry points like `get`, `GetVTablePtr`; uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`, `GetVTablePtr`；通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 6031-6060
```cpp
6031:     return RValue::get(VTablePointer);
6032:   }
6033: 
6034:   case Builtin::BI__exception_code:
6035:   case Builtin::BI_exception_code:
6036:     return RValue::get(EmitSEHExceptionCode());
6037:   case Builtin::BI__exception_info:
6038:   case Builtin::BI_exception_info:
6039:     return RValue::get(EmitSEHExceptionInfo());
6040:   case Builtin::BI__abnormal_termination:
6041:   case Builtin::BI_abnormal_termination:
6042:     return RValue::get(EmitSEHAbnormalTermination());
6043:   case Builtin::BI_setjmpex:
6044:     if (getTarget().getTriple().isOSMSVCRT() && E->getNumArgs() == 1 &&
6045:         E->getArg(0)->getType()->isPointerType())
6046:       return EmitMSVCRTSetJmp(*this, MSVCSetJmpKind::_setjmpex, E);
6047:     break;
6048:   case Builtin::BI_setjmp:
6049:     if (getTarget().getTriple().isOSMSVCRT() && E->getNumArgs() == 1 &&
6050:         E->getArg(0)->getType()->isPointerType()) {
6051:       if (getTarget().getTriple().getArch() == llvm::Triple::x86)
6052:         return EmitMSVCRTSetJmp(*this, MSVCSetJmpKind::_setjmp3, E);
6053:       else if (getTarget().getTriple().getArch() == llvm::Triple::aarch64)
6054:         return EmitMSVCRTSetJmp(*this, MSVCSetJmpKind::_setjmpex, E);
6055:       return EmitMSVCRTSetJmp(*this, MSVCSetJmpKind::_setjmp, E);
6056:     }
6057:     break;
6058: 
6059:   // C++ std:: builtins.
6060:   case Builtin::BImove:
```
- **EN**: This block defines callable entry points like `get`, `EmitMSVCRTSetJmp`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `EmitMSVCRTSetJmp`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 6061-6090
```cpp
6061:   case Builtin::BImove_if_noexcept:
6062:   case Builtin::BIforward:
6063:   case Builtin::BIforward_like:
6064:   case Builtin::BIas_const:
6065:     return RValue::get(EmitLValue(E->getArg(0)).getPointer(*this));
6066:   case Builtin::BI__GetExceptionInfo: {
6067:     if (llvm::GlobalVariable *GV =
6068:             CGM.getCXXABI().getThrowInfo(FD->getParamDecl(0)->getType()))
6069:       return RValue::get(GV);
6070:     break;
6071:   }
6072: 
6073:   case Builtin::BI__fastfail:
6074:     return RValue::get(EmitMSVCBuiltinExpr(MSVCIntrin::__fastfail, E));
6075: 
6076:   case Builtin::BI__builtin_coro_id:
6077:     return EmitCoroutineIntrinsic(E, Intrinsic::coro_id);
6078:   case Builtin::BI__builtin_coro_promise:
6079:     return EmitCoroutineIntrinsic(E, Intrinsic::coro_promise);
6080:   case Builtin::BI__builtin_coro_resume:
6081:     EmitCoroutineIntrinsic(E, Intrinsic::coro_resume);
6082:     return RValue::get(nullptr);
6083:   case Builtin::BI__builtin_coro_frame:
6084:     return EmitCoroutineIntrinsic(E, Intrinsic::coro_frame);
6085:   case Builtin::BI__builtin_coro_noop:
6086:     return EmitCoroutineIntrinsic(E, Intrinsic::coro_noop);
6087:   case Builtin::BI__builtin_coro_free:
6088:     return EmitCoroutineIntrinsic(E, Intrinsic::coro_free);
6089:   case Builtin::BI__builtin_coro_destroy:
6090:     EmitCoroutineIntrinsic(E, Intrinsic::coro_destroy);
```
- **EN**: This block defines callable entry points like `get`, `EmitCoroutineIntrinsic`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `EmitCoroutineIntrinsic`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 6091-6120
```cpp
6091:     return RValue::get(nullptr);
6092:   case Builtin::BI__builtin_coro_done:
6093:     return EmitCoroutineIntrinsic(E, Intrinsic::coro_done);
6094:   case Builtin::BI__builtin_coro_alloc:
6095:     return EmitCoroutineIntrinsic(E, Intrinsic::coro_alloc);
6096:   case Builtin::BI__builtin_coro_begin:
6097:     return EmitCoroutineIntrinsic(E, Intrinsic::coro_begin);
6098:   case Builtin::BI__builtin_coro_end:
6099:     return EmitCoroutineIntrinsic(E, Intrinsic::coro_end);
6100:   case Builtin::BI__builtin_coro_suspend:
6101:     return EmitCoroutineIntrinsic(E, Intrinsic::coro_suspend);
6102:   case Builtin::BI__builtin_coro_size:
6103:     return EmitCoroutineIntrinsic(E, Intrinsic::coro_size);
6104:   case Builtin::BI__builtin_coro_align:
6105:     return EmitCoroutineIntrinsic(E, Intrinsic::coro_align);
6106: 
6107:   // OpenCL v2.0 s6.13.16.2, Built-in pipe read and write functions
6108:   case Builtin::BIread_pipe:
6109:   case Builtin::BIwrite_pipe: {
6110:     Value *Arg0 = EmitScalarExpr(E->getArg(0)),
6111:           *Arg1 = EmitScalarExpr(E->getArg(1));
6112:     CGOpenCLRuntime OpenCLRT(CGM);
6113:     Value *PacketSize = OpenCLRT.getPipeElemSize(E->getArg(0));
6114:     Value *PacketAlign = OpenCLRT.getPipeElemAlign(E->getArg(0));
6115: 
6116:     // Type of the generic packet parameter.
6117:     unsigned GenericAS =
6118:         getContext().getTargetAddressSpace(LangAS::opencl_generic);
6119:     llvm::Type *I8PTy = llvm::PointerType::get(getLLVMContext(), GenericAS);
6120: 
```
- **EN**: This block defines callable entry points like `get`, `EmitCoroutineIntrinsic`, `OpenCLRT`, `getContext`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `EmitCoroutineIntrinsic`, `OpenCLRT`, `getContext`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 6121-6150
```cpp
6121:     // Testing which overloaded version we should generate the call for.
6122:     if (2U == E->getNumArgs()) {
6123:       const char *Name = (BuiltinID == Builtin::BIread_pipe) ? "__read_pipe_2"
6124:                                                              : "__write_pipe_2";
6125:       // Creating a generic function type to be able to call with any builtin or
6126:       // user defined type.
6127:       llvm::Type *ArgTys[] = {Arg0->getType(), I8PTy, Int32Ty, Int32Ty};
6128:       llvm::FunctionType *FTy = llvm::FunctionType::get(Int32Ty, ArgTys, false);
6129:       Value *ACast = Builder.CreateAddrSpaceCast(Arg1, I8PTy);
6130:       return RValue::get(
6131:           EmitRuntimeCall(CGM.CreateRuntimeFunction(FTy, Name),
6132:                           {Arg0, ACast, PacketSize, PacketAlign}));
6133:     } else {
6134:       assert(4 == E->getNumArgs() &&
6135:              "Illegal number of parameters to pipe function");
6136:       const char *Name = (BuiltinID == Builtin::BIread_pipe) ? "__read_pipe_4"
6137:                                                              : "__write_pipe_4";
6138: 
6139:       llvm::Type *ArgTys[] = {Arg0->getType(), Arg1->getType(), Int32Ty, I8PTy,
6140:                               Int32Ty, Int32Ty};
6141:       Value *Arg2 = EmitScalarExpr(E->getArg(2)),
6142:             *Arg3 = EmitScalarExpr(E->getArg(3));
6143:       llvm::FunctionType *FTy = llvm::FunctionType::get(Int32Ty, ArgTys, false);
6144:       Value *ACast = Builder.CreateAddrSpaceCast(Arg3, I8PTy);
6145:       // We know the third argument is an integer type, but we may need to cast
6146:       // it to i32.
6147:       if (Arg2->getType() != Int32Ty)
6148:         Arg2 = Builder.CreateZExtOrTrunc(Arg2, Int32Ty);
6149:       return RValue::get(
6150:           EmitRuntimeCall(CGM.CreateRuntimeFunction(FTy, Name),
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 6151-6180
```cpp
6151:                           {Arg0, Arg1, Arg2, ACast, PacketSize, PacketAlign}));
6152:     }
6153:   }
6154:   // OpenCL v2.0 s6.13.16 ,s9.17.3.5 - Built-in pipe reserve read and write
6155:   // functions
6156:   case Builtin::BIreserve_read_pipe:
6157:   case Builtin::BIreserve_write_pipe:
6158:   case Builtin::BIwork_group_reserve_read_pipe:
6159:   case Builtin::BIwork_group_reserve_write_pipe:
6160:   case Builtin::BIsub_group_reserve_read_pipe:
6161:   case Builtin::BIsub_group_reserve_write_pipe: {
6162:     // Composing the mangled name for the function.
6163:     const char *Name;
6164:     if (BuiltinID == Builtin::BIreserve_read_pipe)
6165:       Name = "__reserve_read_pipe";
6166:     else if (BuiltinID == Builtin::BIreserve_write_pipe)
6167:       Name = "__reserve_write_pipe";
6168:     else if (BuiltinID == Builtin::BIwork_group_reserve_read_pipe)
6169:       Name = "__work_group_reserve_read_pipe";
6170:     else if (BuiltinID == Builtin::BIwork_group_reserve_write_pipe)
6171:       Name = "__work_group_reserve_write_pipe";
6172:     else if (BuiltinID == Builtin::BIsub_group_reserve_read_pipe)
6173:       Name = "__sub_group_reserve_read_pipe";
6174:     else
6175:       Name = "__sub_group_reserve_write_pipe";
6176: 
6177:     Value *Arg0 = EmitScalarExpr(E->getArg(0)),
6178:           *Arg1 = EmitScalarExpr(E->getArg(1));
6179:     llvm::Type *ReservedIDTy = ConvertType(getContext().OCLReserveIDTy);
6180:     CGOpenCLRuntime OpenCLRT(CGM);
```
- **EN**: This block defines callable entry points like `OpenCLRT`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `OpenCLRT`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 6181-6210
```cpp
6181:     Value *PacketSize = OpenCLRT.getPipeElemSize(E->getArg(0));
6182:     Value *PacketAlign = OpenCLRT.getPipeElemAlign(E->getArg(0));
6183: 
6184:     // Building the generic function prototype.
6185:     llvm::Type *ArgTys[] = {Arg0->getType(), Int32Ty, Int32Ty, Int32Ty};
6186:     llvm::FunctionType *FTy =
6187:         llvm::FunctionType::get(ReservedIDTy, ArgTys, false);
6188:     // We know the second argument is an integer type, but we may need to cast
6189:     // it to i32.
6190:     if (Arg1->getType() != Int32Ty)
6191:       Arg1 = Builder.CreateZExtOrTrunc(Arg1, Int32Ty);
6192:     return RValue::get(EmitRuntimeCall(CGM.CreateRuntimeFunction(FTy, Name),
6193:                                        {Arg0, Arg1, PacketSize, PacketAlign}));
6194:   }
6195:   // OpenCL v2.0 s6.13.16, s9.17.3.5 - Built-in pipe commit read and write
6196:   // functions
6197:   case Builtin::BIcommit_read_pipe:
6198:   case Builtin::BIcommit_write_pipe:
6199:   case Builtin::BIwork_group_commit_read_pipe:
6200:   case Builtin::BIwork_group_commit_write_pipe:
6201:   case Builtin::BIsub_group_commit_read_pipe:
6202:   case Builtin::BIsub_group_commit_write_pipe: {
6203:     const char *Name;
6204:     if (BuiltinID == Builtin::BIcommit_read_pipe)
6205:       Name = "__commit_read_pipe";
6206:     else if (BuiltinID == Builtin::BIcommit_write_pipe)
6207:       Name = "__commit_write_pipe";
6208:     else if (BuiltinID == Builtin::BIwork_group_commit_read_pipe)
6209:       Name = "__work_group_commit_read_pipe";
6210:     else if (BuiltinID == Builtin::BIwork_group_commit_write_pipe)
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 6211-6240
```cpp
6211:       Name = "__work_group_commit_write_pipe";
6212:     else if (BuiltinID == Builtin::BIsub_group_commit_read_pipe)
6213:       Name = "__sub_group_commit_read_pipe";
6214:     else
6215:       Name = "__sub_group_commit_write_pipe";
6216: 
6217:     Value *Arg0 = EmitScalarExpr(E->getArg(0)),
6218:           *Arg1 = EmitScalarExpr(E->getArg(1));
6219:     CGOpenCLRuntime OpenCLRT(CGM);
6220:     Value *PacketSize = OpenCLRT.getPipeElemSize(E->getArg(0));
6221:     Value *PacketAlign = OpenCLRT.getPipeElemAlign(E->getArg(0));
6222: 
6223:     // Building the generic function prototype.
6224:     llvm::Type *ArgTys[] = {Arg0->getType(), Arg1->getType(), Int32Ty, Int32Ty};
6225:     llvm::FunctionType *FTy = llvm::FunctionType::get(
6226:         llvm::Type::getVoidTy(getLLVMContext()), ArgTys, false);
6227: 
6228:     return RValue::get(EmitRuntimeCall(CGM.CreateRuntimeFunction(FTy, Name),
6229:                                        {Arg0, Arg1, PacketSize, PacketAlign}));
6230:   }
6231:   // OpenCL v2.0 s6.13.16.4 Built-in pipe query functions
6232:   case Builtin::BIget_pipe_num_packets:
6233:   case Builtin::BIget_pipe_max_packets: {
6234:     const char *BaseName;
6235:     const auto *PipeTy = E->getArg(0)->getType()->castAs<PipeType>();
6236:     if (BuiltinID == Builtin::BIget_pipe_num_packets)
6237:       BaseName = "__get_pipe_num_packets";
6238:     else
6239:       BaseName = "__get_pipe_max_packets";
6240:     std::string Name = std::string(BaseName) +
```
- **EN**: This block defines callable entry points like `OpenCLRT`, `getVoidTy`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `OpenCLRT`, `getVoidTy`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 6241-6270
```cpp
6241:                        std::string(PipeTy->isReadOnly() ? "_ro" : "_wo");
6242: 
6243:     // Building the generic function prototype.
6244:     Value *Arg0 = EmitScalarExpr(E->getArg(0));
6245:     CGOpenCLRuntime OpenCLRT(CGM);
6246:     Value *PacketSize = OpenCLRT.getPipeElemSize(E->getArg(0));
6247:     Value *PacketAlign = OpenCLRT.getPipeElemAlign(E->getArg(0));
6248:     llvm::Type *ArgTys[] = {Arg0->getType(), Int32Ty, Int32Ty};
6249:     llvm::FunctionType *FTy = llvm::FunctionType::get(Int32Ty, ArgTys, false);
6250: 
6251:     return RValue::get(EmitRuntimeCall(CGM.CreateRuntimeFunction(FTy, Name),
6252:                                        {Arg0, PacketSize, PacketAlign}));
6253:   }
6254: 
6255:   // OpenCL v2.0 s6.13.9 - Address space qualifier functions.
6256:   case Builtin::BIto_global:
6257:   case Builtin::BIto_local:
6258:   case Builtin::BIto_private: {
6259:     auto Arg0 = EmitScalarExpr(E->getArg(0));
6260:     auto NewArgT = llvm::PointerType::get(
6261:         getLLVMContext(),
6262:         CGM.getContext().getTargetAddressSpace(LangAS::opencl_generic));
6263:     auto NewRetT = llvm::PointerType::get(
6264:         getLLVMContext(),
6265:         CGM.getContext().getTargetAddressSpace(
6266:             E->getType()->getPointeeType().getAddressSpace()));
6267:     auto FTy = llvm::FunctionType::get(NewRetT, {NewArgT}, false);
6268:     llvm::Value *NewArg;
6269:     if (Arg0->getType()->getPointerAddressSpace() !=
6270:         NewArgT->getPointerAddressSpace())
```
- **EN**: This block defines callable entry points like `string`, `OpenCLRT`, `getLLVMContext`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `string`, `OpenCLRT`, `getLLVMContext`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 6271-6300
```cpp
6271:       NewArg = Builder.CreateAddrSpaceCast(Arg0, NewArgT);
6272:     else
6273:       NewArg = Builder.CreateBitOrPointerCast(Arg0, NewArgT);
6274:     auto NewName = std::string("__") + E->getDirectCallee()->getName().str();
6275:     auto NewCall =
6276:         EmitRuntimeCall(CGM.CreateRuntimeFunction(FTy, NewName), {NewArg});
6277:     return RValue::get(Builder.CreateBitOrPointerCast(NewCall,
6278:       ConvertType(E->getType())));
6279:   }
6280: 
6281:   // OpenCL v2.0, s6.13.17 - Enqueue kernel function.
6282:   // Table 6.13.17.1 specifies four overload forms of enqueue_kernel.
6283:   // The code below expands the builtin call to a call to one of the following
6284:   // functions that an OpenCL runtime library will have to provide:
6285:   //   __enqueue_kernel_basic
6286:   //   __enqueue_kernel_varargs
6287:   //   __enqueue_kernel_basic_events
6288:   //   __enqueue_kernel_events_varargs
6289:   case Builtin::BIenqueue_kernel: {
6290:     StringRef Name; // Generated function call name
6291:     unsigned NumArgs = E->getNumArgs();
6292: 
6293:     llvm::Type *QueueTy = ConvertType(getContext().OCLQueueTy);
6294:     llvm::Type *GenericVoidPtrTy = Builder.getPtrTy(
6295:         getContext().getTargetAddressSpace(LangAS::opencl_generic));
6296: 
6297:     llvm::Value *Queue = EmitScalarExpr(E->getArg(0));
6298:     llvm::Value *Flags = EmitScalarExpr(E->getArg(1));
6299:     LValue NDRangeL = EmitAggExprToLValue(E->getArg(2));
6300:     llvm::Value *Range = NDRangeL.getAddress().emitRawPointer(*this);
```
- **EN**: This block defines callable entry points like `get`, `getContext`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getContext`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 6301-6330
```cpp
6301: 
6302:     // FIXME: Look through the addrspacecast which may exist to the stack
6303:     // temporary as a hack.
6304:     //
6305:     // This is hardcoding the assumed ABI of the target function. This assumes
6306:     // direct passing for every argument except NDRange, which is assumed to be
6307:     // byval or byref indirect passed.
6308:     //
6309:     // This should be fixed to query a signature from CGOpenCLRuntime, and go
6310:     // through EmitCallArgs to get the correct target ABI.
6311:     Range = Range->stripPointerCasts();
6312: 
6313:     llvm::Type *RangePtrTy = Range->getType();
6314: 
6315:     if (NumArgs == 4) {
6316:       // The most basic form of the call with parameters:
6317:       // queue_t, kernel_enqueue_flags_t, ndrange_t, block(void)
6318:       Name = "__enqueue_kernel_basic";
6319:       llvm::Type *ArgTys[] = {QueueTy, Int32Ty, RangePtrTy, GenericVoidPtrTy,
6320:                               GenericVoidPtrTy};
6321:       llvm::FunctionType *FTy = llvm::FunctionType::get(Int32Ty, ArgTys, false);
6322: 
6323:       auto Info =
6324:           CGM.getOpenCLRuntime().emitOpenCLEnqueuedBlock(*this, E->getArg(3));
6325:       llvm::Value *Kernel =
6326:           Builder.CreatePointerCast(Info.KernelHandle, GenericVoidPtrTy);
6327:       llvm::Value *Block =
6328:           Builder.CreatePointerCast(Info.BlockArg, GenericVoidPtrTy);
6329: 
6330:       auto RTCall = EmitRuntimeCall(CGM.CreateRuntimeFunction(FTy, Name),
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 6331-6360
```cpp
6331:                                     {Queue, Flags, Range, Kernel, Block});
6332:       return RValue::get(RTCall);
6333:     }
6334:     assert(NumArgs >= 5 && "Invalid enqueue_kernel signature");
6335: 
6336:     // Create a temporary array to hold the sizes of local pointer arguments
6337:     // for the block. \p First is the position of the first size argument.
6338:     auto CreateArrayForSizeVar =
6339:         [=](unsigned First) -> std::pair<llvm::Value *, llvm::Value *> {
6340:       llvm::APInt ArraySize(32, NumArgs - First);
6341:       QualType SizeArrayTy = getContext().getConstantArrayType(
6342:           getContext().getSizeType(), ArraySize, nullptr,
6343:           ArraySizeModifier::Normal,
6344:           /*IndexTypeQuals=*/0);
6345:       auto Tmp = CreateMemTempWithoutCast(SizeArrayTy, "block_sizes");
6346:       llvm::Value *Alloca = Tmp.getPointer();
6347:       llvm::Value *ElemPtr;
6348:       EmitLifetimeStart(Alloca);
6349:       // Each of the following arguments specifies the size of the corresponding
6350:       // argument passed to the enqueued block.
6351:       auto *Zero = llvm::ConstantInt::get(IntTy, 0);
6352:       for (unsigned I = First; I < NumArgs; ++I) {
6353:         auto *Index = llvm::ConstantInt::get(IntTy, I - First);
6354:         auto *GEP =
6355:             Builder.CreateGEP(Tmp.getElementType(), Alloca, {Zero, Index});
6356:         if (I == First)
6357:           ElemPtr = GEP;
6358:         auto *V =
6359:             Builder.CreateZExtOrTrunc(EmitScalarExpr(E->getArg(I)), SizeTy);
6360:         Builder.CreateAlignedStore(
```
- **EN**: This block defines callable entry points like `get`, `ArraySize`, `getContext`, `EmitLifetimeStart`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`, `ArraySize`, `getContext`, `EmitLifetimeStart`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 6361-6390
```cpp
6361:             V, GEP, CGM.getDataLayout().getPrefTypeAlign(SizeTy));
6362:       }
6363:       return {ElemPtr, Alloca};
6364:     };
6365: 
6366:     // Could have events and/or varargs.
6367:     if (E->getArg(3)->getType()->isBlockPointerType()) {
6368:       // No events passed, but has variadic arguments.
6369:       Name = "__enqueue_kernel_varargs";
6370:       auto Info =
6371:           CGM.getOpenCLRuntime().emitOpenCLEnqueuedBlock(*this, E->getArg(3));
6372:       llvm::Value *Kernel =
6373:           Builder.CreatePointerCast(Info.KernelHandle, GenericVoidPtrTy);
6374:       auto *Block = Builder.CreatePointerCast(Info.BlockArg, GenericVoidPtrTy);
6375:       auto [ElemPtr, TmpPtr] = CreateArrayForSizeVar(4);
6376: 
6377:       // Create a vector of the arguments, as well as a constant value to
6378:       // express to the runtime the number of variadic arguments.
6379:       llvm::Value *const Args[] = {Queue,  Flags,
6380:                                    Range,  Kernel,
6381:                                    Block,  ConstantInt::get(IntTy, NumArgs - 4),
6382:                                    ElemPtr};
6383:       llvm::Type *const ArgTys[] = {
6384:           QueueTy,          IntTy, RangePtrTy,        GenericVoidPtrTy,
6385:           GenericVoidPtrTy, IntTy, ElemPtr->getType()};
6386: 
6387:       llvm::FunctionType *FTy = llvm::FunctionType::get(Int32Ty, ArgTys, false);
6388:       auto Call = RValue::get(
6389:           EmitRuntimeCall(CGM.CreateRuntimeFunction(FTy, Name), Args));
6390:       EmitLifetimeEnd(TmpPtr);
```
- **EN**: This block defines callable entry points like `EmitRuntimeCall`, `EmitLifetimeEnd`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitRuntimeCall`, `EmitLifetimeEnd`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 6391-6420
```cpp
6391:       return Call;
6392:     }
6393:     // Any calls now have event arguments passed.
6394:     if (NumArgs >= 7) {
6395:       llvm::PointerType *PtrTy = llvm::PointerType::get(
6396:           CGM.getLLVMContext(),
6397:           CGM.getContext().getTargetAddressSpace(LangAS::opencl_generic));
6398: 
6399:       llvm::Value *NumEvents =
6400:           Builder.CreateZExtOrTrunc(EmitScalarExpr(E->getArg(3)), Int32Ty);
6401: 
6402:       // Since SemaOpenCLBuiltinEnqueueKernel allows fifth and sixth arguments
6403:       // to be a null pointer constant (including `0` literal), we can take it
6404:       // into account and emit null pointer directly.
6405:       llvm::Value *EventWaitList = nullptr;
6406:       if (E->getArg(4)->isNullPointerConstant(
6407:               getContext(), Expr::NPC_ValueDependentIsNotNull)) {
6408:         EventWaitList = llvm::ConstantPointerNull::get(PtrTy);
6409:       } else {
6410:         EventWaitList =
6411:             E->getArg(4)->getType()->isArrayType()
6412:                 ? EmitArrayToPointerDecay(E->getArg(4)).emitRawPointer(*this)
6413:                 : EmitScalarExpr(E->getArg(4));
6414:         // Convert to generic address space.
6415:         EventWaitList = Builder.CreatePointerCast(EventWaitList, PtrTy);
6416:       }
6417:       llvm::Value *EventRet = nullptr;
6418:       if (E->getArg(5)->isNullPointerConstant(
6419:               getContext(), Expr::NPC_ValueDependentIsNotNull)) {
6420:         EventRet = llvm::ConstantPointerNull::get(PtrTy);
```
- **EN**: This block defines callable entry points like `EmitScalarExpr`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitScalarExpr`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 6421-6450
```cpp
6421:       } else {
6422:         EventRet =
6423:             Builder.CreatePointerCast(EmitScalarExpr(E->getArg(5)), PtrTy);
6424:       }
6425: 
6426:       auto Info =
6427:           CGM.getOpenCLRuntime().emitOpenCLEnqueuedBlock(*this, E->getArg(6));
6428:       llvm::Value *Kernel =
6429:           Builder.CreatePointerCast(Info.KernelHandle, GenericVoidPtrTy);
6430:       llvm::Value *Block =
6431:           Builder.CreatePointerCast(Info.BlockArg, GenericVoidPtrTy);
6432: 
6433:       std::vector<llvm::Type *> ArgTys = {
6434:           QueueTy, Int32Ty, RangePtrTy,       Int32Ty,
6435:           PtrTy,   PtrTy,   GenericVoidPtrTy, GenericVoidPtrTy};
6436: 
6437:       std::vector<llvm::Value *> Args = {Queue,     Flags,         Range,
6438:                                          NumEvents, EventWaitList, EventRet,
6439:                                          Kernel,    Block};
6440: 
6441:       if (NumArgs == 7) {
6442:         // Has events but no variadics.
6443:         Name = "__enqueue_kernel_basic_events";
6444:         llvm::FunctionType *FTy =
6445:             llvm::FunctionType::get(Int32Ty, ArgTys, false);
6446:         return RValue::get(
6447:             EmitRuntimeCall(CGM.CreateRuntimeFunction(FTy, Name), Args));
6448:       }
6449:       // Has event info and variadics
6450:       // Pass the number of variadics to the runtime function too.
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 6451-6480
```cpp
6451:       Args.push_back(ConstantInt::get(Int32Ty, NumArgs - 7));
6452:       ArgTys.push_back(Int32Ty);
6453:       Name = "__enqueue_kernel_events_varargs";
6454: 
6455:       auto [ElemPtr, TmpPtr] = CreateArrayForSizeVar(7);
6456:       Args.push_back(ElemPtr);
6457:       ArgTys.push_back(ElemPtr->getType());
6458: 
6459:       llvm::FunctionType *FTy = llvm::FunctionType::get(Int32Ty, ArgTys, false);
6460:       auto Call = RValue::get(
6461:           EmitRuntimeCall(CGM.CreateRuntimeFunction(FTy, Name), Args));
6462:       EmitLifetimeEnd(TmpPtr);
6463:       return Call;
6464:     }
6465:     llvm_unreachable("Unexpected enqueue_kernel signature");
6466:   }
6467:   // OpenCL v2.0 s6.13.17.6 - Kernel query functions need bitcast of block
6468:   // parameter.
6469:   case Builtin::BIget_kernel_work_group_size: {
6470:     llvm::Type *GenericVoidPtrTy = Builder.getPtrTy(
6471:         getContext().getTargetAddressSpace(LangAS::opencl_generic));
6472:     auto Info =
6473:         CGM.getOpenCLRuntime().emitOpenCLEnqueuedBlock(*this, E->getArg(0));
6474:     Value *Kernel =
6475:         Builder.CreatePointerCast(Info.KernelHandle, GenericVoidPtrTy);
6476:     Value *Arg = Builder.CreatePointerCast(Info.BlockArg, GenericVoidPtrTy);
6477:     return RValue::get(EmitRuntimeCall(
6478:         CGM.CreateRuntimeFunction(
6479:             llvm::FunctionType::get(IntTy, {GenericVoidPtrTy, GenericVoidPtrTy},
6480:                                     false),
```
- **EN**: This block defines callable entry points like `EmitRuntimeCall`, `EmitLifetimeEnd`, `getContext`; uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitRuntimeCall`, `EmitLifetimeEnd`, `getContext`；通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 6481-6510
```cpp
6481:             "__get_kernel_work_group_size_impl"),
6482:         {Kernel, Arg}));
6483:   }
6484:   case Builtin::BIget_kernel_preferred_work_group_size_multiple: {
6485:     llvm::Type *GenericVoidPtrTy = Builder.getPtrTy(
6486:         getContext().getTargetAddressSpace(LangAS::opencl_generic));
6487:     auto Info =
6488:         CGM.getOpenCLRuntime().emitOpenCLEnqueuedBlock(*this, E->getArg(0));
6489:     Value *Kernel =
6490:         Builder.CreatePointerCast(Info.KernelHandle, GenericVoidPtrTy);
6491:     Value *Arg = Builder.CreatePointerCast(Info.BlockArg, GenericVoidPtrTy);
6492:     return RValue::get(EmitRuntimeCall(
6493:         CGM.CreateRuntimeFunction(
6494:             llvm::FunctionType::get(IntTy, {GenericVoidPtrTy, GenericVoidPtrTy},
6495:                                     false),
6496:             "__get_kernel_preferred_work_group_size_multiple_impl"),
6497:         {Kernel, Arg}));
6498:   }
6499:   case Builtin::BIget_kernel_max_sub_group_size_for_ndrange:
6500:   case Builtin::BIget_kernel_sub_group_count_for_ndrange: {
6501:     llvm::Type *GenericVoidPtrTy = Builder.getPtrTy(
6502:         getContext().getTargetAddressSpace(LangAS::opencl_generic));
6503:     LValue NDRangeL = EmitAggExprToLValue(E->getArg(0));
6504:     llvm::Value *NDRange = NDRangeL.getAddress().emitRawPointer(*this);
6505:     auto Info =
6506:         CGM.getOpenCLRuntime().emitOpenCLEnqueuedBlock(*this, E->getArg(1));
6507:     Value *Kernel =
6508:         Builder.CreatePointerCast(Info.KernelHandle, GenericVoidPtrTy);
6509:     Value *Block = Builder.CreatePointerCast(Info.BlockArg, GenericVoidPtrTy);
6510:     const char *Name =
```
- **EN**: This block defines callable entry points like `getContext`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getContext`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 6511-6540
```cpp
6511:         BuiltinID == Builtin::BIget_kernel_max_sub_group_size_for_ndrange
6512:             ? "__get_kernel_max_sub_group_size_for_ndrange_impl"
6513:             : "__get_kernel_sub_group_count_for_ndrange_impl";
6514:     return RValue::get(EmitRuntimeCall(
6515:         CGM.CreateRuntimeFunction(
6516:             llvm::FunctionType::get(
6517:                 IntTy, {NDRange->getType(), GenericVoidPtrTy, GenericVoidPtrTy},
6518:                 false),
6519:             Name),
6520:         {NDRange, Kernel, Block}));
6521:   }
6522:   case Builtin::BI__builtin_store_half:
6523:   case Builtin::BI__builtin_store_halff: {
6524:     CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
6525:     Value *Val = EmitScalarExpr(E->getArg(0));
6526:     Address Address = EmitPointerWithAlignment(E->getArg(1));
6527:     Value *HalfVal = Builder.CreateFPTrunc(Val, Builder.getHalfTy());
6528:     Builder.CreateStore(HalfVal, Address);
6529:     return RValue::get(nullptr);
6530:   }
6531:   case Builtin::BI__builtin_load_half: {
6532:     Address Address = EmitPointerWithAlignment(E->getArg(0));
6533:     Value *HalfVal = Builder.CreateLoad(Address);
6534:     return RValue::get(Builder.CreateFPExt(HalfVal, Builder.getDoubleTy()));
6535:   }
6536:   case Builtin::BI__builtin_load_halff: {
6537:     Address Address = EmitPointerWithAlignment(E->getArg(0));
6538:     Value *HalfVal = Builder.CreateLoad(Address);
6539:     return RValue::get(Builder.CreateFPExt(HalfVal, Builder.getFloatTy()));
6540:   }
```
- **EN**: This block defines callable entry points like `FPOptsRAII`, `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `FPOptsRAII`, `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 6541-6570
```cpp
6541:   case Builtin::BI__builtin_printf:
6542:   case Builtin::BIprintf:
6543:     if (getTarget().getTriple().isNVPTX() ||
6544:         getTarget().getTriple().isAMDGCN() ||
6545:         (getTarget().getTriple().isSPIRV() &&
6546:          getTarget().getTriple().getVendor() == Triple::VendorType::AMD)) {
6547:       if (getTarget().getTriple().isNVPTX())
6548:         return EmitNVPTXDevicePrintfCallExpr(E);
6549:       if ((getTarget().getTriple().isAMDGCN() ||
6550:            getTarget().getTriple().isSPIRV()) &&
6551:           getLangOpts().HIP)
6552:         return EmitAMDGPUDevicePrintfCallExpr(E);
6553:     }
6554: 
6555:     break;
6556:   case Builtin::BI__builtin_canonicalize:
6557:   case Builtin::BI__builtin_canonicalizef:
6558:   case Builtin::BI__builtin_canonicalizef16:
6559:   case Builtin::BI__builtin_canonicalizel:
6560:     return RValue::get(
6561:         emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::canonicalize));
6562: 
6563:   case Builtin::BI__builtin_thread_pointer: {
6564:     if (!getContext().getTargetInfo().isTLSSupported())
6565:       CGM.ErrorUnsupported(E, "__builtin_thread_pointer");
6566: 
6567:     return RValue::get(Builder.CreateIntrinsic(llvm::Intrinsic::thread_pointer,
6568:                                                {GlobalsInt8PtrTy}, {}));
6569:   }
6570:   case Builtin::BI__builtin_os_log_format:
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 6571-6600
```cpp
6571:     return emitBuiltinOSLogFormat(*E);
6572: 
6573:   case Builtin::BI__xray_customevent: {
6574:     if (!ShouldXRayInstrumentFunction())
6575:       return RValue::getIgnored();
6576: 
6577:     if (!CGM.getCodeGenOpts().XRayInstrumentationBundle.has(
6578:             XRayInstrKind::Custom))
6579:       return RValue::getIgnored();
6580: 
6581:     if (const auto *XRayAttr = CurFuncDecl->getAttr<XRayInstrumentAttr>())
6582:       if (XRayAttr->neverXRayInstrument() && !AlwaysEmitXRayCustomEvents())
6583:         return RValue::getIgnored();
6584: 
6585:     Function *F = CGM.getIntrinsic(Intrinsic::xray_customevent);
6586:     auto FTy = F->getFunctionType();
6587:     auto Arg0 = E->getArg(0);
6588:     auto Arg0Val = EmitScalarExpr(Arg0);
6589:     auto Arg0Ty = Arg0->getType();
6590:     auto PTy0 = FTy->getParamType(0);
6591:     if (PTy0 != Arg0Val->getType()) {
6592:       if (Arg0Ty->isArrayType())
6593:         Arg0Val = EmitArrayToPointerDecay(Arg0).emitRawPointer(*this);
6594:       else
6595:         Arg0Val = Builder.CreatePointerCast(Arg0Val, PTy0);
6596:     }
6597:     auto Arg1 = EmitScalarExpr(E->getArg(1));
6598:     auto PTy1 = FTy->getParamType(1);
6599:     if (PTy1 != Arg1->getType())
6600:       Arg1 = Builder.CreateTruncOrBitCast(Arg1, PTy1);
```
- **EN**: This block defines callable entry points like `emitBuiltinOSLogFormat`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitBuiltinOSLogFormat`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 6601-6630
```cpp
6601:     return RValue::get(Builder.CreateCall(F, {Arg0Val, Arg1}));
6602:   }
6603: 
6604:   case Builtin::BI__xray_typedevent: {
6605:     // TODO: There should be a way to always emit events even if the current
6606:     // function is not instrumented. Losing events in a stream can cripple
6607:     // a trace.
6608:     if (!ShouldXRayInstrumentFunction())
6609:       return RValue::getIgnored();
6610: 
6611:     if (!CGM.getCodeGenOpts().XRayInstrumentationBundle.has(
6612:             XRayInstrKind::Typed))
6613:       return RValue::getIgnored();
6614: 
6615:     if (const auto *XRayAttr = CurFuncDecl->getAttr<XRayInstrumentAttr>())
6616:       if (XRayAttr->neverXRayInstrument() && !AlwaysEmitXRayTypedEvents())
6617:         return RValue::getIgnored();
6618: 
6619:     Function *F = CGM.getIntrinsic(Intrinsic::xray_typedevent);
6620:     auto FTy = F->getFunctionType();
6621:     auto Arg0 = EmitScalarExpr(E->getArg(0));
6622:     auto PTy0 = FTy->getParamType(0);
6623:     if (PTy0 != Arg0->getType())
6624:       Arg0 = Builder.CreateTruncOrBitCast(Arg0, PTy0);
6625:     auto Arg1 = E->getArg(1);
6626:     auto Arg1Val = EmitScalarExpr(Arg1);
6627:     auto Arg1Ty = Arg1->getType();
6628:     auto PTy1 = FTy->getParamType(1);
6629:     if (PTy1 != Arg1Val->getType()) {
6630:       if (Arg1Ty->isArrayType())
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 6631-6660
```cpp
6631:         Arg1Val = EmitArrayToPointerDecay(Arg1).emitRawPointer(*this);
6632:       else
6633:         Arg1Val = Builder.CreatePointerCast(Arg1Val, PTy1);
6634:     }
6635:     auto Arg2 = EmitScalarExpr(E->getArg(2));
6636:     auto PTy2 = FTy->getParamType(2);
6637:     if (PTy2 != Arg2->getType())
6638:       Arg2 = Builder.CreateTruncOrBitCast(Arg2, PTy2);
6639:     return RValue::get(Builder.CreateCall(F, {Arg0, Arg1Val, Arg2}));
6640:   }
6641: 
6642:   case Builtin::BI__builtin_ms_va_start:
6643:   case Builtin::BI__builtin_ms_va_end:
6644:     return RValue::get(
6645:         EmitVAStartEnd(EmitMSVAListRef(E->getArg(0)).emitRawPointer(*this),
6646:                        BuiltinID == Builtin::BI__builtin_ms_va_start));
6647: 
6648:   case Builtin::BI__builtin_ms_va_copy: {
6649:     // Lower this manually. We can't reliably determine whether or not any
6650:     // given va_copy() is for a Win64 va_list from the calling convention
6651:     // alone, because it's legal to do this from a System V ABI function.
6652:     // With opaque pointer types, we won't have enough information in LLVM
6653:     // IR to determine this from the argument types, either. Best to do it
6654:     // now, while we have enough information.
6655:     Address DestAddr = EmitMSVAListRef(E->getArg(0));
6656:     Address SrcAddr = EmitMSVAListRef(E->getArg(1));
6657: 
6658:     DestAddr = DestAddr.withElementType(Int8PtrTy);
6659:     SrcAddr = SrcAddr.withElementType(Int8PtrTy);
6660: 
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 6661-6690
```cpp
6661:     Value *ArgPtr = Builder.CreateLoad(SrcAddr, "ap.val");
6662:     return RValue::get(Builder.CreateStore(ArgPtr, DestAddr));
6663:   }
6664: 
6665:   case Builtin::BI__builtin_get_device_side_mangled_name: {
6666:     auto Name = CGM.getCUDARuntime().getDeviceSideName(
6667:         cast<DeclRefExpr>(E->getArg(0)->IgnoreImpCasts())->getDecl());
6668:     auto Str = CGM.GetAddrOfConstantCString(Name, "");
6669:     return RValue::get(Str.getPointer());
6670:   }
6671:   }
6672: 
6673:   // If this is an alias for a lib function (e.g. __builtin_sin), emit
6674:   // the call using the normal call path, but using the unmangled
6675:   // version of the function name.
6676:   const auto &BI = getContext().BuiltinInfo;
6677:   if (!shouldEmitBuiltinAsIR(BuiltinID, BI, *this) &&
6678:       BI.isLibFunction(BuiltinID))
6679:     return emitLibraryCall(*this, FD, E,
6680:                            CGM.getBuiltinLibFunction(FD, BuiltinID));
6681: 
6682:   // If this is a predefined lib function (e.g. malloc), emit the call
6683:   // using exactly the normal call path.
6684:   if (BI.isPredefinedLibFunction(BuiltinID))
6685:     return emitLibraryCall(*this, FD, E, CGM.getRawFunctionPointer(FD));
6686: 
6687:   // Check that a call to a target specific builtin has the correct target
6688:   // features.
6689:   // This is down here to avoid non-target specific builtins, however, if
6690:   // generic builtins start to require generic target features then we
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 6691-6720
```cpp
6691:   // can move this up to the beginning of the function.
6692:   checkTargetFeatures(E, FD);
6693: 
6694:   if (unsigned VectorWidth = getContext().BuiltinInfo.getRequiredVectorWidth(BuiltinID))
6695:     LargestVectorWidth = std::max(LargestVectorWidth, VectorWidth);
6696: 
6697:   // See if we have a target specific intrinsic.
6698:   std::string Name = getContext().BuiltinInfo.getName(BuiltinID);
6699:   Intrinsic::ID IntrinsicID = Intrinsic::not_intrinsic;
6700:   StringRef Prefix =
6701:       llvm::Triple::getArchTypePrefix(getTarget().getTriple().getArch());
6702:   if (!Prefix.empty()) {
6703:     IntrinsicID = Intrinsic::getIntrinsicForClangBuiltin(Prefix.data(), Name);
6704:     if (IntrinsicID == Intrinsic::not_intrinsic && Prefix == "spv" &&
6705:         getTarget().getTriple().getOS() == llvm::Triple::OSType::AMDHSA)
6706:       IntrinsicID = Intrinsic::getIntrinsicForClangBuiltin("amdgcn", Name);
6707:     // NOTE we don't need to perform a compatibility flag check here since the
6708:     // intrinsics are declared in Builtins*.def via LANGBUILTIN which filter the
6709:     // MS builtins via ALL_MS_LANGUAGES and are filtered earlier.
6710:     if (IntrinsicID == Intrinsic::not_intrinsic)
6711:       IntrinsicID = Intrinsic::getIntrinsicForMSBuiltin(Prefix.data(), Name);
6712:   }
6713: 
6714:   if (IntrinsicID != Intrinsic::not_intrinsic) {
6715:     SmallVector<Value*, 16> Args;
6716: 
6717:     // Find out if any arguments are required to be integer constant
6718:     // expressions.
6719:     unsigned ICEArguments = 0;
6720:     ASTContext::GetBuiltinTypeError Error;
```
- **EN**: This block defines callable entry points like `checkTargetFeatures`, `getArchTypePrefix`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `checkTargetFeatures`, `getArchTypePrefix`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 6721-6750
```cpp
6721:     getContext().GetBuiltinType(BuiltinID, Error, &ICEArguments);
6722:     assert(Error == ASTContext::GE_None && "Should not codegen an error");
6723: 
6724:     Function *F = CGM.getIntrinsic(IntrinsicID);
6725:     llvm::FunctionType *FTy = F->getFunctionType();
6726: 
6727:     for (unsigned i = 0, e = E->getNumArgs(); i != e; ++i) {
6728:       Value *ArgValue = EmitScalarOrConstFoldImmArg(ICEArguments, i, E);
6729:       // If the intrinsic arg type is different from the builtin arg type
6730:       // we need to do a bit cast.
6731:       llvm::Type *PTy = FTy->getParamType(i);
6732:       if (PTy != ArgValue->getType()) {
6733:         // XXX - vector of pointers?
6734:         if (auto *PtrTy = dyn_cast<llvm::PointerType>(PTy)) {
6735:           if (PtrTy->getAddressSpace() !=
6736:               ArgValue->getType()->getPointerAddressSpace()) {
6737:             ArgValue = Builder.CreateAddrSpaceCast(
6738:                 ArgValue, llvm::PointerType::get(getLLVMContext(),
6739:                                                  PtrTy->getAddressSpace()));
6740:           }
6741:         }
6742: 
6743:         // Cast vector type (e.g., v256i32) to x86_amx, this only happen
6744:         // in amx intrinsics.
6745:         if (PTy->isX86_AMXTy())
6746:           ArgValue = Builder.CreateIntrinsic(Intrinsic::x86_cast_vector_to_tile,
6747:                                              {ArgValue->getType()}, {ArgValue});
6748:         else
6749:           ArgValue = Builder.CreateBitCast(ArgValue, PTy);
6750:       }
```
- **EN**: This block defines callable entry points like `getContext`, `get`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getContext`, `get`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 6751-6780
```cpp
6751: 
6752:       Args.push_back(ArgValue);
6753:     }
6754: 
6755:     Value *V = Builder.CreateCall(F, Args);
6756:     QualType BuiltinRetType = E->getType();
6757: 
6758:     llvm::Type *RetTy = VoidTy;
6759:     if (!BuiltinRetType->isVoidType())
6760:       RetTy = ConvertType(BuiltinRetType);
6761: 
6762:     if (RetTy != V->getType()) {
6763:       // XXX - vector of pointers?
6764:       if (auto *PtrTy = dyn_cast<llvm::PointerType>(RetTy)) {
6765:         if (PtrTy->getAddressSpace() != V->getType()->getPointerAddressSpace()) {
6766:           V = Builder.CreateAddrSpaceCast(
6767:               V, llvm::PointerType::get(getLLVMContext(),
6768:                                         PtrTy->getAddressSpace()));
6769:         }
6770:       }
6771: 
6772:       // Cast x86_amx to vector type (e.g., v256i32), this only happen
6773:       // in amx intrinsics.
6774:       if (V->getType()->isX86_AMXTy())
6775:         V = Builder.CreateIntrinsic(Intrinsic::x86_cast_tile_to_vector, {RetTy},
6776:                                     {V});
6777:       else
6778:         V = Builder.CreateBitCast(V, RetTy);
6779:     }
6780: 
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 6781-6810
```cpp
6781:     if (RetTy->isVoidTy())
6782:       return RValue::get(nullptr);
6783: 
6784:     return RValue::get(V);
6785:   }
6786: 
6787:   // Some target-specific builtins can have aggregate return values, e.g.
6788:   // __builtin_arm_mve_vld2q_u32. So if the result is an aggregate, force
6789:   // ReturnValue to be non-null, so that the target-specific emission code can
6790:   // always just emit into it.
6791:   TypeEvaluationKind EvalKind = getEvaluationKind(E->getType());
6792:   if (EvalKind == TEK_Aggregate && ReturnValue.isNull()) {
6793:     Address DestPtr = CreateMemTempWithoutCast(E->getType(), "agg.tmp");
6794:     ReturnValue = ReturnValueSlot(DestPtr, false);
6795:   }
6796: 
6797:   // Now see if we can emit a target-specific builtin.
6798:   if (Value *V = EmitTargetBuiltinExpr(BuiltinID, E, ReturnValue)) {
6799:     switch (EvalKind) {
6800:     case TEK_Scalar:
6801:       if (V->getType()->isVoidTy())
6802:         return RValue::get(nullptr);
6803:       return RValue::get(V);
6804:     case TEK_Aggregate:
6805:       return RValue::getAggregate(ReturnValue.getAddress(),
6806:                                   ReturnValue.isVolatile());
6807:     case TEK_Complex:
6808:       llvm_unreachable("No current target builtin returns complex");
6809:     }
6810:     llvm_unreachable("Bad evaluation kind in EmitBuiltinExpr");
```
- **EN**: This block defines callable entry points like `get`, `getAggregate`; uses control flow (if, switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getAggregate`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 6811-6840
```cpp
6811:   }
6812: 
6813:   // EmitHLSLBuiltinExpr will check getLangOpts().HLSL
6814:   if (Value *V = EmitHLSLBuiltinExpr(BuiltinID, E, ReturnValue)) {
6815:     switch (EvalKind) {
6816:     case TEK_Scalar:
6817:       if (V->getType()->isVoidTy())
6818:         return RValue::get(nullptr);
6819:       return RValue::get(V);
6820:     case TEK_Aggregate:
6821:       return RValue::getAggregate(ReturnValue.getAddress(),
6822:                                   ReturnValue.isVolatile());
6823:     case TEK_Complex:
6824:       llvm_unreachable("No current hlsl builtin returns complex");
6825:     }
6826:     llvm_unreachable("Bad evaluation kind in EmitBuiltinExpr");
6827:   }
6828: 
6829:   if (getLangOpts().HIPStdPar && getLangOpts().CUDAIsDevice)
6830:     return EmitHipStdParUnsupportedBuiltin(this, FD);
6831: 
6832:   ErrorUnsupported(E, "builtin function");
6833: 
6834:   // Unknown builtin, for now just dump it out and return undef.
6835:   return GetUndefRValue(E->getType());
6836: }
6837: 
6838: namespace {
6839: struct BuiltinAlignArgs {
6840:   llvm::Value *Src = nullptr;
```
- **EN**: This block introduces declarations such as `BuiltinAlignArgs`; defines callable entry points like `get`, `getAggregate`, `ErrorUnsupported`, `GetUndefRValue`; uses control flow (if, switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `BuiltinAlignArgs` 的声明；定义可调用入口，例如 `get`, `getAggregate`, `ErrorUnsupported`, `GetUndefRValue`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 6841-6870
```cpp
6841:   llvm::Type *SrcType = nullptr;
6842:   llvm::Value *Alignment = nullptr;
6843:   llvm::Value *Mask = nullptr;
6844:   llvm::IntegerType *IntType = nullptr;
6845: 
6846:   BuiltinAlignArgs(const CallExpr *E, CodeGenFunction &CGF) {
6847:     QualType AstType = E->getArg(0)->getType();
6848:     if (AstType->isArrayType())
6849:       Src = CGF.EmitArrayToPointerDecay(E->getArg(0)).emitRawPointer(CGF);
6850:     else
6851:       Src = CGF.EmitScalarExpr(E->getArg(0));
6852:     SrcType = Src->getType();
6853:     if (SrcType->isPointerTy()) {
6854:       IntType = IntegerType::get(
6855:           CGF.getLLVMContext(),
6856:           CGF.CGM.getDataLayout().getIndexTypeSizeInBits(SrcType));
6857:     } else {
6858:       assert(SrcType->isIntegerTy());
6859:       IntType = cast<llvm::IntegerType>(SrcType);
6860:     }
6861:     Alignment = CGF.EmitScalarExpr(E->getArg(1));
6862:     Alignment = CGF.Builder.CreateZExtOrTrunc(Alignment, IntType, "alignment");
6863:     auto *One = llvm::ConstantInt::get(IntType, 1);
6864:     Mask = CGF.Builder.CreateSub(Alignment, One, "mask");
6865:   }
6866: };
6867: } // namespace
6868: 
6869: /// Generate (x & (y-1)) == 0.
6870: RValue CodeGenFunction::EmitBuiltinIsAligned(const CallExpr *E) {
```
- **EN**: This block opens or references namespaces `RValue`; defines callable entry points like `BuiltinAlignArgs`, `EmitBuiltinIsAligned`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块打开或引用命名空间 `RValue`；定义可调用入口，例如 `BuiltinAlignArgs`, `EmitBuiltinIsAligned`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 6871-6900
```cpp
6871:   BuiltinAlignArgs Args(E, *this);
6872:   llvm::Value *SrcAddress = Args.Src;
6873:   if (Args.SrcType->isPointerTy())
6874:     SrcAddress =
6875:         Builder.CreateBitOrPointerCast(Args.Src, Args.IntType, "src_addr");
6876:   return RValue::get(Builder.CreateICmpEQ(
6877:       Builder.CreateAnd(SrcAddress, Args.Mask, "set_bits"),
6878:       llvm::Constant::getNullValue(Args.IntType), "is_aligned"));
6879: }
6880: 
6881: /// Generate (x & ~(y-1)) to align down or ((x+(y-1)) & ~(y-1)) to align up.
6882: /// Note: For pointer types we can avoid ptrtoint/inttoptr pairs by using the
6883: /// llvm.ptrmask intrinsic (with a GEP before in the align_up case).
6884: RValue CodeGenFunction::EmitBuiltinAlignTo(const CallExpr *E, bool AlignUp) {
6885:   BuiltinAlignArgs Args(E, *this);
6886:   llvm::Value *SrcForMask = Args.Src;
6887:   if (AlignUp) {
6888:     // When aligning up we have to first add the mask to ensure we go over the
6889:     // next alignment value and then align down to the next valid multiple.
6890:     // By adding the mask, we ensure that align_up on an already aligned
6891:     // value will not change the value.
6892:     if (Args.Src->getType()->isPointerTy()) {
6893:       if (getLangOpts().PointerOverflowDefined)
6894:         SrcForMask =
6895:             Builder.CreateGEP(Int8Ty, SrcForMask, Args.Mask, "over_boundary");
6896:       else
6897:         SrcForMask = EmitCheckedInBoundsGEP(Int8Ty, SrcForMask, Args.Mask,
6898:                                             /*SignedIndices=*/true,
6899:                                             /*isSubtraction=*/false,
6900:                                             E->getExprLoc(), "over_boundary");
```
- **EN**: This block defines callable entry points like `Args`, `get`, `EmitBuiltinAlignTo`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `Args`, `get`, `EmitBuiltinAlignTo`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 6901-6917
```cpp
6901:     } else {
6902:       SrcForMask = Builder.CreateAdd(SrcForMask, Args.Mask, "over_boundary");
6903:     }
6904:   }
6905:   // Invert the mask to only clear the lower bits.
6906:   llvm::Value *InvertedMask = Builder.CreateNot(Args.Mask, "inverted_mask");
6907:   llvm::Value *Result = nullptr;
6908:   if (Args.Src->getType()->isPointerTy()) {
6909:     Result = Builder.CreateIntrinsic(
6910:         Intrinsic::ptrmask, {Args.SrcType, Args.IntType},
6911:         {SrcForMask, InvertedMask}, nullptr, "aligned_result");
6912:   } else {
6913:     Result = Builder.CreateAnd(SrcForMask, InvertedMask, "aligned_result");
6914:   }
6915:   assert(Result->getType() == Args.SrcType);
6916:   return RValue::get(Result);
6917: }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

## Key Concepts / 关键概念

- **Builtin**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Builder**: Acts as a construction helper that incrementally assembles LLVM IR emission state. / 充当构建辅助器，逐步组装 LLVM IR 生成 状态。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGF**: Central symbol in this file's implementation of LLVM IR emission. / 是该文件实现 LLVM IR 生成 时的核心符号。
- **getArg**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **RValue**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **getType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Intrinsic**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CGBuiltin.h`, `ABIInfo.h`, `CGCUDARuntime.h`, `CGCXXABI.h`, `CGDebugInfo.h`, `CGObjCRuntime.h`, `CGOpenCLRuntime.h`, `CGRecordLayout.h`, and 6 more
- **Clang libraries / Clang 库**: `clang/AST/OSLog.h`, `clang/AST/StmtVisitor.h`, `clang/Basic/DiagnosticFrontend.h`, `clang/Basic/TargetInfo.h`
- **LLVM libraries / LLVM 库**: `llvm/IR/InlineAsm.h`, `llvm/IR/Instruction.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/IntrinsicsX86.h`, `llvm/IR/MatrixBuilder.h`, `llvm/Support/ConvertUTF.h`, `llvm/Support/ScopedPrinter.h`
- **Other headers / 其他头文件**: `optional`, `utility`
