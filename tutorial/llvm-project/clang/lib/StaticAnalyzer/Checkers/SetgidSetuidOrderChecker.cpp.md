# SetgidSetuidOrderChecker.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `clang/lib/StaticAnalyzer/Checkers/SetgidSetuidOrderChecker.cpp`
- **Repository**: `/root/xw/llvm-project` (`llvm-project`)
- **Purpose (EN)**: This file defines a checker to detect possible reversed order of privilege revocations when 'setgid' and 'setuid' is used.
- **Purpose (CN)**: 实现或支撑 `SetgidSetuidOrderChecker` 静态分析检查器的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
   1: //===-- SetgidSetuidOrderChecker.cpp - check privilege revocation calls ---===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: //  This file defines a checker to detect possible reversed order of privilege
  10: //  revocations when 'setgid' and 'setuid' is used.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 14-36
```cpp
  14: #include "clang/StaticAnalyzer/Checkers/BuiltinCheckerRegistration.h"
  15: #include "clang/StaticAnalyzer/Core/BugReporter/BugType.h"
  16: #include "clang/StaticAnalyzer/Core/Checker.h"
  17: #include "clang/StaticAnalyzer/Core/CheckerManager.h"
  18: #include "clang/StaticAnalyzer/Core/PathSensitive/CallDescription.h"
  19: #include "clang/StaticAnalyzer/Core/PathSensitive/CallEvent.h"
  20: #include "clang/StaticAnalyzer/Core/PathSensitive/CheckerContext.h"
  21: #include "clang/StaticAnalyzer/Core/PathSensitive/ProgramState.h"
  22: #include "clang/StaticAnalyzer/Core/PathSensitive/ProgramStateTrait.h"
  23: 
  24: using namespace clang;
  25: using namespace ento;
  26: 
  27: namespace {
  28: 
  29: enum SetPrivilegeFunctionKind { Irrelevant, Setuid, Setgid };
  30: 
  31: class SetgidSetuidOrderChecker : public Checker<check::PostCall, eval::Assume> {
  32:   const BugType BT{this, "Possible wrong order of privilege revocation"};
  33: 
  34:   const CallDescription SetuidDesc{CDM::CLibrary, {"setuid"}, 1};
  35:   const CallDescription SetgidDesc{CDM::CLibrary, {"setgid"}, 1};
  36: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. It introduces or references types such as `SetPrivilegeFunctionKind`, `SetgidSetuidOrderChecker`. Included headers like `BuiltinCheckerRegistration.h`, `BugType.h`, `Checker.h`, `CheckerManager.h` reveal the main APIs consumed by this region. The code explicitly interacts with the Static Analyzer program-state model. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 它引入或引用了诸如 `SetPrivilegeFunctionKind`、`SetgidSetuidOrderChecker` 等类型。 像 `BuiltinCheckerRegistration.h`, `BugType.h`, `Checker.h`, `CheckerManager.h` 这样的头文件说明了该区域依赖的主要 API。 这段代码会显式操作静态分析器的 ProgramState 模型。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

### Lines 37-39
```cpp
  37:   const CallDescription GetuidDesc{CDM::CLibrary, {"getuid"}, 0};
  38:   const CallDescription GetgidDesc{CDM::CLibrary, {"getgid"}, 0};
  39: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 40-44
```cpp
  40:   const CallDescriptionSet OtherSetPrivilegeDesc{
  41:       {CDM::CLibrary, {"seteuid"}, 1},   {CDM::CLibrary, {"setegid"}, 1},
  42:       {CDM::CLibrary, {"setreuid"}, 2},  {CDM::CLibrary, {"setregid"}, 2},
  43:       {CDM::CLibrary, {"setresuid"}, 3}, {CDM::CLibrary, {"setresgid"}, 3}};
  44: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 45-49
```cpp
  45: public:
  46:   void checkPostCall(const CallEvent &Call, CheckerContext &C) const;
  47:   ProgramStateRef evalAssume(ProgramStateRef State, SVal Cond,
  48:                              bool Assumption) const;
  49: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `checkPostCall`, `evalAssume`. The code explicitly interacts with the Static Analyzer program-state model. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `checkPostCall`、`evalAssume`。 这段代码会显式操作静态分析器的 ProgramState 模型。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

### Lines 50-65
```cpp
  50: private:
  51:   void processSetuid(ProgramStateRef State, const CallEvent &Call,
  52:                      CheckerContext &C) const;
  53:   void processSetgid(ProgramStateRef State, const CallEvent &Call,
  54:                      CheckerContext &C) const;
  55:   void processOther(ProgramStateRef State, const CallEvent &Call,
  56:                     CheckerContext &C) const;
  57:   /// Check if a function like \c getuid or \c getgid is called directly from
  58:   /// the first argument of function called from \a Call.
  59:   bool isFunctionCalledInArg(const CallDescription &Desc,
  60:                              const CallEvent &Call) const;
  61:   void emitReport(ProgramStateRef State, CheckerContext &C) const;
  62: };
  63: 
  64: } // end anonymous namespace
  65: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `processSetuid`, `processSetgid`, `processOther`, `isFunctionCalledInArg`. The code explicitly interacts with the Static Analyzer program-state model. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `processSetuid`、`processSetgid`、`processOther`、`isFunctionCalledInArg`。 这段代码会显式操作静态分析器的 ProgramState 模型。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

### Lines 66-77
```cpp
  66: /// Store if there was a call to 'setuid(getuid())' or 'setgid(getgid())' not
  67: /// followed by other different privilege-change functions.
  68: /// If the value \c Setuid is stored and a 'setgid(getgid())' call is found we
  69: /// have found the bug to be reported. Value \c Setgid is used too to prevent
  70: /// warnings at a setgid-setuid-setgid sequence.
  71: REGISTER_TRAIT_WITH_PROGRAMSTATE(LastSetPrivilegeCall, SetPrivilegeFunctionKind)
  72: /// Store the symbol value of the last 'setuid(getuid())' call. This is used to
  73: /// detect if the result is compared to -1 and avoid warnings on that branch
  74: /// (which is the failure branch of the call), and for identification of note
  75: /// tags.
  76: REGISTER_TRAIT_WITH_PROGRAMSTATE(LastSetuidCallSVal, SymbolRef)
  77: 
```
- **EN**: This block uses project macros to register traits, state, or other framework metadata.
- **CN**: 该代码块通过项目宏注册 trait、状态或其他框架元数据。

### Lines 78-89
```cpp
  78: void SetgidSetuidOrderChecker::checkPostCall(const CallEvent &Call,
  79:                                              CheckerContext &C) const {
  80:   ProgramStateRef State = C.getState();
  81:   if (SetuidDesc.matches(Call)) {
  82:     processSetuid(State, Call, C);
  83:   } else if (SetgidDesc.matches(Call)) {
  84:     processSetgid(State, Call, C);
  85:   } else if (OtherSetPrivilegeDesc.contains(Call)) {
  86:     processOther(State, Call, C);
  87:   }
  88: }
  89: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `SetgidSetuidOrderChecker::checkPostCall`, `processSetuid`, `processSetgid`, `processOther`. Conditional branches guard special cases, feature checks, or error paths. The code explicitly interacts with the Static Analyzer program-state model. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `SetgidSetuidOrderChecker::checkPostCall`、`processSetuid`、`processSetgid`、`processOther`。 条件分支用于保护特殊情况、特性检查或错误路径。 这段代码会显式操作静态分析器的 ProgramState 模型。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

### Lines 90-97
```cpp
  90: ProgramStateRef SetgidSetuidOrderChecker::evalAssume(ProgramStateRef State,
  91:                                                      SVal Cond,
  92:                                                      bool Assumption) const {
  93:   SValBuilder &SVB = State->getStateManager().getSValBuilder();
  94:   SymbolRef LastSetuidSym = State->get<LastSetuidCallSVal>();
  95:   if (!LastSetuidSym)
  96:     return State;
  97: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `SetgidSetuidOrderChecker::evalAssume`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. The code explicitly interacts with the Static Analyzer program-state model.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `SetgidSetuidOrderChecker::evalAssume`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 这段代码会显式操作静态分析器的 ProgramState 模型。

### Lines 98-115
```cpp
  98:   // Check if the most recent call to 'setuid(getuid())' is assumed to be != 0.
  99:   // It should be only -1 at failure, but we want to accept a "!= 0" check too.
 100:   // (But now an invalid failure check like "!= 1" will be recognized as correct
 101:   // too. The "invalid failure check" is a different bug that is not the scope
 102:   // of this checker.)
 103:   auto FailComparison =
 104:       SVB.evalBinOpNN(State, BO_NE, nonloc::SymbolVal(LastSetuidSym),
 105:                       SVB.makeIntVal(0, /*isUnsigned=*/false),
 106:                       SVB.getConditionType())
 107:           .getAs<DefinedOrUnknownSVal>();
 108:   if (!FailComparison)
 109:     return State;
 110:   if (auto IsFailBranch = State->assume(*FailComparison);
 111:       IsFailBranch.first && !IsFailBranch.second) {
 112:     // This is the 'setuid(getuid())' != 0 case.
 113:     // On this branch we do not want to emit warning.
 114:     State = State->set<LastSetPrivilegeCall>(Irrelevant);
 115:     State = State->set<LastSetuidCallSVal>(SymbolRef{});
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 116-119
```cpp
 116:   }
 117:   return State;
 118: }
 119: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 120-137
```cpp
 120: void SetgidSetuidOrderChecker::processSetuid(ProgramStateRef State,
 121:                                              const CallEvent &Call,
 122:                                              CheckerContext &C) const {
 123:   bool IsSetuidWithGetuid = isFunctionCalledInArg(GetuidDesc, Call);
 124:   if (State->get<LastSetPrivilegeCall>() != Setgid && IsSetuidWithGetuid) {
 125:     SymbolRef RetSym = Call.getReturnValue().getAsSymbol();
 126:     State = State->set<LastSetPrivilegeCall>(Setuid);
 127:     State = State->set<LastSetuidCallSVal>(RetSym);
 128:     const NoteTag *Note = C.getNoteTag([this,
 129:                                         RetSym](PathSensitiveBugReport &BR) {
 130:       if (!BR.isInteresting(RetSym) || &BR.getBugType() != &this->BT)
 131:         return "";
 132:       return "Call to 'setuid' found here that removes superuser privileges";
 133:     });
 134:     C.addTransition(State, Note);
 135:     return;
 136:   }
 137:   State = State->set<LastSetPrivilegeCall>(Irrelevant);
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `SetgidSetuidOrderChecker::processSetuid`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. The code explicitly interacts with the Static Analyzer program-state model. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `SetgidSetuidOrderChecker::processSetuid`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 这段代码会显式操作静态分析器的 ProgramState 模型。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

### Lines 138-141
```cpp
 138:   State = State->set<LastSetuidCallSVal>(SymbolRef{});
 139:   C.addTransition(State);
 140: }
 141: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 142-159
```cpp
 142: void SetgidSetuidOrderChecker::processSetgid(ProgramStateRef State,
 143:                                              const CallEvent &Call,
 144:                                              CheckerContext &C) const {
 145:   bool IsSetgidWithGetgid = isFunctionCalledInArg(GetgidDesc, Call);
 146:   if (State->get<LastSetPrivilegeCall>() == Setuid) {
 147:     if (IsSetgidWithGetgid) {
 148:       State = State->set<LastSetPrivilegeCall>(Irrelevant);
 149:       emitReport(State, C);
 150:       return;
 151:     }
 152:     State = State->set<LastSetPrivilegeCall>(Irrelevant);
 153:   } else {
 154:     State = State->set<LastSetPrivilegeCall>(IsSetgidWithGetgid ? Setgid
 155:                                                                 : Irrelevant);
 156:   }
 157:   State = State->set<LastSetuidCallSVal>(SymbolRef{});
 158:   C.addTransition(State);
 159: }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `SetgidSetuidOrderChecker::processSetgid`, `emitReport`. Conditional branches guard special cases, feature checks, or error paths. The code explicitly interacts with the Static Analyzer program-state model. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `SetgidSetuidOrderChecker::processSetgid`、`emitReport`。 条件分支用于保护特殊情况、特性检查或错误路径。 这段代码会显式操作静态分析器的 ProgramState 模型。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

### Lines 160-168
```cpp
 160: 
 161: void SetgidSetuidOrderChecker::processOther(ProgramStateRef State,
 162:                                             const CallEvent &Call,
 163:                                             CheckerContext &C) const {
 164:   State = State->set<LastSetuidCallSVal>(SymbolRef{});
 165:   State = State->set<LastSetPrivilegeCall>(Irrelevant);
 166:   C.addTransition(State);
 167: }
 168: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `SetgidSetuidOrderChecker::processOther`. The code explicitly interacts with the Static Analyzer program-state model. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `SetgidSetuidOrderChecker::processOther`。 这段代码会显式操作静态分析器的 ProgramState 模型。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

### Lines 169-176
```cpp
 169: bool SetgidSetuidOrderChecker::isFunctionCalledInArg(
 170:     const CallDescription &Desc, const CallEvent &Call) const {
 171:   if (const auto *CallInArg0 =
 172:           dyn_cast<CallExpr>(Call.getArgExpr(0)->IgnoreParenImpCasts()))
 173:     return Desc.matchesAsWritten(*CallInArg0);
 174:   return false;
 175: }
 176: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `SetgidSetuidOrderChecker::isFunctionCalledInArg`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `SetgidSetuidOrderChecker::isFunctionCalledInArg`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 177-189
```cpp
 177: void SetgidSetuidOrderChecker::emitReport(ProgramStateRef State,
 178:                                           CheckerContext &C) const {
 179:   if (ExplodedNode *N = C.generateNonFatalErrorNode(State)) {
 180:     llvm::StringLiteral Msg =
 181:         "A 'setgid(getgid())' call following a 'setuid(getuid())' "
 182:         "call is likely to fail; probably the order of these "
 183:         "statements is wrong";
 184:     auto Report = std::make_unique<PathSensitiveBugReport>(BT, Msg, N);
 185:     Report->markInteresting(State->get<LastSetuidCallSVal>());
 186:     C.emitReport(std::move(Report));
 187:   }
 188: }
 189: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `SetgidSetuidOrderChecker::emitReport`. Conditional branches guard special cases, feature checks, or error paths. The code explicitly interacts with the Static Analyzer program-state model. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `SetgidSetuidOrderChecker::emitReport`。 条件分支用于保护特殊情况、特性检查或错误路径。 这段代码会显式操作静态分析器的 ProgramState 模型。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

### Lines 190-193
```cpp
 190: void ento::registerSetgidSetuidOrderChecker(CheckerManager &mgr) {
 191:   mgr.registerChecker<SetgidSetuidOrderChecker>();
 192: }
 193: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ento::registerSetgidSetuidOrderChecker`. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ento::registerSetgidSetuidOrderChecker`。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

### Lines 194-196
```cpp
 194: bool ento::shouldRegisterSetgidSetuidOrderChecker(const CheckerManager &mgr) {
 195:   return true;
 196: }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ento::shouldRegisterSetgidSetuidOrderChecker`. Return statements hand back computed state, helper objects, or status values. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ento::shouldRegisterSetgidSetuidOrderChecker`。 返回语句将计算结果、辅助对象或状态值交回调用方。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

## Key Concepts / 关键概念

- **Program-state modeling / ProgramState 建模**: Uses Clang Static Analyzer program-state storage to persist symbolic facts across paths. 使用 Clang Static Analyzer 的 ProgramState 存储在不同路径之间持久化符号事实。
- **Checker callbacks / 检查器回调**: Cooperates with checker callbacks and path-sensitive execution state. 与检查器回调和路径敏感执行状态协同工作。
- **Checker registration / 检查器注册**: Connects logic to Static Analyzer checker registration and dispatch. 将逻辑接入静态分析器的检查器注册与分派机制。
- **Diagnostic reporting / 诊断报告**: Builds bug reports or diagnostics for discovered problems. 为发现的问题构建缺陷报告或诊断信息。
- **Symbolic values / 符号值**: Manipulates symbolic values tracked by the Static Analyzer. 操作由静态分析器跟踪的符号值。
- **Symbol tracking / 符号跟踪**: Tracks symbols that represent abstract runtime objects. 跟踪代表抽象运行时对象的符号。

## Dependencies / 依赖关系

- **Clang / Clang**: `clang/StaticAnalyzer/Checkers/BuiltinCheckerRegistration.h`, `clang/StaticAnalyzer/Core/BugReporter/BugType.h`, `clang/StaticAnalyzer/Core/Checker.h`, `clang/StaticAnalyzer/Core/CheckerManager.h`, `clang/StaticAnalyzer/Core/PathSensitive/CallDescription.h`, `clang/StaticAnalyzer/Core/PathSensitive/CallEvent.h`, `clang/StaticAnalyzer/Core/PathSensitive/CheckerContext.h`, `clang/StaticAnalyzer/Core/PathSensitive/ProgramState.h`, `clang/StaticAnalyzer/Core/PathSensitive/ProgramStateTrait.h`
