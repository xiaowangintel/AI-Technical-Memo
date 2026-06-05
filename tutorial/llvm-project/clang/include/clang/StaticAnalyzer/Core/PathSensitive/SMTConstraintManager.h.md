# SMTConstraintManager.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/StaticAnalyzer/Core/PathSensitive/SMTConstraintManager.h`
- Repository: `llvm-project`
- Purpose (EN): SMTConstraintManager.h -------------------------------------*- C++ -*--==// This file defines a SMT generic API, which will be the base class for every SMT solver specific class.
- 用途（中文）: 该文件为 StaticAnalyzer::Core::PathSensitive 子系统中的 SMT Constraint Manager 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31

```cpp
 1: //== SMTConstraintManager.h -------------------------------------*- C++ -*--==//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: //  This file defines a SMT generic API, which will be the base class for
10: //  every SMT solver specific class.
11: //
12: //===----------------------------------------------------------------------===//
13: 
14: #ifndef LLVM_CLANG_STATICANALYZER_CORE_PATHSENSITIVE_SMTCONSTRAINTMANAGER_H
15: #define LLVM_CLANG_STATICANALYZER_CORE_PATHSENSITIVE_SMTCONSTRAINTMANAGER_H
16: 
17: #include "clang/Basic/JsonSupport.h"
18: #include "clang/Basic/TargetInfo.h"
19: #include "clang/StaticAnalyzer/Core/PathSensitive/BasicValueFactory.h"
20: #include "clang/StaticAnalyzer/Core/PathSensitive/RangedConstraintManager.h"
21: #include "clang/StaticAnalyzer/Core/PathSensitive/SMTConv.h"
22: #include <optional>
23: 
24: typedef llvm::ImmutableSet<
25:     std::pair<clang::ento::SymbolRef, const llvm::SMTExpr *>>
26:     ConstraintSMTType;
27: REGISTER_TRAIT_WITH_PROGRAMSTATE(ConstraintSMT, ConstraintSMTType)
28: 
29: namespace clang {
30: namespace ento {
31: 
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/Basic/JsonSupport.h`, `clang/Basic/TargetInfo.h`, `clang/StaticAnalyzer/Core/PathSensitive/BasicValueFactory.h` and 3 more. It opens, closes, or documents namespace scope for `clang`, `ento`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/Basic/JsonSupport.h`, `clang/Basic/TargetInfo.h`, `clang/StaticAnalyzer/Core/PathSensitive/BasicValueFactory.h` 以及另外 3 项依赖。 它打开、关闭或说明了 `clang`, `ento` 的命名空间作用域。

### Lines 32-62

```cpp
32: class SMTConstraintManager : public clang::ento::SimpleConstraintManager {
33:   mutable llvm::SMTSolverRef Solver = llvm::CreateZ3Solver();
34: 
35: public:
36:   SMTConstraintManager(clang::ento::ExprEngine *EE,
37:                        clang::ento::SValBuilder &SB)
38:       : SimpleConstraintManager(EE, SB) {
39:     Solver->setBoolParam("model", true); // Enable model finding
40:     Solver->setUnsignedParam("timeout", 15000 /*milliseconds*/);
41:   }
42:   virtual ~SMTConstraintManager() = default;
43: 
44:   //===------------------------------------------------------------------===//
45:   // Implementation for interface from SimpleConstraintManager.
46:   //===------------------------------------------------------------------===//
47: 
48:   ProgramStateRef assumeSym(ProgramStateRef State, SymbolRef Sym,
49:                             bool Assumption) override {
50:     ASTContext &Ctx = getBasicVals().getContext();
51: 
52:     QualType RetTy;
53:     bool hasComparison;
54: 
55:     llvm::SMTExprRef Exp =
56:         SMTConv::getExpr(Solver, Ctx, Sym, RetTy, &hasComparison);
57: 
58:     // Create zero comparison for implicit boolean cast, with reversed
59:     // assumption
60:     if (!hasComparison && !RetTy->isBooleanType())
61:       return assumeExpr(
62:           State, Sym,
```
- EN: Key type declarations here include `SMTConstraintManager`. It exposes API surface such as `CreateZ3Solver`, `SimpleConstraintManager`, `setBoolParam`, `setUnsignedParam`.
- 中文: 这里的重要类型声明包括 `SMTConstraintManager`。 它暴露了 `CreateZ3Solver`, `SimpleConstraintManager`, `setBoolParam`, `setUnsignedParam` 等接口。

### Lines 63-93

```cpp
63:           SMTConv::getZeroExpr(Solver, Ctx, Exp, RetTy, !Assumption));
64: 
65:     return assumeExpr(State, Sym, Assumption ? Exp : Solver->mkNot(Exp));
66:   }
67: 
68:   ProgramStateRef assumeSymInclusiveRange(ProgramStateRef State, SymbolRef Sym,
69:                                           const llvm::APSInt &From,
70:                                           const llvm::APSInt &To,
71:                                           bool InRange) override {
72:     ASTContext &Ctx = getBasicVals().getContext();
73:     return assumeExpr(
74:         State, Sym, SMTConv::getRangeExpr(Solver, Ctx, Sym, From, To, InRange));
75:   }
76: 
77:   ProgramStateRef assumeSymUnsupported(ProgramStateRef State, SymbolRef Sym,
78:                                        bool Assumption) override {
79:     // Skip anything that is unsupported
80:     return State;
81:   }
82: 
83:   //===------------------------------------------------------------------===//
84:   // Implementation for interface from ConstraintManager.
85:   //===------------------------------------------------------------------===//
86: 
87:   ConditionTruthVal checkNull(ProgramStateRef State, SymbolRef Sym) override {
88:     ASTContext &Ctx = getBasicVals().getContext();
89: 
90:     QualType RetTy;
91:     // The expression may be casted, so we cannot call getZ3DataExpr() directly
92:     llvm::SMTExprRef VarExp = SMTConv::getExpr(Solver, Ctx, Sym, RetTy);
93:     llvm::SMTExprRef Exp =
```
- EN: It exposes API surface such as `getZeroExpr`, `assumeExpr`, `getBasicVals`, `getRangeExpr`.
- 中文: 它暴露了 `getZeroExpr`, `assumeExpr`, `getBasicVals`, `getRangeExpr` 等接口。

### Lines 94-124

```cpp
 94:         SMTConv::getZeroExpr(Solver, Ctx, VarExp, RetTy, /*Assumption=*/true);
 95: 
 96:     // Negate the constraint
 97:     llvm::SMTExprRef NotExp =
 98:         SMTConv::getZeroExpr(Solver, Ctx, VarExp, RetTy, /*Assumption=*/false);
 99: 
100:     ConditionTruthVal isSat = checkModel(State, Sym, Exp);
101:     ConditionTruthVal isNotSat = checkModel(State, Sym, NotExp);
102: 
103:     // Zero is the only possible solution
104:     if (isSat.isConstrainedTrue() && isNotSat.isConstrainedFalse())
105:       return true;
106: 
107:     // Zero is not a solution
108:     if (isSat.isConstrainedFalse() && isNotSat.isConstrainedTrue())
109:       return false;
110: 
111:     // Zero may be a solution
112:     return ConditionTruthVal();
113:   }
114: 
115:   const llvm::APSInt *getSymVal(ProgramStateRef State,
116:                                 SymbolRef Sym) const override {
117:     BasicValueFactory &BVF = getBasicVals();
118:     ASTContext &Ctx = BVF.getContext();
119: 
120:     if (const SymbolData *SD = dyn_cast<SymbolData>(Sym)) {
121:       QualType Ty = Sym->getType();
122:       assert(!Ty->isRealFloatingType());
123:       llvm::APSInt Value(Ctx.getTypeSize(Ty),
124:                          !Ty->isSignedIntegerOrEnumerationType());
```
- EN: It exposes API surface such as `getZeroExpr`, `checkModel`, `ConditionTruthVal`, `getBasicVals`.
- 中文: 它暴露了 `getZeroExpr`, `checkModel`, `ConditionTruthVal`, `getBasicVals` 等接口。

### Lines 125-155

```cpp
125: 
126:       // TODO: this should call checkModel so we can use the cache, however,
127:       // this method tries to get the interpretation (the actual value) from
128:       // the solver, which is currently not cached.
129: 
130:       llvm::SMTExprRef Exp = SMTConv::fromData(Solver, Ctx, SD);
131: 
132:       Solver->reset();
133:       addStateConstraints(State);
134: 
135:       // Constraints are unsatisfiable
136:       std::optional<bool> isSat = Solver->check();
137:       if (!isSat || !*isSat)
138:         return nullptr;
139: 
140:       // Model does not assign interpretation
141:       if (!Solver->getInterpretation(Exp, Value))
142:         return nullptr;
143: 
144:       // A value has been obtained, check if it is the only value
145:       llvm::SMTExprRef NotExp = SMTConv::fromBinOp(
146:           Solver, Exp, BO_NE,
147:           Ty->isBooleanType() ? Solver->mkBoolean(Value.getBoolValue())
148:                               : Solver->mkBitvector(Value, Value.getBitWidth()),
149:           /*isSigned=*/false);
150: 
151:       Solver->addConstraint(NotExp);
152: 
153:       std::optional<bool> isNotSat = Solver->check();
154:       if (!isNotSat || *isNotSat)
155:         return nullptr;
```
- EN: It exposes API surface such as `fromData`, `reset`, `addStateConstraints`, `check`.
- 中文: 它暴露了 `fromData`, `reset`, `addStateConstraints`, `check` 等接口。

### Lines 156-186

```cpp
156: 
157:       // This is the only solution, store it
158:       return BVF.getValue(Value).get();
159:     }
160: 
161:     if (const SymbolCast *SC = dyn_cast<SymbolCast>(Sym)) {
162:       SymbolRef CastSym = SC->getOperand();
163:       QualType CastTy = SC->getType();
164:       // Skip the void type
165:       if (CastTy->isVoidType())
166:         return nullptr;
167: 
168:       const llvm::APSInt *Value;
169:       if (!(Value = getSymVal(State, CastSym)))
170:         return nullptr;
171:       return BVF.Convert(SC->getType(), *Value).get();
172:     }
173: 
174:     if (const BinarySymExpr *BSE = dyn_cast<BinarySymExpr>(Sym)) {
175:       const llvm::APSInt *LHS, *RHS;
176:       if (const SymIntExpr *SIE = dyn_cast<SymIntExpr>(BSE)) {
177:         LHS = getSymVal(State, SIE->getLHS());
178:         RHS = SIE->getRHS().get();
179:       } else if (const IntSymExpr *ISE = dyn_cast<IntSymExpr>(BSE)) {
180:         LHS = ISE->getLHS().get();
181:         RHS = getSymVal(State, ISE->getRHS());
182:       } else if (const SymSymExpr *SSM = dyn_cast<SymSymExpr>(BSE)) {
183:         // Early termination to avoid expensive call
184:         LHS = getSymVal(State, SSM->getLHS());
185:         RHS = LHS ? getSymVal(State, SSM->getRHS()) : nullptr;
186:       } else {
```
- EN: It exposes API surface such as `getValue`, `getOperand`, `getType`, `Convert`.
- 中文: 它暴露了 `getValue`, `getOperand`, `getType`, `Convert` 等接口。

### Lines 187-217

```cpp
187:         llvm_unreachable("Unsupported binary expression to get symbol value!");
188:       }
189: 
190:       if (!LHS || !RHS)
191:         return nullptr;
192: 
193:       llvm::APSInt ConvertedLHS, ConvertedRHS;
194:       QualType LTy, RTy;
195:       std::tie(ConvertedLHS, LTy) = SMTConv::fixAPSInt(Ctx, *LHS);
196:       std::tie(ConvertedRHS, RTy) = SMTConv::fixAPSInt(Ctx, *RHS);
197:       SMTConv::doIntTypeConversion<llvm::APSInt, &SMTConv::castAPSInt>(
198:           Solver, Ctx, ConvertedLHS, LTy, ConvertedRHS, RTy);
199:       std::optional<APSIntPtr> Res =
200:           BVF.evalAPSInt(BSE->getOpcode(), ConvertedLHS, ConvertedRHS);
201:       return Res ? Res.value().get() : nullptr;
202:     }
203: 
204:     llvm_unreachable("Unsupported expression to get symbol value!");
205:   }
206: 
207:   ProgramStateRef removeDeadBindings(ProgramStateRef State,
208:                                      SymbolReaper &SymReaper) override {
209:     auto CZ = State->get<ConstraintSMT>();
210:     auto &CZFactory = State->get_context<ConstraintSMT>();
211: 
212:     for (const auto &Entry : CZ) {
213:       if (SymReaper.isDead(Entry.first))
214:         CZ = CZFactory.remove(CZ, Entry);
215:     }
216: 
217:     return State->set<ConstraintSMT>(CZ);
```
- EN: It exposes API surface such as `llvm_unreachable`, `tie`, `evalAPSInt`, `get`.
- 中文: 它暴露了 `llvm_unreachable`, `tie`, `evalAPSInt`, `get` 等接口。

### Lines 218-248

```cpp
218:   }
219: 
220:   void printJson(raw_ostream &Out, ProgramStateRef State, const char *NL = "\n",
221:                  unsigned int Space = 0, bool IsDot = false) const override {
222:     ConstraintSMTType Constraints = State->get<ConstraintSMT>();
223: 
224:     Indent(Out, Space, IsDot) << "\"constraints\": ";
225:     if (Constraints.isEmpty()) {
226:       Out << "null," << NL;
227:       return;
228:     }
229: 
230:     ++Space;
231:     Out << '[' << NL;
232:     for (ConstraintSMTType::iterator I = Constraints.begin();
233:          I != Constraints.end(); ++I) {
234:       Indent(Out, Space, IsDot)
235:           << "{ \"symbol\": \"" << I->first << "\", \"range\": \"";
236:       I->second->print(Out);
237:       Out << "\" }";
238: 
239:       if (std::next(I) != Constraints.end())
240:         Out << ',';
241:       Out << NL;
242:     }
243: 
244:     --Space;
245:     Indent(Out, Space, IsDot) << "],";
246:   }
247: 
248:   bool haveEqualConstraints(ProgramStateRef S1,
```
- EN: It exposes API surface such as `get`, `end`, `print`.
- 中文: 它暴露了 `get`, `end`, `print` 等接口。

### Lines 249-279

```cpp
249:                             ProgramStateRef S2) const override {
250:     return S1->get<ConstraintSMT>() == S2->get<ConstraintSMT>();
251:   }
252: 
253:   bool canReasonAbout(SVal X) const override {
254:     const TargetInfo &TI = getBasicVals().getContext().getTargetInfo();
255: 
256:     std::optional<nonloc::SymbolVal> SymVal = X.getAs<nonloc::SymbolVal>();
257:     if (!SymVal)
258:       return true;
259: 
260:     const SymExpr *Sym = SymVal->getSymbol();
261:     QualType Ty = Sym->getType();
262: 
263:     // Complex types are not modeled
264:     if (Ty->isComplexType() || Ty->isComplexIntegerType())
265:       return false;
266: 
267:     // Non-IEEE 754 floating-point types are not modeled
268:     if ((Ty->isSpecificBuiltinType(BuiltinType::LongDouble) &&
269:          (&TI.getLongDoubleFormat() == &llvm::APFloat::x87DoubleExtended() ||
270:           &TI.getLongDoubleFormat() == &llvm::APFloat::PPCDoubleDouble())))
271:       return false;
272: 
273:     if (Ty->isRealFloatingType())
274:       return Solver->isFPSupported();
275: 
276:     if (isa<SymbolData>(Sym))
277:       return true;
278: 
279:     SValBuilder &SVB = getSValBuilder();
```
- EN: It exposes API surface such as `get`, `getBasicVals`, `SymbolVal>`, `getSymbol`.
- 中文: 它暴露了 `get`, `getBasicVals`, `SymbolVal>`, `getSymbol` 等接口。

### Lines 280-310

```cpp
280: 
281:     if (const SymbolCast *SC = dyn_cast<SymbolCast>(Sym))
282:       return canReasonAbout(SVB.makeSymbolVal(SC->getOperand()));
283: 
284:     // UnarySymExpr support is not yet implemented in the Z3 wrapper.
285:     if (isa<UnarySymExpr>(Sym)) {
286:       return false;
287:     }
288: 
289:     if (const BinarySymExpr *BSE = dyn_cast<BinarySymExpr>(Sym)) {
290:       if (const SymIntExpr *SIE = dyn_cast<SymIntExpr>(BSE))
291:         return canReasonAbout(SVB.makeSymbolVal(SIE->getLHS()));
292: 
293:       if (const IntSymExpr *ISE = dyn_cast<IntSymExpr>(BSE))
294:         return canReasonAbout(SVB.makeSymbolVal(ISE->getRHS()));
295: 
296:       if (const SymSymExpr *SSE = dyn_cast<SymSymExpr>(BSE))
297:         return canReasonAbout(SVB.makeSymbolVal(SSE->getLHS())) &&
298:                canReasonAbout(SVB.makeSymbolVal(SSE->getRHS()));
299:     }
300: 
301:     llvm_unreachable("Unsupported expression to reason about!");
302:   }
303: 
304: #if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
305:   /// Dumps SMT formula
306:   LLVM_DUMP_METHOD void dump() const { Solver->dump(); }
307: #endif
308: 
309: protected:
310:   // Check whether a new model is satisfiable, and update the program state.
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It exposes API surface such as `canReasonAbout`, `llvm_unreachable`, `dump`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它暴露了 `canReasonAbout`, `llvm_unreachable`, `dump` 等接口。

### Lines 311-341

```cpp
311:   virtual ProgramStateRef assumeExpr(ProgramStateRef State, SymbolRef Sym,
312:                                      const llvm::SMTExprRef &Exp) {
313:     // Check the model, avoid simplifying AST to save time
314:     if (checkModel(State, Sym, Exp).isConstrainedTrue())
315:       return State->add<ConstraintSMT>(std::make_pair(Sym, Exp));
316: 
317:     return nullptr;
318:   }
319: 
320:   /// Given a program state, construct the logical conjunction and add it to
321:   /// the solver
322:   virtual void addStateConstraints(ProgramStateRef State) const {
323:     // TODO: Don't add all the constraints, only the relevant ones
324:     auto CZ = State->get<ConstraintSMT>();
325:     auto I = CZ.begin(), IE = CZ.end();
326: 
327:     // Construct the logical AND of all the constraints
328:     if (I != IE) {
329:       llvm::SMTExprRef Constraint = I++->second;
330:       while (I != IE) {
331:         Constraint = Solver->mkAnd(Constraint, I++->second);
332:       }
333: 
334:       Solver->addConstraint(Constraint);
335:     }
336:   }
337: 
338:   // Generate and check a Z3 model, using the given constraint.
339:   ConditionTruthVal checkModel(ProgramStateRef State, SymbolRef Sym,
340:                                const llvm::SMTExprRef &Exp) const {
341:     ProgramStateRef NewState =
```
- EN: It exposes API surface such as `add`, `addStateConstraints`, `get`, `begin`.
- 中文: 它暴露了 `add`, `addStateConstraints`, `get`, `begin` 等接口。

### Lines 342-367

```cpp
342:         State->add<ConstraintSMT>(std::make_pair(Sym, Exp));
343: 
344:     llvm::FoldingSetNodeID ID;
345:     NewState->get<ConstraintSMT>().Profile(ID);
346: 
347:     unsigned hash = ID.ComputeHash();
348:     auto I = Cached.find(hash);
349:     if (I != Cached.end())
350:       return I->second;
351: 
352:     Solver->reset();
353:     addStateConstraints(NewState);
354: 
355:     std::optional<bool> res = Solver->check();
356:     return Cached[hash] = res ? ConditionTruthVal(*res) : ConditionTruthVal();
357:   }
358: 
359:   // Cache the result of an SMT query (true, false, unknown). The key is the
360:   // hash of the constraints in a state
361:   mutable llvm::DenseMap<unsigned, ConditionTruthVal> Cached;
362: }; // end class SMTConstraintManager
363: 
364: } // namespace ento
365: } // namespace clang
366: 
367: #endif
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `ento`, `clang`. Key type declarations here include `SMTConstraintManager`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `ento`, `clang` 的命名空间作用域。 这里的重要类型声明包括 `SMTConstraintManager`。

## Key Concepts / 关键概念

- `SMTConstraintManager`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CreateZ3Solver`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `SimpleConstraintManager`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `setBoolParam`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `setUnsignedParam`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `~SMTConstraintManager`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `getBasicVals`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `getExpr`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/Basic/JsonSupport.h`, `clang/Basic/TargetInfo.h`, `clang/StaticAnalyzer/Core/PathSensitive/BasicValueFactory.h`, `clang/StaticAnalyzer/Core/PathSensitive/RangedConstraintManager.h`, `clang/StaticAnalyzer/Core/PathSensitive/SMTConv.h`, `optional`
- Forward declarations / 前向声明: None / 无
- Namespace context / 命名空间上下文: `clang`, `ento`
- Macro-style dependencies / 宏式依赖: `REGISTER_TRAIT_WITH_PROGRAMSTATE`
